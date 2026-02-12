# Enbed SDK

Drop-in UI components for embedding insurance products into your Android, iOS, or Web application.

The Enbed SDK handles the entire purchasing flow — product display, proposal form, payment, and certificate generation — in a single embeddable widget.

---

## Platforms

| Platform | Package | Min Version |
|----------|---------|-------------|
| **Android** | `enbedclient-release.aar` | API 21 (Android 5.0) |
| **iOS** | Swift + WKWebView | iOS 13.0 |
| **Web** | `enbed-product-widget.js` | ES6 browsers |

---

## Integration Flow (All Platforms)

```
1. Generate Auth Token  →  2. Get Proposal Fields  →  3. Initialize Widget
       ↓
4. User Completes Purchase  →  5. Capture Transaction ID
       ↓
6. Trigger Wallet Transaction  →  7. Download Certificate
```

### Key API Endpoints

| Step | Method | Endpoint |
|------|--------|----------|
| Auth Token | POST | `/enbed/v1/auth/generate` |
| Proposal Fields | GET | `/enbed/v1/proposal-fields?productId={id}` |
| Buy via Wallet | POST | `/enbed/v1/products/buy/wallet` |
| Download Certificate | GET | `/enbed/v1/policy-store/{tranId}/certificate/download` |

See the [Enbed API Reference](/api/enbed/v1) for full endpoint documentation.

---

## Android

### Step 1 — Add the AAR

Download `enbedclient-release.aar` and place it in your project's `libs/` directory.

**build.gradle (app):**
```groovy
repositories {
    flatDir {
        dirs 'libs'
    }
}

dependencies {
    implementation(name: 'enbedclient-release', ext: 'aar')
}
```

### Step 2 — Add the Widget to your Layout

```xml
<com.ensuredit.enbedclient.EnbedProduct
    android:id="@+id/enbedProduct"
    android:layout_width="match_parent"
    android:layout_height="wrap_content" />
```

### Step 3 — Initialize

```java
EnbedProduct enbedProduct = findViewById(R.id.enbedProduct);

// Generate auth token first (see API docs)
String authToken = "your_generated_token";

// Build proposal JSON from proposal-fields endpoint
JSONObject proposalData = new JSONObject();
proposalData.put("productId", "PRODUCT_001");
proposalData.put("proposerName", "John Doe");
proposalData.put("proposerDOB", "1990-01-15");
proposalData.put("proposerEmail", "john@example.com");
proposalData.put("proposerPhone", "9876543210");

// Initialize the widget
enbedProduct.init(authToken, proposalData);
```

### Step 4 — Listen for Transaction Completion

```java
enbedProduct.setOnTransactionListener(new EnbedProduct.TransactionListener() {
    @Override
    public void onTransactionComplete(String transactionId) {
        // Transaction successful — use transactionId
        // to trigger wallet transaction and download certificate
        Log.d("Enbed", "Transaction ID: " + transactionId);
    }

    @Override
    public void onTransactionFailed(String error) {
        Log.e("Enbed", "Transaction failed: " + error);
    }
});
```

---

## iOS

### Step 1 — Setup WKWebView

```swift
import UIKit
import WebKit

class EnbedViewController: UIViewController, WKScriptMessageHandler {

    var webView: WKWebView!

    override func viewDidLoad() {
        super.viewDidLoad()

        let config = WKWebViewConfiguration()
        let contentController = WKUserContentController()

        // Register message handler for transaction callbacks
        contentController.add(self, name: "enbedCallback")
        config.userContentController = contentController

        webView = WKWebView(frame: view.bounds, configuration: config)
        view.addSubview(webView)
    }
}
```

### Step 2 — Load the Widget

```swift
func loadEnbedWidget(authToken: String, proposalJSON: String) {
    let baseURL = "https://api.ensuredit.com"
    let widgetURL = "\(baseURL)/enbed/widget"
        + "?token=\(authToken)"
        + "&proposal=\(proposalJSON.addingPercentEncoding(withAllowedCharacters: .urlQueryAllowed)!)"

    if let url = URL(string: widgetURL) {
        webView.load(URLRequest(url: url))
    }
}
```

### Step 3 — Handle Transaction Callbacks

```swift
func userContentController(
    _ userContentController: WKUserContentController,
    didReceive message: WKScriptMessage
) {
    if message.name == "enbedCallback" {
        guard let body = message.body as? [String: Any] else { return }

        if let transactionId = body["transactionId"] as? String {
            print("Transaction complete: \(transactionId)")
            // Trigger wallet transaction and download certificate
        }

        if let error = body["error"] as? String {
            print("Transaction failed: \(error)")
        }
    }
}
```

---

## Web

### Step 1 — Include the Script

```html
<script src="https://api.ensuredit.com/sdk/enbed-product-widget.js"></script>
```

### Step 2 — Add a Container

```html
<div id="enbed-container"></div>
```

### Step 3 — Initialize the Widget

```javascript
// Generate auth token first (see API docs)
const authToken = "your_generated_token";

// Build proposal data from proposal-fields endpoint
const proposalData = {
  productId: "PRODUCT_001",
  proposerName: "John Doe",
  proposerDOB: "1990-01-15",
  proposerEmail: "john@example.com",
  proposerPhone: "9876543210"
};

// Initialize
const enbedWidget = new EnbedProduct({
  container: "#enbed-container",
  token: authToken,
  proposal: proposalData,
  environment: "production",  // or "uat"

  onSuccess: function(transactionId) {
    console.log("Transaction complete:", transactionId);
    // Trigger wallet transaction and download certificate
  },

  onError: function(error) {
    console.error("Transaction failed:", error);
  }
});
```

### Step 4 — Customisation Options

```javascript
const enbedWidget = new EnbedProduct({
  container: "#enbed-container",
  token: authToken,
  proposal: proposalData,
  environment: "production",

  // Theme customisation
  theme: {
    primaryColor: "#4F6EF7",
    fontFamily: "Inter, sans-serif",
    borderRadius: "12px"
  },

  // Callbacks
  onSuccess: function(transactionId) { ... },
  onError: function(error) { ... },
  onClose: function() { ... }
});
```

---

## Post-Transaction Flow

After the widget returns a `transactionId`, complete the purchase:

### 1. Trigger Wallet Transaction

```bash
POST /enbed/v1/products/buy/wallet
Authorization: Bearer <access_token>
Content-Type: application/json

{
  "transactionId": "TXN_123456789"
}
```

### 2. Download Certificate

```bash
GET /enbed/v1/policy-store/TXN_123456789/certificate/download
Authorization: Bearer <access_token>
```

Returns the policy certificate as a PDF file.

---

## Error Handling

| Error Code | Description | Resolution |
|-----------|-------------|------------|
| `AUTH_EXPIRED` | Auth token has expired | Generate a new token via `/auth/generate` |
| `INVALID_PROPOSAL` | Proposal data is incomplete | Check required fields from `/proposal-fields` |
| `PRODUCT_UNAVAILABLE` | Product not available for region | Verify product availability |
| `PAYMENT_FAILED` | Payment processing failed | Retry or use alternate payment method |
| `WIDGET_LOAD_ERROR` | Widget failed to load | Check network and script URL |

---

## Testing

Use the UAT environment for testing:

```
UAT Base URL: https://api-uat.ensuredit.com
```

```javascript
// Web — UAT mode
const enbedWidget = new EnbedProduct({
  container: "#enbed-container",
  token: authToken,
  proposal: proposalData,
  environment: "uat"  // ← switch to UAT
});
```

> **Note:** UAT transactions do not generate real policies or trigger real payments.

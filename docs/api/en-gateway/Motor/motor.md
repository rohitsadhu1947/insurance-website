# Motor Insurance API

End-to-end motor insurance journey — from vehicle details to quote generation, proposal creation, and policy purchase.

---

## Base URL

```
UAT:        https://api-uat.ensuredit.com
Production: https://api.ensuredit.com
```

---

## Available Versions

| Version | Status | Notes |
|---------|--------|-------|
| **v1** | Stable | Complete motor insurance flow |
| **v2** | Latest | Enhanced quote comparison, IDV negotiation |

---

## Flow Overview

The motor insurance journey follows these steps:

```
1. Authenticate  →  2. Vehicle Details  →  3. Get Quote Fields
       ↓
4. Create Quote  →  5. Get Proposal Fields  →  6. Create Proposal
       ↓
7. Payment  →  8. Policy Issued
```

---

## 1. Authentication

<span class="method-post">POST</span> `{{domain}}/login/verifyPassword`

```json
{
  "userId": "your_user_id",
  "password": "your_password"
}
```

**Response** `200 OK`:
```json
{
  "isVerified": true,
  "accessToken": "your_access_token",
  "refreshToken": "your_refresh_token"
}
```

See the [Authentication Guide](/guide/authentication) for details.

---

## 2. Get Quote Fields

<span class="method-get">GET</span> `{{domain}}/preQuote/quoteFields`

Retrieve the dynamic form fields required to generate a motor insurance quote.

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `product` | string | Yes | Insurance product type (`PRIVATE_CAR`, `TWO_WHEELER`, `COMMERCIAL_VEHICLE`) |
| `leadId` | string | Yes | Unique lead identifier |

**Response** `200 OK`:

Returns a `pages` array with dynamic form fields. Each field contains:

- `fieldId` — unique identifier (e.g., `registrationNumber`, `make`, `model`, `variant`, `registrationYear`)
- `viewType` — UI component type (`text`, `autoCompleteText`, `date`, `dropdown`)
- `validation` — regex rules, min/max constraints
- `alignment` — layout hints (col, row, direction)

---

## 3. Vehicle Details Lookup

<span class="method-post">POST</span> `{{domain}}/en-gateway/vehicle/info`

Pre-fill quote form using the vehicle registration number. This step is **optional** but recommended — it fetches make, model, variant, manufacturing year, and existing policy details.

```json
{
  "registrationNumber": "MH01AB1234",
  "consent": "Y",
  "consentText": "I hereby give my consent for data retrieval"
}
```

See the [Vehicle Lookup API](/api/en-gateway/vehicle-lookup/vehicle-lookup) for full response details.

---

## 4. Create Quote

<span class="method-post">POST</span> `{{domain}}/getQuote/{product}`

Submit vehicle and customer details to get insurance quotes from multiple carriers.

**Path Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `product` | string | `PRIVATE_CAR`, `TWO_WHEELER`, or `COMMERCIAL_VEHICLE` |

**Headers:**
```
Authorization: Bearer <access_token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "fieldData": [
    {
      "id": "registrationNumber",
      "value": "MH01AB1234",
      "parentProperty": "vehicleDetails"
    },
    {
      "id": "make",
      "value": "MARUTI SUZUKI",
      "parentProperty": "vehicleDetails"
    },
    {
      "id": "model",
      "value": "SWIFT",
      "parentProperty": "vehicleDetails"
    },
    {
      "id": "variant",
      "value": "VXI 1.2L",
      "parentProperty": "vehicleDetails"
    },
    {
      "id": "registrationYear",
      "value": "2020",
      "parentProperty": "vehicleDetails"
    },
    {
      "id": "previousPolicyType",
      "value": "COMPREHENSIVE",
      "parentProperty": "policyDetails"
    },
    {
      "id": "previousPolicyExpiry",
      "value": "15/03/2025",
      "parentProperty": "policyDetails"
    },
    {
      "id": "claimLastYear",
      "value": "NO",
      "parentProperty": "policyDetails"
    },
    {
      "id": "ncbPercentage",
      "value": "50",
      "parentProperty": "policyDetails"
    },
    {
      "id": "pincode",
      "value": "400001",
      "parentProperty": "communicationAddress"
    }
  ],
  "salesChannelUserId": "2"
}
```

**Response** `200 OK`:
```json
{
  "id": 45210,
  "product": "PRIVATE_CAR",
  "status": "SUCCESS",
  "quotePlans": [
    {
      "companyId": 31,
      "companyName": "ICICI Lombard",
      "planId": 221,
      "planName": "Comprehensive",
      "idv": 485000,
      "ownDamagePremium": 8250,
      "thirdPartyPremium": 2094,
      "netPremium": 10344,
      "gst": 1862,
      "payingAmount": 12206,
      "ncbDiscount": 4125,
      "addOns": [
        {
          "name": "Zero Depreciation",
          "premium": 2100,
          "selected": false
        },
        {
          "name": "Roadside Assistance",
          "premium": 499,
          "selected": false
        },
        {
          "name": "Engine Protection",
          "premium": 1250,
          "selected": false
        }
      ],
      "planData": {
        "paymentModes": [
          { "internalName": "FORM", "displayName": "Payment Gateway" }
        ],
        "brochure": "https://..."
      }
    },
    {
      "companyId": 12,
      "companyName": "HDFC ERGO",
      "planId": 334,
      "planName": "Comprehensive",
      "idv": 478000,
      "payingAmount": 11850,
      "planData": { ... }
    }
  ]
}
```

### Quote Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `quotePlans[].companyId` | integer | Insurance company identifier |
| `quotePlans[].companyName` | string | Insurance company name |
| `quotePlans[].planId` | integer | Plan identifier |
| `quotePlans[].idv` | number | Insured Declared Value (₹) |
| `quotePlans[].ownDamagePremium` | number | OD premium component |
| `quotePlans[].thirdPartyPremium` | number | TP premium component |
| `quotePlans[].netPremium` | number | Net premium before GST |
| `quotePlans[].gst` | number | GST amount (18%) |
| `quotePlans[].payingAmount` | number | Total payable amount |
| `quotePlans[].ncbDiscount` | number | No Claim Bonus discount applied |
| `quotePlans[].addOns` | array | Available add-on covers |

---

## 5. IDV Negotiation <span class="badge-new">v2</span>

<span class="method-post">POST</span> `{{domain}}/quote/{quoteId}/idv`

Adjust the Insured Declared Value (IDV) for a specific quote plan and get recalculated premiums.

**Request Body:**
```json
{
  "quotePlanId": 221,
  "requestedIdv": 520000
}
```

**Response** `200 OK`:
```json
{
  "status": "SUCCESS",
  "quotePlanId": 221,
  "previousIdv": 485000,
  "newIdv": 520000,
  "minIdv": 380000,
  "maxIdv": 560000,
  "revisedPremium": {
    "ownDamagePremium": 8850,
    "netPremium": 10944,
    "gst": 1970,
    "payingAmount": 12914
  }
}
```

---

## 6. Get Proposal Fields

<span class="method-get">GET</span> `{{domain}}/preProposal/proposalFields`

Retrieve form fields required for the proposal step.

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `companyId` | integer | Yes | Insurance company ID from quote |
| `product` | string | Yes | Product type (e.g., `PRIVATE_CAR`) |
| `step` | integer | Yes | Step number in proposal process |
| `sumInsured` | integer | Yes | IDV / coverage amount |
| `quoteId` | integer | Yes | Quote ID from step 4 |
| `quotePlanId` | integer | Yes | Selected plan ID from step 4 |

**Response** `200 OK`:

Returns form pages with fields for:
- Proposer details (name, DOB, gender, contact)
- Communication address
- Vehicle details (engine number, chassis number)
- Nominee details
- Previous policy details

---

## 7. Create Proposal

<span class="method-post">POST</span> `{{domain}}/proposal/{product}/create`

Submit the completed proposal form.

**Request Body:**
```json
{
  "fields": [
    { "id": "proposerTitle", "parentProperty": "proposerDetails", "value": "Mr." },
    { "id": "proposerFullName", "parentProperty": "proposerDetails", "value": "John Doe" },
    { "id": "proposerGender", "parentProperty": "proposerDetails", "value": "Male" },
    { "id": "proposerDOB", "parentProperty": "proposerDetails", "value": "15/01/1990" },
    { "id": "proposerEmail", "parentProperty": "proposerDetails", "value": "john@example.com" },
    { "id": "proposerPhone", "parentProperty": "proposerDetails", "value": "9876543210" },
    { "id": "engineNumber", "parentProperty": "vehicleDetails", "value": "K12MN1234567" },
    { "id": "chassisNumber", "parentProperty": "vehicleDetails", "value": "MA3FJEB1S00123456" },
    { "id": "nomineeName", "parentProperty": "nomineeDetails", "value": "Jane Doe" },
    { "id": "nomineeRelation", "parentProperty": "nomineeDetails", "value": "SPOUSE" },
    { "id": "nomineeDOB", "parentProperty": "nomineeDetails", "value": "20/05/1992" }
  ],
  "quotePlanId": "221",
  "leadId": "29297",
  "salesChannelUserId": "305"
}
```

**Response** `200 OK`:
```json
{
  "id": 31450,
  "product": "PRIVATE_CAR",
  "status": "SUCCESS",
  "paymentData": {
    "modes": [
      { "displayName": "Payment Gateway", "internalName": "FORM" }
    ]
  },
  "proposalId": 31450
}
```

---

## 8. Get Payment Form

<span class="method-get">GET</span> `{{domain}}/proposal/{product}/paymentForm/{leadId}`

Retrieve the payment gateway redirect form.

**Response** `200 OK`:

Returns an HTML form that auto-submits to the payment gateway. The callback URL handles payment confirmation and policy issuance.

---

## Motor Insurance Types

| Product Code | Description |
|-------------|-------------|
| `PRIVATE_CAR` | Personal car insurance |
| `TWO_WHEELER` | Motorcycle / scooter insurance |
| `COMMERCIAL_VEHICLE` | Commercial vehicle insurance |

---

## Policy Types

| Type | Description |
|------|-------------|
| `COMPREHENSIVE` | OD + Third Party coverage |
| `THIRD_PARTY` | Third Party liability only |
| `OD_ONLY` | Own Damage only (standalone) |

---

## Proposal Statuses

| Status | Description |
|--------|-------------|
| `SUCCESS` | Proposal accepted, ready for payment |
| `FAILED` | Proposal rejected — check `insurerSpecificData` for details |
| `PENDING_CKYC` | Awaiting KYC verification |
| `PENDING_INSPECTION` | Vehicle inspection required |
| `PAYMENT_PENDING` | Awaiting payment |
| `POLICY_ISSUED` | Payment successful, policy issued |

---

## NCB (No Claim Bonus) Slabs

| Claim-Free Years | NCB Discount |
|-----------------|--------------|
| 1 year | 20% |
| 2 years | 25% |
| 3 years | 35% |
| 4 years | 45% |
| 5+ years | 50% |

> **Note:** NCB percentages are applied to the Own Damage premium component only.

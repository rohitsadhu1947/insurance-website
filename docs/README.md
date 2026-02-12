# Getting Started

Welcome to the **Ensuredit Developer Documentation**. This guide will help you integrate insurance infrastructure into your platform using our APIs and SDKs.

---

## What is Ensuredit?

Ensuredit is the operating system for insurance — a unified platform that powers quoting, underwriting, policy administration, claims, distribution analytics, and compliance through simple APIs.

Whether you're embedding insurance at checkout, building a full insurance marketplace, or managing distribution across thousands of branches — our APIs handle the heavy lifting.

---

## Quick Links

| Product | Description | Link |
|---------|-------------|------|
| **Enbed API** | Embed insurance products into any checkout flow. Get quotes, create proposals, bind policies. | [View Docs →](/api/enbed/v1) |
| **ICE API** | Full insurance buying journey — health, motor, life. Quotes, proposals, payments. | [View Docs →](/api/ice/Health/v0) |
| **Vehicle Lookup** | Retrieve vehicle registration details, insurance info, and challan data. | [View Docs →](/api/en-gateway/vehicle-lookup/vehicle-lookup) |
| **Motor API** | Motor insurance quoting and policy purchase journey. | [View Docs →](/api/en-gateway/Motor/motor) |
| **Enbed SDK** | Drop-in widget for Android, iOS, and Web. Insurance in 10 lines of code. | [View Docs →](/sdk/enbed/enbed-sdk) |

---

## Authentication

All API requests require a Bearer token. Generate one using your credentials:

```bash
curl -X POST https://api-uat.ensuredit.com/enbed/v1/auth/generate \
  -H "Content-Type: application/json" \
  -d '{"username": "your_username", "password": "your_password"}'
```

**Response:**
```json
{
  "accessToken": "eyJhbGciOiJSUz...",
  "expiresIn": 1800,
  "refreshToken": "eyJhbGciOiJSUz..."
}
```

Use the `accessToken` as a Bearer token in all subsequent requests:

```
Authorization: Bearer eyJhbGciOiJSUz...
```

> **Note:** Tokens expire after 30 minutes. Use the `refreshToken` to generate a new access token without re-authenticating.

---

## Environments

| Environment | Base URL | Purpose |
|-------------|----------|---------|
| **UAT (Sandbox)** | `https://api-uat.ensuredit.com` | Testing and development |
| **Production** | `https://api.ensuredit.com` | Live transactions |

> Contact us at [hello@ensuredit.com](mailto:hello@ensuredit.com) to get your credentials for either environment.

---

## Error Handling

All errors follow a consistent format:

```json
{
  "message": "Description of what went wrong"
}
```

| HTTP Code | Meaning |
|-----------|---------|
| `200` | Success |
| `400` | Bad Request — check your parameters |
| `401` | Unauthorized — invalid or expired token |
| `404` | Not Found — resource doesn't exist |
| `409` | Conflict — precondition failed |
| `500` | Internal Error — contact support |

---

## Need Help?

- **Email:** [hello@ensuredit.com](mailto:hello@ensuredit.com)
- **Website:** [ensuredit.com](https://www.ensuredit.com)
- **LinkedIn:** [linkedin.com/company/ensureditoff](https://www.linkedin.com/company/ensureditoff)

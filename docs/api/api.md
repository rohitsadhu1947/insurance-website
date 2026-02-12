# API Reference

Ensuredit provides REST APIs for integrating insurance infrastructure into your platform.

---

## Available APIs

| API | Description | Status |
|-----|-------------|--------|
| [**Enbed v1**](/api/enbed/v1) | Embedded insurance — products, proposals, purchase, cancellation | Live |
| [**ICE — Health v0**](/api/ice/Health/v0) | Health insurance buying journey — quotes, proposals, payments | Live |
| [**Vehicle Lookup**](/api/en-gateway/vehicle-lookup/vehicle-lookup) | Vehicle registration, insurance info, challan data | Live |
| [**Motor Insurance**](/api/en-gateway/Motor/motor) | Motor insurance quoting and purchase | Live |

---

## Base URL

```
UAT:        https://api-uat.ensuredit.com
Production: https://api.ensuredit.com
```

## Authentication

All APIs require a Bearer token. See the [Authentication Guide](/guide/authentication) for details.

```
Authorization: Bearer <your_access_token>
```

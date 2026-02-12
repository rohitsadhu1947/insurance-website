# Vehicle Info & Challan Lookup API

Retrieve vehicle registration details, insurance information, and traffic violation (challan) data through a unified API.

---

## Base URL

```
UAT:        https://api-uat.ensuredit.com
Production: https://api.ensuredit.com
```

---

## Overview

The Vehicle Lookup API enables authorised platforms to query:

- **Vehicle registration data** — owner details, registration date, vehicle specifications
- **Insurance status** — active policy info, insurer name, expiry date
- **Challan records** — outstanding traffic violations, offence details, payment status

Ideal for insurance companies, lending platforms, fleet management tools, and government agencies.

---

## Available Versions

| Version | Status | Notes |
|---------|--------|-------|
| **v0** | Stable | Basic vehicle registration lookup |
| **v1** | Stable | Added insurance status & challan support |
| **v2** | Latest | Enhanced response format, bulk lookup |

---

## Authentication

All endpoints require a Bearer token. See the [Authentication Guide](guide/authentication).

```
Authorization: Bearer <access_token>
Content-Type: application/json
```

---

## 1. Vehicle Registration Lookup

<span class="method-post">POST</span> `{{domain}}/en-gateway/vehicle/info`

Retrieve detailed registration information for a vehicle by its registration number.

**Request Body:**
```json
{
  "registrationNumber": "MH01AB1234",
  "consent": "Y",
  "consentText": "I hereby give my consent for data retrieval"
}
```

**Response** `200 OK`:
```json
{
  "status": "SUCCESS",
  "data": {
    "registrationNumber": "MH01AB1234",
    "registrationDate": "2020-03-15",
    "ownerName": "JOHN DOE",
    "vehicleClass": "MOTOR CAR",
    "fuelType": "PETROL",
    "makerModel": "MARUTI SUZUKI SWIFT VXI",
    "manufacturingYear": 2020,
    "engineNumber": "K12MN1234567",
    "chassisNumber": "MA3FJEB1S00123456",
    "fitnessUpto": "2035-03-14",
    "insuranceCompany": "ICICI LOMBARD",
    "insuranceUpto": "2025-03-14",
    "insurancePolicyNumber": "3001/12345678/00/000",
    "rtoName": "MUMBAI SOUTH",
    "state": "MAHARASHTRA",
    "vehicleColor": "WHITE",
    "seatingCapacity": 5,
    "cubicCapacity": 1197,
    "grossWeight": 1150,
    "status": "ACTIVE"
  }
}
```

### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `registrationNumber` | string | Vehicle registration number |
| `registrationDate` | string | Date of registration (YYYY-MM-DD) |
| `ownerName` | string | Registered owner name |
| `vehicleClass` | string | Vehicle classification |
| `fuelType` | string | Fuel type (PETROL, DIESEL, EV, CNG) |
| `makerModel` | string | Manufacturer and model |
| `manufacturingYear` | integer | Year of manufacture |
| `engineNumber` | string | Engine number |
| `chassisNumber` | string | Chassis number |
| `fitnessUpto` | string | Fitness certificate validity |
| `insuranceCompany` | string | Current insurer name |
| `insuranceUpto` | string | Insurance policy expiry date |
| `insurancePolicyNumber` | string | Active policy number |
| `rtoName` | string | Regional Transport Office name |
| `status` | string | Vehicle registration status |

---

## 2. Challan Lookup

<span class="method-post">POST</span> `{{domain}}/en-gateway/vehicle/challan`

Retrieve outstanding traffic violation records (challans) for a vehicle.

**Request Body:**
```json
{
  "registrationNumber": "MH01AB1234",
  "consent": "Y",
  "consentText": "I hereby give my consent for data retrieval"
}
```

**Response** `200 OK`:
```json
{
  "status": "SUCCESS",
  "data": {
    "registrationNumber": "MH01AB1234",
    "totalChallans": 2,
    "totalPendingAmount": 2500,
    "challans": [
      {
        "challanNumber": "CH20240315001",
        "challanDate": "2024-03-15",
        "offenceDetails": "Driving without seatbelt",
        "amount": 1000,
        "status": "PENDING",
        "issuingAuthority": "MUMBAI TRAFFIC POLICE",
        "location": "WESTERN EXPRESS HIGHWAY"
      },
      {
        "challanNumber": "CH20240220002",
        "challanDate": "2024-02-20",
        "offenceDetails": "Overspeeding",
        "amount": 1500,
        "status": "PENDING",
        "issuingAuthority": "MUMBAI TRAFFIC POLICE",
        "location": "BANDRA-WORLI SEA LINK"
      }
    ]
  }
}
```

### Challan Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `totalChallans` | integer | Number of challans found |
| `totalPendingAmount` | number | Total unpaid challan amount (₹) |
| `challans[].challanNumber` | string | Unique challan identifier |
| `challans[].challanDate` | string | Date of violation |
| `challans[].offenceDetails` | string | Description of the offence |
| `challans[].amount` | number | Fine amount (₹) |
| `challans[].status` | string | PENDING / PAID / DISPOSED |
| `challans[].issuingAuthority` | string | Authority that issued the challan |
| `challans[].location` | string | Location of the offence |

---

## 3. Bulk Vehicle Lookup <span class="badge-new">v2</span>

<span class="method-post">POST</span> `{{domain}}/en-gateway/vehicle/bulk-info`

Look up registration details for multiple vehicles in a single request. Maximum **50 vehicles** per request.

**Request Body:**
```json
{
  "registrationNumbers": [
    "MH01AB1234",
    "DL01CD5678",
    "KA03EF9012"
  ],
  "consent": "Y",
  "consentText": "I hereby give my consent for data retrieval"
}
```

**Response** `200 OK`:
```json
{
  "status": "SUCCESS",
  "totalRequested": 3,
  "totalFound": 3,
  "results": [
    {
      "registrationNumber": "MH01AB1234",
      "status": "FOUND",
      "data": { ... }
    },
    {
      "registrationNumber": "DL01CD5678",
      "status": "FOUND",
      "data": { ... }
    }
  ]
}
```

---

## Error Responses

| Status Code | Error | Description |
|-------------|-------|-------------|
| `400` | `INVALID_REGISTRATION` | Registration number format is invalid |
| `401` | `UNAUTHORIZED` | Missing or expired access token |
| `404` | `VEHICLE_NOT_FOUND` | No vehicle found for the given registration |
| `429` | `RATE_LIMIT_EXCEEDED` | Too many requests — retry after cooldown |
| `500` | `INTERNAL_ERROR` | Server error — contact support |

**Error Response Format:**
```json
{
  "status": "FAILED",
  "error": {
    "code": "INVALID_REGISTRATION",
    "message": "Registration number must be in the format: XX00XX0000"
  }
}
```

---

## Rate Limits

| Plan | Requests / minute | Bulk limit |
|------|--------------------|------------|
| Standard | 60 | 50 vehicles |
| Enterprise | 300 | 200 vehicles |

---

## Registration Number Format

The API accepts Indian vehicle registration numbers in the following formats:

```
MH01AB1234    — Standard format
MH 01 AB 1234 — With spaces (auto-cleaned)
mh01ab1234    — Case insensitive
```

> **Note:** Special registration formats (diplomatic, defence) may not be available in all versions.

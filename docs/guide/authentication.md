# Authentication

All Ensuredit APIs require authentication via Bearer tokens. This guide covers how to generate, use, and refresh tokens.

---

## Generate a Token

<span class="method-post">POST</span> `/auth/generate`

Send your credentials to receive an access token:

```bash
curl -X POST https://api-uat.ensuredit.com/enbed/v1/auth/generate \
  -H "Content-Type: application/json" \
  -d '{
    "username": "your_username",
    "password": "your_password"
  }'
```

**Response:**

```json
{
  "accessToken": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
  "expiresIn": 1800,
  "refreshExpiresIn": 863999,
  "refreshToken": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

| Field | Type | Description |
|-------|------|-------------|
| `accessToken` | string | Bearer token for API requests |
| `expiresIn` | integer | Token validity in seconds (30 min) |
| `refreshToken` | string | Token to generate new access tokens |
| `refreshExpiresIn` | integer | Refresh token validity in seconds (~10 days) |

---

## Use the Token

Include the token in the `Authorization` header of every API request:

```
Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...
```

---

## Token Expiry

- **Access tokens** expire after **30 minutes**
- **Refresh tokens** expire after **~10 days**
- When your access token expires, use the refresh token to get a new one without re-authenticating

> **Tip:** Implement token refresh logic in your integration to avoid authentication errors during long-running processes.

---

## Getting Credentials

Contact us to obtain your API credentials:

- **Email:** [hello@ensuredit.com](mailto:hello@ensuredit.com)
- **UAT credentials** are provided for testing
- **Production credentials** are issued after integration review

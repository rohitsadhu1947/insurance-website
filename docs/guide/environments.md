# Environments

Ensuredit provides two environments for API access.

---

## Available Environments

| Environment | Base URL | Purpose |
|-------------|----------|---------|
| **UAT (Sandbox)** | `https://api-uat.ensuredit.com` | Testing and development |
| **Production** | `https://api.ensuredit.com` | Live transactions |

---

## UAT (Sandbox)

Use this environment during development and testing. All transactions are simulated — no real policies are created and no real payments are processed.

- Safe to test with dummy data
- Same API structure as production
- Responses may contain test/mock data

---

## Production

Live environment for real transactions. Policies created here are binding and payments are real.

- Requires production credentials (separate from UAT)
- Subject to rate limits
- Full audit logging enabled

> **Important:** Never use production credentials in client-side code or public repositories.

---

## Switching Environments

Simply change the base URL in your API calls. The endpoint paths remain the same.

```bash
# UAT
https://api-uat.ensuredit.com/enbed/v1/products/buyable

# Production
https://api.ensuredit.com/enbed/v1/products/buyable
```

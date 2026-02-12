# Support

Need help with your integration? We're here for you.

---

## Contact

| Channel | Details |
|---------|---------|
| **Email** | [hello@ensuredit.com](mailto:hello@ensuredit.com) |
| **Website** | [ensuredit.com](https://www.ensuredit.com) |
| **LinkedIn** | [linkedin.com/company/ensureditoff](https://www.linkedin.com/company/ensureditoff) |

---

## Before Reaching Out

To help us resolve your issue faster, please include:

1. **API endpoint** you're calling
2. **Request body** (with sensitive data redacted)
3. **Response** you're receiving (status code + body)
4. **Environment** (UAT or Production)
5. **Timestamp** of the failed request

---

## Common Issues

### "Token has expired"
Your access token is only valid for 30 minutes. Implement token refresh using the `refreshToken` from your auth response.

### Getting 404 on valid endpoints
Double-check your base URL matches the environment you're targeting. UAT and Production use different domains.

### Empty product list
Ensure your account has been configured with the right product catalog. Contact us to verify your account setup.

---

## Request a Demo

Want to see the full platform in action? [Request a demo →](https://www.ensuredit.com/contact.html)

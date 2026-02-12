# Error Handling

All Ensuredit APIs return errors in a consistent format.

---

## Error Response Format

```json
{
  "message": "Description of what went wrong"
}
```

---

## HTTP Status Codes

| Code | Status | Description |
|------|--------|-------------|
| `200` | OK | Request succeeded |
| `400` | Bad Request | Invalid parameters or malformed request body |
| `401` | Unauthorized | Missing, invalid, or expired authentication token |
| `404` | Not Found | The requested resource does not exist |
| `409` | Conflict | Precondition failed (e.g., duplicate policy, invalid state transition) |
| `422` | Unprocessable | Request is valid but cannot be processed (e.g., validation errors) |
| `429` | Too Many Requests | Rate limit exceeded — slow down |
| `500` | Internal Error | Something went wrong on our end — contact support |

---

## Common Errors

### Authentication Errors

```json
// Expired token
{ "message": "Token has expired" }

// Invalid credentials
{ "message": "Please enter valid credentials" }
```

**Fix:** Refresh your token or re-authenticate.

### Validation Errors

```json
// Missing required field
{ "message": "product_id is required" }

// Invalid format
{ "message": "Invalid date format. Expected DD/MM/YYYY" }
```

**Fix:** Check your request body against the API reference.

---

## Best Practices

1. **Always check the HTTP status code** before parsing the response body
2. **Implement retry logic** for 5xx errors with exponential backoff
3. **Don't retry** 4xx errors — fix the request instead
4. **Log error responses** for debugging
5. **Handle token expiry** gracefully with automatic refresh

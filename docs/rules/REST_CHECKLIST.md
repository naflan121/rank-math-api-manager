# REST Change Checklist

Use this checklist for any endpoint change:

- new endpoint
- modified route
- changed authentication or authorization
- changed request payload
- changed response payload

Location:

- link or copy this checklist in the PR description
- reference it from `.github/PULL_REQUEST_TEMPLATE.md`

## Required PR Content

### 1. Capability Review

- Purpose of the endpoint change:
- Why the change is needed now:
- Who should be allowed to call it:

### 2. Input Validation

List every input field and its rules:

| Field | Type | Required | Validation |
| --- | --- | --- | --- |
| `field_name` | `string` | `yes/no` | `length, enum, format, range, object existence` |

### 3. Sanitization And Escaping

For each user-supplied field, state:

- how it is sanitized before processing
- how it is sanitized before storage
- how it is escaped before output, if returned or rendered

### 4. Response Shape Review

Document the implemented response contract:

- success status code and JSON shape
- error status codes and JSON shape
- one success example
- one error example

### 5. Manual HTTP Verification

Include at least one runnable manual HTTP example that exercises the changed endpoint and shows the expected response.

Required format:

```bash
curl --request POST "https://example.com/wp-json/your-namespace/v1/route" \
  --header "Content-Type: application/json" \
  --user "user:app-password" \
  --data '{}'
```

## Reviewer Verification

- Confirm the PR description includes all five sections above.
- Confirm the HTTP example matches the implemented route, auth model, payload, and response.
- Confirm the described validation and sanitization rules are implemented in code or tracked explicitly in follow-up work.
- Confirm `docs/rules/SECURITY_CHECKLIST.md` is completed for the same endpoint change.

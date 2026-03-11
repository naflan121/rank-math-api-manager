## Summary

- What changed:
- Why:

## Endpoint Change Details

If this PR changes a route, auth model, request payload, or response payload, complete all sections below. Otherwise write `N/A`.

### Capability Review

- Purpose:
- Why needed:
- Allowed callers:

### Input Validation

| Field | Type | Required | Validation |
| --- | --- | --- | --- |
| `field_name` | `string` | `yes/no` | `length, enum, format, range, object existence` |

### Sanitization And Escaping

- Input sanitization:
- Storage sanitization:
- Output escaping:

### Response Shape Review

- Success status and shape:
- Error status and shape:

Success example:

```json
{}
```

Error example:

```json
{}
```

### Manual HTTP Verification

```bash
curl --request POST "https://example.com/wp-json/your-namespace/v1/route" \
  --header "Content-Type: application/json" \
  --user "user:app-password" \
  --data '{}'
```

Expected response:

```json
{}
```

## Security Checklist

Complete this section for any endpoint change. Otherwise write `N/A`.

- Threat model:
- Authentication and authorization:
- Rate limiting and abuse mitigation:
- Data exposure and PII review:
- Input validation and sanitization summary:
- Logging and error handling:

## Docs And Tests

- Updated docs:
- Updated tests or manual verification:
- Migration note, if needed:

## Reviewer Checklist

- [ ] Forbidden-files check completed
- [ ] REST checklist completed
- [ ] Release packaging exclusions verified
- [ ] Agent scope confirmed
- [ ] Security checklist completed
- [ ] Documentation and test examples updated for route/auth/payload change

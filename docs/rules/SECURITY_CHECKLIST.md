# New Endpoint Security Checklist

Use this checklist for any new endpoint or any change to endpoint auth, payload, storage, or returned data.

Location:

- complete it in the PR description, or
- link to a completed copy from the PR description

## Required Security Notes

### 1. Threat Model

- Relevant threats:
- Why those threats apply here:
- Mitigations implemented:

### 2. Authentication And Authorization

- Who can call the endpoint:
- How identity is enforced:
- Which capability or object-level permission check is used:

### 3. Rate Limiting And Abuse Mitigation

- Existing limit or mitigation:
- If no rate limiting is implemented, state why risk is acceptable or what follow-up is required:

### 4. Data Exposure

- Data returned:
- Data stored:
- PII or sensitive data considerations:
- Fields intentionally excluded from responses or logs:

### 5. Input Validation And Sanitization

- Reference the completed `docs/rules/REST_CHECKLIST.md` section:
- Summarize any field-specific security handling:

### 6. Logging And Error Handling

- What is logged:
- What is redacted:
- What clients see on failure:

## Reviewer Verification

- Confirm all six sections are present.
- Confirm auth, permission, logging, and data-exposure statements match the implementation.
- Confirm sensitive values are not logged or echoed back unnecessarily.
- Confirm unresolved risks are tracked explicitly before approval.

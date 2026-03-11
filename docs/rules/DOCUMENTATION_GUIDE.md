# Documentation Maintenance Rule

When a route, authentication method, or payload changes, update the related documentation and test examples in the same task or pull request.

Location:

- implementation review guidance: this file
- reviewer confirmation: `.github/PULL_REQUEST_TEMPLATE.md`

## Required Actions

- update the user-facing or contributor-facing docs that describe the changed route, auth, or payload
- update the matching HTTP examples
- update tests, fixtures, or manual verification steps that depend on the changed behavior
- if a breaking change remains, add a short migration note in the same PR

## Typical Files To Review

- `README.md`
- `readme.txt`
- `docs/api-documentation.md`
- `docs/integration-guide.md`
- `docs/example-use-cases.md`
- tests or manual verification notes that cover the changed endpoint

## Reviewer Verification

- Confirm docs and examples changed in the same PR as the implementation, or confirm the PR includes a clear migration note explaining why not.
- Confirm tests or manual verification examples were updated to match the implemented behavior.
- Confirm the PR checklist line for docs and tests is checked.

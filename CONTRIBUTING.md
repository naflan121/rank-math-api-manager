# Contributing

This repository ships a public WordPress plugin. Local development aids must stay out of commits, pull requests, and release artifacts unless the project owner explicitly asks for them.

## Private-Only Guardrail

Rule:

Do not commit `.cursor/`, `agent-skills/`, `*.code-workspace`, `transcripts`, local notes, or other local agent artifacts unless explicitly requested by the project owner.

Scope:

- all commits
- all pull requests
- all automated suggestions
- all release packages

Reviewer verification:

- Check the PR file list for forbidden paths before approval.
- Block the PR if any forbidden path is present without explicit owner approval.
- Confirm the PR checklist line for forbidden files is checked.

## Review Documents

Use these documents when reviewing or preparing changes:

- `docs/rules/REST_CHECKLIST.md`
- `docs/rules/SECURITY_CHECKLIST.md`
- `docs/rules/PACKAGING.md`
- `docs/rules/AGENT_SCOPE.md`
- `docs/rules/DOCUMENTATION_GUIDE.md`

Reviewer verification:

- Confirm the PR template sections that apply to the change are filled out.
- If an endpoint, auth flow, payload, packaging rule, or agent doc changes, confirm the matching guidance file is followed or updated in the same PR.

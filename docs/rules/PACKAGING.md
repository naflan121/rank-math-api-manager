# Packaging Rules

This repository must not ship local development artifacts in git or in plugin release ZIP files.

Location:

- repository ignore rules: `.gitignore`
- release reviewer guidance: this file
- release PR confirmation: `.github/PULL_REQUEST_TEMPLATE.md`

## Forbidden In Git And Release Artifacts

- `.cursor/`
- `agent-skills/`
- `*.code-workspace`
- `transcripts/`
- `agent-transcripts/`
- `local-notes/`
- `.notes/`
- other local agent artifacts or workspace-only notes

## Required Actions

- Keep forbidden local artifacts in `.gitignore`.
- Keep release packaging instructions and workflow checks aligned with this file.
- Verify release ZIP contents before publishing.

## Reviewer Verification

Repository check:

```bash
git ls-files | rg '(^|/)\.cursor/|(^|/)agent-skills/|\.code-workspace$|(^|/)transcripts/|(^|/)agent-transcripts/|(^|/)local-notes/|(^|/)\.notes/'
```

Passing result:

- no output

Release ZIP check:

```bash
unzip -Z1 rank-math-api-manager.zip | rg '(^|/)\.cursor/|(^|/)agent-skills/|\.code-workspace$|(^|/)transcripts/|(^|/)agent-transcripts/|(^|/)local-notes/|(^|/)\.notes/'
```

Passing result:

- no output

Release PR confirmation:

- add one line confirming packaging exclusions were verified and the ZIP listing contains no forbidden paths

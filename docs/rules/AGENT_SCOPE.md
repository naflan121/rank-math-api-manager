# Agent Scope

Agents and agent-related docs in this repository must describe this plugin only.

Location:

- agent authoring and review guidance: this file
- contributor policy: `CONTRIBUTING.md`
- reviewer confirmation: `.github/PULL_REQUEST_TEMPLATE.md`

## Allowed Scope

- plugin-local behavior in `rank-math-api-manager.php`, `includes/`, `assets/`, `docs/`, and related tests
- this plugin's REST routes, permissions, payloads, admin UI, release flow, and documentation
- external systems only when directly relevant to this plugin's documented integration points

## Forbidden Assumptions

- cross-project assumptions such as "assume repo X exists"
- generic or absolute repo paths such as `/repo/...` or machine-specific filesystem paths
- references to unrelated stacks, services, or products outside this plugin's scope

## Examples

Allowed example:

- "Review the permission callback for `rank-math-api-manager.php` and confirm the endpoint only allows users who can edit the requested post."

Disallowed example:

- "Open `/repo/apps/api` and update the shared auth middleware used by this plugin and the Bedrock site."

## Reviewer Verification

- Confirm new or changed agent docs only reference this plugin's files, behavior, and documented integrations.
- Confirm the text does not rely on unrelated repositories, absolute repo paths, or other stacks outside this plugin.

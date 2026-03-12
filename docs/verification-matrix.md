# Verification Matrix

## Release Artifact Verification

| Scenario | Expected Result |
| --- | --- |
| Download latest release asset | Asset name is exactly `rank-math-api-manager.zip` |
| Inspect ZIP root | Top-level folder is exactly `rank-math-api-manager/` |
| Inspect main plugin file path | `rank-math-api-manager/rank-math-api-manager.php` exists |
| Open plugin header | Version matches the release tag |

## Update Verification

| Scenario | Expected Result |
| --- | --- |
| Site running `1.0.8` clears update caches and checks again | WordPress shows update to `1.0.9.1` |
| Site running `1.0.9` checks for updates | WordPress shows update to `1.0.9.1` |
| Site running legacy folder name `Rank Math API Manager-plugin-kopi` checks for updates | Update detection still works |
| User opens “View details” modal | Plugin information loads without fatal errors |

## Admin Notice Verification

| Scenario | Expected Result |
| --- | --- |
| Rank Math dependency missing | Dependency notice renders |
| Dependency restored | Success notice renders once |
| Dependency deactivated | Warning notice renders once |
| Legacy folder name detected | Folder-normalization warning renders |
| Administrator dismisses folder notice | Notice stays dismissed for that user |
| Non-admin user visits dashboard/plugins | No plugin notice actions are available |

## Telemetry Verification

| Scenario | Expected Result |
| --- | --- |
| Plugin activates | Anonymous site ID exists and activation event is attempted |
| Telemetry remains enabled | Privacy notice appears until the operator acts |
| Operator disables telemetry | `rank_math_api_telemetry_settings.enabled` becomes false |
| Heartbeat cron fires twice quickly | Rate limit prevents duplicate heartbeat sends |
| Plugin deactivates | Deactivation event is attempted and heartbeat hook is cleared |
| Plugin uninstalls | Telemetry settings, heartbeat state, and dismissals are removed |

## Suggested Local Checks

```bash
wp option get rank_math_api_telemetry_settings
wp option get rank_math_api_dismissed_notices
wp option get rank_math_api_notice_events
wp cron event list | grep rank_math_api
wp option delete _site_transient_update_plugins
wp transient delete rank_math_api_github_release
wp option delete rank_math_api_last_github_check
```

## Acceptance Criteria

- Release ZIP layout is correct
- WordPress update notices appear for supported upgrade paths
- Legacy-folder installs receive guidance without blocking updates
- Telemetry is documented, minimal, and can be disabled
- No syntax or lint errors are introduced by the release

---

**Last Updated**: March 2026  
**Version**: 1.0.9.1

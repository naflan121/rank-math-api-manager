# Telemetry And Privacy

## Overview

Rank Math API Manager `1.0.9.1` introduces privacy-documented anonymous telemetry groundwork so Devora can validate update health and compatibility trends without collecting site content or operator identity.

## Endpoint

- Telemetry endpoint: `https://updates.devora.no/v1/telemetry`
- Transport: HTTPS only
- Remote calls are isolated so failures do not block plugin activation, admin requests, or update checks

## Events

The plugin can send these event types:

- `activate`
- `deactivate`
- `heartbeat`

The recurring heartbeat is scheduled once and sent at most once per day.

## Payload

The plugin sends only the following fields:

- `site_id`
- `plugin_slug`
- `plugin_version`
- `wp_version`
- `php_version`
- `event_type`
- `timestamp`

`site_id` is an anonymous install identifier generated with `wp_generate_uuid4()` and stored in the plugin telemetry settings option.

## Data Not Collected

The plugin does not send:

- Site URL or home URL
- Email addresses
- Usernames
- Application passwords
- SEO metadata or post content
- Plugin inventories
- Stack traces or raw error logs

## Opt-Out

Telemetry is enabled for the minimal payload above and announced to administrators with an admin notice. Administrators can disable it directly from the notice.

Stored option:

- `rank_math_api_telemetry_settings`

Example CLI checks:

```bash
wp option get rank_math_api_telemetry_settings
wp option patch update rank_math_api_telemetry_settings enabled 0
```

## Scheduling And Cleanup

- Scheduled hook: `rank_math_api_telemetry_heartbeat`
- Last heartbeat marker: `rank_math_api_heartbeat_last_run`
- Uninstall cleanup removes telemetry settings, scheduled hooks, and related dismissals

## Failure Isolation

- Telemetry failures are logged only as generic debug messages
- Payload bodies are not written to logs
- Non-telemetry plugin functionality continues even if the remote endpoint is unavailable

## Operational Notes

- Telemetry is groundwork for the future Devora Update API shadow rollout
- GitHub remains the active update source in `1.0.9.1`
- Telemetry does not change the current WordPress update flow

---

**Last Updated**: March 2026  
**Version**: 1.0.9.1

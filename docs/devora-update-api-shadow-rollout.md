# Devora Update API Shadow Rollout

## Goal

Introduce `updates.devora.no` in shadow mode first, while GitHub Releases remains the active update source for the plugin.

## Initial Endpoints

- `POST /v1/telemetry`
- `POST /v1/check`
- `GET /v1/plugin/{slug}/info`

## Shadow-Mode Principles

- The plugin continues to trust GitHub Releases for update metadata in `1.0.9.1`
- Devora Update API responses are validated separately and compared against GitHub responses
- A bad Devora response must never block updates while shadow mode is active

## Expected Request Data

### Telemetry

- Anonymous `site_id`
- `plugin_slug`
- `plugin_version`
- `wp_version`
- `php_version`
- `event_type`
- `timestamp`

### Future Update Checks

- `plugin_slug`
- Installed plugin version
- WordPress version
- PHP version

## Server-Side Data Model

- Aggregate by day/week
- Group by plugin slug, plugin version, event type
- Optionally bucket by WordPress and PHP major.minor versions
- Keep raw request data only briefly for abuse protection and debugging

## Rollout Stages

1. Ship plugin telemetry groundwork in `1.0.9.1`
2. Capture telemetry in Devora Update API without changing plugin update behavior
3. Build a GitHub release metadata cache server-side
4. Compare Devora metadata against GitHub metadata in shadow mode
5. Add plugin-side fallback ordering in a later release:
   - Devora Update API preferred
   - GitHub fallback if Devora is unavailable or invalid

## Validation Checklist

- Devora endpoint accepts only HTTPS requests
- Response schema is strict and versioned
- GitHub parity checks confirm the same version and asset URL
- Invalid Devora responses are ignored cleanly
- GitHub fallback remains healthy throughout shadow mode

## Non-Goals For 1.0.9.1

- No plugin cutover to Devora Update API yet
- No license enforcement
- No hard dependency on Devora infrastructure

---

**Last Updated**: March 2026  
**Version**: 1.0.9.1

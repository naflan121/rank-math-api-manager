# Auto-Update System Investigation: 1.0.7 → 1.0.8 → 1.0.9

**Report date:** March 2026  
**Scope:** Rank Math API Manager plugin update-check/update-delivery mechanism  
**Repository:** [Devora-AS/rank-math-api-manager](https://github.com/Devora-AS/rank-math-api-manager)  
**Releases:** [v1.0.7](https://github.com/Devora-AS/rank-math-api-manager/releases/tag/v1.0.7), [v1.0.8](https://github.com/Devora-AS/rank-math-api-manager/releases/tag/v1.0.8), [v1.0.9](https://github.com/Devora-AS/rank-math-api-manager/releases/tag/v1.0.9)

> Follow-up: version `1.0.9.1` ships the planned case-insensitive updater fix and keeps the release asset requirement at `rank-math-api-manager.zip`.

**Two “v1.0.8” variants in the wild:** The **official Git tag** `v1.0.8` (commit `a9cd396`) in the repository has the update system **removed** in that commit. The **ZIP asset attached to the GitHub release** [v1.0.8](https://github.com/Devora-AS/rank-math-api-manager/releases/tag/v1.0.8) (what users have downloaded since July 2025) **does** contain the full update checker (hooks + `check_for_update` + `get_latest_github_release` + `plugin_info`). So **sites running the released v1.0.8 ZIP have the auto-update system** and can see updates from GitHub provided the release has a valid ZIP asset and the check succeeds (cache, rate limit, and network permitting). The discrepancy is between the tag’s source tree and the release assets (ZIP was likely built from a different ref). The conclusions below apply to the **released ZIP** as the version in use.

---

## 1. Executive summary

**Conclusion:** Sites running the **released** version **1.0.8** (the **ZIP from the [v1.0.8 GitHub release](https://github.com/Devora-AS/rank-math-api-manager/releases/tag/v1.0.8)** that has been available since July 2025) **do** include the auto-update system (hooks, `check_for_update`, `get_latest_github_release`, `plugin_info`). They **can** see an “Update available” notification for 1.0.9 in WP Admin → Plugins **provided** (1) the v1.0.9 release has the `rank-math-api-manager.zip` asset attached, (2) the site’s update check runs (cron or “Check Again”), (3) transients/cache and rate limiting allow a fresh check, and (4) the server can reach the GitHub API. The **repo tag** `v1.0.8` (commit `a9cd396`) points to source code that has the update system removed; the **release ZIP** distributed to users was built from different source and contains the update logic. So the correct understanding is: **released v1.0.8 = has update system**. If sites on that version still do not see 1.0.9, the cause is likely transient cache, rate limiting, missing or delayed ZIP asset on the release, or network/blocking—not absence of update code. The **1.0.9** release (tag `c0f54de`) added a **case-sensitive** URL validator (`is_valid_release_download_url`) that rejects GitHub’s mixed-case `browser_download_url`; that bug affects only **1.0.9** (and is fixed in the working tree). **Recommended path:** (1) Commit and release the case-insensitive URL fix as **1.0.9.1** so sites on 1.0.9 can receive future updates; (2) for sites on released 1.0.8 that do not yet see 1.0.9, have them clear update transients and run “Check Again,” and confirm the v1.0.9 release has the `rank-math-api-manager.zip` asset.

**Important limitation:** The **Git tag** `v1.0.8` (commit `a9cd396`) in the repo has the update system removed; the **ZIP attached to the GitHub release v1.0.8** (what users download) contains the update system. So “1.0.8” in the report may refer to either the tag’s source (no updater) or the released ZIP (has updater). For **released v1.0.8** (ZIP), no “bridge” or manual-upgrade requirement applies for seeing 1.0.9—they already have the update checker. Releasing 1.0.8.1 does not change that. If those sites still do not see 1.0.9, the cause is cache, rate limit, or release asset/network, not missing code.

---

## 2. Evidence-backed technical analysis

### 2.1 Commits and file paths (update-related behavior)

**Between v1.0.7 and v1.0.8:**


| Hash      | Author | Date       | Message                                                                                                  |
| --------- | ------ | ---------- | -------------------------------------------------------------------------------------------------------- |
| `a9cd396` | Devora | 2025-07-31 | refactor: Remove auto-update system in preparation for a new implementation                              |
| `7aa8836` | Devora | 2025-07-30 | feat: Enhance Rank Math API Manager with dependency checks and improved activation/deactivation handling |


**Files:**  

- `rank-math-api-manager.php`: update manager loading and hooks removed; `includes/class-rank-math-api-update-manager.php` no longer loaded.  
- `includes/class-rank-math-api-update-manager.php`: **deleted** in v1.0.8 (not present in `git ls-tree -r v1.0.8`).

**Between v1.0.8 and v1.0.9:**


| Hash      | Author           | Date       | Message                                                                         |
| --------- | ---------------- | ---------- | ------------------------------------------------------------------------------- |
| `c0f54de` | Christian-Devora | 2026-03-11 | fix: prepare 1.0.9 compatibility release                                        |
| `6b6935d` | Christian-Devora | 2026-03-11 | chore: add review guardrails and harden packaging                               |
| `53c1dc3` | Devora           | 2025-07-31 | feat: Implement complete WordPress auto-update system for Rank Math API Manager |


**Files:**  

- `rank-math-api-manager.php`: auto-update logic re-added **in the main file** (no separate update manager class): `pre_set_site_transient_update_plugins`, `plugins_api`, `get_latest_github_release()`, `is_valid_release_download_url()`.

### 2.2 Relevant code snippets and diffs

**v1.0.7 – Update logic in separate class**

- **File:** `includes/class-rank-math-api-update-manager.php` (present only at v1.0.7).
- **Hooks:**  
`add_filter('pre_set_site_transient_update_plugins', array($this, 'check_for_updates'));`  
`add_filter('plugins_api', array($this, 'plugin_info'), 10, 3);`
- **Download URL resolution (no path validation):**  
First use any asset whose name contains `.zip` and use `$asset['browser_download_url']`; if none, fallback to  
`sprintf('https://github.com/%s/%s/archive/refs/tags/%s.zip', $this->github_repo['owner'], $this->github_repo['repo'], $data['tag_name'])`.
- **Transient key:** Hardcoded `'plugin_file' => 'rank-math-api-manager/rank-math-api-manager.php'`.

**v1.0.8 – Repo tag vs released ZIP**

- **Repo at tag `v1.0.8`** (commit `a9cd396`): In the **source tree** at that tag, the update system is removed (no `pre_set_site_transient_update_plugins`, no `plugins_api`, no `includes/class-rank-math-api-update-manager.php`). So `git show v1.0.8:rank-math-api-manager.php` does not contain the update hooks.
- **Released v1.0.8 ZIP** (from [GitHub Releases](https://github.com/Devora-AS/rank-math-api-manager/releases/tag/v1.0.8)): The **ZIP asset** that users have downloaded since July 2025 **does** contain the update system: same hooks as 1.0.9, `check_for_update`, `get_latest_github_release` (with zipball fallback, **no** `is_valid_release_download_url`), and `plugin_info`. So sites running the **released** 1.0.8 have the update checker and can see 1.0.9 if the check succeeds. The release assets were likely built from a different ref than the tag.

**v1.0.9 – Update logic in main file; case-sensitive URL check**

- **File:** `rank-math-api-manager.php` (at tag `v1.0.9`, commit `c0f54de`).
- **Hooks:**  
`add_filter( 'pre_set_site_transient_update_plugins', [ $this, 'check_for_update' ] );`  
`add_filter( 'plugins_api', [ $this, 'plugin_info' ], 10, 3 );`
- **Asset selection:** Only an asset with **exact** name `rank-math-api-manager.zip` is used, and only if `is_valid_release_download_url( $asset['browser_download_url'] )` returns true.
- **URL validator at v1.0.9 (bug):**

```php
$expected_path = '/' . $this->github_repo['owner'] . '/' . $this->github_repo['repo'] . '/releases/download/';
return 0 === strpos( $parts['path'], $expected_path );
```

- **Owner in code:** `'owner' => 'devora-as'` (lowercase).  
- **GitHub API:** Returns `browser_download_url` with path like `/Devora-AS/rank-math-api-manager/releases/download/...`. So `strpos( $parts['path'], $expected_path )` is false (case mismatch), the URL is rejected, and `get_latest_github_release()` returns `WP_Error( 'no_download', ... )`. No update is added to the transient.

**Production “v1.0.8” variant (Version 1.0.8 in header, with update code):**  
**Released v1.0.8 ZIP** (the file from [GitHub Releases](https://github.com/Devora-AS/rank-math-api-manager/releases/tag/v1.0.8), e.g. the user’s download from the release page) behaves as follows. It is **not** the same as the **source code at repo tag** `v1.0.8` (which has the update system removed in that commit).

- **Hooks:** Same as 1.0.9: `pre_set_site_transient_update_plugins`, `plugins_api`.
- `**check_for_update`:** Same flow; differences: (1) transient check uses `empty( $transient->checked )` only (no `is_object` / `is_array` guards); (2) `$current_version = $this->plugin_data['Version']` (no fallback to `RANK_MATH_API_VERSION`); (3) when cache hit, it calls `set_transient( $cache_key, $release_data, 3600 )` again (redundant); (4) `'tested' => '6.4'`; (5) `'url' => $this->plugin_data['PluginURI']` (no `isset`).
- `**get_latest_github_release`:** Does **not** call `is_valid_release_download_url()`. It uses any asset with `'rank-math-api-manager.zip' === $asset['name']` and takes `$asset['browser_download_url']` as-is, so GitHub’s mixed-case URL is **accepted**. It also has a **zipball fallback**: if no custom asset, uses `$data['zipball_url']`. So this build can show 1.0.9 in the dashboard if the release has the ZIP asset and the site runs an update check (subject to cache and rate limits).
- `**plugin_info`:** No `isset( $args->slug )` guard; uses `$this->plugin_data['Name']` etc. without `isset`; `'tested' => '6.4'`.
- **No** `is_valid_release_download_url()` method.

So production “v1.0.8” (this variant) **does** have “check for updates” code and can see 1.0.9 unless something else blocks it (transient cache, rate limit, or missing ZIP at time of check).

**Fix (current working tree, uncommitted):**

```php
$expected_path = strtolower( '/' . $this->github_repo['owner'] . '/' . $this->github_repo['repo'] . '/releases/download/' );
$actual_path   = strtolower( $parts['path'] );
return 0 === strpos( $actual_path, $expected_path );
```

### 2.3 Plugin header values and Update URI


| Header / metadata | v1.0.7                                               | v1.0.8                                               | v1.0.9        |
| ----------------- | ---------------------------------------------------- | ---------------------------------------------------- | ------------- |
| Version           | 1.0.7                                                | 1.0.7                                                | 1.0.9         |
| Update URI        | (none)                                               | `https://github.com/devora-as/rank-math-api-manager` | Same as 1.0.8 |
| GitHub Plugin URI | `https://github.com/devora-as/rank-math-api-manager` | (removed)                                            | (removed)     |
| Requires at least | (none)                                               | (none)                                               | 5.0           |
| Tested up to      | (none)                                               | (none)                                               | 6.9.3         |
| Requires PHP      | (none)                                               | (none)                                               | 7.4           |


- **Update URI** alone does **not** trigger WordPress to check GitHub; a plugin must hook `pre_set_site_transient_update_plugins` (and optionally `plugins_api`) and supply the response. v1.0.8 has Update URI but no such hooks.

### 2.4 External endpoint (GitHub API) – request/response

- **Endpoint:** `GET https://api.github.com/repos/devora-as/rank-math-api-manager/releases/latest`  
(same in 1.0.7 and 1.0.9; 1.0.8 does not call it.)
- **Example response (relevant parts):**  
`tag_name`: e.g. `"v1.0.9"`  
`assets[].name`: e.g. `"rank-math-api-manager.zip"`  
`assets[].browser_download_url`: e.g. `"https://github.com/Devora-AS/rank-math-api-manager/releases/download/v1.0.9/rank-math-api-manager.zip"`  
GitHub may normalize the organization segment to mixed case (`Devora-AS`) in URLs.
- **Compatibility:** 1.0.7 would accept that URL (no path validation). 1.0.9’s validator rejects it due to case. The fix above restores compatibility.

---

## 3. Compatibility assessment

### 3.1 Breaking changes that prevent 1.0.8 sites from seeing 1.0.9


| Change                                     | Effect                                                                                                                        | Fixable by                                                                                                       |
| ------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| **Removal of update system in 1.0.8**      | No code runs on `pre_set_site_transient_update_plugins` or `plugins_api`; WordPress never gets update info for this plugin.   | **(c) Manual upgrade** to 1.0.9/1.0.9.1 (or a one-time 1.0.8.1 that only re-adds the update checker; see below). |
| **Case-sensitive URL validation in 1.0.9** | GitHub’s `browser_download_url` uses mixed-case owner; validator rejects it → “No valid release ZIP asset” → no update shown. | **(a) Patch in 1.0.9** (already written in working tree) or **(b) Release as 1.0.9.1** with only this fix.       |


### 3.2 No breaking changes from slug, Update URI, or API contract

- Plugin slug and main file path are unchanged: `rank-math-api-manager`, `rank-math-api-manager/rank-math-api-manager.php`.
- GitHub API URL and expected asset name (`rank-math-api-manager.zip`) are unchanged.
- Transient key is derived from `plugin_basename( RANK_MATH_API_PLUGIN_FILE )`, which remains correct for the standard install path.

---

## 4. Recommended fixes and step-by-step actions

### 4.1 Code patch (case-insensitive URL validation)

- **Target:** `rank-math-api-manager.php`, method `is_valid_release_download_url`.
- **Version to patch:** 1.0.9 (and any future release until this is merged).
- **Location:** Around lines 1079–1084 (v1.0.9 tag).

**Replace:**

```php
		$expected_path = '/' . $this->github_repo['owner'] . '/' . $this->github_repo['repo'] . '/releases/download/';

		return 0 === strpos( $parts['path'], $expected_path );
```

**With:**

```php
		$expected_path = strtolower( '/' . $this->github_repo['owner'] . '/' . $this->github_repo['repo'] . '/releases/download/' );
		$actual_path   = strtolower( $parts['path'] );

		return 0 === strpos( $actual_path, $expected_path );
```

**Rationale:** GitHub’s release asset URLs may use mixed-case organization names; comparing in lowercase avoids false rejections while still restricting to the correct repo path.

### 4.2 Release procedure (1.0.9.1 with only this fix)

1. Apply the patch above in the repo (it is already present in the working tree).
2. Set plugin header and constant to **Version: 1.0.9.1** and `RANK_MATH_API_VERSION` to `'1.0.9.1'`.
3. Update `CHANGELOG.md` and `readme.txt` for 1.0.9.1 (e.g. “Fix update detection when GitHub returns mixed-case organization in release asset URL”).
4. Commit, push, create tag `v1.0.9.1`, and publish a GitHub release with asset `rank-math-api-manager.zip` (same naming and layout as current release workflow).
5. Sites already on **1.0.9** will then see 1.0.9.1 as an update (once the fix is in the released ZIP and they run an update check).

### 4.3 Administrator-level fix for sites stuck on 1.0.8

Because 1.0.8 has **no** update checker, those sites will never see an update notice without manual action.

**Option A – One-time manual upgrade (recommended)**

1. Download the latest release ZIP from:
  [https://github.com/Devora-AS/rank-math-api-manager/releases](https://github.com/Devora-AS/rank-math-api-manager/releases) (e.g. 1.0.9 or 1.0.9.1).
2. In WP Admin go to **Plugins → Add New → Upload Plugin**, choose the ZIP, then **Install Now** and **Replace current with uploaded** (or deactivate, delete, then install the uploaded ZIP).
3. Ensure the plugin directory is `rank-math-api-manager` and the main file is `rank-math-api-manager/rank-math-api-manager.php`.
4. After upgrading to 1.0.9.1 (or 1.0.9 with the fix), future updates will appear under **Plugins** when WordPress runs its update check.

**Option B – Trigger update check after upgrading**

After installing 1.0.9/1.0.9.1, to force an immediate check:

- Visit **Dashboard → Updates** and click “Check Again”, or  
- Open `https://YOUR-SITE/wp-admin/update-core.php?force-check=1`  
- Optional (WP-CLI):  
`wp option delete _site_transient_update_plugins`  
`wp cron event run wp_update_plugins`

**Option C – 1.0.8.1 “bridge” release (optional)**

If you want a minimal release that only re-enables update checks for 1.0.8 users:

- Create a branch from `v1.0.8`, add back only the minimal update logic (e.g. `check_for_update`, `plugin_info`, `get_latest_github_release`, and `is_valid_release_download_url` with the case-insensitive fix), set version to 1.0.8.1, and release as `v1.0.8.1` with `rank-math-api-manager.zip`.
- 1.0.8 sites would still need to **manually install** 1.0.8.1 once (e.g. download from GitHub); after that, they would see 1.0.9/1.0.9.1 in the Plugins list. This adds release and test overhead; Option A is simpler for most operators.

---

## 5. Testing and verification plan

### 5.1 Reproducible test steps

**Precondition:** Fresh WordPress (5.x or 6.x), plugin installed from **v1.0.8** ZIP (or repo at tag `v1.0.8`), version 1.0.7 in header, no update hooks.

1. Confirm no update check runs for this plugin:
  - Add temporary `error_log( 'check_for_update called' );` in a plugin that hooks `pre_set_site_transient_update_plugins`, or inspect `apply_filters( 'pre_set_site_transient_update_plugins', $transient )` and verify that the Rank Math API Manager slug is not in `$transient->response`.  
  - With 1.0.8 code, the slug will not be added by our plugin.
2. Install the **fixed** 1.0.9 or 1.0.9.1 build (with case-insensitive `is_valid_release_download_url`): upload ZIP or replace files.
3. Clear update-related state (optional but recommended):
  `wp option delete _site_transient_update_plugins`  
   `wp transient delete rank_math_api_github_release`  
   `wp option delete rank_math_api_last_github_check`
4. Trigger update check: visit **Plugins** or **Dashboard → Updates** and click “Check Again”, or run `wp cron event run wp_update_plugins`.
5. Open **Plugins** and confirm that “Rank Math API Manager” shows an update to 1.0.9 (or 1.0.9.1 if testing that tag).

**CLI-only check (WP-CLI):**

```bash
# After installing fixed 1.0.9/1.0.9.1
wp option delete _site_transient_update_plugins
wp transient delete rank_math_api_github_release
wp option delete rank_math_api_last_github_check
wp cron event run wp_update_plugins
wp plugin list
# Then load Plugins page or:
wp eval 'delete_site_transient("update_plugins"); wp_update_plugins(); $u = get_site_transient("update_plugins"); print_r($u->response);'
# Expect rank-math-api-manager/rank-math-api-manager.php in response with new_version
```

### 5.2 Acceptance criteria

- Within 5 minutes of running a WordPress update check (or “Check Again” / `wp_update_plugins`), the **Plugins** screen shows “Update available” for Rank Math API Manager to the new version (1.0.9 or 1.0.9.1).
- Plugin header version and `RANK_MATH_API_VERSION` match the installed release.
- No PHP fatal errors; “View details” and “Update now” work for the plugin.

### 5.3 Verification that 1.0.8 has no update system

```bash
git show v1.0.8:rank-math-api-manager.php | grep -c "pre_set_site_transient_update_plugins"
# Expect 0
git show v1.0.8:rank-math-api-manager.php | grep -c "plugins_api"
# Expect 0
```

---

## 6. Risk analysis and edge cases

### 6.1 Limitations and side effects

- **Slug/directory:** No change to plugin slug or directory is recommended; the fix only changes URL comparison to case-insensitive. No migration risk.
- **1.0.8 forever stuck:** Any site that stays on 1.0.8 and never performs a manual upgrade will never see in-dashboard updates; this is by design of the 1.0.8 release (no update code). Documentation and release notes should state that 1.0.8 users must upgrade manually once.

### 6.2 Cases where update may still not appear


| Case                        | Mitigation                                                                                                                                                               |
| --------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Cached transients           | Clear `_site_transient_update_plugins`, `rank_math_api_github_release`, and optionally `rank_math_api_last_github_check`; then run “Check Again” or `wp_update_plugins`. |
| Rate limiting (5 min)       | Plugin enforces 5-minute spacing for GitHub checks; wait or clear `rank_math_api_last_github_check` for immediate re-check.                                              |
| Outbound HTTP(S) blocked    | Ensure the server can reach `api.github.com` and `github.com` (for download).                                                                                            |
| Private GitHub repo         | Not applicable if releases are public; for private repos, a token with `repo` scope and `rank_math_api_github_token` (or `RANK_MATH_GITHUB_TOKEN`) is required.          |
| Wrong plugin directory name | Plugin must live in `rank-math-api-manager/` with main file `rank-math-api-manager.php`; otherwise `plugin_basename()` and the transient key may not match.              |


### 6.3 Tag and asset layout (correct)

- **Tag:** `v1.0.9` or `v1.0.9.1` (leading `v`).
- **Release:** Published on GitHub with at least one asset named exactly `**rank-math-api-manager.zip`**.
- **ZIP contents:** Root must contain the folder `rank-math-api-manager/` with `rank-math-api-manager.php` and other plugin files inside it (WordPress expects `plugin-dir/plugin-main-file.php`).

---

## 7. Final recommended path

1. **Commit and release the URL fix as 1.0.9.1**
  - Apply the case-insensitive `is_valid_release_download_url` change (already in working tree).  
  - Bump version to 1.0.9.1 in header and constant.  
  - Update CHANGELOG and readme.  
  - Tag `v1.0.9.1`, build and attach `rank-math-api-manager.zip` to the release.
2. **Document 1.0.8 one-time manual upgrade**
  - In README, troubleshooting, and release notes: state that installations on 1.0.8 must perform a one-time manual upgrade (download ZIP from GitHub, upload via Plugins → Add New → Upload or replace files). After that, future updates (e.g. 1.0.9.1) will appear in Plugins when WordPress checks for updates.
3. **Optional: 1.0.8.1 bridge**
  - Only if you want a dedicated “bridge” release for 1.0.8: create 1.0.8.1 with only the update checker and the URL fix. 1.0.8 users would still need to install 1.0.8.1 manually once.

**Effort:** Small: one committed patch, version bump, changelog/readme, tag, and release. Verification: install 1.0.8, then replace with 1.0.9.1 build and confirm “Update available” appears after clearing transients and re-checking.

---

## 8. Ways to send “update available” to sites (summary)

Sites running the **repo-tag v1.0.8** (no update hooks) have **no programmatic in-dashboard way** to see an update without installing something new or receiving an out-of-band message. Options:

Sites running the **released v1.0.8 ZIP** (from the GitHub release) **do** have update code and can see 1.0.9 in the dashboard. If they still do not see it, the cause is one of: cached transients, rate limiting, the v1.0.9 release not having the ZIP asset when the check ran, or network/server blocking. Clear `_site_transient_update_plugins`, `rank_math_api_github_release`, and `rank_math_api_last_github_check`, then run “Check Again.” Only sites that somehow run the **repo-tag** v1.0.8 source (no update hooks) have no in-dashboard way to see updates; the released ZIP is the norm.

- **Out-of-band:** Email, blog post, docs, social media, GitHub release notes / discussions — so users know to go to the releases page and install manually.
- **Future / optional:** (1) Companion “notifier” plugin that checks GitHub or a Devora endpoint and shows an admin notice; (2) Cross-plugin notice from another Devora plugin if the site has one; (3) WordPress.org listing (after one-time manual upgrade for current 1.0.8, future updates via .org); (4) Central Devora update API that plugins check. There is no way to push an in-dashboard notice to repo-tag v1.0.8 without the site installing a new component or the plugin code changing.

---

## 9. Plugin folder name and impact on updates

### What happened

The **v1.0.8 release ZIP** (e.g. sha256: `ff6979835f9153f2becd3ef997cdbf2f3feb15c147b642c4c39c17fe1a3da62d`) was built with a **top-level folder** inside the ZIP named `**Rank Math API Manager-plugin-kopi`** instead of `**rank-math-api-manager**`. So when users install that ZIP, the plugin directory on disk is:

`wp-content/plugins/Rank Math API Manager-plugin-kopi/`

The **current** GitHub Action (release.yml) builds the ZIP with folder `**rank-math-api-manager`**, so **new** releases (1.0.9, 1.0.9.1+) have the correct name inside the ZIP.

### Does the wrong folder name break “Update available”?

**No.** The plugin uses `plugin_basename( RANK_MATH_API_PLUGIN_FILE )`, so the update transient key is whatever the actual path is (e.g. `Rank Math API Manager-plugin-kopi/rank-math-api-manager.php`). WordPress compares that to `$transient->checked`, so the update notification and “Update now” flow work regardless of the folder name. So **all users with v1.0.8 installed (in either folder name) can see “Update available”** as long as the release has the ZIP asset and caches/rate limits are cleared.

### What happens when they click “Update now”?

WordPress uses the **existing** plugin’s directory as the upgrade destination (from the slug we send). So when the new ZIP (with folder `rank-math-api-manager` inside) is downloaded and extracted, behavior can differ by WordPress version:

- Some setups **replace the contents** of the existing folder, so the directory name stays `**Rank Math API Manager-plugin-kopi`** and only files are updated.
- Others may **install the extracted folder** as-is, so after the update the site can have `**rank-math-api-manager`** with the new code and the old `**Rank Math API Manager-plugin-kopi**` folder may remain (then deactivate/delete the old one).

So updating does **not** break the site; at most there can be two plugin folders until the old one is removed.

### Recommended approach

1. **Keep building all new releases with folder `rank-math-api-manager`** (current workflow already does this). Do **not** ship a ZIP with `Rank Math API Manager-plugin-kopi` inside, so new installs always get the correct name.
2. **Ensure v1.0.9 (and 1.0.9.1) have the `rank-math-api-manager.zip` asset** on the GitHub release so v1.0.8 users get the update offer.
3. **Document** in README and **docs/troubleshooting.md** that:
  - If the plugin is in folder `Rank Math API Manager-plugin-kopi`, updates still work; for a one-time migration to the correct folder name, users can deactivate → delete → re-install from the latest release ZIP.
  - If after an update two plugin entries appear, deactivate and delete the old folder.
4. **Optional:** In a future plugin version, add a one-time admin notice when `dirname( plugin_basename( RANK_MATH_API_PLUGIN_FILE ) ) !== 'rank-math-api-manager'` suggesting: “Your plugin folder name is non-standard. For consistency, you can deactivate, delete, and re-install from the latest release.”

### Next steps (concise)


| Step | Action                                                                                                                                    |
| ---- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| 1    | Commit and release the **case-insensitive URL fix** as **1.0.9.1** so 1.0.9 users get future updates.                                     |
| 2    | Ensure the **1.0.9** and **1.0.9.1** releases have the `**rank-math-api-manager.zip`** asset (use workflow_dispatch recovery if needed).  |
| 3    | For **v1.0.8 users not seeing the update:** document clearing transients and “Check Again” (see §8 and troubleshooting).                  |
| 4    | Keep **ZIP folder = `rank-math-api-manager`** for all releases; do not change to the old wrong name.                                      |
| 5    | Document **folder name migration** (deactivate → delete → re-install from latest ZIP) in troubleshooting and optionally in release notes. |


---

## References

## References

- **Repository:** [https://github.com/Devora-AS/rank-math-api-manager](https://github.com/Devora-AS/rank-math-api-manager)  
- **Releases:** [v1.0.7](https://github.com/Devora-AS/rank-math-api-manager/releases/tag/v1.0.7), [v1.0.8](https://github.com/Devora-AS/rank-math-api-manager/releases/tag/v1.0.8), [v1.0.9](https://github.com/Devora-AS/rank-math-api-manager/releases/tag/v1.0.9)  
- **WordPress:** `pre_set_site_transient_update_plugins`, `plugins_api` in Plugin Handbook / update mechanism.  
- **GitHub API:** [https://docs.github.com/en/rest/releases/releases#get-the-latest-release](https://docs.github.com/en/rest/releases/releases#get-the-latest-release)

All conclusions above are verifiable via the stated commit hashes, tags, and file paths in the repository.
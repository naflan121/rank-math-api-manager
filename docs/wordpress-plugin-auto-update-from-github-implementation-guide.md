# WordPress Plugin Auto-Update from GitHub - Implementation Guide

> **Project Reference**: Implementation pattern used in Rank Math API Manager (auto-update re-implemented in v1.0.9 and hardened in v1.0.9.1 with case-insensitive GitHub asset URL validation)
> 

> **Status**: ✅ Production-Ready Solution (for plugins that include the update hooks) 

## 📋 Overview

This guide provides a complete, tested implementation for adding WordPress native auto-update functionality to GitHub-hosted plugins. The solution mimics [`api.wordpress.org`](http://api.wordpress.org) behavior to provide seamless updates for users.

## 🎯 What This Achieves

- ✅ **WordPress Native Updates**: Users see update notifications just like [WordPress.org](http://WordPress.org) plugins
- ✅ **Auto-Update Toggle**: Users can enable/disable automatic updates
- ✅ **"View Details" Modal**: Complete plugin information and changelog
- ✅ **GitHub Integration**: Fetches releases directly from GitHub API
- ✅ **Rate Limiting**: Prevents API abuse with intelligent caching
- ✅ **Production Ready**: Handles errors, edge cases, and user experience

## 🏗️ Core Implementation Strategy

### The WordPress Update System

WordPress checks for updates via two main hooks:

1. **`pre_set_site_transient_update_plugins`** - Core update checking
2. **`plugins_api`** - Plugin information for "View Details" modal

Our implementation hooks into these to provide GitHub-based updates.

### Key Components

1. **Update URI Header** - Prevents [WordPress.org](http://WordPress.org) conflicts
2. **GitHub API Integration** - Fetches latest releases
3. **Version Comparison** - Determines when updates are available
4. **Custom ZIP Assets** - Ensures correct folder structure
5. **Rate Limiting & Caching** - Prevents API abuse
6. **Error Handling** - Graceful failures and debugging

## 🔧 Step-by-Step Implementation

### Step 1: Plugin Header Configuration

```
<?php
/**
 * Plugin Name: Your Plugin Name
 * Plugin URI: https://your-domain.com/plugins/your-plugin
 * Description: Your plugin description
 * Version: 1.0.0
 * Author: Your Company
 * Author URI: https://your-domain.com
 * License: GPL v3 or later
 * License URI: https://www.gnu.org/licenses/gpl-3.0.html
 * Text Domain: your-plugin-textdomain
 * Update URI: https://github.com/your-org/your-plugin-repo
 * Requires at least: 5.0
 * Tested up to: 6.4
 * Requires PHP: 7.4
 */

// Prevent direct access
if (!defined('ABSPATH')) {
    exit;
}
```

**Critical**: The `Update URI` header prevents WordPress from checking [WordPress.org](http://WordPress.org) for updates.

### Step 2: Plugin Constants

```
// Define plugin constants
define('YOUR_PLUGIN_VERSION', '1.0.0');
define('YOUR_PLUGIN_FILE', __FILE__);
define('YOUR_PLUGIN_DIR', plugin_dir_path(__FILE__));
define('YOUR_PLUGIN_URL', plugin_dir_url(__FILE__));
```

### Step 3: Main Plugin Class Structure

```
class Your_Plugin_Manager {
    private static $instance = null;
    private $plugin_data = null;
    private $github_repo = null;
    private $github_token = null;
    
    public static function get_instance() {
        if (null === self::$instance) {
            self::$instance = new self();
        }
        return self::$instance;
    }
    
    private function __construct() {
        $this->init_plugin_data();
        $this->init_hooks();
    }
    
    private function init_plugin_data() {
        if (!function_exists('get_plugin_data')) {
            require_once ABSPATH . 'wp-admin/includes/plugin.php';
        }
        $this->plugin_data = get_plugin_data(YOUR_PLUGIN_FILE);
        
        // GitHub repository configuration
        $this->github_repo = array(
            'owner' => 'your-org',
            'repo' => 'your-plugin-repo',
            'api_url' => 'https://api.github.com/repos/your-org/your-plugin-repo/releases/latest'
        );
        
        // Initialize GitHub authentication for higher rate limits
        $this->init_github_auth();
    }
    
    private function init_hooks() {
        // WordPress update system hooks
        add_filter('pre_set_site_transient_update_plugins', array($this, 'check_for_update'));
        add_filter('plugins_api', array($this, 'plugin_info'), 10, 3);
        
        // Cleanup hooks
        register_deactivation_hook(YOUR_PLUGIN_FILE, array($this, 'deactivation_cleanup'));
        register_uninstall_hook(YOUR_PLUGIN_FILE, array($this, 'uninstall_cleanup'));
    }
}
```

### Step 4: GitHub Authentication (Optional but Recommended)

```
private function init_github_auth() {
    // Check for GitHub token in WordPress options (secure storage)
    $this->github_token = get_option('your_plugin_github_token');
    
    // If no token, check for environment variable
    if (!$this->github_token && defined('YOUR_PLUGIN_GITHUB_TOKEN')) {
        $this->github_token = YOUR_PLUGIN_GITHUB_TOKEN;
    }
}
```

**Benefits of Authentication**:

- Unauthenticated: 60 requests/hour
- With Personal Access Token: 5,000 requests/hour

### Step 5: Core Update Checking Logic

```
public function check_for_update($transient) {
    // Early return if no checked plugins
    if (empty($transient->checked)) {
        return $transient;
    }
    
    $plugin_slug = plugin_basename(YOUR_PLUGIN_FILE);
    $current_version = YOUR_PLUGIN_VERSION;
    
    // Rate limiting - check every 5 minutes maximum
    $last_check = get_option('your_plugin_last_update_check', 0);
    $check_interval = 300; // 5 minutes
    
    if (time() - $last_check < $check_interval) {
        $this->log_debug('Rate limited: skipping update check');
        return $transient;
    }
    
    // Update last check time
    update_option('your_plugin_last_update_check', time());
    
    // Get latest release from GitHub
    $release_data = $this->get_latest_github_release();
    
    if (is_wp_error($release_data)) {
        $this->log_debug('Failed to get GitHub release: ' . $release_data->get_error_message());
        return $transient;
    }
    
    $remote_version = ltrim($release_data['tag_name'], 'v');
    
    $this->log_debug("Comparing versions: Current={$current_version}, Remote={$remote_version}");
    
    // Check if update is available
    if (version_compare($remote_version, $current_version, '>')) {
        $this->log_debug('Update available!');
        
        // Look for custom ZIP asset first
        $download_url = $this->get_download_url($release_data);
        
        $plugin_update = (object) array(
            'slug' => dirname($plugin_slug),
            'plugin' => $plugin_slug,
            'new_version' => $remote_version,
            'url' => $this->plugin_data['PluginURI'],
            'package' => $download_url,
            'tested' => $this->plugin_data['RequiresWP'] ?? '6.0',
            'requires_php' => $this->plugin_data['RequiresPHP'] ?? '7.4',
            'compatibility' => new stdClass(),
        );
        
        $transient->response[$plugin_slug] = $plugin_update;
    } else {
        $this->log_debug('No update needed - version is current');
    }
    
    return $transient;
}
```

### Step 6: GitHub API Integration with Caching

```
private function get_latest_github_release() {
    // Check cache first (1 hour)
    $cache_key = 'your_plugin_github_release';
    $cached_release = get_transient($cache_key);
    
    if (false !== $cached_release) {
        $this->log_debug('Using cached GitHub release data');
        return $cached_release;
    }
    
    // Rate limiting for GitHub API calls
    $last_github_check = get_option('your_plugin_last_github_check', 0);
    $github_interval = 300; // 5 minutes
    
    if (time() - $last_github_check < $github_interval) {
        return new WP_Error('rate_limited', 'GitHub API rate limited');
    }
    
    // Update GitHub check time
    update_option('your_plugin_last_github_check', time());
    
    // Prepare headers with optional authentication
    $headers = array(
        'Accept' => 'application/vnd.github.v3+json',
        'User-Agent' => 'WordPress/' . get_bloginfo('version') . '; ' . get_bloginfo('url')
    );
    
    // Add authentication header if token is available
    if ($this->github_token) {
        $headers['Authorization'] = 'token ' . $this->github_token;
        $this->log_debug('Using authenticated GitHub API request (5000/hour limit)');
    } else {
        $this->log_debug('Using unauthenticated GitHub API request (60/hour limit)');
    }
    
    // Make API request
    $response = wp_remote_get($this->github_repo['api_url'], array(
        'timeout' => 15,
        'headers' => $headers
    ));
    
    if (is_wp_error($response)) {
        return new WP_Error('api_error', 'GitHub API request failed: ' . $response->get_error_message());
    }
    
    $response_code = wp_remote_retrieve_response_code($response);
    if ($response_code !== 200) {
        return new WP_Error('api_error', "GitHub API returned status {$response_code}");
    }
    
    $body = wp_remote_retrieve_body($response);
    $release_data = json_decode($body, true);
    
    if (json_last_error() !== JSON_ERROR_NONE) {
        return new WP_Error('json_error', 'Failed to parse GitHub API response');
    }
    
    if (!isset($release_data['tag_name'])) {
        return new WP_Error('invalid_response', 'GitHub API response missing tag_name');
    }
    
    // Cache the result for 1 hour
    set_transient($cache_key, $release_data, HOUR_IN_SECONDS);
    
    $this->log_debug('GitHub release data cached for 1 hour');
    
    return $release_data;
}
```

### Step 7: Download URL Handling

```
private function get_download_url($release_data) {
    // Look for custom ZIP asset first (recommended)
    if (isset($release_data['assets']) && is_array($release_data['assets'])) {
        foreach ($release_data['assets'] as $asset) {
            if ($asset['name'] === '[your-plugin-name.zip](http://your-plugin-name.zip)') {
                $this->log_debug('Using custom ZIP asset: ' . $asset['browser_download_url']);
                return $asset['browser_download_url'];
            }
        }
    }
    
    // Fallback to GitHub's auto-generated zipball
    $this->log_debug('No custom ZIP found, using zipball: ' . $release_data['zipball_url']);
    return $release_data['zipball_url'];
}
```

**Important**: Custom ZIP assets ensure correct folder structure. GitHub's auto-generated zipball creates folders like `repo-name-commit-hash` which breaks WordPress installations.

### Step 8: Plugin Information for "View Details" Modal

```
public function plugin_info($res, $action, $args) {
    // Only handle plugin_information requests for our plugin
    if ($action !== 'plugin_information' || $args->slug !== dirname(plugin_basename(YOUR_PLUGIN_FILE))) {
        return false;
    }
    
    $release_data = $this->get_latest_github_release();
    
    if (is_wp_error($release_data)) {
        return false;
    }
    
    $remote_version = ltrim($release_data['tag_name'], 'v');
    
    return (object) array(
        'name' => $this->plugin_data['Name'],
        'slug' => dirname(plugin_basename(YOUR_PLUGIN_FILE)),
        'version' => $remote_version,
        'author' => $this->plugin_data['Author'],
        'author_profile' => $this->plugin_data['AuthorURI'],
        'homepage' => $this->plugin_data['PluginURI'],
        'requires' => $this->plugin_data['RequiresWP'] ?? '5.0',
        'tested' => '6.4',
        'requires_php' => $this->plugin_data['RequiresPHP'] ?? '7.4',
        'download_link' => $this->get_download_url($release_data),
        'sections' => array(
            'description' => $this->plugin_data['Description'],
            'changelog' => $this->format_changelog($release_data['body'] ?? 'See GitHub releases for changelog.')
        ),
        'banners' => array(),
        'icons' => array(),
        'last_updated' => $release_data['published_at'] ?? date('Y-m-d'),
    );
}

private function format_changelog($changelog) {
    // Convert markdown to HTML for better display
    $changelog = wp_kses_post($changelog);
    $changelog = wpautop($changelog);
    return $changelog;
}
```

### Step 9: Debug Logging and Utilities

```
private function log_debug($message) {
    if (defined('WP_DEBUG') && WP_DEBUG) {
        error_log('Your Plugin: ' . $message);
    }
}

// Cleanup functions
public function deactivation_cleanup() {
    delete_transient('your_plugin_github_release');
    delete_option('your_plugin_last_update_check');
    delete_option('your_plugin_last_github_check');
}

public static function uninstall_cleanup() {
    delete_transient('your_plugin_github_release');
    delete_option('your_plugin_last_update_check');
    delete_option('your_plugin_last_github_check');
    delete_option('your_plugin_github_token');
}
```

### Step 10: Initialize the Plugin

```
// Initialize the plugin
function your_plugin_init() {
    Your_Plugin_Manager::get_instance();
}
add_action('plugins_loaded', 'your_plugin_init');
```

## 🚀 GitHub Release Automation

### GitHub Action for Automated ZIP Creation

Create `.github/workflows/release.yml`:

```
name: Build and Release Plugin

on:
  release:
    types: [published]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup PHP
        uses: shivammathur/setup-php@v2
        with:
          php-version: "8.1"

      - name: Create production-ready plugin ZIP
        run: |
          # Create plugin directory with correct name
          mkdir -p your-plugin-name

          # Copy core plugin files
          cp your-plugin-name.php your-plugin-name/

          # Copy includes directory if exists
          if [ -d "includes" ]; then
            cp -r includes/ your-plugin-name/
          fi

          # Copy assets directory if exists
          if [ -d "assets" ]; then
            cp -r assets/ your-plugin-name/
          fi

          # Copy essential documentation
          [ -f "[README.md](http://README.md)" ] && cp [README.md](http://README.md) your-plugin-name/
          [ -f "LICENSE" ] && cp LICENSE your-plugin-name/
          [ -f "[CHANGELOG.md](http://CHANGELOG.md)" ] && cp [CHANGELOG.md](http://CHANGELOG.md) your-plugin-name/

          # Remove any development files that may have been copied
          rm -rf your-plugin-name/.git* 2>/dev/null || true
          rm -rf your-plugin-name/.github/ 2>/dev/null || true
          rm -rf your-plugin-name/tests/ 2>/dev/null || true
          rm -rf your-plugin-name/.cursor/ 2>/dev/null || true
          rm -rf your-plugin-name/node_modules/ 2>/dev/null || true
          rm -f your-plugin-name/.DS_Store 2>/dev/null || true
          rm -f your-plugin-name/TODO*.md 2>/dev/null || true
          rm -f your-plugin-name/.env* 2>/dev/null || true
          rm -f your-plugin-name/.gitignore 2>/dev/null || true

          # Create ZIP file with WordPress-compatible name
          zip -r [your-plugin-name.zip](http://your-plugin-name.zip) your-plugin-name/

          # Verify ZIP structure
          echo "📦 ZIP Contents:"
          unzip -l [your-plugin-name.zip](http://your-plugin-name.zip) | head -20

      - name: Upload Release Asset
        uses: actions/upload-release-asset@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          upload_url: ${{ github.event.release.upload_url }}
          asset_path: ./[your-plugin-name.zip](http://your-plugin-name.zip)
          asset_name: [your-plugin-name.zip](http://your-plugin-name.zip)
          asset_content_type: application/zip

      - name: Verify Release Completion
        run: |
          echo "✅ Release asset uploaded successfully"
          echo "📦 File: [your-plugin-name.zip](http://your-plugin-name.zip)"
          echo "📋 Contents: Production-ready plugin files only"
          echo "🚀 Ready for WordPress auto-update system"
```

## 🔐 GitHub Personal Access Token Setup

### Creating the Token

1. **Go to GitHub Settings**: https://github.com/settings/tokens
2. **Generate new token** → "Generate new token (classic)"
3. **Configure Token**:
    - **Name**: `WordPress Plugin Updates`
    - **Expiration**: 1 year (or no expiration)
    - **Scopes**: Only `public_repo` (read access to public repositories)

### Storing the Token Securely

**Method 1: wp-config.php (Recommended)**

```
// Add to wp-config.php
define('YOUR_PLUGIN_GITHUB_TOKEN', 'ghp_your_token_here');
```

**Method 2: WordPress Options (Alternative)**

```
// Store in WordPress database
update_option('your_plugin_github_token', 'ghp_your_token_here');
```

## 📋 Testing and Validation

### Testing Checklist

1. **Update Detection**
    - [ ]  Plugin detects new GitHub releases
    - [ ]  Version comparison works correctly
    - [ ]  Update notification appears in WordPress admin
2. **Download and Installation**
    - [ ]  ZIP file downloads correctly
    - [ ]  Plugin extracts to correct folder name
    - [ ]  All files are present after update
    - [ ]  Plugin activates successfully after update
3. **User Experience**
    - [ ]  "View Details" modal shows correct information
    - [ ]  Auto-update toggle functions properly
    - [ ]  Error handling works gracefully
4. **Performance and Rate Limiting**
    - [ ]  API calls are properly rate limited
    - [ ]  Caching reduces unnecessary API requests
    - [ ]  Debug logging provides useful information

### Debug Testing

Add this to `wp-config.php` for testing:

```
define('WP_DEBUG', true);
define('WP_DEBUG_LOG', true);
```

Force update check:

```
// Add to functions.php temporarily
add_action('admin_init', function() {
    delete_transient('your_plugin_github_release');
    delete_option('your_plugin_last_update_check');
    delete_option('your_plugin_last_github_check');
});
```

## ⚠️ Common Pitfalls and Solutions

### 1. **Incorrect ZIP Folder Structure**

**Problem**: GitHub's auto-generated zipball creates folders like `repo-name-abc123`

**Solution**: Use custom ZIP assets in GitHub releases with correct folder names

### 2. **Version Number Mismatches**

**Problem**: Plugin shows old version after update

**Solution**: Ensure version is updated in plugin header AND constants before creating release

### 3. **Rate Limiting Issues**

**Problem**: Too many API requests causing failures

**Solution**: Implement proper caching and rate limiting (5-minute intervals minimum)

### 4. **Missing Update URI Header**

**Problem**: [WordPress.org](http://WordPress.org) conflicts or doesn't recognize custom updates

**Solution**: Always include `Update URI` header pointing to GitHub repository

### 5. **GitHub API Authentication**

**Problem**: Hitting 60/hour rate limit

**Solution**: Use Personal Access Token for 5,000/hour limit

## 🎯 Best Practices

### Security

- ✅ Always validate and sanitize input
- ✅ Use WordPress capability checks
- ✅ Store tokens securely (wp-config.php or environment variables)
- ✅ Implement proper error handling

### Performance

- ✅ Cache GitHub API responses (1 hour minimum)
- ✅ Rate limit API calls (5 minutes minimum)
- ✅ Use efficient database queries
- ✅ Clean up transients and options on deactivation

### User Experience

- ✅ Provide clear update notifications
- ✅ Support auto-update toggle
- ✅ Include comprehensive "View Details" information
- ✅ Handle errors gracefully with fallbacks

### Development

- ✅ Use semantic versioning (v1.0.0 format)
- ✅ Maintain detailed changelogs
- ✅ Test update process thoroughly
- ✅ Document API usage and rate limits

## 📚 References and Resources

### WordPress Documentation

- [Plugin Update API](https://developer.wordpress.org/reference/hooks/pre_set_site_transient_update_plugins/)
- [Plugins API Filter](https://developer.wordpress.org/reference/hooks/plugins_api/)
- [WordPress Plugin Headers](https://developer.wordpress.org/plugins/plugin-basics/header-requirements/)

### GitHub Documentation

- [GitHub Releases API](https://docs.github.com/en/rest/releases/releases)
- [GitHub Personal Access Tokens](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token)
- [GitHub Actions for Releases](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#release)

### Testing Tools

- [WordPress Plugin Check (PCP)](https://github.com/WordPress/plugin-check)
- [WordPress Coding Standards](https://github.com/WordPress/WordPress-Coding-Standards)

---

**Success Story**: This implementation pattern was used in the Rank Math API Manager plugin. The current production auto-update flow is in v1.0.9.1+, with the GitHub asset URL validator hardened to accept mixed-case organization paths. See `docs/auto-update-investigation-report-1.0.7-1.0.9.md` for version history and limitations.

**Limitation – versions that cannot self-update**: If a release ships without the update hooks (e.g. no `pre_set_site_transient_update_plugins`), sites on that version will never see “Update available” in the dashboard. Releasing a later “bridge” version does not notify those sites, because the code running on their server never runs an update check. The plugin does not send installation or usage data to a central server, so there is no way to push notifications to those users. Reaching them requires out-of-band communication (announcements, docs, email) or directing them to the GitHub releases page.

**Need Help?** For technical questions or implementation support, contact the Engineering team or refer to the project-specific implementation in the Rank Math API Manager repository.
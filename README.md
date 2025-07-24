# WordPress Plugin GitHub Updater

A reusable WordPress plugin updater class that enables automatic updates from GitHub repositories. Built on top of the [`yahnis-elsts/plugin-update-checker`](#dependencies) library.

## Features

- ✅ **Automatic updates** from GitHub releases and branches
- ✅ **Release asset filtering** with regex patterns
- ✅ **Custom branch support** for development/staging
- ✅ **Simple static factory methods** for easy setup
- ✅ **Built-in error handling** and debug logging
- ✅ **Flexible configuration** options

## Table of Contents

- [Quick Start](#quick-start)
- [Configuration](#configuration)
- [GitHub Workflow Setup](#github-workflow-setup)
- [Usage Examples](#usage-examples)
- [Advanced Usage](#advanced-usage)
- [Dependencies](#dependencies)
- [License](#license)


## Quick Start

Copy [`class-github-plugin-updater.php`](class-github-plugin-updater.php) to your plugin directory and include it:

```php
if ( ! class_exists( 'Soderlind\WordPress\GitHub_Plugin_Updater' ) ) {
    require_once 'class-github-plugin-updater.php';
}

// Basic setup (most common)
$updater = \Soderlind\WordPress\GitHub_Plugin_Updater::create(
    'https://github.com/username/plugin-name',
    __FILE__,
    'plugin-name'
);
```

That's it! Your plugin will now check for updates from the specified GitHub repository.

## Configuration

### Parameters

| Parameter | Required | Description | Example |
|-----------|----------|-------------|---------|
| `github_url` | Yes | GitHub repository URL | `'https://github.com/username/plugin-name'` |
| `plugin_file` | Yes | Path to main plugin file | `__FILE__` |
| `plugin_slug` | Yes | Plugin slug for WordPress | `'my-awesome-plugin'` |
| `branch` | No | Git branch to check for updates (default: `'master'`) | `'main'`, `'develop'` |
| `name_regex` | No | Regex pattern for release assets | `'/plugin-name\.zip/'` |
| `enable_release_assets` | No | Whether to enable release assets | `true` if name_regex provided |

> **Note**: When `branch` is set to `master`, the updater prioritizes releases and tags before falling back to the branch itself.

### Initialization Methods

#### 1. Basic Setup
```php
$updater = \Soderlind\WordPress\GitHub_Plugin_Updater::create(
    'https://github.com/username/plugin-name',
    __FILE__,
    'plugin-name'
);
```

#### 2. Custom Branch
```php
$updater = \Soderlind\WordPress\GitHub_Plugin_Updater::create(
    'https://github.com/username/plugin-name',
    __FILE__,
    'plugin-name',
    'develop'  // Branch name
);
```

#### 3. With Release Assets
```php
$updater = \Soderlind\WordPress\GitHub_Plugin_Updater::create_with_assets(
    'https://github.com/username/plugin-name',
    __FILE__,
    'plugin-name',
    '/plugin-name\.zip/'  // Regex pattern for zip file
);
```

#### 4. Full Configuration
```php
$updater = new \Soderlind\WordPress\GitHub_Plugin_Updater([
    'github_url'            => 'https://github.com/username/plugin-name',
    'plugin_file'           => __FILE__,
    'plugin_slug'           => 'plugin-name',
    'branch'                => 'main',
    'name_regex'            => '/plugin-name-v[\d\.]+\.zip/',
    'enable_release_assets' => true,
]);
```

## GitHub Workflow Setup

To automatically create release assets, add this workflow to your repository at `.github/workflows/on-release-add.zip.yml`:

```yaml
name: On Release, Build release zip

on:
  release:
    types: [published]

jobs:
  build:
    name: Build release zip
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Build plugin
        run: composer install --no-dev

      - name: Archive Release
        uses: thedoctor0/zip-release@b57d897cb5d60cb78b51a507f63fa184cfe35554
        with:
          type: 'zip'
          filename: 'your-plugin-name.zip'  # Change this
          exclusions: '*.git* .editorconfig composer* *.md package.json package-lock.json'

      - name: Release
        uses: softprops/action-gh-release@c95fe1489396fe8a9eb87c0abf8aa5b2ef267fda
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          files: your-plugin-name.zip  # Change this
          tag_name: ${{ github.event.release.tag_name }}
```

### Customization Tips

- Update `filename` and `files` to match your plugin name
- Modify `exclusions` to include/exclude specific files
- Adjust the build step for your project needs (npm, webpack, etc.)

This creates downloadable assets at URLs like:
```
https://github.com/username/plugin-name/releases/latest/download/plugin-name.zip
```

## Usage Examples

### Real-World Implementations
- [Additional JavaScript](https://github.com/soderlind/additional-javascript/blob/main/additional-javascript.php#L33-L44)
- [Multisite Exporter](https://github.com/soderlind/multisite-exporter/blob/main/includes/class-multisite-exporter.php#L57-L63)
- [Super Admin Switch to Admin](https://github.com/soderlind/super-admin-switch-to-admin/blob/main/super-admin-switch-to-admin.php#L38-L50)

### Basic Plugin Setup
```php
// In your main plugin file
define( 'MY_PLUGIN_FILE', __FILE__ );
require_once plugin_dir_path( __FILE__ ) . 'class-github-plugin-updater.php';

$updater = \Soderlind\WordPress\GitHub_Plugin_Updater::create(
    'https://github.com/myusername/my-awesome-plugin',
    MY_PLUGIN_FILE,
    'my-awesome-plugin'
);
```

### Development Branch Updates
```php
$updater = \Soderlind\WordPress\GitHub_Plugin_Updater::create(
    'https://github.com/company/enterprise-plugin',
    __FILE__,
    'enterprise-plugin',
    'develop'  // Use development branch
);
```

### Release Assets with Pattern Matching
```php
$updater = \Soderlind\WordPress\GitHub_Plugin_Updater::create_with_assets(
    'https://github.com/company/premium-plugin',
    __FILE__,
    'premium-plugin',
    '/premium-plugin-v[\d\.]+\.zip/'  // Match versioned zip files
);
```
## Advanced Usage

### Custom Class Names
If you need to avoid naming conflicts, extend or wrap the class:

```php
namespace YourCompany\YourPlugin;

class Your_Plugin_Updater extends \Soderlind\WordPress\GitHub_Plugin_Updater {
    // Inherit all functionality, customize as needed
}
```

### Plugin-Specific Wrapper
```php
namespace YourCompany\YourPlugin;

class Your_Plugin_Updater {
    private $updater;
    
    public function __construct() {
        $this->updater = \Soderlind\WordPress\GitHub_Plugin_Updater::create(
            'https://github.com/yourcompany/your-plugin',
            YOUR_PLUGIN_FILE,
            'your-plugin'
        );
    }
}
```

### Best Practices

- **Use constants** for plugin file paths: `define( 'MY_PLUGIN_FILE', __FILE__ )`
- **Prefer static factory methods** like `::create()` for simpler setup
- **Test thoroughly** with different branches and release patterns
- **Document your configuration** for future reference
- **Consider namespacing** to avoid conflicts with other plugins

### Error Handling

The updater includes built-in error handling:

- **Parameter validation** on initialization
- **Exception graceful handling** during update checks
- **Debug logging** when `WP_DEBUG` is enabled
- **Graceful degradation** if updater setup fails

## Dependencies

This updater requires the `yahnis-elsts/plugin-update-checker` library:

```bash
composer require yahnis-elsts/plugin-update-checker
```

Or download manually from: https://github.com/YahnisElsts/plugin-update-checker

## License

MIT (same as the original library)

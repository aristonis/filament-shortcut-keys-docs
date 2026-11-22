# Installation

## Requirements

Filament Shortcut Keys requires the following to run:

- PHP 8.2+
- Laravel v12.0+ or v13.0+
- Filament v4.0+ or v5.0+
- A database with a native JSON column type: MySQL 5.7+, MariaDB 10.2+, PostgreSQL or SQLite

The test suite runs against SQLite, MySQL 8 and PostgreSQL 16.

## Installing the package

Install the package with Composer:

```bash
composer require aristonis/filament-shortcut-keys
```

Laravel discovers the package's service provider automatically.

## Running the migrations

The plugin stores shared presets, each admin's overrides and each admin's selected map in the database. Publish the migrations and run them:

```bash
php artisan vendor:publish --tag="filament-shortcut-keys-migrations"
php artisan migrate
```

You need these tables in the `personal` and `locked` [customization modes](customization.md#choosing-a-customization-mode). In [`config` mode](customization.md#config-mode), nothing is stored and the plugin never queries its tables, so you may skip this step.

## Registering the plugin

Register the plugin in your panel provider:

```php
use Aristonis\FilamentShortcutKeys\FilamentShortcutKeysPlugin;
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->plugin(FilamentShortcutKeysPlugin::make());
}
```

The plugin adds two pages to the panel, **Keyboard shortcuts** and **Manage shortcuts**, and adds the keymap and the cheatsheet to every page of the panel. In [`config` mode](customization.md#config-mode), it adds only the **Keyboard shortcuts** page. If your application has several panels, register the plugin on each panel that should have shortcuts.

The page slugs come from the class names, not from the labels. Use them to link to the pages or to test them:

| Page | Slug | Route name |
|---|---|---|
| Keyboard shortcuts | `shortcut-reference` | `filament.{panel}.pages.shortcut-reference` |
| Manage shortcuts | `manage-shortcuts` | `filament.{panel}.pages.manage-shortcuts` |

`{panel}` is the panel ID. In `config` mode, the `manage-shortcuts` route does not exist. For a panel with the ID `admin`:

```php
$this->assertTrue(Route::has('filament.admin.pages.shortcut-reference'));
```

## Publishing the assets

The keys are handled by a script that Filament publishes with its other assets. After you register the plugin, publish it with:

```bash
php artisan filament:assets
```

!!! warning
    Without the published script, no key does anything, and no error appears. The keymap is still rendered into the page, so everything looks installed.

Run this command once even if your application has the Composer hook described below. Filament publishes the script only for a panel that has the plugin registered. When you run `composer require`, the plugin is not on a panel yet, so the hook has nothing of it to publish.

An application set up with `php artisan filament:install` runs `filament:upgrade` from Composer's `post-autoload-dump` hook, which publishes the assets of every registered plugin after each `composer install` and `composer update`. The hook keeps the script current when you update the package. Check that your `composer.json` contains the hook:

```json
"scripts": {
    "post-autoload-dump": [
        "Illuminate\\Foundation\\ComposerScripts::postAutoloadDump",
        "@php artisan package:discover --ansi",
        "@php artisan filament:upgrade"
    ]
}
```

If it does not, run `php artisan filament:assets` yourself after every update.

## Publishing the configuration

The configuration file is optional. To change a default, publish it:

```bash
php artisan vendor:publish --tag="filament-shortcut-keys-config"
```

This creates `config/shortcut-keys.php`. Every option is described in [Configuration](configuration.md).

## Using a custom theme

The plugin's pages and its cheatsheet carry their own styles and are built from Filament's own components. If your panel uses a custom theme, you do not need to add the plugin to it.

## Laravel Boost

The package ships guidelines and skills for [Laravel Boost](https://laravel.com/docs/boost). When `php artisan boost:install` asks which third-party packages to include, select this one. If you install Boost non-interactively, add the package to `boost.json` and run `boost:install` again:

```json
"packages": ["aristonis/filament-shortcut-keys"]
```

## Trying it out

Open any page in the panel:

1. Press `G` and release it. A small letter badge appears on each sidebar link. Press one of those letters within a second and a half to open that page.
2. Press `?`. The cheatsheet lists every shortcut active on the current page.
3. On a list page, press `J`. A cursor appears on the first row. Use `J` and `K` to move it, `Enter` to edit the row and `Esc` to release the cursor.

Next, read [Shortcuts](shortcuts.md) to see which shortcuts exist and how their letters are chosen.

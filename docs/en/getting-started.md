# Getting started

## Requirements

- PHP 8.2 or newer
- Laravel 12 or 13
- Filament 5.x. Filament 4 is not supported.
- MySQL 5.7+, MariaDB 10.2+, PostgreSQL, or SQLite (the schema uses a native `json` column). The test
  suite runs on SQLite, MySQL 8 and PostgreSQL 16; older versions and MariaDB are not tested.

## Install

```bash
composer require aristonis/filament-shortcut-keys
```

The service provider is auto-discovered.

## Publish and run the migrations

```bash
php artisan vendor:publish --tag="filament-shortcut-keys-migrations"
php artisan migrate
```

This creates `shortcut_maps`, `shortcut_map_entries` and `shortcut_map_selections`. The migrations are
needed in every mode, including `locked`: the shared map, its overrides and each admin's selection all
live there.

## Register the plugin

```php
use Aristonis\FilamentShortcutKeys\FilamentShortcutKeysPlugin;

public function panel(Panel $panel): Panel
{
    return $panel
        ->plugin(FilamentShortcutKeysPlugin::make());
}
```

That is the whole setup. The plugin registers two pages of its own ("Keyboard shortcuts" and "Manage
shortcuts"), injects the keymap into every panel page, and ships the script that listens for keys.
To key your own table record actions, add `->rowActions([...])`; see [Row actions](row-actions.md).

## Publish the assets

```bash
php artisan filament:assets
```

Skipping this is the most common reason nothing happens when you press a key. The keymap still renders
into the page, so there is no error to read; the script that listens for the keys is simply missing.

An app set up with `filament:install` already runs `@php artisan filament:upgrade` from Composer's
`post-autoload-dump` hook, which republishes every plugin's assets after each `composer install` or
`composer update`. Check your `composer.json`:

```json
"scripts": {
    "post-autoload-dump": [
        "Illuminate\\Foundation\\ComposerScripts::postAutoloadDump",
        "@php artisan package:discover --ansi",
        "@php artisan filament:upgrade"
    ]
}
```

Without that hook, run `filament:assets` by hand after installing and after every update of the
package.

## Custom theme

If your panel uses a custom Filament theme, add the plugin's views to the theme's CSS file so its
classes survive Tailwind's purge:

```css
@source '../../../../vendor/aristonis/filament-shortcut-keys/resources/**/*.blade.php';
```

## Publish the config (optional)

```bash
php artisan vendor:publish --tag="filament-shortcut-keys-config"
```

This copies `config/shortcut-keys.php` into your app. You only need it to change a default; see
[Configuration](configuration.md).

## Laravel Boost

The package ships guidelines and skills for [Laravel Boost](https://github.com/laravel/boost). The
interactive `php artisan boost:install` asks which third-party packages to include; pick this one. A
non-interactive install (`-n`) skips third-party packages, so add the package to `boost.json` and run
`boost:install` again:

```json
"packages": ["aristonis/filament-shortcut-keys"]
```

## First steps to try

Open any page in the panel and:

1. Press `G` and let go. While it waits, a small letter badge appears on each sidebar link. Press one
   of those letters within a second and a half and the panel navigates there.
2. Press `?` (`Shift` + `/`). A cheatsheet lists every shortcut active on the current page.
3. On a list page, press `J`. A row cursor appears on the first row. `J` and `K` move it, `Enter`
   edits the row, `Esc` releases the cursor.

Key names in these docs are key caps: `G` means the G key pressed on its own, without `Shift`. Next,
read [The shortcuts](shortcuts.md) for the full key table and the rules behind it.

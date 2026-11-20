# Filament shortcut keys

A Filament panel plugin that gives every panel keyboard shortcuts, assigned automatically. It walks the
panel's registered resources, pages and actions, gives each one a letter, and injects the resulting
keymap into every page. There is no per-resource wiring: add a resource and it gets a shortcut. Admins
can remap or disable their own keys, and someone you trust can edit the shared map everyone else
starts from.

> Status: **1.0.** Supports Filament 5 on Laravel 12 and 13, PHP 8.2 or newer. Filament 4 is not
> supported.

## Install

```bash
composer require aristonis/filament-shortcut-keys
php artisan vendor:publish --tag="filament-shortcut-keys-migrations"
php artisan migrate
```

Then register `FilamentShortcutKeysPlugin::make()` on your panel and make sure the plugin's script is
published. [Getting started](getting-started.md) walks through both.

## What it does

- **Navigation:** `G`, then a letter, opens any registered resource or page.
- **Header actions:** a bare letter clicks the current page's header actions, but only those that are
  safe to press by accident (they open a page, a modal or a confirmation).
- **Tables:** `/` searches, `J` and `K` move a row cursor, `Space` selects, `Enter` edits, `Delete`
  deletes, and the arrows move the cursor and page the table. Works on relation managers and table
  widgets too.
- **Row actions:** register record-action names such as `approve` once and each gets a letter on every
  table that has it.
- **Discoverability:** `?` opens a cheatsheet of the current page, a "Keyboard shortcuts" page lists
  the panel-wide keys, and small key badges sit on the controls they press.
- **Customization:** each admin can remap, disable or add their own keys, or the panel can be locked to
  one shared map. Editing the shared map is gated and denied by default.
- **Safe by construction:** a shortcut clicks the real Filament control or follows the real route, so
  your policies, middleware and confirmation modals decide what happens.
- **Accessibility:** keys carry `aria-keyshortcuts`, and anyone can switch single-key shortcuts off
  from the cheatsheet.

## Where to go next

- [Getting started](getting-started.md): install, register the plugin, publish assets.
- [The shortcuts](shortcuts.md): every key, and the rules that decide which ones exist.
- [Row actions](row-actions.md): keying your own record actions.
- [Letting admins customize](customization.md): personal and locked modes, presets, the authoring gate.
- [Custom bindings](custom-bindings.md): route and selector bindings, and driving them from code.
- [Configuration](configuration.md): every config key.
- [Pruning dead overrides](maintenance.md), [Translations](translations.md),
  [Security](security.md), [Troubleshooting](troubleshooting.md), [Limitations](limitations.md).
- [Architecture](architecture.md): how the plugin is put together.

Source code and issues: [github.com/aristonis/filament-shortcut-keys](https://github.com/aristonis/filament-shortcut-keys).

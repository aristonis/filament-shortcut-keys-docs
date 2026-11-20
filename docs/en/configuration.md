# Configuration

## Introduction

The plugin works without a configuration file. To change a default, publish the file:

```bash
php artisan vendor:publish --tag="filament-shortcut-keys-config"
```

This creates `config/shortcut-keys.php`. The options apply to every panel that registers the plugin. Row actions are the exception: you [register them on the plugin](row-actions.md#registering-row-actions), per panel.

The plugin reads the configuration on every page render, so a change takes effect on the next request. If your application caches its configuration, run `php artisan config:cache` again after changing the file.

## The configuration file

The published file contains these options and defaults:

```php
return [

    'customization' => 'personal',

    'modifiers' => [
        'navigation' => ['g'],
        'global' => [],
        'table' => [],
        'row-action' => [],
        'custom' => ['g'],
    ],

    'authorize' => null,

    'overlay' => [],

    'cache' => [
        'ttl' => 86400,
    ],

];
```

## Choosing the customization mode

The `customization` option is `'personal'`, the default, `'locked'` or `'config'`:

```php
'customization' => 'locked',
```

Any other value throws a `ValueError`. See [Choosing a customization mode](customization.md#choosing-a-customization-mode).

## Changing how keys are pressed

The `modifiers` option sets how each set's keys are pressed. The sets are `navigation`, `global` (header actions), `table`, `row-action` and `custom` (custom bindings). Each takes a list of tokens:

| Value | Meaning |
| --- | --- |
| `['g']` | A leader key: press `G`, release it, then press the letter. Any single lowercase letter works. |
| `[]` | Single keys, with no modifier |
| `['alt']`, `['ctrl', 'shift']`, ... | Keys pressed while holding the modifiers. The tokens are `ctrl`, `alt`, `shift` and `meta`. |

`meta` is the `Cmd` key on a Mac and the Windows key on Windows. The cheatsheet shows it as `Cmd`.

For example, to put header actions on `Alt` and give custom bindings their own `H` leader:

```php
'modifiers' => [
    'navigation' => ['g'],
    'global' => ['alt'],
    'table' => [],
    'row-action' => [],
    'custom' => ['h'],
],
```

The defaults avoid held modifiers on purpose. See [Shortcuts](shortcuts.md#introduction).

These rules apply:

- **Removing a set from the list restores its default.** It does not turn the set into single keys, so deleting a line is a safe way to undo a change.
- **Sets with the same setting share one pool of letters**, so they never get the same key. By default, navigation and custom bindings share the `G` pool, and header actions and row actions share the single-key pool.
- **A leader key is reserved among single keys**, so no single-key shortcut takes it.
- **The scheme is the same for every admin.** There is no per-admin setting.

There is no `mod` token. Whether it would mean `Ctrl` or `Cmd` depends on the admin's platform, and the keymap is built on the server.

### Invalid settings

These settings throw an exception when a panel page renders, with a message that names the problem:

| Mistake | Example |
| --- | --- |
| An unknown token | `['option']`, `['mod']`, or an uppercase leader such as `['G']` |
| A leader combined with other tokens | `['g', 'shift']` |
| A leader on a single-key table key | `'navigation' => ['j']` while the table keys are single keys |
| Header actions or row actions on the same setting as navigation or custom bindings | `'global' => ['g']`, or `'row-action' => []` with `'navigation' => []` |

The last rule exists because those two groups are lettered separately, so a shared pool could give one key to two shortcuts. Header actions and row actions may have a leader of their own, such as `'global' => ['a']`.

## Authorizing preset authors

The `authorize` option decides who may create and edit the panel's shared presets. It accepts `null`, which denies everyone and is the default, the name of a gate ability, or a closure:

```php
'authorize' => 'manage-shortcut-keys',
```

A closure receives `?Authenticatable $user` and `string $panelId` and must return exactly `true` to grant access. Prefer a gate ability, because a closure prevents `php artisan config:cache` from working. See [Granting authoring access](customization.md#granting-authoring-access).

## Pinning and disabling shortcuts

The `overlay` option pins a shortcut to a letter or removes it, from code and for every admin:

```php
'overlay' => [
    'navigation:App\Filament\Resources\ProductResource' => ['letter' => 'p'],
    'global:export' => ['letter' => 'x'],
    'row-action:approve' => ['letter' => 'a'],
    'navigation:App\Filament\Pages\Settings' => ['disabled' => true],
    'table:delete' => ['disabled' => true],
],
```

Each key names a shortcut as `set:name`:

| Set | Name | Example |
| --- | --- | --- |
| `navigation` | The resource or page class | `navigation:App\Filament\Resources\OrderResource` |
| `global` | The header action's name | `global:export` |
| `row-action` | The registered row action's name | `row-action:approve` |
| `table` | A [table key name](table-shortcuts.md#turning-off-a-table-key) | `table:delete` |

Each value is one of:

- `['letter' => 'r']`, which puts the shortcut on that letter,
- `['disabled' => true]`, which removes the shortcut and frees its letter for others.

Pinning a letter also gives a key to a header action that [runs immediately](shortcuts.md#which-header-actions-get-a-shortcut), which otherwise has none. Only pin such an action when a stray keystroke running it is acceptable.

A pinned letter outside `a` to `z`, or on a reserved key in its pool (`G`, `J` or `K` among single keys), is ignored, so one bad entry cannot break the keymap. Custom bindings belong to an admin and cannot be pinned or disabled here.

The overlay is meant for keys you need fixed in code, not for preferences. An overlay disable also overrides an admin's own change, but an admin who remaps a pinned shortcut gets their own letter. See [How changes combine](customization.md#how-changes-combine).

!!! tip
    Pinned letters can collide with each other. After adding pins, open the cheatsheet and check that each pinned shortcut still has its key.

## Caching

The navigation shortcuts and custom bindings are the same on every page, so the plugin caches them in your application's default cache store. The `cache.ttl` option sets how long, in seconds, a cached keymap is kept:

```php
'cache' => [
    'ttl' => 86400,
],
```

You do not need to clear the cache when something changes. Adding, removing, relabelling or re-sorting a resource or page, saving a change on the **Manage shortcuts** page, editing the `overlay` or `modifiers` options, and switching locale each use a new cache entry automatically. Admins who use the same map share one entry.

The old entries are not deleted, and the lifetime is what removes them. Set `ttl` to `null` to cache forever. Only do that on a cache store you manage yourself, because old entries then accumulate with every change your admins make.

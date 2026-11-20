# Configuration

The config file is optional. Publish it only when you need to change a default:

```bash
php artisan vendor:publish --tag="filament-shortcut-keys-config"
```

This copies `config/shortcut-keys.php` into your app. Every value is read at call time, so a change
takes effect on the next page render. With a cached config in production, run `php artisan
config:cache` (or `config:clear`) after editing it.

## The full file

```php
return [
    // 'personal' lets each admin fork and remap their own keys; 'locked' uses one shared map.
    'customization' => 'personal',

    // How each set's letters are pressed.
    'modifiers' => [
        'navigation' => ['g'],
        'global' => [],
        'table' => [],
        'row-action' => [],
        'custom' => ['g'],
    ],

    // Who may edit the panel's shared presets. null denies everyone.
    'authorize' => null,

    // Pin or drop a shortcut from code, keyed by target identity.
    'overlay' => [],

    // Seconds to cache the panel-wide keymap.
    'cache' => [
        'ttl' => 86400,
    ],
];
```

## customization

`'personal'` (default) or `'locked'`. See [Letting admins customize](customization.md#personal-and-locked).

## modifiers

How each set's letters are pressed. The sets are `navigation`, `global` (header actions), `table`,
`row-action` and `custom`. Each takes a list of tokens:

| Value | Meaning |
| --- | --- |
| `['g']` | A leader key: press `G`, let go, then the letter. Any single lowercase letter works. |
| `[]` | Bare keys. |
| `['alt']`, `['ctrl', 'shift']`, ... | Held modifiers. Tokens: `ctrl`, `alt`, `shift`, `meta`. |

The defaults avoid held modifiers on purpose: see
[why nothing is held by default](shortcuts.md#why-nothing-is-held-by-default).

Rules:

- **Removing a set from the list restores its built-in scheme.** It does not turn the set into bare
  keys, so deleting a line is the safe way to undo a change.
- **Two sets on the same scheme share one letter pool**, so they can never get the same key. By
  default navigation and custom share the `G` pool, and header and row actions share the bare pool.
- **A leader key is reserved among bare keys**, so no bare shortcut can take it.
- `mod` is not supported: whether it means `Ctrl` or `Cmd` depends on the user's platform, and the
  keymap is built on the server.

These configurations throw when a panel page renders, with a message naming the problem:

| Mistake | Example |
| --- | --- |
| An unknown token | `['option']`, `['mod']`, or an uppercase leader `['G']` |
| A leader mixed with other tokens | `['g', 'shift']` |
| A leader that is a bare table key | `'navigation' => ['j']` while the table is on bare keys |
| Header or row actions on the same keys as navigation or custom bindings | `'global' => ['g']`, or `'row-action' => []` with `'navigation' => []` |

The last rule exists because the two groups are lettered separately, so a shared pool could hand one
key to two shortcuts. Header and row actions may have keys of their own, for example
`'global' => ['a']`.

Keys are configured once per deployment, for everyone. There is no per-admin modifier scheme.

## authorize

Who may author the shared presets: `null` (deny everyone, the default), the name of a Gate ability, or
a closure `fn (?Authenticatable $user, string $panelId): bool`. Prefer the ability, since a closure
breaks `php artisan config:cache`. See [the authoring gate](customization.md#the-authoring-gate).

## overlay

Pin or drop a shortcut from code, with no database rows:

```php
'overlay' => [
    'navigation:App\Filament\Resources\ProductResource' => ['letter' => 'p'],
    'global:export' => ['letter' => 'x'],
    'row-action:approve' => ['letter' => 'a'],
    'navigation:App\Filament\Pages\Hidden' => ['disabled' => true],
    'table:delete' => ['disabled' => true],
],
```

Each key is a target identity, `set:key`:

| Set | Key | Example |
| --- | --- | --- |
| `navigation` | The resource or page class | `navigation:App\Filament\Resources\OrderResource` |
| `global` | The header action's name | `global:export` |
| `row-action` | The registered row-action name | `row-action:approve` |
| `table` | A table behaviour | `table:delete` |

The table behaviours are `search`, `cursor-down`, `cursor-up`, `row-down`, `row-up`, `page-prev`,
`page-next`, `select`, `edit`, `delete` and `cursor-release`. Custom bindings belong to an admin and
cannot be targeted from the overlay.

Each entry is either `['letter' => 'r']`, which forces that letter, or `['disabled' => true]`, which
removes the shortcut. A disabled shortcut is removed before letters are assigned, so its letter goes
back to the pool.

The overlay beats the conventions, which makes it the right tool for a key you need fixed in code and
the wrong tool for a preference. An overlay disable also beats an admin's own edit; a pinned letter
does not, so an admin who remaps that shortcut gets their own letter. See
[how the layers combine](customization.md#how-the-layers-combine).

A letter pin also turns on a header action that runs on the click itself, which otherwise gets no key
on bare letters. Only pin such an action if a stray keystroke running it is acceptable.

### Forced letters

An overlay pin and a letter stored in an admin's map are both forced letters. They follow the same
rules:

- **Not a–z:** ignored. The shortcut keeps its automatic letter, and one bad entry cannot break the
  keymap.
- **A reserved key in that pool** (`G`, `J`, `K` or another table key among bare keys): ignored the
  same way.
- **A key an automatically lettered shortcut holds:** the forced letter takes it, and the other
  shortcut moves to another free letter.
- **A key another forced letter holds:** the first in pool order keeps it and the later one loses its
  key. Pool order is navigation before custom bindings, and header actions before row actions.

Registered row actions are lettered before the overlay applies, so their letters count as forced. See
[row actions](row-actions.md#how-letters-are-assigned) for what that means when you pin one. Check the
cheatsheet after adding pins.

## cache

```php
'cache' => ['ttl' => 86400],
```

How long, in seconds, to cache the panel-wide keymap (navigation and custom bindings). The page's own
shortcuts are cheap and are resolved on every render.

The lifetime is about reclaiming space, not about freshness. The cache key is a fingerprint of:

- the panel id;
- the navigation: the registered resources and pages, with their labels and sort order;
- the active map's identity and version (every saved change bumps the version);
- the developer config that shapes the map: `overlay`, `modifiers` and the keys they reserve;
- the locale;
- the package's cache format.

Any change to one of these resolves to a different key, so a stale map is not served. The key does not
include who is asking: admins on the same map share one entry. What the fingerprint cannot do is delete
the entry it moved away from, and every edit or deploy that touches navigation leaves one behind. The
lifetime is what reclaims those.

`null` caches forever. Only choose it on a store you actively manage, and expect the keyspace to grow
with the number of edits your admins make.

## rowActions

Not a config key: row actions are registered on the plugin with `->rowActions([...])`. See
[Row actions](row-actions.md).

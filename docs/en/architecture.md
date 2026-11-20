# Architecture

An overview for integrators: enough to know what runs when, what is cached, and which parts you may
build on. The shortcut logic itself knows nothing about Filament; everything Filament-specific sits
behind an interface.

## Layers

- **Core** (`src/Core`): plain PHP. The five shortcut sets, the resolution pipeline that turns them into
  a keymap, and value objects. No framework imports, no URLs, no CSS selectors.
- **Application** (`src/Application`): the use cases an interface calls: `EditUserMap` (personal
  edits), `AuthorSystemMap` (gated preset authoring) and `SelectActiveMap`. Mode and authorization
  checks live here, so every caller gets them, whether it is the shipped page or your own controller.
- **Adapters**: `Persistence` (Eloquent), `Filament` (the panel, the pages, the JSON serializer),
  `Caching` (Laravel's cache) and `Authorization` (Laravel's gate).
- **Client** (`resources/js`, published by `filament:assets`): reads the keymap and handles the keys.

## What happens on a page render

The plugin hooks `PanelsRenderHook::PAGE_END`, the one render hook that runs inside the page's Livewire
component, where the page and its actions can be read. On each render it:

1. builds the sets for this request (the panel's text direction depends on the request locale);
2. resolves the **panel-wide** sets, navigation and custom bindings, through the cache;
3. resolves the **page** sets, header actions, table keys and row actions, fresh;
4. merges the two and serializes them to JSON, giving each binding its physical key code and an
   activation: navigate to a URL, click a selector, or run a table behaviour;
5. emits a `<script type="application/json" id="filament-shortcut-keys-map">` block and the cheatsheet
   markup.

Both halves read the admin's active map, so stored edits apply to both. Resolution runs in a fixed
order for each binding: discover, apply the overlay, apply the active map, add custom bindings, drop
disabled ones, assign letters per pool. See [how the layers combine](customization.md#how-the-layers-combine).

## The cached map and the page sets

Navigation and custom bindings are identical on every page of a panel, so their result is cached. The
cache key is a fingerprint of the panel, its navigation, the active map's identity and version, the
developer config, the locale and the cache format; it does not include the user, so admins on the same
map share one entry. See [the cache](configuration.md#cache).

Header actions, table keys and row actions differ per page and are cheap to build, so they are never
cached. Row actions letter themselves over the full registered list before the page is resolved, and
their letters are passed on as claimed, which is why header-action letters do not depend on the tables
a page has.

The "Keyboard shortcuts" page resolves the same way away from any live page: the panel-wide half hits
the same cache entry, and the table half is resolved against a table with no page around it.

## The client

One `keydown` listener on the document, which survives Livewire's SPA navigation. On each navigation
it re-reads the freshly injected keymap, resets the row cursor and any pending `G`, and repaints the
key badges. Handlers are keyed by set (`custom`, `table`, `global`, `navigation`) and tried in that
order. Every lookup built from a selector is guarded, so a broken custom binding finds nothing rather
than breaking the other shortcuts.

## Internal interfaces

The resolution pipeline runs on interfaces in `Core\Contracts`: `ShortcutSet`, `PageContextProvider`,
`NavigationProvider`, `ReservesKeys`, `AcceptsCustomBindings` and `Resolver`. They are marked
`@internal`. They are the pipeline's own seams, not an extension point: the plugin builds every set and
provider itself, so they may gain methods in a minor release. Do not implement them in your app.

The persistence interfaces (`MapRepository`, `MapEditor`, `SystemMapAuthor`, `ListMaps`, `MapSelector`)
and the `AuthorityGate` are bound in the container by the service provider.

## The three tables

| Table | Holds |
| --- | --- |
| `shortcut_maps` | Every map: system presets (one per panel may be the default) and admins' personal maps, each with a version that bumps on every change |
| `shortcut_map_entries` | One override per target per map: a letter, a disable, or a custom binding's payload |
| `shortcut_map_selections` | Which map each admin uses, one per admin per panel |

A map stores only differences; anything it does not mention follows the conventions. Owners are stored
as a morph type and id string, so integer, UUID and ULID user keys all work. A target is stored as
`set:key` (a class name or an action name), never a slug or a label.

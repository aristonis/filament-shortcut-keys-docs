# Troubleshooting

Work top down. Most reports end in the first three sections, and many of those are the design working
as intended.

## No key works anywhere

Almost always the plugin's script was never published. The keymap is rendered on the server, so it is
in the page either way and everything looks installed. Check that the script is served:

```bash
curl -I https://your-panel.test/js/filament-shortcut-keys/filament-shortcut-keys.js
```

A 404 confirms it. Run `php artisan filament:assets` and reload. To keep it published across updates,
make sure your `composer.json` runs `@php artisan filament:upgrade` in `post-autoload-dump` (see
[Getting started](getting-started.md#publish-the-assets)).

If the script is served and still nothing fires, check that the keymap is in the page: view the source
and look for `filament-shortcut-keys-map`. It is injected at the end of every panel page.

## Only bare keys and `G` sequences are dead

The user may have switched single-key shortcuts off in the `?` cheatsheet. The switch is at the bottom
of the cheatsheet; the choice is saved in that browser's local storage under
`filament-shortcut-keys:single-keys`. Shortcuts on held modifiers keep working when it is off.

## A key does nothing while typing

By design. Bare keys and `G` sequences are ignored while focus is in an input, textarea, select or
contenteditable element, while a modal, an open select, a dropdown, a menu or a tab list has the
keyboard, and for `Enter` or `Space` on a focused button. The usual surprise is focus left in the
table's search box after `/`: keys there type instead of act. Press `Tab` out of it or click the page.

`G` is two presses: `G`, let go, then the letter within about a second and a half. `Alt`+`G` belongs
to the browser.

Right after a shortcut clicks a control that sends a Livewire request, bare keys and `G` sequences are
ignored until the response arrives, for at most about two seconds. A key pressed in that window is
dropped, not queued.

See [When keys are ignored](shortcuts.md#when-keys-are-ignored) for the full list.

## The key fires but nothing happens

A shortcut clicks a real control or follows a real route. When nothing happens, usually there was
nothing to click:

- **The control is not rendered** because the admin is not authorized for it. Intended.
- **The route is refused.** The page opens with a 403. Also intended.
- **The table does not offer that action.** `Enter` needs an edit action on the row, `Delete` a delete
  action, `/` a search field.
- **No row is focused.** `Space`, `Enter`, `Delete` and row actions act on the cursor's row. Start the
  cursor with `J` or `K` first.
- **The arrows scroll the page instead of moving rows.** They move the cursor only while it is live.
  Press `J` or `K` to start it, and again after clicking elsewhere or pressing `Esc`.

A useful check: does clicking the control with the mouse work for this admin? If not, the shortcut is
behaving correctly.

## The shortcut is missing

First check whether it is in the keymap. In the browser console:

```js
JSON.parse(document.getElementById('filament-shortcut-keys-map').textContent)
```

Each group lists its set, its held modifiers, its leader key (null when there is none) and its
bindings, each with a target and a key code. If the binding is there, the problem is in the page (see
the sections above). If it is missing:

- **A header action that runs on the click itself** gets no default key on bare letters. Give it a
  confirmation or a modal, or pin a letter in the [`overlay` config](configuration.md#overlay). An
  action whose only modal is a form schema counts as instant.
- **A hidden or disabled header action**, or one inside an action group, gets no key.
- **A row action** is bound only if you registered its name and the page's table has it. See
  [Row actions](row-actions.md#why-a-registered-action-has-no-key).
- **A page or resource** gets a key only if it is registered on the panel. Items added with
  `$panel->navigationItems([...])` get none.
- **A shortcut disabled** in the overlay, the admin's map or the active preset is removed.
- **Two forced letters landed on one key** (an overlay pin, a stored remap, or a row action's letter):
  the later one in pool order loses its key. See [forced letters](configuration.md#forced-letters).
- **The pool ran out of letters.** A pool has 26 letters (23 for bare keys by default); shortcuts beyond that get
  none.
- **The page cannot see the table.** A table placed through `content()`, `Livewire::make()` or
  `getWidgetsSchemaComponents()` is not detected.

## A different letter than expected

- **A forced letter was ignored** because it is not a–z or sits on a reserved key (`G`, `J`, `K` or
  another table key among bare keys). The shortcut keeps its automatic letter.
- **An automatic letter moved.** A forced letter took its key, or a screen added earlier in the
  navigation sort took it first. Pin letters people rely on.
- **A row action got a different letter** after the `->rowActions([...])` list was reordered. Order
  decides who picks first.
- **A config change moved letters.** Two sets put on the same scheme share one pool, which prevents
  duplicates but reassigns letters.
- **A stored remap seems ignored.** An overlay disable beats it. A letter the overlay merely pins is
  replaced by the stored one.

## The wrong thing fires

When one key could match more than one active set, the browser side resolves in a fixed order:
`custom > table > global > navigation`. Sets on different modifiers never collide, and with the shipped
sets each pool is lettered once, so this is rare. If it happens after a `modifiers` change, the
change put two sets on keys that overlap.

## The keymap is stale

The cache key changes whenever the navigation, the active map's version, the overlay, the `modifiers`
config or the locale changes, so a stale map almost always means something did not move:

- **Rows written to the tables directly.** Every content change must bump the map's `version`. The
  plugin's own pages and use cases do; raw SQL does not.
- **Config cached in production.** Editing `config/shortcut-keys.php` changes the key, but only once
  Laravel reads the new file: run `php artisan config:cache` or `config:clear`.
- **Navigation changes** (adding, removing, relabelling or re-sorting a resource or page) change the
  key by themselves.

## Pagination feels backwards

In a right-to-left panel the paging arrows are mirrored to match Filament's flipped buttons. If they
feel wrong, check the panel's direction (`filament-panels::layout.direction`) rather than the plugin.
See [Translations](translations.md#right-to-left-panels).

## The key works but shows no badge

Only navigation links, header actions and custom bindings get a badge, and `G` badges appear only while
`G` waits. Table keys and row actions never get one.

For a shortcut that clicks, the badge and the key use the same element, so a missing badge means a
dead key too: see [the key fires but nothing happens](#the-key-fires-but-nothing-happens). For a
shortcut that navigates, the badge looks for a link whose `href` matches the keymap's URL exactly.
The key can work while the badge finds nothing; compare the URL in the keymap with the link's `href`.

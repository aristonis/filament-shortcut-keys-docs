# Troubleshooting

## Introduction

Work through these sections from the top. Most problems are solved in the first three, and many turn out to be the plugin working as intended.

## No key works anywhere

The plugin's script is almost always missing. The keymap is rendered on the server, so it is in the page either way and everything looks installed. Check that the script is served:

```bash
curl -I https://your-app.test/js/filament-shortcut-keys/filament-shortcut-keys.js
```

A 404 response confirms it. Run `php artisan filament:assets` and reload the page. To keep the script published across updates, make sure your `composer.json` runs `filament:upgrade` after each update. See [Publishing the assets](installation.md#publishing-the-assets).

If the script is served and still no key works, check that the keymap is in the page. View the page source and look for `filament-shortcut-keys-map`.

## Single keys and G sequences do nothing

The user may have turned off single-key shortcuts. Press `?` and check the **Single-key shortcuts** switch at the bottom of the cheatsheet. The choice is saved per browser. See [Turning off single-key shortcuts](accessibility.md#turning-off-single-key-shortcuts).

## A key does nothing while typing

This is intended. Single keys and `G` sequences do not fire while focus is in a text field, a select or an editable element, inside a modal, a dropdown, a menu or a tab list, or when `Enter` or `Space` is pressed on a focused button or link. The most common case is focus left in the table's search field after pressing `/`: keys typed there go into the field. Press `Tab` to leave it, or click the page.

`G` is two separate presses: press `G`, release it, then press the letter within a second and a half. `Alt`+`G` belongs to the browser.

Right after a shortcut clicks a control that sends a Livewire request, single keys and `G` sequences are ignored until the response arrives, for at most about two seconds. A key pressed in that window is dropped.

See [When shortcuts are ignored](shortcuts.md#when-shortcuts-are-ignored) for the full list.

## The key fires but nothing happens

A shortcut clicks a real control or follows a real URL. When nothing happens, there was usually nothing to click:

- **The admin is not authorized**, so Filament did not render the control. This is intended.
- **The page is refused** with a 403 response. This is also intended.
- **The table does not have the action.** `Enter` needs an `edit` record action, `Delete` a `delete` record action, `Space` a selection checkbox and `/` a search field.
- **No row is under the cursor.** `Space`, `Enter`, `Delete` and row actions act on the cursor's row. Press `J` or `K` first.
- **The arrow keys scroll the page.** They move the cursor and change pages only while the cursor is active. Press `J` or `K` to start it, and again after clicking elsewhere or pressing `Esc`.

A useful check: if clicking the control with the mouse does not work for this admin, the shortcut is behaving correctly.

## A shortcut is missing

First check whether it is in the keymap. Open the cheatsheet with `?`, or run this in the browser console:

```js
JSON.parse(document.getElementById('filament-shortcut-keys-map').textContent)
```

Each group lists its set and its bindings, each with its target and its key. If the shortcut is there, the problem is on the page, described in the sections above. If it is missing:

- **A header action that runs immediately** gets no default key. Give it a confirmation or a modal, or pin its letter in the [overlay](configuration.md#pinning-and-disabling-shortcuts). An action whose only modal content is a form counts as running immediately. See [Which header actions get a shortcut](shortcuts.md#which-header-actions-get-a-shortcut).
- **A hidden or disabled header action**, or one inside an action group, gets no key.
- **A row action** gets a key only when you registered its name and the page's table has it. See [Why a registered action has no key](row-actions.md#why-a-registered-action-has-no-key).
- **A page or resource** gets a key only when it is registered on the panel. Items added with `$panel->navigationItems()` get none.
- **The shortcut was disabled** in the overlay, in the admin's personal map, or in the preset they use.
- **Two pinned or stored letters landed on one key**, and this shortcut came second. See [How changes combine](customization.md#how-changes-combine).
- **The pool ran out of letters.** A pool holds 26 letters, and the default single-key pool holds 23.
- **The page cannot see the table.** A table placed through `content()`, `Livewire::make()` or `getWidgetsSchemaComponents()` is not detected.

## A shortcut has an unexpected letter

- **A pinned or stored letter was ignored** because it is not `a` to `z`, or because it is `G`, `J` or `K` among single keys.
- **An automatic letter moved**, because a pinned letter took its key, or because a page added earlier in the navigation sort took it first. Pin the letters people rely on.
- **A row action's letter changed** after the `rowActions()` list was reordered. The order decides who chooses first.
- **The letter differs between languages**, because navigation letters come from the translated label. See [Letters in other languages](localization.md#letters-in-other-languages).
- **An admin's change seems ignored.** An overlay disable overrides it. A letter the overlay only pins is replaced by the admin's letter.

## The wrong shortcut fires

When the same key matches shortcuts in more than one set, the plugin tries them in a fixed order: custom bindings, then table keys and row actions, then header actions, then navigation. With the default `modifiers` this cannot happen, because each pool is lettered once and the table keys are reserved. If it happens after you changed `modifiers`, the change put two sets on overlapping keys.

## Changes do not appear

The cached keymap is replaced automatically when the panel's resources and pages, an admin's map, the `overlay` or `modifiers` options, or the locale change. If a change does not appear:

- **The configuration is cached.** Run `php artisan config:cache` or `php artisan config:clear` after editing `config/shortcut-keys.php`.
- **The database was changed directly.** Changes made through the **Manage shortcuts** page, the `EditUserMap` and `AuthorSystemMap` classes, and the `shortcut-keys:prune` command are picked up immediately. Rows written to the plugin's tables some other way are not.

## The paging keys feel reversed

In a right-to-left panel the paging keys are mirrored to match Filament's flipped buttons. If they feel wrong, check the panel's direction, which comes from Filament's `filament-panels::layout.direction` translation. See [Right-to-left panels](localization.md#right-to-left-panels).

## A key works but shows no badge

Only navigation links, header actions and custom binding targets get a badge, and badges for `G` shortcuts appear only while `G` waits for its letter. Table keys and row actions never get one.

For a shortcut that clicks, the badge and the key use the same element, so a missing badge means the key has nothing to click either. See [The key fires but nothing happens](#the-key-fires-but-nothing-happens). For a shortcut that opens a page, the badge goes on a link whose `href` matches the shortcut's URL exactly. The key can work while no such link is on the page.

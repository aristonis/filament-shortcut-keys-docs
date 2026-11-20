# Row actions

Table record actions are per row, so the plugin only binds the ones you name. Register the names once
and they work on every table in the panel that exposes an action of that name, relation manager and
widget tables included, firing against the row the cursor is on.

## Register

```php
use Aristonis\FilamentShortcutKeys\FilamentShortcutKeysPlugin;

$panel->plugin(
    FilamentShortcutKeysPlugin::make()
        ->rowActions(['approve', 'reject', 'suspend'])
);
```

These are names only, matched against the action names your tables already declare. You are not
defining actions here, only saying which existing ones deserve a key.

Pressing the key clicks the real Filament record action on the cursor's row, with its own
authorization and confirmation. Start the cursor with `J` or `K` first.

## How letters are assigned

Each name gets a bare letter, assigned across the **full** registered list rather than per page. So
`approve` keeps the same key whether or not `reject` happens to be on the current table, and the key
does not move as you navigate.

The list is ordered. Each name takes the first free letter of the name itself, so whoever comes first
picks first:

```php
->rowActions(['reject', 'refund'])   // reject: R, refund: E
->rowActions(['refund', 'reject'])   // refund: R, reject: E
```

Sorting the array alphabetically during a cleanup therefore rekeys the panel. If a letter has to
survive a reordering, pin it in the [config overlay](configuration.md#overlay):

```php
'overlay' => [
    'row-action:refund' => ['letter' => 'f'],
],
```

Row actions are lettered before the overlay applies, so their letters count as pinned. Pinning one onto
a letter a sibling row action already holds leaves two pinned letters on one key, and the one later in
`->rowActions([...])` loses its key. With `['approve', 'reject']`, pinning `approve` to `r` gives
approve `R` and leaves reject with no key. Pin both, or pick a letter no sibling has. Row actions do
step around letters the overlay pins for header actions.

## Reserved keys

Row actions share the bare pool with header actions, so the two never collide. A row action never gets
a key reserved in that pool: `G` (the navigation leader), `J` and `K` (the row cursor) and the other
table keys. Putting row actions on modifiers of their own, for example `'row-action' => ['alt']`,
frees those letters.

No header action is lettered automatically onto a registered row action's letter, on any page,
including pages with no table. Registering a new name can therefore move a header action's letter
panel-wide, but adding a header action never moves a row action's letter, and adding a relation
manager or a table widget to a page never moves a header action's.

## Edit, delete and view

`edit` and `delete` are already bound to `Enter` and `Delete` by the table keys, so leave them out.
Registering them does nothing.

`view` is not bound by default. If you want a key for it, register `view` like any other name.

## Nested tables

On a page whose only table is its own, a row action is bound only when that table exposes a record
action of that name. Everywhere else it is neither bound nor listed, so the cheatsheet never advertises
a dead key.

Relation managers and table widgets load after the page, so the page cannot know their actions when it
builds the keymap. On a page with one, every registered name is bound. The cheatsheet checks the page
each time it opens and lists a row action only while a row carrying it is on the page; an empty or
filtered-out table shows none. There it is labelled from its name (`mark_paid` reads "Mark paid"),
because the page cannot read a label the action sets itself.

## Why a registered action has no key

In order of likelihood:

1. **The table does not declare the action.** The page filter is working as intended.
2. **The action is a header action, not a record action.** Header actions belong to the actions set.
3. **The action is inside an action group.** Filament flattens record-action groups, so a grouped row
   action is still bound and listed; whether the key reaches a control inside a collapsed dropdown has
   not been verified.
4. **The user is not authorized for it**, so Filament never rendered the control. The key exists and
   does nothing, which is the intended security model.
5. **The page cannot see the table.** Only a page's own table, its relation managers, and header,
   footer or dashboard table widgets are detected. A table placed through the page's `content()`
   schema or `Livewire::make()`, or a widget placed through `getWidgetsSchemaComponents()`, gets no
   table keys and no row actions unless the page is a table page itself.
6. **Its letter was taken.** Another pinned letter on the same key won (see above), or the bare pool
   ran out of letters.

The "Manage shortcuts" page does not offer row-action keys. Move them with the config overlay.

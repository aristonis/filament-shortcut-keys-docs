# Row actions

## Introduction

Table record actions repeat on every row, so the plugin only gives a shortcut to the ones you name. You register the names once on the plugin, and each one gets a letter that works on every table in the panel that has a record action of that name, including relation managers and table widgets. Pressing the letter clicks the action on the [cursor's row](table-shortcuts.md#using-the-row-cursor).

## Registering row actions

Pass the action names to the `rowActions()` method when you register the plugin:

```php
use Aristonis\FilamentShortcutKeys\FilamentShortcutKeysPlugin;
use Filament\Panel;

public function panel(Panel $panel): Panel
{
    return $panel
        // ...
        ->plugin(
            FilamentShortcutKeysPlugin::make()
                ->rowActions(['approve', 'reject', 'suspend'])
        );
}
```

These are the names your tables already give their record actions, such as `Action::make('approve')`. You are not defining actions here, only choosing which existing ones get a key.

Pressing the key clicks the real Filament action on the cursor's row, so the action's authorization, visibility and confirmation apply as they would to a click. Record actions inside an action group are included. See [Limitations](limitations.md#what-gets-a-shortcut).

### Edit, delete and view

`edit` and `delete` already have the `Enter` and `Delete` [table keys](table-shortcuts.md#the-table-keys), so registering them has no effect.

`view` has no key by default. To give it one, register `view` like any other name.

## How letters are assigned

Row action letters are assigned across the whole list you register, not per page. `approve` keeps the same key whether or not `reject` is on the current table, so the key does not change as you move around the panel.

The list is ordered. Each name takes the first free letter of the name itself, so a name earlier in the list chooses first:

```php
->rowActions(['reject', 'refund'])   // reject: R, refund: E
->rowActions(['refund', 'reject'])   // refund: R, reject: E
```

Sorting the list differently therefore changes the keys.

Row actions share the single-key pool with header actions, so the two never collide. No row action takes `G`, `J` or `K`, and no header action is ever given a registered row action's letter automatically, on any page. Adding a header action never moves a row action's letter, and adding a relation manager or a table widget to a page never moves a header action's letter. Registering a new row action can move a header action's letter.

## Pinning a row action's letter

To keep a letter when the list changes, pin it in the [overlay](configuration.md#pinning-and-disabling-shortcuts):

```php
'overlay' => [
    'row-action:refund' => ['letter' => 'f'],
],
```

!!! warning
    A pinned letter that another registered row action already holds leaves two row actions on one key. The one later in the `rowActions()` list loses its key. With `['approve', 'reject']`, pinning `approve` to `r` gives approve `R` and leaves reject with no key. Pin both, or choose a letter no other row action has.

Row actions automatically step around letters pinned for header actions.

## Giving row actions their own modifier

Row actions use single keys by default. To free `G`, `J` and `K` for them, or to keep them apart from header actions, put them on a held modifier with the [`modifiers` option](configuration.md#changing-how-keys-are-pressed):

```php
'modifiers' => [
    'row-action' => ['alt'],
],
```

## Row actions on relation managers and widgets

On a page whose only table is its own, a row action gets a key only when that table has a record action of that name. On other pages it has no key and does not appear in the cheatsheet, so the cheatsheet never lists a key that does nothing.

Relation managers and table widgets load after the page, so the page cannot know their actions when it builds the keymap. On a page with one, every registered name gets its key. The cheatsheet checks the page each time it opens and lists a row action only while a row with that action is on the page. An empty or filtered-out table shows none.

In that case the cheatsheet labels the row action from its name, so `mark_paid` reads "Mark paid", because the page cannot read a label the action sets itself.

## Why a registered action has no key

If a row action you registered has no key, check these causes in order:

1. The table does not have a record action with that name.
2. The action is a header action, not a record action. Header actions get keys of their own, described in [Shortcuts](shortcuts.md#running-header-actions).
3. The admin is not authorized for the action, so Filament does not render it on the row. The key exists and does nothing, which is intended.
4. The page cannot see the table. See [Pages with several tables](table-shortcuts.md#pages-with-several-tables).
5. The letter was taken by another pinned letter, or the single-key pool ran out of letters.

The **Manage shortcuts** page does not offer row action keys. Change them with the overlay.

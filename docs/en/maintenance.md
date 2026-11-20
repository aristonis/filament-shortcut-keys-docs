# Pruning dead overrides

A stored override (an admin's remap or disable, or a preset's) is keyed on the target's class or name,
not its slug or label, so it survives renaming and reordering. It does not survive a deletion: remove a
resource, a page or a registered row action and the rows that remapped it are left pointing at nothing.
They do no harm, but they accumulate. Clear them out with:

```bash
php artisan shortcut-keys:prune
```

## Options

```bash
php artisan shortcut-keys:prune --dry-run       # list what would be removed, delete nothing
php artisan shortcut-keys:prune --panel=admin   # audit one panel (repeat the option for several)
php artisan shortcut-keys:prune --force         # skip the confirmation in production
```

| Option | Effect |
| --- | --- |
| `--dry-run` | Prints the report and exits without deleting. |
| `--panel=` | Audits only the given panel ids. Without it, every panel that registers the plugin is audited. A panel id that does not exist, or whose panel does not use the plugin, fails with an error. |
| `--force` | Deletes without asking, even when the environment is production. |

The command always prints what it found before deleting anything: a table of map id and target for
each orphaned override, and a count of the overrides it left alone. In production it asks for
confirmation unless `--force` is given.

Deleting bumps the version of every affected map in the same transaction, so cached keymaps that still
carry the removed overrides stop being served.

## What it can and cannot judge

The command only judges targets it can list outside a web request:

- navigation targets: the panel's registered resources and pages;
- row actions: the names registered with `->rowActions([...])`;
- the fixed table behaviours.

Overrides of those that no longer exist are removed.

Header-action overrides and custom bindings are reported and left in place. A header action only
exists while its page renders, so a maintenance pass cannot tell a removed action from one that is
simply not on the page it is looking at, and a custom binding belongs to the admin who made it.
Deleting on a guess would destroy live data, so the command does not.

To remove one of those, call `EditUserMap::enable()` for that target on the owner's behalf (see
[driving it from code](custom-bindings.md#driving-it-from-code)), which drops the row and bumps the
map's version. If you delete rows from `shortcut_map_entries` yourself, bump the map's `version` in
the same transaction; without it the cached keymap keeps serving the old row.

## When to run it

After removing a resource, a page or a registered row action, or as an occasional scheduled task:

```php
// routes/console.php
Schedule::command('shortcut-keys:prune --force')->weekly();
```

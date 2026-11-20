# Commands

## Introduction

The plugin ships one Artisan command, `shortcut-keys:prune`, which removes stored changes to shortcuts that no longer exist.

A change an admin or a preset makes to a shortcut, such as a remapped letter or a disabled shortcut, is stored against the resource class, page class or action name, not against its URL or label. It survives renaming and reordering. It does not survive deletion: when you remove a resource, a page or a registered row action, the stored changes for it are left behind. They do no harm, but they accumulate.

## Pruning orphaned overrides

To remove them, run:

```bash
php artisan shortcut-keys:prune
```

By default, the command checks every panel that registers the plugin. It prints what it found before it deletes anything. In the `production` environment, it asks for confirmation before deleting.

In [`config` mode](customization.md#config-mode), nothing is stored, so the command checks no panel. It prints `Customization is 'config', so nothing is stored and there is nothing to prune.` and exits successfully, so a scheduled prune keeps working.

### Options

| Option | Effect |
| --- | --- |
| `--dry-run` | Print what would be removed and exit without deleting anything. |
| `--panel=` | Check only the given panel. Repeat the option to check several panels. |
| `--force` | Delete without asking for confirmation, even in production. |

For example, to see what would be removed from the `admin` panel:

```bash
php artisan shortcut-keys:prune --panel=admin --dry-run
```

If a panel ID passed to `--panel` does not exist, or that panel does not register the plugin, the command prints an error and exits with a failure code without checking any panel.

### Output

When nothing is orphaned, the command prints `No orphaned overrides found.` Otherwise it prints a table with the map and the shortcut of each orphaned change:

```text
+-----+------------------------------------------------+
| Map | Target                                         |
+-----+------------------------------------------------+
| 3   | navigation:App\Filament\Resources\PostResource |
| 7   | row-action:archive                             |
+-----+------------------------------------------------+
```

After deleting, it prints how many changes it removed. Keymaps cached before the prune are rebuilt on the next request, so you do not need to clear the cache.

## What the command can check

The command removes a change only when it can prove the shortcut is gone. It can list these shortcuts outside a web request:

- navigation shortcuts: the resources and pages registered on the panel,
- row actions: the names registered with `rowActions()`,
- the table keys.

Changes to header actions and custom bindings are counted and left in place, and the command prints how many it skipped. A header action only exists while its page renders, so the command cannot tell a removed action from one that is not on the page it looks at. A custom binding belongs to the admin who created it.

To remove one of those changes from an admin's personal map, call `EditUserMap::enable()` for that shortcut on the owner's behalf. See [Managing bindings from code](custom-bindings.md#managing-bindings-from-code).

## Scheduling the command

You may run the command after removing a resource, a page or a row action, or on a schedule. To run it weekly, add it to `routes/console.php`:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('shortcut-keys:prune --force')->weekly();
```

# Letting admins customize

## Introduction

Two separate settings control who may change shortcuts:

- The `customization` option decides whether each admin may change their **own** keys. It never affects anyone else.
- The `authorize` option decides who may edit the **shared presets** and choose the panel's default preset. It denies everyone by default, and being an admin does not grant it.

Both options live in `config/shortcut-keys.php`. See [Publishing the configuration](installation.md#publishing-the-configuration).

## Choosing a customization mode

Set the `customization` option to `personal` or `locked`:

```php
'customization' => 'personal',
```

### Personal mode

`personal` is the default. Every admin can open the **Manage shortcuts** page, choose between the presets you publish, remap and disable their own navigation shortcuts, and add [custom bindings](custom-bindings.md).

### Locked mode

`locked` turns personal editing off:

```php
'customization' => 'locked',
```

Admins use the panel's default preset, which an authorized admin sets on the **Manage shortcuts** page. When no default is set, they get the automatic shortcuts.

An admin who already had a selection when you switched the panel to `locked`, either a preset they picked or a personal map, keeps it and can no longer change it.

In `locked` mode, only admins who pass the [authoring gate](#granting-authoring-access) can open the **Manage shortcuts** page.

## The Manage shortcuts page

In `personal` mode, every admin sees these actions on the **Manage shortcuts** page:

| Action | What it does |
| --- | --- |
| Change active map | Switch to one of the panel's presets, or back to "Your customizations" |
| Remap a shortcut | Put a navigation shortcut on another letter |
| Disable a shortcut | Remove a navigation shortcut |
| Reset to default | Remove your change to one shortcut |
| Add custom binding | Bind a key to a page in the app or an element on the page. See [Custom bindings](custom-bindings.md). |

The page offers navigation shortcuts only. Header action, table and row action keys are changed by a developer through the [overlay](configuration.md#pinning-and-disabling-shortcuts).

A letter outside `a` to `z` is ignored, and the shortcut keeps the letter it would otherwise have.

## How personal maps work

An admin never edits a shared preset. Their first change copies the map they are currently using, either the preset they selected or the panel default, into a personal map, selects it, and saves the change there. The preset itself is not changed.

A personal map stores only the admin's changes. Everything it does not mention keeps its automatic shortcut, so a resource you add next month still gets a key in a map an admin customized today.

The copy is made once. Later changes to the preset it came from do not reach the personal map. **Reset to default** removes the admin's change, so the shortcut goes back to its automatic letter, not to the preset's letter. To go back to a preset entirely, the admin selects it with **Change active map**. Their personal map stays in the list for later.

## Granting authoring access

Authoring lets an admin create and edit the shared presets and choose the panel's default preset. Grant it with the `authorize` option, either as the name of a [gate](https://laravel.com/docs/authorization#gates) ability or as a closure.

We recommend a gate ability. Define it in a service provider. The ability receives the panel ID as its argument:

```php
use App\Models\User;
use Illuminate\Support\Facades\Gate;

public function boot(): void
{
    Gate::define('manage-shortcut-keys', function (User $user, string $panelId): bool {
        return $user->is_developer;
    });
}
```

Then name the ability in the configuration file:

```php
'authorize' => 'manage-shortcut-keys',
```

The ability is checked as `Gate::forUser($user)->allows('manage-shortcut-keys', $panelId)`. A visitor who is not signed in is denied before the gate is asked. A `Gate::before()` callback, such as a super-admin rule, runs before every ability and can grant this one too.

You may use a closure instead. It receives the user, which may be `null`, and the panel ID:

```php
use Illuminate\Contracts\Auth\Authenticatable;

'authorize' => fn (?Authenticatable $user, string $panelId): bool => $user?->is_developer ?? false,
```

Only a return value of exactly `true` grants authoring. A truthy value such as `1` or `'yes'` denies.

!!! warning
    A closure in a configuration file cannot be serialized, so `php artisan config:cache` fails while the closure is there. Use a gate ability in any application that caches its configuration.

`null`, the default, denies everyone, and so does any other value.

## Managing presets

An admin who passes the authoring gate sees four more actions on the **Manage shortcuts** page:

| Action | What it does |
| --- | --- |
| Create preset | Add a new, empty preset to the panel. It is not the default. |
| Clone preset | Copy an existing preset, including its changes |
| Edit preset | Put a navigation shortcut on another letter, for everyone using that preset |
| Set panel default | Choose the preset every admin without a selection of their own uses. In `locked` mode, that is everyone. |

Editing a preset and changing the default both change keys for other admins, so both forms ask for an explicit confirmation before they save.

A preset holds letters and disabled shortcuts, never custom bindings. The **Edit preset** form sets letters. To disable a shortcut in a preset, call `AuthorSystemMap::editInPlace()` from your own code, which runs the same authorization and confirmation checks.

Passing the gate on one panel gives no access to another panel's presets.

## Which map an admin uses

| Situation | Map used |
| --- | --- |
| No selection, no default preset | The automatic shortcuts and the overlay |
| No selection, a default preset is set | The default preset |
| Selected a preset (`personal` mode) | That preset |
| Changed a shortcut (`personal` mode) | Their personal map |
| `locked` mode | Their earlier selection, otherwise the default preset, otherwise the automatic shortcuts |

## How changes combine

A shortcut can be changed in three places: the [overlay](configuration.md#pinning-and-disabling-shortcuts) in your configuration file, the admin's active map (their personal map or a preset), and, for custom bindings, the admin's own bindings. They combine as follows:

- **An overlay disable always wins.** Nothing in a map can bring that shortcut back.
- **A letter stored in the admin's map replaces a letter pinned in the overlay.**
- **A shortcut disabled in the admin's map is removed**, even if the overlay pins a letter for it.
- **A disabled shortcut frees its letter** for the other shortcuts in its pool.
- **A pinned or stored letter takes its key from an automatic one.** The shortcut that had the key automatically moves to its next free letter.
- **When two pinned or stored letters land on the same key, the first one keeps it.** Navigation comes before custom bindings, and header actions before row actions. The other shortcut loses its key.
- **A letter that is not `a` to `z`, or that is a reserved key in its pool, is ignored.** The shortcut keeps the letter it would otherwise have.

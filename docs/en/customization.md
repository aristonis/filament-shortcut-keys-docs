# Letting admins customize

Two separate permissions control who changes keys:

- **Personal customization**, set by `customization` (`personal` or `locked`). It lets an admin change
  their *own* keys and touches nobody else.
- **Authoring**, set by `authorize`. It lets someone change the shared presets and choose the panel
  default. It is denied by default and is not implied by being an admin.

## Personal and locked

```php
// config/shortcut-keys.php
'customization' => 'personal', // or 'locked'
```

**`personal`** (the default) gives each admin the "Manage shortcuts" page, where they can switch
between the presets you publish and customize their own keys.

**`locked`** turns personal editing off. Admins use the panel's default preset, which an authorized
admin sets on the manage page; with no default set, they get the automatic conventions. An admin who
already had a selection when the panel was switched to `locked`, either a preset they picked or a map
they had customized, keeps it and can no longer change it. In `locked` mode the manage page is only
reachable for someone who passes the authoring gate.

## The Manage shortcuts page

In `personal` mode every admin sees these actions:

| Action | What it does |
| --- | --- |
| Change active map | Switch to one of the panel's presets, or back to "Your customizations" |
| Remap a shortcut | Put a navigation shortcut on another letter |
| Disable a shortcut | Remove a navigation shortcut |
| Reset to default | Drop your override for one shortcut |
| Add custom binding | Bind a key to a page in the app or an element on the page (see [Custom bindings](custom-bindings.md)) |

The page offers navigation targets only. Header-action, table and row-action keys are moved by a
developer through the [config overlay](configuration.md#overlay).

A remapped letter is ignored, and the shortcut keeps its automatic letter, when it is not a–z or sits on
a reserved key. See [forced letters](configuration.md#forced-letters).

## Forking on first edit

An admin never edits a shared map. Their first edit copies the map they are using (the selected preset,
or the panel default) into a private map, points their selection at it, and stores the change there.
The shared map is untouched.

A personal map stores only differences. Everything it does not mention follows the live conventions,
which is why a resource added next month still gets a key in a map customized today. It does not follow
the preset it was copied from: later edits to that preset do not reach it. "Reset to default" drops the
override, so the shortcut goes back to its automatic letter rather than the preset's. To go back to a
preset wholesale, select it with "Change active map"; the personal map stays in the list for later.

## The authoring gate

Grant authoring with a Gate ability (recommended) or a closure:

```php
// config/shortcut-keys.php

// the name of an ability you defined
'authorize' => 'manage-shortcut-keys',

// or a closure; needs `use Illuminate\Contracts\Auth\Authenticatable;` at the top of the file
'authorize' => fn (?Authenticatable $user, string $panelId) => $user?->isDeveloper() ?? false,
```

```php
// app/Providers/AppServiceProvider.php
Gate::define('manage-shortcut-keys', fn (User $user, string $panelId) => $user->is_developer);
```

Prefer the ability. A closure in a config file cannot be serialized, so `php artisan config:cache`
fails while one is there.

- A string is checked as `Gate::forUser($user)->allows($ability, $panelId)`. A visitor with no user is
  denied before the gate is asked. A `Gate::before` callback, such as a super-admin rule, runs before
  every ability and can grant this one too.
- A closure receives the user (possibly null) and the panel id, so one config can answer per panel. Only
  a strict `true` grants; a truthy value like `1` or `'yes'` denies.
- `null`, or anything else, denies everyone.

## What authors can do

Whoever passes the gate gets four more actions on the manage page:

| Action | What it does |
| --- | --- |
| Create preset | A new, empty preset for the panel. It is not the default. |
| Clone preset | A copy of an existing preset, including its overrides |
| Edit preset | Put a navigation shortcut on another letter, in place, for everyone on that preset |
| Set panel default | The preset every admin without their own selection uses; in `locked` mode, everyone |

Editing a preset in place and changing the default both change keys for other admins, so both forms
require an explicit confirmation checkbox before they save. A preset holds letters and disables, never
a custom binding. The edit form sets letters; a disable can be written from code with
`AuthorSystemMap::editInPlace()`, which runs the same gate and confirmation checks.

Passing the gate does not reach another panel's maps: every lookup is scoped to the current panel, and
a map id from another panel is treated as not found.

## How the layers combine

For each shortcut, in order:

1. The **conventions** discover it. Table keys arrive with their fixed keys; everything else has no
   letter yet. On bare keys, an instant header action starts switched off.
2. The **config overlay** pins a letter (which also switches on an instant header action) or disables
   it.
3. The admin's **active map** applies: their personal map, the preset they selected, or the panel
   default. A stored letter replaces a letter the overlay pinned; a stored disable turns the shortcut
   off. A stored letter cannot bring back a shortcut the overlay disabled.
4. The admin's **custom bindings** are added.
5. Disabled shortcuts are removed, so their letters go back to the pool.
6. **Letters are assigned** in each pool. Pinned and stored letters are placed first and take their key
   from any automatically lettered shortcut, which moves to another free letter.

So an overlay disable always wins, while an admin's stored letter beats an overlay letter. If two
pinned or stored letters land on one key, the first in pool order keeps it (navigation before custom
bindings, header actions before row actions) and the other loses its key.

## Who sees which map

| Situation | Map used |
| --- | --- |
| No selection, no default set | Conventions (plus the overlay) |
| No selection, default set | The panel default |
| Selected a preset (`personal` mode) | That preset |
| Has edited their keys (`personal` mode) | Their personal map |
| `locked` mode | Any earlier selection, otherwise the panel default, otherwise conventions |

# Custom bindings

## Introduction

A custom binding is a key an admin adds for themselves, on top of the automatic shortcuts. It either opens a page in your application or clicks an element on the current page. Custom bindings need [`personal` mode](customization.md#personal-mode).

Admins add a binding from the **Manage shortcuts** page with **Add custom binding**. The form asks for:

- a **Name**, up to 64 characters, which identifies the binding,
- a **Letter**,
- what the binding **Opens**: a page in this app, or an element on the page,
- the **Destination**: a path or a CSS selector.

Custom bindings use the same `G` leader as navigation and share its pool of letters, so a custom binding and a navigation shortcut never get the same key. Adding a binding with a name that already exists replaces that binding.

## Binding a key to a page

A binding that opens a page takes a path in your application:

```text
/admin/orders
```

The path must start with a single `/`. A full URL, a protocol-relative URL such as `//example.com`, or a scheme such as `javascript:` is rejected when the binding is saved, so a binding can never send an admin to another site.

## Binding a key to an element

A binding that clicks an element takes a CSS selector:

```text
#refresh-button
```

Pressing the key clicks the first element on the current page that matches the selector. The selector is only checked for being a non-empty string when it is saved. A selector that is not valid CSS, or that matches nothing on the current page, does nothing when pressed, and the other shortcuts keep working.

## Choosing the letter

The binding takes the letter the admin entered, with these exceptions:

- A letter outside `a` to `z` is ignored, and the binding gets an automatic letter from its name instead.
- A navigation shortcut whose letter is pinned or stored on the same key keeps it, and the binding loses its key, because navigation comes first in the pool.

A binding's letter takes its key from a navigation shortcut that has the key automatically. That navigation shortcut moves to its next free letter.

## Badges

While `G` waits for its letter, the binding's badge appears on the element it clicks, or on the link whose `href` matches the path exactly. A path with no matching link on the page still works, it just shows no badge.

## Security

A custom binding can only click something the page already rendered, or open a path your application already protects. Filament does not render a control the admin may not use, and your middleware stops a page the admin may not open. A binding never calls a Livewire method or a server action directly.

Presets never accept custom bindings, so one admin cannot publish a selector or a path to everyone else. See [Security](security.md#custom-bindings-click-they-do-not-authorize).

## Managing bindings from code

The **Manage shortcuts** page cannot remove a custom binding. If you need to remove one, or you are building your own interface, use the `EditUserMap` class, which is what the page uses. It takes the customization mode as a string:

```php
use App\Filament\Resources\OrderResource;
use Aristonis\FilamentShortcutKeys\Application\EditUserMap;
use Aristonis\FilamentShortcutKeys\Core\Contracts\MapEditor;
use Filament\Facades\Filament;

$editor = new EditUserMap(app(MapEditor::class), config('shortcut-keys.customization'));

$user = Filament::auth()->user();
$type = $user->getMorphClass();
$id = (string) $user->getAuthIdentifier();

$editor->remap($type, $id, 'admin', 'navigation:' . OrderResource::class, 'o');
$editor->disable($type, $id, 'admin', 'global:export');
$editor->addCustomBinding($type, $id, 'admin', 'custom:reports', 'r', ['route' => '/admin/reports']);

// Remove a change or a custom binding...
$editor->enable($type, $id, 'admin', 'custom:reports');
```

The third argument is the panel ID. The fourth is the shortcut's name: `navigation:` followed by the resource or page class, `global:` followed by a header action name, or `custom:` followed by a binding name. A custom binding takes exactly one of `['route' => ...]` or `['selector' => ...]`, under the same rules as the form.

`enable()` removes the stored change instead of saving a new one. The shortcut goes back to its default letter, from the overlay or the automatic assignment, and follows any later change to them. For a custom binding, `enable()` removes the binding.

To switch an admin to another map, use the `SelectActiveMap` class:

```php
use Aristonis\FilamentShortcutKeys\Application\SelectActiveMap;
use Aristonis\FilamentShortcutKeys\Core\Contracts\MapSelector;

(new SelectActiveMap(app(MapSelector::class), config('shortcut-keys.customization')))
    ->select($type, $id, 'admin', $mapId);
```

Each admin has one active map per panel. Selecting a map that is neither a preset of that panel nor the admin's own personal map fails with a `ModelNotFoundException`.

Both classes throw an exception that extends `Aristonis\FilamentShortcutKeys\Exceptions\ShortcutKeysException` when they refuse a change:

- `EditingLockedException` for every call in `locked` or `config` mode,
- `InvalidCustomBindingException` for a custom binding that breaks the rules above.

!!! warning
    Always take the owner from the authenticated user, never from request input. The type and ID decide whose map is written, so taking them from a route or form parameter would let one admin rewrite another admin's map.

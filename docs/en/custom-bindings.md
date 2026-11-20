# Custom bindings

A custom binding is a key an admin adds for themselves, on top of the automatic ones. It needs
`personal` mode. Admins add them from the "Manage shortcuts" page with **Add custom binding**: a name,
a letter, what it opens, and the destination.

Custom bindings use the same `G` leader as navigation and share its letter pool, so the two never
collide.

The name identifies the binding: adding another binding with the same name replaces it. The manage
page has no action to remove or disable a custom binding; that is done from code with `enable()`
(see [driving it from code](#driving-it-from-code)).

## Route and selector

A binding points at exactly one of two things:

```php
['route' => '/admin/orders']         // navigates to a page in this app
['selector' => '#refresh-button']    // clicks an element already on the page
```

The rules, checked when the binding is saved:

- The payload has exactly one key, `route` or `selector`, never both and nothing else.
- The value is a non-empty string.
- A route is host-relative: it starts with a single `/`. A scheme (`https:`, `javascript:`) or a
  protocol-relative `//` is rejected, which blocks off-site redirects.

A selector is only checked for shape. A selector that is not valid CSS, or matches nothing on the
current page, does nothing when pressed; the rest of the keymap keeps working.

The binding's badge sits on the clicked element, or on the link whose `href` matches the route exactly.
A route with no matching link on the page still works; it just gets no badge.

## The security boundary

These two shapes are the whole surface, and that is deliberate. A binding can only click something the
page already rendered or visit a route the app already guards. Filament does not render a control the
admin cannot use, so there is nothing to click, and your middleware stops a route they cannot open. A
binding never calls a Livewire method or a server action directly. See [Security](security.md).

Shared presets never accept a custom binding. Publishing an admin-supplied selector or route to every
admin on a preset is refused whoever asks.

## When a binding does nothing

- **The letter is not a–z, or sits on a reserved key.** The letter is ignored and the binding gets an
  automatic letter from its name instead.
- **Another pinned or stored letter holds the key.** Navigation comes before custom bindings in the
  pool, so a navigation shortcut pinned to the same letter keeps it and the binding loses its key. A
  binding's letter does take its key from an automatically lettered navigation shortcut, which moves.
- **The selector matches nothing** on the current page, or is not valid CSS.
- **The route is refused.** A route the admin cannot open navigates there and gets your app's 403.

## Driving it from code

Admins normally do all of this on the manage page. If you build your own interface, call the same
use cases the page uses. They take the customization mode as a plain string, so construct them
yourself:

```php
use Aristonis\FilamentShortcutKeys\Application\EditUserMap;
use Aristonis\FilamentShortcutKeys\Core\Contracts\MapEditor;
use App\Filament\Resources\OrderResource;
use Filament\Facades\Filament;

$editor = new EditUserMap(app(MapEditor::class), config('shortcut-keys.customization'));

$user = Filament::auth()->user();
$type = $user->getMorphClass();
$id = (string) $user->getAuthIdentifier();

$editor->remap($type, $id, 'admin', 'navigation:' . OrderResource::class, 'o');
$editor->disable($type, $id, 'admin', 'global:export');
$editor->enable($type, $id, 'admin', 'global:export');   // drops the override, back to the convention
$editor->addCustomBinding($type, $id, 'admin', 'custom:reports', 'r', ['route' => '/admin/reports']);
```

The third argument is the panel id. A custom binding's target must start with `custom:`; the rest is
its name, which also seeds its automatic letter. A payload that breaks the rules above throws
`InvalidCustomBindingException`, and in `locked` mode every call throws `EditingLockedException`. Both
live in `Aristonis\FilamentShortcutKeys\Exceptions` and extend `ShortcutKeysException`.

`enable()` removes the override instead of storing a no-op row, so the shortcut follows whatever the
convention says from then on rather than freezing at today's letter.

To switch an admin's active map:

```php
use Aristonis\FilamentShortcutKeys\Application\SelectActiveMap;
use Aristonis\FilamentShortcutKeys\Core\Contracts\MapSelector;

(new SelectActiveMap(app(MapSelector::class), config('shortcut-keys.customization')))
    ->select($type, $id, 'admin', $mapId);
```

Each admin has one active map per panel. Selecting a map that is neither a preset of that panel nor the
admin's own personal map fails with a not-found.

**Take the owner from the authenticated user, never from request input.** These arguments decide whose
map is written. Taking them from a route or body parameter would let one admin rewrite another's map.

# Security

## Introduction

A shortcut only ever does what the admin could have done with the mouse. It clicks the real Filament control or follows the real URL. It never creates an action of its own and never calls a Livewire method directly, so your policies, middleware and confirmation modals apply exactly as they would to a click.

## The keymap is the same for every admin

The navigation shortcuts are built from the resources and pages in the panel's sidebar, without checking whether the current admin may open them. Admins who see the same sidebar receive the same navigation shortcuts. If your `shouldRegisterNavigation()` depends on the signed-in admin, each admin gets keys for the items in their own sidebar. When a key is pressed, your own authorization decides what happens:

- Filament does not render a header action or row action the admin is not authorized for, so its key has nothing to click.
- A page the admin may not open is stopped by your middleware or Filament's access check, so its key leads to a 403 response.

Filtering the keymap per admin would add no protection, because access is checked when the key is used. As a result, the keymap reveals which pages exist, but not what an admin may do. This includes the plugin's own **Manage shortcuts** page, which appears in the keymap even for an admin who cannot open it.

!!! warning
    If the existence of a page must stay hidden from some admins, do not rely on the shortcut layer to hide it. Such a page does not belong in the panel's registered pages.

## Custom bindings click, they do not authorize

A custom binding with a selector clicks an element that is already on the page. Anything reachable that way must enforce its own server-side authorization, exactly as it would for a mouse click.

A custom binding with a path must use a path in your application, such as `/admin/orders`. Full URLs, protocol-relative URLs such as `//example.com` and schemes such as `javascript:` are rejected when the binding is saved. See [Custom bindings](custom-bindings.md#security).

Presets never accept custom bindings, so one admin cannot push a selector or a path to everyone else.

The keymap is added to the page as JSON with HTML-significant characters escaped, so a selector or path cannot break out of it.

## Taking the owner from the authenticated user

The plugin's own pages read the map owner from `Filament::auth()->user()`. If you call `EditUserMap` or `SelectActiveMap` yourself, do the same. Taking the owner's type or ID from a request parameter would let one admin read and rewrite another admin's map.

## Renaming the user model

A map's owner is stored as the user model's morph class. If you rename the user model, every stored map loses its owner. If you might rename it, register a morph map before admins start customizing their shortcuts:

```php
use App\Models\User;
use Illuminate\Database\Eloquent\Relations\Relation;

Relation::enforceMorphMap([
    'user' => User::class,
]);
```

Registering a morph map later changes the morph class too, so it has the same effect as a rename.

## Authoring is denied by default

Editing the shared presets is a separate permission from personal customization, and the `authorize` option denies everyone until you grant it. A closure must return exactly `true`. Every authoring action is limited to the current panel, so an author on one panel cannot reach another panel's presets. See [Granting authoring access](customization.md#granting-authoring-access).

## Reporting a vulnerability

Please do not open a public issue. Follow the security policy in the [GitHub repository](https://github.com/aristonis/filament-shortcut-keys/security/policy).

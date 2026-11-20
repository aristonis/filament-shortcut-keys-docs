# Security

The plugin's security model is short: a shortcut only ever does what the admin could have done with
the mouse. It clicks the real Filament control or follows the real route. It never invents an action
and never calls a Livewire method directly, so your policies, middleware and confirmation modals apply
exactly as they would to a click.

## The keymap is public

The keymap is built from every registered resource and page, without a `canAccess` check, so every
admin receives the same list of shortcuts whatever their permissions. Pressing a key navigates or
clicks, and your own policies and middleware decide what happens next:

- an unauthorized header or row action is not rendered by Filament, so its key has nothing to click;
- an unauthorized page is stopped by your middleware or Filament's access check, so its key leads to a
  403.

This is deliberate. Filtering the map per user would add no protection, since access is enforced when
the key fires, and it would turn one shared cache entry into one per permission set. The practical
consequence is that the keymap reveals which pages *exist*, not what an admin may do. If the mere
existence of a page is sensitive in your app, do not rely on the shortcut layer to hide it; such a page
does not belong in the panel's navigation.

The plugin's own "Manage shortcuts" page follows the same rule: it can appear in the keymap for an
admin who cannot open it.

## Selector bindings click, they do not authorize

A custom binding with a `selector` payload performs a click on an element already on the page.
Anything reachable that way must enforce its own server-side authorization, exactly as it would for a
mouse click. A `route` payload must be a host-relative path (`/admin/orders`); schemes such as
`javascript:` and protocol-relative `//host` URLs are rejected when the binding is saved. See
[Custom bindings](custom-bindings.md#the-security-boundary).

Shared presets never accept a custom binding, so one admin cannot push a selector or route to everyone
else.

The keymap is injected into the page as a JSON block with HTML-significant characters escaped, so a
selector or route value cannot break out of it.

## Owner identity comes from the authenticated user

The plugin's own pages derive the map owner from `Filament::auth()->user()`. If you call the
application use cases yourself (`EditUserMap`, `SelectActiveMap`), keep doing the same. Passing an
owner type or id from a request parameter would let one admin read and rewrite another admin's map.

Map ownership is stored as the user model's morph class and compared as a string, so renaming the
user model orphans every stored map. If you might rename it, register a morph map before admins start
customizing (registering one later changes the morph class too):

```php
Relation::enforceMorphMap([
    'user' => App\Models\User::class,
]);
```

## Authoring is denied by default

Editing the shared presets is a separate permission from personal customization, and `authorize`
denies everyone until you grant it. A closure must return exactly `true`; anything else denies. Every
authoring lookup is scoped to the current panel, so a manager on one panel cannot reach another
panel's maps. See [the authoring gate](customization.md#the-authoring-gate).

## Reporting a vulnerability

Please do not open a public issue. Follow the security policy in the
[GitHub repository](https://github.com/aristonis/filament-shortcut-keys/security/policy).

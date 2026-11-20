# Limitations

Known limitations of the current release. Most are deliberate trade-offs.

## Scope

- **Filament 5 only.** Filament 4 is not supported.
- **English only.** Every string is translatable and publishing the language files gets you a fully
  translated panel, but only English ships. See [Translations](translations.md).
- **Right to left is unproven.** Pagination keys are mirrored for a right-to-left panel and that
  mirroring is tested, but the experience as a whole has not been verified in a browser.

## What gets a shortcut

- **A page cannot declare its own shortcuts.** Shortcuts come from navigation, header actions and
  tables.
- **Navigation follows registration, not the sidebar.** Every registered resource and page gets a key,
  including a page hidden from the sidebar with `$shouldRegisterNavigation = false`. To drop one,
  disable it in the overlay:
  `'navigation:App\Filament\Pages\Hidden' => ['disabled' => true]`.
- **Items added with `$panel->navigationItems([...])` get no key**, whether they point inside the app
  or elsewhere. An admin can bind a route to one as a custom binding.
- **Header actions inside an action group dropdown get no key.** Row actions inside one are bound,
  because Filament flattens record-action groups, but whether the key reaches a control in a collapsed
  dropdown has not been verified.
- **An action whose only modal is a form schema counts as instant** and gets no default bare key,
  because that cannot be known without mounting it. Give it a modal heading or pin it.
- **Table keys need a table the page can see.** A page's own table, relation managers, and header,
  footer or dashboard table widgets are detected. A table placed through a page's own `content()`
  schema or `Livewire::make()`, or a widget placed through `getWidgetsSchemaComponents()`, is not, so
  that page gets no table keys unless it is a table page itself.
- **A pool holds at most 26 letters**, 23 for bare keys by default (after `G`, `J` and `K`). Shortcuts beyond that
  get no key.

## Letters

- **Automatic letters follow the navigation sort.** Adding a screen can move the letter of a later
  item that lost a clash. Pin the ones people rely on in the config overlay; a pinned letter never
  moves.
- **Row-action letters follow the order of `->rowActions([...])`.** Reordering the list rekeys them.

## Keys and sequences

- **One fixed leader per set is the only sequence.** There are no user-defined sequences or macros,
  and no command palette.
- **Keys are chosen per deployment, not per admin.** The `modifiers` scheme is configured once for
  everyone; each admin can only switch single-key shortcuts off for themselves.
- **Header and row actions cannot share navigation's keys.** The two groups are lettered separately,
  so a configuration that puts them on the same scheme is refused rather than allowed to hand one key
  to two shortcuts.
- **`G` sequences carry no `aria-keyshortcuts`.** The attribute cannot express a sequence; the
  cheatsheet lists them.

## Customization

- **Admins customize navigation only.** The manage page remaps and disables navigation shortcuts and
  adds custom bindings. It does not offer header-action, table or row-action keys; a developer can
  still move those with the config overlay.
- **Custom bindings cannot be removed from the manage page.** Adding one with the same name replaces
  it; removing one is done from code (see [Custom bindings](custom-bindings.md#driving-it-from-code)).
- **A personal map does not follow its preset.** Once an admin's first edit copies a preset, later
  edits to that preset do not reach them.

## Security and storage

- **The keymap is public.** It is built from every registered page without a permission check, so it
  reveals which pages exist, not what an admin may do. Access is enforced when the key fires. See
  [Security](security.md).
- **Map ownership is stored as the user model's morph class.** Register a morph map
  (`Relation::enforceMorphMap([...])`) if you might rename your user model.
- **Stored overrides outlive deleted targets.** Clean them up with
  [`shortcut-keys:prune`](maintenance.md), which only removes what it can prove is gone.

## Internal API

Anything marked `@internal`, including the resolution ports in `Core\Contracts` (`ShortcutSet`,
`PageContextProvider`, `NavigationProvider`, `ReservesKeys`, `AcceptsCustomBindings`, `Resolver`), is
not public API and may change in a minor release. See [Architecture](architecture.md).

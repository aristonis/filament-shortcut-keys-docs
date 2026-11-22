# Shortcuts

## Introduction

The plugin builds its shortcuts from what the panel already declares. Every shortcut belongs to a set, and each set decides what it binds and how its keys are pressed:

| Set | Default keys | What it binds |
| --- | --- | --- |
| Navigation | `G`, then a letter | Every registered resource and page |
| Actions | A letter | The current page's header actions |
| Table | Fixed keys | The tables on the page. See [Table shortcuts](table-shortcuts.md). |
| Row actions | A letter | Record actions you register. See [Row actions](row-actions.md). |
| Custom bindings | `G`, then a letter | Keys an admin adds. See [Custom bindings](custom-bindings.md). |

No default shortcut asks you to hold `Ctrl`, `Alt` or `Cmd`. Browsers, operating systems and keyboard layouts already use many of those combinations (`Alt`+`D` focuses the address bar, `Alt`+`Shift` switches the layout on Windows, `Option` types characters on a Mac), and a web page cannot tell which ones are free. Navigation therefore uses a leader key, and the page's own shortcuts use single keys. You may change this per set with the [`modifiers` option](configuration.md#changing-how-keys-are-pressed).

## Navigating with the leader key

To open a resource or page, press `G`, release it, then press the page's letter within a second and a half. Pressing `Esc` cancels a pending `G`, and so does any key that is not bound.

While `G` waits for its letter, each sidebar link with a shortcut shows its letter in a small badge. The cheatsheet writes these shortcuts in lower case, `g` then `o`, because they are two plain presses and not `Shift`+`G`.

`G` typed into a form field stays a `g`, and `Alt`+`G` is left to the browser.

### Which pages get a navigation shortcut

Every resource and page in the sidebar gets a navigation shortcut, including the plugin's own **Keyboard shortcuts** and **Manage shortcuts** pages.

A resource or page hidden from the sidebar with `$shouldRegisterNavigation = false` gets none. Such pages are often reached only from a link that carries a record or a parameter, so a key to their bare URL would open an error page. To give them keys, [include hidden pages](configuration.md#including-pages-hidden-from-the-sidebar).

Pages and resources inside a cluster get a key, lettered after every sidebar item. A nested resource, a resource without an `index` page and a page whose route needs a parameter get none, because they have no URL a key could open.

Items you add with `$panel->navigationItems()` do not get one. To remove a page's shortcut, [disable it in the overlay](configuration.md#pinning-and-disabling-shortcuts):

```php
'overlay' => [
    'navigation:App\Filament\Pages\Settings' => ['disabled' => true],
],
```

Every admin receives the same navigation shortcuts, whatever their permissions. A shortcut to a page the admin cannot open leads to the same 403 response as the URL would. See [Security](security.md#the-keymap-is-the-same-for-every-admin).

## Running header actions

On every page, the header actions get single-letter shortcuts. Pressing the letter clicks the action, or follows its URL if it has one.

### Which header actions get a shortcut

Because a single key is easy to press by accident, a header action only gets a shortcut by default when a stray press is harmless. That is the case when the action:

- has a URL, so pressing it opens a page. This includes the Create, Edit and View actions of a resource that registers those pages.
- opens a modal, because it has a modal heading, a modal description, modal content, or calls `modal()`.
- requires confirmation through `requiresConfirmation()`.

An action that does its work the moment it is clicked gets no shortcut, so one stray keystroke can never run it. To give it one anyway, [pin a letter for it in the overlay](configuration.md#pinning-and-disabling-shortcuts).

!!! note
    The plugin cannot tell that an action with only a form schema will open a modal without mounting the action. Such an action counts as running immediately. Give it a modal heading, or pin its letter:

    ```php
    use Filament\Actions\Action;

    Action::make('archive')
        ->schema([
            // ...
        ])
        ->modalHeading('Archive order')
    ```

A header action never gets a shortcut, and never takes a letter, when it is:

- hidden, through `visible(false)`, `hidden()`, or a visibility closure that fails for the current admin,
- disabled,
- inside an action group.

If you move header actions to a held modifier or a leader key of their own with the [`modifiers` option](configuration.md#changing-how-keys-are-pressed), every visible, enabled header action gets a shortcut, including the ones that run immediately.

## How letters are assigned

Each shortcut takes the first free letter of its name, then the first free letter of the alphabet. The name is:

- the navigation label, for navigation shortcuts,
- the action name, for header actions and row actions,
- the binding name, for custom bindings.

For example, with an **Orders** resource sorted before an **Organizations** resource, Orders gets `O` and Organizations gets `R`, because `o` is already taken.

Only the letters `a` to `z` count. Letters with accents and letters from other scripts are skipped, so a label written entirely in another script takes the first free letter of the alphabet. See [Localization](localization.md#letters-in-other-languages).

### Letter pools

Shortcuts that are pressed the same way share one pool of letters, and two shortcuts in the same pool never get the same key. By default:

- navigation shortcuts and custom bindings share the `G` pool,
- header actions and row actions share the pool of single keys.

A few keys are reserved in the single-key pool, so no header action or row action takes them: `G` (the navigation leader), and `J` and `K` (the row cursor), along with the other [table keys](table-shortcuts.md#the-table-keys).

A pool holds at most 26 letters, and the default single-key pool holds 23. A shortcut beyond that gets no key.

### Assignment order

Letters are handed out in this order:

- **Navigation** follows the sidebar: items outside any group first, then each navigation group in the order the panel registers it, sorted by navigation sort inside the group. Cluster pages come after every sidebar item, and [hidden pages you include](configuration.md#including-pages-hidden-from-the-sidebar) after those.
- **Row actions** are lettered first, across the whole panel, in the order you register them. No header action is ever given a row action's letter automatically, on any page.
- **Header actions** follow the order in which the page declares them.

Because navigation follows the sidebar, adding a resource can move the letter of a later one that lost a clash.

!!! tip
    Pin the letters people rely on with the [overlay](configuration.md#pinning-and-disabling-shortcuts). A pinned letter does not move when you add or remove other shortcuts.

## When shortcuts are ignored

Single-key shortcuts and `G` sequences do not fire:

- while a text input, a textarea, a select or an editable element has focus. Checkboxes, radio buttons, buttons and other inputs that take no text do not count.
- while focus is inside a modal, a dropdown, a menu, a list box or a tab list, or while any Filament modal or the cheatsheet is open.
- when `Enter` or `Space` is pressed on a focused button or link, which presses that control instead.
- while text is being composed through an input method (IME).
- right after a shortcut clicked a control that sends a Livewire request, until the response arrives or at most about two seconds have passed. A key pressed in that window is dropped, not queued, so it cannot land on the row behind a modal that is about to open.
- when the user has [turned off single-key shortcuts](accessibility.md#turning-off-single-key-shortcuts).

Shortcuts on a held modifier, if you configure any, are only ignored during input-method composition. They fire in all the other cases.

## Holding a key down

A held key fires its shortcut once. The exceptions are the keys that move the row cursor, `J`, `K`, `↑` and `↓`, which repeat while held, like scrolling. `←` and `→` change one page per press.

## Physical keys

Shortcuts are matched on the physical key, not on the character it types. A shortcut therefore works the same whichever keyboard layout is active, including a layout that types another script. On layouts other than US QWERTY, some keys carry a different letter than the one shown. See [Localization](localization.md#keyboard-layouts).

## The cheatsheet

Press `?` (`Shift`+`/`) on any page to open the cheatsheet. It lists every shortcut active on the current page, grouped by set, with each shortcut's label. Press `?` again, press `Esc`, or click outside it to close it. The cheatsheet does not open while you are typing in a field.

The cheatsheet also holds the [single-key shortcuts switch](accessibility.md#turning-off-single-key-shortcuts) and a link to the **Keyboard shortcuts** page.

## The Keyboard shortcuts page

The plugin adds a **Keyboard shortcuts** page to the panel's navigation. It lists the shortcuts that are the same on every page for the current admin: navigation, their custom bindings, the table keys and the row action letters. Header actions change from page to page, so they only appear in the cheatsheet.

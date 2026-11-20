# The shortcuts

Every shortcut belongs to one of five sets. Each set decides what it binds and how its keys are pressed.

## The key table

| Set | Keys | What it covers |
| --- | --- | --- |
| Navigation | `G`, then a letter | Every registered resource and page |
| Actions | bare letter | The current page's header actions |
| Table | bare keys | The page's own table, its relation managers and its table widgets (see below) |
| Row actions | bare letters | Record actions you register by name (see [Row actions](row-actions.md)) |
| Custom | `G`, then a letter | Bindings an admin adds for themselves (see [Custom bindings](custom-bindings.md)) |

The table keys are fixed:

| Key | What it does |
| --- | --- |
| `/` | Focus the table's search field |
| `J` / `K` | Start the row cursor, then move it down / up |
| `↓` / `↑` | Move the cursor, while it is live |
| `←` / `→` | Previous / next page, while the cursor is live (swapped in a right-to-left panel) |
| `Space` | Select the cursor's row (its checkbox) |
| `Enter` | Edit the cursor's row |
| `Delete` | Delete the cursor's row, through Filament's own confirmation |
| `Esc` | Release the cursor |

`J` starts the cursor on the first row and `K` on the last. `Space`, `Enter`, `Delete` and row actions
act on the row the cursor is on, so start it first. `Enter` needs an edit action on the row, `Delete` a
delete action, and `/` a searchable table; without them the key has nothing to press.

Keys are matched on the physical key (`event.code`), not the character it produces, so a shortcut sits
on the same key on any keyboard layout.

## Why nothing is held by default

Browsers, operating systems and keyboard layouts already own many `Alt` and `Ctrl` combinations
(`Alt`+`D` focuses the address bar, `Alt`+`Shift` switches the layout on Windows, `Option` types
characters on a Mac), and a page cannot tell which are free. So no default shortcut holds a modifier.
Navigation and custom bindings use a leader key instead, and the page's own shortcuts use bare keys.
If you prefer held modifiers, set them per set in the [`modifiers` config](configuration.md#modifiers).

## The `G` leader

Press `G`, let go, then press the letter within a second and a half. `Esc` cancels a pending `G`.
Pressing `Shift`, `Ctrl`, `Alt` or `Meta` on its own while `G` waits does not cancel it. `G` typed into
a field stays a `G`, and `Alt`+`G` belongs to the browser.

The cheatsheet writes a sequence in lower case (`g` then `o`), because it is two plain presses, not
`Shift`+`G`.

## Which header actions get a key

On bare letters, a header action only gets a key by default if a stray press is harmless:

- it opens a page (Create, Edit and View on a resource that registers those pages), or
- it opens a modal, or
- it asks for confirmation first.

An action that does its work on the click itself gets no default key, so one stray keystroke can never
run it. To key one anyway, pin a letter for it in the [`overlay` config](configuration.md#overlay).
Filament cannot tell without mounting an action whether a plain form schema will open a modal, so such
an action is treated as instant too; give it a modal heading or pin it.

On a held modifier or a leader, every header action keeps its key, since those are never pressed by
accident.

A header action that is hidden (`->visible(false)`, or a `visible` closure that fails for this admin)
or disabled gets no key at all, so it never takes a letter a live action could have had. Header
actions inside an action group dropdown get no key.

## How letters are picked

Each shortcut takes the first free letter of its name, then falls back through the rest of the
alphabet. The name is the navigation label for navigation, and the action or binding name otherwise.

Shortcuts that are pressed the same way share one pool of letters and never share a key:

- navigation and custom bindings share the `G` pool;
- header actions and row actions share the bare pool.

A few keys are reserved in the bare pool: `G` (the leader is pressed bare), `J` and `K` (the row
cursor), and the other table keys. No header action or row action takes them. Row actions are lettered
first, across the whole panel, and no header action is ever lettered automatically onto a row action's
letter, so header letters do not change with the tables a page has.

Navigation letters are handed out in order of each resource's and page's navigation sort. Adding a
screen can therefore move the letter of a later one that lost a clash. Pin the letters people rely on
in the [`overlay` config](configuration.md#overlay); a pinned letter never moves.

A pool has at most 26 letters (23 in the default bare pool, after `G`, `J` and `K`). Shortcuts
beyond that get no key.

## When keys are ignored

Bare keys and `G` sequences do not fire:

- while an input, textarea, select or contenteditable element has focus (checkboxes, radios and buttons
  do not count as typing);
- while a modal, an open select, a dropdown, a menu or a tab list has the keyboard, or while any
  Filament modal or the cheatsheet is open;
- for `Enter` and `Space` on a focused button or link, which press that control instead;
- for keys typed through an input method (IME);
- right after a shortcut clicked a control that sends a Livewire request, until the response arrives
  (at most about two seconds), so a quick second key cannot land on the row behind a modal that is
  about to open. A key pressed in that window is dropped, not queued;
- when the user has switched single-key shortcuts off (see below).

Shortcuts on held modifiers, if you configure any, are only subject to the input-method rule; they
still fire in all the other cases.

## Held keys

A held key fires an action once. Moving the row cursor (`J`, `K`, `↑`, `↓`) repeats while the key is
held, like scrolling. `←` and `→` turn one page per press.

## The arrows and the cursor

The arrow keys scroll the page until `J` or `K` starts the row cursor, and again once you click
somewhere else or press `Esc`, so a long page or a wide table still scrolls from the keyboard. The
cursor holds real focus and follows the record, not the position, so a sort, a search or a refresh
never points a row key at a different record.

## Pages with several tables

A record page with relation managers, or a page or dashboard with table widgets in its header, footer
or widget area, gets the table keys too. `J` and `K` walk the rows of every table in page order, and
`/` and the paging arrows act on the table the cursor is in. With no cursor, `/` focuses the first
table search field on the page.

A table placed any other way (through the page's own `content()` schema, `Livewire::make()`, or a
widget placed through `getWidgetsSchemaComponents()`) is not detected, so that page gets no table keys
unless it is a table page itself.

## The cheatsheet

`?` (`Shift` + `/`) opens a cheatsheet of everything active on the current page; `?` again or `Esc`
closes it. It is not opened while you are typing in a field.

The cheatsheet has a **Single-key shortcuts** switch. Turning it off silences every bare key and every
`G` sequence in that browser, which assistive technology and voice control users may need. The choice
is stored in the browser's local storage under `filament-shortcut-keys:single-keys`. Shortcuts on held
modifiers, if configured, keep working.

## The Keyboard shortcuts page

The plugin adds a "Keyboard shortcuts" page to the sidebar. It lists what is the same everywhere in the
panel for the current admin: navigation, their custom bindings, the table keys and the row-action
letters. Header actions change from page to page, so they are only in the cheatsheet. The cheatsheet
links to this page.

## Key badges

Where a shortcut maps to one stable element, the plugin paints a small key badge on it: navigation
links, header actions and an admin's custom bindings. Badges for `G` shortcuts appear only while `G`
waits for its letter, so the sidebar is not covered in letters the rest of the time. With single-key
shortcuts off, badges for bare and `G` keys are hidden.

Table keys and row actions get no badge. A table key has no single element behind it, and a row action
repeats on every row. Both are in the cheatsheet.

The badge is marked `aria-hidden`, so a screen reader reads only the control's own label. The key is
announced through `aria-keyshortcuts` on the control instead. A `G` sequence gets no
`aria-keyshortcuts`, because that attribute reads spaces as alternatives (`G O` would mean "G or O");
the cheatsheet lists those. A value your app already put on the control is left alone.

## Styling hooks

The plugin ships no stylesheet; its marks are styled inline.

- `fi-hotkey-hint`: the class on the badge `span`. Restyle it from your panel's stylesheet.
- `fi-hotkey-row-focused`: the class on the row the cursor is on, which also carries an inline
  outline. An `outline` rule on this class needs `!important` to beat the inline one.

```css
.fi-hotkey-row-focused {
    outline: 2px solid var(--primary-500) !important;
}
```

## Which shortcut wins

When one key could match more than one active set, the browser side checks them in a fixed order:

```
custom > table > global > navigation
```

Row actions are fired by the table handler, so they sit at the table tier. With the shipped sets two
groups cannot reach the same key combination, because each pool is lettered once and the table's keys
are reserved, so this order rarely matters.

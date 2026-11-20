# Table shortcuts

## Introduction

Every page that shows a table gets the same fixed table keys, with no configuration. This includes list pages, relation managers and table widgets. The keys act on a row cursor, which marks one row at a time.

## The table keys

| Key | What it does |
| --- | --- |
| `/` | Focus the table's search field |
| `J` | Start the row cursor on the first row, then move it down |
| `K` | Start the row cursor on the last row, then move it up |
| `↓` / `↑` | Move the cursor down / up, while the cursor is active |
| `←` / `→` | Go to the previous / next page, while the cursor is active |
| `Space` | Toggle the selection checkbox of the cursor's row |
| `Enter` | Edit the cursor's row |
| `Delete` | Delete the cursor's row, through Filament's own confirmation |
| `Esc` | Release the cursor |

`Space`, `Enter`, `Delete` and [row actions](row-actions.md) act on the row under the cursor, so start the cursor with `J` or `K` first.

Each key needs something on the table to press:

- `/` needs a searchable table.
- `Space` needs a selection checkbox on the row, which Filament renders when the table has bulk actions.
- `Enter` needs a record action named `edit`, such as `EditAction`.
- `Delete` needs a record action named `delete`, such as `DeleteAction`.

Without it, the key does nothing. The keys press the real controls, so a row the admin may not edit has no edit control and `Enter` does nothing on it.

## Using the row cursor

`J` starts the cursor on the first row and `K` on the last. The cursor row gets an outline and keyboard focus. It stays out of the `Tab` order.

The cursor follows the record, not the position. When a sort, a search or a refresh re-renders the table, the cursor stays on the same record, so a row key never acts on a different record than the one you marked. If the record leaves the table, the cursor goes with it.

The cursor stays active while focus is on its row. Clicking anywhere outside the table rows releases it, and so does `Esc`.

### Arrow keys

Until `J` or `K` starts the cursor, the arrow keys scroll the page as usual, so a long page or a wide table still scrolls from the keyboard. Once the cursor is active, `↑` and `↓` move it and `←` and `→` change the page. After you release the cursor, the arrows scroll the page again.

## Paging in a right-to-left panel

In a right-to-left panel, Filament flips its previous and next page buttons. The plugin swaps the paging keys to match: `←` goes to the next page and `→` to the previous one. `↑` and `↓` are not swapped. See [Localization](localization.md#right-to-left-panels).

## Pages with several tables

A page can show more than one table. The plugin detects:

- the page's own table,
- relation managers on a record page,
- table widgets in a page's header or footer,
- table widgets on a dashboard.

On such a page, `J` and `K` walk the rows of every table in page order. `/` and the paging keys act on the table the cursor is in. Without a cursor, `/` focuses the first table search field on the page.

A table placed any other way is not detected: through the page's `content()` schema, through `Livewire::make()`, or a widget placed through `getWidgetsSchemaComponents()`. Such a page gets no table keys unless it is a table page itself.

## Turning off a table key

To turn off a table key in every table of the panel, disable it in the [overlay](configuration.md#pinning-and-disabling-shortcuts):

```php
'overlay' => [
    'table:delete' => ['disabled' => true],
],
```

The table keys are named:

| Name | Key |
| --- | --- |
| `search` | `/` |
| `cursor-down` | `J` |
| `cursor-up` | `K` |
| `row-down` | `↓` |
| `row-up` | `↑` |
| `page-prev` | `←` (`→` in a right-to-left panel) |
| `page-next` | `→` (`←` in a right-to-left panel) |
| `select` | `Space` |
| `edit` | `Enter` |
| `delete` | `Delete` |
| `cursor-release` | `Esc` |

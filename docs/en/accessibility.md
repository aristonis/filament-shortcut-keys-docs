# Accessibility

## Introduction

Shortcuts are announced to assistive technology, single-key shortcuts can be turned off by each user, and the row cursor and the cheatsheet use real keyboard focus. This page describes that behavior and the hooks you may use to restyle the plugin's marks.

## Key badges

Where a shortcut presses one specific element, the plugin shows a small badge with the key on that element. Badges appear on:

- navigation links,
- header actions,
- the elements and links that admins' custom bindings target.

Badges for `G` shortcuts appear only while `G` waits for its letter, so the sidebar is not covered in letters the rest of the time. Table keys and row actions get no badge, because a table key has no single element and a row action repeats on every row. Both are listed in the cheatsheet.

## Screen readers

Each badge is marked `aria-hidden`, so a screen reader reads only the control's own label. The key is announced through the `aria-keyshortcuts` attribute on the control instead, for example `aria-keyshortcuts="E"`, or `"Alt+E"` when header actions use a held modifier.

`G` sequences get no `aria-keyshortcuts`. The attribute treats a space as a separator between alternatives, so `G O` would mean "G or O". The cheatsheet lists these sequences instead.

If your application already set `aria-keyshortcuts` on a control, the plugin leaves it unchanged.

## Turning off single-key shortcuts

The cheatsheet has a **Single-key shortcuts** switch. Turning it off stops every shortcut that needs no held modifier in that browser, including the `G` sequences and the table keys, and hides their badges. Users of speech input and other assistive technology may need this, because a word they speak or a key they press can otherwise trigger a shortcut. It provides the "turn off" mechanism that WCAG success criterion 2.1.4, Character Key Shortcuts, describes.

The choice is saved in the browser's local storage under the key `filament-shortcut-keys:single-keys`. When local storage is not available, such as in some private windows, shortcuts stay on. Shortcuts on a held modifier, if you [configure any](configuration.md#changing-how-keys-are-pressed), keep working when the switch is off.

## The cheatsheet

The cheatsheet opens as a modal dialog. Opening it moves focus into it, `Tab` stays within it, and closing it returns focus to where it was. On the **Keyboard shortcuts** page, each table has column headers that screen readers announce but that are not shown.

## The row cursor

The [row cursor](table-shortcuts.md#using-the-row-cursor) gives its row real keyboard focus, so assistive technology follows it. The row is never added to the `Tab` order.

## Styling the badges and the row cursor

The plugin ships no stylesheet. Its badges and the row cursor are styled inline, and each carries a class you may target from your panel's theme:

- `fi-hotkey-hint` is the class of the badge `<span>`.
- `fi-hotkey-row-focused` is the class of the row the cursor is on.

Because the inline styles win over a class rule, a rule that changes a property the plugin sets inline needs `!important`. On the badge, those properties are `margin-inline-start`, `padding`, `border-radius`, `border`, `font-size`, `line-height` and `opacity`. On the row, they are `outline` and `outline-offset`.

```css
.fi-hotkey-hint {
    opacity: 1 !important;
}

.fi-hotkey-row-focused {
    outline: 2px solid var(--primary-500) !important;
}
```

To change the markup of the cheatsheet or the plugin's pages, you may publish the views:

```bash
php artisan vendor:publish --tag="filament-shortcut-keys-views"
```

Published views no longer receive changes from package updates.

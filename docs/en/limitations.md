# Limitations

## Introduction

This page lists the known limitations of the current release. Most of them are deliberate trade-offs.

## Scope

- **Filament 5 only.** Filament 4 is not supported.
- **English only.** Every string can be translated, but only English translations ship. See [Localization](localization.md).
- **Right-to-left support is limited to the paging keys.** The paging keys are mirrored in a right-to-left panel, but the right-to-left experience as a whole has not been verified in a browser.

## What gets a shortcut

- **A page cannot declare shortcuts of its own.** Shortcuts come from navigation, header actions and tables, plus the row actions and custom bindings described in this documentation.
- **Navigation follows the default sidebar.** Every resource and page in the sidebar gets a key; pages hidden from it get none unless you [include them](configuration.md#including-pages-hidden-from-the-sidebar). A custom navigation built with `$panel->navigation()` is lettered by navigation group and sort, which may not match what it shows. To remove a key, [disable it in the overlay](configuration.md#pinning-and-disabling-shortcuts).
- **Items added with `$panel->navigationItems()` get no key.** An admin can add a [custom binding](custom-bindings.md) to one instead.
- **Header actions inside an action group get no key.**
- **Row actions inside an action group** are bound and listed, but whether the key reaches the control while the group's dropdown is closed has not been verified.
- **An action whose only modal content is a form counts as running immediately** and gets no default single-key shortcut. Give it a modal heading, or pin its letter.
- **Table keys need a table the page can see.** Tables placed through a page's `content()` schema, `Livewire::make()` or `getWidgetsSchemaComponents()` are not detected. See [Pages with several tables](table-shortcuts.md#pages-with-several-tables).
- **A pool holds at most 26 letters**, and the default single-key pool holds 23. Shortcuts beyond that get no key.

## Letters

- **Automatic letters follow the sidebar order.** Adding a page can move the letter of a later one. Pin the letters people rely on.
- **Letters can differ between admins** when your `shouldRegisterNavigation()` depends on the signed-in admin, because each admin is lettered by their own sidebar. Pin the letters admins share.
- **Row action letters follow the order of the `rowActions()` list.** Reordering the list changes them.
- **Only the letters `a` to `z` are taken from names.** Labels in other scripts take letters in alphabet order, and navigation letters can differ between languages.

## Keys and sequences

- **One leader key per set is the only kind of sequence.** There are no user-defined sequences and no command palette.
- **Keys are configured once for everyone.** The `modifiers` option has no per-admin setting. Each admin can only turn single-key shortcuts off for themselves.
- **Header actions and row actions cannot share navigation's keys.** A configuration that puts them on the same setting is refused rather than allowed to give one key to two shortcuts.
- **`G` sequences have no `aria-keyshortcuts`.** The attribute cannot express a sequence, so the cheatsheet lists them instead.

## Customization

- **Admins customize navigation only.** The **Manage shortcuts** page remaps and disables navigation shortcuts and adds custom bindings. Header action, table and row action keys can only be changed through the overlay.
- **Custom bindings cannot be removed from the Manage shortcuts page.** Adding one with the same name replaces it. To remove one, use [`EditUserMap::enable()`](custom-bindings.md#managing-bindings-from-code).
- **A personal map does not follow its preset.** Once an admin's first change copies a preset, later changes to that preset do not reach them.
- **The Edit preset form only sets letters.** Disabling a shortcut in a preset is done from code, with `AuthorSystemMap::editInPlace()`.

## Security and storage

- **The keymap is the same for every admin.** It reveals which pages exist, not what an admin may do. See [Security](security.md).
- **Map ownership is stored as the user model's morph class.** Register a morph map if you might rename your user model. See [Renaming the user model](security.md#renaming-the-user-model).
- **Stored changes outlive deleted shortcuts.** Remove them with the [`shortcut-keys:prune` command](commands.md), which only deletes what it can prove is gone.

## Public API

The documented ways to work with the plugin are the plugin's `make()` and `rowActions()` methods, the configuration file, the Artisan command, and the `EditUserMap`, `SelectActiveMap` and `AuthorSystemMap` classes. Classes and interfaces marked `@internal` are not public API and may change in a minor release.

# Filament Shortcut Keys

![Filament Shortcut Keys](https://raw.githubusercontent.com/aristonis/filament-shortcut-keys/main/art/cover.jpg)

## Introduction

Filament Shortcut Keys is a panel plugin that adds keyboard shortcuts to a Filament 4 or 5 panel without any per-resource setup. When a page renders, the plugin reads the panel's registered resources and pages, the page's header actions and its tables, gives each one a key, and sends the resulting keymap to the browser. When you add a resource, it gets a shortcut.

Every shortcut presses the real Filament control or follows the real URL. Your policies, middleware and confirmation modals apply exactly as they would to a mouse click.

## Shortcuts at a glance

Once the plugin is registered, every page in the panel has these keys:

| Keys | What they do |
| --- | --- |
| `G`, then a letter | Open a resource or page |
| A letter | Run one of the current page's header actions |
| `/` | Focus the table's search field |
| `J` / `K` | Start the row cursor and move it down / up |
| `Space`, `Enter`, `Delete` | Select, edit or delete the row under the cursor |
| `?` | Open the cheatsheet for the current page |

You may also give your own table record actions a key, such as `approve` or `reject`. See [Row actions](row-actions.md).

## Customizing shortcuts

Each admin can remap or disable their own navigation keys and add bindings of their own from the **Manage shortcuts** page. You may instead lock the panel to one shared map, or keep every key in your configuration file. Editing the shared presets that everyone starts from is a separate permission, denied until you grant it. See [Letting admins customize](customization.md).

## Reading key names

Key names in this documentation are key caps. `G` means the G key pressed on its own, without `Shift`. A sequence such as "`G`, then `O`" means you press and release `G`, then press `O`.

## Next steps

To add the plugin to a panel, follow the [installation guide](installation.md).

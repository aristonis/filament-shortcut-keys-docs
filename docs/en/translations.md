# Translations

The package ships English only. Every string it renders (the cheatsheet, the two pages, the manage
page's forms and notifications, the table behaviour labels) goes through Laravel's translator, so
publishing the files and adding your own locale gives you a fully translated panel.

## Publish the files

```bash
php artisan vendor:publish --tag="filament-shortcut-keys-translations"
```

This copies the English file to `lang/vendor/filament-shortcut-keys/en/shortcut-keys.php`. To add a
locale, copy that folder to the locale's code and translate the values:

```
lang/vendor/filament-shortcut-keys/
├── en/shortcut-keys.php
└── fr/shortcut-keys.php
```

To change the English wording, edit the published `en` file. Only override the keys you change; a key
missing from your file falls back to the package's own.

Navigation labels come from your resources and pages, and header-action labels from your actions, so
they follow your app's own translations. A row action on a relation manager or table widget is
labelled from its name in the cheatsheet (see [Row actions](row-actions.md#nested-tables)), which is
not translated.

The keymap is cached per locale, so each language gets its own entry.

## Right-to-left panels

Keys are matched on the physical key, so a shortcut sits on the same key in any text direction. The
one exception is pagination. In a right-to-left panel Filament flips its own previous and next
buttons, so the plugin swaps the arrows to match: `←` moves to the next page and `→` to the previous
one. The cheatsheet and the "Keyboard shortcuts" page show the key that actually fires.

The panel's direction comes from Filament's own translations (`filament-panels::layout.direction`),
not this package's, so the mirroring applies whatever language you run. Row-movement arrows are not
mirrored: `↑` is up in either direction.

The mirroring is covered by tests, but the right-to-left experience as a whole has not been verified in
a browser. Treat it as working but unproven, and report anything that feels backwards on
[GitHub](https://github.com/aristonis/filament-shortcut-keys/issues).

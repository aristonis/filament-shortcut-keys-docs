# Localization

## Introduction

Every string the plugin renders goes through Laravel's translator: the cheatsheet, the **Keyboard shortcuts** and **Manage shortcuts** pages, the manage page's forms and notifications, and the table key labels. The package ships English translations only.

## Publishing the translations

To change the wording or add a language, publish the translation files:

```bash
php artisan vendor:publish --tag="filament-shortcut-keys-translations"
```

This copies the English file to `lang/vendor/filament-shortcut-keys/en/shortcut-keys.php`.

To change the English wording, edit the published `en` file. You only need to keep the keys you change. A key missing from your file falls back to the package's own.

## Adding a language

To add a language, copy the `en` folder to a folder named after the locale and translate the values:

```text
lang/vendor/filament-shortcut-keys/
├── en/shortcut-keys.php
└── fr/shortcut-keys.php
```

Navigation labels come from your resources and pages, and header action labels from your actions, so they follow your application's own translations. Two things are not translated:

- A row action on a relation manager or a table widget is labelled in the cheatsheet from its name, such as "Mark paid" for `mark_paid`. See [Row actions](row-actions.md#row-actions-on-relation-managers-and-widgets).
- When the **Manage shortcuts** page refuses a change, for example a custom binding whose path does not start with `/`, the notification shows an English message.

## Letters in other languages

Navigation letters come from the navigation label, which is translated. Only the letters `a` to `z` count, so:

- a label written in another script, such as Arabic, has no usable letters and takes the first free letter of the alphabet, in navigation sort order,
- the same page can get a different letter in each language your panel runs in.

Each locale is cached separately, so these letters are correct in every language. If your admins switch between languages and need the same key everywhere, pin those letters in the [overlay](configuration.md#pinning-and-disabling-shortcuts).

Header actions and row actions take their letters from the action name, which is not translated, so their letters are the same in every language.

## Keyboard layouts

Shortcuts are matched on the physical key, in the positions of a US QWERTY keyboard, not on the character the key types. This has two effects:

- Shortcuts keep working when the keyboard types another script. An admin using an Arabic or Russian layout does not need to switch layouts to use them.
- On a layout that places letters differently, the key you press may carry another letter. On a French AZERTY keyboard, the shortcut shown as `A` is the key labelled `Q`, and `/` is on the key labelled `!`.

## Right-to-left panels

In a right-to-left panel, Filament flips its previous and next page buttons. The plugin swaps the paging keys to match: `←` goes to the next page and `→` to the previous one. The cheatsheet and the **Keyboard shortcuts** page show the key that actually works.

The plugin reads the panel's direction from Filament's own translations (`filament-panels::layout.direction`), so the paging keys are mirrored for any right-to-left locale, even without a translation of this package. The keys that move the row cursor are not mirrored: `↑` moves up in both directions.

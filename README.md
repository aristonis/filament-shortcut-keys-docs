# filament-shortcut-keys-docs

Documentation for the [aristonis/filament-shortcut-keys](https://github.com/aristonis/filament-shortcut-keys) package.

The pages live under `docs/en` (English) and `docs/ar` (Arabic). They are built with MkDocs and the
Material theme. This repo is only the source. The published site pulls from it, so nothing here builds
to GitHub Pages.

## Branches

Each major version of the package has its own branch. `main` follows the latest line, and `1.x` holds
the 1.x releases. A fix for an older version goes on that version's branch.

## Local preview

```bash
pip install -r requirements.txt
mkdocs serve   # http://127.0.0.1:8000
mkdocs build   # writes the static site to ./site
```

## Translation

The Arabic pages under `docs/ar` mirror the English pages one to one. A change to an English page
needs the same change in its Arabic copy.

## License

MIT. See [LICENSE](LICENSE).

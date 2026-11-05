# docs

Tossling documentation: install the apps, run your own server.

The site is at https://tossling.github.io/docs/. It is built with [MkDocs Material](https://squidfunk.github.io/mkdocs-material/)
and deployed to GitHub Pages from `main`.

## Local preview

```bash
python3 -m venv .venv
.venv/bin/pip install -r requirements.txt
.venv/bin/mkdocs serve
```

Pull requests run `mkdocs build --strict`, so a broken link or a missing page fails the check.

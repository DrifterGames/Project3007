# VNFramework Wiki

Public documentation site for the **VNFramework** Unreal Engine 5.7 plugin — a designer-first visual novel framework built around the principle that *content is data, not Blueprints*.

📖 **Live site:** https://driftergames.github.io/Project3007/

## What lives here

Just the documentation. The plugin source, the editor project, and the in-progress visual novel that uses VNFramework live in a separate, private repository — this repo is a publish target for GitHub Pages only.

The wiki is built with [MkDocs Material](https://squidfunk.github.io/mkdocs-material/) and rebuilt automatically by a GitHub Actions workflow on every push to `main`.

## Build locally

```bash
pip install -r requirements-docs.txt
mkdocs serve
# → http://localhost:8000
```

## Tracks

- **Designers** — workflows, asset reference, importers (Ink/Yarn/CSV).
- **Developers** — architecture, API reference, extension patterns.
- **Educators** — glossary, suggested curriculum, assignment templates.

## Contributing

Edits are mirrored from the canonical (private) source repo, so direct PRs to this mirror will be overwritten by the next sync. If you spot an issue, open a GitHub issue here and the change will be made upstream.
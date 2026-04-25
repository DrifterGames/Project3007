# Ink import

If your story is written in [Ink](https://www.inklestudios.com/ink/) (the inkle-studios narrative scripting language), VNFramework can convert an `.ink` file (or a precompiled `.ink.json`) directly into a project asset, chapter assets, and scene assets — preserving knots, stitches, choices, conditions, and variable declarations.

This is the most full-featured of the three importers. Whole-project import is one menu click.

## What gets imported

| Ink construct | Becomes |
|---------------|---------|
| Top-level knot | Either a Chapter asset or a Scene asset (configurable — see "Knot mapping" below). |
| Stitch | Scene asset (when knots become chapters). |
| Dialogue line / `Speaker: text` | A Dialogue Line on a Scene. |
| `* choice` | An entry in the scene's End Choice. |
| `-> divert` | The scene's Next Scene (or a choice option's Target Scene). |
| `{ condition: text }` | A line-level Condition. |
| `~ var = expr` | A Variable Assignment on the surrounding line. |
| `VAR varname = default` | A Story variable on the project. |
| `LIST listname = a, b, c` | An Enum definition on the project. |
| Functions | Skipped (no equivalent in VNFramework). |

What doesn't import:

- Character placements / sprite changes (Ink doesn't carry this concept). You'll add them after import.
- Backgrounds, audio, transitions. Same — scene-level metadata is added after.
- External function calls (e.g. `~ play_sound("foo")`). Logged as warnings.

## Prerequisites

The importer needs `inklecate` (the official Ink compiler) on PATH if you're importing raw `.ink` files. If `inklecate` is not available, the importer surfaces an error message pointing you at the install. As an alternative, precompile your Ink elsewhere and import the resulting `.ink.json` instead — that path doesn't need `inklecate` on the editor machine.

## Workflow

### Run the import

1. **Visual Novel → Import Script… → Ink Script (.ink)**.
2. Pick your `.ink` (or `.ink.json`) file.
3. The import dialog opens. Configure options:

| Option | Default | What it does |
|--------|---------|--------------|
| Create Project Asset | true | Create a new VNProjectAsset wrapping the imported chapters. Off if you're importing into an existing project. |
| Knot Mapping | Auto-detect | Knots as Chapters, Knots as Scenes, or Auto-detect (decided by structure depth). |
| Save Path | `/Game/Imported/` | Content folder for created assets. |
| Sync Variables | on | Mirror Ink `VAR` declarations into Story variables on the project. |
| Sync Enums | on | Mirror Ink `LIST` declarations into Enum definitions. |
| Create Character Placeholders | on | Create stub Character assets for each unique speaker. |

4. Click **Import**.

### Knot mapping

Ink stories vary in shape. The two strategies:

- **Knots as Chapters** — top-level knots become chapter assets; their stitches become scene assets. Best for stories with a clear chapter / scene hierarchy in the Ink source.
- **Knots as Scenes** — every knot becomes a single scene asset. The chapter list ends up flat (one chapter containing every scene). Best for short / linear stories without explicit chapter divisions.

**Auto-detect** picks based on whether knots have stitches. If most knots have stitches, it picks Knots as Chapters; otherwise Knots as Scenes.

You can re-run the import with a different mapping if the auto-detect choice doesn't match your intent.

## What you'll have after import

```
/Game/Imported/
├── P_<inkfile>.uasset            ← project asset (if Create Project Asset is on)
├── Chapters/
│   ├── DA_<knot>.uasset
│   └── …
├── Scenes/
│   ├── DA_<stitch_or_knot>.uasset
│   └── …
└── Characters/
    ├── DA_<speaker>.uasset
    └── …
```

The project's Story Variables and Enum Definitions are populated from `VAR` and `LIST` declarations in the Ink source.

## Live re-import (file watcher)

The importer can watch `.ink` files for changes and re-import automatically when you save in your Ink editor. This means you can iterate on dialogue in inklecate's tooling and see the updated VN assets in Unreal without manually re-running the import.

The watcher tracks which Ink files map to which Unreal assets; on detection of a saved Ink file, it re-runs the import with the same options used originally. Initial import "registers" the file with the watcher.

!!! tip
    The watcher is most useful during pre-production when you're iterating on script structure. Once you start adding scene-level metadata (characters, backgrounds, audio) in the Unreal editor, **disable the watcher** — re-imports overwrite the dialogue arrays and may clobber your edits.

## Common patterns

!!! example "Whole-project import from a finished Ink draft"
    Click **Import Script → Ink**, point at your `.ink`, accept defaults. Out the other side: a project asset, chapter assets matching your top-level knots, scene assets matching your stitches, and a Story-variable list mirroring your `VAR` declarations. Total time: under a minute for a 5,000-line script.

!!! example "Pre-compiled JSON for CI"
    Run `inklecate -j story.ink` in your build pipeline. Commit `story.ink.json` alongside the source. Editor users without `inklecate` installed import the `.ink.json` directly.

!!! example "Iterative authoring"
    Import once with the watcher enabled. Edit `.ink` in your favorite editor; the watcher re-imports on save. Once script is locked, disable the watcher and start filling in scene-level state in Unreal.

## Pitfalls

!!! danger "Re-importing overwrites scene Dialogue Lines"
    Like CSV import, Ink re-import replaces the scene's Dialogue Lines array. Per-line metadata you added in the editor (Background Change, Sound Effects, Character Changes) **is wiped** on re-import. Lock down the Ink source before adding scene-level state.

!!! warning "External function calls become warnings"
    Ink lets you call external functions (`~ play_sound("foo")`). VNFramework's importer doesn't know what your custom externals do — it logs a warning and skips them. After import, replace those calls with VNFramework equivalents (e.g. line-level Sound Effects).

!!! warning "Inklecate not on PATH"
    If you're importing `.ink` directly and `inklecate` isn't installed or isn't in PATH, the import fails with a clear error pointing at the missing dependency. Either install inklecate or pre-compile to `.ink.json` and import that.

!!! warning "Speaker IDs in Ink can collide with VNFramework conventions"
    The importer creates a placeholder Character asset for each unique speaker. If your Ink uses speaker tags like `Mom: ...` and you already have a `Mom` character in your project, the import creates `DA_Mom` in `Imported/Characters/` — a duplicate. Rename / merge after import.

## See also

- [Scene asset reference](../reference/scene-asset.md)
- [Variables and scopes](../concepts/variables-scopes.md)
- [Expressions and conditions](../concepts/expressions.md)
- [CSV import](csv.md)
- [Yarn Spinner import](yarn.md)

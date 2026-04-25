# Yarn Spinner import

If your dialogue is written in [Yarn Spinner](https://docs.yarnspinner.dev/), VNFramework can convert a `.yarn` file into chapter and scene assets, with each Yarn node becoming a scene.

The Yarn importer is similar in shape to the Ink importer but more constrained — Yarn's flatter structure (nodes, not knots-with-stitches) maps cleanly onto a single chapter of scenes.

## What gets imported

| Yarn construct | Becomes |
|----------------|---------|
| Yarn node | A Scene asset. |
| Dialogue line / `Speaker: text` | A Dialogue Line on the scene. |
| `-> jump <node>` | The scene's Next Scene. |
| `-> option` | An entry in the scene's End Choice. |
| `<<if condition>>` … `<<endif>>` | A line-level Condition. |
| `<<set var = expr>>` | A Variable Assignment on the surrounding line. |
| `<<declare var = default>>` | A Story variable on the project. |
| `visited(node)` etc. | Synthesized Story variables tracking node visits. |

What doesn't import:

- Character placements / sprites.
- Backgrounds, audio, transitions.
- Custom commands (`<<wait 2>>`, `<<myCustomCmd>>`). Logged as warnings.

## Workflow

### Existing project (typical case)

You already have a VNProjectAsset and want to add a chapter from a Yarn file:

1. **Visual Novel → Import Script… → Yarn Spinner Script (.yarn)**.
2. Pick your `.yarn` file.
3. In the import dialog:

| Option | Default | What it does |
|--------|---------|--------------|
| Target Project | (asset picker) | Pick the existing VNProjectAsset to add the imported chapter to. |
| Create Chapter | true | Create a new chapter asset wrapping the imported scenes. |
| Chapter Name | (filename) | Name for the new chapter. |
| Save Path | `/Game/Imported/` | Where created assets go. |
| Sync Variables | on | Mirror Yarn `<<declare>>` into Story variables. |
| Sync Synthesized Variables | on | Mirror `visited(...)` tracking variables. |
| Create Character Placeholders | on | Create stub Character assets for unique speakers. |

4. Click **Import**.

### Standalone import

If you don't have a project yet, the importer can create a chapter and scene set without registering them on a project. You'll add them to a project asset manually afterward.

### Existing chapter

If you want to add scenes to an *existing* chapter (rather than create a new one), set Create Chapter to off and pick the target chapter.

## What you'll have after import

```
/Game/Imported/
├── Chapters/
│   └── DA_<chaptername>.uasset
├── Scenes/
│   ├── DA_<nodename>.uasset
│   └── …
└── Characters/
    ├── DA_<speaker>.uasset
    └── …
```

Story variables on the target project are updated with declared Yarn variables. The new chapter is added to the project's Chapters array (when imported into a project).

## Variable handling

Yarn declares variables as `<<declare $varname = default as type>>`. The importer maps:

| Yarn type | VNFramework type |
|-----------|------------------|
| Bool | Boolean |
| Number | Float |
| String | String |

There's no native Enum equivalent in Yarn — use String variables or an Enum definition you author manually on the project.

Yarn's `visited()` and `visited_count()` helpers are mirrored as **synthesized variables** the importer adds (one per node referenced by `visited(...)`). This lets imported Conditions that test `visited(<node>)` translate into `Story.visited_<node> > 0` on the VN side.

## Common patterns

!!! example "Single-chapter Yarn project"
    A short story authored as one `.yarn` file with 30 nodes. Import → one chapter, 30 scenes, all variable declarations mirrored to Story scope. Add scene-level state (characters, backgrounds) in Unreal afterward.

!!! example "Multi-chapter from multiple Yarn files"
    Author each chapter as a separate `.yarn` file. Run the importer once per file, picking a different Chapter Name each time. The project ends up with one chapter per file, each containing its scenes. Order them in the project's Chapters array.

!!! example "Re-import iteration"
    Lock down the Yarn-side script structure and dialogue first. Then re-import once final. After that, switch to editing in Unreal — re-import would overwrite per-line scene metadata you've added.

## Pitfalls

!!! danger "Re-importing overwrites scene Dialogue Lines"
    Same warning as Ink and CSV: re-import replaces the dialogue list. Editor-side per-line edits (Character Changes, Sound Effects, Background Change) are wiped. Iterate on Yarn first, then add VN-side state.

!!! warning "Custom Yarn commands aren't translated"
    `<<wait 2>>`, `<<scenechange foo>>`, and any project-specific `<<command>>` calls aren't recognized by the VNFramework importer. They're logged as warnings and skipped. Replace with VN equivalents (Auto Advance Delay, scene Next Scene, line-level effects) post-import.

!!! warning "Yarn's Number type maps to Float, not Int"
    If your Yarn variables are conceptually integers (counters, gold), they import as VNFramework Floats. Cast back to Int in expressions if needed (`int(Story.gold)`), or change the variable type to Integer on the project asset post-import (the runtime tolerates the change).

!!! warning "Character Placeholders may duplicate existing characters"
    If your Yarn speakers have the same names as existing characters in the project, the importer still creates placeholder character assets for them in `Imported/Characters/`. Merge or remove duplicates after import.

## See also

- [Scene asset reference](../reference/scene-asset.md)
- [Variables and scopes](../concepts/variables-scopes.md)
- [Expressions and conditions](../concepts/expressions.md)
- [CSV import](csv.md)
- [Ink import](ink.md)

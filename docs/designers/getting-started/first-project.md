# Your first project

A **Project asset** is the root of every visual novel. It's the hub that points at your chapters, characters, backgrounds, theme, and project-wide settings. Create one before you create anything else.

## Create the asset

1. In the **Content Browser**, navigate to where you want VN content to live. A common convention is `/Game/VN/Project/`, but the framework doesn't enforce paths.
2. **Right-click → Miscellaneous → Data Asset**.
3. In the asset picker that opens, search for `VNProjectAsset` and pick it.
4. Name it. A `P_` prefix (e.g. `P_MyStory`) reads cleanly in the Content Browser, but again — no enforcement.

Double-click to open it in the Details panel.

!!! tip
    You can use the editor's **Visual Novel → Create New…** menu instead. It does the same thing but pre-fills sensible defaults.

## Fill in project info

The top section of the panel is just metadata for you:

| Field | What it is |
|-------|------------|
| Title | Display title shown on the main menu / save list. |
| Version | Free-form string (e.g. `1.0.0`). Useful for build tracking. |
| Description | Multi-line author notes. Strip-cooked out of shipping builds. |

## Add your first chapter

Stories are organized into chapters. You'll create chapter assets (one per chapter) and then list them on the project.

1. In the Content Browser, **Right-click → Miscellaneous → Data Asset → VNChapterAsset**. Name it `C_Prologue` (or whatever your first chapter is called) and save it under `/Game/VN/Chapter/`.
2. Back on the project asset, find the **Project | Content** section.
3. Expand the **Chapters** array, click **+** to add a slot, then assign your new chapter asset to it.
4. Set **Starting Chapter** to the same chapter — this is where the player begins on a fresh playthrough.

You don't have to fill the chapter in yet — we'll come back to it on the next page.

## Pick a theme

The **Theme** controls colors, frames, fonts, and icons across every VN screen. The framework ships with a sensible default fallback if you leave it empty, so you can absolutely skip this for now.

When you're ready:

1. Create a `VNUITheme` asset under `/Game/VN/Theme/`. Name it `T_Default`.
2. On the project asset, set **Project | Presentation → Theme** to `T_Default`.

You can author the theme's colors, fonts and frames any time — the project asset will pick up changes automatically.

See the [Theme pack reference](../reference/theme-pack.md) for the full list of theme settings.

## Optional: a title scene

The **Title Scene** field on the project lets you specify a scene to play as the main-menu backdrop (with its own background, music, and characters). If you leave it empty, the game starts directly at the first chapter — useful for fast playtesting iterations.

You don't need this yet. Come back once you have a scene asset to point at.

## Settings worth glancing at

Under **Project | Settings**, defaults are fine for most projects. The ones designers tweak first:

| Field | Default | When you'd change it |
|-------|---------|----------------------|
| Default Text Speed | 30 | Characters per second for the typewriter effect. |
| Auto Advance Settings | Base 2.0s | How long to wait before auto-continuing on lines without voice. |
| Skip Settings | 0.05s, skip-read-only | How fast skip mode runs and whether it skips unread text. |
| Max Save Slots | 20 | Visible save slot count. |
| Max Backlog Entries | 100 | Lines kept in the in-game history view. |

## Save the asset

**Ctrl+S** (or right-click → Save) on the project asset. Do this often — Unreal won't auto-save your data assets.

## What you have now

A project asset that:

- knows its title and version,
- points at one chapter,
- knows where the player should start,
- (optionally) has a theme.

The chapter is empty. The next page fills it.

## See also

- [Build your first scene](first-scene.md)
- [Project asset reference](../reference/project-asset.md) — every field on this asset.
- [Chapter asset reference](../reference/chapter-asset.md)
- [What is a DataAsset?](../concepts/data-assets.md)

# Your first scene

A **Scene asset** is where the actual writing lives — dialogue lines, character placements, background, audio. This page walks you through building one minimal scene from scratch.

## Create the scene asset

1. Navigate the Content Browser to `/Game/VN/Scene/` (or wherever you keep scenes).
2. **Right-click → Miscellaneous → Data Asset → VNSceneAsset**.
3. Name it `S_Intro` (the `S_` prefix is convention, not required).
4. Open it.

You can also use **Visual Novel → Create New… → Scene** from the editor's main menu — same result.

## Hook the scene to the chapter

Open the chapter asset you made on the previous page and:

1. Expand **Chapter | Scenes → Scenes** array, click **+**, assign `S_Intro`.
2. Set **Starting Scene** to `S_Intro` so the chapter knows where to begin.

Save the chapter.

## Fill in scene metadata

Back on `S_Intro`, the top section:

| Field | What to put |
|-------|-------------|
| Scene ID | A short unique name. Convention: match the asset name (e.g. `S_Intro`). |
| Display Name | Friendly label, only seen in editor / debug overlays. |
| Description | Author notes. Cooked out of shipping. |
| Scene Type | Leave at **2D Sprite** for a traditional VN. **2D Parallax** adds depth-shift layers. **3D Scene** and **Mixed** are experimental — avoid for now. |

## Pick a background

1. Create a `VNBackgroundAsset` under `/Game/VN/Background/` if you don't have one yet. For a simple test, set its **Background Type** to **2D Image** and assign a texture to **Background Texture**.
2. On the scene, **Scene | Visuals → Background** → pick that asset.

Save both.

The background is what shows when the scene loads. You can also swap backgrounds mid-scene per dialogue line (see "More tricks" below).

## Place a character

This is a two-step process: there's a **character asset** that defines who the character is (their sprites, their voice settings), and a **placement** entry on the scene that says "this character is here, in this expression, at this position".

### Character asset

1. Create a `VNCharacterAsset` under `/Game/VN/Character/`. Name it `C_Hero`.
2. Set **Character ID** to `hero`. This short ID is what dialogue lines reference.
3. Set **Display Name** to `Hero` (or whatever shows in the dialogue box).
4. Leave **Character Type** at **2D Sprite**.
5. Expand **Character | 2D → Expressions**, click **+**, and create at least one entry:
    - **Expression Name**: `neutral`
    - **Sprite**: pick a character texture
6. Set **Default Expression** to `neutral`.

Save.

### Add the character to the scene

Back on `S_Intro`, **Scene | Visuals → Characters** → click **+** to add a placement:

| Placement field | Set to |
|-----------------|--------|
| Character | `C_Hero` |
| Character ID | `hero` (must match the asset's CharacterID) |
| Initial Expression | `neutral` |
| Position | `(0.5, 0.6)` — center, slightly below middle |
| Scale | `1.0` (or `1.75` for full-height presence) |
| Layer | `0` (higher = drawn in front) |
| Start Visible | true |

!!! warning
    The **Character ID** field on the placement and the **Character ID** on the character asset must match. If they don't, dialogue lines that reference `hero` won't know which placement to talk to.

## Write some dialogue

Expand **Scene | Dialogue → Dialogue Lines** and click **+** to add lines. Each entry:

| Field | What goes here |
|-------|----------------|
| Speaker ID | `hero` for the hero, empty for narration |
| Dialogue Text | The actual text. Multi-line. Supports rich-text tags (`<color>`, `<bold>`) when the theme enables them. |
| Voice Clip | Optional voice-over audio. Auto-plays when the line shows. |
| Character Changes | Per-line expression / position changes (see below). |

A minimal first scene might look like:

| # | Speaker | Text |
|---|---------|------|
| 1 |         | The lights flicker on. |
| 2 | hero    | Where… am I? |
| 3 | hero    | Hello? Anyone here? |

!!! tip
    Lines with no Speaker ID render as narration. Use them for descriptive beats.

### Character changes per line

On each line, **Character Changes** lets you swap a character's expression, move them, scale them, or hide them. One entry per character affected:

- **Character ID** → which character to change (must be in the scene's Characters list).
- **Expression** → new expression name (leave empty to keep current).
- **bChangePosition / Position** → check and set to move them. New positions animate in over **Transition Duration** seconds.
- **bChangeVisibility / bVisible** → fade in or out.
- **bFlipHorizontal** → mirror the sprite (toggles current state).
- **Transition Duration** → seconds to animate. `0` = instant snap.

Example: on line 2 (hero asks "where am I?") add a Character Change with `Character ID = hero`, `Expression = surprised`, `Transition Duration = 0.3`.

### Auto-advance and conditions

Two more useful per-line settings, both under **Advanced** or visible by default:

- **Auto Advance** + **Auto Advance Delay** → continue automatically after N seconds. Use for narration beats.
- **Condition** → expression that must evaluate true for the line to display. Empty = always shown. See [Expressions](../concepts/expressions.md).

## What happens at the end?

By default the scene plays through every line then continues to **Scene | Flow → Next Scene**. Set this to the next scene asset in the chapter, or leave it empty to end on this scene.

To branch into a player choice instead, see [Add your first choice](first-choice.md).

## More tricks (skim now, return later)

The scene's Details panel has more sections that you don't need yet but should know exist:

- **Scene | Audio** — background music, ambient bed, scene-level reverb. See [Audio](../concepts/audio.md).
- **Scene | Visuals → Entry Transition** — the fade/dissolve when this scene loads. See [Transitions](../concepts/transitions.md).
- **Per-line Background Change** — swap the background mid-scene (camera-cut style) without authoring a new scene asset.
- **Per-line Sound Effects** — schedule one or more SFX cues with delay + volume per cue.
- **Conditional Next Scenes** — branch flow on variable state without showing a player choice.

## Test it

1. Open a level (the framework's default game mode handles the rest).
2. Press **Play in Editor**.
3. The first chapter's starting scene loads, dialogue plays, you click to advance.

If nothing happens, check the [installation troubleshooting](installation.md#troubleshooting) — usually it's the GameMode INI line or a missing Asset Manager block.

## See also

- [Add your first choice](first-choice.md)
- [Scene asset reference](../reference/scene-asset.md) — every field on this asset.
- [Character asset reference](../reference/character-asset.md)
- [Background asset reference](../reference/background-asset.md)
- [Transitions](../concepts/transitions.md)

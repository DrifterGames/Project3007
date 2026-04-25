# Project asset reference

`UVNProjectAsset` is the root data asset for a visual novel. It points at every chapter, character, and background in the game; declares story-wide and system-wide variables; configures default text speed / auto-advance / skip behavior; and holds the project's UI theme.

You'll have exactly one project asset per game.

![Project asset Details panel — DA_Project_the_climb from The Climb](../../assets/screenshots/vnprojectasset-details.png)

## Properties

### Project info

| Name | Type | Default | Used for |
|------|------|---------|----------|
| Title | Text | (empty) | Display title (main menu, save list). |
| Version | String | `1.0.0` | Build / version tracking. Free-form. |
| Description | Text (multi-line) | (empty) | Author notes. Strip-cooked from shipping builds. |

### Content

| Name | Type | Default | Used for |
|------|------|---------|----------|
| Chapters | Array of Chapter assets | empty | The full chapter list. Order is reading order. |
| Characters | Array of Character assets | empty | Cast library. Reference your characters here so the validator sees them. |
| Backgrounds | Array of Background assets | empty | Location library. Same as Characters — registration list. |
| Starting Chapter | Chapter asset | empty | Where a fresh playthrough begins. |
| Title Scene | Scene asset | empty | Optional scene played as the main-menu backdrop. Empty = game starts directly at the first chapter (faster for playtesting). |

### Variables

| Name | Type | Default | Used for |
|------|------|---------|----------|
| Story Variables | Array of Variable definitions | empty | Persist across chapters and saves; reset on new game. |
| System Variables | Array of Variable definitions | empty | Persist across saves and new games. Use sparingly (game completion, accessibility prefs). |
| Enum Definitions | Array of Enum definitions | empty | Named value lists usable as Enum-typed variables. |

### Settings

| Name | Type | Default | Used for |
|------|------|---------|----------|
| Default Text Speed | Float | 30.0 | Characters per second for typewriter reveal. |
| Auto Advance Settings | Struct | base 2.0s, +0.05s/char, wait for voice = true | Default auto-advance timing. |
| Skip Settings | Struct | 0.05s, skip-read-only, stop-at-choices | Default skip mode behavior. |
| Max Save Slots | Int | 20 | Visible save slot count. |
| Max Backlog Entries | Int | 100 | Lines kept in the in-game history view. |

### Presentation

| Name | Type | Default | Used for |
|------|------|---------|----------|
| Theme | UI Theme asset | empty | Project-wide theme. Empty = built-in fallback. |
| Light Wrap Settings | Struct | enabled, width 0.008, intensity 0.5 | Default character-edge background bleed. Per-scene override exists. |

## Common patterns

!!! example "Lean playtesting setup"
    Skip the main menu: leave **Title Scene** empty. Game launches directly into the starting chapter. Saves you a click on every iteration.

!!! example "Per-character variables on the project"
    Declare `Story.rel_<character>` (Float, default 0) for every named character on the project. Story scope means relationships persist across the whole playthrough. Chapter / scene scripts increment them via Variable Assignments on dialogue lines and choice options.

!!! example "Track read text"
    Declare a System variable `seen_lines` (String) that the framework writes line GUIDs into. The skip-read-only behavior reads from this set automatically.

## Pitfalls

!!! danger "Chapters list isn't autopopulated"
    Creating a Chapter asset doesn't add it to the project's Chapters array. You have to add it explicitly. The validator (Visual Novel → VN Asset Validator) will flag chapters not referenced anywhere.

!!! warning "Starting Chapter must be in Chapters"
    Setting Starting Chapter to a chapter that isn't in the Chapters array works at runtime but fails the validator and breaks chapter selection UIs that iterate the array.

!!! warning "System variables don't reset on new game"
    Putting "did the player help the cat?" on a System variable means it carries between playthroughs. Use Story scope for per-playthrough state; reserve System for genuine cross-save state.

## See also

- [Build your first project](../getting-started/first-project.md) — step-by-step.
- [Chapter asset reference](chapter-asset.md)
- [Variables and scopes](../concepts/variables-scopes.md)
- [Theme pack reference](theme-pack.md)

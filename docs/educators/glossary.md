# Glossary

A combined vocabulary list covering visual novel craft and the specific names VNFramework gives to its parts. Use this as a handout, a quick-reference for grading, or a starting point for a class discussion about what each term really means in practice.

Entries are alphabetical. Each one points to where the concept shows up in the [Designer track](../designers/index.md) or the [Developer track](../developers/index.md) so students can dig deeper.

!!! tip "Instructor note"
    The first time a student meets one of these words in your course, take ninety seconds to ask the room what they think it means before showing the definition. The mismatch between everyday usage ("expression") and framework usage ("a named sprite swap on a character asset") is where confusion lives.

---

**Ambience.** A continuous background sound — wind, rain, a crowded cafe — layered under dialogue to establish place. In VNFramework, ambience plays on a dedicated audio channel, separate from music, so it can persist across dialogue lines without ducking. See the [Designer track](../designers/index.md) for the audio channel breakdown.

**Auto-advance.** A playback mode in which dialogue lines progress on a timer instead of waiting for a click. Useful for accessibility and for cinematic pacing. Per-line timing can be authored on individual dialogue entries.

**Backlog.** The scrollable history of dialogue lines the player has already seen, usually with the option to replay voice clips. Treat it as a reading aid, not a "rewind" — choices made earlier do not unmake themselves when the player scrolls back.

**Background asset.** The framework's container for a scene's backdrop. A single asset can describe a still image, a parallax stack, a 3D scene, or a video. Designers pick one type per asset and fill in the relevant fields. Cross-reference: [Designer track](../designers/index.md).

**Backplate.** The semi-transparent panel behind dialogue text. Its job is contrast — keeping text readable over varied art. Often part of the theme's frame pack.

**BGM.** Background music. The looping musical bed of a scene. Distinct from ambience (environmental sound) and SFX (one-shot effects).

**Branching narrative.** A story whose path changes based on player input. The smallest branch is one choice with two outcomes; large branches form trees or webs. The framework expresses branches through choices and conditions on scenes and dialogue lines.

**Chapter asset.** A `UVNChapterAsset` — the mid-level container that groups scenes into a narrative unit. A project asset references chapters; chapters reference scenes. See the [Designer track](../designers/index.md).

**Character asset.** A `UVNCharacterAsset` — a reusable definition of one character, including display name, expressions or animations, and rendering type (2D sprite, 3D skeletal, Live2D, Spine). One asset per character; reuse it across every scene that character appears in.

**Character expression.** A named visual variant of a character — "happy", "angry", "thinking". For 2D characters this is a sprite swap; for 3D, an animation. Authored as entries on the character asset and referenced by name from scenes.

**Choice point.** The moment where the player picks among options to determine what happens next. In the framework, a scene can carry an `EndChoice` with a list of choice options, each pointing at a follow-up scene or setting a variable.

**Choice timer.** A countdown attached to a choice point — if it expires, a default option fires. Useful for tension and for accessibility (no timer = no pressure).

**Condition.** A boolean expression evaluated against the variable store to decide whether a line, choice, or branch is shown. Conditions read variables; choices and assignments write them. See the expression-evaluator notes in the [Developer track](../developers/index.md).

**DataAsset.** Unreal Engine's term for a content-only asset whose data is edited in the Details panel. The whole VNFramework philosophy is "everything is a DataAsset" — students fill in fields rather than write code. Foundational to the [Designer track](../designers/index.md).

**Dialogue line.** One speaker-and-text entry inside a scene. Carries the speaker ID, the line text, optional voice clip, an optional condition, and an auto-advance flag. The atomic unit students will create most of.

**Dialogue tree.** The total shape of a story's branches — the diagram you would draw on a whiteboard to show every path. The framework does not store a "tree" object; the tree is implicit in how scenes link to each other.

**Enum definition.** A named list of allowed values for a narrative variable — e.g. `E_Mood = { Calm, Tense, Furious }`. Authored on the project asset and referenced from variable definitions. Lets students avoid "magic strings."

**Expression (character).** See **Character expression**.

**Expression (logic).** A short string evaluated against variables to produce a value or a boolean — e.g. `Trust >= 3 && Met_Hero`. Used in conditions and in choice availability.

**Frame pack.** The slice of a theme that defines how panels are drawn — 9-slice borders, corner artwork, drop shadows. Swapping frame packs reskins every dialogue and choice widget without touching scene data.

**Font pack.** The theme component that selects display fonts for body text, character names, and UI labels.

**Icon pack.** The theme component that supplies the small images for save/load, settings, and other UI affordances.

**Project asset.** A `UVNProjectAsset` — the top of the asset tree. Holds the chapter list, the character roster, the background list, the theme reference, and the variable and enum definitions for the whole story.

**Save state.** A snapshot of which scene is active, which line is current, and the values of all story- and system-scope variables. The framework writes save states; students rarely have to think about them, but should know what is captured (and therefore what variables to choose for what they want to persist).

**Scene asset.** A `UVNSceneAsset` — a single beat of the story. Contains an ordered list of dialogue lines, the character placements for that scene, the background reference, optional choice prompt, and the link to the next scene. The unit students will most often open and edit.

**Scene type.** How the scene is rendered: `Sprite2D` for traditional 2D, `Parallax2D` for layered scrolling backgrounds, `Full3D` (experimental) for fully 3D scenes, `Mixed` (experimental) for combinations. Picked per scene.

**Skip mode.** A playback mode that fast-forwards through dialogue, traditionally only through already-read text. Distinct from auto-advance, which still respects per-line timing.

**Sprite.** A 2D image of a character, usually transparent-background, swapped between expressions. In the framework, sprites are referenced by character expression entries on the character asset.

**Story variable.** A variable scoped to one playthrough — persists across scenes and chapters but resets when the player starts a new game. The right home for "did the player meet the hero" or "trust level."

**System variable.** A variable that persists across saves and playthroughs — settings, achievements, a count of total endings seen. Use sparingly.

**Theme pack.** A `UVNUITheme` plus its frame pack, font pack, and icon pack. The unit students swap to give their VN a different look without changing any scene content. The third assignment in the [assignment templates](assignment-templates.md) builds one from scratch.

**Transition.** The visual handoff between two scenes or two backgrounds — `None`, `Fade`, `Dissolve`, `Slide`, or `Custom`. A small detail that disproportionately affects perceived polish.

**Variable scope.** Where a variable lives and how long it survives. From narrowest to widest: Scene (transient), Chapter (per-chapter), Story (per-playthrough), System (across saves). Choosing the right scope is one of the most common places students need feedback.

**Voice acting (VO).** Recorded performance of dialogue lines. The framework supports per-line voice clip references. Encourage students to record placeholder VO themselves even if a final cast will replace it — pacing changes everything.

**Widget.** The framework's UI building blocks — dialogue boxes, choice menus, save/load screens. Students rarely build new widgets; they swap themes to reskin existing ones. For students who do want to author widgets, see the [Developer track](../developers/index.md).

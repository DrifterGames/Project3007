# Variables and scopes

Variables let your story remember things. Did the player help the cat? What's their relationship score with the rival? Have they seen this scene before? VNFramework gives you four variable *scopes*, each with a different lifetime — pick the right one and the runtime handles persistence automatically.

## The five variable types

| Type | Holds | Examples |
|------|-------|----------|
| Boolean | `true` / `false` | `helped_cat`, `door_unlocked` |
| Integer | Whole number | `gold`, `times_visited`, `chapter_number` |
| Float | Decimal number | `affection`, `path_alignment`, `hours_elapsed` |
| String | Text | `player_name`, `chosen_alias` |
| Enum | One of a named list | `mood = "happy" | "neutral" | "angry"` |

Booleans and Integers are the most common. Floats are useful when you want gradient stat changes. Strings hold names or freeform tags. Enums are a typed alternative to "use an Integer and remember 1=happy".

!!! tip
    Prefer Boolean over Integer when the answer is yes/no. Prefer Enum over String when the value comes from a fixed list — it stops typos from becoming silent bugs.

## The four scopes

Every variable belongs to one of four scopes. The scope determines **when the variable resets** and **where you declare it**.

| Scope | Declared on | Cleared when | Use for |
|-------|-------------|--------------|---------|
| **Scene** | (transient — not declared) | The scene ends | Tiny per-scene flags. Rare in practice. |
| **Chapter** | Chapter asset | A *different* chapter loads | Per-chapter bookkeeping (e.g. "did the player explore the library this chapter?") |
| **Story** | Project asset | A new playthrough begins | Choices that affect the story arc, relationship scores, inventory |
| **System** | Project asset | Never (persists across saves and playthroughs) | "Has the player ever beaten the game?", read-text tracking, accessibility prefs |

Reading top-to-bottom: the further down the table, the longer the variable lives.

```mermaid
graph LR
    Scene([Scene]) -->|narrowest| Chapter([Chapter])
    Chapter --> Story([Story])
    Story --> System([System])
    System -->|widest| End([Persists forever])
```

## Where to declare variables

### Story variables

Open your project asset → **Project | Variables → Story Variables** → click **+** to add an entry. Set Name, Type, Default Value, Description. Story variables are reset when the player starts a new game; they're saved with the player's save slot.

### System variables

Same place: **Project | Variables → System Variables**. System variables persist across all save slots and survive starting a new game — they're stored in a single global save the framework manages. Use sparingly, mostly for game-completion flags and read-text tracking.

### Chapter variables

Open the chapter asset → **Chapter | Variables → Chapter Variables**. These reset when the player navigates to a different chapter, so use them for scoped state that shouldn't bleed into later acts.

### Scene variables

Scene-scoped variables are *transient*. You don't declare them on an asset; you assign to them mid-scene and they vanish when the scene ends. They're rare in practice — most things you'd want to remember outlive a single scene.

## Referencing variables

In conditions and assignments, prefix the variable name with its scope:

```
Story.helped_cat
Story.affection >= 5
Chapter.times_visited
System.has_seen_credits
Scene.local_flag
```

The scope prefix is required when reading. If you write `helped_cat` with no prefix, the resolver doesn't know which scope to look in.

## Setting variables

There are two places to assign:

**On a dialogue line** (Advanced section → **Variable Assignments** array). One assignment per array entry. Fires when the line displays.

```
Story.affection = Story.affection + 1
```

**On a choice option** (Variable Assignments array). Fires when the player picks the option.

```
Story.chose_path_a = true
```

You can do arithmetic on the right-hand side. See [Expressions](expressions.md) for what's allowed.

## Reading variables in conditions

Anywhere the editor offers a **Condition** field — choice options, conditional next-scene entries, dialogue lines, chapter unlock — you write an expression that evaluates true or false:

```
Story.affection >= 5
Story.helped_cat == true
(Story.gold > 100) && (Story.path == "thief")
!System.has_seen_credits
```

See the [Expressions](expressions.md) page for the full operator list.

## Default values

Every variable definition includes a **Default Value** field (a string parsed according to type). This is the value the variable starts with on a fresh playthrough — for Story variables, on new game; for Chapter variables, on entering that chapter for the first time; for System variables, on first ever launch.

Examples:

| Type | Valid Default Values |
|------|----------------------|
| Boolean | `true`, `false` |
| Integer | `0`, `42`, `-1` |
| Float | `0.0`, `1.5`, `-0.25` |
| String | `Anonymous` (no quotes — the whole string field is the value) |
| Enum | `neutral` (must match one of the enum's declared values) |

## Read-only variables (constants)

Tick **Is Read Only** on a variable definition to make it a constant. The framework will refuse runtime assignments to it. Useful for tuning numbers you want the rest of the story to read but never modify (e.g. `Story.max_affection = 10`).

## Enums

Enums let you define a typed list of valid values, then declare variables of that enum type.

1. On the project asset, **Project | Variables → Enum Definitions** → add an entry:
    - **Enum Name**: `mood`
    - **Values**: `happy`, `neutral`, `angry`
2. Now declare a Story variable of type **Enum**:
    - **Name**: `hero_mood`
    - **Type**: Enum
    - **Enum Type Name**: `mood`
    - **Default Value**: `neutral`

The runtime treats enums as integers under the hood (auto-indexed 1, 2, 3 unless you pin **Explicit Values**), but writes and reads use the string names so your conditions are readable: `Story.hero_mood == "angry"`.

!!! warning
    The Enum Type Name on a variable must match the Enum Name on a definition exactly, including case. Typos here mean the variable falls back to integer-like behavior with empty value lookups.

## Pitfalls

!!! danger "Choose your scope deliberately"
    The most common bug: putting "did they help the cat in chapter 1?" on a *Chapter* variable, then reading it from chapter 3. By the time chapter 3 loads, the chapter 1 variable has been cleared. Use **Story** scope for anything that should outlive its chapter.

!!! warning "Names are case-sensitive"
    `Story.HelpedCat` ≠ `Story.helpedcat`. Pick a convention (snake_case is common) and stick to it.

!!! warning "Strings don't take quotes in Default Value"
    The `Default Value` field is itself a string field. Don't write `"Anonymous"` — write `Anonymous`. Quotes get included literally.

## See also

- [Expressions and conditions](expressions.md) — the syntax used in conditions and assignments.
- [Project asset reference](../reference/project-asset.md) — declaring Story / System variables.
- [Chapter asset reference](../reference/chapter-asset.md) — declaring Chapter variables.

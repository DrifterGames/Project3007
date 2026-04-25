# Your first choice

A choice branches the story. The player picks an option, and the game jumps to a different scene (and optionally sets a variable). Choices live at the *end* of a scene — author the lead-up dialogue, then add the choice menu, then point each option at a different next scene.

## The setup

You need at least three scenes:

- `S_Intro` — the lead-in (you built this on the previous page).
- `S_Path_A` — what happens if the player picks option A.
- `S_Path_B` — what happens if the player picks option B.

Create `S_Path_A` and `S_Path_B` as bare scene assets the same way you created `S_Intro`. They can be empty for now — one dialogue line each is enough to confirm the branch works.

Add all three to the chapter's **Scenes** array.

## Turn on the end choice

Open `S_Intro`. In **Scene | Dialogue**:

1. Tick **Has End Choice**.
2. The **End Choice** struct expands. The **Next Scene** field disappears (it's ignored when there's an end choice).

## Author the prompt

Inside **End Choice**:

| Field | What it does |
|-------|--------------|
| Prompt Text | Optional text shown above the option list (e.g. "What do you do?"). Leave empty for no prompt. |
| Choices | Array of options, one per branch. |
| Time Limit | Seconds the player has to choose. `0` = no limit. |
| Default Choice Index | Which option fires if time runs out (only used when Time Limit > 0). |

## Add two options

Expand **Choices** and click **+** twice. Configure each entry:

### Option 1 — "Run for the door"

| Field | Value |
|-------|-------|
| Choice Text | `Run for the door.` |
| Target Scene | `S_Path_A` |

### Option 2 — "Stay and listen"

| Field | Value |
|-------|-------|
| Choice Text | `Stay and listen.` |
| Target Scene | `S_Path_B` |

That's the minimum. Save and play — you should see the dialogue, then the two-option menu, then jump to whichever scene you picked.

## Tracking the choice

Most of the time you want the choice to *do* something — set a flag the rest of the story can read. That's what **Variable Assignments** are for.

First, declare a story-scoped variable on the project asset. Open your project asset and under **Project | Variables → Story Variables**, add an entry:

| Field | Value |
|-------|-------|
| Name | `chose_path_a` |
| Type | Boolean |
| Default Value | `false` |
| Description | Did the player pick path A in the intro? |

Then back on the choice option:

- On **Option 1**, expand **Variable Assignments** and add an entry: `Story.chose_path_a = true`
- On **Option 2**, add: `Story.chose_path_a = false`

Now any later scene can read `Story.chose_path_a` in a condition. See [Variables and scopes](../concepts/variables-scopes.md) for what `Story.` means and the other scopes.

## Conditional / locked options

Sometimes an option should only appear when something earlier happened. Use **Condition** on the choice option:

| Field | What it does |
|-------|--------------|
| Condition | Expression. The option is hidden if the condition evaluates false. Empty = always available. |
| bShowWhenUnavailable | If true, the option still renders but is greyed out and unselectable when the condition fails. |
| Unavailable Text | Optional alternate text shown when greyed out (e.g. "Need a key" instead of "Open the door"). |

Example: an option that's only available if the player has 5+ relationship points with the hero:

```
Story.rel_hero >= 5
```

See [Expressions and conditions](../concepts/expressions.md) for the full syntax.

## Branching without a player choice

If you want flow to branch on variable state *without* presenting a menu (a "the universe decides for you" moment), don't use End Choice. Instead, open the scene's **Scene | Flow** section, expand **Conditional Next Scenes**, and add entries — each with a Condition and a Target. The first truthy condition wins; if none match, the scene falls through to **Next Scene**.

Example: at the end of a recap scene, send the player to one of three branches based on accumulated stats:

| Condition | Target |
|-----------|--------|
| `Story.path_alignment > 3` | `S_Hero_Ending` |
| `Story.path_alignment < -3` | `S_Villain_Ending` |
|  *(fallback via Next Scene)* | `S_Neutral_Ending` |

This is how Ink-style `{ stat > N: -> target }` branches translate into VNFramework. Reach for it when the player shouldn't see the decision happening.

## See also

- [Expressions and conditions](../concepts/expressions.md) — operator syntax, examples.
- [Variables and scopes](../concepts/variables-scopes.md) — when does `Story.` persist?
- [Scene asset reference](../reference/scene-asset.md) — every field on End Choice and Conditional Next Scenes.

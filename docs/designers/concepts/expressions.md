# Expressions and conditions

VNFramework's expression language is what you write in **Condition** fields and **Variable Assignments**. It looks like math + booleans, with variable lookups via scope prefixes. This page lists every operator and function the evaluator supports.

## Where you'll write expressions

| Field | What the expression should produce |
|-------|------------------------------------|
| Dialogue line **Condition** | Boolean — line shows only if true. |
| Dialogue line **Variable Assignments** | Statement of the form `Scope.var = expression`. |
| Choice option **Condition** | Boolean — option enabled only if true. |
| Choice option **Variable Assignments** | Same as above. |
| Scene **Entry Condition** | Boolean — scene plays only if true. |
| **Conditional Next Scenes → Condition** | Boolean — first truthy entry wins. |
| Chapter **Unlock Condition** | Boolean — chapter selectable only if true. |

## Variable references

Always prefix with the scope:

```
Story.affection
Chapter.times_visited
System.has_seen_credits
Scene.local_flag
```

Without a prefix the resolver doesn't know which scope to read from.

## Literals

```
42              integer
3.14            float
true / false    boolean
"hello world"   string (double-quoted)
```

## Operators

The evaluator parses with conventional precedence. From **highest** to **lowest**:

| Tier | Operators | Example |
|------|-----------|---------|
| 1 | Parentheses, literals, variable refs, function calls | `(Story.gold + 5)` |
| 2 | Unary `-`, `!` | `-x`, `!Story.busy` |
| 3 | `*`, `/`, `%` | `gold * 2`, `n % 4` |
| 4 | `+`, `-` | `gold + 10` |
| 5 | `<`, `<=`, `>`, `>=`, `==`, `!=` | `affection >= 5` |
| 6 | `&&` (logical AND) | `a && b` |
| 7 | `\|\|` (logical OR) | `a \|\| b` |

So `a + b * c >= 10 && !done` parses as `((a + (b * c)) >= 10) && (!done)` — exactly what you'd expect.

### Arithmetic

```
Story.gold + 100
Story.affection - 1
gold * Story.multiplier
Story.total / 2
Story.counter % 4
```

Mixed integer/float behaves intuitively — float wins (`3 + 1.5 = 4.5`).

### Comparison

```
Story.affection >= 5
Story.gold == 0
Chapter.times_visited != 1
Story.player_name == "Alice"
```

`==` and `!=` work on every type, including strings and booleans. `<`, `>`, `<=`, `>=` are for numbers (and lexically for strings).

### Logical

```
Story.has_key && !Story.door_locked
(Story.gold >= 100) || Story.has_voucher
!System.tutorial_done
```

`&&` and `||` short-circuit (the second operand isn't evaluated if the first decides the result).

## Functions

A small set of math helpers:

| Function | Returns | Example |
|----------|---------|---------|
| `abs(x)` | Absolute value | `abs(Story.path_alignment) > 5` |
| `min(a, b)` | Smaller of two | `min(Story.gold, 100)` |
| `max(a, b)` | Larger of two | `max(0, Story.health)` |
| `int(x)` | Convert to integer (truncate) | `int(Story.affection)` |
| `float(x)` | Convert to float | `float(Story.gold) / 100` |

## Assignments

In **Variable Assignments** arrays you write whole statements:

```
Story.affection = Story.affection + 1
Story.gold = max(0, Story.gold - 10)
Story.met_hero = true
Story.path = "thief"
Story.health = min(Story.health + 5, Story.max_health)
```

The left side must be a `Scope.name` reference. The right side is any expression that produces a value of the matching type.

!!! tip
    Multiple assignments on a single line/choice run top-to-bottom in array order. So you can do `Story.gold = Story.gold - 50` *then* `Story.last_purchase_cost = 50` and they both fire.

## Real-world examples

### Gate a line on a stat

> Show this line only when the hero has befriended the rival.

```
Story.rel_rival >= 3
```

### Branch a recap scene by accumulated alignment

In **Conditional Next Scenes**:

| Condition | Target |
|-----------|--------|
| `Story.path_alignment > 3` | `S_HeroEnding` |
| `Story.path_alignment < -3` | `S_VillainEnding` |
| (fallback Next Scene) | `S_NeutralEnding` |

### "Locked, need a key" choice

> Choice visible but greyed out until the player has the key.

| Choice option field | Value |
|--------------------|-------|
| Condition | `Story.has_key == true` |
| bShowWhenUnavailable | true |
| Unavailable Text | `(Locked — need a key)` |

### Compound condition

> Player picked path A, has 5+ affection, and hasn't seen the secret yet.

```
Story.path == "A" && Story.affection >= 5 && !Story.seen_secret
```

### Increment then test

In a choice's Variable Assignments:

```
Story.times_helped = Story.times_helped + 1
```

Then a later line's Condition:

```
Story.times_helped == 3
```

…fires the third time the player picks "help".

## Pitfalls

!!! warning "Strings need double quotes in *expressions*"
    In a **Variable Assignments** entry: `Story.path = "thief"` (quoted). In a string variable's **Default Value** field on the project asset: `thief` (no quotes — the field itself is a string). The two contexts are different.

!!! warning "`=` vs `==`"
    `=` assigns. `==` compares. `Story.gold = 100` *sets* gold to 100. `Story.gold == 100` is a boolean test. Use `==` in Condition fields; `=` only in Variable Assignments.

!!! warning "Missing scope prefix"
    `affection >= 5` fails to resolve. Always write `Story.affection >= 5`. The evaluator will surface the error in the editor log when it can.

!!! warning "Enum values are *string* literals in expressions"
    Compare with quotes: `Story.mood == "angry"`, not `Story.mood == angry`. The runtime stores enums as integers internally but the expression layer matches by name.

## See also

- [Variables and scopes](variables-scopes.md) — declaring the variables you reference.
- [Add your first choice](../getting-started/first-choice.md)
- [Scene asset reference](../reference/scene-asset.md) — Conditional Next Scenes, Entry Condition.

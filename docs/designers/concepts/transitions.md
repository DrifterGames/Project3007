# Transitions

Transitions are the visual handoff between two states — entering a scene, swapping a background mid-scene. VNFramework supports four transition types out of the box, each with a duration and (for some) tuning fields.

## The four types

| Type | What it looks like | Status |
|------|--------------------|--------|
| **None** | Hard cut. No animation. | Stable |
| **Fade** | Fade through a solid color (default black) and back. | Stable |
| **Dissolve** | Crossfade — old and new shown simultaneously, alpha-blending. | Stable |
| **Slide** | New content slides in from a chosen edge. | **Work-in-progress** — currently falls back to Dissolve. |
| **Custom** | Material-driven transition with a `Progress` parameter 0→1. | **Work-in-progress** — not fully wired. |

For shipping work, stick to **None**, **Fade**, and **Dissolve**.

## Where you set them

There are two places transitions appear in the Details panel:

### Scene-entry transition

On a scene asset, **Scene | Visuals → Entry Transition**. This plays when this scene loads from the previous one. Common pattern: most scenes use Dissolve; act-break scenes use a slow Fade through black for emphasis.

### Per-line background change

When a dialogue line's **Background Change** is set, the line also has a **Background Transition** field that controls how that swap animates. Use this for "the camera cuts to…" beats inside a single scene without authoring a separate scene asset.

## The transition fields

When you expand a transition struct in the Details panel, you'll see:

| Field | What it does |
|-------|--------------|
| Type | None / Fade / Dissolve / Slide / Custom. |
| Duration | Seconds the transition takes. `0.5` is a reasonable default. `0` = effectively a hard cut. |
| Fade Color | (Fade only) Color to fade through. Default black. White or scene-tinted color reads as a flashbulb. |
| Slide Direction | (Slide only — WIP) Left / Right / Up / Down. |
| Custom Material | (Custom only — WIP) Material with a `Progress` scalar 0→1. |
| Easing Curve | Optional curve asset to shape the timing (e.g. ease-out instead of linear). |

## Picking the right transition

| Situation | Try |
|-----------|-----|
| Same-room dialogue beats | None or Dissolve at 0.2s. |
| Time skip ("…hours later") | Fade through black, ~1.0s. |
| Dream / flashback open | Fade through white, 0.6–1.0s. |
| Cut to a different POV | Dissolve at 0.3–0.5s. |
| Act break / chapter end | Long Fade, 1.5–2.0s, with audio fade to match. |

!!! tip
    Match transition duration to audio cues where you can. A 1.0s Fade through black with a 1.0s BGM crossfade and a 1.0s ambient crossfade is the cleanest "we're going somewhere new" beat.

## Easing curves

The optional **Easing Curve** field accepts any `UCurveFloat` asset. The X axis is normalized time (0→1), Y is the eased progress. If empty, the transition runs linearly.

Common curves you can build once and reuse:

- **EaseInOut** — slow start and stop, fast middle. Feels cinematic.
- **EaseOut** — fast start, slow stop. Lands softly.

Save these alongside your theme so all transitions share the same feel.

## Pitfalls

!!! warning "Slide and Custom are not fully wired yet"
    The Slide direction and the Custom material fields are exposed in the Details panel but the runtime currently falls back to Dissolve for both. Don't ship a project depending on either until they're marked stable.

!!! warning "Long transitions hide audio handoffs"
    A 2-second Fade plus a 0.2-second BGM crossfade sounds wrong — the music slams in while the screen is still mid-fade. Match the BGM and ambient crossfade durations on the destination scene to the entry transition duration.

!!! note "Per-line transitions don't affect characters"
    A Background Transition swaps the *background only*. Characters on the scene aren't faded by it. Use per-line **Character Changes → Visibility + Transition Duration** to fade them in / out alongside the background swap.

## See also

- [Scene asset reference](../reference/scene-asset.md) — Entry Transition field.
- [Audio](audio.md) — synchronizing BGM and ambient crossfades to scene transitions.
- [Background asset reference](../reference/background-asset.md)

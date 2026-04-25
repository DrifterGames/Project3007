# Audio

VNFramework treats audio as a first-class part of the scene. Music, voice, ambient beds, and one-shot sound effects each have their own channel, their own volume slider in the player's settings, and (for SFX and voice) per-cue tuning. This page is the designer-friendly map: what each channel does, how the framework plumbs them together, and what you can author per scene and per line.

## The four channels

| Channel | What plays here | Authored on |
|---------|------------------|-------------|
| **BGM** | Background music — typically one looping track at a time. | Scene asset (per-scene track). |
| **Voice** | Character voice-over for a single dialogue line. | Dialogue line **Voice Clip**. |
| **Ambient** | Room tone, wind, crowd murmur — distinct from BGM, plays alongside it. | Scene (per-scene bed). Backgrounds also have an ambient slot. |
| **SFX** | One-shot effects: footsteps, doors, glass break, UI chimes. | Per-line **Sound Effects** array. |

Each channel has an independent volume in the player's audio settings (Master, BGM, Voice, Ambient, SFX). The settings UI is fully wired — you don't need to do anything to expose those sliders.

## How the routing works (the short version)

The framework's developer settings (under **Project Settings → Plugins → VNFramework**) point at a set of audio assets:

- **Sound Classes** — one per channel (`SC_VN_Music`, `SC_VN_Voice`, `SC_VN_Ambient`, `SC_VN_SFX`, `SC_VN_UI`). Volume sliders drive these.
- **Sound Mixes** — `SMX_VN_Settings` carries the per-class volume adjusters; `SMX_VN_DuckUnderVoice` is pushed automatically while a voice line is playing.
- **Submixes** — `SUB_VN_SFX` and `SUB_VN_Voice` are reverb attachment points for scene-level reverb.

Defaults ship under `/Game/VNFramework/Audio/` and work out of the box. If you want a custom routing, swap the references in **Project Settings → Plugins → VNFramework** rather than editing the plugin.

!!! tip
    For 95% of projects, you never touch developer settings. They exist for teams who want bespoke mastering chains.

## Background music

### Per-scene track

On any scene asset, **Scene | Audio**:

| Field | What to do |
|-------|------------|
| **Background Music** | Pick a sound asset (USoundWave, USoundCue). Empty = silence. |
| **bChangeBGM** | **Tick this** when you want the scene to apply its BGM setting. If unticked, the previous scene's BGM keeps playing. |
| **BGM Transition Duration** | Seconds to crossfade between the old track and the new. `0` = hard cut, `1.0` is a clean default. |

The "change vs. inherit" toggle is important: in a chapter where ten scenes happen in the same location, you set `bChangeBGM` on the first one and leave it off on the others. The track keeps playing seamlessly.

### Stopping music

Set **Background Music** to `None` and tick **bChangeBGM**. The runtime treats null as "fade out the current track and play silence".

## Voice

### On the line

Each dialogue line has a **Voice Clip** field (a SoundWave). When the line displays, the framework plays the clip on the Voice channel and starts a duration timer. Auto-advance can wait for the clip to finish (see the Auto-advance settings on the project asset).

### Per-character voice tuning

Open the character asset → **Voice** section:

| Field | Default | Use for |
|-------|---------|---------|
| Voice Volume | 1.0 | Per-character base volume (e.g. quiet narrator at 0.7). |
| Voice Class Override | empty | Route this character through a custom Sound Class — useful if you have one character that needs unique mastering (whispery, booming, radio filter). |
| Voice Pitch Multiplier | 1.0 | Scalar applied to all clips for this character. 0.8 = gruffer; 1.2 = lighter. |
| Voice Priority | 5 (0–10) | Higher wins concurrency conflicts. Narrator or unique characters: 7–9. |

### Ducking under voice

Voice ducking ("turn down the music while someone speaks") is on by default. The framework pushes a Sound Mix called `SMX_VN_DuckUnderVoice` while a voice line is playing and pops it when the line finishes. The mix attenuates BGM and Ambient by a configurable amount.

You can:

- **Disable ducking globally** — toggle `bDuckAudioUnderVoice` in the player's audio settings.
- **Override duck depth per scene** — the scene asset has a **Voice Duck Attenuation Db** field (default −6 dB). Set lower (more negative) for louder ducks; set 0 to disable ducking just for this scene.

!!! note
    Ducking is automatic. You don't author "start duck" / "stop duck" on dialogue lines — the audio manager handles it from the Voice channel state.

## Ambient

The ambient channel is for *non-musical* atmosphere that runs alongside BGM: room tone, weather, crowd murmur. Two places to author it.

### Scene-level ambient bed

On the scene asset, **Scene | Audio**:

| Field | What to do |
|-------|------------|
| Ambient Audio | The looping bed for this scene. |
| bChangeAmbient | Tick to apply this scene's bed; untick to inherit the previous scene's. |
| Ambient Transition Duration | Crossfade seconds. |
| Ambient Volume | Per-scene scale 0–1. The Master + Ambient category volumes still apply on top. |

Backgrounds have an **Ambient Sound** field too, but the scene-level one takes precedence when both are set.

### Per-line ambient swap

In a dialogue line, the **Ambient Change** field lets you swap the ambient bed mid-scene without authoring a separate scene asset. Use it when the camera cuts from "kitchen" to "alleyway" inside one scene. Pair with `bStopAmbient` and `Ambient Fade Duration` for fine control.

## SFX (sound effects)

SFX are one-shot effects scheduled per dialogue line. Each line has a **Sound Effects** array; each entry is a Sound Cue with its own delay and tuning.

| Sound Cue field | What it does |
|-----------------|--------------|
| Sound | The audio asset. |
| Delay | Seconds after the line starts before this cue fires. Use small delays (0.2–0.5s) to land the sound on a beat. |
| Volume Multiplier | Per-cue volume (0–2, 1 nominal). |
| Category | Concurrency category — `Generic`, `Impact`, `Loop`, `UI`. Determines how this cue interacts with other cues of the same category playing at once. |
| Attenuation | Optional 2D attenuation settings (screen-position panning, distance LPF). Empty = no attenuation. |

You can stack many cues per line — a swordfight beat can fire a swing whoosh at 0s, an impact at 0.4s, and a stagger grunt at 0.7s, all from the same line.

### SFX categories

Pick the category that matches the *kind* of sound:

| Category | Use for | Behavior |
|----------|---------|----------|
| Generic | Default foley | Many can overlap |
| Impact | Slams, gunshots | Limited stacking |
| Loop | Atmospheric textures that persist | Holds across lines until stopped |
| UI | Button clicks, menu chimes | Never ducked under voice |

### SFX cleanup on line advance

The audio manager **automatically stops every SFX still playing whenever the dialogue advances to the next line**. This is on purpose: looping SFX cues (Category = Loop) have no author-visible stop hook, so the framework clears them on advance. One set of SFX per line, period.

!!! warning
    If you author a long Loop-category cue expecting it to keep playing across multiple lines, it won't. The cue stops on the next line advance. For sustained ambient noise, use the Ambient channel instead — that's what it's for.

## Scene-level reverb

You can attach a reverb effect to the SFX and Voice submixes for the duration of a single scene. Useful for caves, cathedrals, dream sequences.

On the scene asset, **Scene | Audio** (advanced fields):

| Field | What to do |
|-------|------------|
| Reverb Submix | Pick a Sound Submix asset that has the reverb effect chain you want. Empty = no per-scene reverb. |
| Scene Mix Override | Optional Sound Mix pushed for the duration of this scene — lets you author bespoke mastering for dream sequences, underwater, flashbacks. |

The reverb attaches when the scene loads and detaches when the scene ends. The framework also applies a **Reverb Wet Level Multiplier** from the developer settings — a global tuning knob that scales every preset's wet level by the same amount. Tune it once at the project level instead of editing each reverb asset.

## Volume hierarchy

When a sound plays, three volumes multiply:

1. **Master Volume** (player's audio settings).
2. **Channel Volume** — BGM, Voice, Ambient, or SFX (player's audio settings).
3. **Per-cue / per-character / per-line multiplier** — what you author.

So a voice line on a character with `Voice Volume = 0.8`, with the player's Voice slider at 0.5 and Master at 0.7, plays at `0.8 × 0.5 × 0.7 = 0.28` of nominal. This is normal — design assuming the player will adjust their settings.

## Pitfalls

!!! danger "Looping SFX cues stop on line advance"
    The framework stops every active SFX when dialogue advances. Use the Ambient channel for sustained noise, not a long Loop SFX.

!!! warning "BGM not changing? Check bChangeBGM"
    If a scene has a Background Music set but `bChangeBGM` is unchecked, the previous scene's track keeps playing. Same trap on Ambient with `bChangeAmbient`.

!!! warning "Voice ducking too aggressive? Set Voice Duck Attenuation Db on the scene"
    Default scene duck is −6 dB. Set higher (closer to 0) for a softer duck, lower (e.g. −12) for a more dramatic one. `0` disables ducking just for that scene.

!!! warning "Audio settings panel doesn't change anything"
    The framework's Sound Classes / Sound Mixes are wired through **Project Settings → Plugins → VNFramework**. If those slots are empty (you cleared them or wiped the defaults), the player's volume sliders have nothing to drive. Restore the defaults under `/Game/VNFramework/Audio/` or point them at your own equivalents.

## See also

- [Scene asset reference](../reference/scene-asset.md) — every audio field on a scene.
- [Character asset reference](../reference/character-asset.md) — voice tuning per character.
- [Background asset reference](../reference/background-asset.md) — Ambient Sound on backgrounds.
- [Installation](../getting-started/installation.md#vnframework-developer-settings) — where the routing assets are configured.

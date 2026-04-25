# Settings overrides

`UVNFrameworkSettings` is a `UDeveloperSettings` exposed at
*Project Settings → Plugins → VNFramework*. Every audio asset path the
framework consumes at runtime is read from this object — there are no
hardcoded paths in `VNAudioManager`, `VNSettingsTypes::Apply`, or
`VNSceneRenderer::ApplyReverb`.

That means a project can ship its own SoundClass / SoundMix / Submix
tree without touching plugin C++ — just point the settings at your
assets.

Header: `Plugins/VNFramework/Source/VNCore/Public/Settings/VNFrameworkSettings.h`.

## What you can override

| Property | Type | Default points at | What it controls |
|---|---|---|---|
| `MusicSoundClass` | `FSoftObjectPath` (USoundClass) | `/Game/VNFramework/Audio/SC_VN_Music` | Volume bus for `PlayBGM` |
| `VoiceSoundClass` | USoundClass | `SC_VN_Voice` | Volume bus for `PlayVoice` |
| `AmbientSoundClass` | USoundClass | `SC_VN_Ambient` | Volume bus for `PlayAmbient` |
| `SFXSoundClass` | USoundClass | `SC_VN_SFX` | Volume bus for `PlaySFX` |
| `UISoundClass` | USoundClass | `SC_VN_UI` | Volume bus for button hover/click — never ducked |
| `SettingsSoundMix` | USoundMix | `SM_VN_Settings` | Pushed by `FVNAudioSettings::Apply` to carry per-class volume overrides |
| `DuckUnderVoiceSoundMix` | USoundMix | `SM_VN_DuckUnderVoice` | Pushed while voice plays when `bDuckAudioUnderVoice == true` |
| `SFXSubmix` | USoundSubmix | `Submix_VN_SFX` | Target for scene reverb attach |
| `VoiceSubmix` | USoundSubmix | `Submix_VN_Voice` | Target for scene reverb attach (leave empty to skip reverb on voice) |
| `ReverbWetLevelMultiplier` | `float` (0..4) | `1.0` | Global rescale of every scene reverb preset's `WetLevel` at attach time |

## Where the framework reads these

| Setting | Read by | What it does |
|---|---|---|
| `*SoundClass` | `FVNAudioSettings::Apply` | Resolves the soft path, applies the volume to the SoundClass via `SoundClassAdjuster` entries on `SettingsSoundMix` |
| `SettingsSoundMix` | `FVNAudioSettings::Apply` | Pushed via `UGameplayStatics::PushSoundMixModifier` |
| `DuckUnderVoiceSoundMix` | `UVNAudioManager::HandleVoiceStarted` / `OnVoiceFinished` | Push while voice plays, pop on completion |
| `SFXSubmix` / `VoiceSubmix` | `UVNSceneRenderer::ApplyReverb` | Lazy-loaded on first scene reverb attach, then `AddSubmixEffect` / `RemoveSubmixEffect` is called against them on every scene change |
| `ReverbWetLevelMultiplier` | `UVNSceneRenderer::ApplyReverb` | Multiplied against the captured original `WetLevel` before applying to the runtime preset |

## Authoring a per-project audio tree

The shipped `/Game/VNFramework/Audio/` content gives a project a working
default. To replace it:

### 1. Build your own SoundClass tree

Create five `USoundClass` assets — one per channel — with whatever
parent / mix relationships your project needs. Conventional layout:

```
/Game/MyGame/Audio/Classes/
  SC_MyGame_Master       (root)
  SC_MyGame_Music        (parent: Master)
  SC_MyGame_Voice        (parent: Master)
  SC_MyGame_Ambient      (parent: Master)
  SC_MyGame_SFX          (parent: Master)
  SC_MyGame_UI           (parent: Master)
```

The framework only ever drives the `MusicSoundClass`, `VoiceSoundClass`,
`AmbientSoundClass`, `SFXSoundClass`, and `UISoundClass` paths. The
parent chain is up to you.

### 2. Build the SoundMixes

Two are referenced:

- **Settings mix** — passive `USoundMix`, no `SoundClassAdjuster`
  entries pre-authored. `FVNAudioSettings::Apply` rewrites the adjuster
  list each time settings change.
- **Duck-under-voice mix** — passive `USoundMix` with
  `SoundClassAdjuster` entries that attenuate `MusicSoundClass` and
  `AmbientSoundClass` (e.g. -8 dB volume, 1.0 pitch). The framework
  pushes/pops it around voice playback.

### 3. Build the submixes

Two `USoundSubmix` assets that each cue routes through, by `SoundClass`
or by per-asset submix override:

- `Submix_MyGame_SFX` — every SFX/Music/Ambient cue's submix should chain
  here.
- `Submix_MyGame_Voice` — voice cues route here. Leave the settings'
  `VoiceSubmix` empty to skip per-scene reverb on voice (reverb only
  attaches to SFX submix in that case).

### 4. Point the settings at your assets

Open *Project Settings → Plugins → VNFramework* and assign each of your
new assets. The framework picks them up immediately on next play —
no engine restart required (the audio manager re-resolves paths via
`FSoftObjectPath::TryLoad` lazily).

For automated configuration (CI, package scripts), the settings are
config-backed (`Config=Game, defaultconfig`), so the values land in
`Config/DefaultGame.ini`:

```ini
[/Script/VNCore.VNFrameworkSettings]
MusicSoundClass=/Game/MyGame/Audio/Classes/SC_MyGame_Music.SC_MyGame_Music
VoiceSoundClass=/Game/MyGame/Audio/Classes/SC_MyGame_Voice.SC_MyGame_Voice
AmbientSoundClass=/Game/MyGame/Audio/Classes/SC_MyGame_Ambient.SC_MyGame_Ambient
SFXSoundClass=/Game/MyGame/Audio/Classes/SC_MyGame_SFX.SC_MyGame_SFX
UISoundClass=/Game/MyGame/Audio/Classes/SC_MyGame_UI.SC_MyGame_UI
SettingsSoundMix=/Game/MyGame/Audio/Mixes/SM_MyGame_Settings.SM_MyGame_Settings
DuckUnderVoiceSoundMix=/Game/MyGame/Audio/Mixes/SM_MyGame_DuckUnderVoice.SM_MyGame_DuckUnderVoice
SFXSubmix=/Game/MyGame/Audio/Submixes/Submix_MyGame_SFX.Submix_MyGame_SFX
VoiceSubmix=/Game/MyGame/Audio/Submixes/Submix_MyGame_Voice.Submix_MyGame_Voice
ReverbWetLevelMultiplier=1.0
```

## Tuning reverb without per-asset edits

Every scene reverb preset (`USubmixEffectReverbPreset`) ships with an
authored `WetLevel`. At `ApplyReverb` time the framework captures the
original `WetLevel` (only on the very first attach for that preset, into
`OriginalReverbWetLevels`) and writes
`OriginalWetLevel * ReverbWetLevelMultiplier` back to the runtime preset.

Use `ReverbWetLevelMultiplier` to globally dampen or brighten reverb
across every scene without editing each `RVB_*_Preset` asset:

| Multiplier | Effect |
|---|---|
| `0.0` | Dry — disables every scene reverb without un-attaching it |
| `0.5` | Halve every preset's authored wet level |
| `1.0` | Use the authored value (default) |
| `2.0` | Double — caps at the preset's own clamp |

Change at runtime through `GetMutableDefault<UVNFrameworkSettings>()` if
you need a per-platform tweak.

## Programmatic access

```cpp
#include "Settings/VNFrameworkSettings.h"

const UVNFrameworkSettings* Settings = UVNFrameworkSettings::Get();
USoundClass* MusicClass = Cast<USoundClass>(Settings->MusicSoundClass.TryLoad());
const float WetMul = Settings->ReverbWetLevelMultiplier;
```

`Get()` returns the CDO-backed object and is always non-null, so you
don't need a guard.

## See also

- [Architecture: audio routing](../architecture.md#audio-routing) — how
  the SoundClasses, SoundMixes, and Submixes wire together at runtime.
- [VNCore API → UVNFrameworkSettings](../api/vncore.md#uvnframeworksettings)
  — the property table.
- [Designer concept: audio](../../designers/concepts/audio.md) — the
  designer-side overview of channels and ducking.

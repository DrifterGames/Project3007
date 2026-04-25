# VNCore API

`VNCore` is the runtime module that owns story state. It defines all the
DataAsset types content authors edit, the central `UVNSubsystem`, the
expression evaluator, the variable system, the save game classes, and
the developer-settings object that other runtime modules read from.

Module path: `Plugins/VNFramework/Source/VNCore/Public/`.
Header convention: `VN<Name>.h` (no module prefix), with subfolders
`Data/`, `Subsystem/`, `Variable/`, `Expression/`, `Save/`, `Settings/`.

## DataAssets

All VN content lives in `UPrimaryDataAsset` subclasses so they participate
in `UAssetManager`-driven loading and can be cooked individually.

### UVNProjectAsset

`Data/VNProjectAsset.h` — root hub for a single VN. References every
chapter, character, and background, declares story-scoped and
system-scoped variables, and points at the UI theme.

| Property | Type | Notes |
|---|---|---|
| `Title` / `Description` / `Version` | `FText`/`FString` | Project metadata |
| `Chapters` | `TArray<TSoftObjectPtr<UVNChapterAsset>>` | All chapters |
| `StartingChapter` | `TSoftObjectPtr<UVNChapterAsset>` | Where new game starts |
| `Characters` | `TArray<TSoftObjectPtr<UVNCharacterAsset>>` | Project-wide cast |
| `Backgrounds` | `TArray<TSoftObjectPtr<UVNBackgroundAsset>>` | Background pool |
| `StoryVariables` | `TArray<FVNVariableDefinition>` | Persist per playthrough |
| `SystemVariables` | `TArray<FVNVariableDefinition>` | Persist across saves |
| `EnumDefinitions` | `TArray<FVNEnumDefinition>` | Narrative enums (ink LIST etc.) |
| `DefaultTextSpeed` | `float` (cps) | Default chars/sec |
| `AutoAdvanceSettings` | `FVNAutoAdvanceSettings` | Default auto-advance config |
| `SkipSettings` | `FVNSkipSettings` | Default skip config |
| `MaxSaveSlots` | `int32` (1+) | Slot cap |
| `MaxBacklogEntries` | `int32` (10+) | Ring buffer size |
| `Theme` | `TSoftObjectPtr<UVNUITheme>` | Project UI theme |
| `TitleScene` | `TSoftObjectPtr<UVNSceneAsset>` | Optional title backdrop scene |
| `LightWrapSettings` | `FVNLightWrapSettings` | Default character light wrap |

Designer-facing reference:
[Project asset](../../designers/reference/project-asset.md).

### UVNChapterAsset

`Data/VNChapterAsset.h` — groups scenes and provides chapter-scoped
variables that reset on chapter change.

| Property | Type | Notes |
|---|---|---|
| `ChapterID` | `FName` | Lookup key |
| `DisplayName` | `FText` | Player-facing |
| `ChapterNumber` | `int32` | Ordering |
| `Scenes` | `TArray<TSoftObjectPtr<UVNSceneAsset>>` | All scenes in this chapter |
| `StartingScene` | `TSoftObjectPtr<UVNSceneAsset>` | Entry point |
| `ChapterVariables` | `TArray<FVNVariableDefinition>` | Reset on chapter change |
| `UnlockCondition` | `FString` | Expression gate |
| `NextChapter` | `TSoftObjectPtr<UVNChapterAsset>` | Auto-continue target |
| `TitleCard` / `TitleCardDuration` | `UTexture2D` / `float` | Optional intro frame |
| `ThemeOverride` | `TSoftObjectPtr<UVNUITheme>` | Replaces project theme for this chapter |

`GetSceneByID(FName)` resolves a scene reference by ID rather than
asset path.

Designer-facing reference:
[Chapter asset](../../designers/reference/chapter-asset.md).

### UVNSceneAsset

`Data/VNSceneAsset.h` — a single playable scene: visuals, dialogue,
audio, and outbound flow. The largest authoring surface in the system.

#### Visual setup

| Property | Type | Notes |
|---|---|---|
| `SceneID` | `FName` | Save/load key |
| `SceneType` | `EVNSceneType` | `Sprite2D` (default) / `Parallax2D` / `Full3D` (WIP) / `Mixed` (WIP) |
| `Background` | `TSoftObjectPtr<UVNBackgroundAsset>` | Initial background |
| `Characters` | `TArray<FVNCharacterPlacement>` | Initial 2D placements |
| `Config3D` | `FVN3DSceneConfig` | Level sequence + 3D characters (3D scenes only) |
| `bOverrideLightWrap` / `LightWrapOverride` | `bool` / `FVNLightWrapSettings` | Per-scene light-wrap override |
| `EntryTransition` | `FVNTransitionConfig` | Played when entering the scene |

#### Dialogue + flow

| Property | Type | Notes |
|---|---|---|
| `DialogueLines` | `TArray<FVNDialogueLine>` | Lines in playback order |
| `bHasEndChoice` / `EndChoice` | `bool` / `FVNChoicePrompt` | End-of-scene branch |
| `NextScene` | `TSoftObjectPtr<UVNSceneAsset>` | Default continuation |
| `ConditionalNextScenes` | `TArray<FVNConditionalNextScene>` | Evaluated in array order before `NextScene` — first truthy condition wins |
| `EntryCondition` | `FString` | Optional gating expression |

#### Audio

| Property | Type | Notes |
|---|---|---|
| `BackgroundMusic` / `bChangeBGM` / `BGMTransitionDuration` | `USoundBase` / `bool` / `float` | BGM swap on scene entry |
| `AmbientAudio` / `bChangeAmbient` / `AmbientTransitionDuration` / `AmbientVolume` | … | Ambient swap on scene entry |
| `ReverbSubmix` | `TSoftObjectPtr<USoundSubmix>` | Scene reverb attached to VN SFX + Voice submixes |
| `VoiceDuckAttenuationDb` | `float` (-24..0) | Per-scene voice duck override (dB attenuation on Music + Ambient while voice plays) |
| `SceneMixOverride` | `TSoftObjectPtr<USoundMix>` | Optional bespoke SoundMix pushed for the scene's lifetime |

!!! warning "Authored, not yet wired"
    `VoiceDuckAttenuationDb` and `SceneMixOverride` exist on the asset
    and are persisted, but the runtime hooks that consume them are not
    yet implemented. Setting them today is forward-compatible — they'll
    light up when the audio overhaul wires them in.

#### Methods

- `GetDialogueLine(int32)` / `GetDialogueLineByIndex` — bounds-checked.
- `FindLineByID(FGuid)` / `FindLineByGuid` — read-tracking & save lookup.
- `Is3DScene` / `Is2DScene` / `Has2DCharacters` / `Has3DCharacters` /
  `HasLevelSequence` — scene-type queries used by `UVNSceneRenderer`.

Designer-facing reference:
[Scene asset](../../designers/reference/scene-asset.md).

### UVNCharacterAsset

`Data/VNCharacterAsset.h` — a speakable character. Either 2D
sprite-based or 3D skeletal mesh.

| Property | Type | Notes |
|---|---|---|
| `CharacterID` | `FName` | Used by `FVNDialogueLine.SpeakerID` |
| `DisplayName` | `FText` | Default name plate |
| `NameColor` | `FLinearColor` | Speaker name tint |
| `CharacterType` | `EVNCharacterType` | `Sprite2D` / `Skeletal3D` / `Live2D` (future) / `Spine` (future) |
| `DefaultExpression` / `Expressions` | `FName` / `TArray<FVNCharacterExpression>` | 2D pose set |
| `DefaultPosition` / `DefaultScale` | `FVector2D` / `float` | 2D placement defaults |
| `SkeletalMesh` / `AnimBlueprintClass` / `Default3DTransform` | … | 3D rig |
| `DefaultAnimation` / `Animations` | `FName` / `TArray<FVN3DCharacterAnimation>` | 3D pose set |
| `bCastShadows` | `bool` | 3D only |
| `VoiceVolume` | `float` (0..2) | Per-character multiplier applied at voice playback |
| `VoiceClassOverride` | `TSoftObjectPtr<USoundClass>` | Per-character SoundClass override (whispery, booming, etc.) |
| `VoicePitchMultiplier` | `float` (0.5..2.0) | Pitch scalar |
| `VoicePriority` | `int32` (0..10) | Voice concurrency priority |
| `Tags` | `TArray<FName>` | Filtering |

!!! info "Voice class override status"
    `VoiceClassOverride` is authored and saved on the asset. The runtime
    audio path currently routes voice through the global voice
    SoundClass; the per-character override is consumed in a follow-up
    audio overhaul step.

Designer-facing reference:
[Character asset](../../designers/reference/character-asset.md).

### UVNBackgroundAsset

`Data/VNBackgroundAsset.h` — a background source. Type-driven union over
2D image, parallax stack, 3D level reference, or video.

| Property | Type | Notes |
|---|---|---|
| `BackgroundID` | `FName` | Lookup key |
| `BackgroundType` | `EVNBackgroundType` | `Image2D` / `Parallax` / `Scene3D` / `Video` (not yet implemented) |
| `BackgroundTexture` | `TSoftObjectPtr<UTexture2D>` | 2D + parallax |
| `ParallaxLayers` / `ParallaxDepths` | `TArray<…>` | Parallax stack |
| `LevelAsset` / `bUseStreaming` / `StreamingLevelName` | … | 3D scene |
| `CameraLocation` / `CameraRotation` / `CameraFOV` | … | Initial 3D camera (no level sequence) |
| `MediaSource` / `bLoopVideo` | … | Video |
| `AmbientSound` / `AmbientVolume` | `USoundBase` / `float` | Optional ambient bed tied to this background |

Designer-facing reference:
[Background asset](../../designers/reference/background-asset.md).

### UVNUITheme

`Data/VNUITheme.h` — UI styling hub. References three packs (frames,
fonts, icons) plus a small set of colors and layout presets.

| Property | Type | Notes |
|---|---|---|
| `ThemeID` / `DisplayName` | `FName` / `FText` | Identity |
| `FramePack` | `TSoftObjectPtr<UVNFramePack>` | Frame textures |
| `FontPack` | `TSoftObjectPtr<UVNFontPack>` | Per-text-type fonts |
| `IconPack` | `TSoftObjectPtr<UVNIconPack>` | UI icons |
| `AccentColor` / `FrameTint` | `FLinearColor` | Highlights, frame tint |
| `BackgroundColor` / `ButtonBackgroundColor` / `TextColor` / `TextShadowColor` / `NameColor` | … | Fallbacks used only when the corresponding pack is unset |
| `DialoguePosition` / `ChoiceStyle` / `MenuPosition` | enums | Layout presets |
| `ScreenMargin` | `FMargin` | DPI-scaled edge padding |
| `DefaultFadeDuration` / `ButtonHoverSpeed` | `float` | Animation defaults |

Accessors `GetFrameBrush`, `GetFont`, `GetFontScaled`, `GetTextColor`,
`GetIcon`, `GetLayeredFrame` apply the tint and prefer pack values over
fallbacks. `GetDefaultTheme()` returns a CDO with sensible defaults so
the framework works without a configured theme asset.

Designer-facing reference:
[Theme pack](../../designers/reference/theme-pack.md).

### UVNFontPack

`Data/VNFontPack.h` — four `FVNFontConfig` slots (Dialogue, Name, UI,
Caption), each carrying font face, size (inside `FSlateFontInfo`), color,
and shadow. `UVNThemedTextStyle` reads these to populate the
`UCommonTextBlock` style CDO.

Accessors: `GetConfig(EVNTextType)`, `GetFont`, `GetFontScaled`,
`GetTextColor`, `IsShadowEnabled`, `GetShadowOffset`, `GetShadowColor`.

Legacy `*_DEPRECATED` fields are migrated in `PostLoad` and should not
be touched by new code.

### UVNFramePack

`Data/VNFramePack.h` — 9-slice frames + ornaments, with two authoring
modes:

- **Quick setup** (default): one border + one background + optional corner
  ornament, applied uniformly to every frame type. ~15 visible fields.
- **Full mode**: per-frame `FVNLayeredFrame` definitions for Dialogue,
  Name Plate, Choice Panel, Generic Panel, Backlog, Settings, plus shared
  backgrounds, edges, and ornaments.

Auto-generated button states (hover/pressed/disabled) come from
brightness multipliers when `bAutoGenerateButtonStates` is true.

Accessors: `GetLayeredFrame(EVNFrameType)`, `GetLayeredFrameWithDefaults`,
`GetFrame`, `GetChoiceButtonBrushes`, `GetMenuButtonBrushes`,
`GetButtonBrush`, plus the shared-asset getters.

### UVNIconPack

`Data/VNIconPack.h` — flat collection of `TSoftObjectPtr<UTexture2D>`
fields organised by category (Playback, Menu, Navigation, Progress).
Lookup by enum: `GetIcon(EVNIconType)`, `GetIconSoft`, `HasIcon`,
`PreloadAllIcons`.

## Subsystem

### UVNSubsystem

`Subsystem/VNSubsystem.h` — `UGameInstanceSubsystem`. The central
coordinator for everything story-related. Read this header before
modifying runtime behavior.

#### Project / scene loading

- `LoadProject(UVNProjectAsset*)` — initialises variables, loads theme,
  primes preloader, broadcasts `OnProjectChanged` / `OnThemeChanged`,
  resets the session timer.
- `LoadChapter(UVNChapterAsset*)` — switches chapter, resets chapter
  variables, applies chapter theme override.
- `LoadScene(UVNSceneAsset*)` / `LoadSceneAtLine(Scene, int)` — switches
  scene, runs `ProcessSceneVisuals` → `ProcessSceneAudio` →
  `ProcessCurrentLine`. Guards re-entry via `bSceneTransitionInProgress`.
- `GetActiveProject` / `GetCurrentChapter` / `GetCurrentScene`.

#### Dialogue control

- `AdvanceDialogue()` — moves cursor forward; returns false at choice or
  end-of-scene.
- `GetCurrentLine()` / `GetCurrentDialogueLine` — current line.
- `GetCurrentLineIndex` / `IsAtChoice` / `SelectChoice(int32)`.
- `GetEffectiveNextScene()` — resolves `ConditionalNextScenes` then falls
  back to `NextScene`. Synchronously loads target if preload hasn't
  landed.

#### Playback mode

- `Get/SetPlaybackMode(EVNPlaybackMode)` — `Normal` / `Skip` / `Auto`.
  Setter broadcasts `OnPlaybackModeChanged`.

#### Variables (4-tier scoping)

`Get/SetBool/Int/Float/String(EVNVariableScope, FString Name, …)`. The
scope enum is `Scene` < `Chapter` < `Story` < `System`. Internally these
delegate to the matching `FVNVariable` container, with `SystemVariables`
also written through to `UVNSystemSaveGame`.

`EvaluateCondition(FString, bool& OutResult, FString& OutError)` runs the
expression evaluator with the subsystem-bound resolver.
`ResolveTypedVariable` is used by the evaluator to fetch typed values
without heuristic guessing.

#### Save / load

- `SaveToSlot(FString)` / `LoadFromSlot(FString)` / `AutoSave` /
  `QuickSave` / `QuickLoad`.
- `PopulateSave(UVNUserSaveGame*)` — fills a pre-existing save object so
  widgets can pre-populate `SaveName` / thumbnail before commit.
- `SaveSystemData()` — flushes `UVNSystemSaveGame`.
- `GetSystemSave()` / `GetTotalPlaytimeSeconds()`.
- Static slot names: `AutoSaveSlotName`, `QuickSaveSlotName`.

#### Audio control

`Get/SetMasterVolume`, `Get/SetBGMVolume`, `Get/SetVoiceVolume`,
`Get/SetSFXVolume`, `Get/SetAmbientVolume`. Setters write through to
`UVNSystemSaveGame` and re-apply via `FVNAudioSettings`.

`PlaySFX(USoundBase*, float)` broadcasts `OnSFXRequested`. Voice playback
state is reported via `IsVoicePlaying()` / `NotifyVoiceComplete()`.

#### Read tracking

- `IsCurrentLineRead` / `IsLineRead(FGuid)` / `GetReadLineCount` /
  `GetReadPercentage(int32 TotalLines)` — proxied to
  `UVNSystemSaveGame::ReadLineIDs`.

#### Visual state queries

- `GetCurrentCharacters()` — current placements (mid-scene).
- `GetCharacterPlacement(FName, FVNCharacterPlacement&)` /
  `IsCharacterPresent(FName)`.
- `GetCurrentBackground()`.
- `TriggerScreenEffect(FName, float)` — fires `OnScreenEffect`.
- `NotifyTransitionComplete()` — UI layer signals the renderer that a
  transition has finished.

#### UI theme

- `GetTheme()` — returns active theme (override > project > default CDO).
- `SetThemeOverride(UVNUITheme*)` — runtime override; `nullptr` reverts.

#### Multicast delegates (subscribe in your widget / system)

| Delegate | Signature | When |
|---|---|---|
| `OnProjectChanged` | `(UVNProjectAsset*)` | After `LoadProject` |
| `OnChapterChanged` | `(UVNChapterAsset*)` | After `LoadChapter` |
| `OnSceneChanged` | `(UVNSceneAsset*)` | After `LoadScene` (post visuals/audio) |
| `OnDialogueAdvanced` | `(const FVNDialogueLine&, int32)` | Each line tick |
| `OnDialogueComplete` | `()` | End of `DialogueLines[]` |
| `OnChoicePresented` | `(const FVNChoicePrompt&)` | End-of-scene with `bHasEndChoice` |
| `OnChoiceSelected` | `(int32, const FVNChoiceOption&)` | After `SelectChoice` |
| `OnPlaybackModeChanged` | `(EVNPlaybackMode)` | `SetPlaybackMode` |
| `OnThemeChanged` | `(UVNUITheme*)` | Project load, chapter override, runtime override |
| `OnBackgroundChanged` | `(UVNBackgroundAsset*, const FVNTransitionConfig&)` | Scene entry + line-level `BackgroundChange` |
| `OnCharacterAdded` | `(const FVNCharacterPlacement&)` | Scene entry per placement |
| `OnCharacterRemoved` | `(FName)` | Per character removal |
| `OnCharacterChanged` | `(const FVNCharacterChange&)` | Per-line `CharacterChanges` |
| `OnCharactersCleared` | `()` | Scene entry (before adds) |
| `OnTransitionStarted` | `(const FVNTransitionConfig&)` | Background transition begins |
| `OnTransitionComplete` | `()` | UI layer must call `NotifyTransitionComplete` |
| `OnScreenEffect` | `(FName, float)` | Author-driven effects |
| `OnBGMChanged` | `(USoundBase*, float CrossfadeDuration, bool bShouldPlay)` | Scene entry with `bChangeBGM` |
| `OnVoiceStarted` | `(USoundBase*, float CharacterVolume)` | Per-line voice |
| `OnVoiceComplete` | `()` | Audio manager raises via `NotifyVoiceComplete` |
| `OnAmbientChanged` | `(USoundBase*, float Volume, float FadeDuration)` | Scene/line ambient swap |
| `OnSFXRequested` | `(USoundBase*, float Volume)` | Per `FVNSoundCue` |
| `OnPreloadWaitStarted` / `OnPreloadWaitFinished` | `()` | Held transition during async load |

## Settings

### UVNFrameworkSettings

`Settings/VNFrameworkSettings.h` — `UDeveloperSettings`, exposed in
*Project Settings → Plugins → VNFramework*. The single source of truth
for runtime audio asset paths.

| Property | Type | Default | Notes |
|---|---|---|---|
| `MusicSoundClass` | `FSoftObjectPath` (USoundClass) | `/Game/VNFramework/Audio/SC_VN_Music` | BGM + volume |
| `VoiceSoundClass` | `FSoftObjectPath` (USoundClass) | `SC_VN_Voice` | Voice + volume |
| `AmbientSoundClass` | `FSoftObjectPath` (USoundClass) | `SC_VN_Ambient` | Ambient + volume |
| `SFXSoundClass` | `FSoftObjectPath` (USoundClass) | `SC_VN_SFX` | SFX + volume |
| `UISoundClass` | `FSoftObjectPath` (USoundClass) | `SC_VN_UI` | Button cues — never ducked |
| `SettingsSoundMix` | `FSoftObjectPath` (USoundMix) | `SM_VN_Settings` | Pushed by `FVNAudioSettings::Apply` |
| `DuckUnderVoiceSoundMix` | `FSoftObjectPath` (USoundMix) | `SM_VN_DuckUnderVoice` | Pushed during voice when `bDuckAudioUnderVoice` |
| `SFXSubmix` | `FSoftObjectPath` (USoundSubmix) | `Submix_VN_SFX` | Scene reverb attach target |
| `VoiceSubmix` | `FSoftObjectPath` (USoundSubmix) | `Submix_VN_Voice` | Scene reverb attach target |
| `ReverbWetLevelMultiplier` | `float` (0..4) | `1.0` | Global rescale of every scene reverb preset's `WetLevel` |

`UVNFrameworkSettings::Get()` returns the CDO-backed object; it is always
non-null. See [Settings overrides](../extending/settings-overrides.md)
for the override pattern.

## Save

### UVNBaseSaveGame

`Save/VNBaseSaveGame.h` — abstract `USaveGame` base. Carries `CreatedAt`,
`UpdatedAt`, `SaveVersion` (default 1, used for migration). Subclasses
call `UpdateTimestamps()` before commit.

### UVNUserSaveGame

`Save/VNUserSaveGame.h` — per-slot save. One file per playthrough.

| Property | Type | Notes |
|---|---|---|
| `Progress` | `FVNSaveProgress` | Chapter path, scene path, line index, save time, optional thumbnail |
| `StoryVariables` | `FVNVariable` | Persisted |
| `ChapterVariables` | `FVNVariable` | Persisted |
| `SceneVariables` | `FVNVariable` | Transient (UPROPERTY but not `SaveGame`) |
| `Backlog` | `TArray<FVNBacklogEntry>` | Recent dialogue history |
| `SaveName` | `FString` | User-provided slot label |
| `TotalPlaytimeSeconds` | `float` | Cumulative |
| `SlotName` | `FString` (Transient) | Not serialised |

API: `IsValidSlotName`, `GetOrCreate`, `Prepare`, `TryCommit`, `Apply`,
`DoesExist`, `TryDelete`, `GetAllSlotNames`.

### UVNSystemSaveGame

`Save/VNSystemSaveGame.h` — global save. One file shared across all
playthroughs. Stores read tracking, settings, and system variables.

| Property | Type | Notes |
|---|---|---|
| `ReadLineIDs` | `TSet<FGuid>` | Read tracking for skip-unread |
| `CompletedScenes` | `TSet<FName>` | Scene completion |
| `UnlockedEndings` | `TSet<FName>` | Unlocked endings |
| `SystemVariables` | `FVNVariable` | Persist across saves |
| `TextSpeed` | `float` (cps) | Text speed setting |
| `AutoAdvanceSettings` / `SkipSettings` | structs | Auto/skip preferences |
| `MasterVolume` / `BGMVolume` / `SFXVolume` / `VoiceVolume` / `AmbientVolume` | `float` (0..1) | Volume preferences |
| `bFullscreen` | `bool` | Window mode |

API: `GetOrCreate`, `TryCommit`, plus read-tracking helpers
(`MarkLineAsRead`, `IsLineRead`, `MarkSceneCompleted`,
`IsSceneCompleted`, `MarkEndingUnlocked`, `IsEndingUnlocked`,
`GetReadPercentage`).

For adding new persisted state, see
[Save system extensions](../extending/save-system.md).

## Expression evaluator

### FVNExpressionEvaluator

`Expression/VNExpressionEvaluator.h` — recursive-descent evaluator used
by `EntryCondition`, choice `Condition`, line `Condition`, and
`ConditionalNextScenes`.

Operators (lowest → highest precedence):

1. `||`
2. `&&`
3. `==`, `!=`, `<`, `>`, `<=`, `>=`
4. `+`, `-` (binary)
5. `*`, `/`, `%`
6. unary `-`, `!`
7. literals, variables, function calls, parentheses

Built-in functions: `abs`, `min`, `max`, `int`, `float`.
Variable refs: `Story.affection`, `System.seen_ending`, `Scope.Name`.

Public API:

```cpp
bool TryEvaluate(const FString& Expression,
                 const IVNVariableResolver* Resolver,
                 FVNExpressionValue& OutResult, FString& OutError) const;

bool TryEvaluateCondition(const FString& Expression,
                          const IVNVariableResolver* Resolver,
                          bool& OutResult, FString& OutError) const;

bool TryValidate(const FString& Expression, FString& OutError) const;
```

`IVNVariableResolver` is the contract for variable lookup. The subsystem
implements `FVNSubsystemVariableResolver`; for unit tests there's
`FVNSimpleVariableResolver`. Designer reference:
[Expressions](../../designers/concepts/expressions.md).

### Variable types

`Variable/VNVariableTypes.h` defines:

- `EVNVariableScope { Scene, Chapter, Story, System }`.
- `EVNVariableType { Bool, Int, Float, String, Enum }`.
- `FVNVariableDefinition` — `Name`, `Type`, `DefaultValue` (string-encoded),
  `bIsReadOnly`, `Description`, `EnumTypeName` (for `Enum`).
- `FVNEnumDefinition` — `EnumName`, `Values[]`, optional
  `ExplicitValues` map; provides `GetValueFor`, `GetNameFor`,
  `IsValidValue`.
- `FVNVariableDescriptor` — runtime-display projection used by the
  variable inspector / SVNVariableInsertField.

## Asset preloader

### UVNAssetPreloader

`Subsystem/VNAssetPreloader.h` — `UObject` owned by `UVNSubsystem`,
backed by a private `FStreamableManager`.

Public surface:

- `Initialize` / `Shutdown`.
- `PreloadThemeAssets(UVNUITheme*)` / `ReleaseThemeAssets`.
- `PreloadScene(UVNSceneAsset*, FOnVNPreloadComplete)` — two-phase
  (DataAssets, then nested resources).
- `IsScenePreloaded(UVNSceneAsset*)` — true only when both phases done.
- `PromoteNextToCurrent()` — called when transitioning to a preloaded
  scene.
- `PreloadVoiceLines(Scene, FromLine, Count=3)` — voice clips only
  (legacy, narrower scope).
- `PreloadUpcomingLineAssets(Scene, FromLine, Count=6)` — full
  per-line window: voice + per-line `BackgroundChange` + nested
  textures + `AmbientChange` + every `FVNSoundCue`. Replaces
  `PreloadVoiceLines` as the primary per-line entry point.
- `PreloadChoiceTargets(const FVNChoicePrompt&)` /
  `ReleaseChoicePreloads()`.
- `ReleaseAllHandles` / `CancelNextScenePreload`.

Internal handles: current/next scene primary + secondary, voice window,
per-line upcoming primary + secondary, choice targets, theme packs,
theme textures.

!!! tip
    The preloader is the reason `LoadSynchronous` calls inside
    `UVNSubsystem` rarely actually block — assets are already resident.
    If you add new soft refs to dialogue lines or scene assets, extend
    `CollectUpcomingLineAssetPaths` / `CollectUpcomingLineSecondaryPaths`
    so the rolling window covers them.

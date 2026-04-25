# VNUI API

`VNUI` is the runtime presentation module: widgets, scene rendering,
audio playback, transitions, input, and the GameMode that wires
everything to `UVNSubsystem`. All widgets are `UCommonActivatableWidget`
subclasses (CommonUI), and styled containers extend `UVNStyledFrame`.

Module path: `Plugins/VNFramework/Source/VNUI/Public/`.
Subfolders: `Widgets/`, `Elements/`, `Layout/`, `UI/`, `Rendering/`,
`Audio/`, `Playback/`, `Input/`, `Game/`, `Settings/`, `Styles/`.

## Widgets

### UVNBaseWidget

`Widgets/VNBaseWidget.h` — abstract base for VN widgets that don't need
DPI-aware layout. Inherits `UCommonActivatableWidget`. Holds a
`UVNSubsystem` reference and exposes Blueprint-implementable lifecycle
events.

| Method | Purpose |
|---|---|
| `InitializeWidget(UVNSubsystem*)` | Bind subsystem (called by GameMode/UIManager) |
| `GetVNSubsystem()` | Accessor |
| `NativeOnActivated` / `NativeOnDeactivated` | CommonUI lifecycle |
| `NativeOnHandleBackAction` | Default deactivates; override for custom back behavior |
| `NativeGetDesiredFocusTarget` | Initial focus for gamepad/keyboard |

Blueprint events: `OnWidgetInitialized`, `OnWidgetActivated`,
`OnWidgetDeactivated`, `OnHandleBackAction`.

For DPI-aware widgets, prefer `UVNLayoutContainer` / `UVNStyledFrame`.

### UVNDialogueWidget

`Widgets/VNDialogueWidget.h` — extends `UVNStyledFrame` (so it carries
the layered frame + DPI scale + theme hookup). Renders the dialogue box,
name plate, and typewriter text.

Key API:

- `DisplayLine(const FVNDialogueLine&)` — bind to
  `UVNSubsystem::OnDialogueAdvanced`.
- `ClearDialogue()`.
- `StartTypewriter(const FText&)` / `SkipToEnd()` /
  `PauseTypewriter()` / `ResumeTypewriter()`.

`InitializeContainer` overrides `UVNStyledFrame` to also bind the
subsystem-level dialogue events. Set `FrameType = Dialogue` so the
correct `FVNLayeredFrame` is pulled from `FramePack`.

### UVNChoiceWidget

`Widgets/VNChoiceWidget.h` — extends `UVNStyledFrame`. Spawns choice
buttons dynamically from a `FVNChoicePrompt`, supports vertical /
horizontal / grid layouts (driven by `UVNUITheme::ChoiceStyle`).

Display state per option uses `FVNChoiceDisplayInfo` (Index,
DisplayText, bIsAvailable, bShowWhenUnavailable, UnavailableText).

### UVNBacklogWidget

`Widgets/VNBacklogWidget.h` — extends `UVNStyledFrame`. Scrollable
history view backed by `UVNSubsystem::GetBacklog()`. Wraps each
`FVNBacklogEntry` in `FVNBacklogDisplayEntry` (adds index, voice replay
flag, read flag).

### UVNSettingsWidget

`Widgets/VNSettingsWidget.h` — extends `UVNStyledFrame`. Tabbed settings
panel (`EVNSettingsTab { Audio, Video, Gameplay }`) backed by
`UVNSettingsController` for pending-change/preview/apply/revert
semantics.

API: `SetActiveTab` / `GetActiveTab` / `NextTab` / `PreviousTab`,
`GetSettingsController`, plus convenience wrappers for each setting.

### UVNSaveLoadWidget

`Widgets/VNSaveLoadWidget.h` — extends `UVNStyledFrame`. Save / load
slot grid; uses `UVNSaveSlotWidget` per slot.

### UVNSaveSlotWidget

`Widgets/VNSaveSlotWidget.h` — `UUserWidget`. One save slot's display
(thumbnail, save name, timestamp, playtime).

### UVNQuickMenuWidget

`Widgets/VNQuickMenuWidget.h` — extends `UVNStyledFrame`. In-game pause /
quick menu pushed via `UVNUIManagerSubsystem::ShowQuickMenu`.

### UVNMainMenuWidget

`Widgets/VNMainMenuWidget.h` — extends `UVNStyledFrame`. Title menu
shown alongside `TitleScene`; entry point for new game / continue / save
slots.

### UVNCharacterWidget

`Widgets/VNCharacterWidget.h` — `UUserWidget`. Single 2D character
sprite host. Renders the texture from a `FVNCharacterPlacement` /
`FVNCharacterChange`, applies optional flip and light-wrap material.
Spawned by `UVNCharacterLayerWidget`.

### UVNCharacterLayerWidget

`Widgets/VNCharacterLayerWidget.h` — `UUserWidget`. Hosts the per-character
`UVNCharacterWidget` instances on a canvas, sorts them by `Layer`,
animates position / scale / fade based on
`FVNCharacterChange.TransitionDuration`. Bound to the subsystem's
character-change delegates by `UVNSceneRenderer`.

### UVNBackgroundWidget

`Widgets/VNBackgroundWidget.h` — `UUserWidget`. Renders the current
background; participates in background crossfades driven by
`OnBackgroundChanged` + `FVNTransitionConfig`. Pushes its current
texture into `UVNCharacterLayerWidget` so character light wrap can
sample the actual background pixels.

## Styled elements

These are reusable building blocks for VN-themed widgets. They live in
`Plugins/VNFramework/Source/VNUI/Public/Elements/`.

### UVNStyledButton

`Elements/VNStyledButton.h` — themed `UCommonButtonBase` with automatic
state management.

CDO-level (`EditDefaultsOnly`):

- `ButtonType : EVNButtonType` (Choice / Menu / Icon)
- `ClickSound`, `HoverSound`, `SoundVolume`, `bPlaySounds`

Per-instance (`EditAnywhere`):

- `ButtonLabel : FText` — text override (the field is named `ButtonLabel`,
  not `ButtonText`, to avoid colliding with CommonUI template BPs that
  bind a `ButtonText` widget by name)
- `TintOverride`, `bUseTintOverride`

API: `ApplyTheme(UVNUITheme*, float DPIScale)`,
`SetButtonBrushes(const FVNButtonStateBrushes&)`, `SetText`, `GetText`,
`SetTextColor`, `SetHoveredTextColor`, `SetButtonEnabled`,
`UpdateVisualState`. CommonUI events
(`NativeOnHovered`/`Pressed`/`Released`/`Clicked`) are overridden to
play hover/click sounds and update the brush.

Bind-widget-optional slots: `ButtonBackground` (`UImage`),
`ButtonTextBlock` (`UTextBlock`).

### UVNStyledFrame

`Elements/VNStyledFrame.h` — extends `UVNLayoutContainer`. Layered frame
container: background → border → ornaments → content slot.

Configuration:

- `FrameType : EVNFrameType` — picks which `FVNLayeredFrame` to pull from
  the theme's `FramePack`.
- `TintOverride` / `bUseTintOverride`.
- `PaddingOverride` / `bUsePaddingOverride` — DPI-scaled.
- `BackgroundOpacityOverride` (-1 = use frame default).
- `bShowCornerOrnaments` / `bShowEdgeOrnaments`.

API: `ApplyTheme()` (override), `ApplyThemeWithScale`, `SetFrameBrush`,
`SetContentPadding`, `SetTint`, `ApplyLayeredFrame`, `ApplyFramePack`,
`SetBackgroundBrush`, `SetBackgroundOpacity`, `GetContentSlot`,
`GetCurrentBrush`, `HasValidBrush`, `SetFrameVisible`, `IsFrameVisible`.

Bind-widget-optional slots: `BackgroundImage`, `FrameImage`, four corner
ornaments, four tiling edges, four centered bars, `ContentSlot`
(`UNamedSlot`).

### UVNStyledText

`Elements/VNStyledText.h` — `UUserWidget`. Themed text label that pulls
font + color + shadow from `UVNFontPack` based on `EVNTextType`.

### UVNStyledSlider

`Elements/VNStyledSlider.h` — `UUserWidget`. Themed slider used by the
settings panel.

### UVNStyledCheckBox

`Elements/VNStyledCheckBox.h` — themed `UCommonButtonBase` configured as
a checkbox.

### UVNThemedButtonStyle

`Styles/VNThemedButtonStyle.h` — `UCommonButtonStyle` whose per-state
brushes are sourced from the active theme's FramePack
(`FVNButtonStateBrushes`).

`UCommonButtonStyle`'s getters read UPROPERTY fields directly (not
virtual), so this class mutates the CDO fields on theme refresh. Designers
author one BP subclass per category (e.g. `BPStyle_VNChoiceButton`,
`BPStyle_VNMenuButton`) and assign to `UCommonButtonBase::Style`.

API: `RefreshFromTheme(const UVNUITheme*)`, static
`RefreshAllFromTheme`, static `FindAnyTheme`, `RefreshFromDefaultTheme`
(editor-callable).

### UVNThemedTextStyle

`Styles/VNThemedTextStyle.h` — `UCommonTextStyle` whose font / color /
shadow come from the active theme's FontPack via `EVNTextType`. Mirrors
`UVNThemedButtonStyle` structure (subclass per text category, refresh
from theme).

## Layout

### UVNLayoutContainer

`Layout/VNLayoutContainer.h` — abstract `UCommonActivatableWidget` with
DPI-aware scaling, safe-zone handling, and theme integration.

Constants: `BaseDesignWidth = 1920`, `BaseDesignHeight = 1080`. All
pixel-based measurements should be multiplied by `GetDPIScale()`.

API:

- `InitializeContainer(UVNSubsystem*)` (virtual) — fetch subsystem,
  theme, DPI.
- `GetDPIScale` / `ScaleByDPI` / `ScaleByDPI2D` / `ScaleMarginByDPI`.
- `GetViewportSize`, `GetSafeZonePadding`, `GetUsableAreaSize`.
- `GetTheme` / `ApplyTheme` (virtual) / `RefreshLayout` (virtual).
- `GetScreenMargin` / `GetDefaultFadeDuration` (cached from theme).

CommonUI overrides: `NativeConstruct`, `NativeDestruct`,
`NativeOnActivated`, `NativeOnDeactivated`, `NativeOnHandleBackAction`,
`NativeGetDesiredFocusTarget`, `GetDesiredInputConfig` (returns Mode=All
+ NoCapture so dialogue clicks reach the PlayerController and the cursor
stays visible).

Blueprint events: `OnContainerInitialized`, `OnDPIScaleChanged`,
`OnSafeZoneChanged`, `OnThemeApplied`, `OnContainerActivated`,
`OnContainerDeactivated`, `OnHandleBack`.

## Rendering

### UVNSceneRenderer

`Rendering/VNSceneRenderer.h` — `UObject`. Coordinates visual rendering
on a canvas panel: spawns `UVNBackgroundWidget`, `UVNCharacterLayerWidget`,
`UVNTransitionPlayer`, and (for 3D scenes) `UVN3DSceneController`.
Subscribes to the subsystem's visual delegates.

API:

- `Initialize(UCanvasPanel*, UVNSubsystem*, UObject* WorldContext)` /
  `Shutdown`.
- `GetBackgroundWidget` / `GetCharacterLayerWidget` /
  `GetTransitionPlayer` / `Get3DSceneController`.
- `GetCurrentSceneType` / `Is3DMode` / `Set2DOverlayVisible`.

Configurable Blueprint classes: `BackgroundWidgetClass`,
`CharacterLayerWidgetClass`, `TransitionPlayerClass`. Set these on the
GameMode-driven instance before `Initialize` to swap WBP defaults.

Internal: `ApplyReverb(TSoftObjectPtr<USoundSubmix>)` walks the
framework's SFX + Voice submixes and attaches the scene reverb preset
(scaled by `UVNFrameworkSettings::ReverbWetLevelMultiplier`).
`OriginalReverbWetLevels` tracks each preset's authored wet level so
the multiplier always rescales against that, never against the
last-applied value.

### UVN3DSceneController

`Rendering/VN3DSceneController.h` — `UObject`. Manages 3D scene
elements: spawns characters from `FVN3DCharacterPlacement`, plays
`ULevelSequence` for camera animation, broadcasts sequence + character
events.

`FVN3DCharacterInstance` tracks each spawned character (Actor, mesh
component, current animation).

### UVNCameraController

`Rendering/VNCameraController.h` — `UObject`. 2D-canvas "camera"
effects: screen shake (`FVNShakeConfig`), parallax scrolling
(`FVNParallaxConfig`), pan-to.

API: `Shake`, `QuickShake`, `StopShake`, `EnableParallax`,
`DisableParallax`, `SetParallaxTarget`, plus pan/lerp helpers.

### UVNTransitionPlayer

`Rendering/VNTransitionPlayer.h` — `UUserWidget`. Plays scene/background
transitions.

API:

- `PlayTransition(const FVNTransitionConfig&, TFunction<void()> OnComplete)`
- `PlayTransitionBP(const FVNTransitionConfig&)` (Blueprint variant)
- `SkipTransition()` — jump to end.
- `SetHoldAtPeak(bool)` / `ReleaseHold()` — held mid-transition during
  async preload waits.
- `IsPlaying` / `GetProgress`.

Internally dispatches on `EVNTransitionType`:

| Type | Status | Implementation |
|---|---|---|
| `None` | Stable | Instant |
| `Fade` | Stable | `ApplyFadeTransition` (overlay alpha) |
| `Dissolve` | Stable | `ApplyDissolveTransition` (overlay alpha, same path as Fade) |
| `Slide` | WIP | Falls back to Dissolve at runtime |
| `Custom` | WIP | `ApplyCustomTransition` via dynamic material with `Progress` parameter |

For adding a new transition type, see
[Custom transitions](../extending/custom-transitions.md).

## Audio

### UVNAudioManager

`Audio/VNAudioManager.h` — `UObject`. Owns the BGM, voice, ambient, and
SFX `UAudioComponent`s. Subscribes to the subsystem's audio delegates
and to `OnDialogueAdvanced` (for SFX cleanup).

Public API:

- `Initialize(UVNSubsystem*, UObject* WorldContext)` / `Shutdown` /
  `IsInitialized`.
- BGM: `PlayBGM(USoundBase*, float CrossfadeDuration=1)`,
  `StopBGM(float)`, `SetBGMPaused`, `IsBGMPlaying`.
- Voice: `PlayVoice(USoundBase*, float VolumeMultiplier=1)`,
  `StopVoice`, `IsVoicePlaying`, `GetVoiceDuration`,
  `GetVoiceRemainingTime`.
- Ambient: `PlayAmbient(USoundBase*, float Volume=1, float FadeDuration=1)`,
  `StopAmbient(float)`.
- SFX: `PlaySFX(USoundBase*, float VolumeMultiplier=1)`, `StopAllSFX`.
- Volume: `UpdateVolumes` — re-applies computed volumes from
  `UVNSystemSaveGame` to every active component.

Crossfade: BGM and Ambient use paired components (`BGMComponent` +
`BGMCrossfadeComponent`, `AmbientComponent` + `AmbientCrossfadeComponent`).
`CrossfadeTimerHandle` ticks the linear blend over `CrossfadeDuration`
seconds; `OnBGMCrossfadeFinished` / `OnAmbientCrossfadeFinished` swap
roles afterward.

SFX tracking: `ActiveSFXComponents` holds every component spawned by
`PlaySFX` (because UE's `UGameplayStatics::PlaySound2D` is fire-and-forget
with no stop hook). `HandleDialogueAdvanced` calls `StopAllSFX` so
looping cues authored under `EVNSfxCategory::Loop` die at line advance.

!!! warning "Authored, not yet wired"
    `FVNSoundCue.Category` and `FVNSoundCue.Attenuation` are persisted on
    the asset but the runtime `PlaySFX` path doesn't yet route on them.
    Authoring them today is forward-compatible — the audio overhaul will
    consume them in a later step.

## Playback

### UVNPlaybackController

`Playback/VNPlaybackController.h` — `UObject`. Owns timing for skip and
auto modes. Watches typewriter completion, voice completion, and ticks
the auto-advance timer.

API:

- `Initialize(UVNSubsystem*, UVNDialogueWidget*, UObject* WorldContext)` /
  `Shutdown` / `IsInitialized`.
- Mode control: `Start/Stop/ToggleSkipMode`, `Start/Stop/ToggleAutoMode`,
  `ReturnToNormalMode`.
- Queries: `IsSkipping`, `IsAutoMode`, `IsWaitingForAutoAdvance`,
  `GetAutoAdvanceTimeRemaining`, `GetAutoAdvanceProgress`.
- Input: `HandleAdvanceInput()` — call from `UVNInputHandler::OnAdvance`.
  Returns true when the advance was handled.

Delegates: `OnAutoAdvanceReady`, `OnSkipStateChanged`.

Internal handlers for `OnDialogueAdvanced`, `OnDialogueComplete`,
`OnVoiceComplete`, `OnPlaybackModeChanged`, `OnChoicePresented`.

### UVNInputHandler

`Input/VNInputHandler.h` — `UObject`, Blueprintable. Enhanced Input
binding glue.

Configurable input actions:

- `AdvanceAction` — click / space / enter.
- `SkipAction` — hold ctrl.
- `AutoAction` — toggle auto mode.
- `MenuAction` — escape.
- `BacklogAction` — scroll up / page up.
- `QuickSaveAction` / `QuickLoadAction`.
- `VNMappingContext` + `MappingContextPriority`.

Settings:

- `bUseCommonUIRouting` — when true, menu/backlog actions route through
  `UVNUIManagerSubsystem` (push via the layout); when false they only
  broadcast events.
- `bAutoAdvanceOnInput` / `bAutoSkipOnInput` / `bAutoToggleAutoMode`.

Delegates: `OnAdvance`, `OnSkipStarted`, `OnSkipEnded`, `OnAutoToggled`,
`OnMenu`, `OnBacklog`, `OnQuickSave`, `OnQuickLoad`.

`BindToController` / `UnbindFromController` register handles with the
EnhancedInputComponent. `SetVNSubsystem`, `SetPlaybackController`, and
`SetUIManager` link the wiring.

### UVNSettingsController

`Settings/VNSettingsController.h` — `UObject`. Backs the settings widget
with pending-change semantics.

API:

- `Initialize` — snapshot current settings into `OriginalSettings` and
  `PendingSettings`.
- `HasPendingChanges` / `ApplyChanges` / `RevertChanges`.
- Audio: `Get/SetAudioSettings`, `SetMasterVolume`, `SetBGMVolume`,
  `SetSFXVolume`, `SetVoiceVolume`, `SetAmbientVolume`,
  `PreviewAudioSettings` (live preview without commit).
- Video: `Get/SetVideoSettings`, `SetWindowMode`, `SetResolution`,
  `SetVSyncEnabled`, `SetFrameRateLimit`, `GetAvailableResolutions`.
- Gameplay: `Get/SetGameplaySettings`, `SetTextSpeed`,
  `SetAutoAdvanceDefault`, `SetAutoAdvanceDelay`, `SetSkipUnreadText`,
  `SetStopSkipAtChoices`.

Delegates: `OnAudioSettingsPreviewed`, `OnSettingsApplied`,
`OnSettingsReverted`.

## Game

### AVNGameModeBase

`Game/VNGameModeBase.h` — `AGameModeBase`. The conventional bootstrap
for a VN level. Owns the runtime references and wires them up.

Editable defaults:

- `DefaultVNProject` — project to load on `BeginPlay`.
- `bUseCommonUIMode` — pick widget pipeline.
- `LayoutWidgetClass` (CommonUI) / `HUDWidgetClass` (legacy).
- `DialogueWidgetClass`, `ChoiceWidgetClass`, `BackgroundWidgetClass`,
  `CharacterLayerWidgetClass`.
- `InputHandlerClass`.
- `TitleScene`, `TitleMenuWidgetClass` (legacy) /
  `MainMenuWidgetClass` (CommonUI), `bShowTitleOnStartup`.
- `SettingsWidgetClass`, `SaveLoadWidgetClass`, `BacklogWidgetClass`,
  `QuickMenuWidgetClass`.
- `bAutoLoadFirstChapter`, `bAutoLoadFirstScene`.

Runtime references (`BlueprintReadOnly`): `VNSubsystem`, `SceneRenderer`,
`InputHandler`, `AudioManager`, `PlaybackController`, `UIManager`,
`HUDWidget`, `DialogueWidget`, `ChoiceWidget`, `TitleMenuWidget` /
`MainMenuWidget`.

API: `InitializeVNSystem`, `LoadVNProject`, `GetHUDCanvas`,
`IsShowingTitleScene`, `StartGame` (call from main menu's "New Game"
button).

Blueprint events: `OnVNSystemInitialized`, `OnProjectLoaded`,
`OnTitleSceneLoaded`.

### UVNUIManagerSubsystem

`UI/VNUIManagerSubsystem.h` — `ULocalPlayerSubsystem`. Owns the
`UVNUILayout` widget and provides VN-specific push/show shortcuts.

API:

- `CreateLayoutForPlayer(APlayerController*)` / `DestroyLayout` /
  `HasLayout` / `GetLayout`.
- Generic push: `PushWidget`, `PushWidgetToGameLayer`,
  `PushWidgetToMenuLayer`, `PushWidgetToModalLayer`.
- VN shortcuts: `ShowDialogue`, `ShowChoices`, `ShowSettings`,
  `ShowBacklog`, `ShowSaveLoad`, `ShowMainMenu`, `ShowQuickMenu` plus
  toggle variants.
- Cached widget accessors: `GetDialogueWidget`, `GetChoiceWidget`,
  `GetSettingsWidget`, `GetMainMenuWidget`, `GetQuickMenuWidget`.
- Configuration: `SetLayoutWidgetClass`,
  `SetDefaultDialogueWidgetClass`, etc.
- `SetVNSubsystem` / `GetVNSubsystem` — also subscribes to
  `OnProjectChanged` so widgets pushed before the project finished
  loading get re-themed when it lands.
- `RefreshAllWidgetsTheme` — walks every active widget in the layout
  and re-runs init.

Delegates: `OnWidgetPushed`, `OnWidgetPopped`.

### UVNUILayout

`UI/VNUILayout.h` — root layout widget that owns three
`UCommonActivatableWidgetStack` layers:

| Layer | Tag | Purpose |
|---|---|---|
| `GameLayer` | `UI.Layer.Game` | Dialogue, choices, characters |
| `MenuLayer` | `UI.Layer.Menu` | Backlog, save/load, settings |
| `ModalLayer` | `UI.Layer.Modal` | Confirmations, popups |

API: `PushWidgetToLayer(WidgetClass, LayerTag)`,
`PushWidgetInstanceToLayer`, `GetLayerStack`, `GetGameLayer` /
`GetMenuLayer` / `GetModalLayer`, `IsLayerActive`,
`GetActiveWidgetInLayer`. Layer tags are defined in
`UI/VNUITags.h`.

## See also

- [Architecture](../architecture.md) for the runtime ownership graph and
  audio routing diagram.
- [Custom widgets](../extending/custom-widgets.md) for subclassing
  patterns.

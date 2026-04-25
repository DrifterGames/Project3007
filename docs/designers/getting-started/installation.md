# Installing VNFramework

Get the plugin into a project, enable it, and confirm everything is wired before you start authoring content. This page is for designers — no programming required.

## Prerequisites

- **Unreal Engine 5.7** (the framework is built against 5.7 APIs).
- A project that supports C++ compilation. If you only have a Blueprint-only project, you'll need to add a C++ class once so UE will compile the plugin's sources — Tools → New C++ Class → None → "MyClass" is enough.

!!! note
    VNFramework runs on Win64, Mac, and Android at runtime. The editor tooling (asset editors, Ink/Yarn importers) requires Win64 or Mac.

## Required engine plugins

VNFramework depends on these built-in plugins. Enable them in **Edit → Plugins** before enabling VNFramework:

- Enhanced Input
- CommonUI
- Model View ViewModel
- Chooser
- Metasound

## Drop the plugin in

1. Copy the `VNFramework` folder into your project's `Plugins/` directory. If `Plugins/` doesn't exist, create it next to your `.uproject` file.
2. Right-click your `.uproject` file → **Generate Visual Studio project files** (Windows) or **Services → Generate Xcode Project** (Mac).
3. Open the project. The editor will prompt to compile the plugin — click **Yes**.

## Enable VNFramework

In the editor: **Edit → Plugins → Project / VN**. Tick **VNFramework**, then restart the editor when prompted.

You should now see new menu items under the editor's main menu titled **Visual Novel** (asset editors, validator, "Create New…", "Import Script…").

## Project settings — the two INI entries that matter

VNFramework ships with a default GameMode and an Asset Manager registration set. Both live in your project's `Config/` folder. The lines below are what a working project has — copy them in if they aren't already there.

### `Config/DefaultEngine.ini`

```ini
[/Script/EngineSettings.GameMapsSettings]
GlobalDefaultGameMode=/Game/VNFramework/Core/BP_VNGameMode.BP_VNGameMode_C
```

This points the engine at the VN game mode shipped under `/Game/VNFramework/Core/`. Without it, your game starts but the VN subsystem won't be wired to a player controller / HUD.

### `Config/DefaultGame.ini`

VNFramework's data assets are *primary* assets — the Asset Manager has to know about them so they get cooked into shipping builds and can be soft-loaded at runtime. Add this block under `[/Script/Engine.AssetManagerSettings]`:

```ini
[/Script/Engine.AssetManagerSettings]
+PrimaryAssetTypesToScan=(PrimaryAssetType="VNProjectAsset",AssetBaseClass=/Script/VNCore.VNProjectAsset,bHasBlueprintClasses=False,bIsEditorOnly=False,Directories=((Path="/Game")),SpecificAssets=,Rules=(Priority=-1,ChunkId=-1,bApplyRecursively=True,CookRule=AlwaysCook))
+PrimaryAssetTypesToScan=(PrimaryAssetType="VNChapterAsset",AssetBaseClass=/Script/VNCore.VNChapterAsset,bHasBlueprintClasses=False,bIsEditorOnly=False,Directories=((Path="/Game")),SpecificAssets=,Rules=(Priority=-1,ChunkId=-1,bApplyRecursively=True,CookRule=AlwaysCook))
+PrimaryAssetTypesToScan=(PrimaryAssetType="VNSceneAsset",AssetBaseClass=/Script/VNCore.VNSceneAsset,bHasBlueprintClasses=False,bIsEditorOnly=False,Directories=((Path="/Game")),SpecificAssets=,Rules=(Priority=-1,ChunkId=-1,bApplyRecursively=True,CookRule=AlwaysCook))
+PrimaryAssetTypesToScan=(PrimaryAssetType="VNCharacterAsset",AssetBaseClass=/Script/VNCore.VNCharacterAsset,bHasBlueprintClasses=False,bIsEditorOnly=False,Directories=((Path="/Game")),SpecificAssets=,Rules=(Priority=-1,ChunkId=-1,bApplyRecursively=True,CookRule=AlwaysCook))
+PrimaryAssetTypesToScan=(PrimaryAssetType="VNBackgroundAsset",AssetBaseClass=/Script/VNCore.VNBackgroundAsset,bHasBlueprintClasses=False,bIsEditorOnly=False,Directories=((Path="/Game")),SpecificAssets=,Rules=(Priority=-1,ChunkId=-1,bApplyRecursively=True,CookRule=AlwaysCook))
+PrimaryAssetTypesToScan=(PrimaryAssetType="VNUITheme",AssetBaseClass=/Script/VNCore.VNUITheme,bHasBlueprintClasses=False,bIsEditorOnly=False,Directories=((Path="/Game")),SpecificAssets=,Rules=(Priority=-1,ChunkId=-1,bApplyRecursively=True,CookRule=AlwaysCook))
+PrimaryAssetTypesToScan=(PrimaryAssetType="VNFramePack",AssetBaseClass=/Script/VNCore.VNFramePack,bHasBlueprintClasses=False,bIsEditorOnly=False,Directories=((Path="/Game")),SpecificAssets=,Rules=(Priority=-1,ChunkId=-1,bApplyRecursively=True,CookRule=AlwaysCook))
+PrimaryAssetTypesToScan=(PrimaryAssetType="VNFontPack",AssetBaseClass=/Script/VNCore.VNFontPack,bHasBlueprintClasses=False,bIsEditorOnly=False,Directories=((Path="/Game")),SpecificAssets=,Rules=(Priority=-1,ChunkId=-1,bApplyRecursively=True,CookRule=AlwaysCook))
+PrimaryAssetTypesToScan=(PrimaryAssetType="VNIconPack",AssetBaseClass=/Script/VNCore.VNIconPack,bHasBlueprintClasses=False,bIsEditorOnly=False,Directories=((Path="/Game")),SpecificAssets=,Rules=(Priority=-1,ChunkId=-1,bApplyRecursively=True,CookRule=AlwaysCook))
```

You can also do this through **Project Settings → Asset Manager → Primary Asset Types To Scan**, but the INI block above is the fastest path and matches what the framework expects.

!!! warning
    Forgetting the Asset Manager block is the #1 cause of "everything works in PIE but my packaged build is empty" complaints. Cook a test build early to confirm.

## VNFramework developer settings

Open **Edit → Project Settings → Plugins → VNFramework**. This panel sets the audio routing the framework uses at runtime. Defaults point at the shipped `/Game/VNFramework/Audio/...` assets, so a fresh project works out of the box.

| Section | What to set | Notes |
|---------|-------------|-------|
| Sound Classes | Music, Voice, Ambient, SFX, UI | Each is a SoundClass asset whose volume the framework drives from the player's audio settings. |
| Sound Mixes | Settings, Duck Under Voice | Settings carries the per-class adjusters; Duck Under Voice is pushed while a voice line is playing. |
| Submixes | SFX, Voice | Scene-level reverb attaches to these. |
| Reverb | Reverb Wet Level Multiplier | Global tuning knob (1.0 = preset's authored wetness). |

The defaults are fine for most projects. See [Audio](../concepts/audio.md) for what each piece does.

## Folder structure

After install, your **Content Browser** will show a `VNFramework` top-level folder containing assets the framework ships:

```
/Game/VNFramework/
├── Audio/         Sound classes, mixes, submixes, reverb presets
├── CommonUI/      Input data, action bindings
├── Core/          BP_VNGameMode + supporting blueprints
├── Input/         Enhanced Input mappings
└── Materials/     Light wrap, transitions, dialogue effects
```

You can author your own VN content anywhere — common conventions are `/Game/VN/Project/`, `/Game/VN/Chapter/`, `/Game/VN/Scene/`, `/Game/VN/Character/`, `/Game/VN/Background/`, `/Game/VN/Theme/` — but nothing forces those paths. The framework finds assets via the Asset Manager, not by location.

!!! tip
    Don't move or rename anything inside `/Game/VNFramework/`. The developer settings reference these paths by default; if you need to swap them, override the references in **Project Settings → Plugins → VNFramework** instead.

## Verify the install

1. **Right-click in the Content Browser → Miscellaneous → Data Asset.** Filter the picker for "VN" — you should see Project, Chapter, Scene, Character, Background, UI Theme, Frame Pack, Font Pack, Icon Pack.
2. Open the **Visual Novel** menu in the editor's main menu bar. You should see "VN Scene Editor", "VN Character Editor", "VN Asset Validator", "Create New…", "Import Script…".
3. Press **Play in Editor** with a level open. The output log should show `LogVNFramework: VN Subsystem initialized` (or similar) — the subsystem boots whenever a game world starts.

## Troubleshooting

!!! warning "The Visual Novel menu doesn't appear"
    The plugin's editor module didn't compile. Check the Output Log for compile errors and rebuild. On Windows, regenerate Visual Studio project files and recompile from your IDE.

!!! warning "Right-click → Miscellaneous shows no VN assets"
    The runtime module didn't load. Confirm VNFramework is enabled in Plugins, restart the editor, and check that the prerequisite plugins (CommonUI, Enhanced Input, etc.) are also enabled.

!!! warning "Packaged build is empty / no scenes load"
    The Asset Manager block in `DefaultGame.ini` is missing or wrong. Verify it matches the snippet above exactly, then **Re-cook**. The Asset Manager only scans paths declared in `Directories=((Path="/Game"))`.

!!! warning "Audio is silent in PIE"
    Open **Project Settings → Plugins → VNFramework**. If any Sound Class / Mix / Submix slot is empty, audio routing breaks. Either point them at your own assets or restore the defaults under `/Game/VNFramework/Audio/`.

## See also

- [Build your first project](first-project.md)
- [Audio](../concepts/audio.md) — what the framework does with the SoundClasses you configured.
- [Theme pack reference](../reference/theme-pack.md)

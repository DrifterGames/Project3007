# VNEditor API

`VNEditor` is the editor-only module: asset factories, custom asset
editor toolkits, Slate panels, detail customizations, importers (Ink,
Yarn, CSV), and validators. Nothing in this module ships with packaged
builds.

Module path: `Plugins/VNFramework/Source/VNEditor/Public/`.

## Asset factories

`Factories/VNAssetFactories.h` — `UFactory` subclasses that hook the
*Add → New Asset* menu for every VN DataAsset. One factory per asset
type, all auto-prefixed:

| Factory | Creates | Default prefix |
|---|---|---|
| `UVNProjectAssetFactory` | `UVNProjectAsset` | `P_` |
| `UVNChapterAssetFactory` | `UVNChapterAsset` | `Ch_` |
| `UVNSceneAssetFactory` | `UVNSceneAsset` | `S_` |
| `UVNCharacterAssetFactory` | `UVNCharacterAsset` | `C_` |
| `UVNBackgroundAssetFactory` | `UVNBackgroundAsset` | `BG_` |
| `UVNUIThemeAssetFactory` | `UVNUITheme` | `Theme_` |
| `UVNFramePackAssetFactory` | `UVNFramePack` | `FramePack_` |
| `UVNFontPackAssetFactory` | `UVNFontPack` | `FontPack_` |
| `UVNIconPackAssetFactory` | `UVNIconPack` | `IconPack_` |

## Asset type actions

`AssetEditor/VNAssetTypeActions.h` — `FAssetTypeActions_Base` subclasses
that provide thumbnail color, category placement, and the
*Open Asset Editor* hook. One per DataAsset type, mirroring the factories.

## Asset editor toolkits

Custom `FAssetEditorToolkit` toolkits for the assets that benefit from
more than the generic property grid:

- `AssetEditor/VNSceneAssetEditor.h` — scene editor with a dialogue list
  view (`SVNDialogueListView`), character canvas
  (`SVNCharacterCanvas`), preview viewport (`SVNPreviewViewport`), and a
  validator panel (`SVNValidatorPanel`).
- `AssetEditor/VNCharacterAssetEditor.h` — character editor with an
  expression grid (`SVNExpressionGrid`) for 2D characters and a 3D
  preview tab.
- `AssetEditor/VNThemeAssetEditor.h` — theme editor with the theme
  preview viewport (`SVNThemePreviewViewport`).

Each toolkit registers its tabs via `RegisterTabSpawners` and routes
edits back through the standard property system.

## Slate panels

`Slate/` contains the reusable `SCompoundWidget` panels:

- `SVNDialogueListView` — the scrollable, drag-reorderable dialogue line
  list. Edits a `TArray<FVNDialogueLine>` in place.
- `SVNCharacterCanvas` / `SVNCharacterCanvasPanel` — drag-to-place
  character preview overlaid on the scene background.
- `SVNExpressionGrid` — thumbnail grid of all 2D expressions on a
  character.
- `SVNNameDropdown` — dropdown listing valid character IDs for a scene.
  Used by speaker pickers.
- `SVNPreviewViewport` — scene preview surface (background +
  characters).
- `SVNThemePreviewViewport` — renders a sample dialogue + choice + menu
  using the theme being edited so designers see changes live.
- `SVNValidatorPanel` — output panel for `UVNAssetValidator` results,
  filterable by severity. Backed by `FVNValidatorPanelManager`
  for cross-asset persistence.
- `SVNVariableInsertField` — autocomplete text field for inserting
  `Story.foo` / `Chapter.bar` etc. into expression / assignment fields.

## Detail customizations

`Customization/` — `IPropertyTypeCustomization` implementations that
make struct property rows in the details panel readable instead of
defaulted-out generic.

| Customization | Type | Adds |
|---|---|---|
| `FVNDialogueLineCustomization` | `FVNDialogueLine` | Inline speaker dropdown, GUID display, condition text field with autocomplete |
| `FVNCharacterPlacementCustomization` | `FVNCharacterPlacement` | Position/scale sliders with canvas preview |
| `FVNCharacterChangeCustomization` | `FVNCharacterChange` | Bool-gated reveal of position/scale/visibility fields |
| `FVNExpressionCustomization` | `FVNCharacterExpression` | Inline sprite thumbnail |
| `FVNChoicePromptCustomization` | `FVNChoicePrompt` | Compact rows + per-choice preview |
| `FVNSoundCueCustomization` | `FVNSoundCue` | One-line summary in the array header |
| `FVNChoiceOptionCustomization` | `FVNChoiceOption` | One-line summary in the array header |
| `FVNConditionalNextSceneCustomization` | `FVNConditionalNextScene` | One-line summary `Condition → Target` |

`Customization/VNSimpleStructHeaderCustomizations.h` collects the three
"compact header" customizations.

## CSV import / export

`CSV/VNCSVImporter.h` and `CSV/VNCSVExporter.h` — round-trip dialogue
between scene assets and CSV files for external translation /
proofreading.

`UVNCSVImporter`:

- `ImportDialogueFromCSV(FString FilePath, UVNSceneAsset* Scene)
  → FVNCSVImportResult` — matches existing lines by `LineID`, updates
  in place; appends new rows; reports counts and warnings.
- `ValidateCSVFormat(FString, TArray<FString>& Warnings, FString& Error)
  → bool`.
- `GetCSVHeaderRow() → TArray<FString>` — canonical column order.

`UVNCSVExporter`:

- `ExportDialogueToCSV(UVNSceneAsset*, FString FilePath)
  → FVNCSVExportResult`.
- `ExportChapterToCSV(UVNChapterAsset*, FString FilePath)`.
- `ExportDialogueToCSVString(UVNSceneAsset*) → FString` — for clipboard
  / network transports.

## Ink importer

`Ink/` — converts ink scripts (`*.ink`) to VNFramework assets.

- `VNInkParser.h` — `UVNInkParser` parses `.ink` source into
  `FVNInkParseResult` (knots, stitches, dialogue, choices, variables,
  LIST defs).
- `VNInkImporter.h` — `UVNInkImporter` drives the conversion:
  - `ImportToProject(parse, options, savePath)` — fresh project.
  - `ImportToExistingProject(parse, project, options, savePath)` — add
    to existing project.
  - `ImportToChapter(parse, chapter, options, savePath)` — add scenes
    to an existing chapter.
  - `ImportAsScenes(parse, options, savePath)` — standalone scenes,
    no project/chapter.
- `VNInkFileWatcher.h` — `UVNInkFileWatcher` is a `UEditorSubsystem` +
  `FTickableGameObject` that watches a configured ink directory for
  changes and triggers re-imports.
- `VNInkTypes.h` — `FVNInkParseResult`, `FVNInkImportOptions`,
  `FVNInkImportResult`.

The importer maps:

| ink concept | VNFramework target |
|---|---|
| Knot | `UVNChapterAsset` or `UVNSceneAsset` (per options) |
| Stitch | `UVNSceneAsset` |
| Speaker tag | `UVNCharacterAsset` (placeholder created if missing) |
| `* choice` | `FVNChoiceOption` |
| `VAR` | `FVNVariableDefinition` (Story scope) |
| `LIST` | `FVNEnumDefinition` |

## Yarn importer

`Yarn/` — Yarn Spinner script support. Same structural pattern as ink:

- `VNYarnParser.h` — `UVNYarnParser` produces `FVNYarnParseResult`.
- `VNYarnImporter.h` — `UVNYarnImporter`:
  - `ImportToProject(parse, project, options, savePath)
    → FVNYarnImportResult` — adds a new chapter with the parsed nodes
    as scenes.
  - `ImportToChapter(parse, chapter, options, savePath)` — adds nodes
    as scenes to an existing chapter.
- `VNYarnTypes.h` — `FVNYarnParseResult`, `FVNYarnImportOptions`,
  `FVNYarnImportResult`, `FVNYarnVariableSyncResult`.

`FVNYarnImportResult.NodeToSceneMap` reports the Yarn-node-name →
created-scene-path mapping so jump statements in Yarn become valid
soft refs to scene assets.

## Validation

`Validation/VNAssetValidator.h` — static API for asset linting. Used by
the validator panel and CI hooks.

Result types:

- `FVNValidationResult` — Severity (`Info`/`Warning`/`Error`),
  Category (`MissingReference` / `DataIntegrity` / `ExpressionError` /
  `SceneFlow` / `CharacterConfig` / `Audio` / `Configuration`),
  `Message`, `AssetPath`, `LineIndex`, `PropertyName`, `SuggestedFix`.
  Static factories `Info`, `Warning`, `Error`.
- `FVNValidationSummary` — aggregated `Results[]` plus counts and
  `bPassed`. `AddResult`, `Merge`, `GetResultsBySeverity`,
  `GetResultsByCategory`.

Static API on `UVNAssetValidator`:

| Method | Checks |
|---|---|
| `ValidateSceneAsset(const UVNSceneAsset*)` | Missing background/characters, empty dialogue, invalid conditions, choice integrity |
| `ValidateCharacterAsset(const UVNCharacterAsset*)` | Missing sprites/meshes, empty expressions, duplicate names, default-expression resolution |
| `ValidateChapterAsset(const UVNChapterAsset*)` | Missing scenes, dead ends, unreachable scenes, variable refs |
| `ValidateProjectAsset(const UVNProjectAsset*)` | Project configuration, missing chapters |
| `ValidateEntireProject(const UVNProjectAsset*)` | Recursive validation across all chapters and scenes |
| `ValidateConditionExpression(FString, FString& OutError)` | Expression syntax (delegates to `FVNExpressionEvaluator::TryValidate`) |
| `IsAssetReferenceValid(const FSoftObjectPath&)` | Asset existence and loadability |

## Editor support

- `UI/VNPreviewWidget.h` — `UVNPreviewWidget` renders a scene/character
  preview into the asset editor toolkits.
- `Menu/VNMainMenu.h` — `UVNMainMenu` adds the *VN Framework* menu to
  the editor menu bar (importers, validator panel, batch tools).
- `VNEditor.h` — module entry point. Registers asset categories,
  factories, asset type actions, customizations, and the menu extender
  in `StartupModule`; tears down in `ShutdownModule`.

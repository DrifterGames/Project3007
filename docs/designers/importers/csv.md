# CSV import / export

If your dialogue is being written in a spreadsheet — Google Sheets, Excel, an internal CMS that exports CSV — VNFramework can import lines directly into a scene asset, and export an existing scene's lines back to CSV for editing.

This is the simplest of the three importers: it operates on **one scene asset** and only handles the dialogue-line list.

## What gets imported

The CSV importer reads a row-per-line CSV with a fixed header. Columns map to dialogue line fields:

| Column | Dialogue line field | Notes |
|--------|---------------------|-------|
| `LineID` | LineID (GUID) | Empty cells get a freshly-generated GUID at import time. |
| `SpeakerID` | Speaker ID (Name) | Empty = narrator. |
| `SpeakerNameOverride` | Speaker Name Override (Text) | Empty = use the character's display name. |
| `DialogueText` | Dialogue Text (Text) | Multi-line text supported via standard CSV escaping. |
| `VoiceClip` | Voice Clip | Asset path; empty = no voice. |
| `AutoAdvance` | Auto Advance (bool) | `true` / `false`. |
| `AutoAdvanceDelay` | Auto Advance Delay (float) | Seconds. |
| `Condition` | Condition (string expression) | See [Expressions](../concepts/expressions.md). |
| `VariableAssignments` | Variable Assignments (string array) | Pipe-separated for multiple assignments in one cell. |

Header row must be exact (column names case-sensitive).

What CSV import does **not** handle:

- Character placements / Character Changes per line.
- Background Change, Sound Effects, Ambient Change.
- End Choice / Conditional Next Scenes.
- Scene-level metadata (background, audio, transitions).

For those, edit the scene asset directly after import.

## Workflow

### Import a CSV into a scene

1. **Create the scene asset** first (right-click → Miscellaneous → Data Asset → VNSceneAsset). Set Scene ID, background, character placements — everything except dialogue lines.
2. Open the scene asset.
3. Use the importer (the framework exposes an Import action via the asset toolbar / context menu — exact UI varies; check the **Visual Novel** menu for an "Import CSV" entry on the open scene).
4. Pick your `.csv` file.
5. The importer parses the CSV and replaces (or appends to) the scene's Dialogue Lines.

!!! tip
    For round-trip workflows: **export** the scene first to get a CSV with the right header, edit in your spreadsheet, **import** it back.

### Export a scene to CSV

The CSV exporter writes a scene's Dialogue Lines out using the same header. Useful for:

- Sending lines to a localization vendor.
- Editing en masse in a spreadsheet (find / replace, sort by speaker).
- Diffing two versions of a scene.

## CSV format details

- **Encoding:** UTF-8 (with or without BOM).
- **Delimiter:** comma.
- **Quoting:** double-quote a cell that contains a comma, a newline, or a double-quote. Double-quotes inside a quoted cell are escaped as `""`.
- **Multi-line dialogue:** wrap the cell in double quotes; embedded newlines are preserved.
- **Boolean:** `true` / `false` (lowercase).
- **Variable Assignments multi-value cell:** separate entries with `|` (pipe).

Example:

```csv
LineID,SpeakerID,SpeakerNameOverride,DialogueText,VoiceClip,AutoAdvance,AutoAdvanceDelay,Condition,VariableAssignments
,hero,,"Where… am I?",,false,2.0,,
,hero,,"Hello? Anyone here?",,false,2.0,,Story.helloed = true
,,,"The lights flicker on.",,true,1.5,,
```

The first column is empty in all three rows — the importer generates fresh GUIDs.

## Common patterns

!!! example "Localization round-trip"
    Author dialogue in your primary language. Export to CSV. Send to a translator. They edit `DialogueText` in place (everything else stays the same). Re-import the translated CSV — only `DialogueText` changes, all per-line metadata (Auto Advance, Condition, Voice Clip) is preserved.

!!! example "Bulk find/replace"
    Export, open in a spreadsheet, do find/replace across the `DialogueText` column, re-import. Faster than editing one cell at a time in the Details panel for a 200-line scene.

!!! example "Migrating from a custom format"
    Write a one-off script that converts your custom format into the CSV header above. Import into a fresh scene asset. Add scene-level metadata (backgrounds, characters, audio) by hand once, post-import.

## Pitfalls

!!! danger "Re-import replaces the Dialogue Lines array"
    If you've added per-line Background Change, Sound Effects, or Character Changes in the editor and then re-import a CSV, those edits are wiped — the CSV doesn't carry them. Author scene-level state in the editor *after* dialogue is finalized.

!!! warning "Empty LineID column gets fresh GUIDs"
    Every row with an empty `LineID` cell gets a new GUID at import time. This is what you want for a first import, but if you're re-importing edited lines and want to *preserve* the existing GUIDs (so backlog / read-state tracking still matches), export first to capture the GUIDs into the CSV, then edit *that* CSV.

!!! warning "Header order matters"
    The importer looks up columns by position, then validates against the expected header names. A reordered or renamed header will fail validation.

!!! warning "Excel-saved CSVs sometimes use semicolons"
    Excel in some locales saves CSV with `;` as the delimiter. The importer expects `,`. Save as "CSV UTF-8 (Comma delimited)" explicitly.

## See also

- [Scene asset reference](../reference/scene-asset.md)
- [Ink import](ink.md)
- [Yarn Spinner import](yarn.md)

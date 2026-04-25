# VNMCP overview

VNMCP is a Model Context Protocol bridge that exposes the Unreal editor
to Claude Code (and any MCP-compatible client). With it, an AI agent
can author scenes, characters, widgets, materials, MetaSounds, and
GameplayTags by issuing tool calls — no clicking through the editor.

Plugin path: `Plugins/VNMCP/`. Source of truth:
`Plugins/VNMCP/CLAUDE.md`.

## Architecture

```
Claude Code  ──MCP──►  Python server  ──TCP :55557──►  C++ bridge  ──►  UE editor
                       (FastMCP)                       (UDrifterMCPBridge)
```

- **Python server** — `Plugins/VNMCP/Content/Python/vnmcp_server.py`.
  Registers tools, dispatches calls over the local TCP socket. Restart
  by running `/mcp` in Claude Code.
- **C++ bridge** — `UDrifterMCPBridge` (the class names retained the
  `Drifter` prefix from the upstream fork; only the module / `_API`
  macro were renamed for linkage). Receives commands and dispatches to
  domain-specific handlers under `Source/VNMCP/Private/Handlers/`.

## Tool surface

VNMCP ships ~200 tools. To keep the agent's context small, only the
high-traffic ones are auto-loaded; everything else is reachable through
a search-and-execute gateway.

### Always-loaded tools (14)

These are the ones you'll reach for in 80% of authoring sessions.

| Tool | Use for |
|---|---|
| `manage_vn_project` | UVNProjectAsset hub: title, chapters, characters, backgrounds, theme |
| `manage_vn_scene` | UVNSceneAsset: dialogue lines, character placements, end-choice, transitions |
| `manage_vn_character` | UVNCharacterAsset: 2D expressions, 3D animations, voice config |
| `data_asset` | Generic DataAsset CRUD — anything the VN sugar tools don't cover. Supports dotted paths + array indices |
| `data_table` | DataTable CRUD (`get_schema` / `create` / `add_rows` / `import_csv` / `export_csv`) |
| `umg_widget` | Widget Blueprint CRUD — accepts any `parent_class` path including `/Script/VNUI.VN*` |
| `bp_manage` | Blueprint lifecycle: create, add component, add variable, add function, set CDO property, compile |
| `bp_code` | DSL-based Blueprint graph construction (event/function wrappers) |
| `bp_node` | Spawner-based single-node Blueprint authoring (create / configure / connect / describe) |
| `manage_gas_tag` | GameplayTag INI: create / validate / reload |
| `level_spawn_batch` | Spawn multiple actors in a level in one call |
| `asset_search` | Find assets by name pattern, type, or path |
| `editor_screenshot` | Capture viewport / panel / asset editor / Blueprint graph |
| `editor_log` | Read recent UE_LOG output for debugging |
| `system_status` | Connection check, version, project path, metrics |
| `atu_search_tools` / `execute_tool` | Discover and invoke any deferred tool |

### Deferred tools (the other ~180)

Discover with `atu_search_tools(query="…")`:

```
atu_search_tools(query="material")
atu_search_tools(query="metasound|chooser|niagara")
atu_search_tools(query="behavior tree")
```

Then invoke through the gateway:

```python
execute_tool(
    tool_name="manage_material",
    params={"action": "create_from_textures", ...})
```

Categories that live behind the gateway include
`manage_material` / `manage_metasound` / `manage_chooser` /
`manage_input` / `manage_struct` / `manage_enum` / `umg_animation`
plus the rest of the 30+ Drifter-inherited domain tools.

## VN sugar tools

The five `manage_vn_*` tools are thin wrappers over
`data_asset(action="modify", ...)` that add VN-domain ergonomics —
friendly action names, auto-indexing, auto-generated FGuid for `LineID`,
read-modify-write loops for arrays.

For anything they don't sugar, drop down to `data_asset` directly.

### `manage_vn_project` (UVNProjectAsset)

```
create | set_title | set_description | set_version
add_chapter | remove_chapter | list_chapters | set_starting_chapter
add_character | remove_character
add_background | remove_background
set_theme | set_title_background | set_title_music
```

### `manage_vn_chapter` (UVNChapterAsset)

```
create | set_chapter_id | set_display_name | set_chapter_number
       | set_description | set_unlock_condition
add_scene | remove_scene | list_scenes | reorder_scene
set_starting_scene | set_next_chapter
```

### `manage_vn_scene` (UVNSceneAsset)

```
create | set_scene_id | set_background | set_next_scene | set_entry_condition
add_dialogue_line | remove_dialogue_line | reorder_dialogue_line | list_dialogue_lines
add_character_placement | update_character_placement
        | remove_character_placement | list_character_placements
set_end_choice | clear_end_choice | add_choice_option
```

### `manage_vn_character` (UVNCharacterAsset)

```
create | set_character_id | set_display_name | set_character_type
# 2D:
add_expression | remove_expression | list_expressions | set_default_expression
# 3D:
set_skeletal_mesh
add_animation | remove_animation | list_animations | set_default_animation
```

### `manage_vn_variable` (narrative variables + enums)

Variables are struct-array fields on project / chapter assets. Enums
live on the project only.

```
# scope = "story" | "system" (on project) | "chapter" (on chapter)
add_variable | remove_variable | list_variables | set_variable_default
add_enum | remove_enum | list_enums
add_enum_value | remove_enum_value
```

!!! info "Don't confuse with `manage_enum` / `manage_struct`"
    `manage_enum` / `manage_struct` create *engine-level*
    `UUserDefinedEnum` / `UUserDefinedStruct` assets. VN enums are
    `FVNEnumDefinition` structs living **inside**
    `UVNProjectAsset.EnumDefinitions` — addressed via
    `manage_vn_variable`.

## VN class paths cheat sheet

For use with `umg_widget(parent_class=...)` and
`data_asset(asset_class=...)`:

```
# DataAssets
/Script/VNCore.VNProjectAsset      /Script/VNCore.VNChapterAsset
/Script/VNCore.VNSceneAsset        /Script/VNCore.VNCharacterAsset
/Script/VNCore.VNBackgroundAsset   /Script/VNCore.VNUITheme
/Script/VNCore.VNFontPack          /Script/VNCore.VNFramePack
/Script/VNCore.VNIconPack

# Widgets
/Script/VNUI.VNDialogueWidget      /Script/VNUI.VNChoiceWidget
/Script/VNUI.VNBacklogWidget       /Script/VNUI.VNMainMenuWidget
/Script/VNUI.VNSaveLoadWidget      /Script/VNUI.VNSaveSlotWidget
/Script/VNUI.VNSettingsWidget      /Script/VNUI.VNQuickMenuWidget
/Script/VNUI.VNCharacterWidget     /Script/VNUI.VNBackgroundWidget
/Script/VNUI.VNCharacterLayerWidget
/Script/VNUI.VNStyledButton        /Script/VNUI.VNStyledText
/Script/VNUI.VNStyledFrame         /Script/VNUI.VNBaseWidget
```

## Gotchas

- **Widget tree mutation in UE 5.7** — the `WidgetTree` UPROPERTY is
  `WITH_EDITORONLY_DATA` and not Python-accessible. `umg_widget` routes
  through C++ helpers; don't reinvent.
- **`TArray<UStruct>` in Python** — `get_editor_property` returns
  *copies*. Any list mutation must write elements back AND write the
  list back to the asset. The VN sugar tools handle this.
- **`save_asset(only_if_is_dirty=False)`** is mandatory after mutations
  — freshly created assets sometimes aren't "dirty" by UE's criteria
  and otherwise never flush.
- **`/mcp` reconnect** is required after any Python tool change. C++
  module changes require editor close → UBT build → editor reopen.

## See also

- [Recipes](recipes.md) — copy-paste examples for the most common
  authoring patterns.
- `Plugins/VNMCP/CLAUDE.md` — full tool reference (the canonical doc;
  this page is the curated subset).

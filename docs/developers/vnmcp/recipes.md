# VNMCP recipes

Copy-paste patterns for the authoring tasks that come up most often.
Adapted from `Plugins/VNMCP/CLAUDE.md`.

## Create a complete scene from scratch

```python
manage_vn_scene(action="create",
                scene_name="S_Prologue",
                save_path="/Game/VN/Scene",
                scene_id="prologue_01")

manage_vn_scene(action="set_background",
                scene_path="/Game/VN/Scene/S_Prologue",
                background_path="/Game/VN/Background/BG_Street_Night")

manage_vn_scene(action="add_character_placement",
                scene_path="/Game/VN/Scene/S_Prologue",
                character_id="hero",
                character_asset_path="/Game/VN/Character/C_Hero",
                initial_expression="neutral",
                position_x=0.3, position_y=0.5)

manage_vn_scene(action="add_dialogue_line",
                scene_path="/Game/VN/Scene/S_Prologue",
                speaker_id="hero",
                dialogue_text="It's quiet tonight.")

# Empty speaker_id = narrator
manage_vn_scene(action="add_dialogue_line",
                scene_path="/Game/VN/Scene/S_Prologue",
                dialogue_text="(A figure emerges from the alley.)")
```

## Add a branching choice at the end of a scene

```python
manage_vn_scene(action="set_end_choice",
                scene_path="/Game/VN/Scene/S_Crossroads",
                prompt_text="Which way?")

manage_vn_scene(action="add_choice_option",
                scene_path="/Game/VN/Scene/S_Crossroads",
                choice_text="Go left",
                choice_target_scene_path="/Game/VN/Scene/S_Left")

# Conditional choice — only available when the player has the compass
manage_vn_scene(action="add_choice_option",
                scene_path="/Game/VN/Scene/S_Crossroads",
                choice_text="Go right",
                choice_target_scene_path="/Game/VN/Scene/S_Right",
                condition="Story.has_compass == true")
```

## Build a 2D character with expressions

```python
manage_vn_character(action="create",
                    character_name="C_Heroine",
                    character_id="heroine",
                    display_name="Akari",
                    save_path="/Game/VN/Character")

manage_vn_character(action="add_expression",
                    character_path="/Game/VN/Character/C_Heroine",
                    expression_name="smile",
                    sprite_path="/Game/VN/Art/T_Akari_Smile")

manage_vn_character(action="add_expression",
                    character_path="/Game/VN/Character/C_Heroine",
                    expression_name="angry",
                    sprite_path="/Game/VN/Art/T_Akari_Angry")

manage_vn_character(action="set_default_expression",
                    character_path="/Game/VN/Character/C_Heroine",
                    default_name="smile")
```

## Create a Widget Blueprint with a VN parent class

```python
umg_widget(action="create",
           name="WBP_MainDialogue",
           parent_class="/Script/VNUI.VNDialogueWidget",
           save_path="/Game/VN/UI")

# Wire CDO properties (e.g. a default theme reference)
bp_manage(action="set_cdo_property",
          blueprint_path="/Game/VN/UI/WBP_MainDialogue",
          property_name="Theme",
          property_value="/Game/VN/Themes/Theme_Default")
```

After adding child widgets via `umg_widget(action="add", ...)`, remember
to set `is_variable=true` on any slot a `BindWidget*` C++ property is
expecting to find.

## Modify a nested struct property

`data_asset` accepts dotted paths. Use it for anything the VN sugar
tools don't sugar.

```python
# Set the entry transition's type, duration, and fade colour in one call
data_asset(action="modify",
           asset_path="/Game/VN/Scene/S_Intro",
           properties={"EntryTransition.Type": "Fade",
                       "EntryTransition.Duration": 1.5,
                       "EntryTransition.FadeColor": "(R=0,G=0,B=0,A=1)"})
```

## Modify an array element by index

Indexes in the path drive auto-grow — if the array is shorter, missing
slots are appended.

```python
data_asset(action="modify",
           asset_path="/Game/VN/Project/P_MyStory",
           properties={"StoryVariables[0].DefaultValue": "false",
                       "AutoAdvanceSettings.BaseDelay": 3.0})
```

## Bulk edit across multiple assets

`data_asset(action="bulk_modify", ...)` runs each per-asset block
independently — failure on one asset doesn't roll back the others.

```python
data_asset(action="bulk_modify", modifications=[
    {"asset_path": "/Game/VN/Scene/S_1_01",
     "properties": {"Characters[0].Scale": 1.1}},
    {"asset_path": "/Game/VN/Scene/S_1_02",
     "properties": {"Characters[0].Position": "(X=0.3,Y=0.5)"}},
])
```

## Partial update of a character placement

`update_character_placement` only writes the fields you pass. Signature
defaults (position 0.5/0.5, scale 1.0, layer 0) are treated as "no
change" — to set those values explicitly, drop down to `data_asset`.

```python
manage_vn_scene(action="update_character_placement",
                scene_path="/Game/VN/Scene/S_1_01",
                character_id="cyrus",
                expression="happy",
                scale=1.15)
```

## Author per-scene audio (reverb, BGM swap, ambient)

```python
data_asset(action="modify",
           asset_path="/Game/VN/Scene/S_Cathedral",
           properties={
               "bChangeBGM": True,
               "BackgroundMusic": "/Game/VN/Audio/Music/BGM_Choir",
               "BGMTransitionDuration": 2.5,
               "bChangeAmbient": True,
               "AmbientAudio": "/Game/VN/Audio/Ambient/AMB_Hall",
               "AmbientVolume": 0.7,
               "ReverbSubmix": "/Game/VN/Audio/Reverb/RVB_LargeHall",
               "VoiceDuckAttenuationDb": -8.0,
           })
```

!!! info "Authored vs. wired"
    `VoiceDuckAttenuationDb` and `SceneMixOverride` are stored on the
    asset but the runtime hooks aren't wired yet. They're
    forward-compatible — set them today, they activate when the audio
    overhaul lands.

## Add a per-line voice clip + SFX cue

```python
# Add the line first
manage_vn_scene(action="add_dialogue_line",
                scene_path="/Game/VN/Scene/S_1_03",
                speaker_id="hero",
                dialogue_text="Did you hear that?")

# Then stamp voice + SFX onto it via data_asset
data_asset(action="modify",
           asset_path="/Game/VN/Scene/S_1_03",
           properties={
               "DialogueLines[2].VoiceClip": "/Game/VN/Audio/Voice/V_Hero_HearThat",
               "DialogueLines[2].SoundEffects[0].Sound": "/Game/VN/Audio/SFX/SFX_DistantBell",
               "DialogueLines[2].SoundEffects[0].Delay": 0.3,
               "DialogueLines[2].SoundEffects[0].VolumeMultiplier": 0.8,
               "DialogueLines[2].SoundEffects[0].Category": "Generic",
           })
```

## Inspect a scene's lines

```python
manage_vn_scene(action="list_dialogue_lines",
                scene_path="/Game/VN/Scene/S_Prologue")
# Returns [{index, guid, speaker, text}, ...]
```

## Find every DataAsset of a type

```python
asset_search(search_term="*",
             asset_type="VNSceneAsset",
             path="/Game/VN/Scene")
```

## Discover and execute a deferred tool

```python
# Discover
atu_search_tools(query="material.*texture")
# → returns metadata for manage_material etc.

# Execute through the gateway
execute_tool(
    tool_name="manage_material",
    params={"action": "create_from_textures",
            "name": "M_Akari_Sprite",
            "save_path": "/Game/VN/Materials",
            "textures": ["/Game/VN/Art/T_Akari_Smile"]})
```

## Capture a screenshot for review

```python
# Level viewport
editor_screenshot(action="viewport")

# Full Blueprint graph
editor_screenshot(action="full_graph",
                  asset_path="/Game/VN/UI/WBP_MainDialogue")
```

## Read recent log output

```python
editor_log(action="get",
           category_filter="VN",
           min_verbosity="Warning",
           max_lines=50)
```

## See also

- [VNMCP overview](overview.md) — tool surface and architecture.
- `Plugins/VNMCP/CLAUDE.md` — exhaustive tool reference.

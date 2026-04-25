# Developer Track

You're here to extend VNFramework — write a custom widget, hook into the save system, swap a transition implementation, or build new content programmatically. This track assumes UE 5 C++ and CommonUI familiarity.

## Start here

- [Architecture](architecture.md) — module map, dependency flow, runtime data flow on a scene change, audio routing.

## API reference

One section per public class. These are hand-written summaries — for the canonical signatures, read the headers under `Plugins/VNFramework/Source/{module}/Public/`.

- [VNCore](api/vncore.md) — DataAssets, subsystem, settings, save system, expression evaluator.
- [VNUI](api/vnui.md) — widgets, scene renderer, audio manager, playback controller, transitions.
- [VNEditor](api/vneditor.md) — asset editors, importers (Ink/Yarn/CSV), validators.

## Extending the framework

Pattern docs that show real subclassing examples from the codebase.

- [Custom widgets](extending/custom-widgets.md) — extending `UVNBaseWidget` and the styled component family.
- [Custom transitions](extending/custom-transitions.md) — adding new `EVNTransitionType` values + a player.
- [Save system](extending/save-system.md) — adding new persisted state to System or User saves.
- [Settings overrides](extending/settings-overrides.md) — pointing `UVNFrameworkSettings` at your project's audio assets.

## VNMCP — authoring with Claude Code

The VNMCP plugin exposes 200+ tools to Claude Code so you can author scenes, characters, and widgets without clicking through the editor.

- [Overview](vnmcp/overview.md)
- [Recipes](vnmcp/recipes.md)

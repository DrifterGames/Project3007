---
hide:
  - navigation
  - toc
---

# VNFramework Wiki

A designer-first visual novel framework for Unreal Engine 5.7.
**Everything is a DataAsset — no Blueprints required for content.**

Pick the track that matches what you're trying to do.

<div class="grid cards" markdown>

-   :material-script-text-outline:{ .lg .middle } **For Designers**

    ---

    Build a visual novel with no code. Workflows, asset reference, screenshots, and importers for Ink, Yarn, and CSV.

    [:octicons-arrow-right-24: Designer track](designers/index.md)

-   :material-code-tags:{ .lg .middle } **For Developers**

    ---

    Extend the framework. C++ API reference, architecture, custom widgets and transitions, and the VNMCP authoring tools.

    [:octicons-arrow-right-24: Developer track](developers/index.md)

</div>

---

!!! warning "Documentation status"
    This wiki is a fresh rewrite against the current source as of {{ build_date | default('the current main branch') }}.
    The legacy markdown files under `Plugins/VNFramework/Documentation/` predate the audio overhaul and the `/Game/VNFramework/` reorg — treat anything not yet migrated here as potentially stale.

## What is VNFramework?

VNFramework is a runtime + editor plugin for UE 5.7 that turns a project into a visual novel authoring tool. The core idea is that *content is data, not Blueprints*: a story is a tree of `UVNProjectAsset → UVNChapterAsset → UVNSceneAsset → FVNDialogueLine`, and the framework's runtime widgets render that tree.

A designer working in VNFramework spends most of their time in the asset details panel for a Scene asset, not in a graph editor.

## Quick links

- [Install the plugin in your project](designers/getting-started/installation.md)
- [Build your first scene in 10 minutes](designers/getting-started/first-scene.md)
- [Architecture diagram](developers/architecture.md)

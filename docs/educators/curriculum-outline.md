# Curriculum outline

A suggested twelve-week structure for a course that uses VNFramework as its production tool. Each week assumes roughly one lecture-style session and one studio session, plus homework. Compress to eight weeks by merging weeks 1 with 2, 5 with 6, and 11 with 12; expand to a full semester by giving the capstone four weeks instead of two.

The arc moves from narrative theory and tooling, through a guided first scene, into branching and state, through audio and theme, and finally into a self-directed capstone with a public showing.

!!! tip "Instructor note"
    Before week 1, install the engine and the project on every classroom machine yourself if you can. Students will lose half a class to install variance otherwise. If lab machines are locked down, set up a single demo machine and have students install at home for week 2.

---

## Week 1 — Narrative theory and the visual novel form

Students learn what makes the VN form distinctive — the relationship between text, image, and choice — and survey examples across genres.

By end of week, students can:

- Name three visual novels and describe what each does well.
- Explain the difference between a kinetic novel, a branching VN, and a choose-your-own-adventure.
- Identify the role of music, ambience, and silence in a scene.

In-class activity: watch a ten-minute opening from one published VN with sound off, then again with sound on; discuss what the audio added.

Homework: write a one-page "scene treatment" — two characters, one location, one emotional turn — in plain prose. No format requirements yet.

Reading: [Educator index](index.md), [Glossary](glossary.md) (terms only — they will not stick yet).

---

## Week 2 — Tooling, installation, and the editor tour

Students install Unreal Engine and the project, learn to navigate the editor at a survival level, and meet the idea that the whole VN is a tree of DataAssets.

By end of week, students can:

- Open the project and locate the Content Browser, the Details panel, and the Output Log.
- Open an existing project asset and trace the link from project to chapter to scene.
- Save and reopen the editor without losing work.

In-class activity: a guided "scavenger hunt" through the example project — find the title music, find the first dialogue line, find the character with the most expressions.

Homework: re-do the scavenger hunt at home and submit a screenshot of each found item.

Reading: [Designer track index](../designers/index.md), [Glossary](glossary.md) entries for *DataAsset*, *Project asset*, *Scene asset*.

---

## Week 3 — First scene: dialogue and characters

Students author their first scene from scratch — one background, two characters, a half-dozen dialogue lines.

By end of week, students can:

- Create a new scene asset and attach a background.
- Create a character asset with at least two expressions.
- Add and reorder dialogue lines, assigning the correct speaker to each.

In-class activity: pair up — each student authors a scene of their partner's treatment from week 1, then they swap and play each other's.

Homework: convert your own week-1 treatment into a working scene with at least eight dialogue lines and one expression change per character.

Reading: [Designer track](../designers/index.md) section on scene authoring; [Assignment 1](assignment-templates.md#assignment-1-foundations-a-scene-with-feeling) (preview).

---

## Week 4 — Polishing the first scene; submit Assignment 1

Students refine pacing, expression timing, and speaker labels. Assignment 1 is due at the end of the week.

By end of week, students can:

- Critique a peer's scene using vocabulary from the glossary.
- Adjust auto-advance timing and explain why they chose those values.
- Identify and fix the three most common first-scene mistakes (wrong speaker, no expression change, unreadable text over background).

In-class activity: round-robin playtesting — every student plays at least three peers' scenes and writes one sentence of feedback for each.

Homework: submit Assignment 1 with a one-paragraph reflection on what changed between draft and final.

Reading: [Assignment templates: Foundations](assignment-templates.md#assignment-1-foundations-a-scene-with-feeling).

---

## Week 5 — Branching: choices and scene links

Students learn how scenes link to other scenes and how a single choice prompt can route the player down different paths.

By end of week, students can:

- Add a choice prompt to a scene and wire each option to a different follow-up scene.
- Diagram a three-scene branch on paper before building it.
- Distinguish a "real" choice (paths diverge meaningfully) from a "false" choice (paths reconverge with no consequence).

In-class activity: each student diagrams a four-scene branch on paper and trades it with a partner; the partner builds it.

Homework: build a working three-scene branch from your own diagram.

Reading: [Glossary](glossary.md) entries for *Choice point*, *Branching narrative*, *Dialogue tree*.

---

## Week 6 — Variables, conditions, and state

Students learn the four variable scopes and how conditions gate content based on stored state.

By end of week, students can:

- Choose the right scope for a piece of state (Scene, Chapter, Story, System) and justify the choice.
- Write a condition that shows a dialogue line only if a variable has a particular value.
- Use a choice option to set a variable and a later scene to read it.

In-class activity: a "trust meter" exercise — every student adds a Trust integer to a scene, hooks two choices to raise or lower it, and gates a later line on `Trust >= 2`.

Homework: extend last week's branch with a story-scope variable that changes dialogue in a downstream scene.

Reading: [Glossary](glossary.md) entries for *Variable scope*, *Condition*, *Story variable*, *System variable*.

---

## Week 7 — Audio: music, ambience, voice, SFX

Students learn the four audio channels and how to use each. Recording placeholder voice acting is encouraged.

By end of week, students can:

- Assign BGM to a scene and ambience that persists across multiple lines.
- Attach a voice clip to a single dialogue line.
- Explain why ambience and BGM live on different channels.

In-class activity: bring or record a thirty-second ambience clip; swap with a partner; each builds a scene that fits the other's clip.

Homework: add audio to the branching scenes from week 6 — at least BGM on every scene and ambience on at least one.

Reading: [Glossary](glossary.md) entries for *BGM*, *Ambience*, *Voice acting*.

---

## Week 8 — Theme, transitions, and visual identity; submit Assignment 2

Students explore how a theme pack reskins their VN and how transitions between scenes change perceived polish. Assignment 2 (branching narrative) is due at the end of the week.

By end of week, students can:

- Swap an existing theme pack onto their project and observe the change.
- Choose appropriate transition types between scenes and justify the choices.
- Submit a branching narrative with a working choice that drives a downstream condition.

In-class activity: two-theme A/B — every student plays the same scene under two different themes and writes one paragraph on what each theme communicates.

Homework: submit Assignment 2 with a one-paragraph reflection.

Reading: [Glossary](glossary.md) entries for *Theme pack*, *Transition*; [Assignment templates: Branching narrative](assignment-templates.md#assignment-2-branching-narrative-a-choice-that-matters).

---

## Week 9 — Capstone: pitch and pre-production

Students pitch their capstone VN — scope, theme, target length — and produce a one-page design document and a scene diagram.

By end of week, students can:

- Pitch a VN concept in three minutes with at least one visual reference.
- Diagram the full scene graph of the planned project.
- Identify the riskiest part of their plan and propose a mitigation.

In-class activity: pitch day — every student pitches for three minutes, takes two minutes of questions, and gets written feedback from peers.

Homework: revise the design document and scene diagram based on feedback; commit to final scope by next class.

Reading: [Assignment templates: Custom theme](assignment-templates.md#assignment-3-custom-theme-make-it-your-own) (preview, since theme work often happens during the capstone).

---

## Week 10 — Capstone: production

Students build the bulk of their capstone. Studio time is the focus; lecture is light.

By end of week, students can:

- Demonstrate a vertical slice — one full path from start to one ending, even if other paths are stubbed.
- Identify which assets are final and which are placeholder.
- Estimate remaining work in hours.

In-class activity: a fifteen-minute one-on-one with each student to triage scope.

Homework: continue production; aim for all scenes stubbed by next class.

Reading: students should re-read whichever glossary or designer pages they are weakest on.

---

## Week 11 — Capstone: polish and Assignment 3

Students finish a custom theme (Assignment 3) and apply it to their capstone. Polish pass on audio, transitions, and pacing.

By end of week, students can:

- Submit a working custom theme.
- Apply their theme to their capstone and judge whether it fits the tone.
- Run a complete playthrough of the capstone without crashes or dead ends.

In-class activity: bug-bash — every student plays at least two peers' projects, lists every issue found, and trades the list with the author.

Homework: fix every reproducible bug from peer testing; submit Assignment 3.

Reading: [Assignment templates: Custom theme](assignment-templates.md#assignment-3-custom-theme-make-it-your-own).

---

## Week 12 — Showcase

Students present their capstone publicly. The format depends on the venue — an open-house event, a gallery-style room with multiple stations, or recorded video walkthroughs for online courses.

By end of week, students can:

- Present their VN to a non-classmate audience for at least five minutes.
- Answer questions about their design choices using vocabulary from the glossary.
- Identify one thing they would change with another two weeks of work.

In-class activity: showcase event itself; rotate students between presenter and audience roles every twenty minutes.

Homework: a final reflection — one page on what they learned, one page on what they would do differently, and a list of glossary terms they now feel confident teaching to someone else.

Reading: optional — the [Developer track](../developers/index.md), for students who want to extend the framework after the course ends.

---

!!! info "Adjusting for shorter courses"
    For an eight-week version, fold week 1 into week 2, week 5 into week 6, and weeks 11 and 12 into a single combined polish-and-showcase week. The capstone (weeks 9 onward) is the load-bearing arc; protect it before cutting elsewhere.

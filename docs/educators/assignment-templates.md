# Assignment templates

Three ready-to-run assignments, sized to fit the [curriculum outline](curriculum-outline.md). Each is written as if you will hand it directly to students; edit names, dates, and submission channels to match your institution.

The three assignments build on each other. Assignment 1 establishes baseline competence with a single scene. Assignment 2 introduces branching and state. Assignment 3 pushes students into the visual identity of their work and asks them to defend their choices in writing.

!!! tip "Instructor note"
    Encourage students to keep their three assignments thematically connected — same world, same characters, different beats. By the end of the term they will have a coherent body of work that can be combined into a portfolio piece, which is more motivating than three disposable one-offs.

---

## Assignment 1: Foundations — a scene with feeling

!!! info "Concepts exercised"
    Scene assets, character assets with multiple expressions, background assets, dialogue lines, speaker assignment, basic pacing.

A single scene that demonstrates students can use the framework's core authoring loop: pick a background, place characters, write dialogue, and change a character's expression at a meaningful moment.

### Learning goals

- Author a scene asset from scratch and link it to a background.
- Build at least one character asset with multiple expressions.
- Write dialogue that establishes character and situation in fewer than ten lines.
- Use an expression change to mark an emotional turn.

### Deliverables

- One scene asset, playable from the editor.
- At least two character assets, each with at least two expressions.
- One background asset.
- Six or more dialogue lines, with the correct speaker on each.
- At least one expression change at a deliberate emotional beat.
- A one-paragraph artist's statement (PDF or plain text) describing the scene's situation and what the expression change is meant to communicate.

### Step-by-step starter outline

1. Sketch the scene on paper first. Two characters, one location, one emotional turn. Decide where the turn happens before you open the editor.
2. Gather or create one background image and at least four character sprites (two characters, two expressions each). Free placeholder art is fine for this assignment — substance matters more than finish.
3. Create the background asset. Point it at your background image.
4. Create each character asset. Add the expression entries; each expression points at one sprite.
5. Create the scene asset. Set its background. Place both characters.
6. Add dialogue lines one at a time. Read each line aloud as you add it.
7. At the moment you marked as the emotional turn, change one character's expression on that line.
8. Play the scene from the editor. Read it aloud again. Adjust pacing.
9. Write the artist's statement.

Cross-reference: the [Designer track](../designers/index.md) walks through each of these steps in screen-by-screen detail.

### Rubric (1–5 each, 20 total)

- **Technical correctness.** Scene plays without errors; all dialogue has correct speakers; the expression change fires on the intended line.
- **Narrative quality.** The scene establishes character and situation; the emotional turn is legible to a first-time reader.
- **Polish.** Pacing feels deliberate; dialogue reads cleanly aloud; sprite placement is intentional.
- **Documentation.** The artist's statement is specific about the intended turn and how the craft choices serve it.

### Submission format

A zipped folder containing the scene asset and its dependencies, plus the artist's statement as a single PDF or `.txt`. File-name pattern: `LastName_Assignment1.zip`.

---

## Assignment 2: Branching narrative — a choice that matters

!!! info "Concepts exercised"
    Multiple linked scenes, choice prompts, story-scope variables, conditions, downstream consequence.

A small branching story in which a single player decision changes a story-scope variable, and a later scene reads that variable in a condition to show different content. The point is not size — the point is that the choice has a downstream consequence the player can feel.

### Learning goals

- Link multiple scene assets into a branching graph.
- Author a choice prompt with at least two options.
- Use a choice option to write a story-scope variable.
- Use a condition on a later dialogue line or scene to read that variable.
- Recognize the difference between a real choice and a false one.

### Deliverables

- A minimum of three scene assets linked into a branch.
- One choice point, in any of the scenes, with at least two options.
- One story-scope variable defined on the project asset.
- At least one downstream dialogue line or scene that is gated by a condition reading that variable.
- A one-page design document, including a hand-drawn or diagrammed scene graph and a paragraph on what the variable represents and why it matters.

### Step-by-step starter outline

1. Diagram the branch on paper before opening the editor. Three scenes is the minimum; four to six is more interesting. Mark the choice point and the consequence point clearly.
2. Decide what the variable represents in plain English. Examples: trust between two characters; whether the protagonist accepted an invitation; how much the player has lied so far.
3. Add the variable to the project asset's Story variables list. Pick the right scope before doing anything else (re-read the [Glossary](glossary.md) entry on *Variable scope* if unsure).
4. Build the three scenes as if they were three Assignment 1 deliverables — backgrounds, characters, dialogue.
5. On the scene with the choice, add the choice prompt. Each option should point to a follow-up scene and, in the option that should change the variable, set the variable's new value.
6. On the consequence scene, add a dialogue line whose condition reads the variable. Verify both branches behave correctly by playing through twice.
7. Write the design document. Explain the variable in plain English and what each path tells the player about the world or the characters.

Cross-reference: the [Designer track](../designers/index.md) covers the choice prompt fields and the condition syntax.

### Rubric (1–5 each, 20 total)

- **Technical correctness.** Both branches play through cleanly; the variable changes on the intended choice; the condition fires on the intended line.
- **Narrative quality.** The choice is meaningful — the two branches communicate genuinely different things. The student can articulate why this is a real choice and not a false one.
- **Polish.** Scenes feel of a piece; dialogue across branches maintains voice; pacing of the consequence scene rewards the player who took the time to choose carefully.
- **Documentation.** The design document is specific about what the variable represents and how the consequence scene reads it.

### Submission format

A zipped folder containing the project asset, all scene assets, character and background dependencies, and the design document as a PDF. Include a `README.txt` with the entry-point scene name. File-name pattern: `LastName_Assignment2.zip`.

---

## Assignment 3: Custom theme — make it your own

!!! info "Concepts exercised"
    Theme packs, frame packs, font packs, icon packs, applying a theme to a project, the relationship between visual identity and tone.

Author a complete theme — a frame pack, a font pack, and an icon pack — and apply it to a scene from one of the previous two assignments. The deliverable is half technical (the theme works) and half critical (the student defends their design choices in writing).

### Learning goals

- Author a frame pack with a coherent visual style.
- Pick a font pack whose typography matches the tone.
- Author or assemble an icon pack consistent with the frame and font packs.
- Apply the theme to a project and judge whether the result fits the story.
- Write a short design document defending the choices.

### Deliverables

- One theme asset linking to one frame pack, one font pack, and one icon pack.
- The theme applied to a project asset that drives at least one playable scene.
- A design document (one to two pages) covering: the tone the student was aiming for; why each font, frame, and icon choice serves that tone; one alternative the student considered and rejected.
- A side-by-side screenshot — the same scene under the default theme and under the student's theme.

### Step-by-step starter outline

1. Pick a tone in one sentence before opening the editor. Examples: "warm and slightly melancholy"; "sharp and unsettled"; "playful and a little chaotic."
2. Collect references. Three or four screenshots of published VNs, films, or print pieces whose visual identity matches the tone. Annotate each with one sentence on what you are taking from it.
3. Author the frame pack. Decide on borders, corners, and the dialogue backplate. Test on a real scene early — fonts and frames look different on art than they do in isolation.
4. Pick fonts. One for body, one for character names, optionally one for UI labels. Read a paragraph of dialogue on screen before committing — what looks right at large size often fails at body size.
5. Assemble the icon pack. Save, load, settings, advance, and skip icons at minimum. Consistent line weight and corner radius matter more than artistic ambition.
6. Create the theme asset and link the three packs.
7. Apply the theme to your project and play a scene from Assignment 1 or 2 under it.
8. Capture the side-by-side screenshot.
9. Write the design document. Be specific. "The font feels right" is not a defense; "I chose a high-contrast serif for character names because the protagonist is a journalist and the names should read like a byline" is.

Cross-reference: the [Designer track](../designers/index.md) covers theme application; the [Glossary](glossary.md) entries for *Theme pack*, *Frame pack*, *Font pack*, and *Icon pack* clarify the boundaries between them.

### Rubric (1–5 each, 20 total)

- **Technical correctness.** Theme applies cleanly; all three sub-packs are present and used; no missing-asset warnings.
- **Narrative quality.** The visual choices match the stated tone. A reader who has not seen the design document can guess the tone from the screenshots.
- **Polish.** Frame, font, and icon choices feel like one coherent design rather than three unrelated ones.
- **Documentation.** The design document is specific, honest about trade-offs, and engages seriously with the rejected alternative.

### Submission format

A zipped folder containing the theme asset and all three sub-packs, the project asset configured to use the theme, the side-by-side screenshot as PNG, and the design document as PDF. File-name pattern: `LastName_Assignment3.zip`.

---

!!! tip "Grading tip"
    For all three rubrics, the *Documentation* category is where students most often coast. Pre-announce that "good" documentation answers a specific reader question — "why did you do this and not something else?" — and that vague self-praise will not score above a 2.

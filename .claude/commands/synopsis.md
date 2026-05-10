# Generate Study Synopsis

Generate a Cliff's Notes / Monarch Notes-style study synopsis — a document that understands narrative weight, giving turning points full analysis and connective chapters brief treatment.

## Prerequisites

- `scene_index.md` must exist (run `/scene-guide` first)
- `chapters/` directory must exist (run `/split` first)
- `story_bible.md` is helpful if it exists (run `/bible` first) but not required

If prerequisites are missing, tell the user which command(s) to run first and stop.

`book_summary.md` is NOT a primary source — it's too literal and flat. Use it only as a fact-checking reference if it exists. Never derive synopsis content from it.

## Process

### 1. Build the Beats Index

This is the critical first step. Read `scene_index.md` in full and classify every chapter by **narrative weight** — how much the chapter moves or transforms the story.

For each chapter, assign one of:

- **TURNING POINT** — A chapter where something irreversibly changes: a reveal that reframes the story, a decision that closes off paths, a reversal, a climax. The reader's understanding of the story is different after this chapter than before it.
- **ESCALATION** — Raises stakes, deepens conflict, or advances a character arc meaningfully, but doesn't fundamentally redirect the narrative.
- **FOUNDATION** — Establishes character, world, relationships, or tension that later chapters depend on. Important but not where the energy lives.
- **TRANSITION** — Moves characters between situations, handles logistics, provides breathing room. Necessary but low narrative weight.

Signals to use from scene_index.md:
- Key reveals → likely TURNING POINT or ESCALATION
- Blind spots (dramatic irony) → often FOUNDATION (setup for later payoff)
- New major characters introduced → FOUNDATION
- Multiple reveals + high character count → likely TURNING POINT
- No reveals, few characters, single location → likely TRANSITION

Write the beats index to `beats_index.md`:

```markdown
# Beats Index

## Narrative Weight Map

| Ch | Weight | Beat | One-Line |
|----|--------|------|----------|
| 1 | FOUNDATION | Opening state | [What is established] |
| 2 | ESCALATION | Inciting pressure | [What escalates] |
| 5 | TURNING POINT | First reversal | [What changes irreversibly] |
...

## Turning Points (in order)
1. Chapter [N]: [What happens and why it matters — 1-2 sentences]
2. Chapter [N]: ...

## Thematic Threads
- [Theme]: Introduced Ch[N], developed Ch[N,N,N], resolved/transformed Ch[N]
- [Theme]: ...

## Character Arcs (trajectory only)
- [Name]: [Starting state] → [Key shift chapters] → [End state]
```

### 2. Read chapters with weight-proportional depth

Now read chapter files, but **how much you read depends on the beat weight:**

- **TURNING POINT chapters** — Read in full. These are where meaning lives. Extract: the pivotal moment, how it's set up within the chapter, emotional register, key quotations (2-3), and thematic resonance.
- **ESCALATION chapters** — Read first third and last third. Extract: what changes, one key quotation, how it connects to the nearest turning point.
- **FOUNDATION chapters** — Read first and last paragraphs + grep for character names or thematic keywords from the beats index. Extract: what's being established and for whom.
- **TRANSITION chapters** — Do not read. Derive entirely from scene_index.md.

Process in batches of 3-5 chapters at a time.

### 3. Write the synopsis document

Structure gives turning points the most space, transitions the least:

```markdown
# [Book Title] — Study Synopsis

## Overview
[One paragraph: premise, protagonist(s), central conflict, setting, and outcome.
Written with interpretive understanding — not just what happens but what the
book is ABOUT. Spoil the ending.]

## Major Characters
[For each major character: name, role, arc trajectory.
Use beats_index.md character arcs as the skeleton, then add:
- What they represent thematically
- Their key relationships
- The chapter(s) where they transform most]

## Themes & Interpretation
[3-5 major themes. For each:
- What the theme IS (one sentence)
- How it develops — cite the turning points that advance it
- What the novel ultimately says about it (the author's implicit argument)
This is interpretation, not plot summary. A study guide reader wants to
understand MEANING, not just sequence.]

## Structure & Style
[The novel's formal choices and what they accomplish:
- POV, tense, and what that enables
- Part/act structure and how it maps to the thematic arc
- Prose style hallmarks (use style_guide.md if available)
- Pacing choices — where the novel lingers vs. rushes, and why]
```

### 4. Chapter-by-chapter entries (weight-scaled)

```markdown
## Chapter-by-Chapter Analysis

### Chapter [N]: [Descriptive Title]
*[TURNING POINT / ESCALATION / FOUNDATION / TRANSITION]*

[For TURNING POINT chapters — full analysis, 1-2 paragraphs:]
What happens, why it matters, how it reshapes the reader's understanding.
What thematic threads converge here. How characters are transformed.

> "[Key quotation]"

[Why this line matters — what it crystallizes.]

[For ESCALATION chapters — medium treatment, 3-5 sentences:]
What advances, how stakes change, connection to the nearest turning point above or below.

> "[Key quotation]"

[For FOUNDATION chapters — brief treatment, 2-3 sentences:]
What is established. What later chapters depend on from this one.

[For TRANSITION chapters — minimal, 1 sentence:]
[What moves from where to where, or what breathing room is provided.]
```

### 5. Back matter

```markdown
## Key Symbols & Motifs
[Recurring images, objects, or patterns that carry meaning.
For each: what it is, where it appears (chapter numbers),
what it represents, how its meaning shifts across the novel.]

## The Novel's Argument
[1-2 paragraphs: what is this book ultimately saying? What question
does it pose and what answer (or deliberate ambiguity) does it arrive at?
This is the "so what" a study guide reader needs.]

## Discussion Questions
[5-8 questions focused on interpretation and craft, not plot recall.
Reference specific turning points. Good questions have multiple defensible answers.]

## Timeline
[Only if chronology is non-linear or spans significant time.
Brief chronological summary of events in story-world order.]
```

### 6. Save outputs

- `beats_index.md` — the narrative weight map (reusable by other commands)
- `synopsis.md` — the full study synopsis

## Economy Rules

- The beats index is built entirely from scene_index.md — no chapter reads needed for step 1
- Chapter read depth is proportional to narrative weight (turning points: full; transitions: none)
- `book_summary.md` is a fact-check reference only, never a content source — it flattens narrative hierarchy
- Never read `novel.md` or `novel.txt`
- Grep before reading when looking for specific quotations or thematic keywords
- The beats index does the heavy interpretive lifting; chapter reads confirm and enrich it

## Handling $ARGUMENTS

If the user provides arguments:
- `chapters N-M` — generate synopsis entries only for the specified chapter range (still include front/back matter and full beats index)
- `overview only` — generate beats index + front matter sections only, no chapter-by-chapter entries
- `no quotes` — skip reading chapter files entirely; derive all content from scene_index.md + beats index (fastest mode, less rich)
- `beats only` — generate only the beats_index.md, stop there (useful as a prerequisite for other analysis)
- No arguments — full synopsis with all sections

$ARGUMENTS

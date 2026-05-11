# Generate Study Synopsis

Generate a chapter-by-chapter analytical study synopsis — a document that understands narrative weight, giving turning points full analysis and connective chapters brief treatment.

## Prerequisites

- `scene_index.md` must exist (run `/scene-guide` first)
- `chapters/` directory must exist (run `/split` first)
- `story_bible.md` is helpful if it exists (run `/bible` first) but not required

If prerequisites are missing, tell the user which command(s) to run first and stop.

`book_summary.md` is NOT a primary source — it's too literal and flat. Use it only as a fact-checking reference if it exists. Never derive synopsis content from it.

## Process

### 1. Build the Beats Index

This is a two-pass process. The first pass classifies chapters by their local narrative signals. The second pass traces causal chains to find which chapters are structurally load-bearing for the whole plot — regardless of how dramatic they feel in isolation.

#### Pass 1: Local classification

Read `scene_index.md` in full. For each chapter, assign an **initial weight** based on local signals:

- **TURNING POINT** — Something irreversibly changes: a reveal that reframes the story, a decision that closes off paths, a reversal, a climax.
- **ESCALATION** — Raises stakes, deepens conflict, or advances a character arc meaningfully, but doesn't fundamentally redirect the narrative.
- **FOUNDATION** — Establishes character, world, relationships, or tension that later chapters depend on.
- **TRANSITION** — Moves characters between situations, handles logistics, provides breathing room.

Local signals from scene_index.md:
- Key reveals → likely TURNING POINT or ESCALATION
- Blind spots (dramatic irony) → often FOUNDATION (setup for later payoff)
- New major characters introduced → FOUNDATION
- Multiple reveals + high character count → likely TURNING POINT
- No reveals, few characters, single location → likely TRANSITION

#### Pass 2: Causal chain analysis

Local signals catch what's *dramatic* but miss what's *causally central*. A quiet chapter where a character makes a private decision might enable every turning point that follows. A dramatic confrontation might be emotionally intense but causally peripheral.

Trace causal dependency by working **backward from the resolution:**

1. Identify the final chapter's resolution — what is resolved, and what made that resolution possible?
2. For each cause identified, find the chapter where it originates or crystallizes. Mark that chapter as **on the critical path**.
3. Repeat: for each critical-path chapter, ask what prior chapters were necessary for it. Follow the chain back to Chapter 1.
4. When the chain is complete, you have the novel's **causal spine** — the minimal set of chapters without which the ending doesn't work.

Then trace **secondary causal threads**: subplots, character arcs, and thematic developments that aren't on the main spine but still form their own dependency chains.

#### Reclassification rules

After tracing, adjust the initial weights:

- A chapter initially classified as FOUNDATION or TRANSITION that sits on the causal spine → upgrade to ESCALATION or TURNING POINT (it may be quiet but it's load-bearing)
- A chapter initially classified as TURNING POINT that has no downstream dependents (nothing later requires it) → downgrade to ESCALATION (locally dramatic but causally peripheral)
- A chapter that is a junction point (multiple causal threads converge or diverge from it) → TURNING POINT regardless of initial classification

Note reclassifications explicitly in the beats index with a brief reason.

#### Write `beats_index.md`

```markdown
# Beats Index

## Causal Spine
[Ordered list of chapters on the critical path from opening to resolution.
For each: chapter number, one sentence stating what it contributes to the
causal chain. This is the novel's backbone.]

## Narrative Weight Map

| Ch | Weight | Reclassified? | Beat | One-Line |
|----|--------|---------------|------|----------|
| 1 | FOUNDATION | — | Opening state | [What is established] |
| 4 | ESCALATION | ↑ from TRANSITION | Quiet pivot | [Decision that enables Ch 12] |
| 5 | TURNING POINT | — | First reversal | [What changes irreversibly] |
| 8 | ESCALATION | ↓ from TURNING POINT | Dramatic but contained | [Intense but no downstream deps] |
...

## Turning Points (in order)
1. Chapter [N]: [What happens and why it matters — 1-2 sentences]
2. Chapter [N]: ...

## Causal Threads
- **Main spine:** Ch [N] → Ch [N] → Ch [N] → ... → Resolution
- **[Subplot/arc name]:** Ch [N] → Ch [N] → Ch [N] (feeds into spine at Ch [N])
- **[Subplot/arc name]:** Ch [N] → Ch [N] → Ch [N] (resolves independently at Ch [N])

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

- Pass 1 (local classification) uses only scene_index.md — no chapter reads
- Pass 2 (causal chain) uses scene_index.md + targeted grep to confirm dependencies. Read the last chapter's final pages to anchor the resolution. Only read other chapters if the causal link is ambiguous from scene_index alone.
- Chapter read depth in step 2 is proportional to final narrative weight (turning points: full; transitions: none)
- `book_summary.md` is a fact-check reference only, never a content source — it flattens narrative hierarchy
- Never read `novel.md` or `novel.txt`
- Grep before reading when looking for specific quotations or thematic keywords
- The beats index (both passes) does the heavy interpretive lifting; chapter reads confirm and enrich it

## Handling $ARGUMENTS

If the user provides arguments:
- `chapters N-M` — generate synopsis entries only for the specified chapter range (still include front/back matter and full beats index)
- `overview only` — generate beats index + front matter sections only, no chapter-by-chapter entries
- `no quotes` — skip reading chapter files entirely; derive all content from scene_index.md + beats index (fastest mode, less rich)
- `beats only` — generate only the beats_index.md, stop there (useful as a prerequisite for other analysis)
- No arguments — full synopsis with all sections

$ARGUMENTS

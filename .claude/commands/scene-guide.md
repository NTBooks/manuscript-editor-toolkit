# Generate Scene Guide

Generate a compact scene-by-scene index of the manuscript for quick navigation and editorial reference.

## Process

This command processes the manuscript economically to minimize context usage.

### 1. Check for chapter files

If `chapters/` exists with chapter files, use those (preferred). Otherwise, tell the user to run `/split` first, or parse `novel.txt` directly if they prefer.

### 2. Process chapters in batches

Read 5 chapters at a time. For each chapter, extract:

- **Location:** Where the scene takes place
- **Characters:** Who appears (speaking or acting, not just mentioned)
- **Summary:** 2-3 sentences covering what happens. Factual, not interpretive.
- **Key events:** Major plot points, reveals, decisions, or turning points (1-3 bullet points)

If the novel uses multiple narrators or POV characters, note the narrator for each chapter.

### 3. Write scene_index.md incrementally

After each batch of 5 chapters, append the entries to `scene_index.md`. Use this format:

```markdown
# Scene Index

## How to Use
Each entry has: Location, Characters, Summary, and Key events. 
Use this to navigate the manuscript without re-reading full chapters.

---

## [Part/Section Header, if applicable]

### Chapter 1
**Location:** [place]  
**Characters:** [names]  
**Summary:** [2-3 sentences]  
**Key events:**
- [event 1]
- [event 2]

---

### Chapter 2
...
```

### 4. Add character reference

After processing all chapters, add a **Quick Character Reference** section at the top of the file (below the "How to Use" section). List every character who appears in more than one chapter with a one-line description:

```markdown
## Quick Character Reference
- **[Name]:** [role/relationship, 10 words max]
- **[Name]:** [role/relationship, 10 words max]
```

### 5. Report

Tell the user:
- How many chapters were indexed
- The character count
- That the scene index is ready for use with `/find` and `/propose`

## Economy Guidelines

- Do NOT read the entire manuscript at once
- Summaries should be factual. Save interpretation for editorial work.
- If a chapter is very short (under 500 words), still give it a full entry but note its brevity
- Focus on scenes with action, dialog, or character development
- The finished index should be usable as a standalone navigation tool by someone who has read the book

$ARGUMENTS

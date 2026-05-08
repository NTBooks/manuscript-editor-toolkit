# Summarize Manuscript

Generate a chapter-by-chapter plot summary of the entire manuscript. This is the foundation document for query letters, synopses, and pitch materials.

## Process

### 1. Check for existing resources

- If `scene_index.md` exists, use it as a starting framework (saves significant work)
- If `chapters/` exists, use chapter files. Otherwise, tell the user to run `/split` first.

### 2. Build the summary

**If scene_index.md exists:**
Read scene_index.md first. For each chapter, you already have location, characters, summary, and key events. Expand each entry into 2-4 sentences of narrative summary that capture: what happens, why it matters, and what it changes. Read the actual chapter file only when the scene index entry is too thin to write a useful summary (ambiguous key events, missing the emotional stakes or resolution of the scene).

**If scene_index.md does not exist:**
Process chapters in batches of 5. For each chapter, read it and write a 2-4 sentence narrative summary covering: what happens, why it matters, and what it changes.

### 3. Write book_summary.md

Save to `book_summary.md` in the project root:

```markdown
# [Book Title] — Plot Summary

## Overview
[3-5 sentences: protagonist(s), central conflict, setting, stakes, and how it ends. 
This is the entire book in one paragraph. SPOIL THE ENDING.]

## Chapter Summaries

### Chapter 1
[2-4 sentences. What happens, why it matters, what changes.]

### Chapter 2
[2-4 sentences.]

...

### Chapter [N] (Final)
[2-4 sentences. How the story resolves. Do not hedge or tease — state the ending plainly.]

## Character Summary
- **[Name]:** [Role, arc in one sentence — where they start, where they end]

## Key Themes
- [Theme 1]: [one sentence on how it manifests]
- [Theme 2]: [one sentence]
```

### 4. Economy rules

- Lean on scene_index.md heavily if it exists — only read chapter files to fill gaps
- Summaries are factual and plot-focused, not literary analysis
- Include every major plot turn, reveal, and resolution
- Do NOT omit the ending or any late-game twists — this document exists to spoil
- Keep each chapter summary to 2-4 sentences max; the full document should be readable in 5 minutes

$ARGUMENTS

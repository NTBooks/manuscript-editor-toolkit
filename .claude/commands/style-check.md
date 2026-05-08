# Style Check

Audit the manuscript against the style guide (`style_guide.md`). This is done primarily through grep to minimize token usage.

## Steps

### 1. Read the style guide

Read `style_guide.md` to get the current conventions. If it doesn't exist, tell the user to create one (a template is provided when the project is set up).

### 2. Grep for violations

Run targeted greps across `chapters/` for each rule in the style guide. Common checks:

**Punctuation:**
- Em dashes: grep for `—` (U+2014) and `--` (double hyphen used as em dash)
- Semicolons in dialog: grep for `;"` and `;"` (semicolons near closing quotes)
- Semicolons overall: grep for `;` and count per chapter
- Excessive ellipses: grep for `\.\.\.` or `…` and flag chapters with more than 3
- Excessive exclamation marks: grep for `!` and flag chapters with high counts

**Dialog:**
- Creative dialog tags: grep for common offenders near quotes — "exclaimed," "muttered," "hissed," "queried," "opined," "retorted," "declared"
- Capitalization errors after dialog tags: grep for patterns like `," [A-Z][a-z]` that might indicate bad capitalization (note: this catches proper nouns too, so check context)

**AI-tell words:**
- Grep for every word/phrase in the "Banned Words/Phrases" section of the style guide

**Formatting:**
- Bold in narration: grep for `**` in chapter files (if bold is banned)
- ALL CAPS words: grep for `\b[A-Z]{4,}\b` to find shouting/emphasis via caps

### 3. Report results grouped by rule

Save to `style_check.md`:

```markdown
# Style Check Report

## Summary
[X issues found across Y chapters]

## Em Dashes
[count] found
- ch.N, line X: "[surrounding text]"
- ch.M, line Y: "[surrounding text]"

## Semicolons in Dialog
[count] found
- ch.N, line X: "[surrounding text]"

## [Next rule...]

## AI-Tell Words
### "delve"
- ch.N, line X: "[surrounding text]"

### "[next word]"
- [matches]

## Clean
[Rules checked with no violations found]
```

### 4. Do NOT read full chapters

All checks are grep-based. Only include the matching line and enough context (grep -C 1) to verify the match. If a match is ambiguous (e.g., semicolon might be in narration vs. dialog), note it as "needs manual review" rather than reading the full chapter.

$ARGUMENTS

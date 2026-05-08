# Check Event Chronology

Audit the manuscript's timeline for consistency. Look for contradictions in dates, times, seasons, character ages, event ordering, and cause-effect sequences.

## Process

### 1. Build a timeline from the index

**Start with `scene_index.md`** (required). If it doesn't exist, tell the user to run `/scene-guide` first. The scene index gives you the event sequence without reading any chapter files.

### 2. Grep for temporal markers

Before reading any chapters, grep across `chapters/` for temporal keywords to build a raw timeline:

- Dates and years: grep for digit patterns like `19\d\d`, `20\d\d`, month names, day names
- Duration words: "later," "ago," "week," "month," "year," "hour," "morning," "evening," "night"
- Sequence markers: "next day," "that night," "before," "after," "meanwhile"
- Ages: grep for character names near "old," "age," "birthday," "born," "grade," "graduated"
- Seasons: "snow," "winter," "summer," "spring," "fall," "autumn," "cold," "heat"

Compile the grep results with chapter and line numbers. This gives you temporal data without reading full chapters.

### 3. Targeted reads for ambiguities

Only read specific chapter passages when:
- Two grep results appear to contradict each other
- A timeline sequence can't be resolved from the scene index + grep data alone
- You need surrounding context to determine if a reference is literal or figurative

Read only the relevant lines (use offset/limit), not the full chapter.

### 4. Check for contradictions

Look specifically for:
- Events ordered differently in different chapters
- Travel times that don't match real-world distances
- Characters in two places at the same time
- Age math that doesn't add up
- Seasonal contradictions (snow in a month/location that wouldn't have it)
- Day-of-week inconsistencies
- Historical anachronisms (technology, events, or slang that didn't exist in the stated year)
- Pregnancy, school year, or legal timelines that don't track

### 5. Generate the report

Save to `chronology_check.md`:

```markdown
# Chronology Audit

## Established Timeline
[Ordered list of key events with chapter references and dates/times]

## Issues Found

### [Issue Title]
**Chapters:** [N, M]
**Contradiction:** [what doesn't match]
**Suggested fix:** [describe the fix editorially — do not write prose]

### [Next issue...]

## Ambiguities
[Timeline points that aren't contradictions but could confuse attentive readers]

## Clean
[Areas checked with no issues found]
```

### 6. Be conservative

- Only flag clear contradictions, not style choices
- If the timeline is intentionally non-linear, note the structure but don't flag it as an error
- Distinguish between errors the author should fix and ambiguities that are fine to leave

$ARGUMENTS

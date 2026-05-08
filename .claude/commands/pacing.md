# Pacing & Structure Analysis

Generate a comprehensive pacing report analyzing chapter lengths, dialogue density, scene types, information flow, beat structure, tension arcs, and cross-part structural comparison. Output to `pacing_report.md`.

## Steps

### 1. Check prerequisites

- `scene_index.md` must exist. If not, tell the user to run `/scene-guide` first.
- `chapters/` directory must exist with chapter files. If not, tell the user to run `/split` first.

### 2. Write and run the metrics script

Write `pacing_metrics.mjs` to the project root:

```javascript
import { readdirSync, readFileSync, writeFileSync } from 'fs';
import { join } from 'path';

const chaptersDir = join(process.cwd(), 'chapters');
const indexPath = join(process.cwd(), 'scene_index.md');

const files = readdirSync(chaptersDir)
  .filter(f => /^chapter\d+\.(txt|md)$/.test(f))
  .sort((a, b) => parseInt(a.match(/\d+/)[0]) - parseInt(b.match(/\d+/)[0]));

if (!files.length) {
  console.error('No chapter files found in chapters/. Run /split first.');
  process.exit(1);
}

let indexText = '';
try {
  indexText = readFileSync(indexPath, 'utf8');
} catch {
  console.error('scene_index.md not found. Run /scene-guide first.');
  process.exit(1);
}

// Parse scene_index.md for reveals and blind spots per chapter
function parseIndex(text) {
  const chapters = {};
  const chapterBlocks = text.split(/###\s+Chapter\s+(\d+)/);
  for (let i = 1; i < chapterBlocks.length; i += 2) {
    const num = parseInt(chapterBlocks[i]);
    const block = chapterBlocks[i + 1] || '';

    const revealsMatch = block.match(/\*\*Key reveals?:\*\*\s*(.+?)(?=\n\*\*|\n---|\n###|$)/s);
    let revealCount = 0;
    let revealText = '';
    if (revealsMatch) {
      revealText = revealsMatch[1].trim();
      revealCount = revealText.split(/\.\s+/).filter(s => s.trim().length > 5).length;
      if (revealCount === 0 && revealText.length > 5) revealCount = 1;
    }

    const blindMatch = block.match(/\*\*Blind spot:\*\*\s*(.+?)(?=\n\*\*|\n---|\n###|$)/s);
    const hasBlindSpot = !!blindMatch && blindMatch[1].trim().length > 0;

    const summaryMatch = block.match(/\*\*Summary:\*\*\s*(.+?)(?=\n\*\*|\n---|\n###|$)/s);
    const summary = summaryMatch ? summaryMatch[1].trim() : '';

    chapters[num] = { revealCount, revealText, hasBlindSpot, summary };
  }
  return chapters;
}

const indexData = parseIndex(indexText);

// Process chapter files
const dialogRegex = /["""]/;
const chapters = [];
let total = 0;
let currentPart = null;

for (const file of files) {
  const text = readFileSync(join(chaptersDir, file), 'utf8');
  const num = parseInt(file.match(/\d+/)[0]);

  const partMatch = text.match(/^(?:#{1,3}\s+)?PART\s+(ONE|TWO|THREE|FOUR|FIVE|SIX|SEVEN|EIGHT|NINE|TEN|\d+)\b.*$/im);
  if (partMatch) currentPart = partMatch[0].replace(/^#{1,3}\s+/, '').trim();

  const lines = text.split('\n').filter(l => l.trim().length > 0);
  const words = text.split(/\s+/).filter(w => w.length > 0).length;
  total += words;

  let dialogLines = 0;
  let dialogWords = 0;
  for (const line of lines) {
    if (dialogRegex.test(line)) {
      dialogLines++;
      dialogWords += line.split(/\s+/).filter(w => w.length > 0).length;
    }
  }

  const totalLines = lines.length;
  const dialogPct = totalLines > 0 ? Math.round((dialogLines / totalLines) * 100) : 0;
  const dialogWordPct = words > 0 ? Math.round((dialogWords / words) * 100) : 0;

  const idx = indexData[num] || { revealCount: 0, hasBlindSpot: false, summary: '' };

  chapters.push({
    num,
    words,
    part: currentPart,
    totalLines,
    dialogLines,
    dialogPct,
    dialogWords,
    dialogWordPct,
    revealCount: idx.revealCount,
    hasBlindSpot: idx.hasBlindSpot,
    summary: idx.summary
  });
}

const avg = Math.round(total / chapters.length);
const maxWords = Math.max(...chapters.map(c => c.words));
const maxDialog = Math.max(...chapters.map(c => c.dialogPct), 1);
const maxReveals = Math.max(...chapters.map(c => c.revealCount), 1);

// Part grouping
const partGroups = {};
for (const ch of chapters) {
  const key = ch.part || 'No Part';
  if (!partGroups[key]) partGroups[key] = { chapters: [], words: 0, dialogLines: 0, totalLines: 0, reveals: 0, blindSpots: 0, first: ch.num, last: ch.num };
  partGroups[key].chapters.push(ch);
  partGroups[key].words += ch.words;
  partGroups[key].dialogLines += ch.dialogLines;
  partGroups[key].totalLines += ch.totalLines;
  partGroups[key].reveals += ch.revealCount;
  partGroups[key].blindSpots += ch.hasBlindSpot ? 1 : 0;
  partGroups[key].last = ch.num;
}

// Build markdown output
let out = '';

// Chapter length table
out += `## Chapter Length\n\n`;
out += `| Ch | Words | Bar | Part |\n|-----|-------|-----|------|\n`;
for (const ch of chapters) {
  const barLen = Math.round((ch.words / maxWords) * 20);
  const bar = '█'.repeat(barLen) + '░'.repeat(20 - barLen);
  const flag = ch.words > avg * 2 ? ' ⚠' : ch.words < Math.round(avg / 2) ? ' ⚠' : '';
  const partLabel = ch.part ? ch.part.replace(/^(?:#{1,3}\s+)?PART\s+\w+\s*[—\-]\s*/i, '').split(',')[0].trim() : '';
  out += `| ${ch.num} | ${ch.words.toLocaleString()}${flag} | ${bar} | ${partLabel} |\n`;
}

out += `\n**Total:** ${total.toLocaleString()} words | **Average chapter:** ${avg.toLocaleString()} words\n\n`;

// Dialogue density table
out += `## Dialogue Density\n\n`;
out += `| Ch | Dialog % | Bar | Dialog Words % |\n|-----|----------|-----|----------------|\n`;
for (const ch of chapters) {
  const barLen = Math.round((ch.dialogPct / maxDialog) * 20);
  const bar = '█'.repeat(barLen) + '░'.repeat(20 - barLen);
  out += `| ${ch.num} | ${ch.dialogPct}% | ${bar} | ${ch.dialogWordPct}% |\n`;
}

// Reveal density table
out += `\n## Reveal Density\n\n`;
out += `| Ch | Reveals | Bar | Blind Spot |\n|-----|---------|-----|------------|\n`;
for (const ch of chapters) {
  const barLen = Math.round((ch.revealCount / maxReveals) * 15);
  const bar = '█'.repeat(barLen) + '░'.repeat(15 - barLen);
  out += `| ${ch.num} | ${ch.revealCount} | ${bar} | ${ch.hasBlindSpot ? 'Yes' : '—'} |\n`;
}

// Per-part stats
out += `\n## Per-Part Statistics\n\n`;
out += `| Part | Chapters | Words | Avg Ch | Dialog % | Reveals | Blind Spots |\n`;
out += `|------|----------|-------|--------|----------|---------|-------------|\n`;
for (const [name, g] of Object.entries(partGroups)) {
  const partAvg = Math.round(g.words / g.chapters.length);
  const partDialogPct = g.totalLines > 0 ? Math.round((g.dialogLines / g.totalLines) * 100) : 0;
  const shortName = name.replace(/^(?:#{1,3}\s+)?PART\s+\w+\s*[—\-]\s*/i, '').split(',')[0].trim();
  out += `| ${shortName} | ${g.first}–${g.last} (${g.chapters.length}) | ${g.words.toLocaleString()} | ${partAvg.toLocaleString()} | ${partDialogPct}% | ${g.reveals} | ${g.blindSpots} |\n`;
}

// Outlier detection
const outliers = [];
for (const ch of chapters) {
  if (ch.words > avg * 2) outliers.push(`Chapter ${ch.num}: ${ch.words.toLocaleString()} words (over 2x average)`);
  if (ch.words < Math.round(avg / 2)) outliers.push(`Chapter ${ch.num}: ${ch.words.toLocaleString()} words (under half average)`);
}

// Reveal drought detection
let droughtStart = null;
const droughts = [];
for (let i = 0; i < chapters.length; i++) {
  if (chapters[i].revealCount === 0) {
    if (droughtStart === null) droughtStart = i;
  } else {
    if (droughtStart !== null && i - droughtStart >= 3) {
      droughts.push(`Chapters ${chapters[droughtStart].num}–${chapters[i - 1].num}: ${i - droughtStart} consecutive chapters with no key reveals`);
    }
    droughtStart = null;
  }
}
if (droughtStart !== null && chapters.length - droughtStart >= 3) {
  droughts.push(`Chapters ${chapters[droughtStart].num}–${chapters[chapters.length - 1].num}: ${chapters.length - droughtStart} consecutive chapters with no key reveals`);
}

if (outliers.length || droughts.length) {
  out += `\n## Mechanical Flags\n\n`;
  for (const o of outliers) out += `- ${o}\n`;
  for (const d of droughts) out += `- ${d}\n`;
  out += `\nThese are observations, not directives.\n`;
}

console.log(out);

// Write JSON for Claude analysis
const jsonData = {
  total,
  avg,
  chapters: chapters.map(c => ({ ...c })),
  partGroups: Object.fromEntries(
    Object.entries(partGroups).map(([k, v]) => [k, {
      first: v.first,
      last: v.last,
      count: v.chapters.length,
      words: v.words,
      avgChapter: Math.round(v.words / v.chapters.length),
      dialogPct: v.totalLines > 0 ? Math.round((v.dialogLines / v.totalLines) * 100) : 0,
      reveals: v.reveals,
      blindSpots: v.blindSpots
    }])
  ),
  outliers,
  droughts
};
writeFileSync(join(process.cwd(), 'pacing_data.json'), JSON.stringify(jsonData, null, 2));
```

Run: `node pacing_metrics.mjs`

Save the script's stdout output — it becomes the first half of `pacing_report.md`.

### 3. Read scene_index.md for structural analysis

Read `scene_index.md`. For each chapter, classify the **scene type** using the summary text and dialogue density from the script:

- **ACTION** — physical conflict, confrontation, chase, violence, high-stakes physical event
- **DIALOGUE** — conversation-driven, debate, negotiation, group discussion (confirm with dialog % > 70%)
- **INTROSPECTIVE** — character alone, internal processing, reflection, planning
- **TRANSITIONAL** — travel, scene-setting, time passage, setup for next event
- **EXPOSITION** — explanation of mechanics (technical, legal, financial), worldbuilding
- **EMOTIONAL** — grief, confession, relationship turning point, intimate moment

A chapter may have a secondary type (e.g., DIALOGUE/EMOTIONAL). Use dialogue density as a confirming signal, not the sole classifier.

### 4. Map beat structure per part and overall

Using scene_index.md summaries and scene type classifications, identify beats. If the novel has multiple parts, map beats **for each part** using the part names and chapter ranges from the Per-Part Statistics table:

- **Opening state** — what the narrator/protagonist believes at the start
- **Inciting incident** — the event that launches the action
- **Rising action** — escalating complications
- **Midpoint reversal** — the moment understanding shifts
- **Crisis** — the worst moment / point of no return
- **Climax** — the decisive event
- **Resolution** — where the narrator/protagonist lands

Generate a `###` heading for each part using the part name and chapter range from the metrics output. If no parts are detected, use a single `### Full Novel` section.

Then map the **overall novel** beats across all chapters. Note how each part's internal climax relates to the others (if applicable).

### 5. Assess tension arc

Using scene types, reveal counts, blind spot data, chapter position, and beat mapping, assign a **relative tension level (1-10)** to each chapter.

Tension signals (additive):
- High reveal count → +tension
- Action or emotional scene type → +tension
- Blind spot present → +tension (dramatic irony)
- Short chapter after long chapter → +tension (acceleration)
- Proximity to climax → +tension
- Transitional or exposition scene mid-part → -tension

Generate an ASCII tension curve for each part (if applicable) and one for the full novel:

```
10 |                               *
 9 |                          * *    *
 8 |                     *  *         *
 7 |                *   *
 6 |          *  * *
 5 |     *  *
 4 |   *
 3 | *
 2 |*
   +--+--+--+--+--+--+--+--+--+--+-->
    1  2  3  4  5  6  7  ...  Ch
```

### 6. Compare across parts (if multiple parts exist)

If the metrics script detected multiple parts, compare them on:

- Part lengths (word count and chapter count)
- Average chapter length
- Dialogue density trends
- Reveal density and clustering
- Beat structure timing (where does each part's midpoint fall as a %)
- Scene type distribution
- Whether successive parts accelerate (reader needs less setup each time)

If the novel has only one part or no part divisions, skip this section.

### 7. Flag pacing issues

Generate flags for potential problems (observations, not directives):

- Chapters more than 2x average length or less than half average
- Stretches of 3+ chapters with no key reveals
- Consecutive chapters of the same scene type (especially INTROSPECTIVE or TRANSITIONAL)
- Parts where the midpoint falls outside 40-60% of the part's word count
- Dialogue density dropping below 20% for 3+ consecutive chapters
- EXPOSITION chapters exceeding average chapter length
- Final 3 chapters of a part totaling less than 15% of part word count (rushed ending)
- Middle third of a part with lower reveal density than outer thirds (saggy middle)

Present every flag with context explaining why it might or might not be a problem.

### 8. Assemble the report

Combine the script output (metrics tables, charts) and the analysis sections into `pacing_report.md`. Structure:

```
# Pacing Analysis Report

## Overview
[2-3 sentences: total word count, chapter count, structure, overall pacing character]

[Script-generated tables: Chapter Length, Dialogue Density, Reveal Density, Per-Part Statistics]

## Scene Type Map
[Table: Ch | Type | Secondary | One-line summary snippet]
[Distribution per part: N action, N dialogue, N introspective, etc.]

## Beat Structure
[### heading for each part, using detected names and chapter ranges]
[### Overall Novel]

## Tension Arc
[ASCII curves per part (if applicable) and full novel]

## Structural Comparison Across Parts
[Table comparing parts + analysis — only if multiple parts exist]

## Pacing Flags
### [Flag title]
**Chapters:** [N-M]
**Observation:** [what the data shows]
**Context:** [why this might or might not be a problem]

## Notes
- Generated from scene_index.md and mechanical metrics. No chapter files read by Claude.
- Flags are observations, not directives.
- Scene type classifications are inferred from summaries and dialogue density.
```

Do NOT read any chapter files yourself. All analysis comes from `scene_index.md` + script metrics.

### Handling $ARGUMENTS

If the user provides arguments:
- `part N` or `part [name]` — run the full analysis but only report on the specified part (resolve part name/number from the Per-Part Statistics output)
- `beats only` — skip metrics tables, only output beat structure + tension arc
- `flags only` — skip narrative analysis, only output mechanical metrics + pacing flags
- No arguments — full report

$ARGUMENTS

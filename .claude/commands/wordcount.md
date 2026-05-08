# Word Count Report

Generate a detailed word count breakdown of the manuscript by running a script, not by reading chapter files.

## Steps

### 1. Write the word count script

Write `wordcount.mjs` to the project root:

```javascript
import { readdirSync, readFileSync } from 'fs';
import { join } from 'path';

const chaptersDir = join(process.cwd(), 'chapters');
const files = readdirSync(chaptersDir)
  .filter(f => /^chapter\d+\.(txt|md)$/.test(f))
  .sort((a, b) => parseInt(a.match(/\d+/)[0]) - parseInt(b.match(/\d+/)[0]));

if (!files.length) {
  console.error('No chapter files found in chapters/. Run /split first.');
  process.exit(1);
}

const chapters = [];
let total = 0;
let currentPart = null;
const parts = [];

for (const file of files) {
  const text = readFileSync(join(chaptersDir, file), 'utf8');
  const partMatch = text.match(/^PART\s+(ONE|TWO|THREE|FOUR|FIVE|SIX|SEVEN|EIGHT|NINE|TEN|\d+)\b.*$/im);
  if (partMatch) currentPart = partMatch[0].trim();
  const words = text.split(/\s+/).filter(w => w.length > 0).length;
  total += words;
  chapters.push({ num: parseInt(file.match(/\d+/)[0]), words, part: currentPart });
}

const avg = Math.round(total / chapters.length);
const shortest = chapters.reduce((a, b) => a.words < b.words ? a : b);
const longest = chapters.reduce((a, b) => a.words > b.words ? a : b);
const maxWords = longest.words;

const partGroups = {};
for (const ch of chapters) {
  const key = ch.part || 'No Part';
  if (!partGroups[key]) partGroups[key] = { chapters: [], words: 0, first: ch.num, last: ch.num };
  partGroups[key].chapters.push(ch.num);
  partGroups[key].words += ch.words;
  partGroups[key].last = ch.num;
}

let genre = '';
if (total < 7500) genre = 'Short story (under 7,500)';
else if (total < 17500) genre = 'Novelette (7,500-17,500)';
else if (total < 40000) genre = 'Novella (17,500-40,000)';
else if (total < 70000) genre = 'Short novel (40,000-70,000)';
else if (total < 100000) genre = 'Novel / literary fiction range (70,000-100,000)';
else if (total < 120000) genre = 'Novel / genre fiction range (100,000-120,000)';
else genre = 'Long novel / epic (120,000+)';

let out = `# Word Count Report\n\n`;
out += `**Total:** ${total.toLocaleString()} words\n`;
out += `**Chapters:** ${chapters.length}\n`;
out += `**Average chapter:** ${avg.toLocaleString()} words\n`;
out += `**Shortest:** Chapter ${shortest.num} (${shortest.words.toLocaleString()} words)\n`;
out += `**Longest:** Chapter ${longest.num} (${longest.words.toLocaleString()} words)\n\n`;
out += `**Genre context:** ${genre}\n\n`;
out += `## Chapter Breakdown\n\n`;
out += `| Ch | Words | Bar |\n|-----|-------|-----|\n`;

for (const ch of chapters) {
  const barLen = Math.round((ch.words / maxWords) * 20);
  const bar = '█'.repeat(barLen) + '░'.repeat(20 - barLen);
  const flag = ch.words < 500 ? ' *' : ch.words > avg * 2 ? ' **' : ch.words < avg / 2 ? ' *' : '';
  out += `| ${ch.num} | ${ch.words.toLocaleString()}${flag} | ${bar} |\n`;
}

if (Object.keys(partGroups).length > 1) {
  out += `\n## By Part\n\n| Part | Chapters | Words | % |\n|------|----------|-------|---|\n`;
  for (const [name, g] of Object.entries(partGroups)) {
    const pct = Math.round((g.words / total) * 100);
    out += `| ${name} | ${g.first}-${g.last} | ${g.words.toLocaleString()} | ${pct}% |\n`;
  }
}

const outliers = chapters.filter(c => c.words < 500 || c.words > avg * 2 || c.words < avg / 2);
if (outliers.length) {
  out += `\n## Outliers\n\n`;
  for (const ch of outliers) {
    if (ch.words < 500) out += `- **Chapter ${ch.num}** (${ch.words} words): very short\n`;
    else if (ch.words > avg * 2) out += `- **Chapter ${ch.num}** (${ch.words.toLocaleString()} words): over 2x average\n`;
    else out += `- **Chapter ${ch.num}** (${ch.words.toLocaleString()} words): under half average\n`;
  }
  out += `\nThese are observations, not directives. Short and long chapters are valid stylistic choices.\n`;
}

console.log(out);
```

### 2. Run the script

```
node wordcount.mjs
```

### 3. Show the output

Display the script's output directly to the user. The script produces a complete markdown report with tables and bar charts. Save it to `wordcount_report.md` as well.

Do NOT read any chapter files yourself. The script does all the work.

$ARGUMENTS

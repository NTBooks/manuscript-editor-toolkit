# Recombine Chapters

Recombine individual chapter files back into a single manuscript and generate a Word-ready markdown document. This is done entirely by script to avoid consuming tokens on manuscript text.

## Steps

### 1. Ask the user for metadata (if not provided in arguments)

- Book title
- Author name

### 2. Write the recombine script

Write `recombine.mjs` to the project root:

```javascript
import { readdirSync, readFileSync, writeFileSync } from 'fs';
import { join } from 'path';

const args = process.argv.slice(2);
const title = args[0] || 'Untitled';
const author = args[1] || 'Author Name';

const chaptersDir = join(process.cwd(), 'chapters');
const files = readdirSync(chaptersDir)
  .filter(f => /^chapter\d+\.(txt|md)$/.test(f))
  .sort((a, b) => parseInt(a.match(/\d+/)[0]) - parseInt(b.match(/\d+/)[0]));

if (!files.length) {
  console.error('No chapter files found in chapters/.');
  process.exit(1);
}

let novelTxt = '';
let manuscriptMd = '';
let totalWords = 0;

manuscriptMd += `<!-- \nPASTE INTO WORD INSTRUCTIONS:\n1. Select All and paste into a blank Word document\n2. Heading 1 = Part headers, Heading 2 = Chapter headers (apply styles after paste)\n3. Set body text: Times New Roman 12pt, double-spaced, 1-inch margins\n4. Use Find & Replace to convert --- lines to Section Breaks (Next Page) if desired\n5. Add page numbers: top right, starting from first chapter page\n-->\n\n`;

for (const file of files) {
  const text = readFileSync(join(chaptersDir, file), 'utf8');
  const words = text.split(/\s+/).filter(w => w.length > 0).length;
  totalWords += words;

  if (novelTxt.length > 0) novelTxt += '\n\n';
  novelTxt += text.trimEnd();

  const lines = text.split('\n');
  let chapterHeading = `Chapter ${file.match(/\d+/)[0]}`;

  for (const line of lines) {
    const trimmed = line.trim();
    if (/^PART\s+(ONE|TWO|THREE|FOUR|FIVE|SIX|SEVEN|EIGHT|NINE|TEN|\d+)\b/i.test(trimmed)) {
      manuscriptMd += `---\n\n# ${trimmed}\n\n`;
    } else if (/^chapter\s+/i.test(trimmed)) {
      chapterHeading = trimmed;
      manuscriptMd += `---\n\n## ${chapterHeading}\n\n`;
    } else if (trimmed.length > 0) {
      manuscriptMd += trimmed + '\n\n';
    }
  }
}

const titlePage = `# ${title}\n\nby ${author}\n\n${totalWords.toLocaleString()} words\n\n`;
manuscriptMd = titlePage + manuscriptMd;

writeFileSync(join(process.cwd(), 'novel.txt'), novelTxt, 'utf8');
writeFileSync(join(process.cwd(), 'manuscript.md'), manuscriptMd, 'utf8');

console.log(`Recombine complete.`);
console.log(`Chapters: ${files.length}`);
console.log(`Total words: ${totalWords.toLocaleString()}`);
console.log(`Written: novel.txt, manuscript.md`);
```

### 3. Run the script

```
node recombine.mjs "Book Title" "Author Name"
```

### 4. Report the output

Display the script's console output (chapter count, word count, files written). Do NOT read the generated files.

$ARGUMENTS

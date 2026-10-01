---
name: generate-quiz-from-wip
license: MIT
compatibility: |
  Requires: pdftotext (from poppler-utils), Python 3.x
  Environment: Linux/macOS with PDF processing tools
  Project: myquizzes

description: |
  Processes PDF or text files in the wip/ folder, reads their content, and generates
  50-question YAML quiz files in the appropriate 6.o/<subject>/ or 7.o/<subject>/
  folder. Supports singlechoice, multichoice, word, and ordering question types.
  All questions are in Hungarian. Multiple wip/ files produce multiple YAML files.
  For every quiz it also generates a themed HTML wiki page from the same source and
  stores the wiki page path in the quiz's Source: attribute.

metadata:
  category: content-generation
  language: Hungarian
  output-format: YAML + HTML
  question-types: singlechoice, multichoice, word, ordering
---

# Generate Quiz from Wip

Converts raw PDF or text study materials in `wip/` into structured YAML quiz files.

## Overview

This skill processes PDF or text files in the project's `wip/` directory and generates
50-question YAML quiz files organized by grade (6.o or 7.o) and subject.

For every quiz it also generates a companion **HTML wiki page** from the same source
material. The wiki is a readable knowledge base for the children, while the quiz tests
that knowledge. Each quiz records the wiki page path in a `Source:` attribute.


## Setup

Before using this skill, ensure you have the required dependencies:

```bash
# Install poppler-utils for PDF text extraction
sudo apt-get install poppler-utils  # Debian/Ubuntu
sudo dnf install poppler-utils      # Fedora
brew install poppler                # macOS

# Ensure Python 3 is available
python3 --version
```

## Usage

### Basic Usage

```bash
/skill:generate-quiz-from-wip
```

The skill will:
1. Scan the `wip/` directory for PDF or text files
2. Extract and read content from each file
3. Generate 50 questions based on the content
4. Save YAML files to the appropriate grade/subject folders


### With Arguments

```bash
/skill:generate-quiz-from-wip --force
```

## Workflow

1. **Input:** Place PDF or text files in the `wip/` directory
2. **Process:** The skill reads each file's content directly
3. **Generate quiz:** Creates questions based solely on the file content
4. **Generate wiki HTML:** Creates a themed HTML article from the same source (see "HTML Wiki Page")
5. **Output:** Save both files

   - Quiz: `{grade}.o/{subject}/{filename}.yaml`
   - Wiki HTML: same relative path, but under the matching `*_wiki` folder (see "Folder Mapping")
   - PDF files use `pdftotext` for extraction; text files are read directly
   - Grade (6.o or 7.o) and subject determined from filename

6. **Add the `Source:` attribute** to the quiz, pointing at the generated HTML page
7. **Archive the source:** move the processed `wip/` file into `wip/done/`

## Naming Convention

Use a **Hungarian-readable name** following existing patterns:

- `{grade}.o.{subject}.{topic}.yaml`  (e.g., `7.o.kemia.kemeny.anyagok.yaml`)
- `{grade}.o.{subject}.{topic}.{n}.yaml` (if multiple files on same topic)
- Filenames should be lowercase Hungarian, words separated by dots

## YAML Format

### Structure
```yaml
Quiz: {Hungarian title — e.g., "Kőkemény anyagok, régi segítőink a fémek"}
Source: {path to the companion HTML wiki page, relative to the repo root}
Question:
  - Type: singlechoice
    ...
  - Type: multiplechoice
    ...
  - Type: word
    ...
  - Type: ordering
    ...
```

### Question Types

#### singlechoice
One correct answer from four options (A/B/C/D):
```yaml
  - Type: singlechoice
    Text: Melyik fém a legjobb elektromos vezető?
    A: Vas
    B: Réz
    C: Alumínium
    D: Arany
    Correct: B
```

#### multichoice
Multiple correct answers from a list:
```yaml
  - Type: multiplechoice
    Text: Melyek a vasötvözetek fő összetevői?
    Answers: [vas, szén, szilícium, mangán, réz]
    Correct: [vas, szén, szilícium, mangán]
```

#### word
Free-text / short answer:
```yaml
  - Type: word
    Text: Mi a vas kémiai vegyjele?
    Answers: [Fe]
    Correct: [Fe]
```

#### ordering
Put items in the correct order:
```yaml
  - Type: ordering
    Text: Rendezd sorrendbe a következő eseményeket időrendi sorrendben
    Items:
      - Első világháború kezdete
      - Második világháború kezdete
      - Római Birodalom bukása
      - Honfoglalás
    Correct:
      - Római Birodalom bukása
      - Honfoglalás
      - Első világháború kezdete
      - Második világháború kezdete
```

## HTML Wiki Page

Every generated quiz must have a companion HTML wiki page made from the SAME source
material. The wiki is a readable knowledge base for the children, while the quiz tests
that knowledge.

### Content rules

- **One-to-one with the source:** include everything the source contains and nothing
  that it does not. Do not add facts, examples or explanations that are not in the source.
- **Reformat for readability:** turn raw or dense text into semantic HTML — headings
  (`<h1>`–`<h3>`), paragraphs, bullet and numbered lists, definition blocks, and tables
  where the source has tabular data. If the source is a simple text file, format it
  nicely with a clear structure.
- **Language:** Hungarian, same as the source.
- **Self-contained:** one `.html` file with embedded `<style>` (and inline SVG/emoji if
  needed). No build step and no local assets; only the Google Fonts link is allowed.
- Escape `<`, `>`, `&` in the source text as `&lt;`, `&gt;`, `&amp;`.
- Use the quiz title as the page `<h1>`.

### Theme (match the quiz app)

- Font: `'Poppins', sans-serif` via
  `https://fonts.googleapis.com/css2?family=Poppins:wght@300;400;500;600;700&display=swap`
- Page background: `linear-gradient(135deg, #667eea 0%, #764ba2 100%)`, `min-height: 100vh`
- Content card: white background, `border-radius: 15px`,
  `box-shadow: 0 10px 30px rgba(0, 0, 0, 0.1)`, generous padding (`2rem`)
- Main title: white, `font-weight: 700`, `text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.3)`
- Accent gradient: `linear-gradient(45deg, #667eea, #764ba2)`
- Question-type accent colors (for badges/links): singlechoice `#ff6b6b`,
  multiplechoice `#4ecdc4`, word `#45b7d1`, ordering `#9c27b0`
- Mobile friendly and readable on a phone.

### File name

Mirror the quiz file name with an `.html` extension, e.g.
`maja/7.o/biosz/7.o.biosz.a.biologiai.kutatas.yaml` →
`maja_wiki/7.o/biosz/7.o.biosz.a.biologiai.kutatas.html`.

## Folder Mapping (quiz → wiki)

The wiki lives in a parallel top-level folder; all lower folders are identical.

| Quiz location | Wiki location |
|---|---|
| `maja/...` | `maja_wiki/...` |
| `mate/...` | `mate_wiki/...` |

Examples:

- `maja/7.o/biosz/7.o.biosz.a.biologia.tudomanya.yaml` →
  `maja_wiki/7.o/biosz/7.o.biosz.a.biologia.tudomanya.html`
- `mate/8.o/biosz/8.o.biosz.taplaleklancok.anyag.es.energiaaramlas.yaml` →
  `mate_wiki/8.o/biosz/8.o.biosz.taplaleklancok.anyag.es.energiaaramlas.html`

Create the wiki folder if it does not exist.

## Source Attribute

- The quiz YAML must contain a `Source:` attribute directly after `Quiz:`.
- Its value is the path to the companion wiki HTML page, relative to the repository root.
- Example:

```yaml
Quiz: A biológiai kutatás
Source: maja_wiki/7.o/biosz/7.o.biosz.a.biologiai.kutatas.html
Question:
  - Type: singlechoice
    ...
```

- The quiz app reads only `Quiz` and `Question`, so the extra attribute is safe and can
  be used to link a quiz back to its wiki article.

## Question Distribution (per 50 questions)

| Type | Count | Notes |
|------|-------|-------|
| `singlechoice` | ~25-30 | Most common — straightforward factual questions |
| `multichoice` | ~10-15 | Lists with multiple correct options |
| `word` | ~5-10 | Specific facts: years, names, numbers, formulas |
| `ordering` | ~5-8 | Timelines, sequences, processes |

## Rules

- **All text must be in Hungarian** (questions, answers, titles, filenames)
- **Exactly 50 questions** per YAML file
- **Follow the formatting** of existing YAML files in the project
- **Generate the companion HTML wiki page** for every quiz (themed, one-to-one with the source)
- **Add the `Source:` attribute** directly after `Quiz:`, pointing at the wiki HTML page
- `Correct` field for `multichoice` and `word` uses YAML inline list syntax `[...]`
- `Correct` in `singlechoice` is a single capital letter (A, B, C, or D)
- `<` and `>` in question text must be escaped as `&lt;` and `&gt;` in YAML
- **Ordering questions:** list `Items:` in a shuffled display order and put the true order in `Correct:`; never leave `Items` already in the correct order
- If the target folder does not exist, create it
- After generation, verify the YAML is well-formed by reading it back

## Destination Examples

| Source material | Target path |
|----------------|-------------|
| `wip/7 oszt 4 Kőkemény anyagok régi segítőink a fémek...pdf` | `7.o/kemia/7.o.kemia.kemeny.anyagok.yaml` |
| `wip/7 oszy Kémia 7 4 6 Az atom ionná alakul...pdf` | `7.o/kemia/7.o.kemia.az.atom.ionna.alakul.yaml` |

Each quiz's companion wiki HTML goes to the matching `*_wiki` folder (see "Folder Mapping").

## Implementation Notes

- Uses `pdftotext` for PDF text extraction (from poppler-utils package)
- Reads text files directly
- Generates questions based solely on file content
- Generates a themed, self-contained HTML wiki page for each source
- Adds the `Source:` attribute to every quiz (path to the wiki HTML page)
- Maintains consistent YAML formatting across all generated files
- Handles Hungarian character encoding properly
- Creates necessary directories automatically

## Troubleshooting

**Issue: pdftotext not found**
- Solution: Install poppler-utils package for your system

**Issue: YAML files not created**
- Solution: Check that `wip/` directory contains files
- Solution: Verify write permissions in target directories

**Issue: Questions not in Hungarian**
- Solution: Ensure file content is in Hungarian
- Solution: Review extracted text for encoding issues

## References

- Project YAML files in `6.o/` and `7.o/` directories
- Existing quiz structure and naming conventions
- Hungarian educational terminology standards

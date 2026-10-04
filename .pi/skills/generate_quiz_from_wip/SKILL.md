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
  folder. Supports singlechoice, multichoice, word, ordering, and pairing question types.
  All questions are in Hungarian. Multiple wip/ files produce multiple YAML files.
  For every quiz it also generates a themed HTML wiki page from the same source and
  stores the wiki page path in the quiz's Source: attribute.

metadata:
  category: content-generation
  language: Hungarian
  output-format: YAML + HTML
  question-types: singlechoice, multichoice, word, ordering, pairing
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
  - Type: pairing
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

#### pairing
Match items between two columns (e.g., term ↔ definition, cause ↔ effect):
```yaml
  - Type: pairing
    Text: Párosítsd össze a fogalmakat a hozzájuk tartozó meghatározással!
    LeftTitle: Fogalom
    RightTitle: Meghatározás
    Pairs:
      - [sejtmag, az örökítőanyagot tartalmazza]
      - [sejthártya, elválasztja a sejtet a környezetétől]
      - [citoplazma, itt zajlik az anyagcsere]
```
- `Pairs` is a list of `[left, right]` pairs: the first items form the left column, the second items the right column, and the pair itself is the solution.
- **Minimum 2 pairs** per question (so at least 2 items in each column). 3–6 pairs is the practical sweet spot.
- Left values must be unique and right values must be unique, so every match is unambiguous.
- Use only when the source material naturally supports matching pairs (definitions, causes, examples, symbols, etc.).

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
- **Single file:** one `.html` file. It may reference only the shared `/static/style.css`
  and the CDN links listed under "Theme" (Poppins, Bootstrap, Font Awesome); all other
  styling stays in a small inline `<style>` block.
- Escape `<`, `>`, `&` in the source text as `&lt;`, `&gt;`, `&amp;`.
- Use the quiz title as the page `<h1>`.

### Two views: Original and Extended

Every wiki page must have a small toggle at the top with two views (default: **Original**):

- **Original** — the exact, one-to-one source content described above. It must not change.
- **Extended** — a short, kid-friendly enrichment of the SAME topic. Do a little research
  (Wikipedia/web) so every added fact is correct. It should roughly keep the original
  section structure, but explain things in whole sentences and add value:
  - a short intro ("Miről is van szó?") and 1–2 "Tudtad?" boxes
  - a few extra, accurate facts or examples — keep it short, do not turn it into a textbook
  - helpful images and links to Hungarian Wikipedia / Wikimedia Commons
  - occasional prompts to observe or think (e.g. "Figyeld meg…")

Markup: wrap the views in `#view-original` and `#view-extended` (`class="d-none"`), add the
toggle buttons, and a tiny inline `showView()` script. Example:

```html
<div class="view-toggle d-flex justify-content-center mb-4">
  <div class="btn-group" role="group" aria-label="Nézetváltó">
    <button type="button" id="btn-original" class="btn btn-outline-secondary active" aria-pressed="true" onclick="showView('original')">Eredeti</button>
    <button type="button" id="btn-extended" class="btn btn-outline-secondary" aria-pressed="false" onclick="showView('extended')">Kibővített</button>
  </div>
</div>
<h1 class="quiz-title display-5 mb-4">…quiz title…</h1>
<div id="view-original"> …exact source content… </div>
<div id="view-extended" class="d-none"> …richer version… </div>
<script>
  function showView(view) {
    var o = document.getElementById('view-original'), e = document.getElementById('view-extended');
    var bo = document.getElementById('btn-original'), be = document.getElementById('btn-extended');
    var ext = view === 'extended';
    o.classList.toggle('d-none', ext); e.classList.toggle('d-none', !ext);
    bo.classList.toggle('active', !ext); be.classList.toggle('active', ext);
    bo.setAttribute('aria-pressed', String(!ext)); be.setAttribute('aria-pressed', String(ext));
  }
</script>
```

Images: hotlink from Wikimedia Commons with
`https://commons.wikimedia.org/wiki/Special:FilePath/<FileName>?width=800` (verify the URL
returns 200). Always add a caption and a source link. No hardcoded colors (see Theme).

### Theme (inherit the quiz app's live theme)

The wiki page is served by the quiz app through its `/source/...` route, so it runs on
the same origin as the app and can reuse the app's stylesheet and saved theme. Do **not**
hardcode theme colors or gradients: the app's `/static/style.css` already themes every
element for all four themes (`original`, `dark`, `glass`, `glass-dark`).

The page `<head>` must be exactly this (only `<title>` changes):

```html
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>…quiz title…</title>
  <script>
    (function () {
      try {
        var theme = localStorage.getItem('quiz-theme');
        if (theme && theme !== 'original') {
          document.documentElement.setAttribute('data-theme', theme);
        }
      } catch (e) {}
    })();
  </script>
  <link href="https://fonts.googleapis.com/css2?family=Poppins:wght@300;400;500;600;700&display=swap" rel="stylesheet">
  <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
  <link rel="stylesheet" href="/static/style.css">
```

The anti-flash `<script>` is copied from the app's `templates/base.html`; it reads
`localStorage['quiz-theme']` and sets `data-theme` on `<html>` before first paint.

Use this body structure, reusing the app's already-themed classes:

```html
  <body>
    <div class="container py-4">
      <div class="quiz-container p-4 p-md-5">
        <h1 class="quiz-title display-5 mb-4">…quiz title…</h1>
        <div class="card mb-4"><div class="card-body">…article section…</div></div>
        …
      </div>
    </div>
  </body>
```

Requirements:

- Remove every hardcoded theme color/gradient from the generated page: do not set the
  body background, card background/shadow, or title color/shadow yourself.
  `/static/style.css` supplies these for all four themes.
- Keep a small page-specific `<style>` block only for article-specific typography/layout
  that `style.css` does not cover (e.g. `max-width`, definition blocks, tables). Any
  custom color must have `[data-theme="dark"]`, `[data-theme="glass"]` and
  `[data-theme="glass-dark"]` variants, or be avoided entirely.
- Use `quiz-title` for the `<h1>`, and `card` / `card-body` blocks for content sections.
  Use Bootstrap utilities for spacing/layout.
- Do not include a theme chooser; the wiki just inherits the user's saved theme.
- The page stays one `.html` file except for the shared `/static/style.css` and the CDN
  links above.
- **Limitation:** the page is always opened through the app's `/source/...` route (that is
  why `/static/style.css` resolves). If opened directly from disk it will be unstyled.
- **Verification:** after generating, open a wiki page through the app and confirm it
  looks right under all four themes (original, dark, glass, glass-dark).

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
| `pairing` | ~3-5 | Term ↔ definition, cause ↔ effect |

## Rules

- **All text must be in Hungarian** (questions, answers, titles, filenames)
- **Exactly 50 questions** per YAML file
- **Follow the formatting** of existing YAML files in the project
- **Generate the companion HTML wiki page** for every quiz (themed, one-to-one with the source)
- **Every wiki page has an Original / Extended toggle** (default Original); the Extended view is a short, researched, kid-friendly enrichment
- **Add the `Source:` attribute** directly after `Quiz:`, pointing at the wiki HTML page
- `Correct` field for `multichoice` and `word` uses YAML inline list syntax `[...]`
- `Correct` in `singlechoice` is a single capital letter (A, B, C, or D)
- `<` and `>` in question text must be escaped as `&lt;` and `&gt;` in YAML
- **Ordering questions:** list `Items:` in a shuffled display order and put the true order in `Correct:`; never leave `Items` already in the correct order
- **Pairing questions:** put each pair under `Pairs:` as an inline `[left, right]` list; at least 2 pairs, with unique values within each column
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

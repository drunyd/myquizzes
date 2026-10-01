---
name: generate_quiz_from_wip
description: >
  Processes PDF or text files in the wip/ folder, reads their content, and generates
  50-question YAML quiz files in the appropriate 6.o/<subject>/ or 7.o/<subject>/
  folder. Supports singlechoice, multichoice, word, and ordering question types.
  All questions are in Hungarian. Multiple wip/ files produce multiple YAML files.
  For every quiz it also generates a themed HTML wiki page from the same source and
  stores the wiki page path in the quiz's Source: attribute.
---

# Generate Quiz from Wip

Converts raw PDF study materials in `wip/` into structured YAML quiz files.

## Workflow

1. **List files in `wip/`** — find all PDF files.
2. **Determine grade and subject** from the filename (e.g., `7 oszt ...` → 7th grade chemistry, `6 oszt ...` → 6th grade). Map:
   - `kémia` → `kemia`
   - `történelem` → `tori`
   - `természet` / `földrajz` → `termeszet`
   - `biológia` → `biosz`
   - `fizika` → `fizika`
   - `nyelvtan` → `nyelvtan`
   - `matek` / `matematika` → `matek`
   - `angol` → `angol`
3. **Read each PDF** using a tool capable of PDF text extraction (or read text files directly).
4. **Generate 50 questions** per source in Hungarian, in YAML format.
5. **Generate the companion HTML wiki page** from the same source (see "HTML Wiki Page").
6. **Save the quiz** to `{grade}.o/{subject}/{filename}.yaml` with a `Source:` attribute pointing at the wiki HTML page.
7. **Save the wiki HTML** to the matching `*_wiki` folder (see "Folder Mapping").
8. **Archive the source:** move the processed `wip/` file into `wip/done/`.

## Naming Convention

Use a **Hungarian-readable name** following existing patterns (check existing YAML files in the target folder for reference):

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
```

### Question Type: `singlechoice`
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

### Question Type: `multiplechoice`
Multiple correct answers from a list:
```yaml
  - Type: multiplechoice
    Text: Melyek a vasötvözetek fő összetevői?
    Answers: [vas, szén, szilícium, mangán, réz]
    Correct: [vas, szén, szilícium, mangán]
```

### Question Type: `word`
Free-text / short answer:
```yaml
  - Type: word
    Text: Mi a vas kémiai vegyjele?
    Answers: [Fe]
    Correct: [Fe]
```

For `word` questions with numeric answers or multiple acceptable forms, provide all valid options in both `Answers` and `Correct`:
```yaml
  - Type: word
    Text: Melyik évben fedezték fel a vasötvözeteket?
    Answers: [1200, i.e. 1200]
    Correct: [1200]
```

### Question Type: `ordering`
Put items in the correct order (e.g., chronological, sequential, size):
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
- `Items` lists the items in shuffled (display) order.
- `Correct` lists them in the proper order.
- Use only when the source material naturally supports ordering (timelines, sequences, processes, size comparisons, etc.).

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
| `multiplechoice` | ~10-15 | Lists with multiple correct options |
| `word` | ~5-10 | Specific facts: years, names, numbers, formulas |
| `ordering` | ~5-8 | Timelines, sequences, processes — only if material supports it |

## Rules

- **All text must be in Hungarian** (questions, answers, titles, filenames).
- **Exactly 50 questions** per YAML file.
- **Generate the companion HTML wiki page** for every quiz (themed, one-to-one with the source).
- **Add the `Source:` attribute** directly after `Quiz:`, pointing at the wiki HTML page.
- **Follow the formatting of existing YAML files** in the project — no extra blank lines between `Type`/`Text`/options, exactly 1 blank line between questions.
- `Correct` field for `multiplechoice` and `word` uses YAML inline list syntax `[...]`.
- `Answers` in `multiplechoice` contains plausible distractors plus the correct ones.
- `Correct` in `singlechoice` is a single capital letter (A, B, C, or D).
- `<` and `>` in question text must be escaped as `&lt;` and `&gt;` in YAML.
- If the target folder does not exist, create it.
- After generation, verify the YAML is well-formed by reading it back.

## Destination Examples

| Source material | Target path |
|----------------|-------------|
| `wip/7 oszt 4 Kőkemény anyagok régi segítőink a fémek...pdf` | `7.o/kemia/7.o.kemia.kemeny.anyagok.yaml` |
| `wip/7 oszy Kémia 7 4 6 Az atom ionná alakul...pdf` | `7.o/kemia/7.o.kemia.az.atom.ionna.alakul.yaml` |

Each quiz's companion wiki HTML goes to the matching `*_wiki` folder (see "Folder Mapping").

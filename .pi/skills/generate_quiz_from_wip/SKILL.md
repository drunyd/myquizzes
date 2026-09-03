---
name: generate-quiz-from-wip
license: MIT
compatibility: |
  Requires: pdftotext (from poppler-utils), Python 3.x
  Environment: Linux/macOS with PDF processing tools
  Project: myquizzes

description: |
  Processes PDF files in the wip/ folder, reads their content, and generates
  50-question YAML quiz files in the appropriate 6.o/<subject>/ or 7.o/<subject>/
  folder. Supports singlechoice, multichoice, word, and ordering question types.
  All questions are in Hungarian. Multiple wip/ files produce multiple YAML files.

metadata:
  category: content-generation
  language: Hungarian
  output-format: YAML
  question-types: singlechoice, multichoice, word, ordering
---

# Generate Quiz from Wip

Converts raw PDF study materials in `wip/` into structured YAML quiz files.

## Overview

This skill processes PDF files in the project's `wip/` directory and generates
50-question YAML quiz files organized by grade (6.o or 7.o) and subject.

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
1. Scan the `wip/` directory for PDF files
2. Extract text from each PDF
3. Generate 50 questions per PDF
4. Save YAML files to the appropriate grade/subject folders

### With Arguments

```bash
/skill:generate-quiz-from-wip --force
```

## Workflow

1. **List files in `wip/`** — find all PDF files
2. **Determine grade and subject** from the filename (e.g., `7 oszt ...` → 7th grade chemistry, `6 oszt ...` → 6th grade)

   Subject mapping:
   - `kémia` → `kemia`
   - `történelem` → `tori`
   - `természet` / `földrajz` → `termeszet`
   - `biológia` → `biosz`
   - `fizika` → `fizika`
   - `nyelvtan` → `nyelvtan`
   - `matek` / `matematika` → `matek`
   - `angol` → `angol`

3. **Read each PDF** using `pdftotext`
4. **Generate 50 questions** per PDF in Hungarian, in YAML format
5. **Save** to `{grade}.o/{subject}/{filename}.yaml`

## Naming Convention

Use a **Hungarian-readable name** following existing patterns:

- `{grade}.o.{subject}.{topic}.yaml`  (e.g., `7.o.kemia.kemeny.anyagok.yaml`)
- `{grade}.o.{subject}.{topic}.{n}.yaml` (if multiple files on same topic)
- Filenames should be lowercase Hungarian, words separated by dots

## YAML Format

### Structure
```yaml
Quiz: {Hungarian title — e.g., "Kőkemény anyagok, régi segítőink a fémek"}
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
- `Correct` field for `multichoice` and `word` uses YAML inline list syntax `[...]`
- `Correct` in `singlechoice` is a single capital letter (A, B, C, or D)
- `<` and `>` in question text must be escaped as `&lt;` and `&gt;` in YAML
- If the target folder does not exist, create it
- After generation, verify the YAML is well-formed by reading it back

## Destination Examples

| Source material | Target path |
|----------------|-------------|
| `wip/7 oszt 4 Kőkemény anyagok régi segítőink a fémek...pdf` | `7.o/kemia/7.o.kemia.kemeny.anyagok.yaml` |
| `wip/7 oszy Kémia 7 4 6 Az atom ionná alakul...pdf` | `7.o/kemia/7.o.kemia.az.atom.ionna.alakul.yaml` |

## Implementation Notes

- Uses `pdftotext` for PDF text extraction (from poppler-utils package)
- Generates questions based on extracted text content
- Maintains consistent YAML formatting across all generated files
- Handles Hungarian character encoding properly
- Creates necessary directories automatically

## Troubleshooting

**Issue: pdftotext not found**
- Solution: Install poppler-utils package for your system

**Issue: YAML files not created**
- Solution: Check that `wip/` directory contains PDF files
- Solution: Verify write permissions in target directories

**Issue: Questions not in Hungarian**
- Solution: Ensure PDF content is in Hungarian
- Solution: Review extracted text for encoding issues

## References

- Project YAML files in `6.o/` and `7.o/` directories
- Existing quiz structure and naming conventions
- Hungarian educational terminology standards

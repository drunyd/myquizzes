---
name: generate_quiz_from_wip
description: >
  Processes PDF files in the wip/ folder, reads their content, and generates
  50-question YAML quiz files in the appropriate 6.o/<subject>/ or 7.o/<subject>/
  folder. Supports singlechoice, multichoice, and word question types.
  All questions are in Hungarian. Multiple wip/ files produce multiple YAML files.
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
3. **Read each PDF** using a tool capable of PDF text extraction.
4. **Generate 50 questions** per PDF in Hungarian, in YAML format.
5. **Save** to `{grade}.o/{subject}/{filename}.yaml`.

## Naming Convention

Use a **Hungarian-readable name** following existing patterns (check existing YAML files in the target folder for reference):

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

## Question Distribution (per 50 questions)

| Type | Count | Notes |
|------|-------|-------|
| `singlechoice` | ~30-35 | Most common — straightforward factual questions |
| `multiplechoice` | ~10-15 | Lists with multiple correct options |
| `word` | ~5-10 | Specific facts: years, names, numbers, formulas |

## Rules

- **All text must be in Hungarian** (questions, answers, titles, filenames).
- **Exactly 50 questions** per YAML file.
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

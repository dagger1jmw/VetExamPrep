# VetExamPrep — CLAUDE.md

## Project overview

Single-file browser-based veterinary exam prep app hosted on GitHub Pages.

* **Live URL:** `https://dagger1jmw.github.io/VetExamPrep/`
* **Repo:** `https://github.com/dagger1jmw/VetExamPrep`
* **Working file:** `propedeutics\_exam\_prep\_V11.html` (single HTML/CSS/JS file, \~270 KB)
* **Local path:** `C:\\Users\\dagge\\OneDrive\\Desktop\\VetExamPrep\\`
* **`index.html` = `propedeutics\_exam\_prep\_V11.html`** — they are identical. All edits go into `propedeutics\_exam\_prep\_V11.html`; deploy copies it to `index.html` for GitHub Pages. Never edit `index.html` directly and never check which is "newer" — just use V11.html.

## Deploying

```bash
npm run deploy "your commit message here"
```

This builds and pushes to GitHub Pages. Always validate JS before deploying (see below).

## Validating JavaScript before any deploy

Extract the script block and check it with Node:

```bash
python3 -c "
with open('propedeutics\_exam\_prep\_V11.html','r') as f:
    c = f.read()
s, e = c.find('<script>'), c.rfind('</script>')
open('/tmp/check.js','w').write(c\[s+8:e])
"
node --check /tmp/check.js
```

Exit code 0 = safe to deploy. Never deploy if this fails.

\---

## Architecture

One HTML file. No build step, no frameworks, no dependencies. Everything is inline:

* CSS in `<style>`
* All data arrays and JS logic in one `<script>` block at the bottom
* Pages are `<div id="page-\*" class="page">` divs — only one has `class="page active"` at a time

### Page routing

```js
showPage('home')         // propedeutics pages: home, topics, quiz, practical, pastqs, stats, ai
spShowPage('neuro','topics')  // special pathology pages: spShowPage(topic, page)
goToSubjects()           // returns to subjects landing page
```

### Nav strip system

The nav strip (`#subject-nav-strip`) renders dynamically based on `NAV\_CONFIGS`:

```js
NAV\_CONFIGS = {
  propedeutics: { label, btns: \[{id, label, fn}, ...] },
  'special-pathology': { label, btns: \[...] },
  neuro: { label, btns: \[...] },
}
renderNavStrip('neuro')   // renders the correct buttons for that subject/topic
setNavActive('neuro-topics')  // highlights the active button
```

When adding a new SP topic, add its config to `NAV\_CONFIGS` and add a `spShowPage` case.

\---

## Data structures

### Clinical Propedeutics questions — `ALL\_QUESTIONS` array

```js
{
  topic: 't01',           // t01–t31
  type: 'mcq' | 'tf',
  diff: 'easy' | 'medium' | 'hard',
  q: 'Question text',
  opts: \['A', 'B', 'C', 'D'],   // MCQ only
  ans: 1,                        // MCQ: 0-based index | TF: true/false
  exp: {
    text: 'Full explanation paragraph.',
    tip: 'Short memory aid / mnemonic.'
  }
}
```

**Current state:** 368 questions across 31 topics (t01–t31), averaging \~12 per topic.
Propedeutics topics (t01–t31)

|ID|Name|ID|Name|
|-|-|-|-|
|t01|Handling \& Restraint|t17|GI Tract Clinical Signs|
|t02|Physical Exam (IPPA)|t18|Oral Cavity \& Oesophagus|
|t03|Order of Clinical Exam|t19|Abdominal Exam: Small Animals|
|t04|Habitus \& Body Condition|t20|Abdominal Exam: Horses|
|t05|Vital Signs: Temperature|t21|Abdominal Exam: Ruminants|
|t06|Vital Signs: Pulse \& HR|t22|Rectal Examination|
|t07|Vital Signs: Respiration|t23|Respiratory System Exam|
|t08|Rumen \& Rumination|t24|Urinary System|
|t09|Chemistry Profile \& Biochem|t25|Catheterisation \& Urinalysis|
|t10|CBC \& Haematology|t26|Neurological Examination|
|t11|Mucous Membranes \& Lymph Nodes|t27|Dermatology|
|t12|Peripheral Circulation|t28|Small Mammals|
|t13|Blood Pressure Measurement|t29|Liver Examination|
|t14|Weight Changes \& Exercise Testing|t30|Skin Lesions|
|t15|Drug Administration|t31|Abdominocentesis \& Thoracocentesis|
|t16|Heart Examination \& ECG|||

### Practical questions — `PRACTICAL\_QUESTIONS` array

Same format as above but no `topic` field. Currently empty — to be populated.

### Past exam questions — `PAST\_QUESTIONS` array

Same format. Currently empty — to be populated.

### Neuropathology subtopics — `NEURO\_SUBTOPICS` array

```js
{
  icon: '🧬',
  name: 'Cells of the CNS',
  pages: 'Pages 1–2',
  content: `<h2>...</h2><p>...</p>...`   // full HTML lecture notes
}
```

**14 subtopics (index 0–13):**

|Index|Name|Pages|
|-|-|-|
|0|Cells of the CNS|1–2|
|1|Neuronal Injury Responses|2–3|
|2|Glial, Ependymal \& Microglia Responses|3–4|
|3|Circulatory Disturbances|5–6|
|4|Brain Oedema|7|
|5|CNS Defence Mechanisms \& Portals of Entry|7–8|
|6|Malformations \& Developmental Disorders|9–11|
|7|Bacterial CNS Diseases|12–14|
|8|Viral CNS Diseases|15–17|
|9|Fungal \& Parasitic CNS Diseases|18–19|
|10|Prion Diseases \& Degenerative Diseases|20–22|
|11|Traumatic Injuries|22–23|
|12|CNS Tumours|24–25|
|13|Peripheral Nervous System Disorders|26–27|

### Neuropathology questions — `NEURO\_QUESTIONS` object

**Not yet in V11.** When added, keyed by subtopic index:

```js
const NEURO\_QUESTIONS = {
  0: \[ { type:'mcq', diff:'easy', q:'...', opts:\[...], ans:0, exp:{text:'...', tip:'...'} }, ... ],
  1: \[ ... ],
  // etc.
}
```

Target: \~50 questions per subtopic. Questions go alongside (not instead of) the lecture notes — the subtopic card opens notes first, with a "Study X Questions" button at the top.

\---

## AI Tutor

* **Model:** `gemini-2.5-flash`
* **API key:** stored in `localStorage('vpGeminiKey')`. Validation: length ≥ 20 chars only (no prefix restriction — Google has changed prefixes from `AIza` to `AQ` and may change again).
* **Knowledge base:** `.txt` files fetched from GitHub raw:

```
  GITHUB\_RAW = 'https://raw.githubusercontent.com/dagger1jmw/VetExamPrep/main/Clinical%20Propedeutics/'
  ```

* **File mapping:** `AI\_TOPIC\_FILES` object maps `t01`–`t31` to their `.txt` filenames (spaces encoded as `%20`).
* **Max tokens:** 65,535 with auto-continuation loop (up to 5 chunks on MAX\_TOKENS stop reason).

\---

## Subjects structure

Two subjects currently exist:

**Clinical Propedeutics** (`id: 'propedeutics'`)

* Opens with full nav: Home, Topics, Practical, Past Exams, Progress, AI Tutor
* Pages: `page-home`, `page-topics`, `page-quiz`, `page-practical`, `page-pastqs`, `page-stats`, `page-ai`

**Special Pathology** (`id: 'special-pathology'`)

* Opens SP topic menu (`page-sp-home`) with nav showing just "Topics"
* Selecting Neuropathology loads full neuro nav: Home, Topics, Practical, Past Exams, Progress
* Neuro pages: `page-neuro-home`, `page-neuro-topics`, `page-neuro-practical`, `page-neuro-pastqs`, `page-neuro-stats`

### Adding a new SP topic (e.g. Cardiovascular)

1. Add the topic chip to `page-sp-home` HTML
2. Add its pages: `page-cardio-home`, `page-cardio-topics`, etc.
3. Add its nav config to `NAV\_CONFIGS.cardio`
4. Add `openSpTopic('cardio')` case
5. Add its subtopics array `CARDIO\_SUBTOPICS`
6. Add its questions object `CARDIO\_QUESTIONS` when ready

\---

## CSS conventions

Key classes:

* `.page` — all page divs, `display:none` by default; `.page.active` shows it
* `.hero` — gradient header banner with stats
* `.mode-card` — clickable study mode cards on home pages
* `.topic-chip` — propedeutics topic selector chips
* `.sp-topic-chip` — SP topic menu chips (`.sp-available` / `.sp-coming`)
* `.sp-subtopic-chip` — neuro subtopic selector chips
* `.sp-content` — the lecture notes body panel
* `.sp-key` — bold purple keyword highlight
* `.sp-note` — blue left-border info callout
* `.sp-warn` — yellow left-border warning callout
* `.sp-chain` — monospace pathogenesis chain block
* `.feedback.correct` / `.feedback.wrong` — quiz answer feedback
* `.exp-text` / `.exp-tip` — explanation text and memory aid inside feedback

\---

## Common tasks for Claude Code

### Add questions to a propedeutics topic

```
Add 20 more questions for topic t22 (Rectal Examination) to ALL\_QUESTIONS.
Use the same format: {topic:'t22', type:'mcq'|'tf', diff:'easy'|'medium'|'hard',
q:'...', opts:\[...], ans:N, exp:{text:'...', tip:'...'}}
```

### Add neuropathology questions for a subtopic

```
Add 50 questions for NEURO\_QUESTIONS\[1] (Neuronal Injury Responses).
If NEURO\_QUESTIONS doesn't exist yet, create it before NEURO\_SUBTOPICS.
```

### Add a new SP topic

```
Add a Cardiovascular Pathology topic to Special Pathology following the same
pattern as Neuropathology: new pages, NAV\_CONFIGS entry, CARDIO\_SUBTOPICS array.
```

### Expand lecture notes for a neuro subtopic

```
Expand the content for NEURO\_SUBTOPICS\[7] (Bacterial CNS Diseases) with more
detail. Keep the same HTML structure using .sp-content classes.
```

### Validate and deploy

```
Validate the JS, then if clean run: npm run deploy "add t22 questions"
```

\---

## MCQ distractor quality rules (CRITICAL)

All four options in every MCQ **must be equal in length and detail**. A student must not be able to score above chance by selecting the longest or most detailed option.

### The core rule

The correct answer must not stand out visually. If the correct answer is short, keep all distractors equally short. If the correct answer is long, pad distractors to match. Never let one option be 2× or more the word-count of the others.

### What to do when lengths diverge

* **Short correct answer + long distractors** — trim the distractors to match. Example:
  ```
  WRONG: opts:['Cephalic or saphenous vein',
               'Jugular vein (used for rapid volume resuscitation in critically ill patients)',
               'Ear vein (commonly used for sampling in rabbits and pigs)',
               'Coccygeal vein (used for blood collection in cattle and reptiles)']
  RIGHT: opts:['Cephalic or saphenous vein','Jugular vein','Ear vein','Coccygeal vein']
  ```
* **Long correct answer + short distractors** — add a brief parenthetical to each distractor to match length. The parenthetical must be plausible-sounding but wrong. It is acceptable to fabricate realistic-sounding veterinary detail for distractors — wrong answers are supposed to be wrong.
  ```
  WRONG: opts:['Weight loss caused by disease','Starvation','Obesity','Dehydration']
  RIGHT: opts:['Weight loss caused by disease',
               'Exhaustion from prolonged starvation or severe caloric restriction',
               'Overweight condition resulting from chronic overfeeding',
               'Progressive BCS reduction in high-performance working animals']
  ```

### Checklist before writing any batch of MCQs

1. After writing each question, eyeball all four options — if one is noticeably longer or more clause-heavy, fix it before moving on.
2. Vary which index (`ans: 0/1/2/3`) holds the correct answer — avoid clustering on index 1 or 3.
3. Distractors must be plausible at a glance but clearly wrong on reflection — avoid obviously absurd wrong answers.
4. Do not add extra clinical context or mechanism to the correct answer that isn't present in at least two other options.

### Why this matters

A student who has never studied a topic scored 86% on a practice exam simply by picking the longest or most detailed option. This completely defeats the purpose of the app. Every batch of new questions must pass the eyeball test: if you can identify the correct answer by length/detail alone without reading the content, the question is broken.

---

## What NOT to do

* Do not split the file into multiple files — it must stay as one `.html` file for GitHub Pages
* Do not add npm dependencies or a build step
* Do not use `localStorage` in artifacts/previews — it doesn't work there; only in the deployed file
* Do not change the `exp` field from `{text, tip}` object format back to a plain string
* Do not remove the `node --check` validation step before deploying
* Do not change the Gemini API key validation to check for a specific prefix
* Do not write MCQ options where the correct answer is visually distinguishable by length or detail — see MCQ distractor quality rules above


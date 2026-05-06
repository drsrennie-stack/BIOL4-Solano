# Research Workbook Build System — Instructions

A complete guide for building question-scaffolded research workbooks for case-based, TBL-style anatomy courses. Use this document to resume work if a chat session is cut off, or as a template for adapting this system to future courses (BIO 431, BIO 304, or any other case-based anatomy course).

Authored for Dr. Sharilyn Rennie's BIO 004 Human Anatomy, Summer 2026.

---

## 1. What this system produces

For each week of an N-week course, the system produces a single research workbook DOCX file. Each workbook contains seven sections (A through F) totaling roughly 30 to 45 pages of question-scaffolded research, IOA muscle tables, differential research scaffolds, a clue tracker, nightly synthesis pages, a Thursday TBL preparation checklist, and a weekend study plan with lab exam preparation.

The workbook deliberately does NOT contain case clues, differential discriminators, or the diagnosis. Students record clues live in class and research the discriminators themselves. The workbook is the research scaffold that forces them to learn the foundation anatomy and physiology while the case unfolds.

In parallel, the system produces:
- **HTML weekly pages** (one per week) that show the case lede, weekly schedule, day-by-day breakdown, and a stripped differential (option names only — no discriminators visible to students; discriminators stay in the underlying data for a future facilitator manual).
- **A schedule.html** giving the term overview with all lab exams.

---

## 2. Architecture overview

The system has three layers:

### Layer 1: Data files (Python)
- `data_p1.py` — weeks 1 to 3 (cases, schedule, day-by-day content, weekend blocks)
- `data_p2.py` — weeks 4 to 8 (same structure)
- `week_N_modules.py` (one per week, 1 through 8) — foundation research modules with question-scaffolded prompts and IOA muscle tables

### Layer 2: Data export (Python)
- `export_json.py` — reads the Python data files, computes derived fields, exports `weeks.json` for the Node generators

### Layer 3: Generators (Node.js, using the `docx` package)
- `generate_docx.js` — reads `weeks.json` plus `week_N_modules.py` files, emits all 8 weekly workbook DOCX files
- `generate.py` (Python) — emits HTML weekly pages from `weeks.json`
- `generate_schedule.py` (Python) — emits the term-level schedule.html

The Node generator imports module data from the Python `week_N_modules.py` files via `child_process.execSync` calling Python with a tiny inline script that prints the module list as JSON. This keeps the curriculum content authoring in Python (easy to edit, no JSON syntax pain) while the DOCX rendering stays in Node (because the `docx` package is the cleanest tool for the job).

---

## 3. The seven workbook sections

The structure is identical across all 8 weeks:

### Title block
- Week N of 8
- Module title (large)
- "Research workbook" subtitle (terra italics)
- Module label and dates (italic gray)
- "BIO 004 Human Anatomy, Summer 2026 | Dr. Sharilyn Rennie, Solano Community College" with terra horizontal rule beneath
- Name / Section / Team line

### How to use this workbook
Three-bullet description of foundation anatomy + differential research + case work. Closes with a terra-italic warning that case clues are revealed only in class.

### Workbook plan (front-of-document TOC)
A 6-row table (one row per section A-F) showing What and When to work on it. Day banners inside Section A back this up.

### Week at a glance
4-row table: Day, Role, Lecture focus, Lab focus.

### Lab exam block
Only present for weeks 2-8 (no lab exam on Wk 1 Monday). One paragraph naming the exam, when it is, and what it covers.

### Section A: Foundation anatomy research
Question-scaffolded modules covering the week's anatomy curriculum. Modules are sequenced from cell-level to system-level. Day banners (gold-bordered cream callouts) mark where each night's research begins.

Two module types:
- **questions** — numbered research questions with answer-line space sized to question complexity (2 lines for definitions, 3 for descriptions, 4 for comparisons or explanations, 5 for "trace through every structure" prompts)
- **muscle_table** — five-column IOA table (Muscle / Origin / Insertion / Action / Innervation) with one row per muscle. The Muscle column is pre-populated; the other four are blank for student completion. Optional follow-up research questions appear after the table.

Question scaffolding pattern within each module: define → name → locate → relate → compare → apply.

### Section B: Differential research
Five to seven option research blocks (varies by week). Each block lists the option name only, then six research prompts with answer-line space:
1. Underlying basis (anatomy, structure, or genetics)
2. Specific anatomic structures affected
3. Inheritance or developmental pattern (if applicable)
4. Three to four characteristic findings (exam, imaging, pathology)
5. Anatomic feature that distinguishes this option from the others
6. Sources cited

Discriminators are NOT shown. Students research them.

### Section C: Unknown Case N clue tracker
Title with "Unknown Case N" framing. Generic case lede. Demographic blurb. Four blank fields with answer lines: Monday in-class clue, Tuesday in-class clue, Wednesday in-class clue, Thursday TBL morning reveal.

### Section D: Nightly synthesis
Each night gets its own page (page break before each). Four nights total: Sun→Mon, Mon→Tue, Tue→Wed, Wed→Thu.

Per-night structure:
1. **Day heading** (Sun into Mon, Mon into Tue, etc.)
2. **Aim to complete tonight** — naming Section A modules and Section B options to work on
3. **Lab prep tonight** — 75% pre-study tomorrow's round table + 25% review today's, with cross-references to Section A modules and answer space
4. **Synthesis using [day's] clue(s)** — restate clue, name implicated structures, rate differential options STRONG/WEAK/NOT in a ranking table with anatomic reasoning column, predict next finding (or final ranking + master diagram on Wed→Thu)
5. **Other exploration tonight** — four labeled prompt areas with answer space: related pathophysiology, lab values, exam/imaging findings, free notes

The Sun→Mon night has a slight variant: no clues yet, "Going-in case questions" instead of synthesis questions. Lab prep block uses "Lab Exam 3 final touches" as the 25% review since there's no prior round table this week.

### Section E: Thursday TBL preparation checklist
Bullet list of what to bring + what happens in class Thursday (iRAT, tRAT, Thursday morning reveal, application, debrief).

### Section F: Lab Exam X prep + transition to next week
Three-bucket weekend plan:
- **Friday: Rest** — single paragraph, full disconnect
- **Saturday: Lab Exam X prep** — exam logistics (50 min, 25 stations × 2 questions, 80% current week + 20% spiral review), self-test routine using the workbook (cover Section A answers, cover IOA columns, atlas drilling, ID under time, spiral review structures), Track your weak spots block with answer space
- **Sunday: Prime next week, final exam review** — light review, print next workbook, sleep early; pulled from `weekend.blocks[2].items` in the data file

For Wk 1, no upcoming Mon lab exam to prep for (LE 1 covers Wk 1 content, on Wk 2 Monday). Section F still works since Wk 1 has its own LE 1 prep on the weekend after Wk 1.

For Wk 8, the cumulative final (LE 8) is Thursday afternoon of Wk 8 — no following Monday. Section F should reframe to "End-of-term reflection + LE 8 final review" with no Monday-prep framing.

---

## 4. Standing technical rules

These apply to every deliverable:

1. **No em dashes anywhere** — in content, code, comments, or file names. Replace with commas, periods, "or", or rewrite. This rule is locked in user memory.
2. **No smart/curly quotes** — use straight quotes only.
3. **No purple anywhere.**
4. **Author byline** — "Dr. Sharilyn Rennie" only, no credential suffix.
5. **No atlas page placeholders** — student-facing materials should not include "Atlas pages: ___" lines that look incomplete. Atlas is being recreated.
6. **Innervation column kept** on all muscle IOA tables.
7. **Drop intrinsic hand muscles (Wk 4) and intrinsic foot muscles (Wk 6).**
8. **All HTML deliverables must meet WCAG 2.2 AA** — semantic HTML, proper heading hierarchy, label/input association, accessible names for icon-only buttons, aria-live on dynamic regions, aria-expanded on collapsibles, skip links, visible focus indicators, prefers-reduced-motion support, 4.5:1 contrast (normal text), 3:1 (large/UI), fillable form fields (textarea/input, not decorative divs).
9. **Iframe height-sender script required** in all Kajabi/GitHub Pages HTML files: postMessage with id, ResizeObserver, load/resize listeners, before `</body>`. Plus `target="_top"` on every internal/same-domain link; external links use `target="_blank" rel="noopener"`.

---

## 5. Color and font system

Defined in the generator as `COLORS` and used uniformly:

```javascript
const COLORS = {
  navy: "1E3D4C",          // primary heading color
  navySoft: "335E6E",
  terra: "7C2018",          // accent / Section A module headings / day banner caps
  terraLight: "A0402A",
  gold: "C9A961",           // accent rules and day banner left borders
  goldSoft: "E5D5A8",
  seaGlass: "8FB8BF",
  seaGlassMist: "DCE8EB",
  ink: "15262F",            // body text
  inkSoft: "3D4F5A",        // italic captions and intros
  divider: "BFC8CD",        // table borders
  white: "FFFFFF",
  ruleGray: "9CA3AF",       // answer-line bottom borders
};
```

Body font: Arial throughout. Sizes (in half-points, the docx convention):
- Heading 1: 36 (large title)
- Heading 2 (Section heading): 28, bold, navy, gold bottom border
- Heading 3 (module heading): 24, bold, terra
- Heading 4 (sub-heading): 22, bold, navy
- Body: 22, ink
- Caption / italic intro: 22, italic, inkSoft
- Footer / header: 16

---

## 6. The eight cases (locked diagnoses, faculty-only)

These are NOT shown to students in the workbook. They live in the data file as `diagnosis_internal` and drive the case lede, demographic blurb, clue progression, and reveal anchor (which is shown in the HTML page after the week is done, not in the workbook).

| Wk | Case (faculty label) | Anatomy focus |
|---|---|---|
| 1 | TEN (toxic epidermal necrolysis) | Integumentary |
| 2 | OI (osteogenesis imperfecta) | Bone microanatomy + axial |
| 3 | Marfan | Appendicular + joints + muscle as a system |
| 4 | ToF (Tetralogy of Fallot) | Heart + great vessels + fetal circ + vessel histology + thorax/back/UE muscles + thoracic/UE vessels |
| 5 | Sickle cell | Respiratory + lymph + immune + blood + pulmonary vessels |
| 6 | Hirschsprung | Digestive + abdomen/pelvis/LE muscles + abdominal/pelvic/LE vessels |
| 7 | ADPKD | Urinary + reproductive + endocrine + renal/gonadal vessels |
| 8 | NPH (normal pressure hydrocephalus) | Nervous (CNS + PNS + ANS) + head/neck/face/eye muscles + cranial vessels |

Case ledes shown to students are intentionally generic and identical across all 8 weeks: "Your case unfolds Monday through Thursday as new anatomic clues are revealed in class. By Thursday's TBL, your team will investigate a five-option differential and defend your number 1 choice using anatomic reasoning." (Adjust "five-option" to match the actual count for that week.)

Sanitization rule: any topic label that overlaps with the locked diagnosis (e.g., "polycystin distribution" for Wk 7 ADPKD, "neural crest migration" for Wk 6 Hirschsprung) must be rephrased to use general anatomy terminology only ("tubular protein distribution", "embryologic enteric nervous system development") so the workbook does not point at the answer.

---

## 7. The 8-week schedule (locked)

```
Wk 1: Foundations + Integumentary                              | Case 1 (TEN)        | No lab exam Mon
Wk 2: Bone microanatomy + Axial skeleton                       | Case 2 (OI)         | LE 1 Mon (Wk 1 content)
Wk 3: Appendicular + Joints + Muscle as a system               | Case 3 (Marfan)     | LE 2 Mon (Wk 2 content)
Wk 4: Heart + great vessels + fetal circ + vessel histology    | Case 4 (ToF)        | LE 3 Mon (Wk 3 content)
       + thorax/back/UE muscles + thoracic/UE vessels
Wk 5: Resp + Lymph + Immune + Blood + pulmonary vessels        | Case 5 (Sickle)     | LE 4 Mon (Wk 4 content)
Wk 6: Digestive + abdominal/pelvic/LE muscles                  | Case 6 (Hirschsprung) | LE 5 Mon (Wk 5 content)
       + abdominal/pelvic/LE vessels
Wk 7: Urinary + Reproductive + Endocrine + renal/gonadal vsls  | Case 7 (ADPKD)      | LE 6 Mon (Wk 6 content)
Wk 8: Nervous CNS+PNS+ANS + head/neck/face/eye muscles         | Case 8 (NPH)        | LE 7 Mon (Wk 7 content)
       + cranial vessels                                                              | LE 8 cumulative Thu PM
```

Lab exams: 50 minutes, 25 stations, 2 questions per station = 50 questions total. 80% current week + 20% spiral review. Held first thing Monday morning before lecture.

Daily class structure (Mon-Thu): morning lecture, afternoon lab. Thursday is TBL session in lieu of lecture.

---

## 8. The week_N_modules.py file format

One file per week. Same shape across all 8.

```python
"""Week N foundation research modules.
Each module corresponds to one curriculum learning objective area.
Question scaffolding: define -> name -> locate -> relate -> compare -> apply.

Module types:
- 'questions' = numbered research questions with answer space
- 'muscle_table' = IOA table per muscle
"""

WEEK_N_MODULES = [
    {
        "title": "A1. <Topic title>",
        "intro": "<One-to-two-sentence orientation paragraph in italic gray.>",
        "type": "questions",
        "questions": [
            "<Question 1 text>",
            "<Question 2 text>",
            ...
        ],
        "atlas_label": "Atlas pages for <topic>",   # kept for backward compat, not rendered
    },
    {
        "title": "A12. Muscles of <region>",
        "intro": "<orientation>",
        "type": "muscle_table",
        "muscles": [
            "<Muscle 1>",
            "<Muscle 2>",
            ...
        ],
        "follow_up": [   # optional
            "<Follow-up research question 1>",
            "<Follow-up research question 2>",
        ],
        "atlas_label": "Atlas pages for <region> muscles",
    },
    # ... and so on
]
```

Module title format: `"A<N>. <Title>"` where N is the module number, used by the generator to assign day banners.

Question writing principles:
- **Open with definitions or naming** — "Name the four chambers..." "Define the pericardial cavity..."
- **Move to identification** — "Identify which two chambers receive blood..."
- **Push to relationships** — "Trace blood flow from systemic veins..."
- **End with comparison or application** — "Anatomically distinguish AV valves from semilunar valves." "Why does brachioradialis flex the elbow despite being innervated by the radial nerve?"
- Avoid yes/no questions. Prefer "Name", "Identify", "Locate", "Trace", "Distinguish", "Compare", "Explain".
- Avoid leaking the diagnosis. Topic labels and question wording stay general anatomy ("connective tissue distribution" not "Marfan-affected tissues").
- Use "Cross-reference Section A modules X to Y" inside Section D lab prep blocks, not inside Section A questions.

---

## 9. Day banner module boundaries

Each week's modules are partitioned into four nights. The generator inserts a day banner (gold-bordered cream callout) before the first module of each night.

For Week 4 the partition is:
```javascript
const dayBoundaries = {
  1: ["Sunday night", "Begin Section A here. Aim to complete A1 to A4 tonight."],
  5: ["Monday night", "Continue Section A here. Aim to complete A5 to A8 tonight."],
  9: ["Tuesday night", "Continue Section A here. Aim to complete A9 to A11 tonight."],
  12: ["Wednesday night", "Continue Section A here. Aim to complete A12 to A19 tonight."],
};
```

Per-week boundaries are defined in `WEEK_DAY_BOUNDARIES` in the generator. Computing boundaries: aim for a balanced module-count distribution across the four nights, with simpler concept modules earlier and muscle IOA tables (which take longer) later. Wednesday night typically gets the muscle-heavy load.

---

## 10. The differential ranking table (Section D synthesis)

A 3-column table appears under the synthesis question that asks students to rate options. Generated programmatically from the case differential array. One row per option:

```
| Differential option         | Support | Anatomic reasoning |
|-----------------------------|---------|---------------------|
| 1. <Option name>            |         |                     |
| 2. <Option name>            |         |                     |
| ...                         |         |                     |
```

The table appears three times in Section D — once each in Mon→Tue, Tue→Wed, Wed→Thu nights — letting students see how their ratings evolve across the week. The table auto-sizes to however many options are in the differential (5 to 7 typically).

---

## 11. The Other exploration block (Section D)

Appears at the end of every night's page in Section D. Four labeled prompt areas with answer-line space:

```
Other exploration tonight
  (italic intro)

Related pathophysiology, normal variants, or other disorders worth knowing:
  ___________________
  ___________________
  ___________________

Lab values that might be informative for the differential:
  ___________________
  ___________________
  ___________________

Physical exam findings, imaging, or radiology to recognize:
  ___________________
  ___________________
  ___________________

Free notes / questions / connections / curiosities:
  ___________________ × 6
```

This is a reusable block — the same component appears in all four nights.

---

## 12. The lab prep block (Section D)

Appears between "Aim to complete tonight" and the synthesis questions on each night.

Structure (Mon→Tue example):

```
Lab prep tonight
  (italic intro: 75% pre-study tomorrow, 25% review today/spiral)

Pre-study for tomorrow's lab (75%): <Tomorrow's specific lab content>
Cross-reference: <Section A modules>
List the structures you need to identify tomorrow, then practice naming and locating each in your atlas.
  ___________________ × 5

Review today's lab and anything fuzzy from earlier this week (25%): <Today's lab content>
Cross-reference: <Section A modules>. Quick cold recall: cover any notes or tables and re-test yourself.
  ___________________ × 3
```

Per-week lab content data lives in a `WEEK_N_LAB_CONTENT` object in the generator with four entries (sun_to_mon, mon_to_tue, tue_to_wed, wed_to_thu), each specifying today's brief, today's section cross-reference, tomorrow's brief, tomorrow's section cross-reference, and an `isFirst` flag (true only for sun_to_mon to handle the no-prior-lab case).

---

## 13. Generator architecture

The DOCX generator (`generate_docx.js`) is roughly 1500 lines. Sections (in source order):

1. **Imports and setup** — docx package imports, weeks.json load, output dir
2. **Color and helper definitions** — COLORS, p(), multiRunPara(), h2(), h3(), h4(), bullet(), numbered(), spacer(), answerLines()
3. **Day banner helper** — `dayBanner(day, instruction)` — single-cell table with gold left border, terra caps, cream background
4. **Workbook plan TOC** — `buildWorkbookPlan()` — 6-row section/what/when table
5. **Title block** — `buildTitleBlock(week)`
6. **How to use** — `buildHowToUse()`
7. **Week at a glance** — `buildAtAGlanceTable(week)`
8. **Lab exam block** — `buildLabExamSection(week)` (only weeks 2-8)
9. **Section A: foundation research**:
   - `buildQuestionModule(module, idx)` — handles 'questions' type
   - `buildMuscleTableModule(module)` — handles 'muscle_table' type with 5-column IOA table
   - `buildFoundationResearchSection(modules, weekNum)` — orchestrates, inserts day banners at correct module numbers
10. **Section B: differential research** — `buildDifferentialResearchSection(week)` — generates 5-7 option research blocks
11. **Section C: clue tracker** — `buildClueTrackerSection(week)` — 4 blank fields (Mon/Tue/Wed in-class + Thu reveal)
12. **Section D: nightly synthesis**:
    - `diffRankingTable(week)` — generates 3-column ranking table from case differential
    - `otherExplorationBlock()` — reusable Other exploration block
    - `labPrepBlock(weekNum, nightKey)` — generates 75/25 lab prep block from WEEK_N_LAB_CONTENT
    - `buildPreworkSection(week, weekNum)` — orchestrates the four nights, page-breaks between
13. **Section E: Thursday TBL checklist** — `buildThursdayChecklist(week)`
14. **Section F: lab exam prep + weekend** — `buildWeekendBlock(week, weekNum)`
15. **Document assembly** — `buildWeekDoc(week, weekNum, modules)` puts it all together with PageBreaks between sections
16. **Main loop** — iterate over WEEKS, load modules, generate, write to output dir

The Node generator imports Python module data via:

```javascript
const { execSync } = require("child_process");
const modulesJson = execSync(
  `python3 -c "import sys, json; sys.path.insert(0, '/home/claude/weeks_build'); from week${weekNum}_modules import WEEK${weekNum}_MODULES; print(json.dumps(WEEK${weekNum}_MODULES))"`,
  { encoding: "utf8" }
);
const modules = JSON.parse(modulesJson);
```

---

## 14. Build sequence for a fresh course

If adapting this system to a new course (e.g., BIO 431 A&P, BIO 304 online A&P, or any future case-based anatomy course), follow this sequence:

### Step 1: Lock the schedule
- Decide the number of weeks
- Map the curriculum to weeks (use the same regional muscle/vessel distribution principle if anatomy)
- Pick one case per week, lock the diagnoses (faculty-only)
- Decide the lab exam structure and cadence
- Document the schedule in a single table

### Step 2: Sanitize case ledes and topic labels
- Write generic case ledes that don't hint at the diagnosis
- Audit every topic label and question prompt for diagnosis-leaking phrasing; rephrase with general anatomy terminology

### Step 3: Build the data files (Python)
- Create `data_p1.py` and `data_p2.py` (or just one file if fewer weeks) with one entry per week:
  - module label and title, dates, lede, demographic blurb
  - days array (one entry per Mon/Tue/Wed/Thu) with role, lecture_items, lab_items, came_in, tonight_review, tonight_organize
  - case dict with diagnosis_internal, lede, clue_mon, clue_tue, clue_wed, thu_final, reveal_anchor, differential array (each with name, layer, discriminator)
  - has_lab_exam flag, lab_exam dict
  - weekend dict with title, dates, lede, blocks (3 blocks: rest, consolidate, prime)

### Step 4: Build week_N_modules.py for each week
- One file per week, named `week_1_modules.py` through `week_N_modules.py`
- Apply the question scaffolding pattern (define → name → locate → relate → compare → apply)
- Use 'questions' type for concept modules
- Use 'muscle_table' type for muscle ID modules with a list of muscle names
- For each muscle module, optionally include 'follow_up' research questions
- Aim for 10 to 22 modules per week depending on content density

### Step 5: Define WEEK_N_LAB_CONTENT in the generator
- For each week, specify what each night's "today" and "tomorrow" lab content is, with cross-references to specific Section A modules
- Wk 1's sun_to_mon has `isFirst: true` because there's no prior round table

### Step 6: Define WEEK_N_DAY_BOUNDARIES in the generator
- Partition each week's modules into four nights
- Map the first module number of each night to a day banner instruction string

### Step 7: Write export_json.py
- Reads data files, computes derived fields (e.g., section labels, demographic_blurb fallbacks), exports weeks.json

### Step 8: Write generate_docx.js
- Use the structure described in Section 13 of this document
- All section builders are pure functions returning arrays of Paragraph and Table objects
- Document assembly composes them with PageBreaks where needed

### Step 9: Validate the first prototype
- Generate one week (preferably a complex middle week, like Wk 4 in this build) as the prototype
- Check page count, paragraph count, that all sections render
- Run docx schema validation
- Render to PDF and visually check 5-8 representative pages: title, Section A question module, IOA muscle table, Section B differential research, Section C clue tracker (must be empty), Section D synthesis page with ranking table and Other exploration, Section F weekend study plan
- Verify NO case clues leak into Section A or B
- Verify NO discriminators leak into Section B (only option names show)

### Step 10: Iterate the prototype
- Show the prototype to the instructor
- Adjust modules based on feedback (drop sections, expand sections, rephrase questions)
- Adjust day boundaries if pacing is off
- Adjust Section F based on the actual lab exam structure

### Step 11: Bulk-generate all weeks
- Once the prototype is locked, generate all weeks
- Re-validate each
- Render PDFs and spot-check
- Final leak audit: for each week, confirm zero case clues and zero discriminators in the workbook

### Step 12: Generate the HTML weekly pages
- One page per week, structured around: title, case lede, week at a glance, day-by-day breakdown, differential card with option names ONLY (no discriminators), reveal_anchor in a collapsible "after the week" section
- Bake in iframe height-sender script
- Add target="_top" on internal links, target="_blank" rel="noopener" on external
- Verify WCAG 2.2 AA compliance

### Step 13: Generate schedule.html
- Term-level overview table: Week | Anatomy focus | Case | Lab exam (Mon)
- Bake in iframe height-sender script and link rules

---

## 15. Locked design decisions (do not relitigate without explicit instructor approval)

These were settled across the iterative build and should NOT be reopened in future sessions without the instructor explicitly raising them:

- 8-week format
- 1 case per week (8 cases total)
- TBL Thursday + lab exam Monday cadence
- 50 min × 25 stations × 2 questions = 50 questions per lab exam
- 80% current week + 20% spiral review on lab exams
- Question scaffolding pattern: define → name → locate → relate → compare → apply
- Innervation column on muscle IOA tables
- No intrinsic hand muscles (Wk 4) and no intrinsic foot muscles (Wk 6)
- No atlas page references in workbooks (atlas being recreated; future project)
- Discriminators stripped from public HTML weekly pages (kept in data for future facilitator manual)
- Case ledes are generic and identical across weeks
- No em dashes anywhere; no smart quotes; no purple
- "Dr. Sharilyn Rennie" byline only, no credential suffix
- Color and font system as described in Section 5

---

## 16. File outputs

After a complete build, the system produces:

- `/mnt/user-data/outputs/week-1-study-guide.docx` through `/mnt/user-data/outputs/week-8-study-guide.docx` (8 files)
- `/mnt/user-data/outputs/week-1.html` through `/mnt/user-data/outputs/week-8.html` (8 files)
- `/mnt/user-data/outputs/schedule.html` (1 file)

All deliverables are intended for upload to:
- DOCX workbooks → Kajabi as homework bundle attachments per week
- HTML weekly pages → drsrennie-stack/Solano-Anatomy- repo on GitHub Pages, embedded via iframe in Kajabi
- schedule.html → same

---

## 17. Recovery checklist if resuming from this document

If a chat is cut off and you need to resume:

1. **Check what's already built** — look in `/mnt/user-data/outputs` for existing DOCX and HTML files, and in `/home/claude/weeks_build` for data and module files
2. **Read this document** in full to understand the system
3. **Re-read user memories and the journal at /mnt/transcripts/journal.txt** for the latest state of decisions and what's been built
4. **Validate the existing prototype** (the most recent week-N-study-guide.docx) — render to PDF, confirm structure matches Section 3 of this document
5. **Identify what's missing** — typically: some week's modules not yet built, HTML pages not regenerated, schedule.html not updated
6. **Resume from the build sequence in Section 14**, jumping to whatever step is incomplete

---

## 18. Adapting to other classes (BIO 431, BIO 304, future courses)

Same architecture, different content. Specific adaptation notes:

### BIO 431 Human Anatomy and Physiology
- Likely 16-week format instead of 8
- Detailed mechanisms ARE in scope (action potentials, hormone cascades, ventilation mechanics, etc.) — do NOT cap at "primary functions"
- Question scaffolding extends to mechanism: define → name → locate → relate → compare → apply → predict mechanism
- Cases can be the same (TEN, OI, Marfan, ToF, etc.) or expanded — 16 weeks supports 16 cases
- Lab exam structure may differ (check with instructor)

### BIO 304 Online Human Anatomy and Physiology
- Online format means NO in-person lab, NO in-person TBL, NO live case clues
- Section C (clue tracker) becomes obsolete or restructured: clues released asynchronously through LMS
- Section D synthesis still works but adapted for asynchronous learning
- Section F lab exam prep changes: no proctored lab exam, may be replaced with self-paced quizzes or projects
- Critical rule: **no references to TBL, Thursday sessions, in-person team activities, or in-person lab** anywhere in BIO 304 materials
- Workbook may be shorter and more self-paced, with clearer time estimates

### Other case-based anatomy courses (future)
- Use this document as a starting template
- Lock the schedule first, then build out the data and modules
- Maintain the question scaffolding pattern as the core pedagogy
- Maintain the no-clue-leak, no-discriminator-leak rules in workbooks
- Strip discriminators from public-facing HTML pages
- Build a separate facilitator manual with discriminators and reveal anchors when the time comes

---

## 19. Contact and ownership

This system was developed iteratively in collaboration with Dr. Sharilyn Rennie for BIO 004 Human Anatomy at Solano Community College. All locked design decisions in Section 15 are owned by Dr. Rennie. The architecture and code patterns may be reused or adapted for other courses owned by Dr. Rennie or MedMasters Collaborative, LLC.

End of document.

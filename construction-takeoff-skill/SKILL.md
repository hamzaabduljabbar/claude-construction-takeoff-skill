---
name: construction-takeoff
description: Run a quantity takeoff on a construction drawing set that has already been processed by the Drawing Analyzer skill. Identifies every countable or measurable element, marks them up on the drawings for audit, calculates primary and secondary quantities (excavation volume, concrete volume, formwork area, reinforcement weight), and exports the results to an Excel workbook with confidence levels. Trigger on phrases like "run a takeoff", "quantity takeoff", "use my construction takeoff skill", or any request to extract quantities from a drawing set.
---

# Construction Takeoff Skill

Performs a quantity takeoff on drawings already processed by the Drawing Analyzer.
Produces primary counts, derived secondary quantities, drawing markups for audit, and an Excel export.

This skill requires the Drawing Analyzer skill to have been run first.

---

## Why this exists

The Bluebeam Max Claude integration can only read the text layer of a drawing. It cannot measure slab areas, read line types, or understand anything in the image layer. The Construction Takeoff skill uses the output from the Drawing Analyzer (text layer plus visual analysis already done) to deliver quantity takeoffs that work on both text-tagged and visually-shown elements.

It also marks up the drawings to show what was counted, so the estimator can audit the output before using it in a bid.

---

## Prerequisites

Before running this skill:

1. The Drawing Analyzer skill must have been run on the drawing set
2. The `0.ai-context/` folder must exist with:
   - `drawings.md` (master register)
   - `symbols.md` (legend library)
   - `cross-references.md`
   - `processed/[sheet].md` files (one per sheet)
3. An Excel template must exist at `templates/takeoff-template.xlsx` (or specify another path in the prompt)

If any of these are missing, the skill stops and tells the user what to run first.

---

## Workflow

### Phase 1: Confirm scope before processing

Ask the user before starting:

```
TAKEOFF SCOPE
Drawing set:    [Project name] ([N] sheets indexed)
Discipline:     [Structural / Architectural / Mechanical / Electrical / Civil / All]
Element types:  [List specific element types or "all elements"]
Output file:    [outputs/[project]-takeoff.xlsx]

Confirm scope before I start?
```

Do not proceed until the user confirms.

### Phase 2: Identify countable elements

Read the relevant sheet summaries from `0.ai-context/processed/`.

For each sheet in the scope, identify every element that can be counted or measured:

**Countable elements (EA):**
- Tagged footings (F1, F2, etc.)
- Columns (C1, C2, etc.)
- Beams identified in schedules
- Mechanical equipment, fixtures, fittings
- Electrical fittings (light fittings, GPOs, switches)

**Measurable elements:**
- Slab areas (m2) from dimensions
- Beam lengths (LM) from grid spans
- Wall lengths (LM)
- Pipe runs (LM, where shown on plan)

**Schedules:**
- Read any element schedule found on the drawings
- Cross-reference schedule items with elements shown on plan

### Phase 3: Build primary quantities

For every element identified, record:

| Field | Content |
|-------|---------|
| Element type | Plain language description |
| Tag | The tag as shown on the drawing (F1, C2, GB1) |
| Source sheet | Which drawing it came from |
| Source method | Tag count / Schedule entry / Visual count / Dimension read |
| Primary qty | Number, area, or length |
| Unit | EA / m / m2 / m3 / LM |
| Confidence | HIGH / MEDIUM / LOW |

### Phase 4: Calculate secondary quantities

For each primary element, calculate the derived secondary quantities using standard ratios.
The ratios come from business context if available, otherwise from these defaults.

**Pad footing (per footing):**
| Secondary | Calculation |
|-----------|-------------|
| Excavation volume | L × W × (D + 100mm working space) |
| Concrete volume | L × W × D |
| Formwork area | (L × 2 + W × 2) × D |
| Reinforcement weight | Use bar schedule if shown, otherwise 100 kg/m3 of concrete |

**Strip footing (per LM):**
| Secondary | Calculation |
|-----------|-------------|
| Excavation volume | W × (D + 100mm) × 1m |
| Concrete volume | W × D × 1m |
| Formwork area | D × 2 × 1m |
| Reinforcement weight | Use bar schedule or 80 kg/m3 |

**Suspended slab (per m2):**
| Secondary | Calculation |
|-----------|-------------|
| Concrete volume | Area × thickness |
| Formwork area | Same as slab area (underside) |
| Edge formwork | Perimeter × slab thickness |
| Reinforcement weight | Use schedule or 100 kg/m3 for slabs, 120 kg/m3 for transfer slabs |

**Column (per column):**
| Secondary | Calculation |
|-----------|-------------|
| Concrete volume | Section × height |
| Formwork area | Perimeter × height |
| Reinforcement weight | Use schedule or 200 kg/m3 |

### Phase 5: Mark up the drawings

For each element counted, write a markup annotation to the rendered PNG of the source sheet.

The markup includes:
- A coloured polygon or circle around each counted element
- A label showing the tag and the count number (e.g. "F1 #3 of 12")
- A summary callout box on the sheet showing total counts for that sheet

Save marked-up images to `outputs/markups/[sheet].png`.

The estimator opens these images to audit what was counted before using the quantities.

### Phase 6: Build the Excel output

Use the template at `templates/takeoff-template.xlsx`.

Populate the columns:

| Col | Header | Content |
|-----|--------|---------|
| A | Item No | Sequential |
| B | Element | Plain language description |
| C | Tag | Drawing tag |
| D | Source sheet | Drawing number |
| E | Source method | How the quantity was determined |
| F | Primary qty | Number |
| G | Unit | EA / m / m2 / m3 / LM |
| H | Confidence | HIGH / MEDIUM / LOW |
| I | Secondary: Excavation (m3) | Calculated |
| J | Secondary: Concrete (m3) | Calculated |
| K | Secondary: Formwork (m2) | Calculated |
| L | Secondary: Reinforcement (kg) | Calculated |
| M | Notes | Manual check flags |

Section headers by element type (FOUNDATIONS, COLUMNS, BEAMS, SLABS).
Subtotals per section.
Grand totals at the bottom.

Save to `outputs/[project]-takeoff.xlsx`.

### Phase 7: Completion report

```
CONSTRUCTION TAKEOFF COMPLETE

Sheets processed:    [N]
Elements counted:    [N]
  HIGH confidence:   [N] ([X]%)
  MEDIUM confidence: [N] ([X]%)
  LOW confidence:    [N] ([X]%)  ← Needs manual verification

Primary quantities:
  Pad footings:      [N] EA
  Strip footings:    [N] m
  Slabs:             [N] m2 totalling [N] m3 concrete
  Columns:           [N] EA
  [other element types as found]

Secondary quantities (totals):
  Total excavation:    [N] m3
  Total concrete:      [N] m3
  Total formwork:      [N] m2
  Total reinforcement: [N] kg

Files saved:
  outputs/[project]-takeoff.xlsx
  outputs/markups/[N] marked-up drawing images

Items requiring manual verification:
  [List LOW confidence items by element type]

Next step:
  Open outputs/markups/ images to audit what was counted.
  Verify LOW confidence items manually against the original drawings.
  Open the Excel file. Add your rates to the Rate column. The estimate calculates.
```

---

## How to handle the Bluebeam text-layer limitation

If the user is asking this skill to do something the Bluebeam Claude integration cannot do (slab areas, line type interpretation, visual element counts), this skill handles it because:

1. The Drawing Analyzer skill has already done a visual analysis of every sheet
2. The visual analysis output is stored in the sheet's .md file
3. This skill reads that output and uses it for visual-only items

For visual-only items (no text tag, identified from image analysis only), mark as MEDIUM confidence and flag for manual verification.

---

## Rules

- This skill must not run if the Drawing Analyzer output is missing. Tell the user to run it first.
- Every quantity must have a confidence level
- Every LOW confidence item must have a note explaining why
- Markups must be saved as PNG so the estimator can review them without opening Bluebeam
- Secondary quantities are calculations only. Do not invent ratios. Use business context or the defaults documented above.
- Reinforcement weights are estimates. The bar schedule on the drawings is authoritative if present.
- Never write rates or totals. The estimator fills those in.

---

## Output format

Excel file in outputs/ folder.
Marked-up drawing images in outputs/markups/ folder.
Completion report in chat.
Tone: factual, number-driven, no narrative padding.

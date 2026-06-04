# construction-takeoff-skill

**Claude skill for construction quantity takeoffs from indexed drawing sets.**

Built by [Hamza Jabbar](https://hamzajabbar.online) | AutoConst

---

## What this is

A Claude skill that runs a quantity takeoff on a construction drawing set, producing primary element counts, calculated secondary quantities (concrete, formwork, reinforcement), marked-up drawing images for audit, and an Excel export with confidence levels.

This skill is the second step in the drawing analysis workflow. It requires the [construction-drawing-analyzer](https://github.com/hamzaabduljabbar/construction-drawing-analyzer) to have run first.

---

## How it fits in the workflow

```
Step 1: construction-drawing-analyzer   ← Run first. Builds indexes/ and sheets/ folders.
              ↓
Step 2: construction-takeoff-skill      ← This repo. Uses indexed data for the takeoff.
              ↓
        Excel file + marked-up drawings
```

---

## What it produces

- **Excel takeoff** — one row per element with primary quantity, unit, confidence level, and calculated secondary quantities (excavation, concrete, formwork, reinforcement)
- **Marked-up drawing images** — PNG files showing exactly what was counted and where, for estimator audit
- **Completion report** — summary of all quantities found, confidence breakdown, and items needing manual check

---

## What this does that Bluebeam Max cannot

| Capability | Bluebeam Max Claude | This skill |
|-----------|---------------------|-----------|
| Count text-tagged elements | Yes | Yes |
| Measure slab areas | No | Yes (via Drawing Analyzer polygon extraction) |
| Calculate secondary quantities | No | Yes |
| Mark up drawings showing what was counted | No | Yes |
| Subscription required | Bluebeam Max | Standard Claude only |

---

## Requirements

- Claude Pro subscription
- [construction-drawing-analyzer](https://github.com/hamzaabduljabbar/construction-drawing-analyzer) run on the drawing set first
- An Excel template at `templates/takeoff-template.xlsx` in your project folder

---

## Installation

### Option A — Install as a Claude skill

1. Download `SKILL.md` from this repo
2. Open the Claude desktop app
3. Go to **Settings > Skills > Upload**
4. Select `SKILL.md`
5. Name the skill: `construction-takeoff`

### Option B — Use with Antigravity

1. Copy `SKILL.md` into your project folder
2. Open the folder in Antigravity
3. Select Claude Sonnet 4.6 or Opus 4.6

---

## Usage

After the Drawing Analyzer has run:

```
Use my construction-takeoff skill.
Scope: foundations only. Page 2 only.
Output to outputs/[project-name]-foundation-takeoff.xlsx
```

For a full structural takeoff:

```
Use my construction-takeoff skill.
Scope: all structural elements across all pages.
Output to outputs/[project-name]-structural-takeoff.xlsx
```

---

## Excel template structure

Build your template once and reuse it. Columns the skill populates:

| Col | Header | Content |
|-----|--------|---------|
| A | Item No | Sequential |
| B | Element | Element type |
| C | Description | Full spec from drawing |
| D | Source Sheet | Drawing number |
| E | Source Method | Tag count / Schedule / Visual / Dimension |
| F | Qty | Number only |
| G | Unit | EA / m / m2 / m3 / LM / kg |
| H | Confidence | HIGH / MEDIUM / LOW |
| I | Excavation (m3) | Calculated |
| J | Concrete (m3) | Calculated |
| K | Formwork (m2) | Calculated |
| L | Reinforcement (kg) | Calculated |
| M | Rate | Your estimator fills in |
| N | Total | =F*M formula |
| O | Notes | Manual check flags |

---

## Secondary quantity defaults

Default ratios for concrete, formwork, and reinforcement calculations are documented in the [construction-drawing-analyzer references folder](https://github.com/hamzaabduljabbar/construction-drawing-analyzer/blob/main/references/structural-secondary-quantities.md).

Override them by adding your firm's actual ratios to your business context.

---

## Related repos

- [construction-drawing-analyzer](https://github.com/hamzaabduljabbar/construction-drawing-analyzer) — Run this first
- [pdf-markup-skill](https://github.com/hamzaabduljabbar/pdf-markup-skill) — Bulk markup operations

---

## License

MIT

---

## Author

**Hamza Jabbar** | [hamzajabbar.online](https://hamzajabbar.online)
Building AI automations for construction firms at AutoConst.

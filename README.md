# FIT3179 — Assignment 2: Australian Mental Health Data Visualisation

## How to open locally (quick start)

```bash
# 1) Open a terminal in the project folder
# Windows PowerShell example:
cd C:\Users\User\Desktop\FIT3179\FIT3179-Assignment2

# 2) Start a local static server (Python)
py -m http.server 5500

# 3) Open in your browser
http://localhost:5500/
```

Notes:
- Ensure `index.html`, `combined_health.csv`, and `au_admin_1_states_provinces.json` stay in the project root as they are referenced by relative URLs.
- If port 5500 is busy, use a different port (e.g., `py -m http.server 8000`) and open `http://localhost:8000/`.
- A 404 for `/favicon.ico` is harmless.

---

## Project overview
This single-page site (`index.html`) presents interactive visualisations exploring Australia’s mental health landscape (2020–2022), using Vega-Lite rendered via `vega-embed`.

- HTML/CSS is self-contained in `index.html`.
- Data files are loaded at runtime from CSV/TopoJSON in the same directory.
- No build step or external code is required beyond the Vega/Vega-Lite/vega-embed CDNs.

### Libraries
- Vega `v5`
- Vega-Lite `v5`
- vega-embed `v6`

CDN scripts are included in the `<head>` to enable inline JSON specs.

---

## File structure
- `index.html`: The full page, styling, and all Vega-Lite specs.
- `combined_health.csv`: Aggregated data table used by all charts (various `dataset` values).
- `au_admin_1_states_provinces.json`: TopoJSON for Australian Admin-1 boundaries used by the map.

Optional CSVs (sources behind the combined file) are present under `csv/` but not directly requested by the page.

---

## Code walkthrough (index.html)

### Styles and layout
- Defines a warm, readable theme: soft background gradient, shadowed white cards, and a centered content wrapper.
- Reusable classes: `wrap`, `card`, `story-section`, `insights-grid`, `insight-item`, `meta`.
- Accessibility/UX touches: clear headings, readable contrast, consistent legends, and subtle emphasis colors.

### Narrative content
- Introductory “story section” to frame the purpose and context of the data.
- Concluding “insights” section summarising geographic variation, gender differences, disorder prevalence, and policy implications.

### Visualisations
All charts are declared as inline Vega-Lite JSON specs and embedded with `vegaEmbed(selector, spec, { actions:false })`.

1) Map — Mental & Behavioural Conditions by State (2022)
   - Data: `au_admin_1_states_provinces.json` (TopoJSON) joined with `combined_health.csv`.
   - Projection: Albers; includes ocean fill and graticules for geographic context.
   - Color encodes rate per 1,000 residents using a soft red palette. A ranked transform annotates the state with the highest rate.
   - Tooltips: state, total cases, population, rate per 1,000.

2) Grouped Bars — Anxiety Disorders by Sex (2020–2022)
   - Data filtered by `dataset==='anxiety_by_sex'`.
   - Param `disSelect` (dropdown) to focus on a single anxiety category or show All.
   - Legend selection filters by `sex` interactively.
   - Annotations: highest total across categories when All is selected; label for the selected category otherwise.

3) Horizontal Bars — Affective Disorders by Sex (2020–2022)
   - Data filtered by `dataset==='affective_by_sex'`.
   - Param `affectiveSelect` with legend filtering by `sex`.
   - Horizontal layout improves label readability; annotation marks the highest total.

4) Area Chart — Substance Use Disorders by Sex (2020–2022)
   - Data filtered by `dataset==='substance_by_sex'`.
   - Legend selection filters by `sex`; semi-transparent overlays show relative magnitudes.
   - Annotation identifies the highest total category across sexes.

5) Donut Chart — Lifetime Mental Disorders Summary by Sex (2020–2022)
   - Data filtered by `dataset==='lifetime_by_sex'` and excludes the `Total` group.
   - Param `lifetimeSelect` allows focusing on “Any” vs “No” lifetime mental disorder.
   - A right-side callout succinctly communicates key comparison takeaways.

### Data handling
- The combined CSV contains multiple logical datasets separated by a `dataset` field (e.g., `anxiety_by_sex`, `affective_by_sex`, `substance_by_sex`, `lifetime_by_sex`, `mental_behavioural_by_state`).
- Each chart filters its relevant subset with a `transform.filter` and converts `value` fields using `toNumber` before aggregation or encoding.
- The map computes `Rate_per_1000` using a safe fallback: if the provided `rate_per_1000` is invalid, it derives from `persons/population*1000`.

---

## Design decisions and rationale
- Visual grammar: Vega-Lite chosen for declarative, concise, and reproducible visualisations with strong interactivity support.
- Color choices: soft blues/oranges for gender separation; soft reds for choropleth intensity, keeping readability and color-blind awareness in mind.
- Interaction model: lightweight, discoverable interactions (legend filtering, dropdown focus) rather than complex multi-step dashboards to keep cognitive load low.
- Annotations: programmatic highlighting of extremes and selection-dependent notes to guide attention without cluttering visual space.
- Layout and storytelling: cards and sections guide readers through context → exploration → insights.
- Performance: single HTML file with CDN libraries, no framework overhead; CSV and TopoJSON are small and load quickly.
- Accessibility: clear labels, formatted tooltips, readable font sizes, and restrained color contrast.

---

## Interview talking points
- Data pipeline: explain how `combined_health.csv` aggregates source tables and exposes a `dataset` column for chart-specific filtering.
- Join logic in the map: TopoJSON features are filtered to AU states/territories, then joined to CSV by `properties.name` ↔ `group`.
- Interactivity params: `disSelect`, `affectiveSelect`, `lifetimeSelect` as top-level `params` bound to DOM inputs; legend-driven selections for `sex`.
- Ranking-based annotations: `aggregate` + `window(rank)` to compute highlights like “highest”/“lowest”.
- Robustness: `toNumber` and `isValid` filters; derived `Rate_per_1000` fallback; excluding `Total` where composition charts would be distorted.
- Deployment simplicity: any static host works (GitHub Pages, Netlify). No build step.

---

## Week 12 Studio Interview (Hurdle) — Demo Guide

What to bring
- Laptop with this project folder, including `index.html`, `combined_health.csv`, `au_admin_1_states_provinces.json`, and the original source CSVs under `csv/`.
- Internet connection for CDN scripts (or have offline copies of Vega, Vega-Lite, and vega-embed if needed).

How to run (in session)
- Start the local server (see "How to open locally").
- Open `http://localhost:5500/` and keep `index.html` and `combined_health.csv` visible in your editor for reference.

One construction step to demonstrate (choose one)
1) Map: join TopoJSON to CSV and derive rates
   - Explain: Filter AU states/territories; join to CSV by name; compute a fallback rate.
   - Snippet:
```json
"transform": [
  { "filter": { "field": "properties.name", "oneOf": [
    "New South Wales","Victoria","Queensland","South Australia",
    "Western Australia","Tasmania","Northern Territory","Australian Capital Territory"
  ]}},
  { "lookup": "properties.name",
    "from": { "data": { "url": "combined_health.csv" },
               "key": "group", "fields": ["persons","population","rate_per_1000"] } },
  { "calculate": "isValid(datum.rate_per_1000) ? datum.rate_per_1000 : (datum.persons/datum.population)*1000",
    "as": "Rate_per_1000" }
]
```

2) Grouped bars: clean data, add interactions, annotate
   - Explain: Filter dataset subset; convert numbers; guard invalids; add dropdown and legend selection; compute highest.
   - Snippet:
```json
"params": [{ "name": "disSelect", "value": "All", "bind": { "input": "select", "options": ["All", "Panic Disorder", "Agoraphobia", "Social Phobia", "Generalised Anxiety Disorder", "Obsessive-Compulsive Disorder", "Post-Traumatic Stress Disorder", "Any Anxiety disorder"] } }],
"transform": [
  { "filter": "datum.dataset==='anxiety_by_sex'" },
  { "calculate": "toNumber(datum.value)", "as": "value" },
  { "filter": "isValid(datum.value)" }
]
```

3) Donut: composition and filtering
   - Explain: Exclude `Total` to avoid double counting; optional param to focus on a category.
   - Snippet:
```json
"transform": [
  { "filter": "datum.dataset==='lifetime_by_sex'" },
  { "calculate": "toNumber(datum.value)", "as": "value" },
  { "filter": "isValid(datum.value)" },
  { "filter": "datum.group !== 'Total'" }
]
```

Tips for passing the hurdle
- Show the relevant rows in `combined_health.csv` that drive the chart you’re demonstrating.
- Talk through why the chosen idiom supports the user task (comparison, composition, or spatial pattern).
- Point out robustness features (`toNumber`, `isValid`, derived rate) and how they keep visuals consistent.

---

## Troubleshooting
- Blank charts: Ensure you are opening via a web server (not double-clicking the file). Use the quick start above.
- 404 errors:
  - `/favicon.ico`: harmless; you can ignore it.
  - Data files: confirm `combined_health.csv` and `au_admin_1_states_provinces.json` exist at the project root and the filenames match exactly.
- Different port: if `5500` is busy, try `8000` or `5173` and update the URL.

---

## Credits
- Data sources cited in-page (ABS releases). Five design sheet linked in the footer for design ideation and justification.

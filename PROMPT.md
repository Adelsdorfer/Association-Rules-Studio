# Reconstruction Prompt For Codex 5.5 / Claude Opus 4.8

You are an expert coding model. Recreate a complete local web application project called
**Association Rules Studio** (version **1.1**) that reproduces the specification below **exactly**.

This prompt is the **only** source of truth. You have **no reference implementation, repository,
or screenshot** to consult — the finished app must be reconstructable *from this prompt alone*.
Treat every name, number, color, default value, formula, label, and ordering in this document as a
**hard requirement**, not a suggestion. Where this prose and the **authoritative appendices**
(Appendix A–N near the end of this document) describe the same thing, the appendices give the exact
values and take precedence. Do not invent, round, simplify, or "improve" anything that is specified.

This prompt is designed for **Codex 5.5** or **Claude Opus 4.8**. Assume you are working in
an existing local folder and should directly create or update files rather than writing a
high-level plan only.

Your goal is to reproduce the **exact resulting project**, not merely a simplified prototype.
Pay close attention to:

- exact functionality
- exact file structure
- UX details
- styling direction
- browser behavior
- Safari/WebKit constraints
- documentation parity

If a detail below appears unusually specific, keep it. Those details are intentional.

## Primary objective

Build a **single-file, fully client-side HTML application** for **market-basket /
association-rule mining** on Excel data. The app must:

- read `.xlsx` / `.xls`
- mine frequent itemsets with **Apriori**
- generate association rules in plain JavaScript
- show results in:
  - a **Console** tab
  - a **Table** tab
  - a **Graph & Top 20** tab
- run locally with **no server requirement**
- use **no build step**
- use **no framework**
- use **no mlxtend**

This is a browser port of a Python GUI reference file called
`AssociationRulesGUI.py`, but the web app must implement Apriori and rule generation
directly in JavaScript.

## Required project structure

Create or update these files:

- `index.html`
- `README.md`
- `AGENTS.md`
- `DESIGN.md`
- `DESCRIPTION.md`

Assume these local vendored files exist next to `index.html` and must be used instead of
package-manager dependencies:

- `d3.v7.min.js`
- `xlsx.full.min.js`

Also assume there is a sample workbook:

- `Arbeitsdatei-Quelle.xlsx`

Also include this example share/export artifact in the project:

- `association-rule-filter-presets.json`

Do not introduce build tooling, package manifests, framework scaffolding, or a backend.

## Non-negotiable architecture

### Single-file app

`index.html` must contain:

- HTML markup
- CSS in a `<style>` block
- JavaScript in a `<script>` block

No splitting into extra app source files unless absolutely necessary for static assets. The
core app must stay in one file.

### No framework

Use:

- plain DOM APIs
- D3.js for SVG/table/chart work
- SheetJS for Excel read/write

No React, Vue, Svelte, Angular, Vite, webpack, npm, etc.

### Offline-first

The app must work from:

- `file://.../index.html`
- or a simple static server like `python3 -m http.server`

## Functional scope

### 1. Input handling

The app reads the **first worksheet only**.

Read with the equivalent of:

```js
XLSX.utils.sheet_to_json(sheet, { header: 1, defval: null, raw: true })
```

Interpret columns **positionally**, not by header name:

1. transaction ID
2. item / material
3. order number
4. consumption (optional)
5. price (optional)

Requirements:

- at least 3 columns
- at least 2 rows total
- ignore empty rows
- trim items
- de-duplicate items within each transaction

If price and consumption are both available and consumption is greater than zero, derive a
unit price as:

```text
unit price = total price / consumption
```

### 2. Sidebar controls

Create a left sidebar (`<aside class="sidebar">`) with a collapse/expand toggle button
(`#sidebarToggleBtn`, glyph `<` when expanded and `>` when collapsed) followed by a
`.sidebar-content` wrapper containing, in this order:

- a **brand block**: eyebrow `Apriori Workbench`, `<h1>Association Rules Studio</h1>`, then a
  `.brand-meta` row holding a version pill (`#appVersion`, text `v1.1`) and a **GitHub pill**
  (`#githubLink`) linking to `https://github.com/Adelsdorfer/Association-Rules-Studio`
  (`target="_blank"`, `rel="noopener noreferrer"`, an inline GitHub-mark SVG plus the label `GitHub`)
- input file picker and output file name input
- analysis thresholds
- text filters
- filter preset manager
- `Run analysis` (primary) and `Export all` (warm, initially disabled) action buttons
- a subtle sidebar credit line (`#sidebarCopyright`) showing the version above the Roland Emrich contact

Required labels and defaults:

- `Output file name` default: `SPC_Rules.xlsx`
- `Min support` default: `0.003`
- `Min confidence` default: `0.10`
- `Max. itemset size` default: `4`
- `Table limit` default: `5000`
- checkbox `Include consumption = 0` default: enabled

Filter preset section must include:

- dropdown `Saved filters`
- `Save filter`
- `Delete filter`
- `Export JSON`
- `Import JSON`

Use a hidden file input for JSON import.

### 3. Text filter behavior

Before analysis, support:

- `Only items containing terms`
- `Exclude items`

Terms are:

- comma-separated
- case-insensitive
- trimmed

Important validation rule:

- the include field must contain either **zero terms** or **at least two terms**
- exactly one include term must abort analysis with a clear error

The same include/exclude terms must also be applied to the generated rules after analysis
when filtering the result set.

### 4. Core Apriori implementation

Implement Apriori from scratch in JavaScript.

Expected structure:

- preprocess rows into transactions
- count singleton itemsets
- generate higher-order candidates using classic **join + prune**
- stop at `maxItemsetSize`
- compute supports and counts

Use functions with concepts equivalent to:

- `preprocessRows`
- `createCandidates`
- `apriori`
- `buildRules`
- `zhangsMetric`
- `createUnique8DigitId`

### 5. Rule generation

For each frequent itemset of size >= 2:

- enumerate proper antecedent subsets
- derive consequents as complement
- compute all metrics
- keep only rules meeting `minConfidence`

Required rule fields:

- `antecedents`
- `consequents`
- `support`
- `confidence`
- `lift`
- `leverage`
- `conviction`
- `zhangs_metric`
- `combination_count`
- `cost_antecedents`
- `cost_consequents`
- `cost_consequents_weighted`
- `cost_total`
- `Mat_combination`
- `Mat_combination_items`
- `transaction_ids`
- `unique_ID`
- `different items`

### 6. Metrics

Support these formulas:

- `support(A union C)`
- `confidence = support(A union C) / support(A)`
- `lift = confidence / support(C)`
- `leverage = support(A union C) - support(A) * support(C)`
- `conviction = (1 - support(C)) / (1 - confidence)`, with `Infinity` if confidence is 1
- Zhang's metric in the range `[-1, 1]`

The output should remain conceptually aligned with the Python reference implementation.

### 7. unique_ID generation

Generate a deterministic 8-digit identifier from `Mat_combination` using a SHA-256-based
approach via browser crypto.

Important known caveat:

- collisions are still possible because `unique_ID` is derived from the order-number
  combination rather than the full item semantics

This caveat should be acknowledged in the documentation.

## UI layout

### Global structure

Use a two-column desktop layout:

- left: sidebar
- right: workspace

Below roughly `1180px`, collapse to one column.

The workspace must contain:

- a topbar
- a tab bar
- one active panel

Panels:

- `Console`
- `Table`
- `Graph & Top 20`

### Topbar

The topbar must include:

- title area
- current dataset name
- compact stat chips:
  - Transactions
  - Items
  - Rules
  - Visible
- theme toggle
- runtime status pill
- Help button

Do **not** include any graph resize toggle.

### Runtime status

Use a pill labeled `ready` when idle.

When analysis runs:

- switch away from ready
- show a spinner in the run button
- restore ready state when finished

### Sidebar auto-collapse

After a successful analysis:

- auto-collapse the sidebar after 3 seconds

Also provide a manual sidebar collapse/expand button and persist that state.

## Table behavior

The Table tab must support:

- sortable columns
- quick search on antecedents/consequents
- configurable metric visibility
- table row limit

Always-visible columns:

- Antecedents
- Consequents
- Confidence

Default visible optional columns:

- Support
- Lift
- Combination Count
- Different Items

Export all rules to Excel using the currently selected visible columns.

## Console behavior

The Console tab is a timestamped log view.

It should log:

- file loading
- detected columns
- text filtering effects
- consumption filtering effects
- price derivation notes
- frequent-itemset counts
- rule-generation progress
- missing price warnings
- success messages
- runtime errors

Use concise English log messages.

## Top-20 chart behavior

Build a horizontal Top-20 chart of **weighted consequent costs**.

This chart must:

- show the top 20 by `cost_consequents_weighted`
- use a violet-to-cyan gradient
- animate bar width on normal render
- replay the bar animation when a combination is selected
- allow clicking:
  - a bar
  - its y-axis label
  - its value label

Click behavior:

- fill the graph search field with the clicked combination ID
- set graph min-confidence to `0`
- open the Graph tab if needed
- render the graph filtered to that combination
- replay the Top-20 animation
- show a toast confirming the graph filter

Animation details:

- duration is **exactly `650ms`** (JS constant `TOP_BAR_ANIMATION_MS = 650`)
- easing is `d3.easeCubicOut`
- bars animate from `width = 0` to their final width, and each bar stores that final width in a
  `data-final-width` attribute so the animation can be replayed without recomputing the scale
- respect `prefers-reduced-motion`: when reduced motion is requested, set the final width
  instantly and skip the transition

## Graph requirements

### General

Build a D3 force-directed rule graph in SVG.

Nodes are antecedent/consequent itemsets. Edges are rules.

Graph controls:

- `Confidence %` checkbox
- `Fullscreen`
- `New tab`
- `Export PNG`
- `Max. edges`
- `Min. confidence`
- `Graph search`
- `Fit graph`

### Encoding

- edge color = confidence
- edge width = combination count
- node size = visibility/frequency

Use a confidence gradient equivalent to:

- cyan -> violet -> magenta

### Compound itemsets as touching circles

This is important.

If a node represents more than one item, do **not** draw it as one single circle.

Instead:

- 2-item itemsets: two touching circles
- 3+ item itemsets: a touching cluster/ring of circles

Node size semantics:

- the overall compound node should still reflect the same logical size meaning as the
  single-node version
- individual child circles should not misleadingly imply a smaller node importance

### Interaction

Support:

- wheel zoom
- background pan
- draggable nodes
- click node to focus and hide unrelated graph parts
- click same node or background to clear focus
- hover tooltip on nodes and edges
- click node or edge to open a fixed detail panel

### Hover tooltip content

For node hover include:

- node label
- rule connections
- combination count sum
- node cost sum
- transaction IDs
- note that costs are counted once per visible combination

For edge hover include:

- source -> target
- confidence
- lift
- combination count
- transaction IDs
- combination text

### Fixed detail panel

A right-side floating detail panel inside the graph card must open on node or edge click.

For nodes include at least:

- Rule connections
- Combination count sum
- Node cost sum
- Items
- Transaction IDs count

For edges include at least:

- Confidence
- Lift
- Combination count
- Weighted cost
- Combination
- Transaction IDs count

The detail panel must also include:

- a textarea with the transaction IDs
- `Copy IDs` button

Clipboard behavior:

- if clipboard API works, copy and toast success
- otherwise focus/select the textarea and instruct the user to press Cmd+C

### Confidence labels

The `Confidence %` toggle must show percentage labels on edges.

Important edge case:

- if there are bidirectional or paired relationships, the graph must be able to display
  both confidence labels rather than visually collapsing them into one unreadable label
- use edge-label offsets or a similar strategy

### Graph search and filtering

Graph search must match against:

- antecedents
- consequents
- `Mat_combination_items`
- `Mat_combination`
- `unique_ID`

Graph filtering should use:

- graph-only min-confidence
- graph edge limit
- graph search field

### Graph ranking for visibility

When choosing which rules enter the graph, sort by a weighted score that favors:

- higher confidence
- meaningful combination count

Use **exactly** this score, sorted descending, applied to `state.filteredRules` (after the
min-confidence filter and before the edge-limit slice):

```text
score = confidence * Math.log2((combination_count || 0) + 2)
```

### Fit graph

Implement manual fit-to-content with zoom transform.

Behavior (exact):

- auto-fit after a (re)render is applied **instantly** (no transition)
- the manual `Fit graph` button animates with a `450ms` transition
- padding is `48` px on each side
- clamp the fit scale to `[0.18, 1.4]`:
  `scale = max(0.18, min(1.4, min((w - 96) / bbox.width, (h - 96) / bbox.height)))`
- the underlying d3 zoom behavior uses `scaleExtent([0.18, 4])`
- in Safari, round the computed translate `tx` / `ty` to integers before applying the transform

### Fullscreen mode

Support in-page fullscreen for the graph card:

- `Fullscreen` button changes to `Exit fullscreen`
- `Escape` exits fullscreen
- the graph should refit after the transition

### New-tab graph-only mode

Support opening the current graph in a standalone tab by:

- storing a slim graph payload in `localStorage`
- opening the same page with query params like:
  - `graphOnly=1`
  - `graphKey=<payload key>`

In graph-only mode:

- hide sidebar
- hide topbar
- hide tabs
- hide table and console
- hide the left Top-20 card
- show only the graph card filling the page
- hide the `Fullscreen` and `New tab` buttons there

### PNG export

Export the current graph viewport to PNG by:

- cloning the live SVG
- serializing it
- drawing it to canvas
- filling the background with a graph-appropriate solid color

Filename rule:

- if `Graph search` contains a number, use the first digit run as a prefix:
  - `<number>_rule-graph.png`
- otherwise:
  - `rule-graph.png`

Use a higher-resolution export scale based on device pixel ratio.

## Filter presets

Implement filter presets stored in browser storage and shareable through JSON files.

Preset content must include:

- include terms
- exclude terms
- quick search
- graph search
- graph min confidence
- graph edge limit
- confidence-label toggle
- selected metric columns

Required behavior:

- save under arbitrary user-chosen names
- overwrite confirmation on existing names
- dropdown applies a preset immediately
- delete current preset
- export preset collection as JSON
- import preset collection from JSON
- store preset collection in `localStorage`

Use:

- storage key `association-rule-filter-presets-v1`
- export schema version `1`

Important clarification:

- live presets are stored in browser storage
- exported JSON is for sharing/transport
- do not automatically sync live presets back into a physical project JSON file

## Theme and visual design

### Overall look

Design the UI as a polished, intentional product, not a bland CRUD tool.

Target aesthetic:

- dark "Deep Space" theme
- layered nebula gradients
- subtle starfield texture
- glowing violet/cyan accents
- glass-like translucent panels
- warm gold accent for selected actions

There must also be a **light theme** toggle.

### Dark theme

Dark mode should feel premium and atmospheric:

- deep indigo/black background
- luminous but restrained accents
- readable contrast

### Light theme

Light mode must be properly designed, not an inverted afterthought.

Requirements:

- bright but soft paper-like surfaces
- dark readable graph text
- graph card, controls, detail panel, and tooltip must remain legible
- Safari light mode must not fall back to dark low-contrast graph overlays

### Typography

Use:

- expressive display font for headings, e.g. Space Grotesk
- clean readable UI/body font, e.g. Source Sans 3

### Theme persistence

Persist theme selection in browser storage under:

- `association-rule-theme`

## Help modal

Create a searchable in-app Help modal with:

- topic list on the left
- content pane on the right
- search field filtering topics by title/content

The Help content must be comprehensive and in English. It should cover:

- overview
- quick start
- input format
- thresholds
- text filters
- running the analysis
- table and Top 20
- rule graph
- metrics
- output columns
- filter presets
- exports
- architecture
- persistence & privacy
- browser support & limits
- open-source / licensing
- version & contact

This Help content should broadly mirror the Markdown docs.

### Open-source / licensing help topic

This topic should not be generic. It should explicitly list the third-party components used
by the delivered project and state that no Python mining dependency is used in the HTML
version.

Include at least:

- the app itself as GPL-3.0
- `SheetJS / xlsx` via `xlsx.full.min.js` for Excel read/write
- `D3.js v7` via `d3.v7.min.js` for table/chart/graph rendering
- `Space Grotesk` and `Source Sans 3` as the main UI fonts if you load them
- an explicit note that **no mlxtend** is used in the HTML version

If vendored files contain upstream attribution banners, preserve them.

## Browser storage

Use browser storage for:

- filter presets
- sidebar collapsed state
- theme
- graph-only transient payload

Suggested keys:

- `association-rule-filter-presets-v1`
- `association-rule-sidebar-collapsed`
- `association-rule-theme`
- `association-graph-<timestamp>-<rand>`

## Safari / WebKit requirements

This section is critical. Do not ignore it.

The graph must render cleanly in Safari, especially in normal split view.

You must incorporate targeted WebKit-aware behavior:

1. Do not use live `backdrop-filter` overlays over the animating graph in Safari split view.
2. Keep Safari tooltip behavior lightweight.
3. Round cursor-following tooltip positions to integer pixels.
4. Round relevant Safari SVG transforms/coordinates where appropriate to reduce subpixel
   artifacts.
5. Use `shape-rendering: geometricPrecision` selectively for graph shapes and Top-20 bars.
6. Do not apply `crispEdges` or precision hints globally to everything, especially not in a
   way that harms text quality.
7. Avoid `foreignObject` in the graph.
8. Avoid promoting `#ruleGraph` itself to a composited layer.
9. In Safari split view, avoid simultaneous heavy recomposition of the graph and Top-20
   animation.
10. Keep hover-highlighting behavior conservative in split view if it triggers WebKit
    artifacts.

Also preserve these practical behaviors:

- Top-20 click should show the filtered graph immediately
- split-view graph rendering should avoid stale ghost pixels
- fullscreen and graph-only mode can remain richer/more isolated

## Required UX copy and naming

Use English throughout the UI except for filenames already present in the repository.

Required names include:

- `Association Rules Studio`
- `Console`
- `Table`
- `Graph & Top 20`
- `Run analysis`
- `Export all`
- `Output file name`
- `Confidence %`
- `Fullscreen`
- `New tab`
- `Export PNG`
- `Copy IDs`
- `Help`

For subtle credit/contact, include:

```text
In case of questions, contact Roland Emrich
```

Place it subtly, not as a loud hero element.

Also match these small but intentional project details:

- app title: `Association Rules Studio`
- version string: `v1.1` (JS constant `APP_VERSION = "1.1"`)
- `applyAppVersion()` must set the document `<title>`, the `#appVersion` pill, and the
  `#sidebarCopyright` line so they all render `Association Rules Studio v1.1`
- the sidebar brand shows the version pill `v1.1` next to a **GitHub pill** linking to
  `https://github.com/Adelsdorfer/Association-Rules-Studio`
- the sidebar credit line reads `Association Rules Studio v1.1` above
  `In case of questions, contact Roland Emrich`

## Documentation requirements

Also create/update these files in English:

- `README.md`
- `AGENTS.md`
- `DESIGN.md`
- `DESCRIPTION.md`

They must reflect the actual delivered app, not a generic template.

### README.md

Should document:

- what the app is
- repo contents
- sample workbook and example exported preset JSON
- input format
- defaults
- workflow
- table/graph behavior
- filter presets
- exports
- metrics
- persistence
- Safari quirks
- development notes

### AGENTS.md

Should document:

- architecture
- file roles
- important constraints
- graph behavior
- Safari/WebKit gotchas
- state model
- doc-sync expectations

### DESIGN.md

Should document:

- design language
- layout
- themes
- graph-specific visuals
- Safari rendering constraints
- responsive behavior

### DESCRIPTION.md

Should be a concise but accurate project summary.

## Technical behavior details to preserve

Implement these details if you want the recreation to match closely:

- use one central mutable `state` object
- use explicit `render*()` functions rather than reactive framework state
- sort final rules primarily by confidence, then lift
- expose a `ready` pill state in the topbar
- auto-collapse sidebar 3 seconds after successful analysis
- use a green ready state only when idle
- clear/rebuild graph fully on each graph render
- persist sidebar collapsed state
- allow manual graph focus clearing by background click
- keep console/table/graph tabs visually distinct
- keep graph detail panel copy-friendly
- keep Top-20 interactions tightly integrated with graph filtering

## Deliverable quality bar

This should feel like a polished serious local analysis tool, not a quick demo.

The implementation should be:

- visually intentional
- behaviorally complete
- internally coherent
- documented
- browser-aware

---

# Authoritative appendices (exact, binding values)

The sections above describe intent and behavior. The appendices below give the **exact** values
that make the reproduction byte-for-byte faithful. Where prose and an appendix disagree, the
appendix wins. Implement every constant, string, color, formula, and ordering exactly.

## Appendix A — Document head and asset loading

The document starts exactly like this (English `lang`, UTF-8, responsive viewport):

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Association Rules Studio</title>
  <script src="./xlsx.full.min.js"></script>
  <script src="./d3.v7.min.js"></script>
  <style> /* all CSS, see appendices D/E */ </style>
</head>
<body> ... </body>
</html>
```

- **SheetJS is loaded before D3**, both as local relative `<script>` tags in `<head>` (not `defer`,
  not `async`, not from a CDN).
- The first line of the `<style>` block is the Google Fonts import (and the only remote resource):
  ```css
  @import url("https://fonts.googleapis.com/css2?family=Space+Grotesk:wght@500;600;700&family=Source+Sans+3:wght@400;500;600;700&display=swap");
  ```
- Preserve the vendored attribution banners unmodified at the top of each library file:
  - `xlsx.full.min.js`: `/*! xlsx.js (C) 2013-present SheetJS -- http://sheetjs.com */`
  - `d3.v7.min.js`: `// https://d3js.org v7.9.0 Copyright 2010-2023 Mike Bostock`
- The entire app (markup, one `<style>`, one `<script>`) lives in `index.html`. The `<script>`
  runs at the end of `<body>` and ends with the initialization sequence in Appendix N.

## Appendix B — Global constants, timers, and the state object

Declare these constants verbatim at the top of the script:

```js
const APP_VERSION = "1.1";
const FILTER_PRESET_STORAGE_KEY = "association-rule-filter-presets-v1";
const FILTER_PRESET_EXPORT_VERSION = 1;
const SIDEBAR_COLLAPSED_STORAGE_KEY = "association-rule-sidebar-collapsed";
const THEME_STORAGE_KEY = "association-rule-theme";
const TOP_BAR_ANIMATION_MS = 650;
```

The single mutable state object (no framework, no reactivity):

```js
const state = {
  workbookRows: [],
  headers: [],
  sourceName: "",
  allRules: [],
  filteredRules: [],
  visibleRules: [],
  sortColumn: "confidence",
  sortAsc: false,
  stats: { transactions: 0, items: 0, frequentItemsets: 0 },
  graphZoom: null,
  graphSimulation: null,
  isSafari: false,
  savedFilterPresets: {},
  activeFilterPreset: ""
};
```

`state.pendingFit` is also used transiently (set true before a graph re-render that should fit
once the simulation settles).

Exact timer / debounce values:

- toast auto-hide: `3200ms`
- sidebar auto-collapse after a successful analysis: `3000ms`
- graph control inputs (`graphLimit`, `graphMinConfidence`, `graphSearch`) debounce: `220ms`
- window `resize` re-render debounce: `180ms`
- `runAnalysis` yields to the UI with `await new Promise(r => setTimeout(r, 30))` before mining
- new-tab graph-only payload localStorage key:
  `` `association-graph-${Date.now()}-${Math.random().toString(36).slice(2)}` ``

## Appendix C — Complete element / ID inventory

Every `id` and its default text/value/state. Reproduce all of them.

**Sidebar**

- `#sidebarToggleBtn` — button, text `<` (expanded) / `>` (collapsed)
- `#appVersion` — pill, text `v1.1`
- `#githubLink` — anchor pill to the GitHub repo, inline SVG + text `GitHub`
- `#fileStatus` — pill, text `no file`
- `#inputFile` — `type="file"`, `accept=".xlsx,.xls"`
- `#outputName` — `type="text"`, value `SPC_Rules.xlsx`
- `#minSupport` — `type="number"`, `min="0.000001" max="1" step="0.001"`, value `0.003`
- `#minConfidence` — `type="number"`, `min="0.000001" max="1" step="0.01"`, value `0.10`
- `#maxItemsetSize` — `type="number"`, `min="0" step="1"`, value `4` (0 means no limit)
- `#topRuleLimit` — `type="number"`, `min="100" step="100"`, value `5000`
- `#includeZeroConsumption` — checkbox, **checked**, label `Include consumption = 0`
- `#includeTerms` — text, placeholder `e.g. filter, motor`
- `#excludeTerms` — text, placeholder `e.g. service, dummy`
- `#filterPresetStatus` — pill, text `0 saved`
- `#filterPresetSelect` — select, first option value `""` text `No saved filter`
- `#saveFilterPresetBtn` (`Save filter`), `#deleteFilterPresetBtn` (`Delete filter`, disabled)
- `#exportFilterPresetsBtn` (`Export JSON`), `#importFilterPresetsBtn` (`Import JSON`)
- `#filterPresetImportFile` — hidden file input, `accept=".json,application/json"`
- `#runBtn` (`Run analysis`), `#downloadAllBtn` (`Export all`, disabled)
- `#sidebarCopyright` — credit line

**Topbar**

- eyebrow text `Results`; `#datasetTitle` — `<h2>`, text `No analysis loaded yet`
- `#topbarStats` containing four stat chips: `#statTransactions`, `#statItems`, `#statRules`,
  `#statVisible` (all start at `0`) with labels `Transactions`, `Items`, `Rules`, `Visible`
- `#themeToggleBtn` — `role="switch"`, `aria-checked="false"`, contains a moon SVG, a sun SVG,
  and a `.theme-toggle-thumb`
- `#runtimeStatus` — pill with class `is-ready`, text `ready`
- `#helpOpenBtn` — button `Help`

**Tabs** (`<nav class="tabs">`): three buttons with `data-tab` = `consolePanel`, `tablePanel`
(default `active`), `graphPanel`; labels `Console`, `Table`, `Graph & Top 20`.

**Console panel** `#consolePanel`: `<pre class="log" id="logBox">`.

**Table panel** `#tablePanel` (default `active`):

- toolbar: `#quickSearch` (placeholder `Quick search in antecedents and consequents`),
  `#clearSearchBtn` (`Clear search`), `#resetFiltersBtn` (`Reset filters`, disabled),
  `#renderGraphBtn` (`Show graph`, primary, disabled)
- `#metricChooser` (metric toggle chips, built at runtime)
- `#resultsTable` with empty `<thead>` and a `<tbody>` whose initial row reads
  `Select an Excel file and run the analysis.`

**Graph panel** `#graphPanel` → `.chart-grid` with two cards:

- Top-20 card: `<h3>Top 20 weighted consequent costs</h3>`, `#topChartStatus` pill (`empty`),
  `<svg id="topChart">`
- Graph card `#graphCard`: `<h3>Rule Network</h3>`; `.graph-actions` with a `.graph-toggle`
  label wrapping `#showConfidenceLabels` (checkbox) + text `Confidence %`, then `#fullscreenGraphBtn`
  (`Fullscreen`), `#openGraphTabBtn` (`New tab`, disabled), `#exportGraphPngBtn` (`Export PNG`,
  disabled), `#graphStatus` pill (`empty`); `<svg id="ruleGraph">`; a detail panel
  `#graphDetailPanel` containing `#graphDetailType` (`Selection`), `#graphDetailTitle`
  (`No selection`), `#graphDetailCloseBtn` (`×`), `#graphDetailMetrics`, a readonly textarea
  `#graphDetailIds` labelled `Transaction IDs`, and `#graphDetailCopyBtn` (`Copy IDs`);
  `.graph-controls` with `#graphLimit` (`min="5" max="500" step="5"`, value `150`),
  `#graphMinConfidence` (`min="0" max="1" step="0.01"`, value `0`), `#graphSearch`
  (placeholder `Item or combination`), and `#fitGraphBtn` (warm, `Fit graph`, disabled).

**Globals**: `#graphTooltip`, `#toast`, and a help modal `#helpModal` (with `#helpBackdrop`,
`#helpTitle` `Help`, `#helpSearch` placeholder `Search help and metrics`, `#helpCloseBtn` `Close`,
`#helpTopics`, `#helpContent`).

> Note the one deliberate asymmetry: `#graphLimit` has the markup value `150`, but
> `currentGraphRules()` falls back to `90` when the field is empty/NaN, and preset-load defaults it
> to `"90"`. Reproduce all three numbers exactly.

## Appendix D — CSS design tokens (both themes) and chart palettes

**Dark theme** (`:root`, `color-scheme: dark`):

```css
--ink:#eaeefb; --muted:#97a1c2; --paper:#0e1424; --paper-strong:#141c30;
--line:rgba(255,255,255,0.10); --line-strong:rgba(255,255,255,0.20);
--pine:#8b7bff; --moss:#22d3ee; --saffron:#f4c869; --clay:#ff6bb3; --blue:#6ea8ff;
--graph-bg:#060912; --graph-panel:rgba(15,21,38,0.88);
--shadow:0 24px 70px rgba(2,4,12,0.55); --glow:0 0 26px rgba(139,123,255,0.42);
--bg:#060912; --surface:rgba(255,255,255,0.045); --surface-strong:rgba(255,255,255,0.08);
--radius-lg:28px; --radius-md:18px; --radius-sm:12px;
```

**Light theme** (`:root[data-theme="light"]`, `color-scheme: light`):

```css
--ink:#1b2438; --muted:#5b6b8c; --paper:#ffffff; --paper-strong:#eef1f8;
--line:rgba(20,28,48,0.12); --line-strong:rgba(20,28,48,0.22);
--pine:#6d5cf0; --moss:#0891b2; --saffron:#e0a92b; --clay:#e85aa0; --blue:#2563eb;
--graph-bg:#eef2fb; --graph-panel:rgba(255,255,255,0.9);
--shadow:0 24px 55px rgba(31,45,80,0.14); --glow:0 0 24px rgba(109,92,240,0.25);
--bg:#e9eef9; --surface:rgba(255,255,255,0.78); --surface-strong:rgba(20,28,48,0.05);
```

Light theme also overrides many surfaces explicitly (topbar, tabs, table, log, graph card,
graph controls, detail panel, tooltip, help, version pill, GitHub pill, and the `is-ready` pill)
so nothing falls back to a low-contrast dark overlay — **including combined
`:root[data-theme="light"].is-safari` rules** for the in-card graph head/controls.

**Backgrounds.** `html` uses three radial nebula gradients plus a `linear-gradient(160deg, #070b18
0%, #060912 46%, #04060f 100%)` with `background-attachment: fixed`; the light variant uses the
same structure with lighter stops ending in `#e6ecf8`. A starfield is drawn on `body::before`
(seven small radial-gradient "stars", `background-size: 460px 460px`, `opacity: 0.6`) animated by
`@keyframes twinkle` (`0.42 → 0.72`, `7.5s ease-in-out infinite alternate`). In light mode
`body::before { opacity: 0 }`. Scrollbars are thin and violet (`rgba(139,123,255,0.45/0.40/0.62)`).
`::selection` is `rgba(139,123,255,0.40)`.

**`chartInk()`** returns the exact palette used by the SVGs, switched by theme:

```text
DARK:  axisText #97a1c2, axisLine rgba(255,255,255,0.14), yAxisText #c2cae6, barValue #eaeefb,
       nodeFill rgba(255,250,240,0.88), nodeHalo rgba(6,9,18,0.78), edgeFill rgba(255,250,240,0.92),
       edgeHalo rgba(6,9,18,0.86), nodeStroke rgba(255,250,240,0.82), arrow rgba(255,250,240,0.72),
       empty rgba(255,250,240,0.78), pngBg #060912
LIGHT: axisText #5b6b8c, axisLine rgba(20,28,48,0.12), yAxisText #38415c, barValue #1b2438,
       nodeFill rgba(27,36,56,0.94), nodeHalo rgba(255,255,255,0.92), edgeFill rgba(27,36,56,0.96),
       edgeHalo rgba(255,255,255,0.9), nodeStroke rgba(27,36,56,0.42), arrow rgba(27,36,56,0.55),
       empty rgba(27,36,56,0.7), pngBg #eef2fb
```

Fonts: headings/eyebrows/display use `"Space Grotesk"`; body uses
`"Source Sans 3", "Trebuchet MS", sans-serif` at `16px`; monospace (log) uses
`ui-monospace, SFMono-Regular, Menlo, Consolas, monospace`.

## Appendix E — Layout grid and responsive behavior

```css
.app-shell { display:grid; grid-template-columns:minmax(320px,420px) minmax(0,1fr);
             gap:22px; min-height:100vh; padding:22px; }
body.sidebar-collapsed .app-shell { grid-template-columns:72px minmax(0,1fr); }
.sidebar { position:sticky; top:22px; max-height:calc(100vh - 44px); overflow:auto;
           border-radius:28px; padding:22px; }
.workspace { display:grid; grid-template-rows:auto auto minmax(0,1fr);
             height:calc(100vh - 44px); border-radius:28px; overflow:hidden; }
.chart-grid { display:grid; grid-template-columns:minmax(0,0.95fr) minmax(420px,1.35fr);
              gap:18px; min-height:min(680px, calc(100vh - 265px));
              max-height:calc(100vh - 265px); align-items:stretch; }
```

`.sidebar` and `.workspace` share `border:1px solid var(--line)`, `background:var(--surface)`,
`box-shadow:var(--shadow)`, `backdrop-filter:blur(18px)`.

Breakpoints:

- `@media (max-width:1180px)`: `.app-shell` and `.chart-grid` collapse to a single column
  (`grid-template-columns:1fr`); the sidebar becomes static; `.stats` and `.graph-controls`
  become two columns.
- `@media (max-width:720px)`: `.app-shell` padding `12px`; topbar/toolbar/help collapse to one
  column; `.stats`, `.field-row`, `.button-grid`, `.graph-controls` become a single column.

## Appendix F — Data pipeline (exact algorithms)

**Helpers**

- `keyOf(items)` = `[...items].sort().join("\u0001")` (U+0001 separator).
- `displaySet(items)` = sort by `localeCompare(…, "en")`, join with `", "`.
- `parseTerms(v)` = split on `,`, `trim().toLowerCase()`, drop empties.
- `numeric(v)` = treat `null/undefined/""` as `NaN`; otherwise `Number(String(v).replace(",", "."))`
  (German decimal commas supported), `NaN` if not finite.
- `formatCurrency(v)` = `Intl.NumberFormat("en-US", { style:"currency", currency:"EUR",
  minimumFractionDigits:2, maximumFractionDigits:2 })`.

**`preprocessRows(rows, options)`** — positional columns: `id=0, item=1, order=2`,
`consumption=3` only if ≥4 columns, `price=4` only if ≥5 columns. Throw on `<2` rows or `<3`
columns. Drop fully-empty rows. Apply **exclude** terms first, then **include** terms (each is a
case-insensitive substring test on the item column). If consumption exists and
`Include consumption = 0` is off, drop rows with `numeric(consumption) <= 0`. Group items by
transaction id; within a transaction de-duplicate items. Build `itemOrderMap` (first order number
seen per item). For prices: per item collect `unitPrice = (qty>0) ? total/qty : total`, then
`itemPriceMap.set(item, d3.mean(prices))`. Return `{ headers, transactions, transactionsWithIds,
itemOrderMap, itemPriceMap, itemCount }` where `itemCount` is the count of distinct items across
all transactions. Each `transactionsWithIds` entry is `{ id, items, itemSet:new Set(items) }`.

**`apriori(transactions, minSupport, maxItemsetSize)`**

- `transactionSets = transactions.map(t => new Set(t))`, `N = transactionSets.length`.
- `minCount = Math.ceil(minSupport * N)`.
- Singletons: count each item; keep `count >= minCount`; each becomes
  `{ items:[item], count, support:count/N }`; sort by `items[0].localeCompare(b, "de")`.
- For `size = 2 … limit` (`limit = maxItemsetSize>0 ? maxItemsetSize : Infinity`): build candidates
  with `createCandidates`, count by scanning every transaction (skip `transaction.size < size`),
  keep `count >= minCount`, sort by `keyOf().localeCompare(…, "de")`.
- `createCandidates(prev, size)`: classic join — for each pair `(i<j)` union the item sets; keep if
  the union length equals `size`; **prune** if any `(size-1)`-subset is not in the previous
  frequent set; de-duplicate by `keyOf`.
- Maintain `supportMap` and `countMap` keyed by `keyOf`. Return
  `{ frequentItemsets, supportMap, countMap, transactionSets }`.

**`zhangsMetric(supportA, supportC, supportAC)`** (exact):

```js
const leverage = supportAC - supportA * supportC;
const denominator = Math.max(supportAC * (1 - supportA), supportA * (supportC - supportAC));
return (!Number.isFinite(denominator) || denominator === 0) ? 0 : leverage / denominator;
```

**`createUnique8DigitId(combinationString)`** (deterministic, SHA-256 based, exact):

```js
const bytes  = new TextEncoder().encode(combinationString);
const digest = await crypto.subtle.digest("SHA-256", bytes);
const view   = new DataView(digest);
let mod = 0n;
for (let off = 0; off < digest.byteLength; off += 4) {
  mod = ((mod << 32n) + BigInt(view.getUint32(off))) % 100000000n;
}
let text = mod.toString().padStart(8, "0");
const leading = text.match(/^0+/);           // replace any leading zeros with '9's
if (leading) text = "9".repeat(leading[0].length) + text.slice(leading[0].length);
return text;                                  // always 8 digits, never leading zero
```

It is derived from `Mat_combination` (the joined order numbers). Document the collision caveat.

**`buildRules(context, minConfidence)`** — for each frequent itemset with `items.length >= 2`,
enumerate every proper subset as the antecedent and the complement as the consequent. Skip when
`supportA` or `supportC` is missing. `confidence = supportAC / supportA`; keep only
`confidence >= minConfidence`. Compute, per rule:

- `support = supportAC`, `lift = confidence / supportC`,
  `leverage = supportAC - supportA*supportC`
- `conviction = confidence >= 1 ? Infinity : (1 - supportC) / (1 - confidence)`
- `zhangs_metric = zhangsMetric(...)`
- `combination_count = countAC` (fallbacks: `countMap.get(keyOf(union))`, else a transaction scan)
- order numbers = unique `itemOrderMap` values for the union items, `String`-ified and **sorted**;
  `Mat_combination = orderNumbers.join("-")`; `Mat_combination_items = unionItems.join(", ")`
- `cost_antecedents` / `cost_consequents` via `sumPrices` (returns `null` if no priced item in the
  side; missing items are collected in a `missingPrices` set);
  `cost_consequents_weighted = costConsequents===null ? null : costConsequents * countAC`;
  `cost_total = (both null) ? null : (costAntecedents||0) + (costConsequents||0)`
- `transaction_ids` = ids of transactions whose `itemSet` contains every union item
- `different items` = `unionItems.length`; `antecedents`/`consequents` via `displaySet`
- after building all rules, set each `rule.unique_ID = await createUnique8DigitId(rule.Mat_combination)`

**`runAnalysis()`** validation and orchestration (exact):

- require a loaded workbook; `minSupport` and `minConfidence` each must satisfy `>0 && <=1`;
  `maxItemsetSize = Math.max(0, Math.floor(numeric(value) || 0))`
- **include-terms rule**: if exactly **one** include term is present, abort with
  `Please enter at least two items separated by a comma in the include items field.`
- `setBusy(true, "analyzing")`, clear the log, yield 30ms, then preprocess → apriori → buildRules
- **sort**: `rules.sort((a,b) => d3.descending(a.confidence,b.confidence) || d3.descending(a.lift,b.lift))`
  (confidence first, then lift)
- store `state.allRules`, `state.filteredRules = [...rules]`, `state.visibleRules = []`,
  `state.stats`, reset `sortColumn="confidence"`, `sortAsc=false`; call `updateAfterRules()`
- toast `Analysis completed: <n> rules found.`; schedule the 3s sidebar auto-collapse
- wrap in try/catch/finally: on error `log("ERROR: …")` + toast; finally `setBusy(false)`

## Appendix G — Columns, table, metric chooser, and Excel export

The `columns` array (order, keys, labels, and formatting) is exactly:

```text
antecedents  "Antecedents"  always
consequents  "Consequents"  always
confidence   "Confidence"   always, numeric, decimals 6
support      "Support"      numeric, decimals 6
lift         "Lift"         numeric, decimals 6
leverage     "Leverage"     numeric, decimals 6
conviction   "Conviction"   numeric, decimals 6
zhangs_metric "Zhangs Metric" numeric, decimals 6
combination_count "Combination Count" numeric, integer
cost_antecedents  "Cost Antecedents (EUR)" numeric, decimals 2
cost_consequents  "Cost Consequents (EUR)" numeric, decimals 2
cost_consequents_weighted "Cost Consequents x Combination Count (EUR)" numeric, decimals 2
cost_total   "Cost Antecedent+Consequents (EUR)" numeric, decimals 2
Mat_combination        "Mat_combination"
Mat_combination_items  "Mat_combination_items"
unique_ID    "unique_ID"
different items "Different Items" numeric, integer
```

- `defaultVisible = new Set(["support","lift","combination_count","different items"])`.
- The three `always` columns (Antecedents, Consequents, Confidence) are never toggleable; the
  metric chooser renders a `.metric-chip` checkbox for every non-`always` column, pre-checked when
  in `defaultVisible`, and re-renders the table on change.
- `formatValue`: blank for `null/undefined/NaN`; `Infinity → "inf"`; integer columns
  `Math.round`; numeric columns `toFixed(decimals ?? 6)`; otherwise the raw string.
- Sorting: clicking a header toggles `sortAsc` if it is the active column, else switches column and
  sets `sortAsc = !numeric` (text ascending, numbers descending by default). The active header
  shows a `  asc` / `  desc` suffix. Numeric sort treats missing as `-Infinity`.
- Quick search filters `antecedents`/`consequents` (case-insensitive substring); the rendered set
  is sliced to `Math.max(100, Math.floor(numeric(topRuleLimit) || 5000))`.
- `updateStats()` formats the four chips with `d3.format(",")`, sets `#datasetTitle`, and
  enables/disables `downloadAllBtn, resetFiltersBtn, renderGraphBtn, fitGraphBtn, openGraphTabBtn,
  exportGraphPngBtn` based on whether any rules exist.
- **Export all** (`exportRows`): builds rows keyed by column **label** (`Infinity → "inf"`), writes
  one sheet named `Association Rules` via `XLSX.utils.json_to_sheet` / `book_append_sheet`, and
  `XLSX.writeFile` to `outputName.value.trim() || "SPC_Rules.xlsx"`. `Export all` uses **all**
  columns; the table view honors only the visible columns.

## Appendix H — Top-20 chart (exact)

- Source: `state.filteredRules` filtered to rows with a `unique_ID`. Empty → centered text
  `No Top 20 data available.` and the status pill `empty`; return `0`.
- Aggregate with `d3.rollups` keyed by `unique_ID`, summing `cost_consequents_weighted`; the label
  is the first non-empty `Mat_combination_items` (fallback `Mat_combination`, then `unique_ID`).
  Sort descending by total; take the top **20**. Status pill becomes `<n> bars`.
- Margins `{ top:22, right:28, bottom:42, left:224 }`. `x` is `scaleLinear` `[0, max].nice()` to
  `[left, width-right]`; `y` is `scaleBand` over ids `[top, height-bottom]` with `padding 0.22`.
- Bottom axis: `axisBottom(x).ticks(5).tickFormat(d3.format(".2s"))`, domain line removed.
- Left axis labels: `` `${label} (${id})` `` truncated to 32 chars (`slice(0,29)+"..."`); clicking a
  y-axis label calls `showCombinationInGraph(row)`.
- Gradient `#barGradient`: horizontal `#8b7bff` → `#22d3ee`. Bars: `rx 9`,
  `fill url(#barGradient)`, `shape-rendering geometricPrecision`, height = `y.bandwidth()`,
  `finalWidth = Math.max(2, x(total) - left)`, stored in `data-final-width`. Animate width
  `0 → finalWidth` over `TOP_BAR_ANIMATION_MS` with `d3.easeCubicOut` unless reduced motion; return
  the animation ms (or `0`).
- Value labels (`d3.format(",.0f")`) sit at `x(total)+8`, weight 800, size 12. Bars, value labels,
  and y-axis labels are all clickable → `showCombinationInGraph`.
- `replayTopBarAnimation()` interrupts, resets width to 0, and replays to `data-final-width`.
- `showCombinationInGraph(combination)`: set `graphSearch = combination.id`,
  `graphMinConfidence = "0"`, switch to the graph tab with `{ renderGraph:false }` if not already
  active, call `renderGraphAfterLayout()`, replay the bars (delay `90ms` in Safari split view, else
  `0`), and toast `` `Graph filtered to ${label} (${id}).` ``.

## Appendix I — Rule graph (exact)

**Selecting graph rules (`currentGraphRules`)**: from `state.filteredRules`, keep
`confidence >= max(0, numeric(graphMinConfidence) || 0)`, sort descending by
`confidence * Math.log2((combination_count||0)+2)`, optionally filter by the lowercased
`graphSearch` matched against antecedents, consequents, `Mat_combination_items`, `Mat_combination`,
and `unique_ID`, then `slice(0, Math.max(5, Math.floor(numeric(graphLimit) || 90)))`.

**Full teardown on every render** (`renderGraph`): `svg.interrupt()`, interrupt all descendants,
stop and null `state.graphSimulation`, `svg.on(".zoom", null)`, reset
`svg.property("__zoom", d3.zoomIdentity)`, remove all children, null `state.graphZoom`, and
`closeGraphDetail()`. Empty result → centered text `No graph data for the current filters.` and
status `empty`. Size from `clientWidth/clientHeight` (fallbacks `900 × 680`); set the `viewBox`
once.

**Nodes / links.** One node per distinct antecedent string and per distinct consequent string
(`{ id, label, count, degree, role, items: splitItems(id).length, costSum, costKeys:Set,
transactionIds:Set }`). For each rule add a link `{ source, target, confidence, lift,
count:combination_count, cost:cost_consequents_weighted, transactionIds, mat:Mat_combination_items
|| Mat_combination }`, accumulate `count` and `degree` on both endpoints, and add cost (antecedent
side = `cost_antecedents * count`; consequent side = `cost_consequents_weighted`, fallback
`cost_consequents * count`) **deduplicated once per `combinationKey|nodeId`** via `addNodeCost`.

**Scales / colors.**

- `radius = d3.scaleSqrt().domain(extent(count)).range([10, 34])`
- `linkWidth = d3.scaleSqrt().domain(extent(count)).range([1.1, 4.2])`
- `edgeColor = d3.scaleSequential(d3.piecewise(d3.interpolateRgb.gamma(2.2),
  ["#22d3ee","#8b7bff","#ff6bb3"])).domain([minConfidence, maxConfidence])` (cyan → violet → magenta)
- arrow marker `#arrow`: `viewBox "0 -5 10 10"`, `refX 16`, `markerWidth/Height 10`,
  `markerUnits userSpaceOnUse`, `orient auto`, path `M0,-5L10,0L0,5`, fill `ink.arrow`

**Layers** (inside one `zoomLayer` group, in order): link layer (`stroke-linecap round`), edge-label
layer (`pointer-events none`), node layer, label layer.

**Zoom.** `d3.zoom().scaleExtent([0.18, 4])`; on zoom set the `zoomLayer` transform. In Safari, apply
`translate(round(x),round(y)) scale(k)`; otherwise apply the transform object directly. Store
`state.graphZoom = { svg, zoom, width, height, content: zoomLayer.node() }`.

**`svgCoord(v)`** = `state.isSafari ? Math.round(v*2)/2 : v` (half-pixel snapping in Safari only).

**Seeding (`seedNodePositions`)**: barycentric ordering refined over **8 passes**, then place nodes
on a circle of radius `min(width,height) * 0.38` (fallback 240) around the center.

**Force simulation** (exact):

```js
d3.forceSimulation(nodes)
  .force("link", d3.forceLink(links).id(d => d.id)
      .distance(d => 130 + Math.min(160, String(d.source.id||d.source).length + String(d.target.id||d.target).length)))
  .force("charge", d3.forceManyBody().strength(-520))
  .force("collide", d3.forceCollide().radius(d => nodeVisualRadius(d) + 28))
  .force("x", d3.forceX(width/2).strength(0.065))
  .force("y", d3.forceY(height/2).strength(0.075))
  .alphaDecay(0.0182);
```

**Edges** are arcs: `M sx,sy A dr,dr 0 0,1 tx,ty` where `dr = distance*1.45` and the target end is
pulled back by `nodeVisualRadius(target)+10`; all coordinates pass through `svgCoord`. Stroke =
`edgeColor(confidence)`, width = `linkWidth(count)`, `stroke-opacity 0.66`, `marker-end url(#arrow)`,
`shape-rendering geometricPrecision`.

**Edge labels** (only when `Confidence %` is on): text `${Math.round(confidence*100)}%`, size 11,
weight 900, `paint-order stroke`, stroke `ink.edgeHalo` width 5, fill `ink.edgeFill`. Paired/
bidirectional edges between the same two nodes get a perpendicular `labelOffset` of `28`
(`assignEdgeLabelOffsets`) so both labels stay readable.

**Compound nodes (touching circles), exact geometry:**

- `compoundDotRadius(d) = Math.max(9, Math.min(18, radius(d.count) * 0.78))`
- `compoundRingRadius(d)`: `0` for 1 item; `= dotRadius` for 2 items; `= dotRadius * 1.08` for 3+
- `nodeVisualRadius(d)`: single item → `radius(count)`; multi → `ringRadius + dotRadius`
- **1 item** → one circle, `r = radius(count)`, `fill #8b7bff`, stroke `ink.nodeStroke` width 1.6
- **2 items** → two touching circles at `x = ∓dotRadius, y = 0`: left `#22d3ee`, right `#8b7bff`
- **3+ items** → `n` dots on a ring of `ringRadius`, angle `-π/2 + (i/n)·2π`, palette cycled
  `["#8b7bff","#22d3ee","#ff6bb3","#f4c869","#6ea8ff","#42e8c0"]`; child circles use stroke width 1.35
- Node labels: fill `ink.nodeFill`, halo `ink.nodeHalo` width 5, `paint-order stroke`, weight 800,
  size `degree>2 ? 12 : 10`, text truncated to `items>1 ? 30 : 24` chars, placed at
  `y + nodeVisualRadius + 14`.

**Tooltip.** `#graphTooltip` follows the cursor; `moveTooltip` rounds to integers and uses
`translate(x,y)` in Safari but `translate3d(x,y,0)` elsewhere. Node tooltip lines: label,
`Rule connections`, `Combination count sum`, `Node cost sum` (currency),
`Transaction IDs` (`formatTransactionIds`, first 18 + `+N more`), and the dim note
`Costs counted once per visible combination.`. Edge tooltip: `source -> target`, `Confidence`
(4 dp), `Lift` (4 dp), `Combination Count`, `Transaction IDs`, and the combination text.

**Focus / hover.** Clicking a node opens its detail panel and toggles focus on it + direct
neighbors (others fade to `opacity 0`, focused node stroke 3.2 vs 1.6); clicking the same node, the
background, or `Escape`-less background click clears focus; the status pill shows
`Focus: <n> nodes / <m> edges`. **Hover highlighting only runs in fullscreen or graph-only mode**
(`hoverHighlightEnabled` = card is `graph-fullscreen` or body is `graph-only`) — this is a Safari
split-view safeguard. Tooltips still appear on hover in all modes.

**Detail panel.** Node rows: `Rule connections`, `Combination count sum`, `Node cost sum`,
`Items`, `Transaction IDs` (count). Edge rows: `Confidence`, `Lift`, `Combination count`,
`Weighted cost`, `Combination`, `Transaction IDs` (count). The textarea lists the ids one per line;
`Copy IDs` uses the async clipboard, and on failure selects the textarea and toasts
`Transaction IDs selected. Press Cmd+C to copy.`.

**Settle-fit.** On `tick`, when `state.pendingFit && simulation.alpha() < 0.06`, run `fitGraph(false)`
once; also on `simulation.on("end")`.

**Drag.** Standard d3 drag: on start `alphaTarget(0.3).restart()` and pin `fx/fy`; on end
`alphaTarget(0)` and release.

**PNG export (`exportGraphPng`).** `scale = Math.min(3, Math.max(2, devicePixelRatio||1) * 1.5)`;
clone the live SVG, set width/height to the viewBox base, set its `font-family` to
`"Source Sans 3", "Trebuchet MS", sans-serif`, serialize to a blob URL, draw onto a canvas filled
with `chartInk().pngBg`, then `canvas.toBlob(...,"image/png")` and download.
`graphPngFilename()`: if `graphSearch` contains a digit run, prefix `` `${match}_` ``; filename is
`` `${prefix}rule-graph.png` `` (else `rule-graph.png`).

**Fullscreen (`toggleGraphFullscreen`).** Toggle `graph-fullscreen` on the card and
`graph-modal-open` on the body; button text toggles `Fullscreen` ⇄ `Exit fullscreen`;
set `state.pendingFit = true` and `setTimeout(renderGraph, 80)`. `Escape` exits (after help and the
detail panel).

**New tab / graph-only.** `openGraphInNewTab` slims each rule (`slimGraphRule` keeps antecedents,
consequents, confidence, lift, combination_count, cost_consequents_weighted, Mat_combination,
Mat_combination_items, transaction_ids, unique_ID), stores `{ sourceName, stats, rules, controls }`
under the `association-graph-…` key, and opens the same URL with `?graphOnly=1&graphKey=<key>`.
`loadGraphOnlyMode` (run during init) detects `graphOnly=1`, adds `body.graph-only`, loads the
payload, hides `#fullscreenGraphBtn` and `#openGraphTabBtn`, sets the title to
`` `Rule Network - ${sourceName}` ``, switches to the graph tab, and fits after `180ms`.

## Appendix J — Filter presets (exact)

A preset object captures exactly: `includeTerms`, `excludeTerms`, `quickSearch`, `graphSearch`,
`graphMinConfidence`, `graphLimit`, `showConfidenceLabels` (boolean), and `metricKeys` (the checked
metric toggles). Behavior:

- **Save**: `window.prompt("Filter name:", activeName)`; empty name → toast and abort; existing name
  → `window.confirm("Overwrite filter '<name>'?")`; persist to localStorage; toast
  `Filter saved: <name>`.
- **Delete**: confirm `Delete filter '<name>'?`, remove, toast `Filter deleted: <name>`.
- **Select** from the dropdown applies immediately (`applySavedFilterPreset`), restoring all fields
  (graph min-confidence defaults to `"0"`, graph limit defaults to `"90"`), re-running
  `applyResultFilter` if rules exist, and toasting `Filter loaded: <name>`.
- **Export JSON** (`exportFilterPresets`) downloads
  `{ version:1, exportedAt:ISODate, activeFilterPreset, presets, currentFilter }` as
  `association-rule-filter-presets.json`.
- **Import JSON** merges `payload.presets` (or a bare object) into the saved presets, persists, and
  toasts the count. Storage key `association-rule-filter-presets-v1`; export schema `version` = `1`.
- The status pill shows `<n> saved`; live presets live only in localStorage; exported JSON is for
  transport and is never auto-synced back into a project file.

## Appendix K — Theme, sidebar, console, and miscellany (exact)

- **Theme**: `applyTheme` sets/removes `data-theme="light"` on `<html>`, updates the toggle's
  `aria-checked`/`aria-label`, and persists `THEME_STORAGE_KEY` (`dark`/`light`) unless told not to.
  `toggleTheme` flips it and re-renders charts when rules are present. `initTheme` reads the stored
  value without re-persisting.
- **Sidebar**: `setSidebarCollapsed` toggles `body.sidebar-collapsed`, flips the button glyph/labels
  (`<`/`>`), and persists `SIDEBAR_COLLAPSED_STORAGE_KEY` (`"1"`/`"0"`). `initSidebarState` restores it.
- **`setBusy(isBusy, text="ready")`**: disables the run button and swaps its content to
  `` `<span class="spinner"></span>working...` `` while busy; sets `#runtimeStatus` text; the green
  `is-ready` class is present only when idle **and** `text === "ready"`. Statuses used:
  `ready`, `analyzing` (run), `loading file` (file read).
- **`log`**: prepend `` `[${new Date().toLocaleTimeString()}] ` `` and auto-scroll the log box.
  Reproduce these exact console messages: `Reading Excel file: <name>`;
  `Detected columns -> ID: …, Item: …, Order no.: …`;
  `Excluded <n> rows based on exclude terms.`;
  `Include filter kept <n> rows and removed <m> rows.`;
  `Using column '<h>' as consumption indicator.`;
  `Excluded <n> rows with consumption <= 0.`;
  `Including ordered spare parts with consumption = 0 in the analysis.`;
  `Consumption column (4th column) not found. Proceeding without consumption filter.`;
  `Using column '<h>' as price indicator.` / `Price column (5th column) not found. Price-based
  metrics will be empty.`; `Derived unit prices for <n> distinct items.`;
  `Transforming transactions (browser-side one-hot equivalent)`;
  `Mining frequent itemsets (min_support=<x>)`; `Frequent itemsets found: <n>`;
  `Generating association rules (min_confidence=<x>)`;
  `Price information missing for <n> items; treated as zero in sums: …`;
  `Rules found: <n>` / `No rules found with the selected thresholds.`; `Done.`; `ERROR: <message>`.
- **`handleFile`**: `setBusy(true,"loading file")`, read with
  `XLSX.read(buffer, { type:"array" })`, take the first sheet, and
  `XLSX.utils.sheet_to_json(sheet, { header:1, defval:null, raw:true })` into
  `state.workbookRows`; update `#fileStatus`, `#datasetTitle`, log, toast `File loaded: <name>`.

## Appendix L — Help modal topics (exact set)

`helpTopics` is an array (rendered in this exact order; search filters by title + stripped text;
selecting shows its HTML; the topic buttons get an `active` class). Reproduce all **17** topics with
these `id`s and titles, and faithful English content mirroring the docs:

1. `overview` — *Overview*
2. `workflow` — *Quick start & workflow*
3. `input` — *Input data format* (positional columns table; ≥3 columns and ≥2 rows;
   `sheet_to_json(sheet,{header:1})`)
4. `thresholds` — *Thresholds* (defaults table: 0.003 / 0.10 / 4 / 5000 / on)
5. `filters` — *Text filters* (include needs 0 or ≥2 terms)
6. `run` — *Running the analysis*
7. `table` — *Results table & Top 20*
8. `graph` — *Rule graph* (controls table; encodings)
9. `metrics` — *Metrics* (full formula table for support/confidence/lift/leverage/conviction/
   Zhang/combination count/the four costs)
10. `columns` — *Output columns* (the full key→label list; default visible = support, lift,
    combination count, different items)
11. `presets` — *Filter presets*
12. `exports` — *Exports*
13. `architecture` — *How it works* (handleFile → preprocessRows → apriori/createCandidates →
    buildRules → runAnalysis; render layer; the single `state` object)
14. `persistence` — *Persistence & privacy* (the three localStorage keys; data never leaves the
    machine; only remote resource is Google Fonts)
15. `support` — *Browser support & limits* (`crypto.subtle.digest`, `BigInt`, `canvas.toBlob`,
    `backdrop-filter`; the unique_ID collision caveat; first-sheet-only; silent graph errors)
16. `oss` — *License & open-source* — must explicitly state the app is **GPL-3.0** and list the
    bundled components with licenses: **SheetJS / xlsx** (Apache-2.0, banner preserved), **D3.js v7**
    (ISC, full ISC text + banner preserved), **Space Grotesk & Source Sans 3** (SIL OFL 1.1), and an
    explicit note that **no mlxtend / no Python dependency** is used in the HTML version
17. `about` — *Version & contact* — `Association Rules Studio v1.1`, the GitHub repo link, and
    `In case of questions, contact Roland Emrich.`

(Topics that reference the version must interpolate `APP_VERSION`, i.e. show `v1.1`.)

## Appendix M — Safari / WebKit implementation (exact)

```js
function detectSafariBrowser() {
  const ua = navigator.userAgent;
  return /Safari/.test(ua) && !/(Chrome|Chromium|CriOS|FxiOS|Edg\/|OPR\/)/.test(ua);
}
function initBrowserFlags() {
  state.isSafari = detectSafariBrowser();
  document.documentElement.classList.toggle("is-safari", state.isSafari);
}
```

All WebKit workarounds are gated on `state.isSafari` / the `html.is-safari` class so that
Chrome, Edge, and Firefox are completely unaffected:

1. **Deferred graph render in split view.** `renderCharts` renders the Top-20 bars first; if
   `state.isSafari && isGraphSplitView()` and the bars animate, it waits `barMs + 40` and then calls
   `renderGraphAfterLayout` (a **double `requestAnimationFrame`**) so the graph paints on clean
   frames. Otherwise it renders the graph immediately. `isGraphSplitView()` = graph panel active,
   card not fullscreen, body not graph-only.
2. **No live `backdrop-filter`** over the in-card graph head/controls in Safari split view: replace
   them with opaque backgrounds (`rgba(6,9,18,0.86)` / `rgba(9,13,28,0.9)`).
3. **Layer-promote the card, not the SVG.** `html.is-safari .graph-card:not(.graph-fullscreen)` gets
   `transform:translateZ(0)` + `-webkit-backface-visibility:hidden`. **Never** promote `#ruleGraph`
   itself (that breaks hover/zoom).
4. **Half-pixel coordinate snapping** via `svgCoord` (Safari only) for all node/edge/label coords.
5. **Integer-rounded zoom translate** and integer-rounded fit translate in Safari.
6. **Lightweight tooltip** in Safari: `translate(x,y)` instead of `translate3d`, with
   `box-shadow:none; transition:none; will-change:auto; contain:layout paint style`.
7. **Hover highlighting only in fullscreen / graph-only** (`hoverHighlightEnabled`), never in split
   view.
8. `shape-rendering:geometricPrecision` is applied **selectively** to graph paths/markers/circles and
   Top-20 bars — never `crispEdges` globally, and nothing that harms text.
9. No `foreignObject` anywhere in the graph.
10. Light + Safari combined overrides keep the in-card graph head/controls legible
    (`:root[data-theme="light"].is-safari …`).

There is **no graph resize toggle / no `ResizeObserver`-based auto-resize control** anywhere in the
app — do not add one.

## Appendix N — Initialization order (exact)

At the very end of the `<script>`, call, in this order:

```js
initBrowserFlags();
applyAppVersion();
initTheme();
renderMetricChooser();
renderHelpTopics();
initSidebarState();
loadSavedFilterPresets();
updateFilterPresetUi();
setupTabs();
setupEvents();
if (!loadGraphOnlyMode()) {
  updateStats();
}
```

`setupEvents` wires: sidebar toggle; help open/close/backdrop/search; theme toggle; file input;
run; reset filters; preset save/delete/export/import + hidden file change + dropdown change;
`Export all`; clear search; quick search + table-limit re-render; `Show graph` → `switchTab`;
`Fit graph`; fullscreen; new tab; export PNG; detail close/copy; confidence-label re-render; the
three debounced (`220ms`) graph inputs; the global `Escape` handler (help → detail → fullscreen);
and the debounced (`180ms`) window `resize` re-render. `switchTab(panelId,{renderGraph})` toggles
tab/panel `active` classes and, when activating the graph panel fresh and `renderGraph !== false`,
calls `renderChartsAfterLayout` (double rAF).

---

## Acceptance checklist

Your result is only acceptable if all of the following are true:

1. The app runs locally from `index.html` with vendored D3 and SheetJS.
2. The UI is in English.
3. The app has Console, Table, and Graph & Top 20 tabs.
4. Apriori and rule generation are implemented in JavaScript, not delegated to Python.
5. Excel import and Excel export work.
6. Filter presets work through browser storage and JSON import/export.
7. The Top-20 chart is clickable and drives the graph.
8. The graph supports zoom, pan, drag, focus, fullscreen, new-tab mode, confidence labels,
   PNG export, and detail panel with copyable transaction IDs.
9. Multi-item itemsets render as touching multiple circles with the exact geometry in Appendix I.
10. Dark theme and light theme both look deliberate and readable.
11. Safari split-view graph behavior is treated as a first-class constraint (Appendix M).
12. Markdown docs are created and aligned with the implementation.
13. The version is `v1.1` everywhere (`APP_VERSION`, pill, document title, sidebar credit), and the
    sidebar brand shows the GitHub pill linking to the repo in Appendix C.
14. Every constant, default, label, formula, color, storage key, and ordering matches the
    authoritative appendices A–N exactly — the app is reproducible from this prompt alone.

## Execution instruction

Do not answer with a high-level explanation only. Implement the project files directly.

When done:

- ensure the HTML app is coherent
- ensure the docs match the app
- ensure the project still respects the no-build, no-framework, offline-first constraint

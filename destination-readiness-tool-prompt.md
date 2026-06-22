# Destination Readiness Factor Web — Developer Prompt

## Project overview

Build a single-page interactive web tool called the **Destination Readiness Factor Web**. It is designed for humanitarian evacuation planning teams to assess and visualise the real-time status of key factors that influence whether a destination can receive evacuees during armed conflict.

The tool has two modes of use:
- **Training / communication** — helps teams understand how destination factors relate to and depend on each other
- **Operational assessment** — allows field staff to input live status for each factor and instantly see the impact on overall destination readiness

---

## Tech stack

- Plain HTML, CSS, and JavaScript (no framework required)
- Single `index.html` file
- SVG rendered inline for the network diagram
- No external dependencies required (optionally use a small charting or export library — see extension points below)

---

## The seven destination factors

These are the nodes in the network. Each factor has a status that can be set by the user.

| ID | Label | Type | Notes |
|---|---|---|---|
| `security` | Security | **Gatekeeper** | Physical safety at the destination. Upstream of almost everything. |
| `consent` | Local auth. consent | **Gatekeeper** | Permission from local authorities. Unlocks access to resources. |
| `willingness` | Willingness | Standard | Destination community acceptance of evacuees. |
| `capacity` | Capacity | Standard | Physical space available at the destination. |
| `shelter` | Shelter | Standard | Adequate shelter provision. |
| `foodwater` | Food & water | Standard | Sufficient food and water supply. |
| `medical` | Medical capacity | Standard | Healthcare availability at the destination. |

**Gatekeepers** are visually distinct (larger node, star badge) and carry double weighting in the readiness score, because blocking either one cascades and caps the overall feasibility.

---

## Dependency relationships (directed edges)

These define the arrows in the network diagram. The direction indicates dependency or enablement.

```
security      → consent       (enables access)
security      → willingness   (enables acceptance)
consent       → capacity      (unlocks)
consent       → shelter       (unlocks)
consent       → foodwater     (unlocks)
willingness   → shelter       (supports)
willingness   → foodwater     (supports)
capacity      → medical       (enables)
shelter       → medical       (co-enables)
foodwater     → medical       (co-enables)
```

---

## Factor statuses

Each factor can be set to one of four statuses:

| Status | Colour | Meaning |
|---|---|---|
| Operational | Green `#639922` | Factor is fully functional |
| Partial | Amber `#EF9F27` | Factor is present but limited |
| Blocked | Red `#E24B4A` | Factor is not available or denied |
| Unknown | Grey `#B4B2A9` | Status not yet assessed |

---

## Network diagram behaviour

- Render nodes as circles. Gatekeepers should be visibly larger than standard nodes.
- Render edges as directed arrows between nodes.
- Edge colour reflects the combined status of the two nodes it connects:
  - Both operational → green
  - Either partial → amber
  - Either blocked → red (render as dashed line)
  - Both unknown → grey (render as dotted/faint line)
- Node colour reflects its current status using the colour table above.
- Nodes with status `unknown` should appear at reduced opacity.
- Clicking a node in the diagram should highlight its direct connections (dim everything else briefly).

### Suggested node positions (SVG viewBox 680 × 420)

```
security:     x=340, y=60   (top centre — gatekeeper)
consent:      x=120, y=130  (top left — gatekeeper)
willingness:  x=560, y=130  (top right)
capacity:     x=200, y=270  (middle left)
shelter:      x=340, y=310  (middle centre)
foodwater:    x=480, y=270  (middle right)
medical:      x=340, y=390  (bottom centre)
```

---

## Readiness score

Calculate and display an overall **Destination Readiness** percentage bar.

**Algorithm:**
1. For each assessed factor (status ≠ unknown), assign a score: `operational=1.0`, `partial=0.5`, `blocked=0.0`
2. Gatekeeper factors count with weight `2`, standard factors with weight `1`
3. Score = `sum(score × weight) / sum(weight)` × 100, rounded to nearest integer
4. **If any gatekeeper is blocked**, cap the total score at 20% regardless of other factors
5. Display as a coloured progress bar: green ≥70%, amber ≥40%, red <40%
6. If no factors have been assessed yet, show `—` instead of a percentage

---

## UI layout

```
┌──────────────────────────────────────────┐
│  [SVG network diagram]                   │
│                                          │
│  Overall destination readiness  [===  ] 62%│
│                                          │
│  ● Operational  ● Partial  ● Blocked  ● Unknown  (legend) │
│                                          │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ │
│  │ Security │ │ Consent  │ │Willingness│ │
│  │[Op][Pa][Bl][Un]       │ │          │ │
│  └──────────┘ └──────────┘ └──────────┘ │
│  ... (all 7 factor cards in a grid) ...  │
└──────────────────────────────────────────┘
```

Each factor card contains:
- Factor name (bold)
- A note if it is a gatekeeper: `(gatekeeper)`
- Four status buttons: Operational / Partial / Blocked / Unknown
- The active status button should be visually highlighted in the corresponding colour

---

## Extension points (build these later)

The following features are **not required in the initial build** but should be architected so they can be added without a rewrite. Leave clear comments in the code indicating where each extension would slot in.

### 1. Destination name + notes per factor
- Add a text input at the top for the destination name (e.g. "Camp A / Town B")
- Add an optional short notes field per factor card (e.g. "Shelter partially available, 200 capacity only")
- These notes should appear in any export

### 2. Multi-destination comparison
- Allow the user to add multiple destinations (tabs or a dropdown selector)
- Each destination has its own independent set of factor statuses
- A summary comparison table shows all destinations side by side with colour-coded cells

### 3. Export / report generation
- Add an "Export" button that generates either:
  - A plain text summary (destination name, each factor status and notes, overall score)
  - A JSON snapshot of the current assessment state
- This allows field teams to share assessments via email or paste into a coordination system

### 4. Spreadsheet / CSV import
- Add a CSV import option where each row is a destination and columns are factor statuses
- This allows bulk loading of assessments from Excel or Google Sheets
- Expected column headers: `destination`, `security`, `consent`, `willingness`, `capacity`, `shelter`, `foodwater`, `medical`
- Valid cell values: `operational`, `partial`, `blocked`, `unknown`

---

## Design guidance

- Keep the visual language clean and functional — this is a field tool, not a dashboard
- Use sentence case throughout (no ALL CAPS)
- Ensure the tool works on both desktop and tablet (responsive layout)
- Use accessible colour contrast — do not rely on colour alone to communicate status (use dashed lines, opacity, and button labels as secondary indicators)
- Dark mode support is a nice-to-have but not required in v1

---

## Deliverable

A single `index.html` file that can be opened in any modern browser with no server or build step required.

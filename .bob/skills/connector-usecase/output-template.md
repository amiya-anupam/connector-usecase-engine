# Output Template — HTML Layout Rules

> This file defines the exact structure, CSS classes, and content rules Bob must follow
> when generating connector use case HTML output.
> The canonical reference file is: `templates/usecase-template.html`
> **Every generated HTML file must be structurally identical to that reference — no exceptions.**

---

## 1. Document Shell

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Connector Use Cases — {QUARTER} — IBM App Connect Enterprise</title>
<style>
  /* PASTE FULL CSS FROM templates/usecase-template.html — DO NOT MODIFY */
</style>
</head>
<body>
<div class="page">
  <!-- h1 -->
  <!-- .subtitle -->
  <!-- .summary-bar -->
  <!-- .note-box (scope note) -->
  <!-- .toc (only when 2+ connectors) -->
  <!-- track sections -->
  <!-- <hr /> -->
  <!-- .footer -->
</div>
</body>
</html>
```

**Rule:** Copy the CSS block verbatim from `templates/usecase-template.html`. Never rewrite, shorten, or modify it.

---

## 2. Page Header

### `<h1>`
```
Connector Use Cases — {QUARTER}
```
Example: `Connector Use Cases — Q3 2027`

### `.subtitle`
```
IBM App Connect Enterprise · Phase {N} · {X} Connectors · {track summary} · Market-validated enterprise integration patterns with competitive intelligence
```
- Track summary lists each track with connector count, e.g. `Track A: 4 P2 Roadmap · Track B: 16 P3 Roadmap · Track D: 8 Proposed P2 Pull-Forward`
- For single-connector runs: omit track summary, use `1 Connector · {Priority} · {Category}`

---

## 3. Summary Bar (`.summary-bar`)

Always exactly **5 stat boxes** in this order:

| Box | `.num` colour | `.lbl` text | Value |
|-----|--------------|-------------|-------|
| 1 | `#3b82d4` | Connectors | Total connector count this run |
| 2 | `#6d28d9` | Use Cases | Total use cases (connectors × avg per connector) |
| 3 | `#15803d` | Roadmap (v{N}) | Count of roadmap connectors only |
| 4 | `#92400e` | Proposed Pull-Forward | Count of Track D connectors (0 if none) |
| 5 | `#1f2328` | Catalog Connectors | Always `151` (current full ACE catalog size) |

---

## 4. Scope Note (`.note-box`)

Required. Always appears directly after `.summary-bar`. Wording must follow this pattern:

```
All companion connectors in every flow are drawn exclusively from the existing IBM App Connect
connector catalog (151 connectors). Use cases are grounded in B2B enterprise integration demand —
validated against market research, competitive iPaaS analysis (MuleSoft, Boomi, Workato,
Informatica, Tray.ai), and high-frequency enterprise pain points. Each connector block opens
with an orange competitive gap analysis, followed by {N} use cases each with: integration flow,
connector pills, business need, and quantified value box. {TRACK_D_NOTE_IF_APPLICABLE}
```

Track D note (append only if Track D connectors present):
```
<strong>Track D connectors are proposed additions</strong> pending product leadership approval —
they are not yet in the v{N} roadmap but are recommended for immediate inclusion based on P2
priority gaps identified in the Community Connector Priority Register.
```

---

## 5. Table of Contents (`.toc`)

**Required when 2 or more connectors.** Omit for single-connector runs.

```html
<div class="toc">
  <h4>Contents</h4>
  <div class="toc-grid">
    <span class="toc-section">Track A — P2 Roadmap ({N})</span>
    <span class="toc-item">{N}. {ConnectorName}</span>
    ...
    <span class="toc-section">Track B — P3 Roadmap ({N})</span>
    <span class="toc-item">{N}. {ConnectorName}</span>
    ...
    <span class="toc-section">Track D — Proposed P2 Pull-Forward ({N})</span>
    <span class="toc-item">{N}. {ConnectorName}</span>
    ...
  </div>
</div>
```

**Rules:**
- `.toc-section` for each track heading
- `.toc-item` for each connector within the track
- Sequential numbering across all tracks (do not restart per track)
- Include only tracks that have connectors in this run

---

## 6. Track Section (`.track-section`)

One `.track-section` div per track. Tracks used:

| Track | Label | Badge colour | When to use |
|-------|-------|-------------|-------------|
| A | P2 Roadmap Connectors | `.badge-violet` | P2 connectors confirmed in roadmap |
| B | P3 Roadmap Connectors | `.badge-gray` | P3 connectors confirmed in roadmap |
| D | Proposed P2 Pull-Forward | `.badge-orange` | P2 connectors not yet in roadmap, proposed for inclusion |

```html
<div class="track-section">
  <div class="track-header">
    <h2>Track {X} — {Label}</h2>
    <span class="badge badge-{colour}">{N} Connectors</span>
    <span class="badge badge-gray">{Category1} · {Category2} · ...</span>
  </div>
  <div class="note-box" style="margin-bottom:20px">
    <strong>{Priority} — {Short label}.</strong> {1–2 sentence track description.}
  </div>
  <!-- connector blocks for this track -->
</div>
```

**Track note wording:**
- Track A: `<strong>P2 — Should Promote.</strong> These {N} connectors are confirmed in the {QUARTER} roadmap (v{N}). ...`
- Track B: `<strong>P3 — Roadmap.</strong> These {N} connectors are scheduled in the {QUARTER} roadmap. ...`
- Track D: `<strong>Proposed P2 Pull-Forward.</strong> These {N} connectors are not yet in the v{N} roadmap but are proposed for immediate inclusion...`

---

## 7. Competitive Intelligence Box (`.ci-box`)

**Mandatory for every connector. Must appear immediately before its `.connector-card`.**

```html
<div class="ci-box">
  <strong>{sequential_number} / {ConnectorName} — Competitive gap:</strong>
  {2–4 sentence competitive analysis covering:
    1. What the product is + market position (customer count, market share, or Fortune X stat)
    2. Why enterprises need this connector (specific use case demand)
    3. What competitors have (MuleSoft / Boomi / Workato / Informatica — be specific)
    4. IBM App Connect's current state (community / missing / net-new) and why promotion/build matters}
</div>
```

**Rules:**
- Never skip this block
- Cite real market data (customer counts, transaction volumes, market share %)
- Always name at least 2 competitors and their connector status
- End with the IBM ACE gap statement and business consequence

---

## 8. Connector Card (`.connector-card`)

```html
<div class="connector-card">
  <div class="card-header">
    <h3>{sequential_number} — {ConnectorName}</h3>
    <div class="card-meta">
      <span class="{priority_class}">{P1|P2|P3}</span>
      <span class="type-tag">{Action & Event | Action | Event}</span>
      <span class="badge {status_badge_class}">{status_label}</span>
      <span class="badge badge-gray">{Category1} · {Category2} · {Category3}</span>
    </div>
  </div>
  <div class="use-cases">
    <!-- 4–5 .use-case blocks -->
  </div>
  <div class="actions-section">
    <!-- actions table -->
  </div>
</div>
```

### Priority badge classes

| Priority | Class |
|----------|-------|
| P1 | `.p1` |
| P2 | `.p2` |
| P3 | `.p3` |

### Status badge classes

| Status | Class | Label |
|--------|-------|-------|
| Net-new build | `.badge-blue` | `Net-New` |
| Community promoted | `.badge-green` | `Promoted ★` |
| Proposed (Track D) | `.badge-orange` | `Proposed ◆` |

---

## 9. Use Case Block (`.use-case`)

Repeat **4–5 times** per connector. Each use case has exactly 4 rows + 1 value box:

```html
<div class="use-case">
  <div class="uc-title">UC{N} — {ConnectorName} {short description verb phrase}</div>

  <div class="uc-row">
    <span class="uc-label">Flow</span>
    <span class="uc-value">
      {Step 1 trigger (source system + event)} → App Connect {action} →
      updates/creates in <strong>{SystemName}</strong> →
      notifies via <strong>{SystemName}</strong> →
      {final step}.
    </span>
  </div>

  <div class="uc-row">
    <span class="uc-label">Connectors</span>
    <span class="uc-value">
      <div class="connector-pills">
        <span class="{featured_pill_class}">{FeaturedConnector} {symbol}</span>
        <span class="cpill">{CompanionConnector1}</span>
        <span class="cpill">{CompanionConnector2}</span>
        ...
      </div>
    </span>
  </div>

  <div class="uc-row">
    <span class="uc-label">Business Need</span>
    <span class="uc-value">{2–3 sentences. Include a quantified pain point: time lost, error rate, regulatory requirement, or cost.}</span>
  </div>

  <div class="value-box">
    {1–2 sentences. Quantified outcome (time saved, % reduction, compliance met) + IBM selling narrative.}
  </div>
</div>
```

### Connector pill classes

| Pill class | Symbol | When to use |
|-----------|--------|-------------|
| `.cpill-new` | `◆` | The featured connector being documented (net-new) |
| `.cpill-pro` | `★` | The featured connector being documented (promoted) |
| `.cpill-prop` | `◆` | The featured connector (proposed Track D) |
| `.cpill` | *(none)* | Every companion connector from the ACE catalog |

**Rules:**
- Featured connector pill always appears first in the pill row
- All companion connectors must be from the ACE managed catalog or roadmap
- If a required connector is not available, use the substitution from `connector-catalog.md` and note it
- Bold companion connector names in the Flow text using `<strong>`
- Minimum 3 connector pills per use case, maximum 7

### Flow writing rules
- Start with the trigger event (who/what initiates)
- Use `→` arrows between steps
- Bold every connector name mentioned with `<strong>`
- 3–6 steps per flow
- End with a business outcome action (notification, report, record update)

### Business Need writing rules
- Always include at least one quantified data point (stat, time, cost, or regulatory citation)
- Reference a real market pain (manual process, compliance requirement, competitive pressure)
- 2–3 sentences maximum

### Value Box writing rules
- Lead with a quantified outcome (e.g. "Reduces processing time from X to Y")
- Follow with the IBM selling narrative (joint-sell story, vertical relevance, competitive differentiation)
- 1–2 sentences only

---

## 10. Actions Required Table (`.actions-section`)

Always the last section inside `.connector-card`. Always 3 columns.

```html
<div class="actions-section">
  <div style="font-size:11px;font-weight:700;text-transform:uppercase;letter-spacing:0.04em;color:#57606a;margin-bottom:4px">
    Actions Required
  </div>
  <table class="actions-table">
    <thead>
      <tr><th>Action</th><th>Owner</th><th>Timeline</th></tr>
    </thead>
    <tbody>
      <tr><td>{action description}</td><td>{owner role}</td><td>{quarter or sprint}</td></tr>
      ...
    </tbody>
  </table>
</div>
```

**Rules:**
- 2–4 rows per connector
- Owner options: `ACE PM`, `ACE Engineering`, `ACE Connector Engineering`, `IBM Partnerships`, `IBM Legal`
- Timeline format: `{Quarter YYYY}` or `{Quarter YYYY} (prep)` or `{Quarter YYYY} (fast-track, 1–2 weeks)`
- For net-new connectors: include commercial API agreement, sandbox/dev access, and build tasks
- For promotions: include promotion programme task + API compatibility validation

---

## 11. Section Divider & Footer

After all track sections:

```html
<hr />

<div class="footer">
  Made with IBM Bob &nbsp;·&nbsp; {QUARTER} Connector Use Cases ({roadmap_count} roadmap + {proposed_count} proposed) &nbsp;·&nbsp; Market-validated enterprise integration patterns
</div>
```

**Footer rule:** Always `Made with IBM Bob` — never change this text.

---

## 12. Single-Connector Run Rules

When generating output for **1 connector only**:

- **Omit** `.toc`
- **Omit** track section wrapper — render `.ci-box` + `.connector-card` directly under the scope note
- **Omit** `<hr />` track dividers
- **.subtitle** uses: `IBM App Connect Enterprise · 1 Connector · {Priority} · {Category} · Market-validated enterprise integration pattern`
- **Summary bar**: boxes 1 and 2 show `1` and the use case count (4 or 5); boxes 3–5 unchanged
- **Footer**: `Made with IBM Bob · {QUARTER} Connector Use Case · {ConnectorName} · Market-validated enterprise integration pattern`

---

## 13. Multi-Connector Run Rules

When generating output for **2–30 connectors**:

- Group connectors into tracks based on priority and roadmap status
- Always render tracks in order: A → B → D
- Separate tracks with `<hr />`
- Sequential numbering across all tracks (never restart)
- TOC always required
- Summary bar reflects totals across all tracks

---

## 14. Content Quality Checklist

Before finalising any generated HTML, verify:

- [ ] Every connector has a `.ci-box` with ≥2 competitor references and real market data
- [ ] Every use case Flow has ≥3 `<strong>` companion connector names
- [ ] Every use case has a `.value-box` with a quantified outcome
- [ ] All companion connectors are from the ACE catalog (check `connector-catalog.md`)
- [ ] If a required connector is unavailable, substitution is applied and noted in the flow
- [ ] Priority badges match the connector's entry in `connector-catalog.md`
- [ ] Status badges match (Net-New / Promoted / Proposed)
- [ ] Actions table has 2–4 rows with correct owners and timelines
- [ ] Footer reads `Made with IBM Bob`
- [ ] CSS is copied verbatim from `templates/usecase-template.html`

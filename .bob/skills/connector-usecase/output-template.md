# Output Template — HTML Layout Rules

> This file defines the exact structure, CSS classes, and content rules Bob must follow
> when generating use case HTML output for both Connector and Product paths.
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
<title>{ProductName} — {Mode} Use Cases</title>
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
  <!-- sections (tracks / lenses) -->
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

**Connector path:** `{ProductName} — Connector Use Cases`
**Product path:** `{ProductName} — Product Use Cases`

### `.subtitle`

**Connector path (has catalog):**
```
{ProductVendor} {ProductName} · {ProductCategory} · {N} Connectors · Catalog-based · Market-validated integration patterns with gap analysis
```

**Connector path (no catalog):**
```
{ProductVendor} {ProductName} · {ProductCategory} · {N} Adapters · Adapter-based · Market-validated integration patterns with gap analysis
```

**Product path:**
```
{ProductVendor} {ProductName} · {ProductCategory} · {N} Use Cases · Gaps · Opportunities · Integration-Enabled · Strategic use case analysis
```

---

## 3. Summary Bar (`.summary-bar`)

Always exactly **5 stat boxes**. Layout varies by path:

**Connector path:**

| Box | `.num` colour | `.lbl` text | Value |
|-----|--------------|-------------|-------|
| 1 | `#3b82d4` | Connectors | Total connector/adapter items documented |
| 2 | `#6d28d9` | Use Cases | Total use cases written |
| 3 | `#15803d` | Existing | Count of Known/Available connectors or adapters |
| 4 | `#92400e` | Gaps Identified | Count of Gap items |
| 5 | `#1f2328` | Ecosystem Connectors | Total connectors discovered in `integration_ecosystem` (or `151` for IBM App Connect / ACE) |

**Product path:**

| Box | `.num` colour | `.lbl` text | Value |
|-----|--------------|-------------|-------|
| 1 | `#3b82d4` | Use Cases | Total use cases written |
| 2 | `#6d28d9` | Gaps | Count of Lens 1 use cases |
| 3 | `#15803d` | Opportunities | Count of Lens 2 use cases |
| 4 | `#92400e` | Integration-Enabled | Count of Lens 3 use cases |
| 5 | `#1f2328` | Ecosystem Connectors | Total connectors discovered in `integration_ecosystem` (or `151` for IBM App Connect / ACE) |

---

## 4. Scope Note (`.note-box`)

Required. Always appears directly after `.summary-bar`. Wording adapts to path:

**Connector path (has catalog):**
```
All companion connectors in every flow are drawn from {ProductName}'s own integration ecosystem
({N} connectors discovered via research). Use cases are grounded in B2B enterprise integration demand —
validated against market research, competitive iPaaS analysis (MuleSoft, Boomi, Workato, Informatica,
Tray.ai), and high-frequency enterprise pain points. Each connector block opens with a gap analysis box,
followed by {N} use cases each with: integration flow, connector pills, business need, and quantified
value box.
```

**Connector path (no catalog):**
```
Integration capabilities are reconstructed from official documentation, release notes, community sources,
and competitor comparisons. Items marked ⚠️ are inferred from community or partner documentation —
verify with the product team before using in customer-facing materials. Companion connectors in flows
are drawn from {ProductName}'s integration ecosystem discovered via research.
```

**Product path:**
```
Use cases are organised across three analytical lenses: Gaps (unmet user needs), Opportunities
(emerging directions), and Integration-Enabled (new scenarios via connected systems). Companion
connectors where referenced are drawn from {ProductName}'s own integration ecosystem discovered
via research. All flows are grounded in market research and competitive analysis.
```

---

## 5. Table of Contents (`.toc`)

**Required when 2 or more items.** Omit for single-item runs.

**Connector path (has catalog):**
```html
<div class="toc">
  <h4>Contents</h4>
  <div class="toc-grid">
    <span class="toc-section">Existing Connectors ({N})</span>
    <span class="toc-item">{N}. {ConnectorName}</span>
    ...
    <span class="toc-section">Gaps ({N})</span>
    <span class="toc-item">{N}. {ConnectorName}</span>
    ...
  </div>
</div>
```

**Connector path (no catalog):**
```html
<div class="toc">
  <h4>Contents</h4>
  <div class="toc-grid">
    <span class="toc-section">Known Adapters ({N})</span>
    <span class="toc-item">{N}. {AdapterName}</span>
    ...
    <span class="toc-section">Inferred Capabilities ({N})</span>
    <span class="toc-item">{N}. {AdapterName}</span>
    ...
    <span class="toc-section">Gaps ({N})</span>
    <span class="toc-item">{N}. {AdapterName}</span>
    ...
  </div>
</div>
```

**Product path:**
```html
<div class="toc">
  <h4>Contents</h4>
  <div class="toc-grid">
    <span class="toc-section">Gaps ({N})</span>
    <span class="toc-item">{N}. {UseCaseTitle}</span>
    ...
    <span class="toc-section">Opportunities ({N})</span>
    <span class="toc-item">{N}. {UseCaseTitle}</span>
    ...
    <span class="toc-section">Integration-Enabled ({N})</span>
    <span class="toc-item">{N}. {UseCaseTitle}</span>
    ...
  </div>
</div>
```

**Rules:**
- `.toc-section` for each section heading
- `.toc-item` for each item within the section
- Sequential numbering across all sections (do not restart per section)
- Include only sections that have items in this run

---

## 6. Section Wrappers (`.track-section`)

One `.track-section` div per section. Section labels vary by path:

**Connector path (has catalog):**

| Section | Label | Badge colour |
|---------|-------|-------------|
| Existing | Existing Connectors | `.badge-green` |
| Gaps | Gaps — Missing Connectors | `.badge-orange` |

**Connector path (no catalog):**

| Section | Label | Badge colour |
|---------|-------|-------------|
| Known | Known Adapters | `.badge-green` |
| Inferred | Inferred Capabilities | `.badge-orange` |
| Gaps | Gaps — Missing Adapters | `.badge-red` |

**Product path:**

| Section | Label | Badge colour |
|---------|-------|-------------|
| Gaps | Gaps — What {ProductName} Is Not Solving Today | `.badge-red` |
| Opportunities | Opportunities — What {ProductName} Could Do | `.badge-green` |
| Integration-Enabled | Integration-Enabled — New Scenarios via Connected Systems | `.badge-blue` |

```html
<div class="track-section">
  <div class="track-header">
    <h2>{Section Label}</h2>
    <span class="badge badge-{colour}">{N} Items</span>
    <span class="badge badge-gray">{Category1} · {Category2} · ...</span>
  </div>
  <div class="note-box" style="margin-bottom:20px">
    <strong>{Short label}.</strong> {1–2 sentence section description.}
  </div>
  <!-- item blocks for this section -->
</div>
```

---

## 7. Analysis Box (`.ci-box`)

**Mandatory for every item. Must appear immediately before its `.connector-card`.**

The label text and framing adapt to the path and section. All variants use the same `.ci-box` CSS class.

| Path | Section | Label format | Content focus |
|------|---------|-------------|---------------|
| Connector (catalog) | Existing | `{N} / {ConnectorName} — Integration value:` | What the connector enables, market position, why enterprises use it |
| Connector (catalog) | Gaps | `{N} / {ConnectorName} — Gap:` | What is missing, which competitor has it, enterprise pain unaddressed |
| Connector (no catalog) | Known | `{N} / {AdapterName} — Integration value:` | What the adapter enables, where it is documented |
| Connector (no catalog) | Inferred | `{N} / {AdapterName} — Inferred capability:` | Source of inference, confidence level, verification note |
| Connector (no catalog) | Gaps | `{N} / {AdapterName} — Gap:` | What is missing vs competitor adapter coverage |
| Product | Gaps | `{N} / {UseCaseTitle} — Gap Analysis:` | What users can't do today, quantified pain, source of evidence |
| Product | Opportunities | `{N} / {UseCaseTitle} — Opportunity:` | Market trend or competitor direction, why this matters now |
| Product | Integration-Enabled | `{N} / {UseCaseTitle} — Integration-Enabled:` | What systems connect, what the combined capability unlocks |

```html
<div class="ci-box">
  <strong>{sequential_number} / {Name} — {Label}:</strong>
  {2–4 sentences per the content focus for this section type.}
</div>
```

**Rules:**
- Never skip this block
- Cite real market data where available (customer counts, market share %, user quotes, analyst stats)
- For Gap sections: always name at least one competitor that has what is missing
- For Inferred sections: always include the source and the verification note

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

### Status badge classes — Connector path

| Status | Class | Label |
|--------|-------|-------|
| Existing (catalog) | `.badge-green` | `Existing ★` |
| Missing / gap | `.badge-orange` | `Missing ◆` |
| Known adapter | `.badge-green` | `Available ★` |
| Inferred adapter | `.badge-orange` | `Inferred ⚠` |
| Gap adapter | `.badge-red` | `Missing ◆` |

### Status badge classes — Product path

| Lens | Class | Label |
|------|-------|-------|
| Gap | `.badge-red` | `Gap ▲` |
| Opportunity | `.badge-green` | `Opportunity ★` |
| Integration-Enabled | `.badge-blue` | `Integration ◆` |

---

## 9. Use Case Block (`.use-case`)

Repeat **4–5 times** per connector. Each use case has exactly 4 rows + 1 value box:

```html
<div class="use-case">
  <div class="uc-title">UC{N} — {ConnectorName} {short description verb phrase}</div>

  <div class="uc-row">
    <span class="uc-label">Flow</span>
    <span class="uc-value">
      {Step 1 trigger (source system + event)} → <strong>{ProductName}</strong> {action} →
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
| `.cpill-new` | `◆` | Integration-Enabled featured item (product path Lens 3) |
| `.cpill-pro` | `★` | Existing / Known / Opportunity featured item |
| `.cpill-prop` | `◆` | Gap / Inferred / Missing featured item |
| `.cpill` | *(none)* | Every companion connector from the product's integration ecosystem (or `connector-catalog.md` for IBM App Connect / ACE only) |

**Rules:**
- Featured item pill always appears first in the pill row
- All companion connectors must come from the product's own integration ecosystem (discovered in Steps 1–2 and C2a/C2b). For IBM App Connect / ACE only, use `connector-catalog.md` as the companion source instead.
- If a required companion is unavailable, use the nearest equivalent from the same source and note it with `(substitute: {original})`
- Bold every system/connector name in the Flow text using `<strong>`
- Minimum 2 connector pills per use case, maximum 7
- **Product path:** If a use case has no integration flow, omit the Connectors row entirely

### Flow writing rules
- Start with the trigger event (who/what initiates)
- Use `→` arrows between steps
- Bold **every** system or connector name mentioned with `<strong>` (applies to both paths)
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

Always the last section inside `.connector-card`. Always **2 columns** — `Action` and `Owner` only. No Timeline column.

```html
<div class="actions-section">
  <div style="font-size:11px;font-weight:700;text-transform:uppercase;letter-spacing:0.04em;color:#57606a;margin-bottom:4px">
    Actions Required
  </div>
  <table class="actions-table">
    <thead>
      <tr><th>Action</th><th>Owner</th></tr>
    </thead>
    <tbody>
      <tr><td>{action description}</td><td>{owner role}</td></tr>
      ...
    </tbody>
  </table>
</div>
```

**Rules:**
- 2–4 rows per item
- **Connector path owners:** `Product Manager`, `Connector Engineering`, `Engineering`, `Partnerships`, `Legal`
- **Product path owners:** `Product Manager`, `Engineering`, `Design`, `Strategy`, `Partnerships`
- No Timeline column anywhere — not in header, not in rows

---

## 11. Section Divider & Footer

After all sections:

```html
<hr />

<div class="footer">
  Made with IBM Bob &nbsp;·&nbsp; {ProductName} {Mode} Use Cases &nbsp;·&nbsp; {tagline}
</div>
```

**Tagline by path:**
- Connector path: `Market-validated enterprise integration patterns`
- Product path: `Gaps · Opportunities · Integration`

**Footer rule:** Always `Made with IBM Bob` — never change this text.

---

## 12. Single-Item Run Rules

When generating output for **1 connector or 1 use case only**:

- **Omit** `.toc`
- **Omit** section wrapper — render `.ci-box` + `.connector-card` directly under the scope note
- **Omit** `<hr />` section dividers
- **Summary bar**: box 1 shows `1`; remaining boxes reflect counts for that single item

---

## 13. Multi-Item Run Rules

When generating output for **2 or more items**:

- Group items into sections based on path (Existing/Gaps or Gaps/Opportunities/Integration-Enabled)
- Always render sections in the order defined in Section 6
- Separate sections with `<hr />`
- Sequential numbering across all sections (never restart)
- TOC always required
- Summary bar reflects totals across all sections

---

## 14. Content Quality Checklist

Before finalising any generated HTML, verify:

- [ ] Every item has a `.ci-box` with the correct label for its section type
- [ ] `.ci-box` cites real market data, user evidence, or competitor reference where applicable
- [ ] Every use case Flow has `<strong>` tags on every system/connector name mentioned
- [ ] Every use case has a `.value-box` with a quantified outcome
- [ ] All companion connectors come from the product's discovered integration ecosystem (or `connector-catalog.md` for IBM App Connect / ACE only)
- [ ] If a companion is unavailable, substitution is applied from the same source and noted in the flow
- [ ] Status badges match the section type (Existing ★ / Missing ◆ / Available ★ / Inferred ⚠ / Gap ▲ / Opportunity ★ / Integration ◆)
- [ ] Actions table has 2 columns only — Action + Owner, no Timeline
- [ ] Actions table has 2–4 rows with correct owner labels for the path
- [ ] Inferred adapter use cases include the verification annotation
- [ ] Product path use cases with no integration omit the Connectors pills row
- [ ] Footer reads `Made with IBM Bob`
- [ ] CSS is copied verbatim from `templates/usecase-template.html`

---

## 15. Product Use Case Mode — Additional Rules

Applies when `generation_mode = B`. Section structure, badges, pills, summary bar, and filename
are defined in Sections 3, 6, 8, and 11–13. This section covers only what is not specified there.

**`.ci-box` evidence requirements per lens** (extends Section 7 rules):
- Gaps: must cite explicit user evidence — a review quote, stat, community thread, or analyst finding
- Opportunities: must reference a specific trend source or named competitor direction
- Integration-Enabled: must name the connected systems and describe what the combined capability unlocks

**`card-header` for product use cases** (extends Section 8):
- Omit the priority badge (`.p1` / `.p2` / `.p3`) — not applicable to product use cases
- Status badge shows lens type per Section 8 Product path table
- Category tags: 2–3 business domain tags (e.g. `Analytics · Reporting · Self-service`)

---

## 16. No-Catalog Discovery Mode — Additional Rules

Applies when `generation_mode = A` and user answered NO to the catalog check. Section structure,
badges, pills, and section order are defined in Sections 6, 7, and 8. This section covers only
what is not specified there.

**Required annotation string for Inferred items** — must appear verbatim in every Inferred use case,
either inside `.uc-value` or as a note immediately after the `.value-box`:
```
(based on community-documented adapter support — verify with product team)
```

**Required discovery source note** — must appear in the `.ci-box` for every Inferred item:
```
This adapter is referenced in {source type} — {source description}. Not confirmed in official product documentation.
```

Omit any section (Known / Inferred / Gaps) that has zero items in the run.

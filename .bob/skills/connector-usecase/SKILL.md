---
name: connector-usecase
description: >
  Use when generating IBM App Connect connector use cases for a quarterly roadmap cycle.
  Triggers on phrases like "generate connector use cases", "run connector-usecase",
  "write use cases for connectors", "quarterly connector use case doc", or
  "/connector-usecase". Accepts 1 connector or up to 30 connectors per run.
  Produces a fully structured HTML output file identical in layout to
  templates/usecase-template.html.
metadata:
  argument-hint: "quarter=\"Q3 2027\" connectors=\"Salesforce, SAP\" industry_focus=\"BFSI\""
---

# Connector Use Case Engine

Automates the quarterly IBM App Connect connector use case generation workflow:
market research → framework check → priority scoring → substitution analysis → HTML output.

All output HTML must be **structurally and visually identical** to `templates/usecase-template.html`.
All layout and content rules are defined in `output-template.md`.
All connector availability and substitution lookups use `connector-catalog.md`.
Both files are in the same directory as this SKILL.md.

---

## Step 0 — Parse Inputs

Read the invocation arguments. If any required input is missing, ask using `ask_followup_question` before proceeding.

**Required inputs:**
- `quarter` — the target quarter, e.g. `Q3 2027`
- `connectors` — comma-separated list of connector names, e.g. `"Salesforce, SAP Concur, Dynatrace"`

**Optional inputs:**
- `industry_focus` — comma-separated industries to prioritise in use cases, e.g. `"BFSI, Healthcare, Retail"`
- `track` — override track assignment: `A`, `B`, or `D` (if omitted, Bob assigns based on priority)
- `use_case_count` — number of use cases per connector (default: 4, max: 5)

Confirm parsed inputs back to the user in a single summary line before starting Phase 1.

---

## Step 1 — Market & Competitive Research

For **each connector** in the input list, use `mcp__tavily__tavily_search` to run the following searches.
Run searches in parallel where possible to save time.

### Search queries to run per connector (use `search_depth: "advanced"`):

1. `"{ConnectorName} enterprise integration use cases 2025 2026"`
2. `"MuleSoft Boomi Workato {ConnectorName} connector iPaaS"`
3. `"{ConnectorName} market share customers Fortune 500 enterprise"`

### Extract and record for each connector:
- Market position: customer count, market share %, notable enterprise users
- Top 2–3 competitor iPaaS connector statuses (MuleSoft / Boomi / Workato / Informatica / Tray.ai)
- Top enterprise pain points this connector addresses
- Industry verticals where demand is highest (cross-reference with `industry_focus` if provided)

### Also run one market-level search per invocation:
- `"enterprise integration trends {quarter_year} iPaaS market"`
- `"IBM App Connect Enterprise connector roadmap {quarter_year}"`

Store all findings in working memory. They feed Phase 2 (framework check) and Phase 4 (use case writing).

---

## Step 2 — IBM App Connect Framework Check

For **each connector**, use `mcp__product-knowledge_aef3__search` to check IBM's current framework support.

### Search queries per connector:

1. Query: `"IBM App Connect {ConnectorName} connector"` — sources: `["ibm_docs", "cloud_docs"]`
2. Query: `"IBM App Connect Enterprise {ConnectorName} integration flow"` — sources: `["ibm_docs", "redbooks"]`

### Determine and record for each connector:

| Check | Answer |
|-------|--------|
| Does ACE have a managed connector for this? | Yes / No / Community only |
| Is it on the roadmap? | Yes (which quarter) / No / Proposed |
| Which ACE trigger/action types are supported? | Action / Event / Both |
| Are there documented flow patterns for this connector? | Yes / No |

### Cross-reference with `connector-catalog.md`:
- Open `connector-catalog.md` using `read_file`
- Verify priority (P1 / P2 / P3) and status (Managed / Community → Managed / Net-New / Community)
- If the connector is **not in the catalog**, flag it as `Unknown` and note it for the user
- If a connector is marked as unavailable/community-only, identify the best substitute from the
  `Substitution Options` column in `connector-catalog.md`

Record: `{ connector, priority, status, track_assignment, substitute_if_needed }`

---

## Step 3 — Track Assignment & Priority Scoring

Assign each connector to a track based on framework check results:

| Track | Condition |
|-------|-----------|
| **Track A** | P2, confirmed in roadmap for this quarter or adjacent quarter |
| **Track B** | P3, confirmed in roadmap |
| **Track D** | P2, NOT in current roadmap — propose as pull-forward |

Sort connectors within each track by priority score. Use this scoring:

| Signal | Points |
|--------|--------|
| Competitor has certified connector (each) | +10 |
| Fortune 500 / enterprise market relevance | +8 |
| IBM has no current managed connector | +7 |
| P2 priority in register | +6 |
| P3 priority in register | +3 |
| IBM joint-sell story exists | +5 |
| India Stack / regulatory mandate | +4 |

Highest score within track = lowest connector number.

Present a **track assignment summary table** to the user in chat before proceeding to Phase 4.
Wait for confirmation or corrections before writing the HTML.

Example format:
```
Track A (P2 Roadmap): Dynatrace [score: 36], Nutanix [score: 31]
Track B (P3 Roadmap): SAP Concur [score: 22], Webex [score: 19]
Track D (Proposed):   Pipedrive [score: 28]
```

---

## Step 4 — Use Case Generation

For each connector (in track order), generate 4–5 use cases following the rules in `output-template.md`.

Read `output-template.md` using `read_file` before writing any use case content.

### Per connector, write:

**1. Competitive Intelligence block (`.ci-box`)**
- Use market research from Phase 1
- Must include: market position stat, why enterprises need it, ≥2 competitor statuses, IBM current gap
- Format: `{N} / {ConnectorName} — Competitive gap: {2–4 sentences}`

**2. Card header**
- Connector name with sequential number
- Priority badge: `.p1` / `.p2` / `.p3`
- Type tag: `Action & Event` (default), `Action`, or `Event` based on framework check
- Status badge: `Net-New` (blue) / `Promoted ★` (green) / `Proposed ◆` (orange)
- Category tags: 2–3 short category labels

**3. For each use case (UC1–UC4 minimum, UC5 if high-value scenario exists):**

- **Title:** `UC{N} — {ConnectorName} {integration scenario verb phrase}`
- **Flow:** 4–6 steps using `→` arrows. Bold every companion connector name with `<strong>`.
  Use real ACE catalog connectors only (verify against `connector-catalog.md`).
  If a needed connector is unavailable, use the documented substitute and note it inline.
- **Connector pills:** Featured connector first (`.cpill-new ◆` or `.cpill-pro ★`),
  then all companions as `.cpill`. Minimum 3 pills, maximum 7.
- **Business Need:** 2–3 sentences with ≥1 quantified data point (time/cost/stat/regulation).
  Reference the industry vertical from `industry_focus` if provided.
- **Value box:** 1–2 sentences. Quantified outcome + IBM selling narrative.

**4. Actions Required table**
- 2–4 rows
- Owners: `ACE PM`, `ACE Engineering`, `ACE Connector Engineering`, `IBM Partnerships`, `IBM Legal`
- Timelines aligned to the quarter being documented

### Cross-industry use case rotation:
Across the 4–5 use cases per connector, vary the industry verticals:
- UC1: Most universal (works across industries)
- UC2: BFSI / Financial Services angle (if applicable)
- UC3: DevOps / IT Operations angle (if applicable)
- UC4: Compliance / Regulatory angle (if applicable)
- UC5: Industry-specific from `industry_focus` (if provided)

---

## Step 5 — Build Summary Bar Stats

Calculate:
- **Connectors:** total count across all tracks
- **Use Cases:** sum of all use cases written
- **Roadmap (v{N}):** count of Track A + Track B connectors
- **Proposed Pull-Forward:** count of Track D connectors
- **Catalog Connectors:** always `151`

Determine the roadmap version number from `connector-catalog.md` header or default to `v7`.

---

## Step 6 — Assemble the HTML File

Read `output-template.md` using `read_file` to verify all rules before writing.
Read the full CSS block from `templates/usecase-template.html` using `read_file`
and copy it verbatim into the `<style>` tag of the output file.

Assemble the complete HTML in this exact order:
1. `<!DOCTYPE html>` shell with copied CSS
2. `<h1>` and `.subtitle`
3. `.summary-bar` (5 stat boxes)
4. `.note-box` (scope note)
5. `.toc` (omit for single-connector runs)
6. Track sections: A → B → D, each with `.track-header`, track `.note-box`, then connector blocks
7. Each connector block: `.ci-box` → `.connector-card` (header + use-cases + actions-section)
8. `<hr />` between tracks
9. `.footer` with `Made with IBM Bob`

### Output file naming convention:
```
output/{quarter-slug}-connector-use-cases.html
```
Example: `output/q3-2027-connector-use-cases.html`

Use `write_file` to write the complete HTML to the `output/` directory.

---

## Step 7 — Quality Verification

After writing the file, run the content quality checklist from `output-template.md` Section 14.

Report results to the user in chat:
```
✅ Quality check passed — {N} connectors · {N} use cases · {N} tracks
File: output/{quarter-slug}-connector-use-cases.html
```

If any check fails, fix it in the file before reporting completion.

---

## Step 8 — Git Commit

After the file passes quality checks, stage and commit it to the `dev` branch:

```bash
cd {repo_root}
git add output/{quarter-slug}-connector-use-cases.html
git commit -m "feat: {quarter} connector use cases — {connector_count} connectors, {use_case_count} use cases"
git push origin dev
```

Use `execute_command` for the git operations.

Report the commit hash and a link to the file on `dev` in chat.

---

## Step 9 — Final Summary

Report to the user:

```
## ✅ {QUARTER} Connector Use Cases Complete

| | |
|---|---|
| **File** | output/{quarter-slug}-connector-use-cases.html |
| **Connectors** | {N} ({track_summary}) |
| **Use Cases** | {N} total |
| **Committed** | dev branch — {commit_hash} |

**To merge to main** when the quarterly cycle is validated:
  gh pr create --base main --head dev --title "Release: {QUARTER} connector use cases"
```

---

## Error Handling

| Situation | Action |
|-----------|--------|
| Connector not found in `connector-catalog.md` | Flag to user, use Tavily research to infer priority, continue |
| Tavily search returns no results | Use IBM product-knowledge MCP as fallback, note limitation |
| Companion connector unavailable in catalog | Look up substitute in `connector-catalog.md`, use substitute in flow, note it with `(substitute: {original})` |
| IBM product-knowledge returns no framework docs | Mark framework support as `Unknown`, write use cases conservatively |
| More than 30 connectors requested | Warn user, suggest splitting into two runs (e.g. Track A+B in run 1, Track D in run 2) |

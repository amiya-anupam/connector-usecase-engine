---
name: connector-usecase
description: >
  Use when generating use cases for any software product — either connector use cases
  (what integrations the product supports or is missing) or product use cases
  (gaps, opportunities, and integration-enabled scenarios).
  Triggers on phrases like "generate use cases", "run connector-usecase",
  "write use cases for", "connector use cases", "product use cases", or
  "/connector-usecase". Accepts any product name. Interactively discovers
  the product, its connector model, and the user's goal before generating output.
  Produces a fully structured HTML output file identical in layout to
  templates/usecase-template.html.
metadata:
  argument-hint: "product=\"Dynatrace\" industry_focus=\"BFSI\""
---

# Use Case Engine

Automates product use case generation through interactive product discovery:
product intelligence → competitive landscape → mode selection → path-specific analysis → HTML output.

All output HTML must be **structurally and visually identical** to `templates/usecase-template.html`.
All layout and content rules are defined in `output-template.md`.
Both files are in the same directory as this SKILL.md.

`connector-catalog.md` (also in this directory) is the companion connector reference **only when
`product_name` resolves to IBM App Connect / App Connect Enterprise / ACE**. For every other
product, companion connectors come exclusively from research — `integration_ecosystem` in
`landscape_profile` and the connector/adapter list discovered in Steps C2a or C2b.

---

## Step 0 — Onboarding

Ask using `ask_followup_question` before doing anything else.

**Required — ask this:**
> "What is the name of the product you want to generate use cases for?"

Store as `product_name`.

**Optional — ask this only if not already provided:**
> "Do you want to focus on any specific industries? (e.g. BFSI, Healthcare, Retail — leave blank for all)"

Store as `industry_focus` (default: all industries).

Do not ask for quarter, connectors, track, or any other input at this stage.
Confirm back in one line: `"Got it — researching {product_name}. Running product intelligence and competitive landscape analysis..."`
Then proceed immediately to Step 1 without waiting for further input.

---

## Step 1 — Product Intelligence

Run all searches automatically. Do not ask the user anything during this step.

**Search order — IBM Docs first, Tavily fallback:**

1. `mcp__product-knowledge_aef3__search`
   - Query: `"{product_name}"` — sources: `["ibm_docs", "cloud_docs"]`
   - Query: `"{product_name} enterprise integration"` — sources: `["ibm_docs", "redbooks", "cloud_docs"]`

2. If IBM product-knowledge returns fewer than 2 useful results, run Tavily:
   - `mcp__tavily__tavily_search`: `"{product_name} product overview enterprise"` — `search_depth: "advanced"`
   - `mcp__tavily__tavily_search`: `"{product_name} customers use cases industry verticals"` — `search_depth: "advanced"`

**Synthesise and store as `product_profile`:**

| Field | What to extract |
|-------|----------------|
| `product_category` | e.g. CRM, DevOps, ITSM, Payments, Observability, ERP, Supply Chain |
| `product_description` | 1–2 sentence summary of what it does |
| `product_key_users` | who uses it — developers, finance teams, operations, etc. |
| `product_data_types` | what data it produces or consumes — events, records, metrics, transactions |
| `product_vendor` | who makes it — IBM, Salesforce, etc. |
| `product_market_size` | customer count, market share %, or notable enterprise users if available |

Store `product_profile`. It feeds all subsequent steps.

---

## Step 2 — Competitive Landscape Research

Run all searches automatically in parallel. Do not ask the user anything during this step.

**Tavily searches (use `search_depth: "advanced"` for all):**

1. `"{product_name} competitors market 2025 2026"`
2. `"{product_name} alternative tools enterprise comparison"`
3. `"{product_name} MuleSoft Boomi Workato integration connector iPaaS"`
4. `"enterprise integration trends {product_category} iPaaS market 2025 2026"`

**Synthesise and store as `landscape_profile`:**

| Field | What to extract |
|-------|----------------|
| `competitor_list` | top 3–5 competing products in the same category |
| `integration_ecosystem` | systems this product typically integrates with |
| `market_pain_points` | top 3–5 enterprise pain points the product addresses |
| `competitor_ipaas_connectors` | which iPaaS platforms have connectors for this product and what those connectors do |
| `industry_verticals` | which industries have highest demand for this product |

Store `landscape_profile`. It feeds both the Connector Path and Product Path.

---

## Step 3 — Mode Selection

Present findings to the user and ask what they want to generate.

Show a brief summary (3–4 lines) of what was found:
```
{product_name} is a {product_category} platform used by {product_key_users}.
It competes with {competitor_list[0]}, {competitor_list[1]}, and {competitor_list[2]}.
It integrates with systems like {integration_ecosystem[0..2]}.
```

Then use `ask_followup_question`:

> "What would you like to generate?"

- **Option A:** `"Connector Use Cases — analyse what integrations/connectors the product supports or is missing, and generate use cases around those"`
- **Option B:** `"Product Use Cases — explore gaps the product isn't solving, opportunities it could fulfil, and new use cases enabled by integration"`

Store selection as `generation_mode`. Branch to the appropriate path.

---

# ══════════════════════════════════════════
# CONNECTOR PATH  (generation_mode = A)
# ══════════════════════════════════════════

> **Standing instruction for the entire Connector Path:**
> **If `product_name` is IBM App Connect / ACE:** read `connector-catalog.md` once with `read_file`
> at the start of this path — use it as the companion connector source and substitution reference.
> **For all other products:** companion connectors come from `integration_ecosystem` in
> `landscape_profile` and the catalog/adapter list discovered in C2a or C2b. Do not consult
> `connector-catalog.md`. No Track A/B/D sections. No quarter anywhere in output.

## Step C1 — Catalog Check

Use `ask_followup_question`:

> "Does {product_name} have a public connector catalog or integration listing page
> (a browsable directory of connectors it supports natively)?
> Examples of products that do: IBM App Connect, MuleSoft, Boomi, Workato, Zapier.
> Examples that do not: IBM Sterling, mainframe middleware, legacy ETL tools."

- **Option YES:** `"Yes — it has a connector catalog or marketplace page"`
- **Option NO:** `"No — it uses adapters, nodes, or protocol-based integration without a formal catalog"`

Branch to **Step C2a** (YES) or **Step C2b** (NO).

---

## Step C2a — Catalog Read (YES path)

Fetch the product's connector catalog directly.

**Run these searches in parallel (use `search_depth: "advanced"`):**
1. `mcp__tavily__tavily_search`: `"{product_name} connector catalog all connectors list"`
2. `mcp__tavily__tavily_search`: `"{product_name} integration marketplace connectors directory"`
3. `mcp__product-knowledge_aef3__search`: `"{product_name} connectors"` — sources: `["ibm_docs", "cloud_docs"]`

**Also run Tavily extract** on any official catalog/marketplace page found:
- `mcp__tavily__tavily_extract` on the catalog URL to get the full connector list

**Record for each connector found:**

| Field | Value |
|-------|-------|
| `connector_name` | name of the connector |
| `category` | e.g. CRM, Payments, DevOps, Security |
| `status` | Native/Managed / Community / Partner / Third-party |
| `trigger_action` | Action / Event / Both |

Store as `catalog_connectors[]`.

Proceed to **Step C3a**.

---

## Step C3a — Catalog Gap Analysis (YES path)

Compare `catalog_connectors[]` against competitor iPaaS catalogs to find gaps.

**Run these searches in parallel:**
1. `"{product_name} missing connectors user requests community"`
2. `"MuleSoft {product_name} connector list"`
3. `"Boomi {product_name} connector list"`
4. `"Workato {product_name} connector list"`

**Produce three lists:**

**List 1 — Existing connectors and what they enable:**
For each connector in `catalog_connectors[]`, identify the top integration use case it enables based on `landscape_profile`.

**List 2 — Gaps (present in competitor catalogs, absent here):**
Connectors that MuleSoft / Boomi / Workato have for this product's ecosystem that `catalog_connectors[]` does not include.
For each gap, record: what use case is unaddressed, which competitor has it, why it matters.

**List 3 — Use cases unsolved today:**
Based on `market_pain_points` from `landscape_profile`, which pain points have no corresponding connector in the product's own catalog (i.e. `catalog_connectors[]` discovered in Step C2a)?

Proceed to **Step C4a**.

---

## Step C4a — Generate Connector Use Cases (YES path)

For each item across List 1 and List 2 from Step C3a, generate 4–5 use cases.

Apply `.ci-box` labels, status badges, and featured pills per `output-template.md` Section 7 (Connector catalog — Existing and Gaps rows).

- **List 1 items:** flows show what the connector enables end-to-end
- **List 2 items:** flows show what would become possible if this connector existed; note inline `(not yet available — competitor gap)`

Proceed to **Step C-Final**.

---

## Step C2b — Discovery Phase (NO path)

No single source of truth exists. Reconstruct the integration picture from four sources in parallel.

**Source 1 — Official docs (IBM Docs + product docs):**
- `mcp__product-knowledge_aef3__search`: `"{product_name} adapter"` — sources: `["ibm_docs", "redbooks"]`
- `mcp__product-knowledge_aef3__search`: `"{product_name} node integration protocol"` — sources: `["ibm_docs", "cloud_docs"]`
- `mcp__tavily__tavily_search`: `"{product_name} adapter documentation supported systems"` — `search_depth: "advanced"`

**Source 2 — Release notes and changelogs:**
- `mcp__tavily__tavily_search`: `"{product_name} release notes new adapter connector version history"` — `search_depth: "advanced"`

**Source 3 — Community and partner sources:**
- `mcp__tavily__tavily_search`: `"{product_name} adapter implementation guide community"` — `search_depth: "advanced"`
- `mcp__tavily__tavily_search`: `"{product_name} integration partner documentation Stack Overflow"` — `search_depth: "advanced"`

**Source 4 — Competitor comparison:**
- `mcp__tavily__tavily_search`: `"{product_name} vs {competitor_list[0]} integration adapters supported"` — `search_depth: "advanced"`
- `mcp__tavily__tavily_search`: `"{product_category} adapter support comparison enterprise"` — `search_depth: "advanced"`

**Synthesise into three confidence tiers:**

| Tier | Symbol | Meaning |
|------|--------|---------|
| Known | ✅ | Documented in official product docs or IBM Docs |
| Inferred | ⚠️ | Found only in community, partner, or third-party docs |
| Gap | ❌ | Present in competitor products, absent here |

Proceed to **Step C3b**.

---

## Step C3b — User Confirmation Gate (NO path)

Present the synthesised picture to the user and wait for confirmation before proceeding.

Format the confirmation message as:
```
Based on research, here is what I found {product_name} supports:

✅ Known (documented in official sources):
- {adapter/node/protocol name} — {brief description}
- ...

⚠️ Inferred (found only in community or partner docs):
- {adapter/node/protocol name} — {source reference}
- ...

❌ Gaps vs competitors:
- {adapter/node/protocol name} — {competitor that has it} has this; {product_name} does not
- ...

Is this picture accurate? Anything to add or correct?
```

Use `ask_followup_question`:
- **Option A:** `"Yes, this looks accurate — proceed"`
- **Option B:** `"I have corrections — let me provide them"`

If Option B, accept the user's corrections and update the picture before proceeding.

Proceed to **Step C4b**.

---

## Step C4b — Gap Analysis (NO path)

Using the confirmed picture from Step C3b, produce three lists:

**List 1 — Discovered capabilities and what they enable:**
For each ✅ Known item, identify the top use case it enables based on `landscape_profile`.

**List 2 — Inferred capabilities (lower confidence):**
For each ⚠️ Inferred item, identify the use case it could enable.
All use cases from this list must be annotated: `(based on community-documented adapter support — verify with product team)`

**List 3 — Gaps:**
For each ❌ Gap item, describe: what use case is unaddressed, which competitor covers it, why it matters to enterprises.

Proceed to **Step C5b**.

---

## Step C5b — Generate Node/Connector Use Cases (NO path)

For each item across List 1, List 2, and List 3 from Step C4b, generate 4–5 use cases.

Apply `.ci-box` labels, status badges, featured pills, and confidence annotations per `output-template.md` Sections 7 and 16 (no-catalog rows and annotation rules).

- **List 1 (Known):** flows show what the adapter enables end-to-end
- **List 2 (Inferred):** flows show what the adapter could enable; every use case must include the annotation from `output-template.md` Section 16
- **List 3 (Gaps):** flows show what would become possible if this adapter existed; note inline `(not yet available — gap vs {CompetitorName})`

Proceed to **Step C-Final**.

---

## Step C-Final — Connector Path HTML Assembly

Read `output-template.md` using `read_file` and `templates/usecase-template.html` using `read_file`.
Assemble the HTML file strictly following all rules in `output-template.md` (Sections 1–14, 16).
Use the **Connector path** variants for header, subtitle, summary bar, scope note, TOC, and section wrappers.

Proceed to **Step Final — Quality, Commit, Summary**.

---

# ══════════════════════════════════════════
# PRODUCT PATH  (generation_mode = B)
# ══════════════════════════════════════════

> **Standing instruction for the entire Product Path:**
> **If `product_name` is IBM App Connect / ACE:** read `connector-catalog.md` once with `read_file`
> at the start of this path — use it as the Lens 3 companion connector reference.
> **For all other products:** Lens 3 integration flows use `integration_ecosystem` from
> `landscape_profile` as the companion source. Do not consult `connector-catalog.md`.

## Step P1 — Three-Lens Discovery

Run all three lenses automatically in parallel. Do not ask the user anything during this step.

**Lens 1 — GAPS: What is the product NOT solving today?**

Search for documented limitations, complaints, and unmet needs:
- `mcp__tavily__tavily_search`: `"{product_name} limitations missing features user complaints 2025"` — `search_depth: "advanced"`
- `mcp__tavily__tavily_search`: `"{product_name} reviews pain points G2 Gartner TrustRadius"` — `search_depth: "advanced"`
- `mcp__tavily__tavily_search`: `"{product_name} vs {competitor_list[0]} disadvantages weaknesses"` — `search_depth: "advanced"`

Extract: specific things users say the product cannot do that they wish it could.

**Lens 2 — OPPORTUNITIES: What could the product do?**

Search for trends, adjacent markets, and unaddressed directions:
- `mcp__tavily__tavily_search`: `"{product_category} trends emerging use cases 2025 2026 enterprise"` — `search_depth: "advanced"`
- `mcp__tavily__tavily_search`: `"{product_name} roadmap future vision AI automation"` — `search_depth: "advanced"`
- `mcp__tavily__tavily_search`: `"{product_name} adjacent markets expansion opportunity"` — `search_depth: "advanced"`

Extract: new directions competitors are heading, trends unaddressed by the product, adjacent market openings.

**Lens 3 — INTEGRATION-ENABLED: What becomes possible by connecting this product to other systems?**

Using `integration_ecosystem` from `landscape_profile` as the companion connector source:
- Reason about what automation becomes possible when this product is connected to the systems its users already use
- Identify data flows that would create business value not achievable without integration
- If `product_name` is IBM App Connect / ACE, cross-reference with `connector-catalog.md` (managed connectors list) instead

Search:
- `mcp__tavily__tavily_search`: `"{product_name} integration workflow automation use case enterprise"` — `search_depth: "advanced"`

---

## Step P2 — Present Use Case Candidates

Synthesise the three lenses into 6–10 candidate use cases. Present them clearly grouped by lens.

Format:
```
## Use Case Candidates for {product_name}

**Gaps — what the product is not solving today:**
  UC-G1: {title} — {1-sentence rationale with a data point or user quote}
  UC-G2: {title} — {1-sentence rationale with a data point or user quote}

**Opportunities — what the product could do:**
  UC-O1: {title} — {1-sentence rationale referencing a trend or competitor direction}
  UC-O2: {title} — {1-sentence rationale referencing a trend or competitor direction}

**Integration-enabled — new scenarios via connected systems:**
  UC-I1: {title} — connectors involved: {list}  — {1-sentence value statement}
  UC-I2: {title} — connectors involved: {list}  — {1-sentence value statement}
```

Use `ask_followup_question`:

> "How many use cases do you want in the output, and would you like to remove any of the above candidates?"

- **Option A:** `"Proceed with all candidates (4–5 recommended per section)"`
- **Option B:** `"Use fewer — I'll specify which to keep"`

If Option B, accept the user's selections before proceeding.

---

## Step P3 — Generate Product Use Cases

For each confirmed candidate, generate a use case block per `output-template.md` rules (Sections 7–10, 15).

Apply `.ci-box` labels, status badges, and featured pills per `output-template.md` Section 7 (Product path rows).
Follow connector pill, flow, business need, and value box rules from Sections 9 and 15.
Follow actions table rules (2 columns, Product path owners) from Section 10.

---

## Step P-Final — Product Path HTML Assembly

Read `output-template.md` using `read_file` and `templates/usecase-template.html` using `read_file`.
Assemble the HTML file strictly following all rules in `output-template.md` (Sections 1–15).
Use the **Product path** variants for header, subtitle, summary bar, scope note, TOC, and section wrappers.

Proceed to **Step Final — Quality, Commit, Summary**.

---

# ══════════════════════════════════════════
# SHARED FINAL STEPS (both paths)
# ══════════════════════════════════════════

## Step Final-1 — Quality Verification

After writing the HTML file, verify against the checklist in `output-template.md` Section 14.

Report in chat:
```
✅ Quality check passed — {N} connectors/use cases · {N} sections
File: output/{filename}.html
```

If any check fails, fix it in the file before reporting.

---

## Step Final-2 — Git Commit

```bash
git add output/{filename}.html
git commit -m "feat: {product-slug} {mode} use cases — {N} items"
git push origin dev
```

Use `execute_command`. Report commit hash.

---

## Step Final-3 — Final Summary

```
## ✅ {product_name} {Mode} Use Cases Complete

| | |
|---|---|
| **File** | output/{filename}.html |
| **Product** | {product_name} ({product_category}) |
| **Mode** | {Connector Use Cases / Product Use Cases} |
| **Items** | {N} total |
| **Committed** | dev branch — {commit_hash} |

To merge to main:
  gh pr create --base main --head dev --title "{product_name} {Mode} Use Cases"
```

---

## Error Handling

| Situation | Action |
|-----------|--------|
| Product not found in IBM Docs or Tavily | Ask user for a 1–2 sentence description, use it as `product_description` and proceed |
| Tavily search returns no results | Use IBM product-knowledge MCP as fallback; if both fail, note limitation and proceed with what is known |
| Catalog page found but not extractable | Record connectors manually from search snippets; note "catalog partially extracted" |
| Fewer than 3 sources return results in discovery phase (NO path) | Inform user, present what was found, proceed with lower-confidence picture — all items marked ⚠️ Inferred |
| User confirms zero candidates in product path | Ask user to describe 2–3 use cases they have in mind; use those as seeds |
| Companion connector unavailable (App Connect product) | Look up substitute in `connector-catalog.md`, use it in flow, note with `(substitute: {original})` |
| Companion connector unavailable (any other product) | Use nearest equivalent from `integration_ecosystem` or `landscape_profile`, note with `(substitute: {original})` |
| More than 15 connectors/adapters discovered | Present top 15 by relevance; ask user to confirm or narrow scope before generation |

---
name: connector-usecase
description: >
  Use when generating use cases for any software product — either connector use cases
  (what integrations the product supports or is missing), connector candidate evaluation
  (should we build a connector for this product on a target platform?), or product use cases
  (gaps, opportunities, and integration-enabled scenarios).
  Triggers on phrases like "generate use cases", "run connector-usecase",
  "write use cases for", "connector use cases", "product use cases",
  "connector candidate", "should we build a connector for", or "/connector-usecase".
  Accepts any product name. Interactively discovers the product, the user's intent,
  and the target platform (for connector candidate evaluation) before generating output.
  Produces a fully structured HTML output file identical in layout to
  templates/usecase-template.html.
  To run the flow test suite, say "/test connector-usecase" or
  "run flow tests for connector-usecase" — Bob will read
  tests/flow-test.md and execute all 14 scenarios as dry-runs.
metadata:
  argument-hint: "product=\"Salesforce\" industry_focus=\"BFSI\""
---

# Use Case Engine

Automates product use case generation through interactive product discovery:
product intelligence → competitive landscape → intent clarification → path-specific analysis → HTML output.

All output HTML must be **structurally and visually identical** to `templates/usecase-template.html`.
All layout and content rules are defined in `output-template.md`.
Both files are in the same directory as this SKILL.md.

### companion_reference loading (intent-aware)

After Step 0 resolves `product_name` and `user_intent`, Bob derives slugs and checks for catalog files:

**For `user_intent = A` or `user_intent = C` (Connector Existing or Product path):**
- Derive `product_slug` from `product_name`.
- Run `glob(".bob/skills/connector-usecase/{product_slug}-catalog.md")`.
- If found → `read_file` and store as `companion_reference`. Use throughout as the companion connector source.
- If not found → try first two slug segments, then set `companion_reference = null`.

**For `user_intent = B` (Connector Candidate path):**
- Derive `target_platform_slug` from `target_platform` (NOT from `product_name`).
  Examples: `"IBM App Connect"` → `"ibm-app-connect"`, `"MuleSoft"` → `"mulesoft"`.
- Run `glob(".bob/skills/connector-usecase/{target_platform_slug}-catalog.md")`.
- If found → `read_file` and store as `companion_reference`. This is the platform's connector catalog — use it for gap analysis throughout the Connector Candidate Path.
- If not found → `companion_reference = null`. Use landscape research for platform context.

This mechanism requires **zero product-name conditionals** anywhere in this skill.
It works for any product/platform that has a pre-built reference file and equally for those that do not.

---

## Step 0-A — Product Name Collection

Ask using `ask_followup_question` before doing anything else.

**Required — ask this:**
> "What is the name of the product you want to work with?"

Store as `product_name`.

---

## Step 0-B — Intent Clarification Gate

Immediately after storing `product_name`, ask using `ask_followup_question`:

> "What are you trying to do with {product_name}?"

- **Option A:** `"Analyse {product_name}'s existing integrations — what it connects to, what connectors it already has, where the gaps are"`
- **Option B:** `"Evaluate {product_name} as a new connector to build for a platform — should we build it, what would it unlock?"`
- **Option C:** `"Explore product use cases — gaps {product_name} isn't solving and new scenarios integration could enable"`
- **Option D:** `"Not sure yet — explain each option"`

Store selection as `user_intent`. Branch:
- **A → Step 0-C** (slug + companion_reference for Connector Existing path)
- **B → Step 0-B2** (collect target_platform, then Step 0-C)
- **C → Step 0-C** (slug + companion_reference for Product path)
- **D → Step 0-D** (explain then ask again)

---

### Step 0-D — Explain Options

Respond in chat:
```
Here's what each option does:

**A — Connector Use Cases (Existing)**
Looks at what integrations {product_name} already has (e.g. in its connector catalog or adapter list),
finds gaps vs. competitors, and generates use cases showing what each connector enables.
Best when: {product_name} is a platform and you want to document its integration landscape.

**B — Connector Candidate Evaluation**
Evaluates {product_name} as a new connector to build for a target platform (e.g. IBM App Connect).
Researches platform gaps, API fit, competitive pressure, and demand signals.
Generates a build-recommendation card and up to 5 cross-platform use cases.
Best when: your team is deciding whether to invest in building a {product_name} connector.

**C — Product Use Cases**
Looks at what {product_name} is NOT doing today, finds opportunities via trends and adjacent markets,
and explores what becomes possible by connecting {product_name} to other systems.
Best when: you want strategic use cases for a product roadmap or pitch.
```

Then loop back to **Step 0-B** and ask the same four options again.

---

### Step 0-B2 — Collect Target Platform (Intent B only)

Ask using `ask_followup_question`:

> "Which platform are you building this {product_name} connector for?"

- **Option A:** `"IBM App Connect"`
- **Option B:** `"MuleSoft Anypoint"`
- **Option C:** `"Workato"`
- **Option D:** `"Boomi"`
- **Option E:** `"Another platform — I'll type the name"`

Store as `target_platform`. If Option E, accept the user's typed platform name.

Proceed to **Step 0-C**.

---

## Step 0-C — Slug Derivation and companion_reference Loading

1. Derive `product_slug`: convert `product_name` to lower-case, replace spaces and punctuation with hyphens, collapse multiple hyphens.
   Examples: `"Tenable"` → `"tenable"`, `"SAP S/4HANA"` → `"sap-s-4hana"`, `"IBM App Connect Enterprise"` → `"ibm-app-connect-enterprise"`.

2. **If `user_intent = B`** (Connector Candidate):
   - Derive `target_platform_slug` the same way from `target_platform`.
     Example: `"IBM App Connect"` → `"ibm-app-connect"`.
   - Run `glob(".bob/skills/connector-usecase/{target_platform_slug}-catalog.md")`.
   - If found → `read_file` it and store as `companion_reference` (this is the platform catalog).
   - If not found → `companion_reference = null`.

3. **If `user_intent = A` or `user_intent = C`**:
   - Run `glob(".bob/skills/connector-usecase/{product_slug}-catalog.md")`.
   - If found → `read_file` it and store as `companion_reference`.
   - If not found → try first two segments of `product_slug` and glob again.
   - If still not found → `companion_reference = null`.

4. Log in memory: `{ product_slug, user_intent, target_platform_slug (if B), companion_reference_file: "{slug}-catalog.md" | null }`.

**Optional — ask this only if not already provided:**
> "Do you want to focus on any specific industries? (e.g. BFSI, Healthcare, Retail — leave blank for all)"

Store as `industry_focus` (default: all industries).

Do not ask for quarter, connectors, track, or any other input at this stage.

Confirm in one line:
- If `user_intent = B`: `"Got it — evaluating {product_name} as a connector candidate for {target_platform}. Running platform gap analysis and subject intelligence..."`
- Otherwise: `"Got it — researching {product_name}. Running product intelligence and competitive landscape analysis..."`

Then proceed immediately to **Step 1** without waiting for further input.

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
| `product_category` | e.g. CRM, DevOps, ITSM, Payments, Observability, ERP, Supply Chain, Security |
| `product_description` | 1–2 sentence summary of what it does |
| `product_key_users` | who uses it — developers, finance teams, operations, security teams, etc. |
| `product_data_types` | what data it produces or consumes — events, records, metrics, transactions, vulnerabilities |
| `product_vendor` | who makes it |
| `product_market_size` | customer count, market share %, or notable enterprise users if available |
| `product_api_model` | REST API / GraphQL / webhooks / SOAP — relevant for Connector Candidate path |

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

Store `landscape_profile`. It feeds all three paths.

---

## Step 3 — Mode Routing

**If `user_intent` was set in Step 0-B (A, B, or C):** skip this step entirely and route directly:
- `user_intent = A` → **Connector Path** (Step C1)
- `user_intent = B` → **Connector Candidate Path** (Step N1)
- `user_intent = C` → **Product Path** (Step P1)

**Only show this step if the user chose Option D in Step 0-B** (not sure yet) and the clarification loop has not resolved.

Show a brief summary (3–4 lines) of what was found:
```
{product_name} is a {product_category} platform used by {product_key_users}.
It competes with {competitor_list[0]}, {competitor_list[1]}, and {competitor_list[2]}.
It integrates with systems like {integration_ecosystem[0..2]}.
```

Then use `ask_followup_question`:

> "What would you like to generate?"

- **Option A:** `"Connector Use Cases — analyse what integrations/connectors the product supports or is missing, and generate use cases around those"`
- **Option B:** `"Connector Candidate Evaluation — evaluate {product_name} as a new connector to build for a platform (e.g. IBM App Connect)"`
- **Option C:** `"Product Use Cases — explore gaps the product isn't solving, opportunities it could fulfil, and new use cases enabled by integration"`

Store selection as `generation_mode`/`user_intent`. For Option B, ask **Step 0-B2** before proceeding. Branch accordingly.

---

# ══════════════════════════════════════════
# CONNECTOR PATH  (user_intent = A)
# ══════════════════════════════════════════

> **Standing instruction for the entire Connector Path:**
> `companion_reference` was resolved in Step 0-C (runtime file discovery).
> If `companion_reference` is not null, use it as the companion connector source and substitution reference.
> If `companion_reference` is null, use `integration_ecosystem` from `landscape_profile`
> and the catalog/adapter list discovered in C2a or C2b.
> No Track A/B/D sections. No quarter anywhere in output.

## Step C1 — Catalog Check

Use `ask_followup_question`:

> "Does {product_name} have a public connector catalog or integration listing page
> (a browsable directory of connectors it supports natively)?
> Examples of products that do: iPaaS platforms like workflow automation tools, integration hubs, and marketplace-driven integration platforms.
> Examples that do not: protocol-based middleware, legacy ETL tools, or adapter-framework products."

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

For each entry in `competitor_ipaas_connectors` from `landscape_profile` (up to 4 competitors), run:
2. `"{competitor_name} {product_name} connector list"`

(Do not hardcode competitor names. Use what was discovered in Step 2.)

**Produce three lists:**

**List 1 — Existing connectors and what they enable:**
For each connector in `catalog_connectors[]`, identify the top integration use case it enables based on `landscape_profile`.

**List 2 — Gaps (present in competitor catalogs, absent here):**
Connectors that `competitor_ipaas_connectors` entries have for this product's ecosystem that `catalog_connectors[]` does not include.
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

Output filename: `output/{product-slug}-connector-use-cases.html`

Proceed to **Step Final — Quality, Commit, Summary**.

---

# ══════════════════════════════════════════
# CONNECTOR CANDIDATE PATH  (user_intent = B)
# ══════════════════════════════════════════

> **Standing instruction for the entire Connector Candidate Path:**
> The subject (`product_name`) is being evaluated as a NEW connector to build ON the `target_platform`.
> Research is platform-first: understand what the platform already has, then assess the subject's fit.
> `companion_reference` was loaded from `{target_platform_slug}-catalog.md` (not from `{product_slug}-catalog.md`).
> If `companion_reference` is not null, it is the target platform's connector catalog — use it for all gap analysis.
> **Hard cap: maximum 5 use cases in the output.** Do not exceed this.
> Output filename: `output/{product-slug}-connector-candidate-{target-platform-slug}.html`

## Step N1 — Platform Gap Analysis

Understand what the target platform already has in the same category as `{product_name}`.

**If `companion_reference` is not null:** scan it for connectors in the same `product_category` as `{product_name}`.
List all connectors in that category and note their status (Managed, Community, Planned, etc.) and planned quarter if present.

**Run these searches in parallel:**
1. `mcp__tavily__tavily_search`: `"{target_platform} {product_category} connectors list"` — `search_depth: "advanced"`
2. `mcp__tavily__tavily_search`: `"{target_platform} connector catalog 2025 2026"` — `search_depth: "advanced"`
3. `mcp__product-knowledge_aef3__search`: `"{target_platform} connector"` — sources: `["ibm_docs", "cloud_docs"]`

**Synthesise and store as `platform_gap_analysis`:**
- `existing_category_connectors[]` — connectors already on the platform in the same category as `{product_name}`
- `category_gaps[]` — what the platform is missing in this category (vs. competitor iPaaS platforms)
- `platform_audience` — who the platform's customers are (the buyer/integrator persona)

---

## Step N2 — Subject Intelligence

Research what `{product_name}` actually does, what API and data model it exposes, and what events or triggers it fires.

`product_profile` from Step 1 is already in memory. Augment it with connector-specific detail.

**Run these searches in parallel:**
1. `mcp__tavily__tavily_search`: `"{product_name} REST API documentation triggers webhooks events"` — `search_depth: "advanced"`
2. `mcp__tavily__tavily_search`: `"{product_name} API objects records data model"` — `search_depth: "advanced"`
3. `mcp__tavily__tavily_search`: `"{product_name} enterprise integration SIEM SOAR ticketing"` — `search_depth: "advanced"`

**Synthesise and store as `subject_intel`:**
- `api_model` — REST / GraphQL / SOAP / webhooks / polling
- `key_triggers` — events/webhooks the API fires (e.g. "new critical vulnerability", "asset state change")
- `key_actions` — operations a connector can perform (e.g. "get vulnerabilities", "close finding")
- `data_objects` — primary records (e.g. vulnerabilities, assets, findings, scans)
- `auth_model` — API key / OAuth2 / Basic — affects connector build complexity

---

## Step N3 — Fit Analysis

Which platform connectors does `{product_name}` pair well with? What flows become possible that don't exist today?

Using `platform_gap_analysis.existing_category_connectors` and `subject_intel.key_triggers` and `subject_intel.key_actions`:

**Reason about cross-connector flows:**
For each of the top 5–8 connectors already on `{target_platform}`, consider:
- "When `{product_name}` fires event X, what can the platform route it to?"
- "What action in `{product_name}` would complete a workflow that starts in connector Y?"

**Run these searches:**
1. `mcp__tavily__tavily_search`: `"{product_name} {target_platform} integration workflow"` — `search_depth: "advanced"`
2. `mcp__tavily__tavily_search`: `"{product_name} ServiceNow Jira Slack SIEM integration automation"` — `search_depth: "advanced"`

**Synthesise and store as `fit_analysis`:**
- `top_pairings[]` — ranked list of `{target_platform}` connectors that pair best with `{product_name}`, with rationale
- `sample_flows[]` — 5–7 concrete end-to-end flows (these become use case seeds)
- `integration_complexity` — Low / Medium / High (based on API model, auth, data volume)

---

## Step N4 — Competitive Pressure

Do MuleSoft / Boomi / Workato already have a `{product_name}` connector? What do those connectors do?

**Run these searches in parallel:**
1. `mcp__tavily__tavily_search`: `"MuleSoft {product_name} connector"` — `search_depth: "advanced"`
2. `mcp__tavily__tavily_search`: `"Boomi {product_name} connector integration"` — `search_depth: "advanced"`
3. `mcp__tavily__tavily_search`: `"Workato {product_name} recipe integration"` — `search_depth: "advanced"`
4. `mcp__tavily__tavily_search`: `"{product_name} connector iPaaS integration platform"` — `search_depth: "advanced"`

**Synthesise and store as `competitive_pressure`:**
- `has_connector_on[]` — list of iPaaS platforms that already have a `{product_name}` connector
- `competitor_connector_actions[]` — what actions/triggers those connectors support
- `competitive_gap` — is `{target_platform}` the only major iPaaS missing this connector? (yes/no + evidence)

---

## Step N5 — Demand Validation

How widely deployed is `{product_name}` among enterprises? What evidence of manual workarounds exists?

**Run these searches in parallel:**
1. `mcp__tavily__tavily_search`: `"{product_name} enterprise customers market share Fortune 500"` — `search_depth: "advanced"`
2. `mcp__tavily__tavily_search`: `"{product_name} manual integration workaround scripting community"` — `search_depth: "advanced"`
3. `mcp__tavily__tavily_search`: `"{product_name} {target_platform} integration request community feature"` — `search_depth: "advanced"`

**Synthesise and store as `demand_signals`:**
- `customer_footprint` — estimated enterprise customer count or market share
- `industry_verticals[]` — top 3 industries where `{product_name}` is deployed (use `landscape_profile.industry_verticals` if already known)
- `manual_workaround_evidence` — any documented scripting, export/import, or custom code workarounds between `{product_name}` and `{target_platform}` or its ecosystem
- `explicit_requests` — any community posts, feature requests, or forum threads asking for a `{product_name}` connector on `{target_platform}`

---

## Step N6 — Unique Angle Check

What does `{product_name}` offer that the functionally similar connectors already on `{target_platform}` don't?

Using `platform_gap_analysis.existing_category_connectors` and `companion_reference` (if not null):
- Identify the connector(s) on `{target_platform}` that are closest to `{product_name}` in category.
- Compare: what does `{product_name}` do that those connectors don't? (Different data objects? Richer API? Better enterprise auth? Different market segment?)

**Run this search:**
1. `mcp__tavily__tavily_search`: `"{product_name} vs {closest_existing_connector} comparison enterprise features"` — `search_depth: "advanced"`

**Synthesise and store as `unique_angle`:**
- `closest_platform_connector` — the most similar connector already on `{target_platform}`
- `differentiators[]` — what `{product_name}` offers that the closest connector doesn't
- `overlap_risk` — Low / Medium / High (would this connector cannibalise the existing one?)
- `recommended_positioning` — one sentence on how to position this connector in the catalog

---

## Step N-Final — Connector Candidate HTML Assembly

Read `output-template.md` using `read_file` and `templates/usecase-template.html` using `read_file`.

Assemble the HTML file as a **single connector candidate card** with the following structure.
Strictly follow all layout, typography, and colour rules from `output-template.md`.
Use the **Connector path** base layout but with Candidate-specific overrides described below.

### Header overrides
- Page title: `{product_name} — Connector Candidate Evaluation`
- Subtitle: `Target platform: {target_platform} · Category: {product_category}`
- Summary bar: show `Build Recommendation`, `Integration Complexity`, `Competitive Pressure`, `Top Pairings Count`

### Build Recommendation `.ci-box`
At the top of the content area, render a single evaluation `.ci-box` (use the featured/highlighted variant) with:
- **Recommendation:** `Build` / `Evaluate Further` / `Deprioritise` — determined by:
  - **Build**: competitive_gap = yes AND customer_footprint > 1000 enterprises AND overlap_risk ≤ Medium
  - **Evaluate Further**: one or two of those conditions are met
  - **Deprioritise**: none of the conditions are met or overlap_risk = High
- **Rationale:** 2–3 sentences combining evidence from `competitive_pressure`, `demand_signals`, and `unique_angle`
- **API Complexity:** `subject_intel.integration_complexity`
- **Closest existing connector:** `unique_angle.closest_platform_connector` with `unique_angle.overlap_risk`

### Use Cases section (hard cap: 5 maximum)

Select the **5 best** flows from `fit_analysis.sample_flows[]`, ranked by:
1. Business value (which solves the most painful enterprise problem)
2. Cross-connector richness (which uses the most platform connectors)
3. Uniqueness (which can't be done with existing platform connectors)

For each use case, generate a full use case block per `output-template.md` Sections 7–10:
- Apply `.ci-box` label, status badge (`Candidate`), and featured pill if top-ranked
- Show the full connector flow (Trigger → Steps → Action) naming `{product_name}` alongside `{target_platform}` connectors
- Include business need and value box
- Actions table must use these owner columns: `Product Manager`, `Connector Engineering`, `Partnerships`, `Engineering`

**Do NOT generate more than 5 use cases.** If `fit_analysis.sample_flows[]` contains more than 5 items, select the top 5 by the ranking criteria above and discard the rest.

### Competitive Context section
Below the use cases, add a compact table showing:
| iPaaS Platform | Has {product_name} connector? | Actions / Triggers |
with one row per entry from `competitive_pressure.has_connector_on[]` plus a row for `{target_platform}` (marked "Candidate").

### Demand Signals section
A brief evidence block showing `demand_signals.customer_footprint`, top industries, and any `demand_signals.explicit_requests`.

Output filename: `output/{product-slug}-connector-candidate-{target-platform-slug}.html`

Proceed to **Step Final — Quality, Commit, Summary**.

---

# ══════════════════════════════════════════
# PRODUCT PATH  (user_intent = C)
# ══════════════════════════════════════════

> **Standing instruction for the entire Product Path:**
> `companion_reference` was resolved in Step 0-C (runtime file discovery).
> If `companion_reference` is not null, use it as the Lens 3 companion connector reference.
> If `companion_reference` is null, use `integration_ecosystem` from `landscape_profile` for Lens 3 flows.

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

Using `companion_reference` if not null, otherwise `integration_ecosystem` from `landscape_profile`:
- Reason about what automation becomes possible when this product is connected to the systems its users already use
- Identify data flows that would create business value not achievable without integration

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

Output filename: `output/{product-slug}-product-use-cases.html`

Proceed to **Step Final — Quality, Commit, Summary**.

---

# ══════════════════════════════════════════
# SHARED FINAL STEPS (all paths)
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
## ✅ {product_name} {Mode} Complete

| | |
|---|---|
| **File** | output/{filename}.html |
| **Product** | {product_name} ({product_category}) |
| **Mode** | {Connector Use Cases / Connector Candidate Evaluation / Product Use Cases} |
| **Target Platform** | {target_platform} (Connector Candidate only) |
| **Items** | {N} total |
| **Committed** | dev branch — {commit_hash} |

To merge to main:
  gh pr create --base main --head dev --title "{product_name} {Mode}"
```

(Omit the Target Platform row if `user_intent ≠ B`.)

---

## Step Final-4 — Continue or Close

After displaying the Step Final-3 summary, use `ask_followup_question`:

> "Is there anything else you'd like to generate?"

Present only the options that are still available for this session:

- **If `user_intent = A` (Connector Existing) and Product path not yet run**, offer:
  - **Option A:** `"Yes — generate Product Use Cases for {product_name} (reuses research already done, skips straight to Lens Discovery)"`
  - **Option B:** `"Yes — evaluate {product_name} as a connector candidate for a platform"`
  - **Option C:** `"Yes — run the full flow for a different product"`
  - **Option D:** `"No — I'm done for now"`

- **If `user_intent = B` (Connector Candidate)**, offer:
  - **Option A:** `"Yes — generate Connector Use Cases for {product_name} (analyse its own integrations)"`
  - **Option B:** `"Yes — generate Product Use Cases for {product_name}"`
  - **Option C:** `"Yes — run the full flow for a different product"`
  - **Option D:** `"No — I'm done for now"`

- **If `user_intent = C` (Product) and Connector path not yet run**, offer:
  - **Option A:** `"Yes — generate Connector Use Cases for {product_name} (reuses research already done)"`
  - **Option B:** `"Yes — evaluate {product_name} as a connector candidate for a platform"`
  - **Option C:** `"Yes — run the full flow for a different product"`
  - **Option D:** `"No — I'm done for now"`

- **If all three modes have been run for this product**, omit A and B and offer only:
  - **Option A:** `"Yes — run the full flow for a different product"`
  - **Option B:** `"No — I'm done for now"`

---

### Branch: Switch mode for the same product

`product_profile` and `landscape_profile` are already in memory. Do not repeat Steps 0–2.

For switching **to Connector Candidate path**: ask **Step 0-B2** to collect `target_platform`, then go to **Step 0-C** (slug derivation) and proceed to **Step N1**.

For switching **to Connector Existing path**: go directly to **Step C1** (Catalog Check).

For switching **to Product path**: go directly to **Step P1** (Three-Lens Discovery).

Mark the new mode as complete after its Final-3 summary. Then loop back to Step Final-4 with updated available options.

---

### Branch: Different product

Confirm in one line:
```
Starting fresh for a new product.
```

Then go to **Step 0-A** (Product Name Collection) and run the full flow from the beginning.

---

### Branch: No — Done

Respond with exactly one closing line:
```
All done — {product_name} {mode(s) completed} saved to output/. Open output/{filename}.html in a browser to review.
```

Do not ask any further questions.

---

## Error Handling

| Situation | Action |
|-----------|--------|
| Product not found in IBM Docs or Tavily | Ask user for a 1–2 sentence description, use it as `product_description` and proceed |
| Tavily search returns no results | Use IBM product-knowledge MCP as fallback; if both fail, note limitation and proceed with what is known |
| Catalog page found but not extractable | Record connectors manually from search snippets; note "catalog partially extracted" |
| Fewer than 3 sources return results in discovery phase (NO path) | Inform user, present what was found, proceed with lower-confidence picture — all items marked ⚠️ Inferred |
| User confirms zero candidates in product path | Ask user to describe 2–3 use cases they have in mind; use those as seeds |
| Companion connector unavailable and `companion_reference` is not null | Look up substitute in `companion_reference`, use it in flow, note with `(substitute: {original})` |
| Companion connector unavailable and `companion_reference` is null | Use nearest equivalent from `integration_ecosystem` or `landscape_profile`, note with `(substitute: {original})` |
| More than 15 connectors/adapters discovered | Present top 15 by relevance; ask user to confirm or narrow scope before generation |
| User picks Intent B but does not name a target platform | Ask Step 0-B2 before any research proceeds |
| Connector Candidate path generates more than 5 use cases | Silently discard the lowest-ranked ones; never write more than 5 to the output file |
| `target_platform_slug`-catalog.md not found for Intent B | Set `companion_reference = null`; use Tavily search results for platform gap analysis instead |


## Supporting files in this skill directory:
- output-template.md
- `{product_slug}-catalog.md` (if it exists for the product — discovered at runtime via glob)
- `{target_platform_slug}-catalog.md` (if it exists for the target platform — used by Intent B)
- tests/flow-test.md

Use the read_file and glob tools with paths relative to: .bob/skills/connector-usecase/

# Flow Test Suite — connector-usecase skill

> **How to run:** Say `/test connector-usecase` or "run flow tests for connector-usecase".
> Bob will read this file and execute each scenario as a dry-run against SKILL.md logic —
> no HTML files are written, no git commits are made, no live searches are fired.
> For each scenario, Bob walks through every decision point, verifies expected behaviour,
> and reports PASS or FAIL per assertion. At the end, a summary table is printed.

---

## Test configuration

```
dry_run: true          # No HTML output, no git commits, no real searches
mock_search: true      # Searches return "product found with N results" stub — no real API calls
mock_commit: true      # Git steps print the command they would run, but do not execute
slug_glob_check: true  # product_slug derivation and glob for {slug}-catalog.md is always verified
```

---

## Decision tree (reference)

```
Step 0  → product_name, industry_focus
Step 1  → IBM Docs search (fallback: Tavily)          ← Special case SC-1, SC-2
Step 2  → Competitive landscape research
Step 3  → Mode: A (Connector) or B (Product)

[CONNECTOR PATH]
  Step C1 → Catalog: YES or NO
    YES → C2a (fetch catalog) → C3a (gap analysis) → C4a (generate) → C-Final
    NO  → C2b (4-source discovery) → C3b (user confirmation: accurate or corrections)
            → C4b → C5b → C-Final

[PRODUCT PATH]
  Step P1 → Three-lens discovery (silent)
  Step P2 → Candidates: all or select fewer
  Step P3 → Generate → P-Final

[SHARED FINAL]
  Final-1 (quality check) → Final-2 (git commit) → Final-3 (summary)
  Final-4:
    one mode done  → A: switch mode | B: new product | C: done
    both modes done → A: new product | B: done

[SPECIAL CASES]
  SC-1: product not found in IBM Docs → Tavily fallback
  SC-2: product not found anywhere → ask user for description
  SC-3: product_slug glob matches a catalog file → companion_reference loaded from that file
  SC-4: C3b user says "I have corrections" → corrections accepted before proceeding
```

---

## Scenarios

### T-01 — Connector path · YES catalog · single mode · done

**Product:** `Dynatrace`
**Mode:** Connector Use Cases
**Catalog:** YES
**Final-4:** No — done

| Step | Input | Expected behaviour | Assert |
|------|-------|--------------------|--------|
| Step 0 | product=`Dynatrace`, industry=blank | Stores `product_name=Dynatrace`, `industry_focus=all`. Confirms "Got it — researching Dynatrace..." | ✓ No quarter asked. No connector list asked. |
| Step 1 | IBM Docs search stub → 3 results | Builds `product_profile`: category=Observability, vendor=Dynatrace | ✓ `product_profile` populated. No Tavily fallback fired (IBM Docs sufficient). |
| Step 2 | Tavily stub → competitors found | `landscape_profile` populated: competitors=[Datadog, New Relic, AppDynamics], integration_ecosystem=[Kubernetes, PagerDuty, ServiceNow, Slack, AWS] | ✓ `landscape_profile` stored. |
| Step 3 | User selects A (Connector) | `generation_mode=A`. Connector Path standing instruction checked: `product_name=Dynatrace` ≠ App Connect → `connector-catalog.md` NOT read. | ✓ No `connector-catalog.md` access. |
| Step C1 | User selects YES | Branches to C2a. | ✓ Correct branch. |
| Step C2a | Stub: 12 connectors found | `catalog_connectors[]` = 12 items with name, category, status, trigger_action. | ✓ Catalog stored. |
| Step C3a | Stub: 3 gap items found | List 1 (existing): 12 items. List 2 (gaps): 3 items. List 3 (unsolved): 2 pain points unaddressed. | ✓ Three lists produced. Gaps name a competitor for each. |
| Step C4a | Generate | 4–5 use cases per List 1+2 item. Flow text uses `<strong>` tags. Pills use `integration_ecosystem`, NOT `connector-catalog.md`. | ✓ No App Connect companions. |
| C-Final | Assemble HTML | Title: `Dynatrace — Connector Use Cases`. Subtitle: "Dynatrace's own integration ecosystem". Scope note: "drawn from Dynatrace's integration ecosystem". Summary bar box 5: count from `integration_ecosystem`, NOT 151. | ✓ Zero App Connect / quarter references. |
| Final-1 | Quality check | All 14 checklist items pass. Reports ✅. | ✓ No fix required. |
| Final-2 | Git | Would run: `git commit -m "feat: dynatrace connector use cases — N items"`. No quarter in commit message. | ✓ Commit message clean. |
| Final-3 | Summary | Table shows: File, Product=Dynatrace (Observability), Mode=Connector Use Cases, Items, Committed. | ✓ No quarter in summary. |
| Final-4 | One mode done → 3 options | Option A: "generate Product Use Cases for Dynatrace". Option B: new product. Option C: done. | ✓ Exactly 3 options. Option A names the OTHER mode. |
| Final-4 | User selects C (done) | One closing line: "All done — Dynatrace Connector Use Cases use cases saved to output/. Open output/dynatrace-connector-use-cases.html..." | ✓ No further questions after done. |

**Result: PASS if all rows ✓**

---

### T-02 — Connector path · NO catalog · corrections at gate · single mode · done

**Product:** `IBM Sterling`
**Mode:** Connector Use Cases
**Catalog:** NO
**C3b gate:** User says "I have corrections"
**Final-4:** No — done

| Step | Input | Expected behaviour | Assert |
|------|-------|--------------------|--------|
| Step 0 | product=`IBM Sterling`, industry=`Supply Chain` | `product_name=IBM Sterling`, `industry_focus=Supply Chain`. | ✓ |
| Step 1–2 | IBM Docs stub → found | `product_profile` populated. `landscape_profile` populated. | ✓ |
| Step 3 | User selects A (Connector) | `connector-catalog.md` guard: `IBM Sterling` ≠ App Connect → NOT read. | ✓ |
| Step C1 | User selects NO | Branches to C2b. | ✓ Correct branch. |
| Step C2b | 4-source discovery stub | ✅ Known: 4 adapters. ⚠️ Inferred: 2 adapters. ❌ Gaps: 3 items. All tiered. | ✓ Three tiers populated. |
| Step C3b | Confirmation shown to user | Shows ✅ / ⚠️ / ❌ formatted list. Asks: "Is this picture accurate?" Options: "Yes, accurate" or "I have corrections". | ✓ Gate fires before generation. User is not skipped past. |
| Step C3b | User selects "I have corrections" | Skill accepts user's corrections. Updates picture before proceeding. Does NOT skip to C4b until user confirms. | ✓ Corrections applied. Picture updated. |
| Step C4b | Lists produced | List 1 (Known/updated), List 2 (Inferred/updated), List 3 (Gaps). | ✓ Uses updated picture, not original. |
| Step C5b | Generate | List 2 (Inferred) use cases all include annotation: `(based on community-documented adapter support — verify with product team)`. | ✓ Annotation present on every Inferred item. |
| C-Final | HTML | Scope note says "reconstructed from official documentation, release notes...". No "151 connectors". No "App Connect". | ✓ Zero leakage. |
| Final-4 | User selects C (done) | One closing line only. | ✓ |

**Result: PASS if all rows ✓**

---

### T-03 — Product path · all candidates · single mode · switch to Connector after

**Product:** `Salesforce`
**Mode:** Product Use Cases first, then switch to Connector Use Cases
**Final-4 (1st):** Switch mode → Connector
**Final-4 (2nd):** Done

| Step | Input | Expected behaviour | Assert |
|------|-------|--------------------|--------|
| Step 0–2 | product=`Salesforce` | `product_profile` + `landscape_profile` stored. | ✓ |
| Step 3 | User selects B (Product) | `generation_mode=B`. Product Path standing instruction: `connector-catalog.md` NOT read (not App Connect). | ✓ |
| Step P1 | Silent 6-search sweep | Lens 1 (Gaps), Lens 2 (Opportunities), Lens 3 (Integration-Enabled) all populated. | ✓ No user questions during P1. |
| Step P2 | 8 candidates shown | Groups: 3 Gaps, 3 Opportunities, 2 Integration-Enabled. User selects "Proceed with all". | ✓ Candidates grouped by lens. Each has evidence citation. |
| Step P3 | Generate | Card headers omit priority badge (P1/P2/P3) per Section 8 product-path rules. `.cpill` companions from `integration_ecosystem`, not `connector-catalog.md`. | ✓ No priority badges. |
| P-Final | HTML | Title: `Salesforce — Product Use Cases`. Scope note uses `Salesforce's own integration ecosystem`. Summary bar: box 2=Gaps count, 3=Opportunities count, 4=Integration-Enabled count, 5=ecosystem connector count (not 151). | ✓ |
| Final-4 (1st) | One mode done → 3 options | Option A: "generate Connector Use Cases for Salesforce (reuses research already done, skips straight to generation)". | ✓ Correct other-mode named. |
| Final-4 (1st) | User selects A (switch mode) | One-line confirm: "Switching to Connector Use Cases for Salesforce — picking up where we left off." | ✓ No re-research. No Step 0–3 repeat. |
| Entry point | Jumps to C1 | Asks catalog check for Salesforce. `product_profile` + `landscape_profile` reused without re-fetching. | ✓ C1 is first question, not Step 0. |
| C1 → C4a | YES path | Connector use cases generated. Companions from `integration_ecosystem`. | ✓ |
| Final-3 (2nd) | Second summary | Shows Salesforce Connector Use Cases. | ✓ |
| Final-4 (2nd) | Both modes done → 2 options | Option A: new product. Option B: done. Option for "switch mode" NOT offered. | ✓ Switch-mode option suppressed. Exactly 2 options. |
| Final-4 (2nd) | User selects B (done) | One closing line. | ✓ |

**Result: PASS if all rows ✓**

---

### T-04 — Product path · select fewer candidates · new product after

**Product:** `ServiceNow`
**Mode:** Product Use Cases
**P2 selection:** User removes some candidates
**Final-4:** New product

| Step | Input | Expected behaviour | Assert |
|------|-------|--------------------|--------|
| Step 0–3 | product=`ServiceNow`, mode=B | Setup correct. | ✓ |
| Step P2 | 9 candidates shown; user selects B (fewer) | Skill accepts user's selections. Only confirmed candidates proceed to P3. Removed candidates do NOT appear in output. | ✓ User-selected subset only. |
| Step P3 | Generate for subset | Use case count matches selected subset × 4–5. | ✓ No phantom use cases. |
| Final-4 | User selects B (new product) | One-line confirm: "Starting fresh for a new product." | ✓ Exact wording. |
| Step 0 restart | New product name asked | Full Step 0 fires: "What is the name of the product...". `product_profile` and `landscape_profile` from ServiceNow are discarded/replaced. | ✓ Clean restart. No ServiceNow data bleeds into new product. |

**Result: PASS if all rows ✓**

---

### T-05 — Connector path · YES catalog · switch to Product · then done

**Product:** `MuleSoft Anypoint`
**Mode:** Connector first → then Product (via Final-4)
**Final-4 (2nd):** Done

| Step | Input | Expected behaviour | Assert |
|------|-------|--------------------|--------|
| Step 0–3 | product=`MuleSoft Anypoint`, mode=A | Setup correct. `connector-catalog.md` NOT read. | ✓ |
| C1→C4a | YES catalog path | Connector use cases generated. | ✓ |
| Final-4 (1st) | Switch to Product Use Cases | "Switching to Product Use Cases for MuleSoft Anypoint — picking up where we left off." Jumps to P1. | ✓ Entry point is P1 (silent search), not Step 0. |
| P1 | Silent | `product_profile` + `landscape_profile` reused. P1 fires new lens searches. | ✓ Research reuse confirmed. |
| P2→P-Final | Product use cases generated | Correct path. | ✓ |
| Final-4 (2nd) | Both done → 2 options | Exactly 2 options. No switch-mode option. | ✓ |
| User selects B (done) | One closing line. | ✓ |

**Result: PASS if all rows ✓**

---

### T-06 — Connector path · NO catalog · accurate at gate · both modes · new product

**Product:** `SAP Integration Suite`
**C3b gate:** User confirms accurate
**Run both modes**
**Final-4 (2nd):** New product → new product runs completely

| Step | Input | Expected behaviour | Assert |
|------|-------|--------------------|--------|
| Step 0–3 | product=`SAP Integration Suite`, mode=A | `connector-catalog.md` NOT read. | ✓ |
| C1 | NO | Branches to C2b. | ✓ |
| C2b | 4-source discovery | Known/Inferred/Gap tiers populated. | ✓ |
| C3b | User selects "Yes, this looks accurate" | Proceeds directly to C4b without asking for corrections. | ✓ No extra prompt. |
| C4b→C5b | Lists + use cases | Inferred items carry annotation. Gap items carry `(not yet available — gap vs {CompetitorName})`. | ✓ Both annotations present. |
| Final-4 (1st) | Switch to Product | Jumps to P1 with stored profiles. | ✓ |
| Final-4 (2nd) | New product | "Starting fresh for a new product." Step 0 fires for new product. | ✓ SAP data cleared. |
| New product run | Complete run | Fresh `product_profile`, fresh `landscape_profile`. No SAP Integration Suite data in output. | ✓ Zero cross-contamination. |

**Result: PASS if all rows ✓**

---

### T-07 — IBM App Connect (ACE) — ibm-app-connect-catalog.md MUST be loaded via slug glob

**Product:** `IBM App Connect Enterprise`
**Mode:** Connector Use Cases
**Expected:** `companion_reference` is loaded from `ibm-app-connect-catalog.md` at Step 0

| Step | Input | Expected behaviour | Assert |
|------|-------|--------------------|--------|
| Step 0 | product=`IBM App Connect Enterprise` | `product_name` stored. `product_slug` derived = `ibm-app-connect-enterprise`. | ✓ |
| Step 0 | Glob check | `glob(".bob/skills/connector-usecase/ibm-app-connect-enterprise-catalog.md")` → not found. Try broader slug `ibm-app-connect` → `ibm-app-connect-catalog.md` found. `companion_reference` loaded. | ✓ `companion_reference` is not null. |
| Step 3 | Mode=A (Connector) | Connector Path standing instruction: `companion_reference` is not null → use it as companion source. No name-based conditional fires. | ✓ File-discovery mechanism, not name check. |
| Step C1 | YES | Branches to C2a. | ✓ |
| C2a | Catalog fetched | `catalog_connectors[]` populated from research (not from `companion_reference` — that is the companion/substitution reference, not the catalog source). | ✓ |
| C4a | Generate | Companion connectors sourced from `companion_reference`. Summary bar box 5: count from `companion_reference`. Scope note mentions "pre-built reference catalog loaded". | ✓ `companion_reference` used. No product-name conditional anywhere. |
| C-Final | HTML | No personal data, no quarter. | ✓ |

**Result: PASS if all rows ✓**

---

### T-08 — IBM App Connect — Product path · companion_reference as Lens 3 companion

**Product:** `IBM App Connect`
**Mode:** Product Use Cases
**Expected:** `companion_reference` loaded from `ibm-app-connect-catalog.md` at Step 0 and used for Lens 3

| Step | Input | Expected behaviour | Assert |
|------|-------|--------------------|--------|
| Step 0 | product=`IBM App Connect` | `product_slug` derived = `ibm-app-connect`. Glob check finds `ibm-app-connect-catalog.md`. `companion_reference` loaded. | ✓ `companion_reference` is not null. |
| Step 3 | Mode=B (Product) | Product Path standing instruction: `companion_reference` is not null → use it for Lens 3. No name-based conditional fires. | ✓ File-discovery mechanism. |
| P1 Lens 3 | Integration-enabled scenarios | Companion connectors in Lens 3 flows reference systems from `companion_reference`, not raw `integration_ecosystem`. | ✓ Correct source. |
| P-Final | HTML | Summary bar box 5 = count from `companion_reference`. Scope note says "pre-built reference catalog loaded". | ✓ Count from actual file, not hardcoded. |
| Non-ACE check | Contrast | If same test run for Dynatrace, glob finds no `dynatrace-catalog.md` → `companion_reference` is null → `integration_ecosystem` used instead. Box 5 ≠ ACE count. | ✓ Confirms discovery works for both cases. |

**Result: PASS if all rows ✓**

---

### T-09 — Product not found in IBM Docs — Tavily fallback (SC-1)

**Product:** `Niche ERP Tool X` (unknown to IBM Docs)
**Expected:** IBM Docs returns <2 results → Tavily fallback fires

| Step | Input | Expected behaviour | Assert |
|------|-------|--------------------|--------|
| Step 1 | IBM Docs stub → 0 results | Fewer than 2 useful results → Tavily fallback fires. Two Tavily searches run: overview + use cases. | ✓ Fallback triggered. No error thrown. |
| Step 1 | Tavily stub → 2 results | `product_profile` built from Tavily data. Marked as "sourced from Tavily". | ✓ Profile populated from fallback. |
| Step 2 onwards | Normal flow | Flow continues as normal. | ✓ No interruption to user. |

**Result: PASS if all rows ✓**

---

### T-10 — Product not found anywhere — user description fallback (SC-2)

**Product:** `UnknownTool99`
**Expected:** IBM Docs + Tavily both return no results → user asked for description

| Step | Input | Expected behaviour | Assert |
|------|-------|--------------------|--------|
| Step 1 | IBM Docs stub → 0 results | Fallback fires. | ✓ |
| Step 1 | Tavily stub → 0 results | Both fail → Error handling: "Product not found in IBM Docs or Tavily" rule fires. | ✓ |
| Error handling | Skill asks user | Asks: "I couldn't find information about UnknownTool99. Please provide a 1–2 sentence description of what it does." | ✓ User prompted. Flow does NOT crash. |
| User provides description | "UnknownTool99 is a supply chain planning tool for mid-market retail." | `product_description` set from user input. Flow continues using this as the `product_profile` seed. | ✓ Flow continues. Description used as profile. |

**Result: PASS if all rows ✓**

---

### T-11 — More than 15 connectors discovered — cap and confirm gate

**Product:** `Boomi`
**Mode:** Connector Use Cases
**Catalog:** YES → 22 connectors found

| Step | Input | Expected behaviour | Assert |
|------|-------|--------------------|--------|
| C2a | Stub returns 22 connectors | Exceeds 15-item cap. | ✓ |
| Cap handling | Skill presents top 15 by relevance | Shows top 15. Asks: "I found 22 connectors. Here are the top 15 by relevance — confirm or narrow scope before I generate." | ✓ User not silently truncated. Confirmation required. |
| User confirms | "Proceed with top 15" | Generation proceeds with exactly 15. | ✓ |
| Output | HTML | Exactly 15 connector sections. Summary bar box 1 = 15. | ✓ |

**Result: PASS if all rows ✓**

---

### T-12 — User confirms zero P2 candidates — seed fallback

**Product:** `IBM Turbonomic`
**Mode:** Product Use Cases
**P2:** User removes all candidates

| Step | Input | Expected behaviour | Assert |
|------|-------|--------------------|--------|
| P2 | User removes all 8 candidates | Zero confirmed. | ✓ |
| Error handling | "User confirms zero candidates" rule fires | Skill asks: "Could you describe 2–3 use cases you have in mind? I'll use those as seeds." | ✓ Not a crash. Not silent skip. Explicit prompt. |
| User provides seeds | "1. Predictive cost savings via AI. 2. Workload rebalancing on alert." | Seeds used as P3 input. 4–5 use cases per seed generated. | ✓ User seeds honoured. |

**Result: PASS if all rows ✓**

---

### T-13 — Catalog page found but not extractable

**Product:** `Zapier`
**Mode:** Connector Use Cases
**C2a:** Catalog URL found but Tavily extract fails

| Step | Input | Expected behaviour | Assert |
|------|-------|--------------------|--------|
| C2a | Tavily extract stub → returns empty/error | "Catalog page found but not extractable" error rule fires. | ✓ |
| Fallback | Record connectors from search snippets | Skill continues with partial data. Notes "catalog partially extracted" in scope note. | ✓ No crash. User-visible note added. |
| C-Final | HTML scope note | Contains "catalog partially extracted" notice. | ✓ Transparency preserved. |

**Result: PASS if all rows ✓**

---

### T-14 — Connector path · NO catalog · fewer than 3 sources return results

**Product:** `Legacy Mainframe ETL Tool`
**Mode:** Connector Use Cases
**C2b:** Only 1 of 4 sources returns data

| Step | Input | Expected behaviour | Assert |
|------|-------|--------------------|--------|
| C2b | Stub: Sources 1, 2, 3 return 0; Source 4 returns 2 items | Fewer than 3 sources have results → error rule fires. | ✓ |
| Error handling | Inform user | "I could only find data from 1 source. All items will be marked ⚠️ Inferred." | ✓ User informed. |
| C3b | All items marked ⚠️ Inferred | No ✅ Known items. No ❌ Gap items (no competitor data to compare against). | ✓ Single-tier picture. |
| C5b | Generate | Every use case carries `(based on community-documented adapter support — verify with product team)`. | ✓ All annotated. |

**Result: PASS if all rows ✓**

---

## Test summary table

After running all scenarios, Bob prints this table:

```
| ID    | Scenario                                                    | Result |
|-------|-------------------------------------------------------------|--------|
| T-01  | Connector · YES catalog · done                              | —      |
| T-02  | Connector · NO catalog · corrections at gate · done         | —      |
| T-03  | Product path · switch to Connector · done                   | —      |
| T-04  | Product path · select fewer · new product                   | —      |
| T-05  | Connector first · switch to Product · done                  | —      |
| T-06  | NO catalog · accurate · both modes · new product            | —      |
| T-07  | IBM App Connect — connector-catalog.md consulted (Conn)     | —      |
| T-08  | IBM App Connect — connector-catalog.md as Lens 3 (Prod)     | —      |
| T-09  | Product not found in IBM Docs → Tavily fallback             | —      |
| T-10  | Product not found anywhere → user description               | —      |
| T-11  | >15 connectors discovered → cap + confirm gate              | —      |
| T-12  | Zero P2 candidates → seed fallback                          | —      |
| T-13  | Catalog not extractable → partial note                      | —      |
| T-14  | NO catalog · <3 sources · all Inferred                      | —      |
|-------|-------------------------------------------------------------|--------|
| TOTAL |                                                             | 0/14   |
```

Replace `—` with `PASS` or `FAIL — {assertion that failed}` as each scenario is evaluated.

---

## Permutation coverage map

```
Mode (A/B)                    ──┬── A (Connector)
                                │     ├── C1: YES ──────────── T-01, T-03, T-05, T-07, T-11, T-13
                                │     └── C1: NO
                                │             ├── C3b: accurate ─ T-06, T-14
                                │             └── C3b: corrections ─ T-02
                                └── B (Product)
                                      ├── P2: all ─────────────── T-03, T-05, T-06, T-08
                                      ├── P2: fewer ─────────────── T-04
                                      └── P2: zero ─────────────── T-12

Final-4 (one mode done)        ──┬── Switch mode (A→B) ─────── T-03, T-06
                                  ├── Switch mode (B→A) ─────── T-05
                                  ├── New product ─────────────── T-04, T-06
                                  └── Done ─────────────────────── T-01, T-02

Final-4 (both modes done)      ──┬── New product ─────────────── T-06
                                  └── Done ─────────────────────── T-03, T-05

companion_reference discovery  ──┬── File found → loaded (App Connect) ─ T-07, T-08
                                   └── No file → null (all others) ────── T-01–T-06, T-09–T-14

Search fallback                ──┬── IBM Docs sufficient ──────── T-01–T-08, T-11–T-14
                                  ├── Tavily fallback ─────────── T-09
                                  └── Both fail → user prompt ─── T-10
```

All 12 end-to-end paths and all 4 special cases are covered.

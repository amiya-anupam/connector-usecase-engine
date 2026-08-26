# connector-usecase-engine

> Generic product use case engine — powered by IBM Bob

Generates market-validated use cases for **any software product** through an
interactive, research-first workflow. Supports two output modes:

- **Connector Use Cases** — analyses what integrations a product supports or is
  missing, benchmarked against competitor iPaaS catalogs
- **Product Use Cases** — explores gaps the product isn't solving, opportunities
  it could fulfil, and new scenarios enabled by integration

## How it works

1. **Onboarding** — Bob asks for the product name (and optional industry focus).
   Nothing else.
2. **Product intelligence** — IBM Docs and Tavily are searched automatically to
   build a product profile.
3. **Competitive landscape** — market, competitor, and integration ecosystem data
   are discovered automatically.
4. **Mode selection** — you choose Connector Use Cases or Product Use Cases.
5. **Path-specific analysis** — connector catalog check, gap analysis, or
   three-lens discovery depending on mode and product type.
6. **HTML output** — a fully structured file is written to `output/`, committed,
   and ready to share.

## Usage

Open this repository in IBM Bob and trigger:

```
/connector-usecase
```

Bob will ask:
> "What is the name of the product you want to generate use cases for?"

Everything else — research, competitive analysis, mode selection, and generation —
is handled interactively from there.

## Repository structure

```
connector-usecase-engine/
├── README.md
├── templates/
│   └── usecase-template.html        ← canonical HTML layout reference (CSS source)
├── output/
│   └── (generated HTML files)
└── .bob/
    └── skills/
        └── connector-usecase/
            ├── SKILL.md             ← Bob skill — full interactive flow instructions
            ├── output-template.md   ← HTML layout rules (sections, badges, pills)
            ├── connector-catalog.md ← IBM App Connect connector reference
            │                          (consulted only when product = App Connect / ACE)
            └── tests/
                └── flow-test.md     ← 14-scenario dry-run test suite
```

> **Note on `.bob/`:** This is Bob's permanent skill registry — not a temporary
> folder. Bob scans `.bob/skills/` on workspace open and loads every `SKILL.md`
> it finds. This is how `/connector-usecase` becomes a live command.

## Branch strategy

| Branch | Purpose |
|--------|---------|
| `main` | Stable. Receives merges from `dev` after each run is validated. |
| `dev`  | Active. All skill edits and generated output land here first. |

## Running the test suite

To verify all flow paths without generating any output:

```
/test connector-usecase
```

Bob will dry-run all 14 scenarios in `tests/flow-test.md` — covering every
permutation of mode, catalog YES/NO, confirmation gates, Final-4 branches,
error cases, and the App Connect guard — and print a PASS/FAIL summary table.

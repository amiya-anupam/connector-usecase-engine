# connector-usecase-engine

> IBM App Connect Enterprise — Connector Use Case Automation via IBM Bob

Automates the quarterly process of researching, analysing, and generating high-value enterprise integration use cases for IBM App Connect connectors.

## What this does

Each quarter, this engine drives a structured workflow through IBM Bob that:

1. **Market & competitive analysis** — benchmarks IBM App Connect against MuleSoft, Boomi, Workato, Informatica, and Tray.ai using live web research
2. **IBM App Connect framework check** — queries IBM Docs and Redbooks to validate which use cases the current ACE framework supports
3. **Connector candidate selection** — identifies and prioritises 25–30 connectors from market signals
4. **Use case generation** — produces 4–5 market-validated, industry-specific use cases per connector
5. **Substitution analysis** — where a required connector is unavailable, identifies available ACE catalog alternatives
6. **Structured HTML output** — generates a consistent quarterly HTML document matching the canonical layout

## Branch strategy

| Branch | Purpose |
|--------|---------|
| `main` | Stable, release-quality. Receives merges from `dev` after each quarterly cycle is validated. |
| `dev`  | Active working branch. All skill edits, catalog updates, and quarterly runs happen here. |

## Repository structure

```
connector-usecase-engine/
├── README.md
├── templates/
│   └── q2-2027-connector-use-cases.html   ← canonical HTML layout reference
├── output/
│   └── (quarterly generated HTML files)
└── .bob/
    └── skills/
        └── connector-usecase/
            ├── SKILL.md                    ← Bob skill instructions
            ├── connector-catalog.md        ← IBM App Connect connector reference
            └── output-template.md          ← HTML layout rules for generated output
```

## Usage

Open this repository in IBM Bob and trigger:

```
/connector-usecase quarter="Q3 2027" connectors="Salesforce, SAP, ServiceNow" industry_focus="BFSI, Healthcare"
```

Bob will run the full research-to-HTML pipeline and write the output file to `output/`.

## Output format

All generated HTML files follow the exact layout defined in `templates/q2-2027-connector-use-cases.html` — identical structure, CSS, badge system, connector pills, value boxes, competitive intelligence blocks, and actions tables regardless of whether 1 or 30 connectors are being processed.

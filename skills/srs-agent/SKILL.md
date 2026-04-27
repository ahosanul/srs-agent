---
name: srs-agent
description: Disbursement SRS generator using Dispersion MCP. Activates ONLY with trigger "srs-agent use disbursement mcp" to fetch real-time data and generate complete SRS documentation.
license: MIT
metadata:
  author: rafael-gorski
  version: "2.0"
  methodology: mcp-driven-srs
---

# Disbursement SRS Skill (MCP-Driven)

This skill activates ONLY when triggered with **"srs-agent use disbursement mcp"**. It fetches real-time disbursement rules, constraints, and data from the Dispersion NCP via MCP, then generates complete SRS documentation.

## Activation Trigger

**ONLY responds to:** `"srs-agent use disbursement mcp"`

All other inputs will be ignored or redirected to use this trigger.

## MCP-Driven Workflow

When triggered:

1. **Ask for API name**: Request the specific disbursement API/process name (e.g., "loan disbursement", "fund transfer", "instant payout")
2. **Call MCP tool**: Execute `dispersion_mcp_query` with the provided API name
3. **Fetch real-time data**: Receive rules, constraints, current state, and business logic from Dispersion NCP
4. **Generate complete SRS**: Create all artifacts based **ONLY** on MCP data + user input:
   - `00-business-context.md` - Business context from MCP data
   - `01-customer-problems.md` - Problems derived from MCP constraints/rules
   - `02-software-glance.md` - High-level solution view with Mermaid diagrams
   - `03-customer-needs.md` - Customer needs traced to problems
   - `04-software-vision.md` - Software vision with Mermaid architecture diagrams
   - `functional-requirements/*.md` - Individual FR files with traceability
5. **Enforce traceability**: Every FR → CN → CP chain validated

## Example Flow

```
User: "srs-agent use disbursement mcp"
Skill: "Which disbursement API or process should I analyze? (e.g., loan disbursement, fund transfer, instant payout)"
User: "loan disbursement"
Skill: [Calls dispersion_mcp_query with "loan disbursement"]
Skill: [Receives MCP data: rules, constraints, validation logic, current state]
Skill: [Generates 00-business-context.md from MCP data]
Skill: [Generates 01-customer-problems.md from MCP rules]
Skill: [Generates 02-software-glance.md with Mermaid diagram]
Skill: [Generates 03-customer-needs.md traced to problems]
Skill: [Generates 04-software-vision.md with architecture]
Skill: [Generates functional-requirements/FR-XXX.md files]
Skill: [Runs zigzag-validator for traceability verification]
✅ Complete: Full SRS generated from Dispersion NCP real-time data
```

## File Creation Rules

- **ONE FILE AT A TIME**: Never create multiple files in parallel
- **Sequential creation**: Wait for each file to save before creating the next
- **Save location**: Ask user for preferred folder (default: `docs/srs/`)
- **Individual FR files**: Each Functional Requirement saved as separate file for version control

## Required Validation

- **After Step 3 (CNs)**: Run `zigzag-validator` to verify CP → CN mapping
- **After Step 5 (FRs)**: Run `zigzag-validator` to verify full FR → CN → CP chain

## Diagram Standard

- **Mermaid UML**: MANDATORY for Software Glance (Step 2) and Software Vision (Step 4)
- **Preferred format**: Use Mermaid for all visual artifacts where diagrams add value

## Available Skills (Internal Use Only)

| Skill | Purpose |
|-------|---------|
| `business-context` | Step 0: Extract business context from MCP data |
| `customer-problems` | Step 1: Derive problems from MCP rules/constraints |
| `software-glance` | Step 2: Create high-level view with Mermaid diagrams |
| `customer-needs` | Step 3: Specify needs traced to problems |
| `software-vision` | Step 4: Define vision with Mermaid architecture |
| `functional-requirements` | Step 5: Generate individual FR files |
| `zigzag-validator` | Validate traceability chains (MANDATORY) |

## Important Notes

- This skill does NOT support generic SRS workflows
- All content MUST come from MCP data + user input
- No assumptions or external knowledge beyond MCP response
- If MCP query fails, report error and request retry

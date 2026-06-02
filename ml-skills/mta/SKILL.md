---
name: fde-mta
description: |
  MTA (Multi-Touch Attribution) Journey Analytics for Treasure Data. Configures workflows that build unified customer journeys from pageviews, email, sales, and orders, then runs attribution models (Markov, Shapley, linear, time-decay) to measure channel contribution to conversions. Trigger on: MTA, multi-touch attribution, channel attribution, conversion paths, journey analytics, Markov, Shapley, marketing attribution, channel mix, CPA, CPB, requirements gathering for MTA, MTA workflow setup, MTA configuration, MTA runbook, MTA output tables.
---

# MTA Journey Analytics

Multi-Touch Attribution workflow that builds unified customer journeys from multiple touchpoint sources and runs attribution models to measure channel contribution to conversions.

## How to Start

Read `../shared/SKILL.md` — it defines the entry point (new vs. resume), the phase sequence, and when to read each shared pattern file. Follow that flow. This file provides the MTA-specific context you need at each phase.

## Phase Summary

| Phase | What happens |
|---|---|
| Phase 1 | Data exploration — discover touchpoint tables, confirm schema, check UTM coverage |
| Phase 2 | Create Confluence folder + Current Project State + Requirements Doc |
| Phase 3 | Push minimal workflow, validate output tables |
| Phase 4 | Deploy Foundry agent, run integration check |
| Phase 5 | Re-configure workflow from filled requirements, re-validate |
| Phase 6 | Parent Segment update + example audiences, then customer-facing documentation set |

---

## MTA-Specific Context by Phase

### Phase 1: Data Exploration

Use `tdx-skills:tdx-basic` to explore the customer's TD databases. **Do not search by table name patterns.** Instead follow these steps:

**Step 1 — List every table in the customer database.**
Run `tdx tables "<database>.*"` to get the full table list.

**Step 2 — Classify each table.**
For every table, run `tdx describe` to inspect its schema. Classify it into one of three buckets:

| Bucket | Criteria |
|---|---|
| **Touchpoint / activity table** | Has a user ID column + a Unix timestamp column + rows represent individual customer events or interactions (one row = one event). Candidates for MTA source tables. |
| **Profile / dimension table** | Has a user ID column but rows represent one record per customer (master tables, attributes, membership tiers). Not a source table. |
| **System / output table** | Workflow output, audit logs, ML result tables. Not a source table. |

**Step 3 — Within touchpoint tables, classify by MTA table type.**
For each candidate table, determine which MTA type it maps to:

| MTA Table Type | What to look for |
|---|---|
| **Pageviews / web activity** | URL column (`td_url`, `page_url`), referrer column, path column — web events from JS SDK |
| **Email events** | `event_type` column with values like open/click/send, `campaign_name` or `email_name` column |
| **Sales / CRM interactions** | `source`, `topic`, or `rep_name` columns — offline sales touchpoints |
| **Orders / conversions** | `order_status`, `unit_price` or revenue column — these ARE the conversion events |

**Step 4 — For web tables: check UTM coverage.**
> ⚠️ **Do NOT assume UTM absence from visual inspection of sample URL values.** Always run the coverage query — clean-looking path values may still have UTM params on a large fraction of rows.

```sql
-- Step 4a: identify URL column name via DESCRIBE, then substitute <url_col> below

-- Step 4b: check UTM coverage
SELECT
  COUNT(*) AS total_rows,
  COUNT(CASE WHEN url_extract_parameter(<url_col>, 'utm_source') IS NOT NULL THEN 1 END) AS has_utm_source,
  COUNT(CASE WHEN url_extract_parameter(<url_col>, 'utm_medium') IS NOT NULL THEN 1 END) AS has_utm_medium,
  COUNT(CASE WHEN url_extract_parameter(<url_col>, 'utm_campaign') IS NOT NULL THEN 1 END) AS has_utm_campaign
FROM database_name.web_table
WHERE td_interval(time, '-90d')

-- Step 4c: if UTMs present, sample the values
SELECT
  url_extract_parameter(<url_col>, 'utm_source') AS utm_source,
  url_extract_parameter(<url_col>, 'utm_medium') AS utm_medium,
  url_extract_parameter(<url_col>, 'utm_campaign') AS utm_campaign,
  COUNT(*) AS cnt
FROM database_name.web_table
WHERE <url_col> LIKE '%utm_%'
  AND td_interval(time, '-90d')
GROUP BY 1, 2, 3 ORDER BY 4 DESC LIMIT 20;
```

**Step 5 — Present the recommended table list to the user before proceeding.**
Show a table of ALL candidate tables with your recommendation (include / exclude), MTA type, and a one-line reason for each. Use the `AskUserQuestion` tool to present the confirmation — never as a plain text list. Provide pre-populated options (e.g. "Confirm as listed", "Exclude one or more tables") with "Other" for custom input. Do not skip this step.

**Step 6 — For each confirmed source table, note down:**
- Full table name
- User ID column name
- Timestamp column name (`time`, `event_time`, etc.)
- URL and referrer column names (web tables only)
- Channel/campaign columns available
- Whether any column can serve as a conversion signal
- Revenue column (order tables)

These become `input_params.yml` values.

**Confluence folder name for this solution:** `MTA Journey Analytics`
**Title variants for fuzzy matching:** `MTA`, `Journey Analysis`, `Multi-Touch Attribution`, `MTA Journey Analytics`

### Phase 2: Requirements Doc

Page title: `MTA Requirements Gathering - <Customer>`

Body template: `workflow-setup/references/requirements_doc.md`

Key questions the requirements doc must answer: unique user ID column, conversion definition (URL pattern / order status / form submit), touchpoint sources beyond pageviews, time range or lookback period, ML models scope (Markov + Shapley vs standard only), customer enrichment (RFM join), sink database name.

### Phase 3: Push Minimal Workflow

**GitHub repo:** `https://github.com/treasure-data-ps/mta_journey_analysis`
**Workflow project name:** `mta_journey_agent`
**Workflow path:** `mta_journey_analysis/td_wf/mta_journey_agent/`
**Config file:** `mta_journey_agent/config/input_params.yml`

For the minimal push, populate `input_params.yml` with what you know from Phase 1 (tables, user ID column, timestamp column, UTM coverage findings). Use Phase 2 answers if already available; otherwise use safe defaults.

Read `workflow-setup/references/workflow_setup_guide.md` for full parameter generation instructions.
Read `workflow-setup/references/yaml_structure.md` for the parameter reference.
Read `workflow-setup/references/table_configuration.md` for per-source-type config.
Use `workflow-setup/references/input_params_template.yml` as the starting point.

After the workflow run finishes, validate output tables following instructions in `workflow-setup/references/eval.md`.

### Phase 4: Foundry Agent Setup

**Foundry agent project name:** `MTA Journey Analysis`
**Agent name:** `MTA Journey Analysis`

Read `agent-skills/foundry-agent/SKILL.md` for the files-to-edit table and integration check.

Populate `business_context.md` with what you learned in Phases 1–3: customer industry, source tables used, conversion definition, output table name, UTM coverage quality, and any notable data quirks.

### Phase 5: Update Workflow with Customer Requirements

After reading the filled requirements doc, present a summary of the parameter changes you plan to make before touching any files. Key areas that typically change between Phase 3 and Phase 5:

- Conversion flag logic (customer may have a more precise URL pattern or use orders table as conversion)
- Adding or removing touchpoint table types
- Channel regex rules if the customer has non-standard UTM naming
- Time range / lookback period adjustments
- ML model scope (enable/disable Markov + Shapley)
- Customer enrichment join (add RFM segment if available)

Read `workflow-setup/references/workflow_setup_guide.md` for update mechanics.
Re-run the workflow and re-validate output tables (same checks as Phase 3).

### Phase 6: Customer Documentation

**Step 1 — Confluence documentation pages.**
Read `../shared/customer_docs_pattern.md` for the 5-page set.

MTA-specific content for each page is in `prod-docs/references/`:

| Page | Reference file |
|---|---|
| Architecture | `prod-docs/references/runbook.md` (architecture section) |
| Runbook | `prod-docs/references/runbook.md` |
| Customer-facing overview | `prod-docs/references/customer_docs.md` |
| Technical handoff | `prod-docs/references/technical_handoff.md` |

---

## Quick Reference

- **GitHub repo**: `https://github.com/treasure-data-ps/mta_journey_analysis`
- **Workflow project name**: `mta_journey_agent` (TD Workflow)
- **Foundry agent project name**: `MTA Journey Analysis` (Foundry LLM project)
- **Workflow path**: `mta_journey_analysis/td_wf/mta_journey_agent/`
- **Config file**: `mta_journey_agent/config/input_params.yml`
- **Key outputs**: `mta_attribution_results`, `mta_top_conversion_journeys`, `mta_channel_summary` in the sink database
- **Models**: Markov, Shapley, Linear, Time-Decay
- **Confluence folder name for this solution**: `MTA Journey Analytics`
- **Title variants for fuzzy matching**: `MTA`, `Journey Analysis`, `Multi-Touch Attribution`, `MTA Journey Analytics`

## Companion agent skills (separate entry points)

These are registered as their own skills — they trigger directly from user prompts, no need to route through this file:

- **`fde-mta-foundry-skill`** — deploy the MTA Journey Analytics AI Foundry agent template to a customer's TD instance
- **`fde-mta-journey-agent`** — query the MTA output tables to answer attribution and journey questions (Sankey, path analysis, model comparison)

## How It Works

1. **Union**: Combine pageviews, email, sales, orders into a single touchpoint table
2. **Sessionize**: Group touchpoints into sessions based on inactivity gap
3. **Build journeys**: Create per-customer journey sequences with conversion flags
4. **Attribute**: Run Markov, Shapley, linear, time-decay models
5. **Output**: Channel attribution scores, top conversion paths, spend efficiency

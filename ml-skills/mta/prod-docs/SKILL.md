---
name: mta-prod-docs
description: |
  Production documentation for the MTA Journey Analytics workflow. Covers workflow architecture, output tables, attribution models, and operational runbook.
---

# MTA Journey Analytics — Production Documentation

## Workflow Overview

The MTA Journey Analytics workflow builds a unified customer journey from multiple touchpoint sources, sessionizes interactions, and runs attribution models to measure channel contribution to conversions.

**GitHub Repository**: `https://github.com/treasure-data-ps/mta_journey_analysis`
**Workflow Path**: `mta_journey_analysis/td_wf/mta_journey_agent/`

## Architecture

```
Source Tables (pageviews, email, sales, orders)
    ↓
Union & Standardize (src_union_table)
    ↓
Sessionize (session_length param)
    ↓
Backfill NULL channels (backfill_partition_col)
    ↓
Build Journeys (journeys_input_table)
    ↓
Attribution Models (Markov, Shapley, Linear, Time-Decay)
    ↓
Output Tables → Dashboard / Agent Analysis
```

## Output Tables

All written to `sink_database`:

| Table | Description |
|-------|-------------|
| `journey_src_union` | Unified touchpoint table from all sources |
| `journey_src_union_temp_converted` | Sessionized with conversion flags |
| `journey_src_union_final_converted` | Final journey table for attribution |
| `mta_attribution_results` | Channel attribution scores (all models) |
| `mta_top_conversion_journeys` | Top-K distinct conversion paths |
| `marketing_channel_spends` | Channel spend data for ROI calculation |
| `mta_channel_summary` | Aggregated channel performance stats |

## Attribution Models

| Model | Description | Best For |
|-------|-------------|----------|
| Markov | Probability-based transition model | Understanding channel interdependencies |
| Shapley | Game-theory fair value distribution | Budget allocation across channels |
| Linear | Equal credit to all touchpoints | Baseline comparison |
| Time-Decay | More credit to recent touchpoints | Recency-weighted analysis |

## Configuration Reference

See `workflow-setup/SKILL.md` for the full configuration workflow and `workflow-setup/references/` for:
- `yaml_structure.md` — complete parameter reference
- `table_configuration.md` — per-table-type setup guide
- `input_params_template.yml` — working example config

## Operational Runbook

### Running the Workflow
```bash
tdx wf push mta_journey_agent
tdx wf run mta_journey_agent.main
tdx wf sessions mta_journey_agent --status running
```

### Monitoring
```bash
tdx wf timeline mta_journey_agent.main --follow
tdx wf attempt <id> tasks
tdx wf attempt <id> logs +failed_task
```

### Common Failure Points
- **Source table missing or renamed**: Verify `src_table` values in `input_params.yml`
- **No conversions found**: Check `conversion_flag` logic against actual data
- **Memory errors on large datasets**: Narrow time range or increase `top_k_channel_perc`
- **NULL channels dominate**: Improve UTM tagging or adjust `backfill_partition_col`

### Re-running After Config Change
1. Update `input_params.yml`
2. `tdx wf push` to deploy changes
3. `tdx wf run` to execute with new config
4. Verify output tables have expected row counts

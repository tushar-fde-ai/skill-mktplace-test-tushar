# NBA Engagement Scores — Production Runbook

Production architecture, output schema, and operational reference for the NBA Engagement Scores workflow.

## Workflow Overview

NBA unions customer behavioral data from multiple sources, derives per-profile engagement scores across **Channel** (where), **Time of Day** (when), and **Campaign** (what), and writes a single output table joinable to a Parent Segment in Audience Studio.

- **GitHub repository**: `https://github.com/treasure-data-ps/nba_eng_scores`
- **Workflow project name**: `nba_eng_prod`
- **Workflow path**: `nba_eng_scores/td_wf/`
- **Workflow entry point**: `nba_eng_launch.dig`
- **Config file**: `nba_eng_scores/td_wf/config/input_params.yml`

## Architecture

```
Source Tables (pageviews, email, sales, orders, custom)
    ↓
Union & Sessionize (nba_combined_user_events)
    ↓
Parse channel from UTM + regex rules (utm_parcing)
    ↓
┌──────────────────┬──────────────────┬──────────────────┐
│ Next Best        │ Next Best Time   │ Next Best        │
│ Channel          │ (daypart bucket) │ Campaign         │
│ (per-channel     │                  │ (cart-abandon +  │
│  affinity score) │                  │  new-visitor)    │
└──────────────────┴──────────────────┴──────────────────┘
    ↓                       ↓                       ↓
              Combine on ${unique_user_id}
                          ↓
            nba_combined_metrics_final
                          ↓
        Dashboard tables → TI Dashboard / NBA Insights Agent
```

## Output Tables

All written to `sink_database`:

| Table | Description |
|-------|-------------|
| `nba_combined_metrics_final` | **Primary output** — per-profile NBA scores, joinable to Parent Segment |
| `nba_combined_user_events` | Sessionized union of all touchpoint sources |
| `nba_combined_user_events_final` | Channel-parsed version (after `utm_parcing`) used as input to all NBA blocks |
| `nba_next_best_channel` | Per-profile channel-affinity scores |
| `nba_next_best_time` | Per-profile daypart-affinity scores |
| `nba_next_best_campaign` | Combined cart-abandon + new-visitor + ad-engagement flags |
| `nba_cart_abandon` | Profiles flagged as having abandoned a cart in the lookback window |
| `nba_new_visitor_no_ads` | Profiles flagged as new visitors not driven by paid ads |
| `nba_custom_campaign_flags` | Custom campaign flags (extension hook) |
| `nba_dash_stats_summary` | Dashboard — aggregate run statistics |
| `nba_dash_model_metrics` | Dashboard — per-metric score distributions |
| `nba_dash_source_tables` | Dashboard — per-source row counts and time ranges |

## Scoring Strategies

Set via `scoring_logic` in `input_params.yml`:

| Strategy | When to use |
|----------|-------------|
| `percentile` | Default for evenly distributed activity. Score = percentile rank within audience. |
| `quartile` | Coarse 4-bucket banding. Easier to communicate to marketers. |
| `minmax` | Hivemall min-max scaling. Best when activity distribution is heavily skewed. |

## Time-of-Day Buckets (default)

```yaml
day_breakdown:
  - period: morning      # time_hour 6-11
  - period: afternoon    # time_hour 12-17
  - period: evening      # time_hour 18-23
  - period: overnight    # time_hour 0-5
```

Override `day_breakdown` in `next_best_time` if the customer wants finer granularity (hourly, weekday/weekend split, etc.).

## Configuration Reference

For workflow setup and configuration, see `../../workflow-setup/references/`:
- `workflow_setup_guide.md` — full setup walkthrough (10 steps)
- `requirements_doc.md` — requirements gathering template (12 NBA-specific questions)
- `yaml_structure.md` — complete parameter reference
- `table_configuration.md` — per-source-type configuration
- `input_params_template.yml` — working example config
- `github_instructions.md` — clone, push, secrets, scheduling

## Operational Runbook

### Running the Workflow

```bash
# From inside nba_eng_scores/td_wf
tdx wf push -y                                    # default name from tdx.json: nba_eng_prod
tdx wf run nba_eng_prod.nba_eng_launch
tdx wf sessions nba_eng_prod --status running
```

### Monitoring

```bash
tdx wf timeline nba_eng_prod.nba_eng_launch --follow
tdx wf attempt <id> tasks
tdx wf attempt <id> logs +failed_task
```

### Validate Output

```sql
-- Final output exists and has expected row count
SELECT COUNT(*) AS profiles_scored FROM sink_database.nba_combined_metrics_final;

-- Score distribution by strategy
SELECT
  COUNT(*) AS total_profiles,
  COUNT(CASE WHEN cart_abandon_flag = 1 THEN 1 END) AS cart_abandoners,
  COUNT(CASE WHEN new_visitor_flag = 1 THEN 1 END) AS new_visitors
FROM sink_database.nba_combined_metrics_final;

-- Channel coverage (NBC scores should be non-null for active users)
SELECT
  COUNT(*) AS total,
  COUNT(nbc_score) AS has_channel_score,
  COUNT(nbt_score) AS has_time_score
FROM sink_database.nba_combined_metrics_final;
```

### Common Failure Points

| Problem | Action |
|---------|--------|
| Source table missing or renamed | Verify `src_table` values in `aggregate_metrics_tables` |
| `nba_combined_user_events` row count is 0 | Check `custom_filter` per source — too aggressive filters drop everything |
| Cart-abandon flag count is 0 | `abandon_regexp` doesn't match the customer's add-to-cart event name; update both `abandon_regexp` and `abandon_regexp_string` (must stay in sync) |
| New-visitor flag count is 0 | Increase `new_customers_days` or `max_number_visits` thresholds |
| Channel scores all NULL | UTM coverage on pageviews is too low; either tag UTM upstream or relax `min_percent` in `utm_parcing` |
| Memory errors | Narrow `lookback_period` or increase `top_k_channel_perc` to collapse rare channels |
| Dashboard tables empty | Check `create_dashboard: 'yes'` and that `secret_key` workflow secret is set |

### `abandon_regexp` Sync Rule

The two copies must stay identical (one for raw SQL, one for interpolated SQL with doubled quotes):

```yaml
abandon_regexp: REGEXP_LIKE(lower(event_type), '(?=.*add)(?=.*cart)')
abandon_regexp_string: REGEXP_LIKE(lower(event_type), ''(?=.*add)(?=.*cart)'')
```

Update both if the customer logs add-to-cart as `basket_add`, `cart_item_added`, etc.

### Re-running After Config Change

1. Update `config/input_params.yml`
2. `tdx wf push -y` to deploy changes
3. `tdx wf run nba_eng_prod.nba_eng_launch` to execute
4. Verify output tables have expected row counts

### Scheduling

```bash
tdx wf schedule --set nba_eng_prod.nba_eng_launch --cron "0 6 * * *"
```

### Secrets

NBA needs the TD API key for dashboard creation:

```bash
tdx wf secrets --set secret_key=<TD_API_KEY> --project nba_eng_prod
```

## Companion Agent

The NBA workflow ships with one Foundry agent (`NBA Insights Agent`) that reads the three `nba_dash_*` dashboard tables.

- **Deploy the agent template** for a customer → use the `fde-nba-scores-foundry-skill` (separate registered skill)
- **Query the dashboard tables directly inside Treasure Work** → use the `fde-nba-scores-insights-agent` (separate registered skill)

## Activation in Audience Studio

`nba_combined_metrics_final` is shaped for direct ingestion as a Parent Segment attribute table:
- Join key: `${unique_user_id}` (typically `canonical_id`)
- Pre-computed columns: `nbc_score`, `nbt_score`, `cart_abandon_flag`, `new_visitor_flag`, plus per-channel and per-daypart score columns
- Refresh cadence: typically daily or every 6 hours

Build child segments using these columns directly (e.g., `cart_abandon_flag = 1 AND nbt_evening > 0.7`).

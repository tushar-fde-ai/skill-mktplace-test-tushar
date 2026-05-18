# NBA Engagement Scores — YAML Configuration Structure

This document is the complete reference for `td_wf/config/input_params.yml` in the `nba_eng_scores` workflow project. Use it alongside `input_params_template.yml` (a working filled-in example) and `table_configuration.md` (per-source-type guidance).

The workflow is composed of seven configuration sections. Skip any section your customer doesn't use, but every key documented as required must be present.

## Section Map

| Section | Purpose |
|---------|---------|
| Global Params | Where output goes, dashboard switch, timezone, scoring strategy |
| Filter Params | Time window applied across all sources |
| Output Tables | Names of the union table and final per-profile metrics table |
| `aggregate_metrics_tables` | List of source tables to union (web, email, sales, orders, etc.) |
| `next_best_time` | Daypart breakdown for the time-of-day affinity score |
| `utm_parcing` | UTM parsing rules for the channel affinity score |
| `next_best_campaign` | Cart-abandon, new-visitor, and ad-engagement business rules |
| `temporary_tables` | Output table names + which columns to carry into the final table |

---

## Global Parameters

| Parameter | Type | Description | Example |
|-----------|------|-------------|---------|
| `sink_database` | string | Database for ALL output tables | `td_agents`, `analytics_prod` |
| `unique_user_id` | string | Profile join key across every source table | `canonical_id`, `cdp_profile_id` |
| `api_endpoint` | string | TD region endpoint | `api.treasuredata.com`, `api.eu01.treasuredata.com` |
| `model_config_table` | string | Lookup table tracking which TI datamodels already exist | `datamodel_build_history` |
| `create_dashboard` | yes/no | If `yes`, runs `nba_datamodel_create.dig` + `nba_datamodel_build.dig` after the metrics build | `yes` |
| `cleanup_temp_tables` | yes/no | If `yes`, drops every temp table after the final metrics table is written | `yes` |
| `prefix` | string | Prepended to every generated output table name | `nba` |

## Filter Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `time_filter_type` | string | `range` for fixed start/end dates OR `interval` to look back from `time_range_end_date` |
| `time_range_start_date` | date | YYYY-MM-DD; used with `range` |
| `time_range_end_date` | date | YYYY-MM-DD; leave default `2222-22-22` to always end at the latest available data |
| `lookback_period` | string | Used only with `interval`. Examples: `-1M`, `-30d`, `-2w`, `-180d` |
| `session_length` | int | Seconds of inactivity before a new session starts. `3600` = 1 hr |
| `backfill_partition_col` | string | `${unique_user_id}` backfills NULL channels across the whole journey; `session_id` only within a session |
| `top_k_channel_perc` | float | Channels below this share of touchpoints collapse into `'others'` |
| `top_k_conv_perc` | float | Channels below this share of conversions collapse into `'others'` |
| `time_zone` | string | Three-letter IANA-ish code, e.g. `'UTC'`, `'JST'`, `'PST'` |
| `timeshift_change` | string | `'+'` adds, `'-'` subtracts `timeshift_hours` from the source timestamp |
| `timeshift_hours` | int | Hours to shift original timestamps to the customer's marketing timezone |
| `scoring_logic` | string | `percentile` \| `quartile` \| `minmax`. See **Scoring Strategy** below |
| `topk_values` | int | Top-K distinct values surfaced in `nba_dash_stats_summary` per metric |

### Scoring Strategy

Set via `scoring_logic`:

| Strategy | Behavior | Use when |
|----------|----------|----------|
| `percentile` | Score = percentile rank within the audience | Default; activity is reasonably evenly distributed |
| `quartile` | Buckets users into 4 evenly-sized groups | Marketers want simple "top quartile" segments |
| `minmax` | Hivemall min-max scaling (0-1) | Activity distribution is heavily skewed (a few power users dominate) |

## Output Tables

| Parameter | Description |
|-----------|-------------|
| `src_union_table` | Intermediate unioned-activity table written before scoring |
| `final_nba_metrics_table` | Final wide per-profile table with all NBA scores and flags. **This is the table you join to the Parent Segment.** |

---

## `aggregate_metrics_tables` — Source Tables to Union

Each entry below becomes one source feeding the unioned activity table. Add as many as you have.

| Parameter | Type | Description |
|-----------|------|-------------|
| `src_table` | string | Full `database.table_name` of the source |
| `name` | string | Short identifier for this source (used in event_type concatenation and SQL file lookup) |
| `unixtime_col` | string | UNIX-time column. If the source only has datetime, wrap with `TD_TIME_PARSE(...)` |
| `join_key` | string | Profile ID column on this table (almost always `${unique_user_id}`) |
| `market_col` | SQL/string | Optional market label. Use `"''none''"` if N/A |
| `brand_col` | SQL/string | Optional brand label. Use `"''none''"` if N/A |
| `url_col` | string/SQL | URL column for UTM extraction; `CAST(NULL AS VARCHAR)` for non-web sources |
| `referral_col` | string/SQL | Referrer column or hardcoded label like `"''email''"` |
| `medium_col` | string/SQL | Marketing medium — UTM extraction or hardcoded |
| `source_col` | string/SQL | Marketing source — UTM extraction or hardcoded ESP name (`"''sfmc''"`) |
| `campaign_col` | string/SQL | Campaign column or normalized expression |
| `context_col` | string/SQL | Touchpoint context (page title, email name, sales topic, order type) |
| `event_type` | string/SQL | Event-type label expression — usually `CONCAT('<source>_', event_type)` |
| `custom_filter` | string | SQL WHERE clause |
| `conversion_flag` | float/SQL | `0.0` for touchpoints, `1.0` for conversion sources, or a SQL expression like `IF(REGEXP_LIKE(lower(td_path), 'thank|download'), 1.0, 0.0)` |
| `item_price` | float/string | `0.0` for non-monetary sources, or a column name like `unit_price` for the conversion source |
| `apply_time_filter` | bool | Whether the global time filter is applied to this source |
| `query_type` | string | Blank to use the YAML-driven SQL; `'custom'` to read from `sql/src_tables/<name>.sql` |

> **Important:** Inside YAML strings, single quotes are doubled (`''…''`) because the SQL is later interpolated into another SQL template. Match the template exactly when in doubt.

---

## `next_best_time` — Time-of-Day Affinity

```yaml
next_best_time:
  outputs:
    database: ${sink_database}
    union_table: nba_combined_user_events_final
    activity_period_column: activity_period
    day_breakdown:
      - period: morning
        logic: time_hour >= 6 AND time_hour <= 11
      - period: afternoon
        logic: time_hour >= 12 AND time_hour <= 17
      - period: evening
        logic: time_hour >= 18 AND time_hour <= 23
      - period: overnight
        logic: time_hour >= 0 AND time_hour <= 5
```

| Parameter | Description |
|-----------|-------------|
| `outputs.database` | Where the temp/final NBT tables go (almost always `${sink_database}`) |
| `outputs.union_table` | The post-UTM-parsed activity table this section reads from |
| `outputs.activity_period_column` | Column holding the assigned daypart label |
| `day_breakdown` | List of `(period, logic)` pairs. The `logic` is raw Trino SQL evaluated against `time_hour`. Override the default 6-hour buckets if the customer wants finer granularity (e.g., 8 three-hour periods). |

---

## `utm_parcing` — Channel Affinity

```yaml
utm_parcing:
  date_field: time
  channel_column: channel
  source_column: source
  exclude_channel_regexp: '%|!|@|#|\(|\)'
  min_percent: 0.4

  outputs:
    union_table: nba_combined_user_events_final
```

| Parameter | Description |
|-----------|-------------|
| `date_field` | Timestamp column on the unioned activity table |
| `channel_column` | Output column name for the resolved channel |
| `source_column` | Output column name for the resolved source |
| `exclude_channel_regexp` | Regex of characters that disqualify a row from being treated as a real channel signal (junk UTM params) |
| `min_percent` | Minimum share of a channel within a profile's activity to keep that signal. Lowering this gives more profiles a channel score but adds noise. |

The actual channel→regex mapping (`social: 'faceboo|instag|tik|snap|twitt'`, etc.) lives in the workflow SQL templates and is shared across customers. If a customer needs a custom channel taxonomy, edit `td_wf/queries/next_best_channel/` directly.

---

## `next_best_campaign` — Cart-Abandon, New-Visitor, Ad-Engagement Flags

```yaml
next_best_campaign:
  database: ${sink_database}
  src_table: nba_combined_user_events_final
  join_key: ${unique_user_id}
  date_field: time
  abandon_regexp: REGEXP_LIKE(lower(event_type), '(?=.*add)(?=.*cart)')
  abandon_regexp_string: REGEXP_LIKE(lower(event_type), ''(?=.*add)(?=.*cart)'')
  conversion_logic: conversion_flag > 0
  ad_engagement_logic: channel IN ('paid search', 'tiktok', 'web ads', 'dsp_rtb', 'maddict', 'call center', 'others', 'display')
  ad_engagement_logic_string: channel IN (''paid'', ''search'', ''tiktok'', ''web ads'', ''dsp_rtb'', ''maddict'', ''call center'', ''others'', ''display'')
  event_lookback: 90
  new_customers_days: 45
  max_number_visits: 15
  custom_flags_table: nba_custom_campaign_flags
```

| Parameter | Description |
|-----------|-------------|
| `abandon_regexp` / `abandon_regexp_string` | Two copies of the same expression — one used in raw SQL, one (with doubled quotes) in interpolated SQL. Keep them in sync. Customize for the customer's add-to-cart event signature. |
| `conversion_logic` | What counts as a conversion when scoring cart-abandon. Default `conversion_flag > 0` — only override if conversions are tracked in a non-standard way. |
| `ad_engagement_logic` / `ad_engagement_logic_string` | Channels considered "paid/ad" for the new-visitor rule. Two copies, same sync rule as above. |
| `event_lookback` | Days back to scan for cart-abandon signals |
| `new_customers_days` | Max age (in days) of first visit to qualify as a "new visitor" |
| `max_number_visits` | Max page visits a "new visitor" can have. Increase if the customer has chatty pages (multi-step checkouts, gallery scrolling). |
| `custom_flags_table` | Output table for the boolean flags |

### New-Visitor Rule (codified)

A profile is flagged `new_visitor = 1` when **all** of the following hold within the lookback window:
1. First visit occurred ≤ `new_customers_days` ago
2. Total page visits ≤ `max_number_visits`
3. Was NOT referred by any source matching `ad_engagement_logic`
4. Did NOT land on a URL containing a `utm_*` ad-click parameter
5. Has NOT triggered a conversion event

If any of those is too tight or too loose for a given customer, this is the section to adjust.

---

## `temporary_tables`

```yaml
temporary_tables:
  next_best_channel: nba_next_best_channel
  next_best_time: nba_next_best_time
  next_best_campaign: nba_next_best_campaign
  cart_abandon: nba_cart_abandon
  new_visitors: nba_new_visitor_no_ads
  regexp_columns: 'visitor|aban|perc|minmax'

  table_list:
    - nba_next_best_time
    - nba_next_best_channel
    - nba_next_best_campaign
    - nba_cart_abandon
    - nba_new_visitor_no_ads
    - schema
```

| Parameter | Description |
|-----------|-------------|
| Top-level keys (`next_best_channel`, etc.) | Names of the per-metric temp tables. Rename only if you have a naming-convention requirement. |
| `regexp_columns` | Regex of column names to **keep** when joining temp tables into `final_nba_metrics_table`. Default keeps the `*_visitor`, `*_abandon`, `*_perc`, `*_minmax` score/flag columns and drops everything else. Add to it if you create a custom score column. |
| `table_list` | List of temp tables that get cleaned up when `cleanup_temp_tables: 'yes'`. Always include `schema` — it stores the column schema for the dashboard tables. |

---

## Common Parameter Changes Per Customer

When deploying for a new customer, the parameters that almost always need to change are:

1. `sink_database` — the customer's working database
2. `unique_user_id` — usually `canonical_id` post-unification, sometimes a custom ID
3. `aggregate_metrics_tables` — every source table name + columns
4. `conversion_flag` on the pageviews source — what URL path indicates conversion
5. `event_lookback` / `new_customers_days` / `max_number_visits` — business-rule windows
6. `scoring_logic` — based on the customer's data distribution
7. `time_zone` / `timeshift_*` — match the customer's marketing operations timezone

Everything else can stay at template defaults for the first run and be tuned later based on `nba_dash_stats_summary` distributions.

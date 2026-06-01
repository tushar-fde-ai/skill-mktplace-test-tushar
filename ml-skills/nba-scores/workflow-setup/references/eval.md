# NBA Engagement Scores — Workflow Output Validation

Run these checks after every workflow execution (Phase 3 minimal push and Phase 5 re-run). Use `tdx-skills:tdx-basic` to execute each query against `${sink_database}`.

---

## 1. Primary Output Table — `nba_combined_metrics_final`

**What it is:** Final per-profile NBA scores. This is the table joined to the Parent Segment in Audience Studio.

```sql
-- Row count and distinct profile count
SELECT COUNT(*) AS total_rows, COUNT(DISTINCT canonical_id) AS distinct_profiles
FROM ${sink_database}.nba_combined_metrics_final;
```

**Pass criteria:**
- `total_rows > 0`
- `distinct_profiles` is in the same order of magnitude as the Parent Segment profile count (within ~20%)

```sql
-- Confirm expected score columns exist and are non-null
SELECT
  COUNT(*) AS total,
  COUNT(CASE WHEN next_best_channel IS NOT NULL THEN 1 END) AS has_channel,
  COUNT(CASE WHEN next_best_time IS NOT NULL THEN 1 END) AS has_activity_period,
  COUNT(CASE WHEN cart_abandon_flag IS NOT NULL THEN 1 END) AS has_cart_abandon,
  COUNT(CASE WHEN new_visitor_flag IS NOT NULL THEN 1 END) AS has_new_visitor
FROM ${sink_database}.nba_combined_metrics_final;
```

**Pass criteria:** All four score/flag columns are non-null for the majority of rows (>80%).

```sql
-- Score distribution sanity check (quartile strategy → expect 4 distinct values per score)
SELECT next_best_channel, COUNT(*) AS cnt
FROM ${sink_database}.nba_combined_metrics_final
GROUP BY 1 ORDER BY 2 DESC LIMIT 20;
```

**Pass criteria (quartile):** Channel values should resolve to recognizable channel names (e.g., `email`, `social`, `search`, `direct`) — not mostly `others`. If >70% of profiles land in `others`, UTM coverage may be too low or `top_k_channel_perc` needs tuning.

---

## 2. Dashboard Table — `nba_dash_stats_summary`

**What it is:** Per-metric score distribution stats for the latest run. Consumed by the NBA Insights Agent.

```sql
SELECT MAX(session_id) AS latest_session FROM ${sink_database}.nba_dash_model_metrics;
```

```sql
SELECT metric_name, metric_value, profile_count, converted_users
FROM ${sink_database}.nba_dash_stats_summary
WHERE session_id = (SELECT MAX(session_id) FROM ${sink_database}.nba_dash_model_metrics)
ORDER BY metric_name, profile_count DESC
LIMIT 50;
```

**Pass criteria:**
- Rows exist for the latest session
- `metric_name` values cover channel, activity_period, cart_abandon, new_visitor
- Score distributions look reasonable — for quartile, expect roughly equal bucket sizes (~25% each)

---

## 3. Dashboard Table — `nba_dash_model_metrics`

**What it is:** Config audit table — one row per source table per run. Stores the exact `input_params.yml` parameters used for each source (filters, conversion logic, lookback period, etc.) plus `profiles_scored` (total profiles in the union before scoring). Use it to verify the correct config was applied and to debug unexpected results.

```sql
-- One row per source table — confirm all expected sources ran
SELECT event_name, src_table, profiles_scored, lookback_period,
       apply_time_filter, conversion_flag, custom_filter
FROM ${sink_database}.nba_dash_model_metrics
WHERE session_id = (SELECT MAX(session_id) FROM ${sink_database}.nba_dash_model_metrics)
ORDER BY event_name;
```

**Pass criteria:**
- One row per configured source (expect 4 rows: `pageviews`, `email_events`, `sales_rep_interactions`, `order_events`)
- `profiles_scored` is the same value across all rows (all sources feed the same union) and is ≥ `distinct_profiles` in `nba_combined_metrics_final`
- `apply_time_filter` = `true` for all sources (confirms time filter was applied)
- `lookback_period` matches the configured value (e.g. `-365d`)
- `conversion_flag` = `1` only for `order_events`; `0.0` for all other sources
- `custom_filter` for `email_events` contains `send` exclusion
- `custom_filter` for `order_events` contains `order_status` filter

---

## 4. Dashboard Table — `nba_dash_source_tables`

**What it is:** One row per source table per run — event counts and profile counts per source.

```sql
SELECT source_table, num_events, unique_profiles, total_conversions, total_spend, day_range, min_date, max_date
FROM ${sink_database}.nba_dash_source_tables
WHERE session_id = (SELECT MAX(session_id) FROM ${sink_database}.nba_dash_model_metrics)
ORDER BY num_events DESC;
```

**Pass criteria:**
- One row per configured source table (expect 4 rows: `pageviews`, `email_events`, `sales_rep_interactions`, `order_events`)
- `event_count` for each source is non-zero and proportional to what we know about the data:
  - `pageviews` should have the highest event count (~568K rows in source)
  - `order_events` should be lowest (~97K valid rows after status filter)
  - `email_events` and `sales_rep_interactions` should fall in between
- `profile_count` per source should be less than or equal to `total_profiles` in `nba_dash_model_metrics`

---

## 5. Flag Sanity Checks

```sql
-- Cart-abandon flag distribution
SELECT cart_abandon_flag, COUNT(*) AS cnt
FROM ${sink_database}.nba_combined_metrics_final
GROUP BY 1;

-- New-visitor flag distribution
SELECT new_visitor_flag, COUNT(*) AS cnt
FROM ${sink_database}.nba_combined_metrics_final
GROUP BY 1;
```

**Pass criteria:**
- Both flags have a mix of 0 and 1 values — all-zero means the regexp or lookback window is not matching
- Cart-abandon rate should be a small fraction of total profiles (typically 5–20%)
- New-visitor rate depends on data age but should not be 0% or 100%

---

## Common Failure Modes

| Symptom | Likely cause | Fix |
|---|---|---|
| `nba_combined_metrics_final` is empty | Workflow failed mid-run or `sink_database` doesn't exist | Check attempt logs: `tdx wf attempt <id> logs +nba_metrics_table` |
| All `channel` values are `others` | UTM coverage too low or `top_k_channel_perc` too high | Lower `top_k_channel_perc` to `0.0001`; check UTM coverage on pageviews |
| `cart_abandon` is all 0 | `abandon_regexp` doesn't match actual event_type values | Query `event_type` distinct values from `nba_combined_user_events` and update `abandon_regexp` |
| `nba_dash_source_tables` missing a source | That source table had 0 rows after time filter | Check `apply_time_filter` + `lookback_period`; verify source table has data in the window |
| Profile count much lower than Parent Segment | Source tables don't cover the full audience | Confirm `canonical_id` join key is consistent; check for NULL profile IDs in source tables |

---
name: nba-scores-master
description: |
  NBA (Next Best Action) Engagement Scores for Treasure Data. Configures workflows that union customer activity, derive per-profile Next Best Channel / Next Best Time / Next Best Campaign affinity scores plus cart-abandon and new-visitor flags, and write the combined output into Audience Studio. Trigger on: NBA, Next Best Action, engagement scores, channel affinity, time of day affinity, cart abandon, new visitor, NBA dashboard.
---

# NBA Engagement Scores

Next Best Action (NBA) workflow that unions customer behavioral data from multiple sources, scores per-profile engagement across **Channel** (where), **Time of Day** (when), and **Campaign** (what), and combines the metrics into a single profile-level table that can be joined to a Parent Segment in Audience Studio.

## Sub-Folder Routing

| Task | Action |
|------|--------|
| Configure the NBA workflow (`input_params.yml`) | Read `workflow-setup/SKILL.md` |
| Build / deploy the NBA Insights companion Foundry agent | Read `agent-skills/SKILL.md` |
| Production docs, output tables, operational runbook | Read `prod-docs/SKILL.md` |

## Quick Reference

- **GitHub repo**: `https://github.com/treasure-data-ps/nba_eng_scores`
- **Workflow project name**: `nba_eng_prod`
- **Workflow path**: `nba_eng_scores/td_wf/`
- **Workflow entry point**: `nba_eng_launch.dig`
- **Config file**: `nba_eng_scores/td_wf/config/input_params.yml`
- **Foundry agent project**: `NBA Engagement Scores` (single-agent: `NBA Insights Agent`)
- **Key output**: `nba_combined_metrics_final` — per-profile NBA scores joinable to Parent Segment
- **Dashboard tables**: `nba_dash_stats_summary`, `nba_dash_model_metrics`, `nba_dash_source_tables`

## How It Works

1. **Union activity**: Combine pageviews, email events, sales-rep interactions, orders into a single `nba_combined_user_events` table (sessionized by `session_length`).
2. **Next Best Channel**: Parse UTM params + apply channel regex rules → per-profile affinity score per channel (e.g. social, email, search). Scored via `percentile`, `quartile`, or `minmax` (Hivemall) — chosen by config.
3. **Next Best Time**: Bucket activity into morning / afternoon / evening / overnight → per-profile affinity score per daypart, same scoring strategy.
4. **Next Best Campaign**: Apply business-rule flags — **cart-abandon** (added to cart in lookback window without conversion) and **new-visitor** (recent first visit, low page count, not paid-referred, no purchase).
5. **Combine**: Join all metric tables on `${unique_user_id}` into `nba_combined_metrics_final`.
6. **Dashboard stats**: Append config + per-source stats + score distributions to `nba_dash_*` tables for the TI dashboard and the NBA Insights agent.

## Scoring Strategies

Set via `scoring_logic` in `input_params.yml`:

| Strategy | When to use |
|----------|-------------|
| `percentile` | Default for evenly distributed activity. Score = percentile rank within audience. |
| `quartile` | Coarse 4-bucket banding. Easier to communicate to marketers. |
| `minmax` | Hivemall min-max scaling. Best when activity distribution is heavily skewed. |

---
name: nbp
description: |
  MTA (Multi-Touch Attribution) Journey Analytics for Treasure Data. Configures workflows that build unified customer journeys and run attribution models (Markov, Shapley). Trigger on: MTA, multi-touch attribution, channel attribution, conversion paths, journey analytics, Markov, Shapley, marketing attribution, channel mix, CPA, CPB.
---

# MTA Journey Analytics

Multi-Touch Attribution workflow that builds unified customer journeys from multiple touchpoint sources and runs attribution models to measure channel contribution to conversions.

## Sub-Folder Routing

| Task | Action |
|------|--------|
| Configure the MTA workflow (`input_params.yml`) | Read `workflow-setup/SKILL.md` |
| Production docs, output tables, operational runbook | Read `prod-docs/SKILL.md` |
| Build a companion LLM agent for MTA analysis | Read `agent-setup/` references (scaffold only) |

## Quick Reference

- **GitHub repo**: `https://github.com/treasure-data-ps/mta_journey_analysis`
- **Workflow path**: `mta_journey_analysis/td_wf/mta_journey_agent/`
- **Config file**: `mta_journey_agent/config/input_params.yml`
- **Key output**: Attribution scores per channel (Markov, Shapley, linear, time-decay)

## How It Works

1. **Union**: Combine pageviews, email, sales, orders into a single touchpoint table
2. **Sessionize**: Group touchpoints into sessions based on inactivity gap
3. **Build journeys**: Create per-customer journey sequences with conversion flags
4. **Attribute**: Run Markov, Shapley, linear, time-decay models
5. **Output**: Channel attribution scores, top conversion paths, spend efficiency


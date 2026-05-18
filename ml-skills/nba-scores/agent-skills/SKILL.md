---
name: nba-scores-agent-skills
description: |
  Routes between two NBA agent paths: deploying the NBA Insights Foundry agent template to a customer's TD instance, or using the local NBA Insights skill directly inside Treasure Work to query the NBA dashboard tables.
---

# NBA Agent Skills — Routing Guide

The NBA workflow ships with one companion AI agent (`NBA Insights Agent`) that reads three dashboard tables (`nba_dash_stats_summary`, `nba_dash_model_metrics`, `nba_dash_source_tables`) and explains what the model is telling the user.

You can interact with it two ways:

## Path A — Deploy the Foundry Agent template to a customer's TD instance

If the user wants to **set up the NBA Insights Agent in TD AI Foundry** for a customer (so end users can chat with it inside the TD console):
- Read `foundry-agent/SKILL.md`
- Covers cloning the template, configuring knowledge bases for the customer's `sink_database`, naming the project, and `tdx agent push`.

## Path B — Use the NBA Insights skill directly inside Treasure Work

If the user is **sending NBA analysis questions directly in Claude / Treasure Work** (no TD Foundry deployment, just local analysis against the dashboard tables):
- Read `nba-insights/SKILL.md`
- Covers the three tables, the SQL query patterns for each common question type, and the visualization rules.

## Quick decision

| User prompt looks like… | Route to |
|------------------------|----------|
| "Deploy the NBA agent for this customer" | `foundry-agent/SKILL.md` |
| "Push the NBA Insights Agent" | `foundry-agent/SKILL.md` |
| "Set up the NBA Foundry agent" | `foundry-agent/SKILL.md` |
| "Summarize the latest NBA run" | `nba-insights/SKILL.md` |
| "What's the channel affinity distribution?" | `nba-insights/SKILL.md` |
| "How many users were flagged as cart-abandon last run?" | `nba-insights/SKILL.md` |
| "Compare the last two NBA runs" | `nba-insights/SKILL.md` |

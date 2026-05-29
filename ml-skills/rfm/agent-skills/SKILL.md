---
name: fde-rfm-agent-skills
description: |
  Route to correct folder depending on whether Foundry Agent needs to be built from scratch or you want to directly use the RFM analysis skill. Trigger when users mention RFM agent, RFM Foundry agent, or RFM analysis agent.
---

# Routing Guide

## Foundry Agent
If user wants to set up the RFM Analysis Agent template in Foundry then refer to `foundry-agent/SKILL.md`

## TD Studio tdx Claude Skill
If user is sending direct questions to the RFM analysis agent in Claude, route prompts to `rfm-analysis/SKILL.md`

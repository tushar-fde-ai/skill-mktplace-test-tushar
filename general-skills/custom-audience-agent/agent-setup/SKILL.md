---
name: fde-custom-audience-agent-setup
description: |
  Setup and deploy a custom audience agent to TD AI Foundry. Covers requirements gathering, agent template configuration, and deployment.
---

# Custom Audience Agent — Agent Setup

This skill handles the setup and deployment phases for a custom audience agent.

## Routing

| Task | Action |
|------|--------|
| Gather requirements for a new audience agent | Read `references/requirements_doc.md` and follow the requirements workflow |
| Generate eval prompts for testing the agent | Read `references/eval.md` and follow the eval framework workflow |

## Requirements Gathering Summary

The requirements doc (`references/requirements_doc.md`) walks through:

1. **Locate the customer's Confluence folder** — search CUST space, create FDE Solutions and Custom Audience Agent sub-folders if needed
2. **Check for existing requirements** — ask if user has a pre-filled doc
3. **Collect initial questions** — data readiness, tables to include, business context, target users, output preferences, data restrictions
4. **Validate & confirm** — present the full requirements summary and get user approval before proceeding

After requirements are confirmed, proceed to agent template setup (Phase 3 in the parent `SKILL.md`).

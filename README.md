# FDE Skills

Templatized skill library for deploying ML solutions and custom AI agents on Treasure Data. Designed as a hierarchical routing system for Claude Code / Treasure Studio.

## Routing Waterfall

The root `SKILL.md` is the entry point. It reads the user's request, matches keywords, and routes to the correct sub-folder by reading that folder's `SKILL.md`. This cascades down until a leaf skill handles the task:

```
SKILL.md  (Root — routes to ml-skills/ or general-skills/)
  │
  ├── ml-skills/SKILL.md  (Routes by keyword: rfm/, mta/, nba/, nbp/)
  │     │
  │     └── <workflow>/SKILL.md  (Routes to sub-task folder below)
  │           ├── workflow-setup/SKILL.md  + references/
  │           ├── agent-skills/SKILL.md    + references/
  │           └── prod-docs/SKILL.md       + references/
  │
  └── general-skills/SKILL.md  (Routes: custom-audience-agent/, custom-analytics-agent/, etc.)
        │
        └── <solution>/SKILL.md  (Routes to sub-task folder below)
              ├── agent-setup/SKILL.md     + references/
              └── prod-docs/SKILL.md       + references/
```

At every level, if the request is ambiguous the SKILL.md instructs the agent to ask a clarifying question.

## Repeating Folder Template

Every workflow or solution folder follows the same structure:

```
<skill-name>/
├── SKILL.md              # Overview, routing table, quick reference
├── workflow-setup/       # TD workflow configuration (ML skills)
│   ├── SKILL.md
│   └── references/       # YAML templates, table config docs, GitHub instructions
├── agent-skills/         # Companion LLM agent (Foundry or Claude skill)
│   ├── SKILL.md
│   ├── foundry-agent/    # TD AI Foundry agent setup
│   └── <claude-skill>/   # Claude skill with domain-specific references
├── prod-docs/            # Production docs, runbooks, customer-facing material
│   ├── SKILL.md
│   └── references/
```

The `references/` folders contain supporting material: YAML templates, data dictionaries, eval frameworks, requirements docs, and visualization instructions.

## Adding a New Skill

1. Create a folder under `ml-skills/` or `general-skills/`.
2. Add a `SKILL.md` with YAML frontmatter (`name`, `description`) and sub-task routing.
3. Create the standard sub-folders (`workflow-setup/`, `agent-skills/`, `prod-docs/`) as needed.
4. Add a keyword entry in the parent category's `SKILL.md` routing table.

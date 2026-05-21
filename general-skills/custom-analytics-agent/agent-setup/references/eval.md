# Analytics Agent — Test Categories & Failure-to-Fix Mapping

> **STATUS — Scaffold.** Analytics-specific test categories TBD once template repo defines the agent's tools and capabilities.

This file holds the **analytics-specific content** for the test case lifecycle. The generic two-round flow + Confluence page format + `tdx agent test` mechanics live in `../../../shared/test_cases_pattern.md`.

## Skills to Load

- `tdx-skills:agent-test` — for `tdx agent test` mechanics, `test.yml` format, output parsing
- TODO: schema-discovery skill once template repo defines which databases the agent queries

## Test Case Categories

Generate 10-15 cases total across the categories below. Mix complexity (simple / medium / complex).

TODO — likely categories for an analytics agent (refine once template lands):

### 1. TODO — Schema / Data Discovery (2-3 cases)
Confirms the agent knows which tables and columns it has access to.

Example prompts:
- "What tables can you query?"
- "Show me the schema of [table]."

Pass criteria: TODO

### 2. TODO — SQL Generation (2-3 cases)
Single-query analytical questions.

Example prompts:
- "What were total sales by region last quarter?"
- "Which products have the highest margin?"

Pass criteria: TODO

### 3. TODO — Multi-step Analysis (2-3 cases)
Questions requiring multiple queries or follow-up reasoning.

Example prompts: TODO

Pass criteria: TODO

### 4. TODO — Dashboard / Visualization (2-3 cases)
Requests for charts or dashboards.

Example prompts: TODO

Pass criteria: TODO

### 5. TODO — Ambiguous / Guardrail (2-3 cases)
- "Tell me about the data." (should ask for scope)
- "Write Python code." (should refuse if guardrail set)
- "[off-topic question]" (should politely decline)

Pass criteria: TODO

## Example test.yml entries

TODO once categories are finalized.

## Round 2 — Failure-to-Fix Mapping

TODO: which file to edit for each common failure pattern.

| Failure pattern | Fix location |
|-----------------|--------------|
| TODO | TODO |

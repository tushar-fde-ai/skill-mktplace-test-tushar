# Audience Agent — Test Categories & Failure-to-Fix Mapping

This file holds the **audience-specific content** for the test case lifecycle. The generic two-round flow + Confluence page format + `tdx agent test` mechanics live in `../../../shared/test_cases_pattern.md`.

**Round 1 vs Round 2 context:**
- **Round 1** runs against the *schema-derived draft* of `business_context.md` (Phase 4 Step 2 in the parent SKILL — Priority Attributes + PII exclusions auto-populated from `tdx ps desc -o`). The agent knows real columns; failures should skew toward business-specific gaps (terms like "VIP", segment naming, business rules) rather than schema ignorance.
- **Round 2** runs after the customer's filled requirements doc has been merged into `business_context.md` (Phase 5).

## Skills to Load

- `tdx-skills:agent-test` — for `tdx agent test` mechanics, `test.yml` format, output parsing
- `tdx-skills:parent-segment-analysis` — for schema discovery (Phase 4 Step 1)

## Test Case Categories

Generate 10-15 cases total, mixing complexity (simple / medium / complex) across these 5 (or 6) categories.

### 1. Schema Discovery (2-3 cases)
Confirms the agent uses `data_source_finder` and reports data quality. With the Phase 4 schema-derived draft loaded, the agent should *succeed* here in Round 1 (not just attempt).

- "What customer attributes are available?"
- "Show me an overview of the parent segment."
- "What behavior tables exist?"

Pass criteria: mentions `customers` table + at least one `behavior_*` table; reports null ratios for high-null columns; references the Priority Attributes from `business_context.md` when relevant.

### 2. Attribute Queries (2-3 cases)
Single-table queries on the customers table.

- "How many customers do we have by [signup_channel]?"
- "What's the distribution of [priority_attribute]?"

Pass criteria: uses `query_database`, JOINs `customers` as base, returns valid SQL.

### 3. Behavior Aggregations (2-3 cases)
Multi-table queries crossing customers + behavior tables.

- "How many [behavior_event] events per customer in the last 3 months?"
- "Which customers had more than N [behavior] in the last 30 days?"

Pass criteria: uses `TD_INTERVAL` correctly, joins on `cdp_customer_id`, respects 3-month default window.

### 4. Segment Draft Creation (2-3 cases)
Exercises the `:segment:` output.

- "Create a segment of customers who [attribute condition] AND [behavior condition]."
- "Create a segment that combines [existing_segment_name] but excludes [condition]."

Pass criteria: produces valid JSON matching the `:segment:` output schema, acknowledges segment size before drafting, reuses `baseSegmentIds` for named-segment references.

### 5. Ambiguous / Guardrail (2-3 cases)
Confirms the agent asks for clarification or refuses appropriately.

- "Tell me about the data." (should ask which type — attributes/behaviors/segments)
- "Write Python code to analyze this." (should refuse per guardrails)
- "[off-topic question]" (should politely decline)

Pass criteria: ambiguous → calls `request_clarification`; off-topic → short polite rejection.

### 6. SQL Templates *(only if customer provided in §9 of requirements)*
Confirms the agent surfaces customer-supplied SQL patterns.

- "[Question that maps to one of the customer's templates]"

Pass criteria: agent references the template by name, adapts the SQL to the question, doesn't rewrite the template wholesale.

## Example test.yml entries

```yaml
# TC-001
- user_input: "What customer attributes are available?"
  criteria:
    - The response mentions the `customers` table
    - The response lists at least 5 attribute columns
    - The response includes null_ratio information for any columns with >50% nulls

# TC-002
- user_input: "How many customers do we have by signup_channel?"
  criteria:
    - The response contains a SQL query
    - The SQL uses the `customers` table as base
    - The SQL uses GROUP BY on signup_channel
    - The response includes a chart or table output

# TC-004
- user_input: "Create a segment of VIP customers who haven't purchased in 90 days."
  criteria:
    - The agent reports the estimated segment size before drafting
    - The final segment JSON includes a behavioral condition (purchases) and an attribute condition (VIP indicator)
    - The JSON is valid against the `:segment:` output schema
```

Multi-round (Discovery → Execution) test for complex cases:

```yaml
# TC-010
- rounds:
    - user_input: "I want to find high-value customers who haven't purchased recently."
      criteria:
        - The agent announces it will search data sources
        - The agent identifies at least one attribute (value) and one behavior (recency)
    - user_input: "Yes, proceed with those columns."
      criteria:
        - The agent presents an execution plan with exact table/column names
        - The response includes a segment size estimate before drafting
```

## Round 2 — Failure-to-Fix Mapping

When a test case still fails after Round 1 → customer requirements → Round 2:

| Failure pattern | Fix location |
|-----------------|--------------|
| Agent uses wrong column name, misses business-specific term, includes excluded PII | `knowledge_bases/business_context.md` |
| Agent ignores or rewrites a customer SQL template | `knowledge_bases/sql_templates.md` |
| Agent tone is off, skips Discovery Stage step, drafts segments without sizing first | `Custom Audience Agent/prompt.md` |
| Agent calls wrong sub-tool or fails schema lookup | Inspect `Clone Data Source Finder` / `Clone Questions Suggester` prompts (rare) |

After each fix, re-push and re-test:

```bash
tdx agent push -y
tdx agent test
```

Update Round 2 column on the test cases Confluence page (full-page replace via `updateConfluencePage`). Stop when pass rate is acceptable. Document remaining failures as known limitations on the Phase 6 Eval Results page.

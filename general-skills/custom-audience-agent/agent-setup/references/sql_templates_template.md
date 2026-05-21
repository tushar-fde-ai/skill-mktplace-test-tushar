# SQL Templates Template

If the customer provides recurring SQL patterns in §9 of their requirements doc, drop a new KB at `knowledge_bases/sql_templates.md` using the structure below. Same YAML-frontmatter + markdown shape as `business_context.md`.

---

The actual file content starts here (do not include this line or anything above it):

`---`
`name: sql_templates`
`---`

`# SQL Templates`

Customer-supplied SQL patterns the agent should reference when relevant. Each template is short — name, when to use it, and the SQL itself.

`## <template_name_1>`
**Use when:** 1-line description of the question/pattern this answers.

The SQL itself goes inside a fenced code block, e.g.:

    SELECT ...
    FROM customers c
    LEFT JOIN behavior_purchase b ON c.cdp_customer_id = b.cdp_customer_id
    WHERE ...

`## <template_name_2>`
**Use when:** ...

(SQL block as above)

---

## Rules of thumb

- The file is plain markdown with a YAML frontmatter block (`---` `name: sql_templates` `---` at the top). Reproduce that frontmatter exactly when authoring `sql_templates.md`.
- Wrap each SQL example in a triple-backtick `sql` code block — same as you would in any markdown file.
- Keep each template focused on one question pattern. Multiple variations → multiple templates.
- Use the customer's exact column names from `business_context.md` Priority Attributes.
- Don't include `LIMIT` — let the agent decide. Don't hardcode date ranges — use placeholders the agent will substitute.
- The agent uses these as a reference, not literal copies. It may adapt the SQL to the user's specific question.

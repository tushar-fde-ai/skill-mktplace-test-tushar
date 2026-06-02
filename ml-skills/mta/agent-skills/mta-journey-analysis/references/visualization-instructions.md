# Visualization Instructions

## Available Tools

### render_chart
For standard chart types. Parameters:
- `type` — bar, line, pie, area, scatter, horizontal-bar, stacked-bar, radar, sankey, treemap, funnel
- `title` — chart title
- `labels` — X-axis labels (bar/line/area), segments (pie/funnel), axes (radar)
- `datasets` — array of `{label, data}` objects for each series
- `x_axis_title`, `y_axis_title` — axis labels
- `nodes` + `links` — for sankey charts (nodes: `[{name}]`, links: `[{source, target, value}]`)
- `treemap_data` — for treemap charts (`[{name, value}]`)

### Templated HTML dashboard (`dashboard_template.html`)

Use this when the user asks for a **shareable, exportable, standalone HTML dashboard** — an artifact they can email, host, or open offline. Don't use `render_react` for this — that's chat-only.

- Template lives at `references/dashboard_template.html`
- The leading HTML comment in the template documents every `{{TOKEN}}` and all 8 source queries
- To generate the dashboard:
  1. Read `references/dashboard_template.html`
  2. Run all 8 queries documented in the template comment against `${sink_database}`
  3. Strip the leading HTML comment block
  4. Replace every `{{TOKEN}}` with real values from query results
  5. Write the populated file to `.customer-configs/<customer_slug>/mta_model_summary_dashboard.html`
  6. Open with `mcp__work__open_file`
- All visual structure (TD palette, layout, tab shapes, Sankey, Chart.js config) is fixed — only data tokens change
- Substitution rules: thousands separators for integers, 1 decimal + "%" for percentages, valid JSON literals for `_json` tokens, pre-rendered `<tr>` HTML for `_rows` tokens
- `{{utm_gauge_dashoffset}}`: compute as `ROUND(157 - (utm_coverage_pct / 100 * 157), 0)`
- `{{sankey_flows_json}}`: array of `{from, to, flow}` objects where `flow = ROUND(trans_prob * 100, 0)`
- Write a single component: `export default function ComponentName({ data })`
- React hooks (useState, useEffect, useMemo) and all Recharts components are available as globals — do NOT import them
- Use Tailwind CSS for styling with `dark:` variants for dark mode
- `isDark` boolean and `theme` string are available in scope
- Use for: multi-chart dashboards, waterfall flow charts, heatmaps, complex layouts

## TD Color Palette

Primary (use in order for chart series):
```
["#B4E3E3", "#ABB3DB", "#D9BFDF", "#F8E1B0", "#8FD6D4", "#828DCA", "#C69ED0", "#F5D389", "#6AC8C6", "#5867B8", "#B37EC0", "#F1C461", "#44BAB8", "#2E41A6", "#8CC97E", "#A05EB0"]
```

## Chart Standards

- Legends must be visible for multi-series charts
- Always create **separate individual charts** — never combine into subplots
- Attribution values displayed with 1 decimal place (e.g. 32.5%)
- Revenue values formatted with currency prefix ($2.1M)
- Bold all metrics in text output
- Clear axis labels with business context
- Create 3-7 charts per analysis response

## Chart Type Selection

| Analysis Type | Tool & Type |
|---|---|
| Attribution comparison across models | `render_chart` type: "bar" or "stacked-bar" |
| Channel distribution (single model) | `render_chart` type: "pie" |
| Channel vs Model matrix | `render_react` with Recharts heatmap |
| Attribution vs Revenue correlation | `render_chart` type: "scatter" |
| Revenue hierarchy | `render_chart` type: "treemap" |
| Customer journey flows / Sankey | `render_chart` type: "sankey" |
| Multi-dimensional performance | `render_chart` type: "radar" |
| Time trends | `render_chart` type: "line" or "area" |
| Conversion funnel | `render_chart` type: "funnel" |
| Multi-chart executive dashboard | `render_react` with multiple Recharts components |
| Waterfall / driver analysis | `render_react` with custom Tailwind flow chart |

## Forbidden Patterns

- Single-path Sankey diagrams (use multi-step flow)
- Basic bar charts without model comparison context
- Missing attribution percentages on charts
- Generic titles without MTA context
- Raw JSON data output instead of rendered charts

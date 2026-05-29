---
name: fde-rfm-prod-docs
description: |
  Generate production documentation for the RFM Customer Segmentation workflow. Covers runbook creation, customer-facing documentation, and technical handoff documents for ops/CSMs. Trigger when users want to create RFM documentation, RFM runbook, RFM handoff doc, or customer-facing RFM docs.
---

# RFM Production Documentation

This skill generates production-ready documentation for the RFM Customer Segmentation workflow.

## Available Document Types

| Document | Reference | Purpose |
|----------|-----------|---------|
| **Runbook** | `references/runbook.md` | Architecture, output schema, operational procedures, failure modes |
| **Customer Docs** | `references/customer_docs.md` | Customer-facing documentation — what RFM is, how scores work, segment definitions |
| **Technical Handoff** | `references/technical_handoff.md` | Internal handoff for ops/CSMs — configuration details, monitoring, maintenance |

## Workflow

1. Ask the user which document type they need
2. Read the corresponding reference template
3. Gather customer-specific details (database name, tables, segments, etc.)
4. Generate the document
5. Publish to Confluence under the customer's RFM sub-folder (see `../../workflow-setup/references/requirements_doc.md` for folder structure)

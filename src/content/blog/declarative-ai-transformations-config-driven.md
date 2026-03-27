---
title: "Declarative AI Transformations: Building Data Pipelines Agents Can Understand"
description: "How config-driven pipelines and AI transformations let agents autonomously build, execute, and optimize data workflows without writing code."
date: "2026-03-27"
author: Todd Fearn
tags: ["ai-transformations", "config-driven", "datris-platform", "agents"]
draft: true
---

In the old world of data engineering, you wrote ETL jobs. Someone sat down, opened a notebook or IDE, and hand-coded every transformation step. If the schema changed, you fixed the code. If the business logic shifted, you refactored. If you needed a new pipeline, you started from scratch.

That works fine when humans are driving. But we're not anymore — or at least, we shouldn't be.

Today's AI agents need to build, modify, and optimize data pipelines on their own. Not by writing Python or Scala, but by *declaring* what they want to happen in a format they can reason about, version, test, and iterate on. That's where declarative AI transformations come in.

## The Problem With Code-First Pipelines

Traditional ETL/ELT tools force a choice: either keep it simple (SQL) or get powerful (code). Simple tools can't express complex business logic. Powerful tools require humans to read, review, and maintain the code.

For AI agents, this becomes a bottleneck. Here's why:

1. **Agents think in steps, not in code.** They reason about *what* data needs to transform, *why*, and *how to validate it*. They don't naturally think in for-loops and SQL joins.

2. **Auditability is harder.** When an agent modifies a Python pipeline, you have to review the code to understand what changed. With declarations, the diff is transparent.

3. **Composability breaks down.** Agents can't easily combine partial transformations or create variants of a pipeline without rewriting logic.

4. **Version control becomes messy.** Code-based pipelines treat transformations as monolithic units. Declarative pipelines can be diffed, merged, and composed like configuration.

Enter declarative transformations.

## What Are Declarative AI Transformations?

A declarative AI transformation describes *what* a data transformation should do — not *how* to do it. It specifies:

- **Input schema:** What columns and types the transformation expects
- **Output schema:** What the transformation should produce
- **Transformation rules:** The logic applied to each row or batch
- **Validation:** How to detect and handle errors
- **Optimization hints:** Partitioning, caching, parallel execution

The platform — in this case, Datris Platform — interprets the declaration and figures out the most efficient execution strategy.

Here's a simple example:

```yaml
name: "calculate_portfolio_returns"
version: "1.0"
input:
  schema:
    - {name: "portfolio_id", type: "string"}
    - {name: "start_value", type: "decimal"}
    - {name: "end_value", type: "decimal"}
    - {name: "cash_flows", type: "array<decimal>"}
output:
  schema:
    - {name: "portfolio_id", type: "string"}
    - {name: "return_pct", type: "decimal"}
    - {name: "twr", type: "decimal"}  # Time-weighted return
    - {name: "calculation_date", type: "timestamp"}
rules:
  - type: "aiTransform"
    description: "Calculate simple return and time-weighted return"
    model: "claude-3.5-sonnet"
    instruction: |
      Given start_value, end_value, and cash_flows, calculate:
      1. Simple return: (end_value - start_value) / start_value
      2. Time-weighted return using modified Dietz method
      Return as JSON with return_pct and twr fields.
  - type: "validation"
    rule: "return_pct must be between -1 and 10"
    onError: "log"  # or "skip" or "fail"
  - type: "metadata"
    timestamp_field: "calculation_date"
    partition_by: "portfolio_id"
```

An agent can understand this. It can see exactly what's being calculated. It can modify the instruction to the AI model. It can add new rules or swap the validation logic. It can version it, test it, and handoff to another agent downstream.

## How Agents Use This

Here's a real workflow:

**Step 1: Agent discovers data quality issue**
An agent scanning portfolio data notices that return calculations are sometimes missing when cash flows are zero. Instead of filing a ticket or waiting for a human, it:

1. Reads the existing transformation declaration
2. Identifies the gap: "cash_flows array is empty in 2.3% of records"
3. Proposes a modification: Add a rule to handle zero cash flows explicitly
4. Writes a new version of the transformation YAML with the fix
5. Tests it against historical data
6. If validation passes, promotes it to the next environment

**Step 2: Agent optimizes performance**
A monitoring agent notices the transformation is running slow on large portfolios. It:

1. Analyzes the declaration and sees it's not partitioned
2. Adds partition hints based on typical query patterns
3. Adds caching rules for intermediate calculations
4. Reruns benchmark tests
5. If latency improves, increments the version and redeploys

**Step 3: Agent adapts to schema changes**
When a data source upstream changes its schema, a schema-monitoring agent:

1. Detects the mismatch against the transformation's input schema
2. Proposes adapter logic: "Map old field 'nav' to new field 'net_asset_value'"
3. Tests the adapter with sample data
4. Rolls it into the transformation as a preprocessing step

None of this requires human intervention. The agent reasons about the data, makes changes it can justify, tests, and deploys.

## Integration With Datris Platform

On Datris Platform, these transformations live in the **config-driven pipeline layer**. They work seamlessly with:

- **MCP Server:** Agents access transformations via the Datris MCP server's `datris.transformation.create`, `datris.transformation.test`, and `datris.transformation.deploy` tools.
- **Data Quality Rules:** AI transformations can be paired with aiRule-based validation to catch issues in real-time.
- **Multi-Destination Routing:** A transformation can be declared once and routed to multiple destinations (data warehouse, vector DB, real-time sink) based on the output schema.
- **Docker-Native Deployment:** Each transformation spins up as a containerized executor, making it trivial to scale or isolate.

## Why This Matters Now

The shift to AI-first data systems isn't just about using LLMs to query data. It's about letting AI autonomously *operate* the data layer. Declarative transformations are the interface that makes that possible.

In financial services especially, this matters. You can't have agents rewriting your ETL code without audit trails. You *can* have agents modifying transformation declarations — the changes are clear, testable, and versionable.

## Getting Started

If you're building agent-native data infrastructure, start here:

1. **Audit your existing transformations.** What patterns repeat? What could be expressed as rules instead of code?
2. **Export as declarations.** Map your current logic to transformation YAML or JSON.
3. **Integrate with an MCP server.** Give your agents tools to read, propose, and test changes.
4. **Add validation gates.** Not every agent change goes live — some need human review first.

Datris Platform already supports this workflow out of the box. Check out the GitHub repo for examples, templates, and best practices for declarative transformations.

The future of data engineering isn't about writing more ETL code. It's about describing what you want, letting agents optimize it, and staying in control through declarations.

---

**Todd Fearn** is the founder of Datris.ai and has been building AI solutions and data infrastructure for financial services for 25+ years, including at Goldman Sachs, Bridgewater Associates, Deutsche Bank, Freddie Mac and others.

**Explore Datris Platform:** [github.com/datris/datris-platform-oss](https://github.com/datris/datris-platform-oss)

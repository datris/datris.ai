---
title: "Agent-to-Agent Protocols Are Coming. Your Data Layer Isn't Ready."
description: "As A2A, MCP, and ACP mature, AI agents won't just use tools — they'll coordinate with each other. The data infrastructure underneath needs to catch up fast."
date: "2026-03-25"
author: "Todd Fearn"
tags: ["ai-agents", "a2a", "mcp", "data-infrastructure"]
draft: true
---

We spent the last year getting excited about agents using tools. MCP gave us a standard way for an agent to discover a tool, understand its interface, and invoke it. That was a genuine leap forward — I wrote about [what it means for data pipelines](/blog/mcp-server-data-pipelines-ai-agents) last week.

But here's what's coming next, and it's going to be a much bigger deal: agents talking to *each other*.

## The Protocol Landscape Is Shifting

If you've been tracking the space, you've seen the alphabet soup emerging:

- **MCP (Model Context Protocol)** — Anthropic's open protocol for agent-to-tool communication. Think of it as the USB standard for AI tools. It's already seeing real adoption.
- **A2A (Agent-to-Agent)** — Google's protocol for agents to discover, negotiate with, and delegate tasks to other agents. This is peer-to-peer agent coordination.
- **ACP (Agent Communication Protocol)** — Emerging standards for structured agent conversations, session management, and handoffs between agent runtimes.

Each of these solves a different layer of the problem. MCP handles the "agent uses a tool" case. A2A handles "agent delegates to another agent." ACP handles "agents maintain persistent collaborative sessions."

What none of them solve — yet — is the data layer underneath.

## Why Data Is the Bottleneck

Here's a scenario that's about to become very common. You have three agents:

1. **Ingestion Agent** — monitors upstream data sources, pulls new files, handles format detection and parsing
2. **Quality Agent** — validates incoming data against business rules, flags anomalies, routes exceptions
3. **Analytics Agent** — runs downstream transformations, updates dashboards, generates reports

Today, most teams wire these together with message queues, shared databases, and a lot of glue code. The agents are "coordinating" only in the loosest sense — they're reading and writing to shared state, and a human designed the choreography.

With A2A, this changes fundamentally. The Ingestion Agent can *discover* the Quality Agent, negotiate what data format it expects, send it a batch, and get back a validated result — all without a human defining the handoff. The Quality Agent can then discover the Analytics Agent and push clean data downstream.

But here's the catch: every one of these handoffs involves data. Schemas, formats, volumes, lineage, quality metadata. If the data layer can't keep up — if it can't describe itself to agents, adapt to negotiated formats, track provenance across agent-to-agent hops — the whole thing falls apart.

## What an Agent-Ready Data Layer Actually Needs

I've been building data infrastructure for financial services since the late '90s. The patterns for machine-to-machine data exchange are well-established: schemas, contracts, versioning, lineage. What's new is that the *machines negotiating these contracts are now AI agents*, and they need different affordances than humans or traditional software.

Here's what I think the data layer needs to support multi-agent workflows:

### 1. Self-Describing Schemas

Agents can't read a wiki page or ask a colleague what a field means. Your data needs to describe itself — not just types and constraints, but semantic meaning. What does `notional_amount` mean in the context of a credit default swap vs. an interest rate swap? An agent needs that context to make routing and transformation decisions.

This is why Datris Platform generates AI-assisted schema descriptions. When an agent discovers a dataset through MCP, it doesn't just get `{"field": "notional_amount", "type": "number"}` — it gets natural language context about what that field represents and how it relates to other fields.

### 2. Format Negotiation

When Agent A sends data to Agent B, they need to agree on format. CSV? Parquet? JSON? What encoding? What delimiter? In A2A terms, this is part of the capability negotiation phase — but the data platform needs to support it.

Config-driven pipelines help here. If your pipeline definition is declarative — "here's the source, here's the format, here are the transformations" — then an agent can modify that configuration to match what another agent needs. It's not rewriting code; it's adjusting parameters.

```yaml
# An agent could negotiate this configuration with another agent
source:
  type: file
  format: parquet
  path: /data/trades/incoming/

destinations:
  - type: file
    format: json
    path: /data/trades/validated/
    
transforms:
  - aiTransform: "Normalize currency codes to ISO 4217"
  - aiTransform: "Convert timestamps to UTC ISO-8601"
```

### 3. Provenance Across Agent Hops

When data passes through three agents, you need to know what each one did to it. This isn't just an audit requirement (though in financial services, it absolutely is). It's a debugging requirement. When the Analytics Agent produces a weird number, you need to trace back through the Quality Agent's decisions and the Ingestion Agent's parsing to find the root cause.

The data layer needs to capture agent-level lineage: which agent touched this data, what operation did it perform, what was the input, what was the output, and what model/version was driving the decisions.

### 4. Streaming and Batching Flexibility

Some agent-to-agent interactions will be real-time. An anomaly detection agent spots something and immediately alerts a remediation agent. Others will be batch — nightly reconciliation runs that process millions of records.

The data layer can't pick one. It needs to support both patterns, ideally with the same pipeline definition. You shouldn't need two different architectures for "process this file tonight" and "react to this event now."

### 5. Error Semantics That Agents Understand

When a pipeline fails, humans read stack traces and log files. Agents need structured error information — what failed, why, what the likely fix is, and whether it's retryable. This is where AI-powered error explanation becomes critical. Instead of `NullPointerException at line 347`, an agent needs: "The `settlement_date` field was null for 3 records in batch 2024-Q1-0847. This field is required for downstream margin calculations. Likely cause: source system omitted the field for pending trades. Suggested action: filter pending trades or apply a default date."

Datris Platform already does this — `aiErrorExplanation` translates technical failures into actionable descriptions. In a multi-agent world, that's not a nice-to-have. It's how agents recover gracefully instead of failing silently.

## The Coordination Problem Nobody's Talking About

Here's what keeps me up at night about multi-agent data workflows: **conflict resolution**.

What happens when the Quality Agent rejects a batch that the Ingestion Agent already told the Analytics Agent was coming? What happens when two agents try to write to the same destination simultaneously? What happens when an upstream schema change propagates through a chain of five agents, and each one has cached the old schema?

These are distributed systems problems. We solved them (mostly) for microservices over the last decade. Now we need to solve them again for agents — but agents are non-deterministic, which makes coordination harder. You can't just retry with the same input and expect the same output.

I don't think we have great answers yet. But I do think the data layer is where most of these problems will need to be solved. Agents can negotiate protocols and formats, but the data infrastructure underneath needs to provide the primitives: transactions, versioning, conflict detection, rollback.

## What You Can Do Now

If you're building agent-based systems — especially in data-heavy domains like financial services — start thinking about your data layer as a first-class participant in agent coordination:

1. **Make your data self-describing.** Add semantic metadata to your schemas. AI-generated descriptions are a good start.
2. **Use config-driven pipelines.** If agents can modify pipeline configurations without touching code, they can adapt to new requirements autonomously.
3. **Build in lineage from day one.** Retrofitting provenance tracking is painful. Bake it in now.
4. **Design for both streaming and batch.** Your agent workflows will need both, often for the same data.
5. **Make errors machine-readable.** Structured, contextual error information is how agents self-heal.

The [Datris Platform](https://github.com/datris/datris-platform-oss) was designed with these principles. It's open-source, config-driven, and built for a world where AI agents are the primary operators of data infrastructure. If you're navigating this shift, it's worth a look.

The agent-to-agent era is coming fast. Make sure your data can keep up.

---

*Todd Fearn is the founder of Datris.ai and has been building AI solutions and data infrastructure for financial services for 25+ years, including at Goldman Sachs, Bridgewater Associates, Deutsche Bank, Freddie Mac and others.*

---
title: "Your Data Pipeline Has an MCP Server. Now What?"
description: "Datris Platform ships with a built-in MCP server, turning every pipeline into a tool that AI agents can discover, invoke, and orchestrate autonomously."
date: "2026-03-24"
author: "Todd Fearn"
tags: ["mcp", "ai-agents", "datris-platform", "data-pipelines"]
draft: false
---

Here's something that would have sounded absurd five years ago: your data pipeline should be a tool that AI agents can pick up and use.

Not "query via API." Not "trigger via webhook." I mean an agent should be able to *discover* your pipeline exists, understand what it does, figure out how to configure it, run it, and handle the results — all without a human writing glue code.

That's what the Model Context Protocol (MCP) makes possible. And Datris Platform ships with an MCP server built in.

## What MCP Actually Is (No Hype Version)

MCP is an open protocol — originally from Anthropic — that lets AI agents discover and invoke tools through a standardized interface. Think of it as USB for AI: a universal connector that lets any compliant agent talk to any compliant tool.

An MCP server exposes three things:

1. **Tools** — functions the agent can call, with typed parameters and descriptions
2. **Resources** — data the agent can read (schemas, configs, pipeline status)
3. **Prompts** — templated instructions that guide the agent through complex workflows

The key insight is *discovery*. The agent doesn't need to know your API ahead of time. It connects to the MCP server, asks "what can you do?", and gets back a structured catalog of capabilities. That's fundamentally different from hardcoding REST endpoints.

## What Datris Exposes via MCP

When you start Datris Platform with the MCP server enabled, every pipeline in your configuration becomes a set of tools an agent can interact with. Here's what's available out of the box:

**Pipeline Operations**
- `run_pipeline` — Execute a specific pipeline with optional parameter overrides
- `get_pipeline_status` — Check if a pipeline is running, succeeded, or failed
- `list_pipelines` — Discover all configured pipelines and their descriptions

**Schema & Data Discovery**
- `get_schema` — Retrieve the inferred or configured schema for any dataset
- `profile_data` — Run AI-powered data profiling on a source
- `preview_data` — Sample rows from a source or destination

**Configuration**
- `get_pipeline_config` — Read the full YAML configuration for a pipeline
- `validate_config` — Check a config for errors before deploying

**Quality & Monitoring**
- `get_quality_report` — Fetch the latest data quality results
- `get_error_details` — Get AI-generated explanations for pipeline failures

Here's a concrete example. Say you have a pipeline that ingests daily NAV files from a fund administrator:

```yaml
pipelines:
  nav_daily:
    description: "Ingest daily NAV files from fund admin SFTP"
    source:
      type: sftp
      host: sftp.fundadmin.example.com
      path: /outbound/nav/*.csv
    schema:
      infer: true
    quality:
      rules:
        - aiRule: "NAV values must be positive and fund_id must match a known fund in the reference dataset"
    destinations:
      - type: postgres
        table: nav_daily
```

An agent connecting to the MCP server can:

1. Call `list_pipelines` and learn this pipeline exists
2. Call `get_schema` to understand the data shape
3. Call `run_pipeline` with `{ "pipeline": "nav_daily" }` to trigger ingestion
4. Call `get_quality_report` to check if the AI validation rules passed
5. Call `get_error_details` if something failed — and get a plain-English explanation

No SDK. No custom integration. The agent just... uses it.

## Why This Changes the Game for Data Operations

I've been building data infrastructure in financial services since the late '90s. The operational tax on pipelines — monitoring, triage, reprocessing, schema changes — has always been the hidden cost. You build the pipeline in a week and spend the next two years babysitting it.

MCP changes the economics. Here's what I've seen work in practice:

**Autonomous Monitoring:** An agent polls pipeline status on a schedule. When something fails, it reads the error details, determines if it's a transient issue (network timeout, file not yet available) or a structural problem (schema drift, bad data), and takes action — either retrying automatically or escalating with full context.

**Self-Service Data Access:** A downstream analyst agent needs data for a report. Instead of filing a ticket and waiting for an engineer to grant access and explain the schema, it discovers the pipeline via MCP, profiles the data, reads the schema, and starts working. The data platform becomes self-documenting through the protocol.

**Cross-Pipeline Orchestration:** A coordinator agent manages dependencies between pipelines — "run the reference data pipeline first, then the trades pipeline, then quality checks, then the report." Each step is a tool call through MCP. The agent handles sequencing, error handling, and retries.

## Running the MCP Server

Datris ships as a Docker image. Enabling the MCP server is one config flag:

```yaml
server:
  mcp:
    enabled: true
    port: 3001
    auth:
      type: bearer
      token: ${MCP_AUTH_TOKEN}
```

Then point your agent at it. If you're using Claude, that's an entry in your MCP config:

```json
{
  "mcpServers": {
    "datris": {
      "url": "http://localhost:3001/mcp",
      "headers": {
        "Authorization": "Bearer ${MCP_AUTH_TOKEN}"
      }
    }
  }
}
```

That's it. Your agent now has access to every pipeline, every schema, every quality report in your Datris deployment.

## The Bigger Picture

MCP is still early. The spec is evolving, the ecosystem is growing, and we're all figuring out the patterns. But the direction is clear: data infrastructure needs to be agent-accessible, not just human-accessible.

The alternative — building custom integrations for every agent framework, every LLM provider, every orchestration layer — doesn't scale. We tried that with REST APIs and ended up with a mess of client libraries and versioning headaches. MCP gives us a standard, and Datris is built on it from day one.

This isn't about replacing engineers. It's about giving agents the same operational access that engineers have, so the humans can focus on architecture and strategy instead of monitoring dashboards and reprocessing failed jobs at 3 AM.

If you've been curious about what agent-native data infrastructure actually looks like in practice, [check out the repo](https://github.com/datris/datris-platform-oss) or visit [datris.ai](https://datris.ai).

---

*Todd Fearn is the founder of Datris.ai and has been building AI solutions and data infrastructure for financial services for 25+ years, including at Goldman Sachs, Bridgewater Associates, Deutsche Bank, Freddie Mac and others.*

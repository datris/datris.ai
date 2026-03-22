---
title: "Why Agent-Native Data Platforms Matter"
description: "AI agents are becoming autonomous operators of data infrastructure. But most data platforms weren't built for them. Here's why that needs to change — and what agent-native actually means."
date: "2026-03-22"
author: "Todd Fearn"
tags: ["ai-agents", "mcp", "data-platform"]
draft: true
---

The term "AI-native" gets thrown around a lot. Usually it means "we added an LLM endpoint." That's not what we mean by agent-native.

An agent-native data platform is one where AI agents are **first-class operators** — not bolted-on consumers of a human-designed API, but autonomous participants who can discover, configure, ingest, validate, transform, and query data through the same interfaces humans use.

## The Problem with Bolt-On AI

Most data platforms today treat AI as a feature. You get a chatbot that can answer questions about your pipeline status, or an LLM that generates SQL. That's useful, but it's fundamentally limited — the AI is a passenger, not a driver.

When an AI agent needs to:

- Discover what datasets exist and their schemas
- Create a new ingestion pipeline from scratch
- Set up validation rules in plain English
- Route data to multiple destinations
- Diagnose why a pipeline failed at 3 AM

...it shouldn't need a human to write glue code, configure webhooks, or build a custom integration. It should just talk to the platform.

## What Agent-Native Looks Like

At Datris, we built [Model Context Protocol (MCP)](https://modelcontextprotocol.io/) support directly into the platform. MCP is becoming the standard for how AI agents interact with tools and data sources — think of it as USB-C for AI.

Through MCP, an agent can:

- **List all datasets** and inspect their configurations
- **Create new datasets** with AI-generated schemas
- **Upload and ingest files** — CSV, JSON, XML, Excel, PDFs
- **Set validation rules in plain English** — "reject rows where email format is invalid"
- **Run AI-powered data profiling** — get summary stats, quality issues, and suggested rules
- **Query processing history** and diagnose errors
- **Route data** to PostgreSQL, MongoDB, vector databases, message queues — all configured through conversation

No integration code. No middleware. The agent talks to the platform the same way a developer would use the API — but through natural language.

## Why This Matters Now

Three trends are converging:

1. **Agents are getting autonomous.** Claude, GPT, and open-source models can now plan multi-step workflows, use tools, and recover from errors. They're ready to operate infrastructure.

2. **Data pipelines are getting more complex.** The average enterprise runs dozens of ingestion jobs across multiple sources, formats, and destinations. Managing this manually doesn't scale.

3. **MCP is creating a standard.** Before MCP, every AI integration was bespoke. Now there's a protocol that lets any agent talk to any tool. Platforms that support it become instantly accessible to the entire AI ecosystem.

## The First, Not the Last

We believe Datris is the first open-source data platform with native MCP support. We won't be the last — and that's the point. The industry is moving toward agent-operated infrastructure. The platforms that embrace this early will define the category.

If you're building data pipelines and thinking about how AI agents fit into your architecture, [check out the repo](https://github.com/datris/datris-platform-oss) or [reach out](https://datris.ai). We'd love to compare notes.

---

*Todd Fearn is the founder of Datris.ai and has been building data infrastructure for financial services for 25+ years, including at Goldman Sachs, Bridgewater Associates, Deutsche Bank, and Freddie Mac.*

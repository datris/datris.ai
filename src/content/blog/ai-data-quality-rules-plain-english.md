---
title: "Write Data Quality Rules in Plain English — and Actually Mean It"
description: "Datris lets you define data validation rules in natural language. No regex, no custom code — just describe what valid data looks like and let the AI enforce it."
date: "2026-03-23"
author: "Todd Fearn"
tags: ["data-quality", "ai-rules", "datris-platform"]
draft: false
---

I've spent 25 years writing data validation code. Regex patterns for email formats. Boundary checks for price fields. Custom functions to verify cross-column relationships. It's tedious, error-prone, and — here's the thing nobody says out loud — it's often harder to write the validation than it is to write the pipeline itself.

What if you could just *say* what valid data looks like?

## AI Rules: Validation as Conversation

Datris Platform lets you define data quality rules in plain English using `aiRule`. Not a DSL. Not a simplified scripting language. Actual sentences that the AI model evaluates against your data.

Here's a real-world scenario. You're ingesting a multi-asset trade blotter from a prime broker — equities, options, futures, and FX all mixed together. Each asset class has different validation logic:

```json
{
  "function": "ai",
  "parameters": [
    "Validate each trade row based on its asset class: For equities, the trade price must be between the day's high and low, and the settlement date must be T+1 for US markets or T+2 for non-US. For listed options, verify the strike price is a standard increment for the underlying's price range, the expiry falls on a valid monthly or weekly expiration date, and the contract multiplier is 100 for US equity options. For FX forwards, the forward points must be consistent with the spot rate and tenor — a higher-yielding currency should trade at a forward discount. Flag any row where the notional exceeds $50M for a single trade without a block trade indicator.",
    "100"
  ],
  "onFailureIsError": true
}
```

That's a single rule covering four asset classes, settlement conventions across jurisdictions, options expiration calendars, FX interest rate parity, and risk limits. Writing that in code means maintaining holiday calendars for every market, options expiration lookup tables, an interest rate differential model, and a pile of conditional branching. The AI model already understands all of it.

When a row fails, you don't get "validation error on row 47." You get: *"Row 47: EUR/USD 3-month forward shows 45 forward points with spot at 1.0850, but with ECB rates above Fed rates the euro should trade at a forward premium (positive points), not a discount. Likely a sign error in the forward points field."*

That's not just validation — it's diagnosis.

## AI Schema Generation: Upload a File, Get a Pipeline

Before you validate data, you need to define its structure. Traditionally that means manually writing a schema — field names, types, formats, constraints. For a 50-column trade file, that's an hour of mind-numbing work.

Datris flips this. Upload a sample file and the AI analyzes it:

```bash
curl -X POST http://localhost:8080/api/generate-schema \
  -F "file=@prime_broker_blotter.csv"
```

The AI examines your data and returns a complete dataset configuration — field types inferred from actual values (not just "string" for everything), date formats detected, enum fields identified, and even suggested validation rules based on what it sees in the data. A column with values like `BUY`, `SELL`, `SHORT`, `COVER`? It generates an enum constraint. A column that looks like CUSIP identifiers? It sets the format and length validation.

You get a production-ready config in seconds instead of writing one from scratch.

## AI Transformations: Reshape Data in Plain English

Once data passes validation, you often need to transform it. Datris AI transformations let you describe the change you want:

```json
{
  "transformations": [
    {
      "type": "ai",
      "instruction": "Normalize all currency amounts to USD using the fx_rate column. For rows where the trade currency is already USD, keep the original amount. Add a new column 'amount_usd' with the converted value and a column 'conversion_applied' set to true or false."
    }
  ]
}
```

The AI generates and applies the transformation logic row by row. This is particularly powerful for messy data — inconsistent date formats across sources, free-text fields that need categorization, or columns that encode multiple values (like "NYC-EQ-ALGO" that needs to be split into location, asset class, and execution type).

## AI Data Profiling: Understand What You're Working With

Before writing any rules at all, you might not fully understand the data. Datris can profile an entire dataset using AI:

```bash
curl -X POST http://localhost:8080/api/profile \
  -F "file=@mystery_data.csv"
```

The AI analyzes distributions, detects anomalies, identifies relationships between columns, and surfaces quality issues you didn't know to look for. It might tell you:

- *"The 'price' column has a bimodal distribution suggesting two different instruments are mixed together"*
- *"Column 'trade_id' has 3 duplicate values — rows 1204, 1205, and 1206 appear to be the same trade reported three times"*
- *"The 'timestamp' column switches from UTC to EST after row 8,000, likely a source system timezone change"*

These are the kinds of issues that cause silent data corruption — they pass traditional validation but poison downstream analytics. The AI catches them because it looks at the data holistically, not field by field.

## AI Error Explanation: Root Cause in Plain English

When a pipeline fails — and pipelines always fail eventually — Datris automatically generates an AI-powered explanation of what went wrong:

Instead of a stack trace and "ArrayIndexOutOfBoundsException at line 847," you get:

*"The pipeline failed during the transformation step. The input file contains 47 columns but the dataset configuration expects 45. The two extra columns appear to be 'regulatory_flag' and 'mifid_category', likely added by the upstream system after a regulatory update. Suggested fix: add these columns to the dataset schema or configure the pipeline to ignore unexpected columns."*

No log spelunking. No guessing. The AI reads the error, examines the data and configuration, and tells you what happened and what to do about it.

## MCP Server: Let AI Agents Run the Pipeline

All of these AI features are available through Datris's built-in MCP (Model Context Protocol) server. This means AI agents can autonomously:

- **Generate schemas** from uploaded files
- **Create datasets** with AI validation rules
- **Profile data** to discover quality issues
- **Upload files** and trigger pipeline runs
- **Monitor jobs** and diagnose failures
- **Query results** across PostgreSQL, MongoDB, and five vector databases

An agent can receive a new data file, profile it to understand its structure, generate a schema, create a pipeline with appropriate AI validation rules, ingest the data, and report back on quality issues — all without human intervention.

```python
# An AI agent's workflow via MCP
schema = mcp.call("generate_schema", file="new_feed.csv")
mcp.call("create_dataset", config=schema)
mcp.call("upload_file", dataset="new_feed", file="new_feed.csv")
status = mcp.call("get_job_status", dataset="new_feed")
# Agent autonomously handles the entire pipeline lifecycle
```

This is what we mean by "agent-native." The platform wasn't designed for humans and then adapted for agents — the MCP interface is a first-class citizen.

## Running It All Locally

Every AI feature in Datris works with Ollama, meaning you can run the entire platform — including AI validation, transformations, profiling, and error explanation — on your own hardware with zero API costs and no data leaving your network.

```bash
git clone https://github.com/datris/datris-platform-oss.git
cd datris-platform-oss
docker-compose up --build
```

Point the AI provider at your local Ollama instance, and you've got an air-gapped, AI-powered data platform. For production workloads, swap in Anthropic Claude or OpenAI for higher accuracy on complex rules — the platform handles the provider abstraction.

We're building this in the open at [datris.ai](https://datris.ai). The full source is at [GitHub](https://github.com/datris/datris-platform-oss). Feedback, issues, and PRs are welcome.

---

*Todd Fearn is the founder of Datris.ai and has been building AI solutions and data infrastructure for financial services for 25+ years, including at Goldman Sachs, Bridgewater Associates, Deutsche Bank, Freddie Mac and others.*

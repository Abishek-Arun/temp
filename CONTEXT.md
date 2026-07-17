# CONTEXT.md

# FinGraph AI
### Multi-Agent Human-in-the-Loop Financial Research & Investment Committee Platform

---

# Project Overview

FinGraph AI is a production-grade multi-agent financial research platform built using:

- FastAPI (Backend API Layer)
- Chainlit (Frontend UI)
- LangGraph (Agent Orchestration)
- LangChain (LLM Integration)
- PostgreSQL (Persistence)
- Redis (Caching & Session Storage)
- Qdrant (Vector Database)
- Docker / Docker Compose (Deployment)

The platform simulates a real-world investment research committee where multiple AI analyst agents collaborate to evaluate stocks, generate research, debate investment decisions, and involve humans at key approval checkpoints before proceeding.

This is not a chatbot.

This is a workflow-driven, human-governed AI system.

---

# Primary Goal

A user submits an investment research request:

Example:

"Analyze NVIDIA as a long-term investment for the next 3 years."

The system:

1. Creates a research plan
2. Requests human approval
3. Runs multiple specialist agents
4. Conducts bull vs bear analysis
5. Builds a committee recommendation
6. Requests human approval
7. Generates a final research report
8. Requests final sign-off
9. Saves research history

---

# Core Business Objective

Demonstrate:

- Multi-Agent Architecture
- LangGraph Workflow Orchestration
- Human-in-the-Loop Design
- Agent Parallelization
- Graph Checkpointing
- Graph Resume Capability
- Tool Calling
- Financial Data Analysis
- RAG
- Production Deployment

---

# User Journey

User submits:

"Analyze Microsoft stock."

↓

Research Manager Agent creates a research plan.

↓

HUMAN APPROVAL CHECKPOINT #1

Approve / Modify / Reject

↓

If approved:

Financial Agent
News Agent
Risk Agent
Market Sentiment Agent

run in parallel.

↓

Results are aggregated.

↓

Bull Analyst Agent

↓

Bear Analyst Agent

↓

Investment Committee Agent

↓

Draft Recommendation Generated.

↓

HUMAN APPROVAL CHECKPOINT #2

Proceed
Request More Research
Reject

↓

If approved:

Report Generator Agent

↓

Final PDF / Markdown Report

↓

HUMAN APPROVAL CHECKPOINT #3

Publish Report

↓

Research Complete

---

# Human-In-The-Loop Requirements

Human involvement is a mandatory part of the workflow.

The graph must pause using LangGraph interrupts.

---

## Checkpoint 1: Research Plan Approval

Purpose:

Validate research scope before spending resources.

Example:

Research Plan:

- Financial Performance
- Competitive Analysis
- Market Sentiment
- Risk Assessment

User Actions:

- Approve
- Modify
- Reject

Graph pauses until user responds.

---

## Checkpoint 2: Recommendation Approval

Purpose:

Validate AI-generated investment recommendation.

Example:

Recommendation:

BUY

Confidence:

82%

User Actions:

- Approve Recommendation
- Request More Analysis
- Reject Findings

If additional analysis requested:

Graph routes back into additional research nodes.

---

## Checkpoint 3: Report Sign Off

Purpose:

Approve final research publication.

User Actions:

- Generate Final Report
- Regenerate Sections
- Reject

---

# Multi-Agent System

---

## 1. Research Manager Agent

Role:

Graph orchestrator.

Responsibilities:

- Understand user query
- Build research strategy
- Define required investigations
- Coordinate agent execution
- Consolidate outputs

Output:

Structured Research Plan

---

## 2. Financial Analyst Agent

Role:

Fundamental analysis.

Responsibilities:

- Revenue Growth Analysis
- Profitability Analysis
- Cash Flow Analysis
- Valuation Metrics
- Debt Evaluation

Tools:

- yFinance
- SEC Filings
- Company Financial Statements

Output:

Structured financial assessment.

---

## 3. News Intelligence Agent

Role:

Recent event analysis.

Responsibilities:

- Earnings Reports
- Mergers & Acquisitions
- Product Releases
- Regulatory Issues
- Industry News

Tools:

- News APIs
- Search APIs

Output:

Recent events summary.

---

## 4. Risk Assessment Agent

Role:

Identify investment risks.

Responsibilities:

- Market Risks
- Regulatory Risks
- Competition Risks
- Economic Risks
- Company-Specific Risks

Output:

Risk Report
Risk Score

---

## 5. Market Sentiment Agent

Role:

Analyze market perception.

Responsibilities:

- Industry Trend Analysis
- Analyst Opinions
- Market Momentum
- Investor Sentiment

Output:

Sentiment Score

Bullish / Neutral / Bearish

---

## 6. Bull Analyst Agent

Role:

Argue the investment case.

Responsibilities:

Produce strongest argument for buying.

Output:

Bull Thesis

Example:

- AI growth opportunities
- Strong cash flow
- Industry leadership

---

## 7. Bear Analyst Agent

Role:

Challenge assumptions.

Responsibilities:

Produce strongest argument against investment.

Output:

Bear Thesis

Example:

- Overvaluation
- Competitive threats
- Regulatory risks

---

## 8. Investment Committee Agent

Role:

Final decision-maker.

Inputs:

- Financial Analysis
- News Analysis
- Risk Analysis
- Market Sentiment
- Bull Thesis
- Bear Thesis

Output:

BUY
HOLD
SELL

Confidence Score

Supporting Rationale

---

## 9. Report Generator Agent

Role:

Generate final report.

Formats:

- Markdown
- PDF

Sections:

- Executive Summary
- Financial Analysis
- Market Analysis
- Risk Analysis
- Bull Case
- Bear Case
- Final Recommendation

---

# LangGraph Workflow

```text
START
   |
   v
Research Manager
   |
   v

[ HUMAN APPROVAL #1 ]

   |
   v

+------------------------------------------------+
|                    PARALLEL                    |
|                                                 |
v                                                 v

Financial Agent                              News Agent

Risk Agent                                   Sentiment Agent

+------------------------------------------------+
                     |
                     v

               Bull Agent

                     |
                     v

               Bear Agent

                     |
                     v

         Investment Committee Agent

                     |
                     v

[ HUMAN APPROVAL #2 ]

                     |
        +------------+------------+
        |                         |
        v                         v

Generate Report       Additional Research

        |
        v

Report Generator

        |
        v

[ HUMAN APPROVAL #3 ]

        |
        v

END
```

---

# LangGraph State Design

Suggested State Object

```python
FinancialResearchState
```

Contains:

- user_query
- company_name
- research_plan
- approval_status
- financial_analysis
- news_analysis
- risk_analysis
- sentiment_analysis
- bull_thesis
- bear_thesis
- recommendation
- report
- feedback
- current_stage

---

# RAG System

Purpose:

Allow agents to reason using company-specific documents.

Supported Documents:

- Annual Reports
- Quarterly Reports
- Investor Presentations
- Earnings Call Transcripts
- SEC Filings

Pipeline:

PDF Upload

↓

Chunking

↓

Embeddings

↓

Qdrant

↓

Retriever

↓

Agent Context

---

# Backend Architecture

```text
backend/

├── app/
│
├── api/
│   ├── research.py
│   ├── approvals.py
│   ├── reports.py
│
├── graph/
│   ├── workflow.py
│   ├── state.py
│
├── agents/
│   ├── manager.py
│   ├── financial.py
│   ├── news.py
│   ├── risk.py
│   ├── sentiment.py
│   ├── bull.py
│   ├── bear.py
│   ├── committee.py
│   ├── report.py
│
├── tools/
│   ├── finance_tools.py
│   ├── search_tools.py
│   ├── rag_tools.py
│
├── services/
│   ├── postgres.py
│   ├── redis.py
│   ├── qdrant.py
│
├── models/
│
└── main.py
```

---

# FastAPI Responsibilities

Responsibilities:

- Trigger graph execution
- Resume graph execution
- Persist checkpoints
- Store sessions
- Generate reports
- Serve APIs to Chainlit

Example Endpoints:

```text
POST /research/start

POST /research/approve

POST /research/reject

POST /research/resume

GET /research/status/{id}

GET /research/report/{id}
```

---

# Chainlit Responsibilities

Purpose:

Human interaction layer.

Features:

- Chat Interface
- Approval Forms
- Research Dashboard
- Agent Progress Tracking
- Research History
- Report Downloads

Right Sidebar Example:

```text
✅ Research Manager

✅ Financial Agent

✅ News Agent

🔄 Committee Agent

⏸ Waiting For Approval
```

---

# Persistence

Database:

PostgreSQL

Tables:

sessions

research_runs

approvals

reports

user_feedback

agent_logs

---

# Cache

Redis

Usage:

- Session State
- Agent Responses
- API Result Caching
- Temporary Checkpoint Storage

---

# Vector Database

Qdrant

Stores:

- Financial PDFs
- Filings
- Research Documents
- Earnings Transcripts

---

# Monitoring

LangSmith

Track:

- Agent executions
- State transitions
- Interrupt events
- Token usage
- Latency
- Failures

---

# Docker Architecture

```text
docker-compose

├── chainlit
├── fastapi
├── postgres
├── redis
├── qdrant
```

Future:

Kubernetes Deployment

---

# Future Enhancements

## Version 2

Portfolio Analysis

Example:

Analyze:

- AAPL
- MSFT
- NVDA
- AMD

Generate:

- Risk Distribution
- Diversification Score
- Portfolio Recommendation

---

## Version 3

Investment Watchlists

Daily monitoring of selected companies.

---

## Version 4

Multi-Company Comparison

Example:

Compare:

- Nvidia
- AMD
- Intel

Generate comparative investment report.

---

## Version 5

Investment Committee Debate UI

Display live debate between:

- Bull Agent
- Bear Agent

before final recommendation.

---

# Expected Outcome

FinGraph AI should feel like an internal AI research platform used by a professional investment firm.

The project must showcase:

- LangGraph Multi-Agent Workflows
- Human-in-the-Loop Architecture
- FastAPI Backend Engineering
- Chainlit Frontend Design
- Dockerized Deployment
- RAG Integration
- Production Readiness
- Resume-Level System Design
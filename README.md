# OmniDesk AI: Enterprise Autonomous IT Copilot & Agentic RAG System

[![Python Version](https://img.shields.io/badge/Python-3.11%2B-blue.svg)](https://www.python.org/)
[![Framework](https://img.shields.io/badge/Framework-FastAPI-009688.svg)](https://fastapi.tiangolo.com/)
[![Orchestration](https://img.shields.io/badge/Agent_Orchestration-LangGraph-FF6F00.svg)](https://langchain-ai.github.io/langgraph/)
[![Vector DB](https://img.shields.io/badge/Vector_DB-Chroma_DB-FF6600.svg)](https://www.trychroma.com/)
[![Relational DB](https://img.shields.io/badge/Relational_DB-SQLite_%2F_SQL-003B57.svg)](https://www.sqlite.org/)
[![LLM & Embeddings](https://img.shields.io/badge/LLM-OpenAI_GPT--4o--mini-412991.svg)](https://openai.com/)
[![Docker](https://img.shields.io/badge/Deployment-Docker-2496ED.svg)](https://www.docker.com/)
[![License](https://img.shields.io/badge/License-Apache_2.0-green.svg)](LICENSE)

**OmniDesk AI** is a production-grade, autonomous IT Support and Knowledge Operations Copilot designed for modern enterprises. Built with a **stateful Agentic RAG architecture powered by LangGraph, FastAPI, Chroma DB, SQL databases, and OpenAI**, OmniDesk AI shifts traditional static retrieval systems into an active decision-making assistant. 

Deployed for enterprise IT and platform operations, OmniDesk dynamically routes user intents across **structured relational SQL databases** (live asset inventories, pending tickets, employee records) and **unstructured Chroma DB vector documents** (SOPs, hardware refresh policies, compliance runbooks). It actively prevents LLM hallucinations through self-reflective grading, while solving real-world enterprise pain points like **redundant duplicate ticket proliferation** and SLA bottlenecks.

---

## 📑 Table of Contents
- [Business Problem & Operational Context](#-business-problem--operational-context)
- [System Architecture & Agentic Workflow](#-system-architecture--agentic-workflow)
- [Dual-Engine Hybrid Routing (SQL + Vector)](#-dual-engine-hybrid-routing-sql--vector)
- [Key Engineering Differentiators](#-key-engineering-differentiators)
- [Technology Stack](#-technology-stack)
- [Repository Structure](#-repository-structure)
- [Getting Started](#-getting-started)
  - [Prerequisites](#prerequisites)
  - [Local Installation](#local-installation)
  - [Environment Configuration](#environment-configuration)
  - [Knowledge Base & Database Setup](#knowledge-base--database-setup)
  - [Running the Application](#running-the-application)
- [Docker Deployment](#-docker-deployment)
- [REST API Reference](#-rest-api-reference)
- [Enterprise Verification Scenarios](#-enterprise-verification-scenarios)
- [Security & Production Readiness](#-security--production-readiness)
- [Author & Engineering Portfolio](#-author--engineering-portfolio)

---

## 🏢 Business Problem & Operational Context

### Enterprise Context
In fast-scaling engineering and technology organizations (400+ employees across Engineering, AI & Data, Product, Sales, and Ops), teams deliver mission-critical platforms and customer-facing high-availability systems.

### Operational Bottlenecks & The "Duplicate Ticket" Drain
Internal IT and platform support teams manage sprawling, multi-format knowledge bases and transactional helpdesk queues. Traditional support setups suffered from:
1. **Redundant Duplicate Ticket Bombing ($15–$25 triage cost per ticket):** When employees experience device wear or await approvals, they frequently create duplicate hardware requests without realizing automated warranty systems or managers already submitted an active ticket in the queue.
2. **Disconnected Structured vs. Unstructured Silos:** Standard chatbots only read static PDFs (handbooks) or execute canned database queries—failing completely when a query requires cross-referencing live employee asset dates against corporate replacement thresholds.
3. **Naive RAG Hallucinations:** Traditional semantic search blindly injects irrelevant chunks into prompts, leading models to invent plausible-sounding yet erroneous troubleshooting steps.
4. **Data Staleness & Lack of External Telemetry:** When third-party vendors (e.g., cloud platforms, Microsoft Teams, auth providers) suffer service disruptions, internal KBs have no answers, causing naive systems to fail silently.
5. **Zero Observability:** Standard chatbots act as black boxes with no auditable trace of retrieval decisions, confidence grading, or data lineage.

### OmniDesk AI Solution
OmniDesk AI delivers an **adaptive, self-evaluating copilot** that:
- Seamlessly routes and cross-validates between **structured SQL tables** and **unstructured Chroma DB knowledge vectors**.
- Acts as an operational gatekeeper that **detects and deflects duplicate tickets** before they pollute ServiceNow/Jira queues.
- Prioritizes private enterprise documents with strict data sovereignty.
- Autonomously grades retrieved evidence before synthesizing an answer.
- Routes to real-time web intelligence only when private documentation is genuinely deficient.
- Rewrites ambiguous questions and re-queries vector indexes under a loop-safe budget.
- Returns comprehensive execution traces for compliance, debugging, and end-user transparency.

---

## 🧠 System Architecture & Agentic Workflow

OmniDesk AI executes an **agentic state machine** in LangGraph with multi-path routing, cyclic self-correction, and tool cross-validation:

```
                                  ┌───────────────────────────┐
                                  │   Incoming User Query     │
                                  └─────────────┬─────────────┘
                                                │
                                                ▼
                                 ┌─────────────────────────────┐
                                 │      [1] Intent Router      │
                                 └──────┬───────┬───────┬──────┘
             Casual Chat                │       │       │       Complex Asset/Policy Inquiry
   ┌────────────────────────────────────┘       │       │       (e.g., Eligibility & Tickets)
   │                       SQL Metric/State     │       │       
   │                      ┌─────────────────────┘       │       
   │                      │                             ▼       
   │                      ▼             ┌─────────────────────────────┐
   │        ┌─────────────────────────┐ │  [2] Private KB Retrieval   │
   │        │ [SQL Generator & Exec]  │ │      (Chroma DB Top-K)      │
   │        │ - Query Device & Ticket │ └──────────────┬──────────────┘
   │        │   Tables (Read-Only)    │                │
   │        └─────────────┬───────────┘                ▼
   │                      │             ┌─────────────────────────────┐
   │                      │             │  [3] Evidence Self-Grader   │
   │                      │             └──────┬───────────────┬──────┘
   │                      │                    │               │
   │                      │          Evidence HIGH           Evidence WEAK
   │                      │                    │               │
   │                      │                    │               ▼
   │                      │                    │  ┌───────────────────────────┐
   │                      │                    │  │ [4] Tavily Web Fallback   │
   │                      │                    │  └────────────┬──────────────┘
   │                      │                    │               │
   │                      │                    │               ▼
   │                      │                    │  ┌───────────────────────────┐
   │                      │                    │  │  [5] Web Evidence Grader  │
   │                      │                    │  └────┬─────────────────┬────┘
   │                      │                    │       │                 │
   │                      │                    │   Evidence HIGH     Evidence WEAK
   │                      │                    │       │                 │
   │                      │                    │       │                 ▼
   │                      │                    │       │  ┌───────────────────────────┐
   │                      │                    │       │  │    [6] Query Rewriter     │
   │                      │                    │       │  │ (Reformulate & Retry KB)  │
   │                      │                    │       │  └──────────────┬────────────┘
   │                      │                    │       │                 │ (Max Retry Guard)
   │                      ▼                    ▼       ▼                 ▼
   │        ┌─────────────────────────────────────────────────────────────────────────┐
   │        │      [7] Cross-Validation & Duplicate Ticket Resolution Node            │
   │        │  - Correlates SQL hardware age with Chroma DB policy eligibility rules  │
   │        │  - Detects active pending tickets to deflect duplicate submissions     │
   │        └────────────────────────────────────┬────────────────────────────────────┘
   ▼                                             ▼
┌──────────────────────┐        ┌──────────────────────────────────────────────┐
│ Direct Answer Engine │        │ Grounded Synthesis & Source Attribution Node │
└──────────┬───────────┘        └──────────────────────┬───────────────────────┘
           │                                           │
           └───────────────────┬───────────────────────┘
                               ▼
            ┌──────────────────────────────────────┐
            │   JSON Response + Step-by-Step Trace │
            │      Audit Persisted to SQLite       │
            └──────────────────────────────────────┘
```

---

## 🔀 Dual-Engine Hybrid Routing (SQL + Vector)

A major differentiator in OmniDesk AI is its capacity to combine **tabular facts** with **semantic text**:

```text
                                  User: "Can I upgrade my laptop, and do I have an active ticket?"
                                                                  │
                                 ┌────────────────────────────────┴────────────────────────────────┐
                                 ▼                                                                 ▼
                  [Structured SQL Pipeline]                                       [Unstructured Vector Pipeline]
              Table: assigned_devices, support_tickets                            Collection: compny_data_kb
                                 │                                                                 │
                                 ▼                                                                 ▼
                     Extracted Database Facts                                          Extracted Policy Rules
       - Assigned: Lenovo X1 Carbon (3.4 yrs ago)                        - Threshold: Laptops eligible after 3 years
       - Open Ticket: #TK-3091 (Status: Pending Approval)                - Rule: Engineering qualifies for ThinkPad/M3
                                 │                                                                 │
                                 └────────────────────────────────┬────────────────────────────────┘
                                                                  ▼
                                            [Cross-Validation & Decision Engine]
                                - 3.4 yrs > 3.0 yrs -> Hardware Refresh ELIGIBLE
                                - Active Ticket #TK-3091 DETECTED -> DUPLICATE DEFLECTION
                                                                  ▼
                                           "Eligible for upgrade (3.4 yrs in service), 
                                            but Ticket #TK-3091 is already pending approval. 
                                            No new ticket required."
```

---

## ⚡ Key Engineering Differentiators

| Capability | Static / Naive RAG | OmniDesk AI (Agentic Hybrid Copilot) |
|---|---|---|
| **Data Scope** | Unstructured text documents only | Joint-reasoning across relational SQL tables + Chroma DB documents |
| **Duplicate Prevention** | None; blindly advises user to open tickets | Cross-checks open service desk tickets to deflect redundant requests |
| **Data Routing** | Indiscriminately searches vector DB | Intent-based router selects between SQL, Chroma DB, Hybrid, or Direct |
| **Hallucination Prevention** | None; injects top-k blindly into prompt | Dual-stage LLM-as-a-Judge grading filters irrelevant context |
| **Out-of-Distribution Handling** | Hallucinates or outputs generic failure | Intelligently falls back to Tavily real-time web search |
| **Ambiguous Queries** | Fails silently on poor keywords | Autonomous Query Rewriter reframes query for higher recall |
| **Infinite Loop Guard** | N/A | Deterministic state transition guards prevent cyclic retries |
| **Observability & Audit** | Black-box output | Full execution graph trace returned in response and stored in SQLite |
| **Dynamic Ingestion** | Static notebook / offline pipeline | Live multipart upload endpoint (`.pdf`, `.docx`, `.md`, `.txt`) |

---

## 🛠 Technology Stack

- **Agent Orchestration:** `LangGraph` (StateGraph, conditional edges, structured outputs)
- **Backend API:** `FastAPI` (Asynchronous endpoints, Pydantic data contracts)
- **Server:** `Uvicorn` (ASGI production runner)
- **Vector Database:** `Chroma DB` (Persistent vector store & embeddings index)
- **Structured Database:** `SQLite / SQL` (IT asset inventory, ticket state, audit records)
- **Embeddings:** `OpenAI text-embedding-3-small` (1536-dimensional dense vectors)
- **Primary LLM:** `OpenAI gpt-4o-mini` (High-speed, cost-optimized inference)
- **External Search:** `Tavily Search API` (Context-optimized search for AI agents)
- **Document Processing:** `LangChain Text Splitters`, `PyPDF`, `python-docx`
- **Frontend Interface:** Vanilla JavaScript, HTML5, Modern Responsive CSS
- **Containerization:** `Docker` (Multi-stage Python 3.11-slim container)

---

## 📂 Repository Structure

```
OmniDesk_AI/
├── app/
│   ├── api/
│   │   ├── __init__.py
│   │   └── routes.py              # REST endpoints (/api/chat, /api/ingest, /api/health)
│   ├── core/
│   │   ├── __init__.py
│   │   ├── config.py              # Centralized Pydantic settings & environment loader
│   │   └── logging.py             # Structured logging configuration
│   ├── rag/
│   │   ├── __init__.py
│   │   ├── state.py               # TypedDict LangGraph state schema (SQL + Vector + Trace)
│   │   ├── vectorstore.py         # Vectorstore client, embeddings, and vector retriever
│   │   └── workflow.py            # LangGraph StateGraph, evaluation nodes & hybrid edges
│   ├── services/
│   │   ├── __init__.py
│   │   ├── audit.py               # SQLite audit logger for inquiry telemetry & analytics
│   │   └── ingestion.py           # Document parsing (PDF, DOCX, MD, TXT) and chunking
│   └── main.py                    # FastAPI application initialization & route mounting
│
├── data/
│   ├── sample_kb/
│   │   ├── company_it_handbook.md       # IT policies, VPN, MFA, password & hardware rules
│   │   ├── compny_data.md               # Company information, organizational structure, reliability
│   │   └── service_desk_runbook.md      # IT service desk runbooks & ticket priority
│   └── company_records.db               # Relational SQLite database (assets, tickets, employees)
│
├── static/
│   ├── css/style.css              # Enterprise dashboard styling & responsive layout
│   └── js/app.js                  # Async client handling chat, streaming trace & file upload
│
├── templates/
│   └── index.html                 # Main web application dashboard
│
├── uploads/                       # Document ingestion staging directory
│   ├── .gitkeep
│   └── compny_data.pdf
│
├── .dockerignore                  # Docker build exclusion rules
├── .env.example                   # Environment configuration template
├── .gitignore                     # Git tracking exclusions (secrets, logs, caches)
├── Dockerfile                     # Production container specification
├── LICENSE                        # Apache 2.0 Open Source License
├── README.md                      # Engineering documentation & architecture reference
├── ingest_sample_kb.py            # Automated CLI ingestion script for sample knowledge base
├── requirements.txt               # Locked production dependencies
└── run.py                         # Application entrypoint
```

---

## 🚀 Getting Started

### Prerequisites
- **Python:** 3.11 or higher
- **OpenAI API Key:** For embeddings and LLM reasoning
- **Chroma DB:** Persistent local/embedded vector database
- **Tavily API Key:** For external web search fallback

### Local Installation

1. **Clone the repository:**
   ```bash
   git clone https://github.com/bitaayushsinha/OmniDesk_AI.git
   cd OmniDesk_AI
   ```

2. **Create and activate a virtual environment:**
   ```bash
   # On macOS/Linux
   python3 -m venv venv
   source venv/bin/activate

   # On Windows (PowerShell)
   python -m venv venv
   .\venv\Scripts\Activate.ps1
   ```

3. **Install dependencies:**
   ```bash
   pip install --upgrade pip
   pip install -r requirements.txt
   ```

### Environment Configuration

Copy the example environment file and populate your API credentials:
```bash
cp .env.example .env
```

Edit `.env`:
```ini
# Application Metadata
APP_NAME="OmniDesk AI - Autonomous IT Copilot"
APP_ENV=development

# OpenAI API
OPENAI_API_KEY=sk-proj-your-openai-key
OPENAI_MODEL=gpt-4o-mini
EMBEDDING_MODEL=text-embedding-3-small

# Chroma Vector Database
CHROMA_PERSIST_DIRECTORY=./data/chroma
CHROMA_COLLECTION_NAME=compny_data_kb

# Tavily External Web Search
TAVILY_API_KEY=tvly-your-tavily-key

# RAG Parameters
TOP_K=4
MAX_RETRIES=3

# Security & Admin Access
ADMIN_API_KEY=your-secure-admin-token
```

### Knowledge Base & Database Setup

Seed the Chroma vector database and initialize sample records:
```bash
python ingest_sample_kb.py
```
*Output:*
```text
Indexed 3 files -> 18 chunks -> 18 Chroma vectors
```

### Running the Application

Launch the FastAPI ASGI server:
```bash
python run.py
```

The application will be live at:
- **Web Interface:** [http://127.0.0.1:8000](http://127.0.0.1:8000)
- **Interactive OpenAPI Documentation:** [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)

---

## 🐳 Docker Deployment

The application includes an optimized container configuration ready for container orchestration platforms (AWS ECS, Google Cloud Run, Azure Container Apps, Kubernetes).

1. **Build the Docker Image:**
   ```bash
   docker build -t omnidesk-ai:latest .
   ```

2. **Run the Container:**
   ```bash
   docker run -d \
     -p 8080:8080 \
     --env-file .env \
     --name omnidesk-copilot \
     omnidesk-ai:latest
   ```

3. **Verify Container Health:**
   ```bash
   curl http://localhost:8080/api/health
   ```

---

## 📡 REST API Reference

### 1. Execute Copilot Query (Hybrid & RAG)
- **Endpoint:** `POST /api/chat`
- **Payload:**
  ```json
  {
    "question": "Can I replace my company laptop, and do I have an active hardware ticket?"
  }
  ```
- **Response:**
  ```json
  {
    "answer": "Yes, you are eligible for a replacement because your Lenovo X1 Carbon was issued 3.4 years ago (policy requires 3 years of service). However, Ticket #TK-3091 ('Laptop Upgrade Request') was already submitted on October 8 and is currently Pending Manager Approval. No duplicate ticket is required.",
    "source_used": "hybrid_sql_chroma",
    "trace": [
      "Router → HYBRID_INTENT",
      "SQL Tool → Executed assigned_devices & support_tickets lookup (1 device, 1 ticket)",
      "Chroma DB → Retrieved hardware refresh policy (compny_data.md)",
      "Cross-Validator → Verified eligibility (3.4 yrs > 3.0 yrs threshold)",
      "Duplicate Guard → Detected active ticket #TK-3091 (Deflected duplicate)",
      "Answer generation → GROUNDED_HYBRID"
    ],
    "citations": [
      {
        "source": "compny_data.md",
        "snippet": "Company laptops are eligible for standard hardware refresh after 36 months..."
      }
    ]
  }
  ```

### 2. Live Document Ingestion (Protected)
- **Endpoint:** `POST /api/ingest`
- **Headers:** `X-Admin-Key: <ADMIN_API_KEY>`
- **Content-Type:** `multipart/form-data`
- **Form Data:** `file: <path-to-document>` (Supports `.pdf`, `.docx`, `.md`, `.txt`)
- **Response:**
  ```json
  {
    "filename": "compny_data.pdf",
    "chunks_indexed": 12,
    "status": "success"
  }
  ```

### 3. Health & Readiness Probe
- **Endpoint:** `GET /api/health`
- **Response:**
  ```json
  {
    "status": "healthy",
    "app_name": "OmniDesk AI - Autonomous IT Copilot",
    "environment": "development"
  }
  ```

---

## 🧪 Enterprise Verification Scenarios

The following scenarios validate the autonomous decision paths of the agent:

### Scenario A: Hybrid Reasoning & Duplicate Ticket Deflection
- **User Prompt:** *"Can I request a MacBook Pro upgrade, and what is the status of my pending hardware request?"*
- **Agent Execution Path:**
  ```text
  Router → HYBRID → SQL Query (devices & tickets) + Chroma Retrieval (policy) → Cross-Validation → Duplicate Ticket Guard → Answer
  ```
- **Result:** Detects that the employee's current laptop exceeds the 3-year refresh threshold, but **deflects a duplicate ticket** by reporting that Ticket `#TK-3091` is already in the procurement approval pipeline.

### Scenario B: Structured SQL Telemetry Lookup (SQL Only)
- **User Prompt:** *"What is the priority and current status of ticket TK-1042?"*
- **Agent Execution Path:**
  ```text
  Router → SQL_ONLY → Parameterized SQLite Query → Row Validation → Structured Answer
  ```
- **Result:** Returns precise status without touching vector embeddings or hallucinating ticket history.

### Scenario C: High-Confidence Private KB Resolution (Vector Only)
- **User Prompt:** *"What does error code ERR_CONN_8421 indicate, and what is our P95 latency target?"*
- **Agent Execution Path:**
  ```text
  Router → VECTOR_ONLY → Chroma DB Retrieval → Document Grading (GOOD) → Grounded Synthesis
  ```
- **Result:** Directly quotes reliability standards (P95 < 250ms; `ERR_CONN_8421` indicates primary database connection pool exhaustion). **Zero external web calls are initiated.**

### Scenario D: Sensitive Security Policy Enforcement
- **User Prompt:** *"Can IT support ask me for my MFA code to diagnose a login problem?"*
- **Agent Execution Path:**
  ```text
  Router → VECTOR_ONLY → Chroma DB Retrieval → Document Grading (GOOD) → Grounded Synthesis
  ```
- **Result:** Answers with strict adherence to Information Security Handbook: Employees must **never** share passwords or MFA codes with support staff.

### Scenario E: Fallback to External Intelligence
- **User Prompt:** *"Is there an active Microsoft Teams outage or disruption affecting auth today?"*
- **Agent Execution Path:**
  ```text
  Router → VECTOR_ONLY → Chroma DB Retrieval → Document Grading (WEAK) → Tavily Search → Web Grading (GOOD) → Grounded Synthesis with External Attribution
  ```
- **Result:** Detects private KB inadequacy, queries Tavily for current vendor telemetry, and synthesizes an up-to-date response citing public sources.

---

## 🔒 Security & Production Readiness

- **Role-Based Admin Protection:** The `/api/ingest` endpoint is protected via an HTTP header secret (`X-Admin-Key`), preventing unauthorized data manipulation.
- **Data Sovereignty:** Enterprise documents are indexed strictly in private Chroma DB collections and local relational tables. No internal data is passed to third-party search APIs.
- **Fail-Safe Loop Budgeting:** Graph iterations are bounded by deterministic retry guards, preventing unbounded token burn.
- **Input Sanitization & Type Safety:** All payloads are validated at runtime via Pydantic v2 schemas, and database queries use read-only execution patterns.
- **Audit Persistence:** Every prompt, source classification, and execution trace is asynchronously recorded to SQLite for compliance review.

---

## 👨‍💻 Author & Engineering Portfolio

Developed by **Aayush Sinha**
- **GitHub:** [@bitaayushsinha](https://github.com/bitaayushsinha)
- **Repository:** [https://github.com/bitaayushsinha/OmniDesk_AI](https://github.com/bitaayushsinha/OmniDesk_AI)
- **Role:** Forward Deployed Engineer / AI Systems Architect

---

## 📄 License

This project is licensed under the Apache License 2.0 - see the [LICENSE](LICENSE) file for details.

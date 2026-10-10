# OmniDesk AI: Enterprise Autonomous IT Copilot & Agentic RAG System

[![Python Version](https://img.shields.io/badge/Python-3.11%2B-blue.svg)](https://www.python.org/)
[![Framework](https://img.shields.io/badge/Framework-FastAPI-009688.svg)](https://fastapi.tiangolo.com/)
[![Orchestration](https://img.shields.io/badge/Agent_Orchestration-LangGraph-FF6F00.svg)](https://langchain-ai.github.io/langgraph/)
[![Vector DB](https://img.shields.io/badge/Vector_DB-Chroma_DB-FF6600.svg)](https://www.trychroma.com/)
[![LLM & Embeddings](https://img.shields.io/badge/LLM-OpenAI_GPT--4o--mini-412991.svg)](https://openai.com/)
[![Docker](https://img.shields.io/badge/Deployment-Docker-2496ED.svg)](https://www.docker.com/)
[![License](https://img.shields.io/badge/License-Apache_2.0-green.svg)](LICENSE)

**OmniDesk AI** is a production-grade, autonomous IT Support and Knowledge Operations Copilot designed for modern enterprises. Built with a **stateful Agentic RAG architecture powered by LangGraph, FastAPI, Chroma DB, and OpenAI**, OmniDesk AI shifts traditional static retrieval systems into an active decision-making assistant. 

Deployed for enterprise IT and platform operations, OmniDesk resolves internal technical inquiries, enforces data sovereignty, self-corrects retrieval failures, and falls back to real-time external web intelligence only when internal knowledge is verified to be insufficient.

---

## 📑 Table of Contents
- [Business Problem & Operational Context](#-business-problem--operational-context)
- [System Architecture & Agentic Workflow](#-system-architecture--agentic-workflow)
- [Key Engineering Differentiators](#-key-engineering-differentiators)
- [Technology Stack](#-technology-stack)
- [Repository Structure](#-repository-structure)
- [Getting Started](#-getting-started)
  - [Prerequisites](#prerequisites)
  - [Local Installation](#local-installation)
  - [Environment Configuration](#environment-configuration)
  - [Knowledge Base Ingestion](#knowledge-base-ingestion)
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

### Operational Bottlenecks
Internal IT and platform support teams manage sprawling, multi-format knowledge bases (VPN setups, MFA enforcement, SEV incident runbooks, P95 latency thresholds, database connection pools, cloud cost policies, and hardware allowances). Traditional support setups suffered from:
1. **High Support Desk Load:** Engineers repeatedly submitted tickets for known policies and standard operating procedures.
2. **Naive RAG Hallucinations:** Traditional semantic search blindly injects irrelevant chunks into prompts, leading models to invent plausible-sounding yet erroneous troubleshooting steps.
3. **Data Staleness & Lack of External Telemetry:** When third-party vendors (e.g., cloud platforms, Microsoft Teams, auth providers) suffer service disruptions, internal KBs have no answers, causing naive systems to fail silently.
4. **Zero Observability:** Standard chatbots act as black boxes with no auditable trace of retrieval decisions, confidence grading, or data lineage.

### OmniDesk AI Solution
OmniDesk AI delivers an **adaptive, self-evaluating copilot** that:
- Prioritizes private enterprise documents with strict data sovereignty.
- Autonomously grades retrieved evidence before synthesizing an answer.
- Routes to real-time web intelligence only when private documentation is genuinely deficient.
- Rewrites ambiguous questions and re-queries vector indexes under a loop-safe budget.
- Returns comprehensive execution traces for compliance, debugging, and end-user transparency.

---

## 🧠 System Architecture & Agentic Workflow

Unlike conventional RAG pipelines (`Question → Retrieve → Answer`), OmniDesk AI executes an **agentic state machine** with conditional routing and dynamic feedback loops:

```
                          ┌───────────────────────────┐
                          │   Incoming User Query     │
                          └─────────────┬─────────────┘
                                        │
                                        ▼
                         ┌─────────────────────────────┐
                         │      [1] Intent Router      │
                         └──────┬───────────────┬──────┘
       Casual/Greeting          │               │  Technical / Policy Inquiry
   ┌────────────────────────────┘               ▼
   │                             ┌─────────────────────────────┐
   │                             │  [2] Private KB Retrieval   │
   │                             │      (Chroma DB Top-K)      │
   │                             └──────────────┬──────────────┘
   │                                            │
   │                                            ▼
   │                             ┌─────────────────────────────┐
   │                             │  [3] Evidence Self-Grader   │
   │                             └──────┬───────────────┬──────┘
   │                                    │               │
   │                          Evidence Is HIGH       Evidence Is WEAK
   │                                    │               │
   │                                    │               ▼
   │                                    │  ┌───────────────────────────┐
   │                                    │  │ [4] Tavily Web Fallback   │
   │                                    │  └────────────┬──────────────┘
   │                                    │               │
   │                                    │               ▼
   │                                    │  ┌───────────────────────────┐
   │                                    │  │  [5] Web Evidence Grader  │
   │                                    │  └────┬─────────────────┬────┘
   │                                    │       │                 │
   │                                    │   Evidence HIGH     Evidence WEAK
   │                                    │       │                 │
   │                                    │       │                 ▼
   │                                    │       │  ┌───────────────────────────┐
   │                                    │       │  │    [6] Query Rewriter     │
   │                                    │       │  │ (Reformulate & Retry KB)  │
   │                                    │       │  └──────────────┬────────────┘
   │                                    │       │                 │ (Max Retry Guard)
   ▼                                    ▼       ▼                 ▼
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

### Agentic State Machine Transitions
1. **Intent Router:** Classifies queries between direct conversational replies (greetings, acknowledgments) vs. enterprise knowledge inquiries.
2. **Private KB Retrieval:** Extracts dense vector embeddings via OpenAI `text-embedding-3-small` and performs similarity search against Chroma DB.
3. **Self-Reflective Document Grader:** Employs structured output evaluation to assess whether retrieved chunks directly answer the question.
4. **Tavily Web Fallback:** If private documents do not contain the answer, the agent escalates to Tavily external search to fetch real-time vendor updates.
5. **Web Evidence Grader:** Assesses public search results for answerability.
6. **Query Rewriter & Loop Guard:** If both initial passes fail, the agent optimizes the query semantics and retries retrieval up to a bounded maximum retry threshold (`MAX_RETRIES=1`).
7. **Grounded Synthesis & Attribution:** Generates factual responses strictly grounded in the approved evidence, outputting transparent citation metadata.

---

## ⚡ Key Engineering Differentiators

| Capability | Static / Naive RAG | OmniDesk AI (Agentic RAG) |
|---|---|---|
| **Data Routing** | Indiscriminately searches vector DB | Intent-based routing separates casual banter from KB lookups |
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
- **Vector Database:** `Chroma DB` (Persistent open-source vector store & embeddings index)
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
│   │   ├── state.py               # TypedDict LangGraph state schema & Pydantic models
│   │   ├── vectorstore.py         # Vectorstore client, embeddings, and vector retriever
│   │   └── workflow.py            # LangGraph StateGraph, evaluation nodes & edges
│   ├── services/
│   │   ├── __init__.py
│   │   ├── audit.py               # SQLite audit logger for inquiry telemetry
│   │   └── ingestion.py           # Document parsing (PDF, DOCX, MD, TXT) and chunking
│   └── main.py                    # FastAPI application initialization & route mounting
│
├── data/
│   └── sample_kb/
│       ├── company_it_handbook.md       # IT policies, VPN, MFA, password & hardware rules
│       ├── compny_data.md               # Company information, organizational structure, reliability
│       └── service_desk_runbook.md      # IT service desk runbooks & ticket priority
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

### Knowledge Base Ingestion

Seed the Chroma vector database with company documentation:
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

### 1. Execute Copilot Query
- **Endpoint:** `POST /api/chat`
- **Payload:**
  ```json
  {
    "question": "What is the policy on remote work and home internet expenses?"
  }
  ```
- **Response:**
  ```json
  {
    "answer": "Employees may work remotely up to three days per week, with teams designating common office days for collaboration. Fully remote arrangements require approval from both the department head and HR. In addition, all employees may expense up to $50 per month for work-related home internet, and new full-time hires may request a one-time home-office equipment allowance of up to $600.",
    "source_used": "private_kb",
    "trace": [
      "Router → KB",
      "Private KB retrieval → 4 chunks",
      "KB evidence grade → GOOD",
      "Answer generation → PRIVATE KB"
    ],
    "citations": [
      {
        "source": "compny_data.md",
        "snippet": "Employees may work remotely up to three days per week..."
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

### Scenario A: High-Confidence Private KB Resolution
- **User Prompt:** *"What does error code ERR_CONN_8421 indicate, and what is our P95 latency target?"*
- **Agent Execution Path:**
  ```text
  Router → KB → Private KB Retrieval → Document Grading (GOOD) → Grounded Synthesis
  ```
- **Result:** Directly quotes reliability standards (P95 < 250ms; `ERR_CONN_8421` indicates primary database connection pool exhaustion). **Zero external web calls are initiated.**

### Scenario B: Sensitive Security Policy Enforcement
- **User Prompt:** *"Can IT support ask me for my MFA code to diagnose a login problem?"*
- **Agent Execution Path:**
  ```text
  Router → KB → Private KB Retrieval → Document Grading (GOOD) → Grounded Synthesis
  ```
- **Result:** Answers with strict adherence to Information Security Handbook: Employees must **never** share passwords or MFA codes with support staff.

### Scenario C: Fallback to External Intelligence
- **User Prompt:** *"Is there an active Microsoft Teams outage or disruption affecting auth today?"*
- **Agent Execution Path:**
  ```text
  Router → KB → Private KB Retrieval → Document Grading (WEAK) → Tavily Search → Web Grading (GOOD) → Grounded Synthesis with External Attribution
  ```
- **Result:** Detects private KB inadequacy, queries Tavily for current vendor telemetry, and synthesizes an up-to-date response citing public sources.

### Scenario D: Intent-Aware Query Rewriting
- **User Prompt:** *"My communication tool is breaking with connection errors after the recent release."*
- **Agent Execution Path:**
  ```text
  Router → KB → Private KB Retrieval → Document Grading (WEAK) → Query Rewriter → Reformulated Retrieval → Document Grading (GOOD) → Answer
  ```
- **Result:** Rewrites query to match Payment Platform / Database pool incident documentation, surfacing the connection-pool rollback details.

---

## 🔒 Security & Production Readiness

- **Role-Based Admin Protection:** The `/api/ingest` endpoint is protected via an HTTP header secret (`X-Admin-Key`), preventing unauthorized data manipulation.
- **Data Sovereignty:** Enterprise documents are indexed strictly in private Chroma DB collections. No company documents are passed to third-party search APIs.
- **Fail-Safe Loop Budgeting:** Graph iterations are bounded by deterministic retry guards, preventing unbounded token burn.
- **Input Sanitization & Type Safety:** All payloads are validated at runtime via Pydantic v2 schemas.
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

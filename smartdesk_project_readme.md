# SmartDesk

SmartDesk is an enterprise-grade multi-agent orchestration runtime designed for intelligent customer support and ticket operations. Moving beyond simple conversational chatbots, it delivers an end-to-end engineering lifecycle integrating fine-grained intent recognition, adaptive multi-agent routing, intent-driven RAG retrieval, tiered memory management, dynamic business skill injection, real-time observability, and automated LLM-as-Judge evaluation.

---

## System Architecture

```text
User Query (/chat)
       │
       ▼
┌─────────────────────────────────────────────────────────────┐
│ 1. Context Hydration                                        │
│    ├── Redis: Working Memory (Sliding Window & Session TTL) │
│    └── ChromaDB: Episodic & User Profile Memory             │
└──────────────────────────────┬──────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────┐
│ 2. Tri-Path Fusion Intent Recognizer                         │
│    ├── Intent Category & Urgency Classification             │
│    └── Named Entity Extraction                              │
└──────────────────────────────┬──────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────┐
│ 3. Intent-Driven RAG (ChromaDB)                             │
│    └── Hybrid Retrieval & Dynamic Relevance Scoring        │
└──────────────────────────────┬──────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────┐
│ 4. Multi-Agent Orchestrator & Dispatcher                    │
│    ├── General Inquiry Agent                                │
│    ├── Technical Support Agent                              │
│    ├── Billing & Subscription Agent                         │
│    └── Escalation & Human Handover Agent                    │
└──────────────────────────────┬──────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────┐
│ 5. Tool Management & Skill Injection Layer                  │
│    ├── MCP Tool Execution (Caching, Retry, Circuit Breaker) │
│    └── Declarative Business Rule Engine (Hot Reloading)     │
└──────────────────────────────┬──────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────┐
│ 6. Response Generation & Async Memory Persistence           │
│    ├── SSE / Streaming Output                               │
│    └── Async Write-back to Redis & ChromaDB                 │
└──────────────────────────────┬──────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────┐
│ 7. Telemetry & Automated Quality Evaluation                 │
│    ├── Prometheus / Online Metrics (Latency, Token, Cost)   │
│    └── LLM-as-Judge End-to-End Evaluation Pipeline          │
└─────────────────────────────────────────────────────────────┘
```

---

## Key Modules & Capabilities

| Module | Core Endpoints | Technical Highlights |
|---|---|---|
| **Core Conversational Pipeline** | `POST /chat` | Tiered memory lookup $\to$ intent triage $\to$ dynamic RAG $\to$ agent dispatching $\to$ asynchronous state persistence. |
| **Knowledge Base (RAG)** | `POST /search`<br>`POST /knowledge/upload`<br>`GET /knowledge/stats` | Chunking pipelines, vector indexing via ChromaDB, semantic metadata filtering, and relevance thresholding. |
| **Dynamic Skills Engine** | `GET /skills`<br>`POST /skills/reload` | Decoupled declarative business rules injected at runtime without restarting backend processes. |
| **Tool Execution (MCP)** | Internal MCP Tools | Tool invocation registry, Redis result caching, fault-tolerant retry policies, and circuit breaking. |
| **Observability & Evaluation** | `GET /monitor`<br>`POST /eval/run` | Real-time tracking of latency, token consumption, fallback frequencies, and automated offline/online LLM-as-Judge benchmarking. |

---

## Repository Structure

```text
.
├── SmartDesk/                     # Core Backend Service
│   ├── api/
│   │   └── main.py                # FastAPI entry point and routing declarations
│   ├── agents/
│   │   └── agent_orchestrator.py  # Multi-agent state orchestration, dispatching & fallbacks
│   ├── core/
│   │   ├── intent_recognizer.py   # Multi-path intent classification and slot extraction
│   │   └── skill_loader.py        # Runtime declarative skill loading & hot reload
│   ├── memory/
│   │   └── conversation_memory.py # Tiered memory: Redis buffer + ChromaDB semantic memory
│   ├── mcp/
│   │   ├── tool_manager.py        # Tool calling registry, caching, and circuit breakers
│   │   └── knowledge_base.py      # ChromaDB client, indexing, and vector similarity search
│   ├── monitor/
│   │   └── performance_monitor.py # Metrics collection (latency, tokens, error rates)
│   ├── evaluation/
│   │   └── evaluator.py           # End-to-end LLM-as-Judge quality evaluation suite
│   ├── skills/                    # Business rule files and declarative skill configurations
│   ├── data/                      # Persistent vector indexes and test dataset fixtures
│   ├── Dockerfile                 # Backend container definition
│   └── requirements.txt           # Python runtime dependencies
├── SmartDeskFrontend/             # Web Client & Support Desk Dashboard
├── docker-compose.yml             # Container orchestration config (App, Redis, ChromaDB, Prometheus)
├── .env.example                   # Environment configuration template
└── README.md                      # Project documentation
```

---

## Quick Start

### 1. Prerequisites

- [Docker](https://docs.docker.com/get-docker/) and [Docker Compose](https://docs.docker.com/compose/)
- API Key for LLM provider (Anthropic or OpenAI-compatible endpoint)

### 2. Environment Configuration

Clone the repository and initialize your environment settings:

```bash
cp .env.example .env
```

Set the target credentials inside `.env`:

```env
# LLM Provider Configuration
ANTHROPIC_API_KEY=your_actual_api_key
ANTHROPIC_BASE_URL=https://api.deepseek.com/anthropic
ANTHROPIC_MODEL=deepseek-v4-pro

# Redis Settings
REDIS_PASSWORD=smartdesk123
REDIS_HOST=redis
REDIS_PORT=6379

# ChromaDB Settings
CHROMA_SERVER_HOST=chromadb
CHROMA_SERVER_HTTP_PORT=8001
```

### 3. Build & Launch

Start all services (FastAPI Backend, ChromaDB, Redis, and Web Console) using Docker Compose:

```bash
docker compose up -d --build
```

Verify service health:

```bash
docker compose ps
```

Stream backend application logs:

```bash
docker compose logs -f smartdesk
```

---

## Service Endpoints & Ports

| Service Component | Port | URL / Access Path | Description |
|---|---|---|---|
| **SmartDesk API** | `8000` | `http://localhost:8000/docs` | Interactive Swagger API documentation |
| **Health Check** | `8000` | `http://localhost:8000/health` | Service liveness probe |
| **Support Desk Frontend** | `80` | `http://localhost` | Operational agent web interface |
| **ChromaDB Vector Store** | `8001` | `http://localhost:8001` | Standalone vector database server |
| **Prometheus Metrics** | `9090` | `http://localhost:9090` | Time-series performance metrics |

---

## Verification & Debugging Workflow

To validate the deployment pipeline step-by-step, run through the following commands:

```bash
# 1. Check API Liveness
curl -X GET "http://localhost:8000/health"

# 2. Test Multi-Agent Conversation Flow
curl -X POST "http://localhost:8000/chat" \
  -H "Content-Type: application/json" \
  -d '{
    "session_id": "demo-test-001",
    "user_id": "usr_9981",
    "message": "I need help with my monthly invoice payment failure."
  }'

# 3. Query Loaded Business Skills
curl -X GET "http://localhost:8000/skills"

# 4. Inspect Performance Metrics
curl -X GET "http://localhost:8000/monitor"

# 5. Trigger Automated Quality Evaluation
curl -X POST "http://localhost:8000/eval/run"
```

---

## License

This project is licensed under the [MIT License](LICENSE).
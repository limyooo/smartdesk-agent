=# SmartDesk

SmartDesk is a multi-agent orchestration runtime designed for intelligent customer support and ticket operations. Moving beyond simple conversational bots, it establishes an end-to-end engineering lifecycle integrating fine-grained intent detection, adaptive routing, knowledge retrieval, tiered memory management, dynamic rule injection, and automated evaluation.

## Core Architecture & Workflow

```text
User Request (/chat)
  -> Context Hydration (Redis working memory + ChromaDB episodic memory)
  -> Intent Recognition (fine-grained intent, entity extraction, urgency scoring)
  -> Intent-Driven RAG (conditional ChromaDB vector retrieval)
  -> Agent Orchestration & Routing (General / Technical / Billing / Escalation)
  -> Dynamic Skills Injection & Tool Execution
  -> Response Generation & Asynchronous Memory Persistence
  -> Online Metric Telemetry & LLM-as-Judge Quality Evaluation

SmartDesk/
├── api/main.py                   # FastAPI entry point and route definitions
├── agents/agent_orchestrator.py  # Multi-agent dispatching, state graph, and fallbacks
├── core/intent_recognizer.py     # Multi-path fusion intent classification
├── core/skill_loader.py          # Dynamic rule loading and hot swapping
├── memory/conversation_memory.py # Tiered memory (Redis buffer + ChromaDB semantic store)
├── mcp/
│   ├── tool_manager.py           # Tool invocation chain, circuit breakers, and caching
│   └── knowledge_base.py         # Vector search, document chunking, and index management
├── monitor/performance_monitor.py# Real-time metrics (latency, token costs, fallback rates)
├── evaluation/evaluator.py       # End-to-end evaluation pipeline (LLM-as-Judge)
├── skills/                       # Declarative operational rules and skill definitions
├── data/                         # Persistent storage and local vector index snapshots
└── wiki/                         # System architecture and technical deep-dives
SmartDeskFrontend/                # Customer service agent workspace and dashboard

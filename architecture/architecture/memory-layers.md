# 4-Layer Cognitive Memory Architecture

## Overview

The platform uses a biologically-inspired 4-layer memory system,
where each layer serves a distinct purpose and uses the optimal
storage technology.

## Memory Layers

```
┌─────────────────────────────────────────────┐
│           MEMORY ARCHITECTURE               │
├─────────────────────────────────────────────┤
│                                             │
│  1. WORKING MEMORY        → Redis           │
│     Active dialog context                  │
│     Multi-turn session state               │
│     TTL: 30 minutes                        │
│                                             │
│  2. EPISODIC MEMORY       → PostgreSQL      │
│     "What happened?" — event history       │
│     Ticket resolution history              │
│     Engineer activity logs                 │
│                                             │
│  3. SEMANTIC MEMORY       → Qdrant (RAG)   │
│     "What does it know?" — knowledge base  │
│     Runbooks, procedures                   │
│     Technical documentation               │
│     46,000+ enterprise tickets            │
│                                             │
│  4. PROCEDURAL MEMORY     → LoRA / vLLM    │
│     "How does it behave?" — learned logic  │
│     AzInTelecom terminology               │
│     Business rules, priorities            │
│     Fine-tuned into the model             │
│                                             │
└─────────────────────────────────────────────┘
```

## How It Works in a Request

```
Query arrives
     ↓
Working Memory   → Redis       (what did we discuss?)
     ↓
Episodic Memory  → PostgreSQL  (has this problem occurred before?)
     ↓
Semantic Memory  → Qdrant/RAG  (what does the knowledge base say?)
     ↓
Procedural Memory → LoRA model (how should I respond?)
     ↓
LLM generates response
```

## Storage Summary

| Layer | Storage | Status |
|-------|---------|--------|
| Working | Redis | ✅ Active |
| Episodic | PostgreSQL | ✅ Active |
| Semantic | Qdrant | ✅ Active |
| Procedural | LoRA/vLLM | 🔄 Training |

## Key Design Decisions

- **Graceful degradation** — if any layer is unavailable, 
  the system continues with remaining layers
- **TTL management** — Working memory expires after 30 minutes
  to prevent context pollution
- **RAG enrichment** — every query is enriched with semantic 
  context before reaching the LLM
- **Persistent storage** — Episodic and Semantic layers stored 
  on NFS (955GB) for durability

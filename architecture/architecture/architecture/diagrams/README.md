# Architecture Diagrams

## 1. Platform Overview

```
┌─────────────────────────────────────────────────────────┐
│                    TAC AI PLATFORM                       │
├─────────────────┬───────────────────────────────────────┤
│   HPC CLUSTER   │         KUBERNETES CLUSTER            │
│                 │                                        │
│  ┌───────────┐  │  ┌────────────┐  ┌────────────────┐  │
│  │   vLLM    │◄─┼──│Orchestrator│  │   RabbitMQ     │  │
│  │  H200 GPU │  │  │  FastAPI   │  │  4 Queues      │  │
│  │Qwen2.5-7B │  │  └─────┬──────┘  └────────────────┘  │
│  └───────────┘  │        │                              │
│                 │        ▼                              │
│  ┌───────────┐  │  ┌───────────┐  ┌──────────────────┐ │
│  │   SLURM   │  │  │LLM Router │  │  Memory Layer    │ │
│  │ Scheduler │  │  │ Classifier│  │  Redis (Working) │ │
│  └───────────┘  │  └───────────┘  │  PgSQL (Episode) │ │
│                 │                 │  Qdrant (Semant.) │ │
│  ┌───────────┐  │                 │  LoRA (Procedur.) │ │
│  │   LoRA    │  │                 └──────────────────┘ │
│  │ Fine-tune │  │         NFS Storage (955GB)          │
│  └───────────┘  │                                      │
└─────────────────┴──────────────────────────────────────┘
```

---

## 2. LLM Routing Pipeline

```
Request
   ↓
┌─────────────────────┐
│  Constraint Filter  │  — Auth · Rate limit · Validation
└─────────┬───────────┘
          ↓
┌─────────────────────┐
│  Classifier (SLM)   │  — Complexity score
│  intent detection   │  — Token analysis
└─────────┬───────────┘
          ↓
    ┌─────┴──────┐
    ↓            ↓
Small LLM     Large LLM
score < 0.4   score ≥ 0.4
max 512 tok   max 1024 tok
Fast ~1-2s    Deep analysis
    ↓            ↓
    └─────┬──────┘
          ↓
┌─────────────────────┐
│   Fallback Rules    │  — Timeout · Model down · Escalate
└─────────────────────┘
```

---

## 3. 4-Layer Memory Architecture

```
┌─────────────────────────────────────────┐
│         COGNITIVE MEMORY LAYERS         │
├─────────────────────────────────────────┤
│                                         │
│  1. WORKING MEMORY    → Redis           │
│     Active dialog · Session state       │
│     TTL: 30 minutes                     │
│                                         │
│  2. EPISODIC MEMORY   → PostgreSQL      │
│     Ticket history · Event logs         │
│     Who · When · What happened          │
│                                         │
│  3. SEMANTIC MEMORY   → Qdrant (RAG)    │
│     Knowledge base · Runbooks           │
│     46,000+ enterprise tickets          │
│                                         │
│  4. PROCEDURAL MEMORY → LoRA / vLLM     │
│     Learned behavior · Terminology      │
│     Business logic · Fine-tuned model   │
│                                         │
└─────────────────────────────────────────┘
```

---

## 4. RAG Pipeline

```
User Query
     ↓
Embedding Model (all-MiniLM-L6-v2, 384-dim)
     ↓
Qdrant Vector Search (cosine similarity)
Top-3 results · score > 0.5
     ↓
Context Enrichment
System prompt + retrieved docs + query
     ↓
vLLM Inference (H200 GPU)
     ↓
Accurate · Grounded · Traceable Response
```

---

## 5. Request Flow (End-to-End)

```
User
  ↓
TAC Orchestrator (FastAPI · K8s NodePort 31184)
  ↓
Constraint Filter
  ↓
Memory Retrieval ──────────────────────────┐
  ↓                                        │
  ├── Redis      (Working Memory)          │
  ├── PostgreSQL (Episodic Memory)         │
  └── Qdrant     (Semantic Memory / RAG)   │
  ↓                                        │
Classifier (complexity score) ◄────────────┘
  ↓
  ├── score < 0.4 → Small LLM (fast)
  └── score ≥ 0.4 → Large LLM (deep)
  ↓
vLLM Server (NVIDIA H200 · HPC Cluster)
  ↓
Response + Save to Memory
  ↓
User
```

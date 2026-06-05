# Platform Architecture Overview

## System Design

TAC AI Multi-Agent Platform consists of two main infrastructure layers:

### HPC Layer (GPU Inference)
- SLURM job scheduler for GPU workload management
- NVIDIA H200 GPU for vLLM inference server
- Shared Lustre filesystem for model storage
- Job-based auto-restart (watchdog)

### Kubernetes Layer (Microservices)
- 3-node cluster (1 master + 2 workers)
- Calico CNI for network policy
- NFS persistent storage (955GB)
- Namespace isolation (tac-ai)

## Request Flow

```
User Request
     ↓
TAC Orchestrator (FastAPI)
     ↓
Constraint Filter → Classifier
     ↓
Memory Retrieval (Redis + PostgreSQL + Qdrant)
     ↓
LLM Routing (Small / Large)
     ↓
vLLM Inference Server (H200 GPU)
     ↓
Response + Memory Save
```

## Agent Communication

All agents communicate asynchronously via RabbitMQ:

```
tac.monitoring    — monitoring alerts
tac.tickets       — ticket operations
tac.notifications — user notifications
tac.analytics     — analytics pipeline
```

## Data Flow

```
Input → RAG Search (Qdrant) → Context Enrichment → LLM → Output
                                     ↑
                              PostgreSQL (history)
                              Redis (session)
```

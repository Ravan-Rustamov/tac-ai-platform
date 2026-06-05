# TAC AI Multi-Agent Platform
### Enterprise AI Ecosystem Architecture — AzInTelecom LLC

![Architecture](diagrams/platform-overview.png)

> ⚠️ **Note:** Source code and configuration files are not publicly 
> available. This repository contains only architectural documentation 
> and design decisions.
> 
> **Reason:** This system operates on critical national government cloud 
> infrastructure. Full implementation details are kept confidential 
> for security reasons.

---

## Overview

A production-grade AI Multi-Agent Ecosystem designed for enterprise 
ITSM automation, built on sovereign on-premise infrastructure.

**Result:** 9,482 man-hours/year saved · 620+ tickets automated 24/7

---

## Architecture Highlights

- **LLM Inference** — vLLM on NVIDIA H200 GPU (HPC/SLURM cluster)
- **Orchestration** — Kubernetes (kubeadm) + LangGraph
- **Message Broker** — RabbitMQ (async agent communication)
- **RAG Pipeline** — Qdrant vector DB + Sentence Transformers
- **Memory Layers** — 4-layer Cognitive Memory Architecture
- **Fine-tuning** — LoRA on 46,000+ domain-specific enterprise tickets
- **Storage** — NFS persistent storage (955GB)

---

## Platform Components

| Component | Technology | Purpose |
|-----------|-----------|---------|
| LLM Server | vLLM + Qwen2.5-7B | Inference engine |
| Orchestrator | LangGraph + FastAPI | Agent coordination |
| Message Broker | RabbitMQ | Async communication |
| Vector DB | Qdrant | RAG knowledge base |
| Episodic Memory | PostgreSQL | Interaction history |
| Working Memory | Redis | Session context |
| Container Platform | Kubernetes v1.29 | Microservices |
| Job Scheduler | SLURM | GPU workload management |

---

## Key Architectural Decisions

### 1. LLM Routing Layer

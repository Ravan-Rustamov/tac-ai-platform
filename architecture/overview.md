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

# RAG Pipeline Architecture

## Overview

Retrieval-Augmented Generation (RAG) enriches every LLM query
with relevant context from the enterprise knowledge base before
inference — dramatically improving response accuracy and relevance.

## Pipeline

```
User Query
     ↓
Embedding Model (all-MiniLM-L6-v2)
Convert query to 384-dimensional vector
     ↓
Qdrant Vector Search
Find top-3 most similar documents (cosine similarity)
     ↓
Context Filtering (score > 0.5)
Only high-relevance results pass
     ↓
Prompt Enrichment
System prompt + retrieved context + user query
     ↓
vLLM Inference
     ↓
Response
```

## Knowledge Base

| Source | Documents | Status |
|--------|-----------|--------|
| TAC procedures & runbooks | 12 | ✅ Active |
| 2023 enterprise tickets | 46,000+ | 🔄 Loading |
| 2024-2026 tickets | Pending | ⏳ Planned |
| Email communications | Pending | ⏳ Planned |

## Embedding Model

```
Model:      all-MiniLM-L6-v2
Dimensions: 384
Distance:   Cosine Similarity
Language:   Multilingual (Azerbaijani + English)
```

## Vector Database

```
Engine:     Qdrant v1.13.0
Collection: tac_knowledge
Storage:    NFS persistent (10.6.15.13:/tac-storage/qdrant)
```

## Example

```
Query: "P1 ticket SLA nə qədərdir?"

RAG Results:
  score=0.703 → "P1 tiket ən yüksək prioritetdir. 4 saatda həll."
  score=0.665 → "P2 tiket. 8 saatda həll."
  score=0.656 → "P3 tiket. 24 saatda həll."

Enriched prompt sent to vLLM →
Accurate, context-aware response generated
```

## Benefits

- **Accuracy** — responses grounded in real enterprise data
- **Up-to-date** — knowledge base updated without retraining
- **Traceable** — every response has a source document
- **Efficient** — only relevant context sent to LLM

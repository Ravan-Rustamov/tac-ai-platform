# LLM Routing Layer

## Overview

Every request passes through a 4-stage routing pipeline before 
reaching the LLM. This optimizes both response speed and GPU resources.

## Pipeline

```
Request
   ↓
┌─────────────────────┐
│  Constraint Filter  │  — Auth, rate limit, input validation
└─────────┬───────────┘
          ↓
┌─────────────────────┐
│  Classifier (SLM)   │  — Complexity scoring
│  complexity score   │  — Intent detection
└─────────┬───────────┘
          ↓
    ┌─────┴──────┐
    ↓            ↓
Small LLM     Large LLM
(score < 0.4) (score ≥ 0.4)
Fast response  Deep analysis
max 512 tokens max 1024 tokens
    ↓            ↓
    └─────┬──────┘
          ↓
┌─────────────────────┐
│   Fallback Rules    │  — Timeout, low confidence, model down
└─────────────────────┘
```

## Complexity Scoring

| Signal | Weight |
|--------|--------|
| Token count > 50 | +0.3 |
| Token count > 20 | +0.15 |
| Analytics keywords | +0.2 each |
| Simple keywords | -0.1 each |
| Context depth > 3 | +0.2 |

## Large LLM Keywords
```
analiz, hesabat, trend, root cause, statistika,
proqnoz, araşdır, müqayisə, korrelyasiya
```

## Fallback Rules

| Condition | Action |
|-----------|--------|
| Small LLM timeout | Escalate to Large LLM |
| Large LLM timeout | Default response |
| Model down | Static fallback message |
| 3x retry fail | Human agent escalation |

## Benefits

- **Speed** — simple queries answered in ~1-2 seconds
- **Efficiency** — GPU resources used only when needed
- **Reliability** — graceful degradation on failures

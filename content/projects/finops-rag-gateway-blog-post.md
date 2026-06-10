---
title: "What I Learned Building a FinOps Control Plane for an AI Gateway"
date: 2026-06-09
description: "Hybrid RAG, cost routing, LangSmith traces, and Grafana metrics on a local AI gateway lab."
---

# What I Learned Building a FinOps Control Plane for an AI Gateway

Most teams add a chatbot and discover two problems later: **token spend has no owner**, and **retrieval changes are impossible to compare**.

I built [FinOps AI Gateway](https://github.com/muhammadahmed-01/FinOps-AI-Gateway) as a learning lab to tackle both. It is not a production deployment at Careem. It is a demo with real instrumentation: hybrid RAG, tiered model routing, LangSmith traces, and Grafana dashboards.

## The problem I wanted to explore

Shipping RAG in production usually fails quietly:

1. **Cost visibility**: every query hits the same model tier. Nobody can answer "what did last Tuesday's support bot traffic cost?"
2. **Retrieval quality**: you change BM25 weights or add a reranker and hope answers feel better. There is no before/after score.

I wanted a gateway that treats those as engineering problems, not product guesses.

## What the gateway does

One query flows through:

1. **Classify** query complexity (Groq)
2. **Retrieve** with hybrid RAG (pgvector + BM25 + rerank)
3. **Route** to cheap vs expensive tiers (local Ollama vs Groq)
4. **Export** routing labels, latency breakdown, modeled cost, and eval scores to **Grafana + LangSmith**

Stack: Python, PostgreSQL/pgvector, Prometheus, Pushgateway, Grafana, LangSmith, Docker Compose.

## Measured results (from committed benchmarks)

### Routing (load test, n=12)

| Tier | Share |
|------|-------|
| Simple (local Ollama) | 7/12 (58%) |
| Medium | 3/12 |
| Complex | 2/12 |

### Latency

| Component | p50 (approx.) |
|-----------|----------------|
| End-to-end | ~276s (concurrency=2, local Ollama) |
| Retrieval | ~2.3s |
| Generation (simple tier) | ~356s (local Ollama dominates wall time) |
| Generation (medium/complex) | ~1–4s (Groq in demo mode) |

Retrieval is fast. Local generation is slow and free. That tradeoff is visible in the Grafana **Latency Breakdown p50** panel.

### Cost (simulated FinOps model, not actual API spend)

Demo mode uses Groq + Ollama ($0 actual spend). Grafana applies Claude list rates × tokens for medium/complex tiers as a **counterfactual model**.

| Metric | Value |
|--------|-------|
| Simulated spend (tier routing) | $0.0143 (12 queries) |
| Simulated all-complex counterfactual | $0.0609 |
| Modeled savings vs counterfactual | ~77% (simulation only) |
| **Actual API spend (demo mode)** | **$0** |

Label simulated numbers clearly. Do not present them as production FinOps proof.

### RAG eval (RAGAS pilot, n=8)

Eight Q&A pairs is enough to compare pipelines directionally. It is not statistically powered for strong faithfulness claims.

| Metric | Baseline (cosine) | Hybrid+rerank | Delta |
|--------|-------------------|---------------|-------|
| **context_precision** | 0.6042 | **0.7917** | **+0.1875** |
| faithfulness | 0.7690 | 1.0000 | +0.2310 *(noisy at n=8)* |
| answer_relevancy | 0.7716 | 0.7947 | +0.0231 |

**Lead metric:** context_precision +0.19 on the pilot set. Do not over-claim faithfulness at n=8.

## What I would do differently for production

1. **Scale the eval set** to n=50+ before claiming retrieval wins to stakeholders.
2. **Separate measured spend from modeled spend** in every dashboard panel (this repo does; many demos do not).
3. **Add circuit breakers** when cheap tiers fail open to expensive models.
4. **Wire real billing exports** (AWS Cost Explorer, OpenAI usage API) instead of list-price simulation.

## What this lab is good for

- Showing you can instrument an LLM gateway like any other backend service
- Comparing retrieval pipelines with structured eval, not vibes
- Explaining tiered routing tradeoffs with numbers attached

## What it is not

- A replacement for production PostgreSQL/API performance work (my day job at Careem)
- Proof that hybrid RAG is "production ready" at n=8 eval
- A $3,500 Upwork audit deliverable by itself

## Try it locally

```powershell
git clone https://github.com/muhammadahmed-01/FinOps-AI-Gateway
copy .env.example .env   # GROQ_API_KEY + LANGCHAIN_API_KEY
.\scripts\bootstrap_demo.ps1
```

Grafana: http://localhost:3001 → dashboard **FinOps AI Gateway**.

---

*Muhammad Ahmed · Backend engineer @ Careem · [GitHub](https://github.com/muhammadahmed-01) · [LinkedIn](https://www.linkedin.com/in/muhammadahmed19/)*

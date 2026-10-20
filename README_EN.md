# Huanhang Smart Agriculture

> **Project showcase repository**  
> This repository is a project showcase. The complete source code is maintained in a **private** repository: [`huanhang-smart-agriculture-core`](https://github.com/lsfjames/huanhang-smart-agriculture-core).

A local-first smart agriculture AI platform: **leaf disease classification · lesion detection · knowledge-graph Q&A · microservice business flows**.

Target roles: AI Application Engineer / FDE · AI Solutions / AI Algorithm Engineer.

[中文 README](README.md) · [Architecture](docs/ARCHITECTURE.md) · [Stack](docs/STACK.md) · [Metrics](docs/METRICS.md) · [Demo](docs/DEMO_SCRIPT.md)

---

## 60-second overview

| | |
| --- | --- |
| **Problem** | Hard to judge leaf diseases from photos; expert cost is high; generic LLMs invent unsafe advice |
| **Approach** | Classification + detection + **traceable RAG Q&A**; roles (farmer / expert / admin) |
| **Stack** | Vue3 · Spring Cloud · FastAPI · MySQL · Neo4j · Docker Compose |
| **Honest limits** | Detection mAP is a **low baseline**; Docker/CI **not fully verified** on the authoring machine |

```text
Upload leaf ──► MobileNetV3 3-way class ──► diagnosis record
              └─► YOLOv12n lesion boxes ──► overlay (label-space isolated)
Expert review ◄── cross-user review queue      Admin: users / model versions / knowledge review
Q&A ──► Neo4j graph + hybrid RAG ──► citation checks / refuse when insufficient
Inspection ──► Offline Replay (historical frames, NOT a live drone)
```

---

## Contents

1. [Background](#1-background)
2. [Architecture](#2-architecture)
3. [Tech stack](#3-tech-stack)
4. [Business features](#4-business-features)
5. [AI results (real)](#5-ai-results-real)
6. [Engineering & tests](#6-engineering--tests)
7. [Deployment status](#7-deployment-status)
8. [Hard problems](#8-hard-problems)
9. [Demo](#9-demo)
10. [Roadmap](#10-roadmap)
11. [Limits & license](#11-limits--license)

---

## 1. Background

- Field diagnosis is hard (early vs late blight vs Septoria).
- Unsafe LLM advice (dosage / prediction) must be blocked via **reviewed cards + refusal**.
- Need multi-role workflow, not a single demo API.
- Full engineering path: microservices, async tasks, authz, tests, release audit.

## 2. Architecture

See [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) (diagram + module duties).

**Path:** Vue3 → Spring Cloud Gateway → user-service / diagnosis-service → FastAPI (vision & language) → MySQL + Neo4j.

## 3. Tech stack

| Layer | Choices | Role |
| --- | --- | --- |
| Frontend | Vue 3 · TS · Vite · Element Plus | 9 real pages incl. expert/admin/inspection |
| Backend | Spring Boot 3.4 · Gateway · JWT · Flyway | Authz, diagnosis domain, inter-service HTTP |
| AI | FastAPI · PyTorch | Independent model lifecycles |
| Models | MobileNetV3 · YOLOv12n · Qwen2.5-0.5B | Classify / detect / grounded Q&A |
| Data | MySQL 5.7 · Neo4j 5.26 | Business DB + knowledge graph |
| Quality | JUnit · unittest · Compose · Actions | Tests & delivery (status below) |

## 4. Business features

| Role | Capabilities |
| --- | --- |
| Farmer | Register/login, plots, classify+detect, history, Q&A, offline inspection replay |
| Expert | Cross-user review queue, verdict + comments, review history |
| Admin | User roles, **model version inventory**, **knowledge card review workbench** |

## 5. AI results (real)

Offline validation only — **not** field deployment claims.

| Model | Task | Metrics | Notes |
| --- | --- | --- | --- |
| MobileNetV3-Small | Tomato leaf 3-way | **Acc ≈ 0.935 · Macro-F1 ≈ 0.934** | Uncalibrated softmax |
| YOLOv12n | PlantDoc 29-class | **mAP@50 = 0.074 · mAP50-95 = 0.045** | ~11 epochs then **OOM on 4GB GPU** |
| Qwen + RAG | Bounded Q&A | hybrid hit rate **0.9286** (28 dev qs) | Refuse when insufficient; citations ⊆ context |

Details: [docs/METRICS.md](docs/METRICS.md).

## 6. Engineering & tests

JUnit **26/26** · knowledge-store **6/6** · audit probes **17/17** · gateway E2E **11/11** · fault injection **9/9**.

## 7. Deployment status

| Path | Status |
| --- | --- |
| Native Windows processes | **Verified** end-to-end |
| Docker Compose | **Not verified** (Docker not installed on authoring machine) |
| GitHub Actions | Workflow exists; **not claimed green** until remote run succeeds |

## 8. Hard problems

Traceable RAG · label-space isolation for detect vs classify · 4GB training limits · clean microservice boundaries · honest **Offline Replay**.

## 9. Demo

[docs/DEMO_SCRIPT.md](docs/DEMO_SCRIPT.md) · [docs/screenshots/](docs/screenshots/)

## 10. Roadmap

v1.1 Docker+CI clean verify & better detection · v1.2 knowledge store/DB & model rollout · v2.0 real field data.

## 11. Limits & license

- Not a production plant-protection device.
- Detection mAP is a low baseline (training interrupted).
- Showcase docs: MIT. Full private code includes **AGPL-3.0 YOLO stack**, **CC-BY-4.0 PlantDoc**, Qwen terms — see [docs/LICENSES.md](docs/LICENSES.md).

© 2026 Project Showcase · [lsfjames](https://github.com/lsfjames)

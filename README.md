# 🎯 AI & Tech Interview — Complete Study Resource

> Personal, interview-style study material covering AI Engineering, System Design, Coding, LLD, and Behavioral questions. Built for focused prep—not generic feature bloat.

## 📱 Study Hub App

The installable app source is in [`study-app/`](study-app/). The included GitHub Pages workflow publishes it to `https://vanshikalud04.github.io/interview-resources/` once GitHub Pages is enabled for this repository with **Source: GitHub Actions**. After the first online load, it can run from your home screen/app launcher and keeps its study shell available offline.

---

## 📁 AI Engineering Interview Notes

Comprehensive, interview-ready Q&A with diagrams, follow-up chains, scenario breakdowns, and code implementations.

| # | File | Topics | Questions |
|---|------|--------|-----------|
| 1 | [LLM Fundamentals](ai-engineering/01_llm_fundamentals.md) | Transformers, Attention, Tokenization, Embeddings, KV Cache, MoE, Training & Alignment, Production Scenarios | ~50 |
| 2 | [Prompt Engineering](ai-engineering/02_prompt_engineering.md) | Zero/Few-shot, CoT, ReAct, Tree-of-Thought, Prompt Injection, Jailbreaking, Structured Output, Security | ~25 |
| 3 | [RAG](ai-engineering/03_rag.md) | RAG Architecture, Chunking, Hybrid Search, Re-ranking, Agentic RAG, GraphRAG, HyDE, Production RAG | ~30 |
| 4 | [AI Agents & Agentic Systems](ai-engineering/04_ai_agents.md) | ReAct, Plan-and-Execute, Tool Use, MCP, Memory, Multi-Agent, LangChain/LangGraph, Agent Safety | ~35 |
| 5 | [Fine-Tuning & Model Adaptation](ai-engineering/05_fine_tuning.md) | LoRA, QLoRA, PEFT, RLHF, DPO, Instruction Tuning, Catastrophic Forgetting, Data Preparation | ~25 |
| 6 | [Vector Databases & Embeddings](ai-engineering/06_vector_databases.md) | Cosine Similarity, ANN/HNSW, Hybrid Search, Embedding Models, Multi-tenant, Quantization | ~20 |
| 7 | [AI System Design](ai-engineering/07_ai_system_design.md) | Design ChatGPT, RAG System, AI Coding Agent, Multi-Agent Support, LLM Inference Platform, 30+ designs | ~40 |
| 8 | [LLMOps & Production AI](ai-engineering/08_llmops_production.md) | vLLM, SGLang, Quantization, Guardrails, Observability, A/B Testing, Streaming, Cost Optimization | ~35 |
| 9 | [Evaluation & Testing](ai-engineering/09_evaluation_testing.md) | BLEU/ROUGE/BERTScore, LLM-as-Judge, Red Teaming, RAG Eval, Agent Eval, Bias & Fairness | ~30 |
| 10 | [AI Safety, Ethics & Responsible AI](ai-engineering/10_ai_safety_ethics.md) | Hallucinations, Prompt Injection, GDPR, EU AI Act, Bias, Differential Privacy, Incident Response | ~30 |
| 11 | [Multimodal AI](ai-engineering/11_multimodal_ai.md) | CLIP, Vision Transformers, Diffusion Models, Whisper, VQA, Multimodal RAG, Text-to-Video | ~25 |
| 12 | [AI Infrastructure & Scalability](ai-engineering/12_ai_infrastructure.md) | GPU Selection, Tensor/Pipeline Parallelism, Continuous Batching, Speculative Decoding, PagedAttention | ~25 |
| 13 | [Coding & Implementation](ai-engineering/13_coding_implementation.md) | RAG Pipeline, Agent with Tools, Semantic Search, Chunking, Guardrails, Caching — all with Python code | ~22 |
| 14 | [Behavioral & Scenario Questions](ai-engineering/14_behavioral.md) | AI vs Traditional, ROI, Stakeholder Management, Cost Optimization, STAR Framework | ~22 |

**Total: ~400+ interview questions with complete answers**

---

## 📁 System Design

| File | Description |
|------|-------------|
| [System Design Framework](system-design/frameworks.md) | How to approach ANY system design question — 4-step framework, building blocks, AI-specific considerations |
| [Major App Walkthroughs](system-design/major_app_walkthroughs.md) | Diagram-led walkthroughs for URL shortener, feeds, chat, file sync, video, ride hailing, rate limiting, and RAG — with reasoning follow-ups |
| [Questions by Company](system-design/questions_by_company.md) | 150+ system design questions organized by company with topic tags and approach hints |

---

## 📁 Coding (DSA)

| File | Description |
|------|-------------|
| [DSA Pattern Guide](coding/patterns.md) | 18 major patterns with templates, complexity, signal words, and Python code |
| [Maths, Bits & Binary Search](coding/dsa_math_binary_companion.md) | C++-oriented binary-search boundaries, bit manipulation, number theory, modular arithmetic, combinatorics, geometry, and practice drills |
| [Questions by Company](coding/questions_by_company.md) | 1,700+ coding questions from 160+ companies with DSA pattern tags |

---

## 📁 Low-Level Design (OOD)

| File | Description |
|------|-------------|
| [Questions by Company](low-level-design/questions_by_company.md) | 28+ LLD/OOD questions with design patterns and key considerations |

---

## 📁 AI Coding

| File | Description |
|------|-------------|
| [Questions by Company](ai-coding/questions_by_company.md) | 20+ AI coding questions (build/repair/optimize) with approach hints |

---

## 🛡️ Resume-Based Interview Prep

> Layer-by-layer defense material for the claims you choose to make on your resume. Keep it precise and only use implementation details and metrics you can personally verify.

### Opening & Claim Map

| File | What It Covers |
|------|----------------|
| [Intro & Opening](resume-prep/01_intro_and_opening.md) | 60-second pitches, follow-up defense, EE→CS story, and company research framework. |
| [Resume Claim Map](resume-prep/projects/00_resume_claim_map.md) | Technical claim → concept → drill-down path, including questions to verify before an interview. |

### 🔥 Deep Interrogation System (Priority Study Material)

> Q→A→Follow-up→A→Deeper Follow-up→A chains. Every resume bullet deconstructed. Every technology at 7 levels.

| File | Lines | What It Is |
|------|-------|-----------|
| [RAGOS Interrogation](resume-prep/projects/ragos_interrogation.md) | 302 | All 4 bullets deconstructed, architecture, tech defense, attack mode |
| [LLM Cost Guard Interrogation](resume-prep/projects/llm_cost_guard_interrogation.md) | 235 | Redis Lua deep dive, TOCTOU, concurrency testing, provider abstraction |
| [Focus Lock Interrogation](resume-prep/projects/focus_lock_interrogation.md) | 379 | Shannon entropy math, YOLOv8 internals, threading/GIL, metric defense |
| [Experience Interrogation](resume-prep/projects/experience_interrogation.md) | Evidence-backed | Cordum OSS + Research — verified ownership, safe interview framing, and research boundaries |
| [Technology Defense](resume-prep/projects/technology_defense.md) | 601 | ALL 18+ resume technologies at 7 levels (Basic→Scaling) |

### 🎯 Start Here — Interview Tree & Priority Sheet

Use these after the final introduction. They turn its technical claims into a finite, defensible preparation plan instead of an endless list of subjects.

| File | What It Is |
|------|------------|
| [Interview Follow-up Tree](resume-prep/projects/intro_interview_tree.md) | Project-specific L1–L5 interviewer follow-ups for every claim in the introduction, including strong-answer criteria, expected depth, pitfalls, and likely next questions. |
| [AI/Backend Priority Sheet](resume-prep/projects/priority_sheet.md) | The 30 highest-value questions, ranked by probability × depth × risk, with a six-day study sequence. |

### CSE Fundamentals

| Subject | File | Key Topics |
|---------|------|------------|
| Operating Systems | [OS Q&A](resume-prep/05_cse_fundamentals/operating_systems.md) | Process/threads, scheduling, sync, memory, paging, deadlocks |
| DBMS | [DBMS Q&A](resume-prep/05_cse_fundamentals/dbms.md) | ACID, normalization, indexing, transactions, isolation levels |
| Computer Networks | [CN Q&A](resume-prep/05_cse_fundamentals/computer_networks.md) | OSI/TCP-IP, HTTP, DNS, TCP handshake, load balancing |
| SQL Practice | [SQL Questions](resume-prep/05_cse_fundamentals/sql_practice.md) | 30 questions: basic → advanced, window functions, CTEs |
| OOP | [OOP Q&A](resume-prep/05_cse_fundamentals/oops.md) | 4 pillars, SOLID, design patterns, Python/C++ examples |

### Supplementary

| File | What It Covers |
|------|----------------|
| [DevOps & Infra](resume-prep/07_devops_basics.md) | Docker, K8s, CI/CD, Git, Linux, AWS, OpenTelemetry |
| [HR & Behavioral](resume-prep/08_hr_behavioral.md) | STAR framework, 15 big questions, hackathon stories, tricky HR traps |
| [EE Basics](resume-prep/09_ee_basics.md) | Circuit theory, digital electronics, signals, control systems, EE↔CS bridges |

---

## 🗺️ How to Use This Resource

### For Resume Defense (Priority #1)
Start with [Intro & Opening](resume-prep/01_intro_and_opening.md) → [Interview Follow-up Tree](resume-prep/projects/intro_interview_tree.md) → [AI/Backend Priority Sheet](resume-prep/projects/priority_sheet.md). Use the project interrogations only for the topics you need to deepen.

### For AI Engineering Roles
Start with files 1-6 (core concepts), then 7-8 (system design + production), then 9-14 (specialized topics).

### For General SWE Roles
Start with `coding/patterns.md`, then work through `coding/questions_by_company.md` filtered by your target company.

### For System Design Rounds
Read `system-design/frameworks.md` first, then practice questions from `system-design/questions_by_company.md`.

### For LLD/OOD Rounds
Review `low-level-design/questions_by_company.md` and practice implementing the core patterns.

---

## 📊 Coverage

```
AI Engineering Notes ████████████████████████ 14 files, ~400+ Q&A
Resume Defense       ████████████████████████ 14 files, layered
  ├─ Interview Flow  ███                      3 layers (intro → experience → skills)
  ├─ Project Dives   ███                      3 projects (code-analyzed)
  ├─ CSE Fundamentals████                     5 subjects (OS, DBMS, CN, SQL, OOP)
  └─ Supplementary   ███                      3 files (DevOps, HR, EE)
System Design        ██████████████████       150+ questions
Coding (DSA)         ████████████████████████ 1,700+ questions
Low-Level Design     ████                     28+ questions
AI Coding            ███                      20+ questions
─────────────────────────────────────────────────────────────
Total                                         ~2,300+ questions + full resume defense
```

---

*Built for interview training. Not quick revision — comprehensive study material.*

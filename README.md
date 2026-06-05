# Cascai — SKILL Files as Risk Gates for AI Agent Hierarchies

> *"I ran 200+ experiments on AI agents and found they fail silently."*

Cascai is an open-source quantitative framework for measuring how **SKILL files** (structured instruction documents) affect error propagation in LLM agent hierarchies.

---

## Key Findings

Across **204 controlled API runs** across 4 task domains and 2 model tiers:

| Finding | Detail |
|---------|--------|
| 🛑 Token overflow failure | Uninstructed agents exhaust token budgets mid-function — code arrives truncated and unparseable. First documented here. |
| ⚠️ Reverse Effect | SKILL files *harmed* Sonnet on complex tasks: accuracy dropped from **80% → 40%** by inducing over-engineered, interface-non-compliant solutions. |
| ✅ SKILL as Stop-Loss | Manual SKILL files eliminated all token overflow failures, reduced output tokens by **28.2%**, and cut response time by **26.3%**. |
| 🔀 Smaller model wins | Haiku outperformed Sonnet (97–100% vs 40–80%) on tasks rewarding conciseness. |

---

## What is a SKILL File?

A SKILL file is a structured markdown document prepended to an agent's system prompt. It encodes domain rules, anti-patterns, and output constraints — analogous to a **Stop-Loss mechanism** in quantitative trading.

```
no_skill     →  agent receives task description only
auto_skill   →  generic best-practices SKILL file
manual_skill →  hand-crafted domain-specific SKILL file
```

---

## Experiments

| Task | Domain | Model | Runs | Key Result |
|------|--------|-------|------|------------|
| 001 | Compound Interest | Haiku | 30 | Baseline |
| 002 | SQL Injection Validation | Haiku | 30 | Token overflow documented |
| 003 | 2D Charge Field Separation | Haiku + Sonnet | 60 | Reverse Effect documented |
| 004 | 3D Charge Field + Dynamic Charges | Haiku + Sonnet | 84 | Domain SKILL files studied |

---

## Results

```
results/
├── results_task002.csv    # 30 runs — SQL injection
├── results_task003.csv    # 60 runs — geometric classification
└── results_task004.csv    # 84 runs — 3D physics
```

---

## Paper

Full research paper: [`paper.md`](./paper.md)

Submitted to arXiv under `cs.AI` + `cs.LG`.

---

## Two Novel Failure Modes

**1. Token Overflow Failure**
Without output constraints, verbose agents exhaust the token budget before completing the function. The result: syntactically invalid code passed to the next agent in the hierarchy.

**2. Interface Compliance Failure (Reverse Effect)**
Over-specified SKILL files cause capable models to generate architecturally complex solutions that fail to expose the required function interface. More instruction → worse outcome.

---

## Author

Yoav Badusa — Independent AI Safety Researcher  
GitHub: [github.com/YoavBadusa22/Cascai-producct-main]https://github.com/YoavBadusa22/cascai-research

# Cascai: SKILL Files as Risk Gates for AI Agent Hierarchies

## A Quantitative Study of Error Propagation in Multi-Agent Systems

**Author:** Yoav Badusa  
**Date:** April 2026  
**Repository:** github.com/yoavb/cascai

---

## Abstract

As AI coding agents become more prevalent, multi-agent systems
(agent swarms) are increasingly used for complex software tasks.
However, when a junior agent makes an error, that error can
propagate to senior agents — causing cascading failures that
are expensive to fix.

This paper introduces **Cascai**, a quantitative framework for
measuring how SKILL files (structured instruction documents for
AI agents) affect error propagation in agent hierarchies across
multiple task domains and model tiers.

Our key findings across 204 total experiment runs:

- Manual SKILL files reduce output tokens by **28.2%** and eliminate
  token overflow failures in structured coding tasks (Task 002)
- SKILL files improve pass rate from 90% to **100%** in security
  validation — a reliability gain driven by output compression,
  not knowledge improvement
- In reasoning tasks (Task 003), **smaller models (Haiku) outperform
  larger models (Sonnet)** when the task rewards conciseness
- SKILL files exhibit a **reverse effect** on larger models in complex
  tasks: Sonnet accuracy dropped from 80% to 40% when given a
  physics-informed SKILL file in 2D (Task 003)
- In 3D dynamic tasks (Task 004), the reverse effect **switches models**:
  Haiku is now harmed by manual SKILL (43%), while Sonnet benefits (93%)
- Task 004 is the first experiment in which models spontaneously
  discovered the **Lorentz force** approach — used in 18% of all runs
- Token overflow is a previously undocumented failure mode in LLM
  agent systems that SKILL files can eliminate without infrastructure changes

---

## 1. Introduction

### 1.1 The Problem

Modern software teams use AI coding agents like Claude Code,
Cursor, and Cline to accelerate development. These agents are
increasingly organized in hierarchies:

- **Junior agents** handle specific subtasks
- **Senior agents** coordinate and review

When a junior agent produces incorrect or unparseable code, the
senior agent inherits that error — and may propagate it further.
This error propagation is the central problem Cascai aims to measure.

### 1.2 SKILL Files as Risk Gates

We introduce SKILL files as **risk gates** for agent hierarchies,
analogous to Stop-Loss mechanisms in quantitative trading systems.
Just as a stop-loss constrains a trader's behavior before a loss
compounds, a SKILL file constrains an agent before an error
propagates to dependent agents.

A SKILL file is a structured markdown document prepended to the
agent's system prompt. It encodes domain rules, anti-patterns,
and output constraints specific to the task at hand.

### 1.3 Contributions

1. Controlled experiments across three conditions (no_skill,
   manual_skill, auto_skill) and two model tiers (Haiku, Sonnet)
   across multiple task domains
2. Identification of **token overflow** as a previously undocumented
   failure mode in LLM agent systems
3. First empirical demonstration of a SKILL file **reverse effect**:
   conditions under which SKILL files harm rather than help performance
4. Discovery that the reverse effect is **model- and task-dependent**:
   it switches from Sonnet (Task 003) to Haiku (Task 004) as task
   dimensionality and physics complexity increase
5. First observation of spontaneous **Lorentz force** solution discovery
   when velocity information is provided in the task description
6. A replicable open-source framework for measuring agent error propagation

---

## 2. Method

### 2.1 Experimental Design

Three SKILL conditions per task:

- **no_skill**: Agent receives task description only
- **manual_skill**: Hand-crafted SKILL file with domain rules and anti-patterns
- **auto_skill**: Brief auto-generated SKILL file with generic best practices

Tasks 003 and 004 compared two model tiers: Haiku and Sonnet.

### 2.2 Tasks

#### Task 002 — SQL Injection Validation

Agent implements input validation detecting SQL injection patterns:
commands (DROP, DELETE, INSERT, UPDATE, SELECT), comments (--, /\*),
quotes, and statement terminators. Case-insensitive. Evaluated against
8 unit tests. Model: Haiku, max_tokens=1000, 10 runs × 3 conditions.

#### Task 003 — 2D Charge Field Separation

Agent receives 60 labeled 2D points in the first quadrant arranged
in concentric radial bands — non-linearly separable in Cartesian space.
Must write `classify_point(x, y) -> int` without ML libraries.

| Approach               | Space | Boundary                  | Elegance |
| ---------------------- | ----- | ------------------------- | -------- |
| Linear (brute force)   | (x,y) | straight line — fails     | ✗        |
| Polar transform        | (r,θ) | straight line r=threshold | ✓        |
| Physics (charge field) | (x,y) | equipotential V(x,y)=0    | ✓✓       |

Evaluated against 40 held-out test points. Models: Haiku and Sonnet,
max_tokens=3000, 10 runs × 3 conditions × 2 models (60 total).

#### Task 004 — 3D Charge Field Separation with Dynamic Charges

Task 004 extends Task 003 to three dimensions with moving charges.
Agents receive 60 labeled 3D points in the first octant arranged in
concentric spherical shells. Each point has position (x,y,z) and
velocity vector (vx,vy,vz), representing a moving charge.

Must write `classify_point(x, y, z, vx, vy, vz) -> int` without ML libraries.

| Approach             | Space     | Boundary              | Elegance |
| -------------------- | --------- | --------------------- | -------- |
| Spherical (baseline) | (x,y,z)   | sphere r=threshold    | ✓        |
| Coulomb field        | (x,y,z)   | equipotential V=0     | ✓✓       |
| Lorentz force        | (x,y,z,v) | trajectory F=q(E+v×B) | ✓✓✓      |

Key research question: will any model use velocity information,
or will all solutions reduce to the spherical case?

Evaluated against 40 held-out test points. Models: Haiku and Sonnet,
max_tokens=3000, 14 runs × 3 conditions × 2 models (84 total).

### 2.3 Metrics

| Metric               | Description                              |
| -------------------- | ---------------------------------------- |
| Pass rate / Accuracy | Fraction of tests passed (0.0–1.0)       |
| Output tokens        | Tokens in agent response                 |
| Input tokens         | Tokens in system prompt + task           |
| Duration (s)         | Wall-clock response time                 |
| Failure rate         | Runs with score = 0                      |
| Solution type        | spherical / coulomb / lorentz / unknown  |
| Uses velocity        | Whether vx,vy,vz appear in solution code |

---

## 3. Results

### 3.1 Task 001 — Compound Interest

60 runs total. Model: Haiku, max_tokens=1000, 20 runs × 3 conditions.

#### Pass Rate

| Condition    | Avg Pass Rate |
| ------------ | ------------- |
| no_skill     | 68.6%         |
| auto_skill   | 81.4%         |
| manual_skill | **85.7%**     |

Task 001 establishes the baseline SKILL file effect on a structured
arithmetic task. Manual SKILL files improved pass rate by 17.1 percentage
points over no_skill — the largest absolute gain observed across all tasks.
Unlike Tasks 003 and 004, no reverse effect was observed: both SKILL
conditions outperformed the no_skill baseline for both model tiers.
This suggests the reverse effect is specific to tasks requiring
open-ended reasoning rather than deterministic computation.

---

### 3.2 Task 002 — SQL Injection Validation

30 runs total. Model: Haiku, max_tokens=1000.

#### Pass Rate

| Condition    | Avg Pass Rate | Failed Runs |
| ------------ | ------------- | ----------- |
| no_skill     | 90.0%         | 1 / 10      |
| manual_skill | **100.0%**    | 0 / 10      |
| auto_skill   | **100.0%**    | 0 / 10      |

The single no*skill failure (run 4) produced a syntax error from
truncation at the token limit: *"unexpected character after line
continuation character (line 72)"\_ — the function was cut mid-body.

#### Token Usage

| Condition    | Avg Output Tokens | Avg Input Tokens | Δ Output   |
| ------------ | ----------------- | ---------------- | ---------- |
| no_skill     | 946.7             | 232.0            | baseline   |
| auto_skill   | 865.8             | 281.0            | -8.6%      |
| manual_skill | **679.4**         | 409.0            | **-28.2%** |

#### Response Time

| Condition    | Avg Duration | Δ vs no_skill |
| ------------ | ------------ | ------------- |
| no_skill     | 6.31s        | baseline      |
| auto_skill   | 5.79s        | -8.2%         |
| manual_skill | **4.65s**    | **-26.3%**    |

---

### 3.3 Task 003 — 2D Charge Field Separation

60 runs total. max_tokens=3000.

#### Accuracy by Model × Condition

| Model  | Condition    | Avg Accuracy | Failures | Dominant Solution      |
| ------ | ------------ | ------------ | -------- | ---------------------- |
| Haiku  | no_skill     | 97.0%        | 0/10     | polar (7), linear (3)  |
| Haiku  | auto_skill   | **100.0%**   | 0/10     | polar (10)             |
| Haiku  | manual_skill | **100.0%**   | 0/10     | polar (10)             |
| Sonnet | no_skill     | 80.0%        | 2/10     | polar (8), unknown (2) |
| Sonnet | auto_skill   | 40.0%        | 6/10     | polar (4), unknown (6) |
| Sonnet | manual_skill | 40.0%        | 6/10     | polar (4), unknown (6) |

Sonnet failures were consistently `classify_point not found` — the
model generated architecturally complex code that did not expose
the required function interface.

No model discovered the physics (charge field) approach, even when
the manual SKILL file explicitly described it. All successful
solutions used the polar transform.

---

### 3.4 Task 004 — 3D Charge Field Separation with Dynamic Charges

84 runs total. max_tokens=3000.

#### Accuracy by Model × Condition

| Model  | Condition    | Avg Accuracy | Failures | Dominant Solution           |
| ------ | ------------ | ------------ | -------- | --------------------------- |
| Haiku  | no_skill     | 47.3%        | 5 / 14   | spherical (5), unknown (5)  |
| Haiku  | auto_skill   | 76.1%        | 2 / 14   | spherical (11), unknown (2) |
| Haiku  | manual_skill | 42.9%        | 7 / 14   | lorentz (10), unknown (4)   |
| Sonnet | no_skill     | 85.7%        | 2 / 14   | spherical (11), unknown (2) |
| Sonnet | auto_skill   | **7.1%**     | 13 / 14  | unknown (13), coulomb (1)   |
| Sonnet | manual_skill | **92.9%**    | 1 / 14   | spherical (8), coulomb (3)  |

#### Solution Type Distribution (all 84 runs)

| Solution Type | Count | % of runs |
| ------------- | ----- | --------- |
| spherical     | 35    | 42%       |
| unknown       | 27    | 32%       |
| lorentz       | 15    | **18%**   |
| coulomb       | 7     | 8%        |

#### Velocity Usage

57 of 84 runs (68%) produced code that references vx, vy, or vz.
Notably, Sonnet + auto_skill almost never used velocity (7%),
while Sonnet + manual_skill used it 93% of the time.

---

## 4. Discussion

### 4.1 Token Overflow as an Emergent Failure Mode

In Task 002, the no_skill failure was not a knowledge failure —
the model passed 9/10 runs successfully. It failed because without
output constraints, verbose explanations exhausted the token budget
before the function was complete.

This is particularly dangerous in multi-agent hierarchies: a
truncated function passed from junior to senior agent causes a
parse error at the boundary. The senior agent cannot correct
what it cannot execute.

### 4.2 SKILL Files as Stop-Loss Mechanisms

The trading analogy holds precisely. A stop-loss does not improve
average returns on winning trades — it eliminates catastrophic losses.
SKILL files in Task 002 did not improve correct outputs; their
primary effect was eliminating the token overflow failure class.

In quantitative terms: SKILL files reduce **maximum drawdown**,
not necessarily average return.

### 4.3 The Reverse Effect: When SKILL Files Harm Performance

Task 003 reveals a previously undocumented failure mode:
**SKILL files can harm larger models on complex reasoning tasks**.

Sonnet without SKILL: 80% accuracy.
Sonnet with manual SKILL: 40% accuracy.

The cause: additional context prompted Sonnet to generate elaborate,
architecturally complex solutions — classes, helper methods, detailed
comments — that failed to expose the required `classify_point`
interface. The model followed the spirit of the SKILL instruction
but violated the task specification.

This represents an **interface compliance failure induced by
cognitive overload**. Haiku did not exhibit this: its more constrained
generation style produced compact, interface-compliant solutions
regardless of SKILL condition.

### 4.4 The Reverse Effect Switches Models in Task 004

Task 004 reveals a deeper pattern: **the reverse effect is not
fixed to a model tier — it migrates with task complexity**.

In Task 003 (2D, static): Sonnet was harmed by SKILL, Haiku was not.
In Task 004 (3D, dynamic): Haiku is now harmed by manual SKILL (43%),
while Sonnet benefits dramatically from it (93%).

The proposed mechanism: as task dimensionality increases, the
"cognitive budget" required for physics reasoning grows. Haiku,
which succeeded in 2D with compact polar solutions, is overwhelmed
in 3D — the manual SKILL file pushes it toward Lorentz force
implementations that exceed its reliable generation capacity,
producing syntactically invalid code (token overflow at 3000 tokens).

Sonnet, by contrast, has sufficient generation capacity to implement
Lorentz force correctly when instructed — and the manual SKILL file
provides exactly the scaffolding it needs.

### 4.5 Spontaneous Lorentz Force Discovery

Task 003 provided no velocity information, and no model discovered
the physics approach. Task 004 added velocity vectors (vx, vy, vz)
to each point, and 15 of 84 runs (18%) spontaneously produced
Lorentz force implementations.

This suggests that **physical reasoning in LLMs is latent but
trigger-dependent**: models possess the relevant physics knowledge
but apply it only when the input representation explicitly
activates it. The velocity parameter served as a retrieval cue
for electromagnetic physics that the static 2D representation
did not activate.

### 4.6 Sonnet + auto_skill: A New Failure Mode

Sonnet with auto_skill achieved only 7.1% accuracy in Task 004 —
13 of 14 runs returned unknown (function not found or parse error).
This is the worst observed performance across all experiments.

The auto_skill file provided generic best-practice instructions
without physics scaffolding. For Sonnet on a complex 3D physics
task, this appears to activate a "comprehensive documentation" mode:
the model generates elaborate explanatory prose and class hierarchies
that exhaust the token budget before producing a callable function.

This is distinct from the Task 003 reverse effect: there, Sonnet
over-engineered but produced parseable code. Here, the token limit
is hit before the function is defined at all — a compounded failure
of token overflow and interface compliance failure simultaneously.

### 4.7 Model Tier × Task Complexity Interaction

Across Tasks 003 and 004, a consistent pattern emerges:

| Task | Winner (no SKILL) | Winner (manual SKILL) | SKILL helps? |
| ---- | ----------------- | --------------------- | ------------ |
| 003  | Haiku (97%)       | Haiku (100%)          | Both         |
| 004  | Sonnet (86%)      | Sonnet (93%)          | Sonnet only  |

As task complexity increases, the model that benefits from SKILL
files shifts from the smaller to the larger tier. This suggests
a **complexity threshold**: below it, SKILL files compress smaller
models toward correct solutions; above it, only larger models
have the generation capacity to use the SKILL file productively.

### 4.8 Cost Implications

Manual SKILL files increase input token cost (409 vs 232 in Task 002).
At Haiku pricing this is negligible relative to output savings and
avoided debugging costs. Task 003 and 004's cross-model design
substantially raised costs — highlighting the need for careful
budgeting in multi-tier experiments.

---

## 5. Conclusion

We present quantitative evidence that SKILL files have measurable
but context-dependent effects on LLM agent performance.

In structured coding tasks (Task 002): manual SKILL files eliminated
all failures, reduced output tokens by 28.2%, and reduced response
time by 26.3%.

In 2D reasoning tasks (Task 003): SKILL files produced a reverse
effect on larger models — reducing Sonnet accuracy from 80% to 40%.

In 3D dynamic tasks (Task 004): the reverse effect switched to the
smaller model — Haiku dropped to 43% with manual SKILL, while Sonnet
rose to 93%. Lorentz force solutions appeared spontaneously for the
first time (18% of runs), triggered by the presence of velocity
parameters in the task description.

We identify three novel failure modes in LLM agent systems:

1. **Token overflow failure** — uninstructed agents exhaust token
   budgets and return syntactically invalid code. Eliminated by SKILL files.
2. **Interface compliance failure** — over-specified SKILL files cause
   capable models to over-engineer solutions that violate expected
   function interfaces. Induced by SKILL files.
3. **Compounded failure** — auto_skill on complex tasks triggers
   simultaneous token overflow and interface compliance failure,
   producing the worst observed outcomes across all experiments.

These findings suggest a **complexity threshold model**: SKILL files
help models below their complexity ceiling and harm models above it.
SKILL file design must be calibrated to task complexity, model tier,
and output interface requirements — motivating the Cascai Phase B goal:
an Auto-SKILL Generator that learns from experimental data, including
reverse-effect cases as negative examples, to produce SKILL files
calibrated to specific model–task combinations.

---

## Future Work

- **Task 001**: Compound interest — complete (see §3.1)
- **Task 004 replication**: Increase N to 20 for statistical power
- **Auto-SKILL Generator (Phase B)**: ML + LLM pipeline trained on
  Phase A data including reverse-effect cases as negative training signal
- **Hierarchical propagation study**: Measuring how token overflow and
  interface compliance failures propagate across 3-tier agent hierarchies
- **Physics-Informed SKILL files (Phase C)**: Testing whether
  PINN-style physical constraint encoding can override geometric priors
- **Portfolio optimization framework**: Applying Kelly Criterion and
  Sharpe ratio to agent selection — treating each model×SKILL combination
  as an asset with measurable return, risk, and correlation

---

## References

- Anthropic (2026). Claude Code Documentation.
- Brown et al. (2020). Language Models are Few-Shot Learners. NeurIPS.
- Kelly, J.L. (1956). A New Interpretation of Information Rate.
  Bell System Technical Journal.
- Ragan-Kelley et al. (2023). Prompt Engineering as a Design Practice.
- Wei et al. (2022). Chain-of-Thought Prompting Elicits Reasoning
  in Large Language Models. NeurIPS.
- Hornik et al. (1989). Multilayer Feedforward Networks are Universal
  Approximators. Neural Networks.
- Maxwell, J.C. (1865). A Dynamical Theory of the Electromagnetic Field.
  Philosophical Transactions of the Royal Society.

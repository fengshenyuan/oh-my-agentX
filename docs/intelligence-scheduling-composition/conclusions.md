# Intelligence Scheduling / Composition

## Conclusions

### 1. Intelligence is not a single scalar

For agentic systems, “intelligence” should not be reduced to one model's benchmark score. A useful system can compose multiple intelligence sources: general models, specialist models, reasoning compute, search, tools, compilers, tests, verifiers, other agents, and humans.

The final intelligence of the system is therefore a **system property produced by composition and orchestration**.

### 2. Model capability and test-time compute are different dimensions

A useful mental model is:

- **Luna → Terra → Sol:** primarily changes the capability/intelligence base.
- **Low → Medium → High → xHigh:** primarily changes the amount of test-time computation available to that capability base.

They can compensate for each other over some task distributions, but they are not equivalent.

This explains why Sol Low can substantially outperform Luna Medium: Sol may have a stronger capability base and therefore recognize, discriminate, or solve important aspects of a task that Luna Medium misses, even with less reasoning budget.

Conversely, Terra High can sometimes outperform Sol Low when the task is within Terra's capability ceiling but benefits from additional search, planning, or verification.

Therefore similar Artificial Analysis Intelligence Index scores for configurations such as Sol Low and Terra High should be understood as **aggregate benchmark equivalence**, not as evidence that their underlying intelligence or task-level behavior is identical.

### 3. The key diagnostic is capability gap vs. reasoning gap

When an agent struggles, distinguish:

- **Capability gap:** the model cannot reliably identify the important issue, understand the problem, or find the right conceptual path. Escalate to a stronger capability base.
- **Reasoning gap:** the model has the right conceptual direction but needs more computation, search, planning, or verification. Increase reasoning effort.
- **Search runaway:** the model already has sufficient evidence but continues low-value exploration. Do not increase compute; stop or redirect it.

This is more useful than classifying tasks simply as “easy” or “hard.”

### 4. Manual model/effort selection is an intermediate UX, not the end state

Users should not have to act as an inference scheduler while simultaneously doing their engineering work.

The desired product behavior is closer to an automatic transmission:

```text
Task
 ↓
cheap/fast intelligence
 ↓
observe evidence and uncertainty
 ↓
allocate more computation when useful
 ↓
escalate capability when necessary
 ↓
invoke independent verification/tools
 ↓
correct mistakes
 ↓
stop when further computation has low expected value
```

The user should express intent and constraints; the runtime should handle most model and reasoning allocation.

### 5. The real problem is broader than model routing

“Which model should answer this request?” is only the first layer.

The deeper problem is:

> **Given the current task and state, what intelligence should be invoked next, from which source, for how much computation, in what sequence, with what feedback, and when should the system stop?**

This is the proposed meaning of **Cognitive Scheduler**.

### 6. Cognitive Scheduler = intelligence allocation and composition

A Cognitive Scheduler should be understood as an orchestrator of heterogeneous intelligence, not merely as a model router.

Possible sources include:

```text
Luna / Terra / Sol
Other model providers
Specialist models
Search
Compiler / static analyzer
Tests
Browser / external tools
Verifiers
Other agents
Human expertise
```

The scheduler determines:

- **Source:** where intelligence comes from.
- **Allocation:** how much compute is spent.
- **Sequencing:** which source acts before which other source.
- **Feedback:** what evidence changes the next decision.
- **Escalation:** when to move to stronger or more specialized intelligence.
- **Verification:** when independent evidence is required.
- **Termination:** when the expected value of further computation is too low.

### 7. Do not optimize for perfect individual inferences

A robust intelligence system should be **designed with model error** rather than pretending it can eliminate it.

The objective should be closer to:

> **Maximize the probability and quality of the final outcome under bounded time, cost, and reliability constraints.**

Not:

> Make every individual model call correct.

Fast, cheap, imperfect attempts can be valuable when mistakes are detected quickly and corrected by independent intelligence.

A desirable error-recovery loop is:

```text
attempt
 ↓
possible mistake
 ↓
independent evidence
 ↓
error detection
 ↓
correction / reallocation
 ↓
verification
 ↓
continue or stop
```

This mirrors effective human collaboration: individual judgments are imperfect, but the system can use review, specialization, experiments, escalation, and feedback.

### 8. Agent intelligence is a system property

A stronger model does not automatically produce a stronger agent.

For example:

```text
Sol xHigh
→ one highly capable model
→ may spend excessive compute on a bad hypothesis

vs.

Luna Low
→ Terra Medium
→ test / compiler
→ Sol Low
→ verification
```

The second system can potentially be faster, cheaper, and more reliable because it uses multiple independent intelligence sources and feedback loops.

Therefore:

> **Agent intelligence ≠ model intelligence.**

A better formulation is:

> **Agent intelligence = intelligence sources + orchestration + feedback + verification + error recovery.**

### 9. A likely architectural progression

```text
Level 1 — Model selection
Task → Model

Level 2 — Adaptive inference
Task/state → Model + reasoning budget

Level 3 — Model routing
Task/state → among multiple models

Level 4 — Cognitive scheduling
Task/state → composition of heterogeneous intelligence sources

Level 5 — Self-organizing intelligence system
Dynamic composition of models, reasoning, tools, verification, and humans
```

The working hypothesis is that the industry in 2026 is moving from Level 2/3 toward Level 4.

### 10. Implication for oh-my-agentX

This suggests an important architectural layer beyond configuration and basic harness/runtime concerns:

> **Intelligence Scheduling / Composition**

The long-term goal should not be to expose an ever-growing collection of model and reasoning knobs to users. Instead, the runtime should progressively absorb those decisions and turn heterogeneous intelligence into a coherent product capability.

In this view, a future Agent Runtime / Inference OS needs an **Intelligence Scheduler** that can treat models, reasoning budgets, tools, verifiers, and human escalation as composable computational resources.

The user-facing abstraction should eventually be the desired outcome and constraints—not the exact intelligence primitive used to obtain it.

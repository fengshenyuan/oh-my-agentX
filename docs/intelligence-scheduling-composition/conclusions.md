# Intelligence Scheduling / Composition

> **Core thesis:** The next major abstraction above model routing is not “which model should answer this prompt?”, but **how to compose heterogeneous sources of intelligence into the best final intelligence product/outcome**.
>
> A future Agent Runtime / Inference OS should therefore treat models, reasoning budgets, tools, verifiers, external knowledge, other agents, and human escalation as composable intelligence resources, and dynamically allocate them according to the task, evidence, constraints, and expected value of further computation.

---

## 1. The core problem

Today's frontier coding agents expose a growing matrix of choices:

```text
model × reasoning effort × harness × tools
```

For example, a user may have to choose between different model families and Low / Medium / High / xHigh reasoning modes.

This is a transitional UX. It makes the human act as a low-level **inference scheduler**.

The user should not have to answer questions such as:

- Is this task difficult enough for the strongest model?
- Is the model currently suffering from insufficient capability or insufficient reasoning budget?
- Should I increase thinking or switch models?
- Should I let the agent continue exploring or interrupt it?
- Should I ask another model to review the result?

Those are runtime-level decisions.

The long-term product should look more like an **automatic transmission** than a manual gearbox:

```text
User
  │
  │ desired outcome + constraints
  ▼
┌──────────────────────────────┐
│   Intelligence Scheduler     │
└──────────────┬───────────────┘
               │
      dynamically allocates
               │
   ┌───────────┼────────────┐
   ▼           ▼            ▼
 Models     Reasoning      Tools
   │          budget          │
   └───────────┬────────────┘
               ▼
          Evidence / State
               │
               └──────► Scheduler
                         (repeat)
               │
               ▼
            Outcome
```

The user asks for the outcome. The runtime decides how much and what kind of intelligence to spend.

---

## 2. The fundamental conceptual shift: from Model Routing to Intelligence Composition

### Model routing asks

> Which model should handle this request?

This is already useful, but it is only one dimension.

### Intelligence Scheduling asks

> **Given the task and current state, what intelligence should be invoked next, from which source, for how much computation, in what sequence, with what feedback, and when should the system stop?**

The intelligence source need not be an LLM.

It can be:

```text
General model
Specialist model
Different provider
Different reasoning tier
Search
Browser
Compiler
Static analyzer
Test suite
Verifier
Other agent
Human expert
```

Therefore the final product is not necessarily “the answer produced by model X.”

It is a **composition of heterogeneous intelligence sources**.

This is the central reason the term **Intelligence Scheduling / Composition** is preferable to simply “model routing.”

---

## 3. Intelligence is a system property, not just a model property

A benchmark score measures the capability of a model/configuration over a particular task distribution. It does not fully characterize the intelligence of an agent system.

A stronger model can still produce a weaker agent if it:

- locks onto a bad hypothesis,
- spends excessive compute defending it,
- lacks independent verification,
- explores irrelevant parts of the environment,
- or has no mechanism for recognizing that further computation has low value.

Conversely, a system composed from several weaker/cheaper steps can outperform a single very strong model if it has better:

- decomposition,
- specialization,
- verification,
- error detection,
- escalation,
- and stopping behavior.

A useful system-level formulation is:

> **Agent intelligence = intelligence sources + orchestration + feedback + verification + error recovery.**

This is a system property rather than a scalar property of one model.

---

## 4. The most important design principle: Design for Model Error

### Do not design around the assumption of perfect inference

A model will sometimes:

- choose the wrong hypothesis,
- misunderstand the task,
- select the wrong tool,
- overthink an irrelevant issue,
- underthink a subtle issue,
- or make an incorrect routing decision.

Trying to make every individual inference call perfect creates systems that are expensive, slow, and overly conservative.

The more productive principle is:

> **Design with model error.**

Allow cheap, fast, imperfect attempts when the system has mechanisms that can expose and correct mistakes.

### The optimization target changes

Do not optimize for:

> `P(each individual inference is correct)`

Optimize for something closer to:

> **`P(final outcome is correct | bounded time, cost, and reliability constraints)`**

A desirable loop is:

```text
Attempt
  ↓
Possible error
  ↓
Independent evidence
  ↓
Error detection
  ↓
Correction / reallocation
  ↓
Verification
  ↓
Continue or stop
```

This is analogous to effective human collaboration. A team does not require every engineer to be correct on the first attempt; it uses review, experiments, specialization, escalation, and feedback.

### Why cheap mistakes can be an advantage

Consider two strategies:

```text
A: one extremely expensive attempt
   → try to be perfect

B: several cheap intelligence calls
   → make an inexpensive hypothesis
   → verify
   → correct
   → escalate only when needed
```

Strategy B can be faster, cheaper, and more reliable even when individual calls are less capable, because **error recovery becomes part of the architecture**.

The goal is not to eliminate mistakes. It is to make mistakes:

> **cheap to make, fast to detect, and reliable to recover from.**

---

## 5. Human collaboration is the right analogy

A strong human engineering organization does not route every problem to its smartest engineer and wait for perfect certainty.

A typical pattern is closer to:

```text
Engineer A
  ↓
initial investigation
  ↓
Engineer B
  ↓
review / challenge
  ↓
Specialist
  ↓
solve a local uncertainty
  ↓
Experiment / test
  ↓
objective evidence
  ↓
Staff / architect
  ↓
final synthesis
```

The system works because different forms of intelligence are **composed**.

The important property is not that every participant is perfect. It is that the organization has a methodology for:

- exposing uncertainty,
- challenging assumptions,
- obtaining independent evidence,
- escalating when necessary,
- and converging on a reliable outcome.

A future agent system can implement the same principle with models and tools.

---

## 6. “Automatic transmission” is the right product metaphor

Today:

```text
User
 ↓
Choose model
 ↓
Choose reasoning effort
 ↓
Observe agent
 ↓
Maybe switch model
 ↓
Maybe increase effort
 ↓
Maybe interrupt
```

This is a large cognitive burden.

The desired future:

```text
User
 ↓
Describe task
 ↓
Runtime
 ├─ starts cheap/fast
 ├─ escalates capability when necessary
 ├─ increases reasoning when useful
 ├─ calls independent verifiers
 ├─ tries another intelligence source after failure
 ├─ stops when marginal value becomes low
 ↓
Outcome
```

The model/effort matrix should progressively become an internal implementation detail of the runtime.

The user should express **intent, constraints, and acceptable tradeoffs**, not micromanage the intelligence machinery.

---

## 7. Why “adaptive computation” is not the final abstraction

A common current formulation is:

> Monitor the agent trajectory and dynamically increase/decrease reasoning or switch models based on the current state.

This is useful, but it should be understood as **one implementation strategy**, not the definition of the problem.

The deeper abstraction is not “monitor one agent better.”

It is:

> **Compose multiple intelligence sources into a reliable final intelligence product.**

Trajectory monitoring may be one signal available to the scheduler. Other signals include:

- test results,
- compiler errors,
- static-analysis findings,
- external search results,
- conflicting model opinions,
- tool failures,
- latency/cost budget,
- explicit user constraints,
- and independent verification.

A scheduler can therefore make decisions without assuming that the primary model itself is a trustworthy omniscient monitor.

---

## 8. What the Cognitive Scheduler actually schedules

A true Cognitive Scheduler should control at least seven dimensions.

### 8.1 Source

Which intelligence source should act?

```text
Luna / Terra / Sol
Other provider
Specialist model
Search
Verifier
Tool
Human
```

### 8.2 Allocation

How much compute or effort should be spent?

```text
Low → Medium → High → xHigh
```

or more generally any continuous/structured compute budget.

### 8.3 Sequencing

Which source should act first, and which should follow?

### 8.4 Diversity

Should the next intelligence source be similar to the previous one, or intentionally independent to reduce correlated failure?

This is important because calling the same model repeatedly is not necessarily an effective verification strategy.

### 8.5 Verification

When should the system obtain an independent correctness signal?

Examples:

```text
compiler
unit test
integration test
static analyzer
second model
formal checker
human review
```

### 8.6 Escalation

When should the system move from cheap intelligence to stronger or more specialized intelligence?

### 8.7 Termination

When is the expected value of another intelligence call too low to justify its cost/latency/risk?

This last dimension is particularly important for avoiding the **Sol xHigh scanning `.local` forever** failure mode discussed in the original conversation.

---

## 9. The key economic principle: Intelligence is a resource

The scheduler should treat intelligence as a resource that can be allocated, just as an operating system allocates compute resources.

This makes the CPU-scheduler analogy particularly useful.

A programmer does not normally decide:

> “This thread deserves exactly 13.7 ms of CPU time, then move it to a larger core.”

The OS schedules execution based on system policy and observed state.

Similarly, a future Agent Runtime should decide:

```text
which model
+ which reasoning depth
+ which tool
+ which verifier
+ how much compute
+ in what order
```

based on the system's current state.

The model pool becomes analogous to heterogeneous compute resources.

---

## 10. Model capability and reasoning effort remain useful primitives

This framework does **not** make model capability or reasoning effort irrelevant.

They become internal resources rather than primary user-facing decisions.

A useful mental model is:

```text
Model family
→ capability ceiling / capability profile

Reasoning effort
→ available test-time computation

Tools / verifiers
→ external evidence and correctness signals

Multiple providers
→ diversity / specialization / redundancy

Scheduler
→ composition and allocation
```

This explains the practical observation that a stronger model at Low effort can outperform a weaker model at Medium/High effort on some tasks, while the weaker model at higher effort can still win on other tasks.

Those are not contradictions. They are different points in a multidimensional intelligence resource space.

---

## 11. The architectural progression

The discussion suggests the following progression:

```text
Level 1 — Fixed model
Task → Model

Level 2 — Adaptive inference
Task/state → Model + reasoning budget

Level 3 — Model routing
Task/state → choose among models

Level 4 — Intelligence Scheduling / Composition
Task/state → compose models + reasoning + tools + verifiers

Level 5 — Self-organizing intelligence system
Dynamic composition of models, tools, verification,
other agents, external knowledge, and humans
```

The important transition is **Level 3 → Level 4**.

At Level 3 the product says:

> “We know which model to call.”

At Level 4 it says:

> “We know how to build the intelligence required for this outcome.”

That is a much more powerful abstraction.

---

## 12. Current industry state — 2026

The industry has clearly recognized the problem, but most commercial systems are still early forms of automatic routing rather than full Cognitive Scheduling.

### Cursor Router

Cursor launched Cursor Router in July 2026. Its Auto mode analyzes each request and routes it to a model appropriate for the task. It exposes Intelligence / Balance / Cost optimization objectives and describes the router as moving users along a cost–intelligence Pareto frontier. Cursor reports results from production traffic and millions of requests.

This is an important product-level example of the transition from manual model selection to automatic routing.

Reference: [Cursor Router](https://cursor.com/blog/router)

### Claude Code

Claude Code already contains several forms of automatic intelligence allocation:

- `opusplan`: Opus for planning and Sonnet for execution.
- Adaptive reasoning/effort: the model can decide whether and how much to think at each step within the configured effort level.

This demonstrates that model selection and reasoning allocation are already becoming runtime behavior, but the system is still exposed to users through explicit model/effort controls and predefined strategies.

Reference: [Claude Code model configuration](https://code.claude.com/docs/en/model-config)

### OpenRouter

OpenRouter's Auto Router is an explicit example of a meta-router: it analyzes a prompt and selects among a pool of models. Its newer Auto Beta approach uses task classification and aggregate real-world usage signals to rank candidate models, with a cost/quality tradeoff control.

This demonstrates that the “automatic transmission” idea is already commercially useful at the request-routing level.

References:

- [OpenRouter Auto Router](https://openrouter.ai/openrouter/auto)
- [OpenRouter Auto Router documentation](https://openrouter.ai/docs/guides/routing/routers/auto-router)

### What is still missing

These systems mostly solve variants of:

```text
request → choose model
```

or:

```text
agent phase → choose model
```

The larger unsolved problem is:

```text
long-running task
→ continuously compose heterogeneous intelligence
→ detect errors
→ verify independently
→ escalate / diversify / retry
→ allocate compute dynamically
→ terminate
```

That is the level at which **Intelligence Scheduling / Composition** becomes a distinct systems problem.

---

## 13. Academic state — 2025–2026

The research community has several adjacent but increasingly converging lines of work.

### 13.1 Adaptive test-time compute

The test-time-compute literature explicitly studies the problem that fixed reasoning budgets cause both **overthinking simple problems** and **underthinking hard problems**.

A 2025 survey, *Reasoning on a Budget: A Survey of Adaptive and Controllable Test-Time Compute in LLMs*, organizes work around controllability and adaptiveness of test-time compute.

Reference: [arXiv:2507.02076](https://arxiv.org/abs/2507.02076)

### 13.2 Adaptive reasoning effort

*Ares: Adaptive Reasoning Effort Selection for Efficient LLM Agents* studies per-step dynamic reasoning effort in multi-step agents. Instead of assigning one fixed effort level to the entire task, it predicts the lowest appropriate reasoning level for each step based on interaction history. The reported results show substantial reasoning-token reductions relative to fixed high-effort strategies while maintaining similar task success.

Reference: [Ares, arXiv:2603.07915](https://arxiv.org/abs/2603.07915)

This is an important step toward automatic transmission, but it still primarily treats the problem as **adaptive reasoning allocation**.

### 13.3 General LLM routing

*LLMRouterBench: A Massive Benchmark and Unified Framework for LLM Routing*, Findings of ACL 2026, evaluates 33 models across more than 400K instances and 21 datasets.

Its most important result for this discussion is not simply that routing works. It finds strong model complementarity, but also that many routing methods are similar in performance, some commercial routers do not reliably outperform simple baselines, and a substantial gap remains to an oracle router, driven particularly by model-recall failures.

Reference: [LLMRouterBench, ACL 2026](https://aclanthology.org/2026.findings-acl.1881/)

This is evidence that automatic routing is useful but still far from solved.

### 13.4 Agentic / step-level routing

*TwinRouterBench: Fast Static and Live Dynamic Evaluation for Realistic Agentic LLM Routing* makes an important conceptual move: long-horizon agents such as coding agents issue many model calls, so routing should be evaluated at the **individual step / trajectory** level rather than only on the initial user prompt.

It includes SWE-bench-based evaluation and tests whether cheaper model substitutions preserve downstream task success.

Reference: [TwinRouterBench, arXiv:2605.18859](https://arxiv.org/abs/2605.18859)

This is much closer to the Cognitive Scheduler problem, although it is still primarily framed as model routing.

---

## 14. The important research gap

The academic literature can already answer several narrower questions:

```text
Can we allocate reasoning adaptively?        → yes, partially
Can we route among models?                  → yes, partially
Can we route per agent step?                → yes, emerging
Can we benchmark routing?                   → increasingly
```

The broader question is harder:

> **Can we dynamically compose heterogeneous intelligence sources so that the final system is more reliable and efficient than any single model, while allowing individual intelligence calls to be imperfect?**

That requires jointly reasoning about:

```text
model capability
reasoning budget
model diversity
verification
tool evidence
trajectory state
error correlation
latency
cost
risk
termination
human escalation
```

This is closer to a **systems / runtime / scheduling problem** than to ordinary prompt engineering.

---

## 15. The core principle in one sentence

> **Do not build an intelligence system that assumes every inference must be correct; build an intelligence system in which mistakes are cheap, errors are quickly exposed, independent intelligence can correct them, and the scheduler continuously reallocates intelligence toward the final outcome.**

This is the central principle behind **Intelligence Scheduling / Composition**.

---

## 16. What this means for oh-my-agentX

The concept fits naturally into the broader oh-my-agentX architecture.

A simplified evolution is:

```text
                    oh-my-agentX
                         │
          ┌──────────────┼──────────────┐
          ▼              ▼              ▼
   Configuration      Harness       Runtime
                                       │
                                       ▼
                              Intelligence Scheduler
                                       │
                 ┌─────────────────────┼─────────────────────┐
                 ▼                     ▼                     ▼
              Models              Tools / I/O          Verification
                 │                     │                     │
                 └─────────────────────┼─────────────────────┘
                                       ▼
                                  Agent State
                                       │
                                       └──────────────► Scheduler
```

The existing ideas around shared configuration and harness interoperability answer:

> **How should agents be configured and operated consistently across clients?**

Intelligence Scheduling / Composition adds a deeper runtime question:

> **How should the runtime dynamically construct the intelligence needed to accomplish the task?**

This suggests a future **Intelligence OS / Agent Runtime** in which models are not the product boundary. They are interchangeable intelligence resources behind a scheduler.

The user-facing abstraction should increasingly become:

```text
Goal
Constraints
Quality requirement
Latency preference
Risk tolerance
Human-in-the-loop policy
```

rather than:

```text
Model = X
Reasoning = Y
```

---

## 17. Open research questions

The concept raises a number of concrete research problems for oh-my-agentX:

1. **Intelligence valuation** — How do we estimate the expected value of another intelligence call before making it?
2. **Capability profiles** — How do we represent model capabilities beyond a single benchmark score?
3. **Error correlation** — When is a second model genuinely independent evidence rather than the same failure reproduced twice?
4. **Verification selection** — Which verifier is appropriate for which uncertainty?
5. **Escalation policy** — When should the scheduler switch model families versus increase reasoning effort?
6. **Diversity scheduling** — When should the scheduler intentionally choose a different provider/model to obtain independent reasoning?
7. **Trajectory state** — What compact state representation is sufficient for scheduling without exploding context?
8. **Termination** — How can a runtime detect diminishing returns and avoid runaway exploration?
9. **Budgeting** — How should cost, latency, reliability, and user patience be combined into an objective function?
10. **Human escalation** — When is a human the highest-value intelligence source?
11. **Learning the scheduler** — Can the scheduler learn from historical task outcomes, interventions, verification results, and real-world success rather than relying on hand-authored rules?
12. **State / ABI** — What runtime-level representation allows different model providers and intelligence sources to participate in a common agent state without being tightly coupled?

These questions are likely more fundamental to the next generation of agent runtimes than another round of model-picker UI improvements.

---

## 18. Final position

The key insight is not:

> “Users need to become better at choosing Sol vs. Terra vs. Luna.”

That is a transitional problem.

The deeper insight is:

> **Users should eventually not need to choose at all.**

The system should turn a pool of imperfect, heterogeneous intelligence sources into a coherent, reliable intelligence product through scheduling, composition, feedback, verification, escalation, and error recovery.

In that sense, **Intelligence Scheduling / Composition is not merely a feature of an agent harness. It is a candidate core abstraction of the next-generation Agent Runtime / Inference OS.**

The likely end state is not “one perfect model.”

It is:

> **a system that knows how to spend intelligence.**

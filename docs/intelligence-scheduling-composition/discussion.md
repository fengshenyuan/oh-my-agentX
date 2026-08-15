# Intelligence Scheduling / Composition

## Original Discussion

### 1. The practical problem: choosing model × thinking effort

We started from a concrete daily-development problem: among GPT-5.6 Luna, Terra, and Sol, each with Low / Medium / High / xHigh reasoning effort, how should a developer choose?

A representative example was a Spring Boot application deployed in Kubernetes where the final ~1 second of logs could be lost during Pod termination. The problem initially looked difficult, so Sol + xHigh was enabled in Codex. The agent then entered an extremely long exploratory reasoning loop: it tried to inspect and verify almost everything, including scanning `.local` / Maven-related state for marginal clues. The user felt that the agent should have been interrupted rather than allowed to pursue theoretical completeness.

The key observation was that **thinking effort is not equivalent to task difficulty**. A task can be objectively small but have high uncertainty; conversely, a large code change can be mechanically straightforward.

### 2. Intelligence Index and the apparent equivalence of different configurations

We examined Artificial Analysis's Intelligence Index and noticed configurations such as:

- Sol + Low
- Sol + Medium
- Terra + High
- Terra + xHigh

can cluster relatively closely in aggregate intelligence/cost space.

The important question was: if Luna Medium scores substantially below Sol Low, does that represent a real intelligence difference, rather than merely a difference in inference cost or thinking time?

The conclusion reached was: **yes, broadly speaking**. The benchmark gap should be interpreted as evidence that, over the benchmark distribution, Sol Low can reliably solve a class of problems that Luna Medium is more likely to fail on. It should not be interpreted as a literal universal IQ-like scalar or as saying Sol Low dominates Luna Medium on every individual task.

This led to a useful distinction:

- Luna → Terra → Sol primarily changes the **capability/intelligence base**.
- Low → Medium → High → xHigh primarily changes the **test-time compute / reasoning budget** available to a given capability base.

The two dimensions interact, but they are not interchangeable.

### 3. Capability gap vs. reasoning gap

We then reframed the practical question from “How difficult is this task?” to:

> Does the agent lack the capability to recognize/solve the problem, or does it have the capability but need more computation to fully realize it?

This produced three useful cases:

1. **Capability gap** — the model fails to notice the key issue, misunderstands the problem, or locks onto an incorrect conceptual direction. Increasing thinking on the same weaker model may not solve the problem. Moving from Terra to Sol, for example, can be more useful.
2. **Reasoning gap** — the model has found the right direction but needs more search, planning, verification, or reasoning depth. Moving Low → Medium → High can help.
3. **Search runaway** — the model already has a plausible, verified solution but continues exploring marginal or unrelated possibilities. Increasing reasoning is counterproductive; the correct action is to stop/intervene.

This explains why Sol Low can outperform Luna Medium: the stronger capability base may recognize and discriminate the important hypothesis within a small reasoning budget, whereas the weaker model may spend more time reasoning around the wrong hypothesis.

### 4. Why not simply use Sol Low for everything?

The next question was the important crossover problem: if Sol Low has a much stronger intelligence base, why not always use Sol Low?

The answer is that Sol Low and higher-effort configurations can fail in different ways. Sol Low may have the capability to solve a problem but insufficient test-time compute to explore competing hypotheses, verify a subtle conclusion, or complete a long reasoning chain.

Conversely, Terra High can sometimes match or exceed Sol Low when the task is within Terra's capability ceiling but benefits substantially from additional reasoning/search.

Therefore the configurations can have similar aggregate benchmark scores while having different **task-distribution profiles**. Aggregate Intelligence Index equivalence does not imply equivalence on every real-world task.

### 5. The user-experience problem: manual transmission

We then questioned whether requiring users to manually select model and thinking effort is itself a product-design failure.

The analogy was a manual-transmission car: users should not need to repeatedly reason about model capability and inference budget while trying to solve a software problem.

An ideal agent should instead do something like:

```text
User gives task
  ↓
Agent/runtime starts with inexpensive intelligence
  ↓
Observes uncertainty and evidence
  ↓
Allocates more reasoning when useful
  ↓
Escalates to a stronger model when capability is insufficient
  ↓
Uses tools/verifiers/tests as independent evidence
  ↓
Stops when the expected value of further computation is low
```

Model choice and reasoning allocation should eventually become runtime decisions rather than user-facing gear shifting.

### 6. Industry and academic state

We then examined the state of the industry and research direction. The industry already has early forms of automatic transmission:

- Cursor has introduced model routing that automatically chooses models based on request/context/task characteristics and routing objectives.
- Claude Code has strategies such as `opusplan`, which use a stronger model for planning and a different model for execution, and Claude's adaptive thinking/effort mechanisms.
- Research systems increasingly study adaptive test-time compute, dynamic reasoning effort, model routing, and agent/trajectory-level routing.

However, current products are mostly still at the level of **model routing / adaptive inference**, rather than a general intelligence scheduler that dynamically composes many forms of intelligence throughout an agent trajectory.

### 7. The deeper reframing: Cognitive Scheduler

The discussion then moved beyond “automatic model selection.” The deeper abstraction was called **Cognitive Scheduler**.

A Cognitive Scheduler should not merely ask:

> Which model should answer this prompt?

It should ask:

> Given the current task, state, evidence, uncertainty, constraints, and available resources, what intelligence should be invoked next, from which source, for how much computation, in what sequence, and when should the system stop?

The possible intelligence sources are broader than LLMs:

- Luna / Terra / Sol
- models from different providers
- specialist models
- search
- compiler
- static analyzer
- tests
- browser/tool execution
- verifiers
- other agents
- humans

Thus the final intelligence product is not simply a model response. It is a **composition of heterogeneous intelligence sources**.

### 8. Error should be designed into the system

A key correction to the earlier framing was that the scheduler should not assume the model must make a perfect decision at every step.

Models will make mistakes. That is acceptable if the system is designed around:

```text
cheap/fast attempt
  ↓
error or uncertainty
  ↓
independent evidence / verification
  ↓
correction
  ↓
reallocation of intelligence
  ↓
verification
```

The optimization target should therefore be the **final outcome**, not the correctness of every individual inference call.

This is analogous to human collaboration: no participant is perfectly reliable, but teams can use review, experiments, specialization, escalation, and feedback to detect and correct mistakes.

### 9. Final conceptual shift

The final discussion distinguished several levels:

```text
Level 1: Model selection
Task → Model

Level 2: Adaptive inference
Task/state → Model + reasoning budget

Level 3: Model routing
Task/state → among multiple models

Level 4: Cognitive scheduling
Task/state → compose multiple intelligence sources

Level 5: Self-organizing intelligence system
Dynamic composition of models, reasoning, tools, verification, and humans
```

The hypothesis is that the industry is currently moving from Level 2/3 toward Level 4.

The core idea is therefore not “build a smarter router,” but:

> **Build a runtime that can dynamically allocate and compose intelligence.**

This is closely related to the broader oh-my-agentX ideas around harnesses, runtimes, inference systems, and eventually an Intelligence OS.

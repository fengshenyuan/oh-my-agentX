# Dynamic Intelligence Composition

**Status: Research extension — 2026-08-16**

This document extends the Intelligence Scheduling / Composition research with a distinction that is increasingly important as agent systems become more capable:

> **Intelligence composition should not start from a catalog of predefined roles. It should dynamically compose a task-specific cognitive configuration from persistent capabilities, knowledge, memory, methodologies, preferences, tools, and available intelligence resources.**

This is a refinement of the existing Intelligence Scheduling / Composition thesis. It does **not** change the canonical three-layer Agent OS architecture in the main project.

## 1. Composition and scheduling are related, but different

The two terms should not be collapsed into one mechanism.

### Intelligence Composition

> **What cognitive configuration is required for this task, in this context?**

### Intelligence Scheduling

> **Which intelligence resources should that configuration consume, when, in what order, for how much computation, with what feedback, and when should the system stop?**

A simplified relationship is:

```text
Task + Context + Constraints
             │
             ▼
   Dynamic Intelligence Composition
             │
             ▼
   Cognitive Configuration
             │
             ▼
    Intelligence Scheduling
             │
       ┌─────┼─────┐
       ▼     ▼     ▼
     Model  Tools  Verifiers
```

Composition therefore determines the **shape of intelligence** that should participate; scheduling determines how available intelligence resources are allocated during execution.

## 2. Do not make predefined roles the primitive

A tempting design is:

```text
Engineer
Researcher
Reviewer
Teacher
Poet
Content Creator
```

and then to compose several predefined roles for each task.

This can be useful as a convenience layer, but it is a poor foundational abstraction. It assumes that human and agent capability can be cleanly partitioned into a fixed taxonomy of personas.

A more general model is:

```text
Persistent Intelligence Substrate
        │
        ├── capabilities
        ├── knowledge
        ├── memory
        ├── methodologies
        ├── preferences
        ├── styles
        ├── tools / services
        └── learned state
                │
                ▼
          Task + Context
                │
                ▼
      Dynamic Composition
                │
                ▼
     Current Cognitive Configuration
```

In this model, a role, persona, or operating mode is an **emergent configuration**, not a primitive object that must be selected from a registry.

## 3. The same agent can express different configurations

A long-lived agent may accumulate a broad substrate of capabilities and preferences.

For example, one person may simultaneously have:

```text
software engineering
systems architecture
technical research
Chinese writing
literary knowledge
aesthetic judgment
video scripting
audience awareness
personal preferences
```

The system should not require these to be frozen into separate agents such as:

```text
engineer-agent
poet-agent
writer-agent
creator-agent
```

Instead, a task can produce a temporary configuration such as:

```text
Technical topic for a public video

composition:
  systems architecture
  + technical research
  + explanatory writing
  + narrative structure
  + audience awareness
  + personal voice
```

Another task might produce:

```text
Creative writing

composition:
  literary knowledge
  + language ability
  + aesthetic preferences
  + selected stylistic patterns
```

The underlying agent identity and long-lived state remain the same.

## 4. Reusable patterns still matter

Rejecting predefined roles does **not** mean rejecting reusable structure.

A system can maintain reusable:

```text
methodologies
patterns
playbooks
styles
domain knowledge
skills
tool configurations
verification strategies
```

For example:

```text
code-review methodology
research methodology
poetry style
technical-video narrative pattern
incident-response playbook
```

These are reusable building blocks or priors that can participate in composition. They should not be confused with a complete role ontology.

The distinction is:

```text
Pattern / Method / Skill
        → reusable component

Role / Persona / Mode
        → possible runtime composition
```

## 5. Persistent substrate vs runtime configuration

This suggests a useful distinction for Agent OS design:

```text
Persistent Agent Substrate
    identity
    persistent principles
    capabilities
    knowledge
    durable memory
    preferences
    reusable methods

Dynamic Cognitive Configuration
    selected capabilities
    active context
    active methods / styles
    selected tools
    model resources
    reasoning budget
    verification strategy
    task-specific constraints
```

The first changes relatively slowly and should survive runtime changes. The second is assembled and adapted during execution.

This distinction is compatible with the project's existing separation between Agent Definition, Agent State, and Lifecycle State.

## 6. Why this matters for Intelligence Scheduling

Once a cognitive configuration is dynamic, scheduling becomes more than model routing.

The scheduler may need to decide not only:

```text
which model?
how much reasoning?
```

but also:

```text
which capabilities should be activated?
which methodology should be applied?
which tools are relevant?
which knowledge should be retrieved?
should a different intelligence source challenge the current configuration?
should a specialist capability be introduced temporarily?
when should the configuration be revised?
```

This creates a feedback loop:

```text
Task
  ↓
Compose initial configuration
  ↓
Execute
  ↓
Observe evidence
  ↓
Detect uncertainty / error
  ↓
Recompose
  ↓
Reschedule intelligence
  ↓
Verify
  ↓
Stop or continue
```

Composition is therefore not necessarily a one-time planning step. It may be **iterative and trajectory-dependent**.

## 7. Composition is not limited to models

The composition space can include heterogeneous intelligence sources:

```text
models
reasoning budgets
tools
search / retrieval
compilers
static analyzers
test suites
verifiers
other agents
external services
human escalation
```

A cognitive configuration can therefore be thought of as a temporary arrangement of these resources around a task rather than as a persona prompt.

## 8. Implications for Agent OS

This refinement should be understood as a runtime capability inside the existing Agent OS architecture:

```text
Agent Definition
    │
    │ Runtime ABI
    ▼
Agent Runtime
    ├── context
    ├── loop
    ├── tools
    ├── state
    ├── capabilities
    ├── dynamic composition
    └── intelligence scheduling
    │
    │ Lifecycle Contract
    ▼
Agent Management Plane
```

The project's canonical architecture therefore remains unchanged.

The new question is narrower and deeper:

> **How can a runtime dynamically construct and evolve the cognitive configuration through which a persistent agent approaches a task?**

## 9. Research questions

The following questions should be treated as open research questions rather than settled design choices:

1. What should be the smallest useful primitives in a persistent intelligence substrate?
2. How should capabilities, knowledge, memory, methods, and preferences be represented?
3. How much of a cognitive configuration should be explicit versus inferred by the runtime?
4. How can a runtime compose capabilities without collapsing them into a fixed persona taxonomy?
5. How should composition interact with model selection and reasoning effort?
6. How can the runtime detect that the current configuration is insufficient or misaligned?
7. How should composition change when new evidence arrives?
8. How should reusable patterns be packaged so they remain composable rather than becoming rigid roles?
9. Which parts of a cognitive configuration must be portable across different runtimes?
10. How should the system expose dynamic composition to users without forcing them back into low-level orchestration?

## 10. Current working hypothesis

The current working hypothesis is:

> **A long-lived agent should provide a persistent intelligence substrate; the runtime should dynamically compose a task-specific cognitive configuration from that substrate and available external intelligence resources; the scheduler should then allocate and adapt those resources according to evidence, constraints, and expected value.**

This is a research hypothesis, not a proposed standard. The next validation step should be implementation experiments rather than further taxonomy-building.

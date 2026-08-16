# Runtime Dynamic Composition

**Status: Research extension — 2026-08-16**

This document clarifies the relationship between **Dynamic Intelligence Composition** and **Intelligence Scheduling** within the existing Agent Runtime layer.

## 1. Two different meanings of scheduling

The Agent OS architecture uses the word **scheduling** in two different contexts, and they should not be conflated.

### Lifecycle scheduling

The Agent Management Plane may schedule **when an agent is activated**:

```text
cron
webhook
timer
GitHub event
email
calendar event
user message
        ↓
   Wake / Resume Agent
```

This is lifecycle orchestration. It answers:

> **When should this long-lived agent run?**

### Intelligence scheduling

The Agent Runtime may schedule **what intelligence should participate during an execution**:

```text
Task / Context
      ↓
Dynamic Cognitive Configuration
      ↓
Intelligence Scheduler
      ↓
model / reasoning / tool / verifier / agent / human
```

It answers:

> **How should intelligence be allocated while the agent is running?**

These are related, but they belong to different architectural concerns. Lifecycle scheduling belongs primarily to the Management Plane; intelligence scheduling is a runtime capability.

## 2. Dynamic composition comes before and during scheduling

The runtime does not need to select from a fixed registry of personas such as `engineer`, `researcher`, or `reviewer`.

Instead, a persistent agent can expose a broad substrate of reusable resources:

```text
capabilities
knowledge
memory
methodologies
preferences
styles
skills
tools
model access
verification mechanisms
```

Given a task and its context, the runtime can dynamically assemble a **cognitive configuration**:

```text
Persistent Intelligence Substrate
            +
      Task / Context
            +
       Constraints
            ↓
 Dynamic Intelligence Composition
            ↓
 Dynamic Cognitive Configuration
```

The resulting configuration is temporary and task-dependent. A human-like role or persona may emerge from this composition, but it is not required to exist as a predefined object.

## 3. Composition and scheduling form a feedback loop

Composition is not necessarily a one-time planning step.

```text
Task
  ↓
Compose initial configuration
  ↓
Execute
  ↓
Observe state / evidence
  ↓
Uncertainty, error, or new requirement?
  ├── no ──→ continue / verify / stop
  └── yes
          ↓
       Recompose
          ↓
      Reschedule
          ↓
        Execute
```

A new compiler error may introduce a need for a debugging capability. Conflicting evidence may justify an independent verifier. A change in user intent may require a different methodology or style.

The cognitive configuration can therefore evolve along the trajectory instead of being fixed for the whole task.

## 4. What the scheduler allocates

Once the cognitive configuration is defined, the scheduler can allocate heterogeneous resources such as:

```text
model family
reasoning budget
tool calls
retrieval / search
specialist capability
subagent
independent verifier
human escalation
```

The scheduler should optimize for the final outcome under constraints such as:

```text
quality
cost
latency
risk
reliability
```

This extends the existing Intelligence Scheduling / Composition research rather than replacing it.

## 5. Why this distinction matters for Agent OS

Without this distinction, "scheduling" can be interpreted as only a management-plane concern such as cron or background execution.

The Agent OS should instead recognize two scheduling loops:

```text
Management Plane
    lifecycle scheduling
    │
    └── when to wake / suspend / resume

Runtime
    intelligence scheduling
    │
    └── what intelligence to compose / allocate / verify / stop
```

The two loops can interact. For example, lifecycle policy may impose a time or cost budget on an execution, while the runtime scheduler decides how to spend that budget across intelligence resources.

## 6. Open research questions

1. What signals should trigger recomposition during execution?
2. Which parts of a cognitive configuration should be persistent versus ephemeral?
3. Can composition and scheduling share one optimization objective, or should they remain separate planners?
4. How should lifecycle constraints such as deadlines and quotas become inputs to runtime intelligence scheduling?
5. What semantic runtime events are required to make intelligence allocation observable and portable across harnesses?
6. How should a runtime explain or expose dynamic composition without forcing users back into low-level model selection?

The current hypothesis is intentionally narrow: **lifecycle scheduling determines when an agent runs; intelligence scheduling determines how intelligence is allocated within a run; dynamic composition determines the cognitive configuration that participates in that allocation.**
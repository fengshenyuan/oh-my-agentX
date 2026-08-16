# Vision and Architecture

**Status: Current research vision — 2026-08-16**

This document describes the current architectural model behind **oh-my-agentX** and how it evolved from the project's original portability problem.

The original idea is preserved separately in [Initial Idea](init_idea.md). The purpose of keeping that document is historical continuity: the theory should deepen over time without erasing the observations and simpler questions that produced it.

## 1. From the initial portability problem to Agent OS

The project began with a concrete question:

> **Why should the same agent definition have to be maintained separately for every harness?**

That led to the first hypothesis:

```text
Agent Definition
       ↓
Harness / Runtime
       ↓
Client / Shell
```

The first goal was portability across Codex, Claude Code, OpenCode, Pi, and future runtimes.

DeepSeek Harness then made the Runtime layer substantially more concrete, while the study of long-lived agents exposed another boundary: an agent that persists over time needs more than a runtime loop. It needs identity, deployment, triggers, observability, recovery, evaluation, policy, versioning, and retirement.

The current architecture therefore becomes:

```text
┌─────────────────────────────────────────────────────┐
│              Agent Management Plane                 │
│                                                     │
│  Create · Identity · Deploy · Schedule              │
│  Observe · Evaluate · Recover · Update              │
│  Govern · Cost · Retire                             │
└──────────────────────────┬──────────────────────────┘
                           │
                   Lifecycle Contract
                           │
┌──────────────────────────▼──────────────────────────┐
│                    Agent Runtime                    │
│                                                     │
│  Model · Context · Loop · Tools · Sandbox           │
│  State · Events · Capabilities                      │
└──────────────────────────┬──────────────────────────┘
                           │
                      Runtime ABI
                           │
┌──────────────────────────▼──────────────────────────┐
│                  Agent Definition                   │
│                                                     │
│  Identity · Skills · Instructions · Tools           │
│  Policies · Knowledge · Preferences                 │
└─────────────────────────────────────────────────────┘
```

This is the current **canonical architecture** of oh-my-agentX. It remains a research hypothesis rather than an established standard.

### Agent OS theory map

The three-layer architecture provides the stable boundaries. Within the Runtime, intelligence composition and scheduling become related but distinct capabilities:

```text
                         Agent OS
                            │
       ┌────────────────────┼────────────────────┐
       │                    │                    │
       ▼                    ▼                    ▼
Agent Definition      Agent Runtime      Management Plane
       │                    │                    │
       │                    │                    │
persistent substrate       │               lifecycle
       │                    │                    │
       │          ┌─────────┴──────────┐         │
       │          │                    │         │
       │          ▼                    ▼         │
       │     Composition          Scheduling     │
       │          │                    │         │
       │          └────────┬───────────┘         │
       │                   ▼                     │
       │       Dynamic Cognitive Configuration   │
       │                   │                     │
       │          ┌────────┼────────┐            │
       │          ▼        ▼        ▼            │
       │        Model     Tools   Verifier       │
       │                                         │
       └────────────── Runtime ABI ──────────────┘
```

This map is an explanatory refinement, not a replacement for the canonical three-layer architecture above. **Intelligence Scheduling / Composition is one important Runtime capability, not the project's overall thesis.**

## 2. Agent Definition

Agent Definition answers:

> **What is the agent?**

It is the portable, relatively stable specification that should not be owned by any one harness.

Candidate concepts include:

```text
identity
instructions / guidelines
skills
knowledge
services / tool requirements
policies / permissions
preferences
operating modes
model preferences
authentication bindings
```

The important property is not the exact manifest syntax. It is that the definition has an identity independent of whichever runtime executes it.

The definition should **not** be interpreted as a frozen catalog of roles or personas. A long-lived agent can expose a broad, persistent substrate of capabilities, knowledge, memory, preferences, and reusable methods; the runtime may dynamically compose a task-specific cognitive configuration from that substrate and the current context. See [Dynamic Intelligence Composition](intelligence-scheduling-composition/dynamic-composition.md).

This distinction is important:

```text
Persistent Agent Substrate
    → relatively stable identity, capabilities, knowledge,
      durable memory, preferences, and reusable methods

Dynamic Cognitive Configuration
    → task-specific combination of capabilities, context,
      methods, tools, model resources, reasoning, and verification
```

A role, persona, or operating mode may emerge from the latter, but it should not be assumed to be a foundational ontology of the Agent OS.

## 3. Agent Runtime

Agent Runtime answers:

> **How does the agent execute?**

A runtime sits between the portable agent definition and the underlying models and execution environment.

It may own:

```text
model adapters
context construction
agent loop
tool registration / execution
sandbox / filesystem
permissions / approvals
session state
subagents / orchestration
event streams
capability discovery
background execution
dynamic intelligence composition
intelligence scheduling
```

Different runtime philosophies should remain possible:

```text
Model-driven
Runtime-driven
Workflow-driven
Hybrid
```

The Runtime is not the model API. The model provides intelligence; the runtime determines how that intelligence participates in an agent that observes, reasons, acts, persists state, and interacts with an environment.

### Dynamic intelligence composition and scheduling

Composition and scheduling are closely related, but they answer different questions.

> **Composition:** What cognitive configuration should be assembled for the current task and context?
>
> **Scheduling:** Which heterogeneous intelligence resources should that configuration consume, when, for how much computation, with what feedback, and when should the system stop?

A runtime can therefore follow a loop such as:

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
      ┌──────┼──────┐
      ▼      ▼      ▼
    Model   Tools  Verifier
             │
             ▼
          Evidence
             │
             ├── recompose
             └── continue / stop
```

The composition need not be fixed at the beginning of a task. New evidence, failed hypotheses, tool results, verification signals, or changing constraints may cause the runtime to recompose the active cognitive configuration and reschedule intelligence resources.

This is an open research area. The project currently treats the full discussion in [Intelligence Scheduling / Composition](intelligence-scheduling-composition/conclusions.md) and [Dynamic Intelligence Composition](intelligence-scheduling-composition/dynamic-composition.md) as research hypotheses rather than protocol commitments.

### Runtime ABI

The key boundary between Definition and Runtime is the proposed **Runtime ABI**.

The research question is:

> **What is the smallest stable contract that allows the same agent definition to execute faithfully on substantially different runtimes?**

Likely concerns include:

```text
agent loading
capability negotiation
session inputs
semantic runtime events
tool invocation
state access
permissions
completion / failure
```

The goal is not to standardize implementation internals, but to identify a useful interoperability boundary.

## 4. Agent State and Lifecycle State

Agent Definition should not be conflated with dynamic state.

```text
Agent Definition
    identity
    instructions
    skills
    tools
    policies
    preferences

Agent State
    sessions
    memory
    task progress
    tool results
    learned state

Lifecycle State
    created
    provisioned
    running
    suspended
    recovering
    updating
    degraded
    retired
```

Dynamic cognitive configuration is also not the same thing as persistent Agent State. It is a runtime-level composition assembled from relatively stable substrate plus current task state and available resources.

This three-way separation is important for portability. A runtime change should not necessarily require redefining the agent, and updating a definition should not necessarily destroy accumulated state.

## 5. Agent Management Plane

Agent Management Plane answers:

> **How does the agent exist and evolve across time?**

A long-lived agent may live for days, weeks, or indefinitely. Its lifecycle therefore includes more than execution:

```text
Create
  ↓
Define / Provision
  ↓
Deploy
  ↓
Run
  ↓
Observe
  ↓
Recover
  ↓
Evaluate
  ↓
Update / Evolve
  ↓
Govern
  ↓
Retire
```

The management plane may own:

```text
identity / ownership
specification / versioning
provisioning
deployment / placement
scheduling / event subscriptions
permissions / policy
audit / observability
evaluation
failure detection / recovery
rollout / rollback
cost / quota controls
retirement
```

The analogy to Kubernetes, Vercel, or Datadog is intentionally about the **role of a control surface**, not a claim that Agent Management Plane is already a mature product category.

### Lifecycle Contract

The proposed boundary between Runtime and Management Plane is the **Lifecycle Contract**.

The research question is:

> **What contract allows an agent's identity, specification, state, policy, and lifecycle to remain coherent across time and potentially across different runtimes?**

A runtime can be excellent at executing an agent without being responsible for the complete lifecycle of that agent or a fleet of agents.

## 6. Scheduling is not the core primitive

A scheduled task is one trigger, not the architecture.

An agent may be awakened by:

```text
Schedule
Webhook
Email
Calendar event
Price change
GitHub event
Database event
User message
Timer
```

These can be normalized as:

```text
Event
  ↓
Wake Agent
  ↓
Runtime + State
  ↓
Observe / Decide
  ├── Act
  └── Ignore
        ↓
      Record
```

The deeper primitive is therefore:

> **persistent observation + durable state + event-driven activation + policy-controlled action**

Notification is also a policy decision. An agent may observe many changes while deciding that most should be silently recorded.

```text
100 observations
       ↓
state comparison + reasoning + policy
       ↓
97 ignored
2 recorded silently
1 user notification
```

This motivates a potential **attention / notification policy** as part of future management systems.

## 7. Client / Shell remains a separate concern

The client answers:

> **How does a human interact with the agent?**

Examples include:

```text
CLI
TUI
IDE
Web
Mobile
Voice
Chat
```

A client should consume semantic runtime events rather than scrape terminal output.

That implies another useful boundary:

```text
Agent Runtime
     ↓
Structured Event Stream
     ↓
Client / Shell
```

The client is therefore not the definition of the agent and not the runtime itself.

## 8. DeepSeek Harness as the runtime reference point

DeepSeek Harness is important because it provides a serious open implementation of many Runtime-layer ideas that were previously hypothetical.

Its central principle is:

> **Everything is a plugin.**

Its architecture makes model adapters, tool registry, session log, agent loop, and other components replaceable through plugin/context/event mechanisms, with typed events, durable session history, profiles, bundles, and capability seams.

For oh-my-agentX, the key distinction is:

```text
DeepSeek Harness / Cordis
    → composability WITHIN a runtime

oh-my-agentX
    → interoperability ACROSS runtimes
      + lifecycle management ACROSS time
```

DeepSeek therefore strengthens the case for the Runtime ABI while simultaneously making the gap above the runtime more visible.

See [DeepSeek Harness observations](deepseek-harness.md).

## 9. The evolving research thesis

The original thesis was:

> **An agent should not belong to its harness.**

The current thesis is broader:

> **An agent should be a portable, long-lived entity whose definition, runtime execution, and lifecycle management are separated by explicit contracts.**

That produces two related independence questions:

```text
Why should an agent belong to its harness?

Why should an agent's lifecycle belong to one runtime implementation?
```

If the answer to both is "it shouldn't", then the architecture needs to define three things:

1. a portable Agent Definition;
2. a Runtime ABI for executing that definition;
3. a Lifecycle Contract for managing that agent over time.

Intelligence Scheduling / Composition is deliberately **not** a fourth top-level layer. It is a runtime capability that becomes increasingly important as runtimes gain heterogeneous model, tool, verifier, and delegation primitives.

## 10. Research direction

The project should not attempt to implement a complete Agent OS immediately.

Instead, the research should proceed through increasingly strong experiments:

```text
1. Define one agent once
          ↓
2. Execute it on two substantially different runtimes
          ↓
3. Preserve state while switching runtimes
          ↓
4. Add lifecycle operations: create / run / suspend / resume / update
          ↓
5. Add event-driven wakeups and policy
          ↓
6. Test management of multiple agents / a fleet
```

The Intelligence Scheduling / Composition work adds a parallel runtime research track:

```text
1. Characterize heterogeneous intelligence resources
          ↓
2. Define a useful persistent intelligence substrate
          ↓
3. Experiment with dynamic cognitive configurations
          ↓
4. Adapt composition as evidence changes
          ↓
5. Allocate compute / tools / verification dynamically
          ↓
6. Evaluate quality, cost, latency, reliability, and stopping behavior
```

These are research tracks, not claims that a final protocol has already been established.

Each step should validate or falsify an abstraction before that abstraction is turned into a formal interface.

## 11. Relation to existing standards

The ecosystem is already producing useful pieces:

```text
AGENTS.md
    → portable instructions

Agent Skills
    → portable capabilities

MCP
    → portable tools / services

Agent package managers
    → portable dependencies

Agent harnesses
    → runtime implementations

Agent infrastructure
    → sandboxing / persistence / scheduling

Management systems
    → identity / policy / deployment / observability
```

oh-my-agentX is primarily interested in the **connections between these layers** rather than replacing existing standards.

## 12. Current open questions

The most important unanswered questions are now:

1. What is the minimum useful Agent Definition?
2. What must the Runtime ABI guarantee?
3. Which parts of Agent State can be portable?
4. How should capabilities be negotiated?
5. What belongs in the Lifecycle Contract?
6. Where should schedules and event subscriptions live?
7. How should identity, ownership, credentials, and policy survive runtime migration?
8. How should continuous evaluation and recovery work?
9. How can updates preserve state and behavioral continuity?
10. What is the correct abstraction for one agent versus an agent fleet?
11. What are the minimum reusable primitives of a persistent intelligence substrate?
12. How should dynamic cognitive configurations be represented, inspected, and revised?
13. Which parts of composition should be inferred by the runtime versus explicitly controlled by the user?
14. How should composition interact with model capability, reasoning effort, delegation, tools, and verification?
15. What evidence should trigger recomposition, escalation, diversification, or termination?

The project is deliberately not answering these questions prematurely.

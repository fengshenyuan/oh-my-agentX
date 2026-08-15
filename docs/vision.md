# Vision and Architecture

**Status: Current research vision — 2026-08-15**

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
```

Different runtime philosophies should remain possible:

```text
Model-driven
Runtime-driven
Workflow-driven
Hybrid
```

The Runtime is not the model API. The model provides intelligence; the runtime determines how that intelligence participates in an agent that observes, reasons, acts, persists state, and interacts with an environment.

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

The project is deliberately not answering these questions prematurely.

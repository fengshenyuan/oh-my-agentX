# oh-my-agentX

> **One agent, any harness.**

**oh-my-agentX** is an open research project exploring a portable architecture for **long-lived AI agents**: how an agent can be defined independently of a particular harness, executed by different runtimes, and managed across its full lifecycle.

The project started from a practical problem—having to duplicate the same agent identity, instructions, skills, knowledge, preferences, and tools across Codex, Claude Code, OpenCode, Pi, and other clients—but has evolved into a broader architectural question:

> **What should be standardized so that an agent can survive changes of runtime, client, and time?**

The original problem and the first-generation architecture are preserved in [Initial Idea](docs/init_idea.md). The current README describes the latest architectural hypothesis; historical reasoning should remain part of the project rather than being recoverable only from Git history.

---

## Core thesis

Our current hypothesis is that a useful Agent OS needs **three distinct layers** connected by two different contracts:

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

This is the **current canonical architecture of oh-my-agentX**. It is a research hypothesis, not an established industry standard.

### 1. Agent Definition — *What is the agent?*

A portable, client- and runtime-neutral description of an agent.

Potentially includes:

- identity and persistent principles
- instructions and guidelines
- skills
- knowledge and references
- tools and service requirements
- policies and permissions
- preferences and operating modes
- model preferences
- authentication bindings

The goal is a **single source of truth** rather than separate copies in `.claude/`, `.codex/`, OpenCode, Pi, and other runtime-specific directories.

### 2. Agent Runtime — *How does the agent execute?*

The execution substrate between an agent definition and the underlying model and environment.

A runtime may own:

- model adapters
- context construction
- agent loop
- tool registration and execution
- sandbox and filesystem access
- permissions and approvals
- session and state management
- subagents and orchestration
- event streams
- capability discovery and negotiation
- background execution

The runtime is **not** the model API. A model API exposes intelligence; a runtime determines how an agent observes, reasons, acts, persists state, and interacts with its environment.

The key research question here is the **Runtime ABI**:

> What is the smallest stable contract that lets the same agent definition execute faithfully on substantially different runtimes?

### 3. Agent Management Plane — *How does the agent exist and evolve?*

A long-lived agent needs more than an execution loop. It may exist for days, weeks, or indefinitely and therefore needs lifecycle management.

The management plane may own:

- creation and provisioning
- identity and ownership
- deployment and placement
- schedules and event subscriptions
- permissions and governance
- observability
- continuous evaluation
- failure detection and recovery
- versioning, rollout, and rollback
- cost controls
- retirement

The key research question here is the **Lifecycle Contract**:

> What contract lets an agent's identity, specification, state, policy, and lifecycle remain coherent across time and potentially across different runtimes?

---

## Why the third layer matters

A scheduled task is easy to mistake for the whole problem:

```text
cron → prompt → model → notification
```

We think this is only one small mechanism inside a much larger system.

A long-lived agent can be awakened by many kinds of events:

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
             │
             ▼
        Wake Agent
             │
             ▼
      Runtime + State
             │
        Observe / Decide
        ┌────┴────┐
        ▼         ▼
       Act      Ignore
        │
        ▼
     Observe
        │
        ▼
   Continue / Notify
```

The interesting primitive is therefore not **"scheduled prompt"**, but:

> **persistent observation + durable state + event-driven activation + policy-controlled action**

Even notification is a decision problem. An agent may observe hundreds of changes and decide that almost all of them should be silently recorded rather than shown to the user.

```text
100 observations
       ↓
state comparison + reasoning + policy
       ↓
97 ignored
2 recorded silently
1 user notification
```

This motivates the idea of an explicit **attention / notification policy** as part of future agent systems.

---

## From portable configuration to an Agent OS

The original motivation for oh-my-agentX was configuration portability:

```text
One agent definition
        │
        ├── Codex
        ├── Claude Code
        ├── OpenCode
        └── Pi
```

That remains important, but it is only the bottom of the architecture.

The broader model is now:

```text
                       Agent OS
                          │
        ┌─────────────────┼─────────────────┐
        ▼                 ▼                 ▼
   Agent Definition   Management Plane   Client / Shell
        │                 │                 │
        │             lifecycle control     │
        │                 │                 │
        └──────────────┬──┴──┬──────────────┘
                       ▼     ▼
                  Runtime A  Runtime B ...
                       │     │
                       ▼     ▼
                  Environment / Sandbox
```

The client remains an important presentation and interaction layer, but it is not the definition of the agent. A CLI, TUI, IDE, web UI, mobile UI, or voice interface should be able to interact with the same underlying agent through structured runtime events.

---

## Agent Definition, Agent State, Lifecycle State

One of the project's current hypotheses is that these three concepts should not be conflated:

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

This distinction matters for portability. A runtime change should not necessarily require redefining the agent, and an upgrade should not necessarily destroy useful state.

---

## DeepSeek Harness changed the research direction

The release of [DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness) in August 2026 provided an important concrete reference implementation for the runtime layer.

Its central architectural principle is:

> **Everything is a plugin.**

DeepSeek's runtime makes model adapters, tool registry, session log, agent loop, and other pieces replaceable through a plugin/context/event system. Its architecture also introduces typed events, durable session history, profiles, bundles, capability seams, and lifecycle-oriented execution primitives.

This matters to oh-my-agentX because it gives us a serious open runtime to study rather than leaving the runtime layer entirely hypothetical.

But the boundary is important:

```text
DeepSeek Harness / Cordis
    → composability WITHIN a runtime

oh-my-agentX
    → interoperability ACROSS runtimes
      + lifecycle management ACROSS time
```

DeepSeek Harness therefore does not remove the portability problem. Instead, it makes the runtime boundary more concrete and strengthens the case for asking what belongs above that boundary.

See [DeepSeek Harness observations](docs/deepseek-harness.md).

---

## What the two contracts mean

### Runtime ABI

The Runtime ABI is the proposed boundary between **Agent Definition** and **Agent Runtime**.

It should eventually describe things such as:

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

The goal is not to standardize every runtime implementation detail. It is to identify the smallest useful common denominator.

### Lifecycle Contract

The Lifecycle Contract is the proposed boundary between **Agent Runtime** and **Agent Management Plane**.

It may eventually describe:

```text
create / provision
identity / ownership
deploy / suspend / resume
schedule / event subscriptions
observe / evaluate
recover / restart
upgrade / rollback
policy / governance
cost / quota
retire
```

This is a separate problem from the Runtime ABI. A runtime can be excellent at executing an agent without being responsible for the agent's complete lifecycle, just as a workload runtime does not automatically become a fleet management system.

---

## What exists in the ecosystem

The architecture is intentionally grounded in existing systems rather than invented in isolation.

| Layer | Representative examples | What they demonstrate |
|---|---|---|
| Agent applications | Manus, Perplexity Computer, ChatGPT Agent, Claude Cowork | Long-running general-purpose work |
| Agent runtimes / harnesses | DeepSeek Harness, Claude Code, Codex, OpenCode, Pi | Agent execution models and harness abstractions |
| Runtime infrastructure | Cloudflare Agents and similar systems | Durable state, scheduling, sandboxing, persistent execution |
| Enterprise management | Microsoft Copilot Studio, Salesforce Agentforce | Identity, governance, deployment, enterprise integrations |

No single system is currently treated here as the definitive implementation of the full architecture.

The purpose of oh-my-agentX is to identify the interfaces between these categories.

---

## Research questions

The project is now organized around a small set of architectural questions:

1. **Agent Definition:** What is the minimum portable representation of an agent?
2. **Runtime ABI:** What contract allows multiple substantially different runtimes to execute that definition?
3. **State portability:** Which parts of session, memory, task, and learned state can move between runtimes?
4. **Capability negotiation:** How does an agent declare requirements and discover what a runtime can provide?
5. **Lifecycle Contract:** What management interface is needed for long-lived agents?
6. **Event model:** Which semantic events need to cross runtime, client, and management boundaries?
7. **Identity and policy:** How should ownership, credentials, permissions, and governance survive runtime changes?
8. **Evaluation and recovery:** How can an agent be continuously evaluated and recovered when degraded or stuck?
9. **Versioning and evolution:** How can an agent be updated without destroying state or violating policy?
10. **Fleet model:** What is the right abstraction for managing one agent versus thousands of related agents?

---

## What this project is — and is not

oh-my-agentX is **not** trying to become another monolithic coding agent.

It is also not currently claiming to define an industry standard.

Instead, it is a research project for finding the smallest useful open architecture through concrete experiments:

```text
Agent Definition
       ↓
Runtime ABI
       ↓
Multiple Runtime Adapters
       ↓
Lifecycle Contract
       ↓
Management experiments
```

The implementation should remain deliberately small until experiments demonstrate that an abstraction is real.

The first compelling experiment remains simple:

> **Define one agent once, then make that same agent usable in at least two substantially different runtimes.**

The next experiment is harder:

> **Preserve the agent's identity, state, and policy while its runtime changes or its lifecycle continues autonomously.**

---

## Research history and current documents

The project intentionally keeps both its current model and the reasoning that produced it.

- [Initial idea and first-generation architecture](docs/init_idea.md)
- [Current vision and architecture](docs/vision.md)
- [DeepSeek Harness observations](docs/deepseek-harness.md)
- [Agent Lifecycle and Management Plane](docs/agent-lifecycle.md)

## Status

**Early research / experimental.**

The architecture is intentionally presented as a set of hypotheses. The project will be validated by implementation, interoperability experiments, and comparison with real agent systems rather than by declaring the abstraction correct in advance.

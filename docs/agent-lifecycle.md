# Agent Lifecycle and Management Plane

**Status: Research hypothesis — 2026-08-15**

DeepSeek Harness sharpened the runtime side of the oh-my-agentX architecture. Our more recent work suggests that the original model is still missing a distinct layer responsible for the **long-lived existence and lifecycle of agents**.

This document records the current architectural hypothesis.

## 1. The core Agent OS model

The current oh-my-agentX architecture should be understood as three layers connected by two contracts:

```text
┌──────────────────────────────────────────────┐
│              Agent Management Plane          │
│                                              │
│  Create · Identity · Deploy · Schedule       │
│  Observe · Evaluate · Recover · Update       │
│  Govern · Cost · Retire                      │
└───────────────────────┬──────────────────────┘
                        │
                Lifecycle Contract
                        │
┌───────────────────────▼──────────────────────┐
│                Agent Runtime                 │
│                                              │
│  Model · Context · Loop · Tools · Sandbox    │
│  State · Events · Capabilities               │
└───────────────────────┬──────────────────────┘
                        │
                  Runtime ABI
                        │
┌───────────────────────▼──────────────────────┐
│                Agent Definition              │
│                                              │
│  Identity · Skills · Instructions · Tools    │
│  Policies · Knowledge · Preferences          │
└──────────────────────────────────────────────┘
```

This is now the **canonical architecture hypothesis** for oh-my-agentX.

The three layers answer three different questions:

- **Agent Definition** — What is this agent?
- **Agent Runtime** — How does this agent execute?
- **Agent Management Plane** — How is this agent created, deployed, operated, evolved, governed, and retired over time?

The two contracts define the boundaries:

- **Runtime ABI** — the contract that allows an Agent Definition to be executed by different runtimes.
- **Lifecycle Contract** — the contract that allows the lifecycle and operational state of an agent to be managed independently of one particular runtime implementation.

The phrase **Agent OS** remains a hypothesis. It does not imply that these interfaces are already standardized or that an operating-system-like implementation necessarily exists.

## 2. Why the original three-layer model was incomplete

The original architecture asked:

```text
Agent Definition
    → What is the agent?

Harness / Runtime
    → How does it run?

Shell / Client
    → How is it used?
```

That remains useful, but it is optimized for an agent viewed as a session or execution process.

A long-lived agent changes the problem. Once an agent is intended to exist for days, weeks, or indefinitely, somebody must answer questions outside the moment-to-moment runtime loop:

```text
Who created it?
Who owns it?
What identity does it have?
What capabilities and permissions does it have?
Where is it deployed?
What wakes it up?
What state does it currently hold?
Is it healthy?
What did its previous runs do?
How is it evaluated?
What version is running?
What happens when it gets stuck?
How is it upgraded or rolled back?
How much does it cost?
When should it be retired?
```

These are management-plane concerns.

## 3. Agent Definition

Agent Definition is the relatively stable, portable description of an agent.

Conceptually it may contain:

```text
identity
skills
instructions
tools
policies
knowledge
preferences
model preferences
capability requirements
```

The definition should remain independent of a particular runtime wherever practical.

This continues the original oh-my-agentX thesis:

> **The agent should not belong to its harness.**

A single Agent Definition should ideally be usable by multiple runtimes through the Runtime ABI.

## 4. Agent Runtime

The runtime is the execution/data plane.

It owns the mechanisms required for an agent to act:

```text
model invocation
context construction
agent loop
tool calling / execution
sandbox
session state
event stream
capability discovery
permission enforcement
subagents / orchestration
```

A runtime may implement different execution philosophies:

```text
Model-driven
Runtime-driven
Workflow-driven
Planner / executor
Human-in-the-loop
Multi-agent
Hybrid
```

The architecture does not assume that one loop philosophy is correct.

### DeepSeek Harness as a reference implementation

DeepSeek Harness is important because it provides a serious open-source implementation of a highly composable runtime. Its pluginized model, typed events, durable session history, replaceable model/tool/loop components, profiles, bundles, and capability seams make the runtime boundary unusually concrete.

For oh-my-agentX, the key distinction is:

```text
DeepSeek Harness
    → how an agent runtime is composed and executed

oh-my-agentX Runtime ABI
    → what contract different runtimes would need to expose
```

DeepSeek does not by itself define a neutral Agent Definition or a cross-runtime Lifecycle Contract.

## 5. Agent Management Plane

The Management Plane is the control plane for long-lived agents.

It is responsible for the lifecycle of agents rather than the internal execution loop of one runtime.

Its conceptual responsibilities are:

```text
Create
Identity
Deploy
Schedule
Observe
Evaluate
Recover
Update
Govern
Cost
Retire
```

In a mature system, the management plane would answer questions such as:

- What agents exist?
- Who owns each agent?
- What specification and version define each agent?
- Which runtime is currently executing it?
- Which tools, services, credentials, and policies are attached?
- What triggers wake it up?
- What is its lifecycle state?
- Is it healthy, stuck, or degraded?
- What happened during recent executions?
- Which version is deployed?
- How is it upgraded, rolled back, or migrated?
- How much has it consumed?
- When should it be suspended or retired?

This is the layer where an eventual product category could emerge analogous in role—not implementation—to the control surfaces that Kubernetes, Vercel, or Datadog provide for conventional software systems.

## 6. Lifecycle Contract

The Lifecycle Contract is the second major research boundary in oh-my-agentX.

The Runtime ABI answers:

> **Can this runtime execute this Agent Definition?**

The Lifecycle Contract asks:

> **Can this agent's identity, specification, state, policy, and operational lifecycle be controlled consistently even when the runtime changes?**

Conceptually, the contract may need to cover:

```text
agent identity
specification / version
ownership
capability requirements
runtime binding
state attachment
schedules / event subscriptions
permissions / policy
health / status
execution history
recovery intent
upgrade / rollback
cost / usage
retirement
```

This is deliberately a research target, not a proposed standard vocabulary yet.

## 7. Scheduling is only one wake-up mechanism

A scheduled task is one way to activate a long-lived agent. It should not define the architecture.

Potential triggers include:

```text
Schedule
Webhook
Email
Calendar
Price change
GitHub event
Database event
File change
User message
Timer
```

These can be normalized conceptually as:

```text
Event / Trigger
       │
       ▼
  Wake Agent
       │
       ▼
Agent Runtime + State
       │
 ┌─────┴─────┐
 ▼           ▼
Observe      Act
 │           │
 └─────┬─────┘
       ▼
     Decide
       │
  ┌────┴────┐
  ▼         ▼
Continue   Notify
```

The important primitive is therefore not `cron → LLM`, but:

> **persistent observation + state + event-driven activation + policy-controlled action**.

Notification is also part of the decision layer rather than merely a transport mechanism. A long-lived agent may observe many changes and intentionally notify the user about only a small subset.

This suggests an eventual **attention / notification policy** as a management concern.

## 8. Agent Definition, Agent State, and Lifecycle State

The lifecycle model adds an important distinction to the earlier configuration-vs-state discussion.

```text
Agent Definition
    │
    ├── identity
    ├── instructions
    ├── skills
    ├── tools
    ├── policies
    └── preferences

Agent State
    │
    ├── sessions
    ├── memory
    ├── task progress
    ├── tool results
    └── learned state

Lifecycle State
    │
    ├── created
    ├── provisioned
    ├── deployed
    ├── running
    ├── suspended
    ├── recovering
    ├── updating
    ├── degraded
    └── retired
```

This three-way distinction may become a fundamental part of the future architecture:

```text
Definition = what the agent is
State      = what the agent currently knows / has experienced
Lifecycle  = where the agent is in its operational existence
```

A key open question is which portions of Agent State and Lifecycle State should be portable across runtimes.

## 9. Runtime ABI vs Lifecycle Contract

These two boundaries should not be conflated.

```text
                  Agent Definition
                         │
                    Runtime ABI
                         │
              ┌──────────▼──────────┐
              │    Agent Runtime   │
              └──────────┬──────────┘
                         │
                  Lifecycle Contract
                         │
              ┌──────────▼──────────┐
              │ Management Plane   │
              └─────────────────────┘
```

The ordering here is conceptual, not necessarily a strict network topology.

A runtime may be deeply stateful and a management plane may delegate some functions into the runtime. The point is to keep the contracts logically distinct:

- **Runtime ABI:** execution interoperability.
- **Lifecycle Contract:** operational interoperability.

This distinction is one of the main new theoretical directions for oh-my-agentX.

## 10. Open Questions

1. What is the minimum declarative **Agent Spec** required to create a useful long-lived agent?
2. Which parts of Agent State must be portable across runtimes?
3. Should schedules and event subscriptions belong to Agent Definition, Lifecycle Contract, or both?
4. How should identity, credentials, permissions, and ownership survive runtime migration?
5. What is the minimum semantic event model required by both runtime clients and management systems?
6. How should agents be evaluated continuously rather than only per task?
7. How can a management plane detect a stuck, looping, degraded, or misbehaving agent?
8. How should agents be updated without losing useful state?
9. Can an agent migrate from one runtime to another while preserving behavioral continuity?
10. What is the difference between one persistent agent and a fleet of related agent instances?
11. Should Management Plane itself be portable, interoperable infrastructure, or a product-specific control plane?
12. Where should observability, evaluation, policy, and cost accounting live: runtime, management plane, or both?

## 11. Strategic implication for oh-my-agentX

The current research problem is therefore broader than the project's original configuration-sharing idea.

We are now investigating whether three portable concepts can form a coherent open architecture:

```text
Agent Definition
       │
       ▼
Runtime ABI
       │
       ▼
Agent Runtime
       │
       ▼
Lifecycle Contract
       │
       ▼
Agent Management Plane
```

The smallest compelling experiment should eventually test both boundaries:

1. Define one agent once and execute it on at least two substantially different runtimes.
2. Manage the same agent's lifecycle through a runtime-neutral control model.

DeepSeek Harness is a strong reference implementation for the runtime side. Cloudflare Agents is a useful reference point for persistent runtime infrastructure. General-purpose agents such as Manus, OpenAI's agent products, Claude Cowork, Gemini agent experiences, and Perplexity Computer are useful reference points for the application side.

The research goal is not to reproduce any one of them. It is to determine whether **Agent Definition + Runtime ABI + Agent Runtime + Lifecycle Contract + Management Plane** can become a coherent, useful, and eventually implementable architecture.

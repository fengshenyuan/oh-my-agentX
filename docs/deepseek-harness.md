# DeepSeek Harness: Initial Observations

**Observation date: 2026-08-13**

DeepSeek released [DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness), an open-source agent harness implemented in TypeScript and built on [Cordis](https://github.com/cordiverse/cordis).

The repository describes the central design principle as:

> **Everything is a plugin.**

It is currently a developer preview and explicitly warns that compatibility-breaking changes will occur.

## 1. Why this release matters to oh-my-agentX

Before this release, we were treating the Harness / Runtime layer largely as a conceptual gap:

```text
Agent Definition
      ↓
Harness / Runtime
      ↓
Shell / Client
```

DeepSeek now provides a substantial open implementation of that middle layer.

That changes the research question.

We no longer need to ask:

> Should someone build a highly composable agent runtime?

There is now a serious open-source reference implementation to study.

The more interesting question becomes:

> **What should exist above a runtime like DeepSeek Harness so that the same agent can move between DeepSeek Harness, Pi, OpenCode, Claude Code, Codex, and future runtimes?**

## 2. DeepSeek's runtime architecture

DeepSeek's architecture documentation says Cordis provides a shared context where plugins contribute:

- services
- typed events
- reversible effects

The documentation explicitly says that the model adapter, tool registry, session log, and agent loop are all plugins, with no privileged core that must be patched directly.

This is important because it makes the runtime itself highly composable.

Conceptually:

```text
                    Shared Context
                         │
          ┌──────────────┼──────────────┐
          ▼              ▼              ▼
        Model           Tools         Session
        plugin          plugin        plugin
          │              │              │
          └──────────────┼──────────────┘
                         ▼
                     Agent Loop
                       plugin
```

DeepSeek's architecture also introduces:

```text
profiles
bundles
patch layers
capability seams
typed events
durable session events
```

These are not merely UI abstractions. They affect how the runtime itself is composed and replaced.

## 3. The most important distinction

DeepSeek Harness and oh-my-agentX are related but solve different problems.

### DeepSeek Harness

> Composability **within** a runtime.

Its core question is:

> How can the pieces of an agent runtime be independently replaced and composed?

### oh-my-agentX

> Interoperability **across** runtimes.

Its core question is:

> How can an agent definition survive when the runtime changes?

Therefore:

```text
DeepSeek Harness
    ↓
one runtime can be highly composable

oh-my-agentX
    ↓
the agent can move between runtimes
```

DeepSeek does not make the portability problem disappear.

## 4. DeepSeek makes the runtime boundary more concrete

The release strengthens several ideas from our original architecture.

### Agent loop should not necessarily be sacred

If the agent loop is a plugin, different loop implementations can potentially coexist.

For example:

```text
ReAct
Planner → Executor
Workflow
Human-in-the-loop
Multi-agent
Model-driven
Hybrid
```

The runtime becomes a substrate for agent execution strategies rather than one fixed cognitive architecture.

### Capabilities should have explicit seams

DeepSeek documents a capability seam as an interface with:

```text
Service Definition
Service Provider
Consumer
```

This is very close to the capability-contract idea in oh-my-agentX.

### Events are first-class runtime primitives

DeepSeek separates durable session events from live extension points and uses typed events throughout the execution lifecycle.

This strongly reinforces our belief that a future client/runtime interface should be event-oriented and semantic rather than based on terminal text.

## 5. Session state is particularly interesting

DeepSeek treats the session log as the source from which model-visible context is derived.

Its architecture makes an explicit invariant:

> Anything visible to the model must be reconstructable from the log.

This is a strong design choice because it links:

```text
context
replay
resume
fork
UI fidelity
telemetry
persistence
```

into one event-based state model.

For oh-my-agentX, this suggests that **Agent State** may deserve its own protocol rather than being treated as a collection of files.

## 6. Cordis and the paper

The DeepSeek Harness is powered by Cordis, whose design is described in the paper:

**A Programming Paradigm for Spatiotemporal Composability**

Repository:

https://github.com/cordiverse/paper

The significance of the paper is not that it proves one "correct" architecture for agents.

Its importance is that it provides a formal vocabulary for a runtime based on dynamic composition, context, events, lifecycle, and temporal relationships.

That is highly relevant because agent runtimes are dynamic systems:

```text
model
  ↓
tool call
  ↓
event
  ↓
new context
  ↓
model
  ↓
background task
  ↓
user interaction
  ↓
subagent
```

A useful runtime abstraction therefore needs to describe not just components, but how components interact over time.

## 7. What DeepSeek does NOT solve for us

Even with DeepSeek Harness, the following remain open research questions for oh-my-agentX:

### Portable Agent Definition

Can we define an agent once using a neutral representation of:

```text
identity
instructions
skills
knowledge
memory
tools
services
preferences
tasks
etc.
```

and load it into multiple runtimes?

### Agent State Interoperability

Can session/memory/task state survive a runtime change?

### Runtime Capability Negotiation

Can an agent declare required capabilities and discover what a runtime can actually provide?

### Runtime Event ABI

Can a runtime expose a stable enough semantic event vocabulary for independent clients to consume?

### Client / Shell Interoperability

Can one client talk to multiple runtimes without understanding each runtime's internal event model?

## 8. A useful updated architecture

After observing DeepSeek Harness, the architecture becomes:

```text
                         Agent Definition
                                │
                         Agent State
                                │
                    ┌───────────▼───────────┐
                    │   Runtime Interface   │
                    │      (research)       │
                    └───────────┬───────────┘
                                │
            ┌───────────────────┼───────────────────┐
            ▼                   ▼                   ▼
      DeepSeek Harness          Pi               OpenCode
          + Cordis
            │                   │                   │
            └───────────────────┼───────────────────┘
                                │
                       Client Interface
                                │
                     ┌──────────┼──────────┐
                     ▼          ▼          ▼
                    TUI        IDE        Web
```

DeepSeek gives us a strong concrete implementation inside the middle box.

oh-my-agentX is interested in the contracts that cross that box.

## 9. New insight: from Runtime ABI to Agent Lifecycle

Our subsequent discussion suggests that the architecture has another missing layer.

A runtime answers:

> How is an agent composed and executed?

A **Lifecycle / Management Plane** answers:

> How is an agent created, provisioned, scheduled, observed, recovered, evaluated, updated, governed, and retired over time?

This motivates an emerging Agent OS model:

```text
                         Agent OS
                            │
             ┌──────────────┼──────────────┐
             ▼              ▼              ▼
          Create          Deploy         Operate
             │              │              │
        Agent Spec       Runtime        Observe
        Skills           Sandbox        Evaluate
        Identity         State          Recover
        Policies         Memory         Update
        Tools            Schedule        Govern
        Capabilities     Events          Cost
             │              │              │
             └──────────────┼──────────────┘
                            ▼
                     Long-lived Agent
```

The key insight is that **Schedule** is only one activation mechanism. A long-lived agent may be awakened by:

```text
cron
webhook
email
calendar event
price change
GitHub event
database change
user request
```

These are better understood as events that activate a persistent agent with state and policy.

Likewise, notification is not merely a transport concern. An agent may observe many changes and decide that most should not interrupt its human owner.

Therefore a management plane may eventually own an attention / notification policy in addition to execution scheduling.

## 10. DeepSeek's place in the emerging stack

The current evidence suggests a useful separation:

```text
Agent Application
    Manus / OpenAI agent products / Claude Cowork / etc.
                │
                ▼
Agent Lifecycle / Management Plane       ← open research area
                │
                ▼
Agent Runtime / Harness
    DeepSeek Harness / other runtimes
                │
                ▼
Runtime Infrastructure
    sandbox / storage / network / compute
```

DeepSeek Harness is therefore not itself the entire Agent Lifecycle Management layer.

Its importance is that it makes the **runtime substrate** concrete enough that we can ask what should sit above it.

In particular, its pluginized components, durable session events, typed lifecycle events, capability seams, profiles, and bundles provide a useful reference for the execution side of a future lifecycle architecture.

## 11. Strategic implication for oh-my-agentX

We should **not compete with DeepSeek Harness by building another harness**.

Instead, DeepSeek Harness should become one of the first runtimes we study and potentially target.

A good direction is:

```text
                 oh-my-agentX
                  Agent Package
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
         dsh           Pi       OpenCode
       adapter       adapter     adapter
          │            │            │
          ▼            ▼            ▼
       runtime       runtime      runtime
```

But the long-term architecture may need one more interface above the runtime adapters:

```text
                 Agent Definition
                        │
                        ▼
              Runtime Interface / ABI
                        │
                        ▼
          Agent Lifecycle / Management API
                        │
              ┌─────────┴─────────┐
              ▼                   ▼
        Runtime Adapter A   Runtime Adapter B
```

The smallest compelling experiment would be:

> Define one agent once, make that same definition usable in at least two substantially different runtimes, and preserve enough identity/state/lifecycle semantics that the agent remains meaningfully the same agent.

That would test whether the abstraction is real before we attempt to define a full "Agent ABI" or lifecycle standard.

## 12. Research attitude

DeepSeek Harness makes this project more, not less, interesting.

It provides:

- a serious open runtime to study
- a concrete plugin model
- a real event/state architecture
- a formal programming model underneath it
- evidence that agent runtime architecture itself is becoming an important engineering discipline
- a concrete reference point from which to investigate the layer above the runtime

Our goal is not to declare a standard.

Our goal is to discover whether a **portable agent layer + runtime contract + lifecycle / management contract** can be made real.

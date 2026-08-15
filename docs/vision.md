# Vision and Architecture

This document records the initial architecture discussion behind **oh-my-agentX**. It is a working research note, not a specification.

## 1. The original problem

Modern agent clients tend to own both the execution environment and the agent's persistent configuration.

For example, a developer may use several of:

- Codex
- Claude Code
- OpenCode
- Pi

Each client may have its own:

- instruction file
- skills directory
- hooks
- tools
- memories
- personal preferences
- company/project guidance
- model settings
- agent-specific state

This creates unnecessary duplication.

A simple desired property is:

```text
One agent definition
        │
        ├── Codex
        ├── Claude Code
        ├── OpenCode
        └── Pi
```

rather than:

```text
Codex agent config
Claude Code agent config
OpenCode agent config
Pi agent config
```

The deeper question is whether the agent itself should be portable independently of any particular client.

## 2. The three-layer model

The initial model separates the system into three layers.

### Layer A — Agent Definition

This answers:

> What is the agent?

Candidate resources include:

```text
identity/
soul/
guidelines/
skills/
knowledge/
memory/
tools/
services/
hooks/
scripts/
tasks/
crons/
mindsets/
preferences/
models/
auth/
```

We should not assume all of these deserve first-class status in the eventual standard. The point is to identify the primitives through experiments.

A future manifest might conceptually look like:

```yaml
agent:
  name: architect

identity:
  soul: ./soul/soul.md

instructions:
  - ./guidelines/general.md

skills:
  - ./skills/code-review

knowledge:
  - ./knowledge/company

memory:
  provider: local

tools:
  - filesystem
  - github

services:
  - github

models:
  default: openai/gpt-5.6
```

This is deliberately illustrative. The exact format is not decided.

### Layer B — Harness / Runtime

This answers:

> How does the agent execute?

The runtime sits between the agent definition and the underlying model.

It may own:

```text
agent loop
model adapters
context construction
tool registry
tool execution
hooks
permissions
sandbox
session state
memory integration
subagents
background jobs
event stream
capabilities
```

Different runtime philosophies should remain possible:

```text
Model-driven
    model controls most of the loop

Runtime-driven
    human-designed execution loop controls behavior

Workflow-driven
    explicit state machine / workflow

Hybrid
    human-defined control structure + model-directed decisions
```

The architecture should not decide which philosophy is best.

### Layer C — Shell / Client

This answers:

> How does a human interact with the agent?

Examples:

```text
CLI
TUI
IDE
Web
Mobile
Voice
Chat
```

The shell should consume structured runtime events rather than scraping terminal text.

## 3. Configuration is not state

One important refinement is that configuration and dynamic state should not be conflated.

Relatively stable:

```text
identity
soul
skills
guidelines
tool declarations
preferences
```

Dynamic:

```text
conversation
working memory
task progress
tool results
background jobs
learned memory
```

This suggests that a future architecture should distinguish:

```text
Agent Definition
        +
Agent State
```

rather than treating everything as configuration.

## 4. Capability negotiation

Runtimes will differ in their available capabilities.

For example:

```text
Agent requires:
    github
    postgres
    browser
```

but a runtime may only provide:

```text
github
browser
```

Therefore the architecture likely needs explicit capability discovery / negotiation.

Conceptually:

```text
required capability
        │
        ▼
runtime capability set
        │
   ┌────┴────┐
   │         │
  yes        no
   │         │
execute    degrade/fail
```

This may become one of the most important parts of a future runtime interface.

## 5. Tools vs services

A useful distinction is:

### Service

An external capability or resource:

```text
GitHub
Slack
Postgres
AWS
Kubernetes
```

### Tool

A concrete operation exposed to the model:

```text
github.search()
github.create_issue()
filesystem.read()
```

Conceptually:

```text
Service
   │
   └── Tools
        ├── search
        ├── create
        └── update
```

Authentication should bind to services rather than being embedded directly into agent resources.

## 6. Soul vs mindset

Potentially useful distinction:

### Soul

Persistent identity, values, principles.

### Mindset

A temporary cognitive mode:

```text
architect
researcher
debugger
reviewer
teacher
```

The runtime could activate a mindset without changing the agent's underlying identity.

## 7. Runtime protocol

A key hypothesis is that the runtime should expose a structured contract.

Possible runtime input:

```json
{
  "session": {},
  "agent": {},
  "message": {},
  "context": {},
  "capabilities": {}
}
```

Possible runtime events:

```text
AgentStarted
MessageReceived
ThinkingStarted
TextDelta
ToolCallStarted
ToolCallInput
ToolCallOutput
ThinkingFinished
PermissionRequest
MessageCompleted
Error
AgentStopped
```

The exact vocabulary is unknown.

The important idea is that the runtime event stream becomes the abstraction consumed by clients.

## 8. The deeper architectural question

The project is ultimately investigating whether this is possible:

```text
                  Agent Definition
                         │
                 Runtime Interface
                         │
        ┌────────────────┼────────────────┐
        ▼                ▼                ▼
      Runtime A       Runtime B        Runtime C
        │                │                │
        └────────────────┼────────────────┘
                         │
                  Client Interface
                         │
              ┌──────────┼──────────┐
              ▼          ▼          ▼
             TUI        IDE        Web
```

The desired property is:

> **Define the agent once; run it on different runtimes; interact with it through different clients.**

## 9. From Runtime Portability to Agent Lifecycle

The three-layer model exposes another missing concern when an agent is expected to live beyond a single interaction.

A long-lived agent needs to be created, provisioned, deployed, awakened, observed, recovered, evaluated, updated, governed, and retired. Scheduling is only one possible trigger in this lifecycle.

This motivates a fourth architectural perspective, above an individual runtime:

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

This is not intended to assert that an "Agent OS" is already a standard product category. It is a research hypothesis for the next layer of abstraction.

### Runtime vs Management Plane

It is useful to distinguish:

```text
Harness / Runtime
    How is the agent composed and executed?

Agent Management Plane
    How is the agent created, deployed, observed, recovered,
    evaluated, updated, governed, and retired across time?
```

The management plane may eventually need primitives for:

```text
identity
specification
provisioning
deployment
scheduling
subscriptions / event triggers
permissions
policy
audit
observability
evaluation
recovery
versioning
rollout / rollback
cost controls
retirement
```

This resembles the role that management/control planes play in conventional infrastructure. It is possible that an equivalent layer for long-lived agents will become an important infrastructure category, but that remains an open hypothesis.

### Why this matters to portability

The original portability question was:

> Can the same agent definition run on different runtimes?

The lifecycle perspective expands it to:

> **Can an agent preserve its identity, definition, state, policy, and behavioral continuity while moving between runtimes and operating for long periods?**

This suggests that a future oh-my-agentX architecture may need not only an **Agent Definition + Runtime Interface**, but also a **Lifecycle / Management Interface**.

## 10. What not to build first

Do not start by implementing every possible resource:

```text
skills
memory
hooks
tasks
cron
soul
auth
services
lifecycle management
etc.
```

That risks creating another giant agent framework.

A better first experiment is:

1. a minimal agent manifest
2. portable instructions
3. portable skills
4. portable tools/services declarations
5. one common event model
6. two or more runtime adapters
7. a minimal lifecycle model for create/run/suspend/resume/update

The point is to discover the true common denominator.

## 11. Position relative to the existing ecosystem

The ecosystem is already standardizing pieces:

```text
AGENTS.md
    → portable instructions

Agent Skills
    → portable capabilities

MCP
    → portable external tools/services

Agent package managers
    → portable dependencies

Agent harnesses
    → runtime implementations

Agent management / lifecycle systems
    → still an open and fragmented space
```

oh-my-agentX is interested in the missing connections between these pieces, including whether runtime portability and lifecycle portability can be expressed through interoperable contracts.

## 12. The central question

The project can be reduced to two related questions:

> **Why should an agent belong to its harness?**

and, for long-lived agents:

> **Why should an agent's lifecycle belong to one runtime implementation?**

If the answer to both is "it shouldn't", then the next questions are:

> What is the minimum portable representation of an agent?

> What contract must a runtime implement to execute it faithfully?

> What contract must a management plane implement to create and operate it over time?

That is the evolving research problem.


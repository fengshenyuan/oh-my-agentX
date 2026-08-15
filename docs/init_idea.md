# Initial Idea

**Status: Historical research note — August 2026**

This document preserves the original problem statement and the first architecture formed when **oh-my-agentX** was conceived. It is intentionally kept separate from the current architecture so that the project can evolve without erasing the reasoning that led to it.

## 1. The original problem

Modern agent clients increasingly become capable enough to accumulate their own identity, instructions, skills, tools, memories, preferences, and project/company guidance.

A developer may use several agent clients at the same time:

```text
Codex
Claude Code
OpenCode
Pi
```

But each client tends to have its own configuration surface:

```text
Codex agent config
Claude Code agent config
OpenCode agent config
Pi agent config
```

This creates duplication and drift.

The simplest expression of the original problem was:

> **Why do I have to maintain the same agent identity, instructions, skills, knowledge, preferences, and other configuration separately for every agent client I use?**

The desired property was:

```text
One agent definition
        │
        ├── Codex
        ├── Claude Code
        ├── OpenCode
        └── Pi
```

rather than copying the same definition into every client.

## 2. The first deeper question

The configuration problem quickly led to a more fundamental question:

> **Can an agent be defined independently from the runtime that executes it and the client that presents it to the user?**

This suggested that the agent itself should become a portable artifact rather than a side effect of a particular harness.

## 3. The initial three-layer model

The first architecture separated three concerns:

```text
                 Agent Definition
                       │
              "What is this agent?"
                       │
                       ▼
              Harness / Runtime
                       │
               "How does it run?"
                       │
                       ▼
                 Shell / Client
                       │
                "How is it used?"
```

### Layer A — Agent Definition

A portable, client-neutral representation of an agent.

Candidate resources included:

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

The exact list was intentionally exploratory rather than normative.

An illustrative manifest looked like:

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

The central principle was **single source of truth**.

### Layer B — Harness / Runtime

The runtime was the execution layer between the agent definition and the underlying model.

It might own:

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

An important early observation was that "runtime" is larger than a model API. A model API describes how to invoke intelligence; the runtime also determines how an agent acts, persists, interacts with tools, and exposes events.

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

### Layer C — Shell / Client

The human-facing interaction layer:

```text
CLI
TUI
IDE
Web
Mobile
Voice
Chat
```

The shell should consume semantic runtime events rather than scraping terminal text.

For example:

```json
{
  "type": "tool.call",
  "id": "call_123",
  "tool": "github.search",
  "input": {
    "query": "..."
  }
}
```

A TUI, IDE, or web client could render the same event differently.

## 4. Early refinement: configuration is not state

The initial design quickly revealed that not everything belonging to an agent should be treated as configuration.

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
session history
```

This produced the first distinction:

```text
Agent Definition
        +
Agent State
```

That distinction later became an important input to the current three-way model of Agent Definition, Agent State, and Lifecycle State.

## 5. Early capability model

Different runtimes would not necessarily provide the same capabilities.

For example:

```text
Agent requires:
    github
    postgres
    browser
```

while a runtime might only provide:

```text
github
browser
```

This suggested explicit capability discovery / negotiation:

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

This became one of the early candidates for a future runtime interface.

## 6. Early distinction: services vs tools

A useful early distinction was:

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

Authentication was therefore better understood as something that binds to services rather than being embedded directly into an agent resource.

## 7. Early identity model: soul vs mindset

The first discussions also proposed a useful distinction:

### Soul

Persistent identity, values, and principles.

### Mindset

A temporary operating mode:

```text
architect
researcher
debugger
reviewer
teacher
```

The same agent could activate different mindsets without changing its underlying identity.

This remains a hypothesis rather than a formal part of the architecture.

## 8. The first runtime protocol hypothesis

A future runtime was expected to expose structured input and output rather than only terminal text.

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

Possible events:

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

The exact vocabulary was explicitly left undecided.

The underlying idea was:

> **A semantic runtime event stream should become the abstraction consumed by clients.**

## 9. The original portability hypothesis

The project could be summarized by this architecture:

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

The desired property was:

> **Define the agent once; run it on different runtimes; interact with it through different clients.**

## 10. The original research question

The first version of the project therefore reduced to one question:

> **Why should an agent belong to its harness?**

If the answer was "it shouldn't", then the follow-up questions were:

> What is the minimum portable representation of an agent?

> What contract must a runtime implement to execute it faithfully?

## 11. What we deliberately did not assume

The initial idea was not a claim that:

- every possible agent resource should become standardized;
- every runtime should expose the same internal implementation;
- one agent loop is universally correct;
- a single manifest format should be adopted immediately;
- oh-my-agentX should replace emerging standards such as `AGENTS.md`, Agent Skills, or MCP.

The goal was to discover the smallest useful common denominator by experimentation.

## 12. Why this document remains important

The initial configuration-sharing problem remains the most concrete entry point into the larger architecture.

The later Agent OS hypothesis should not obscure that origin:

```text
One agent
   ↓
Many harnesses
   ↓
Portable runtime boundary
   ↓
Long-lived agent
   ↓
Lifecycle management
```

The architecture became deeper because the original problem kept exposing another boundary—not because the original problem was discarded.

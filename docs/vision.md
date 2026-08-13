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

A TUI, IDE, and Web client can render that event differently while consuming the same semantic protocol.

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
session history
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

> Define the agent once; run it on different runtimes; interact with it through different clients.

## 9. What not to build first

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

The point is to discover the true common denominator.

## 10. Position relative to the existing ecosystem

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
```

oh-my-agentX is interested in the missing connections between these pieces.

## 11. The central question

The project can be reduced to one question:

> **Why should an agent belong to its harness?**

If the answer is "it shouldn't", then the next question is:

> What is the minimum portable representation of an agent, and what contract must a runtime implement to execute it faithfully?

That is the research problem.


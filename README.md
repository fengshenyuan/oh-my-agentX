# oh-my-agentX

> One agent, any harness.

**oh-my-agentX** is a personal research project exploring how AI agents can become portable and interoperable across agent clients and runtimes such as Codex, Claude Code, OpenCode, Pi, and future harnesses.

The project started from a simple personal annoyance:

> Why do I have to maintain the same agent identity, instructions, skills, knowledge, preferences, and other configuration separately for every agent client I use?

The bigger question behind that annoyance is much more ambitious:

> **Can an agent be defined independently from the runtime that executes it and the client that presents it to the user?**

This repository is where we explore that question by thinking big and stepping small.

## The initial idea

The working architecture has three major layers:

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

### 1. Agent Definition

A portable, client-neutral representation of an agent. It may eventually cover:

- identity, soul, and personal preferences
- `AGENTS.md`-style instructions and guidelines
- skills
- knowledge and references
- memory and persistent state
- tools and services
- hooks and scripts
- tasks and scheduled jobs
- mindsets and operating modes
- model preferences
- authentication bindings and permissions

The important principle is **single source of truth**: a user should not have to copy the same skills or agent instructions into `.claude/`, `.codex/`, OpenCode, Pi, and other client-specific directories.

### 2. Harness / Runtime

The execution layer between an agent definition and an LLM. It owns the agent loop and the mechanisms around it:

- model invocation
- context construction
- tool calling and execution
- hooks and lifecycle
- permissions and sandboxing
- session and state management
- subagents and orchestration
- event streams
- capability discovery / negotiation

The runtime is **not** the model API. A model API describes how to call intelligence; the runtime also defines how an agent lives, acts, observes, persists state, and exposes structured events to a client.

A central hypothesis of this project is that a future ecosystem may benefit from a **runtime interface / ABI** so that multiple runtimes can execute the same agent definition.

### 3. Shell / Client

The human-facing interaction layer: CLI, TUI, IDE, Web UI, mobile UI, voice UI, and others.

The client should translate user actions into standardized runtime input and render structured runtime events correctly rather than depending on a particular runtime's internal output format.

## Why this matters

Today the ecosystem is becoming increasingly modular, but the boundaries are still fragmented. Standards and projects such as `AGENTS.md`, Agent Skills, MCP, agent package managers, and agent harnesses each solve important pieces.

The long-term question for oh-my-agentX is whether these pieces can compose into a portable agent model:

```text
                         Agent
                           │
                 ┌─────────▼─────────┐
                 │  Agent Definition │
                 │ skills / memory /  │
                 │ identity / tools / │
                 │ knowledge / etc.   │
                 └─────────┬─────────┘
                           │
                    Runtime Interface
                           │
             ┌─────────────┼─────────────┐
             ▼             ▼             ▼
          Runtime A     Runtime B     Runtime C
          (dsh/Cordis)      Pi        OpenCode
             │             │             │
             └─────────────┼─────────────┘
                           │
                    Client Interface
                           │
                  ┌────────┼────────┐
                  ▼        ▼        ▼
                 TUI      IDE      Web
```

The goal is **not** to build another monolithic coding agent. The goal is to investigate the boundaries that would allow different agents, runtimes, and shells to interoperate.

## A major observation: DeepSeek Harness

On 2026-08-13, DeepSeek released [DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness), an open-source TypeScript agent harness built on [Cordis](https://github.com/cordiverse/cordis). Its core architectural claim is:

> **Everything is a plugin.**

DeepSeek's architecture makes the model adapter, tool registry, session log, agent loop, and other runtime pieces replaceable through a plugin/context/event system. Its architecture documentation describes a shared Cordis context, typed events, reversible effects, profiles, bundles, capability seams, durable session events, and the turn/step lifecycle.

This is highly relevant to oh-my-agentX because it is a concrete, serious implementation of much of the **Harness / Runtime** layer we had been theorizing about.

However, an important distinction remains:

```text
DeepSeek Harness / Cordis
    solves composability WITHIN a runtime

oh-my-agentX
    investigates interoperability ACROSS runtimes
```

DeepSeek gives us an important real-world runtime to study and potentially target; it does not by itself define a neutral agent package or a cross-runtime ABI.

See [DeepSeek Harness observations](docs/deepseek-harness.md) and the [Cordis paper](https://github.com/cordiverse/paper).

## What we believe today

These are hypotheses, not standards:

1. **The agent should not belong to the harness.** Codex, Claude Code, Pi, OpenCode, and future runtimes should be replaceable execution environments.
2. **Agent definition and agent state are different things.** Skills and identity are relatively stable; sessions, working memory, tasks, and accumulated state are dynamic.
3. **The runtime needs a structured event model.** A client should receive semantic events such as model output, tool calls, tool results, permission requests, state changes, and errors—not merely a stream of terminal text.
4. **Capabilities need explicit contracts.** Services, tools, filesystems, sandboxes, models, and subagents should be describable and negotiable rather than silently assumed.
5. **The runtime should not dictate the agent philosophy.** A model-driven loop, human-programmed workflow, state machine, planner/executor architecture, or hybrid design should all be possible.
6. **Portability should be layered.** It may be possible to standardize an agent definition without standardizing every runtime implementation detail.
7. **We should not reinvent standards that are already emerging.** `AGENTS.md`, Agent Skills, MCP, and related work should be reused or integrated where appropriate.

## What this repository is for

This is intentionally a research project first and an implementation project second.

We will use the repository to:

- study existing agent ecosystems and emerging standards
- map concepts across Codex, Claude Code, OpenCode, Pi, DeepSeek Harness, and others
- document architectural hypotheses
- prototype portable agent definitions
- build small adapters and interoperability experiments
- identify where a true runtime interface is feasible
- challenge our own assumptions before attempting a formal specification

The first implementation should be small enough to be useful personally. The architecture can grow only when experiments demonstrate that the abstraction is real.

## Current research documents

- [Vision and architecture](docs/vision.md)
- [DeepSeek Harness observations](docs/deepseek-harness.md)

## Status

**Very early research / experimental.**

There is deliberately no claim here that the architecture is complete, correct, or an industry standard. The purpose of this repository is to find out what the smallest useful version of that idea looks like.

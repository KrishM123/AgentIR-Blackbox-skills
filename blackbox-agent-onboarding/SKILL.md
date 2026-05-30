---
name: blackbox-agent-onboarding
description: Use when you need to inspect a repository and onboard it to AgentIR Blackbox by adding supported LangGraph or AI SDK annotations, compile bootstrap, and runtime wiring guidance. LangGraph and AI SDK are first-class code modification paths; unsupported stacks get best-effort Blackbox guidance without invented wrappers or fake compile support.
---

# Blackbox Agent Onboarding

Inspect the repository first. Do not ask the user to restate their stack if
the codebase can answer it.

## Workflow

1. Detect the stack from the repository.
2. If it is LangGraph, read [references/langgraph.md](references/langgraph.md).
3. If it is AI SDK, read [references/ai-sdk.md](references/ai-sdk.md).
4. If it is neither, read [references/generic.md](references/generic.md).
5. Apply only the supported Blackbox onboarding path for the detected stack.
6. Report the compile command, remaining environment variables, and any
   blockers.

## Detect The Stack

Look for direct evidence before making changes:

- LangGraph:
  Python files importing `langgraph`, `StateGraph`, `MessagesState`, or
  existing `agentir_langgraph` symbols
- AI SDK:
  TypeScript or JavaScript files importing `ai`, `tool`, `generateText`,
  `streamText`, or existing `@agentir-annotators/ai-sdk` symbols
- Unsupported:
  repos that make LLM calls but do not match one of the supported annotation
  surfaces above

If multiple stacks are present, scope the work to the user-requested area. If
that area is ambiguous, prefer the smallest coherent integration unit instead
of touching both stacks speculatively.

## Supported Automation Boundary

LangGraph and AI SDK are the only first-class code modification paths.

For those stacks:

- add the official AgentIR annotation dependency
- annotate or wrap only call sites you can map confidently
- add compile bootstrap only when one compile-visible root exists
- keep package-manager changes consistent with the repo

For unsupported stacks:

- do not invent decorators, wrappers, or contract emitters
- do not claim compile support that the repo does not actually have
- provide a concrete manual Blackbox onboarding path instead

## Compile Bootstrap Boundary

Bootstrap means all of the following together:

- `agentir.config.json`
- `.agentir/cli`
- `.github/workflows/agentir-compile.yml`
- a local `agentir:compile` command

Only add or update those files when they are absent or already marked as
AgentIR-managed. If `agentir.config.json` or the compile workflow already
exists and is user-owned, stop and explain the conflict instead of clobbering
it.

If a supported stack has no single compile-visible root, do not guess. Explain
why bootstrap is blocked and what root the user still needs to expose.

## Environment And Runtime Reporting

The repository changes can be prepared without live credentials.

If `BLACKBOX_API_KEY` or `BLACKBOX_URL` are missing:

- still make the safe repository changes
- report the exact compile command:
  `BLACKBOX_API_KEY=... BLACKBOX_URL=... agentir compile --config agentir.config.json`
- say which variables are missing

Always report the runtime contract the user still needs:

- requests go to `$BLACKBOX_URL/v1/chat/completions`
- `Authorization: Bearer $BLACKBOX_API_KEY`
- one stable `rid` for a workflow run
- `node-name` for the current annotated or compiled node

## Boundaries

- Do not execute the target repository's application code just to infer
  support.
- Do not add fallback behavior or speculative abstractions.
- Do not overwrite user-owned Blackbox files.
- Do not describe a workaround as supported automation.

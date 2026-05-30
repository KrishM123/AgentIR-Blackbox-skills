# LangGraph Integration

Use this path only when the repository has direct LangGraph evidence.

## Goal

Make the workflow compile-visible to Blackbox without changing its intended
graph shape.

## What To Add

- Dependency: `agentir-langgraph`
- Imports:
  `from agentir_langgraph.decorators import llm_call, writes`
- Import `GraphProxy` only when needed for the compile-visible contract root

## Annotation Rules

Annotate supported node functions that issue provider calls or clearly
correspond to one LLM node.

Use:

- `@llm_call(...)` for the model-bearing node annotation
- `@writes(...)` only for state keys you can prove from the node's return shape

Populate `@llm_call(...)` from repo evidence:

- `model`: the concrete model id or the most direct stable model binding
- `reads`: state fields the node clearly consumes
- `static_vars`: prompt constants or static prompt context that materially shape
  the request

Do not guess dynamic state fields or model ids.

## GraphProxy And Contract Root

Add `GraphProxy` only when the workflow needs a compile-visible root and one
can be exposed cleanly.

The contract root should resolve to one exported symbol:

- usually `<file>#<proxy_var>` when GraphProxy is required
- otherwise another single compile-visible root only if the integration surface
  already supports it cleanly

If there is no single root, do not bootstrap compile support.

## Compile Bootstrap

When the repo has one compile-visible root and the managed files are absent or
already AgentIR-managed, add:

- `agentir.config.json`
- `.agentir/cli`
- `.github/workflows/agentir-compile.yml`
- a local `agentir:compile` command

The compile command is:

```bash
BLACKBOX_API_KEY=... BLACKBOX_URL=... agentir compile --config agentir.config.json
```

If `agentir.config.json` or the workflow file already exists and is not
AgentIR-managed, stop and report the ownership conflict.

## Runtime Contract

Compile is not enough. The runtime still needs:

- `BLACKBOX_API_KEY`
- `BLACKBOX_URL`
- `Authorization: Bearer $BLACKBOX_API_KEY`
- one stable `rid` across the workflow run
- `node-name` matching the annotated node

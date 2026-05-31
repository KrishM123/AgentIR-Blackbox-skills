# AI SDK Integration

Use this path only when the repository has direct Vercel AI SDK style
evidence.

## Goal

Expose a stable workflow contract from a tool loop or agent loop that would
otherwise be opaque to Blackbox.

## What To Add

- Dependency: `@agentir-annotators/ai-sdk`
- Supported root-package imports:
  `defineAgentIRTool`, `defineToolLoopAgent`, `defineManualAgent`, `llmCall`,
  and `bindBlackboxHeaders`

## Annotation Rules

Wrap supported tool definitions with `defineAgentIRTool(...)` when you can map
one tool identity clearly.

Wrap the single root loop agent with `defineToolLoopAgent(...)` when the repo
exposes one compile-visible root agent.

Use `defineManualAgent(...)` only when the repository already owns a manual
loop. Do not rewrite a real `ToolLoopAgent` into a manual agent just to make it
compile-visible.

Mark supported provider call sites inside tool execution or the loop with
`llmCall(...)`.

Only annotate call sites you can trace confidently from the code. Do not invent
wrapper structure for unsupported abstractions or highly dynamic orchestration.

Do not import annotator internals from package subpaths, sibling `dist/` files,
or sibling `src/` files. All user code should import from the package root
only.

Do not add hidden tool-call registration, local tool planners, or replayed fake
streams in application code. The supported path is endpoint-driven.

## ToolLoopAgent Rules

When the repo already uses AI SDK `ToolLoopAgent` semantics:

- keep the existing OpenAI-compatible model or gateway wrapper if it already
  exists
- forward real `/v1/chat/completions` tool calls and streaming chunks from the
  endpoint
- use `bindBlackboxHeaders(workflowApiKey, headers?)` on outbound requests
- declare explicit `allowedToolSets` that match the real tool-call shapes
- keep `prepareStep(...)` aligned with those sets, typically through
  `activeTools`

The top-level model should not decide tool calls locally. It should pass
through the tool-call payloads returned by the endpoint.

## Tool Body Invariant

If a tool `execute(...)` performs LLM work, the compile-visible `body` markers
must describe that same LLM work.

The `body` exists for compile visibility. It is not a fallback runtime path.

## Contract Root

Compile bootstrap requires exactly one compile-visible root agent.

Use a contract path in the form:

- `<file>#<root_agent_variable>`

If more than one plausible root agent exists, stop and explain the ambiguity
instead of picking one arbitrarily.

## Compile Bootstrap

When exactly one root agent exists and the managed files are absent or already
AgentIR-managed, add:

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

For AI SDK repos that keep their own OpenAI-compatible wrapper, that means:

- requests still go to `$BLACKBOX_URL/v1/chat/completions`
- the wrapper preserves the repo's real `messages`, `tools`, `tool_choice`,
  `stream`, and `stream_options`
- `bindBlackboxHeaders(...)` attaches auth plus the current `rid` and
  `node-name`

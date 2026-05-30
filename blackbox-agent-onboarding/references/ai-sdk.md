# AI SDK Integration

Use this path only when the repository has direct Vercel AI SDK style
evidence.

## Goal

Expose a stable workflow contract from a tool loop or agent loop that would
otherwise be opaque to Blackbox.

## What To Add

- Dependency: `@agentir-annotators/ai-sdk`
- Supported imports:
  `defineAgentIRTool`, `defineToolLoopAgent`, and `llmCall`

## Annotation Rules

Wrap supported tool definitions with `defineAgentIRTool(...)` when you can map
one tool identity clearly.

Wrap the single root loop agent with `defineToolLoopAgent(...)` when the repo
exposes one compile-visible root agent.

Mark supported provider call sites inside the loop with `llmCall(...)`.

Only annotate call sites you can trace confidently from the code. Do not invent
wrapper structure for unsupported abstractions or highly dynamic orchestration.

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

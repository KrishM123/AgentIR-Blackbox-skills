# AgentIR Blackbox Skills

Public install surface for AgentIR Blackbox coding-agent assets.

This repository currently ships one skill:

- `blackbox-agent-onboarding`

It helps Codex or Claude Code inspect an agent repository, detect whether it
uses LangGraph or AI SDK patterns, apply the supported AgentIR annotation
layer, add compile bootstrap when a single compile-visible root exists, and
report the remaining Blackbox runtime wiring.

Unsupported agent stacks get best-effort Blackbox onboarding guidance only.
The skill does not invent framework-specific wrappers or fake compile support
for unsupported systems.

## Install In Codex

```bash
python3 "${CODEX_HOME:-$HOME/.codex}/skills/.system/skill-installer/scripts/install-skill-from-github.py" \
  --repo KrishM123/AgentIR-Blackbox-skills \
  --path blackbox-agent-onboarding
```

Restart Codex after installation so it loads the skill.

If your local Python trust store rejects GitHub during the default download
path, retry with the installer's Git transport:

```bash
python3 "${CODEX_HOME:-$HOME/.codex}/skills/.system/skill-installer/scripts/install-skill-from-github.py" \
  --repo KrishM123/AgentIR-Blackbox-skills \
  --path blackbox-agent-onboarding \
  --method git
```

## Install In Claude Code

```bash
mkdir -p .claude/agents
curl -fsSL \
  https://raw.githubusercontent.com/KrishM123/AgentIR-Blackbox-skills/main/blackbox-agent-onboarding/assets/claude/blackbox-agent-onboarding.md \
  -o .claude/agents/blackbox-agent-onboarding.md
```

Restart the Claude Code session after copying the file.

## Example Prompt

```text
Use $blackbox-agent-onboarding to inspect this repo, add the right Blackbox
annotation layer, bootstrap compile support if a single contract root exists,
and tell me which env vars or runtime headers still need wiring.
```

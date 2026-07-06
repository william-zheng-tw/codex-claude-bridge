---
name: codex-claude-bridge
description: Use when Codex should ask local Claude Code for a read-only second opinion, code review, adversarial review, proposed diff, debugging perspective, or implementation-plan critique through the local `claude` CLI. Trigger when the user asks Codex to ask Claude, use Claude Code, get Claude's review, compare Claude's judgment, ask Claude to propose a patch, or route a focused review to Claude from inside Codex.
---

# Codex Claude Bridge

Use this skill as a thin read-only bridge from Codex to the local Claude Code CLI. Claude is a reviewer or advisor; Codex remains the operator. Do not let Claude directly edit files, run follow-up commands, or apply patches.

## Core Rule

Treat Claude output as external advice. After receiving it, Codex must decide whether the advice is correct before presenting it or acting on it.

## Quick Start

Run the bundled wrapper script from this skill directory:

```bash
python scripts/claude_code_bridge.py setup
python scripts/claude_code_bridge.py ask "Question for Claude"
python scripts/claude_code_bridge.py review
python scripts/claude_code_bridge.py adversarial-review "Focus on rollback and data-loss risks"
python scripts/claude_code_bridge.py propose-diff "Suggest the smallest fix"
```

## Commands

### setup

Use `setup` before the first Claude call in a new environment or when Claude invocation fails. It checks that `claude` is on `PATH`, reports its version when available, and runs a minimal `claude -p` smoke test.

### ask

Use `ask` for a focused second opinion that does not require repository diff context.

```bash
python scripts/claude_code_bridge.py ask "Challenge this migration plan."
```

### review

Use `review` for normal review of the current git working tree. The wrapper collects git status, staged diff, unstaged diff, and diff stats. Untracked files are listed by name only.

```bash
python scripts/claude_code_bridge.py review
```

### adversarial-review

Use `adversarial-review` when the user wants Claude to challenge assumptions, edge cases, safety, rollback, concurrency, data-loss, or reliability risks.

```bash
python scripts/claude_code_bridge.py adversarial-review "Challenge the cache invalidation design"
```

### propose-diff

Use `propose-diff` when the user wants Claude to participate in implementation without granting write access. The wrapper sends the current git context and asks Claude to return a proposed unified diff only. Codex must review and apply any patch manually.

```bash
python scripts/claude_code_bridge.py propose-diff "Fix the parser edge case with the smallest change"
```

## Boundaries

- Keep all Claude calls foreground and read-only.
- Do not add Claude Code flags that grant edit or shell permissions.
- Do not ask Claude to modify files.
- Do not automatically apply Claude's patch suggestions.
- For `propose-diff`, ask Claude to propose a patch as text only; Codex must review and apply it separately.
- Do not execute commands suggested by Claude without Codex reviewing them first.
- Do not use this skill for background job orchestration, session memory, or multi-provider routing.

## Failure Handling

If the wrapper reports that `claude` is missing or unauthenticated, tell the user to install or sign in to Claude Code and rerun `setup`. If `review` reports that the current directory is not a git repository, ask for a repository path or use `ask` with explicit context instead.

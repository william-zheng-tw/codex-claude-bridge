# Roadmap

This roadmap explains how `codex-claude-bridge` can grow from a read-only review bridge into a deeper collaboration workflow while preserving its core safety boundary:

> Codex remains the operator. Claude remains an advisor.

Claude may provide critique, plans, risk analysis, and proposed patches. Codex is responsible for validating suggestions, editing files, running commands, applying patches, and deciding what to trust.

This is a direction document, not a release schedule. Items may change, be split, or be declined if they weaken the operator/advisor boundary.

## Current State

The bridge currently supports:

- `setup`: checks whether the local Claude Code CLI is installed, available on `PATH`, and signed in.
- `ask`: sends Claude a focused read-only question.
- `review`: sends Claude the current git status, diff stats, staged diff, and unstaged diff for review.
- `adversarial-review`: asks Claude to challenge assumptions, edge cases, rollback, data-loss, concurrency, and reliability risks.
- `propose-diff`: asks Claude to return a proposed unified diff as text only. Codex must review and apply it manually.

The current implementation deliberately avoids granting Claude write access, shell access, or automatic patch application.

## Phase 1: Patch Advisor Workflow

Goal: make Claude's proposed diffs easier and safer for Codex to inspect.

Planned improvements:

- Add a read-only `validate-diff` command that accepts a patch from stdin or a file, exits non-zero on validation failure, and never applies changes.
- Treat raw unified diffs as the primary input and reject missing, ambiguous, prose-mixed, multi-block, or malformed patch output.
- Validate patch syntax and applicability with a non-mutating check such as `git apply --check`.
- Require touched paths to be repo-relative and apply best-effort blocking for absolute paths, parent traversal, `.git/`, and sensitive-looking paths.
- Emit clear machine-testable failure categories, such as `no-diff`, `ambiguous-diff`, `malformed-diff`, `blocked-path`, and `not-applicable`.

Expected workflow:

```bash
python3 scripts/claude_code_bridge.py propose-diff "Fix the failing parser test" > proposed.patch
python3 scripts/claude_code_bridge.py validate-diff < proposed.patch
```

Codex then decides whether to apply the patch, run tests, ask for another review, or discard the suggestion.

Validation hooks:

- Unit-test diff parsing and path classification without invoking Claude.
- CLI-test `validate-diff` with valid, malformed, prose-mixed, sensitive-path, new-file, delete-file, and rename patches.

## Phase 2: Review-After-Apply Loop

Goal: support a tighter human/Codex/Claude iteration cycle without changing the permission boundary.

Planned workflow:

1. Codex gathers the task, relevant repo context, and current git diff.
2. Claude proposes a patch or implementation plan.
3. Codex validates and applies accepted changes.
4. Codex runs tests, lint, type checks, or targeted verification.
5. Claude reviews the applied diff and any failing output.
6. Codex decides the next action.

Possible commands:

```bash
python3 scripts/claude_code_bridge.py critique-applied "Focus on regression risk"
python3 scripts/claude_code_bridge.py repair-advice "Explain this test failure and suggest the smallest fix"
```

This keeps Claude useful during implementation while making Codex responsible for actual edits and verification.

Before adding `repair-advice`, define how Codex captures, truncates, and redacts verification output before sending it to Claude.

## Phase 3: Sandbox Proposal Workflow

Goal: explore deeper patch collaboration in an isolated workspace.

In this model, Codex may create and manage a temporary git worktree or repository copy, provide selected read-only context to Claude, and ask Claude for text-only implementation guidance or a proposed diff. Claude still does not receive write access or command execution authority, even in the sandbox.

Planned safety properties:

- Claude does not modify any working tree directly, including the sandbox and the main workspace.
- Codex controls which repository state and files are exposed.
- Codex imports only reviewed diffs from the sandbox.
- Proposed changes pass the same path, secret, and patch validation gates as Phase 1.

Possible command:

```bash
python3 scripts/claude_code_bridge.py sandbox-propose "Refactor the parser with minimal behavior changes"
```

This phase is intentionally later because it introduces more moving parts than the plain patch-advisor flow.

## Phase 4: Policy-Gated Collaboration Mode

Goal: define a higher-level collaboration mode without creating an unrestricted autonomous agent.

A policy-gated mode could let Claude propose:

- implementation plans
- patch sets
- commands Codex may consider running, surfaced as text only and never executed automatically
- follow-up reviews
- risk assessments

Codex would still enforce policy gates before acting:

- which file contents Codex may expose to Claude as read-only context
- which target paths Codex may modify when applying an accepted proposal
- whether new files are allowed
- whether deletes are allowed
- whether Codex may consider dependency changes
- whether Codex may consider network or install commands
- which verification commands are acceptable

This mode should remain proposal-oriented. Claude can recommend actions; Codex executes only after review.

## Non-Goals

The project is not trying to provide:

- direct Claude writes to the main workspace
- automatic application of Claude-generated patches
- autonomous Claude command execution
- execution of Claude-proposed commands
- silent extraction or transformation of Claude output into apply-ready patches
- bypassing Codex or user review for file edits, command execution, dependency installation, or network access
- background job orchestration with unrestricted repository access
- persistent multi-provider session memory
- a replacement for Codex's judgment

These constraints are intentional. The value of this bridge is deeper model collaboration with a clear operator/advisor boundary.

## Contribution Ideas

Useful contributions include:

- robust unified-diff parsing and validation
- path allowlist and denylist support
- better redaction and sensitive-file detection
- focused tests for failure modes
- examples of safe Codex-driven collaboration loops
- documentation for real-world workflows

The highest-priority contributions are small, well-tested Phase 1 improvements: diff validation, sensitive-path blocking, redaction tests, and clear error messages. Security-relevant parsing and redaction changes require extra review.

When proposing new features, please explain how the feature preserves the boundary that Codex operates and Claude advises.

# agent-team-theory

Generation-grammar agent team composition for Claude Code.

## What This Does

Given a task, this plugin runs a first-principles pipeline:

```text
Task -> Type (Sigma) -> Derive pressures (Delta) -> Generate obligations (Req) -> Compute minimal cover (Gamma) -> Realize as team -> Evaluate
```

Based on `references/AGENT_TEAM_THEORY.md`.

## Installation

```bash
claude plugins add /absolute/path/to/agent-team-theory
```

## Skills

- `/team` (or `agent-team-theory:team`) — analyze task, confirm structure with user, self-assign role, execute

## Agents

| Agent | Obligations | Role |
|---|---|---|
| operator | omega_par, omega_seq, omega_variety | Task execution |
| coordinator | omega_dispatch, omega_exception, omega_boundary, omega_g_rule | Dispatch, boundaries, exceptions, governance rules |
| evaluator | omega_verify, omega_select | Verify, select, adversarial, audit (4 modes) |
| rewriter | omega_rewrite | Rule modification proposals (DP5 double-loop) |

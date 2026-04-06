# agent-team-theory-v2

Generation-grammar agent team composition for Claude Code.

## What This Does

Given a task, this plugin runs a first-principles pipeline:

```
Task → Type (Σ) → Derive pressures (Δ) → Generate obligations (Req) → Compute minimal cover (Γ) → Realize as team → Evaluate
```

Based on `AGENT_TEAM_THEORY_SINGLE_VOLUME.md`.

## Installation

```bash
claude plugins add /path/to/skill-v2
```

## Skills

- `/team` (or `agent-team-theory-v2:team`) — full orchestration
- `/team-analyze` (or `agent-team-theory-v2:team-analyze`) — analysis only

## Agents

| Agent | Obligations | Role |
|---|---|---|
| operator | ω_par, ω_seq, ω_variety | Task execution |
| coordinator | ω_dispatch, ω_exception, ω_boundary | Dispatch, boundaries, exceptions |
| evaluator | ω_verify, ω_select | Verify, select, adversarial, audit (4 modes) |
| rewriter | ω_rewrite, ω_g_rule | Rule modification (DP5 double-loop) |

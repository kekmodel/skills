---
name: autoresearch
description: Set up and run an autonomous experiment loop for any optimization target. Supports solo mode (single agent) and swarm mode (strategist + workers with worktree isolation). Use when asked to "run autoresearch", "optimize X in a loop", "set up autoresearch for X", "start experiments", or "autoresearch swarm".
---

# Autoresearch

Autonomous experiment loop: try ideas, keep what works, discard what doesn't, never stop.
Two modes: **solo** (single agent) and **swarm** (strategist + parallel workers).

---

## Mode Selection

- `/autoresearch <goal>` → solo mode (single agent, simple loop)
- `/autoresearch swarm <goal>` → swarm mode (strategist + 1-3 workers in worktrees)
- `/autoresearch resume` → resume existing session (reads autoresearch.md + log)
- `/autoresearch off` → stop and summarize

---

## Solo Mode

### Setup

1. Ask (or infer): **Goal**, **Command**, **Metric** (name + unit + direction: lower/higher), **Files in scope**, **Constraints**.
2. `git checkout -b autoresearch/<goal>-<date>`
3. Read source files in scope. Understand deeply before writing anything.
4. Write `autoresearch.md`, `autoresearch.sh`, optionally `autoresearch.checks.sh`. Commit all.
5. Set log path: `AUTORESEARCH_LOG="${AUTORESEARCH_HOME:-$HOME/.autoresearch}/$(basename $(pwd))/experiments.jsonl"` — create dirs if needed.
6. Run baseline → record in log → start looping.

### The Solo Loop

**LOOP FOREVER:**

1. Read `autoresearch.md` for context.
2. Choose an experiment idea based on code understanding + past results.
3. Modify in-scope file(s).
4. `git commit -m "<description>"`
5. `bash autoresearch.sh > experiment.log 2>&1`
6. `grep "^METRIC " experiment.log`
7. No METRIC → crash. `tail -50 experiment.log` to diagnose.
8. Append to log:
   ```json
   {"commit":"<7char>","base_commit":"<branch_base>","metric":<val>,"metrics":{},"status":"keep|discard|crash|checks_failed","description":"<what>","signature":"<readable_sig>","timestamp":<epoch>,"confidence":<val>,"insights":"<what learned>"}
   ```
9. Improved → **keep**. Equal/worse → **discard** (`git reset --hard HEAD~1`).
10. checks.sh exists and failed → **checks_failed** (revert).
11. Every 5-10 experiments, update autoresearch.md "What's Been Tried".
12. Go to 1.

---

## Swarm Mode

### Architecture: Centralized Critic, Decentralized Actors

```
strategist (opus, main worktree)
  ├── owns incumbent_commit + autoresearch.md + experiments.jsonl
  ├── assigns bounded tasks via TaskCreate
  ├── promotes wins after clean-base verification
  │
  ├── worker-1 (sonnet, isolated worktree)
  │   └── bounded search: e.g., "depth 10-14, 12 trials"
  ├── worker-2 (sonnet, isolated worktree)
  │   └── bounded search: e.g., "LR sweep 0.02-0.08"
  └── worker-3 (sonnet, isolated worktree)
      └── bounded search: e.g., "optimizer experiments"
```

### Swarm Setup

1. Same as solo setup (goal, command, metric, files, constraints).
2. Create session files (autoresearch.md, autoresearch.sh, autoresearch.checks.sh).
3. Set shared log: `${AUTORESEARCH_HOME:-$HOME/.autoresearch}/<project>/experiments.jsonl`
4. Ask (or decide): How many workers? What directions?
5. Spawn the strategist agent:
   ```
   Agent(subagent_type: "strategist")
   ```
   The strategist then spawns workers and runs the swarm.

### Shared State

| Artifact | Location | Owner | Workers |
|----------|----------|-------|---------|
| autoresearch.md | repo root | strategist writes | workers read only |
| experiments.jsonl | ~/.autoresearch/\<project\>/ | strategist appends | workers read, send results via message |
| autoresearch.sh | repo root | created at setup | all execute |
| autoresearch.checks.sh | repo root (optional) | created at setup | all execute |
| git worktrees | auto-created | one per worker | isolated |

### Communication Protocol

Workers → Strategist:
```
STARTED    task_id=<id> direction=<dir> base_commit=<sha>
RESULT     task_id=<id> candidate_commit=<sha> base_commit=<sha> family=<dir> signature=<sig> metric=<val> confidence=<val> status=<keep|discard|crash> insights=<text>
CANDIDATE  task_id=<id> candidate_commit=<sha> metric=<val> delta=<improvement> confidence=<val>x signature=<sig>
BLOCKED    task_id=<id> reason=<text>
DONE       task_id=<id> direction=<dir> trials=<n> best_delta=<val> summary=<text>
```

Strategist → Workers:
```
NEW_INCUMBENT commit=<sha>
CANCEL        task_id=<id> reason=<text>
```

### Promotion Protocol

1. Worker reports CANDIDATE (confidence >= 1.5x)
2. Strategist checks confidence:
   - >= 2.0x → proceed to verification
   - 1.0-2.0x → ask worker to re-run
   - < 1.0x → discard
3. Strategist verification: checkout incumbent, apply minimal diff from candidate_commit, rerun benchmark
4. If confirmed: land diff, update incumbent_commit in autoresearch.md
5. Broadcast NEW_INCUMBENT to all workers
6. Workers use new incumbent for their NEXT task (no mid-flight rebase)

### Dedup

Before starting an experiment, check experiments.jsonl for matching signatures:
```
family=depth|base=abc1234|depth=12|lr=0.04|batch=262144
```
Include base_commit in the key. If code changes are significant, add variant field.

### Swarm Rules

- **Strategist is the single source of truth** for incumbent and experiment log
- **Workers NEVER merge** to main branch — only strategist promotes
- **Workers NEVER edit** autoresearch.md — only strategist updates it
- **Workers NEVER write** directly to experiments.jsonl — send RESULT messages
- **Bounded tasks only** — every assignment has direction + budget + early stop
- **No mid-flight rebase** — workers finish current experiment, then adopt new incumbent
- **NEVER STOP** — strategist keeps directing until manually interrupted

---

## Session Files

### `autoresearch.md`

```markdown
# Autoresearch: <goal>

## Objective
<What we're optimizing and the workload.>

## Incumbent
commit: <sha>
metric: <best_value>

## Metrics
- **Primary**: <name> (<unit>, lower/higher is better)
- **Secondary**: <name>, ... — tradeoff monitors

## How to Run
`./autoresearch.sh` — outputs `METRIC name=number` lines.

## Files in Scope
<Files the agent may modify, with notes.>

## Off Limits
<What must NOT be touched.>

## Constraints
<Hard rules.>

## What's Been Tried
<Key wins, dead ends, insights. Updated every 5-10 experiments.>

## Ideas Backlog
<Promising untried ideas. Prune as tested.>
```

### `autoresearch.sh`

```bash
#!/bin/bash
set -euo pipefail
# Pre-check (<1s)
python -c "import ast; ast.parse(open('train.py').read())"
# Run workload
uv run train.py > run.log 2>&1
# Output structured metrics
grep "^val_bpb:" run.log | sed 's/.*: */METRIC val_bpb=/'
grep "^peak_vram_mb:" run.log | sed 's/.*: */METRIC peak_vram_mb=/'
```

### `autoresearch.checks.sh` (optional)

```bash
#!/bin/bash
set -euo pipefail
pytest tests/ -x -q 2>&1 | tail -20
```

---

## Confidence Scoring (MAD)

After 3+ experiments:
```
noise_floor = MAD(all metric values in session)
best_improvement = |best_metric - baseline_metric|
confidence = best_improvement / noise_floor
```

- >= 2.0x 🟢 likely real
- 1.0-2.0x 🟡 marginal, re-run to confirm
- < 1.0x 🔴 within noise

---

## Loop Rules (Both Modes)

**LOOP FOREVER.** Never ask "should I continue?"

- **Primary metric is king.** Improved → keep. Worse/equal → discard.
- **Annotate with insights.** What was learned, not just what was done.
- **Simpler is better.** Removing code for equal perf = keep.
- **Don't thrash.** Same idea failing? Try structurally different.
- **Think when stuck.** Re-read source, reason about hardware, try radical changes.
- **Resuming:** read autoresearch.md + experiments.jsonl + git log, continue.

**NEVER STOP.**

---

## Example Domains

| Domain | Metric | Direction | Command |
|--------|--------|-----------|---------|
| LLM training | val_bpb | lower | `uv run train.py` |
| Test speed | seconds | lower | `pytest tests/` |
| Bundle size | KB | lower | `npm run build && du -sb dist` |
| Model accuracy | accuracy | higher | `python eval.py` |
| RL reward | mean_reward | higher | `python train_rl.py` |
| Inference speed | tok/sec | higher | `python benchmark.py` |
| Build speed | seconds | lower | `make build` |

---

## Configuration

Set via environment variables:
- `AUTORESEARCH_HOME` — base directory for logs (default: `~/.autoresearch`)
- `AUTORESEARCH_WORKERS` — number of swarm workers (default: 2)
- `AUTORESEARCH_WORKER_MODEL` — model for workers (default: sonnet)

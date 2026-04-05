---
name: team-analyze
description: "Analyze a task and recommend the best supported team architecture without creating a team or executing anything."
user_invocable: true
---

# Team Analyze

You analyze a task and recommend the best supported architecture for the current Claude Code substrate.

This skill is **analysis only**.

Do NOT:
- create a team
- spawn agents
- ask for execution approval
- continue into execution after the report

## Substrate Constraints

- In Claude Code, only the MAIN/orchestrator agent can create teams.
- This skill recommends from the supported execution set: **A0-A5, A7, A8**.
- **A6 (Swarm)** is useful conceptually for large homogeneous pooled work, but is **not supported for direct execution here**. If A6 is the best conceptual fit, say so and recommend the closest supported fallback, usually A3 or A8.

## Agent Primitives

- **Teammate**: persistent agent within a team (Agent + SendMessage via TeamCreate). Used for all team members.
- **Subagent**: one-shot agent without team membership. Used for pre-team task analysis and by teammates internally.

---

## Step 1: A0 Fast-Path Check

Quick scan before deep analysis:
- Is this a single, self-contained task?
- Can one agent handle it without meaningful coordination overhead?
- Does it look like less than about 30 minutes of focused work?
- Is verification straightforward enough that failure would be obvious quickly?
- If it goes wrong, are the effects easy to undo, contain, or recover from?

If **all** answers are yes → recommend **A0 (solo)**. If any answer is no or unclear, do the full analysis instead of taking the fast path.

**Principle:** Never pay coordination cost you can't justify.

---

## Step 2: 3-Axis Analysis

Assess the task on three independent axes:

**Axis 1 — Interdependence Type**

| Signal | Type |
|--------|------|
| Subtasks share no state, can run independently | **Pooled** |
| Output of A feeds into B feeds into C | **Sequential** |
| A and B must iterate on shared state, mutual feedback | **Reciprocal** |
| Same task should be attempted independently, then selected or ranked after the fact | **Replicated** |

Proxies for code tasks: import graph coupling, test co-occurrence, shared config.
Proxies for knowledge tasks: sub-question dependency, argument chain structure.

**Axis 2 — Scale × Heterogeneity**

| Scale | Heterogeneity | Meaning |
|-------|---------------|---------|
| Small (<3 subtasks) | Low | One person could do it, just needs help |
| Medium (3-8) | Medium | Multiple domains or perspectives needed |
| Large (8+) | High | Multi-domain, requires deep specialization |

Proxies: file count, domain count, required expertise breadth.

**Axis 3 — Uncertainty**

| Level | Signal |
|-------|--------|
| Low | Concrete spec, known solution pattern, clear success criteria |
| Medium | Some ambiguity, multiple valid approaches, partial spec |
| High | Exploratory, no clear solution, subjective quality |

**Separate assessment — Verification Risk** (not part of axis 3):
- How hard is it to verify the output is correct?
- 0.0 (trivial — run tests) to 1.0 (impossible — subjective/novel)

**Separate assessment — Reversibility / Externality**:
- If this goes wrong, how costly is it to undo, contain, or recover from it?
- Would the side effects stay local, or propagate to users, systems, data, or downstream work?
- Low reversibility or high externality means the recommendation should be more conservative even if the architecture fit looks simple.

---

## Step 3: Confidence Check

Rate your analysis confidence as **Low / Medium / High**:
- **Low** (roughly < 0.3): ask the user for clarification instead of forcing a recommendation
- **Medium** (roughly 0.3-0.5): prefer the lowest-cost architecture with clear escalation; start simple, but do not force A0 if A1 or A8 makes risk boundaries clearer
- **High** (roughly ≥ 0.5): proceed normally

State why confidence is not high:
- missing task facts
- unclear success criteria
- hidden dependencies
- subjective quality bar
- uncertain verification path

**Principle:** Under uncertainty, prefer a low-cost structure with clear escalation over a high-cost structure with weak rationale.

---

## Step 4: Candidate Narrowing

**Axis 1 → Base candidates:**

| Interdependence | Candidates |
|----------------|------------|
| Pooled | A3, A8 |
| Sequential | A2 (if sequence is stable), A1/A8 (if not) |
| Reciprocal | A1 (small), A4 (medium), A5 (large) |
| Replicated | A7 |

**A6 note:** A6 (Swarm) exists in the architecture taxonomy but is NOT directly executable here. If A6 is the best conceptual fit, disclose the mismatch and choose the closest supported fallback (usually A3 or A8).

**Axis 2 → Refine:**

| Scale × Heterogeneity | Adjustment |
|-----------------------|------------|
| Small, low | Prefer A1 or A0 |
| Medium, medium | A4, A8 |
| Large, high | A5, A8 |
| Large, homogeneous pooled work | Theoretical fit: A6 (unsupported here) → recommend A3/A8 with disclosure |
| Quality-first, multiple plausible approaches | A7 viable |

**Axis 3 → Overlay:**

| Uncertainty | Overlay |
|------------|---------|
| Low | DP9 (interface standardization: explicit handoff schema, typed outputs, clear success shape) |
| Medium | DP10 (exception queue: isolate ambiguous cases, keep clear cases moving, escalate the queue later) |
| High | DP10 + DP5 (double-loop rule revision: revisit the governing rule, not just the latest mistake) + consider DP3 (adversarial review: actively search for hidden flaws) |

---

## Step 5: Quality/Verification Overlays

- **Verification risk ≥ 0.5** → Add DP6 (delegate-verify: separate producer from checker) or A7 overlay
- **Verification risk ≥ 0.7** → Add DP3 (adversarial review: ask a reviewer to attack the proposal, not just inspect it)
- **Low reversibility or high externality** → Tighten approval, prefer independent verification before irreversible actions, use staged execution where possible
- **Multiple domains** → Check DP8 (weak tie exploration: add one external perspective with a meaningfully different brief) if capability gap exists
- **Long-running** → Add DP5 (double-loop meta-agent: update the rules when the same exception keeps recurring)

**Overlay agents for the team composition preview:**

| Overlay | Agent | When |
|---------|-------|------|
| DP3 | r3-adversary ×1 | High-stakes, hidden assumptions, or to pressure-test the proposal |
| DP5 | r1-legislator ×1 | Long-running work where repeated exceptions suggest the rules need revision |
| DP6 | r3-judge ×1 | High verification risk; producer should not grade its own work |
| DP8 | r2-executor ×1 (different brief) | Capability gap, OOD, or team needs one meaningfully different viewpoint |

When two architectures both look plausible, prefer the one that localizes mistakes more cleanly, makes escalation more obvious, and keeps the fewest assumptions hidden inside one agent.

---

## Step 6: Anti-Pattern Guard

Before finalizing, check all six:

| Check | Anti-Pattern | Fix |
|-------|-------------|-----|
| Team size > 5? | AP2 (coordination explosion) | Reduce or hierarchize (A5) |
| All agents same model/prompt? | AP1 (groupthink) | Diversify prompts or approaches |
| Reviewer shares executor model, context, or rubric? | AP13 (mirror verification) | Re-isolate reviewer; prefer different model, otherwise use separate prompt/rubric/evidence path |
| No escalation path? | AP9 (ghost governance) | Add explicit L1-L4 escalation |
| Deep delegation chain? | AP6 (infinite delegation) | Cap at 2 levels |
| Independent tasks split as dependent? | AP11 (false modularity) | Re-analyze interdependence |

---

## Step 7: Report

Present all ten sections to the user, then STOP. Do not create a team or spawn agents.

1. **3-axis analysis** — Interdependence type, Scale × Heterogeneity, Uncertainty level (with proxy evidence cited)
2. **Verification risk** — score (0.0–1.0) and why
3. **Reversibility / externality assessment** — what happens if this goes wrong and how far side effects spread
4. **Analysis confidence** — Low/Medium/High, and why it is not higher
5. **Recommended architecture and why** — name the pattern AND explain the concrete behavior it implies
6. **Fallbacks considered** — alternatives you rejected and the reason each was ruled out
7. **Overlay suggestions** — specific DPs with the concrete behavior each implies (do not just list the label)
8. **Team composition preview** — roles × instances × models

   Role-architecture mapping:

   | Architecture | Roles | Instance Pattern |
   |-------------|-------|-----------------|
   | A0 Solo | (orchestrator alone) | — |
   | A1 Simple | R2 workers | 1-3 R2; orchestrator acts as leader + R3 |
   | A2 Pipeline | R2 per stage | N R2 (one per stage); direct chain handoff |
   | A3 Network | R2 specialists | N R2 (independent); star from orchestrator |
   | A4 Adhocracy | R2 peers | 2-4 peers (symmetric); mesh comms |
   | A5 Divisional | R1 + R6 + R2 | hierarchy, max 2 levels |
   | A7 Replicate-Select | R2 replicas + R3 judge | N R2 (isolated from each other) + 1 R3 (independent context) |
   | A8 Federation | R6 + R2 specialists + R3 | R6 routes R2s; R3 verifies independently |

   Model defaults: R1/R3 Judge → opus/high; R2 → sonnet/high (→ opus if reciprocal interdependence or high uncertainty); R3 Adversary/R4 Auditor → opus/max. R3/R4 MUST have an independent error mode from executors — prefer a different model; if same model, isolate context, rubric, and evidence path.

9. **Risks and caveats** — assumptions that would change the recommendation, including verification risk and reversibility concerns
10. **Decision path taken** — which branches you followed through the axes and why

If A6 is the best conceptual fit, explicitly state: why A6 fits in theory, why it is not directly supported here, and which supported fallback you recommend instead.

---

## Quick Reference: Selection Decision Tree

```
Task received
├── Simple, self-contained? → A0 Solo
├── Interdependence?
│   ├── Pooled
│   │   ├── Large, homogeneous batch work? → A6 in theory; unsupported here → A3 or A8 with disclosure
│   │   ├── Need specialist routing? → A8 Federation
│   │   └── Independent experts, no routing needed? → A3 Network
│   ├── Sequential
│   │   ├── Stable sequence? → A2 Pipeline
│   │   └── Unstable? → A1 or A8
│   ├── Reciprocal
│   │   ├── Small team? → A1 Simple
│   │   ├── Medium, creative? → A4 Adhocracy
│   │   └── Large, multi-domain? → A5 Divisional
│   └── Replicated
│       └── Same task, multiple independent attempts + post-hoc selection? → A7 Replicate-Select
├── High verification risk (≥ 0.5)? → Add DP6 or A7 overlay
├── Very high verification risk (≥ 0.7)? → Add DP3 adversary
├── High stakes? → Add DP3 adversary
├── Long-running? → Add DP5 meta-agent
└── Low confidence? → Ask for clarification or choose lowest-cost structure with clear escalation
```

---

## Key Principles

- Don't pay coordination cost you can't justify. A0 is always available, but not always the right fast path.
- A0 fast paths are only for work that is easy to verify and easy to recover from if wrong.
- The user has final authority over whether to execute, change, or cancel the plan.
- Keep decisions where the relevant information is. Don't over-centralize by default.
- Use A7 when quality depends on multiple independent attempts plus post-hoc selection.
- Escalate when uncertain. Better to pause than to guess badly.
- Judges and verifiers must be independent from executors. Same-model verification is acceptable only with isolated context and a separate rubric/evidence path.
- Labels like DP3, DP5, DP8, DP9, and DP10 are reminders for concrete behaviors — not substitutes for explaining the actual behavior you want.
- When reporting a recommendation, substance beats taxonomy. Explain the behavior you want even if you also name the pattern.

This file is an analyze-only entry point, but it teaches the same judgment habits as `/team`: do not invent a separate decision policy. If you use a pattern label, also explain the concrete behavior it implies.

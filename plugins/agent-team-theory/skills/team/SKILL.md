---
name: team
description: "Execute any task with the best supported agent team for the current Claude Code substrate, and support analyze-only previews when requested."
user_invocable: true
---

# Agent Team Orchestrator

You are the orchestrator. Given a task, you analyze it using the rules in this file, select the best SUPPORTED architecture for the current Claude Code substrate, and execute it when the user wants execution.

**No Python dependency. No learning loop. Pure rule-based composition.**

### Substrate Constraints

- In Claude Code, only the MAIN/orchestrator agent can create teams.
- This skill therefore supports direct execution of **A0-A5, A7, and A8**.
- **A6 (Swarm)** is part of the framework but is intentionally NOT executed by this skill. A faithful leaderless swarm is a poor fit for a substrate where team creation authority is centralized in the main agent.
- The Python harness can approximate A6 with shared-state waves and file locking, but that is a **LOW-FIDELITY** approximation and is intentionally excluded from this user-facing skill.
- If the user asks for a recommendation, dry run, preview, or analyze-only mode, run **Phase 1 + Phase 2 only** and STOP before team creation.

### Agent Primitives

- **Teammate**: persistent agent within a team (Agent + SendMessage via TeamCreate). Used for all team members.
- **Subagent**: one-shot agent without team membership. Used for pre-team task analysis and by teammates internally for their own subtasks.

Team members MUST be Teammates. Subagents are permitted outside of and within team scope.

### Communication Model

Agents with SendMessage can talk to each other DIRECTLY by name — not just through you.

| Role | SendMessage | Direct comms | Rationale |
|------|-------------|-------------|-----------|
| R1 Legislator | ✓ | peers, R2, R6 | Governance requires direct coordination |
| R2 Executor | ✓ | peers, R1, R6 | Peer coordination, report to dispatcher |
| R3 Judge | ✗ | orchestrator only | Independence — AP13 |
| R3 Adversary | ✗ | orchestrator only | Independence — AP13 |
| R4 Auditor | ✗ | orchestrator only | Independence — AP13 |
| R6 Dispatcher | ✓ | R2, R1, orchestrator | Direct routing and assignment |

**R3/R4 are intentionally isolated.** They receive inputs from you (orchestrator) only. This prevents executor influence on verification (AP13).

**R1/R2/R6 form a direct-communication mesh.** This breaks the star topology for execution, enabling true hierarchies and peer coordination.

---

## Phase 1: Task Analysis

### Step 1: A0 Fast-Path Check

Before deep analysis, quick scan:
- Is this a single, self-contained task?
- Can one agent handle it without coordination overhead?
- Estimated effort < 30 minutes of work?
- Is verification straightforward enough that you will know quickly if the result is wrong?
- If it goes wrong, are the effects easy to undo, contain, or recover from?

If ALL yes → **A0 (solo)**. If any answer is "no" or unclear, do the full analysis instead of taking the fast path. If execution is desired, skip to Phase 3. If the user asked for analyze-only, report the A0 recommendation and STOP. Do not over-engineer.

**Principle:** Never pay coordination cost you can't justify.

### Step 2: 3-Axis Analysis

Assess the task on 3 independent axes:

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
- Low reversibility or high externality means execution should be more conservative even if the architecture fit looks simple.

### Step 3: Confidence Check

Rate your analysis confidence as **Low / Medium / High**.
- **Low** (roughly < 0.3) → **Ask the user** for clarification. Do not guess.
- **Medium** (roughly 0.3-0.5) → **Prefer the lowest-cost architecture with clear escalation.** Start simple, but do not force A0 if A1 or A8 makes risk boundaries clearer.
- **High** (roughly ≥ 0.5) → Proceed with architecture selection.

State **why** confidence is low or medium:
- missing task facts
- unclear success criteria
- hidden dependencies
- subjective quality bar
- uncertain verification path

**Principle:** Under uncertainty, prefer a low-cost structure with clear escalation over a high-cost structure with weak rationale.

---

## Phase 2: Architecture Selection

### Step 4: Candidate Narrowing

Use the 3-axis matrix to narrow candidates:

**Axis 1 → Base candidates:**

| Interdependence | Candidates |
|----------------|------------|
| Pooled | A3, A8 |
| Sequential | A2 (if sequence is stable), A1/A8 (if not) |
| Reciprocal | A1 (small), A4 (medium), A5 (large) |
| Replicated | A7 |

**A6 note:** A6 (Swarm) exists in the architecture taxonomy, but this skill does NOT execute it. Claude Code centralizes team creation in the main agent, so only a low-fidelity swarm approximation would be possible here. If A6 is the best conceptual fit, disclose the mismatch and choose the closest supported fallback (usually A3 or A8).

**Axis 2 → Refine:**

| Scale × Heterogeneity | Adjustment |
|-----------------------|------------|
| Small, low | Prefer A1 or A0 |
| Medium, medium | A4, A8 |
| Large, high | A5, A8 |
| Large, homogeneous pooled work | Theoretical fit: A6 (unsupported here) → use A3/A8 with disclosure |
| Quality-first, multiple plausible approaches | A7 viable |

**Axis 3 → Overlay:**

| Uncertainty | Overlay |
|------------|---------|
| Low | DP9 (interface standardization: explicit handoff schema, typed outputs, clear success shape) |
| Medium | DP10 (exception queue: isolate ambiguous cases, keep clear cases moving, escalate the queue later) |
| High | DP10 + DP5 (double-loop rule revision: revisit the governing rule, not just the latest mistake) + consider DP3 (adversarial review: actively search for hidden flaws) |

### Step 5: Quality/Verification Overlays

- **Verification risk ≥ 0.5** → Add DP6 (delegate-verify: separate producer from checker) or A7 overlay
- **Verification risk ≥ 0.7** → Add DP3 (adversarial review: ask a reviewer to attack the proposal, not just inspect it)
- **Low reversibility or high externality** → Tighten approval, prefer independent verification before irreversible actions, and use staged execution where possible
- **Multiple domains** → Check DP8 (weak tie exploration: add one external perspective with a meaningfully different brief) if capability gap exists
- **Long-running** → Add DP5 (double-loop meta-agent: update the rules when the same exception keeps recurring)

When two architectures both look plausible, prefer the one that:
- localizes mistakes more cleanly
- makes escalation more obvious
- keeps the fewest assumptions hidden inside one agent

### Step 6: Anti-Pattern Guard

Before finalizing, check:

| Check | Anti-Pattern | Fix |
|-------|-------------|-----|
| Team size > 5? | AP2 (coordination explosion) | Reduce or hierarchize (A5) |
| All agents same model/prompt? | AP1 (groupthink) | Diversify prompts/approaches |
| Reviewer shares executor model, context, or rubric? | AP13 (mirror verification) | Re-isolate reviewer; prefer a different model, otherwise use a separate prompt/rubric/evidence path |
| No escalation path? | AP9 (ghost governance) | Add L1-L4 rules |
| Deep delegation chain? | AP6 (infinite delegation) | Max 2 levels |
| Independent tasks split as dependent? | AP11 (false modularity) | Re-analyze interdependence |

### Step 7: Recommendation / Approval Gate

If the user asked for **analyze-only**, **preview**, **dry run**, or **recommendation only**:
- Present the report below and STOP.
- Do NOT create a team.
- Do NOT spawn agents.
- Do NOT ask for execution approval.

Present to the user:
1. Task analysis (3-axis results + confidence)
2. Selected architecture + why
3. Overlay bundle + why
4. Team composition (roles × instances × models)
5. Estimated cost (relative: low/medium/high)
6. Fallbacks considered and why they were rejected
7. Decision path taken
8. Risks and limitations, including verification risk, reversibility/externality concerns, and assumptions that would change the recommendation

If execution is desired, user can: approve, change architecture, modify overlays, re-analyze, cancel.

---

## Phase 3: Team Composition & Execution

### Step 8: Create Team (Required)

Before spawning any agents, create a team via TeamCreate:
```
TeamCreate: {team_name: "<task-slug>", description: "<task summary>"}
```

**This is non-negotiable.** Agents can only communicate directly with each other within a team. Without TeamCreate, agents have NO SendMessage — they become isolated subagents.

All subsequent Agent spawns MUST include `team_name` parameter matching this team.

### Role-Architecture Mapping

| Architecture | Roles | Instance Pattern | Communication |
|-------------|-------|-----------------|---------------|
| A0 Solo | (you) | — | — |
| A1 Simple | R2 workers | 1-3 R2, you as leader+R3 | Star (you → R2) |
| A2 Pipeline | R2 per stage | N R2 (one per stage) | Chain (R2 → R2 direct) |
| A3 Network | R2 specialists | N R2 (independent) | Star (you → R2) |
| A4 Adhocracy | R1+R2 peers | 2-4 peers (symmetric by default; diversify if error-mode diversity matters more) | Mesh (R2 ↔ R2 direct) |
| A5 Divisional | R1 strategist + R6 + R2 | hierarchy, max 2 levels | Tree (R1 → R6 → R2 direct) |
| A7 Replicate-Select | R2 replicas + R3 judge | N R2 + 1 R3 | Star (isolated R2s, you → R3) |
| A8 Federation | R6 + R2 specialists + R3 | R6 + N R2 + 1 R3 | Hub (R6 ↔ R2 direct, you → R3) |

**A6 note:** intentionally omitted from execution in this skill. See "Substrate Constraints" above.

### Overlay Agents

| Overlay | Agent | When |
|---------|-------|------|
| DP3 | r3-adversary ×1 | High-stakes, hidden assumptions, or when you want someone to pressure-test the proposal |
| DP5 | r1-legislator ×1 | Long-running work where repeated exceptions suggest the operating rules themselves need revision |
| DP6 | r3-judge ×1 | High verification risk, especially when the producer should not grade its own work |
| DP8 | r2-executor ×1 (external perspective brief) | Capability gap, OOD, or when the team needs one meaningfully different viewpoint |

### Consensus Mechanisms

When multiple agents produce outputs that need resolution:

| Mechanism | When | How |
|-----------|------|-----|
| **Judge selects** | A7, quality-critical | R3 evaluates all outputs, picks best |
| **Majority vote** | ≥3 peers, factual questions | Each agent votes, majority wins |
| **Unanimous** | High-stakes, irreversible decisions | All must agree, or escalate |
| **Synthesize** | A4, creative tasks | Designated synthesizer merges, team votes accept/reject |
| **Orchestrator decides** | Tiebreak, time pressure | You decide based on arguments |

### Model Assignment

| Role | Default Model | Upgrade Condition |
|------|--------------|-------------------|
| R1 Legislator | opus / high | — |
| R2 Executor | sonnet / high | → opus if reciprocal interdependence or high uncertainty |
| R3 Judge | opus / high | → opus/max if verification_risk ≥ 0.7 |
| R3 Adversary | opus / max | Always max — must find non-obvious flaws |
| R4 Auditor | opus / max | Always max — protect audit integrity |
| R6 Dispatcher | sonnet / high | → opus if heterogeneity > 0.5 or large scale |
| A4 peers | Prefer same model/effort; allow diversity when it improves error-mode coverage | Preserve peer symmetry without forcing monoculture |

**AP13 rule**: Judge/Verifier MUST have an **independent error mode** from executors. Prefer a different model. If using the same model, isolate context, prompt/rubric, and evidence path.

---

## Architecture Execution Rules

### A0: Solo
Execute the task yourself. No agents needed.

### A1: Simple Structure
You are the leader (R2+R3 combined).
1. Decompose task into subtasks.
2. Spawn R2 Teammates, one per subtask. Assign briefs.
3. Collect results via SendMessage.
4. Review each output (your R3 function).
5. Integrate results. Reassign on failure.

Escalation: L1 (agent retries) → L2 (you reassign) → L3 (add overlay agent) → L4 (user).

### A2: Pipeline
1. Define stages and typed schema for each handoff (DP9).
2. Spawn R2 Teammate per stage. Give each the name of the NEXT stage agent.
3. Send input to Stage 1 via SendMessage.
4. Each stage, on completion, **sends output directly to next stage** via SendMessage.
5. Final stage reports result to you.

Direct chain: R2(stage1) → R2(stage2) → R2(stage3) → you. You only intervene on failure.

Escalation: L1 (stage retries) → L2 (re-execute from previous stage) → L3 (inject DP6 verification between stages) → L4.

### A3: Professional Network
1. Post task decomposition as a task board (shared file or message).
2. Spawn N specialist R2 Teammates with different expertise briefs.
3. Each specialist claims and works on matching subtasks.
4. Collect independent outputs.
5. Integrate (you or designated integrator).

No central routing — specialists self-select. This is NOT A8.

Escalation: L1 (retry) → L2 (different specialist claims) → L3 (inject DP3 cross-review) → L4.

### A4: Adhocracy
**All peers MUST be Teammates.**

Peer symmetry matters, but monoculture is not required.
- Prefer the same model+effort when balanced participation matters most.
- Allow different models or briefs when reciprocal work benefits from genuinely different error modes.
- If peers share the same model, diversify prompts, evidence focus, or critique responsibilities to avoid AP1.

1. Spawn N Teammates (2-4). Keep them symmetric enough that no hidden hierarchy is created. All have SendMessage.
2. Give each peer the names of ALL other peers — they communicate directly.
3. Convergence protocol (peers run this themselves):
   1. **Parallel explore**: each works independently, **posts rationale to all peers directly**
   2. **Structured critique**: each reviews ONE peer's approach, **sends critique directly to that peer**
   3. **Synthesis**: designated synthesizer merges critique → amendment, **broadcasts to all peers**
   4. **Vote**: each peer sends accept/reject to synthesizer
   5. **On reject**: different synthesizer, max 3 rounds
   6. **After 3 rounds**: best-rated attempt + minority opinion → report to you → L4
4. Governance: Ostrom-based (DP4) — shared rules, peer monitoring, graduated sanctions.

**You do NOT relay messages between peers.** You spawn them, give the protocol, and wait for the result. Intervene only on L3/L4.

Escalation: L1 (self-correct) → L2 (peer critique → revision) → L3 (you inject DP3 adversary or DP8 expert) → L4.

### A5: Divisional Hierarchy
**Max 2 levels. Non-negotiable. You spawn ALL agents.**

Teammates cannot spawn other Teammates — only you can. So you create the full tree upfront and wire connections.

1. Spawn 1 R1 strategist Teammate + 1 R6 dispatcher Teammate.
2. Spawn N division lead Teammates (R2). Give each lead the names of their assigned workers.
3. Spawn M worker Teammates (R2). Give each worker the name of their lead.
4. R1 defines division scope. R6 creates subtasks (TaskCreate), assigns to leads.
5. Leads **message their workers directly**. Workers **report to their lead directly**.
6. Cross-division issues: leads message R6 directly, R6 coordinates.
7. You receive progress from R1/R6 only.

**You build the tree, agents run it**: you → spawn all → R1+R6 coordinate → leads ↔ workers communicate directly.

Escalation: L1 (worker retries) → L2 (lead reassigns within division) → L3 (R1 injects overlay) → L4 (you → user).

### A6: Swarm (Framework note — not executed by this skill)
This skill does **NOT** instantiate A6.

Reason:
1. In Claude Code, only the main/orchestrator agent can create teams.
2. A faithful A6 swarm is leaderless and coordinated indirectly via shared state.
3. On this substrate, A6 becomes a low-fidelity approximation: wave scheduling + shared state + main-agent supervision.

If a task is theoretically A6-shaped:
1. Say so explicitly in the recommendation.
2. Prefer **A3** for independent homogeneous workers without routing.
3. Prefer **A8** for high-volume pooled work that benefits from routing or verification.
4. Prefer **A7** only when quality comes from multiple independent attempts, not throughput.

### A7: Replicate-Select
1. Spawn N R2 Teammates with DIFFERENT approach labels:
   - "systematic" — methodical, step-by-step
   - "creative" — lateral thinking, unconventional
   - "defensive" — edge cases and failure modes first
   - (additional: "minimal" — simplest possible solution)
2. Each works independently. **No coordination between replicas.**
3. Spawn 1 R3 Judge Teammate with an **independent verification context**. Prefer a different model; same model is acceptable only with isolated context, rubric, and evidence path.
4. Collect all outputs, send to Judge.
5. Judge sees ONLY outputs (not reasoning). Selects best.
6. If indistinguishable → escalate to L4.

Escalation: L1 (replica retries) → L2 (N/A) → L3 (add more replicas with new approach) → L4.

### A8: Routed Federation
**R6 Dispatcher runs routing. You manage verification.**

1. Spawn 1 R6 Dispatcher Teammate. Give it the task decomposition + specialist descriptions.
2. Spawn N specialist R2 Teammates. Give each their expertise brief.
3. Spawn 1 R3 Verifier Teammate with an **independent verification context**. Prefer a different model; same model is acceptable only with isolated context, rubric, and evidence path. Verifier only sees outputs.
4. R6 **creates subtasks (TaskCreate), assigns to specialists, messages them directly**.
5. Specialists work and **report completion to R6 directly**.
6. R6 collects outputs, **sends to you**. You relay to R3 Verifier (AP13 — verifier stays isolated).
7. Verifier rejects → you tell R6 → R6 re-routes or reassigns.
8. R6 accepts misrouting feedback from specialists (AP10 prevention).

**Communication split**: R6 ↔ R2 (direct, operational), You ↔ R3 (isolated, verification).
**Verifier restriction**: receives output artifacts from you only, NOT from R6 or specialists.

Escalation: L1 (retry) → L2 (R6 re-routes) → L3 (you inject DP3) → L4.

---

## Governance

### L1-L4 Escalation

| Level | Trigger | Action | Authority |
|-------|---------|--------|-----------|
| L1 Agent-local | Error, confidence < 0.3 | Agent self-retries | Agent (pre-delegated) |
| L2 Team-local | L1 exhausted, repeated failures | Reassign, peer help | Architecture coordinator |
| L3 Overlay injection | Same exception ≥3×, coordination breakdown | Add DP3/DP5/DP8 agent | You (additive only — never remove) |
| L4 Architecture switch | All L1-L3 failed, fundamental mismatch | Stop, return to user | User (D5) |

**L3 is ADDITIVE ONLY.** You can add agents, never remove existing ones.
**L4 trigger: any ONE of**: L1-L3 all exhausted, fundamental scope change, user request.

### Budget Awareness

No formal token budget tracking, but:
- If agents are producing diminishing returns, don't spawn more.
- If coordination overhead exceeds productive work → signal AP2, consider simplifying.
- Quality of verification (R3/R4) is protected — reduce executor count before downgrading judges.

### Anti-Pattern Detection During Execution

Monitor for these and intervene:

| Pattern | Signal | Intervention |
|---------|--------|-------------|
| AP1 Groupthink | All agents agreeing without debate | Inject DP3 adversary |
| AP2 Coordination explosion | More messaging than working | Reduce team or hierarchize |
| AP7 Hallucination cascade | Unverified output feeding next stage | Inject DP6 verification |
| AP12 Context overload | Agent producing incoherent output | Split role, reduce scope |
| AP13 Mirror verification | Reviewer rubber-stamping or sharing executor context/rubric | Re-isolate reviewer; switch model only if prompt/rubric separation is insufficient |

---

## Workflow Patterns (Multi-Phase)

For complex tasks that need multiple architecture phases:

### WP1: Analyze → Route → Execute → Verify
```
Phase 1: You analyze (A0)
Phase 2: Route subtasks (A8 or A3)
Phase 3: Execute (parallel R2 Teammates)
Phase 4: Verify (R3 Judge) ↺ back to Phase 3 on failure
```
**Use for**: Standard development, bug fixes, defined deliverables.

### WP2: Explore → Debate → Select
```
Phase 1: Parallel exploration (A3 or A7 — R2 Teammates with diverse briefs)
Phase 2: Structured debate (A4 — mutual critique)
Phase 3: Selection (R3 Judge or team vote)
```
**Use for**: Design decisions, creative problem-solving, research questions.

### WP3: Decompose → Parallel → Integrate
```
Phase 1: Decompose (you, A0)
Phase 2: Parallel execution (A3 — independent specialists)
Phase 3: Integration (you or designated integrator)
```
**Use for**: Large-scale independent work, multi-domain tasks.

### WP4: Propose → Review → Iterate
```
Phase 1: R2 produces proposal
Phase 2: R3 or R3-adversary reviews
Phase 3: R2 revises based on review ↺ repeat until pass or max rounds
```
**Use for**: Quality-critical deliverables, iterative refinement.

---

## Quick Reference: Selection Decision Tree

```
Task received
├── Simple, self-contained? → A0 Solo
├── Interdependence?
│   ├── Pooled
│   │   ├── Large, homogeneous batch work? → A6 in theory; unsupported here, disclose mismatch → use A3 or A8
│   │   ├── Need specialist routing? → A8 Federation
│   │   └── Independent experts? → A3 Network
│   ├── Sequential
│   │   ├── Stable sequence? → A2 Pipeline
│   │   └── Unstable? → A1 or A8
│   ├── Reciprocal
│       ├── Small team? → A1 Simple
│       ├── Medium, creative? → A4 Adhocracy
│       └── Large, multi-domain? → A5 Divisional
│   └── Replicated
│       └── Same task, multiple independent attempts + post-hoc selection? → A7 Replicate-Select
├── High verification risk? → Add DP6 or A7 overlay
├── High stakes? → Add DP3 adversary
├── Long-running? → Add DP5 meta-agent
└── Low confidence? → Ask for clarification or choose the lowest-cost structure with clear escalation
```

## Key Principles

- Don't pay coordination cost you can't justify. A0 is always available, but not always the right fast path.
- A0 fast paths are only for work that is easy to verify and easy to recover from if wrong.
- The user has final authority over whether to execute, change, or cancel the plan.
- Keep decisions where the relevant information is. Don't over-centralize by default.
- Use A7 when quality depends on multiple independent attempts plus post-hoc selection.
- Escalate when uncertain. Better to pause than to guess badly.
- Judges and verifiers must be independent from executors. Same-model verification is acceptable only with isolated context and a separate rubric/evidence path.
- Labels like DP3, DP5, DP8, DP9, and DP10 are reminders for concrete behaviors, not substitutes for explaining the actual behavior you want.
- When reporting a recommendation, substance beats taxonomy. Explain the behavior you want even if you also name the pattern.

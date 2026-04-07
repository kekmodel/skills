---
name: team
description: "Analyze task, derive team structure, confirm with user, then self-assign role and execute (T → Σ → Δ → Req → Γ → confirm → team → evaluation)."
user_invocable: true
---

# Agent Team

You execute the generation grammar pipeline to derive the optimal team structure, determine your own role within it, spawn the remaining atoms, and coordinate execution.

Phases 1-3 are pre-team analysis — you have no team role yet. At Phase 4 you assign yourself a role from the derived structure (Section 7, Structure Grammar) and become that atom. From that point on, you operate under the same theory constraints as every other atom.

Reference: `../../references/AGENT_TEAM_THEORY.md`

---

## Structure Grammar Atoms

4 agent types + 1 substrate:

- **Operator**: fulfills ω_par, ω_seq, ω_variety
- **Coordinator**: fulfills ω_dispatch, ω_exception, ω_boundary
- **Evaluator**: fulfills ω_verify, ω_select (+ DP3 adversarial, DP6+ audit overlays)
- **Rewriter**: fulfills ω_rewrite
- **State**: substrate (NOT agent) — task system + shared context files

### Obligation-to-Atom Mapping

| Obligation | Atom | Instance pattern |
|---|---|---|
| ω_par | Operator | 1 per parallel subtask |
| ω_seq | Operator | 1 per pipeline stage |
| ω_dispatch | Coordinator(dispatch) | 1 |
| ω_contract | State(contract) | substrate — tasks/{team}/*.json |
| ω_shared | State(shared) | substrate — _shared/context.md |
| ω_exception | Coordinator(exception) | same instance as ω_dispatch unless governance assigns a dedicated Coordinator |
| ω_boundary | Coordinator | same instance — boundary assignment and spillover mediation |
| ω_verify | Evaluator(verify) | 1 |
| ω_select | Evaluator(select) | 1 |
| ω_rewrite | Rewriter | 1 |
| ω_variety | Operator | k with diversified briefs |
| ω_g_rule | Coordinator(rule) | governance: defines/manages rules |
| ω_g_resolve | Coordinator(resolve) | governance: mediates conflicts |
| ω_g_monitor | Evaluator(audit) | governance: compliance monitoring |
| ω_g_enforce | Coordinator(enforce) | governance: enforcement |
| ω_g_escalate | Coordinator(escalate) | governance: routes to Coordinator(enforce) |

Model defaults: Operator → sonnet; Coordinator → sonnet; Evaluator → opus; Rewriter → opus.

---

## State Substrate Model

State is substrate, not an agent. Three modes, all file-system-based, persistent, lock-protected, and auditable via Read:

| Mode | Storage | Semantics | Write | Read |
|---|---|---|---|---|
| **State(contract)** | `~/.claude/tasks/{team}/*.json` | What to do (acceptance criteria) | TaskCreate/TaskUpdate | TaskGet/TaskList |
| **State(shared)** | `_shared/context.md` (project dir) | Collective knowledge across phases | Write/Edit | Read |
| **State(message)** | `~/.claude/teams/{team}/inboxes/{name}.json` | Communication log (per-atom inbox) | SendMessage | Read |

- **State(message)** is an implementation convenience for coordination and audit metadata. It is not a separate theory-level obligation and is never primary evidence for output correctness in verify/select modes.
- State(contract): Coordinator writes contracts; all atoms read via TaskGet/TaskList; Evaluator(verify) enforces.
- State(shared): cross-cutting state that multiple atoms may reference at different times (omega_shared: continuity + multi-party access). Operators are the primary writers; Coordinator mediates conflicts. **Size limit: 200 lines.** When State(shared) exceeds 200 lines, Coordinator must split it into topic-specific files (e.g., `_shared/api-design.md`, `_shared/decisions.md`) and maintain `_shared/context.md` as an index with links. Atoms read the index first, then follow links as needed. This derives from P1 (Finite Cognition) applied to shared substrate.
- State(message): point-to-point communication with a known recipient — requests, reports, status updates (P3: Information Locality). Evaluator(audit) may inspect routing, escalation, and protocol-compliance metadata only. Verify/select modes must ignore operator self-reports as evidence.

**State(shared) vs SendMessage:** Write to State(shared) when the information has no single recipient and may be read by any atom at any time. Use SendMessage when the recipient is known and the message is addressed to a specific atom. Do not duplicate: if it belongs in shared state, write it there; if it is a direct communication, send it.
- State(contract) and State(message) are deleted by TeamDelete. State(shared) lives in the project directory and survives.

### Access Model

Spawned teammate permissions:

| Capability | Coordinator | Operator | Evaluator | Rewriter |
|---|---|---|---|---|
| TaskCreate / TaskUpdate | ✅ | ❌ | ❌ | ❌ |
| TaskGet / TaskList | ✅ | ✅ | ❌ | ❌ |
| SendMessage | ✅ | ✅ | ❌ | ✅ |
| Read (all 3 State modes) | ✅ | ✅ | ✅ | ✅ |
| Write / Edit | ✅ | ✅ | ❌ | ✅ |

- **Evaluator**: Read-only access to all three State modes. No write to any State mode. Independence required by D6 / AP13.
- **Operator**: reads tasks and messages peers, may write scoped deliverables plus State(shared), but cannot create/modify tasks or governance artifacts.
- **Coordinator**: owns State(contract) write side for its dispatch domain. Mediates State(shared) — writes mediation results and propagates rule changes from Rewriter.
- **Rewriter**: Write/Edit for governance artifact files only. Does NOT write to State(shared) directly — submits proposals to Coordinator, who applies approved changes. Canonical fragment: Rewriter connects to Coordinator, not to State.

---

## Constraints

1. **Flat atom structure.** No atom may create sub-teams. New structure must be derived through the composition calculus.
2. **Evaluator isolation.** Evaluator communicates ONLY with the Coordinator fulfilling omega_dispatch (D6 + AP13). Never with Operators or Rewriter.
3. **Evaluator(audit) at phase boundaries.** Evaluator reads State(contract) and State(shared) directly, and may inspect State(message) metadata for routing/escalation compliance only. It must not treat operator self-assessments as evidence for output correctness.
4. **Structure overflow.** If atom count exceeds ~8 → consolidate Operators or split scope into phases. If still over → F5, L4.

---

## Communication Model

| Atom | SendMessage | Communicates with |
|---|---|---|
| Operator | yes | peer Operators, Coordinator |
| Coordinator | yes | Operators, Rewriter, other Coordinators |
| Evaluator | no | Coordinator(dispatch) only |
| Rewriter | yes | Coordinator |

Evaluator NEVER receives reasoning or self-assessments from Operators. It evaluates outputs against contracts only.

---

## Phase 1: Typing `T → Σ(T)`

**Procedure:** For each of the 12 predicates, ask the question and answer true/false with evidence.

| # | Predicate | Question to answer |
|---|---|---|
| 1 | seq | Does the output of one subtask feed as required input to another? |
| 2 | par | Can two or more subtasks run independently in parallel? |
| 3 | recip | Do two subtasks need to iterate bidirectionally on shared state? |
| 4 | dispatch | Must work be classified and routed to different specialists? |
| 5 | handoff | Are there discrete stage transitions between work phases? |
| 6 | continuity | Must information persist across phases or agents? |
| 7 | recurrence | Do similar exceptions or task patterns recur? |
| 8 | candidate_multiplicity | Can multiple valid solutions be generated independently? |
| 9 | reviewability | Can a reviewer assess output quality without being the producer? |
| 10 | spillover | Can one subtask's failure or output affect others beyond its boundary? |
| 11 | checkable | Do tests, specs, or objective criteria exist for pass/fail? |
| 12 | natural_counterhypothesis | Do meaningfully different solution strategies exist? |

**Decision rules:**
- If a predicate cannot be determined: mark `unknown` with what information is needed.
- If 3+ predicates are unknown → STOP. Ask the user for clarification before proceeding.
- Output: `Σ(T) = {predicate: value, evidence: "..."}` for all 12.

---

## Phase 2: Derivation `Σ(T) → Δ(T) → Req(Δ,Σ)`

### Step 1: Derive Active Pressures

**Procedure:** For each pressure, check the activation condition against the task.

| Pressure | Activation condition |
|---|---|
| D1 | Task exceeds single-agent capacity |
| D2 | Decomposition adds coordination cost |
| D2a | All coordination flows through one point |
| D3 | Handoffs require explicit state representation |
| D4 | Where decisions are made affects outcomes |
| D4a | Cannot maximize both local freedom and global coherence |
| D5 | Edge cases fall outside defined rules |
| D5a | Same exceptions keep happening |
| D6 | Delegated work may be wrong and must be checked |
| D7 | Team lacks enough response diversity |
| D8 | Feedback doesn't distinguish quality levels |
| D9 | Selection exists but candidate pool is homogeneous |

**Decision rule:** If NO pressures are active → Δ(T) = ∅ → solo execution, no team needed. Execute the task directly as a single agent and STOP.

### Step 2: Generate Obligations

**Procedure:** For each rule, check if conditions are met. If yes, add the obligation to Req.

| # | Conditions | Obligation |
|---|---|---|
| R1 | D2 ∧ seq | ω_seq |
| R2 | (D1 ∨ D2) ∧ par | ω_par |
| R3 | D1 ∧ dispatch | ω_dispatch |
| R4 | D3 ∧ handoff | ω_contract |
| R5 | D3 ∧ continuity | ω_shared |
| R6 | (D4 ∨ D4a) ∧ spillover | ω_boundary |
| R7 | D5 | ω_exception |
| R8 | D5a ∧ recurrence | ω_rewrite |
| R9 | D6 | ω_verify |
| R10 | D7 | ω_variety |
| R11 | D8 ∧ D9 ∧ candidate_multiplicity ∧ reviewability | ω_select |
| R12 | D4a ∧ D5 ∧ recip | ω_g_rule, ω_g_resolve, ω_g_monitor, ω_g_enforce, ω_g_escalate |

Output: `Req(Δ,Σ) = {ω_..., ω_...}` with which rule produced each.

---

## Phase 3: Cover `Req → Γ → minimality`

### Step 1: Well-Typedness (hard gate)

Check in order:
1. **wt_exec**: Is D1 active? If yes, does Σ contain at least one of {seq, par, dispatch}? If no topology witness → **STOP: F1. Report to user.**
2. **wt_select**: Are D8 and D9 active? If yes, are both candidate_multiplicity and reviewability true? If not → re-enter Phase 1 to re-assess those two predicates. If still fails → **STOP: report to user.**
3. **wt**: Both pass → proceed.

### Step 2: TF Shortcut

Compare Req and Σ against Task Family canonical tables.

**IMPORTANT:** A TF shortcut is valid ONLY if `Req(Δ,Σ) ⊆ cl(TF.base_Γ)` — the base Γ's closure must cover ALL of the TF's base Req. The corrected table:

| Family | base Σ | base Req | base Γ (closure-verified) |
|---|---|---|---|
| TF1 Frontline Intake | {seq, handoff} | {ω_seq, ω_contract} | {DP1} |
| TF2a Dispatch-Ops | {dispatch, handoff} | {ω_dispatch, ω_contract} | {DP11, DP9} |
| TF2b Program Orchestration | {dispatch, continuity, spillover} | {ω_dispatch, ω_shared, ω_boundary} | {DP11, DP12, DP7} |
| TF3 Sales/Brokerage | {continuity, spillover} | {ω_shared, ω_boundary} | {DP12, DP7} |
| TF4 Procurement/Planning | {continuity, spillover, candidate_multiplicity, reviewability} | {ω_shared, ω_boundary, ω_select} | {DP12, DP7, DP13} |
| TF5 Regulated Review | {handoff, checkable} | {ω_contract, ω_verify} | {DP9, DP6} |
| TF6 Technical Build | {par, handoff, continuity, checkable} | {ω_par, ω_contract, ω_shared, ω_verify} | {DP2, DP12, DP6} |
| TF7 Care Delivery | {dispatch, continuity} | {ω_dispatch, ω_shared} | {DP11, DP12} |
| TF8 Creative/Editorial | {par, continuity} | {ω_par, ω_shared} | {DP2, DP12} |

**Procedure:**
1. For each TF: check `TF.base_Σ ⊆ Σ(T)` AND `TF.base_Req ⊆ Req(Δ,Σ)`.
2. Full match → adopt TF.base_Γ as seed. Check `Req(Δ,Σ) ⊆ cl(seed)`. If yes, go to Step 4. If Req has extra obligations beyond TF base, supplement via Step 3.
3. No match → Step 3 entirely.

### Step 3: Bottom-Up Cover

DP coverage table:

| DP | Covers | Admissibility |
|---|---|---|
| DP1 | {ω_seq, ω_contract} | -- |
| DP2 | {ω_par, ω_contract} | -- |
| DP11 | {ω_dispatch} | -- |
| DP9 | {ω_contract} | -- |
| DP12 | {ω_shared} | -- |
| DP7 | {ω_boundary} | -- |
| DP10 | {ω_exception} | -- |
| DP6 | {ω_verify} | -- |
| DP13 | {ω_select} | ONLY if candidate_multiplicity ∧ reviewability |
| DP3 | {ω_verify} | Review alias; ONLY if checkable ∧ natural_counterhypothesis |
| DP5 | {ω_rewrite} | -- |
| DP8 | {ω_variety} | -- |
| DP4 | {ω_g_rule, ω_g_resolve, ω_g_monitor, ω_g_enforce, ω_g_escalate} | ONLY if D4a ∧ D5 ∧ recip |

**Procedure (greedy set cover with inline admissibility):**
1. `R_remaining = Req - cl(Γ_seed)`. If no seed: `R_remaining = Req`, `Γ = ∅`.
2. While R_remaining is non-empty:
   a. For each DP NOT in Γ: check admissibility first. Skip inadmissible DPs.
   b. Among admissible DPs: pick the one covering the most obligations in R_remaining.
   c. Tie-break: lower DP number (simpler first).
   d. Add selected DP to Γ. Update `R_remaining = Req - cl(Γ)`.
3. If no admissible DP covers any remaining ω → STOP: **F2 cover failure**. Report partial cover + uncovered ω to user.

### Step 4: Closure Verification

- `cl(Γ) = ∪{covers(dp) | dp ∈ Γ}`
- Check: `Req(Δ,Σ) ⊆ cl(Γ)`?
- Pass → Step 5.
- Fail → should not happen if Step 3 completed, but if so → F2, L4.

### Step 5: Minimality

- For each dp in Γ: would `Γ - {dp}` still have `Req ⊆ cl(Γ - {dp})`?
- If yes: remove dp (redundant).
- Repeat until irreducible.

---

## Confirmation Gate

After Phase 3 completes, present the analysis report to the user before proceeding. This is the boundary between pure analysis (no side effects) and realization (spawning, state creation, token consumption).

**Report contents:**

1. **Task Typing Σ(T)** — 12 predicates, each with value + evidence
2. **Active Pressures Δ(T)** — active D list with activation reasoning
3. **Required Obligations Req(Δ,Σ)** — ω list with rule # that produced each
4. **Well-Typedness** — wt_exec, wt_select, wt results
5. **Pattern Cover Γ** — DPs selected, admissibility verified, minimality verified
6. **Task Family Match** — TF matched (full/partial/none) with reasoning
7. **Realized Structure Preview** — atom roster with self-assignment:

   | Obligation | Atom | Count | Notes |
   |---|---|---|---|
   | ω_par | Operator | 1 per parallel subtask | |
   | ω_seq | Operator | 1 per pipeline stage | |
   | ω_dispatch | Coordinator | 1 | base instance (self-assignment target) |
   | ω_contract | State(contract) | substrate | |
   | ω_shared | State(shared) | substrate | |
   | ω_exception | Coordinator | same instance | base instance |
   | ω_boundary | Coordinator | same instance | base instance |
   | ω_verify | Evaluator(verify) | 1 | spawned (D6 independence) |
   | ω_select | Evaluator(select) | 1 | spawned (D6 independence) |
   | ω_rewrite | Rewriter | 1 | spawned (D5a reactive) |
   | ω_variety | Operator | k with diversified briefs | |
   | ω_g_rule | Coordinator(rule) | governance stack | spawned separately |
   | ω_g_resolve | Coordinator(resolve) | governance stack | spawned separately |
   | ω_g_monitor | Evaluator(audit) | governance stack | |
   | ω_g_enforce | Coordinator(enforce) | governance stack | base instance |
   | ω_g_escalate | Coordinator(escalate) | governance stack | spawned separately |

   Draw the Structure Grammar diagram.

8. **Preliminary Structural Assessment** — G0 (closure already verified in Phase 3 Step 4). G1 preliminary: D5 → DP10? D6 → DP6? D8/D9 → DP13? Governance stack complete?
9. **Risks & Caveats** — uncertain predicates, edge cases
10. **Decision Path** — trace through each phase showing choices and reasoning

After the report, ask the user: **"Proceed with this structure?"** Then **STOP. Do not proceed to Phase 4 until the user explicitly confirms.**

- **User confirms** → Phase 4.
- **User requests changes** → adjust Σ, Δ, or Γ as requested, re-verify closure and minimality, present updated report, and STOP again.

---

## Phase 4: Realization `Γ → team`

### Step 1: Count Atom Instances

From the computed Γ and Req, determine the concrete atom roster:

| Obligation in Req | Atom to instantiate | How many |
|---|---|---|
| ω_par | Operator | 1 per parallel subtask (examine task decomposition to determine k) |
| ω_seq | Operator | 1 per pipeline stage |
| ω_dispatch | Coordinator | 1 |
| ω_exception | Coordinator | same instance as dispatch |
| ω_boundary | Coordinator | same instance — boundary rules in brief |
| ω_contract | State(contract) | substrate, not an agent |
| ω_shared | State(shared) | substrate, not an agent |
| ω_verify | Evaluator (mode=verify) | 1 |
| ω_select | Evaluator (mode=select) | 1 |
| ω_rewrite | Rewriter | 1 |
| ω_variety | Operator | k with diversified briefs |

### Step 2: Realize ω_g_* Individually

If R12 produced governance obligations, the governance stack is mandatory, not optional. Realize each one separately before execution:

| Obligation | Realized by | Concrete behavior |
|---|---|---|
| ω_g_rule | Coordinator(rule) | Defines and manages governance rules. Receives modification proposals from Rewriter (omega_rewrite). |
| ω_g_resolve | Coordinator(resolve) | Mediates conflicts between atoms per boundary rules. |
| ω_g_monitor | Evaluator(audit) | At phase boundaries, checks rule adherence, reports AP violations. |
| ω_g_enforce | Coordinator(enforce) | Enforcement: L3 (add atom) or L4 (return to user). |
| ω_g_escalate | Coordinator(escalate) | Routes unresolvable issues to Coordinator(enforce). |

### Step 3: Self-Assignment

You are one of the atoms in the derived structure. Determine which role you fulfill.

**Exclusions (theory-derived):**
- **Evaluator**: excluded. D6 requires the verifier to be independent from the entity that designed the contracts and observed the process. AP13 warns against shared error modes between producer and verifier.
- **Rewriter**: excluded. Rewriter is a reactive role (D5a + recurrence → omega_rewrite) activated when recurring exceptions are detected, not at team formation.

**Assignment rule:**
- **Coordinator in roster** → self-assign to the base Coordinator instance (the one carrying the hub obligations: dispatch + exception + boundary, per Section 7 "same instance" clustering). Governance-specific instances (resolve, escalate) are spawned separately. Spawn all other atoms.
- **No Coordinator in roster** → self-assign to one of the Operators. Spawn the remaining atoms.

### Step 4: Atom Limit Check

Count total active agents (excluding State substrate):
- If total > 8 atoms → consolidate: merge Operators with overlapping scope, or split the task into sequential phases.
- If still exceeds limits after consolidation → F5 (normal-form failure), L4.

### Step 5: Model Assignment

| Atom | Model | Rationale |
|---|---|---|
| Operator | sonnet | Cost-efficient for execution; multiple instances expected |
| Coordinator | sonnet | Routing and dispatch do not require deep reasoning |
| Evaluator | opus | Judgment accuracy is critical for verification and selection |
| Rewriter | opus | Meta-reasoning (double-loop) requires strongest reasoning |

### Step 6: State Setup

- If ω_shared is in Req → create the shared context file (e.g., `_shared/context.md`) with initial task context.
- If ω_contract is in Req → create tasks via TaskCreate. Each task description contains acceptance criteria — this IS the contract. Set up dependencies with `addBlockedBy` for sequential tasks (e.g., Evaluator tasks blocked by Operator tasks).

### Step 7: TeamCreate

Call TeamCreate to create the team structure and task list:

```
TeamCreate({ team_name: "<name>", description: "<purpose>" })
```

**TeamCreate only creates team metadata.** It does NOT spawn agents. Agent spawning happens in Step 8.

### Step 8: Spawn Remaining Atoms

For each atom in the roster **except your self-assigned role**, spawn a teammate using the **Agent tool**:

```
Agent({
  team_name: "<team from Step 7>",
  name: "<unique name>",                          // e.g., "coordinator", "operator-1", "evaluator"
  subagent_type: "agent-team-theory:<atom>",    // operator, coordinator, evaluator, rewriter
  model: "<model>",                                // from Step 5; omit to use agent default
  prompt: "<instruction brief>",                   // role, scope, boundaries, shared state path, task IDs
  description: "<3-5 word summary>"
})
```

**Spawning rules:**
- Spawn all **parallel Operators simultaneously** (multiple Agent calls in one message).
- If governance is active, spawn additional Coordinators for ω_g_resolve / ω_g_escalate.
- Spawn **Evaluator** alongside others; if ω_g_monitor is active, include audit checkpoints in its brief.
- Spawn **Rewriter** when ω_rewrite is active (D5a + recurrence detected).
- Spawn **Coordinator(rule)** when ω_g_rule is active (governance stack).
- Each prompt MUST include: assigned task IDs, shared context file path, team name, communication boundaries.

**After spawning, assign tasks:**
- Use `TaskUpdate({ taskId, owner: "<teammate-name>" })` to assign tasks to spawned teammates.
- Teammates discover their contracts via `TaskGet` and begin work.

### Step 9: Execute as Your Assigned Atom

From this point you operate under the theory constraints of your self-assigned role:

**If you are Coordinator:**
- Dispatch to Operators via SendMessage.
- If additional Coordinators present: SendMessage task decomposition for domain-specific dispatch.
- After Operator tasks complete: SendMessage Evaluator with contract IDs and artifact locations. Do not forward Operator self-assessments as evidence.
- After multiple candidates produced: SendMessage all candidates to Evaluator for ranked selection.
- Activate Rewriter via SendMessage when recurring exceptions are detected.

**If you are Operator:**
- Execute your assigned subtask within scope.
- Report completion and artifacts.
- Coordinate with peer Operators via State(shared).

Teammates go **idle after each turn**. SendMessage to wake them.

### Step 10: Monitor During Execution

Watch for failure types and anti-patterns:

**Failure detection:**
- F1 (typing): predicate mismatch surfacing during execution
- F2 (cover): obligation with no atom covering it
- F3 (admissibility): inadmissible pattern producing bad results
- F4 (minimality): unnecessary atoms adding overhead without value
- F5 (normal-form): structural inconsistency (e.g., Evaluator receiving direct Operator messages)

**Anti-pattern surveillance (AP diagnostic dictionary):**
- AP1 Groupthink: all Operators converging on identical approach (F2, D7 active but ω_variety uncovered)
- AP2 Coordination Explosion: overloaded coordination bundle (F4)
- AP5 Information Silo: Operator working without reading shared state (F2, D3 active but ω_contract/ω_shared missing)
- AP9 Ghost Governance: governance obligations exist but no atom enforces them (F2, D4a+D5+recip active but governance bundle missing)
- AP12 Context Overload: single locus overloaded without topology witness (F1)
- AP13 Mirror Verification: evaluator shares producer's error mode (F3, independence violation)
- AP14 Untyped Selection: Evaluator(select) active without candidate_multiplicity ∧ reviewability (F1)

---

## Phase 5: Evaluation

### Step 1: G0 — Structural Validity (hard gate)

For each ω in Req, verify an atom instance covers it:
- Check: every obligation has at least one assigned atom or substrate.
- Check: no illegal selection (DP13 without admissibility).
- Check: no illegal composite (atoms merged in ways that violate isolation).
- Check: no obvious under-cover (obligation present but atom missing) or over-build (atom present but no obligation requires it).

**G0 fail → Reject.** Return to Phase 3 for structural redesign.

### Step 2: G1 — Safety (hard gate)

For each active pressure, verify its safety path exists:
- D5 active → exception handling path exists? (Coordinator with ω_exception, or escalation route)
- D6 active → independent verification exists? (Evaluator(verify) present and isolated from Operators)
- D8 ∧ D9 active → typed selection exists? (Evaluator(select) present with candidate_multiplicity ∧ reviewability confirmed)
- D4a ∧ D5 ∧ recip active → governance stack complete? (all 5 ω_g_* individually realized per Step 2 of Phase 4)

**G1 fail → Reject.** Add missing atom (L3). If insufficient → L4.

### Step 3: F1-F5 Detection

Review execution results against each failure type:

| Failure | What to check | Action |
|---|---|---|
| F1 typing | Predicate assumptions contradicted by execution evidence | Re-enter Phase 1 with new evidence |
| F2 cover | Any obligation went unfulfilled during execution | Re-enter Phase 3 to add covering DP |
| F3 admissibility | Inadmissible pattern produced incorrect or harmful results | Remove pattern, substitute admissible alternative |
| F4 minimality | Atom added no value, only overhead | Remove redundant atom from future runs |
| F5 normal-form | Structural violation detected (e.g., broken isolation, recursive delegation) | Correct structure immediately; Rewriter if recurring |

### Step 4: S1-S4 Scoring

Score each dimension using concrete measurements. If no execution logs exist, mark as `unscored`.

**S1 Outcome:**
1. Count total tasks assigned to Operators.
2. Count tasks with Evaluator(verify) verdict = `pass`.
3. S1 = verified_pass / total_tasks.
4. Compare to baseline: would solo execution achieve the same?

**S2 Efficiency:**
1. Count total messages: Read all inbox files in `~/.claude/teams/{team}/inboxes/`.
2. Count coordination messages: messages where `from` is a Coordinator.
3. Coordination ratio = coordination_messages / total_messages.
4. High coordination ratio indicates AP2 (Coordination Explosion) risk.
5. Estimate cognitive saturation per D1 (Finite Cognition): is any single Operator handling disproportionately many subtasks?

**S3 Mechanism Fit:**
For each active pressure, check its D-linked observable:
- D3 active → E10 (information reachability): can each atom access the state it needs? E11 (integration rework): how many times did work need redoing due to state gaps?
- D5 active → E4 (exception incidence): did exception rate decrease after Coordinator intervention?
- D6 active → E5 (error propagation depth): did errors stop at Evaluator or propagate further? E6 (governance enforcement): were rules actually enforced?
- D7 active → E3 (effective variety): did diversified Operators produce genuinely different solutions?
- D8/D9 active → E12 (feedback-performance alignment): did selection improve outcomes vs. random choice?

Each observable should move in the predicted direction. Misalignment → mechanism not working.

**S4 Robustness:**
1. Count all escalation events (L2 through L4).
2. Classify each: avoidable (could have been handled at a lower level) vs. necessary.
3. E7a = avoidable escalation count. Target: 0.
4. E7b = missed escalation count (issues that should have escalated but did not). Target: 0.
5. E8 (learning rate): if Rewriter present, count rule modifications that prevented recurrence.

### Step 5: Cost Ledger

Estimate token usage per atom type:

```
total_cost = cost_operator + cost_state + cost_coordinator + cost_evaluator + cost_rewriter
```

Derived ratios:
- `execution_share = cost_operator / total_cost`
- `coordination_share = cost_coordinator / total_cost`
- `evaluation_share = cost_evaluator / total_cost`
- `rewrite_share = cost_rewriter / total_cost`
- `state_share = cost_state / total_cost`

Anomaly signals (theory-linked):
- coordination_share dominates → AP2 (Coordination Explosion), consider structural simplification
- evaluation_share exceeds execution_share → check Evaluator scope for AP8 (Excess Variety Retention)
- rewrite_share disproportionate → structural instability, check for AP4 (Protocol Rigidity)

### Step 6: Final Verdict

| Verdict | Conditions |
|---|---|
| **Adopt** | G0 pass + G1 pass + S1 non-inferior to baseline + S2 improves over solo + D-linked observables move in predicted direction |
| **Revise** | G0/G1 weak pass, or mechanism fit misalignment, or cost distortion, or gates pass but failures recurring |
| **Reject** | G0 fail, G1 fail, or S1 worse than baseline |

**Without execution logs:** S1-S4 are unscored. Maximum possible verdict is `Revise (pilot-ready)`. Report the structural assessment (G0, G1) and recommend a pilot run.

### Step 7: Shutdown & Cleanup

Cleanup depends on the verdict:

| Verdict | Action |
|---|---|
| **Adopt** | Shutdown teammates → ask user whether to TeamDelete or preserve as record |
| **Revise** | Keep team alive — modify structure (add/remove atoms, reassign tasks) and re-execute. Only shutdown after final resolution. |
| **Reject** | Shutdown teammates → TeamDelete to free resources |

**Shutdown procedure** (when applicable):
1. SendMessage `{type: "shutdown_request"}` to each teammate. Wait for shutdown confirmation.
2. Call `TeamDelete()` to remove team config and task directories. TeamDelete fails if active members remain — ensure all are shut down first.
3. Shared state (`_shared/context.md`) and task history are execution records — preserve unless user requests cleanup.

---

## Escalation Protocol

| Level | Actor | Action | Basis |
|---|---|---|---|
| L2 | Coordinator(exception) | Re-route, reassign, or adjust scope | omega_exception (DP10) |
| L3 | Coordinator(enforce) | Add atom to cover unmet obligation. Additive only | omega_g_enforce |
| L4 | Coordinator(enforce) | Return to user with full diagnostic | team boundary |

Escalation is monotonic. L3 is additive only — never remove atoms mid-execution.

---

## Per-Phase Failure Table

| Phase | Failure | Type | Action |
|---|---|---|---|
| Phase 1 | Predicate undetermined (insufficient information) | F1 | Ask user for additional context |
| Phase 1 | Predicate contradiction (e.g., seq ∧ par without recip) | F1 | Re-judge once; if still contradictory → L4 |
| Phase 2 | No active pressures (trivial task) | -- | Solo execution, no team needed. Execute directly |
| Phase 2 | Req generation rule conditions mismatch | F1 | Re-enter Phase 1 to verify predicates |
| Phase 3 | wt_exec failure (D1 active, no topology witness) | F1 | Escalate to user — report topology witness absence |
| Phase 3 | wt_select failure (D8/D9 active, candidate/review missing) | F1 | Re-enter Phase 1 for candidate_multiplicity/reviewability |
| Phase 3 | TF matching fails | -- | Normal — proceed to bottom-up cover |
| Phase 3 | Closure failure (admissible DPs exhausted) | F2 | Report partial cover + uncovered ω, L4 |
| Phase 3 | Admissibility violation detected | F3 | Remove violating DP, substitute admissible alternative |
| Phase 3 | Minimality violation (redundant DP) | F4 | Auto-remove redundant DP |
| Phase 4 | Atom limit exceeded (too many agents) | F5 | Consolidate Operators or split scope; if still over → L4 |
| Phase 4 | Atom mapping impossible (e.g., ω_g_rule without Coordinator(rule)) | F2 | Add missing atom (L3) |
| Phase 5 | G0 failure | F2/F3 | **Reject** — re-enter Phase 3 or L4 |
| Phase 5 | G1 failure | F2 | **Reject** — add missing atom (L3) or L4 |
| Phase 5 | S1 worse than baseline | -- | **Reject** — report results, L4 |
| Phase 5 | F1-F5 recurring during execution | F5 | Activate Rewriter (omega_rewrite) for double-loop rule modification |

---

## Key Principles

From the generation grammar theory:

1. **Obligations before atoms.** Never select an agent type because it exists. Derive obligations from pressures, then cover obligations with patterns, then realize patterns as atoms.
2. **Solo is a derivation result.** Solo execution (no team) is the natural outcome when Δ(T) = ∅ after Phase 2. It is never a shortcut applied before the pipeline.
3. **Admissibility is a gate, not a filter.** Every DP must pass its admissibility check BEFORE being added to Γ, inside the greedy loop (Step 3, sub-step 2a). Post-hoc filtering is insufficient.
4. **Closure is verifiable.** `Req ⊆ cl(Γ)` is a binary check. If it fails, the structure is incomplete — no amount of good execution compensates.
5. **Evaluator independence is structural.** Evaluator isolation from Operators is a structural requirement (D6 + AP13). Violation is a G1 safety failure.
6. **State is substrate.** State(contract) and State(shared) are implemented via the task system and shared files. They are NOT agents and do NOT count toward atom limits.
7. **Governance obligations are individually realized.** Each ω_g_* maps to a specific atom with a concrete behavior. "Governance stack" is the collective name, not a single implementation.
8. **Minimality is mandatory.** Every atom must cover at least one obligation that no other atom in the team covers. Redundant atoms add coordination cost without benefit.
9. **Escalation is monotonic and additive.** L2 → L3 → L4. At L3 you may add atoms but never remove them during execution.
10. **Measurement precedes verdict.** S1-S4 require concrete evidence. Without execution data, the maximum verdict is Revise (pilot-ready). Claims of success require verification.
11. **Self-assignment is theory-constrained.** Evaluator is excluded (D6 + AP13: independence). Rewriter is excluded (D5a: reactive). Self-assign to the base Coordinator instance (Section 7: dispatch + exception + boundary cluster) if present, Operator otherwise. Governance-specific Coordinator instances (resolve, escalate) are spawned separately.

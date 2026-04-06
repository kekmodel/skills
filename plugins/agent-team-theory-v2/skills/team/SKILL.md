---
name: team
description: "Execute any task with the best team structure derived from the generation grammar pipeline (T → Σ → Δ → Req → Γ → team → evaluation)."
user_invocable: true
---

# Agent Team Orchestrator (v2 — Generation Grammar)

You are the orchestrator. Run the generation grammar pipeline to derive the optimal team, create it, execute, and evaluate.

Reference: `AGENT_TEAM_THEORY_SINGLE_VOLUME.md`

---

## Structure Grammar Atoms

4 active agents + 1 substrate:

- **Operator**: fulfills ω_par, ω_seq, ω_variety
- **Coordinator**: fulfills ω_dispatch, ω_exception, ω_boundary
- **Evaluator**: fulfills ω_verify, ω_select (+ DP3 adversarial, DP6+ audit overlays)
- **Rewriter**: fulfills ω_rewrite, ω_g_rule
- **State**: substrate (NOT agent) — task system + shared context files

### Obligation-to-Atom Mapping

| Obligation | Atom | Instance pattern |
|---|---|---|
| ω_par | Operator | 1 per parallel subtask |
| ω_seq | Operator | 1 per pipeline stage |
| ω_dispatch | Coordinator | 1 |
| ω_contract | State(contract) | substrate — TaskCreate descriptions |
| ω_shared | State(shared) | substrate — shared context file |
| ω_exception | Coordinator | same instance as ω_dispatch |
| ω_boundary | Coordinator | same instance — boundary assignment |
| ω_verify | Evaluator(verify) | 1 |
| ω_select | Evaluator(select) | 1 |
| ω_rewrite | Rewriter | 1 |
| ω_variety | Operator | k with diversified briefs |
| ω_g_rule | Rewriter | governance: proposes/applies rules |
| ω_g_resolve | Coordinator | governance: mediates conflicts |
| ω_g_monitor | Evaluator(audit) | governance: checkpoint compliance |
| ω_g_enforce | Orchestrator (you) | governance: L3/L4 enforcement |
| ω_g_escalate | Coordinator | governance: routes to orchestrator |

Model defaults: Operator → sonnet; Coordinator → sonnet; Evaluator → opus; Rewriter → opus.

---

## State Substrate Model

State is substrate, not an agent. Two modes:

- **State(contract):** Implemented via TaskCreate/TaskUpdate — task descriptions carry acceptance criteria. Coordinator writes contracts; all atoms read via TaskGet/TaskList; Evaluator(verify) enforces.
- **State(shared):** Implemented via a shared context file (e.g., `_shared/context.md`) + TaskUpdate metadata. Operators write state; all atoms read it; Coordinator mediates conflicts.
- **State(contract+shared):** Both active simultaneously. Task description = contract, shared file = continuity.

---

## Substrate Constraints

1. **TeamCreate is orchestrator-only.** Only you call TeamCreate. The team topology is created once in Phase 4.
2. **Maximum 2-level hierarchy.** Orchestrator (you) → atoms. No atom may create sub-teams.
3. **Evaluator isolation.** Evaluator communicates ONLY with you (the orchestrator). Never with Operators, Coordinator, or Rewriter directly.
4. **Evaluator(audit) is checkpoint-based.** Claude Code teammates are idle-after-turn, so audit runs at phase boundaries when you send snapshots — not continuously.
5. **Structure overflow.** If atom count exceeds substrate limits → consolidate Operators or split scope into phases. If still over → F5, escalate L4.

---

## Communication Model

| Atom | SendMessage | Communicates with | Rationale |
|---|---|---|---|
| Operator | yes | peers, Coordinator | Work coordination, progress reports |
| Coordinator | yes | Operators, Rewriter, you | Dispatch, exception escalation |
| Evaluator | no | you only | Independence (verify/select integrity) |
| Rewriter | yes | Coordinator, you | Rule modification proposals |

Evaluator NEVER receives reasoning or self-assessments from Operators. It evaluates outputs against specs only.

---

## Prior Analysis Check

Before running Phases 1-3: check if a `/team-analyze` (v2) report exists in the current conversation.

- **If a valid team-analyze report exists:** Adopt its Σ(T), Δ(T), Req(Δ,Σ), and Γ. Skip directly to Phase 4.
- **If not:** Run the full pipeline from Phase 1.

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
| TF1 Frontline Intake | {seq, handoff} | {ω_seq, ω_contract} | {DP1, DP9} |
| TF2a Dispatch-Ops | {dispatch, handoff} | {ω_dispatch, ω_contract} | {DP11, DP9} |
| TF2b Program Orchestration | {dispatch, continuity, spillover} | {ω_dispatch, ω_shared, ω_boundary} | {DP11, DP12, DP7} |
| TF3 Sales/Brokerage | {continuity, spillover} | {ω_shared, ω_boundary} | {DP12, DP7} |
| TF4 Procurement/Planning | {continuity, spillover, candidate_multiplicity, reviewability} | {ω_shared, ω_boundary, ω_select} | {DP12, DP7, DP13} |
| TF5 Regulated Review | {handoff, checkable} | {ω_contract, ω_verify} | {DP9, DP6} |
| TF6 Technical Build | {par, handoff, continuity, checkable} | {ω_par, ω_contract, ω_shared, ω_verify} | {DP2, DP9, DP12, DP6} |
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
| DP1 | {ω_seq} | -- |
| DP2 | {ω_par} | -- |
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
3. If no admissible DP covers any remaining ω → STOP: **F2 cover failure**. Report partial cover + uncovered ω to user (L4).

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

## Phase 4: Realization `Γ → team`

### Step 1: Count Atom Instances

From the computed Γ and Req, determine the concrete atom roster:

| Obligation in Req | Atom to instantiate | How many |
|---|---|---|
| ω_par | Operator | 1 per parallel subtask (examine task decomposition to determine k) |
| ω_seq | Operator | 1 per pipeline stage |
| ω_dispatch | Coordinator | 1 |
| ω_exception | Coordinator | same instance as dispatch |
| ω_boundary | Coordinator | same instance -- boundary rules in brief |
| ω_contract | State(contract) | substrate, not an agent |
| ω_shared | State(shared) | substrate, not an agent |
| ω_verify | Evaluator (mode=verify) | 1 |
| ω_select | Evaluator (mode=select) | 1 |
| ω_rewrite | Rewriter | 1 |
| ω_variety | Operator | k Operators with diversified briefs |

### Step 2: Realize ω_g_* Individually

If R12 produced governance obligations, realize each one separately:

| Obligation | Realized by | Concrete behavior |
|---|---|---|
| ω_g_rule | Rewriter | Proposes and applies governance rules. Receives exception patterns from Coordinator, drafts rule modifications, submits to orchestrator for approval. |
| ω_g_resolve | Coordinator | Mediates conflicts between atoms. When two atoms disagree on scope or approach, Coordinator brokers resolution per boundary rules. |
| ω_g_monitor | Evaluator(audit) | Checkpoint compliance monitoring. At each phase boundary, receives execution snapshot from orchestrator, checks rule adherence, reports AP violations. |
| ω_g_enforce | Orchestrator (you) | L3/L4 enforcement. When monitoring reveals violations, you take corrective action: adding atoms (L3) or returning to user (L4). |
| ω_g_escalate | Coordinator | Routes unresolvable issues to orchestrator. When Coordinator's own resolution authority is insufficient, escalates with context. |

### Step 3: Substrate Limit Check

Count total active agents (excluding State substrate):
- If total > 8 atoms → consolidate: merge Operators with overlapping scope, or split the task into sequential phases.
- If still exceeds limits after consolidation → F5 (normal-form failure), L4 escalation to user.

### Step 4: Model Assignment

| Atom | Model | Rationale |
|---|---|---|
| Operator | sonnet | Cost-efficient for execution; multiple instances expected |
| Coordinator | sonnet | Routing and dispatch do not require deep reasoning |
| Evaluator | opus | Judgment accuracy is critical for verification and selection |
| Rewriter | opus | Meta-reasoning (double-loop) requires strongest reasoning |

### Step 5: State Setup

- If ω_shared is in Req → create the shared context file (e.g., `_shared/context.md`) with initial task context.
- If ω_contract is in Req → prepare contract templates. Coordinator will create tasks with acceptance criteria via TaskCreate.

### Step 6: TeamCreate

Call TeamCreate to create the team:
- Include all atom agents determined in Steps 1-2.
- Each agent receives an instruction brief specifying: its role, obligations, scope boundaries, communication rules, and completion format.

### Step 7: Execute

- **If Coordinator present:** Send the task decomposition to Coordinator. Coordinator dispatches subtasks to Operators via TaskCreate + SendMessage.
- **If no Coordinator:** Send briefs directly to Operators.
- **If Evaluator(verify) present:** After Operator completion, route artifacts to Evaluator for judgment.
- **If Evaluator(select) present:** After multiple candidates produced, route all to Evaluator for ranked selection.
- **If Rewriter present:** Activate when Coordinator reports recurring exceptions.

### Step 8: Monitor During Execution

Watch for failure types and anti-patterns:

**Failure detection:**
- F1 (typing): predicate mismatch surfacing during execution
- F2 (cover): obligation with no atom covering it
- F3 (admissibility): inadmissible pattern producing bad results
- F4 (minimality): unnecessary atoms adding overhead without value
- F5 (normal-form): structural inconsistency (e.g., Evaluator receiving direct Operator messages)

**Anti-pattern surveillance (priority):**
- AP1 Groupthink: all Operators converging on identical approach
- AP5 Information Silo: Operator working without reading shared state
- AP7 Rubber-Stamp Evaluator: Evaluator passing everything without challenge
- AP9 Ghost Governance: governance obligations exist but no atom enforces them
- AP12 Audit Theater: Evaluator(audit) reports always show 1.0 compliance
- AP13 Bottleneck Coordinator: all messages routing through Coordinator causing delays
- AP14 Untyped Selection: Evaluator(select) active without candidate_multiplicity ∧ reviewability

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

**G1 fail → Reject.** Add missing atom (L3 escalation). If L3 insufficient → L4 to user.

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
1. Count total messages exchanged across all atoms.
2. Count coordination messages (Coordinator SendMessage, orchestrator instructions).
3. Coordination ratio = coordination_messages / total_messages.
4. Warning threshold: ratio > 0.4 indicates excessive coordination overhead.
5. Estimate cognitive saturation: any Operator handling > 5 distinct subtasks simultaneously?

**S3 Mechanism Fit:**
For each active pressure, check its D-linked observable:
- D3 active → E10 (information reachability): can each atom access the state it needs? E11 (integration rework): how many times did work need redoing due to state gaps?
- D5 active → E4 (exception incidence): did exception rate decrease after Coordinator intervention?
- D6 active → E5 (error propagation depth): did errors stop at Evaluator or propagate further? E6 (governance enforcement): were rules actually enforced?
- D7 active → E3 (effective variety): did diversified Operators produce genuinely different solutions?
- D8/D9 active → E12 (feedback-performance alignment): did selection improve outcomes vs. random choice?

Each observable should move in the predicted direction. Misalignment → mechanism not working.

**S4 Robustness:**
1. Count all escalation events (L1 through L4).
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

Anomaly signals:
- `coordination_share > 0.4` → Coordinator bottleneck, consider structural simplification
- `evaluation_share > execution_share` → evaluation heavier than production, check Evaluator scope
- `rewrite_share > 0.2` → excessive meta-reasoning, structural instability

### Step 6: Final Verdict

| Verdict | Conditions |
|---|---|
| **Adopt** | G0 pass + G1 pass + S1 non-inferior to baseline + S2 improves over solo + D-linked observables move in predicted direction |
| **Revise** | G0/G1 weak pass, or mechanism fit misalignment, or cost distortion, or gates pass but failures recurring |
| **Reject** | G0 fail, G1 fail, or S1 worse than baseline |

**Without execution logs:** S1-S4 are unscored. Maximum possible verdict is `Revise (pilot-ready)`. Report the structural assessment (G0, G1) and recommend a pilot run.

---

## Escalation Ladder

| Level | Actor | Action |
|---|---|---|
| L1 | Operator | Internal retry within scope |
| L2 | Coordinator | Reassign task, request peer assistance |
| L3 | Orchestrator (you) | Add atom (additive only — never remove during execution) |
| L4 | Orchestrator (you) | Return to user with full diagnostic |

Escalation is monotonic: each level implies the previous was insufficient. L3 is additive only — you may add an Evaluator or Operator, never remove one mid-execution.

---

## Per-Phase Failure Table

| Phase | Failure | Type | Action |
|---|---|---|---|
| Phase 1 | Predicate undetermined (insufficient information) | F1 | Ask user for additional context (L4) |
| Phase 1 | Predicate contradiction (e.g., seq ∧ par without recip) | F1 | Re-judge once; if still contradictory → L4 |
| Phase 2 | No active pressures (trivial task) | -- | Solo execution, no team needed. Execute directly |
| Phase 2 | Req generation rule conditions mismatch | F1 | Re-enter Phase 1 to verify predicates |
| Phase 3 | wt_exec failure (D1 active, no topology witness) | F1 | L4 — report topology witness absence |
| Phase 3 | wt_select failure (D8/D9 active, candidate/review missing) | F1 | Re-enter Phase 1 for candidate_multiplicity/reviewability |
| Phase 3 | TF matching fails | -- | Normal — proceed to bottom-up cover |
| Phase 3 | Closure failure (admissible DPs exhausted) | F2 | Report partial cover + uncovered ω, L4 |
| Phase 3 | Admissibility violation detected | F3 | Remove violating DP, substitute admissible alternative |
| Phase 3 | Minimality violation (redundant DP) | F4 | Auto-remove redundant DP |
| Phase 4 | Substrate limit exceeded (too many agents) | F5 | Consolidate Operators or split scope; if still over → L4 |
| Phase 4 | Atom mapping impossible (e.g., ω_g_* without Rewriter) | F2 | Add Rewriter (L3 escalation) |
| Phase 5 | G0 failure | F2/F3 | **Reject** — re-enter Phase 3 or L4 |
| Phase 5 | G1 failure | F2 | **Reject** — add missing atom (L3) or L4 |
| Phase 5 | S1 worse than baseline | -- | **Reject** — report results, L4 |
| Phase 5 | F1-F5 recurring during execution | F5 | Activate Rewriter (L3) for double-loop rule modification |

---

## Key Principles

These derive from the generation grammar theory, not from any prior architecture numbering:

1. **Obligations before atoms.** Never select an agent type because it exists. Derive obligations from pressures, then cover obligations with patterns, then realize patterns as atoms.
2. **Solo is a derivation result.** Solo execution (no team) is the natural outcome when Δ(T) = ∅ after Phase 2. It is never a shortcut applied before the pipeline.
3. **Admissibility is a gate, not a filter.** Every DP must pass its admissibility check BEFORE being added to Γ, inside the greedy loop (Step 3, sub-step 2a). Post-hoc filtering is insufficient.
4. **Closure is verifiable.** `Req ⊆ cl(Γ)` is a binary check. If it fails, the structure is incomplete — no amount of good execution compensates.
5. **Evaluator independence is structural.** Evaluator isolation from Operators is not a preference but a structural requirement. Violation is a G1 safety failure.
6. **State is substrate.** State(contract) and State(shared) are implemented via the task system and shared files. They are NOT agents and do NOT count toward atom limits.
7. **Governance obligations are individually realized.** Each ω_g_* maps to a specific atom with a concrete behavior. "Governance stack" is the collective name, not a single implementation.
8. **Minimality is mandatory.** Every atom must cover at least one obligation that no other atom in the team covers. Redundant atoms add coordination cost without benefit.
9. **Escalation is monotonic and additive.** L1 < L2 < L3 < L4. At L3 you may add atoms but never remove them during execution.
10. **Measurement precedes verdict.** S1-S4 require concrete evidence. Without execution data, the maximum verdict is Revise (pilot-ready). Claims of success require verification.

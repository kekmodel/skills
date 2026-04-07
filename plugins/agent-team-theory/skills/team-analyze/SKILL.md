---
name: team-analyze
description: "Analyze a task using the generation grammar pipeline (T → Σ → Δ → Req → Γ) and recommend team structure. Analysis only — no execution."
user_invocable: true
---

# Team Analyze

You analyze a task using the generation grammar pipeline and recommend a team structure.

This skill is **analysis only**. Do NOT create a team, spawn agents, or execute.

Reference: `../../references/AGENT_TEAM_THEORY.md`

## Structure Grammar Atoms

4 active agents + 1 substrate:

- **Operator**: fulfills ω_par, ω_seq, ω_variety
- **Coordinator**: fulfills ω_dispatch, ω_exception, ω_boundary
- **Evaluator**: fulfills ω_verify, ω_select (+ DP3 adversarial, DP6+ audit overlays)
- **Rewriter**: fulfills ω_rewrite, ω_g_rule
- **State**: substrate (NOT agent) — task system + shared context files

## Substrate Constraints

- **TeamCreate orchestrator-only**: only the orchestrator can create teams. Team topology is created upfront in Phase 4.
- **Max 2-level hierarchy**: orchestrator → atoms. No deeper nesting.
- **Evaluator isolation**: Evaluator communicates with orchestrator only (independence required by D6 / G1).
- **Checkpoint audit**: Evaluator(audit) operates at phase boundaries only (teammates go idle after each turn).
- **Practical atom limit**: ~8 agents. If the derived structure exceeds this, Operators must be consolidated or the task split into phases.

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

**Decision rule:** If NO pressures are active → Δ(T) = ∅ → solo execution, no team needed. Report this and STOP.

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
| DP1 | {ω_seq} | — |
| DP2 | {ω_par} | — |
| DP11 | {ω_dispatch} | — |
| DP9 | {ω_contract} | — |
| DP12 | {ω_shared} | — |
| DP7 | {ω_boundary} | — |
| DP10 | {ω_exception} | — |
| DP6 | {ω_verify} | — |
| DP13 | {ω_select} | ONLY if candidate_multiplicity ∧ reviewability |
| DP3 | {ω_verify} | Review alias; ONLY if checkable ∧ natural_counterhypothesis |
| DP5 | {ω_rewrite} | — |
| DP8 | {ω_variety} | — |
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

## Report Format

Present all 10 sections, then STOP.

1. **Task Typing Σ(T)** — 12 predicates, each with value + evidence
2. **Active Pressures Δ(T)** — active D list with activation reasoning
3. **Required Obligations Req(Δ,Σ)** — ω list with rule # that produced each
4. **Well-Typedness** — wt_exec, wt_select, wt results
5. **Pattern Cover Γ** — DPs selected, admissibility verified, minimality verified
6. **Task Family Match** — TF matched (full/partial/none) with reasoning
7. **Realized Structure Preview** — atom instances × models:

   | Obligation | Atom | Count |
   |---|---|---|
   | ω_par | Operator | 1 per parallel subtask |
   | ω_seq | Operator | 1 per pipeline stage |
   | ω_dispatch | Coordinator | 1 |
   | ω_contract | State(contract) | substrate |
   | ω_shared | State(shared) | substrate |
   | ω_exception | Coordinator | same instance |
   | ω_boundary | Coordinator | same instance |
   | ω_verify | Evaluator(verify) | 1 |
   | ω_select | Evaluator(select) | 1 |
   | ω_rewrite | Rewriter | 1 |
   | ω_variety | Operator | k with diversified briefs |
   | ω_g_rule | Rewriter | governance stack |
   | ω_g_resolve | Coordinator | governance stack |
   | ω_g_monitor | Evaluator(audit) | governance stack |
   | ω_g_enforce | Orchestrator | governance stack |
   | ω_g_escalate | Coordinator | governance stack |

   Model defaults: Operator → sonnet; Coordinator → sonnet; Evaluator → opus; Rewriter → opus.

   If `ω_g_*` appears in Req, the preview must reserve at least: 1 peer Coordinator for `ω_g_resolve` / `ω_g_escalate`, 1 Rewriter for `ω_g_rule`, and 1 Evaluator in audit mode for `ω_g_monitor`.

   Draw the Structure Grammar diagram.

8. **Preliminary Structural Assessment** — G0 is assessable (closure test already done in Step 4). G1 is **preliminary** — full G1 requires Phase 4 realization.
   - **Predictable from Γ alone**: D5 active → DP10 in Γ? D6 active → DP6 in Γ? D8/D9 active → DP13 in Γ?
   - **Requires Phase 4**: D4a+D5+recip governance stack completeness (need actual atom realization to confirm all 5 ω_g_* are covered by distinct atoms).

9. **Risks & Caveats** — uncertain predicates, edge cases, substrate limitations

10. **Decision Path** — trace through each phase showing choices and reasoning

After the report: **"To execute this structure, use `/team` or say 'Proceed'."**

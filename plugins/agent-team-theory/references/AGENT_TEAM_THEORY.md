# Agent Team Theory

---

## 1. Scope

This theory is about `agent-team design`.

```text
task T
  -> task typing Σ(T)
  -> active derived pressures Δ(T)
  -> required obligations Req(Δ, Σ)
  -> admissible pattern cover Γ
  -> realized team structure
```

The key claims are:

- This theory is not a list of good role names.
- This theory is not a catalog of correct org charts.
- This theory is a generative grammar that interprets a task structurally and derives the minimum organizational structure that fits that task.

---

## 2. Basic Objects and Notation

The basic objects are:

- `T`: task
- `Σ(T)`: typed structural predicates of the task
- `Δ(T)`: active derived pressure set
- `Req(Δ, Σ)`: primitive obligations required by the active pressures
- `Γ`: selected pattern set
- `cl_Σ(Γ)`: the obligation-level closure of the selected patterns
- `nf_Σ(Γ)`: sanctioned normal form

The primitive obligation set is:

```text
Ω = {
  ω_seq,
  ω_par,
  ω_dispatch,
  ω_contract,
  ω_shared,
  ω_boundary,
  ω_exception,
  ω_rewrite,
  ω_verify,
  ω_variety,
  ω_select,
  ω_g_rule,
  ω_g_resolve,
  ω_g_monitor,
  ω_g_enforce,
  ω_g_escalate
}
```

Important points:

- What the core derives directly is not `roles` but `obligations`.
- Role names, org charts, and architecture labels are all downstream representational layers.

---

## 3. First Principles

### P1. Finite Cognition

The cognitive and processing capacity of a single execution unit is finite.

### P2. Task Decomposition with Residual Interdependence

Tasks can be decomposed, but residual interdependence remains after decomposition.

### P3. Information Locality

Relevant information is not distributed uniformly, and lossless full centralization is generally impossible or costly.

### P4. Specification Incompleteness

A finite rule set cannot completely cover every relevant state.

### P5. Coordination Friction

Coordination is not free, and boundaries and handoffs create cost and delay.

### P6. Requisite Variety

To absorb environmental disturbances, the interior must contain a sufficiently rich effective response repertoire.

### P7. Differential Performance Feedback and Selection

For cumulative improvement to occur, performance differences must be reflected monotonically in selection and updating.

---

### Meta-Structure

The theory has three layers:

- `P1~P5`: structural constraints
- `P6`: search / coverage constraints
- `P7`: selection / cumulative-improvement constraints

That is:

```text
P1~P5 tell us why structure is needed
P6 tells us why variety is needed
P7 tells us why learning/selection must be typed
```

---

## 4. Derived Pressures

Layer 2 centers on `D`. The key distinction is between base derived pressures and specializations.

### Base Propositions

- `D1`: necessity of multiple execution units
- `D2`: load transfer under decomposition
- `D3`: primacy of boundary representation and state externalization
- `D4`: non-neutrality of authority placement
- `D5`: inevitability of residual decision-right allocation
- `D6`: verification risk under delegation
- `D7`: coverage failure from insufficient variety
- `D8`: learning invalidation under non-differential feedback
- `D9`: non-cumulativity of variety without selection

### Important Specializations

- `D2a`: bottlenecks from concentrated coordination
- `D4a`: non-dominance in the autonomy-integration tradeoff
- `D5a`: rewrite pressure from recurring exceptions

### Completeness Rule

For a new `D` to qualify as an independent base item, it must introduce a new primitive loss channel that cannot be reduced to existing `D`s.

If a candidate can be expressed as one of the following,

- an operating point,
- a repeated form,
- a concentrated form,
- a mixed-state authority form,

then it is not an independent `D` but a specialization or prescription.

This theory closes the space of primitive loss channels with the following nine axes:

- capacity insufficiency -> `D1`
- load transfer under decomposition -> `D2`
- boundary representation / state externalization loss -> `D3`
- authority relocation loss -> `D4`
- residual governance loss -> `D5`
- delegated verification opacity -> `D6`
- coverage deficit -> `D7`
- selection misalignment -> `D8`
- non-cumulative search -> `D9`

So `D` is not a list of good ideas. It is the generating basis of the loss pressures that structure must absorb.

---

## 5. Design Patterns

`DP` is the prescriptive layer over `D`.

### Decomposition / Routing

- `DP1`: Sequential Pipeline
- `DP2`: Fan-out / Fan-in
- `DP11`: Explicit Dispatch / Triage

### Boundary / State

- `DP9`: Contract-First Interface
- `DP12`: Shared State / Blackboard

### Authority / Governance

- `DP7`: Hierarchical Subsidiarity
- `DP10`: Exception Queue / Escalation Gate
- `DP4`: Self-Organizing with Governance

### Verification / Selection

- `DP6`: Delegate-and-Verify
- `DP13`: Performance-Aligned Judging / Selective Retention
- `DP3`: Adversarial Review

### Adaptation / Variety

- `DP5`: Double-Loop Meta-Agent
- `DP8`: Diversity Injection

---

## 6. Composition Calculus

The final generative layer of the core is `C`.

### Formal Objects

- `Δ(T) ⊆ {D1, ..., D9, D2a, D4a, D5a}`: the set of active pressures in task `T`
- `Σ(T) = {σ | T ⊢ σ}`: the set of structural predicates derived by the typing judgment
- `Ω`: the primitive obligation set
- `Γ ⊆ {DP1, ..., DP13}`: the selected pattern set
- `O_Σ(dp) ⊆ Ω`: the set of obligations satisfied by pattern `dp` under structure `Σ`
- `cl_Σ(Γ) = ⋃_{dp ∈ Γ} O_Σ(dp)`
- `nf_Σ(dp)`: the representation of `dp` in sanctioned normal form
- `nf_Σ(Γ) = ⋃_{dp ∈ Γ} nf_Σ(dp)`

### Typing Judgment

The task is first typed by the following predicates:

```text
seq
par
recip
dispatch
handoff
continuity
recurrence
candidate_multiplicity
reviewability
spillover
checkable
natural_counterhypothesis
```

The intended reading of each predicate is:

- `T ⊢ seq`: removing some ordered handoff edge increases loss.
- `T ⊢ par`: two or more execution units can run concurrently within the same episode under a common contract and conditional independence.
- `T ⊢ recip`: two or more execution units must update decisions within the same episode by observing each other's intermediate states.
- `T ⊢ dispatch`: changing the routing choice before execution materially changes expected loss.
- `T ⊢ handoff`: an output crosses an execution boundary and becomes the input to the next unit.
- `T ⊢ continuity`: temporally separated execution units must read and write the same latent state, and lossless full externalization is difficult or expensive.
- `T ⊢ recurrence`: the same exception class reappears across episodes, and each occurrence incurs online workaround cost.
- `T ⊢ candidate_multiplicity`: at least two substantively different candidate outputs or policy variants are generated.
- `T ⊢ reviewability`: there exists a common basis, evidence, or ex post signal for comparing candidates.
- `T ⊢ spillover`: a local decision in one unit creates material external effects on another unit or on downstream integration cost.
- `T ⊢ checkable`: an independent validator can inspect factual correctness or constraint satisfaction.
- `T ⊢ natural_counterhypothesis`: the current candidate set already contains opposing hypotheses or error modes, reducing the cost of creating a separate adversarial role.

### Well-Typedness

- `wt_exec`: if `D1 ∈ Δ(T)`, then `{seq, par, dispatch} ∩ Σ(T) ≠ ∅`
- `wt_select`: if `D8 ∈ Δ(T)` or `D9 ∈ Δ(T)`, then `{candidate_multiplicity, reviewability} ⊆ Σ(T)`
- `wt`: both `wt_exec` and `wt_select` hold

`Req`, `cover`, and `minimality` are evaluated only on typed tasks for which `T ⊢ wt`.

### Pattern Semantics

- `DP1 -> {ω_seq, ω_contract}`
- `DP2 -> {ω_par, ω_contract}`
- `DP11 -> {ω_dispatch}`
- `DP9 -> {ω_contract}`
- `DP12 -> {ω_shared}`
- `DP7 -> {ω_boundary}`
- `DP10 -> {ω_exception}`
- `DP5 -> {ω_rewrite}`
- `DP6 -> {ω_verify}`
- `DP8 -> {ω_variety}`
- `DP13 -> {ω_select}`
- `DP3 -> {ω_select} ∪ {ω_verify if checkable ∈ Σ(T)}`
- `DP4 -> {ω_g_rule, ω_g_resolve, ω_g_monitor, ω_g_enforce, ω_g_escalate}`

### Req Generation

- `D2 + seq -> ω_seq`
- `(D1 or D2) + par -> ω_par`
- `D1 + dispatch -> ω_dispatch`
- `D3 + handoff -> ω_contract`
- `D3 + continuity -> ω_shared`
- `(D4 or D4a) + spillover -> ω_boundary`
- `D5 -> ω_exception`
- `D5a + recurrence -> ω_rewrite`
- `D6 -> ω_verify`
- `D7 -> ω_variety`
- `D8 + D9 + candidate_multiplicity + reviewability -> ω_select`
- `D4a + D5 + recip -> ω_g_*`

### Admissibility

- `DP13` may be used only when `candidate_multiplicity` and `reviewability` are both typed.
- `DP3` is a review alias over typed compare-select, and candidate generation must already be reflected in task typing.
- If `natural_counterhypothesis ∉ Σ(T)`, then `DP3` usually requires `DP8` or another candidate-generation mechanism as a companion.
- `DP4` may be used only when `D4a + D5 + recip` are jointly active.

### Sanctioned Normal Form

- `nf_Σ(DP3) = {DP13} ∪ {DP6 if checkable ∈ Σ(T)}`
- `nf_Σ(DP4) = {DP4}`
- Every other `DP` is its own normal form.

### Minimality

A good structure must satisfy:

```text
Req(Δ, Σ) ⊆ cl_Σ(Γ)
```

while also being irreducible to a smaller admissible normal form.

In other words, the core searches not for a merely possible structure but for a minimal admissible cover that closes every required obligation.

### Canonical Minimal Covers

- `D2 + D3 + seq -> DP1`
- `(D1 or D2) + D3 + par -> DP2`
- `D3 + handoff -> DP9`
- `D3 + continuity -> DP12`
- `D4 or D4a` with `spillover -> DP7`
- `D5 -> DP10`
- `D5a + recurrence -> DP5` and usually `DP10`
- `D6 -> DP6`
- `D7 -> DP8`
- `D8 + D9 + candidate_multiplicity + reviewability -> DP13`
- `D7 + D8 + D9 + candidate_multiplicity + reviewability -> DP8 + DP13`
- `D6 + D8 + D9 + candidate_multiplicity + reviewability -> DP3`
- `D4a + D5 + recip -> DP4`

---

## 7. Structure Grammar

To render structure, we need a more general grammar than domain-specific role names.

Primitive elements:

- `Operator`
- `State`
- `Coordinator`
- `Evaluator`
- `Rewriter`

### Interpretation

- `Operator`: performs the underlying work
- `State`: common state, records, contracts, working memory
- `Coordinator`: assignment, boundary alignment, exception routing, procedural coordination
- `Evaluator`: verification, comparison, judgment, selection
- `Rewriter`: rule revision and structural update

Important:

- `Operator / Coordinator / Evaluator / Rewriter` are active team atoms.
- `State` is not a team member but a shared substrate.

### Topology and Constraints

The following are not nodes:

- topology: `seq`, `par`, `recip`, `handoff`
- constraints: `boundary`, `variety`, `candidate_multiplicity`, `reviewability`, `checkable`

### State Roles

`State` has at least two roles:

- `State(contract)`
- `State(shared)`

When both are performed at once, write:

```text
State(contract+shared)
```

### Obligation Mapping

- `ω_seq` is realized as an ordered execution edge between execution units
- `ω_par -> Operator[1..k]`
- `ω_dispatch -> Coordinator`
- `ω_contract -> State(contract)`
- `ω_shared -> State(shared)`
- `ω_exception -> Coordinator(exception)`
- `ω_verify -> Evaluator(verify)`
- `ω_select -> Evaluator(select)`
- `ω_rewrite -> Rewriter`
- `ω_g_rule -> Coordinator(rule)`
- `ω_g_resolve -> Coordinator(resolve)`
- `ω_g_monitor -> Evaluator(monitor)`
- `ω_g_enforce -> Coordinator(enforce)`
- `ω_g_escalate -> Coordinator(escalate)`
- `ω_boundary` is realized as a boundary assignment rule
- `ω_variety` is realized as diversified `Operator[1..k]`

### Canonical Fragments

Sequential pipeline:

```text
[Operator] -> [State(contract)] -> [Operator]
```

Parallel build:

```text
[Operator] [Operator] [Operator]
        \       |       /
      [State(contract+shared)]
```

Verification-critical:

```text
[Operator] -> [State] -> [Evaluator]
```

Dispatch-orchestrated:

```text
      [Coordinator]
     /      |      \
[Operator][Operator][Operator]
     \      |      /
          [State]
```

Rewrite loop:

```text
[Coordinator(exception)] -> [Rewriter]
```

Governance-heavy fragment:

```text
[Coordinator(rule)] ---recip--- [Coordinator(resolve)]
         \                        /
           [State(shared)]
                 |
        [Evaluator(monitor)]
                 |
   [Coordinator(enforce/escalate)]
```

---

## 8. Task-Family Atlas

Before mapping occupations or work settings to org charts, first compress them into dominant task families.

### Activation Rules

- Do not activate `D5` from `continuity`, `spillover`, and `dispatch` alone. Activate `D5` only when residual judgment remains structurally after pre-specified rules have absorbed what they can, and that judgment must be sent online to an owner.
- Do not activate `D6` from `checkable ∈ Σ(T)` alone. Activate `D6` only when the correctness of delegated sub-judgment cannot be fully identified from outputs alone, and errors materially affect loss.
- When `recip + D4a + D5` are tightly coupled, treat the governance-heavy institutional mode primarily as a governance lift over a base family rather than as a separate family.

### Mode Lifts

- compare-select lift: `Δ += {D8, D9}`, `Σ += {candidate_multiplicity, reviewability}`, yielding `DP13`
- high-stakes review lift: if compare-select is present together with `D6` and `checkable`, `DP3` can be used as an alias
- exploration lift: `Δ += {D7}`, yielding `DP8`
- rewrite lift: if `D5` and `recurrence` are present, add `D5a`, yielding `DP5`
- governance lift: if `D5`, `recip`, and `D4a` couple together, add `DP4`

### TF1 Frontline Intake / Transaction Flow

- base `Σ = {seq, handoff}`
- base `Δ = {D2, D3}`
- base `Req = {ω_seq, ω_contract}`
- base `Γ = {DP1}`

```text
[Operator] -> [State(contract)] -> [Operator]
```

### TF2a Dispatch-Ops Supervision

- base `Σ = {dispatch, handoff}`
- base `Δ = {D1, D3}`
- base `Req = {ω_dispatch, ω_contract}`
- base `Γ = {DP11, DP9}`

```text
      [Coordinator]
    /   |   |   \
[Operator]...[Operator]
        \ | /
        [State(contract)]
```

### TF2b Program / Service Orchestration

- base `Σ = {dispatch, continuity, spillover}`
- base `Δ = {D1, D3, D4}`
- base `Req = {ω_dispatch, ω_shared, ω_boundary}`
- base `Γ = {DP11, DP12, DP7}`

```text
      [Coordinator]
    /      |      \
[Operator][Operator][Operator]
    \      |      /
      [State(shared)]
            |
      [Coordinator]
```

The institutional governance-heavy mode is:

```text
 [Coordinator] ---recip--- [Coordinator]
          \                  /
            [State(shared)]
                  |
             [Coordinator]
                  |
              [Rewriter]
```

### TF3 Sales / Brokerage / Account Conversion

- base `Σ = {continuity, spillover}`
- base `Δ = {D3, D4}`
- base `Req = {ω_shared, ω_boundary}`
- base `Γ = {DP12, DP7}`

### TF4 Procurement / Allocation / Analytical Planning

- base `Σ = {continuity, spillover, candidate_multiplicity, reviewability}`
- base `Δ = {D3, D4, D8, D9}`
- base `Req = {ω_shared, ω_boundary, ω_select}`
- base `Γ = {DP12, DP7, DP13}`

```text
[Operator] [Operator] [Operator]
        \      |      /
         [State(shared)]
               |
        [Evaluator(select)]
               |
          [Coordinator]
```

### TF5 Regulated Review / Compliance / Investigation

- base `Σ = {handoff, checkable}`
- base `Δ = {D3, D6}`
- base `Req = {ω_contract, ω_verify}`
- base `Γ = {DP9, DP6}`

```text
[Operator] -> [State(contract)] -> [Evaluator(verify)]
```

### TF6 Technical Build / Engineering Delivery

- base `Σ = {par, handoff, continuity, checkable}`
- base `Δ = {D1, D2, D3, D6}`
- base `Req = {ω_par, ω_contract, ω_shared, ω_verify}`
- base `Γ = {DP2, DP12, DP6}`

```text
[Operator] [Operator] [Operator] [Operator]
        \       |       |       /
      [State(contract+shared)]
                |
         [Evaluator(verify)]
```

companion modes:

- add `DP11` if allocation becomes nontrivial
- add `DP13` if compare-select becomes explicit
- use `DP3` as an alias if review-heavy, high-stakes compare-select is present

### TF7 Care Delivery / Human Casework

- base `Σ = {dispatch, continuity}`
- base `Δ = {D1, D3}`
- base `Req = {ω_dispatch, ω_shared}`
- base `Γ = {DP11, DP12}`

```text
[Coordinator] -> [Operator] -> [State(shared)]
```

### TF8 Creative / Editorial / Investigative Media Production

- base `Σ = {par, continuity}`
- base `Δ = {D1, D2, D3}`
- base `Req = {ω_par, ω_shared}`
- base `Γ = {DP2, DP12}`

```text
[Operator] [Operator] [Operator] [Operator]
        \       |       |       /
           [State(shared)]
```

### Macro-Governance Exemplar

The U.S. Constitution is not a worker family but a macro-governance exemplar.

- base `Σ = {recip, spillover, continuity, recurrence}`
- base `Δ = {D3, D4, D4a, D5, D5a}`
- base `Γ = {DP12, DP7, DP10, DP5, DP4}`

```text
                [State]
                   |
 [Rewriter] -- [Coordinator] -- [Evaluator]
                   |
             [Coordinator]
                   |
               [Rewriter]
```

---

## 9. Failure Semantics

The true dual of `C` is `F1~F5`.

- `F1`: typing failure
- `F2`: cover failure
- `F3`: admissibility failure
- `F4`: minimality failure
- `F5`: normal-form failure

Above that, `AP` is a diagnostic dictionary that reverse-maps observed pathologies through `F`.

```text
F = strict theoretical dual of C
AP = observed symptom clusters reverse-mapped through F
```

### AP Interpretation Grades

- `strict reverse-map`: a strong diagnosis that points almost directly to a specific `F`
- `mixed diagnostic`: a diagnosis that suggests a specific `F` but must be read together with other failures
- `heuristic label`: a weak diagnosis unless supported by operating context

### Compact AP Dictionary

- `AP1 Groupthink`: `strict reverse-map`, `F2`, `D7 ∈ Δ(T)` but `ω_variety ∉ cl_Σ(Γ)`, repair with `DP8`, and `DP3` when needed
- `AP2 Coordination Explosion`: `mixed diagnostic`, `F4`, an overloaded coordination bundle is retained even though a smaller `nf_Σ(Γ')` is available, repair with `DP7`, `DP11`, `DP9`
- `AP3 Single Point of Failure`: `heuristic label`, `F2`, authority/exception support structure is missing under `D2a` or `D5` pressure, repair with `DP7`, `DP10`, and `DP4` when needed
- `AP4 Protocol Rigidity`: `strict reverse-map`, `F2`, `D5a` and `recurrence` are typed but `ω_rewrite ∉ cl_Σ(Γ)`, repair with `DP5`, `DP10`
- `AP5 Information Silo`: `strict reverse-map`, `F2`, `D3` and `handoff`/`continuity` are typed but `ω_contract` or `ω_shared` is missing, repair with `DP9`, `DP12`
- `AP6 Infinite Delegation Chain`: `heuristic label`, `F3`, `ω_exception` is repeatedly routed but no terminal verifier or owner exists, repair by naming a terminal owner and adding `DP6`
- `AP7 Hallucination Propagation`: `mixed diagnostic`, `F2`, `D6` or `D3` is active but `ω_verify` or `ω_contract` is missing, repair with `DP6`, `DP9`, `DP10`
- `AP8 Excess Variety Retention`: `strict reverse-map`, `F4`, `DP8 ∈ Γ` even though `D7 ∉ Δ(T)`, repair by removing or shrinking `DP8`
- `AP9 Ghost Governance`: `strict reverse-map`, `F2`, `D4a + D5 + recip` is present but the governance bundle is missing, repair with `DP4`, `DP10`
- `AP10 Router Tyranny`: `mixed diagnostic`, `F5`, one operating point of `DP11` is frozen as if it were primitive authority, repair with `DP7`, `DP5`
- `AP11 False Modularity`: `strict reverse-map`, `F1`, the task is typed as `par` but actual `spillover`/`handoff`/`continuity` remains high, repair by correcting the typing and adding `DP9`, `DP7`
- `AP12 Context Overload`: `strict reverse-map`, `F1`, `D1` or `D2a` is active but a single locus is overloaded without a topology witness, repair by making `seq`/`par`/`dispatch` explicit and adding `DP1`/`DP2`/`DP11`
- `AP13 Mirror Verification`: `heuristic label`, `F3`, `DP6` exists but the verifier shares the worker's error mode and is therefore not independent, repair by separating the verifier and the error mode
- `AP14 Untyped Selection`: `strict reverse-map`, `F1`, `D8` or `D9` is active but `{candidate_multiplicity, reviewability} ⊄ Σ(T)`, repair by typing those first and then applying `DP13`

---

## 10. Evaluation and Observability

Structure evaluation does not begin with outcomes alone. It first checks whether the structure is legal and admissible.

### Evaluation Stack

```text
G0 Structural validity
 -> G1 Safety / admissibility
 -> S1 Outcome
 -> S2 Efficiency
 -> S3 Mechanism fit
 -> S4 Robustness / adaptivity
 -> Final decision
```

### G0 Structural Validity

Must check:

- `T ⊢ wt_exec`
- `T ⊢ wt_select` when required
- `T ⊢ wt`
- whether all of `Req(Δ, Σ)` are satisfied
- whether any sanctioned composite is used without its legal antecedent
- whether there is obvious under-cover or over-build

hard fail:

- unmet obligation
- illegal selection
- illegal composite

weak pass:

- obligations are satisfied, but there is clear redundancy that leaves `G0` and `G1` unchanged when removed

### G1 Safety / Admissibility

Especially check:

- whether an exception path exists when `D5` is active
- whether independent verification exists when `D6` is active
- whether selection is typed when `D8/D9` are active
- whether the governance-heavy corner is closed when `D4a + D5 + recip` are present

### Scored Dimensions

- `S1 Outcome`
- `S2 Efficiency`
- `S3 Mechanism Fit`
- `S4 Robustness / Adaptivity`

### Reporting Scale

Use the following reporting scale for `S1~S4`:

- `strong`
- `adequate`
- `mixed`
- `weak`
- `unscored`

Prefix a score with `provisional_` when it is inferred structurally rather than measured empirically.

### Atom-Level Cost Ledger

```text
total_cost
 = cost_operator
 + cost_state
 + cost_coordinator
 + cost_evaluator
 + cost_rewriter
```

derived ratios:

- `coordination_share`
- `evaluation_share`
- `rewrite_share`
- `execution_share`
- `state_share`

### Event Attribution Rule

The atom cost ledger is computed at the event level, and each event is assigned to exactly one primary atom.

Priority order:

1. `Rewriter`
2. `Evaluator`
3. `Coordinator`
4. `State`
5. `Operator`

So the same event is never double-counted across two atoms.

### AP Warning Grades

- `strict AP warning`: a warning that points almost directly to a specific `AP` or `F`
- `mixed diagnostic cue`: a warning that suggests a specific failure but must be read with other structural signals
- `heuristic cue`: a weak warning that does not justify a conclusion without operating context

### Metric Dictionary

- `E1`: ratio of completed tasks that pass independent verification
- `E2`: share of total resources spent on coordination, waiting, and handoff; sharp increases point to `AP2`
- `E3`: effective variety, including actual error correlation rather than merely formal diversity; low values suggest `AP1`
- `E4`: exception frequency and successful handling rate; accumulation without rewrite suggests `AP4`
- `E5`: propagation depth from error occurrence to detection; deeper chains suggest `AP7`
- `E6`: proportion of defined governance rules that are actually enforced; low values point to `AP9`
- `E7a`: avoidable escalation rate, where cases that could have been resolved locally are escalated upward; high values suggest `AP12`
- `E7b`: missed escalation rate, where cross-dependent cases are decided locally and fail; high values suggest `AP11`
- `E8`: learning rate and stability, measured by rewrite frequency together with post-rewrite exception reduction
- `E9`: cognitive load and saturation at a single locus; spikes suggest `AP12` or a `D2a` bottleneck
- `E10`: rate at which required information actually reaches the decision packet at decision time; low values suggest `AP5`
- `E11`: integration rework rate; high values suggest `AP11`
- `E12`: alignment between feedback rank and realized performance rank; low or negative values suggest selection without learning or adverse selection

### D-Linked Reading Rule

- `D3`: `E10`, `E11`
- `D5`: `E4`
- `D5a`: `E8`
- `D6`: `E5`, `E6`
- `D7`: `E3`
- `D8/D9`: `E12`
- `D4/D4a`: `E7a`, `E7b`

### Standard Comparison Arms

Recommended comparison uses at least three arms:

- `B0`: simplest plausible structure
- `B1`: common-practice structure
- `P`: proposed core-derived structure

### Final Decision Rules

`Adopt`:

- `G0 = pass`
- `G1 = pass`
- outcome is non-inferior or better
- efficiency improves
- D-linked observables move in the predicted direction
- the atom cost ledger is structurally credible

`Revise`:

- `G0` or `G1` is only weak pass
- mechanism fit is off
- cost decomposition is distorted
- gates pass but key failure modes keep recurring

`Reject`:

- `G0 fail`
- `G1 fail`
- outcome is worse than baseline
- structural cost increases without a corresponding mechanism improvement

---

## 11. Worked Example: TF6 / Software Developers

A representative instantiation of `TF6 Technical Build / Engineering Delivery`:

```text
implement a feature across multiple modules,
with parallel work, interface alignment,
shared state continuity, and independent verification
```

### Base Typing

```text
Σ(T) = {par, handoff, continuity, checkable}
Δ(T) = {D1, D2, D3, D6}
Req(Δ, Σ) = {ω_par, ω_contract, ω_shared, ω_verify}
Γ = {DP2, DP12, DP6}
```

### Candidate Structures

`B0` single-operator:

```text
[Operator] -> [State]
```

`B1` naive parallel:

```text
[Operator] [Operator] [Operator]
        \    |    /
      [State(shared)]
```

`P` core-derived:

```text
[Operator] [Operator] [Operator] [Operator]
        \       |       |       /
      [State(contract+shared)]
                |
         [Evaluator(verify)]
```

### Structural Verdict

- `B0` does not close `Req(Δ, Σ)`; in particular it leaves `ω_par` and `ω_verify` unmet, so `G0(B0) = fail` and `G1(B0) = fail`.
- `B1` closes the parallel side but still leaves `ω_contract` and `ω_verify` unmet, so `G0(B1) = fail` and `G1(B1) = fail`.
- `P` closes `ω_par`, `ω_contract`, `ω_shared`, and `ω_verify` with `DP2 + DP12 + DP6`, so `G0(P) = pass` and `G1(P) = pass`.

Since there are no empirical run logs:

- `S1 = unscored`
- `S2 = unscored`
- `S3 = provisional_strong`
- `S4 = provisional_adequate`

framework verdict:

```text
Revise
```

That is, the structure is legal and shows strong mechanism fit, but without actual outcome and efficiency logs it is not yet an adopt.

---

## 12. Practical Reading Rule

The core procedure is:

```text
Task T
 -> type it
 -> derive active pressures
 -> generate required obligations
 -> choose a minimal admissible pattern cover
 -> render that cover in a general structure grammar
 -> evaluate the realized structure against baseline
```

A good structure is one that first closes:

```text
Req(Δ, Σ)
```

legally, admissibly, and minimally, and only then improves:

```text
verified outcome / total resource
```

more accurately and more stably than the relevant baseline.

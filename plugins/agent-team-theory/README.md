# Agent Team Theory

*What if we applied computational organization theory to agent team design?*

Organization theory has spent decades explaining why human teams need specific structures — capacity limits, delegation risks, coordination costs, information asymmetry. These aren't human problems. They're structural problems. And agents have them too.

Yet multi-agent systems are still designed by intuition: "let's add a planner, a coder, and a reviewer." No one asks whether that structure is sufficient, minimal, or even correct.

This plugin does. It takes established organizational theory — Ashby, Simon, Williamson, Galbraith — formalizes it into a generation grammar, and uses it to **derive** the team structure any task actually requires.

```
/team
```

Describe your task. The plugin analyzes it, derives the structure, shows it to you for confirmation, then builds the team and executes.

## From Organization Theory to Agent Teams

The pipeline borrows from four foundational results:

- **Ashby's Law of Requisite Variety** (1956) — a team's response repertoire must match the disturbance it faces
- **Simon's near-decomposability** (1962; Nobel 1978) — tasks decompose into modules, but residual interdependence remains
- **Williamson's transaction cost economics** (1975; Nobel 2009) — every boundary and handoff creates friction that structure must absorb
- **Galbraith's information processing view** (1973) — structure is a response to information load and uncertainty

These are synthesized into a closed formal system: 7 first principles → 9 organizational pressures → 16 obligations → 13 design patterns → 4 structural atoms. The pipeline compiles any task through this system and outputs the minimal team.

## How It Works

1. **Type the task.** Check 12 structural properties: can subtasks run in parallel? Do handoffs need contracts? Can outputs be independently verified?
2. **Identify pressures.** Which organizational problems are active? Capacity limits, delegation risk, recurring exceptions?
3. **Generate obligations.** Each pressure produces specific requirements the team must satisfy.
4. **Compute minimal cover.** Find the smallest set of design patterns that covers all obligations. If the task needs no team, it says so.
5. **Build, execute, evaluate.** Assign itself a role, spawn the remaining agents, run, then assess whether the structure worked.

## Four Primitives

Every team is composed from the same four atoms:

| Agent | Role |
|---|---|
| **Operator** | Does the work. Multiple can run in parallel or in sequence. |
| **Coordinator** | Routes tasks, enforces boundaries, handles exceptions. Never executes. |
| **Evaluator** | Verifies, selects, challenges. Structurally independent from Operators. |
| **Rewriter** | Diagnoses recurring failures, proposes rule changes. Appears only when needed. |

Plus **State** — shared substrate (contracts + cross-cutting context), not an agent.

These aren't role names chosen from a menu. They're the irreducible atoms that the theory derives from its first principles.

## Example

Task: migrate a legacy payment system to a new API while keeping live traffic running, maintaining regulatory audit logs, and preparing a rollback plan.

A coding agent might throw three workers at this. The pipeline sees more:

- Work spans parallel domains (migration, traffic, rollback) → multiple Operators
- Modules affect each other (migration state drives routing, routing drives rollback) → boundary enforcement
- Legacy edge cases will recur → exception handling + Rewriter
- Regulatory compliance requires independent verification → Evaluator
- Cross-domain coordination with bidirectional dependencies → governance stack

Derived structure — 14 obligations, 9 patterns, 8 atoms:

```
           [Coordinator]
          /       |       \
  [Operator]  [Operator]  [Operator]
  migration    traffic     rollback
          \       |       /
       [State(contract+shared)]
              |          |
  [Evaluator(verify)]  [Evaluator(monitor)]
              |
  [Coordinator(governance)]
              |
         [Rewriter]
```

Two Coordinators (one for dispatch, one for governance), three domain Operators, two Evaluators (verification + compliance monitoring), and a Rewriter for recurring legacy issues. Try designing that by intuition.

## No Orchestrator

After deriving the structure, the plugin assigns itself a role *within* the team — typically Coordinator. Not a supervisor above it. The Evaluator is always spawned independently (structural requirement, not a design choice).

When the pipeline determines no pressures are active, it executes solo. A team of one is a valid derivation, not a fallback.

## What the Theory Provides

| Component | What it answers |
|---|---|
| First Principles (P1-P7) | Why does structure exist at all? |
| Derived Pressures (D1-D9) | What can go wrong without it? |
| Design Patterns (DP1-DP13) | What structural responses are available? |
| Composition Calculus | How to compute the minimal sufficient team? |
| Structure Grammar | How to render it in universal primitives? |
| Task Families (TF1-TF8) | What do canonical structures look like? |
| Failure Semantics (F1-F5) | What broke and why? |
| Anti-Patterns (AP1-AP14) | What symptom maps to what cause? |
| Evaluation (G0-S4) | Did the structure actually work? |

Full theory (~900 lines): [`references/AGENT_TEAM_THEORY.md`](references/AGENT_TEAM_THEORY.md)

## Installation

Inside Claude Code:

```
/plugin marketplace add kekmodel/skills
/plugin install agent-team-theory@kekmodel-skills
```

Or from CLI:

```bash
claude plugins marketplace add kekmodel/skills
claude plugins install agent-team-theory
```

## License

MIT

## Author

JD Kim ([@kekmodel](https://github.com/kekmodel))

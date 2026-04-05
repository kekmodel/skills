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
- For that reason, this skill recommends from the supported execution set: **A0-A5, A7, A8**.
- **A6 (Swarm)** is a useful conceptual architecture for large homogeneous pooled work, but it is **not supported for direct execution here**. If A6 is the best conceptual fit, say so and recommend the closest supported fallback, usually A3 or A8.

## Step 1: A0 Fast-Path Check

Quick scan:
- Is this a single, self-contained task?
- Can one agent handle it without meaningful coordination overhead?
- Does it look like less than about 30 minutes of focused work?
- Is verification straightforward enough that failure would be obvious quickly?
- If it goes wrong, are the effects easy to undo, contain, or recover from?

If all answers are yes, recommend **A0**. If any answer is no or unclear, do the full analysis instead of taking the fast path.

## Step 2: 3-Axis Analysis

Analyze the task on these axes:

**Axis 1 — Interdependence**

| Signal | Type |
|--------|------|
| Subtasks share no state and can run independently | **Pooled** |
| Output of one step feeds the next | **Sequential** |
| Multiple actors must iterate on shared state | **Reciprocal** |
| The same task should be attempted independently, then selected or ranked after the fact | **Replicated** |

**Axis 2 — Scale × Heterogeneity**

| Scale | Heterogeneity | Meaning |
|-------|---------------|---------|
| Small (<3 subtasks) | Low | One person could mostly do it |
| Medium (3-8) | Medium | Several domains or perspectives needed |
| Large (8+) | High | Broad, multi-domain, or specialization-heavy |

**Axis 3 — Uncertainty**

| Level | Signal |
|-------|--------|
| Low | Concrete spec, clear success criteria |
| Medium | Some ambiguity, several valid approaches |
| High | Exploratory, underspecified, or subjective |

Also assess **verification risk** separately:
- Low: success is easy to test or verify
- High: success is hard to verify, subjective, or hidden-state dependent

Also assess **reversibility / externality** separately:
- Reversibility: if this goes wrong, how costly is it to undo, contain, or recover?
- Externality: do side effects stay local, or spread to users, systems, data, or downstream work?
- Low reversibility or high externality should make the recommendation more conservative.

## Step 3: Confidence Check

Rate your analysis confidence as **Low / Medium / High**:
- Low (roughly < 0.3): ask the user for clarification instead of forcing a recommendation
- Medium (roughly 0.3-0.5): prefer the lowest-cost architecture with clear escalation
- High (roughly >= 0.5): proceed normally

State why confidence is not high:
- missing task facts
- unclear success criteria
- hidden dependencies
- subjective quality bar
- uncertain verification path

## Step 4: Candidate Narrowing

Use these base candidates:

| Interdependence | Candidates |
|----------------|------------|
| Pooled | A3, A8 |
| Sequential | A2, or A1/A8 if the sequence is unstable |
| Reciprocal | A1, A4, A5 depending on scale |
| Replicated | A7 |

Then refine:

| Situation | Preference |
|----------|------------|
| Small, low-overhead | A0 or A1 |
| Medium, creative, reciprocal | A4 |
| Large, multi-domain | A5 or A8 |
| Pooled with specialist routing needs | A8 |
| Pooled with independent experts | A3 |
| Quality-first, several plausible independent attempts | A7 |
| Large homogeneous pooled work | A6 in theory, but unsupported here → recommend A3 or A8 with disclosure |

## Step 5: Overlays and Risk Checks

- High verification risk: add strong verification, or prefer A7 when independent attempts help
- High stakes: add adversarial review that actively tries to break the proposal
- Low reversibility or high externality: recommend tighter approval and independent verification before risky actions
- Long-running work: add governance / rule-revision support so repeated exceptions can change the rule, not just the latest step
- Multiple domains or obvious capability gap: add an external perspective with a meaningfully different brief
- When two recommendations look close, prefer the one that localizes mistakes, keeps escalation obvious, and hides fewer assumptions inside one agent

Before finalizing, check:

| Check | Risk | Fix |
|------|------|-----|
| Team too large for the task | Coordination explosion | Simplify or hierarchize |
| Everyone uses the same approach | Groupthink | Diversify prompts or roles |
| Reviewer shares executor context or rubric | Mirror verification | Re-isolate reviewer; different model is preferred but not required |
| No escalation path | Drift / stuck failure | Add explicit escalation |
| Delegation chain too deep | Endless routing | Cap depth |
| Independent work modeled as dependent | False modularity | Re-analyze interdependence |

## Step 6: Report

Present:
1. 3-axis analysis
2. Verification risk
3. Reversibility / externality assessment
4. Analysis confidence
5. Recommended architecture and why
6. Fallbacks considered
7. Overlay suggestions
8. Team composition preview
9. Risks and caveats
10. Decision path taken

This file is an analyze-only entry point, but it should teach the same judgment habits as `/team`: do not invent a separate decision policy.
If you use a pattern label, also explain the concrete behavior it implies.

If A6 is the best conceptual fit, explicitly say:
- why A6 fits in theory
- why it is not directly supported here
- which supported fallback you recommend instead

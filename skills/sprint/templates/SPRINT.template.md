# Sprint NN — {{SPRINT_TITLE}}

**Goal:** {{ONE_PARAGRAPH — what this sprint makes true that isn't true today.}}

**Definition of done:** {{CONCRETE, VERIFIABLE end-state — phrased so a director can test
it, not vibe it.}}

## Tracks

| Track | Slug/branch | Model | Scope |
|---|---|---|---|
| A | `sNN/{{slug}}` | {{tier}} | {{one-line scope}} |
<!-- init: 1-4 rows. Note any shared-scaffolding races between tracks and how the
director pre-empts them (pre-create shared skeletons before spawning, or stagger spawns). -->

## Gate 0 (before tracks spawn)

1. {{CHECK — credentials fresh, main green, store reachable, disk/quota, approvals.}}

## Mid-sprint gates

- {{GATE — external signups, reviews, anything human that tracks must not block on.
  Structure track scope so unmet mid-gates degrade gracefully (ship tested module +
  documented gap, not a stalled track).}}

## Merge order & rationale

{{ORDER + one line of why (who owns shared scaffolding/migrations, who deploys last).}}

## Close gate (operator + director together)

1. {{STEP — deploys the operator runs, end-to-end verifications with expected outputs,
   detached jobs to start + record in STATE.md.}}

## Risks specific to this sprint

- {{RISK + the mitigation already designed into a track's instructions.}}

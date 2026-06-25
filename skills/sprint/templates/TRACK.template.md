# Track {{LETTER}} — {{TRACK_TITLE}}

Branch `sNN/{{slug}}`. Read `DESIGN.md` ({{relevant sections}}) first.
{{ONE PARAGRAPH: what you build and why it matters to the sprint.}}

## What exists (verified — do not re-derive)

<!-- init/director: every fact the track needs that's already established — endpoints,
schemas, file IDs, repo patterns to follow, with "verified <date>" stamps. The test of a
good track file: a fresh subagent with NO other context executes end-to-end without
asking anything. -->
- {{FACT}}

## Build

1. {{STEP — concrete: files/modules to create, behaviors, edge cases to handle.}}
<!-- init/director: include the project's conventions inline where they bite (units,
timezones, batch sizes, naming) rather than assuming the agent internalized DESIGN.md. -->

## Acceptance gates (quote actual outputs in your report)

1. {{GATE — command/query + EXPECTED result. The director re-runs these at audit;
   mismatches between your quoted output and the director's re-run fail the track.}}

## Out of scope

{{EXPLICIT non-goals — neighboring work this track must not touch, future-sprint items
it must not pull forward, other tracks' files it must not edit.}}

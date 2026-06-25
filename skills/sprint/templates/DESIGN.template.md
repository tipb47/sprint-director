# DESIGN — {{PROJECT_NAME}} Contract

Canonical design contract. Tracks and directors obey this file; changes are
operator-approved and logged in `STATE.md`.

## 1. Mission & scope boundary

{{MISSION_PARAGRAPH}}
<!-- init: include explicit NON-goals / scope boundaries — what this arc will NOT build.
Scope creep dies here or nowhere. -->

## 2. Architecture

<!-- init: a compact diagram (ascii) + prose: languages, packages, deploy targets,
how components talk. Pull from interview + repo inspection. -->

## 3. Stores & surfaces

<!-- init: where data/artifacts live (DBs, buckets, registries), with the access mechanism
(connection method, secret location pattern — NEVER literal secrets). Table schemas if a
DB is involved; migration policy: migrations are FILES in {{MIGRATIONS_PATH}}, applied
ONLY by the director, in order, at merge time. -->

## 4. Conventions & reference constants

<!-- init: the constants agents need: naming, units, timezone policy (recommend UTC
everywhere, convert at ingest), ports, IDs, paths. Everything a track would otherwise
guess at or re-derive. -->

## 5. Verification contract

<!-- init: per interview — what makes work provably done. Test suites + commands; or
data-quality gates (verification queries with EXPECTED outputs, reference counts); build
artifacts; or a mix. Be concrete: a track quotes these outputs in its report and the
director re-runs them at audit. -->

## 6. Security & hygiene

- Tracks NEVER read or commit secrets/credential files; never weaken `.gitignore`.
- External account signups are operator gates — agents never create accounts.
<!-- init: add project-specific rules (PII, rate limits on external services, prod-DB
safety like batch sizes / no table locks). -->

## 7. Standing decisions ledger (operator-confirmed)

| Date | Decision |
|---|---|
| {{DATE}} | {{EACH_INTERVIEW_DECISION_AS_A_ROW}} |

# Sprint Director Guidelines (runbook)

You are a fresh Claude Code session directing the current sprint end to end: analyze →
gates → tracks → audit → merge → close. You write code only to resolve merge conflicts or
trivial audit fixes; tracks build.

## Phase 0 — Boot & analyze

1. Read `STATE.md`, `DESIGN.md`, `SPRINT_GUIDELINES.md`, then the current
   `sprints/sprint-NN/SPRINT.md` + its `TRACK-*.md` files.
2. Analyze sprint requirements **vs reality**:
   - `git pull --ff-only`; repo state vs the sprint doc's assumptions.
   - Verify claimed prior state against actual stores: {{STORE_CHECKS}}.
     <!-- init: concrete checks — DB queries via <mechanism>, bucket listings, artifact
     versions. Include known tooling quirks from STATE.md verified-facts. -->
   - Background jobs in `STATE.md`: check each manifest/status source — progressed,
     stalled, failed? A stalled critical job may reshape the sprint.
   - Credentials/tooling alive ({{CRED_CHECKS}}).
3. **If reality contradicts the sprint doc: amend `SPRINT.md`, log the deviation in
   `STATE.md`, THEN proceed.** Fiction in docs poisons the next session.

## Phase 1 — Clarify, reconcile, plan

1. Ask ALL blocking/open/clarifying questions (batched, concrete options; genuine
   operator decisions only — not reassurance).
2. **Reconcile & re-present.** If answers shift scope, amend `SPRINT.md` (and log in
   `STATE.md`), then re-surface any new ambiguity — loop until the sprint is unambiguous
   BEFORE planning. A track must never inherit a question you could have closed here.
3. Present Gate 0 items from `SPRINT.md`. Don't spawn tracks that depend on unmet gates.
4. Present the execution plan (plan mode where available): spawn order, parallelism,
   merge order, audit checks. On approval, execute.

## Phase 2 — Spawn tracks

- One subagent per track, prompt = absolute path to its `TRACK-*.md` + instruction to obey
  `SPRINT_GUIDELINES.md`. Model per the sprint file's per-track policy.
- Same-repo parallel tracks: worktree isolation; each pushes its own `sN/<slug>` branch.
- Independent tracks spawn in ONE message. While they run: handle gate items; never
  duplicate track work.

## Phase 3 — Audit & merge (in SPRINT.md's merge order)

1. Fetch the branch; review the FULL diff against `DESIGN.md`: contract conformance, no
   secrets, no scope creep.
2. **Re-run the track's verification gates yourself** against real state. Reports are
   claims; your audit produces facts.
3. {{MIGRATION_APPLY_STEP}}
   <!-- init: if DB — apply migration files in order after audit, verify schema + run
   verification queries; if none, drop this step. -->
4. Rebase onto main if needed; resolve conflicts yourself; re-run the verification
   contract on the branch; merge `--no-ff`; re-run on main; push.
5. Long-running jobs a track delivered: verify the sample chunk end-to-end, then start the
   full run detached and record it in `STATE.md` § Background jobs.

## Phase 4 — Close

1. Walk the operator through the close gate as an actionable checklist (exact commands).
2. Update `STATE.md`: shipped-per-track, deviations, background jobs, learnings (update
   GUIDELINES/DESIGN if warranted — log that you did). Propose promoting project-agnostic
   learnings to the sprint skill's templates.
3. Draft/amend the NEXT sprint's files from what actually happened (ROADMAP is the
   skeleton; reality wins).
4. Final report: outcomes, merge summary, gate status, background-job dashboard,
   next-sprint pointer + kickoff instructions.

## Respawn protocol

Resuming a half-finished sprint: `STATE.md` + `git branch -a` + worktree list + job
manifests tell you where it stopped. Track branches are durable; re-audit anything
unmerged rather than trusting prior-session memory.

## Hard rules

- All `SPRINT_GUIDELINES.md` git rules. Verify "pushed"/"loaded"/"running" claims against
  git/store/process state itself.
- Long jobs: detached + manifest; never a foreground job that dies with your session.
  Check actual process liveness, not just manifest freshness.
- Never merge red; never spawn tracks on a red main; never apply un-reviewed SQL/DDL;
  never let a track apply DDL.
- {{PROD_SAFETY_RULES}}
  <!-- init: project-specific production-safety rules (shared live DBs, rate limits,
  deploy freezes), from interview. -->
- Keep token use lean: direct, audit, merge — don't rebuild.

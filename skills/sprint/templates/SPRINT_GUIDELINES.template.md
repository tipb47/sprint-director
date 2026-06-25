# Sprint Guidelines

Rules of work for every sprint, track, and director. Director runbook:
`DIRECTOR_GUIDELINES.md`. Both obey this file and `DESIGN.md`.

## What a sprint is

- One bounded milestone, run end-to-end by ONE fresh Claude Code session (the
  **director**), which spawns each **track** as a subagent.
- {{TRACK_COUNT_POLICY}} tracks per sprint.
  <!-- init: 1-3 for pipeline-shaped projects with sequential dependencies; 2-4 for
  multi-surface products. State which shape this project is and why. -->
- Sprints contain **operator gates** — explicit pauses for the human
  ({{GATE_EXAMPLES_FROM_INTERVIEW}}). Gates live in `SPRINT.md`, never inside track
  prompts. Gate 0 runs before tracks spawn; the close gate runs after merges.

## Track rules

1. **Self-contained prompts.** A track reads its `TRACK-*.md` + `DESIGN.md` and needs
   nothing else. Mid-flight human input required = the sprint was planned wrong.
2. **No blocking questions.** Ambiguity → decide per `DESIGN.md`, log under "Decisions &
   open questions" in the track report.
3. **Branch discipline.** One branch per track: `sN/<slug>`, from latest `origin/main`.
   Push the branch; never merge, never touch main. Same-repo parallel tracks run in
   worktrees.
4. **The verification contract (DESIGN §5) is the gate.**
   <!-- init: instantiate concretely — exact test commands; for data work: tracks ship
   verification queries WITH expected results and reports quote ACTUAL output. -->
5. **Never trust, always verify.** "Done"/"pushed"/"loaded" are claims; the report quotes
   command/query output, not assertions. Directors re-verify at audit.
6. {{MIGRATIONS_RULE}}
   <!-- init: if DB: "Migrations are files in <path>, applied ONLY by the director at
   merge time; tracks code against the post-migration schema." If no DB: drop this rule. -->
7. **No fabricated data, ever,** outside clearly-marked test fixtures under test dirs.
   A job that can't fetch real data records the failure — it never fabricates rows.
8. **Report format:** what shipped; branch + commits; verification outputs (verbatim);
   files changed; decisions & open questions; what the director must verify by hand.
<!-- init: if the project has long-running jobs, add the AFK-grade standard as a rule:
checkpoint manifest, detached execution, incremental sync, inline validation, idempotent
re-runs — tracks deliver the harness verified on a sample chunk; full runs start at close. -->

## Definition of done (sprint)

- All track branches audited, merged `--no-ff` to main in SPRINT.md's order, pushed;
  verification contract green on main after each merge.
- {{DB_DONE_CLAUSE}} <!-- init: migrations applied + verified, if DB; else drop. -->
- Operator close-gate checklist delivered/executed.
- `STATE.md` updated; next sprint's files drafted or amended from reality.

## Token discipline

- One subagent per track; no orchestration frameworks unless the operator explicitly asks.
- Director audits diffs directly; at most ONE review subagent per sprint, only for a
  genuinely risky merge.
- Tracks read what their file points to; no exploratory fan-outs — `DESIGN.md` is the map.
- Model policy: {{MODEL_POLICY}}.
  <!-- init: from interview — default tier for tracks; what escalates to the strongest
  tier (contract/interface-defining work is the usual answer). Directors run on the
  strongest available tier. -->

## Git conventions

- Commits: imperative summary, prefixed with the track tag `[sNN/<slug>]` (director
  ops/merge commits use `[sNN/ops]`); end with the project's standard co-author trailer.
- Stage with EXPLICIT paths — never `git add -A` / `git add .`.
- **Ops bookkeeping stays out of code commits.** {{OPS_COMMIT_POLICY}}
  <!-- init: resolve from the interview's tracked/untracked answer:
  • UNTRACKED → "The <ops> dir is gitignored; never stage it."
  • TRACKED → "<ops> churn (STATE.md / SPRINT.md / TRACK-*.md) commits SEPARATELY from
    code, tagged `[sNN/ops]` — never mixed into a code or merge commit. Keeps the code
    history clean and bisectable." -->
- With worktrees, ALWAYS `git -C <absolute-path>`.
- Never chain push/merge after a PIPED build/test in one compound — pipes mask exit codes.
  Verify, check the exit code, THEN push. Judge builds by exit code, not stderr noise.
- Director merges `--no-ff` per track; never force-push main.
<!-- init: if out-of-band human pushers exist: add "pull + check their recent commits at
boot; never clobber; rebase, don't force; keep changes to their hot files minimal+flagged." -->

---
name: sprint
description: Sprint-directed multi-agent development methodology. Use for `/sprint init` (scaffold the ops/ sprint system into a project, interview-driven), `/sprint direct` (boot as sprint director for the current project's active sprint), `/sprint status` (cross-project dashboard of all registered sprint projects). Invoke when the user mentions sprint directing, sprint methodology, starting/running a sprint, or sprint status.
argument-hint: init | direct | status
---

# Sprint System — methodology engine

This skill is the **engine** for a sprint-directed development methodology. Each project
keeps its own **instance**: an `ops/` folder holding its design contract, guidelines,
roadmap, sprint files, and a living state ledger. **The project's local ops/ docs are
always authoritative — wherever they conflict with this skill, the local docs win.** This
skill provides the boot rituals, the scaffolding templates, and the cross-project registry.

## The methodology in one paragraph

Work is divided into **sprints**: bounded milestones, each run end-to-end by ONE fresh
Claude Code session (the **director**) for context hygiene. The director never builds; it
analyzes, gates, spawns **track** subagents (each with a self-contained `TRACK-*.md`
prompt, each on its own git branch, parallel tracks in git worktrees), then audits their
diffs, verifies their claims against real state, resolves conflicts, merges in a defined
order, and closes by updating the ledger (`STATE.md`) and drafting the next sprint from
what actually happened. Humans appear at **operator gates** — explicit pauses for
deployments, account signups, approvals — never as mid-flight track interruptions.

## Subcommand routing

Parse the argument. No argument or unrecognized → print the usage summary below and stop.

```
/sprint init    — scaffold or adopt the sprint system in the current project
/sprint direct  — become the sprint director for this project's current sprint
/sprint status  — dashboard across all registered sprint projects
```

---

## /sprint init

Scaffold the sprint system into the current project, interview-driven. Never overwrite
existing files.

1. **Detect existing instance.** Look for an ops folder (`ops/`, `.docs/ops/`, or ask).
   If one exists → **adopt mode**: map its files against the canonical layout (below),
   report gaps (no STATE.md? no guidelines? missing sections?), offer to fill ONLY the
   gaps using the templates, then register (step 5) and stop. Never modify existing docs
   beyond explicitly approved gap-fills.
2. **Interview** (AskUserQuestion, batched — adapt wording to what you can already see in
   the repo; skip questions the codebase answers):
   - Ops location and git tracking: `ops/` vs `.docs/ops/`; tracked or untracked? If
     tracked, ops bookkeeping (STATE.md/SPRINT.md churn) commits SEPARATELY from code under
     an `[sNN/ops]` tag; if untracked, the ops dir is gitignored. Either way the code
     history stays clean — encode the resulting rule into the generated guidelines.
   - Project surfaces: repos/packages/deploy targets a sprint touches; how deploys happen
     (manual command? CI?) — deploys are usually operator gates.
   - Team shape: solo, or other humans pushing out-of-band? (Out-of-band pushers need
     pull-and-check-recent-commits rules in the guidelines.)
   - Verification contract: what makes a track's work *provably* done — test suites,
     data-quality gates (row counts / verification queries with expected output), build
     artifacts, or a mix? This shapes the guidelines' "gates" section.
   - Database/migrations: does the project have a DB? If so: migrations are FILES tracks
     write and ONLY the director applies — confirm the apply mechanism (CLI, MCP, etc.).
   - Track model policy: which model tier for track subagents by default, and what class
     of work escalates to the strongest tier (contract/interface-defining work is the
     usual answer)?
   - Roadmap seed: the next 3–6 milestone-level goals, and which ONE becomes sprint-01.

   After the fixed questions, do a **clarify pass**: if the roadmap seed or sprint-01 scope
   is ambiguous, contradictory, or too thin for a self-contained track, ask focused
   follow-ups (AskUserQuestion) until sprint-01 can be written with zero placeholders.
   Never generate hollow docs over unresolved ambiguity.
3. **Generate** from `templates/` (this skill's folder), substituting interview answers
   for `{{PLACEHOLDERS}}` and resolving every `<!-- init: ... -->` directive (these
   comments instruct YOU what to fill; they never survive into generated files):
   `README.md`, `STATE.md`, `DESIGN.md`, `SPRINT_GUIDELINES.md`, `DIRECTOR_GUIDELINES.md`,
   `ROADMAP.md`, and `sprints/sprint-01/SPRINT.md` (+ one `TRACK-*.md` per sprint-01 track).
   Sprint-01 track files must be genuinely self-contained: a fresh subagent with no other
   context executes one end-to-end.
4. **Quality bar:** generated docs must contain zero TODOs and zero template placeholders.
   If an interview answer is "not sure yet", write the current best decision into the doc
   and log it as provisional in STATE.md's decisions table — docs ship decided, not hollow.
5. **Register** the project in the cross-project registry at `~/.claude/sprint/projects.json`
   (`{name, path, ops}`). Create the file (and its directory) if missing.
6. Print the kickoff instructions: the user starts a FRESH session and runs `/sprint direct`
   (or pastes the kickoff prompt from the generated README).

### Canonical instance layout

```
<ops>/
├── README.md               ← layout + kickoff prompt for fresh director sessions
├── STATE.md                ← living ledger: current sprint, decisions, history, running jobs. READ FIRST.
├── DESIGN.md               ← canonical contract: architecture, schemas, conventions, standing decisions
├── SPRINT_GUIDELINES.md    ← rules for tracks + directors: gates, branch discipline, definition of done
├── DIRECTOR_GUIDELINES.md  ← director runbook: boot → analyze → gate → tracks → audit → merge → close
├── ROADMAP.md              ← sprint arc at milestone level (reality wins over this outline)
└── sprints/sprint-NN/
    ├── SPRINT.md           ← sprint brief: goal, gates, tracks, merge order, close checklist
    └── TRACK-*.md          ← self-contained prompt per track subagent
```

---

## /sprint direct

Become the sprint director for the current project's active sprint.

1. **Locate the instance:** current project in `~/.claude/sprint/projects.json`, else detect
   an ops folder, else tell the user to run `/sprint init`.
2. **Boot in this order:** `STATE.md` (current-sprint pointer, decisions, running jobs) →
   `DIRECTOR_GUIDELINES.md` → `SPRINT_GUIDELINES.md` + `DESIGN.md` → the current
   `sprints/sprint-NN/SPRINT.md` and its `TRACK-*.md` files.
3. **Analyze requirements vs reality** before anything else (the project's director
   guidelines define specifics; universal minimum): `git pull --ff-only` and compare repo
   state against the sprint doc's assumptions; verify claimed prior state against actual
   stores (DBs, buckets, artifacts); check any background-job manifests/status files
   STATE.md references — progressed, stalled, failed?; confirm credentials/tooling the
   sprint needs are alive. **If reality contradicts the sprint doc: amend SPRINT.md and
   log the deviation in STATE.md BEFORE spawning tracks.** Docs never drift from what runs.
4. **Clarify, reconcile, then plan:** raise ALL blocking/open/clarifying questions via
   AskUserQuestion (batched, concrete options; genuine operator decisions only). If answers
   shift scope, reconcile — amend SPRINT.md and log in STATE.md — then re-surface any new
   ambiguity, looping until the sprint is unambiguous. A track must never inherit a question
   you could have closed here. Present Gate 0 items. Then present your execution plan (plan
   mode where available): spawn order, parallelism, merge order, what you'll verify at
   audit. On approval, execute.
5. **Run the sprint** per the local DIRECTOR_GUIDELINES. Universal rules that always apply:
   - Directors direct: write code only for merge conflicts and trivial audit fixes.
   - One subagent per track; parallel same-repo tracks get `isolation: "worktree"`;
     independent tracks spawn in one message.
   - **Verify, never trust:** "pushed"/"loaded"/"done" are claims — check git/remote/DB/
     artifact state yourself before acting on them. Re-run tracks' verification gates
     yourself at audit.
   - Audit the FULL diff per track against DESIGN.md before merging; merge `--no-ff` in
     SPRINT.md's stated order; suites green on main after each merge.
   - Track branches are durable; if resuming a dead session, re-audit anything unmerged
     rather than trusting prior-session memory.
6. **Close:** operator close-gate as an actionable checklist; update STATE.md (shipped,
   deviations, learnings, background jobs); draft/amend the next sprint's files from what
   actually happened; final report with the next kickoff pointer.
   **Promotion path:** if a learning is project-agnostic (a git scar, an audit technique),
   propose promoting it into this skill's templates — keep the engine improving.

### Universal scars (hard-won; bake into every project's guidelines)

- With worktrees, ALWAYS `git -C <absolute-path>` — a bare git command in a stale cwd has
  reset a branch mid-merge.
- Stage with EXPLICIT paths; never `git add -A`/`git add .` — bulk adds have swept local
  config into pushed commits.
- Never chain push/merge after a PIPED build/test in one compound command — the pipe masks
  the exit code. Verify, check exit code, THEN push. Judge builds by exit code, not stderr.
- A green suite can mask a silently-skipping subtree: compare skip COUNTS against the
  track's reported numbers, not just exit codes.
- Before trusting an e2e run, check what's actually listening on the port — zombie dev
  servers from dead agents produce phantom results.
- Check a misbehaving process's ACTUAL environment (`/proc/<pid>/environ`), not its `.env`
  file — dotenv never overrides an already-exported shell var.
- A clean `synth`/`build`/`plan` does NOT prove a deployed artifact runs — infrastructure-as-code
  only validates the spec, not the runtime. After deploying a scheduled or containerized job, run
  it ONCE in its real environment and check the actual effect (correct exit code AND the row/file
  it should produce), never just the synth. (Container gotcha: a `command` override on an image
  with a hardcoded `ENTRYPOINT` is often APPENDED as args rather than substituted — the spec looks
  right but the program never changes. To change what actually runs, override the entrypoint, not
  the command.)
- The real-environment run-once must check the **EXIT CODE**, not just that the row/file landed —
  a job can do its work correctly and STILL exit non-zero. (A native extension segfaulting during
  interpreter teardown — after the real work has already committed — is a classic case: the
  orchestrator sees exit≠0, marks a fully-successful job FAILED, and retry-loops forever. Neither
  `synth` nor a local run catches it; only the real-environment run does. **Fix for a one-shot
  CLI/array-job unit:** flush stdout/stderr and hard-exit with the intended code after the
  idempotent write has committed (e.g. `os._exit(code)` in `__main__`) to bypass the buggy
  teardown — never let a native-library finalization crash flip a committed job to FAILED.)

---

## /sprint status

Cross-project dashboard.

1. Read `~/.claude/sprint/projects.json`. Empty/missing → say so, point at `/sprint init`.
2. Per project: read `<ops>/STATE.md` — extract current-sprint pointer, status line,
   last-updated date, background-jobs table (if present, also check any referenced
   manifest/status files for staleness: `updated_at` older than expected cadence = flag),
   and open items. Unreadable path → report it as broken registration, don't crash the dashboard.
3. Report a compact table: project | current sprint | status | background jobs (healthy/
   stalled/none) | last updated | top open item. Flag anything that needs human attention.

## Registry format (`~/.claude/sprint/projects.json`)

```json
{ "projects": [ { "name": "<short-name>", "path": "/abs/path/to/repo", "ops": "/abs/path/to/ops" } ] }
```

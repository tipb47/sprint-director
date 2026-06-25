# Sprint Director

**A sprint-directed, multi-agent development methodology for [Claude Code](https://claude.com/claude-code).**

Sprint Director is a single Claude Code **skill** (`/sprint`) that turns a messy "spawn a bunch
of agents and hope" workflow into a disciplined assembly line. Work is divided into **sprints** —
bounded milestones. Each sprint is run end-to-end by ONE fresh Claude Code session, the
**director**. The director never writes feature code. It analyzes, gates, spawns **track**
subagents (each isolated on its own git branch / worktree), audits their diffs against a design
contract, verifies their claims against real state, merges in a defined order, and closes by
updating a living state ledger and drafting the next sprint from what actually happened.

Humans appear only at **operator gates** — explicit pauses for deployments, account signups, and
approvals — never as mid-flight interruptions of a running agent.

> One engine, many instances. The skill is the reusable engine. Each project you run it on gets
> its own self-contained `ops/` folder (design contract, guidelines, roadmap, sprint files, state
> ledger). **The project's local `ops/` docs are always authoritative** — where they disagree with
> the engine, the local docs win.

---

## Why use it

- **Context hygiene.** A fresh director per sprint means no context rot across a long build. The
  ledger (`STATE.md`), not the chat history, is the source of truth.
- **Parallelism without collisions.** Independent tracks run as separate subagents in separate git
  worktrees, each on its own branch. The director merges `--no-ff` in a planned order.
- **Verify, never trust.** "Done" / "pushed" / "loaded" are treated as *claims*. The director
  re-runs every track's verification gates against real state before merging.
- **No hollow docs.** `/sprint init` interviews you and refuses to generate placeholder-laden
  scaffolding — every sprint ships decided, not "TODO".
- **Hard-won scars baked in.** Git footguns, zombie dev servers, IaC-synth-isn't-runtime, and
  other real failure modes are encoded as standing rules.

---

## Install

Sprint Director ships as a Claude Code plugin **and** as a plain skill folder. Pick one.

### Option A — Plugin (recommended)

From inside Claude Code:

```
/plugin marketplace add tipb47/sprint-director
/plugin install sprint-director@sprint-director
```

Restart the session (or `/exit` and relaunch) so the `/sprint` skill loads. Verify:

```
/sprint
```

…should print the usage summary (`init | direct | status`).

### Option B — Manual copy

No marketplace, just drop the skill into your user skills directory:

```bash
git clone https://github.com/tipb47/sprint-director.git
cp -r sprint-director/skills/sprint ~/.claude/skills/sprint
```

Restart Claude Code. The `/sprint` skill is now available globally.

> **Requirements:** Claude Code, a `git` repo per project you direct, and (recommended) the ability
> to run subagents in git worktrees. The methodology is git-centric; non-git projects work but lose
> branch isolation.

---

## Usage

The skill has three subcommands.

### `/sprint init` — scaffold the system into a project

Run it inside the project you want to manage. It interviews you (ops location, project surfaces,
team shape, verification contract, DB/migrations, model policy, and a 3–6 milestone roadmap seed),
then generates a complete `ops/` instance with a fully-detailed `sprint-01`. If the project already
has an ops folder, it switches to **adopt mode** and fills only the gaps.

It then registers the project in `~/.claude/sprint/projects.json` so `/sprint status` can see it.

### `/sprint direct` — become the director for the current sprint

Run this in a **fresh session**. The director boots the ledger and guidelines, analyzes the sprint
requirements *against current reality* (pulls the repo, checks stores, checks background jobs),
clarifies any open questions with you, presents an execution plan for approval, then runs the
sprint end-to-end: spawn tracks → audit → merge → close.

### `/sprint status` — cross-project dashboard

Reads every registered project's `STATE.md` and prints a compact table: current sprint, status,
background-job health, last updated, top open item. Flags anything that needs human attention.

---

## The `ops/` instance layout

Generated per project by `/sprint init`:

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

The engine's templates for these live in [`skills/sprint/templates/`](skills/sprint/templates).

---

## How the pieces fit

| Role | Who | Responsibility |
|---|---|---|
| **Engine** | This skill (`/sprint`) | Boot rituals, scaffolding templates, cross-project registry |
| **Director** | One fresh Claude Code session per sprint | Analyze, gate, spawn, audit, merge, close — never builds |
| **Track** | A subagent per unit of work | Builds on its own branch; reports verifiable output; never merges |
| **Operator** | You, the human | Appears only at gates: approvals, deploys, signups |

State lives in files, not chat:

- `~/.claude/sprint/projects.json` — the cross-project registry (per-user, never committed).
- `<project>/ops/STATE.md` — the living ledger the director reads first and updates last.

---

## Concepts in one screen

- **Sprint** — a bounded milestone with a goal, a definition of done, gates, tracks, and a merge order.
- **Track** — a self-contained unit of work. Its `TRACK-*.md` is a complete prompt: a fresh subagent
  with no other context can execute it end-to-end. One branch per track (`sNN/<slug>`).
- **Operator gate** — an explicit pause for the human. Gate 0 runs before tracks spawn; the close
  gate runs after merges. Gates live in `SPRINT.md`, never inside track prompts.
- **Verification contract** — what makes work *provably* done (test commands, data-quality queries
  with expected output, build artifacts). Tracks quote actual output; the director re-runs it at audit.
- **Scars** — hard-won failure modes encoded as standing rules, so the same footgun is never fired twice.

---

## FAQ

**Does this lock me into a folder structure?** No. The local `ops/` docs are authoritative; the
engine adapts. `init` even has an adopt mode for projects that already have an ops folder.

**Do I need GitHub / a remote?** No. It's `git`-centric (branches, worktrees, `--no-ff` merges) but
works fine on a purely local repo. Remotes just add `git pull`/`push` steps the director already handles.

**Can other AI agents set this up?** Yes — see [`AGENTS.md`](AGENTS.md) for a deterministic,
machine-followable install and bootstrap procedure.

**Where does my data go?** Nowhere external. Everything is local files: the per-project `ops/` folder
and the per-user registry at `~/.claude/sprint/projects.json`.

---

## Contributing

Issues and PRs welcome. The most valuable contributions are **new scars** — concrete, reproducible
failure modes with the rule that prevents them — and template improvements that generalize across
projects. Keep additions project-agnostic: no traces of any specific company's infrastructure.

## License

[MIT](LICENSE) © 2026 tipb47

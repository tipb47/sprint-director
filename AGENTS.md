# AGENTS.md — setup & operation for AI agents

This file is a deterministic, machine-followable guide for an AI coding agent (Claude Code or
similar) tasked with **installing** Sprint Director for a user, or **operating** it on their behalf.
Humans: see [README.md](README.md).

If you are an agent and the user said "set up sprint director" / "install the sprint skill", follow
**Part 1**. If they said "run a sprint" / "be the director", follow **Part 2**.

---

## Part 1 — Install

### Preconditions to check first

1. The host is running **Claude Code** (this skill targets Claude Code's skill/plugin system).
2. `~/.claude/` exists (it does if Claude Code has run at least once).
3. `git` is available (`git --version`).

### Install path A — plugin (preferred if the user uses plugins)

These are slash commands the **user** runs in their Claude Code session; you cannot run them via
shell. Print them and ask the user to run them:

```
/plugin marketplace add tipb47/sprint-director
/plugin install sprint-director@sprint-director
```

Then tell the user to restart the session and confirm with `/sprint`.

### Install path B — manual copy (you can do this yourself via shell)

```bash
# Clone to a temp location and copy just the skill into the user's global skills dir.
git clone https://github.com/tipb47/sprint-director.git /tmp/sprint-director
mkdir -p ~/.claude/skills
cp -r /tmp/sprint-director/skills/sprint ~/.claude/skills/sprint
```

Verification (the skill is correctly installed if this prints a YAML frontmatter block with
`name: sprint`):

```bash
head -5 ~/.claude/skills/sprint/SKILL.md
```

After copying, the user must restart Claude Code for the `/sprint` skill to register. The skill is
then available in **every** project, globally.

### What gets created at runtime (do NOT create these by hand)

- `~/.claude/sprint/projects.json` — the cross-project registry. Created automatically the first
  time `/sprint init` registers a project. It is per-user runtime state and must **never** be
  committed to any repo.
- `<project>/ops/` (or `<project>/.docs/ops/`) — the per-project instance, created by `/sprint init`.

---

## Part 2 — Operate

The skill exposes three subcommands. As an agent you typically *invoke the skill* and let its own
instructions drive; this section is orientation so you know what each does and what state it touches.

| Command | When | Reads | Writes |
|---|---|---|---|
| `/sprint init` | Once per project, to scaffold | repo, user interview | `<ops>/*`, registry |
| `/sprint direct` | Start of each sprint, in a **fresh** session | `STATE.md`, guidelines, `SPRINT.md` | branches, merges, `STATE.md` |
| `/sprint status` | Anytime | registry + each `STATE.md` | nothing |

### Invariants you must preserve when operating

- **The director never writes feature code.** It analyzes, gates, spawns tracks, audits, merges,
  and closes. Feature code is written only by track subagents (except trivial merge-conflict or
  audit fixes).
- **One subagent per track**, each on its own branch `sNN/<slug>`. Same-repo parallel tracks run
  with `isolation: "worktree"`. Independent tracks are spawned in a single message.
- **Verify, never trust.** Treat every "done/pushed/loaded" as a claim. Re-check git/remote/DB/
  artifact state before acting. Re-run each track's verification gates at audit.
- **Local `ops/` docs are authoritative.** Where they conflict with the skill engine, the local
  docs win. Never silently override them.
- **State lives in files.** Read `STATE.md` first; update it last. Do not rely on chat history as
  the source of truth.
- **Operator gates are human-only.** Never auto-approve a deploy, account signup, or other gated
  action on the user's behalf — surface it and wait.

### Registry shape (`~/.claude/sprint/projects.json`)

```json
{ "projects": [ { "name": "<short-name>", "path": "/abs/path/to/repo", "ops": "/abs/path/to/ops" } ] }
```

If you must read or repair it, preserve this exact shape. Missing file == no projects registered
yet (point the user at `/sprint init`).

---

## Safety notes for agents

- Never commit `projects.json` or any `ops/` bookkeeping into a code commit. If ops is git-tracked,
  it commits **separately** under an `[sNN/ops]` tag; if untracked, it is gitignored.
- Never weaken a project's `.gitignore`, read or commit secrets, or create external accounts — those
  are operator gates.
- Stage with **explicit paths**; never `git add -A` / `git add .`.
- With worktrees, always `git -C <absolute-path>` — a bare git command in a stale cwd can reset a
  branch mid-merge.

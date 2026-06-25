# {{PROJECT_NAME}} Ops — Sprint System

Sprint-directed build of {{ONE_LINE_MISSION}}.
<!-- init: state whether this folder is git-tracked, per interview. -->

## How to start a sprint (kickoff for a fresh Claude Code session)

Run `/sprint direct` in a fresh session, or paste:

> You are the {{PROJECT_NAME}} sprint director. Read `{{OPS_PATH}}/STATE.md`, then
> `{{OPS_PATH}}/DIRECTOR_GUIDELINES.md`, then the sprint files STATE.md points to.
> Analyze the sprint requirements against current reality (repo, stores, background jobs).
> Ask all blocking, open, and clarifying questions. Present your execution plan for
> approval. Then run the sprint end to end.

**Contract rule:** `SPRINT.md` is the *what*; the director's plan is the *how*. If
analysis shows the sprint doc no longer matches reality, amend `SPRINT.md` and log the
deviation in `STATE.md` BEFORE spawning tracks.

## Layout

```
{{OPS_DIRNAME}}/
├── README.md                  ← you are here
├── STATE.md                   ← living ledger. READ FIRST.
├── DESIGN.md                  ← canonical design contract
├── SPRINT_GUIDELINES.md       ← rules for tracks + directors
├── DIRECTOR_GUIDELINES.md     ← the director runbook
├── ROADMAP.md                 ← sprint arc at milestone level
└── sprints/sprint-NN/
    ├── SPRINT.md              ← sprint brief: goal, gates, tracks, merge order
    └── TRACK-*.md             ← self-contained prompt per track subagent
```

## Project surfaces

<!-- init: table of repos/packages/deploy targets + how each deploys, from interview. -->
| Surface | Where | Notes |
|---|---|---|
| {{SURFACE}} | {{PATH}} | {{DEPLOY_NOTES}} |

<!-- init: team shape note — solo, or named out-of-band pushers and the rules that implies. -->

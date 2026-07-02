# project-inspector

A Claude Code plugin that turns an existing repository into an AI-ready, audit-tracked project: it generates `CLAUDE.md`, a topic-indexed `AUDIT.md`, a `docs/project/` knowledge base, and detailed per-topic audit files — one approval-gated step at a time, without ever touching source code.

Its output is the context layer consumed by the rest of the Ono Apps dev cycle: downstream tooling can require `CLAUDE.md` and read `AUDIT.md` + `docs/`, and product/spec teams can use `docs/project/overview.md` and `components.md` before writing specs.

## Workflow

```
/inspect ──────────────► CLAUDE.md + AUDIT.md          (project-analysis)
     │
/inspect-docs ─────────► docs/project/                 (project-docs)
     │                     ├── overview.md      ← SPAC team reads this
     │                     ├── components.md    ← SPAC team + DD generation
     │                     ├── patterns.md      ← Claude dev sessions
     │                     └── integrations.md
     │
/inspect-breakdown ────► audits/<topic>/<topic>-audit.md   (audit-breakdown)
     │                   one topic per run, Status: Draft
     │
  developer reviews the Draft
     │
/inspect-sync ─────────► AUDIT.md row → Approved            (audit-sync)
                         CLAUDE.md managed blocks updated
                         (Caution Areas + Important Files)

Downstream consumers:
  every Claude Code session → sees approved findings via CLAUDE.md
  any spec/design workflow  → can read docs/project/overview.md + components.md
```

`/inspect-status` at any point prints the topic table, counts, and docs freshness.

## Target-repo output layout

```
<repo-root>/
├── CLAUDE.md              # compact AI context (required by dev-design-start)
├── AUDIT.md               # audit topic index (| # | Status | Topic | Priority | File | Notes |)
├── audits/
│   └── <topic-slug>/<topic-slug>-audit.md
└── docs/project/
    ├── overview.md
    ├── components.md
    ├── patterns.md
    └── integrations.md
```

## Components

| Piece | What it does |
|-------|--------------|
| `skills/project-analysis` | Step 1 — scans the repo (read-only), writes CLAUDE.md + AUDIT.md |
| `skills/project-docs` | Step 2 — writes the docs/project/ knowledge base; requires CLAUDE.md |
| `skills/audit-breakdown` | Step 3 — expands exactly one Pending Breakdown topic into a Draft audit |
| `skills/audit-sync` | Step 4 — on explicit developer approval, marks topics Approved and syncs HIGH/MEDIUM findings into CLAUDE.md's managed blocks |
| `agents/repo-scanner` | Read-only investigation subagent the skills delegate broad scans to |
| `hooks/` + `scripts/guard-readonly.sh` | PreToolUse hook that mechanically blocks destructive shell commands while an inspector skill is running |
| `commands/` | `/inspect`, `/inspect-docs`, `/inspect-breakdown [topic]`, `/inspect-sync [topics]`, `/inspect-status` |

## Safety model

Three layers keep inspection runs read-only:

1. **Prose constraints** in each SKILL.md (allowed/forbidden commands, output contracts).
2. **Agent tool scoping** — `repo-scanner` is instructed read-only and returns digests, not mutations.
3. **Mechanical enforcement** — each skill creates `~/.claude/project-inspector.active` on start and removes it on finish. While that marker exists, the PreToolUse hook blocks `rm`, `mv`, `cp`, `touch`, `tee`, `sed -i`, state-changing `git` commands, and output redirection (except `/dev/null`). When no inspector skill is running, the hook allows everything — it never interferes with normal dev work.

Approval gates: `audit-breakdown` processes one topic per run and stops; only the developer approves Drafts; `audit-sync` acts only on explicitly named topics.

## Install

Local (for development of the plugin itself):

```bash
claude --plugin-dir /path/to/onoapps-claude-plugins/project-inspector
```

Team distribution: add this repo to the `onoapps-claude-plugins` marketplace, then:

```bash
/plugin install project-inspector@onoapps-claude-plugins
```

> After installing the plugin, remove the loose copies of `project-analysis` and `audit-breakdown` from `~/.claude/skills/` to avoid double-triggering.

# Agents & Skills — Setup Source of Truth

Verbatim definitions of every agent (`~/.claude/agents/`) and skill (`~/.claude/skills/`) in this environment, kept **one file per unit** under this folder. This is the canonical copy to update when a definition changes, and the source of truth for deploying them on a new machine.

Read this only when creating, editing, or recreating an agent or skill. It is not referenced from `~/.claude/CLAUDE.md` and is intentionally kept out of Claude's default context (the whole `_meta/` folder is a single `claudeMdExcludes` entry, so these per-unit files never load into a session) — the setup flow in `README.md` points here.

## Layout

Each unit's verbatim content lives in its own file — edit the matching file, not this index:

- `_meta/agents/<name>.md` — one per agent (verbatim agent definition)
- `_meta/skills/ahead-<name>.md` — one per skill (verbatim `SKILL.md` body)
- `_meta/templates/<name>.html` — template assets that some skills ship

When a live definition changes (`~/.claude/agents/` or `~/.claude/skills/ahead:*/`), update **only the matching file here** with its exact current content, per the maintenance rule in `think-ahead.md`. This index changes only when a unit is **added or removed**.

## Mapping — repo file → deploy target

### Agents

Live in `~/.claude/agents/` — **not** in this repo, since any `.md` inside the repo would be loaded as instructions.

| Repo file | Deploy target |
|-----------|---------------|
| `agents/observer.md` | `~/.claude/agents/observer.md` |
| `agents/impact-check.md` | `~/.claude/agents/impact-check.md` |
| `agents/chaos-agent.md` | `~/.claude/agents/chaos-agent.md` |

### Skills

Each skill is a folder `~/.claude/skills/ahead:<name>/` containing a `SKILL.md`. The repo file `ahead-<name>.md` deploys to that folder's `SKILL.md`.

| Repo file | Deploy target |
|-----------|---------------|
| `skills/ahead-next-steps.md` | `~/.claude/skills/ahead:next-steps/SKILL.md` |
| `skills/ahead-specs.md` | `~/.claude/skills/ahead:specs/SKILL.md` |
| `skills/ahead-instructions.md` | `~/.claude/skills/ahead:instructions/SKILL.md` |
| `skills/ahead-mr.md` | `~/.claude/skills/ahead:mr/SKILL.md` |
| `skills/ahead-handoff.md` | `~/.claude/skills/ahead:handoff/SKILL.md` |
| `skills/ahead-decision.md` | `~/.claude/skills/ahead:decision/SKILL.md` |
| `skills/ahead-recall.md` | `~/.claude/skills/ahead:recall/SKILL.md` |
| `skills/ahead-visualize.md` | `~/.claude/skills/ahead:visualize/SKILL.md` |
| `skills/ahead-resolve.md` | `~/.claude/skills/ahead:resolve/SKILL.md` |
| `skills/ahead-autopilot.md` | `~/.claude/skills/ahead:autopilot/SKILL.md` |
| `skills/ahead-reviewer-report.md` | `~/.claude/skills/ahead:reviewer-report/SKILL.md` |
| `skills/ahead-reference-docs.md` | `~/.claude/skills/ahead:reference-docs/SKILL.md` |

### Templates

Some skills ship an extra asset copied into the skill's folder alongside its `SKILL.md`.

| Repo file | Deploy target |
|-----------|---------------|
| `templates/visualize-template.html` | `~/.claude/skills/ahead:visualize/template.html` |
| `templates/reviewer-report-template.html` | `~/.claude/skills/ahead:reviewer-report/reviewer-report-template.html` |

## Deploy on a new machine

Run from the repo root (`~/.claude/rules`):

```bash
# Agents
mkdir -p ~/.claude/agents
cp _meta/agents/*.md ~/.claude/agents/

# Skills — repo file ahead-<name>.md -> ~/.claude/skills/ahead:<name>/SKILL.md
for f in _meta/skills/ahead-*.md; do
  name=$(basename "$f" .md); name=${name#ahead-}
  mkdir -p "$HOME/.claude/skills/ahead:${name}"
  cp "$f" "$HOME/.claude/skills/ahead:${name}/SKILL.md"
done

# Skill templates
cp _meta/templates/visualize-template.html       "$HOME/.claude/skills/ahead:visualize/template.html"
cp _meta/templates/reviewer-report-template.html "$HOME/.claude/skills/ahead:reviewer-report/reviewer-report-template.html"
```

Verify fidelity after deploy: `diff -rq _meta/agents ~/.claude/agents` and, per skill, `diff _meta/skills/ahead-<name>.md ~/.claude/skills/ahead:<name>/SKILL.md`.

## Adding or removing a unit

- **New agent/skill** — create its live file, add a verbatim copy here (`agents/` or `skills/ahead-<name>.md`), and add a row to the matching table above.
- **Removed unit** — delete its file here and its table row.

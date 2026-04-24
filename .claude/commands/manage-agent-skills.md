# Manage Agent Skills

Create or update skills in `.agents/skills/` so they're available to all agents. Codex finds
them natively; Claude Code invokes them as `/skill-name` slash commands via `.claude/commands/`.

## How the system works

```
.agents/skills/<name>/SKILL.md  ← canonical source; Codex finds this natively at $REPO_ROOT/.agents/skills/
                                    Gemini and any agent that reads the repo can also read it directly

.claude/commands/<name>.md      ← Claude Code slash command; same content as SKILL.md
                                    but without the frontmatter block; committed to git
```

No symlinks. No install scripts. Both files are committed to the repo and work everywhere.

## When to use

- **CREATE**: Encoding a new reusable workflow as a skill
- **UPDATE**: Editing an existing skill's content, description, or metadata

---

## Creating a new skill

### Step 1 — Choose a name

Use kebab-case. Keep it short and action-oriented: `ci-loop`, `deploy-prod`, `db-migrate`.

### Step 2 — Create `.agents/skills/<name>/SKILL.md`

```
.agents/skills/<name>/SKILL.md
```

`SKILL.md` must open with this frontmatter block:

```markdown
---
name: <name>
description: <one sentence — what the skill router reads to decide when to invoke>
risk: low | medium | high | critical
source: local
---
```

**Risk guidelines:**

| Level | When to use |
|---|---|
| `low` | Read-only, no side effects |
| `medium` | Writes files, runs tests |
| `high` | Commits, pushes, modifies config |
| `critical` | Deploys, touches production, irreversible actions |

### Step 3 — Write the skill body

Recommended structure:

1. **Goal** — one sentence
2. **Preconditions** — what must be true before starting
3. **Steps** — numbered phases with exact commands
4. **Stop conditions** — what "done" looks like
5. **Guardrails** — what NOT to do
6. **Project context** — repo-specific commands, paths, identifiers (keep at the bottom
   so the generic logic stays reusable)

### Step 4 — Create `.claude/commands/<name>.md`

Copy the skill body (everything after the frontmatter) into a new file:

```
.claude/commands/<name>.md
```

The commands file does **not** include the `---` frontmatter block — just the markdown body.
This file is what Claude Code loads when you type `/<name>`.

### Step 5 — Register in the README

Add a row to the skills table in `.agents/skills/README.md`:

```markdown
| Skill Name | [`<name>/`](<name>/) | `TRIGGER` — one-line description |
```

### Step 6 — Note in AGENTS.md (optional)

Codex finds skills natively — no AGENTS.md entry is required. If you want to preserve a
legacy trigger phrase for non-Codex agents, add a one-liner under `### Custom Agent Commands`:

```markdown
## TRIGGER_PHRASE
Full definition: `.agents/skills/<name>/SKILL.md`
```

---

## Updating an existing skill

1. Edit `.agents/skills/<name>/SKILL.md` (canonical source).
2. Mirror the same changes in `.claude/commands/<name>.md` (keep them in sync).
3. If `description` changed, update the README table row to match.
4. If the skill was **renamed**:
   - Use `git mv` to preserve file history:
     ```bash
     git mv .agents/skills/<old-name> .agents/skills/<new-name>
     git mv .claude/commands/<old-name>.md .claude/commands/<new-name>.md
     ```
   - Update the `name` field in the frontmatter of the renamed `SKILL.md`.
   - Update the README table row to the new name and path.

---

## Completion checklist

- [ ] Directory exists at `.agents/skills/<name>/` (kebab-case)
- [ ] `SKILL.md` has valid frontmatter: `name`, `description`, `risk`, `source: local`
- [ ] `.claude/commands/<name>.md` created with skill body (no frontmatter)
- [ ] Row added/updated in `.agents/skills/README.md` skills table

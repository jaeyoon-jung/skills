# skills

A collection of useful skills for [Claude Code](https://claude.com/claude-code).

## Skills

- **[changelog](changelog/SKILL.md)** — Maintain `CHANGELOG.md` with date-grouped flat bullets.
- **[ship](ship/SKILL.md)** — Commit current working changes and open a GitHub PR.
- **[triage](triage/SKILL.md)** — File GitHub issues for unresolved code review findings.

## Install

Symlink (or copy) any skill directory into `~/.claude/skills/`:

```sh
ln -s "$PWD/changelog" ~/.claude/skills/changelog
ln -s "$PWD/ship" ~/.claude/skills/ship
ln -s "$PWD/triage" ~/.claude/skills/triage
```

Then invoke with `/changelog`, `/ship`, or `/triage` in Claude Code.

## Keep skills version-controlled

Symlinking (rather than copying) makes this repo the single source of truth — but only if every edit and every new skill ends up committed here. Add the following to `~/.claude/CLAUDE.md` so any Claude Code session, in any directory, knows the workflow:

```markdown
## Skills

Skills under `~/.claude/skills/` are symlinks into `/Users/<you>/code/skills`, which is the source of truth.

- **Editing an existing skill**: edit the file (the symlink resolves to the repo), then `cd` into the repo and commit so the change isn't lost.
- **Creating a new skill**: first check whether `~/.claude/skills/<name>` already exists.
  - If it's a symlink into `/Users/<you>/code/skills/<name>` → the skill already exists in the repo; edit it instead of creating a new one.
  - If it's a real directory → an unversioned copy exists. Diff it against `/Users/<you>/code/skills/<name>` (if present); reconcile with the user before clobbering. Once reconciled, move the canonical version into the repo and replace `~/.claude/skills/<name>` with a symlink.
  - If nothing exists at that path → create `SKILL.md` under `/Users/<you>/code/skills/<name>/`, then `ln -s /Users/<you>/code/skills/<name> ~/.claude/skills/<name>`.

  Always commit the new or reconciled skill in the repo.
```

`~/.claude/CLAUDE.md` is loaded into every Claude Code session as user-level memory, so the instructions apply regardless of which project directory you're in.

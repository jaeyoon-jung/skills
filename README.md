# Skills

A collection of reusable agent skills for [Claude Code](https://code.claude.com/docs/en/slash-commands) and [Codex](https://developers.openai.com/codex/skills).

## Skills

- **[changelog](changelog/SKILL.md)** — Maintain `CHANGELOG.md` with date-grouped flat bullets.
- **[implement](implement/SKILL.md)** — Execute approved feature tasks in verified, atomic slices.
- **[next-feature](next-feature/SKILL.md)** — Run the next roadmap item through specification, implementation, and shipping.
- **[project-charter](project-charter/SKILL.md)** — Draft or update `constitution.md` (≤1000 words) and `tech-stack.md` with iterative clarifying questions.
- **[roadmap](roadmap/SKILL.md)** — Draft or maintain a forward-looking `ROADMAP.md`.
- **[ship](ship/SKILL.md)** — Commit current working changes, open a GitHub PR, request a cross-agent review, and watch the PR.
- **[spec-repair](spec-repair/SKILL.md)** — Audit and repair spec, plan, and task artifacts before phase handoff.
- **[specify](specify/SKILL.md)** — Turn a feature into approved specification, plan, and task artifacts.
- **[triage](triage/SKILL.md)** — File GitHub issues for unresolved code review findings.

## Install

Keep this repository as the source of truth and symlink any skill into one or both host-specific discovery directories.

### Claude Code

```sh
repo_root="$(git rev-parse --show-toplevel)"
mkdir -p "$HOME/.claude/skills"
for skill in changelog implement next-feature project-charter roadmap ship spec-repair specify triage; do
  target="$HOME/.claude/skills/$skill"
  if [ ! -e "$target" ] && [ ! -L "$target" ]; then
    ln -s "$repo_root/$skill" "$target"
  fi
done
```

Invoke explicitly with `/changelog`, `/ship`, `/specify`, and so on. Claude Code can also select a skill implicitly from its description.

### Codex

```sh
repo_root="$(git rev-parse --show-toplevel)"
mkdir -p "$HOME/.agents/skills"
for skill in changelog implement next-feature project-charter roadmap ship spec-repair specify triage; do
  target="$HOME/.agents/skills/$skill"
  if [ ! -e "$target" ] && [ ! -L "$target" ]; then
    ln -s "$repo_root/$skill" "$target"
  fi
done
```

Invoke explicitly by mentioning `$changelog`, `$ship`, `$specify`, and so on. Codex can also select a skill implicitly from its description.

## Keep skills version-controlled

Symlinking rather than copying makes this repository the single source of truth. Put this guidance in your host's global instruction file—`~/.claude/CLAUDE.md` for Claude Code or `~/.codex/AGENTS.md` for Codex—if you want it to apply in every project. In the snippet, `<skills-repo>` means the absolute path returned by `git rev-parse --show-toplevel` from your local checkout:

```markdown
## Skills

Personal skills are symlinks into `<skills-repo>`, the local checkout that is the source of truth.

- **Editing an existing skill**: edit the file (the symlink resolves to the repo), then `cd` into the repo and commit so the change isn't lost.
- **Creating a new skill**: first check whether the host's personal skills directory already contains `<name>`.
  - If it's a symlink into `<skills-repo>/<name>` → the skill already exists in the repo; edit it instead of creating a new one.
  - If it's a real directory → an unversioned copy exists. Diff it against `<skills-repo>/<name>` (if present); reconcile with the user before clobbering. Once reconciled, move the canonical version into the repo and replace the installed copy with a symlink.
  - If nothing exists at that path → create `SKILL.md` under `<skills-repo>/<name>/`, then symlink that directory into the host's personal skills directory.

  Always commit the new or reconciled skill in the repo.
```

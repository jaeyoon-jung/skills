# skills

A collection of useful skills for [Claude Code](https://claude.com/claude-code).

## Skills

- **[ship](ship/SKILL.md)** — Commit current working changes and open a GitHub PR.
- **[triage](triage/SKILL.md)** — File GitHub issues for unresolved code review findings.

## Install

Symlink (or copy) any skill directory into `~/.claude/skills/`:

```sh
ln -s "$PWD/ship" ~/.claude/skills/ship
ln -s "$PWD/triage" ~/.claude/skills/triage
```

Then invoke with `/ship` or `/triage` in Claude Code.

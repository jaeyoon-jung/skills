---
name: changelog
description: Maintain CHANGELOG.md in the project root with date-grouped flat bullets. Two modes — backfill from git history if no changelog exists, otherwise append entries under today's date for work done since the last update. Use when the user says "update the changelog", "add to the changelog", "regenerate the changelog", "/changelog", or otherwise signals they want the CHANGELOG.md refreshed (typically right before merging a branch).
tools: Bash, Read, Edit, Write
---

# /changelog — maintain CHANGELOG.md

User-invoked, runs right before a merge. Decide which mode you're in by checking whether `CHANGELOG.md` exists at the repo root, then do the smallest correct write.

## 1. Detect the repo root and check for an existing changelog

Run in parallel:

- `git rev-parse --show-toplevel` — anchor every path off this
- `git rev-parse --abbrev-ref HEAD` — current branch
- `ls <root>/CHANGELOG.md` — does it exist?
- Today's date in `YYYY-MM-DD` (use the date already in your environment context, not `date` from the shell — environment date is authoritative)

If you are not in a git repo, stop and tell the user this skill needs a git repo.

## 2. Pick a mode

- **Backfill mode** — `CHANGELOG.md` does not exist. Walk all of `main`'s history and write the full file.
- **Update mode** — `CHANGELOG.md` exists. Add bullets for work done since the most recent date heading in the file.

## 3. Backfill mode

Goal: a single `CHANGELOG.md` summarizing the project's history, grouped by date, most recent date first.

1. Read every commit on `main`:

   ```
   git log main --reverse --date=short --pretty=format:'%ad%x09%h%x09%s%x09%an'
   ```

   `--reverse` makes it easier to dedupe and group chronologically; you'll flip the order when writing.

2. Group commits by their author-date (`YYYY-MM-DD`).

3. For each commit, decide whether it belongs in the changelog and how to phrase it:

   - **Skip** merge commits (`Merge pull request`, `Merge branch`), dependency bumps with no behavior change, formatting-only commits, and chore commits that don't change product behavior.
   - **Rewrite** the subject into a reader-facing bullet — past tense, focused on the user-visible change or the load-bearing internal change. Don't just copy the commit subject. Drop conventional-commit prefixes (`feat:`, `fix:`, etc.).
   - **Collapse** multiple commits that describe the same change (e.g., "WIP", "fix typo", "address review") into one bullet describing the final state.

4. Write `CHANGELOG.md` with this structure:

   ```markdown
   # Changelog

   ## YYYY-MM-DD

   - Reader-facing bullet
   - Another bullet

   ## YYYY-MM-DD

   - …
   ```

   - Most recent date at the top.
   - One blank line between sections.
   - No subheadings inside a date (flat bullets only — this is intentional).
   - No "Unreleased" section; dates only.

5. Show the user the file you wrote and stop. Do not commit — the user will commit before merging.

## 4. Update mode

Goal: add bullets for work that has happened since the changelog was last updated, under today's date heading.

1. Find the most recent date heading already in `CHANGELOG.md`. Call it `LAST_DATE`.

2. Collect candidate changes from two sources:

   - **Committed since `LAST_DATE`**: `git log --since=<LAST_DATE> --date=short --pretty=format:'%ad%x09%h%x09%s%x09%an'` on the current branch.
   - **Uncommitted**: `git status` + `git diff` (staged and unstaged). These are about-to-be-merged changes that the user wants captured now.

   Dedupe across sources — a commit on the current branch covers its own diff; don't list it twice.

3. Apply the same skip/rewrite/collapse rules as backfill (step 3.3 above).

4. Locate today's heading (`## <today>`) in the file:

   - **If it exists**: append the new bullets to the end of that section. Do not reorder existing bullets.
   - **If it does not exist**: insert a new `## <today>` section immediately after the `# Changelog` title (or at the top if no top-level title), above the previous most recent date.

5. Never modify dates, headings, or bullets that already exist. Only add.

6. If there is nothing new worth recording (only skipped categories), tell the user that and do not edit the file.

7. Show the user the diff and stop. Do not commit.

## Writing good bullets

- One sentence, past tense, ≤100 chars when possible.
- Lead with the change, not the file. ✅ "Made the home page responsive at 320px." ❌ "Edited static/style.css."
- Mention user-visible impact when there is one; otherwise describe the architectural change.
- Group tightly related work into one bullet rather than three near-identical bullets.
- Don't include commit hashes, PR numbers, or author names — the heading date is enough context.

## Refuse to

- Commit, push, or merge anything — this skill only writes the file.
- Reorder or rewrite previous changelog entries.
- Backdate entries (use today's date for new work, even if the underlying commit is older — the rule is "added to the changelog on date X," not "committed on date X"). Exception: backfill mode, which uses commit author-dates because there is no prior history to anchor against.
- Invent dates the user has not specified and that don't come from git.

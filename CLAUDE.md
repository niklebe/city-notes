## Agent skills

### Issue tracker

Issues live in GitHub Issues for `niklebe/city-notes` (via the `gh` CLI). See `docs/agents/issue-tracker.md`.

### Triage labels

Default five-role vocabulary: `needs-triage`, `needs-info`, `ready-for-agent`, `ready-for-human`, `wontfix`. See `docs/agents/triage-labels.md`.

### Domain docs

Single-context: one `CONTEXT.md` + `docs/adr/` at the repo root. See `docs/agents/domain.md`.

## Git workflow

- Commit directly to the working branch (`main`); do not open pull requests unless explicitly asked.
- After a task is done, commit the work and clean up: remove stale worktrees and delete stale local and remote branches (including throwaway `research/*` branches).

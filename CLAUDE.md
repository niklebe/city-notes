## Agent skills

### Issue tracker

Issues live in GitHub Issues for `niklebe/city-notes` (via the `gh` CLI). See `docs/agents/issue-tracker.md`.

### Triage labels

Default five-role vocabulary: `needs-triage`, `needs-info`, `ready-for-agent`, `ready-for-human`, `wontfix`. See `docs/agents/triage-labels.md`.

### Domain docs

Single-context: one `CONTEXT.md` + `docs/adr/` at the repo root. See `docs/agents/domain.md`.

## Tool docs

- For every dev tool, library or service you use (Next.js, MapLibre, BullMQ, Coolify, PostGIS, the vision LLM API, etc.), search for its LLM-specific docs (`llms.txt` / `llms-full.txt`, "docs for AI" pages, MCP docs servers) before relying on memory.
- Download what you find into `docs/vendor/<tool>/`, with the source URL and fetch date at the top of each file, and add a pointer to it in the index below.
- Read the local copy first when working with that tool.

### Vendored docs index

<!-- one line per tool: - [<tool>](docs/vendor/<tool>/) — what it covers, version -->

- [Anthropic Claude API](docs/vendor/anthropic/) — vision, structured outputs, multilingual support, models, pricing, errors, rate limits, batch, TypeScript SDK; fetched 2026-10-11 from platform.claude.com (`llms-index.md` lists every other page as a `.md` URL to fetch on demand)

## Git workflow

- Commit directly to the working branch (`main`); do not open pull requests unless explicitly asked.
- After a task is done, commit the work and clean up: remove stale worktrees and delete stale local and remote branches (including throwaway `research/*` branches).

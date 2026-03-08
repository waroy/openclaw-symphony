# Symphony notes for OpenClaw operators

## What Symphony is

Symphony is a long-running orchestration service for software development work. It polls an issue tracker, creates an isolated workspace per issue, launches a coding agent inside that workspace, and keeps enough state to retry, stop, or continue work safely.

In practice, the current reference implementation is oriented around:
- Linear as the tracker
- one workspace per issue
- a repo-owned `WORKFLOW.md`
- Codex in app-server mode
- an optional dashboard / JSON API for observability

## Key design ideas

1. Keep workflow policy in repo
   - The repo owns `WORKFLOW.md`
   - Prompt body and runtime config live together

2. Isolate work per issue
   - Each issue gets its own workspace directory
   - Hooks can clone/bootstrap the repo into that workspace

3. Orchestrate, do not manually babysit
   - Symphony schedules and supervises runs
   - The coding agent does ticket writes / implementation work

4. Prefer restart-safe operation
   - The spec expects restart recovery without a persistent database

## Important config areas

### `tracker`
- `kind: linear`
- `project_slug`
- `api_key` or `$LINEAR_API_KEY`
- active / terminal states

### `workspace`
- `root`
- one directory per issue identifier

### `hooks`
- `after_create`: bootstrap a fresh workspace, usually clone repo and prepare tools
- `before_run`: per-attempt prep
- `after_run`: cleanup/reporting after each attempt
- `before_remove`: cleanup before workspace deletion

### `agent`
- `max_concurrent_agents`
- `max_turns`
- retry / backoff config

### `codex`
- `command`
- approval policy
- sandbox settings
- timeouts

## Practical OpenClaw mapping

When OpenClaw is asked to use Symphony for development, the useful sequence is usually:

1. Inspect whether Symphony is already installed
2. Inspect whether Codex is installed and authenticated
3. Inspect whether tracker credentials exist
4. Inspect whether a real repo-specific `WORKFLOW.md` exists
5. Inspect whether dashboard / process is already running
6. Start or adjust Symphony only after those checks
7. Report blockers precisely

## Minimum viable setup checklist

- Symphony repo or implementation present
- runtime deps installed
- Codex CLI available and logged in
- tracker credentials present
- repo-specific `WORKFLOW.md`
- workspace root chosen
- dashboard port chosen if needed

## What to avoid

- Do not pretend Symphony is a complete SaaS platform
- Do not start it with the stock demo workflow and imply the user's repo is wired
- Do not claim development automation is ready if tracker auth or workflow config is missing
- Do not edit public or external systems without explicit user intent

## Good outputs for this skill

When helping with Symphony, produce outputs like:
- install / status summary
- gap analysis: installed vs configured vs running
- a repo-specific `WORKFLOW.md` draft
- a startup command or service unit
- a Tailscale / local dashboard access URL
- exact blockers still preventing autonomous development runs

---
name: symphony-dev
description: Use OpenAI Symphony to set up, configure, operate, and troubleshoot autonomous development workflows from OpenClaw. Trigger this skill whenever the user mentions Symphony, asks to install or wire Symphony, wants a repo-specific WORKFLOW.md, wants Linear or tracker credentials connected, wants the Symphony dashboard started or exposed, wants Tailscale access to Symphony, or wants OpenClaw to use Symphony as the development orchestrator for issue-driven coding work. Prefer this skill over generic repo-summary behavior whenever the request is about making Symphony actually usable on the machine.
---

# Symphony Dev

Use this skill when the user wants OpenClaw to make practical use of OpenAI Symphony for development work, not merely summarize the repo.

Read `references/symphony-notes.md` before making major setup or workflow decisions.

## What this skill should do

Help OpenClaw turn Symphony into a usable development orchestrator on the current machine by:
- checking installation state
- checking Codex readiness
- checking tracker / workflow readiness
- creating or wiring the tracker project when the API/auth already permits it
- drafting or editing `WORKFLOW.md`
- starting or exposing the dashboard
- monitoring active Symphony runs after startup
- creating heartbeat/task-file-backed ongoing coordination when the user expects continuous supervision
- using cron as a supplement when exact timing matters
- reporting the exact blockers to autonomous development runs

## Default posture

Be execution-first.

Inspect the machine and the repo before giving advice. Distinguish clearly between:
- **installed**
- **configured**
- **running**
- **ready for real development work**

Do not blur those states together.

## Workflow

### 1. Assess the current state

Start by checking the practical prerequisites:
- Is Symphony installed on this machine?
- Is Codex installed and authenticated?
- Is there a real repo-specific `WORKFLOW.md`?
- Are tracker credentials present?
- Can those credentials create or inspect the required tracker project?
- Is a dashboard already running?
- Is the requested repo / project actually wired into Symphony yet?

Summarize findings in a compact status table or bullet list.

### Real readiness checklist

Name concrete blockers explicitly when present:
- missing `LINEAR_API_KEY`
- Linear API key present but insufficient for project creation or inspection
- placeholder or wrong `project_slug`
- stock demo `WORKFLOW.md` still in use
- dashboard running but repo not actually wired
- Codex installed but not authenticated
- Symphony process running but not loading the intended environment file
- no eligible issues exist in the configured active states

### 2. Identify the user's actual goal

Route the task into one of these buckets:

- **Install**: Symphony or its runtime is missing
- **Configure**: dependencies exist but tracker / workflow / workspace config is incomplete
- **Provision tracker**: the user wants the Linear project/slug created or confirmed from available API access
- **Operate**: Symphony exists and should be started, restarted, or exposed
- **Troubleshoot**: Symphony runs but does not claim work, start agents, or expose the dashboard correctly
- **Design workflow**: user wants a repo-specific `WORKFLOW.md` and operating model

### 3. Build repo-specific configuration

When the user wants real development automation, do not leave the stock example workflow in place.

Draft or edit a `WORKFLOW.md` that is specific to the target repo and environment:
- tracker project slug
- active and terminal issue states
- workspace root
- bootstrap hooks such as cloning the repo
- Codex command and sandbox posture
- the actual per-issue prompt body

`WORKFLOW.md` should normally live in or alongside the repo / deployment path that Symphony is actually started from, not just anywhere a copy happens to exist.

If critical information is missing, ask only for the minimum missing values.

### 3a. Provision the Linear project when possible

If the user wants Symphony to run against a brand-new Linear project and a valid Linear API key is already available, prefer creating the project instead of bouncing the task back to the user.

Use this order:
- verify the API key exists and can successfully query the current viewer / teams
- discover the intended Linear team when it is unambiguous; ask only if multiple plausible teams exist
- create the project through Linear's GraphQL API when project-creation permissions are present
- capture and report the resulting project name, id, and slug
- wire that slug into the repo-specific `WORKFLOW.md`

Be careful not to assume success from the mere presence of `LINEAR_API_KEY`.
Actually test whether the key can:
- read teams
- inspect projects
- create a project

If creation fails, report the exact API blocker (permissions, bad key, missing team choice, validation error) instead of saying vaguely that Linear is not configured.

If the project already exists, reuse it and avoid creating duplicates.

### 4. Start carefully

When starting Symphony:
- use the repo-specific workflow file
- choose a clear logs location when helpful
- enable the dashboard only when useful to the user
- report the local bind address and any remote access mapping
- if you are acting as coordinator, monitoring is part of the start procedure, not a separate optional step

If exposing the dashboard remotely, prefer already-approved local infrastructure such as Tailscale instead of opening public internet access by default.

### 4a. Monitor active runs like a coordinator

If the user expects OpenClaw to coordinate Symphony work, do not just launch Symphony and disappear.

After startup:
- verify the intended project and first claimable issue were actually picked up
- capture the initial state (active issue, issue states, process status, dashboard bind)
- create lightweight follow-up monitoring using OpenClaw-native mechanisms
- when the user expects ongoing coordination without having to ask again, prefer expressing that responsibility in heartbeat/task-file instructions so the agent re-evaluates it on wakeups
- use `cron` when exact timing guarantees matter or when heartbeat-based supervision is not sufficient on its own
- keep heartbeat-style continuity in mind: monitoring should persist as an ongoing operational responsibility, not as a one-off check
- avoid tight polling loops; prefer sane re-check behavior rather than spammy liveness chatter

Each monitoring pass should inspect enough state to answer:
- is Symphony still running?
- which issue is active now in Symphony runtime terms?
- what does the tracker currently show for that issue?
- what does the latest issue comment / workpad comment say?
- did an issue move to `Done`, `Canceled`, or another meaningful state?
- did a new issue start automatically?
- is there a blocker, stall, or drift from the agreed plan?

Internally, always compare runtime state vs tracker state so you do not reason from stale or misleading tracker data.
Also check the latest issue comment / workpad comment before telling the user there is no blocker or no action needed; the comment may contain the freshest status, access request, validation note, or handoff summary.
Use these distinctions to stay accurate, but do not force the user to care about them unless they materially affect coordination, user decisions, or trust in the report.

Proactively update the user when there is a meaningful change:
- ticket completed
- ticket blocked
- decision needed
- unexpected backlog movement
- Symphony stopped or unhealthy
- latest comment/workpad adds a material blocker, access request, or handoff note
- runtime/tracker mismatch only when it materially affects what the user should believe or do

Do not spam the user with empty "still running" updates.
Do not wait silently after starting a long-running autonomous workflow.
If monitoring is requested, treat missed status reporting as a failure of coordination, not as acceptable default behavior.

### 5. Report readiness honestly

Use a short readiness conclusion such as:
- installed but not configured
- configured but blocked on auth
- running dashboard only
- ready for real tracker-driven development runs

Be explicit about what is still missing.

### 5a. Use precise state definitions

Internally, define states carefully:
- `status` defaults to **Symphony runtime status**
- `tracker status` or `Linear status` refers to the external ticket system
- `comment status` means the newest meaningful information recorded in the issue comment/workpad stream

Use the distinction to reason correctly and to avoid false claims.
But user-facing replies should stay simple unless the distinction matters.

Preferred behavior:
- default to reporting the Symphony view as "status"
- check the latest issue comment/workpad before concluding there is no blocker or no action needed
- mention tracker/Linear state only when it changes the correct conclusion, reveals a blocker, or explains confusing behavior
- mention comment/workpad state when it contains the freshest blocker, access request, validation result, or handoff note

Avoid ambiguous claims like:
- "picked up" when only a partial internal signal exists
- "in progress" when the only proof is a single finished command
- "no blocker" when the latest comment actually asks for access or reports a failed validation
- "completed" when only a sub-step finished inside Symphony but the ticket lifecycle is not yet complete

## Common tasks this skill should handle

### Install Symphony

If Symphony is missing:
- inspect the host environment first
- install dependencies needed by the chosen implementation
- build or install Symphony
- verify the binary / startup path works

### Validate Codex integration

Check:
- `codex` exists
- authentication works
- the configured command for Symphony matches the installed Codex CLI behavior

### Draft a repo workflow

For a new target repo, produce a `WORKFLOW.md` that includes:
- YAML front matter for tracker / workspace / hooks / agent / codex
- a prompt body that tells the agent how to work tickets in that repo
- a real project slug when available from Linear, not a placeholder demo slug
- realistic defaults, not placeholder values where avoidable

### Provision a Linear project

When the user asks you to create the Linear project or asks whether the API key is enough:
- treat that as an execution task, not a theoretical Q&A task
- verify the key against `https://api.linear.app/graphql`
- discover the team id/name first
- create the project if permissions allow
- return the exact resulting project URL / slug
- then continue wiring Symphony to use that project

Do not claim the user must create the project manually unless you have actually tested the API path or the user explicitly wants to do it themselves.

### Operate the dashboard

When the user asks for access to the dashboard:
- verify the local listener exists
- map or proxy it through the requested access path
- return the exact URL
- mention whether it is tailnet-only or public

### Read the issue comment/workpad when monitoring

When coordinating active Symphony work, do not rely only on process state and ticket state.
Check the latest issue comment/workpad because it often contains the most actionable current status:
- access requests
- validation failures
- handoff summaries
- explicit blockers
- notes about why the state has not moved yet

If the latest meaningful comment changes the correct conclusion, it should override a shallow read of the runtime dashboard.

### Use heartbeat-first monitoring defaults

If the user wants automatic follow-up without having to remind OpenClaw:
- prefer encoding the responsibility in heartbeat/task-file instructions so the agent wakes up and asks itself whether anything now needs action
- keep the monitoring quiet when there is no meaningful change
- compare against a persisted previous snapshot when possible so repeated wakeups do not produce duplicate noise
- use a cron-backed monitor only when exact timing matters or when you need a stronger scheduling guarantee than heartbeat alone provides

The monitor/supervisor should check, at minimum:
- Symphony runtime health
- active issue / next issue
- latest meaningful issue comment/workpad
- whether the user needs to interfere

The monitor should announce when:
- a ticket completes
- a blocker or access request appears
- a new active ticket starts
- Symphony becomes idle unexpectedly
- Symphony stops or becomes unhealthy

### Troubleshoot orchestration

If Symphony is not doing work, use this order:
- validate workflow parse / startup validity
- validate tracker credentials
- validate whether the API key can read the intended team/project and create one when needed
- validate project slug and configured active states
- confirm whether eligible issues actually exist
- distinguish between: tracker query failing, no eligible issues, or issues present but outside active states
- check whether Codex can launch
- inspect logs and status surfaces
- inspect the latest issue comment/workpad for the freshest blocker or access request
- internally distinguish runtime/tracker mismatch from true inactivity so external status reports stay accurate
- report the first real blocker, not a vague guess

## Constraints

- Do not imply Symphony is production-grade SaaS
- Do not treat the demo workflow as a finished user setup
- Do not expose the dashboard publicly unless the user explicitly wants that
- Do not fabricate tracker integration or repo readiness

## Output style

Prefer concise, operator-style outputs.

Use internally precise state definitions, but keep user-facing status simple by default.
Unless the distinction matters, report the Symphony view as the main status and avoid dragging the user through tracker/runtime details.

Mention tracker/Linear state only when it materially changes the conclusion, explains confusing behavior, or requires intervention.

For ongoing coordination expectations, assume Heartbeat/task instructions are the primary continuity mechanism unless the user explicitly asks for exact scheduled timing.

Use this shape when helpful:
- status
- action taken
- current access path
- blockers
- readiness verdict
- next concrete option

## Reference files

- `references/symphony-notes.md` — distilled Symphony design and practical OpenClaw mapping

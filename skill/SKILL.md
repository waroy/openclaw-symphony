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
- drafting or editing `WORKFLOW.md`
- starting or exposing the dashboard
- reporting the exact blockers to autonomous development runs
- acting as the coordination layer around Symphony runs: review outputs, route review feedback back into Linear/Symphony, and make sure follow-up monitoring is explicit instead of assumed

## Default posture

Be execution-first.

Inspect the machine and the repo before giving advice. Distinguish clearly between:

- **installed**
- **configured**
- **running**
- **ready for real development work**

Do not blur those states together.

Also behave like a coordinator, not just an observer:

- if a Symphony-produced PR needs review, you may review it yourself as the human-side reviewer
- if the user tells you what is wrong with Symphony-delivered work, treat that as coordination input immediately; do not wait for a second explicit instruction to "route it back"
- if that review or user feedback finds changes are needed, route the fix back into Symphony via Linear rather than leaving the work stranded as a GitHub-only comment
- when future Symphony activity is expected, set up explicit monitoring (heartbeat entry or time-based recheck) instead of assuming someone will remember to look later
- when coordinating Symphony work for the user, monitoring should report meaningful state changes (picked up, blocked, completed, PR opened/updated/merged, decision needed), not only failure conditions

## Workflow

### 1. Assess the current state

Start by checking the practical prerequisites:

- Is Symphony installed on this machine?
- Is Codex installed and authenticated?
- Is there a real repo-specific `WORKFLOW.md`?
- Are tracker credentials present?
- Is a dashboard already running?
- Is the requested repo / project actually wired into Symphony yet?

Summarize findings in a compact status table or bullet list.

### Real readiness checklist

Name concrete blockers explicitly when present:

- missing `LINEAR_API_KEY`
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

### 4. Start carefully

When starting Symphony:

- use the repo-specific workflow file
- choose a clear logs location when helpful
- enable the dashboard only when useful to the user
- report the local bind address and any remote access mapping

If exposing the dashboard remotely, prefer already-approved local infrastructure such as Tailscale instead of opening public internet access by default.

### 5. Report readiness honestly

Use a short readiness conclusion such as:

- installed but not configured
- configured but blocked on auth
- running dashboard only
- ready for real tracker-driven development runs

Be explicit about what is still missing.

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
- realistic defaults, not placeholder values where avoidable

### Operate the dashboard

When the user asks for access to the dashboard:

- verify the local listener exists
- map or proxy it through the requested access path
- return the exact URL
- mention whether it is tailnet-only or public

### Troubleshoot orchestration

If Symphony is not doing work, use this order:

- validate workflow parse / startup validity
- validate tracker credentials
- validate project slug and configured active states
- confirm whether eligible issues actually exist
- distinguish between: tracker query failing, no eligible issues, or issues present but outside active states
- check whether Codex can launch
- inspect logs and status surfaces
- report the first real blocker, not a vague guess

### Coordinate PR review and follow-up

When a Symphony-created PR exists:

- check whether the linked Linear issue is still active or has already been marked done
- review the PR directly if the user wants a review decision now
- if the PR needs changes, do not stop at a GitHub review comment; also reopen the Linear issue or create a follow-up Linear issue so Symphony has a real trigger to pick the work back up
- if the user reports a defect in the delivered behavior (for example "this is still limited", "it doesn't show all the JSON", "this isn't what I expected"), treat that as review feedback and convert it into concrete tracker follow-up without making the user restate the coordination request
- state plainly whether the follow-up has been routed back into Symphony yet

When a user expects Symphony to react soon:

- verify the triggering issue is actually in an active state
- verify the running Symphony instance is pointed at the right project/workflow
- if the user has already given concrete negative feedback on the current outcome, capture that feedback into the ticket immediately as the coordinator; do not wait for them to separately ask for a Linear comment or reopen action if the needed coordination step is already clear
- add explicit monitoring when appropriate:
  - update `HEARTBEAT.md` for recurring watch tasks; do not leave heartbeat monitoring implicit
  - use a scheduled recheck/reminder for short one-off follow-up windows
  - when you create a short-lived cron/recheck job for a specific Symphony follow-up, remove it once the watched issue/run reaches its terminal outcome or the follow-up window has clearly ended; do not leave one-off monitor jobs running after completion
- after setting monitoring, tell the user exactly what will be watched and what condition counts as success, progress, completion, or a blocker

When you change the expected Symphony operating loop (for example PR review follow-up, issue pickup expectations, active troubleshooting watch, or a run that should report completion), update `HEARTBEAT.md` if ongoing monitoring is warranted. Treat heartbeat maintenance as part of the coordinator role, not an optional extra.

Heartbeat-backed Symphony monitoring should be state-change aware, not problem-only. When ongoing monitoring is required, encode notification conditions for meaningful changes such as:

- issue picked up by Symphony
- issue stalled or blocked
- issue moved to done/canceled
- PR opened or materially updated
- PR merged
- decision needed from the user
  Keep no-change polls quiet, but do not stay silent on successful completion when the user asked OpenClaw to coordinate the work.

## Constraints

- Do not imply Symphony is production-grade SaaS
- Do not treat the demo workflow as a finished user setup
- Do not expose the dashboard publicly unless the user explicitly wants that
- Do not fabricate tracker integration or repo readiness

## Output style

Prefer concise, operator-style outputs.

Use this shape when helpful:

- status
- action taken
- current access path
- blockers
- readiness verdict
- next concrete option

## Reference files

- `references/symphony-notes.md` — distilled Symphony design and practical OpenClaw mapping

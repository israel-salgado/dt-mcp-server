# Dynatrace AI Workspace — Session Briefing

## Governing Reference for Tenant Interaction (Agent-Agnostic)

**The single governing reference file for GitHub Copilot is `.github/copilot-instructions.md` (auto-loaded at session start). For Claude it is `CLAUDE.md`. Both are kept in sync.**

This file (and its counterpart) defines **exactly** how any agent interacts with a Dynatrace tenant:
- Default/fallback MCP servers (the live bridge to the tenant via the Model Context Protocol).
- How to switch tenants/contexts when changing agents.
- Global rule, available prompts, skills, notebook guardrails, and agent-agnostic DQL rules.

**Baseline tenant.** `guu84124` (production, public URL `demo.live.dynatrace.com`) is the only tenant ID referenced in this repo's source files — it is publicly reachable and safe to demo. The local nickname registry may seed it as `demo.live` (matching the public URL), but this is purely a convenience; the user may rename it or skip it entirely.

**Session start.** The agent runs `dtctl config current-context` and reports the active context in one line (e.g. `Active dtctl context: <NICKNAME> · <TENANTID> · <class> · <safety>`). dtctl's active context is **persistent on disk** — it carries over between VS Code sessions — so the agent does **not** auto-switch. There is no "default tenant."

**Switch context** (preferred → nickname; fallback → raw tenant ID):
```
"switch to <NICKNAME>"        # resolved via temp_dtctl_files/tenant-memory/tenants.json
"switch to <TENANTID>"        # raw 8-char ID always works
```
The agent always echoes a one-line confirmation (`Switching context → <NICKNAME> · <TENANTID> · <class> · <safety>`) before running `dtctl config use-context`. Ambiguous or fuzzy names are never auto-resolved.

**Connecting to a brand-new tenant via dtctl** (no `.mcp.json` tenant routing required for dtctl-only access): see `CONVENTIONS.md` → *Connecting to a New Tenant* for the full procedure (prompt user for URL + safety level, run `dtctl auth login`, verify). Quick reference also in `CHEATSHEET.md` → *Session Management*.

**Local tenant nickname registry**: when the user says *"switch to <NICKNAME>"* using a short name, resolve it via `temp_dtctl_files/tenant-memory/tenants.json` per `CONVENTIONS.md` → *Local Tenant Nickname Registry*. Never auto-resolve ambiguous or fuzzy matches — always ask. The registry is local-only and never committed.

**Clickable options for short choices.** Use `vscode_askQuestions` for any short fork-in-the-road (2–6 options); leave freeform input on. Use plain text for explanations and multi-paragraph recommendations. (See `CONVENTIONS.md` → *Agent Behavior*.)

**Mandatory agent initialization sequence** (review files first, then run/validate):
1. Read this file + `copilot-instructions.md` + `CONVENTIONS.md` + `ARCHITECTURE.md`.
2. **ALWAYS load `.agents/skills/dt-dql-essentials/SKILL.md` FIRST** (before any DQL).
3. Review **all** relevant workspace files (`current-notebook.json`, `temp_dtctl_files/**`, `clean-dashboard.json`, skills).
4. For dtctl/MCP tenant context: Run `dtctl config current-context`, `dtctl auth whoami --plain`, and/or MCP `get_environment_info` / `find_entity_by_name`.
5. Follow the Global Rule and rules in `CONVENTIONS.md` strictly. No tenant-specific names/IDs in root source files.

See `CONVENTIONS.md` for full Workspace & Temp File Conventions, Live State Reconciliation & Conflict Protection, DQL rules, and Sync Checklist.

This ensures identical, predictable behavior across agent switches.

See `CONVENTIONS.md` for full details on Workspace & Temp File Conventions, Live State Reconciliation & Conflict Protection, DQL rules, Sync Checklist, and agent behavior.

## Environment

| | |
|---|---|
| **Baseline tenant routing** | `demo.live` → https://guu84124.apps.dynatrace.com (public, only tenant ID in repo source) |

This workspace runs one local MCP server launched from `.mcp.json`; entries there are **tenant routings** of that one server, not additional servers. Add more tenants per machine via `dtctl auth login` (and optionally a matching tenant routing in `.mcp.json`). See `CONVENTIONS.md` → *Connecting to a New Tenant* and *Local Tenant Nickname Registry*.

## Always-On Behaviors

These rules apply every turn, regardless of topic. Topic-specific rules (DQL syntax, notebook structure, etc.) live in their respective sections below.

- **Echo every dtctl context switch.** One-line confirmation (`Switching context → <NICKNAME> · <TENANTID> · <class> · <safety>`) before running `dtctl config use-context`. Never auto-resolve ambiguous or fuzzy tenant nicknames — always ask.
- **Clickable options for short choices.** Use `vscode_askQuestions` for any 2–6-option fork-in-the-road; leave freeform input on. Use plain text for explanations and multi-paragraph recommendations.
- **File-system boundaries.** Default scope is the workspace folder. Reads outside the workspace require a stated plain-language reason first; writes outside the workspace require explicit user permission. Subagents inherit this rule. Full details in `CONVENTIONS.md` → *File-System Boundaries*.
- **Live-state reconciliation before any modification.** Before any `dtctl apply`, MCP update, or write to a Dynatrace resource (notebook, dashboard, workflow, settings), re-export that resource's live state by ID first. Smart-merge unrelated user UI edits; stop and ask only on conflicting overwrites (options: stop / let AI overwrite / do something else). Keep a timestamped before-user-edit snapshot for revert.
- **Always start with problems — never broad log searches.** See `## Global Rule` below for full text.
- **Load `.agents/skills/dt-dql-essentials/SKILL.md` before any DQL.** Including dashboards and notebooks. Non-negotiable.
- **Run `scripts/validate-tenant-write.ps1` before any tenant write.** Targeted at the single resource being modified.
- **Keep root source files generic.** No tenant-specific names or IDs in root source files. Tenant artifacts live only in `temp_<type>_files/`.

## Global Rule

**Always start with problems — never run broad log searches.**
Broad queries without problem context hit Dynatrace's 500GB scan limit and return zero results.
All investigation workflows enforce this automatically.

## Prompts

Type `/` in Copilot Chat to access these slash commands:

| Prompt | When to use |
|---|---|
| `/health-check` | Routine service health — metrics, problems, deployments, vulnerabilities |
| `/daily-standup` | Morning report across services — today vs yesterday comparison |
| `/daily-standup-notebook` | Standup report + Dynatrace notebook creation + dtctl verification |
| `/investigate-error` | Error-focused investigation from a service name |
| `/troubleshoot-problem` | Deep 7-step investigation into a specific Dynatrace problem |
| `/incident-response` | Full triage of all active problems during a live incident |
| `/performance-regression` | Before vs after deployment comparison with rollback/hotfix recommendation |

## Skills

17 domain knowledge skills are installed in `.agents/skills/`. They load automatically when relevant — no manual loading required.

## Notebook (and App) Update Contract

This workspace follows a per-app smart reconciliation contract (full details in `CONVENTIONS.md`):

- Use per-app folders (`temp_<type>_files/`) with `current-<type>.json` and index. Auto-create for new types (e.g. business_flow).
- Target **only the specific app** being modified. Refresh current reference when starting work on a type.
- On user UI edits: give 1-2 sentence summary. Smart-merge unrelated changes into local JSON. Stop and ask (with options: stop/let AI overwrite/do something else) only on conflicting overwrites.
- Keep timestamped before-user-edit snapshot for revert.
- Prefer JSON payloads, ID-based operations, explicit DQL metadata, re-export + verify after apply.

**File-System Boundaries**: Default scope for all file operations is the **workspace folder** (wherever the user installed it). Reads outside the workspace are allowed when there is a clear, legitimate reason — but the agent must state the reason in plain language first so the user can approve or deny. Writes outside the workspace always require explicit user permission and a stated reason. When in doubt, copy needed material into `temp_dtctl_files/` and work locally. Subagents inherit this rule. Full details in `CONVENTIONS.md` → *File-System Boundaries*.

**Agent-Agnostic DQL Rules** (apply to ALL agents — see also copilot-instructions.md):
- **ALWAYS load dt-dql-essentials/SKILL.md FIRST**. Review the relevant per-app folder and current reference first.
- Unique `event.type` + provider for isolation. Validate in exact context (dashboard tiles require `fields`/`bin()`/`sort`/`limit` fallback).
- Prefer JSON, start with problems, record generic lessons only. Ensures identical safe behavior. Root remains standardized; per-app temp folders hold context.

Failure mode reminders:
- Duplicate names can point to different ownership.
- Mixed encoding and non-ASCII punctuation can create parser issues.
- Missing `type: dql` can produce empty or non-functional query sections.
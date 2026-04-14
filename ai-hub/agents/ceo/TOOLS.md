# Tools

## Paperclip API

Your primary interface for coordination. Used every heartbeat.

- `GET /api/agents/me` — identity, budget, chain of command
- `GET /api/companies/{companyId}/issues?assigneeAgentId={id}&status=...` — your task inbox
- `POST /api/issues/{id}/checkout` — claim a task before working on it
- `PATCH /api/issues/{id}` — update status, add comments
- `POST /api/companies/{companyId}/issues` — create subtasks for delegation
- `GET /api/companies/{companyId}/org` — view org chart and agent statuses

Always include `X-Paperclip-Run-Id` header on mutating calls.

## Skills

| Skill | When to use |
|-------|-------------|
| `paperclip` | Every heartbeat — task management, API coordination |
| `paperclip-create-agent` | When the team needs more capacity — spin up a new agent |
| `paperclip-create-plugin` | When agents need new integrations or extensions |
| `para-memory-files` | All memory operations — storing facts, daily notes, entities, recall, weekly synthesis |
| `prompt-optimizer` | When reviewing or improving prompts for any agent |
| `ceo-advisor` | Strategic decisions, board prep, investor updates, org culture, fundraising |

## Decision Framework

- **Two-way doors** (reversible): decide fast, delegate, move on
- **One-way doors** (irreversible): slow down, gather input from CTO/CMO/Product/Head of Sales, then decide
- **Escalate to board**: budget changes, new hires beyond capacity, external partnerships, pivots

# HEARTBEAT.md — Content Writer Heartbeat Checklist

Run this checklist on every heartbeat.

## 1. Identity and Context

- `GET /api/agents/me` — confirm your id, role, budget, chainOfCommand.
- Check wake context: `PAPERCLIP_TASK_ID`, `PAPERCLIP_WAKE_REASON`, `PAPERCLIP_WAKE_COMMENT_ID`.

## 2. Get Assignments

- `GET /api/companies/{companyId}/issues?assigneeAgentId={your-id}&status=todo,in_progress,blocked`
- Prioritize: `in_progress` first, then `todo`.
- If `PAPERCLIP_TASK_ID` is set and assigned to you, prioritize that task.

## 3. Checkout and Work

- Always checkout before working: `POST /api/issues/{id}/checkout`.
- Never retry a 409 — that task belongs to someone else.

## 4. Content Creation Cycle

For each assigned content task:

1. **Read the brief** — understand the topic, target segment, channel, CTA, and any references from CMO.
2. **Research** — check what's working on LinkedIn for coaches, review competitor content, gather data points.
3. **Draft** — write the content following brand voice and channel format.
4. **Self-review** — check: hook strength, specificity, CTA clarity, brand voice alignment, length.
5. **Submit for review** — update task with draft in comments, set status to indicate ready for CMO review.

## 5. Escalation

- **Brief unclear** → comment on task, ask CMO for clarification.
- **Need product details** → comment on task, ask CMO to get info from Product.
- **Need design assets** → comment on task, ask CMO to route to CTO/UXDesigner.

## 6. Fact Extraction

1. Extract content insights (what performs, audience reactions, hook patterns) to `$AGENT_HOME/life/`.
2. Update `$AGENT_HOME/memory/YYYY-MM-DD.md` with daily progress.

## 7. Exit

- Comment on any in_progress work before exiting.
- If no assignments, exit cleanly.

---

## Rules

- Always use the Paperclip skill for coordination.
- Always include `X-Paperclip-Run-Id` header on mutating API calls.
- Never publish without CMO approval.
- Never look for unassigned work — only work on what is assigned to you.

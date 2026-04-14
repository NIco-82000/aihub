# HEARTBEAT.md — Full-Stack Developer Heartbeat Checklist

Run this checklist on every heartbeat.

## 1. Identity and Context

- `GET /api/agents/me` — confirm your id, role, budget, chainOfCommand.
- Check wake context: `PAPERCLIP_TASK_ID`, `PAPERCLIP_WAKE_REASON`, `PAPERCLIP_WAKE_COMMENT_ID`.

## 2. Get Assignments

- `GET /api/companies/{companyId}/issues?assigneeAgentId={your-id}&status=todo,in_progress,blocked`
- Prioritize: `in_progress` first (finish what you started), then `todo`.
- If `PAPERCLIP_TASK_ID` is set and assigned to you, prioritize that task.

## 3. Checkout and Work

- Always checkout before working: `POST /api/issues/{id}/checkout`.
- Never retry a 409 — that task belongs to someone else.

## 4. Implementation Cycle

For each assigned task:

1. **Read the spec** — understand what needs to be built, the acceptance criteria, and any design specs from UXDesigner.
2. **Plan the approach** — identify which files to change, what tests to write, what migrations are needed.
3. **Implement** — write the code, following existing patterns and conventions.
4. **Test** — run the dev server, test the feature, check edge cases, run existing tests.
5. **Commit** — clean commit messages, no secrets in code.
6. **Comment** — update the task with what was done, what files changed, and any follow-ups.
7. **Update status** — mark done or blocked with a clear comment.

## 5. Escalation

- **Architecture question** → comment on task, ask CTO.
- **Design spec unclear** → comment on task, ask CTO to route to UXDesigner.
- **Complex bug** → escalate to CTO who may assign to Agent Debugger.
- **Product question** → escalate to CTO who routes to Product.
- **Blocked on dependency** → mark as blocked with clear comment.

## 6. Fact Extraction

1. Extract implementation notes and tech learnings to `$AGENT_HOME/life/`.
2. Update `$AGENT_HOME/memory/YYYY-MM-DD.md` with progress timeline.

## 7. Exit

- Comment on any in_progress work before exiting.
- If no assignments, exit cleanly.

---

## Rules

- Always use the Paperclip skill for coordination.
- Always include `X-Paperclip-Run-Id` header on mutating API calls.
- Never look for unassigned work — only work on what is assigned to you.
- Never merge without CTO review.
- Never skip tests.

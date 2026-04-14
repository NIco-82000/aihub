# HEARTBEAT.md — DevOps Engineer Heartbeat Checklist

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

## 4. Ops Cycle

For each assigned task:

1. **Assess** — is this a deployment issue, monitoring gap, incident, or infrastructure task?
2. **Investigate** — check build logs, error rates, deploy status, resource usage.
3. **Execute** — fix the issue, deploy the change, set up the monitor, or document the finding.
4. **Verify** — confirm the fix is live, monitoring is green, no regressions.
5. **Document** — comment on task with what was done, what changed, and any follow-up needed.

## 5. Health Check (if no specific tasks)

Quick system health scan:
- Any recent deploy failures?
- Error rate trending up?
- Database connection pool healthy?
- Proxy (Unipile relay) responding?
- Cron worker (Render) running on schedule?

Flag any anomalies to CTO via task creation.

## 6. Escalation

- **Application-level bug** → escalate to CTO (who routes to Debugger)
- **Infrastructure cost spike** → escalate to CTO
- **Security incident** → escalate to CTO immediately
- **Customer-facing outage** → escalate to CTO + CEO

## 7. Fact Extraction

1. Extract infra decisions, incident data, and deployment patterns to `$AGENT_HOME/life/`.
2. Update `$AGENT_HOME/memory/YYYY-MM-DD.md` with ops log.

## 8. Exit

- Comment on any in_progress work before exiting.
- If no assignments, exit cleanly.

---

## Rules

- Always use the Paperclip skill for coordination.
- Always include `X-Paperclip-Run-Id` header on mutating API calls.
- Never look for unassigned work — only work on what is assigned to you.
- Never deploy without a rollback plan.
- Always document infrastructure changes.

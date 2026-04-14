# HEARTBEAT.md — Agent Debugger Heartbeat Checklist

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

## 4. Debug Cycle

For each assigned bug/incident:

1. **Read the issue** — understand symptoms, error messages, affected users/flows.
2. **Reproduce** — set up the conditions to trigger the bug locally or in staging.
3. **Investigate** — form hypotheses, test systematically, collect evidence.
4. **Fix** — implement the minimal correct fix.
5. **Validate** — verify the fix resolves the issue and doesn't break adjacent behavior.
6. **Document** — leave a postmortem comment (symptom, root cause, fix, prevention).
7. **Update status** — mark done with a summary comment.

## 5. Escalation

- **Can't reproduce after 3 attempts** → comment with findings, ask CTO for more context.
- **Fix requires architecture change** → escalate to CTO with a proposal.
- **Fix requires design change** → escalate to CTO (who routes to UXDesigner).
- **Blocked on external dependency** → mark as blocked with clear comment.

## 6. Fact Extraction

1. Extract debugging patterns and lessons to `$AGENT_HOME/life/`.
2. Update `$AGENT_HOME/memory/YYYY-MM-DD.md` with investigation timeline.

## 7. Exit

- Comment on any in_progress investigation before exiting.
- If no assignments, exit cleanly.

---

## Rules

- Always use the Paperclip skill for coordination.
- Always include `X-Paperclip-Run-Id` header on mutating API calls.
- Never look for unassigned work — only work on what is assigned to you.
- Never fix bugs by suppressing errors or disabling checks.
- Always leave a postmortem comment on resolved bugs.

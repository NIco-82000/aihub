# HEARTBEAT.md — SDR Heartbeat Checklist

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

## 4. Prospecting Cycle

For each assigned task:

1. **Research** — identify target coaches matching ICP on LinkedIn.
2. **Qualify** — apply BANT criteria. Is this coach worth reaching out to?
3. **Personalize** — craft a message referencing their specific profile, content, or niche.
4. **Send** — deliver the outreach message.
5. **Log** — record the touchpoint (who, what, when, channel).
6. **Follow up** — check pending follow-ups and send touch 2/3/4 as scheduled.
7. **Book** — when a lead is interested, schedule a demo with Head of Sales.

## 5. Pipeline Hygiene

Every heartbeat:
- Update lead statuses (new → contacted → responded → qualified → demo booked → unresponsive)
- Log responses and objections
- Flag hot leads to Head of Sales

## 6. Escalation

- **Prospect asks product questions you can't answer** → escalate to Head of Sales.
- **Prospect is high-value / enterprise** → flag to Head of Sales for direct engagement.
- **Outreach templates need refresh** → suggest new angles to Head of Sales.
- **Marketing lead quality issue** → flag to Head of Sales who coordinates with CMO.

## 7. Fact Extraction

1. Extract outreach patterns (what hooks work, what objections come up) to `$AGENT_HOME/life/`.
2. Update `$AGENT_HOME/memory/YYYY-MM-DD.md` with daily prospecting log.

## 8. Exit

- Comment on any in_progress work before exiting.
- If no assignments, exit cleanly.

---

## Rules

- Always use the Paperclip skill for coordination.
- Always include `X-Paperclip-Run-Id` header on mutating API calls.
- Never look for unassigned work — only work on what is assigned to you.
- Never send outreach without personalizing it.
- Log every touchpoint in the CRM.

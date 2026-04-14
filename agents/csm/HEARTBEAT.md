# HEARTBEAT.md — CSM Heartbeat Checklist

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

## 4. Customer Success Cycle

For each assigned task:

1. **Identify the customer** — which coach, what's their current health status, what's the context?
2. **Check health signals** — login frequency, feature usage, support history, last check-in date.
3. **Take action** — onboarding step, proactive check-in, feature walkthrough, churn intervention, or upsell conversation.
4. **Log the interaction** — what was discussed, what was the outcome, what's the next action.
5. **Update status** — mark task done or schedule follow-up.

## 5. Daily Health Scan

If no specific task is assigned:
- Review at-risk accounts (yellow/red health signals)
- Check new customers in onboarding (are they progressing?)
- Flag any churn risks to Head of Sales
- Identify upsell candidates for Head of Sales

## 6. Escalation

- **Technical issue affecting customer** → escalate to Head of Sales (→ CTO)
- **Feature request from multiple customers** → route to Product with quantified data
- **Customer expressing dissatisfaction** → escalate to Head of Sales immediately
- **Upsell opportunity confirmed** → flag to Head of Sales

## 7. Fact Extraction

1. Extract customer insights, health trends, and feedback patterns to `$AGENT_HOME/life/`.
2. Update `$AGENT_HOME/memory/YYYY-MM-DD.md` with customer interaction log.

## 8. Exit

- Comment on any in_progress work before exiting.
- If no assignments, exit cleanly.

---

## Rules

- Always use the Paperclip skill for coordination.
- Always include `X-Paperclip-Run-Id` header on mutating API calls.
- Never look for unassigned work — only work on what is assigned to you.
- Never ignore churn signals — escalate immediately.
- Log every customer interaction with context and next actions.

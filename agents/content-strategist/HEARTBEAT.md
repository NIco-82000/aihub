# HEARTBEAT.md — Content Strategist Heartbeat Checklist

Run this checklist on every heartbeat.

## 1. Identity and Context

- `GET /api/agents/me` — confirm your id, role, budget, chainOfCommand.
- Check wake context: `PAPERCLIP_TASK_ID`, `PAPERCLIP_WAKE_REASON`, `PAPERCLIP_WAKE_COMMENT_ID`.

## 2. Check Pending Approvals

- `GET /api/companies/{companyId}/approvals?status=pending` — check if any of your engagement requests were approved/rejected.
- For approved engagements: execute the action (like or comment) exactly as submitted.
- For rejected engagements: read `decisionNote`, learn from the feedback.

## 3. Get Assignments

- `GET /api/companies/{companyId}/issues?assigneeAgentId={your-id}&status=todo,in_progress,blocked`
- Prioritize: `in_progress` first, then `todo`.
- If `PAPERCLIP_TASK_ID` is set and assigned to you, prioritize that task.

## 4. Checkout and Work

- Always checkout before working: `POST /api/issues/{id}/checkout`.
- Never retry a 409 — that task belongs to someone else.

## 5. Veille Cycle

If no specific task assigned, run the veille cycle:

### 5a. Search for Recent Posts

Use `linkedin-search` skill to search posts:
- Query 1: `coaching clients LinkedIn` (date: past-week)
- Query 2: `coach professionnel presence en ligne` (date: past-week)
- Query 3: `coaching business developpement` (date: past-week)
- Rotate keywords from your keyword matrix each heartbeat.

### 5b. Evaluate Posts

For each interesting post found:
1. **Engagement level** — likes, comments count (high engagement = trending)
2. **Relevance** — directly about coaching, LinkedIn presence, or client acquisition?
3. **Author** — is this an influencer, a potential customer, or a competitor?
4. **Age** — posts < 24h old have the best engagement window

### 5c. Prepare Engagement Actions

For posts worth engaging:
- **Like**: add to batch approval request
- **Comment**: draft a thoughtful comment, add to batch approval request

Submit batch approval:
```
POST /api/companies/{companyId}/approvals
{
  "type": "linkedin_engagement_batch",
  "payload": {
    "likes": [
      { "postUrl": "...", "author": "...", "reason": "..." }
    ],
    "comments": [
      { "postUrl": "...", "author": "...", "comment": "...", "reason": "..." }
    ]
  }
}
```

### 5d. Generate Content Ideas

From veille findings, create content idea briefs. Submit as task comments for CMO review.

## 6. Fact Extraction

1. Extract veille findings (trending topics, engagement data, keyword performance) to `$AGENT_HOME/life/`.
2. Update `$AGENT_HOME/memory/YYYY-MM-DD.md` with veille log.

## 7. Exit

- Comment on any in_progress work before exiting.
- If no assignments, exit cleanly.

---

## Rules

- Always use the Paperclip skill for coordination.
- Always include `X-Paperclip-Run-Id` header on mutating API calls.
- **NEVER like or comment without board approval.**
- Never look for unassigned work outside veille cycle.
- Log every veille search with keywords, result count, and findings.

---
name: "crm-pipeline"
description: "Read and interact with the SonorIA CRM pipeline. Use when agents need to check prospect status, pipeline data, lead scores, discussions, or customer health. Covers prospect lifecycle, pipeline stages, discussion history, and activity tracking."
slug: "crm-pipeline"
key: "sonoria/crm-pipeline"
---

# CRM Pipeline — SonorIA CRM

Interact with the SonorIA CRM to manage prospects, pipeline, and customer data.

## ⛔ APPROVAL REQUIRED FOR WRITES

**Reading CRM data is FREE — no approval needed.** Query prospects, pipeline, discussions, metrics at will.

**ANY write operation (create, update, delete) requires board approval BEFORE execution.**

Before modifying CRM data, you MUST:

1. Prepare the change (operation, target, specific fields)
2. Submit an approval request:
   ```
   POST /api/companies/{companyId}/approvals
   { "type": "crm_write", "requestedByAgentId": "<your-id>",
     "payload": { "action": "Creer/modifier prospect dans le CRM",
     "operation": "create | update | change_status",
     "prospectName": "<name>",
     "changes": { "status": "...", "tags": [...], "notes": "..." },
     "justification": "<why>" }}
   ```
3. **STOP. Do NOT modify CRM data.** Wait for the next heartbeat.
4. Only execute if `PAPERCLIP_APPROVAL_STATUS === "approved"`.
5. Reference the `approvalId` in your task comment after execution.

See `shared/GUARDRAILS.md` for the full protocol.

---

## Architecture

The CRM is a Next.js app with a PostgreSQL database accessed via Prisma ORM. Data is organized by **workspace** (multi-tenant). The CRM API is at `$CRM_BASE_URL`.

---

## Data Model

### Prospect Lifecycle (Pipeline Stages)

```
TO_CONTACT → CONNECTION_SENT → CONNECTION_ACCEPTED → CONTACTED → FOLLOW_UP → RESPONDED → MEETING → CONVERTED / LOST
```

Additional statuses: `VIEWED`, `LOOM_SENT`, `LOOM_ACCEPTED`

### Key Entities

| Entity | Description |
|--------|-------------|
| **Prospect** | A coach/lead with LinkedIn profile, status, tags, lead score |
| **Discussion** | Conversation thread linked to a prospect (with sentiment tracking) |
| **DiscussionMessage** | Individual messages with sentiment classification |
| **FollowUp** | Scheduled follow-up action for a prospect |
| **Activity** | Logged activities (calls, emails, meetings) |
| **AiTask** | AI-recommended actions (SEND_FOLLOW_UP, etc.) with status tracking |

### Lead Score

100-point hybrid scoring:
- **Engagement** (20pts): profile visits, content interaction
- **Responsiveness** (15pts): reply speed, conversation activity
- **Profile/ICP fit** (25pts): AI-scored match against Ideal Customer Profile
- **Conversation quality** (40pts): AI-scored based on message content and sentiment

Malus system: ghosting, dormance, blocage, concurrent interest, desinteret

### Sentiment Classification

Received messages are auto-classified into 6 categories:
- `POSITIVE` — interested, enthusiastic
- `NEUTRAL` — informational, non-committal
- `NEGATIVE` — rejection, frustration
- `OBJECTION` — price, timing, competitor concerns
- `QUESTION` — asking for more info
- `OUT_OF_SCOPE` — unrelated to coaching/product

### System Tags (auto-applied)

| Tag | Trigger |
|-----|---------|
| `HOT_LEAD` | High lead score + positive sentiment |
| `OBJECTION` | Objection sentiment detected |
| `GHOST` | No response after multiple follow-ups |
| `RDV_REQUEST` | Meeting/call request detected |
| `NEEDS_FOLLOWUP` | Response received, needs action |

---

## How Agents Should Use CRM Data

### SDR
- Check prospect status before outreach (don't contact someone already in MEETING stage)
- Verify lead score and sentiment before qualifying
- Update prospect status after actions (CONNECTION_SENT after invitation, etc.)
- Log activities for all touchpoints

### CSM
- Monitor customer health via lead score trends and sentiment
- Check Discussion.lastSentiment for mood shifts
- Track onboarding progress (prospect status progression)
- Flag at-risk accounts (negative sentiment, ghost pattern)

### Head of Sales
- Pipeline overview: count prospects per stage
- Win/loss analysis: CONVERTED vs LOST ratios
- Deal velocity: average time per stage transition
- Revenue forecasting based on pipeline stages

### Product Manager
- Feature requests from Discussion messages with QUESTION sentiment
- Common objections (OBJECTION sentiment patterns)
- User behavior patterns for roadmap input

---

## Integration Notes

### Webhooks (real-time events)

The CRM receives webhooks from Unipile at `/api/webhooks/unipile` for:
- `message_received` — new LinkedIn message → triggers sentiment classification → auto-tagging → proactive actions → sequence evaluation
- `new_relation` — connection accepted → updates prospect status to CONNECTION_ACCEPTED
- `account_status` — LinkedIn account connection changes
- `CREATION_SUCCESS` — triggers conversation sync + chat remap

### Sequence Engine

Conditional follow-up sequences:
- **Time-driven**: daily cron processes overdue enrollments
- **Event-driven**: webhook triggers evaluate sentiment conditions
- **Guard**: max 1 active enrollment per prospect
- **Steps**: each step has delay, message template, and branching conditions based on sentiment

---

## Safety Rules

- **Always filter by workspaceId** — never query across tenants.
- **Never delete prospect data** without CTO approval.
- **Never modify lead scores manually** — they are computed by the scoring engine.
- **Respect the sequence guard** — never manually enroll a prospect in multiple sequences.
- **Log all CRM interactions** for audit trail.

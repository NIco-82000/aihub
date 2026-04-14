---
name: "linkedin-connect"
description: "Send LinkedIn connection requests via Unipile API. Use when the SDR needs to send invitations to qualified prospects. Includes provider_id resolution, personalized messages, and quota tracking."
slug: "linkedin-connect"
key: "sonoria/linkedin-connect"
---

# LinkedIn Connect — Unipile API

Send LinkedIn connection requests (invitations) through the Unipile proxy.

## ⛔ APPROVAL REQUIRED

**EVERY invitation requires board approval BEFORE sending.** No exceptions.

Before calling ANY endpoint in this skill, you MUST:

1. Prepare the invitation (target, message, qualification score)
2. Submit an approval request:
   ```
   POST /api/companies/{companyId}/approvals
   { "type": "linkedin_invitation", "requestedByAgentId": "<your-id>",
     "payload": { "action": "Envoyer invitation LinkedIn", "target": "<name>",
     "targetUrl": "<linkedin-url>", "message": "<message>",
     "qualificationScore": "<BANT score>", "justification": "<why>" }}
   ```
3. **STOP. Do NOT send the invitation.** Wait for the next heartbeat.
4. Only execute if `PAPERCLIP_APPROVAL_STATUS === "approved"`.
5. Reference the `approvalId` in your task comment after execution.

See `shared/GUARDRAILS.md` for the full protocol.

---

## Setup

```
BASE_URL = $UNIPILE_BASE_URL
API_KEY  = $UNIPILE_API_KEY
ACCOUNT  = $UNIPILE_ACCOUNT_ID
```

## Quota

| Action | Limit | Notes |
|--------|-------|-------|
| invitations | 80/day | Hard limit — LinkedIn may restrict account if exceeded |
| profiles | 100/day | Consumed when resolving provider_id from URL |

**Critical:** LinkedIn is aggressive about invitation abuse. Stay well under 80/day. Recommended: max 30-40/day to be safe.

---

## Step 1: Resolve provider_id

Unipile requires the LinkedIn internal `provider_id`, NOT the profile URL. You must resolve it first.

Extract the slug from the URL, then fetch the profile:

```bash
# Extract slug from URL: https://www.linkedin.com/in/marie-dupont → marie-dupont
SLUG="marie-dupont"

curl -X GET "$UNIPILE_BASE_URL/api/v1/users/$SLUG?account_id=$UNIPILE_ACCOUNT_ID&linkedin_sections=*" \
  -H "X-API-KEY: $UNIPILE_API_KEY"
```

**Response includes:** `provider_id` field — this is what you need.

**Quota:** +1 profile fetch.

**Optimization:** If you already have the profile data from a recent search, check if `provider_id` is included in the search results to avoid burning a profile quota.

## Step 2: Send Invitation

```bash
curl -X POST "$UNIPILE_BASE_URL/api/v1/users/invite" \
  -H "X-API-KEY: $UNIPILE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "account_id": "'$UNIPILE_ACCOUNT_ID'",
    "provider_id": "ACoAAB1234567",
    "message": "Bonjour Marie, votre approche du coaching en leadership m'\''a interpellé. J'\''aimerais échanger avec vous sur les outils qui aident les coachs à développer leur présence en ligne."
  }'
```

**Parameters:**
- `account_id`: Unipile account ID
- `provider_id`: LinkedIn internal ID (from step 1, starts with `ACoAA...`)
- `message` (optional): personalized connection note. Max ~300 chars. LinkedIn limits personalized invitations to ~5/month on free tier — use them wisely.

**Quota:** +1 invitation.

---

## Message Personalization Rules

1. **Always reference something specific** from their profile (recent post, headline, niche).
2. **Keep it under 200 characters** — short messages have higher accept rates.
3. **No pitch in the connection request.** Just the reason for connecting.
4. **Use their first name.** Never "Cher professionnel" or generic openers.
5. **Match their language.** If their profile is in French, write in French.

### Good examples:
```
Bonjour Marie, votre post sur le burnout des dirigeants m'a parlé. Je travaille aussi dans l'accompagnement — j'aimerais échanger.
```

```
Bonjour Thomas, j'ai vu que vous accompagnez des managers en transition. Je développe des outils pour les coachs — serait ravi d'en discuter.
```

### Bad examples (never do this):
```
Bonjour, j'ai un outil qui va révolutionner votre pratique de coach...
```
```
Hi, I'd like to add you to my network.
```

---

## Batch Sending

When sending multiple invitations:

1. **Random delay 3-8s between each** — mandatory to avoid LinkedIn detection
2. **Stop after 3 consecutive failures** — likely quota exceeded or account issue
3. **Log every attempt** with result (success/failure/error) for audit
4. **Never send more than 40/day** even if quota allows 80

```bash
# Delay pattern between calls
sleep $((RANDOM % 6 + 3))
```

---

## Error Handling

The SDK throws `UnsuccessfulRequestError` with empty `message`. The real error is in `error.body.detail`.

Common errors:
- `422 errors/invalid_recipient` — wrong provider_id format (you passed a URL instead)
- `429 Too Many Requests` — rate limited, back off exponentially
- `403` — account may be restricted by LinkedIn

**On any error:** log it, decrement quota (the call still counts), and stop if 3+ consecutive failures.

---

## Safety Rules

- **Never exceed 40 invitations/day** (hard quota is 80, but play it safe).
- **Never send generic/templated invitations** — always personalize.
- **Never send invitations without qualification** — only send to ICP-matching prospects.
- **Always log the invitation** with prospect details and message content.
- **Never re-invite** someone who already rejected or hasn't responded in 2+ weeks.

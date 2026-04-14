---
name: "linkedin-dm"
description: "Send LinkedIn direct messages via Unipile API. Use when sending follow-ups, onboarding messages, or any direct communication to a prospect or customer. Supports existing conversations and new chats."
slug: "linkedin-dm"
key: "sonoria/linkedin-dm"
---

# LinkedIn DM — Unipile API

Send direct messages on LinkedIn through the Unipile proxy.

## ⛔ APPROVAL REQUIRED

**EVERY message requires board approval BEFORE sending.** No exceptions.

Before calling `sendMessage` or `startNewChat`, you MUST:

1. Prepare the message (recipient, content, touchNumber, context)
2. Submit an approval request:
   ```
   POST /api/companies/{companyId}/approvals
   { "type": "linkedin_dm", "requestedByAgentId": "<your-id>",
     "payload": { "action": "Envoyer DM LinkedIn", "target": "<name>",
     "chatId": "<id>", "touchNumber": <N>,
     "previousContext": "<resume echanges precedents>",
     "message": "<message complet>", "justification": "<why>" }}
   ```
3. **STOP. Do NOT send the message.** Wait for the next heartbeat.
4. Only execute if `PAPERCLIP_APPROVAL_STATUS === "approved"`.
5. Reference the `approvalId` in your task comment after execution.

**Reading conversations (`getAllChats`, `getAllMessages`, `getAttendees`) does NOT require approval.** Only SENDING does.

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
| messages | 100/day | Both new chats and replies count |

---

## 1. Send Message to Existing Chat

When you already have a `chat_id` (from CRM or previous conversation):

```bash
curl -X POST "$UNIPILE_BASE_URL/api/v1/chats/{chat_id}/messages" \
  -H "X-API-KEY: $UNIPILE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "text": "Bonjour Marie, je voulais faire suite à notre échange. Avez-vous eu le temps de regarder la plateforme ?"
  }'
```

**Response:** `{ "message_id": "msg_xxx" }`

**Quota:** +1 message.

## 2. Start a New Conversation

When messaging someone for the first time (must be a 1st-degree connection):

```bash
curl -X POST "$UNIPILE_BASE_URL/api/v1/chats" \
  -H "X-API-KEY: $UNIPILE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "account_id": "'$UNIPILE_ACCOUNT_ID'",
    "attendees_ids": ["ACoAAB1234567"],
    "text": "Bonjour Marie, merci d'\''avoir accepté ma demande de connexion ! Je développe une app qui aide les coachs à créer une page personnalisée à partir de leur profil LinkedIn. Seriez-vous ouverte à un échange de 15 min ?"
  }'
```

**Parameters:**
- `account_id`: Unipile account ID
- `attendees_ids`: array of `provider_id` (LinkedIn internal ID, NOT URL slug)
- `text`: the message

**Important:** `attendees_ids` requires the `provider_id` (starts with `ACoAA...`). Use the `linkedin-connect` skill's provider_id resolution if needed.

**Response:** `{ "chat_id": "chat_xxx" }`

**Quota:** +1 message.

## 3. Get Conversation History

Useful before sending a follow-up — check what was already said:

```bash
curl -X GET "$UNIPILE_BASE_URL/api/v1/chats/{chat_id}/messages?limit=30" \
  -H "X-API-KEY: $UNIPILE_API_KEY"
```

**Response:**
```json
{
  "items": [
    {
      "id": "msg_xxx",
      "text": "Message content",
      "is_sender": true,
      "timestamp": "2026-04-10T14:30:00Z"
    }
  ]
}
```

**No quota consumed.**

## 4. List All Chats

```bash
curl -X GET "$UNIPILE_BASE_URL/api/v1/chats?account_id=$UNIPILE_ACCOUNT_ID&limit=50" \
  -H "X-API-KEY: $UNIPILE_API_KEY"
```

**No quota consumed.**

## 5. Get Chat Participants

```bash
curl -X GET "$UNIPILE_BASE_URL/api/v1/chats/{chat_id}/attendees" \
  -H "X-API-KEY: $UNIPILE_API_KEY"
```

**Response:** `{ items: [{ name, profile_picture_url, profile_url, is_self }] }`

**No quota consumed.**

---

## Messaging Guidelines

### Follow-up Cadence
- **Touch 1** (after connection accepted): immediate, value-driven intro
- **Touch 2** (if no reply): 3 days later, different angle
- **Touch 3** (if still no reply): 5 days later, softer ask
- **Touch 4** (final): 7 days later, graceful close
- **After touch 4:** mark as unresponsive, stop messaging

### Message Quality Rules
1. **Always read the conversation history first** before sending a follow-up
2. **Never send the same message twice** to the same person
3. **Max 3-4 sentences** for LinkedIn DMs
4. **One question or CTA per message**
5. **Reference previous conversation** if it exists
6. **Match their tone** — if they're formal, be formal; if casual, be casual
7. **Never send messages before 8am or after 8pm** in the recipient's timezone

### Rate Limiting
- **Add 1-3s random delay between consecutive messages**
- **Never send more than 50 messages/day** (hard limit is 100, stay safe)
- **Space follow-ups by at least 3 days**

---

## Error Handling

Common errors:
- `404 chat not found` — chat_id is invalid or was remapped after account reconnection
- `422 invalid_recipient` — provider_id is wrong (URL slug instead of internal ID)
- `429 Too Many Requests` — back off, wait 30s minimum
- `403` — not connected to this person (can't DM non-connections)

**On 404 "chat not found":** the chat may have been remapped after a LinkedIn account reconnection. Check the CRM for updated chat_id.

---

## Safety Rules

- **Never DM someone who hasn't accepted your connection request** (will fail with 403).
- **Never send automated mass messages.** Every message must be contextual and personalized.
- **Always read the conversation before replying** — context matters.
- **Never share confidential information** (pricing, other prospect details) in DMs.
- **Log every message sent** with recipient, content, and timestamp.

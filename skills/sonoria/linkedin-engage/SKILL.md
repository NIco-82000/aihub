---
name: "linkedin-engage"
description: "Like and comment on LinkedIn posts via Unipile API. Use when the Content Strategist needs to engage with posts in the coach niche for visibility and relationship building. Every action requires board approval."
slug: "linkedin-engage"
key: "sonoria/linkedin-engage"
---

# LinkedIn Engage — Unipile API

Like and comment on LinkedIn posts through the Unipile proxy.

## ⛔ APPROVAL REQUIRED

**EVERY like and comment requires board approval BEFORE execution.** No exceptions.

Before calling ANY endpoint in this skill, you MUST:

1. Prepare the engagement action (post URL, author, content if comment, justification)
2. Submit an approval request (see Approval section below)
3. **STOP. Do NOT engage.** Wait for the next heartbeat.
4. Only execute if `PAPERCLIP_APPROVAL_STATUS === "approved"`.
5. Reference the `approvalId` in your task comment after execution.

See `shared/GUARDRAILS.md` for the full protocol.

---

## Setup

```
BASE_URL = $UNIPILE_BASE_URL   # e.g. http://178.16.131.17:3100
API_KEY  = $UNIPILE_API_KEY
ACCOUNT  = $UNIPILE_ACCOUNT_ID
```

## Quota

| Action | Limit | Notes |
|--------|-------|-------|
| post_engagements | 100/day | Likes + comments combined |

---

## 1. Like a Post

```bash
curl -X POST "$UNIPILE_BASE_URL/api/v1/posts/{postId}/reactions" \
  -H "X-API-KEY: $UNIPILE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "account_id": "'$UNIPILE_ACCOUNT_ID'",
    "reaction_type": "LIKE"
  }'
```

**Parameters:**
- `postId`: the LinkedIn post ID (from search results or post URL)
- `reaction_type`: `"LIKE"` (default), `"CELEBRATE"`, `"SUPPORT"`, `"LOVE"`, `"INSIGHTFUL"`, `"FUNNY"`

**Quota:** +1 post_engagement.

**Note:** Verify the exact endpoint against Unipile API documentation. The path follows the pattern of existing GET endpoints (`/api/v1/posts/{postId}/reactions`) but may differ for writes.

## 2. Comment on a Post

```bash
curl -X POST "$UNIPILE_BASE_URL/api/v1/posts/{postId}/comments" \
  -H "X-API-KEY: $UNIPILE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "account_id": "'$UNIPILE_ACCOUNT_ID'",
    "text": "Point tres pertinent sur la visibilite des coachs. Ce que j'\''observe aussi, c'\''est que les coachs qui postent regulierement sur un sujet precis attirent des clients plus qualifies. Vous avez constate la meme chose ?"
  }'
```

**Quota:** +1 post_engagement.

**Note:** Same caveat as above — verify exact endpoint with Unipile docs.

## 3. Get Post Details (no approval needed)

Before engaging, read the post's existing engagement:

```bash
# Get comments
curl -X GET "$UNIPILE_BASE_URL/api/v1/posts/{postId}/comments?account_id=$UNIPILE_ACCOUNT_ID&limit=100" \
  -H "X-API-KEY: $UNIPILE_API_KEY"

# Get reactions
curl -X GET "$UNIPILE_BASE_URL/api/v1/posts/{postId}/reactions?account_id=$UNIPILE_ACCOUNT_ID&limit=250" \
  -H "X-API-KEY: $UNIPILE_API_KEY"
```

**No approval needed** for reading. **Quota consumed** for reading (counts toward post_engagements).

---

## Approval Formats

### Single Like

```json
{
  "type": "linkedin_like",
  "requestedByAgentId": "<your-id>",
  "payload": {
    "action": "Liker un post LinkedIn",
    "postUrl": "https://www.linkedin.com/feed/update/urn:li:activity:123",
    "author": "Marie Dupont — Coach en leadership",
    "postSummary": "Post sur les 3 erreurs des coachs sur LinkedIn (450 likes, 67 commentaires)",
    "reactionType": "LIKE",
    "justification": "Post a fort engagement dans notre niche, auteur est une influenceuse coach avec 15k followers"
  }
}
```

### Single Comment

```json
{
  "type": "linkedin_comment",
  "requestedByAgentId": "<your-id>",
  "payload": {
    "action": "Commenter un post LinkedIn",
    "postUrl": "https://www.linkedin.com/feed/update/urn:li:activity:123",
    "author": "Marie Dupont — Coach en leadership",
    "postSummary": "Post sur les 3 erreurs des coachs sur LinkedIn",
    "comment": "Point tres pertinent sur la visibilite. Ce que j'observe aussi, c'est que les coachs qui postent regulierement sur un sujet precis attirent des clients plus qualifies. Vous avez constate la meme chose ?",
    "justification": "Engagement strategy — construire la relation avec cette influenceuse coach"
  }
}
```

### Batch Engagement

Pour efficacite, soumettre un lot de likes/commentaires en une seule approbation :

```json
{
  "type": "linkedin_engagement_batch",
  "requestedByAgentId": "<your-id>",
  "payload": {
    "action": "Engagement batch LinkedIn",
    "likes": [
      { "postUrl": "...", "author": "...", "postSummary": "...", "justification": "..." },
      { "postUrl": "...", "author": "...", "postSummary": "...", "justification": "..." }
    ],
    "comments": [
      { "postUrl": "...", "author": "...", "postSummary": "...", "comment": "...", "justification": "..." }
    ],
    "totalActions": 5,
    "quotaRemaining": 85
  }
}
```

---

## Comment Quality Rules

### DO

1. **Add value** — share a complementary insight, a data point, or a question
2. **Be specific** — reference what the author actually said
3. **Ask a question** — drives conversation and visibility
4. **Match the language** — French post = French comment
5. **Keep it short** — 2-4 sentences max
6. **Engage early** — posts < 2h old get the most comment visibility

### DON'T

1. **Never promote** — no product mentions, no links, no CTAs
2. **Never generic praise** — "Super post ! 👏" is worse than no comment
3. **Never disagree aggressively** — constructive counterpoints only
4. **Never comment on sensitive topics** — politics, religion, controversial takes
5. **Never comment on competitor posts negatively** — engage positively or skip
6. **Never spam** — max 10-15 comments/day

---

## Engagement Strategy

### Priority Matrix

| Post Characteristic | Action | Priority |
|---------------------|--------|----------|
| ICP coach, < 24h, high engagement | Like + Comment | P1 |
| ICP coach, < 24h, low engagement | Like + Comment (support early) | P1 |
| Industry influencer, relevant topic | Like + Comment | P2 |
| Competitor post, interesting angle | Like only (no comment) | P3 |
| ICP coach, > 48h old | Skip (engagement window passed) | P4 |

### Timing

- **Best engagement window**: 0-2h after post publication (algorithm boost)
- **Acceptable**: 2-24h (still visible)
- **Diminishing returns**: > 24h
- **Skip**: > 48h (wasted quota)

---

## Rate Limiting

- **Random delay 2-5s between engagement actions**
- **Max 15 comments/day** (even if quota allows more — avoid LinkedIn flags)
- **Max 30 likes/day** (same reason)
- **Space actions across heartbeats** — don't dump 20 likes in one session

---

## Safety Rules

- **NEVER engage without board approval.** Zero exceptions.
- **NEVER promote the product** in comments.
- **NEVER engage with controversial or sensitive content.**
- **Log every engagement** with post URL, action type, and result.
- **Track quota** — stop when approaching daily limits.

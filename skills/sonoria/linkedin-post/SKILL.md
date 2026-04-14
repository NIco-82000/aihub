---
name: "linkedin-post"
description: "Create and publish LinkedIn posts via Unipile API. Use when the Content Writer or CMO needs to publish content on the company LinkedIn page or founder's profile."
slug: "linkedin-post"
key: "sonoria/linkedin-post"
---

# LinkedIn Post — Unipile API

Publish content on LinkedIn through the Unipile proxy.

## ⛔ APPROVAL REQUIRED

**EVERY post requires board approval BEFORE publishing.** No exceptions.

Before calling ANY publish endpoint, you MUST:

1. Draft the complete post (text, hashtags, media, target account)
2. Submit an approval request:
   ```
   POST /api/companies/{companyId}/approvals
   { "type": "linkedin_post", "requestedByAgentId": "<your-id>",
     "payload": { "action": "Publier post LinkedIn",
     "account": "Profil fondateur / Page entreprise",
     "content": "<full post text>", "hashtags": ["#tag1"],
     "scheduledTime": "<YYYY-MM-DD HH:MM TZ>",
     "justification": "<why this post now>" }}
   ```
3. **STOP. Do NOT publish.** Wait for the next heartbeat.
4. Only publish if `PAPERCLIP_APPROVAL_STATUS === "approved"`.
5. Publish the EXACT content that was approved. No modifications.
6. Reference the `approvalId` in your task comment after publication.

See `shared/GUARDRAILS.md` for the full protocol.

---

## Setup

```
BASE_URL = $UNIPILE_BASE_URL
API_KEY  = $UNIPILE_API_KEY
ACCOUNT  = $UNIPILE_ACCOUNT_ID
```

---

## 1. Create a Text Post

```bash
curl -X POST "$UNIPILE_BASE_URL/api/v1/posts" \
  -H "X-API-KEY: $UNIPILE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "account_id": "'$UNIPILE_ACCOUNT_ID'",
    "text": "3 erreurs que font 90% des coachs sur LinkedIn :\n\n1. Parler de leur méthode au lieu du résultat client\n2. Poster 1x par mois au lieu de 3x par semaine\n3. Ne pas avoir de page personnalisée qui convertit\n\nLa bonne nouvelle ? Le point 3 se règle en 5 minutes.\n\n#coaching #linkedin #presence"
  }'
```

**Note:** Verify the exact endpoint path against Unipile API documentation. The post creation endpoint may vary by API version.

## 2. Create a Post with Image

```bash
curl -X POST "$UNIPILE_BASE_URL/api/v1/posts" \
  -H "X-API-KEY: $UNIPILE_API_KEY" \
  -F "account_id=$UNIPILE_ACCOUNT_ID" \
  -F "text=Résultats du mois : 12 coachs ont créé leur page personnalisée. Voici ce que ça change 👇" \
  -F "media=@/path/to/image.png"
```

---

## LinkedIn Post Best Practices

### Format Rules
- **First 2 lines are the hook** — LinkedIn truncates after ~210 chars with "...voir plus"
- **Use line breaks** for readability — walls of text get skipped
- **3-5 hashtags max** at the end, not inline
- **Optimal length:** 800-1200 chars for text posts
- **Post timing:** Tuesday-Thursday, 8-10am or 5-7pm (coach timezone)

### Content Types That Work for Coaches
1. **Listicles** — "3 erreurs que font les coachs sur LinkedIn"
2. **Before/after** — "Avant : 0 clients via LinkedIn. Après 30 jours : 8 découvertes bookées"
3. **Contrarian takes** — "Arrêtez de poster des citations motivationnelles"
4. **How-to** — "Comment créer une page qui convertit en 5 minutes"
5. **Social proof** — "Marie, coach en leadership, a bookée 5 clients en 2 semaines"
6. **Questions** — "Quel est votre plus grand défi pour trouver des clients ?"

### Posting Cadence
- **Minimum:** 3 posts/week
- **Optimal:** 5 posts/week (weekdays)
- **Never:** more than 1 post/day (LinkedIn deprioritizes)

---

## Pre-Publishing Checklist

Before every post, verify:
- [ ] Hook is in the first 2 lines
- [ ] Message is clear and specific (numbers, names, outcomes)
- [ ] CTA is present (comment, DM, link)
- [ ] Brand voice is consistent (check CMO guidelines)
- [ ] No typos or broken formatting
- [ ] Hashtags are relevant and not excessive
- [ ] CMO has approved the content

---

## Safety Rules

- **Never post without CMO approval.** All content goes through review.
- **Never post controversial or polarizing content** without CEO sign-off.
- **Never mention competitor products by name** in negative context.
- **Never post customer data** without explicit permission.
- **Log every post** with content, time, and initial engagement after 24h.

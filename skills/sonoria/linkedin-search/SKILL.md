---
name: "linkedin-search"
description: "Search LinkedIn for people, companies, and posts via Unipile API. Use when prospecting for coaches, researching companies, or finding relevant posts to engage with. Handles people search with boolean filters, company search, post search, and post engagement (comments/reactions)."
slug: "linkedin-search"
key: "sonoria/linkedin-search"
---

# LinkedIn Search — Unipile API

Search LinkedIn for people, companies, and posts through the Unipile proxy.

## Setup

All requests go through the Unipile proxy:

```
BASE_URL = $UNIPILE_BASE_URL   # e.g. http://178.16.131.17:3100
API_KEY  = $UNIPILE_API_KEY
ACCOUNT  = $UNIPILE_ACCOUNT_ID  # LinkedIn account ID in Unipile
```

Headers for all requests:
```
X-API-KEY: $API_KEY
Content-Type: application/json
```

## Quotas (daily limits)

| Action | Limit | Notes |
|--------|-------|-------|
| searches | 1000/day | Counted per page of results |
| post_engagements | 100/day | Comments + reactions fetch |
| profiles | 100/day | Full profile fetch (not search results) |

**Always check quota before calling.** If quota is exceeded, stop and report to Head of Sales or CTO.

---

## 1. Search People

Find LinkedIn profiles matching criteria.

```bash
curl -X POST "$UNIPILE_BASE_URL/api/v1/linkedin/search?account_id=$UNIPILE_ACCOUNT_ID" \
  -H "X-API-KEY: $UNIPILE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "api": "classic",
    "category": "people",
    "keywords": "coach leadership",
    "location": ["103644278"],
    "network_distance": [2, 3],
    "profile_language": ["fr"],
    "role": [{"keywords": "coach", "priority": "current", "scope": "title"}]
  }'
```

**Parameters:**
- `api`: `"classic"` (standard) or `"sales_navigator"` (premium)
- `category`: `"people"`
- `keywords`: search terms (string)
- `location`: array of LinkedIn location IDs
- `industry`: `{ "include": ["id1", "id2"] }`
- `company`: `{ "include": ["id1"] }`
- `network_distance`: array of `[1, 2, 3]` (connection degree)
- `profile_language`: array of language codes `["fr", "en"]`
- `role`: array of `{ keywords, priority: "current"|"past", scope: "title"|"function" }`
- `skills`: array of `{ id, priority: "good_to_have"|"must_have" }`
- `cursor`: pagination cursor (from previous response)

**Response:**
```json
{
  "items": [
    {
      "name": "Marie Dupont",
      "headline": "Coach en leadership | Accompagnement dirigeants",
      "location": "Paris, France",
      "current_positions": [{"title": "Coach", "company_name": "Indépendant"}],
      "profile_url": "https://www.linkedin.com/in/marie-dupont",
      "connections_count": 1200,
      "network_distance": 2,
      "profile_picture_url": "..."
    }
  ],
  "paging": {"start": 0, "page_count": 10, "total_count": 150},
  "cursor": "next-page-cursor"
}
```

**Quota:** +1 search per page_count returned.

## 2. Search Companies

```bash
curl -X POST "$UNIPILE_BASE_URL/api/v1/linkedin/search?account_id=$UNIPILE_ACCOUNT_ID" \
  -H "X-API-KEY: $UNIPILE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "api": "classic",
    "category": "companies",
    "keywords": "coaching formation"
  }'
```

**Response:** `{ items: CompanyResult[], paging, cursor? }`

## 3. Search Posts

```bash
curl -X POST "$UNIPILE_BASE_URL/api/v1/linkedin/search?account_id=$UNIPILE_ACCOUNT_ID" \
  -H "X-API-KEY: $UNIPILE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "api": "classic",
    "category": "posts",
    "keywords": "coaching clients",
    "date_posted": "past-week",
    "sort_by": "relevance"
  }'
```

## 4. Get Post Comments

```bash
curl -X GET "$UNIPILE_BASE_URL/api/v1/posts/{postId}/comments?account_id=$UNIPILE_ACCOUNT_ID&limit=100" \
  -H "X-API-KEY: $UNIPILE_API_KEY"
```

**Quota:** +N post_engagements (N = number of comments returned).

## 5. Get Post Reactions

```bash
curl -X GET "$UNIPILE_BASE_URL/api/v1/posts/{postId}/reactions?account_id=$UNIPILE_ACCOUNT_ID&limit=250" \
  -H "X-API-KEY: $UNIPILE_API_KEY"
```

**Quota:** +N post_engagements.

## 6. Lookup Search Parameters (autocomplete)

Get LinkedIn IDs for locations, industries, companies, skills:

```bash
curl -X GET "$UNIPILE_BASE_URL/api/v1/linkedin/search/parameters?account_id=$UNIPILE_ACCOUNT_ID&type=LOCATION&keywords=Paris&limit=20" \
  -H "X-API-KEY: $UNIPILE_API_KEY"
```

**Types:** `LOCATION`, `INDUSTRY`, `COMPANY`, `SCHOOL`, `SKILL`
**No quota consumed.**

---

## Safety Rules

- **Never exceed daily quotas.** Track your calls. Stop and report when approaching limits.
- **Add 2-5s random delay between consecutive searches** to avoid LinkedIn rate limiting.
- **Never search more than 500 results** per query (use cursor pagination, cap at 50 pages).
- **Log every search** with parameters and result count for audit trail.

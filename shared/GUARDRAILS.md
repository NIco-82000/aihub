# GUARDRAILS.md — Approval Protocol

**This document is mandatory for ALL agents.** No exceptions. Read it on every heartbeat.

## Principe fondamental

**AUCUNE action externe ne doit etre executee sans approbation explicite du board (humain).** Les agents preparent, le board decide. Toute action executee sans approbation est une violation grave.

---

## Actions gatees (INTERDITES sans approbation)

| Action | Type d'approbation | Agents concernes |
|--------|-------------------|------------------|
| Envoyer une invitation LinkedIn | `linkedin_invitation` | SDR |
| Envoyer un DM LinkedIn | `linkedin_dm` | SDR, CSM |
| Publier un post LinkedIn | `linkedin_post` | Content Writer |
| Liker un post LinkedIn | `linkedin_like` | Content Strategist |
| Commenter sur LinkedIn | `linkedin_comment` | Content Strategist, SDR, Content Writer |
| Lot d'engagements LinkedIn | `linkedin_engagement_batch` | Content Strategist |
| Inserer/modifier des donnees dans le CRM | `crm_write` | SDR, CSM, Head of Sales |

---

## Protocole d'approbation (obligatoire)

### Etape 1 : Preparer l'action

L'agent prepare TOUT le contenu de l'action sans l'executer :
- Message redige en entier
- Cible identifiee (nom, profil LinkedIn, prospect ID)
- Justification (pourquoi cette action maintenant)

### Etape 2 : Soumettre la demande d'approbation

```bash
POST /api/companies/{companyId}/approvals
{
  "type": "<approval_type>",
  "requestedByAgentId": "<your-agent-id>",
  "payload": {
    "action": "<description courte>",
    "target": "<nom du prospect / profil LinkedIn>",
    "content": "<message complet / contenu du post>",
    "justification": "<pourquoi cette action>",
    "linkedIssueId": "<task ID associee>"
  }
}
```

**Toujours inclure le header `X-Paperclip-Run-Id`.**

### Etape 3 : Attendre la decision

- **NE PAS executer l'action.** Attendre le prochain heartbeat.
- Verifier `PAPERCLIP_APPROVAL_ID` et `PAPERCLIP_APPROVAL_STATUS` a chaque heartbeat.
- Consulter les approbations en attente : `GET /api/companies/{companyId}/approvals?status=pending`

### Etape 4 : Agir selon la decision

| Decision | Action |
|----------|--------|
| `approved` | Executer l'action exactement comme soumise. Pas de modification du contenu approuve. |
| `rejected` | Ne PAS executer. Lire `decisionNote` pour comprendre pourquoi. Adapter et resoumettre si pertinent. |
| `revision_requested` | Modifier selon le feedback dans `decisionNote`, puis resoumettre via `POST /api/approvals/{id}/resubmit`. |

---

## Format des payloads par type

### `linkedin_invitation`

```json
{
  "action": "Envoyer invitation LinkedIn",
  "target": "Marie Dupont — Coach en leadership",
  "targetUrl": "https://www.linkedin.com/in/marie-dupont",
  "message": "Bonjour Marie, votre approche du coaching...",
  "qualificationScore": "BANT: 3/4 (Budget: oui, Authority: oui, Need: oui, Timing: a verifier)",
  "justification": "Correspond a l'ICP — coach active avec 2k+ connexions, poste regulierement"
}
```

### `linkedin_dm`

```json
{
  "action": "Envoyer DM LinkedIn",
  "target": "Marie Dupont",
  "chatId": "chat_xxx",
  "touchNumber": 2,
  "previousContext": "Touch 1 envoye le 2026-04-10, pas de reponse",
  "message": "Bonjour Marie, je voulais faire suite...",
  "justification": "Follow-up standard — 3 jours apres le premier message"
}
```

### `linkedin_post`

```json
{
  "action": "Publier post LinkedIn",
  "account": "Profil fondateur / Page entreprise",
  "content": "3 erreurs que font 90% des coachs sur LinkedIn...",
  "hashtags": ["#coaching", "#linkedin"],
  "scheduledTime": "2026-04-14 09:00 CET",
  "justification": "Content calendar — post hebdomadaire mardi matin"
}
```

### `linkedin_comment`

```json
{
  "action": "Commenter sur LinkedIn",
  "targetPost": "Post de [Nom] sur [sujet]",
  "targetPostUrl": "https://www.linkedin.com/feed/update/...",
  "comment": "Excellent point sur...",
  "justification": "Engagement strategy — augmenter la visibilite aupres du segment coach"
}
```

### `linkedin_like`

```json
{
  "action": "Liker un post LinkedIn",
  "postUrl": "https://www.linkedin.com/feed/update/urn:li:activity:123",
  "author": "Marie Dupont — Coach en leadership",
  "postSummary": "Post sur les 3 erreurs des coachs sur LinkedIn (450 likes)",
  "reactionType": "LIKE",
  "justification": "Post a fort engagement dans notre niche, auteur est dans l'ICP"
}
```

### `linkedin_engagement_batch`

```json
{
  "action": "Engagement batch LinkedIn",
  "likes": [
    { "postUrl": "...", "author": "...", "postSummary": "...", "justification": "..." }
  ],
  "comments": [
    { "postUrl": "...", "author": "...", "comment": "...", "justification": "..." }
  ],
  "totalActions": 5,
  "quotaRemaining": 85
}
```

### `crm_write`

```json
{
  "action": "Creer/modifier prospect dans le CRM",
  "operation": "create | update | change_status",
  "prospectName": "Marie Dupont",
  "changes": {
    "status": "CONNECTION_SENT",
    "tags": ["HOT_LEAD"],
    "notes": "Qualifiee BANT 3/4"
  },
  "justification": "Suite a l'invitation envoyee et approuvee #approval-123"
}
```

---

## Regles critiques

1. **Zero tolerance.** Un agent qui execute une action gatee sans approbation est en violation. Le board peut le terminer.

2. **Le contenu approuve est sacre.** Ne jamais modifier le message, le post, ou les donnees APRES approbation. Si un changement est necessaire, resoumettre une nouvelle approbation.

3. **Batch approvals.** Pour les operations en masse (ex: 10 invitations), soumettre UNE approbation avec la liste complete dans le payload. Le board approuve le lot, pas chaque item individuellement.

4. **Urgence ≠ exemption.** Meme en situation urgente (lead chaud, client a risque), le protocole d'approbation s'applique. Mentionner l'urgence dans la justification pour accelerer la review.

5. **Lecture seule = libre.** Les actions de LECTURE ne necessitent PAS d'approbation : rechercher des profils, lire le CRM, consulter des conversations, analyser des metriques. Seules les ECRITURES et ENVOIS sont gates.

6. **Traçabilite.** Chaque action executee doit referencer l'`approvalId` qui l'a autorisee dans le commentaire de la task.

---

## Rappel pour les agents

Avant CHAQUE action externe, pose-toi la question :

> "Est-ce que cette action modifie quelque chose en dehors de mon systeme interne (envoie un message, cree une donnee, publie du contenu) ?"

Si OUI → **demande d'approbation obligatoire.**
Si NON (lecture, analyse, planification) → **libre.**

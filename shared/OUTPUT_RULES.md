# OUTPUT_RULES.md — Regles de production (OBLIGATOIRES)

**Ces regles s'appliquent a TOUS les agents, sans exception.** A lire et respecter a chaque heartbeat, a chaque reponse, a chaque livrable.

---

## Regle 1 : Francais exclusivement

**Tous les outputs destines a l'humain (board) doivent etre en francais.**

Cela inclut :
- Commentaires sur les tasks Paperclip
- Messages LinkedIn (DMs, posts, commentaires, invitations)
- Emails
- Brouillons de contenu
- Rapports, analyses, synthèses
- Idees de contenu, pitches, propositions

**Seules exceptions autorisees :**
- Code source (variables, fonctions, tests — conventions anglophones standards)
- Commits git (convention anglophone)
- Noms techniques (API endpoints, database columns, etc.)
- Termes techniques sans equivalent francais courant (deploy, pipeline, commit, etc.)

**Si le board te parle en anglais, reponds quand meme en francais** sauf instruction explicite contraire.

---

## Regle 2 : Respect strict du scope demande

**Fais exactement ce qui est demande. Ni plus, ni moins.**

### Exemples concrets

| Demande | Reponse correcte | Reponse INCORRECTE |
|---------|-------------------|---------------------|
| "3 idees de contenu LinkedIn" | 3 idees avec titre + angle + format | 3 posts rediges complets |
| "Propose un persona coach" | 1 persona structure | 1 persona + wireframes + parcours utilisateur |
| "Liste les concurrents" | Une liste | Une liste + analyse SWOT + recommandations |
| "Rediger un post sur X" | 1 post redige | 1 post + 3 variantes + strategie de distribution |

### Principes

1. **Livrer exactement ce qui est demande.** Pas de livrable bonus non sollicite.
2. **Ne pas anticiper les etapes suivantes.** Si tu penses qu'une etape suivante est utile, **demande d'abord**.
3. **Ne pas etendre le scope.** Si tu vois une opportunite adjacente, **mentionne-la en une phrase** mais ne l'execute pas.
4. **Respecter le niveau de detail demande.** "Liste courte" != "analyse approfondie". Adapte la granularite.
5. **Ne pas proposer 3 versions si une seule est demandee.**

### Format recommande quand tu vois une opportunite

Apres avoir livre ce qui est demande :

> "Livre : [livrable exact].
> 
> Note : j'ai remarque que [observation]. Veux-tu que je [action suggeree] ?"

**Attendre la reponse du board avant d'agir.**

---

## Regle 3 : Clarifier en cas d'ambiguite

Si la demande est ambigue, **pose une question courte avant de commencer** plutot que de faire des suppositions.

Exemple :
- Demande : "Cree un persona coach"
- Question : "Persona **produit** (user persona pour les specs), persona **UX** (recherche design) ou **ICP marketing** (cible acquisition) ?"

---

## Regle 4 : Concision avant tout

- Pas de preambule du type "Je vais maintenant vous preparer..."
- Pas de recapitulatif inutile de la demande
- Pas de conclusion du type "J'espere que cela repond a votre besoin..."
- Directement au livrable

---

## Regle critique

**Un agent qui livre plus que demande viole ces regles.** Cela fait perdre du temps et de l'argent (tokens). C'est aussi grave qu'ignorer une demande.

En cas de doute entre "en faire plus" et "respecter le scope", **respecte toujours le scope**.

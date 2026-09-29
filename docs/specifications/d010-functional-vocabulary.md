# D-010 — Vocabulaire fonctionnel minimal du MVP

## Statut

**Proposed — validation humaine requise.**

Ce document définit un vocabulaire conceptuel. Il ne crée ni enum de code, ni table SQL, ni annotation physique.

## 1. Objectifs

Le vocabulaire doit permettre au MVP de :

- décrire les rôles utiles des cartes dans un deck donné ;
- représenter un sous-ensemble de relations factuelles ou stratégiques connues ;
- analyser les rôles présents et manquants ;
- récupérer des candidats dans l'ensemble du catalogue légal ;
- filtrer des incompatibilités connues ;
- produire des raisons structurées et auditables ;
- représenter explicitement l'absence d'information.

Il s'applique à tout le catalogue TCG Advanced EMEA. Les fixtures D-009 servent à le tester, pas à en limiter la portée.

## 2. Principes

1. **Mieux vaut `UNKNOWN` qu'une information inventée.**
2. Un tag est un rôle analytique, pas une propriété réglementaire ni une recommandation automatique.
3. Une carte peut porter zéro, un ou plusieurs rôles.
4. Un rôle peut être intrinsèque à l'usage habituel d'une carte ou dépendre du deck et du contexte.
5. Une relation ne prouve ni optimalité, ni légalité, ni inclusion obligatoire.
6. Les règles de format, zones, copies et banlist restent dans le Rules Engine.
7. Les faits extraits du texte, les dérivations, les annotations humaines et les suggestions LLM restent distinguables.
8. Toute information est versionnée et liée à une provenance.
9. Le vocabulaire est extensible par nouvelle version ; les anciennes annotations restent interprétables.
10. `GENERIC`, `ENGINE`, `BOSS`, `DEFENSIVE` ou `SYNERGIZES_WITH` ne sont pas retenus comme vérités non qualifiées : leur sens est trop dépendant du contexte.

## 3. Modèle conceptuel d'un FunctionalTag

Une attribution de rôle est conceptuellement un objet et non une simple colonne booléenne :

```text
FunctionalAssignment
  subject_card_id
  tag
  scope: INTRINSIC | DECK_CONTEXT | ARCHETYPE_CONTEXT
  context_ref: optional
  condition: optional structured predicate
  provenance
  confidence
  review_status
  vocabulary_version
```

- `INTRINSIC` signifie que le rôle est directement soutenu par le texte ou par une utilisation indépendante du deck, par exemple une carte activable depuis la main en réponse à l'adversaire.
- `DECK_CONTEXT` signifie que le rôle dépend d'une decklist ou d'un état de deck précis.
- `ARCHETYPE_CONTEXT` est un raccourci analytique versionné, jamais une restriction du catalogue.
- Une absence d'attribution signifie **non renseigné**, pas « faux ».

### Frontière normative Tag / Relation

- Un `FunctionalTag` répond à : **« quel rôle cette carte peut-elle jouer dans ce contexte ? »**
- Une `CardRelation` répond à : **« quelle assertion relie cette carte source à cette cible ou ce sélecteur ? »**
- Un tag ne contient pas implicitement de cible.
- Une relation doit toujours avoir une source, un type et une cible explicite.
- Le même effet peut justifier simultanément un tag et une relation, sans duplication sémantique : le tag sert à l'analyse agrégée du deck, la relation sert à expliquer le mécanisme et à retrouver les cartes concernées.

```text
Card A
  TAG: STARTER

Card A
  RELATION: SEARCHES
  TARGET: Card B ou Selector X
```

### Vérification de chaque FunctionalTag

| Tag | Pourquoi c'est bien un tag | Relation(s) pouvant fournir la preuve sans remplacer le tag |
|---|---|---|
| `STARTER` | Rôle de la source dans une ligne initiale ; aucune cible unique n'est implicite | `SEARCHES`, `SPECIAL_SUMMONS`, `ENABLES` |
| `EXTENDER` | Rôle contextuel de prolongation après une première action | `SPECIAL_SUMMONS`, `RECOVERS`, `ENABLES` |
| `SEARCHER` | Capacité fonctionnelle agrégée de la carte | Une ou plusieurs assertions `SEARCHES` vers des cibles/sélecteurs précis |
| `ENGINE_REQUIREMENT` | Rôle d'une pièce requise par un moteur mais parfois peu utile isolément | `REQUIRES` dans le sens des cartes qui dépendent de cette pièce |
| `PAYOFF` | Rôle de résultat recherché d'une ligne | `ENABLES`, `MATERIAL_FOR`, `SPECIAL_SUMMONS` peuvent expliquer l'accès |
| `MATERIAL_PROVIDER` | Rôle de ressource d'invocation dans un contexte | `MATERIAL_FOR` vers une cible précise |
| `DRAW` | Capacité fonctionnelle sans cible carte spécifique | Aucun lien obligatoire ; payload d'effet factuel |
| `RECOVERY` | Rôle de récupération/recyclage | `RECOVERS` vers cible/sélecteur et zones |
| `INTERRUPTION` | Rôle large de réponse au jeu adverse | `BANISHES`, `LOCKS` ou autre effet précis |
| `NEGATION` | Sous-rôle précis d'interruption annulant activation/effet | Une modalité d'effet ; une carte peut être aussi `INTERRUPTION` |
| `REMOVAL` | Rôle de réponse retirant une menace | `BANISHES` ou effet structuré de destruction/renvoi |
| `PROTECTION` | Rôle défensif de la source | `PROTECTS` vers la cible/sélecteur protégé |
| `HANDTRAP` | Propriété fonctionnelle d'activation depuis la main pour interagir | Relations d'effet éventuelles, mais aucune cible unique obligatoire |
| `BOARD_BREAKER` | Rôle contextuel contre un plateau déjà établi | Relations/modalités d'effet selon la carte |
| `FLOODGATE` | Rôle d'une restriction persistante et générale | `LOCKS` décrit précisément ce qui est restreint |

### Chevauchements explicitement résolus

- **`STARTER` vs `SEARCHER` :** distinction conservée. Une carte peut démarrer une ligne sans chercher ; une carte peut chercher une ressource sans être un starter autonome. `STARTER` est contextuel, `SEARCHER` décrit une capacité.
- **`EXTENDER` vs `SPECIAL_SUMMONS` :** distinction conservée. `EXTENDER` qualifie le rôle de la carte dans une ligne ; `SPECIAL_SUMMONS` relie une source à ce qu'elle peut invoquer. Une carte qui s'invoque elle-même peut être extender sans relation vers une autre carte.
- **`ENGINE_REQUIREMENT` vs `REQUIRES` :** distinction conservée sous réserve de nom. Le tag qualifie une pièce dans un moteur ; la relation exprime quelle source requiert quelle cible/condition. Renommage recommandé avant validation : `REQUIRED_ENGINE_PIECE`, plus explicite et moins susceptible d'être lu comme une contrainte portée par la carte elle-même.
- **`PAYOFF` vs `MATERIAL_PROVIDER` :** distinction conservée. Le premier est le résultat recherché ; le second fournit une ressource pour y accéder. Une carte peut porter les deux dans des contextes différents.
- **`INTERRUPTION` vs `NEGATION` :** les deux sont conservés. Toute `NEGATION` pertinente pendant le tour adverse peut aussi être `INTERRUPTION`, mais une interruption peut détruire, bannir, renvoyer ou imposer une restriction sans nier.
- **`PROTECTION` vs `PROTECTS` :** `PROTECTION` est le rôle de la source ; `PROTECTS` identifie les cibles et modalités protégées.
- **`RECOVERY` vs `RECOVERS` :** `RECOVERY` est le rôle de la source ; `RECOVERS` indique quelles ressources sont déplacées depuis quelles zones vers quelles destinations.

### Traitement de `FLOODGATE`

`FLOODGATE` est conservé selon une approche combinée :

1. le texte de carte peut produire un **candidat déterministe** lorsqu'il contient une restriction persistante structurée ;
2. la qualification `FLOODGATE` nécessite une **revue humaine**, car la portée stratégique et le caractère général sont contextuels ;
3. l'attribution porte un scope `INTRINSIC` ou `DECK_CONTEXT`, une condition et une provenance ;
4. une ou plusieurs relations `LOCKS` décrivent les restrictions concrètes ;
5. le tag seul ne peut pas servir de filtre dur sans `LOCKS` fiable et validation du Rules Engine.

### FunctionalTag proposés pour le MVP

| Tag | Définition minimale | Utilité MVP | Nature habituelle | Contextuel / ambiguïté | Décision proposée |
|---|---|---|---|---|---|
| `STARTER` | Carte capable d'initier seule ou avec une condition légère une ligne produisant un état utile | Mesurer l'accès au moteur et détecter un manque de départs | Annotation humaine ou dérivation revue | Très contextuel ; préciser condition et résultat minimal | **Inclure maintenant** |
| `EXTENDER` | Carte qui poursuit ou élargit une ligne après qu'une première ressource est disponible | Détecter capacité de prolongation et résilience | Annotation humaine | Dépend de l'état, des invocations déjà faites et des verrous | **Inclure maintenant** |
| `SEARCHER` | Carte dont un effet permet d'obtenir depuis le Deck une carte correspondant à un sélecteur explicite | Expliquer la consistance et relier aux cibles | Dérivable du texte puis vérifiable | Peut chercher des familles différentes selon l'effet | **Inclure maintenant** |
| `ENGINE_REQUIREMENT` | Carte nécessaire ou fortement requise par un moteur mais souvent indésirable à piocher seule | Détecter requirements/garnets et ratios incohérents | Annotation humaine avec relation `REQUIRES` | « Nécessaire » dépend de la ligne et de la construction | **Inclure maintenant** |
| `PAYOFF` | Carte ou effet représentant un résultat recherché d'une ligne de moteur | Expliquer ce que produit le moteur et compléter Main/Extra Deck | Annotation humaine | Remplace `BOSS`, trop subjectif ; préciser zone et contexte | **Inclure maintenant** |
| `MATERIAL_PROVIDER` | Carte dont le rôle principal dans le contexte est de fournir un matériau, tribut ou ressource d'invocation | Analyser accès à l'Extra Deck/Rituel/Tribut | Dérivation partielle + annotation | Le même monstre peut être starter ou payoff ailleurs | **Inclure maintenant** |
| `DRAW` | Carte fournissant directement une ou plusieurs pioches selon une condition connue | Analyser accès aux ressources sans confondre avec search | Dérivable du texte | La qualité ou le net advantage ne sont pas inclus | **Inclure maintenant** |
| `RECOVERY` | Carte récupérant ou recyclant une ressource depuis le Cimetière, bannissement ou autre zone | Mesurer grind et continuité | Dérivable du texte + contexte | Destination et type de ressource doivent être précisés | **Inclure maintenant** |
| `INTERRUPTION` | Carte utilisable pour perturber une action adverse pendant son tour ou une chaîne | Mesurer les réponses interactives | Dérivation partielle + annotation | Catégorie large ; mécanisme détaillé par relations | **Inclure maintenant** |
| `NEGATION` | Sous-rôle d'interruption annulant explicitement activation ou effet | Distinguer negate de destruction/bannissement | Dérivable du texte puis vérifiable | Ne signifie pas que la carte est toujours activable | **Inclure maintenant** |
| `REMOVAL` | Carte capable de retirer une carte adverse du terrain ou de la rendre non persistante | Identifier réponses aux menaces | Dérivable du texte | Détruire, bannir, renvoyer et envoyer restent des modalités distinctes | **Inclure maintenant** |
| `PROTECTION` | Carte prévenant ou remplaçant une destruction, un ciblage, une affectation ou une perte connue | Détecter résilience défensive | Dérivable du texte + annotation | Toujours préciser ce qui est protégé et contre quoi | **Inclure maintenant** |
| `HANDTRAP` | Carte activable depuis la main, principalement pendant le tour adverse, afin d'interagir | Recommandations génériques et analyse d'interaction | Dérivation partielle + annotation | Terme communautaire ; condition d'activation obligatoire | **Inclure maintenant** |
| `BOARD_BREAKER` | Carte principalement employée pour réduire un plateau adverse déjà établi | Tester recommandations going-second sans profil complet | Annotation humaine | Fortement contextuel ; ne promet ni efficacité ni optimalité | **Inclure maintenant** |
| `FLOODGATE` | Carte imposant une restriction persistante et générale sur un ensemble d'actions ou de cartes | Détecter contraintes et incompatibilités majeures | Annotation humaine soutenue par texte | Terme large et sensible au contexte ; relation `LOCKS` requise | **Inclure maintenant, usage prudent** |

### Concepts non retenus comme FunctionalTag MVP

| Concept | Traitement recommandé |
|---|---|
| `BOSS` | Utiliser `PAYOFF` avec contexte ; « boss » est subjectif |
| `ENGINE` | Représenter l'appartenance à un moteur dans le contexte du deck, pas comme qualité intrinsèque |
| `GENERIC` | Déduire la compatibilité à partir des contraintes et annotations ; aucune carte n'est universellement générique |
| `DEFENSIVE` | Trop large ; utiliser `PROTECTION`, `INTERRUPTION`, `NEGATION` ou une explication contextuelle |
| `EXTRA_DECK_PAYOFF` | Utiliser `PAYOFF` + zone `EXTRA` |
| `NORMAL_SUMMON_REQUIRED` | Condition structurée ou relation `REQUIRES`, pas rôle fonctionnel |
| `SPECIAL_SUMMON_REQUIRED` | Condition/restriction structurée, pas rôle fonctionnel |
| `OTK_ENABLER` | À différer jusqu'à définition d'un modèle de lignes/dégâts fiable |

## 4. Modèle conceptuel d'une CardRelation

Une relation relie une carte source à une carte cible ou à un sélecteur de cartes versionné. Le sélecteur est nécessaire lorsqu'un texte vise « une carte Branded », « un monstre Wyrm », etc.

```text
CardRelationAssertion
  source_card_id
  relation_type
  target: card_id | selector
  direction
  condition: optional structured predicate
  effect_detail: optional structured payload
  context_ref: optional
  provenance
  confidence
  review_status
  vocabulary_version
```

Créer des milliers d'arêtes carte-à-carte à partir d'un même sélecteur est facultatif et dérivé ; l'assertion canonique peut conserver le sélecteur.

### CardRelation proposées pour le MVP

| Relation | Définition | Direction | Nature / confiance minimale | Utilité MVP | Exemple conceptuel | Inconnu possible |
|---|---|---|---|---|---|---|
| `SEARCHES` | La source ajoute depuis le Deck une cible correspondant à un sélecteur | Source → cible/sélecteur | Texte ou dérivation revue ; `HIGH` | Consistance, starters et explication des accès | A cherche une Magie/Piège de sa famille | Oui si extraction impossible |
| `SPECIAL_SUMMONS` | La source peut invoquer spécialement la cible ou une carte correspondant au sélecteur | Source → cible/sélecteur | Texte ; `HIGH` | Extenders, accès aux matériaux et lignes | A invoque un monstre depuis le Cimetière | Oui |
| `SENDS_TO_GRAVEYARD` | La source envoie la cible depuis une zone identifiée vers le Cimetière | Source → cible/sélecteur | Texte ; `HIGH` | Setup, coûts et effets déclenchés | A envoie un monstre du Deck au Cimetière | Oui |
| `BANISHES` | La source bannit la cible depuis une zone identifiée | Source → cible/sélecteur | Texte ; `HIGH` | Ressources bannies, removal et incompatibilités | A bannit une carte du Cimetière | Oui |
| `RECOVERS` | La source déplace une cible depuis Cimetière/bannissement vers main, terrain, Deck ou Extra Deck | Source → cible/sélecteur | Texte ; `HIGH` | Grind et recyclage | A ajoute une carte bannie à la main | Oui |
| `MATERIAL_FOR` | La source est utilisable ou prévue comme matériau/tribut pour la cible dans un contexte explicite | Source → cible | Règle dérivée ou annotation revue ; `MEDIUM/HIGH` | Construction de l'Extra Deck et Rituel | A sert de matériau à un Synchro de niveau donné | Oui, souvent contextuel |
| `REQUIRES` | L'utilisation ou l'effet pertinent de la source exige la présence, propriété ou état de la cible | Source → cible/sélecteur | Texte/dérivation revue ; `HIGH` pour fait, `MEDIUM` pour ligne | Détecter requirements et suggestions incomplètes | A exige un monstre Wyrm en main | Oui |
| `ENABLES` | La source rend accessible une action ou ligne impliquant la cible, sans la chercher/invoquer directement | Source → cible | Annotation humaine revue ; `MEDIUM+` | Relations stratégiques explicables | A permet d'accéder à un payoff donné | Oui, valeur par défaut |
| `SUPPORTS` | La source améliore ou complète de manière documentée l'utilisation de la cible | Source → cible/sélecteur | Annotation humaine revue ; `MEDIUM+` | Candidats de support connus | A protège ou alimente une famille | Oui, valeur par défaut |
| `CONFLICTS_WITH` | Source et cible ont une incompatibilité documentée dans le contexte donné | Symétrique par défaut, directionnelle si causalité | Dérivation ou annotation revue ; `HIGH` pour rejet dur, `MEDIUM` pour conflit souple | Filtrer ou pénaliser un candidat | Verrou de A empêche le plan principal de B | Oui |
| `LOCKS` | L'activation/utilisation de la source impose une restriction structurée | Source → prédicat affectant des cartes/actions | Texte/dérivation revue ; `HIGH` | Rejet déterministe d'incompatibilités connues | A limite les invocations à un type | Oui si verrou non structuré |
| `PROTECTS` | La source protège la cible contre une modalité structurée | Source → cible/sélecteur | Texte ; `HIGH` | Résilience et explication | A empêche la destruction de B | Oui |

### Vérification de chaque CardRelation

Toutes les relations proposées respectent la forme source → cible :

- `SEARCHES`, `SPECIAL_SUMMONS`, `SENDS_TO_GRAVEYARD`, `BANISHES`, `RECOVERS` et `PROTECTS` décrivent un effet de la source sur une carte ou classe de cartes.
- `MATERIAL_FOR` relie une ressource source à la cible pour laquelle elle sert de matériau dans un contexte donné.
- `REQUIRES` relie la source à la carte, propriété ou condition dont elle dépend.
- `ENABLES` et `SUPPORTS` sont des annotations stratégiques directionnelles ; elles exigent contexte et justification.
- `CONFLICTS_WITH` est symétrique uniquement lorsque l'incompatibilité l'est réellement ; sinon la causalité reste directionnelle.
- `LOCKS` relie la source à un sélecteur/prédicat décrivant les actions ou cartes restreintes.

Une relation sans cible résolue ni sélecteur valide n'est pas enregistrée comme assertion : elle reste `UNKNOWN`.

### Relations différées ou rejetées

- `SYNERGIZES_WITH` seul est trop vague : utiliser `ENABLES`, `SUPPORTS` ou une annotation contextuelle motivée.
- `ADDS_TO_HAND` est trop général pour le premier vocabulaire : `SEARCHES` couvre Deck → main et `RECOVERS` les zones de récupération. Il pourra être ajouté si les analyses exigent une modalité générique.
- `RESTRICTS` est absorbé par `LOCKS` avec un prédicat structuré.
- Les relations de ciblage, coût, destruction, renvoi ou changement de contrôle peuvent vivre dans le payload d'effet avant de justifier de nouveaux types.

## 5. Propriété intrinsèque et rôle contextuel

Une carte n'est jamais réduite à un rôle unique.

Exemple conceptuel :

```text
Carte A
  HANDTRAP / INTRINSIC
    condition: activable depuis la main pendant le tour adverse
  EXTENDER / DECK_CONTEXT(deck-version-42)
    condition: un monstre compatible est déjà présent
  MATERIAL_PROVIDER / DECK_CONTEXT(deck-version-42)
    condition: cible Extra Deck accessible
```

Le moteur agrège les rôles applicables au contexte courant. Il ne convertit pas une annotation contextuelle en propriété universelle.

## 6. Provenance, confiance et statut de revue

### Origine de l'information

| `origin_type` | Signification | Peut devenir canonique ? |
|---|---|---|
| `SOURCE_CARD_TEXT` | Affirmation directement soutenue par un texte de carte versionné | Oui, après extraction validée ou saisie vérifiée |
| `DETERMINISTIC_DERIVATION` | Résultat d'une règle transparente appliquée à des données sources | Oui, avec version de règle et entrées conservées |
| `HUMAN_ANNOTATION` | Interprétation stratégique saisie par un humain identifié | Oui, selon politique de revue |
| `LLM_SUGGESTION` | Proposition produite ou reformulée avec assistance LLM | **Non**, tant qu'elle n'est pas revue et promue explicitement |
| `UNKNOWN` | Information absente, contradictoire ou insuffisamment fiable | Non ; doit rester visible comme indisponible |

### Confiance

- `HIGH` : preuve directe ou annotation revue sans contradiction connue.
- `MEDIUM` : annotation contextuelle plausible et revue, avec limites documentées.
- `LOW` : hypothèse à investiguer ; inutilisable pour rejeter une carte ou affirmer un fait à l'utilisateur.
- `UNKNOWN` : aucune assertion exploitable.

La confiance ne remplace pas la provenance. Une suggestion LLM à confiance déclarée élevée reste une suggestion LLM.

### Recommandation sur la forme de confiance

Pour le MVP, conserver exclusivement la confiance **catégorielle** `HIGH/MEDIUM/LOW/UNKNOWN`.

- Elle est compréhensible par les réviseurs et les utilisateurs.
- Elle évite la fausse précision d'un score comme `0,83` qui ne serait ni calibré ni statistiquement démontré.
- Les règles d'utilisation peuvent être explicites par catégorie.
- Un score numérique ne sera envisagé qu'après constitution d'un corpus évalué, d'une méthode de calibration et d'un besoin produit mesurable.

### Statut de revue

- `UNREVIEWED`
- `REVIEWED`
- `REJECTED`
- `SUPERSEDED`

### Promotion d'une suggestion LLM

```text
LLM_SUGGESTION / UNREVIEWED
        ↓ revue humaine documentée
HUMAN_APPROVED
        ↓ création d'une nouvelle assertion canonique
HUMAN_ANNOTATION / REVIEWED
```

- `HUMAN_APPROVED` est un événement de workflow, pas un nouveau type de provenance canonique.
- La suggestion d'origine est conservée pour l'audit ; elle n'est pas réécrite silencieusement.
- L'assertion approuvée reçoit l'identité du réviseur, l'horodatage, la justification, les références examinées et le lien vers la suggestion d'origine.
- Toute `HUMAN_ANNOTATION` canonique doit obligatoirement avoir une provenance explicite et une trace de création/révision.
- Le LLM ne peut ni s'auto-approuver, ni modifier le statut de revue, ni publier directement une annotation.

### Champs de preuve minimaux

- identifiant de la source ou annotateur ;
- référence du texte/snapshot ou justification humaine ;
- date de création et de revue ;
- version du vocabulaire et de la règle de dérivation ;
- contexte d'application ;
- confiance et statut de revue.

### Règles d'utilisation

1. Un filtre dur de recommandation nécessite une règle déterministe ou une assertion `HIGH` revue.
2. Une annotation `MEDIUM` peut influencer le classement mais doit rester explicable.
3. `LOW`, `LLM_SUGGESTION` non revue et `UNKNOWN` ne peuvent jamais justifier une exclusion dure.
4. Une affirmation utilisateur doit indiquer si elle relève d'un fait, d'une dérivation ou d'une appréciation.
5. Toute modification produit une nouvelle version ou marque l'ancienne `SUPERSEDED`.

## 7. Exemples conceptuels

### Searcher factuel

```text
tag: SEARCHER
scope: INTRINSIC
origin: SOURCE_CARD_TEXT
confidence: HIGH
relation: SEARCHES -> selector(archetype = X, card_type = SPELL)
condition: effect activation requirements
```

### Starter contextuel

```text
tag: STARTER
scope: DECK_CONTEXT
context: deck-version-42
origin: HUMAN_ANNOTATION
confidence: MEDIUM
evidence: reviewed test line
```

### Conflit dur

```text
relation: CONFLICTS_WITH
source: card A
target: card B
reason: LOCKS(summon_type = FUSION_ONLY)
origin: DETERMINISTIC_DERIVATION
confidence: HIGH
```

### Information indisponible

```text
card: outside-pilot-card
functional_roles: UNKNOWN
strategic_relations: INFORMATION_UNAVAILABLE
legality: evaluated independently by Rules Engine
```

La carte reste recherchable, ajoutable et validable malgré l'absence d'annotation stratégique.

### Carte cible et sélecteur cible

```text
Card A
  SEARCHES
  target_card_id: Card B
```

Cette assertion concerne une carte déterminée et stable.

```text
Card A
  SEARCHES
  target_selector:
    card_category: MONSTER
    race: WYRM
```

```text
Card A
  SUPPORTS
  target_selector:
    archetype: BRANDED
    card_category: SPELL_OR_TRAP
```

Un sélecteur est un prédicat versionné évalué contre un `CatalogueSnapshot`. Il ne constitue pas une liste figée : une nouvelle carte peut correspondre au sélecteur dans un snapshot futur. Les arêtes carte-à-carte éventuellement matérialisées sont des dérivations reproductibles. Un sélecteur qui ne correspond à aucune carte dans le snapshot retourne un ensemble vide ; il ne doit pas être remplacé par une cible inventée.

## 8. Utilisation future par le moteur de recommandation

```text
DeckState versionné
  → compter les rôles connus et signaler la couverture inconnue
  → détecter un besoin selon des règles explicables
  → interroger TOUT le catalogue légal
  → appliquer règles de format, zone, copies et banlist
  → appliquer LOCKS / REQUIRES / CONFLICTS_WITH fiables
  → classer avec tags et relations contextuelles revues
  → produire suggestions + raisons + provenance
  → revalider déterministement la decklist résultante
  → demander au LLM uniquement la contextualisation/explanation
```

Le LLM ne détermine ni l'existence des cartes, ni leur légalité, ni leur rôle canonique. Il peut proposer une annotation dans un workflow séparé, mais celle-ci reste `LLM_SUGGESTION/UNREVIEWED` jusqu'à validation humaine.

## 9. Compatibilité avec le catalogue complet

- Une carte hors D-009 utilise exactement les mêmes structures qu'une fixture pilote.
- Une carte sans archétype peut avoir des tags et relations vers des sélecteurs de type/Attribut/rôle.
- Une carte générique n'a pas besoin d'un tag `GENERIC` ; sa compatibilité est évaluée par contexte.
- Une carte peut avoir plusieurs assignments simultanés, éventuellement contradictoires dans des contextes différents.
- Une carte sans annotation retourne `UNKNOWN / INFORMATION_UNAVAILABLE` pour l'analyse stratégique, sans affecter sa légalité.
- Aucune requête du moteur ne doit filtrer le catalogue sur l'appartenance au dataset D-009.

## 10. Éléments volontairement hors MVP

- matchup complet et plans de Side Deck ;
- mesure de puissance réelle, tier list ou prédiction de tournoi ;
- métagame et popularité ;
- probabilités de pioche et simulation de mains ;
- simulation de duel ou de lignes exhaustives ;
- chaînes complètes, Damage Step et timing exhaustif ;
- rulings complexes non formalisés ;
- optimisation automatique complète du Side Deck ;
- scoring opaque produit uniquement par un LLM ;
- taxonomie exhaustive de tous les effets Yu-Gi-Oh! ;
- inférence automatique non revue de toutes les synergies du catalogue.

## 11. Questions restant à valider

1. La liste des quinze `FunctionalTag` MVP est-elle acceptée telle quelle ?
2. Le renommage recommandé `ENGINE_REQUIREMENT` → `REQUIRED_ENGINE_PIECE` est-il accepté ?
3. `MATERIAL_PROVIDER` est-il conservé sous ce nom ?
4. `NEGATION` reste-t-il un tag séparé pouvant coexister avec `INTERRUPTION` ?
5. La liste des douze `CardRelation` est-elle acceptée sans ajout ni retrait ?
6. Le modèle de cible carte précise ou sélecteur versionné est-il accepté ?
7. La confiance catégorielle `HIGH/MEDIUM/LOW/UNKNOWN` est-elle acceptée pour le MVP ?
8. Une seule revue humaine suffit-elle, ou une double revue est-elle exigée pour les assertions utilisées comme filtres durs ?
9. Quels rôles projet sont autorisés à promouvoir une `LLM_SUGGESTION` ?
10. Le traitement combiné de `FLOODGATE` — candidat déterministe, revue humaine, relation `LOCKS` — est-il accepté ?

## 12. Recommandation finale avant validation

- **FunctionalTag à conserver :** les quinze tags proposés, sous réserve du renommage ci-dessous.
- **FunctionalTag à renommer :** `ENGINE_REQUIREMENT` → `REQUIRED_ENGINE_PIECE` recommandé ; aucun remplacement n'est appliqué avant validation.
- **FunctionalTag à reporter :** aucun parmi les quinze ; `OTK_ENABLER`, `GENERIC`, `BOSS` et autres concepts déjà exclus restent reportés.
- **CardRelation à conserver :** les douze relations proposées.
- **CardRelation à renommer :** aucune nécessaire ; `MATERIAL_FOR` doit seulement rester contextuelle et conditionnée.
- **CardRelation à reporter :** `ADDS_TO_HAND` générique, `RESTRICTS` distinct et `SYNERGIZES_WITH` vague restent reportés.

La frontière Tag/Relation est suffisamment propre pour une validation future : les points encore ouverts relèvent du nommage et de la gouvernance de revue, non d'une refonte de la taxonomie.

## 13. Critères d'acceptation de D-010

- [ ] Les FunctionalTag MVP et leurs définitions sont approuvés.
- [ ] Les CardRelation MVP et leur direction sont approuvées.
- [ ] La représentation intrinsèque/contextuelle est approuvée.
- [ ] Les origines, niveaux de confiance et statuts de revue sont approuvés.
- [ ] La politique de traitement des suggestions LLM est approuvée.
- [ ] Les concepts différés et les frontières avec le Rules Engine sont approuvés.


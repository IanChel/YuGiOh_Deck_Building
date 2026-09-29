# ADR-0010 — Vocabulaire minimal de rôles et relations

## Status

Accepted

## Context

Le moteur doit analyser des decks et préparer des recommandations explicables sans transformer les fixtures D-009 en whitelist ni tenter de modéliser toutes les interactions Yu-Gi-Oh!.

Les rôles sont souvent contextuels. Les relations peuvent provenir du texte, d'une dérivation ou d'une annotation. Une absence d'information doit rester représentable et le LLM ne doit jamais promouvoir seul une hypothèse en fait canonique.

## Decision

Adopter le vocabulaire, les règles de contexte et le modèle de provenance définis dans [`docs/specifications/d010-functional-vocabulary.md`](../specifications/d010-functional-vocabulary.md).

Le vocabulaire proposé comprend quinze `FunctionalTag` :

- `STARTER`, `EXTENDER`, `SEARCHER`, `ENGINE_REQUIREMENT`, `PAYOFF` ;
- `MATERIAL_PROVIDER`, `DRAW`, `RECOVERY` ;
- `INTERRUPTION`, `NEGATION`, `REMOVAL`, `PROTECTION` ;
- `HANDTRAP`, `BOARD_BREAKER`, `FLOODGATE`.

Il comprend douze `CardRelation` :

- `SEARCHES`, `SPECIAL_SUMMONS`, `SENDS_TO_GRAVEYARD`, `BANISHES`, `RECOVERS` ;
- `MATERIAL_FOR`, `REQUIRES`, `ENABLES`, `SUPPORTS` ;
- `CONFLICTS_WITH`, `LOCKS`, `PROTECTS`.

Une attribution porte un scope intrinsèque ou contextuel, une condition éventuelle, une provenance, une confiance, un statut de revue et une version de vocabulaire. Les suggestions LLM restent non canoniques jusqu'à revue humaine explicite.

La frontière proposée est normative : un `FunctionalTag` décrit le rôle de la carte sans cible implicite ; une `CardRelation` décrit une assertion source → carte cible ou sélecteur versionné. Un même effet peut fournir à la fois un tag agrégé et une relation explicative sans que les deux concepts soient confondus.

La confiance reste catégorielle (`HIGH/MEDIUM/LOW/UNKNOWN`) au MVP afin d'éviter une fausse précision. Une `LLM_SUGGESTION` doit être revue, approuvée par un humain puis recréée comme `HUMAN_ANNOTATION/REVIEWED` avec provenance et trace de révision ; le LLM ne peut jamais effectuer cette promotion.

`FLOODGATE` est conservé selon une approche combinée : détection candidate depuis le texte, qualification humaine contextuelle, relation `LOCKS` pour la restriction structurée et validation déterministe avant tout filtre dur. `ENGINE_REQUIREMENT` est conservé sous ce nom.

Les cibles de relation sont soit une carte précise, soit un sélecteur versionné évalué contre un `CatalogueSnapshot`.

La provenance canonique est limitée à `SOURCE_CARD_TEXT`, `DETERMINISTIC_DERIVATION`, `HUMAN_ANNOTATION`, `LLM_SUGGESTION` et `UNKNOWN`. La confiance reste catégorielle : `HIGH`, `MEDIUM`, `LOW`, `UNKNOWN` ; aucun score numérique n'est utilisé au MVP.

Le workflow de promotion est : `LLM_SUGGESTION / UNREVIEWED` → revue humaine documentée → `HUMAN_ANNOTATION / REVIEWED`. La promotion conserve l'identité du réviseur, la date, la justification, les sources examinées et le lien vers la suggestion initiale.

Une seule revue humaine traçable suffit au MVP pour une assertion susceptible de servir de filtre dur. Une suggestion LLM non revue ne peut jamais devenir un filtre dur.

## Consequences

- Une carte peut avoir plusieurs rôles ou aucun rôle connu.
- Une carte sans annotation reste accessible et légalement validable.
- Les relations ne valent ni recommandation automatique, ni preuve de légalité, ni optimalité.
- Un sélecteur cible est évalué contre un snapshot ; il ne devient jamais une liste de cartes inventée ou figée à travers les versions.
- `ADDS_TO_HAND`, `RESTRICTS` et `SYNERGIZES_WITH` restent explicitement différés.
- Les filtres durs exigent une règle déterministe ou une information revue à confiance élevée.
- L'extension future se fait par version du vocabulaire sans réécrire les anciennes assertions.

## Alternatives considered

- Un tag unique par carte : rejeté car les rôles sont multiples et contextuels.
- Une taxonomie exhaustive dès le MVP : rejetée comme coûteuse et fragile.
- Un graphe de synergies produit par LLM : rejeté comme non fiable et non auditable.
- Aucun vocabulaire stratégique : rejeté car les recommandations seraient inexplicables.

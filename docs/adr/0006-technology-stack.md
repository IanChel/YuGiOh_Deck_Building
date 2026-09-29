# ADR-0006 — Stack et architecture du MVP

## Status

Accepted

## Context

Le projet combine une interface d'édition riche, une API, un domaine réglementaire, des traitements algorithmiques et une intégration IA. Le MVP doit rester simple à développer et exploiter.

## Decision

La stack est :

- Frontend : Next.js et TypeScript.
- Backend : Python et FastAPI.
- Base de données : PostgreSQL.
- Architecture : monolithe modulaire.

Les responsabilités sont séparées conceptuellement en Frontend, Backend API, Domain/Models, Catalogue/Search, Deck Building Engine, Rules/Validation Engine, Analysis Engine, Candidate Retrieval, AI Integration, Data Import et Database. Ces frontières sont des modules internes, pas des microservices au MVP.

Le flux de recommandation part d'un `DeckState`, interroge le catalogue légal général via `Candidate Retrieval`, applique les contraintes et données structurées, puis transmet uniquement les suggestions vérifiées à l'intégration IA pour contextualisation ou explication. Aucun module n'est organisé autour d'une liste fermée d'archétypes.

## Consequences

- Le domaine et les moteurs ne dépendent pas directement de FastAPI, PostgreSQL ou du fournisseur LLM.
- Le catalogue et la recherche restent indépendants des fixtures d'archétypes utilisées dans les tests.
- Le LLM ne sert jamais de moteur de recherche de cartes.
- Les contrats API doivent être versionnés et partagés avec le frontend.
- Le déploiement peut rester unitaire tout en maintenant des frontières extractibles.
- Deux écosystèmes de langage sont assumés pour adapter chaque couche à son usage.

## Alternatives considered

- TypeScript de bout en bout : cohérent, mais moins adapté au futur travail algorithmique/IA de l'équipe.
- Python de bout en bout : écosystème frontend insuffisant pour l'éditeur visé.
- Microservices : rejetés comme complexité opérationnelle prématurée.


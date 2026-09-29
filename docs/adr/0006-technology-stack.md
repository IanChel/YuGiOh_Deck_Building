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

Les responsabilités sont séparées conceptuellement en Frontend, Backend API, Domain/Models, Deck Building Engine, Rules/Validation Engine, Analysis Engine, AI Integration, Data Import et Database. Ces frontières sont des modules internes, pas des microservices au MVP.

## Consequences

- Le domaine et les moteurs ne dépendent pas directement de FastAPI, PostgreSQL ou du fournisseur LLM.
- Les contrats API doivent être versionnés et partagés avec le frontend.
- Le déploiement peut rester unitaire tout en maintenant des frontières extractibles.
- Deux écosystèmes de langage sont assumés pour adapter chaque couche à son usage.

## Alternatives considered

- TypeScript de bout en bout : cohérent, mais moins adapté au futur travail algorithmique/IA de l'équipe.
- Python de bout en bout : écosystème frontend insuffisant pour l'éditeur visé.
- Microservices : rejetés comme complexité opérationnelle prématurée.


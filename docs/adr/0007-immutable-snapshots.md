# ADR-0007 — Versions et snapshots immuables des résultats

## Status

Accepted

## Context

Une decklist dépend du catalogue, du format, de la banlist et de l'instant de génération. Ces référentiels évoluent et doivent rester auditables.

## Decision

Tout résultat de génération enregistre au minimum :

- le format et sa version de règles ;
- la version de banlist ;
- la version du catalogue de cartes ;
- l'horodatage de génération.

Les snapshots publiés sont immuables. Une correction ou mise à jour produit une nouvelle version plutôt qu'une modification rétroactive.

## Consequences

- Le contexte d'une génération peut être reconstruit.
- Les références historiques doivent être conservées selon une politique de rétention documentée.
- Les imports distinguent staging, validation et publication d'un snapshot.
- Les analyses et logs doivent propager les identifiants de version.

## Alternatives considered

- Conserver seulement la decklist : rejeté car son contexte disparaît.
- Versionner uniquement la banlist : insuffisant si le catalogue ou les règles changent.
- Mettre à jour les snapshots en place : rejeté car non auditable.


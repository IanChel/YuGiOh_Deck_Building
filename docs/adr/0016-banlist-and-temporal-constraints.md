# ADR-0016 — Banlists et contraintes temporelles

## Status

Proposed

## Context

Les limitations officielles doivent rester reproductibles à travers les changements de liste, de format, de catalogue et de disponibilité régionale. Une restriction ne peut pas être stockée directement sur `Card`, et l'absence d'entrée ne doit pas masquer une banlist incomplète ou une référence non résolue.

## Proposed decision

- représenter la famille officielle par une identité `Banlist` stable liée à `Format` et `RegionalScope` ;
- représenter chaque publication par un `BanlistSnapshot` immuable avec source, publication, effet, fin éventuelle, intégrité et supersession ;
- conserver `FormatSnapshot` et `BanlistSnapshot` comme références séparées du contexte reproductible, avec contrôle de compatibilité ;
- représenter chaque restriction par une `BanlistEntry` vers la `Card` stable ;
- limiter les restrictions canoniques à `FORBIDDEN`, `LIMITED` et `SEMI_LIMITED` ;
- dériver les limites numériques 0, 1 et 2 sans stocker une seconde vérité canonique ;
- interpréter une absence valide comme « aucune restriction explicite », jamais comme une entrée `UNLIMITED` ;
- conditionner cette absence valide à un snapshot complet et à un contexte résolu ;
- utiliser les intervalles `[effective_from, effective_until)` sans chevauchement ordinaire ;
- corriger par nouveau snapshot et supersession explicite, sans réécriture ;
- mettre en quarantaine toute référence inconnue ou ambiguë et empêcher une conclusion positive de légalité tant que l'intégrité n'est pas rétablie ;
- maintenir banlist, disponibilité régionale et règles générales de format comme dimensions distinctes.

La proposition complète et ses questions de validation sont décrites dans [`docs/specifications/d016-banlist-and-temporal-constraints.md`](../specifications/d016-banlist-and-temporal-constraints.md).

## Consequences

- Une même carte conserve son identité à travers les snapshots du catalogue.
- Une banlist future peut être publiée sans devenir immédiatement applicable.
- Les corrections historiques n'altèrent pas les analyses déjà épinglées.
- Une carte sans entrée n'est traitée comme non restreinte par la liste que si la complétude est démontrée.
- Les anomalies de mapping ou les doublons contradictoires bloquent la publication fiable.
- La future validation devra combiner format, région, catalogue, disponibilité et banlist.

## Alternatives considered

- Stocker la restriction dans `Card` : écarté car elle varie dans le temps et selon le contexte.
- Faire de chaque publication une nouvelle `Banlist` : écarté car l'identité de la famille serait perdue.
- Rattacher les entrées à `CardSnapshot` : écarté car la restriction vise l'identité stable de carte.
- Stocker restriction et `max_copies` comme deux vérités : écarté en raison du risque de contradiction.
- Créer des entrées `UNLIMITED` pour toutes les autres cartes : écarté car volumineux et susceptible de masquer une liste incomplète.
- Corriger un snapshot en place : écarté car non reproductible.
- Résoudre automatiquement une carte par nom approximatif ou suggestion LLM : écarté car non fiable pour une donnée normative.


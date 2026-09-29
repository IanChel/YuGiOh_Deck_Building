# ADR-0004 — Identifiants de cartes et stratégie linguistique

## Status

Accepted

## Context

Les noms et textes peuvent varier selon la langue et évoluer. Les relations métier ne doivent pas dépendre d'une chaîne traduite, tandis que le MVP doit offrir une interface française.

## Decision

Un identifiant numérique de carte constitue l'identité technique principale. Les données canoniques utilisent l'anglais lorsque nécessaire. L'interface MVP est française.

Les valeurs canoniques, identifiants externes et contenus d'affichage localisés sont séparés. Le modèle autorise l'ajout ultérieur d'autres traductions sans modifier l'identité d'une carte.

## Consequences

- Les noms ne sont jamais utilisés comme clé ou relation.
- La résolution d'un nom localisé aboutit à un identifiant canonique.
- Une traduction absente peut revenir à l'anglais avec indication appropriée.
- Les imports doivent préserver la langue et la provenance de chaque texte.

## Alternatives considered

- Nom anglais comme clé : rejeté car mutable et ambigu.
- Français comme unique représentation : rejeté car limite les sources et l'internationalisation.
- Dupliquer une carte par langue : rejeté car fragmente l'identité métier.


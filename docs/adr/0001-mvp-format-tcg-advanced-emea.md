# ADR-0001 — Format MVP : TCG Advanced EMEA

## Status

Accepted

## Context

Le produit doit valider et générer des decks contre un référentiel de règles non ambigu. Supporter plusieurs formats dès le MVP multiplierait les pools de cartes, banlists, règles, tests et choix d'interface avant validation de la valeur produit.

## Decision

Le MVP prend en charge uniquement **Yu-Gi-Oh! TCG Advanced EMEA**.

Le format est explicite dans chaque opération mais aucun sélecteur multi-format n'est affiché au MVP. Le domaine expose un contrat de règles de format afin que Master Duel, OCG, d'autres régions, des formats historiques ou d'anciens jeux puissent être ajoutés ultérieurement sans réécrire le cœur.

## Consequences

- Le pool de cartes et la légalité sont évalués pour le territoire EMEA.
- Les règles et tests MVP peuvent être bornés à un seul référentiel.
- La notion de format reste un concept de premier rang et non une constante dispersée.
- L'ajout d'un format exige un adaptateur, ses données versionnées et sa suite de conformité.

## Alternatives considered

- TCG uniquement avec architecture couplée : plus simple à court terme, mais coûteux à étendre.
- Master Duel uniquement : pool et banlist spécifiques, sans Side Deck standard.
- Plusieurs formats au MVP : rejeté pour sa complexité produit, données et tests.


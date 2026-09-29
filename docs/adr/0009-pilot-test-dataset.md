# ADR-0009 — Jeu de données et archétypes pilotes pour valider le moteur

## Status

Proposed

## Context

Le développement et les tests ont besoin d'un petit ensemble de cartes, relations, stratégies et résultats attendus pouvant être revu humainement. L'expression « archétypes pilotes » peut cependant être interprétée à tort comme une limitation du catalogue ou des fonctionnalités accessibles aux utilisateurs.

Le périmètre de production vise toutes les cartes couvertes par TCG Advanced EMEA que la source et les droits applicables permettent d'exposer. Les utilisateurs peuvent construire librement un deck, avec ou sans archétype, et les recommandations doivent rechercher des candidats dans l'ensemble du catalogue légal.

## Decision

Proposition soumise à validation : définir un jeu de données de test réduit et représentatif d'environ dix familles, comprenant des archétypes, cartes génériques et cas sans archétype, exclusivement pour :

- les fixtures d'import et de validation ;
- les tests de détection d'archétype ;
- les relations, rôles, synergies et conflits connus ;
- les golden decks et scénarios de génération ;
- les tests de recommandation et d'optimisation ;
- la couverture de mécaniques et stratégies différentes.

Le choix exact, la taille et les cas représentés restent à valider. Ce dataset ne limite jamais le catalogue de production, la recherche utilisateur, la construction manuelle, le pool de candidats ou les archétypes utilisables.

La sélection candidate, sa matrice de couverture, ses lacunes et ses cas indépendants sont documentés dans [`docs/specifications/d009-pilot-dataset-proposal.md`](../specifications/d009-pilot-dataset-proposal.md). Tant que cette proposition n'est pas explicitement approuvée, le présent ADR reste `Proposed`.

## Consequences

- Les fixtures restent petites, déterministes, versionnées et révisables.
- Le pipeline de production est testé sur ces données mais conçu pour le catalogue général.
- Les labels « supporté/non supporté » ne sont pas appliqués aux archétypes de production.
- Une recommandation peut reconnaître que les relations stratégiques d'une carte sont inconnues tout en permettant sa recherche, son ajout et sa validation.
- Les tests doivent inclure des decks sans archétype et des cartes hors du dataset pilote.

## Alternatives considered

- Limiter le MVP aux archétypes pilotes : rejeté car contraire à la vision produit.
- Annoter exhaustivement toutes les interactions avant le MVP : rejeté comme irréaliste.
- N'utiliser aucun dataset représentatif : rejeté car les résultats du moteur seraient difficiles à valider et à reproduire.

# ADR-0009 — Jeu de données et archétypes pilotes pour valider le moteur

## Status

Accepted

## Context

Le développement et les tests ont besoin d'un petit ensemble de cartes, relations, stratégies et résultats attendus pouvant être revu humainement. L'expression « archétypes pilotes » peut cependant être interprétée à tort comme une limitation du catalogue ou des fonctionnalités accessibles aux utilisateurs.

Le périmètre de production vise toutes les cartes couvertes par TCG Advanced EMEA que la source et les droits applicables permettent d'exposer. Les utilisateurs peuvent construire librement un deck, avec ou sans archétype, et les recommandations doivent rechercher des candidats dans l'ensemble du catalogue légal.

## Decision

Le dataset pilote est constitué des dix familles de fixtures suivantes, sans classement de puissance :

1. Blue-Eyes ;
2. Branded / Despia / Fallen of Albaz ;
3. Swordsoul / Tenyi ;
4. Purrely ;
5. Salamangreat ;
6. D/D / D/D/D / Dark Contract ;
7. Drytron ;
8. Labrynth ;
9. Sky Striker ;
10. Floowandereeze.

Ces familles sont utilisées exclusivement pour :

- les fixtures d'import et de validation ;
- les tests de détection d'archétype ;
- les relations, rôles, synergies et conflits connus ;
- les golden decks et scénarios de génération ;
- les tests de recommandation et d'optimisation ;
- la couverture de mécaniques et stratégies différentes.

Elles sont complétées par des micro-fixtures indépendantes : deck générique sans archétype, carte légale hors dataset, petits moteurs compatibles/incompatibles, carte sans relation connue, cartes génériques, frontières de taille, limites de copies, snapshots temporels, légalité EMEA, localisation, import YDK invalide et requêtes LLM adversariales.

Chaque fixture épingle `FormatSnapshot`, `BanlistSnapshot`, `CatalogueSnapshot` et ses résultats attendus. Une golden decklist ne peut être déclarée valide qu'après contrôle de l'existence des cartes, de leur sortie/légalité EMEA, de leur zone, de leurs quantités cumulées et de la banlist du snapshot. Une évolution crée une nouvelle version ; elle ne modifie pas rétroactivement un résultat historique.

La sélection validée, sa matrice de couverture, ses lacunes et ses micro-fixtures sont documentées dans [`docs/specifications/d009-pilot-dataset-proposal.md`](../specifications/d009-pilot-dataset-proposal.md).

Les frontières sont contractuelles :

- **Production catalogue :** toutes les cartes du périmètre TCG Advanced EMEA défini que les sources et droits permettent d'exposer.
- **Pilot dataset :** les dix familles ci-dessus, uniquement pour les tests.
- **User deckbuilding :** accès libre au catalogue, sans archétype obligatoire.
- **AI recommendation :** candidats récupérés dans l'ensemble du catalogue légal, jamais uniquement dans le dataset pilote.

Ce dataset n'est ni une whitelist, ni la liste des archétypes supportés. Une carte absente du dataset reste recherchable, ajoutable, importable, exportable, validable et éligible à la récupération de candidats si elle appartient au catalogue légal.

## Consequences

- Les fixtures restent petites, déterministes, versionnées et révisables.
- Le pipeline de production est testé sur ces données mais conçu pour le catalogue général.
- Les labels « supporté/non supporté » ne sont pas appliqués aux archétypes de production.
- Une recommandation peut reconnaître que les relations stratégiques d'une carte sont inconnues tout en permettant sa recherche, son ajout et sa validation.
- Les tests doivent inclure des decks sans archétype et des cartes hors du dataset pilote.
- La composition physique, les annotations et les golden decklists seront créées uniquement pendant les tâches autorisées de Phase 1.

## Alternatives considered

- Limiter le MVP aux archétypes pilotes : rejeté car contraire à la vision produit.
- Annoter exhaustivement toutes les interactions avant le MVP : rejeté comme irréaliste.
- Ajouter un archétype pour chaque sous-mécanique : rejeté au profit de micro-fixtures ciblées.
- N'utiliser aucun dataset représentatif : rejeté car les résultats du moteur seraient difficiles à valider et à reproduire.

# ADR-0005 — Périmètre fonctionnel du MVP

## Status

Accepted

## Context

Le produit doit démontrer la valeur d'un deck builder fiable sans inclure prématurément comptes, métagame, simulation ou couverture exhaustive des interactions.

## Decision

Le MVP expose le catalogue général des cartes couvertes par le format et disponibles dans la source validée. Il permet de rechercher et consulter librement les cartes, filtrer selon les propriétés réellement disponibles, construire entièrement un deck à la main sans sélectionner d'archétype, imposer/exclure/verrouiller des cartes, générer une proposition principale lorsque demandé, éditer Main/Extra/Side, valider la légalité, analyser ratios et rôles, utiliser uniquement les synergies explicitement connues, proposer des échanges ajout/retrait, expliquer les résultats avec un LLM, importer/exporter en YDK et conserver localement un brouillon sans compte.

Le Main Deck est généré ; l'Extra Deck l'est lorsque pertinent. Le Side Deck est éditable et validé, mais son optimisation automatique est différée. Toute optimisation requiert une action explicite de l'utilisateur.

Le système reste utilisable sans LLM pour recherche, édition, validation, génération, métriques et optimisations déterministes. L'analyse part du deck réellement construit. La récupération de candidats recherche dans l'ensemble du catalogue légal, puis applique contraintes, rôles, compatibilités et relations connues. Le LLM comprend et explique ; il ne recherche pas les cartes dans sa mémoire, ne crée pas de fait, ne tranche pas la légalité, ne choisit pas la banlist et ne modifie pas le deck.

Les « archétypes pilotes » désignent uniquement un jeu de test représentatif pour fixtures, annotations initiales, golden decks et évaluations. Ils ne constituent ni une liste de contenus supportés, ni une condition d'utilisation, ni une limite du catalogue ou du moteur.

Sont hors MVP : comptes et cloud, historique utilisateur, profils spécialisés, variantes multiples, tournois/métagame, partage/social/marketplace, simulation, rulings exhaustifs, modèle propriétaire, mobile natif et interface multi-format.

## Consequences

- Une proposition unique et explicable suffit au MVP.
- La construction manuelle libre constitue un parcours principal autonome.
- Une synergie absente des données est déclarée inconnue, jamais inventée.
- L'archétype est une dimension de recherche et d'analyse parmi d'autres, jamais la racine de l'architecture.
- Les contrats préservent la possibilité de variantes futures sans les implémenter.
- Le fonctionnement sans IA est un critère d'acceptation, pas seulement un fallback technique.

## Alternatives considered

- Chatbot générant directement une liste : rejeté pour manque de fiabilité.
- Plusieurs variantes et profils dès le MVP : différés pour limiter le scoring et l'évaluation.
- Comptes obligatoires : rejetés car inutiles pour tester la valeur principale.


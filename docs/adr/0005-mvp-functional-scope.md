# ADR-0005 — Périmètre fonctionnel du MVP

## Status

Accepted

## Context

Le produit doit démontrer la valeur d'un deck builder fiable sans inclure prématurément comptes, métagame, simulation ou couverture exhaustive des interactions.

## Decision

Le MVP permet de rechercher et consulter les cartes, filtrer par type et archétype, choisir un archétype pilote, imposer/exclure/verrouiller des cartes, générer une proposition principale, éditer Main/Extra/Side, valider la légalité, analyser ratios et rôles, utiliser uniquement les synergies explicitement connues, proposer des échanges ajout/retrait, expliquer les résultats avec un LLM, importer/exporter en YDK et conserver localement un brouillon sans compte.

Le Main Deck est généré ; l'Extra Deck l'est lorsque pertinent. Le Side Deck est éditable et validé, mais son optimisation automatique est différée. Toute optimisation requiert une action explicite de l'utilisateur.

Le système reste utilisable sans LLM pour recherche, édition, validation, génération, métriques et optimisations déterministes. Le LLM comprend et explique ; il ne crée pas de fait, ne tranche pas la légalité, ne choisit pas la banlist et ne modifie pas le deck.

Sont hors MVP : comptes et cloud, historique utilisateur, profils spécialisés, variantes multiples, tournois/métagame, partage/social/marketplace, simulation, rulings exhaustifs, modèle propriétaire, mobile natif et interface multi-format.

## Consequences

- Une proposition unique et explicable suffit au MVP.
- Une synergie absente des données est déclarée inconnue, jamais inventée.
- Les contrats préservent la possibilité de variantes futures sans les implémenter.
- Le fonctionnement sans IA est un critère d'acceptation, pas seulement un fallback technique.

## Alternatives considered

- Chatbot générant directement une liste : rejeté pour manque de fiabilité.
- Plusieurs variantes et profils dès le MVP : différés pour limiter le scoring et l'évaluation.
- Comptes obligatoires : rejetés car inutiles pour tester la valeur principale.


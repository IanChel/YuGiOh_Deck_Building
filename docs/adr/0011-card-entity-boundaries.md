# ADR-0011 — Identité Card et frontières du catalogue

## Status

Proposed

## Context

`Card` doit être la référence centrale du catalogue tout en restant stable lorsque les noms, textes, traductions, sources, disponibilités ou annotations évoluent. Placer toutes ces données dans une ligne unique créerait un couplage entre identité, snapshot, région, légalité et stratégie.

## Decision

Proposition soumise à validation :

- utiliser un `card_id` numérique interne et immuable comme référence métier ;
- placer les identifiants catalogue, passcodes et identifiants fournisseurs dans des références externes multiples, sans en faire l'identité interne ;
- porter les propriétés intrinsèques dans un `CardSnapshot` lié à un `CatalogueSnapshot`, avec au plus une représentation logique par paire carte/snapshot ;
- laisser ouverte l'implémentation physique des snapshots (copie, partage ou déduplication) ;
- placer tous les noms et textes linguistiques dans `CardLocalization`, y compris l'anglais canonique du MVP ; le français est facultatif avec fallback anglais explicite et aucune traduction LLM n'est officielle sans validation et provenance ;
- limiter `CardArchetypeMembership` à l'appartenance structurelle sourcée ; différer la référence textuelle explicite et conserver les associations stratégiques dans les relations D-010 ;
- rattacher `CardAvailability` au `CatalogueSnapshot`, indépendamment de toute banlist ;
- traiter les images comme des ressources externes référencées et versionnées, sans préjuger D-008 ;
- maintenir banlists, annotations D-010, relations et calculs de recommandation hors de `Card` ;
- distinguer pour ATK/DEF une valeur numérique, `?` et la non-applicabilité, sans multiplier les types ;
- différer `identity_status` et `merged_into_card_id` jusqu'à la définition d'un workflow fiable de fusion/dédoublonnage.

La spécification complète et les questions ouvertes sont décrites dans [`docs/specifications/d011-card-entity.md`](../specifications/d011-card-entity.md).

## Consequences

- Les decks, banlists et relations référencent une identité stable.
- Les corrections de catalogue créent un nouveau snapshot sans altérer les résultats historiques.
- La recherche bilingue résout plusieurs noms vers le même `card_id`.
- L'existence d'une carte reste distincte de sa disponibilité EMEA et de son statut de banlist.
- La disponibilité régionale peut différer entre deux snapshots sans réécriture historique.
- Les cartes sans annotations stratégiques restent recherchables, constructibles et validables.
- Le modèle physique comportera plusieurs structures, mais celles-ci ne sont pas encore décidées ni implémentées.

## Alternatives considered

- Utiliser l'identifiant YGOPRODeck comme clé primaire : rejeté dans la proposition car il couple le domaine au fournisseur.
- Utiliser le passcode comme identité universelle : rejeté car son applicabilité ne doit pas être supposée totale.
- Stocker toutes les langues et versions dans `Card` : rejeté car non reproductible et difficile à étendre.
- Ajouter légalité, banlist et tags comme colonnes de `Card` : rejeté car ces informations dépendent d'autres contextes.

# ADR-0012 — Localisation des cartes et fallback linguistique

## Status

Accepted

## Context

Le MVP utilise le français comme langue d'interface prioritaire, mais la couverture française peut être absente ou partielle. Les noms et textes doivent rester reproductibles par snapshot, permettre une recherche anglaise et française et ne jamais transformer une traduction générée ou non vérifiée en donnée officielle.

## Decision

- rattacher conceptuellement chaque `CardLocalization` à une `Card`, une langue et un `CatalogueSnapshot` ;
- exiger la cohérence avec le `CardSnapshot` de la même paire carte/snapshot sans imposer de stockage physique ;
- utiliser l'anglais comme langue canonique du MVP et rendre le français facultatif ;
- placer les noms et textes officiels dans `CardLocalization` et les anciens noms ou variantes validées dans `CardNameAlias` ;
- appliquer un fallback de lecture français → anglais, champ par champ, sans modifier les données persistées ;
- exposer la langue réellement servie et les états partiels ou manquants ;
- limiter les états métier à `COMPLETE`, `PARTIAL` et `UNAVAILABLE`, toute absence technique ou incohérence restant une anomalie de qualité ou d'intégrité ;
- rechercher sur les noms officiels et alias validés des deux langues, avec normalisation dérivée et ambiguïtés explicites ;
- accepter comme alias les translittérations validées et variantes de sources identifiées, en excluant du MVP les surnoms communautaires, traductions libres et variantes non validées ;
- conserver localisations, alias, provenance et règles dérivées dans l'historique des snapshots ;
- résoudre les imports et exports YDK par identifiants, indépendamment des noms et de la langue ;
- maintenir toute suggestion LLM hors des localisations publiées jusqu'à une revue humaine documentée ;
- permettre la publication d'une localisation humaine validée avec provenance explicite, sans la qualifier d'officielle lorsque sa source ne permet pas cette qualification.

La spécification complète et les décisions validées sont décrites dans [`docs/specifications/d012-card-localization.md`](../specifications/d012-card-localization.md).

## Consequences

- Un ancien résultat conserve les noms et textes du snapshot qui l'a produit.
- L'interface française reste utilisable malgré une couverture incomplète sans inventer de traduction.
- Le frontend consomme une résolution cohérente et n'implémente pas lui-même le fallback.
- Deux formes linguistiques peuvent résoudre vers le même `card_id`.
- Une collision de nom ou d'alias reste explicite et ne fusionne pas les identités.
- L'ajout futur d'une langue ne nécessite pas de modifier la structure conceptuelle.

## Alternatives considered

- Stocker le nom anglais directement dans `CardSnapshot` : écarté, car la frontière linguistique serait incohérente et dupliquée.
- Dupliquer l'anglais dans une fausse localisation française lorsque le français manque : écarté, car le fallback deviendrait indiscernable d'une traduction réelle.
- Résoudre automatiquement les collisions par correspondance approximative : écarté, car une mauvaise identité de carte serait silencieusement sélectionnée.
- Utiliser une traduction LLM comme donnée officielle : écarté sans revue, provenance et validation humaine.
- Créer dès le MVP une taxonomie exhaustive des langues et variantes régionales : écarté comme surmodélisation.


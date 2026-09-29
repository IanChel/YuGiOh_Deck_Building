# ADR-0013 — Types contrôlés du catalogue

## Status

Accepted

## Context

`CardSnapshot` doit représenter les propriétés intrinsèques des cartes sans dépendre de chaînes libres, de catégories combinatoires ou du vocabulaire particulier d'un fournisseur. Le modèle doit couvrir les cartes hybrides, distinguer propriétés non applicables et données inconnues, et rester interprétable dans les anciens snapshots.

## Decision

- limiter la catégorie principale à `MONSTER`, `SPELL` ou `TRAP` ;
- représenter Normal, Effect, Ritual, Fusion, Synchro, Xyz, Link et Pendulum dans un ensemble contrôlé conceptuellement nommé `monster_classifications`, sans supposer qu'il s'agit toujours de « frames » au même sens ;
- représenter Tuner, Flip, Gemini, Union, Spirit et Toon dans un ensemble contrôlé séparé de capacités orthogonales ;
- gérer la race par un registre normatif versionné issu d'une source officielle TCG, distinct des valeurs simplement observées chez un fournisseur ;
- contrôler les attributs de monstre et les propriétés Spell/Trap dans des domaines distincts ; `SPELL.NORMAL`, `TRAP.NORMAL` et `MONSTER.NORMAL` restent sémantiquement qualifiés et non ambigus ;
- maintenir Level, Rank et Link Rating comme trois propriétés séparées ;
- représenter les Link markers par un ensemble de huit directions contrôlées ;
- conserver `scale_left` et `scale_right` séparément, le texte Pendulum restant dans `CardLocalization` ;
- distinguer pour ATK/DEF une valeur numérique, `PRINTED_UNKNOWN`, `NOT_APPLICABLE` et `UNKNOWN` ;
- traiter une donnée invalide comme une anomalie de qualité, sans valeur métier `INVALID` ;
- laisser toute valeur externe inconnue non mappée et en quarantaine jusqu'à un mapping versionné ou une revue humaine validée, bloquer le fait qui la requiert et interdire tout fallback arbitraire vers `OTHER` ;
- versionner l'évolution du vocabulaire sans réécrire les anciens `CatalogueSnapshot`.

La spécification complète et les décisions validées sont décrites dans [`docs/specifications/d013-controlled-catalog-types.md`](../specifications/d013-controlled-catalog-types.md).

## Consequences

- Les filtres reposent sur des concepts stables plutôt que sur les libellés d'une API.
- Les cartes hybrides ne nécessitent pas une catégorie combinatoire supplémentaire.
- Une propriété absente, inconnue ou non applicable conserve un sens distinct.
- Une nouveauté externe est mise en quarantaine jusqu'à une extension contrôlée du vocabulaire.
- Une version historique reste interprétable après l'ajout d'une nouvelle valeur.
- Le dictionnaire et les mappings de source devront préciser bornes, règles de complétude et valeurs officielles.

## Alternatives considered

- Un type unique combinatoire par monstre : écarté car fragile face aux cartes hybrides et aux futures combinaisons.
- Une colonne booléenne par classification ou capacité : écartée car elle impose une évolution structurelle à chaque nouveauté.
- Réutiliser directement les chaînes YGOPRODeck : écarté car le domaine interne ne doit pas dépendre du fournisseur.
- Employer `NULL` pour inconnu et non applicable : écarté car les deux situations ont des conséquences différentes.
- Ajouter `OTHER` ou `INVALID` au vocabulaire métier : écarté car cela masquerait un mapping absent ou une anomalie de qualité.


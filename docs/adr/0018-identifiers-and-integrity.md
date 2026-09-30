# ADR-0018 — Identifiants et intégrité du modèle logique

## Status

Accepted

## Context

Le domaine combine des identités durables, des représentations versionnées, des publications immuables et des identifiants externes susceptibles de changer ou d'entrer en conflit. Sans règles explicites, une correction pourrait réécrire le passé, une clé fournisseur pourrait devenir une identité canonique ou un résultat historique pourrait se résoudre vers une version courante différente.

## Decision

- utiliser des identifiants internes générés, immuables et non recyclables pour les entités durables ;
- donner une identité immuable propre à chaque snapshot, capture, dataset publié, fait importé, annotation et relation ;
- identifier logiquement les représentations contextuelles par des combinaisons entité + snapshot + dimensions pertinentes ;
- maintenir les contraintes composites même si une future implémentation ajoute des clés substitutives ;
- conserver les identifiants externes comme assertions typées, sourcées et versionnées, jamais comme identité canonique de `Card` ;
- interdire les mappings externes concurrents sur une même période sans résolution explicite ;
- définir les codes métier comme clés candidates non recyclables, distinctes des identifiants internes ;
- épingler toute référence historique vers la version exacte utilisée, sans résolution implicite vers la dernière version ;
- imposer la cohérence des combinaisons de snapshots avant publication ou analyse canonique ;
- préserver toute identité et version référencée par l'historique, même après archivage ou supersession ;
- mettre collisions, références pendantes et violations d'unicité en quarantaine sans réparation implicite ;
- conserver la lignée D-017 permettant de remonter d'un résultat aux publications, faits, captures et mappings.

La spécification complète et les décisions validées sont décrites dans [`docs/specifications/d018-identifiers-and-integrity.md`](../specifications/d018-identifiers-and-integrity.md).

## Consequences

- Un changement de nom, contenu ou fournisseur ne change pas automatiquement l'identité métier.
- Les contraintes composites restent visibles et vérifiables dans le modèle logique.
- Les données historiques ne dépendent jamais d'un alias `latest`.
- Les collisions externes demandent une décision auditée et ne fusionnent pas des cartes.
- Les objets historiques restent conservés aussi longtemps qu'une publication ou un résultat les référence.
- Une future conception physique devra traduire ces invariants en clés, contraintes et index sans en modifier le sens.

## Alternatives considered

- Utiliser le passcode ou l'identifiant YGOPRODeck comme clé de `Card` : écarté car externe, incomplet et susceptible d'évoluer.
- Utiliser le nom comme identité : écarté car localisé, corrigible et non nécessairement unique.
- Donner uniquement des clés substitutives à toutes les associations : écarté car cela masquerait les doublons contextuels.
- Imposer des clés composites physiques partout : écarté car D-018 définit l'identité logique, pas la stratégie de stockage.
- Résoudre les snapshots historiques vers leur successeur : écarté car non reproductible.
- Réutiliser les identifiants archivés : écarté car les anciennes références deviendraient ambiguës.
- Supprimer physiquement les versions supersédées : écarté car la provenance et l'audit seraient rompus.
- Réparer automatiquement une collision externe : écarté car elle peut révéler une erreur de source ou d'identité.

## Related decisions

- D-003, D-004 et D-007 pour les sources, identifiants internes et contextes reproductibles.
- D-010 à D-014 pour les assertions, cartes, localisations, vocabulaires et archétypes.
- D-015 et D-016 pour les identités temporelles de format et banlist.
- D-017 pour la provenance complète et les publications immuables.


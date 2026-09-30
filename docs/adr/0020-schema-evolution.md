# ADR-0020 — Évolution du schéma sans réécriture des snapshots

## Status

Proposed

## Context

Le modèle évoluera après la publication de snapshots et de résultats historiques. Sans stratégie explicite, une migration technique pourrait réinterpréter des valeurs anciennes, modifier des identifiants, appliquer un mapping courant à une publication passée ou rendre un snapshot illisible.

## Proposed decision

- distinguer les versions de schéma, données métier, snapshots, publications, mappings et vocabulaires ;
- rattacher chaque publication à un contrat d'interprétation et une version de schéma immuables ;
- préserver le contenu et la sémantique observables de tout snapshot publié ;
- classer chaque évolution comme additive, rétrocompatible, nécessitant adaptation ou incompatible ;
- autoriser les backfills techniques uniquement lorsqu'ils sont déterministes, traçables et sémantiquement équivalents ;
- interdire les changements destructifs en place sur l'historique publié ;
- conserver les anciennes valeurs de vocabulaire, mappings et relations pour leur contexte historique ;
- produire une nouvelle publication et, si nécessaire, un nouveau snapshot lorsqu'une nouvelle interprétation métier est introduite ;
- préserver tous les identifiants internes et codes historiques pendant les migrations ;
- exiger des migrations vérifiables, sans état partiel activé comme canonique ;
- distinguer rollback de code, schéma, publication et supersession de snapshot ;
- permettre des transitions progressives avec source canonique et matrice de compatibilité explicites ;
- tester la reproductibilité historique, la provenance et les invariants D-018 avant activation ;
- différer le choix de l'outil et de l'implémentation des migrations.

La proposition complète et ses questions de validation sont décrites dans [`docs/specifications/d020-schema-evolution.md`](../specifications/d020-schema-evolution.md).

## Consequences

- Un ancien snapshot peut nécessiter un lecteur ou adaptateur historique maintenu.
- Une évolution apparemment simple doit être évaluée selon son impact sémantique, pas uniquement structurel.
- Les vocabulaires et mappings anciens restent disponibles aussi longtemps que des publications les référencent.
- Les migrations destructives exigent une nouvelle représentation et une conservation reconstructible de l'ancienne.
- Les déploiements devront déclarer leurs compatibilités et valider les backfills avant bascule.
- La maintenance historique augmente, mais empêche les réinterprétations silencieuses et protège l'audit.

## Alternatives considered

- Migrer toutes les données historiques vers le modèle courant et supprimer l'ancien : écarté car le sens et la reproductibilité pourraient changer.
- Créer un nouveau snapshot pour chaque migration technique : écarté car cela confond structure et publication métier.
- Appliquer le mapping courant lors de toute lecture : écarté car une publication passée serait réinterprétée.
- Modifier le sens d'une valeur contrôlée existante : écarté ; une nouvelle version/valeur est requise.
- Autoriser les migrations partielles visibles : écarté car l'état canonique deviendrait incohérent.
- Imposer systématiquement double écriture : différé, car son coût et son risque dépendent de l'évolution.
- Choisir Alembic dès maintenant : différé jusqu'à la conception physique et à l'outillage du projet.

## Related decisions

- D-017 pour les mappings, publications et provenance.
- D-018 pour les identifiants et références immuables.
- D-019 pour les chemins d'accès indépendants de la technologie physique.
- D-007 et D-011 à D-016 pour les snapshots métier historiques.


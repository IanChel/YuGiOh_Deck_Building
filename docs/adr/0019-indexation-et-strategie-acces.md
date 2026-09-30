# ADR-0019 — Indexation logique et stratégie d'accès

## Status

Accepted

## Context

Le modèle versionné doit servir la recherche multilingue, les filtres contrôlés, les parcours d'archétype, la future validation de légalité et l'audit de provenance. Sans stratégie d'accès explicite, une implémentation pourrait mélanger les snapshots, résoudre par des noms instables, multiplier les requêtes par carte ou choisir prématurément une technologie de recherche.

## Decision

- définir les index logiques à partir de chemins d'accès fonctionnels, sans choisir leur réalisation physique ;
- ancrer toute recherche de contenu de carte dans un `CatalogueSnapshot` et une langue ;
- séparer recherche exacte, préfixe, partielle, alias et texte d'effet ;
- appliquer le fallback français → anglais uniquement entre localisations du même snapshot ;
- conserver l'origine de chaque correspondance nominale ou textuelle ;
- offrir des filtres structurés sur les vocabulaires D-013 et des parcours bidirectionnels de membership D-014 ;
- prévoir des accès par lot pour la future validation de deck et de légalité ;
- interpréter l'absence de `BanlistEntry` uniquement lorsque le snapshot de banlist est complet ;
- parcourir la provenance D-017 dans les deux sens par identifiants immuables ;
- combiner candidats textuels et dimensions structurées dans un même contexte versionné ;
- fournir les accès nécessaires aux contraintes d'unicité D-018 sans indexer chaque champ par défaut ;
- exiger des identifiants de snapshots explicites pour les requêtes historiques ;
- imposer un tri total déterministe pour la pagination ;
- différer tout choix B-tree, GIN, GiST, trigramme, full-text ou moteur externe jusqu'à mesures réelles.

La spécification complète et les décisions validées sont décrites dans [`docs/specifications/d019-indexation-et-strategie-acces.md`](../specifications/d019-indexation-et-strategie-acces.md).

## Consequences

- Les besoins fonctionnels pourront être traduits et testés indépendamment de la technologie choisie.
- Les recherches et jointures historiques ne mélangeront pas les snapshots.
- Le futur validateur pourra résoudre un deck par lots plutôt que par recherches nominales répétées.
- Les collisions de noms et alias resteront visibles et déterministes.
- Les recherches textuelles coûteuses pourront recevoir une solution spécialisée seulement si les mesures le justifient.
- La conception physique devra démontrer la couverture des chemins critiques et mesurer le coût de maintenance des index.

## Alternatives considered

- Choisir immédiatement les index PostgreSQL : différé car les volumes, requêtes et sélectivités ne sont pas encore mesurés.
- Installer un moteur de recherche externe : différé faute de besoin démontré et pour éviter une nouvelle dépendance.
- Utiliser une recherche textuelle unique pour noms, alias et effets : écarté car les sémantiques et priorités sont différentes.
- Rechercher un membership d'archétype dans le texte : écarté car D-014 définit une relation structurelle dédiée.
- Résoudre la légalité carte par carte avec des accès unitaires : écarté comme chemin imprévisible pour les decks.
- Indexer chaque colonne : écarté car chaque accès doit avoir une justification fonctionnelle.
- Paginer uniquement par nom : écarté car les collisions rendraient l'ordre instable.
- Utiliser implicitement le dernier snapshot : écarté conformément à D-007 et D-018.

## Related decisions

- D-012 pour recherche multilingue, alias et fallback.
- D-013/D-014 pour filtres contrôlés et archétypes.
- D-015/D-016 pour jointures de légalité.
- D-017 pour la provenance.
- D-018 pour les contraintes et références immuables.


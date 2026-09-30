# D-019 — Indexation logique et stratégie d'accès

## Status

**Accepted — validée le 2026-09-30.**

Cette spécification définit les chemins d'accès fonctionnels du modèle logique. Elle ne choisit aucun type d'index PostgreSQL, moteur de recherche, partitionnement, SQL, migration ou implémentation physique.

## 1. Scope

D-019 décrit les accès nécessaires pour :

- résoudre les cartes par identité, nom et alias ;
- rechercher dans plusieurs langues sans mélanger les snapshots ;
- filtrer sur les propriétés contrôlées D-013 ;
- parcourir les appartenances d'archétype dans les deux sens ;
- préparer les jointures de légalité et de validation de deck ;
- remonter la provenance D-017 ;
- combiner recherche textuelle et filtres structurés ;
- vérifier les unicités D-018 ;
- exécuter des requêtes historiques sur des versions exactes ;
- paginer et trier de façon déterministe.

La spécification exprime **quels accès doivent rester efficaces et prévisibles**, pas comment un moteur de données les réalisera.

## 2. Definitions

### Index logique

Contrat indiquant qu'un ensemble de clés doit permettre une résolution, un filtrage, une jointure, un contrôle d'unicité ou un parcours ordonné. Il peut être réalisé plus tard par une contrainte, un index, une matérialisation, un moteur spécialisé ou une combinaison de mécanismes.

### Chemin d'accès

Entrées connues, contexte requis, résultat attendu et invariants d'une opération fréquente ou critique.

### Recherche exacte

Comparaison d'une forme normalisée et versionnée avec un nom ou alias complet. Elle n'implique pas l'unicité du résultat.

### Recherche préfixe ou partielle

Recherche sur le début ou une partie contrôlée d'un nom/alias. Elle reste distincte d'une recherche dans le texte d'effet.

### Recherche textuelle

Recherche de termes dans un contenu linguistique comme `effect_text`, `normal_text` ou `pendulum_text`. Elle n'est activée que pour un besoin explicite et ne devient pas automatiquement un moteur full-text.

### Filtrage structuré

Sélection sur une valeur canonique D-013, une appartenance D-014, une disponibilité D-011/D-015, une restriction D-016 ou une annotation D-010.

### Clé de tri déterministe

Suite de valeurs ordonnant totalement les résultats. Une égalité de nom est toujours départagée par une identité stable.

## 3. Access patterns

Les chemins principaux sont regroupés par finalité :

| Groupe | Entrées principales | Résultat |
|---|---|---|
| Identité | identifiant interne | Entité stable exacte |
| Représentation | identité + snapshot | Version métier exacte |
| Nom/localisation | snapshot + langue + terme | Cartes candidates et origine de correspondance |
| Filtres catalogue | snapshot + valeurs contrôlées | Ensemble de `card_id` |
| Archétypes | snapshot + carte ou archétype | Memberships dans le sens demandé |
| Légalité | contexte exact + lot de cartes | Faits nécessaires à la future validation |
| Provenance | publication/fait/source | Lignée avant ou arrière |
| Historique | identifiants de snapshots | État exact sans résolution courante |
| Intégrité | signature d'unicité | Existence, collision ou contradiction |

Principes transversaux :

- le `catalogue_snapshot_id` fait partie de tout accès à une propriété versionnée de carte ;
- les jointures de lots utilisent les identifiants internes, jamais les noms ;
- un chemin historique reçoit les identifiants de versions au lieu de deviner « l'actuel » ;
- les résultats de recherche exposent pourquoi une carte correspond : nom officiel, alias, texte ou filtre ;
- les accès canoniques excluent explicitement les données rejetées ou en quarantaine.

## 4. Card search

### 4.1 Résolution par identité

- `card_id → Card` doit être un accès direct.
- `(card_id, catalogue_snapshot_id) → CardSnapshot` doit distinguer représentation trouvée, absente ou snapshot incompatible.
- Une résolution externe passe d'abord par l'espace de référence typé D-018, puis vers `card_id` ; elle ne cherche pas le nom comme solution silencieuse.

### 4.2 Recherche par nom

Les accès nominaux sont toujours ancrés dans un `CatalogueSnapshot` et une langue :

```text
(catalogue_snapshot_id, language_code, normalized_name)
  → zéro, une ou plusieurs correspondances
```

La forme normalisée est dérivée du nom original par une règle versionnée. Le nom original reste la valeur affichable et auditée. Une correspondance exacte normalisée ne prouve pas une identité unique : plusieurs cartes peuvent partager ou avoir partagé des formes identiques.

Les modes restent distincts :

- exact sur nom officiel ;
- préfixe sur nom officiel ;
- partiel sur nom officiel ;
- exact/préfixe/partiel sur alias ;
- recherche textuelle dans les champs de texte.

Le mode de recherche et l'origine de la correspondance sont conservés dans le résultat.

### 4.3 Recherche dans le texte

Une recherche textuelle indique :

- le `CatalogueSnapshot` ;
- la langue ;
- les champs interrogés (`effect_text`, `normal_text`, `pendulum_text`) ;
- le mode demandé ;
- les filtres structurés complémentaires.

Le moteur ne recherche pas implicitement tous les textes lorsqu'un utilisateur demande un nom. La nécessité d'un mécanisme spécialisé sera évaluée ultérieurement sur des usages et volumes réels.

### 4.4 Filtres contrôlés

Le système doit permettre de filtrer dans un snapshot précis sur :

- `card_category` ;
- `monster_classifications` ;
- `monster_abilities` ;
- attribut et race ;
- Level, Rank et Link Rating ;
- Link markers ;
- échelles Pendulum ;
- propriété Spell/Trap ;
- états et valeurs d'ATK/DEF.

Les valeurs multi-valuées utilisent une sémantique explicite : « contient au moins », « contient toutes » ou autre opérateur défini. Aucun filtre ne repose sur un libellé traduit.

## 5. Localization and aliases

### 5.1 Accès aux localisations

L'identité logique D-018 fournit l'accès :

```text
(card_id, catalogue_snapshot_id, language_code)
  → CardLocalization unique
```

Pour une demande française, le résolveur doit pouvoir obtenir dans le **même snapshot** :

- la localisation française ;
- la localisation anglaise canonique ;
- l'état de chaque champ ;
- la langue réellement servie après fallback D-012.

Le fallback ne consulte jamais une localisation anglaise d'un autre snapshot et ne persiste pas la valeur résolue comme donnée française.

### 5.2 Accès aux alias

`CardNameAlias` doit permettre :

- la recherche exacte par `(langue, alias normalisé, contexte de validité)` ;
- la recherche préfixe ou partielle lorsqu'elle est explicitement demandée ;
- le parcours de tous les alias d'une carte ;
- la distinction entre type d'alias, nom officiel et translittération validée ;
- l'exclusion des alias hors période/snapshot.

Un alias n'est pas globalement unique. Une collision retourne plusieurs candidats avec leur origine ; elle n'est jamais résolue arbitrairement. Un doublon exact pour la même carte, langue, type et période relève de l'intégrité D-018.

### 5.3 Ordre de résolution nominale

La politique de recherche peut prioriser :

1. nom officiel exact dans la langue demandée ;
2. nom officiel exact anglais si fallback autorisé ;
3. alias exact validé ;
4. préfixe/partiel demandé ;
5. correspondances textuelles uniquement si le mode les inclut.

Cette priorité ordonne les candidats ; elle ne supprime jamais une ambiguïté d'identité.

## 6. Archetype access

Les accès bidirectionnels obligatoires sont :

```text
(card_id, catalogue_snapshot_id)
  → archetype_id[]

(archetype_id, catalogue_snapshot_id)
  → card_id[]
```

Ils doivent également permettre :

- la résolution d'un archétype par localisation ou alias validé ;
- l'intersection membership + catégories/attributs/niveaux ;
- le filtrage de plusieurs archétypes avec sémantique « au moins un » ou « tous » ;
- l'audit de la provenance et du statut de revue du membership ;
- l'exclusion des memberships d'un autre snapshot.

La recherche du nom d'un archétype dans `card_text` n'est jamais un substitut au parcours de `CardArchetypeMembership`. Les relations `SUPPORTS` ou `ENABLES` restent des accès D-010 distincts.

## 7. Legality-validation joins

D-019 ne calcule pas la légalité. Elle définit les accès permettant au futur moteur d'obtenir, pour un lot de `card_id` et un contexte exact :

```text
CatalogueSnapshot
FormatSnapshot
RegionalScope
BanlistSnapshot
```

### 7.1 Parcours critique par lot

Pour chaque carte :

1. résoudre `Card` par `card_id` ;
2. vérifier l'existence de `(card_id, catalogue_snapshot_id)` ;
3. obtenir `CardAvailability` par `(card, catalogue snapshot, région, format)` ;
4. obtenir l'éventuelle `BanlistEntry` par `(banlist snapshot, card)` ;
5. vérifier la complétude du `BanlistSnapshot` avant d'interpréter une absence d'entrée ;
6. obtenir les caractéristiques D-013 nécessaires aux règles de format ;
7. vérifier les compatibilités temporelles et identitaires D-015/D-016.

Ces accès doivent fonctionner pour un lot de cartes afin d'éviter une résolution indépendante et imprévisible par carte.

### 7.2 États distingués

| Cas | Résultat d'accès attendu |
|---|---|
| Carte inconnue | Identité non résolue |
| Carte connue mais absente du catalogue | Représentation absente dans le snapshot demandé |
| Carte non disponible dans la région | `CardAvailability=UNAVAILABLE` |
| Disponibilité inconnue | `UNKNOWN`, aucune conclusion positive |
| Carte `FORBIDDEN` | Entrée exacte du snapshot |
| Carte `LIMITED` / `SEMI_LIMITED` | Entrée exacte et limite dérivée D-016 |
| Aucune entrée dans une banlist complète | Aucune restriction explicite |
| Banlist incomplète | Résultat indéterminé, pas `UNLIMITED` |
| Snapshots incompatibles | Anomalie explicite, parcours interrompu |
| Référence historique | Versions exactes uniquement |

Une carte présente dans un `CatalogueSnapshot` n'est pas supposée présente dans un autre.

## 8. Provenance access

La lignée doit être parcourable sans recherche textuelle :

```text
PublishedDataset
  → provenance links
  → ImportedFact
  → DataSnapshot
  → DataSource
```

Accès nécessaires :

- publication vers tous ses `DataSnapshot`, mappings et rapports d'intégrité ;
- valeur publiée vers ses `ImportedFact` contributeurs ;
- `ImportedFact` vers sa capture et sa source ;
- fait vers mapping/transformation/revue/anomalie ;
- capture vers tous ses faits ;
- fait ou capture vers les publications auxquelles ils ont contribué ;
- anomalie/quarantaine vers les éléments concernés et sa décision de résolution.

Les relations de lignée utilisent des identifiants immuables D-017/D-018. Un champ de justification textuel peut être affiché, mais ne sert jamais de clé de jointure.

## 9. Deck-validation access

Les futurs parcours de deck nécessitent :

- résolution de toutes les références de cartes vers des `card_id` ;
- comptage groupé des copies par `card_id` ;
- lecture par lot des `CardSnapshot` dans un catalogue épinglé ;
- lecture par lot des disponibilités régionales ;
- lecture par lot des entrées de banlist ;
- accès aux catégories, classifications et propriétés nécessaires ;
- regroupement par appartenance d'archétype dans le même snapshot ;
- détection de toute référence absente, ambiguë ou incompatible.

Le comptage appartient au futur validateur, mais les accès doivent lui permettre de travailler sur des identités stables plutôt que sur des noms ou identifiants externes.

D-019 ne définit ni taille de deck, zone, limite générale de copies ou ordre des validations.

## 10. Combined filtering

Une recherche combinée suit conceptuellement deux familles d'opérations :

1. produire un ensemble de candidats textuels si un terme est fourni ;
2. appliquer/intersecter les dimensions structurées dans le même `CatalogueSnapshot`.

Exemples requis :

- `MONSTER` + attribut ;
- archétype + Level/Rank/Link Rating ;
- archétype + catégorie principale ;
- terme dans un champ textuel + type contrôlé ;
- disponibilité dans un contexte + restriction d'une banlist précise ;
- plusieurs `FunctionalTag` applicables dans un contexte D-010.

Chaque filtre indique :

- son snapshot ou contexte versionné ;
- son opérateur ;
- sa provenance/canonicalité requise ;
- le comportement envers `UNKNOWN` et `NOT_APPLICABLE` ;
- la logique `AND`/`OR` pour les valeurs multiples.

Une annotation non revue ne satisfait pas un filtre canonique dur. Un candidat textuel d'un snapshot ne peut pas être combiné avec des propriétés d'un autre snapshot.

## 11. Uniqueness and integrity access

Les invariants D-018 nécessitent des chemins d'accès capables de vérifier rapidement :

- `(card_id, catalogue_snapshot_id)` ;
- `(card_id, catalogue_snapshot_id, language_code)` ;
- `(card_id, archetype_id, catalogue_snapshot_id)` ;
- `(banlist_snapshot_id, card_id)` ;
- `(card_id, catalogue_snapshot_id, regional_scope_id, format_id)` ;
- l'espace `(data_source_id, external_id_type, external_id)` et ses périodes ;
- les références de version dans une famille de snapshot/dataset/mapping ;
- la signature sémantique des annotations et relations canoniques ;
- les périodes de validité et liens de supersession.

Une contrainte d'unicité et un chemin de recherche peuvent parfois partager un mécanisme physique, mais D-019 ne le suppose pas. Un accès logique est justifié seulement s'il sert une vérification, une résolution, une jointure ou un usage fonctionnel identifié.

Il n'est pas proposé d'indexer chaque colonne isolément.

## 12. Snapshot/history access

Les accès historiques acceptent explicitement :

- `catalogue_snapshot_id` ;
- `format_snapshot_id` ;
- `banlist_snapshot_id` ;
- `regional_scope_id` ;
- `analyzed_at` et, lorsque nécessaire, une version de moteur.

Ils permettent :

- l'accès direct par identifiant de snapshot ;
- le parcours d'une famille par référence de version ou intervalle d'effet ;
- la navigation explicite des liens de supersession ;
- la vérification de compatibilité ;
- la reproduction d'un contexte épinglé.

Une vue « actuelle » pourra exister pour l'usage courant, mais elle est une résolution explicite et distincte. Elle ne remplace jamais les identifiants des requêtes ou résultats historiques.

## 13. Pagination and deterministic ordering

Tout résultat paginé doit conserver :

- le snapshot et la langue de recherche ;
- la règle de normalisation et le mode de correspondance ;
- les filtres appliqués ;
- un ordre total stable.

Ordre conceptuel recommandé pour une liste par nom :

```text
resolved_display_name_normalized
then card_id
```

`resolved_display_name_normalized` applique le fallback D-012 dans le même snapshot et reste une clé de tri dérivée, pas une identité. `card_id` départage les noms identiques.

Un classement par pertinence textuelle, s'il est ajouté, doit être déterministe pour une version donnée et utiliser `card_id` comme dernier départage. La stratégie de pagination par offset, curseur ou autre est différée.

Une pagination ne doit pas traverser implicitement un changement de snapshot ou de règle de classement.

## 14. Performance-critical paths

### Fréquents et interactifs

- résolution exacte de nom/alias en français et anglais ;
- chargement d'une carte et de sa localisation avec fallback ;
- recherche catalogue avec filtres structurés ;
- parcours archétype → cartes ;
- pagination et tri par nom.

### Critiques pour la validation

- résolution par lot de cartes d'un deck ;
- existence dans un `CatalogueSnapshot` ;
- disponibilité régionale par lot ;
- restriction de banlist par lot ;
- caractéristiques contrôlées nécessaires aux règles ;
- vérification de compatibilité des snapshots.

### Potentiellement coûteux

- recherche partielle large sur noms et alias ;
- recherche dans les textes d'effet ;
- combinaisons de nombreux filtres multi-valués ;
- filtres par plusieurs tags contextuels ;
- audit complet de provenance et parcours inverse sur de nombreuses publications.

### Hors chemin interactif principal

- rapports d'intégrité exhaustifs ;
- exploration complète des lignées historiques ;
- recalculs et validations de publication ;
- recherches administratives sur la quarantaine.

Aucun objectif chiffré ni mécanisme physique n'est fixé avant mesures sur des données représentatives.

## 15. Deferred physical indexing decisions

Sont explicitement différés :

- B-tree, hash ou autre index généraliste ;
- GIN, GiST et trigrammes ;
- full-text search PostgreSQL ;
- Elasticsearch, OpenSearch ou autre moteur externe ;
- index fonctionnels ou expressions spécifiques ;
- index partiels ou couvrants ;
- partitionnement ;
- vues matérialisées ;
- dénormalisation ;
- stratégie physique de pagination.

Une décision physique future devra s'appuyer sur :

- volumes réels ;
- sélectivité des dimensions ;
- fréquence des parcours ;
- latence mesurée ;
- coût de maintenance à l'import ;
- qualité attendue des recherches textuelles ;
- contraintes du SGBD effectivement retenu.

Elle devra démontrer qu'elle respecte les identités, snapshots et règles de provenance sans introduire de résolution implicite.

## 16. Edge cases

| Cas | Comportement attendu |
|---|---|
| Deux cartes au même nom | Retourner deux candidats ; tri stable par `card_id` |
| Nom français absent | Fallback anglais du même snapshot, langue servie exposée |
| Alias partagé par plusieurs cartes | Ambiguïté explicite, aucune sélection automatique |
| Ancien alias hors période | Exclu du contexte courant, accessible historiquement |
| Accent/apostrophe/tiret | Forme originale préservée, normalisation versionnée pour l'accès |
| Nom identique dans deux snapshots | Requêtes séparées par `catalogue_snapshot_id` |
| Membership ancien | Jamais mélangé au snapshot demandé |
| Carte sans entrée de banlist | « aucune restriction explicite » seulement si snapshot complet |
| Banlist et catalogue incompatibles | Jointure refusée comme anomalie |
| Annotation LLM non revue | Exclue des filtres canoniques durs |
| Pagination après changement de snapshot | Nouveau contexte de pagination requis |
| Recherche texte vide ou trop large | Validation fonctionnelle de requête à définir ; aucun scan implicite garanti |
| Valeur `UNKNOWN` | Ne satisfait pas un filtre positif |
| Provenance supprimée logiquement | Ancienne lignée reste accessible par identifiant historique |

## 17. Final decisions

1. Les recherches exactes sont ancrées dans un `CatalogueSnapshot` précis. Pour les localisations, la langue et la forme normalisée versionnée font partie du contexte ; toutes les collisions restent visibles et la valeur officielle demeure distincte de sa clé d'accès.
2. Le fallback français → anglais charge uniquement les localisations du même `CatalogueSnapshot`, s'applique champ par champ et expose la langue réellement servie. Aucun autre snapshot n'est consulté implicitement.
3. Les alias disposent de chemins de résolution distincts des noms officiels, respectent leur contexte de validité et retournent explicitement toute ambiguïté sans sélection arbitraire.
4. Les modes exact, préfixe, partiel et textuel restent distincts. Aucun mode n'est implicitement assimilé à un autre et toute technologie spécialisée reste différée jusqu'à mesure du besoin.
5. Les filtres structurés utilisent les vocabulaires contrôlés, un `CatalogueSnapshot` explicite et des opérateurs déclarés, notamment pour `AND`/`OR`, `UNKNOWN` et `NOT_APPLICABLE`. Une valeur inconnue n'est jamais assimilée à une valeur connue.
6. `CardArchetypeMembership` offre les parcours `(card, snapshot) → archetypes` et `(archetype, snapshot) → cards`. Le snapshot est obligatoire et ni le texte ni les relations stratégiques ne remplacent ce membership.
7. Les résolutions de cartes et contrôles futurs de légalité fonctionnent par lots dans un contexte épinglé `FormatSnapshot + RegionalScope + CatalogueSnapshot + BanlistSnapshot`. L'absence d'entrée n'est interprétable que pour une banlist complète.
8. La provenance est parcourable dans les deux sens par identifiants immuables, de la valeur publiée à la source et du fait aux publications, sans dépendre de recherches textuelles.
9. Les recherches combinées produisent et intersectent leurs candidats dans un même `CatalogueSnapshot`, avec séparation explicite entre recherche linguistique, filtres structurés et annotations revues. Aucun mélange implicite de snapshots n'est admis.
10. Les contraintes d'unicité D-018 disposent des chemins d'accès logiques nécessaires à leur vérification et résolution, sans imposer un index physique par colonne ou contrainte.
11. Toute requête historique fournit ou résout explicitement les identifiants exacts de son contexte. Aucune résolution implicite vers `latest` ou une vue actuelle n'est autorisée.
12. Toute pagination épingle son contexte de recherche et utilise un ordre total déterministe afin d'éviter doublons et omissions. La stratégie physique de pagination reste différée.

## 18. Status

D-019 est **Accepted**. Aucun index physique, moteur de recherche, SQL, migration ou dépendance technique n'est décidé.

## Exclusions

- migrations et SQL PostgreSQL ;
- types d'index physiques ;
- moteur de recherche externe ;
- partitionnement et dénormalisation ;
- implémentation du validateur de deck ou de légalité ;
- API ;
- import des cartes et données de production.

## Dépendances

- **D-004/D-011 :** identité `Card` interne et propriétés versionnées.
- **D-012 :** localisations, alias, recherche multilingue et fallback.
- **D-013/D-014 :** filtres contrôlés et memberships par snapshot.
- **D-015/D-016 :** contexte de légalité et restrictions de banlist.
- **D-017 :** lignée de provenance par identifiants.
- **D-018 :** identités, unicités et références immuables.


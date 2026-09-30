# D-018 — Identifiants et contraintes d'intégrité du modèle logique

## Status

**Accepted — validée le 2026-09-30.**

Ce document définit les identités, unicités et références du modèle logique. Il ne choisit ni clés SQL, types physiques, ORM, migrations, index ou stratégie de stockage.

## 1. Scope

D-018 établit les invariants permettant :

- d'identifier durablement les entités métier et les publications immuables ;
- de distinguer identifiants internes, codes métier et identifiants externes ;
- d'identifier sans ambiguïté les représentations dépendant d'un snapshot ;
- d'empêcher les doublons et références incohérentes ;
- de conserver les anciennes versions et leur provenance ;
- de reproduire un résultat sans résolution implicite vers une version courante.

Le modèle logique couvre `Card`, les entités associées, les formats, régions, banlists, sources, captures, faits, datasets publiés et snapshots métier déjà définis par D-011 à D-017.

## 2. Definitions

### Identifiant interne

Valeur générée par le système dans son propre espace d'identité. Elle est opaque, immuable, non porteuse de sens éditorial et jamais réutilisée.

### Identité stable

Identité durable d'un objet à travers les changements de contenu, de nom, de localisation ou de source. `Card`, `Format`, `Archetype`, `RegionalScope`, `Banlist` et `DataSource` en sont des exemples.

### Identifiant de publication ou de snapshot

Identifie une capture ou publication immuable précise. Il est stable pour cette version, mais ne représente pas l'identité durable de toutes les versions de la famille.

### Identité logique composite

Combinaison de références qui identifie une représentation contextuelle, par exemple `(card_id, catalogue_snapshot_id, language_code)`. Elle constitue un invariant métier même si une implémentation future ajoute une clé technique substitutive.

### Code métier

Code contrôlé lisible, par exemple `TCG_ADVANCED`, `EMEA` ou un `source_code`. Il peut être une clé candidate unique, mais ne remplace pas automatiquement l'identifiant interne.

### Identifiant externe

Valeur attribuée par une source ou un standard extérieur : identifiant YGOPRODeck, passcode, identifiant de base officielle, identifiant d'image ou référence documentaire. Son espace de noms, son type et sa provenance font partie de sa signification.

### Référence immuable

Référence conservant exactement l'identité ou la version initialement utilisée. Elle ne peut pas être résolue implicitement vers « latest », « current » ou un successeur.

## 3. Stable identifiers

### 3.1 Principes généraux

- Les identifiants internes sont générés par le système et immuables.
- Ils ne sont jamais dérivés d'un nom, d'une URL, d'un passcode ou d'un identifiant fournisseur.
- Ils ne sont jamais recyclés après archivage, fusion future ou retrait logique.
- La correction d'un contenu ou d'une provenance ne change pas l'identité durable concernée.
- Une nouvelle version de snapshot reçoit un nouvel identifiant de snapshot ; elle ne crée pas une nouvelle entité stable sous-jacente.
- Un identifiant interne est technique par sa forme, mais devient la référence métier durable lorsqu'il identifie une entité du domaine.

### 3.2 Matrice d'identité

| Entité | Identité logique | Nature et invariant |
|---|---|---|
| `Card` | `card_id` généré | Identité stable, immuable, jamais réutilisée |
| `Archetype` | `archetype_id` généré | Identité stable distincte de ses noms |
| `Format` | `format_id` généré ; `format_code` unique | Identité stable ; le code est une clé métier candidate |
| `RegionalScope` | `regional_scope_id` généré ; `scope_code` unique | Identité stable ; `EMEA` reste un code contrôlé |
| `Banlist` | `banlist_id` généré | Famille stable liée à un format et une région |
| `DataSource` | `data_source_id` généré ; `source_code` unique | Source stable, indépendante de ses captures |
| `FormatSnapshot` | `format_snapshot_id` généré | Identité immuable d'une version précise |
| `BanlistSnapshot` | `banlist_snapshot_id` généré | Identité immuable d'une publication précise |
| `DataSnapshot` | `data_snapshot_id` généré | Identité immuable d'une capture précise, même si son hash est répété |
| `PublishedDataset` | `published_dataset_id` généré | Identité immuable d'une publication validée |
| `CatalogueSnapshot` | `catalogue_snapshot_id` généré | Identité immuable d'un état métier du catalogue |
| `ImportedFact` | `imported_fact_id` généré | Identifie une assertion source extraite et auditée |
| `FunctionalAnnotation` | `functional_annotation_id` généré | Assertion révisable/supersédable avec provenance propre |
| `CardRelation` | `card_relation_id` généré | Assertion source→cible/sélecteur avec cycle de revue propre |

Les identifiants de snapshots, faits, annotations et relations ne sont pas « stables à travers les versions » : ils sont immuables pour l'occurrence exacte qu'ils identifient. Une nouvelle occurrence ou correction produit un nouvel identifiant et, lorsque pertinent, une relation de supersession.

### 3.3 Entités contextuelles sans identité durable artificielle

Les entités suivantes sont identifiées conceptuellement par leur contexte :

| Entité | Identité logique composite |
|---|---|
| `CardSnapshot` | `(card_id, catalogue_snapshot_id)` |
| `CardLocalization` | `(card_id, catalogue_snapshot_id, language_code)` |
| `CardAvailability` | `(card_id, catalogue_snapshot_id, regional_scope_id, format_id)` |
| `CardArchetypeMembership` | `(card_id, archetype_id, catalogue_snapshot_id)` |
| `BanlistEntry` | `(banlist_snapshot_id, card_id)` |

Une implémentation pourra ajouter une clé technique pour des raisons pratiques, mais cette clé ne remplacera ni n'affaiblira l'unicité composite normative.

`CardExternalReference` et `CardImage` sont des assertions versionnées dont l'identité logique dépend de leur espace externe et de leur contexte ; leurs règles sont précisées dans les sections 4 et 5.

## 4. External identifiers

### 4.1 Règles générales

Un identifiant externe n'est valide qu'avec :

- `data_source_id` ou l'autorité de son espace de noms ;
- un `external_id_type` ;
- la valeur originale `external_id` ;
- la période ou les snapshots dans lesquels le mapping est affirmé ;
- la provenance du mapping.

La valeur est conservée comme chaîne exacte afin de préserver zéros initiaux, formats futurs et valeurs non numériques. Aucun identifiant externe n'est requis universellement pour créer une `Card` si la source ne le fournit pas.

### 4.2 YGOPRODeck et références de carte

- L'identifiant YGOPRODeck/YGOProDeck reste une référence de la source opérationnelle.
- Le passcode reste un type de référence externe, pas `card_id`.
- Un identifiant KONAMI ou fournisseur appartient à son propre espace de noms.
- Plusieurs références externes de sources ou types différents peuvent pointer vers le même `card_id`.
- Une même carte peut ne posséder aucune référence dans une source donnée.

Le nom orthographique de la source (`YGOPRODeck`, `YGOProDeck` ou variante) est normalisé via `data_source_id`; il ne crée pas plusieurs espaces de noms si la source réelle est la même.

### 4.3 Historique et réattribution

Un mapping externe peut apparaître, disparaître ou être corrigé entre deux snapshots. Il est représenté comme une assertion versionnée ; l'ancien mapping reste auditable.

Dans un même espace `(data_source_id, external_id_type)`, une valeur externe ne peut pas pointer simultanément vers plusieurs `Card` sur une même période de validité. Une réutilisation ou réattribution observée est une collision en quarantaine jusqu'à revue ; elle ne fusionne ni ne redirige automatiquement les identités internes.

Un changement d'identifiant côté source crée une nouvelle référence et clôt ou supersède l'ancienne assertion. Il ne change pas `card_id`.

## 5. Composite identities

### 5.1 Représentations de catalogue

- `CardSnapshot` : une représentation au plus par carte et `CatalogueSnapshot`.
- `CardLocalization` : une localisation au plus par carte, snapshot et langue.
- `CardAvailability` : une assertion au plus par carte, snapshot, région et format.
- `CardArchetypeMembership` : une appartenance au plus par carte, archétype et snapshot.

### 5.2 Restrictions

`BanlistEntry` possède l'identité `(banlist_snapshot_id, card_id)`. Il ne peut exister qu'une restriction canonique effective pour cette combinaison.

### 5.3 Références externes

Une assertion `CardExternalReference` est identifiée logiquement par :

```text
(card_id, data_source_id, external_id_type, external_id, valid_from_snapshot)
```

L'unicité inverse interdit deux cartes actives pour `(data_source_id, external_id_type, external_id)` sur des périodes qui se chevauchent.

### 5.4 Images

Une référence `CardImage` est distinguée au minimum par :

```text
(card_id, catalogue_snapshot_id, data_source_id,
 external_image_id, resolution_kind)
```

Plusieurs résolutions ou artworks restent possibles sans utiliser l'URL mutable comme identité. D-008 continue de gouverner les droits et n'est pas résolue par cette clé.

### 5.5 Localisation d'archétype

Lorsque les concepts D-014 sont matérialisés :

- `ArchetypeLocalization` : `(archetype_id, catalogue_snapshot_id, language_code)` ;
- `ArchetypeNameAlias` : identité logique incluant archétype, langue, alias normalisé, type et début de validité.

## 6. Uniqueness constraints

### 6.1 Unicité globale

- Chaque identifiant interne généré est unique dans l'espace de son entité.
- `format_code`, `scope_code` et `source_code` sont uniques et ne sont jamais réaffectés à une autre identité.
- Une référence de version publiée est unique dans sa famille : `(format_id, version_ref)`, `(banlist_id, version_ref)`, `(dataset_scope, publication_version)` et la référence équivalente du catalogue.
- Le hash d'un `DataSnapshot` n'est pas globalement unique : deux récupérations identiques restent deux captures.

### 6.2 Unicité contextuelle

- Une seule localisation par `(card, catalogue snapshot, langue)`.
- Une seule restriction par `(banlist snapshot, card)`.
- Une seule membership par `(card, archetype, catalogue snapshot)`.
- Une seule représentation `CardSnapshot` par `(card, catalogue snapshot)`.
- Une seule disponibilité par `(card, catalogue snapshot, région, format)`.

### 6.3 Unicité temporelle

- Les mappings d'un même identifiant externe vers des cartes différentes ne se chevauchent jamais.
- Les versions temporelles de `FormatSnapshot` et `BanlistSnapshot` respectent leurs intervalles et règles de supersession D-015/D-016.
- Les alias et références ayant une validité ne possèdent pas deux assertions canoniques contradictoires sur la même période.

### 6.4 Unicité conditionnelle et assertions

Plusieurs `FunctionalAnnotation` ou `CardRelation` candidates peuvent coexister si leur provenance, contexte ou statut de revue diffère. En revanche :

- un duplicata exact de la même assertion, même contexte, même provenance et même version ne doit pas être publié deux fois ;
- deux assertions canoniques actives ayant la même signature mais des valeurs incompatibles déclenchent un conflit ;
- une assertion supersédée reste conservée mais n'est pas simultanément active comme vérité canonique ;
- une relation exacte source→cible/type/contexte ne doit pas être dupliquée silencieusement.

### 6.5 Versions de mapping

Une version de mapping est unique dans sa famille de règles. Modifier son comportement crée une nouvelle version ; une version existante ne change jamais de sens après utilisation dans un `PublishedDataset`.

## 7. Immutable references

### 7.1 Références vers une identité stable

Référencent une entité stable :

- decks futurs, banlists et relations vers `card_id` ;
- memberships vers `card_id` et `archetype_id` ;
- snapshots de format vers `format_id` ;
- snapshots de banlist vers `banlist_id` ;
- captures vers `data_source_id`.

### 7.2 Références vers une version précise

Référencent obligatoirement l'identifiant exact :

- résultats et analyses vers `catalogue_snapshot_id`, `format_snapshot_id` et `banlist_snapshot_id` ;
- représentations/localisations/disponibilités vers `catalogue_snapshot_id` ;
- entrées de banlist vers `banlist_snapshot_id` ;
- faits importés vers `data_snapshot_id` ;
- datasets publiés vers les `DataSnapshot`, `ImportedFact` et versions de mapping contributrices ;
- décisions de revue vers la suggestion ou assertion initiale exacte.

Aucune de ces références ne peut être remplacée par une requête dynamique « dernière version » dans un résultat historique.

### 7.3 Contexte reproductible

Un résultat ancien conserve au minimum :

```text
card_id ou ensemble de card_id concernés
catalogue_snapshot_id
format_snapshot_id
regional_scope_id
banlist_snapshot_id
analyzed_at
engine_version_ref lorsque disponible
provenance/publication refs nécessaires
```

Une supersession permet de découvrir une version corrigée, mais ne redirige jamais implicitement une référence historique.

## 8. Snapshot consistency

### 8.1 Cohérence dans CatalogueSnapshot

Les entités partageant un `catalogue_snapshot_id` appartiennent au même contexte de catalogue :

- une `CardLocalization` exige le `CardSnapshot` correspondant pour `(card_id, catalogue_snapshot_id)` ;
- `CardAvailability` et `CardArchetypeMembership` exigent une `Card` résolue et une représentation compatible dans ce snapshot ;
- une localisation ou membership d'archétype exige l'identité d'archétype connue dans ce contexte ;
- aucune donnée d'un autre snapshot ne peut être substituée implicitement pour compléter un champ manquant.

### 8.2 Banlist et catalogue

`BanlistEntry` référence `Card`, pas `CardSnapshot`. Lors d'une analyse, le `CatalogueSnapshot` épinglé doit toutefois résoudre ce `card_id`. Dans le cas contraire, le contexte est indéterminé ou incohérent ; il ne déclenche pas une résolution vers le catalogue courant.

### 8.3 Provenance et publications

- `ImportedFact.data_snapshot_id` cible exactement la capture d'origine.
- Un `PublishedDataset` référence les captures, faits et mappings exacts utilisés.
- Seul un `PublishedDataset` au statut publiable peut contribuer comme source canonique d'un snapshot métier publié.
- `CatalogueSnapshot` conserve les identifiants des publications contributrices ; il ne copie pas uniquement un numéro de « dernière version ».

### 8.4 Format, région et banlist

Le contexte vérifie les compatibilités D-015/D-016 entre `FormatSnapshot`, `RegionalScope` et `BanlistSnapshot`. L'égalité d'une date seule ne suffit pas à rendre deux snapshots compatibles.

## 9. Referential integrity

Toute référence doit résoudre vers une identité existante dans l'espace attendu et respecter son état logique :

- une référence stable peut cibler une entité archivée si l'historique l'exige ;
- une nouvelle publication ne peut pas introduire une référence pendante ;
- une analyse canonique ne référence pas un snapshot `DRAFT`, `REJECTED`, incomplet ou non publié ;
- une référence de supersession cible une version antérieure de la même famille logique et ne forme aucun cycle ;
- une relation vers un sélecteur D-010 épingle sa version de définition et son `CatalogueSnapshot` d'évaluation ;
- une clé externe non résolue reste un fait en quarantaine, jamais une pseudo-clé étrangère canonique.

Les dépendances de publication forment un graphe auditable. Une publication ne peut pas dépendre circulairement d'elle-même, directement ou indirectement.

## 10. Deletion and archival invariants

- Aucun identifiant interne ou code métier historique n'est réutilisé.
- Une entité référencée par un snapshot publié ou un résultat historique n'est pas supprimée physiquement au niveau conceptuel.
- L'archivage ou le retrait logique préserve l'identité, les anciennes versions et la provenance.
- L'absence d'une carte, localisation, membership ou disponibilité dans un nouveau snapshot ne supprime pas l'entité ni ses occurrences historiques.
- Un snapshot supersédé reste accessible et identifiable.
- Un fait rejeté ou mis en quarantaine peut être conservé pour audit sans devenir canonique.

D-018 ne prescrit aucun champ de soft-delete. Elle impose seulement la préservation logique des références historiques.

## 11. Conflict and anomaly handling

| Anomalie | Règle logique |
|---|---|
| Doublon exact | Rejeter ou dédupliquer explicitement avec trace ; ne pas publier deux occurrences canoniques |
| Collision d'identifiant externe | Quarantaine des mappings concurrents et revue ; aucune fusion automatique |
| Référence vers objet inexistant | Référence pendante bloquante pour une publication canonique |
| Snapshots incompatibles | Contexte rejeté ou indéterminé ; aucune substitution par la version actuelle |
| Deux valeurs pour une unicité | Conflit explicite bloquant jusqu'à résolution/supersession |
| Référence historique vers version non publiée | Résultat non canonique/invalide ; aucune promotion automatique de la cible |
| Source supposant un changement d'identité | Nouvelle assertion candidate en quarantaine ; `card_id` inchangé jusqu'à décision humaine documentée |
| Identifiant externe réutilisé | Conflit temporel explicite, historique conservé, mapping non résolu silencieusement |
| Cycle de supersession | Anomalie d'intégrité bloquante |

Les corrections créent de nouvelles assertions, versions ou décisions reliées aux anciennes. Elles ne réparent jamais silencieusement l'historique.

## 12. Historical reproducibility

Pour reproduire un résultat :

1. résoudre les identités stables originales ;
2. charger exactement les snapshots épinglés ;
3. vérifier leur compatibilité et leur statut publié ;
4. retrouver les `PublishedDataset` contributeurs ;
5. retrouver les `ImportedFact`, `DataSnapshot` et versions de mapping ;
6. utiliser `analyzed_at` et la version du moteur lorsqu'elle existe.

Le système peut proposer une reproduction « corrigée » avec des versions supersédantes, mais celle-ci constitue une nouvelle exécution et ne remplace pas le résultat historique.

## 13. Edge cases

| Cas | Traitement attendu |
|---|---|
| Carte renommée | Même `card_id`, nouvelle localisation dans un nouveau snapshot |
| Identifiant YGOPRODeck modifié | Nouvelle référence externe versionnée, `card_id` inchangé |
| Passcode absent | Identité interne valide ; aucune valeur factice |
| Même passcode prétendument attribué à deux cartes | Collision en quarantaine, aucune fusion |
| Deux captures au même hash | Deux `data_snapshot_id`, rapprochement par hash possible |
| Correction d'un mapping | Nouvelle version et nouveau dataset publié, historique intact |
| Localisation française absente | Pas de ligne fictive ; statut/fallback D-012 dans le même snapshot |
| Carte absente d'un nouveau catalogue | Anciennes représentations conservées ; pas de suppression de `Card` |
| Banlist visant une carte absente du catalogue épinglé | Contexte indéterminé, anomalie de synchronisation |
| Relation stratégique corrigée | Nouvelle assertion ou supersession ; l'ancienne provenance reste accessible |
| Snapshot supersédé | Anciennes références restent sur l'ancien identifiant |
| Code métier renommé | Alias/libellé séparé ou migration de code explicitement gouvernée ; aucun recyclage |

## 14. Final decisions

1. Les entités durables (`Card`, `Archetype`, `Format`, `RegionalScope`, `Banlist`, `DataSource`) utilisent des identifiants internes générés, immuables et jamais réutilisés, distincts des clés métier et identifiants externes.
2. Chaque snapshot, capture, publication, fait, annotation et relation possède sa propre identité immuable lorsque l'objet constitue une assertion ou une occurrence versionnée identifiable. Une nouvelle version ne réutilise jamais l'identité précédente.
3. Les identités composites de `CardSnapshot`, `CardLocalization`, `CardAvailability`, `CardArchetypeMembership` et `BanlistEntry` restent des invariants métier normatifs. Une future clé technique ne supprime jamais leur contrainte d'unicité conceptuelle.
4. Les identifiants YGOPRODeck, passcodes et autres références externes restent typés, sourcés et versionnés. Ils ne remplacent jamais `card_id` ; toute collision ou réattribution incohérente est mise en quarantaine sans réconciliation silencieuse.
5. Les codes métier explicitement définis comme stables, notamment `format_code`, `scope_code` et `source_code`, peuvent être des clés candidates uniques et non recyclables, distinctes des identifiants internes.
6. `FunctionalAnnotation` et `CardRelation` possèdent chacune une identité immuable propre, indépendante d'un simple hash ou contenu mutable. Les contraintes empêchant les doublons conceptuels restent séparées de cette identité technique.
7. Toute référence historique épingle explicitement la version et le contexte exacts. Aucune résolution implicite vers `latest`, l'état courant ou une version supersédante n'est autorisée.
8. Toute incompatibilité de contexte ou de snapshots bloque la publication ou l'analyse canonique concernée et produit une anomalie explicite et traçable.
9. Les identités historiques référencées ne sont jamais réutilisées. Leur destruction physique ne peut pas rendre impossible la reproduction d'un contexte publié ; l'absence dans un nouveau snapshot ne constitue pas une suppression historique.
10. Toute collision d'identifiant, violation d'unicité, référence pendante ou incohérence est mise en quarantaine. Aucune réparation implicite ni stratégie « last value wins » n'est admise.
11. Tout résultat reproductible conserve les identifiants exacts de `FormatSnapshot`, `RegionalScope`, `CatalogueSnapshot`, `BanlistSnapshot`, l'instant d'analyse et, lorsque pertinent, les publications et provenances contributrices. Une référence historique n'est jamais recalculée depuis l'état courant.

## 15. Status

D-018 est **Accepted**. Aucun choix de clé physique, de type SQL, d'index, de moteur de base de données ou d'ORM n'est effectué.

## Exclusions

- SQL, PostgreSQL et migrations ;
- ORM et classes applicatives ;
- code Python ou TypeScript ;
- API et importeurs ;
- clés ou index physiques ;
- optimisation de requêtes ;
- stratégie de suppression technique.

## Dépendances

- **D-003/D-004 :** sources externes distinctes des identités internes numériques.
- **D-007 :** contexte exact et reproductible pour chaque résultat.
- **D-010 :** provenance, revue et sélecteurs versionnés des assertions stratégiques.
- **D-011/D-012 :** identité `Card`, représentations et localisations par catalogue snapshot.
- **D-013/D-014 :** mappings contrôlés et memberships versionnés.
- **D-015/D-016 :** format, région, banlist et contraintes temporelles.
- **D-017 :** chaîne de provenance et publications immuables.


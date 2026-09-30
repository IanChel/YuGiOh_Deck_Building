# D-017 — Spécification de la provenance et de la publication des données

## Statut

**Accepted — validée le 2026-09-30.**

Cette spécification définit conceptuellement `DataSource`, `DataSnapshot`, `ImportedFact` et `PublishedDataset`. Elle ne crée aucun code, SQL, ORM, migration, importeur, stockage ou dataset physique.

## 1. Objectif

D-017 formalise la chaîne permettant d'expliquer chaque donnée métier publiée :

```text
DataSource
  ↓ capture
DataSnapshot
  ↓ extraction structurée
ImportedFact
  ↓ mapping / résolution / validation
PublishedDataset
  ↓ consommation métier
CatalogueSnapshot et autres snapshots métier
```

Le système doit pouvoir retrouver la valeur externe exacte, sa source, son instant de capture, sa référence externe, les transformations appliquées et la décision ayant permis sa publication.

## 2. Principes fondamentaux

### 2.1 Une source n'est pas une donnée

`DataSource` décrit une origine durable et ses responsabilités. Elle ne contient pas les données capturées.

Une même source peut produire plusieurs `DataSnapshot`, changer de contenu ou de contrat et avoir une autorité différente selon le domaine considéré.

### 2.2 Un snapshot est une capture déterminée

`DataSnapshot` décrit le contenu observé lors d'une récupération précise. Deux récupérations de la même source ne sont pas confondues avec l'identité de cette source.

Une version externe est utile lorsqu'elle existe, mais l'identité d'une capture ne dépend pas uniquement d'elle : date de récupération, référence de requête et empreinte de contenu permettent aussi de distinguer ou rapprocher les captures.

## 3. DataSource

Proposition conceptuelle minimale :

| Information | Rôle |
|---|---|
| `data_source_id` | Identité interne stable |
| `source_code` | Code contrôlé et durable |
| `display_name` | Nom lisible de la source |
| `source_roles` | Un ou plusieurs rôles couverts |
| `covered_domains` | Catalogue, règles, banlist ou autre domaine explicite |
| `authority_profile_ref` | Politique versionnée indiquant l'autorité par domaine |
| `external_reference` | Site, documentation ou identifiant externe de la source |
| `operational_status` | Active, inactive ou temporairement indisponible |

Les rôles minimaux reconnus sont :

- `OPERATIONAL_CATALOG` ;
- `OFFICIAL_RULES` ;
- `OFFICIAL_BANLIST`.

Cette liste est extensible par gouvernance ; elle n'est ni une hiérarchie exhaustive ni un score de confiance. Une source peut posséder plusieurs rôles. Son statut opérationnel ne change pas son autorité historique.

## 4. Autorité de la source

L'autorité est évaluée par domaine et par type d'assertion, pas par un niveau global unique.

- **YGOPRODeck** est la source opérationnelle du catalogue local selon D-003.
- **KONAMI** est l'autorité normative retenue pour les règles et banlists officielles.
- Une source opérationnelle peut transporter une valeur utile sans devenir normative.
- Une source officielle dans un domaine n'est pas automatiquement normative dans tous les autres.

La priorité entre sources est une politique explicite, versionnée et auditable. Elle ne peut pas être déduite du nom de la source, de l'ordre d'import ou de la dernière valeur reçue.

Autorité, qualité observée, disponibilité technique et droits d'utilisation sont quatre dimensions distinctes.

## 5. DataSnapshot

`DataSnapshot` est une capture immuable d'une `DataSource`.

| Information | Rôle conceptuel |
|---|---|
| `data_snapshot_id` | Identité interne de la capture |
| `data_source_id` | Source capturée |
| `retrieved_at` | Instant de récupération |
| `external_version_ref` | Version ou révision externe lorsqu'elle existe |
| `external_published_at` | Date de publication externe lorsqu'elle est connue |
| `effective_from` | Date d'effet métier lorsqu'elle existe et a un sens |
| `request_ref` | Endpoint, document, paramètres ou référence de requête |
| `content_hash` | Empreinte du contenu capturé lorsqu'elle est calculable |
| `artifact_ref` | Référence auditée du contenu brut conservé selon la politique applicable |
| `retrieval_status` | Capture complète ou partielle |
| `integrity_status` | Résultat des contrôles techniques |
| `schema_or_contract_ref` | Version du contrat externe observé |
| `duplicate_content_of` | Capture antérieure au contenu identique, le cas échéant |

`retrieved_at`, `external_published_at` et `effective_from` ne sont jamais substitués l'un à l'autre. Une source sans version ou date officielle conserve ces champs absents et s'appuie sur la capture, la requête et l'empreinte réellement disponibles.

Une tentative échouée sans contenu capturé reste un événement d'acquisition auditable, mais ne prétend pas être un `DataSnapshot` complet. Une réponse partielle peut produire un snapshot marqué partiel, inutilisable pour une publication complète sans règle explicite.

## 6. Immutabilité

Après enregistrement d'une capture :

- son contenu et son empreinte ne sont jamais remplacés silencieusement ;
- une nouvelle récupération produit une nouvelle identité de capture, même si le contenu est identique ;
- `duplicate_content_of` peut signaler un contenu identique sans effacer l'événement de récupération ;
- les anciennes références restent accessibles pour l'audit ;
- une correction substantielle de provenance crée un amendement audité ou un nouveau snapshot, selon ce qui est corrigé.

Distinctions :

- **nouvelle récupération** : nouveau `DataSnapshot` ;
- **correction de métadonnée** : amendement traçable ne changeant jamais le contenu prétendument capturé ;
- **correction d'un fait importé** : nouveau résultat d'extraction/mapping ou décision de revue, sans réécrire la valeur brute ;
- **nouvelle publication métier** : nouveau `PublishedDataset`, puis éventuellement nouveau snapshot métier.

## 7. ImportedFact

`ImportedFact` représente une assertion structurée attribuable à un `DataSnapshot`. Il décrit ce que la source fournit, et non ce que le domaine considère déjà comme canonique.

Proposition minimale :

| Information | Rôle conceptuel |
|---|---|
| `imported_fact_id` | Identité interne de l'assertion importée |
| `data_snapshot_id` | Capture d'origine |
| `source_subject_ref` | Sujet externe, par exemple identifiant de carte |
| `source_path` | Chemin, champ, ligne ou emplacement dans la source |
| `fact_kind` | Nature structurée de l'assertion |
| `raw_value` | Valeur externe originale préservée |
| `parsed_value` | Représentation technique sans perte lorsqu'elle est possible |
| `external_record_ref` | Référence du record source |
| `extracted_at` | Instant d'extraction |
| `extractor_version_ref` | Version de la règle d'extraction |
| `quality_findings` | Anomalies observées sans altérer la valeur source |

Un fait peut représenter un champ atomique ou une assertion structurée indissociable. D-017 n'impose pas une ligne physique par scalaire ; elle impose que la granularité permette une provenance et une résolution non ambiguës.

## 8. Brut et normalisé

Trois niveaux restent séparés :

```text
raw_value
  = ce que la source a fourni

parsed_value
  = lecture technique fidèle, sans décision métier

canonical candidate / published value
  = résultat d'un mapping et d'une validation explicites
```

Exemple :

```text
raw_value: "Effect Monster"
  ↓ mapping versionné
canonical candidate: MONSTER + EFFECT
```

La valeur brute, sa casse, son format et son emplacement restent disponibles. Une normalisation de texte ou de type ne doit jamais remplacer la preuve de ce que la source disait.

## 9. Provenance de chaque fait

Chaque `ImportedFact` est directement relié à son `DataSnapshot`, qui est lui-même relié à `DataSource`. La lignée complète peut ajouter :

- référence et identifiant externes ;
- chemin exact dans la source ;
- empreinte du record ou fragment ;
- instant et version d'extraction ;
- version de mapping/transformation ;
- décision de résolution ;
- réviseur, justification et sources examinées ;
- fait publié qui en résulte.

Une valeur publiée doit pouvoir pointer vers un ou plusieurs faits contributeurs, y compris les faits écartés lors d'un conflit lorsque leur existence explique la décision.

## 10. Transformation et mapping

Les transformations incluent notamment :

- mapping des types, attributs et propriétés D-013 ;
- normalisation de texte sans perte de l'original ;
- résolution d'identifiants externes vers `Card` ;
- mapping d'archétypes D-014 ;
- mapping des restrictions de banlist D-016 ;
- conversion de formats et de représentations.

Chaque résultat candidat conserve :

- l'identité des `ImportedFact` d'entrée ;
- la version de la règle ou table de mapping ;
- les paramètres déterminants ;
- le résultat produit ;
- les diagnostics ;
- toute revue ou dérogation humaine.

Un changement de mapping ne réécrit pas les publications antérieures. Il peut produire une nouvelle validation et un nouveau `PublishedDataset` à partir du même `DataSnapshot`.

## 11. Anomalies

D-017 distingue les états métier des constats de qualité :

- `UNKNOWN` : la propriété est applicable mais sa valeur métier n'est pas établie ;
- `UNMAPPED` : une valeur externe existe, mais aucune correspondance interne validée n'est disponible ;
- `INVALID` : la valeur ou structure viole un contrat attendu ; c'est un constat de qualité, pas une valeur métier ;
- `AMBIGUOUS` : plusieurs résolutions restent possibles ;
- `CONFLICT` : des assertions candidates incompatibles subsistent ;
- `MISSING` : une donnée attendue selon le contrat n'est pas présente ;
- snapshot `PARTIAL` ou source inaccessible : problème au niveau acquisition/couverture.

Ces termes constituent un vocabulaire minimal de diagnostics, pas des valeurs à injecter dans tous les champs métier. Une anomalie conserve sa portée, sa sévérité, sa preuve, son statut de résolution et l'élément concerné.

Une anomalie n'est jamais convertie silencieusement en valeur canonique, `OTHER`, chaîne vide ou valeur par défaut.

## 12. Quarantaine

La quarantaine conserve les faits et candidats non résolus sans les rendre consommables comme données canoniques.

Elle couvre notamment :

- race ou type externe inconnu ;
- attribut non mappé ;
- archétype ambigu ou absent du référentiel ;
- identifiant de carte non résolu ;
- entrée de banlist inconnue ;
- conflit entre sources non arbitré ;
- snapshot partiel prétendant couvrir un périmètre complet.

Une donnée en quarantaine :

- reste reliée à sa source et à sa valeur brute ;
- peut être revue et résolue ;
- peut être rejetée avec justification ;
- ne contribue pas à une publication canonique complète tant qu'elle est bloquante ;
- n'est jamais supprimée uniquement pour faire réussir une publication.

## 13. PublishedDataset

`PublishedDataset` est une publication immuable d'un ensemble de données canoniques validées, destinée aux consommateurs métier.

| Information | Rôle conceptuel |
|---|---|
| `published_dataset_id` | Identité de la publication |
| `dataset_scope` | Domaine et couverture annoncés |
| `publication_version` | Référence stable de version |
| `publication_status` | Cycle de validation/publication |
| `created_at` | Création du candidat |
| `published_at` | Publication canonique interne |
| `supersedes_dataset_id` | Publication remplacée, le cas échéant |
| `source_snapshot_refs` | `DataSnapshot` contributeurs |
| `mapping_version_refs` | Règles et transformations utilisées |
| `integrity_report_ref` | Contrôles, couverture, anomalies et décisions |
| `provenance_manifest_ref` | Lignée détaillée des valeurs publiées |

Le dataset publié ne contient pas nécessairement tout le catalogue métier. Son périmètre doit être explicite : catalogue canonique candidat, banlist validée, localisations validées ou autre lot cohérent.

## 14. PublishedDataset et CatalogueSnapshot

Les concepts répondent à des questions différentes :

- `PublishedDataset` : quelles données issues des sources ont passé mappings, arbitrages et validations dans cette publication ?
- `CatalogueSnapshot` : quel état cohérent du catalogue métier l'application expose-t-elle et épingle-t-elle ?

Un `CatalogueSnapshot` peut consommer un ou plusieurs `PublishedDataset`, appliquer des invariants métier et référencer les versions de vocabulaire nécessaires. Un même `PublishedDataset` peut contribuer à plusieurs snapshots métier si les règles de composition le permettent.

La séparation évite de confondre la preuve du pipeline de données avec la vue métier reproductible. Elle permet aussi aux `FormatSnapshot` et `BanlistSnapshot` de conserver leur propre identité tout en référençant la provenance appropriée.

## 15. Plusieurs sources

Un `PublishedDataset` peut utiliser plusieurs `DataSnapshot` provenant de sources différentes.

La provenance agrégée ne remplace jamais la provenance par valeur :

- chaque valeur publiée pointe vers ses faits contributeurs ;
- chaque fait pointe vers son snapshot et sa source ;
- la règle d'arbitrage ou de combinaison est versionnée ;
- les faits rejetés mais pertinents pour expliquer un conflit restent auditables.

Un manifeste global liste les captures utilisées, mais il ne suffit pas à lui seul pour expliquer chaque valeur.

## 16. Conflits entre sources

Lorsqu'une source affirme `LIGHT` et une autre `DARK` :

1. les deux `ImportedFact` sont conservés séparément ;
2. un diagnostic `CONFLICT` relie les assertions ;
3. une politique d'autorité versionnée détermine si une source est normative pour ce domaine ;
4. si la politique ne suffit pas, une revue humaine documentée tranche ou maintient la quarantaine ;
5. la valeur publiée conserve la décision, la règle et les deux lignées ;
6. aucune stratégie « dernière valeur gagne » ou fusion silencieuse n'est admise.

La priorité peut varier selon le domaine : une source officielle de banlist peut prévaloir pour une restriction sans être choisie pour une localisation ou une image.

## 17. Publication

Le cycle minimal de `PublishedDataset` est :

- `DRAFT` : assemblage en cours ;
- `VALIDATING` : contrôles et revues en cours ;
- `PUBLISHED` : publication immuable consommable dans son périmètre annoncé ;
- `SUPERSEDED` : une publication ultérieure la remplace sans la supprimer ;
- `REJECTED` : candidat refusé, conservé pour audit si nécessaire.

Pour devenir `PUBLISHED`, un dataset doit posséder :

- un périmètre et une couverture déclarés ;
- des captures immuables identifiées ;
- des mappings et transformations versionnés ;
- toutes les références obligatoires résolues ;
- aucun diagnostic bloquant non résolu dans le périmètre annoncé ;
- un rapport d'intégrité validé ;
- une provenance par valeur suffisamment complète ;
- les revues humaines requises.

Une publication partielle est possible uniquement si son périmètre réduit est explicitement déclaré et si aucun consommateur ne peut la confondre avec un dataset complet. Réduire silencieusement la couverture pour éviter une anomalie est interdit.

## 18. Reproductibilité

Une publication identifie :

```text
PublishedDataset
  ├── DataSnapshot(s)
  │     └── ImportedFact(s)
  ├── mapping/transformation version(s)
  ├── décisions de résolution et revues
  └── rapport d'intégrité
```

Le système doit pouvoir expliquer dans les deux sens :

- d'une valeur publiée vers tous ses faits et règles d'origine ;
- d'un fait importé vers les publications auxquelles il a contribué ou dans lesquelles il a été rejeté.

## 19. Relation avec les snapshots métier

`DataSnapshot` est une capture de source externe. `CatalogueSnapshot`, `FormatSnapshot` et `BanlistSnapshot` sont des états métier versionnés.

Ils ne partagent ni identité ni cycle de vie :

- une nouvelle capture ne force pas automatiquement une publication métier ;
- une nouvelle publication validée ne force pas automatiquement tous les snapshots métier à évoluer ;
- chaque snapshot métier référence les `PublishedDataset` ou manifestes de provenance nécessaires ;
- un résultat métier épingle les snapshots métier, dont la lignée permet ensuite de retrouver les sources.

## 20. Source officielle et donnée importée

Une source officielle apporte une autorité de contenu dans un domaine défini. Elle ne confère pas automatiquement un droit de copie, de stockage, d'affichage ou de redistribution.

D-017 trace l'origine, la capture et l'usage dans le pipeline. D-008 reste seule compétente pour les licences, droits, marques et conditions de publication. Une donnée peut être normative mais juridiquement non publiable par le produit.

## 21. LLM

Un LLM n'est ni une `DataSource` normative, ni l'auteur d'un `ImportedFact` officiel.

Il peut produire une suggestion séparée de mapping, normalisation, détection d'anomalie ou résolution. Cette suggestion conserve modèle/version, prompt ou référence de tâche, entrées, sortie, date et statut de revue lorsque la politique le permet.

Le workflow D-010 s'applique :

```text
LLM_SUGGESTION / UNREVIEWED
  → revue humaine documentée
  → HUMAN_ANNOTATION / REVIEWED ou REJECTED
```

Même revue, une suggestion ne change pas rétroactivement ce que la source affirmait. La valeur publiée conserve le fait source, la suggestion éventuelle et la décision humaine comme provenances distinctes.

## 22. Cas limites obligatoires

| Cas | Comportement conceptuel |
|---|---|
| 1. Même source récupérée deux fois | Deux événements/captures ; hash identique et `duplicate_content_of` possibles, sans écrasement |
| 2. Snapshot sans version externe | Utiliser identité interne, `retrieved_at`, requête et hash ; version externe explicitement absente |
| 3. Source modifiée sans nouvelle version | Nouveau hash et nouveau snapshot ; anomalie de contrat/version signalée |
| 4. Source inaccessible | Enregistrer l'échec d'acquisition ; ne pas fabriquer de snapshot complet ni réutiliser silencieusement l'ancien |
| 5. Capture partielle | Snapshot marqué partiel et mis en quarantaine pour toute publication prétendant être complète |
| 6. Valeur externe inconnue | `UNMAPPED`, valeur brute conservée, pas de valeur canonique |
| 7. Valeur ambiguë | Diagnostic `AMBIGUOUS`, résolution ou revue requise |
| 8. Conflit entre sources | Faits conservés, politique d'autorité versionnée ou revue, aucune fusion silencieuse |
| 9. Mapping modifié | Nouvelle version de mapping et nouvelle publication candidate ; ancienne publication inchangée |
| 10. Correction d'un fait historique | Conserver le fait brut ; nouvelle décision/publication avec lien de supersession |
| 11. Dataset supersédé | Ancien dataset accessible, nouveau lien explicite, consommateurs historiques inchangés |
| 12. Source officielle, droits non résolus | Provenance conservée ; D-008 bloque l'usage concerné sans nier l'autorité de la source |
| 13. Suggestion LLM | Suggestion séparée, revue D-010, jamais assimilée à un fait officiel |
| 14. Plusieurs sources | Manifeste agrégé et lignée par valeur obligatoires |

## 23. Hors périmètre

D-017 exclut explicitement :

- implémentation d'importeurs ;
- API et UI ;
- stockage physique, SQL, ORM et migrations ;
- jobs planifiés et pipelines CI/CD ;
- téléchargement réel de sources ;
- résolution détaillée des droits et licences D-008 ;
- catalogue ou dataset de production ;
- validation complète de deck ;
- scoring et recommandations ;
- RAG et base vectorielle.

## 24. Décisions finales validées

1. `DataSource` peut porter plusieurs rôles et utilise un profil d'autorité versionné par domaine. Aucun niveau global de confiance ne s'applique indistinctement à toutes ses données.
2. Chaque récupération constitue une capture immuable et traçable, même lorsque son hash est identique à celui d'une capture précédente. Un échec sans contenu exploitable reste un événement d'acquisition et ne devient pas un `DataSnapshot` complet.
3. `ImportedFact` est une assertion de source non canonique. Il conserve obligatoirement la valeur brute, son emplacement, son `DataSnapshot` et la version d'extraction applicable.
4. `PublishedDataset` publie des données validées ; `CatalogueSnapshot` compose l'état métier versionné du catalogue. Ces deux concepts restent distincts.
5. Toutes les assertions contradictoires sont conservées. Leur résolution utilise une politique d'autorité versionnée ou une revue humaine documentée ; la stratégie implicite « dernière valeur reçue » est interdite.
6. Un dataset devient `PUBLISHED` uniquement lorsqu'aucune anomalie bloquante ne subsiste dans son périmètre déclaré. Toute publication partielle annonce explicitement et trace sa couverture.
7. Chaque donnée publiée conserve ses `ImportedFact` contributeurs ainsi que les versions des mappings et transformations. Un changement de mapping produit une nouvelle publication sans réécriture historique.
8. Toute contribution LLM reste une suggestion distincte des `ImportedFact`, suit le workflow D-010 et ne contribue à une publication canonique qu'après revue humaine traçable.

## 25. Proposition minimale

```text
DataSource
  data_source_id
  source_code
  source_roles[]
  covered_domains[]
  authority_profile_ref
  operational_status

DataSnapshot
  data_snapshot_id
  data_source_id
  retrieved_at
  external_version_ref?
  external_published_at?
  effective_from?
  request_ref
  content_hash?
  artifact_ref?
  retrieval_status
  integrity_status
  schema_or_contract_ref?

ImportedFact
  imported_fact_id
  data_snapshot_id
  source_subject_ref
  source_path
  fact_kind
  raw_value
  parsed_value?
  extractor_version_ref
  quality_findings[]

PublishedDataset
  published_dataset_id
  dataset_scope
  publication_version
  publication_status
  source_snapshot_refs[]
  mapping_version_refs[]
  integrity_report_ref
  provenance_manifest_ref
  supersedes_dataset_id?
```

## Dépendances

- **D-003 :** distingue catalogue opérationnel YGOPRODeck et autorité normative KONAMI.
- **D-008 :** conserve la responsabilité des droits et licences, indépendamment de la provenance.
- **D-010 :** fournit le workflow de revue des suggestions LLM et annotations humaines.
- **D-013 :** exige des mappings de vocabulaires contrôlés versionnés.
- **D-014 :** exige la quarantaine des archétypes externes non résolus.
- **D-016 :** exige une lignée officielle et complète pour les snapshots de banlist.


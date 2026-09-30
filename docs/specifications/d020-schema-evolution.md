# D-020 — Stratégie d'évolution du schéma

## Status

**Proposed — validation humaine requise.**

Cette spécification définit les invariants architecturaux d'évolution du modèle et du schéma. Elle ne crée aucune migration, aucun SQL, aucun fichier Alembic, aucun code applicatif et ne modifie aucun snapshot existant.

## 1. Scope

D-020 doit permettre au modèle d'évoluer tout en garantissant :

- le sens historique des données publiées ;
- la reproductibilité des résultats épinglés ;
- la stabilité des identifiants D-018 ;
- la lignée de provenance D-017 ;
- l'interprétation des anciens vocabulaires, mappings et relations ;
- une coexistence contrôlée des composants applicatifs pendant une transition ;
- des migrations vérifiables, atomiques du point de vue canonique et réversibles lorsque cela est réellement possible.

Principe directeur :

> Une migration de schéma est une évolution technique. Une publication ou un snapshot est une évolution métier. L'une ne vaut jamais automatiquement l'autre.

## 2. Definitions

### Schema version

Version du contrat structurel et d'interprétation utilisé pour stocker ou lire des données canoniques. Elle décrit les structures disponibles, leurs sémantiques, leurs compatibilités et les migrations associées.

### Business data version

Version d'une donnée métier ou d'une assertion lorsque son contenu ou sa signification publiée évolue. Elle ne se déduit pas du numéro de schéma.

### Snapshot version

Identité immuable d'un état publié : `CatalogueSnapshot`, `FormatSnapshot`, `BanlistSnapshot` ou autre snapshot métier.

### Publication version

Identité immuable d'un `PublishedDataset` validé, avec sa provenance et ses mappings.

### Mapping version

Version d'une règle transformant des valeurs externes en représentations internes. Une nouvelle règle ne réinterprète pas les anciennes publications.

### Vocabulary version

Version du registre définissant les valeurs contrôlées, leur sens, leurs dépréciations et leurs remplacements éventuels.

### Interpretation contract

Ensemble des versions de schéma, vocabulaire, mapping et règles nécessaires pour comprendre une donnée publiée comme elle l'était lors de sa production.

### Backfill technique

Transformation déterministe ajoutant ou reconstruisant une représentation techniquement nécessaire sans créer une nouvelle vérité métier ni changer le sens observable d'un snapshot.

### Republishing

Création d'une nouvelle publication ou d'un nouveau snapshot à partir de nouvelles données, d'un mapping corrigé ou d'une sémantique modifiée. Elle ne remplace pas silencieusement l'historique.

## 3. Schema vs data versioning

Les versions suivantes évoluent indépendamment :

| Version | Déclencheur | Effet attendu |
|---|---|---|
| Schéma | Structure ou contrat technique modifié | Nouvelle capacité de lecture/écriture, sans publication métier automatique |
| Donnée métier | Valeur canonique modifiée | Nouvelle assertion/version selon son modèle |
| Snapshot | Nouvel état métier publié | Nouvelle identité immuable |
| Publication | Nouveau dataset validé | Nouveau `published_dataset_id` |
| Mapping | Nouvelle transformation externe→interne | Nouvelles productions seulement, sauf republication explicite |
| Vocabulaire | Ajout, dépréciation ou sens versionné | Anciennes valeurs restent interprétables dans leur version |

Chaque publication canonique conserve `schema_version_ref` ainsi que les références de vocabulaire et mapping nécessaires à son interprétation. Un snapshot métier doit permettre de retrouver ces références via sa provenance D-017.

Une migration de schéma n'incrémente pas un `CatalogueSnapshot`. Une nouvelle publication métier ne requiert pas de réécrire les anciennes représentations si leur contrat historique reste lisible.

## 4. Snapshot immutability

Après une évolution de schéma, un snapshot publié doit conserver :

- les mêmes identités et références ;
- le même contenu métier observable ;
- la même signification des valeurs ;
- les mêmes états `UNKNOWN`, `NOT_APPLICABLE`, absents ou présents ;
- la même provenance et les mêmes décisions de revue ;
- la possibilité de reproduire les résultats historiques qui l'utilisent.

### Ajout de propriété

L'absence du nouveau champ dans un snapshot ancien signifie « non représenté dans ce contrat historique », et non automatiquement `UNKNOWN`, `NULL`, `false` ou une valeur par défaut.

### Suppression logique

Un champ historique peut être déprécié pour les nouvelles écritures, mais sa définition et sa lecture restent disponibles tant que des publications le référencent.

### Renommage

Un renommage introduit un nom canonique nouveau et une règle de compatibilité explicite. Il ne change pas silencieusement le sens de l'ancien champ et ne nécessite pas de réécrire sa provenance.

### Changement de type ou de relation

Une nouvelle représentation est introduite. L'ancienne reste lisible ou reconstructible par un adaptateur versionné et sans perte. Une conversion ambiguë ou destructive exige une nouvelle publication métier.

### Vocabulaire ou mapping

La publication continue d'utiliser les versions épinglées lors de sa création. La version courante n'est jamais appliquée rétroactivement par défaut.

Une migration physique peut déplacer ou réencoder des données historiques uniquement si une vérification démontre une équivalence sémantique et préserve la lignée. Sinon, l'ancienne représentation doit être conservée.

## 5. Compatible changes

D-020 classe les évolutions en quatre catégories logiques.

### 5.1 Additive

Ajoute une capacité sans modifier les contrats existants : champ facultatif, nouvelle table logique, nouvelle langue possible, nouvelle relation ou annotation facultative.

- Les anciennes données restent lisibles sans backfill métier.
- Les anciens snapshots ne reçoivent aucune valeur inventée.
- Les nouveaux écrivains peuvent utiliser la capacité lorsque leur contrat le permet.
- Les anciens lecteurs peuvent l'ignorer si leur compatibilité le déclare.

### 5.2 Backward-compatible

Change une représentation tout en conservant le même sens et une lecture équivalente : renommage avec alias, élargissement non ambigu d'un type, métadonnée dérivée ou nouveau format de stockage réversible.

- Un adaptateur ou une projection versionnée peut exposer l'ancien contrat.
- La provenance et les identifiants restent identiques.
- Aucun nouveau snapshot métier n'est requis si le comportement observable est strictement équivalent.

### 5.3 Requires adaptation

Exige une transformation ou un backfill vérifié : nouveau champ requis calculable, séparation non ambiguë d'une valeur, normalisation technique ou changement de relation préservant toute l'information.

- La transformation est versionnée, déterministe et auditée.
- L'ancien contrat reste disponible pendant la transition.
- Le résultat transformé ne devient canonique qu'après validation complète.
- Une republication est requise si le résultat constitue une nouvelle interprétation métier plutôt qu'une équivalence technique.

### 5.4 Incompatible

Supprime de l'information, change la sémantique, rend une conversion ambiguë ou invalide un ancien contrat.

- Interdiction de modifier en place les snapshots publiés.
- Nouvelle représentation et nouvelle version de contrat obligatoires.
- Conservation d'un lecteur historique, d'une archive reconstructible ou d'une projection fidèle.
- Nouvelle publication/snapshot si une nouvelle vérité métier est produite.

La classification d'une évolution est documentée avant migration et validée à partir de son impact observable, pas seulement de la syntaxe du schéma.

## 6. Incompatible changes

### Suppression d'un champ

Le champ est d'abord déprécié. Sa lecture historique et sa définition restent conservées. Sa suppression physique n'est autorisée que si toute donnée historique reste reconstruisible et tous les consommateurs concernés ont migré.

### Fusion de champs

La fusion conserve les valeurs sources et la règle utilisée. Si plusieurs anciennes combinaisons produisent la même valeur nouvelle avec perte d'information, l'ancienne représentation reste nécessaire pour l'audit.

### Séparation d'un champ

La séparation utilise une règle déterministe. Les cas ambigus sont mis en quarantaine et ne reçoivent pas de valeurs inventées.

### Changement de sémantique

Un même identifiant de champ ou de valeur ne peut pas recevoir un nouveau sens. Une nouvelle propriété ou valeur versionnée est créée, avec lien de remplacement éventuel.

### Changement de cardinalité

Passer de un-à-un à plusieurs-à-plusieurs ou inversement constitue une évolution de contrat. La réduction de cardinalité exige une règle métier explicite et une republication si plusieurs valeurs historiques ne peuvent pas être représentées sans perte.

### Changement destructif de type

Une conversion non totale ou non réversible ne peut pas remplacer la donnée historique. L'ancienne forme et ses références restent disponibles.

## 7. Vocabulary evolution

Chaque valeur contrôlée possède une identité stable dans une version de vocabulaire.

### Ajouter

- Publier une nouvelle version de vocabulaire.
- Ne pas ajouter rétroactivement la valeur aux anciens snapshots.
- Les anciens lecteurs signalent une valeur non supportée au lieu de la remapper approximativement.

### Déprécier

- Interdire éventuellement la valeur dans les nouvelles publications après une date/version définie.
- Continuer à l'interpréter dans les snapshots historiques.
- Conserver sa définition et sa provenance.

### Remplacer

- Créer une nouvelle valeur et une relation explicite de remplacement.
- Documenter si la conversion est équivalente, contextuelle ou impossible.
- Ne jamais changer le sens de l'ancien identifiant.

### Corriger une valeur historique

- Ne pas modifier le snapshot publié.
- Corriger la règle/vocabulaire, produire un nouveau `PublishedDataset` et, si nécessaire, un snapshot correctif lié par supersession.
- Conserver l'ancien vocabulaire pour reproduire l'état original.

Une valeur historique ne devient jamais invalide uniquement parce qu'elle est dépréciée dans la version courante.

## 8. External mapping evolution

Tout mapping externe possède une famille et une version immuable.

Lorsqu'un mapping change :

1. créer une nouvelle version de mapping ;
2. conserver les anciennes règles et leurs entrées/sorties ;
3. appliquer la nouvelle version uniquement aux nouveaux candidats par défaut ;
4. produire un nouveau `PublishedDataset` pour republier des données anciennes avec la nouvelle interprétation ;
5. produire un nouveau snapshot métier si cette republication modifie l'état canonique exposé ;
6. relier les versions et publications par provenance/supersession.

Une valeur brute d'un ancien `ImportedFact` reste interprétée par le mapping épinglé à sa publication d'origine. Un reprocessing avec le mapping courant est une nouvelle exécution, jamais une relecture silencieuse de l'historique.

## 9. Relation evolution

### Ajout

Un nouveau type de relation est ajouté dans une nouvelle version de vocabulaire/contrat. Son absence historique ne signifie pas que la relation était fausse ; elle n'était pas représentée.

### Cardinalité

Une cardinalité élargie peut être additive si les anciennes valeurs restent valides. Une cardinalité réduite est incompatible si elle perd des relations existantes.

### Changement de cible

Passer d'une carte à un sélecteur, d'un identifiant stable à une version ou inversement crée un nouveau contrat de relation. Les anciennes cibles restent interprétables selon leur version.

### Remplacement ou dépréciation

Une relation est supersédée par une nouvelle assertion ou dépréciée pour les nouvelles écritures. Elle n'est pas supprimée des publications historiques.

### Assertions D-010

`FunctionalAnnotation` et `CardRelation` conservent leur identité, provenance, statut de revue, contexte et version. Une évolution de schéma ne recalcule pas leur confiance ni leur canonicalité.

## 10. Identifier evolution

- Aucun identifiant interne n'est recyclé, renommé ou recalculé par une migration.
- Une migration ne crée pas une nouvelle `Card` pour une correction de contenu.
- Une fusion d'identités future exige une décision métier distincte ; elle n'est jamais un effet secondaire technique.
- Les codes métier dépréciés restent réservés et non réattribués.
- Un identifiant externe modifié crée une nouvelle assertion versionnée ; l'ancienne reste traçable.
- Une collision externe reste en quarantaine selon D-018, sans rapprochement automatique.

Un changement de type physique d'un identifiant interne n'en change ni la valeur logique ni les références historiques.

## 11. Application compatibility

Chaque version de composant déclare les versions de schéma qu'elle peut lire et écrire.

Principes :

- les lecteurs historiques nécessaires restent disponibles ou sont remplacés par un adaptateur fidèle ;
- pendant une transition, le backend, le frontend, les imports, le validateur et les outils d'analyse utilisent une matrice de compatibilité documentée ;
- une version d'écrivain ne produit pas de données qu'un consommateur actif obligatoire ne peut pas interpréter ;
- les capacités nouvelles peuvent être activées après migration et validation, pas uniquement après déploiement du schéma ;
- les snapshots et publications exposent leur `schema_version_ref` ou permettent de la retrouver sans ambiguïté ;
- un composant incompatible échoue explicitement plutôt que d'interpréter approximativement.

D-020 ne définit pas le versionnement détaillé de l'API ni le mécanisme de négociation technique.

## 12. Data migration invariants

Toute migration ou transformation de données canoniques respecte les invariants suivants :

1. **Déterminisme :** mêmes entrées et même version de transformation produisent le même résultat.
2. **Traçabilité :** version, auteur/processus, date, entrées, sorties et diagnostics sont auditables.
3. **Identités conservées :** aucun identifiant stable n'est modifié ou recyclé.
4. **Provenance conservée :** les liens vers sources, faits, mappings et revues survivent.
5. **Sémantique préservée :** un backfill technique ne change pas le contenu métier observable.
6. **Atomicité canonique :** un état partiellement migré ne devient jamais la version canonique active.
7. **Vérifiabilité :** comptages, contraintes, rapports de différences et échantillons/golden cases sont contrôlés.
8. **Idempotence ou reprise définie :** une relance ne produit pas de doublons ni d'état ambigu.
9. **Quarantaine :** les cas non transformables sont isolés et empêchent l'activation lorsque bloquants.
10. **Historique reconstructible :** toute transformation destructive conserve l'ancienne forme ou une preuve suffisante pour la reconstruire fidèlement.

Une migration technique et une republication métier possèdent des identités et journaux séparés.

## 13. Rollback

### Rollback de code

Réactiver une version applicative antérieure n'est autorisé que si elle peut lire le schéma actif. Sinon, un mode de compatibilité ou un déploiement correctif est requis.

### Rollback de schéma

Possible uniquement pour une évolution réellement réversible et sans perte des écritures produites depuis la migration. Un rollback destructif est interdit comme mécanisme ordinaire.

### Rollback de publication

Une publication n'est pas supprimée. Le système peut retirer son statut courant, réactiver explicitement une publication antérieure ou publier un correctif, tout en conservant les deux identités et l'audit.

### Supersession de snapshot

La supersession crée un nouveau snapshot et un lien explicite. Elle ne modifie ni ne supprime le snapshot remplacé ni les résultats qui le référencent.

Un rollback technique ne change jamais silencieusement le snapshot métier épinglé par un résultat.

## 14. Progressive deployment

Une évolution peut utiliser, selon son risque :

1. extension compatible du schéma ;
2. déploiement de lecteurs capables de comprendre ancien et nouveau contrats ;
3. activation d'un nouvel écrivain ;
4. backfill ou transformation traçable ;
5. validation d'intégrité et de reproductibilité ;
6. bascule explicite ;
7. dépréciation puis retrait éventuel de l'ancien chemin.

La double lecture ou double écriture n'est pas une règle générale. Lorsqu'elle est utilisée :

- une source canonique par phase est désignée ;
- les divergences sont mesurées et bloquantes selon une politique définie ;
- la durée et les conditions de sortie sont documentées ;
- les écritures ne créent pas deux vérités indépendantes.

Un backfill n'est activé comme canonique qu'après validation. D-020 ne prescrit aucun outil ou mécanisme de déploiement particulier.

## 15. Migration testing

Avant activation, les tests vérifient au minimum :

- intégrité référentielle et absence de cycles interdits ;
- contraintes d'unicité D-018 ;
- conservation exacte des identifiants ;
- présence et lisibilité des snapshots publiés ;
- conservation de la provenance D-017 ;
- reproduction de résultats historiques de référence ;
- interprétation des anciennes et nouvelles versions de vocabulaire ;
- application du mapping versionné correct ;
- cohérence des liens de supersession ;
- absence de publication canonique partiellement migrée ;
- comparaison des valeurs observables avant/après pour les migrations déclarées équivalentes ;
- comportement d'un rollback ou d'une reprise simulée.

Les fixtures D-009 peuvent fournir des golden cases mécaniquement variés, sans limiter les tests au dataset pilote ni au catalogue de production.

## 16. Schema versioning

Chaque version de schéma possède conceptuellement :

| Information | Rôle |
|---|---|
| `schema_version_ref` | Identité stable de la version |
| `released_at` | Date de mise à disposition |
| `parent_version_ref` | Version précédente ou base de dérivation |
| `compatibility_profile` | Versions lisibles/inscriptibles et capacités |
| `migration_refs` | Transformations permettant d'atteindre cette version |
| `interpretation_contract_ref` | Sémantique des structures et dépendances de vocabulaire |
| `status` | Proposée, active, dépréciée ou retirée pour les nouvelles écritures |

Une version déjà utilisée par une publication ne change jamais de sens. Une correction de son contrat produit une nouvelle version.

Les `PublishedDataset` enregistrent directement la version de schéma utilisée. Les snapshots métier permettent de la retrouver par leurs publications contributrices et peuvent également l'épingler lorsqu'une lecture directe le nécessite.

Aucun outil de migration, format de numéro ou technologie de registre n'est choisi dans D-020.

## 17. Edge cases

| Cas | Comportement attendu |
|---|---|
| Ajout d'une langue | Capacité additive ; nouveaux contenus dans une nouvelle publication/snapshot, aucun faux contenu historique |
| Nouveau type contrôlé | Nouvelle version de vocabulaire ; anciens snapshots restent valides |
| Champ renommé | Alias/adaptateur de contrat ; ancien nom interprétable |
| Champ supprimé des nouvelles écritures | Dépréciation ; lecture historique maintenue |
| Champ scindé avec cas ambigus | Cas en quarantaine ; aucune valeur inventée |
| Deux champs fusionnés avec perte | Ancienne représentation conservée pour l'audit |
| Mapping externe corrigé | Nouvelle version de mapping et nouvelle publication |
| Identifiant fournisseur modifié | Nouvelle référence externe, identité interne inchangée |
| Migration interrompue | Version canonique précédente reste active ; état partiel non publiable |
| Ancien backend après migration | Autorisé uniquement par matrice de compatibilité |
| Rollback après nouvelles écritures | Refusé si perte ou mauvaise interprétation possible |
| Snapshot correctif | Nouvelle identité et supersession ; ancien snapshot intact |
| Valeur dépréciée dans un historique | Toujours lisible avec son vocabulaire épinglé |
| Index physique remplacé | Aucun effet métier si les accès D-019 et résultats restent équivalents |

## 18. Validation questions

1. Les versions de schéma, donnée métier, snapshot, publication, mapping et vocabulaire doivent-elles rester indépendantes, une migration technique ne créant jamais automatiquement un nouveau snapshot métier ?
2. Tout snapshot publié doit-il conserver son contenu et son sens observables après migration, avec lecteur/adaptateur historique lorsque la représentation courante ne suffit plus ?
3. Un changement additif doit-il laisser les anciens champs absents plutôt que d'inventer une valeur, et une nouvelle donnée métier doit-elle passer par une nouvelle publication ?
4. Toute suppression, fusion, séparation ambiguë, réduction de cardinalité ou modification sémantique doit-elle être interdite en place pour les snapshots publiés et introduire une nouvelle représentation versionnée ?
5. Les valeurs de vocabulaire dépréciées doivent-elles rester interprétables historiquement, tout ajout ou remplacement produisant une nouvelle version sans réaffecter les anciens identifiants ?
6. Tout changement de mapping externe doit-il créer une nouvelle version et, pour modifier une donnée canonique existante, une nouvelle publication/snapshot plutôt qu'une réinterprétation rétroactive ?
7. Toute évolution de relation doit-elle préserver l'ancien contrat et ses assertions, les changements de cible, cardinalité ou sens utilisant une nouvelle version explicite ?
8. Les identifiants internes et codes historiques doivent-ils rester immuables et non recyclables pendant toute migration, un changement d'identifiant externe ne créant jamais automatiquement une nouvelle `Card` ?
9. Toute migration de données doit-elle être déterministe, traçable, vérifiable et atomique du point de vue canonique, avec quarantaine des cas non transformables et conservation de la provenance ?
10. Les rollbacks de code, schéma, publication et snapshot doivent-ils rester distincts, aucun rollback technique ne supprimant ou modifiant un snapshot publié ?
11. Les composants applicatifs doivent-ils déclarer une matrice de compatibilité lecture/écriture et utiliser, selon le risque, une transition progressive validée avant activation ?
12. Chaque publication doit-elle épingler sa `schema_version_ref` et son contrat d'interprétation, une version utilisée historiquement ne pouvant jamais changer de sens ?

## 19. Status

D-020 reste **Proposed**. Aucun outil de migration, SQL, schéma physique, code applicatif ou modification de snapshot n'est produit.

## Exclusions

- migrations SQL et fichiers Alembic ;
- choix d'outil de migration ;
- PostgreSQL et schéma physique ;
- backend, frontend et API ;
- modification ou backfill réel des snapshots ;
- données de production.

## Dépendances

- **D-007/D-011 à D-016 :** snapshots métier immuables et contextes reproductibles.
- **D-017 :** publications, provenance et mappings versionnés.
- **D-018 :** identifiants non recyclables et références historiques exactes.
- **D-019 :** accès par snapshot indépendants de l'implémentation physique.


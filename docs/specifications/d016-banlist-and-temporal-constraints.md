# D-016 — Spécification de Banlist, BanlistSnapshot et BanlistEntry

## Statut

**Proposed — validation humaine requise.**

Cette spécification définit le modèle conceptuel et les invariants temporels des banlists. Elle ne crée aucun code, table SQL, ORM, migration, import, dataset ou logique de validation de deck.

## 1. Objectif

D-016 distingue :

- `Banlist` : identité stable d'une famille officielle de restrictions ;
- `BanlistSnapshot` : version publiée, datée et immuable de cette famille ;
- `BanlistEntry` : restriction explicite appliquée à une `Card` stable dans un snapshot.

Le modèle doit permettre de déterminer ultérieurement quelle liste était applicable à un format, une région et une date, tout en conservant les snapshots réellement utilisés par les analyses historiques.

## 2. Identité de Banlist

`Banlist` est l'identité stable d'un régime de limitations officiel associé à un `Format` et à un `RegionalScope`.

```text
Banlist
  banlist_id
  format_id
  regional_scope_id
  created_at
  source_authority_ref
```

- `banlist_id` est une identité interne stable.
- `format_id` rattache la famille de restrictions à `TCG_ADVANCED` au MVP.
- `regional_scope_id` rattache cette famille au périmètre EMEA sans fusionner format et région.
- `source_authority_ref` identifie l'autorité normative attendue, KONAMI selon D-003.

`Banlist` ne contient ni dates d'effet, ni entrées de cartes, ni limitations de copies. Ces informations appartiennent aux snapshots et entrées.

Elle ne se confond ni avec `Format`, `FormatSnapshot`, `CatalogueSnapshot`, `RegionalScope` ou `Card`.

## 3. BanlistSnapshot

`BanlistSnapshot` représente une publication immuable et historiquement reproductible d'une `Banlist`.

| Information | Rôle conceptuel |
|---|---|
| `banlist_snapshot_id` | Identité stable de cette publication |
| `banlist_id` | Famille de banlist concernée |
| `version_ref` | Référence officielle ou interne auditée |
| `published_at` | Instant de publication officielle ou, si distingué, référence vers cette date |
| `effective_from` | Début inclusif de l'effet métier |
| `effective_until` | Fin exclusive éventuelle de l'effet métier |
| `source_ref` | Document ou publication officielle |
| `captured_at` | Instant de collecte par le système |
| `published_in_system_at` | Instant de publication interne après validation |
| `publication_status` | Cycle interne `staged`, `validated`, `published` ou `rejected` déjà établi pour les snapshots |
| `supersedes_snapshot_id` | Snapshot corrigé ou remplacé, si applicable |
| `integrity_report_ref` | Résultat des contrôles de complétude, unicité et résolution |

La date de publication officielle, la collecte et la publication interne sont distinctes. Si la source ne permet pas de distinguer certaines dates, l'incertitude reste explicite plutôt que d'être inventée.

L'intervalle d'effet est `[effective_from, effective_until)`. Un snapshot publié n'est jamais modifié en place. Une correction produit un nouveau snapshot lié par supersession.

D-016 ne choisit pas de stratégie physique de stockage.

## 4. Rattachement au format

La représentation proposée combine stabilité et versionnement sans dupliquer le format :

```text
Banlist ──→ Format
   │
   └──────→ RegionalScope

BanlistSnapshot ──→ Banlist

ReproducibleContext
  ├── FormatSnapshot
  ├── BanlistSnapshot
  ├── CatalogueSnapshot
  └── RegionalScope
```

`Banlist` référence le `Format` stable et le `RegionalScope` stable. `BanlistSnapshot` référence sa `Banlist`, mais ne contient pas par défaut un `FormatSnapshot` : les cycles de publication du cadre de format et de la banlist restent indépendants.

Le contexte reproductible épingle séparément les deux snapshots et vérifie leur compatibilité :

- `FormatSnapshot.format_id` correspond à `Banlist.format_id` ;
- le `RegionalScope` du contexte correspond à celui de `Banlist` ;
- leurs périodes couvrent la date métier demandée ;
- aucune source ou décision explicite ne les déclare incompatibles.

Si une future banlist exige normativement une version précise du cadre de format, cette contrainte pourra être ajoutée comme référence de compatibilité versionnée, sans fusionner les identités.

## 5. Régionalité

La régionalité est portée par la référence de `Banlist` vers le `RegionalScope` stable. Elle n'est ni encodée dans `format_id`, ni héritée implicitement d'un nom, ni répétée sur chaque entrée.

Pour le MVP :

```text
Format = TCG_ADVANCED
RegionalScope = EMEA
```

Cette décision signifie qu'une même identité de format pourrait être associée à une autre famille de banlist régionale ultérieurement, sans créer une taxonomie mondiale ni un format artificiel `TCG_ADVANCED_EMEA`.

## 6. BanlistEntry

Une `BanlistEntry` affirme qu'une carte stable possède une restriction explicite dans un `BanlistSnapshot`.

```text
BanlistEntry
  banlist_snapshot_id
  card_id
  restriction
  source_entry_ref
```

Le vocabulaire contrôlé minimal de `restriction` est :

- `FORBIDDEN` ;
- `LIMITED` ;
- `SEMI_LIMITED`.

Aucune catégorie `UNLIMITED`, `UNKNOWN`, `OTHER` ou `INVALID` n'est ajoutée comme entrée métier. Les données non résolues appartiennent au flux de qualité ; l'absence de restriction explicite est traitée selon la section 9.

Dans un même `BanlistSnapshot`, une carte possède au plus une entrée canonique. Des entrées contradictoires constituent une anomalie bloquante.

## 7. Limite numérique dérivée

La restriction est la source de vérité normative. Le nombre maximal découlant de la banlist est dérivé :

| Restriction | Limite dérivée de banlist |
|---|---:|
| `FORBIDDEN` | 0 |
| `LIMITED` | 1 |
| `SEMI_LIMITED` | 2 |

`max_copies` ne doit pas être stocké comme seconde donnée canonique indépendante, car une divergence avec `restriction` créerait deux vérités contradictoires.

Une projection numérique peut être calculée ou matérialisée pour la lecture et les performances, à condition d'être reconstructible depuis la restriction et la version de la règle. La limite effective d'un deck pourra aussi dépendre des règles générales du format ; D-016 ne les redéfinit pas.

## 8. Portée de BanlistEntry

`BanlistEntry.card_id` référence la `Card` stable définie par D-011, pas un `CardSnapshot`.

Cette décision permet à la restriction de suivre la même identité officielle malgré une correction de nom, de texte, de localisation ou de propriété catalogue. Le contexte d'analyse référence séparément le `CatalogueSnapshot` utilisé pour résoudre et afficher cette carte.

Une entrée ne peut être publiée comme canonique que si la référence officielle a été résolue sans ambiguïté vers un `card_id`. Les identifiants externes et passcodes restent des éléments de mapping et de provenance, pas la clé de l'entrée.

## 9. Cartes absentes de la banlist

L'absence de `BanlistEntry` signifie :

> aucune restriction explicite de banlist n'est publiée pour cette carte dans ce `BanlistSnapshot`.

Elle ne constitue pas une entrée `UNLIMITED`. La limite générale applicable est déterminée ultérieurement par les règles du format.

Cette interprétation n'est valide que si :

- la carte est résolue dans le `CatalogueSnapshot` du contexte ;
- la portée régionale et le format sont compatibles ;
- le `BanlistSnapshot` a passé ses contrôles de complétude et de résolution ;
- aucune entrée candidate non résolue ne pourrait concerner cette carte.

Le résultat conceptuel d'une consultation distingue donc :

- **restriction explicite** : entrée canonique trouvée ;
- **aucune restriction explicite** : absence valide dans un snapshot complet ;
- **indéterminé** : carte, snapshot, mapping ou complétude non résolus.

Ces résultats de consultation ne sont pas de nouvelles valeurs de `BanlistEntry`.

## 10. Cartes inconnues ou hors catalogue

| Situation | Comportement conceptuel |
|---|---|
| Entrée officielle pour une carte absente du catalogue local | Conserver la référence brute en quarantaine, signaler une anomalie de synchronisation et ne pas créer d'entrée canonique avant résolution |
| Carte connue mais `UNAVAILABLE` en EMEA | Évaluer séparément la disponibilité ; une éventuelle entrée de banlist reste un fait de restriction, pas une disponibilité |
| Carte connue mais non publiée à la date du catalogue retenu | Résultat indéterminé/hors contexte ; ne pas appliquer silencieusement une entrée à une identité non résolue dans le snapshot |
| Identifiant externe invalide | Rejeter le fait candidat et tracer l'anomalie |
| Référence ambiguë | Bloquer la résolution et exiger une revue documentée |

Une anomalie non résolue empêche le `BanlistSnapshot` d'être déclaré complet et utilisable pour une conclusion positive de légalité. Aucun rapprochement par nom approximatif ou LLM n'est automatique.

## 11. Temporalité

Les dates ont des rôles distincts :

- `published_at` : publication officielle de la liste ;
- `effective_from` : entrée en vigueur inclusive ;
- `effective_until` : fin d'effet exclusive ;
- `captured_at` : collecte par le système ;
- `published_in_system_at` : publication interne après validation ;
- `analyzed_at` : instant de l'analyse, conservé dans son contexte.

Invariants proposés :

1. `effective_until`, lorsqu'il existe, est strictement postérieur à `effective_from`.
2. Une banlist future peut être publiée avant `effective_from` sans devenir active immédiatement.
3. Pour une même `Banlist`, deux snapshots applicables ordinaires ne doivent pas avoir d'intervalles d'effet se chevauchant.
4. La fin de l'ancien snapshot correspond normalement au début du suivant.
5. Un chevauchement n'est admis que pour une correction explicitement liée par supersession et ne doit jamais rendre la sélection implicite ambiguë.
6. Un snapshot correctif conserve l'ancien ; les analyses déjà épinglées ne changent pas.
7. Une sélection « à la date métier » et une sélection « telle que connue à la date d'analyse » doivent appliquer des politiques distinctes et documentées.

Une liste officiellement publiée à l'avance est sélectionnable pour la planification, mais n'est applicable à la légalité courante qu'à partir de `effective_from`.

## 12. Cohérence temporelle avec FormatSnapshot

Pour une date métier `t`, un contexte cohérent exige :

- `t` dans l'intervalle du `FormatSnapshot` ;
- `t` dans l'intervalle du `BanlistSnapshot` ;
- égalité des identités `Format` attendues ;
- correspondance du `RegionalScope` ;
- `CatalogueSnapshot` capable de résoudre la carte et l'assertion de disponibilité pertinentes ;
- absence d'ambiguïté de supersession selon la politique de lecture choisie.

Les dates de publication peuvent différer des dates d'effet sans incohérence. En revanche, deux banlists ordinaires concurrentes pour la même famille et le même instant constituent une anomalie temporelle bloquante.

D-016 définit ces invariants sans implémenter le résolveur temporel.

## 13. Relation avec CardAvailability

Les assertions suivantes sont indépendantes :

```text
Card ── FORBIDDEN dans BanlistSnapshot
Card ── AVAILABLE / UNAVAILABLE / UNKNOWN dans CatalogueSnapshot + RegionalScope
```

Une carte `FORBIDDEN` peut être commercialement disponible. Une carte indisponible en EMEA n'est pas automatiquement `FORBIDDEN`. La disponibilité constitue une condition régionale d'éligibilité ; la banlist constitue une restriction de copies dans son contexte.

Ni `BanlistEntry` ni `CardAvailability` ne doivent être déduites l'une de l'autre.

## 14. Relation avec les règles de format

D-016 traite uniquement les restrictions explicites de banlist. Les règles générales de construction — limite normale de copies, tailles des zones, catégories autorisées et autres contraintes — appartiennent à `FormatSnapshot` ou à de futurs travaux.

L'absence d'entrée signifie donc « aucune réduction explicite par cette banlist », pas « toute quantité est autorisée ».

## 15. Source et provenance

Chaque snapshot publié conserve conceptuellement :

- la source officielle KONAMI ;
- la référence/version du document ;
- la date de publication officielle ;
- la date d'effet ;
- la valeur brute ou copie d'audit permise ;
- la date de collecte ;
- la version du mapping vers `card_id` ;
- le rapport d'intégrité et le statut de publication interne (`staged`, `validated`, `published` ou `rejected`).

Les niveaux de provenance restent distingués :

- **donnée officielle** : autorité normative de la restriction ;
- **donnée opérationnelle importée** : transport ou représentation secondaire à vérifier ;
- **correction humaine** : action traçable avec auteur, date, justification et sources ;
- **suggestion LLM** : information non canonique et non publiable sans source officielle appropriée et revue humaine.

Une correction humaine ne transforme pas une hypothèse en banlist officielle. Elle peut corriger un mapping ou une transcription lorsque la source normative le justifie.

## 16. Immutabilité et corrections

Une `BanlistSnapshot` publiée est immuable. Lorsqu'une erreur est découverte :

```text
ancien BanlistSnapshot
        ↓ superseded by
nouveau BanlistSnapshot correctif
```

Le correctif indique le snapshot remplacé, sa justification, ses sources et son propre instant de publication interne. L'ancien snapshot reste accessible aux analyses qui l'ont explicitement utilisé.

La politique de lecture choisit explicitement entre l'état original et l'état corrigé selon le besoin d'audit. Aucune mise à jour silencieuse ni stratégie physique particulière n'est décidée ici.

## 17. Reproductibilité

`BanlistSnapshot` reste une pièce indépendante du contexte reproductible :

```text
FormatSnapshot
  + RegionalScope
  + CatalogueSnapshot
  + BanlistSnapshot
  + analyzed_at
```

Une analyse conserve `banlist_snapshot_id`, pas seulement une date ou le mot « actuelle ». Elle reste donc reproductible après une nouvelle liste, une correction ou un changement de catalogue.

## 18. Cas limites obligatoires

| Cas | Comportement conceptuel |
|---|---|
| 1. Carte absente du catalogue mais présente dans la banlist | Quarantaine, anomalie de synchronisation, snapshot non complet pour la légalité jusqu'à résolution |
| 2. Carte connue mais sortie après `effective_from` | Disponibilité évaluée au snapshot/date concernés ; si elle devient disponible pendant la période, l'absence d'entrée signifie seulement aucune restriction explicite |
| 3. Carte catalogue mais indisponible EMEA | Disponibilité négative indépendante, même si aucune restriction de banlist existe |
| 4. Carte `FORBIDDEN` mais disponible commercialement | Deux faits compatibles : disponible et interdite dans le format |
| 5. Carte sans entrée | Aucune restriction explicite uniquement si snapshot complet et contexte résolu |
| 6. Future banlist déjà publiée | Consultable pour planification, applicable seulement à `effective_from` |
| 7. Correction historique | Nouveau snapshot avec supersession ; ancien conservé |
| 8. Intervalles incohérents | Anomalie temporelle bloquant la sélection/publication ordinaire |
| 9. Changement de format | Nouvelle `Banlist` si l'identité `Format` change ; simple contrôle de compatibilité si seul `FormatSnapshot` évolue |
| 10. Plusieurs entrées contradictoires pour une carte | Violation d'unicité, publication bloquée jusqu'à résolution |

## 19. Hors périmètre

D-016 exclut explicitement :

- validation complète de deck ;
- calcul des ratios et probabilités de pioche ;
- génération et recommandations IA ;
- métagame, simulation et matchups ;
- Side Deck et rulings complexes ;
- contenu exhaustif des banlists actuelles ;
- importer de production et dataset physique ;
- SQL, ORM et migrations ;
- API et UI.

## 20. Questions de validation finales

1. Le modèle `Banlist` stable / `BanlistSnapshot` immuable / `BanlistEntry` par carte est-il accepté, sans contenu de liste directement dans `Banlist` ?
2. `Banlist` doit-elle référencer le `Format` et le `RegionalScope` stables, tandis que le contexte reproductible épingle séparément `FormatSnapshot` et `BanlistSnapshot` et vérifie leur compatibilité ?
3. `BanlistEntry` doit-elle référencer la `Card` stable et ne contenir que `FORBIDDEN`, `LIMITED` ou `SEMI_LIMITED`, avec au plus une entrée par carte et snapshot ?
4. Les limites numériques 0/1/2 doivent-elles être dérivées de la restriction et ne jamais devenir une seconde source de vérité canonique ?
5. L'absence d'entrée doit-elle signifier « aucune restriction explicite » uniquement lorsque le snapshot est complet et le contexte résolu, tout autre cas restant indéterminé plutôt que `UNLIMITED` ?
6. Les intervalles `[effective_from, effective_until)`, l'absence de chevauchement ordinaire et la supersession explicite doivent-ils constituer les invariants temporels normatifs ?
7. Une référence de carte inconnue, ambiguë ou non mappée doit-elle placer le fait en quarantaine et empêcher le snapshot d'être déclaré complet pour une conclusion de légalité jusqu'à résolution ?

## 21. Proposition minimale

```text
Banlist
  banlist_id
  format_id
  regional_scope_id
  source_authority_ref
  created_at

BanlistSnapshot
  banlist_snapshot_id
  banlist_id
  version_ref
  published_at
  effective_from
  effective_until?
  source_ref
  captured_at
  published_in_system_at
  publication_status
  supersedes_snapshot_id?
  integrity_report_ref

BanlistEntry
  banlist_snapshot_id
  card_id
  restriction: FORBIDDEN | LIMITED | SEMI_LIMITED
  source_entry_ref
```

Invariants minimaux : identité de carte stable, unicité par carte/snapshot, restriction comme vérité canonique, absence d'entrée conditionnée par la complétude, intervalles non ambigus et corrections par supersession.

## Dépendances

- **D-001 :** fixe TCG Advanced EMEA.
- **D-002 :** impose les banlists officielles, datées, versionnées et immuables.
- **D-003 :** KONAMI est l'autorité normative ; YGOPRODeck reste une source opérationnelle secondaire.
- **D-007 :** impose l'épinglage du snapshot de banlist dans les résultats.
- **D-011 :** impose la référence à `Card` et la séparation avec `CardAvailability`.
- **D-015 :** fixe `Format`, `FormatSnapshot`, `RegionalScope` et les intervalles temporels.


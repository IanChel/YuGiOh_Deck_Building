# D-011 — Spécification conceptuelle de l'entité Card

## Statut

**Proposed — validation humaine requise.**

Ce document définit le modèle fonctionnel de `Card`. Il ne crée aucun modèle de code, enum, table SQL, migration, import ou dataset.

## 1. Définition de Card

`Card` représente l'identité stable d'une carte Yu-Gi-Oh! dans le catalogue interne.

Cette identité :

- ne dépend ni du nom ni de la langue ;
- ne change pas lors d'une correction de texte, traduction, errata ou mise à jour de source ;
- peut être référencée durablement par un deck, une banlist, une annotation ou une relation ;
- ne contient aucune décision de légalité, de recommandation ou de stratégie ;
- ne représente pas une impression physique particulière ni un artwork.

Le concept fonctionnel « carte consultable » est un agrégat composé de :

```text
Card — identité stable
  + CardSnapshot — propriétés intrinsèques dans un CatalogueSnapshot
  + CardLocalization — noms et textes par langue/snapshot
  + CardExternalReference — identifiants des sources
  + CardAvailability — disponibilité territoriale/versionnée
  + CardImage — références d'images autorisées
  + CardArchetypeMembership — appartenance structurelle
  + FunctionalAnnotation / CardRelation — informations stratégiques séparées
```

Cette séparation peut être matérialisée différemment lors de l'implémentation, mais ses frontières sémantiques doivent être conservées.

### Relation normative Card / CardSnapshot / CatalogueSnapshot

```text
Card
  = identité durable

CatalogueSnapshot
  = publication immuable d'un état du catalogue

CardSnapshot
  = représentation des propriétés intrinsèques de Card
    telle qu'exposée dans un CatalogueSnapshot donné
```

- Une `Card` peut apparaître dans zéro, un ou plusieurs `CatalogueSnapshot`.
- Pour une paire `(card_id, catalogue_snapshot_id)`, il existe conceptuellement au plus une représentation de la carte.
- Un `CatalogueSnapshot` regroupe toutes les représentations publiées ensemble et fournit la version de référence aux decks, validations et analyses.
- Une correction de propriété ou un errata apparaît dans un nouveau `CatalogueSnapshot` ; l'ancien `CardSnapshot` demeure observable dans son contexte historique.
- Une carte nouvellement publiée apparaît dans un nouveau snapshot sans altérer les snapshots antérieurs.

Le concept « une représentation de carte par `CatalogueSnapshot` » est adapté au MVP parce qu'il rend la lecture et la reproductibilité explicites. Il s'agit d'une **vue logique**, pas d'une décision de stockage : copie complète, révisions partagées, déduplication ou autre stratégie physique restent ouvertes.

## 2. Identité et identifiants

### Décision proposée

- `card_id` est un entier interne, immuable, généré par l'application et utilisé par toutes les références métier.
- `source_card_id` est l'identifiant de la carte chez une source donnée ; il n'est jamais la clé primaire interne.
- `passcode` est le numéro imprimé de huit chiffres lorsqu'il existe ; il est optionnel et ne doit pas être supposé universel.
- Les identifiants multiples sont portés par `CardExternalReference`, avec `source`, `external_id`, type d'identifiant et période de validité.

Distinction normative :

- `card_id` : identité interne numérique, seule clé métier utilisée entre les modules du produit ;
- identifiant catalogue externe : identifie un enregistrement chez YGOPRODeck, KONAMI ou une autre source ;
- passcode/code officiel : référence imprimée ou officielle lorsqu'elle existe, conservée comme un type de référence externe ;
- identifiant fournisseur : toute autre clé technique propre à une API, un artwork ou une base.

Plusieurs références externes peuvent pointer vers le même `card_id`. Aucune n'est promue implicitement en identité interne principale.

Cette proposition respecte D-004 : l'identité technique principale est numérique tout en évitant de coupler le domaine à YGOPRODeck ou à une source externe.

### Règles d'identité

- Un changement de nom ou de texte ne crée pas une nouvelle `Card` s'il s'agit toujours de la même identité officielle.
- Un alternate artwork ne crée pas une nouvelle `Card` ; il crée une nouvelle référence d'image/impression.
- Une carte distincte portant un nom proche ou réutilisé reçoit un autre `card_id`.
- Un rapprochement incertain entre deux identifiants externes reste non résolu et doit être revu ; aucune fusion automatique destructive.

## 3. Liste des champs proposés

### 3.1 Card — identité stable

| Champ | Type conceptuel | Obligatoire | Nature | Justification |
|---|---|---:|---|---|
| `card_id` | entier positif immuable | Oui | Canonique interne | Référence stable dans tout le domaine |
| `created_at` | instant | Oui | Métadonnée interne | Audit de création de l'identité |

`Card` ne contient pas de nom, texte, statistique, langue ou statut légal directement : ces valeurs peuvent varier ou nécessitent une provenance de snapshot.

`identity_status` et `merged_into_card_id` sont **différés**. Aucun workflow fiable de fusion/dédoublonnage d'identités n'est encore défini. Au MVP, une collision ou correspondance incertaine bloque la publication du snapshot et déclenche une revue d'import ; elle ne produit pas une fusion métier automatique. Ces champs pourront être réintroduits avec un ADR dédié si un cas réel justifie redirection, fusion ou retrait.

### 3.2 CardExternalReference — identifiants de source

| Champ | Type conceptuel | Obligatoire | Nature | Justification |
|---|---|---:|---|---|
| `card_id` | référence `Card` | Oui | Canonique interne | Relie la source à l'identité stable |
| `source_code` | identifiant de `DataSource` | Oui | Externe/versionné | Distingue YGOPRODeck, KONAMI ou une future source |
| `external_id` | chaîne préservée telle que fournie | Oui | Externe | Évite les pertes de zéros ou formats futurs |
| `external_id_type` | type contrôlé (`CATALOGUE_ID`, `PASSCODE`, `DATABASE_CID`, etc.) | Oui | Canonique interne | Évite de confondre passcode et identifiant de base |
| `valid_from_snapshot` | référence de snapshot | Oui | Versionné | Date l'apparition du mapping |
| `valid_to_snapshot` | référence de snapshot | Non | Versionné | Permet de remplacer un mapping sans l'effacer |

Le `passcode` est donc représenté comme référence externe de type `PASSCODE`, et non comme hypothèse obligatoire dans `Card`.

### 3.3 CardSnapshot — propriétés intrinsèques versionnées

| Champ | Type conceptuel | Obligatoire | Nature | Justification |
|---|---|---:|---|---|
| `card_id` | référence `Card` | Oui | Canonique interne | Identité concernée |
| `catalogue_snapshot_id` | référence `CatalogueSnapshot` | Oui | Versionné | Rend les valeurs reproductibles |
| `card_category` | `MONSTER`, `SPELL`, `TRAP` | Oui | Canonique de source | Discriminant principal |
| `monster_frame_kinds` | ensemble contrôlé | Pour Monster | Canonique de source | Représente Normal/Effect/Ritual/Fusion/Synchro/Xyz/Link/Pendulum sans exclusivité erronée |
| `monster_abilities` | ensemble contrôlé | Pour Monster, sinon vide | Canonique de source | Tuner, Flip, Gemini, Union, Spirit, Toon et futures propriétés orthogonales |
| `monster_race` | Dragon, Wyrm, Spellcaster, etc. | Pour Monster | Canonique de source | Filtre et contraintes de cartes |
| `attribute` | DARK, LIGHT, EARTH, WATER, FIRE, WIND, DIVINE | Pour Monster | Canonique de source | Filtre et relations déterministes |
| `level` | entier borné | Conditionnel | Canonique de source | Monstres à Level ; absent pour Xyz/Link et cas non applicables |
| `rank` | entier borné | Conditionnel | Canonique de source | Uniquement monstres Xyz |
| `link_rating` | entier borné | Conditionnel | Canonique de source | Uniquement monstres Link |
| `link_markers` | ensemble de directions contrôlées | Conditionnel | Canonique de source | Uniquement Link ; cohérent avec `link_rating` |
| `pendulum_scale_left` | entier borné | Conditionnel | Canonique de source | Monstres Pendulum uniquement |
| `pendulum_scale_right` | entier borné | Conditionnel | Canonique de source | Préserve les deux emplacements du cadre sans supposer leur égalité éternelle |
| `atk` | entier ou valeur inconnue | Pour Monster lorsque applicable | Canonique de source | Certaines valeurs imprimées sont `?`, distinctes de `NULL` |
| `def` | entier ou valeur inconnue | Pour Monster non-Link lorsque applicable | Canonique de source | Absent pour Link ; `?` distinct de non applicable |
| `spell_trap_property` | `NORMAL`, `CONTINUOUS`, `EQUIP`, `FIELD`, `QUICK_PLAY`, `RITUAL`, `COUNTER` | Pour Spell/Trap | Canonique de source | Classification propre aux Magies/Pièges |
| `has_effect` | booléen dérivé ou classification contrôlée | Oui | Dérivé transparent | Facilite validation/recherche ; ne remplace pas la classification source |
| `source_record_ref` | référence au record brut/import | Oui | Provenance | Permet d'expliquer l'origine de chaque révision |

Les bornes exactes et domaines seront fixés dans le dictionnaire de données, pas dans cette décision conceptuelle.

#### Valeurs numériques conditionnelles

Pour `atk` et `def`, le modèle conceptuel doit distinguer trois états métier sans utiliser une valeur sentinelle numérique :

- **numérique** : une valeur entière est fournie, y compris `0` ;
- **inconnue imprimée** : la valeur officielle est `?` ;
- **non applicable** : la propriété n'existe pas pour cette catégorie de carte, par exemple la DEF d'un monstre Link.

Une absence accidentelle dans la source n'est pas un quatrième état métier : c'est une anomalie de qualité à mettre en quarantaine ou à signaler avant publication. Conceptuellement, une paire simple « état + valeur facultative » suffit pour ATK/DEF ; D-011 n'impose ni classe générique ni type spécialisé par propriété.

Pour `level`, `rank`, `link_rating` et les échelles Pendulum, l'applicabilité découle de la classification. Une valeur numérique est présente lorsqu'elle est applicable et la propriété est absente lorsqu'elle ne l'est pas. Une valeur manquante alors qu'elle est requise constitue également une anomalie de qualité. La valeur spéciale `?` ne doit pas être généralisée à une propriété qui ne la prévoit pas officiellement.

### 3.4 CardLocalization — noms et textes

| Champ | Type conceptuel | Obligatoire | Nature | Justification |
|---|---|---:|---|---|
| `card_id` | référence `Card` | Oui | Canonique interne | Identité traduite |
| `catalogue_snapshot_id` | référence snapshot | Oui | Versionné | Texte reproductible et compatible avec les errata |
| `language_code` | code BCP 47 ou ensemble validé | Oui | Canonique interne | Anglais/français au MVP, extensible |
| `display_name` | chaîne Unicode | Oui pour EN ; facultatif pour FR | Externe/versionné | Nom officiel ou catalogue dans la langue |
| `card_text` | chaîne Unicode | Oui pour EN si fourni | Externe/versionné | Texte d'effet ou description normale principale |
| `pendulum_text` | chaîne Unicode | Pour Pendulum si fourni | Externe/versionné | Zone de texte Pendulum distincte |
| `translation_status` | `OFFICIAL`, `SOURCE_PROVIDED`, `MISSING`, `UNVERIFIED` | Oui | Provenance | Ne présente pas une traduction non vérifiée comme officielle |
| `source_record_ref` | référence au record brut/import | Oui | Provenance | Audit du texte exact |

Frontière finale :

- `CardSnapshot` contient uniquement les propriétés non linguistiques d'une représentation de carte.
- `CardLocalization` contient tous les noms et textes dépendant d'une langue, y compris l'anglais.
- Une localisation est liée à la même paire `card_id + catalogue_snapshot_id` afin qu'un texte corresponde exactement à la version des propriétés.
- L'anglais est la localisation canonique exigée pour publier un snapshot MVP lorsque la source le fournit.
- Le français est facultatif ; son absence déclenche un fallback d'affichage explicite vers la localisation anglaise du même snapshot.
- Une traduction générée par LLM peut être conservée uniquement comme suggestion non canonique hors de `CardLocalization` officielle ; elle ne reçoit jamais `OFFICIAL` ou `SOURCE_PROVIDED` sans validation et provenance adéquates.

Il ne faut donc pas dupliquer `canonical_name_en` ou le texte anglais dans `CardSnapshot`.

### 3.5 CardNameAlias — alias de recherche

| Champ | Type conceptuel | Obligatoire | Nature | Justification |
|---|---|---:|---|---|
| `card_id` | référence `Card` | Oui | Canonique interne | Carte ciblée |
| `language_code` | langue de l'alias | Oui | Métadonnée | Recherche localisée |
| `alias` | chaîne | Oui | Externe ou annotation | Ancien nom, variation contrôlée ou nom mis à jour |
| `alias_kind` | `FORMER_OFFICIAL_NAME`, `SOURCE_VARIANT`, `SEARCH_ALIAS` | Oui | Canonique interne | Distingue nom officiel et aide à la recherche |
| `provenance` | référence de source/annotation | Oui | Provenance | Empêche les alias inventés d'apparaître comme noms officiels |
| `validity` | intervalle/snapshot facultatif | Non | Versionné | Gestion des changements de nom |

Un surnom communautaire n'entre pas automatiquement dans le catalogue canonique ; il exige une politique distincte.

### 3.6 CardArchetypeMembership — appartenance structurelle

| Champ | Type conceptuel | Obligatoire | Nature | Justification |
|---|---|---:|---|---|
| `card_id` | référence `Card` | Oui | Canonique interne | Carte membre/associée |
| `archetype_id` | référence `Archetype` | Oui | Externe/versionné | Appartenance déclarée par la source retenue |
| `catalogue_snapshot_id` | référence snapshot | Oui | Versionné | Reproductibilité |
| `provenance` | référence source | Oui | Provenance | Les archétypes YGOPRODeck restent éditoriaux et traçables |

Pour le MVP, `CardArchetypeMembership` représente uniquement une **appartenance structurelle sourcée**. La distinction `MEMBER` / `EXPLICITLY_LISTED_SUPPORT` est supprimée de la proposition minimale : la source opérationnelle ne fournit pas nécessairement une sémantique assez fiable pour maintenir cette distinction sans interprétation.

Les trois concepts restent séparés :

1. **Appartenance structurelle :** `CardArchetypeMembership`, assertion de catalogue sourcée et versionnée.
2. **Référence explicite dans le texte :** fait linguistique/dérivé pouvant être extrait ultérieurement dans une structure dédiée ; reporté au MVP tant qu'un besoin de recherche ne l'exige pas.
3. **Association stratégique :** annotation D-010 telle que `SUPPORTS` ou `ENABLES`, jamais transformée en appartenance.

Les « séries » non équivalentes à un archétype restent reportées. Une carte peut avoir plusieurs appartenances structurelles si la source et la revue le justifient.

### 3.7 CardImage — référence d'image minimale

| Champ | Type conceptuel | Obligatoire | Nature | Justification |
|---|---|---:|---|---|
| `card_id` | référence `Card` | Oui | Canonique interne | Carte illustrée |
| `external_image_id` | identifiant d'artwork/source | Oui | Externe | Distingue plusieurs artworks |
| `variant_kind` | `PRIMARY`, `ALTERNATE` | Oui | Canonique interne | Désigne l'image principale sans la mettre dans `Card` |
| `resolution_kind` | `THUMBNAIL`, `STANDARD`, `HIGH` | Oui | Métadonnée | Choix d'affichage sans surmodéliser |
| `url_or_storage_ref` | référence d'emplacement | Oui | Externe | URL source ou stockage autorisé, décidé après D-008 |
| `source_code` | source | Oui | Provenance | Droits et retrait |
| `catalogue_snapshot_id` | snapshot | Oui | Versionné | Audit de la référence |

Une image est une **ressource externe référencée par le catalogue**, pas une propriété identitaire ni un blob nécessaire dans `CardSnapshot`. Pour le MVP conceptuel, seuls sont nécessaires : carte, snapshot, source, identifiant/URL externe, caractère principal ou alternatif et variante de résolution disponible. Largeur, hauteur, format MIME, checksum ou stockage local ne seront ajoutés que si le pipeline ou D-008 les exige.

Une URL d'image ne fait pas partie de l'identité stable de `Card`. D-008 reste un prérequis à la publication ou à l'auto-hébergement ; D-011 ne conclut rien sur les droits.

## 4. Classification des champs

### Intrinsèques versionnés

- catégorie Monster/Spell/Trap ;
- familles de cadre et capacités de monstre ;
- race, attribut, Level/Rank/Link, marqueurs, échelles ;
- ATK/DEF ;
- propriété Spell/Trap ;
- noms et textes dans leur langue.

Ils sont « intrinsèques » au sens produit, mais stockés dans un snapshot car une source ou un errata peut les corriger.

### Identité stable

- `card_id` ;
- date de création interne de cette identité.

### Externes

- identifiants YGOPRODeck/KONAMI/passcode ;
- records bruts ;
- textes/noms fournis ;
- URLs ou identifiants d'image ;
- appartenance d'archétype issue d'une source.

### Dérivés persistables et reconstructibles

- nom normalisé pour recherche ;
- texte normalisé/tokenisé pour recherche ;
- `has_effect` ;
- type principal d'affichage ;
- zone structurelle possible Main ou Extra à partir de la classification ;
- expansions carte-à-carte d'un sélecteur D-010.

Ces valeurs peuvent être matérialisées pour les performances, mais leur règle et leurs entrées doivent être versionnées. Elles ne deviennent pas des faits sources.

### Calculés à la volée

- éligibilité au format et à la région pour une date ;
- statut de banlist et nombre de copies autorisées ;
- légalité dans Main/Extra/Side pour un deck donné ;
- rôles applicables à un `DeckState` ;
- candidats de recommandation ;
- texte de fallback de langue.

## 5. Classification Yu-Gi-Oh! extensible

Une seule enum `card_type` contenant des valeurs comme `FUSION_PENDULUM_EFFECT_MONSTER` serait fragile et combinatoire. La classification est décomposée :

```text
card_category
  MONSTER | SPELL | TRAP

monster_frame_kinds (ensemble)
  NORMAL | EFFECT | RITUAL | FUSION | SYNCHRO | XYZ | LINK | PENDULUM

monster_abilities (ensemble)
  TUNER | FLIP | GEMINI | UNION | SPIRIT | TOON | ...

spell_trap_property
  NORMAL | CONTINUOUS | EQUIP | FIELD | QUICK_PLAY | RITUAL | COUNTER
```

Règles :

- `card_category` est exclusif.
- Les ensembles de monstre autorisent les combinaisons réelles, par exemple Fusion/Pendulum/Effect.
- Un mécanisme d'invocation n'est pas confondu avec une race ou une capacité.
- Les combinaisons impossibles sont contrôlées par validation de données, pas par multiplication de champs booléens.
- Une valeur source encore inconnue est mise en quarantaine ou représentée comme `UNKNOWN_SOURCE_VALUE`, jamais silencieusement mappée vers une valeur proche.

## 6. Frontière Card / autres entités

| Concept | Séparation | Motif |
|---|---|---|
| `CardSnapshot` | **Nécessaire** | Les propriétés peuvent être corrigées ; les anciens résultats doivent rester reproductibles |
| `CardLocalization` | **Nécessaire** | Langues, textes, traductions manquantes et errata évoluent indépendamment de l'identité |
| `CardAvailability` | **Nécessaire** | Territoire et date ne sont ni intrinsèques ni équivalents à la banlist |
| `FunctionalAnnotation` | **Nécessaire** | Rôles D-010 contextuels, provenance et revue distincts des faits catalogue |
| `CardRelation` | **Nécessaire** | Graphe source→cible/sélecteur versionné, non contenu dans une ligne Card |
| `CatalogueSnapshot` | **Nécessaire** | Unité immuable de publication et de reproductibilité D-007 |
| `CardExternalReference` | **Recommandée fortement** | Découple les identifiants internes des fournisseurs |
| `CardImage` | **Recommandée** | Plusieurs artworks/résolutions, droits et URLs indépendants |
| `CardNameAlias` | **Recommandée** | Recherche et changements de noms sans polluer les localisations officielles |
| `CardPrinting` / set / rareté | **Prématurée pour le MVP** | Pas nécessaire à la construction/légalité de base, sauf si la légalité régionale exige une trace de sortie plus fine |
| `Series` distinct d'`Archetype` | **Prématurée** | À ajouter seulement si recherche ou relations structurelles le justifient |

## 7. Gestion des cartes particulières

| Cas | Représentation attendue |
|---|---|
| Monstre Normal | `MONSTER`, frame `NORMAL`, `card_text` descriptif, `has_effect=false` |
| Monstre à Effet | frame `EFFECT`, `card_text` d'effet |
| Fusion | frame `FUSION`, généralement zone Extra dérivée |
| Synchro | frame `SYNCHRO`, Level présent, zone Extra dérivée |
| Xyz | frame `XYZ`, Rank présent, Level absent |
| Link | frame `LINK`, Link Rating/markers présents, Level/Rank/DEF absents |
| Ritual | frame `RITUAL`, Level présent ; Main Deck structurel sauf règle future distincte |
| Pendulum | frame `PENDULUM`, échelles et `pendulum_text` présents ; combinable avec Effect/Fusion/Synchro/Xyz selon la carte |
| Tuner/Flip/Gemini/Union/Spirit/Toon | entrée dans `monster_abilities`, combinable avec les frames compatibles |
| Spell | catégorie `SPELL`, propriété Spell/Trap adaptée, statistiques monstre absentes |
| Trap | catégorie `TRAP`, propriété Normal/Continuous/Counter, statistiques monstre absentes |
| ATK/DEF numériques | entier, y compris `0` |
| ATK/DEF `?` | état conceptuel « inconnue imprimée », sans valeur numérique |
| ATK/DEF non applicables | état « non applicable », distinct de `?` et de `0` |
| Carte sans traduction française | localisation FR absente ou `MISSING`, fallback UI anglais explicite |

Les règles d'applicabilité empêchent de confondre `0`, `?`, absent et non applicable.

## 8. Langues, noms et recherche

- L'anglais est la localisation canonique obligatoire du MVP lorsque la source la fournit.
- Le français est une localisation d'affichage optionnelle mais prioritaire dans l'UI.
- Si le français manque, l'UI utilise l'anglais et peut signaler le fallback ; aucune traduction LLM n'est présentée comme officielle.
- La recherche interroge noms anglais, français et alias validés, tous résolus vers le même `card_id`.
- Les formes normalisées (casse, accents, ponctuation) sont des index dérivés, pas des noms canoniques.
- Les anciens noms officiels et noms mis à jour restent dans `CardNameAlias` avec provenance.
- Le texte de carte est localisé et versionné ; le texte normalisé pour recherche est reconstruisible.

## 9. Provenance

### Données intrinsèques/importées

Chaque `CardSnapshot`, localisation, référence externe, appartenance et image pointe vers :

- `DataSource` ;
- `DataSnapshot` ou record brut ;
- date de collecte ;
- version de mapping/import ;
- éventuelle décision de correction manuelle.

### Données dérivées

Elles portent :

- règle/algorithme et version ;
- références des entrées ;
- date de calcul ;
- statut de recalcul possible.

### Annotations et LLM

- Les rôles et relations suivent D-010 et ne sont jamais copiés dans `CardSnapshot`.
- Une annotation humaine possède auteur, justification et trace de revue.
- Une `LLM_SUGGESTION` demeure non canonique jusqu'à promotion humaine documentée.
- `UNKNOWN` reste une valeur d'information, jamais remplacée par une supposition.

## 10. Snapshots et historique

### Appartenance stable

`Card` conserve uniquement l'identité stable et sa date de création interne.

### Appartenance au CatalogueSnapshot

Le snapshot contient ou référence :

- propriétés `CardSnapshot` ;
- localisations disponibles ;
- références externes valides ;
- appartenances d'archétype ;
- références d'images si autorisées ;
- rapport de qualité et provenance.

### Changements entre snapshots

- correction de type/statistique ;
- errata ou texte corrigé ;
- traduction ajoutée/modifiée ;
- nouvel alias ou changement de nom ;
- changement d'association d'archétype dans la source ;
- nouvelle référence d'image ;
- nouvelle carte.

Un nouveau snapshot crée de nouveaux enregistrements versionnés ou de nouvelles références immuables. Il ne met pas à jour en place les valeurs utilisées par un ancien résultat. Une implémentation par copie complète ou révisions dédupliquées est une décision physique ultérieure ; le comportement observable reste identique.

## 11. Disponibilité régionale

`CardAvailability` est séparée de `Card` et de `BanlistEntry`.

Champs conceptuels minimaux :

| Champ | Rôle |
|---|---|
| `card_id` | Carte concernée |
| `catalogue_snapshot_id` | État du catalogue auquel la disponibilité appartient |
| `territory_code` | EMEA au MVP, extensible |
| `format_family` | TCG au MVP |
| `available_from` | Date de première disponibilité vérifiée si connue |
| `available_until` | Exception/retrait éventuel, normalement absent |
| `availability_status` | `AVAILABLE`, `NOT_RELEASED`, `RESTRICTED_EVENT_ONLY`, `UNKNOWN` |
| `evidence_ref` | Source officielle justifiant l'assertion |

La disponibilité répond : « cette carte appartient-elle au pool régional à cette date ? » La banlist répond séparément : « combien de copies sont autorisées dans ce format/snapshot ? »

`CardAvailability` est donc conceptuellement rattachée à `CatalogueSnapshot`, ou à une représentation versionnée strictement équivalente. Une même carte peut avoir une disponibilité différente dans deux snapshots ; l'ancienne assertion reste immuable et consultable. Les dates décrivent la période métier connue, tandis que `catalogue_snapshot_id` indique dans quelle publication cette connaissance a été retenue. Aucune entrée de banlist, limite de copies ou légalité calculée n'est stockée dans cette entité.

`UNKNOWN` ne devient ni automatiquement légal ni automatiquement illégal : le cas d'usage doit appliquer une politique explicite et conservatrice.

## 12. Champs à explicitement exclure de Card

- `is_legal_emea` ou tout booléen global de légalité ;
- `banlist_status` et `allowed_copies` ;
- appartenance au Main/Extra/Side dépendant du format, sauf zone structurelle dérivée ;
- `is_starter`, `is_extender`, `is_generic`, `is_meta` ;
- tous les `FunctionalTag` et `CardRelation` D-010 ;
- score de puissance, popularité ou recommandation ;
- synergies supposées ;
- données de matchup ou tournoi ;
- prix, rareté et collection au MVP ;
- URL d'image unique directement dans l'identité ;
- texte normalisé présenté comme texte officiel ;
- sortie LLM ou explication générée ;
- règles d'invocation exécutables ou rulings complets.

## 13. Risques de surmodélisation

- Créer une enum pour chaque combinaison de types de monstre.
- Créer un champ booléen pour chaque mécanique ou terme du texte.
- Dupliquer les données par langue au niveau de l'identité.
- Confondre carte, impression physique et artwork.
- Copier banlist, disponibilité et stratégie dans `Card`.
- Versionner chaque champ avec une architecture événementielle avant d'en avoir besoin.
- Transformer le texte de carte en moteur de règles exhaustif pendant la Phase 1.
- Matérialiser toutes les relations d'un sélecteur sans besoin de performance mesuré.
- Ajouter dès maintenant sets, raretés, prix, collection, rulings et historiques complets.

## 14. Proposition finale minimale

Pour le MVP, valider conceptuellement les frontières suivantes :

```text
Card
  card_id
  created_at

CardSnapshot
  card_id + catalogue_snapshot_id
  card_category
  monster_frame_kinds / monster_abilities
  monster_race / attribute
  level / rank / link_rating / link_markers
  pendulum scales
  atk / def (numérique, ? ou non applicable)
  spell_trap_property
  source_record_ref

CardLocalization
  card_id + catalogue_snapshot_id + language
  display_name
  card_text
  pendulum_text?
  translation_status
  source_record_ref

CardExternalReference
  plusieurs identifiants externes typés par card_id

CardAvailability
  card_id + catalogue_snapshot_id + territoire/format

CardArchetypeMembership
  appartenance structurelle sourcée uniquement

CardImage (minimal, sous réserve D-008)
```

Les alias, index de recherche et valeurs dérivées sont conservés comme concepts séparés mais peuvent être matérialisés seulement si le besoin d'implémentation le justifie.

## 15. Questions nécessitant validation humaine

1. Valider définitivement `card_id` comme entier interne généré, indépendant du passcode et des fournisseurs.
2. Valider le report de `identity_status` et `merged_into_card_id` jusqu'à la définition d'un véritable workflow de fusion d'identités.
3. Valider le principe logique d'au plus un `CardSnapshot` par paire carte/snapshot, sans choisir la stratégie physique.
4. Valider que tous les noms et textes, anglais compris, résident dans `CardLocalization`, avec français facultatif et fallback anglais.
5. Valider la simplification de `CardArchetypeMembership` à la seule appartenance structurelle sourcée ; les références textuelles explicites restent différées et les associations stratégiques restent dans D-010.
6. Confirmer si les deux échelles Pendulum doivent être conservées séparément dès le MVP.
7. Décider si `CardNameAlias` est inclus dans le MVP ou reporté après la recherche bilingue de base.
8. Confirmer que les séries distinctes des archétypes restent reportées.
9. Décider si les métadonnées minimales d'image peuvent être préparées avant la résolution de D-008, sans ingestion ni publication d'image.
10. Fixer la politique de validation lorsque `CardAvailability.availability_status=UNKNOWN` ; la structure reste indépendante de la banlist.

## 16. Dépendances

- **D-007 — Snapshots immuables :** impose `CatalogueSnapshot` et la non-réécriture de l'historique.
- **D-009 — Dataset pilote :** exercera toutes les variantes de carte mais ne limite ni le schéma ni le catalogue.
- **D-010 — Vocabulaire fonctionnel :** impose que tags, relations, provenance stratégique et suggestions LLM restent hors de `CardSnapshot`.
- **D-008 — Questions juridiques :** conditionne la publication, le stockage et l'affichage des textes/images, sans changer la frontière conceptuelle.


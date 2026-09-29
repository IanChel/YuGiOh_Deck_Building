# D-015 — Spécification de Format, FormatSnapshot et du périmètre TCG Advanced EMEA

## Statut

**Accepted — validée le 2026-09-29.**

Cette spécification définit le contexte conceptuel du format MVP. Elle ne crée aucun code, classe, enum technique, SQL, ORM, migration, import, seed, dataset ou logique de validation de deck.

## 1. Objectif

D-015 sépare explicitement :

1. l'identité stable d'un format ;
2. une représentation temporelle et immuable de ce format ;
3. le périmètre régional ;
4. la disponibilité régionale d'une carte ;
5. sa légalité dans un contexte précis ;
6. la banlist applicable.

La légalité n'est jamais une propriété globale de `Card`. Elle résulte ultérieurement de l'évaluation d'une carte dans un contexte épinglant les snapshots et le périmètre nécessaires.

## 2. Définition de Format

`Format` est l'identité stable d'un cadre de jeu. Pour le MVP, l'identité active est :

```text
TCG_ADVANCED
```

Proposition conceptuelle minimale :

```text
Format
  format_id
  format_code = TCG_ADVANCED
  created_at
```

`Format` :

- reste stable à travers les évolutions temporelles de ses règles ;
- n'est pas une version ou un snapshot ;
- ne contient pas une liste de cartes légales ;
- ne contient pas les limitations de copies ;
- ne contient pas directement une banlist ;
- ne représente pas une région ;
- n'est pas une propriété d'une carte.

D'autres formats pourront disposer ultérieurement de leur propre identité sans modifier le sens de `TCG_ADVANCED`. D-015 ne les énumère ni ne les implémente.

## 3. Définition de FormatSnapshot

`FormatSnapshot` est une représentation publiée, versionnée et reproductible de l'état du cadre de format applicable à une période.

Champs conceptuels minimaux proposés :

| Information | Rôle |
|---|---|
| `format_snapshot_id` | Identité stable de cette version publiée |
| `format_id` | Référence le `Format` stable |
| `version_ref` | Référence ou libellé de version traçable |
| `effective_from` | Début inclusif de la période d'effet |
| `effective_until` | Fin exclusive éventuelle de la période d'effet |
| `published_at` | Instant de publication du snapshot dans le système |
| `source_ref` | Sources normatives ou décisions justifiant cet état |
| `supersedes_snapshot_id` | Référence facultative en cas de correction/succession explicite |
| `integrity_status` | Résultat de la validation documentaire avant publication |

Le snapshot décrit les paramètres et références du cadre de format nécessaires à son interprétation, sans dupliquer le catalogue ni le contenu de la banlist. Il est immuable une fois publié. D-015 ne choisit ni table, copie complète, déduplication ou autre représentation physique.

## 4. Périmètre TCG Advanced EMEA

Le périmètre MVP est la combinaison de trois notions :

- environnement de règles : **TCG** ;
- format : **Advanced**, représenté par `TCG_ADVANCED` ;
- périmètre régional : **EMEA**.

`Europe` n'est jamais utilisé comme synonyme automatique d'`EMEA`. EMEA est le périmètre validé par D-001 et doit conserver une définition et une provenance propres.

La région reste indépendante de l'identité du format :

```text
Format = TCG_ADVANCED
RegionalScope = EMEA
```

Ainsi, une future prise en charge d'un autre périmètre n'impose pas de créer artificiellement un nouveau format régional si le cadre de format reste le même.

## 5. Région et périmètre géographique

Pour le MVP, `RegionalScope` est une petite identité contrôlée et référencée, et non une chaîne libre ni une taxonomie mondiale :

```text
RegionalScope
  regional_scope_id
  scope_code = EMEA
  definition_ref
  source_ref
```

- `regional_scope_id` fournit une identité stable interne.
- `scope_code` fournit le code contrôlé utilisé par le produit.
- `definition_ref` décrit le périmètre retenu sans prétendre modéliser chaque pays.
- `source_ref` permet de tracer l'autorité ou la décision définissant ce périmètre.

D-015 ne crée pas une hiérarchie de continents, marchés ou pays. Un changement futur de définition doit être documenté et versionné sans réinterpréter silencieusement les anciens résultats.

## 6. Relation avec CatalogueSnapshot

`FormatSnapshot` et `CatalogueSnapshot` répondent à des questions différentes :

- `FormatSnapshot` : quel état du cadre de format est retenu ?
- `CatalogueSnapshot` : quelles cartes et données de catalogue étaient publiées ensemble ?
- `BanlistSnapshot` : quelles limitations officielles datées s'appliquent ?
- `RegionalScope` : pour quel périmètre géographique l'évaluation est-elle demandée ?

Le contexte conceptuel de légalité et d'analyse est une composition de références :

```text
FormatSnapshot
  + RegionalScope
  + CatalogueSnapshot
  + BanlistSnapshot
  + analyzed_at
        ↓
Contexte reproductible de légalité / analyse
```

D-015 ne spécifie pas physiquement `BanlistSnapshot`. Il référence le concept issu de D-002 et laisse sa structure détaillée à la décision dédiée.

Un `FormatSnapshot` ne contient pas le `CatalogueSnapshot`. Une analyse épingle les deux afin qu'ils puissent évoluer indépendamment tout en restant reproductibles.

## 7. Disponibilité et légalité

Les états suivants sont conceptuellement possibles et distincts :

| Catalogue | Disponibilité EMEA | Banlist/contexte | Conclusion possible |
|---|---|---|---|
| Carte connue | `AVAILABLE` | Interdite | Connue et disponible, mais interdite |
| Carte connue | `AVAILABLE` | Autorisée | Candidate à la légalité sous réserve des autres règles |
| Carte connue | `UNAVAILABLE` | Quelconque | Non disponible dans le périmètre |
| Carte connue | `UNKNOWN` | Quelconque | Aucune autorisation positive possible |
| Carte absente du snapshot | Non évaluée | Quelconque | Non résolue dans ce contexte de catalogue |

Deux déductions sont interdites :

```text
présente dans le catalogue ⇒ légale
disponible en EMEA ⇒ légale
```

La disponibilité est une condition régionale. La banlist apporte notamment les limitations de copies. La légalité finale dépendra également des règles de construction du format, qui restent hors de D-015.

## 8. Relation avec CardAvailability

D-011 reste normative : `CardAvailability` est liée à `Card`, à `CatalogueSnapshot` et au périmètre concerné. Elle ne devient pas un champ de `Card`.

D-015 précise uniquement que :

- l'évaluation utilise l'assertion de disponibilité du `CatalogueSnapshot` épinglé ;
- la région demandée doit correspondre au `RegionalScope` de l'assertion ;
- `AVAILABLE`, `UNAVAILABLE` et `UNKNOWN` restent distincts ;
- `UNKNOWN`, absence d'assertion ou provenance insuffisante ne valent jamais disponibilité ;
- une évolution de disponibilité est publiée dans un nouveau snapshot sans réécrire l'ancien.

D-015 ne définit ni la source concrète de disponibilité ni son import.

## 9. Relation avec la banlist

D-002 impose des banlists officielles, datées, versionnées et immuables. D-015 établit les règles de rattachement suivantes :

- le contexte d'analyse référence explicitement un `FormatSnapshot` et un `BanlistSnapshot` compatibles ;
- le `BanlistSnapshot` indique le format auquel ses limitations s'appliquent et, si nécessaire, son périmètre réglementaire ;
- la banlist ne fusionne pas avec l'identité `Format` ;
- une nouvelle banlist ne crée pas automatiquement un nouveau `Format` ;
- les limitations de copies restent dans le domaine banlist, jamais dans `Card`, `Format` ou `CardAvailability`.

La définition de Forbidden, Limited, Semi-Limited, des entrées et de leurs dates appartient à la future spécification des banlists.

## 10. Date et temporalité

Quatre notions temporelles sont distinctes :

- `published_at` : instant auquel un snapshot est publié dans le système ;
- `effective_from` : début inclusif de son effet métier ;
- `effective_until` : fin exclusive éventuelle de son effet métier ;
- `analyzed_at` : instant auquel une analyse ou validation est exécutée.

L'intervalle d'effet conceptuel est :

```text
[effective_from, effective_until)
```

L'absence de `effective_until` signifie que la fin n'est pas encore déclarée ; elle ne garantit pas que le snapshot restera indéfiniment applicable.

Pour répondre à « quelle était la situation à cette date ? », le système doit disposer des identifiants de snapshots retenus et de `analyzed_at`. Une résolution à partir de la seule date applique une politique de sélection déterministe qui devra être documentée ; elle ne remplace pas l'épinglage des identifiants dans un résultat enregistré.

Une publication rétroactive ou une correction historique doit conserver la différence entre « état effectif à la date métier » et « information connue/publiée à la date d'analyse ».

## 11. Chevauchements et transitions

| Événement | Conséquence conceptuelle |
|---|---|
| Nouvelle version du cadre de format | Nouveau `FormatSnapshot`, même `Format` si l'identité du cadre demeure |
| Nouvelle banlist | Nouveau `BanlistSnapshot` ; le `Format` reste stable |
| Changement de disponibilité régionale | Nouveau `CatalogueSnapshot`/assertion de disponibilité versionnée |
| Nouvelle sortie de carte | Nouveau `CatalogueSnapshot` contenant la carte et sa disponibilité connue |
| Carte absente puis disponible | Les deux snapshots conservent leurs états respectifs |
| Correction historique | Nouveau snapshot avec lien de supersession ; aucun écrasement |
| Dates proches ou identiques | Identifiants, dates d'effet, publication et supersession lèvent l'ambiguïté |
| Changement à une date précise | Fin exclusive de l'ancien intervalle et début inclusif du nouveau |

Deux snapshots publiés ne doivent pas être sélectionnables de façon ambiguë pour le même format et le même instant selon une même politique. En cas de correction couvrant une période existante, la relation de supersession et la politique « tel que connu » ou « corrigé » doivent être explicites.

Un résultat historique déjà épinglé conserve toujours ses références originales, même lorsqu'un snapshot correctif est publié.

## 12. EMEA et données externes

La présence d'une carte dans YGOPRODeck prouve seulement qu'elle est connue de la source opérationnelle. Elle ne prouve pas sa disponibilité EMEA.

Pour EMEA, le système doit pouvoir conserver :

- `AVAILABLE` ;
- `UNAVAILABLE` ;
- `UNKNOWN`.

Chaque assertion conserve sa source, son snapshot de provenance, sa date de collecte et toute correction humaine. D-015 n'invente aucune règle de disponibilité et ne transforme ni une absence de donnée ni une présence chez YGOPRODeck en autorisation.

## 13. Identité et immutabilité

- `Format` et `RegionalScope` possèdent des identités stables.
- `FormatSnapshot`, `CatalogueSnapshot` et `BanlistSnapshot` possèdent des identités versionnées.
- Un snapshot publié est immuable.
- Une correction significative produit un nouveau snapshot et une relation explicite avec la version corrigée ou remplacée.
- Un ancien résultat reste lié aux identifiants exacts qui l'ont produit.
- Le libellé d'un format ou d'une région peut évoluer sans changer arbitrairement son identité ni son sens historique.

Ces règles appliquent D-007 sans imposer une stratégie de stockage.

## 14. Relation avec la future validation de deck

La future validation d'une carte ou d'un deck recevra au minimum :

```text
Card
  + CatalogueSnapshot
  + FormatSnapshot
  + BanlistSnapshot
  + RegionalScope
```

Ce contexte permet de séparer existence dans le catalogue, disponibilité régionale, restrictions de banlist et règles du format. D-015 ne définit ni ordre algorithmique, ni messages d'erreur, ni validation de Main/Extra/Side Deck.

## 15. Reproductibilité

Une analyse reproductible doit enregistrer :

- `format_id` ;
- `format_snapshot_id` ;
- `regional_scope_id` ;
- `catalogue_snapshot_id` ;
- `banlist_snapshot_id` ;
- `analyzed_at` ;
- ultérieurement, la version du moteur de règles ou d'analyse lorsqu'elle existera.

Les sources, dates d'effet et dates de publication sont récupérables depuis les snapshots référencés. Le résultat ne doit pas dépendre implicitement du « dernier snapshot » au moment de sa relecture.

D-015 ne définit pas le modèle complet de provenance des analyses.

## 16. Hors périmètre

D-015 exclut explicitement :

- contenu détaillé des banlists ;
- règles complètes de construction de deck ;
- validation algorithmique ;
- YDK ;
- scoring, recommandations et métagame ;
- prix et collection ;
- matchmaking et simulation de duel ;
- API et UI ;
- SQL, ORM, migrations et index ;
- importer de production ;
- catalogue physique et dataset de production.

## 17. Décisions finales validées

1. `Format` est une identité stable et `FormatSnapshot` une représentation temporelle immuable. Le snapshot ne contient directement ni liste de cartes ni contenu de banlist.
2. EMEA est une identité contrôlée minimale `RegionalScope`, distincte de `Format`, avec définition et provenance propres. Aucune taxonomie mondiale n'est introduite au MVP.
3. Les périodes d'effet utilisent les intervalles `[effective_from, effective_until)`, distincts de `published_at` et `analyzed_at`. Toute correction historique produit un nouveau snapshot avec supersession explicite, sans réécriture.
4. Le contexte reproductible épingle séparément `FormatSnapshot`, `RegionalScope`, `CatalogueSnapshot` et le futur `BanlistSnapshot`. Aucune fusion physique prématurée de ces concepts n'est décidée.
5. Une conclusion positive de légalité exige une disponibilité `AVAILABLE` dans le contexte régional et catalogue retenu, puis l'évaluation de la banlist et des règles du format. `UNKNOWN`, `UNAVAILABLE` ou l'absence d'information n'autorisent jamais une conclusion positive implicite.

## 18. Proposition minimale

```text
Format
  format_id
  format_code = TCG_ADVANCED
  created_at

RegionalScope
  regional_scope_id
  scope_code = EMEA
  definition_ref
  source_ref

FormatSnapshot
  format_snapshot_id
  format_id
  version_ref
  effective_from
  effective_until?
  published_at
  source_ref
  supersedes_snapshot_id?
  integrity_status

ReproducibleContext
  format_snapshot_id
  regional_scope_id
  catalogue_snapshot_id
  banlist_snapshot_id
  analyzed_at
```

`ReproducibleContext` est ici une composition conceptuelle de références, pas une nouvelle décision de persistance.

## Dépendances

- **D-001 :** fixe le périmètre MVP TCG Advanced EMEA.
- **D-002 :** impose des banlists officielles, datées, versionnées et immuables.
- **D-003 :** YGOPRODeck est la source opérationnelle du catalogue, pas une preuve de disponibilité EMEA.
- **D-007 :** impose l'épinglage des snapshots et de la date d'analyse.
- **D-011 :** sépare `CardAvailability`, banlist et identité de carte.


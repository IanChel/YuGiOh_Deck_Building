# D-012 — Spécification conceptuelle de CardLocalization

## Statut

**Accepted — validée le 2026-09-29.**

Ce document précise la représentation multilingue des noms et textes de cartes. Il ne crée aucun code, schéma SQL, ORM, migration, import ou dataset et ne tranche aucune question juridique relevant de D-008.

## 1. Définition

`CardLocalization` représente les données linguistiques officielles ou validées d'une `Card` dans une langue et dans l'état historique d'un catalogue.

Une localisation est identifiée conceptuellement par :

- un `card_id`, qui désigne l'identité stable définie par D-011 ;
- un `language_code` ;
- un `catalogue_snapshot_id`, qui fixe la version publiée des données.

Le rattachement normatif est direct à `Card` et à `CatalogueSnapshot`. La localisation correspond au `CardSnapshot` de la même paire `(card_id, catalogue_snapshot_id)`, mais ne dépend pas de l'identité physique d'un enregistrement `CardSnapshot`. Cette frontière permet de faire évoluer ou dédupliquer le stockage ultérieurement sans modifier le contrat conceptuel.

Pour une combinaison `(card_id, catalogue_snapshot_id, language_code)`, il existe au plus une localisation publiée. Une correction linguistique produit un nouveau `CatalogueSnapshot` et n'écrase jamais la localisation utilisée par un résultat historique.

`CardLocalization` contient uniquement des données dépendant de la langue. Les propriétés non linguistiques restent dans `CardSnapshot`.

## 2. Champs

### Champs minimaux proposés

| Champ | Obligatoire | Rôle conceptuel |
|---|---:|---|
| `card_id` | Oui | Référence l'identité stable de la carte |
| `catalogue_snapshot_id` | Oui | Rend la localisation historique et reproductible |
| `language_code` | Oui | Identifie la langue de la localisation |
| `official_name` | Requis pour `COMPLETE` ou `PARTIAL` | Nom officiel ou nom de catalogue validé dans cette langue ; absent pour `UNAVAILABLE` |
| `effect_text` | Conditionnel | Texte d'effet des cartes qui en possèdent un |
| `normal_text` | Conditionnel | Description d'un Monstre Normal ou autre texte non traité comme effet |
| `pendulum_text` | Conditionnel | Texte propre à la zone Pendulum |
| `localization_status` | Oui | État métier limité à `COMPLETE`, `PARTIAL` ou `UNAVAILABLE` |
| `provenance_ref` | Oui | Référence la source et, le cas échéant, la validation humaine |

Le MVP n'ajoute pas de champs séparés pour le coût, les conditions, les matériaux, les citations, les rulings ou chaque paragraphe du texte. Les retours à la ligne du texte source sont préservés dans les champs textuels pertinents.

### Absence, vide et indisponibilité

- **Non applicable :** le champ n'a pas de sens pour cette carte, par exemple `pendulum_text` pour une carte non-Pendulum. Il est absent.
- **Vide valide :** la source affirme que le champ applicable est intentionnellement vide. Ce cas exceptionnel doit rester distinct de l'absence technique.
- **Indisponible :** la donnée linguistique attendue n'a pas été obtenue ou validée. L'état est explicite ; une chaîne vide ne sert jamais à représenter cette situation.
- **Localisation partielle :** le nom est connu, mais au moins un texte applicable manque ou n'est pas validé.

Les seuls états métier sont :

- `COMPLETE` : tous les champs linguistiques applicables et attendus sont disponibles et validés ;
- `PARTIAL` : au moins un champ est disponible et validé, mais un autre champ applicable et attendu manque ;
- `UNAVAILABLE` : aucune localisation publiable n'est disponible dans cette langue.

Une absence technique, une incohérence entre champs ou une rupture de référence est une anomalie de qualité ou d'intégrité. Elle ne crée jamais un quatrième état métier.

Une langue dont l'indisponibilité est connue est représentée explicitement par une `CardLocalization` au statut `UNAVAILABLE`. L'absence inattendue de l'enregistrement requis constitue une anomalie technique signalée par le contrôle de qualité ; elle n'équivaut pas à `UNAVAILABLE`.

## 3. Langues

- `en` est la langue canonique du MVP et doit être présente pour publier une carte lorsque la source de référence fournit ces données.
- `fr` est la langue d'interface prioritaire, mais sa localisation est facultative.
- Une localisation française `UNAVAILABLE` ne bloque pas la présence de la carte dans le catalogue si l'anglais canonique est disponible.
- Une localisation anglaise attendue mais absente place la carte dans un état de données non résolues ; elle ne peut pas être masquée par une traduction française ou générée.
- `language_code` utilise un code de langue validé et extensible. Le MVP limite les valeurs acceptées à `en` et `fr` sans créer une énumération exhaustive mondiale.
- Ajouter ultérieurement une langue consiste à autoriser un nouveau code et de nouvelles localisations ; la structure de `Card`, `CardSnapshot` et `CardLocalization` ne change pas.

La « langue canonique » désigne la langue de référence fonctionnelle du catalogue MVP, pas une supériorité sur les noms officiels publiés dans d'autres langues.

## 4. Fallback français → anglais

Le fallback est une règle de résolution de lecture. Il ne crée, ne traduit et ne modifie aucune donnée persistée.

### Affichage

Pour une demande `fr` :

1. utiliser le champ français demandé s'il est disponible et validé dans le snapshot ;
2. sinon utiliser le champ anglais correspondant du même snapshot ;
3. si l'anglais est également indisponible, retourner un état explicite de donnée manquante.

Le fallback s'applique champ par champ pour une localisation partielle. Une même réponse peut donc afficher le nom français et le texte anglais, mais doit exposer la langue réellement utilisée pour chacun afin de ne pas présenter un mélange comme une localisation française complète.

### Recherche

Une recherche en interface française interroge les noms français, anglais et les alias validés des deux langues. Elle ne remplace pas les noms absents et ne crée aucun alias automatiquement.

### Import YDK

Le fallback n'intervient pas. La résolution utilise l'identifiant externe contenu dans le fichier, puis le `card_id` interne.

### Export

Le contenu structurel exporté utilise les identifiants requis par le format cible. Un nom éventuellement affiché à l'utilisateur est décoratif et résolu avec la règle de fallback sans affecter l'identifiant exporté.

### API

L'API applique la politique de fallback commune et retourne, pour chaque champ linguistique, la valeur, la langue demandée, la langue effectivement servie et l'indication de fallback ou de donnée manquante.

### Analyse interne

Les règles, validations, relations et calculs utilisent `card_id` et les propriétés structurées. Un texte localisé n'est utilisé que par une analyse linguistique explicitement versionnée ; il doit alors être demandé dans une langue précise, sans fallback silencieux.

## 5. Recherche multilingue

La recherche résout des formes linguistiques vers un ou plusieurs `card_id` ; elle ne change jamais l'identité d'une carte.

Les candidats de recherche proviennent de :

- `official_name` anglais et français du snapshot interrogé ;
- `CardNameAlias` validés et applicables à ce snapshot ;
- formes d'index dérivées de ces valeurs, reconstructibles et non canoniques.

La normalisation de recherche peut harmoniser de manière documentée :

- casse ;
- accents et formes Unicode ;
- apostrophes droites ou typographiques ;
- tirets ;
- ponctuation ;
- espaces multiples ;
- variantes typographiques strictement équivalentes.

La valeur originale reste préservée. La normalisation ne doit pas supprimer assez d'information pour fusionner silencieusement deux noms distincts. Une recherche exacte est prioritaire ; les correspondances normalisées ou par alias sont classées comme telles.

Une requête telle que « Dragon Blanc » peut résoudre vers le même `card_id` que « Blue-Eyes White Dragon » si ces formes sont présentes comme nom officiel ou alias validé. Une collision produit plusieurs candidats ou un état ambigu ; elle n'est jamais résolue arbitrairement.

L'algorithme, les pondérations et les index physiques sont différés.

## 6. CardNameAlias

`CardLocalization` porte le nom officiel courant d'une carte dans une langue et un snapshot. `CardNameAlias` porte une autre forme validée qui permet de retrouver la même identité sans prétendre être le nom officiel courant.

### Champs conceptuels minimaux

| Champ | Obligatoire | Rôle conceptuel |
|---|---:|---|
| `card_id` | Oui | Carte résolue par l'alias |
| `language_code` | Oui | Langue ou contexte linguistique de l'alias |
| `alias` | Oui | Forme alternative préservée |
| `alias_kind` | Oui | Ancien nom officiel, variante de source, translittération validée ou variante de recherche contrôlée |
| `valid_from_snapshot` | Oui | Première publication où l'alias est reconnu |
| `valid_to_snapshot` | Non | Fin éventuelle de validité de l'alias |
| `provenance_ref` | Oui | Source ou validation de l'alias |

Peuvent être des alias :

- anciens noms officiels anglais ou français ;
- variantes attestées d'une source retenue ;
- translittérations explicitement validées ;
- variantes typographiques qui nécessitent une conservation explicite au-delà de la normalisation.

Ne sont pas admis automatiquement : surnoms communautaires, traductions libres, approximations, termes stratégiques ou synonymes inventés. Un alias identique au nom ou à l'alias d'une autre carte reste autorisé comme fait sourcé, mais la résolution devient ambiguë et doit le signaler.

## 7. Canonicalité

Pour le MVP :

- le **nom canonique de catalogue** est `official_name` de la localisation anglaise publiée dans le `CatalogueSnapshot` concerné ;
- le **nom français officiel** est le nom publié et validé dans la localisation française du même snapshot ; il n'est ni un alias ni le nom canonique anglais ;
- un **ancien nom** est conservé comme `CardNameAlias` et reste lié au même `card_id` si l'identité de carte n'a pas changé ;
- un **alias** facilite la résolution sans devenir un nom officiel ;
- une **traduction non vérifiée** ou une sortie LLM n'est ni canonique, ni officielle, ni utilisable comme fallback officiel.

La canonicalité est toujours relative à un snapshot. Une correction officielle du nom anglais devient canonique dans le nouveau snapshot sans réécrire l'ancien.

## 8. Provenance

Toute localisation publiée conserve au minimum :

- la source des noms et textes ;
- le snapshot ou enregistrement source ;
- la date de collecte ;
- la version de la règle de mapping ;
- toute correction humaine appliquée ;
- pour une correction, l'identité du réviseur, la date, la justification et les sources examinées.

Les origines conceptuelles admises sont : source officielle, source catalogue identifiée, correction humaine documentée ou autre source explicitement validée. Cette origine reste explicite et ne doit pas être déduite du seul fait que la localisation a été publiée.

Une traduction LLM peut uniquement exister comme suggestion séparée, non publiée et non utilisée par le fallback officiel. Sa promotion exige une revue humaine documentée, une vérification contre des sources acceptables et la conservation du lien vers la suggestion initiale. Après approbation, la donnée publiée est qualifiée de localisation humaine validée ; elle n'est jamais présentée comme une traduction officielle de l'éditeur sans provenance permettant explicitement cette qualification.

D-012 définit cette traçabilité fonctionnelle sans résoudre l'autorisation de collecter, conserver ou publier les textes ; cette question reste dans D-008.

## 9. Snapshots et historique

Sont versionnés par `CatalogueSnapshot` :

- les localisations et chacun de leurs champs publiés ;
- leur statut de complétude ;
- leur provenance publiée ;
- les alias reconnus et leur période de validité ;
- les formes d'index dérivées, directement ou par la version de leur règle de construction.

Restent stables :

- `card_id` ;
- l'identité conceptuelle de la carte ;
- les références d'un ancien résultat à son `CatalogueSnapshot`.

Ainsi, si le Snapshot A contient `Old Name` et le Snapshot B `Updated Name`, un résultat produit avec A continue d'afficher `Old Name`. La même règle s'applique à une correction française, à un texte anglais corrigé et à un alias ajouté ou retiré.

Un consommateur qui demande explicitement le dernier snapshot accepte les données les plus récentes. Un consommateur qui reproduit un résultat utilise obligatoirement le snapshot enregistré avec ce résultat.

## 10. Import / export YDK

- L'import YDK résout le passcode ou identifiant externe du fichier via `CardExternalReference`, puis obtient le `card_id`.
- Aucun import YDK ne dépend du nom affiché, de la langue de l'interface ou du fallback.
- L'export YDK utilise l'identifiant exigé par le format, jamais un nom localisé comme clé.
- Les noms peuvent être présentés dans une prévisualisation ou un rapport d'erreur, avec langue effectivement servie et fallback explicite.
- Un identifiant externe inconnu ou ambigu produit une erreur de résolution ; un rapprochement par nom ne le remplace pas silencieusement.

Ces règles sont conceptuelles et ne définissent ni parseur ni format physique.

## 11. API / UI

Pour une demande `language=fr`, l'API fournit conceptuellement :

- le `card_id` et le `catalogue_snapshot_id` ;
- les champs linguistiques résolus ;
- la langue demandée ;
- la langue réellement utilisée par champ ;
- un indicateur de fallback ;
- l'état complet, partiel ou manquant ;
- éventuellement les noms officiels disponibles nécessaires à la recherche ou au changement de langue.

La politique de résolution appartient à un service partagé côté domaine/API. Le frontend affiche la valeur et peut signaler le fallback, mais ne réimplémente pas l'ordre français → anglais ni les règles de complétude.

L'API doit également permettre, pour l'audit ou l'édition, de demander une localisation brute sans fallback. Une réponse résolue ne doit jamais être persistée comme si elle était une localisation française réelle.

## 12. Cas limites

| Cas | Comportement conceptuel |
|---|---|
| Traduction française absente | Servir l'anglais avec fallback explicite ; ne pas créer de ligne française fictive |
| Traduction française partielle | Résoudre champ par champ et exposer les langues réellement servies |
| Traduction française obsolète | La conserver dans l'ancien snapshot ; publier la correction dans un nouveau snapshot |
| Deux formes françaises proches | Conserver les valeurs sourcées et retourner plusieurs candidats si la résolution est ambiguë |
| Ancien nom anglais ou français | Le conserver comme alias versionné avec provenance |
| Alias identique au nom d'une autre carte | Signaler une collision ; ne jamais sélectionner arbitrairement un `card_id` |
| Accents | Conserver la forme originale ; autoriser une forme normalisée pour la recherche |
| Apostrophes et tirets | Harmoniser uniquement dans l'index dérivé ; préserver la forme officielle |
| Parenthèses et caractères spéciaux | Préserver ; une normalisation ne doit pas effacer une distinction sémantique |
| Variantes d'espaces ou Unicode | Normaliser dans l'index avec une règle versionnée |
| Texte sur plusieurs lignes | Préserver les retours à la ligne de la source |
| Nom français présent, texte français absent | Afficher le nom français et le texte anglais avec métadonnées de fallback par champ |
| Anglais canonique absent | Retourner un état non résolu et bloquer toute publication qui exige ce champ |

Les collisions sont des résultats structurés de recherche ou des anomalies de qualité selon leur origine. Elles ne sont jamais réparées par une traduction, un fuzzy match ou une suggestion LLM automatique.

## 13. Exclusions

Ne font pas partie de `CardLocalization` :

- légalité et validation de deck ;
- banlist et nombre de copies autorisées ;
- disponibilité TCG Advanced EMEA ;
- propriétés structurelles de `CardSnapshot` ;
- `FunctionalTag` et `CardRelation` D-010 ;
- appartenance d'archétype ;
- scoring, recommandations et explications IA ;
- puissance, popularité et métagame ;
- prix, rareté et collection ;
- rulings et interprétations exhaustives ;
- données de duel, compte ou tournoi ;
- identifiants externes et passcodes ;
- suggestions LLM non validées ;
- texte normalisé présenté comme texte officiel.

## 14. Proposition minimale

```text
Card
  card_id

CatalogueSnapshot
  catalogue_snapshot_id

CardSnapshot
  card_id + catalogue_snapshot_id
  propriétés non linguistiques

CardLocalization
  card_id + catalogue_snapshot_id + language_code
  official_name
  effect_text?
  normal_text?
  pendulum_text?
  localization_status
  provenance_ref

CardNameAlias
  card_id + language_code + alias
  alias_kind
  valid_from_snapshot
  valid_to_snapshot?
  provenance_ref
```

Règles minimales :

- une localisation publiée au maximum par carte, snapshot et langue ;
- anglais canonique requis lorsque disponible à la source ; français facultatif ;
- résolution d'affichage française puis anglaise, champ par champ et sans écriture ;
- recherche sur noms officiels et alias validés en anglais et français ;
- ambiguïtés retournées explicitement ;
- noms/textes et alias historiques conservés par snapshot ;
- provenance obligatoire ; aucune suggestion LLM non revue dans les données publiées ;
- aucune architecture linguistique supplémentaire avant qu'un besoin réel ne l'exige.

## 15. Décisions finales validées

1. `CardLocalization` est normativement rattachée à `(Card, CatalogueSnapshot)` et reste cohérente avec le `CardSnapshot` correspondant.
2. Le fallback français → anglais s'applique champ par champ aux localisations françaises partielles. Le contrat indique la langue réellement servie pour chaque champ.
3. Les états métier sont strictement limités à `COMPLETE`, `PARTIAL` et `UNAVAILABLE`. Toute absence technique ou incohérence relève de la qualité ou de l'intégrité des données.
4. Les translittérations validées et variantes provenant de sources identifiées peuvent être des `CardNameAlias`. Les surnoms communautaires, traductions libres et variantes non validées restent hors MVP.
5. Une correction humaine validée peut être publiée comme localisation humaine validée lorsque sa source n'est pas l'éditeur officiel. Elle ne reçoit la qualification « officielle » que si sa provenance l'autorise explicitement.

## Dépendances

- **D-007 — Snapshots immuables :** impose la conservation des anciennes localisations et des règles de résolution reproductibles.
- **D-008 — Cadre juridique :** reste seule compétente pour l'utilisation et la publication des noms et textes.
- **D-011 — Entité Card :** fixe l'identité stable, la frontière linguistique et l'existence de `CardNameAlias`.


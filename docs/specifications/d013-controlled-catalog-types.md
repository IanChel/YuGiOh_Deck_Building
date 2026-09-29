# D-013 — Spécification des types contrôlés du catalogue

## Statut

**Accepted — validée le 2026-09-29.**

Cette spécification définit le vocabulaire conceptuel minimal des propriétés intrinsèques de `CardSnapshot`. Elle ne crée ni code, classe, enum technique, SQL, ORM, migration, import, seed ou dataset.

## 1. Objectif

D-013 remplace les chaînes libres par des vocabulaires contrôlés stables pour les propriétés intrinsèques des cartes : type principal, classifications et capacités de monstres, race, attribut, propriétés Spell/Trap, Level, Rank, Link, Pendulum, ATK et DEF.

Une valeur contrôlée possède une signification documentée et un identifiant conceptuel stable. Le vocabulaire reste extensible par une procédure versionnée ; il n'essaie pas d'anticiper toutes les évolutions futures du jeu.

## 2. Relation avec D-011

Les types D-013 structurent principalement les champs non linguistiques de `CardSnapshot`, versionnés par `CatalogueSnapshot`. Ils décrivent une carte, pas son usage stratégique ni sa légalité.

Restent hors de ces types :

- légalité, banlist et disponibilité EMEA ;
- `FunctionalTag` et `CardRelation` D-010 ;
- scoring, popularité, prix et collection ;
- rulings et interprétations de texte ;
- associations stratégiques ;
- données de deck, de duel ou de YDK.

Une information n'entre dans D-013 que si elle constitue une propriété intrinsèque de la représentation versionnée d'une carte.

## 3. Type principal de carte

Le champ conceptuel `card_category` accepte exactement, pour le MVP :

- `MONSTER` ;
- `SPELL` ;
- `TRAP`.

Chaque `CardSnapshot` publié possède exactement une catégorie principale. `PENDULUM`, `RITUAL`, `FUSION` ou `LINK` ne sont jamais des catégories principales : ce sont des classifications applicables aux monstres ou des propriétés Spell/Trap selon le contexte.

Une catégorie absente, externe non reconnue ou contradictoire bloque la publication du record concerné et produit une anomalie de qualité. `UNKNOWN` peut décrire temporairement l'état d'une donnée en staging, mais n'est pas une quatrième catégorie métier publiée.

## 4. Sous-classifications des monstres

Le champ conceptuel `monster_classifications` est un **ensemble de classifications contrôlées**, et non une valeur unique ni une collection de colonnes booléennes :

- `NORMAL` ;
- `EFFECT` ;
- `RITUAL` ;
- `FUSION` ;
- `SYNCHRO` ;
- `XYZ` ;
- `LINK` ;
- `PENDULUM`.

Ce choix permet de représenter les cartes hybrides, par exemple `FUSION + PENDULUM + EFFECT`, sans créer une valeur combinatoire pour chaque combinaison ni modifier le schéma à chaque nouveauté.

Règles conceptuelles :

- l'ensemble est applicable uniquement à `MONSTER` ;
- plusieurs valeurs peuvent coexister ;
- les combinaisons admises sont contrôlées par des règles d'intégrité versionnées ;
- l'ordre des valeurs n'a aucune signification ;
- une valeur absente de l'ensemble signifie que la classification ne s'applique pas ; elle ne signifie pas que la donnée est inconnue ;
- l'ensemble entier inconnu ou incohérent est un état de donnée à résoudre avant publication.

Le nom `monster_classifications` est retenu conceptuellement afin de ne pas laisser entendre que toutes les valeurs désignent des « frames » au même sens. Il ne prescrit aucun nom de colonne ni aucune représentation physique.

`NORMAL` et `EFFECT` restent des classifications de cadre/texte, distinctes de la présence d'un rôle stratégique. La liste MVP pourra être étendue si une nouvelle classification intrinsèque officielle apparaît.

## 5. Capacités orthogonales des monstres

Le champ conceptuel `monster_abilities` est un ensemble contrôlé distinct de `monster_classifications` :

- `TUNER` ;
- `FLIP` ;
- `GEMINI` ;
- `UNION` ;
- `SPIRIT` ;
- `TOON`.

Ces capacités sont orthogonales : un monstre peut en posséder zéro, une ou plusieurs lorsque les données officielles le permettent. Une absence dans l'ensemble signifie « capacité non déclarée », sous réserve que l'ensemble ait été validé.

Elles ne sont ni :

- la race du monstre ;
- son attribut ;
- son mode d'invocation ou cadre ;
- un `FunctionalTag` D-010 ;
- une interprétation libre de son texte.

La représentation en ensemble contrôlé évite les colonnes booléennes extensibles et permet d'ajouter une capacité officielle sans changer la structure conceptuelle.

## 6. Race des monstres

`monster_race` est une valeur contrôlée unique pour un `MONSTER`. Elle est distincte de l'attribut, des classifications de cadre et des capacités orthogonales. Elle appartient au `CardSnapshot` et peut donc évoluer dans un nouveau snapshot si la source ou une correction officielle change.

D-013 ne reproduit pas une liste approximative de races. Le registre normatif doit être constitué à partir de la base de données officielle de cartes KONAMI et du rulebook TCG officiel applicables au périmètre, avec pour chaque valeur : identifiant interne stable, libellé officiel anglais, source, date d'effet et éventuels alias de mapping. YGOPRODeck fournit les valeurs observées et opérationnelles, mais n'est pas à lui seul l'autorité normative de cette liste.

Trois ensembles sont distingués :

1. **registre normatif** : valeurs officiellement admises par la version du vocabulaire ;
2. **valeurs rencontrées** : sous-ensemble réellement présent dans un `CatalogueSnapshot` ;
3. **extension candidate** : nouvelle valeur externe mise en quarantaine jusqu'à vérification et ajout contrôlé au registre.

Pour un monstre publié, la race est requise. Une valeur manquante ou incohérente est une anomalie de qualité ; une valeur externe inconnue n'est pas convertie en race proche.

## 7. Attribut

Le vocabulaire MVP de `attribute` pour les monstres est :

- `DARK` ;
- `LIGHT` ;
- `EARTH` ;
- `WATER` ;
- `FIRE` ;
- `WIND` ;
- `DIVINE`.

L'attribut est une propriété intrinsèque, unique et versionnée d'un `MONSTER`. Il ne s'applique pas aux catégories `SPELL` et `TRAP` dans ce modèle. Pour ces catégories, son état est `NOT_APPLICABLE`, et non `UNKNOWN`.

Pour un monstre, une absence dans la source donne un état de donnée `UNKNOWN` en staging et empêche la publication tant qu'elle n'est pas résolue. Une valeur non reconnue ou contradictoire est une anomalie de qualité, jamais un tag fonctionnel ou un attribut choisi par approximation.

## 8. Propriétés Spell / Trap

`spell_trap_property` est une propriété contrôlée dont le domaine dépend de `card_category`.

Pour `SPELL` :

- `NORMAL` ;
- `CONTINUOUS` ;
- `QUICK_PLAY` ;
- `EQUIP` ;
- `FIELD` ;
- `RITUAL`.

Pour `TRAP` :

- `NORMAL` ;
- `CONTINUOUS` ;
- `COUNTER`.

Pour `MONSTER`, cette propriété est `NOT_APPLICABLE`. Une Spell ou Trap publiée possède exactement une propriété admise pour sa catégorie.

Les domaines Spell et Trap sont conceptuellement distincts. `SPELL.NORMAL` signifie « propriété Normal Spell » et `TRAP.NORMAL` signifie « propriété Normal Trap » ; leur libellé commun `NORMAL` ne crée ni identité ambiguë entre ces domaines, ni confusion avec la classification `MONSTER.NORMAL`. Cette qualification sémantique n'impose aucune représentation physique.

Ces propriétés décrivent l'iconographie/classification intrinsèque. Elles ne décrivent pas l'effet ni le rôle stratégique : `DRAW`, `SEARCHER`, `FLOODGATE`, `REMOVAL` et les autres termes D-010 n'appartiennent pas à ce vocabulaire.

## 9. Level, Rank et Link Rating

`level`, `rank` et `link_rating` sont trois propriétés numériques conceptuellement distinctes :

| Propriété | Applicabilité attendue |
|---|---|
| `level` | Monstres qui possèdent officiellement un Level ; notamment hors `XYZ` et `LINK` |
| `rank` | Monstres `XYZ` |
| `link_rating` | Monstres `LINK` |

Une propriété non applicable est `NOT_APPLICABLE`, pas zéro et pas `UNKNOWN`. Lorsqu'une propriété est applicable, une valeur numérique officielle est conservée telle quelle, y compris une valeur inhabituelle admise par le jeu.

D-013 ne fixe pas de bornes numériques par mémoire ou approximation. Les bornes et exceptions seront documentées à partir d'une source normative dans le dictionnaire de données. Une valeur applicable manquante reste `UNKNOWN` en staging ; une valeur mal formée ou incompatible avec les classifications est une anomalie de qualité.

Un champ générique unique `level` ne doit jamais représenter indistinctement Level, Rank et Link Rating.

## 10. Link markers

`link_markers` est un ensemble de directions contrôlées pour un monstre `LINK` :

- `TOP` ;
- `TOP_RIGHT` ;
- `RIGHT` ;
- `BOTTOM_RIGHT` ;
- `BOTTOM` ;
- `BOTTOM_LEFT` ;
- `LEFT` ;
- `TOP_LEFT`.

Un Link Monster peut posséder plusieurs marqueurs ; l'ordre de stockage n'a aucune signification. L'ensemble doit être cohérent avec le `link_rating` selon les règles de validation applicables.

Pour une carte non-Link, la propriété est `NOT_APPLICABLE`. Pour une carte Link dont les marqueurs n'ont pas été obtenus, l'ensemble est `UNKNOWN` en staging : un ensemble vide ne doit pas masquer cette absence. Une direction externe inconnue ou mal formée provoque une anomalie de mapping ou de qualité.

## 11. Pendulum

`PENDULUM` est une valeur de `monster_classifications`, combinable avec les autres classifications autorisées. Elle ne constitue ni une catégorie principale ni une propriété Spell/Trap.

Les deux échelles sont conservées séparément :

- `scale_left` ;
- `scale_right`.

Elles sont conceptuellement distinctes même lorsqu'elles ont la même valeur. Pour un monstre Pendulum, chaque échelle applicable contient une valeur numérique officielle ; une valeur manquante reste `UNKNOWN` en staging. Pour une carte non-Pendulum, les deux propriétés sont `NOT_APPLICABLE`. Elles ne sont jamais fusionnées dans un champ unique.

La classification `PENDULUM`, les échelles et le texte Pendulum sont trois concepts distincts. `pendulum_text` reste dans `CardLocalization` conformément à D-012 et n'appartient pas aux types non linguistiques de D-013.

## 12. ATK / DEF

ATK et DEF utilisent chacun une représentation conceptuelle capable de distinguer :

- **valeur numérique** : entier officiel, y compris zéro ;
- **`PRINTED_UNKNOWN`** : la valeur imprimée/officielle est explicitement `?` ;
- **`NOT_APPLICABLE`** : la propriété n'existe pas pour cette carte, par exemple la DEF d'un Link Monster ;
- **`UNKNOWN`** : la propriété devrait s'appliquer mais la donnée n'a pas été obtenue ou validée.

`PRINTED_UNKNOWN` est une valeur métier officielle. `UNKNOWN` décrit une insuffisance de données et ne doit pas être publié silencieusement comme `?`. `NOT_APPLICABLE` ne doit pas être représenté par zéro. Une chaîne ou valeur numérique mal formée est une anomalie de qualité, pas un cinquième état métier.

D-013 impose cette distinction sémantique sans choisir sa représentation physique.

## 13. Valeurs inconnues, absentes et incohérentes

Règle générale :

- **valeur connue** : valeur admise par le vocabulaire et soutenue par une provenance ;
- **`NOT_APPLICABLE`** : la propriété n'a pas de sens compte tenu de la catégorie et des classifications ;
- **`UNKNOWN`** : la propriété est applicable, mais sa valeur n'est pas encore établie ;
- **anomalie de qualité/intégrité** : donnée mal formée, contradictoire, interdite ou rupture de référence.

`INVALID` n'est pas ajouté comme valeur métier. Les anomalies sont enregistrées par la couche de qualité avec la valeur brute, la règle violée et la provenance. Elles ne deviennent pas des propriétés canoniques de la carte.

Pour un ensemble applicable et validé, l'ensemble vide signifie réellement « aucune valeur ». Pour un ensemble dont la collecte n'est pas fiable ou terminée, l'état est `UNKNOWN` ; ces deux situations ne sont pas interchangeables.

Un snapshot publié doit satisfaire ses règles de complétude. Les états `UNKNOWN` autorisés à la publication, s'il en existe, devront être explicitement décidés propriété par propriété dans le dictionnaire de données ; ils ne sont jamais assimilés à une valeur connue.

## 14. Sources et provenance

Le vocabulaire contrôlé définit les valeurs admissibles et leur sens. La provenance établit pourquoi une valeur précise a été attribuée à une carte dans un snapshot.

Chaque valeur réelle doit pouvoir être reliée à :

- la source et son enregistrement versionné ;
- l'instant de collecte ;
- la version du mapping externe → interne ;
- la version du vocabulaire contrôlé ;
- une éventuelle correction et revue humaines.

Une donnée de catalogue externe n'est pas automatiquement une vérité normative. Une correction humaine conserve le réviseur, la date, la justification, les sources examinées et la valeur remplacée. Une suggestion LLM reste non canonique tant qu'elle n'a pas fait l'objet d'une validation humaine traçable ; elle ne peut pas contourner la source normative ni la couche de qualité.

## 15. Relation avec les filtres et la recherche

Ces propriétés permettent conceptuellement des filtres exacts et composables :

- catégorie principale `MONSTER` / `SPELL` / `TRAP` ;
- présence d'une ou plusieurs classifications de monstre ;
- présence d'une capacité orthogonale ;
- race et attribut ;
- propriété Spell/Trap ;
- Level, Rank ou Link Rating avec opérateurs numériques ;
- directions de Link markers ;
- présence Pendulum et valeurs des échelles ;
- état ou valeur d'ATK/DEF.

Un filtre doit respecter l'applicabilité : rechercher une DEF numérique exclut les cartes où la DEF est `NOT_APPLICABLE`, et une valeur `UNKNOWN` ne satisfait jamais un filtre positif. Les libellés localisés servant à l'affichage ou à la saisie sont résolus vers ces identifiants contrôlés ; ils ne changent pas leur sens.

D-013 ne définit ni index, syntaxe de requête, classement, API ou interface.

## 16. Compatibilité avec les données externes

Le mapping YGOPRODeck → vocabulaire interne est explicite, versionné et distinct du vocabulaire lui-même.

| Situation externe | Traitement conceptuel |
|---|---|
| Valeur reconnue sans ambiguïté | Mapper vers la valeur interne et conserver la valeur brute/provenance |
| Valeur inconnue | Conserver la valeur brute, marquer `UNKNOWN` en staging et ouvrir une anomalie de mapping |
| Valeur ambiguë | Ne pas choisir arbitrairement ; demander une règle ou une revue documentée |
| Nouvelle valeur plausible | Mettre en quarantaine, vérifier une source normative puis versionner une extension du vocabulaire |
| Donnée mal formée | Rejeter le fait candidat et produire une anomalie de qualité |

Le libellé ou regroupement externe n'est jamais supposé identique au domaine interne. Une valeur inconnue reste non mappée et en quarantaine jusqu'à la création d'un mapping versionné ou une revue humaine validée. Si la propriété contrôlée est requise, la publication du fait concerné est bloquée jusqu'à résolution. Aucune conversion silencieuse vers `OTHER` n'est autorisée. Une table détaillée de mapping sera spécifiée avec les contrats de source, sans modifier les principes D-013.

## 17. Évolution du vocabulaire

- Chaque valeur contrôlée possède un identifiant conceptuel stable et une définition versionnée.
- Ajouter une valeur ne modifie pas rétroactivement le sens des valeurs existantes.
- Une nouvelle version du mapping ou du vocabulaire peut produire un nouveau `CatalogueSnapshot`.
- Les snapshots historiques conservent la version nécessaire à leur interprétation.
- Une ancienne valeur n'est pas renommée ou réaffectée silencieusement ; un libellé d'affichage peut évoluer sans changer son identifiant ni son sens.
- Retirer une valeur des nouveaux imports ne la supprime pas des anciens snapshots.
- La version du vocabulaire est distincte de `card_id`, de `CardSnapshot` et de la version commerciale d'une carte.

Une extension motivée par une nouvelle mécanique doit être validée comme propriété intrinsèque avant d'entrer dans le registre ; elle ne doit pas devenir un tag générique par commodité.

## 18. Hors périmètre

D-013 exclut explicitement :

- légalité, banlist et disponibilité EMEA ;
- tags et relations D-010 ;
- scoring, tier lists, popularité et prix ;
- collection utilisateur ;
- decks et formats YDK ;
- rulings et moteur de règles exhaustif ;
- simulation de duel, matchups et métagame ;
- prédictions compétitives ;
- API et UI ;
- SQL, ORM, migrations et index physiques ;
- importer, seed, fixtures physiques et dataset de production.

## 19. Décisions finales validées

1. Deux ensembles contrôlés distincts sont retenus : `monster_classifications` et `monster_abilities`. Chacun accepte plusieurs valeurs, sans catégorie combinatoire ni multiplication obligatoire de colonnes booléennes.
2. Le registre interne des races repose sur une source normative appropriée et reste indépendant du sous-ensemble observé chez YGOPRODeck. Une valeur externe inconnue n'intègre jamais automatiquement ce registre.
3. `UNKNOWN` désigne une propriété applicable dont la donnée n'est pas établie ; `NOT_APPLICABLE` une propriété qui ne s'applique pas ; une valeur concrète une donnée connue. Une donnée invalide ou incohérente reste une anomalie de qualité sans état métier `INVALID`.
4. Les six propriétés Spell et trois propriétés Trap proposées sont acceptées. Les valeurs homonymes, notamment `NORMAL`, appartiennent à des domaines qualifiés distincts.
5. Une valeur externe non reconnue reste non mappée et en quarantaine jusqu'à un mapping versionné ou une revue humaine validée. La publication d'un fait nécessitant cette valeur est bloquée et aucun fallback `OTHER` n'est permis.

## 20. Proposition minimale

```text
CardSnapshot
  card_category: MONSTER | SPELL | TRAP

  monster_classifications[]:
    NORMAL | EFFECT | RITUAL | FUSION | SYNCHRO | XYZ | LINK | PENDULUM

  monster_abilities[]:
    TUNER | FLIP | GEMINI | UNION | SPIRIT | TOON

  monster_race: registre officiel versionné
  attribute: DARK | LIGHT | EARTH | WATER | FIRE | WIND | DIVINE

  spell_trap_property:
    Spell: NORMAL | CONTINUOUS | QUICK_PLAY | EQUIP | FIELD | RITUAL
    Trap:  NORMAL | CONTINUOUS | COUNTER

  level | rank | link_rating: propriétés numériques distinctes
  link_markers[]: huit directions contrôlées
  scale_left | scale_right: valeurs distinctes
  atk | def: numérique | PRINTED_UNKNOWN | NOT_APPLICABLE | UNKNOWN
```

Les notions `UNKNOWN`, `NOT_APPLICABLE` et anomalie de qualité sont des distinctions conceptuelles transversales ; D-013 ne prescrit pas leur stockage physique.

## Dépendances

- **D-003 :** YGOPRODeck est la source opérationnelle, avec mapping explicite vers le vocabulaire interne.
- **D-007 :** les snapshots historiques et leurs versions de vocabulaire restent interprétables.
- **D-010 :** les rôles et relations stratégiques restent hors des types intrinsèques.
- **D-011 :** fixe les champs et frontières de `CardSnapshot`.
- **D-012 :** maintient les textes, noms et libellés linguistiques hors des propriétés intrinsèques.


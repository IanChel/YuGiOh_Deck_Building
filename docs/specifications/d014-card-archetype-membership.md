# D-014 — Spécification de l'appartenance des cartes aux archétypes

## Statut

**Proposed — validation humaine requise.**

Cette spécification définit conceptuellement `Archetype` et `CardArchetypeMembership`. Elle ne crée aucun code, classe, enum technique, SQL, ORM, migration, import, seed, dataset ou logique de recommandation.

## 1. Objectif

`CardArchetypeMembership` affirme qu'une carte appartient **structurellement** à un archétype identifié du catalogue.

Cette assertion est strictement distincte de :

1. soutenir stratégiquement un archétype ;
2. être en synergie avec ses cartes ;
3. être compatible avec une stratégie ou un deck ;
4. mentionner son nom ou un mot-clé dans un texte ;
5. être fréquemment jouée avec lui.

Seule l'appartenance structurelle entre dans `CardArchetypeMembership`. Les autres notions relèvent des relations et annotations D-010, du texte localisé ou de futurs travaux stratégiques.

## 2. Relation avec les décisions précédentes

D-011 a retenu une appartenance structurelle simplifiée. D-014 conserve ce choix :

- aucune distinction `MEMBER` / `EXPLICITLY_LISTED_SUPPORT` ;
- aucun sous-type équivalent réintroduit sous un autre nom ;
- une appartenance ne mesure ni intensité, ni qualité, ni pertinence stratégique ;
- `SUPPORTS`, `ENABLES`, `CONFLICTS_WITH` et les autres relations D-010 restent hors de cette relation.

Une carte peut donc être membre d'un archétype sans être pertinente dans une situation donnée. Elle peut aussi le soutenir, fonctionner dans son deck ou lui être recommandée sans en être membre.

D-012 gouverne les principes de localisation et d'alias linguistiques. D-013 gouverne les propriétés intrinsèques contrôlées des cartes ; race, attribut, classifications et capacités ne constituent jamais une appartenance d'archétype.

## 3. Définition normative d'un archétype

Un `Archetype` est une identité stable interne représentant un groupe de cartes reconnu structurellement comme tel par les données normatives ou les sources de catalogue validées pour le produit.

Proposition minimale :

```text
Archetype
  archetype_id
  created_at

ArchetypeLocalization
  archetype_id + catalogue_snapshot_id + language_code
  official_or_validated_name
  provenance_ref

ArchetypeNameAlias
  archetype_id + language_code + alias
  alias_kind
  valid_from_snapshot
  valid_to_snapshot?
  provenance_ref
```

- `archetype_id` est une identité interne stable, indépendante d'un nom ou d'un fournisseur.
- Un archétype peut être relié à zéro, une ou plusieurs cartes selon le snapshot observé.
- Ses noms et alias peuvent évoluer sans changer son identité.
- Une chaîne externe identique à son nom ne devient pas automatiquement son identité interne.

Un archétype n'est pas automatiquement :

- une série éditoriale ou commerciale ;
- un thème visuel ;
- une famille informelle de cartes ;
- une mécanique de jeu ;
- un type ou une classification de monstre ;
- une race ou un attribut ;
- le nom d'une carte individuelle ;
- une stratégie ou un deck.

D-014 ne cherche pas à résoudre toute la taxonomie Yu-Gi-Oh!. Les futures entités `Series`, `Theme` ou équivalentes restent différées conformément à D-011.

## 4. Critère d'appartenance structurelle

Une appartenance canonique doit être soutenue par au moins un fondement traçable accepté :

- une donnée officielle ou normative établissant l'appartenance ;
- une donnée structurée du catalogue opérationnel, mappée vers un archétype interne et acceptée par les règles de qualité ;
- une convention de nommage dont le caractère structurel est établi par une règle documentée et versionnée ;
- une correction humaine documentée fondée sur des sources examinées.

La convention de nommage n'est pas un test naïf de préfixe ou de sous-chaîne. Elle n'est utilisable que si sa portée, ses exceptions et sa source sont établies. Une ressemblance lexicale ne suffit pas.

Ne prouvent pas une appartenance :

- la simple mention de l'archétype dans le texte de carte ;
- la capacité à chercher, invoquer ou protéger ses cartes ;
- une synergie, une compatibilité ou une présence habituelle dans ses decks ;
- une recommandation humaine ou automatique ;
- un mot commun partagé par les noms ;
- un résultat LLM non revu.

Une carte générique reste sans appartenance si aucun fondement structurel ne la relie à un archétype, même si elle fonctionne dans plusieurs decks. Une carte de support explicitement conçue pour un archétype n'en devient pas membre par ce seul fait.

## 5. Source de vérité et provenance

D-014 réutilise les concepts de provenance D-010 :

- `SOURCE_CARD_TEXT` ;
- `DETERMINISTIC_DERIVATION` ;
- `HUMAN_ANNOTATION` ;
- `LLM_SUGGESTION` ;
- `UNKNOWN`.

Et les statuts de revue :

- `UNREVIEWED` ;
- `REVIEWED` ;
- `REJECTED` ;
- `SUPERSEDED`.

Une appartenance publiée conserve conceptuellement :

| Information | Rôle |
|---|---|
| `card_id` | Carte membre |
| `archetype_id` | Archétype interne ciblé |
| `catalogue_snapshot_id` | Version dans laquelle l'assertion est publiée |
| `provenance` | Origine selon le vocabulaire D-010 |
| `source_ref` | Record, règle ou annotation justificative |
| `review_status` | État de la revue |
| `review_ref` | Réviseur, date, justification et sources si revue humaine |

`SOURCE_CARD_TEXT` ne signifie pas que toute mention de texte prouve l'appartenance : il indique seulement l'origine d'une assertion dont la règle structurelle doit être justifiée. Une dérivation déterministe conserve sa règle et sa version.

Une `LLM_SUGGESTION / UNREVIEWED` reste une proposition séparée et non canonique. Elle ne crée jamais une `CardArchetypeMembership` publiée. Sa promotion exige une revue humaine documentée et devient `HUMAN_ANNOTATION / REVIEWED`, avec lien vers la suggestion initiale. Une correction humaine conserve l'auteur, la date, la justification, les sources examinées et l'assertion remplacée.

`UNKNOWN` décrit une provenance inconnue ou héritée à assainir ; il ne constitue pas une preuve suffisante pour publier une nouvelle appartenance canonique.

## 6. Appartenance multiple

La cardinalité est plusieurs-à-plusieurs :

- une carte peut appartenir à zéro, un ou plusieurs archétypes ;
- un archétype peut comporter zéro, une ou plusieurs cartes dans un snapshot ;
- chaque paire carte–archétype constitue une assertion indépendante ;
- chaque assertion possède sa propre provenance, revue et période de validité par snapshot.

Une appartenance à un archétype ne peut pas être déduite de l'appartenance à un autre, sauf règle normative explicite et versionnée. Les familles imbriquées ou partageant des noms n'entraînent aucune propagation automatique.

Une carte appartenant structurellement à plusieurs familles reçoit plusieurs `CardArchetypeMembership`. Aucun champ « archétype principal » n'est ajouté au MVP.

## 7. Archétypes et noms/localisation

L'identité `Archetype` est séparée de ses noms. Le système utilise un mécanisme analogue à D-012, mais ne réutilise pas `CardLocalization` :

- `ArchetypeLocalization` porte le nom officiel ou validé d'un archétype dans une langue et un `CatalogueSnapshot` ;
- l'anglais est la forme canonique du MVP lorsqu'elle est disponible ;
- le français est facultatif et peut utiliser un fallback explicite vers l'anglais ;
- `ArchetypeNameAlias` porte les anciennes dénominations, translittérations validées et variantes de source ;
- aucun alias ou nom localisé ne remplace `archetype_id`.

Une recherche française et une recherche anglaise peuvent donc résoudre vers le même `archetype_id`. Les surnoms communautaires et traductions libres non validées ne deviennent pas automatiquement des alias.

Les décisions détaillées de D-012 sur la distinction entre donnée officielle, validation humaine, fallback non persistant et provenance s'appliquent par analogie. D-014 ne crée pas une architecture linguistique supplémentaire au-delà de ces concepts minimaux.

## 8. Archétypes et snapshots

Relations conceptuelles :

```text
Card
  └── CardSnapshot(card_id, catalogue_snapshot_id)

Archetype
  └── ArchetypeLocalization(archetype_id, catalogue_snapshot_id, language)

CardArchetypeMembership
  card_id
  archetype_id
  catalogue_snapshot_id
  provenance et revue
```

Une appartenance publiée affirme que la `Card` et l'`Archetype` sont reliés dans un `CatalogueSnapshot` précis. Elle doit référencer une carte représentée dans ce snapshot et un archétype connu du référentiel interne applicable à ce snapshot.

Une correction, un ajout ou un retrait d'appartenance apparaît dans un nouveau snapshot. Les anciens enregistrements ne sont pas modifiés et les résultats historiques continuent d'utiliser les appartenances de leur snapshot d'origine.

La présence d'une relation dans le Snapshot A et son absence dans le Snapshot B signifie que la relation n'est pas publiée dans B. La raison de cette transition reste traçable dans les données de provenance ou le rapport de qualité. D-014 ne choisit ni copie complète, ni intervalle physique, ni stratégie de déduplication.

## 9. YGOPRODeck et mapping externe

Une valeur d'archétype YGOPRODeck est une donnée externe, pas un `archetype_id`. Le mapping externe → interne est explicite, versionné et traçable.

| Situation | Traitement conceptuel |
|---|---|
| Archétype externe connu et mappable | Résoudre vers l'`archetype_id`, conserver valeur brute, source et version du mapping |
| Connu mais non encore mappé | Mettre en quarantaine ; ne pas publier d'appartenance |
| Valeur ambiguë | Suspendre le mapping et demander une revue ; ne pas choisir par ressemblance |
| Archétype interne absent | Créer un candidat de référentiel à examiner, jamais une identité canonique automatique |
| Valeur vide ou absente | Ne créer aucune assertion ; signaler l'état de couverture si une valeur était attendue |
| Nomenclature externe modifiée | Versionner le mapping ou l'alias sans remplacer silencieusement l'identité interne |

La comparaison de chaînes, même exacte, peut proposer un candidat de mapping mais ne suffit pas à créer ou fusionner une identité. Les identifiants externes et anciennes dénominations restent dans des références/alias traçables.

## 10. Archétype inexistant ou inconnu

`CardArchetypeMembership` contient uniquement des assertions positives vers un `Archetype` interne connu. Il ne contient pas de cible factice `UNKNOWN`.

- **Archétype connu :** l'appartenance peut être publiée si son fondement et sa provenance satisfont les règles.
- **Archétype externe non référencé :** la valeur brute et la source sont placées en quarantaine comme candidat de mapping/référentiel.
- **Appartenance inconnue :** aucune assertion canonique n'est créée ; l'incertitude appartient au rapport de couverture ou de qualité du snapshot.
- **Donnée contradictoire :** les faits candidats et sources sont conservés, la publication est bloquée pour cette assertion et une revue est requise.

Une valeur inconnue ne doit pas être ignorée silencieusement si la source prétend fournir une appartenance. Elle est enregistrée dans le flux de qualité, mais ne crée ni faux archétype ni faux membre. Un archétype interne n'est créé qu'après validation de son identité et de son caractère structurel.

## 11. Distinction avec D-010

| Cas conceptuel | Représentation |
|---|---|
| A — Une carte appartient structurellement à `X` | Une `CardArchetypeMembership` vers `X` |
| B — Une carte générique recherche ou invoque des cartes de `X` sans en être membre | Relation/annotation D-010 telle que `SEARCHES`, `SPECIAL_SUMMONS`, `SUPPORTS` ou `ENABLES`, selon le fait exact |
| C — Une carte mentionne `X` dans son texte | Fait textuel localisé et éventuellement relation dérivée ; aucune appartenance automatique |
| D — Une carte fonctionne très bien dans un deck `X` | Compatibilité ou synergie stratégique ; aucune appartenance |
| E — Une carte appartient structurellement à `X` et `Y` | Deux `CardArchetypeMembership` indépendantes |

Une carte peut cumuler une appartenance et des relations D-010 : les assertions répondent à des questions différentes et ne se remplacent pas.

## 12. Cas limites

| Cas | Protection attendue |
|---|---|
| Plusieurs archétypes | Créer plusieurs assertions indépendantes et sourcées |
| Archétypes partageant une partie du nom | Résoudre par identité/mapping, jamais par sous-chaîne seule |
| Faux positif fondé sur le nom | Exiger une règle structurelle documentée |
| Texte citant plusieurs archétypes | Ne créer aucune appartenance par la seule citation |
| Carte générique | Autoriser zéro appartenance |
| Support conçu pour un archétype | Utiliser D-010 sauf preuve structurelle indépendante |
| Variantes linguistiques | Résoudre via `ArchetypeLocalization` vers le même `archetype_id` |
| Translittération | Alias uniquement si validé et sourcé |
| Ancienne dénomination | Alias versionné, identité stable inchangée |
| Changement de nomenclature | Nouveau nom/alias et mapping dans un nouveau snapshot |
| Sources contradictoires | Bloquer l'assertion concernée et déclencher une revue traçable |
| Archétype absent du référentiel | Mettre la valeur externe en quarantaine |
| Suggestion LLM non vérifiée | Conserver comme suggestion non canonique, sans membership publiée |

Ces règles protègent l'intégrité sans prétendre résoudre automatiquement tous les cas éditoriaux ou historiques du jeu.

## 13. Recherche et filtres

Le système doit pouvoir distinguer explicitement :

- rechercher un archétype par nom anglais, français ou alias validé ;
- retourner les cartes qui **appartiennent** à cet `archetype_id` dans un snapshot ;
- retourner séparément les cartes qui le soutiennent ou interagissent avec lui via D-010 ;
- filtrer par une ou plusieurs appartenances structurelles ;
- signaler les valeurs externes non mappées sans les exposer comme archétypes canoniques.

Une recherche textuelle du nom d'un archétype dans `card_text` ne transforme pas ses résultats en membres. Les filtres d'appartenance consultent exclusivement les assertions publiées du snapshot demandé.

D-014 ne définit ni index, syntaxe, classement, API ou interface.

## 14. Impact sur l'IA

Le futur moteur de recommandation peut utiliser séparément :

- l'appartenance structurelle D-014 ;
- les relations et rôles D-010 ;
- le texte de carte localisé D-012 ;
- les propriétés intrinsèques contrôlées D-013.

L'appartenance n'est ni un score de pertinence ni une recommandation. Une carte non membre peut être un excellent candidat pour un deck ; une carte membre peut être inadaptée au besoin ou à la construction observée.

Le moteur ne doit donc ni exclure par défaut les non-membres, ni favoriser automatiquement tous les membres. D-014 ne définit aucun score, classement ou algorithme.

## 15. Hors périmètre

D-014 exclut explicitement :

- scoring et classement des archétypes ;
- puissance, méta, popularité, tier list et matchups ;
- deckbuilding algorithmique et recommandation ;
- détail des relations stratégiques D-010 ;
- simulation de duel ;
- légalité, banlist et disponibilité EMEA ;
- prix et collection ;
- formats YDK ;
- SQL, ORM, migrations et index ;
- API et UI ;
- importer de production, seed et dataset physique.

## 16. Questions de validation finales

1. L'appartenance structurelle peut-elle être publiée lorsqu'elle est soutenue par une donnée officielle, un mapping de catalogue validé, une règle de nommage normative versionnée ou une correction humaine revue, toute simple mention/synergie restant insuffisante ?
2. Le modèle minimal `Archetype` avec identité interne stable, `ArchetypeLocalization` et `ArchetypeNameAlias` séparés est-il accepté, sans créer d'entités `Series` ou `Theme` au MVP ?
3. L'appartenance plusieurs-à-plusieurs est-elle acceptée sans notion d'« archétype principal », chaque assertion possédant sa propre provenance et son propre historique ?
4. Les appartenances importées peuvent-elles réutiliser les provenances et statuts de revue D-010, avec publication réservée aux assertions suffisamment sourcées/revues et interdiction de promouvoir automatiquement une suggestion LLM ?
5. Une valeur d'archétype externe inconnue, ambiguë ou non mappée doit-elle rester en quarantaine hors de `CardArchetypeMembership`, bloquer l'assertion concernée et ne jamais créer automatiquement un `Archetype` ou une cible `UNKNOWN` ?

## 17. Proposition minimale

```text
Archetype
  archetype_id
  created_at

ArchetypeLocalization
  archetype_id + catalogue_snapshot_id + language_code
  official_or_validated_name
  provenance_ref

ArchetypeNameAlias
  archetype_id + language_code + alias
  alias_kind
  valid_from_snapshot
  valid_to_snapshot?
  provenance_ref

CardArchetypeMembership
  card_id + archetype_id + catalogue_snapshot_id
  provenance
  source_ref
  review_status
  review_ref?
```

Règles minimales :

- une assertion positive seulement, vers des identités internes connues ;
- zéro, une ou plusieurs appartenances par carte ;
- aucune distinction `MEMBER` / `EXPLICITLY_LISTED_SUPPORT` ;
- aucune inférence depuis une simple mention, synergie ou recommandation ;
- mapping externe versionné ; inconnus et contradictions en quarantaine ;
- ancien historique conservé par `CatalogueSnapshot` ;
- noms et alias séparés de l'identité ;
- suggestions LLM non revues exclues des assertions publiées.

## Dépendances

- **D-003 :** YGOPRODeck fournit la donnée opérationnelle à mapper vers les identités internes.
- **D-007 :** les appartenances historiques restent reproductibles par snapshot.
- **D-010 :** porte les rôles et relations stratégiques distincts de l'appartenance.
- **D-011 :** impose l'appartenance structurelle simplifiée et reporte `Series`.
- **D-012 :** fournit les principes de localisation, alias, fallback et provenance linguistique.
- **D-013 :** maintient races, attributs et classifications intrinsèques hors des archétypes.


# D-009 — Dataset pilote validé pour tester le moteur

## Statut

**Accepted — validé explicitement le 29 septembre 2026.**

Ce document spécifie le corpus de fixtures validé par D-009. Il ne crée encore aucun dataset physique et ne sélectionne aucun deck à jouer.

## Frontière contractuelle

- **Catalogue de production :** toutes les cartes couvertes par TCG Advanced EMEA que la source validée et les droits applicables permettent d'exposer.
- **Dataset pilote :** environ dix familles stratégiques utilisées uniquement pour fixtures, annotations initiales, golden decks et tests.
- **Construction utilisateur :** libre, depuis un deck vide ou importé, sans sélection obligatoire d'un archétype.
- **Recommandation :** candidats récupérés dans l'ensemble du catalogue légal ; le dataset pilote ne constitue jamais une whitelist.

D-009 n'a aucun effet sur le catalogue, la recherche, les filtres, la construction manuelle, l'import/export, le Rules Engine, le pool de candidats ou les cartes accessibles à l'utilisateur final.

## Référentiel de format

- Format : **TCG Advanced EMEA** conformément à ADR-0001.
- Chaque fixture doit épingler `FormatSnapshot`, `BanlistSnapshot` et `CatalogueSnapshot`.
- Une famille reste utile comme cas de test même si certaines de ses cartes changent de statut : les quantités attendues sont recalculées pour le snapshot, jamais codées comme vérités intemporelles.
- La légalité régionale dépend de la sortie officielle EMEA et de la banlist officielle ; le nom d'un archétype ne suffit jamais à déclarer une carte légale.

## Critères de sélection

La proposition maximise la couverture fonctionnelle plutôt que la puissance ou la popularité :

1. **Complexité :** cas simples, intermédiaires et complexes.
2. **Gameplay :** combo, midrange, contrôle, tempo, grind et pression offensive.
3. **Extra Deck :** faible dépendance, forte dépendance et couverture Fusion/Synchro/Xyz/Link/Pendule/Rituel.
4. **Main Deck :** moteurs compacts ou volumineux, nombreux slots génériques, contraintes fortes et ratios sensibles.
5. **Relations :** starters, extenders, searchers, boss monsters, coûts, prérequis, verrous, noms mentionnés et synergies génériques.
6. **Recommandations :** moteur, consistance, interaction, défense, board breakers, cartes génériques et Extra Deck.
7. **Modélisation :** familles croisées, cartes appartenant ou faisant référence à plusieurs ensembles, et effets dépendant de l'état.
8. **Testabilité :** textes et cartes identifiables dans les sources retenues, scénarios bornables et résultats pouvant être revus.
9. **Temporalité :** au moins un cas sensible aux changements de banlist.
10. **Non-dépendance à l'archétype :** compléments obligatoires avec decks sans archétype et cartes hors corpus.

## Sélection validée — sans classement

| Famille de fixture | Utilité pour le logiciel | Fonctionnalités testées | Mécaniques et contraintes | Dépendances à surveiller |
|---|---|---|---|---|
| **Blue-Eyes** | Cas d'entrée relativement lisible avec vaste support historique et références textuelles à une carte précise | Recherche par nom/texte, Normal Monster, searchers, ratios de cartes clés, suggestions de support et d'Extra Deck | Construction orientée Dragon/LIGHT ; Fusion, Synchro et options Rituel ; cartes qui mentionnent explicitement « Blue-Eyes White Dragon » | Grand volume de support et variantes : la fixture doit rester bornée et ne pas confondre mention textuelle, série et appartenance d'archétype |
| **Branded / Despia / Fallen of Albaz** | Stress test d'un moteur Fusion volumineux et de plusieurs familles interconnectées | Matériaux, lignes de recherche, garnets/requirements, ratios moteur, Extra Deck, recommandations de cartes liées | Fusion centrale, envoi depuis différentes zones, verrous d'Extra Deck et dépendances entre sous-familles | Restrictions évolutives de banlist, notamment sur des cartes « Branded » ; snapshot obligatoire et cartes interdites exclues |
| **Swordsoul / Tenyi** | Exemple intermédiaire de moteur hybride où deux familles et des types de monstres coopèrent | Starter nécessitant une révélation, tokens, Tuners implicites, extenders, niveaux Synchro, compatibilité Wyrm | Synchro fortement utilisé ; relations entre cartes Swordsoul, Tenyi et génériques Wyrm | Tester les contraintes de niveau, de token et les recommandations multi-engine sans fusionner arbitrairement les archétypes |
| **Purrely** | Cas Xyz atypique où les Magies deviennent des matériaux et où les quantités de ressources comptent | Attachement de matériaux, Rank, search/reveal, ratios de « Memory », grind et dépendance à l'état | Xyz important, cartes devenant meilleures selon les matériaux attachés et effets utilisés | Certaines « Memory » peuvent être limitées ou semi-limitées selon le snapshot ; ne pas figer leurs quantités |
| **Salamangreat** | Cas Link centré sur récursion et réutilisation du même nom | Link Ratings, flèches/zones, cimetière, recyclage, starters/extenders et Extra Deck toolbox | Link Summon importante et invocation utilisant un monstre du même nom ; moteur Cyberse compatible avec des génériques | Distinguer support Salamangreat, génériques Cyberse et cartes simplement compatibles |
| **D/D / D/D/D / Dark Contract** | Cas volontairement complexe couvrant plusieurs types de cartes et relations nominales | Multi-archetype, coûts récurrents, séquences complexes, searchers, conditions, scoring de lignes et détection de conflits | Pendule plus Fusion, Synchro, Xyz et Link ; Échelles et « Dark Contract » | Ne pas utiliser cette famille comme fixture d'onboarding ; prévoir des états intermédiaires et résultats attendus très précis |
| **Drytron** | Cas Rituel non conventionnel utile pour éviter des hypothèses génériques erronées | Ritual Monster/Spell, searchers, tribut fondé sur l'ATK, contraintes de Special Summon, moteurs compatibles | Rituel central, monstres de Niveau 1 et Xyz de soutien ; conditions propres à la famille | Interactions avec supports Rituel génériques ; le moteur doit lire les contraintes structurées et non inférer seulement par Niveau |
| **Labrynth** | Contrôle/grind à faible dépendance obligatoire à l'Extra Deck | Normal Traps, set/activation, récursion, discard/coûts, ratios de pièges, slots génériques défensifs | Jeu principalement Main Deck, réponses à l'adversaire et boucles de ressources | Bien séparer cartes Labrynth, Normal Traps génériques et pièges légalement disponibles ; Extra Deck souvent utilitaire plutôt qu'identitaire |
| **Sky Striker** | Moteur compact orienté Magies avec fortes contraintes d'état et choix de toolbox | Nombre de Magies au cimetière, zone Monstre Main libre, rotation de Links, grind, slots de handtraps/board breakers | Link-1, Magies à conditions contextuelles et moteur pouvant cohabiter avec des cartes génériques | Exige une représentation correcte des zones et de l'état ; ne pas recommander un monstre générique qui invalide silencieusement une condition |
| **Floowandereeze** | Cas sans dépendance normale à l'Extra Deck et avec verrou de Special Summon | Chaînes de Normal Summons, bannissement face recto, récupération, tributs, cohérence sous contraintes | Main Deck fortement structuré ; faible usage de l'Extra Deck ; incompatibilités importantes avec de nombreux extenders génériques | Excellent test négatif : une carte individuellement forte peut être incompatible avec la stratégie ou ses verrous |

## Matrice de couverture

| Fonctionnalité / cas de test | Fixtures couvrant le cas |
|---|---|
| Complexité relativement simple | Blue-Eyes, Floowandereeze |
| Complexité intermédiaire | Swordsoul/Tenyi, Purrely, Labrynth, Salamangreat, Sky Striker |
| Complexité élevée | D/D/D, Branded/Despia, Drytron |
| Combo | D/D/D, Drytron, Salamangreat |
| Midrange / tempo | Swordsoul/Tenyi, Branded/Despia, Salamangreat |
| Control | Labrynth, Sky Striker, Floowandereeze |
| Grind / récursion | Purrely, Labrynth, Sky Striker, Branded/Despia, Salamangreat |
| Pression offensive / OTK à tester | Blue-Eyes, D/D/D, Drytron |
| Fusion | Branded/Despia, Blue-Eyes, D/D/D |
| Synchro | Swordsoul/Tenyi, Blue-Eyes, D/D/D |
| Xyz | Purrely, D/D/D, Drytron |
| Link | Salamangreat, Sky Striker, D/D/D |
| Pendule | D/D/D |
| Rituel | Drytron, Blue-Eyes |
| Extra Deck fortement structurant | Branded/Despia, Swordsoul/Tenyi, Purrely, Salamangreat, D/D/D, Sky Striker |
| Extra Deck faible ou utilitaire | Labrynth, Floowandereeze |
| Moteur compact / nombreux slots génériques | Sky Striker, Salamangreat selon configuration |
| Moteur volumineux | Branded/Despia, D/D/D, Floowandereeze |
| Ratios et requirements sensibles | Blue-Eyes, Branded/Despia, Purrely, Drytron |
| Forte contrainte de construction | Floowandereeze, Drytron, D/D/D, Sky Striker |
| Starters / extenders / searchers | Toutes, avec formes et conditions différentes |
| Boss monsters / conditions d'accès | Toutes |
| Cartes génériques compatibles | Swordsoul/Tenyi, Salamangreat, Labrynth, Sky Striker, Blue-Eyes |
| Recommandations négatives pour incompatibilité | Floowandereeze, Sky Striker, Drytron, D/D/D |
| Handtraps et board breakers | Scénarios transversaux sur Sky Striker, Salamangreat, Swordsoul/Tenyi et Labrynth |
| Support direct d'archétype | Toutes |
| Familles croisées / multi-engine | Branded/Despia, Swordsoul/Tenyi, D/D/D/Dark Contract |
| État de zones | Sky Striker, Salamangreat |
| Banlist versionnée | Branded/Despia, Purrely, plus cas sentinelles indépendants |
| Deck sans archétype central | **Non couvert par un archétype : scénario indépendant obligatoire** |
| Carte hors dataset pilote | **Scénarios indépendants obligatoires** |

## Revue finale de cohérence

### Corrections proposées à la sélection

Aucun remplacement n'est nécessaire. Les dix entrées sont à considérer comme **dix familles de fixtures**, pas toujours comme dix archétypes strictement isolés : Branded/Despia/Fallen of Albaz, Swordsoul/Tenyi et D/D/D/Dark Contract testent précisément les relations entre plusieurs ensembles nominaux.

Deux précisions doivent être appliquées lors de la future création des fixtures :

- **Blue-Eyes** n'est « relativement simple » que dans un scénario d'entrée borné. La totalité de son support historique ne doit pas être chargée dans une unique golden fixture.
- **D/D/D** est une fixture de stress multi-mécaniques. Elle ne doit pas être utilisée pour définir le comportement nominal minimal de chaque mécanique ; des micro-fixtures isolées restent nécessaires.

### Redondances examinées

| Recouvrement apparent | Pourquoi les fixtures restent distinctes |
|---|---|
| Blue-Eyes / Drytron sur le Rituel | Blue-Eyes teste nom exact, Normal Monster et support historique ; Drytron teste un système Rituel atypique et des restrictions propres |
| Branded / D/D/D sur Fusion et complexité | Branded teste matériaux, famille narrative croisée et verrou Fusion ; D/D/D teste Pendule et coexistence de plusieurs invocations |
| Labrynth / Sky Striker sur le contrôle | Labrynth est centré Normal Traps et faible Extra Deck ; Sky Striker dépend des Magies, des zones et d'une rotation Link |
| Salamangreat / Sky Striker sur Link et grind | Salamangreat teste Cyberse, récursion et même nom ; Sky Striker teste Link-1, état de zone et moteur compact de Magies |
| Floowandereeze / Labrynth sur faible Extra Deck | Floowandereeze teste Normal Summons, bannissement et verrous de Special Summon ; Labrynth teste pièges et réactions à l'adversaire |
| Swordsoul/Tenyi / Salamangreat sur starters-extenders | Le premier teste tokens, niveaux Synchro et type Wyrm ; le second teste Link, cimetière et contraintes Cyberse |

Il existe donc des recouvrements utiles pour vérifier que le moteur ne confond pas un même rôle fonctionnel avec une seule mécanique. Aucun doublon ne justifie un retrait à ce stade.

### Légalité et disponibilité à surveiller

Les dix familles disposent de cartes publiées dans le TCG et consultables dans la base officielle. Cela ne valide pas automatiquement une decklist : la légalité dépend de la date de sortie EMEA, du snapshot de banlist et des cartes génériques ajoutées.

| Fixture | Risque de légalité/disponibilité à contrôler lors de sa matérialisation |
|---|---|
| Blue-Eyes | Support réparti sur de nombreuses éditions ; vérifier la date EMEA de toute carte récente et ne pas reprendre une liste OCG/Master Duel |
| Branded/Despia | Plusieurs cartes associées ont changé de statut ; contrôler notamment les cartes Fusion/Trap centrales et exclure toute carte interdite du snapshot |
| Swordsoul/Tenyi | Vérifier séparément les boss ou floodgates génériques historiquement associés ; ils ne font pas partie de la légalité implicite de la famille |
| Purrely | Certaines Magies « Memory » ont connu des limitations ; leurs ratios doivent provenir du `BanlistSnapshot` |
| Salamangreat | Les cartes de la famille sont disponibles, mais les extenders/Links Cyberse génériques doivent être contrôlés individuellement |
| D/D/D | Aucun droit à une carte générique multi-invocation ne doit être inféré ; chaque composant et chaque quantité sont validés séparément |
| Drytron | Les packages Rituel/Fairy génériques associés peuvent avoir leurs propres restrictions ; ne pas les assimiler à Drytron |
| Labrynth | Les Normal Traps génériques proposées peuvent être limitées ou interdites ; le moteur doit distinguer support de famille et cible compatible |
| Sky Striker | Les ratios des Magies et Links doivent être dérivés du snapshot ; une ancienne decklist ne constitue pas une source de légalité |
| Floowandereeze | Les cartes génériques de bannissement ou de contrôle historiquement associées peuvent être interdites ; la fixture de base doit rester indépendante de ces cartes |

**Gate obligatoire :** aucune golden decklist ne peut être publiée comme fixture valide sans rapport automatique `card exists + EMEA released + zone eligible + copy limit + banlist snapshot`. Si la source EMEA et une autre page régionale divergent, le référentiel EMEA d'ADR-0001 prévaut.

### Couverture des recommandations

| Nature de recommandation | Cas permettant de l'évaluer |
|---|---|
| Support direct | Les dix familles |
| Starter | Swordsoul, Salamangreat, Branded, Purrely, Floowandereeze |
| Extender | Salamangreat, Swordsoul/Tenyi, D/D/D, Drytron |
| Searcher | Blue-Eyes, Branded, Drytron, D/D/D, Labrynth |
| Handtrap | Scénarios transversaux sur moteurs laissant des slots génériques ; jamais présumée compatible |
| Board breaker | Scénarios going-second transversaux, avec contrôle des verrous et ratios |
| Interaction | Labrynth, Swordsoul, Purrely, Sky Striker, Salamangreat |
| Défense | Labrynth, Sky Striker, Blue-Eyes et scénarios génériques |
| Carte générique | Sky Striker, Salamangreat, Swordsoul/Tenyi, Labrynth, plus deck sans archétype |
| Correction d'une faiblesse | Manque de starter, ratio incohérent, absence de réponse, Extra Deck incomplet ou conflit détecté selon la fixture |

### Couverture des incompatibilités

| Incompatibilité à rejeter | Fixtures principales |
|---|---|
| Type d'invocation | Branded, D/D/D, Drytron, Floowandereeze |
| Restriction d'archétype/nom | Salamangreat, Branded/Despia, D/D/D, Blue-Eyes |
| Type ou Attribut | Swordsoul/Tenyi, Blue-Eyes, Drytron, Salamangreat |
| Contrainte d'Extra Deck | Branded, Salamangreat, Sky Striker, D/D/D |
| Restriction de Special Summon | Floowandereeze, Drytron |
| Incompatibilité de gameplay/zone | Sky Striker, Floowandereeze, Purrely, Labrynth |
| Banlist et limites de copies | Branded, Purrely et toutes les micro-fixtures sentinelles |
| État ou condition structurée | Purrely (matériaux), Sky Striker (zone/cimetière), Swordsoul (révélation/token), Labrynth (Normal Trap) |

### Conclusion de la revue

La sélection présente une diversité suffisante, sans redondance éliminatoire ni lacune critique imposant un onzième archétype. Les trous restants sont mieux couverts par les micro-fixtures indépendantes listées ci-dessous. D-009 peut être soumise à validation humaine, mais demeure `Proposed` jusqu'à approbation explicite.

## Trous de couverture acceptés

Environ dix familles ne couvrent pas raisonnablement toutes les sous-mécaniques du jeu. Les lacunes suivantes ne justifient pas l'ajout automatique de nouvelles familles au dataset initial :

- stratégies centrées Equip, Union, Gemini, Spirit, Flip ou Toon ;
- conditions de victoire alternatives, burn, deck-out et FTK ;
- manipulation approfondie des colonnes, zones Pendule et Extra Link ;
- stratégies centrées sur les compteurs, pièces assemblées ou changements de nom/type/attribut ;
- interactions exhaustives de chaîne, Damage Step et rulings complexes ;
- Side Deck et adaptation statistique aux matchups, hors MVP.

Ces cas seront ajoutés sous forme de micro-fixtures synthétiques ou de tests ciblés seulement lorsqu'une règle ou un défaut le justifie.

## Cas de test indépendants des archétypes

1. **Deck sans archétype central :** pile de cartes génériques légales avec rôles connus.
2. **Carte hors dataset pilote :** recherche, ajout, export YDK et validation réussis sans annotation stratégique.
3. **Moteurs mixtes :** deux petits moteurs compatibles, puis incompatibles, sans archétype racine déclaré.
4. **Aucune relation connue :** le système répond « information indisponible » sans inventer de synergie.
5. **Handtraps/board breakers :** candidats génériques évalués selon slots, légalité et conflits, pas selon appartenance.
6. **Frontières de deck :** Main 39/40/60/61, Extra 0/15/16 et Side 0/15/16.
7. **Copies cumulées :** même carte répartie entre Main/Extra/Side selon les zones possibles et la banlist.
8. **Temporalité :** même deck validé contre deux snapshots de banlist donnant des résultats différents et reproductibles.
9. **Légalité EMEA :** carte connue du catalogue mais non légale dans la région à la date du snapshot.
10. **Localisation :** nom français et anglais résolus vers le même identifiant ; texte absent avec fallback explicite.
11. **Import YDK :** carte inconnue, doublon, quantité invalide et identifiant légal hors dataset pilote.
12. **Sécurité LLM :** demande citant une carte inexistante ou demandant d'ignorer la banlist.

## Conditions de mise en œuvre de D-009

- [x] Les dix familles sont approuvées comme fixtures, sans notion de classement.
- [ ] Un propriétaire de revue métier est désigné pour chaque golden deck ou scénario.
- [ ] Les versions de catalogue et de banlist de référence sont choisies.
- [ ] Le volume maximal initial de cartes et relations annotées est fixé.
- [x] Les scénarios indépendants obligatoires sont approuvés.
- [ ] Les critères de résultat attendu et de maintenance d'une fixture sont définis.
- [x] Il est confirmé par écrit que le dataset n'est jamais utilisé comme filtre du catalogue ou whitelist de recommandation.

## Aptitude à démarrer la Phase 1

La sélection validée est suffisamment diverse pour commencer raisonnablement la **spécification du schéma, des mappings, des fixtures et des contrôles de qualité** de Phase 1. Elle n'autorise pas encore la création du dataset complet ni l'implémentation de l'importateur.

## Sources de cadrage

- Banlist TCG officielle EMEA : <https://www.yugioh-card.com/eu/play/forbidden-and-limited-list/>
- Légalité des cartes EMEA : <https://www.yugioh-card.com/eu/play/card-legality/>
- Rulebook TCG : <https://www.yugioh-card.com/eu/play/tcg-rulebook/>
- Base officielle de cartes KONAMI/NEURON : <https://www.db.yugioh-card.com/yugiohdb/card_search.action>
- Catalogue opérationnel retenu : <https://ygoprodeck.com/api-guide/>

La composition exacte de chaque fixture devra être vérifiée contre les snapshots retenus au moment de sa création. Les usages stratégiques communautaires peuvent aider à construire les scénarios, mais ne remplacent ni les textes de cartes, ni les règles, ni la banlist officielle.

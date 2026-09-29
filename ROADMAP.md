# Roadmap — Yu-Gi-Oh! Intelligent Deck Builder

> Document vivant et tableau de bord principal du projet. Dernière mise à jour : 29 septembre 2026.
>
> Une case n'est cochée que lorsque le résultat correspondant existe, a été vérifié et satisfait ses critères d'acceptation.

# 📊 Progression du projet

- [ ] Phase 0 — Conception
- [ ] Phase 1 — Données
- [ ] Phase 2 — Backend
- [ ] Phase 3 — Deck Engine
- [ ] Phase 4 — IA
- [ ] Phase 5 — Frontend
- [ ] Phase 6 — Intégration
- [ ] Phase 7 — Tests
- [ ] Phase 8 — MVP
- [ ] Phase 9 — Déploiement
- [ ] Phase 10 — Fonctionnalités avancées

## Current Status

| Élément | État |
|---|---|
| Phase actuelle | **Phase 0 — Conception** |
| Tâche en cours | Revue et validation par le porteur du projet de la spécification candidate de Phase 0 |
| Prochaine tâche | Approuver ou amender les décisions bloquantes D-001 à D-008 |
| Blocages | Format TCG Advanced EMEA, source YGOPRODeck + Konami, langue canonique, cinq archétypes pilotes, stack et budget LLM proposés mais non approuvés |
| Décisions récentes | Spécification candidate produite ; recommandation d'une architecture multi-format avec uniquement TCG Advanced EMEA implémenté au MVP ; aucune de ces recommandations n'est encore considérée comme validée |

### Règles de mise à jour

- Cocher une tâche uniquement après réalisation et vérification de ses critères d'acceptation.
- Lorsqu'une tâche est terminée, mettre à jour cette section, la progression de phase et la prochaine tâche.
- Ajouter les nouvelles dépendances et tâches dans la phase appropriée ; ne jamais supprimer les tâches terminées.
- En cas de changement d'architecture ou de périmètre, mettre à jour les décisions, dépendances, risques et critères concernés.
- Une phase n'est cochée dans la progression globale que lorsque tous ses critères de sortie obligatoires sont satisfaits.

---

## 1. Vision produit

### Problème à résoudre

Les joueurs disposent de bases de cartes, de decklists publiques et de conseils dispersés, mais peu d'outils réunissent contraintes réglementaires, cohérence stratégique, optimisation et explications fiables. Le produit doit transformer une intention de jeu en deck légal, éditable, analysable et compréhensible, sans déléguer les règles déterministes à un LLM.

### Utilisateurs cibles

- Cible MVP : joueurs débutants à intermédiaires souhaitant construire ou améliorer un deck pour un format donné.
- Cible secondaire : joueurs confirmés voulant accélérer l'exploration de variantes et documenter leurs choix.
- Extensions futures : joueurs compétitifs, créateurs de contenu, équipes et communautés.

### Proposition de valeur

Construire et optimiser un deck légal à partir de données traçables, avec un moteur déterministe garantissant les contraintes et une IA cantonnée à la compréhension, à l'analyse stratégique et à l'explication.

### Principes produit

- **Fiabilité avant sophistication** : aucune carte inventée, aucune violation silencieuse de règle ou de banlist.
- **Explicabilité** : chaque alerte, score ou recommandation indique sa raison et sa source.
- **Contrôle utilisateur** : toute proposition reste modifiable et revalidée en temps réel.
- **Versionnement temporel** : formats et banlists sont datés et reproductibles.
- **MVP étroit** : un format, un parcours principal, un moteur vérifiable avant les fonctions avancées.

---

## 2. Périmètre du MVP

### Inclus et indispensable

- Catalogue consultable de cartes avec recherche et filtres essentiels.
- Un format officiel clairement identifié et une banlist versionnée.
- Sélection d'un archétype ou de cartes imposées.
- Éditeur Main Deck / Extra Deck / Side Deck avec ajout, retrait et quantités.
- Validation déterministe de la taille des zones, des limites de copies et de la banlist.
- Génération algorithmique d'une proposition de deck à partir de données et règles locales.
- Analyse basique : légalité, ratios, catégories fonctionnelles, cartes clés, avertissements et synergies connues.
- Optimisation guidée avec suggestions d'ajout/retrait vérifiées avant affichage.
- Explication IA fondée exclusivement sur les résultats structurés du moteur et les données récupérées.
- Import/export d'un format de decklist simple et documenté.
- Traçabilité des versions de données, de banlist, de moteur et de modèle IA utilisées.
- Interface responsive et gestion claire des états de chargement, d'erreur et d'absence de résultat.

### Intéressant après le MVP

- Comptes, decks sauvegardés, duplication et historique des versions.
- Profils de construction : stabilité, combo, going first, going second ou budget.
- Side Deck et analyse de matchups approfondis.
- Plusieurs propositions classées et comparaison côte à côte.
- Statistiques de decks et résultats de tournois.
- Partage public ou privé et liens de consultation.
- Prise en charge complète de plusieurs langues.

### Avancé / futur

- TCG, OCG, Master Duel et formats historiques simultanés.
- Simulation de mains d'ouverture, probabilités avancées et tests de lignes de jeu.
- Simulateur de duel ou intégration avec un moteur de jeu.
- Apprentissage à partir de résultats compétitifs et métagame temporel.
- Collaboration, notation communautaire et recommandations personnalisées.
- Application mobile native et mode hors ligne.

### Hors périmètre explicite du MVP

- Arbitrage exhaustif de tous les rulings et interactions de chaînes.
- Simulation complète des duels.
- Réseau social, marketplace ou monétisation.
- Entraînement d'un modèle fondation propriétaire.
- Couverture simultanée de tous les formats et de toutes les langues.

---

## 3. Architecture et stack recommandées

### Architecture logique cible

```text
Navigateur
  └─ Application web Next.js
       └─ API FastAPI
            ├─ Catalogue et formats
            ├─ Validateur déterministe
            ├─ Deck Engine algorithmique
            ├─ Orchestrateur IA avec sorties structurées
            ├─ Workers d'import et d'enrichissement
            └─ PostgreSQL + Redis optionnel
                 └─ Sources externes versionnées
```

Le MVP démarre en **monolithe modulaire** : modules séparés par responsabilités dans un même backend et un même déploiement. Cette structure réduit le coût opérationnel tout en permettant d'extraire ultérieurement l'import, l'IA ou le moteur en services indépendants.

### Choix recommandés à valider

| Domaine | Recommandation | Alternatives | Justification |
|---|---|---|---|
| Frontend | Next.js + TypeScript + React | Vite/React, Nuxt | UX riche, rendu web moderne, typage, écosystème de tests |
| UI | Tailwind CSS + composants accessibles | CSS Modules, MUI | Itération rapide sans imposer un design system lourd |
| Backend | Python + FastAPI | NestJS, Django | Python est adapté à l'optimisation, aux données et à l'intégration IA ; contrats OpenAPI natifs |
| Validation | Pydantic | Zod côté Node | Schémas explicites et sorties IA structurées |
| Base de données | PostgreSQL | SQLite pour prototype local | Relations riches, JSONB, recherche et versionnement fiables |
| ORM/migrations | SQLAlchemy 2 + Alembic | SQLModel, Django ORM | Contrôle du modèle, maturité et migrations explicites |
| Cache/jobs | Redis + worker seulement si justifié | Tâches intégrées au départ | Import et appels IA asynchrones sans complexifier prématurément |
| Source cartes | API publique reconnue + snapshots locaux | Dataset sous licence | Découplage de l'application et reproductibilité des imports |
| Deck Engine | Python pur, règles et scoring configurables | Solveur OR-Tools ultérieur | Simple à tester d'abord ; optimisation sous contraintes ajoutable ensuite |
| IA | API LLM abstraite, sorties JSON validées | Fournisseur interchangeable, modèle local | Évite le verrouillage et empêche le modèle de contourner le moteur |
| Tests | Pytest, Vitest, Testing Library, Playwright | Cypress | Couverture unitaire, contrat, intégration et E2E |
| Observabilité | Logs structurés + OpenTelemetry/Sentry | Solution hébergeur | Diagnostic des imports, erreurs moteur et coût/latence IA |
| CI/CD | GitHub Actions | CI de l'hébergeur | Qualité reproductible avant fusion et déploiement |
| Hébergement MVP | Frontend managé + API/conteneurs + PostgreSQL managé | PaaS unique | Faible charge opérationnelle ; choix final selon budget et région |

### Répartition des responsabilités

- Le **frontend** gère l'interaction, l'édition et la visualisation ; il ne décide jamais seul de la légalité.
- Le **backend** expose des contrats versionnés, orchestre les cas d'usage et centralise les autorisations.
- La **base** conserve données normalisées, provenance, snapshots, formats, banlists et métadonnées de versions.
- Le **moteur de règles** tranche toutes les contraintes objectives.
- Le **Deck Engine** filtre, score, assemble, répare et classe des candidats sans inventer de cartes.
- Le **LLM** interprète l'intention et rédige des explications à partir d'identifiants et de faits fournis.
- Une sortie IA est toujours validée par schéma puis repassée au moteur avant d'atteindre l'utilisateur.

### Structure de repository cible

```text
YuGiOh_Deck_Building/
├── apps/
│   ├── web/                    # Frontend Next.js
│   └── api/                    # API FastAPI et orchestration
├── packages/
│   ├── contracts/              # Schémas/API partagés ou générés
│   ├── deck-engine/            # Construction, scoring, validation
│   └── ui/                     # Composants réutilisables
├── data/
│   ├── fixtures/               # Jeux de données de tests versionnés
│   └── schemas/                # Contrats d'import documentés
├── infra/                      # Conteneurs et infrastructure déclarative
├── scripts/                    # Import, maintenance et contrôles opératoires
├── tests/                      # Intégration, E2E, performance et golden sets
├── docs/                       # ADR, modèle de données, règles et exploitation
├── .github/workflows/          # CI/CD
├── README.md
└── ROADMAP.md
```

Cette arborescence est une cible de conception, pas l'état actuel du dépôt.

---

## 4. Données et modèle conceptuel

### Données à conserver localement

- `Card` : identifiant stable, noms localisés, texte d'effet, type, sous-types, attribut, niveau/rang/link, ATK/DEF, échelle Pendule.
- `CardPrinting` ou équivalent : identifiants externes, image, set et métadonnées utiles si le périmètre les requiert.
- `Archetype` et associations carte–archétype, avec provenance et niveau de confiance.
- `Format` : règles de zones, tailles, pool de cartes et paramètres spécifiques.
- `Banlist` et `BanlistEntry` : date d'effet, statut, limite de copies et source.
- `CardRelation` : synergie, recherche, invocation, coût, verrou, conflit, cible et justification.
- `FunctionalTag` : starter, extender, engine requirement, interruption, board breaker, draw, removal, etc.
- `DataSnapshot` : source, date de récupération, version, hash, statut d'import et erreurs.
- Plus tard : `Deck`, `DeckVersion`, `DeckCard`, `User`, `Tournament`, `TournamentDeck`, `MatchupStat`.

### Données externes et enrichissement

- Récupérer les faits de carte et images depuis une source autorisée ; conserver les identifiants et la provenance.
- Importer les banlists depuis une source officielle ou vérifiée et ne jamais écraser les versions historiques.
- Traiter les rulings comme un corpus séparé, sourcé et optionnel ; ne pas prétendre à l'exhaustivité dans le MVP.
- N'intégrer statistiques et données de tournois qu'après validation des licences, du grain et des biais.
- Dériver les tags et relations dans une couche versionnée, révisable et distincte des faits officiels.

---

## 5. Moteur hybride de deckbuilding

### Différence entre les approches

- **LLM seul** : utile pour le langage et les explications, mais non fiable pour les listes exactes, l'actualité des banlists et les contraintes combinatoires.
- **Algorithme seul** : reproductible et contrôlable, mais moins apte à comprendre une demande libre ou à expliquer une stratégie avec nuance.
- **Hybride recommandé** : le LLM transforme la demande en contraintes structurées et explique ; le moteur filtre, construit, score et valide.

### Pipeline cible

1. Normaliser la requête utilisateur en objectif structuré.
2. Résoudre le format, la banlist, l'archétype et les cartes imposées.
3. Construire un pool de candidats provenant exclusivement du catalogue local.
4. Appliquer les contraintes dures : disponibilité, copies, zones, banlist et taille.
5. Scorer les candidats selon rôles, synergies, conflits, courbe et profil demandé.
6. Assembler puis réparer la liste jusqu'à satisfaire toutes les contraintes.
7. Produire des métriques et raisons structurées.
8. Revalider la liste complète avec le moteur de règles.
9. Demander au LLM une explication fondée sur ces faits, sans lui confier la légalité.
10. Rejeter ou corriger toute sortie non conforme avant affichage.

### Rôle autorisé de l'IA

- Comprendre la demande en langage naturel et proposer des paramètres confirmables.
- Synthétiser forces, faiblesses et plan de jeu à partir des données du moteur.
- Expliquer les choix, alternatives et compromis.
- Formuler des conseils de matchup sur la base de données traçables.
- Générer des variantes de stratégie qui seront ensuite contraintes et validées.

### Ce qui reste déterministe

- Existence et identité des cartes.
- Nombre de cartes et zones autorisées.
- Limites de copies, banlist et pool du format.
- Validation des identifiants, types et statistiques.
- Conditions formalisées d'éligibilité et contraintes de construction.
- Calcul des métriques, provenance et contrôles de sécurité.

---

## 6. Spécification candidate de Phase 0 — à valider

> Cette section constitue une proposition suffisamment précise pour démarrer après approbation. Les recommandations ne sont pas des décisions validées. Les points nécessitant une décision explicite sont répertoriés en section 10.

### 6.1 Fonctionnalités classées par priorité

| Capacité | Classement | Spécification du premier incrément |
|---|---|---|
| Choisir format et banlist | **Indispensable MVP** | Un seul format et une version de banlist explicite ; aucun choix multi-format dans l'interface initiale |
| Rechercher une carte | **Indispensable MVP** | Recherche par nom avec fiche minimale, filtres type et archétype |
| Choisir un archétype | **Indispensable MVP** | Sélection dans un ensemble pilote annoté et supporté |
| Imposer/exclure/verrouiller des cartes | **Indispensable MVP** | Identifiants du catalogue uniquement, quantité comprise entre 0 et la limite légale |
| Générer un deck | **Indispensable MVP** | Une proposition Main + Extra Deck lorsque pertinent ; Side Deck éditable mais non optimisé automatiquement |
| Éditer le deck | **Indispensable MVP** | Ajouter, retirer et modifier une quantité ; validation après chaque changement |
| Valider la légalité | **Indispensable MVP** | Existence, zone, taille, copies cumulées, pool régional et banlist datée |
| Analyser la cohérence | **Indispensable MVP** | Ratios et couverture des rôles définis ; synergies/conflits uniquement s'ils sont annotés |
| Optimiser | **Indispensable MVP** | Une proposition d'échanges ajout/retrait avec impact, jamais appliquée sans action utilisateur |
| Expliquer avec l'IA | **Indispensable MVP** | Plan de jeu, rôle des cartes clés, alertes et limites à partir des faits fournis par le moteur |
| Import/export | **Indispensable MVP** | Format YDK proposé ; import avec rapport des cartes inconnues et export des trois zones |
| Fonctionner sans IA | **Indispensable MVP** | Édition, validation, génération algorithmique et métriques restent disponibles |
| Comptes et sauvegarde cloud | **Après MVP** | Authentification, bibliothèque personnelle et historique synchronisé |
| Historique de versions | **Après MVP** | Snapshots immuables, comparaison et restauration |
| Optimisation automatique du Side Deck | **Après MVP** | Nécessite un modèle de matchup et des données de menace/réponse |
| Profils budget/going first/going second | **Après MVP** | Paramètres de scoring séparés et évalués |
| Plusieurs variantes classées | **Après MVP** | Diversité contrôlée et comparaison des compromis |
| Statistiques de tournois/métagame | **Après MVP** | Uniquement avec source, licence, période et biais documentés |
| Partage public et collaboration | **Après MVP** | Permissions, modération et confidentialité préalables |
| Multi-langue complet | **Après MVP** | Anglais canonique proposé, interface française possible au MVP |
| Formats multiples visibles | **Après MVP** | Un adaptateur supplémentaire seulement après stabilisation du TCG |
| Simulation de duel | **À ne pas prévoir maintenant** | Projet distinct par sa complexité réglementaire et combinatoire |
| Rulings exhaustifs | **À ne pas prévoir maintenant** | Le MVP affiche ses limites au lieu de prétendre arbitrer toute interaction |
| Entraînement d'un LLM propriétaire | **À ne pas prévoir maintenant** | Aucun avantage validé face à un modèle hébergé et des données structurées |
| Réseau social/marketplace | **À ne pas prévoir maintenant** | Hors proposition de valeur initiale |
| Application mobile native | **À ne pas prévoir maintenant** | Web responsive suffisant pour tester la valeur |

#### Contrat fonctionnel du MVP proposé

1. L'utilisateur choisit l'un des archétypes pilotes ou part d'une liste existante.
2. Il ajoute des cartes obligatoires, interdit certaines cartes et peut formuler une intention libre courte.
3. Le système affiche le format et la banlist datée qui seront appliqués.
4. Le moteur génère une unique proposition légale ou explique précisément pourquoi aucune solution n'a été trouvée.
5. L'utilisateur modifie Main/Extra/Side et reçoit des erreurs déterministes en temps réel.
6. L'analyse affiche légalité, répartition Monstre/Magie/Piège, rôles fonctionnels, cartes clés, synergies annotées, conflits connus et avertissements.
7. L'optimiseur propose des échanges vérifiés ; l'utilisateur décide de les appliquer.
8. Le LLM explique les résultats structurés sans ajouter de carte ni modifier la liste.
9. L'utilisateur exporte le deck ; aucun compte n'est requis au MVP et le brouillon peut être conservé localement.

#### Définition proposée d'un résultat MVP acceptable

- La decklist est légale pour le snapshot de format affiché.
- Toutes les cartes et tous les faits cités existent dans le catalogue local.
- Les cartes verrouillées sont conservées ou une impossibilité motivée est retournée.
- L'analyse sépare explicitement règles, heuristiques et texte généré.
- Une réponse sans solution est préférable à une liste illégale ou inventée.
- La génération algorithmique cible une réponse p95 inférieure à 3 secondes hors import ; l'explication IA cible un p95 inférieur à 15 secondes.

### 6.2 Comparaison des stratégies de format

| Option | Avantages | Inconvénients | Verdict |
|---|---|---|---|
| **A — TCG actuel seulement** | Banlist et règles officielles publiques ; Side Deck pertinent ; public papier large ; historique disponible | Risque de coupler le domaine au TCG ; différences régionales et dates de sortie à traiter | Bon périmètre, architecture trop fermée si appliquée littéralement |
| **B — Master Duel seulement** | Environnement numérique homogène ; intérêt pour le ladder ; pas de légalité régionale papier | Banlist et pool propres, règles de match/Side Deck différentes ; source officielle automatisable moins évidente ; mises à jour fréquentes | Plus risqué pour une première source de vérité |
| **C — Plusieurs formats dès le MVP** | Valeur immédiate pour plusieurs publics ; modèle testé tôt | Multiplie règles, imports, snapshots, tests, UX et maintenance avant validation produit | Rejeté pour le MVP |
| **D — Architecture multi-format, un format implémenté** | Maintient le MVP petit ; force des frontières propres ; facilite TCG/OCG/Master Duel/historique plus tard | Exige une abstraction minimale et discipline pour ne pas surconcevoir | **Recommandé** |

#### Recommandation précise à approuver

- Architecture **Option D**, avec un contrat `FormatRules` versionné et des données séparées des règles.
- Seul adaptateur MVP : **TCG Advanced, territoire EMEA**, parties en match, banlist officielle applicable à la date du snapshot.
- Anglais comme identifiant textuel canonique et français comme langue d'interface ; les identifiants numériques restent la référence technique.
- Main Deck de 40 à 60 cartes, Extra Deck de 0 à 15 et Side Deck de 0 à 15 ; limite normale de trois copies cumulées, réduite par la banlist.
- Légalité régionale fondée sur la date de sortie TCG EMEA lorsque l'information est suffisamment fiable ; toute donnée incertaine est signalée et non inventée.
- Le Side Deck peut être édité et validé au MVP, mais sa génération/optimisation par matchup est différée.
- Les formats futurs ajoutent un adaptateur, un pool de cartes, des versions de banlist et des tests de conformité, sans branche conditionnelle dispersée dans l'application.

### 6.3 Pipeline du Deck Builder

```text
Entrée formulaire ou langage naturel
  → résolution de l'intention et des noms [LLM optionnel + catalogue]
  → confirmation des ambiguïtés [interface]
  → chargement FormatSnapshot/BanlistSnapshot [déterministe]
  → normalisation des contraintes [déterministe]
  → création du pool de candidats [requêtes + règles déterministes]
  → enrichissement par rôles/synergies connus [données versionnées]
  → filtrage des contraintes dures [Rules Engine]
  → génération de plusieurs candidats internes [algorithme]
  → scoring et réparation [algorithme explicable]
  → validation finale indépendante [Rules Engine]
  → analyse structurée [algorithme]
  → explication en langage naturel [LLM]
  → vérification des références et rendu [backend + frontend]
```

| Étape | Nature | Règle de conception |
|---|---|---|
| Compréhension d'une phrase libre | LLM facultatif | Sortie JSON stricte ; l'utilisateur confirme toute ambiguïté importante |
| Résolution des noms | Déterministe | Recherche dans le catalogue ; aucun nom libre ne devient une carte |
| Choix format/banlist | Déterministe/UI | Valeur explicite, jamais déduite silencieusement |
| Pool de candidats | Déterministe | Cartes locales légales et relations connues uniquement |
| Construction | Algorithmique | Heuristique versionnée, seed enregistrable et budget de calcul borné |
| Légalité | Déterministe | Validateur indépendant utilisé avant et après toute transformation |
| Synergies/conflits | Données + algorithme | Faits annotés avec provenance et confiance ; pas d'affirmation absolue non sourcée |
| Optimisation | Algorithmique | Recherche d'échanges améliorant des objectifs explicites sous contraintes dures |
| Explication | LLM | Reçoit deck, métriques et raisons ; ne peut pas changer les identifiants |

### 6.4 Contrat exact du LLM

#### Le LLM est autorisé à

- Transformer une intention libre en `DeckRequest` structuré : archétype, style, cartes imposées/exclues et contraintes souples.
- Reformuler une ambiguïté sous forme de question courte.
- Expliquer le plan de jeu, les rôles, les compromis et les alertes calculés.
- Comparer deux versions à partir de leurs différences structurées.
- Proposer des objectifs d'optimisation ; le moteur reste seul habilité à proposer les cartes finales.
- Produire du texte français à partir de données canoniques anglaises.

#### Le LLM n'est pas autorisé à

- Affirmer l'existence, l'effet, le type, les statistiques ou la légalité d'une carte de mémoire.
- Choisir implicitement une banlist ou substituer un format.
- Ajouter, retirer ou changer une quantité dans la decklist validée.
- Calculer tailles, limites de copies, probabilités ou scores affichés.
- Déclarer qu'une interaction/ruling est légale sans fait vérifié fourni.
- Appeler directement la base ou une source externe depuis une réponse utilisateur.
- Masquer l'incertitude, les données manquantes ou l'échec du validateur.

#### Garde-fous contractuels

- Entrées minimales et structurées ; textes de cartes marqués comme données non fiables pour les instructions.
- Sorties JSON validées par schéma, identifiants recoupés avec la base et références inconnues rejetées.
- Température faible pour extraction et explication factuelle ; prompts et modèle versionnés.
- Après toute suggestion affectant un deck, passage obligatoire par le Rules Engine.
- Mode dégradé sans LLM : formulaire, génération, validation et analyse chiffrée restent utilisables.
- Aucun RAG sémantique ni base vectorielle au MVP : requêtes SQL, recherche textuelle et relations structurées suffisent. Un RAG ne sera étudié que pour un corpus de rulings sourcé devenu nécessaire.

### 6.5 Modèle conceptuel proposé

| Entité | Rôle et données principales | Relations |
|---|---|---|
| `Card` | Identité canonique, passcode, noms localisés, texte, catégorie, type/race, attribut, niveau/rang/link, ATK/DEF, échelle, statut et provenance | N–N avec `Archetype`, 1–N avec `DeckCard` et `BanlistEntry`, N–N dirigée via `CardRelation` |
| `Archetype` | Nom canonique, alias localisés, description courte, statut de support MVP | N–N avec `Card`; 1–N avec profils/annotations stratégiques futurs |
| `CardRelation` | Carte source/cible, type de relation, sens, poids, justification, provenance, confiance et version | Deux relations vers `Card`; optionnellement limitée à un `Format` |
| `FunctionalTag` | Rôle stratégique contrôlé et versionné | N–N avec `Card`, avec poids, contexte et provenance |
| `Format` | Code stable, nom, famille TCG/OCG/MD/historique, territoire, règles de zones et politique de légalité | 1–N avec `FormatSnapshot`, `Banlist` et `Deck` |
| `FormatSnapshot` | Version immuable du pool et des paramètres à une date d'effet | N–1 `Format`; N–1 `DataSnapshot`; référencé par analyses/decks |
| `Banlist` | Nom, format, territoire, date d'annonce, date d'effet, source, statut et version | N–1 `Format`; 1–N `BanlistEntry`; référencée par `Deck`/`DeckVersion` |
| `BanlistEntry` | Carte, limite 0/1/2/3, note et provenance | N–1 `Banlist`; N–1 `Card`; unicité carte/banlist |
| `Deck` | Identité logique, titre, format, archétype principal, propriétaire facultatif, état et dates | 1–N `DeckVersion`; N–1 `User` optionnel; N–1 `Format` |
| `DeckVersion` | Snapshot immuable, numéro, parent, banlist, format snapshot, paramètres moteur/LLM, métriques et commentaire | N–1 `Deck`; 1–N `DeckCard`; N–1 `Banlist` et `FormatSnapshot` |
| `DeckCard` | Version, carte, zone MAIN/EXTRA/SIDE, quantité, verrou utilisateur et origine manuelle/générée | N–1 `DeckVersion`; N–1 `Card`; unicité version/carte/zone |
| `User` | Identifiant, fournisseur d'authentification, préférences minimales, consentements et dates | 1–N `Deck`; **table différée après MVP**, deck anonyme au MVP |
| `Matchup` | Deux stratégies/archétypes, format/période, échantillon, métriques, source et confiance | Lié à `Format`, archétypes et données de tournoi ; **différé après MVP** |
| `DataSource` | Fournisseur, URL, licence/conditions, priorité et type de données | 1–N `DataSnapshot` |
| `DataSnapshot` | Horodatage, version distante, hash, schéma, statut, volumes et rapport qualité | N–1 `DataSource`; source des faits importés et `FormatSnapshot` |

Principes : deck et version sont séparés pour rendre une analyse reproductible ; banlists et snapshots sont immuables ; les annotations stratégiques ne sont jamais mélangées aux faits officiels ; `User` et `Matchup` restent modélisés mais ne doivent pas imposer leur implémentation au MVP.

### 6.6 Stratégie de sources de données proposée

| Donnée | Source candidate | Stockage local | Mise à jour proposée | Risques / décision |
|---|---|---|---|---|
| Cartes, propriétés, textes, archétypes, images | YGOPRODeck API v7 comme source opérationnelle candidate | Snapshot complet ; images auto-hébergées seulement si droits confirmés | Vérification quotidienne de version, import si changement | API communautaire, archétypes éditoriaux, limite 20 req/s, demande de ne pas hotlinker ; conditions d'usage à valider |
| Texte/référence officielle d'une carte | Base officielle KONAMI/NEURON pour contrôle humain ciblé | Ne pas scraper sans autorisation ; conserver URL/provenance | À la revue des anomalies | Pas d'API publique documentée retenue ; automatisation et licence à clarifier |
| Banlist TCG EMEA | Page officielle KONAMI Forbidden & Limited | Snapshot immuable avec date d'effet et copie structurée vérifiée | Surveillance quotidienne + vérification humaine à annonce | HTML susceptible de changer ; importer par workflow contrôlé et tests sentinelles |
| Légalité régionale/date de sortie | Pages KONAMI + dates de sets de la source catalogue | Oui, versionnée par territoire | À chaque nouvelle sortie/import | Produits/dates différents selon territoire ; données tierces à recouper |
| Règles générales | Rulebook et Tournament Policy KONAMI | Références documentaires, règles codées avec citation/version | À chaque publication de politique | Une règle codée nécessite revue ; le moteur MVP ne couvre pas tous les rulings |
| Rulings | Base officielle lorsque consultable, corpus autorisé à déterminer | Pas de corpus complet au MVP | Manuel pour cas pilotes | Couverture, structure, langue et droits incertains |
| Relations/synergies/rôles | Curation interne à partir d'expertise et tests | Oui, avec auteur, justification, confiance et version | À chaque release de contenu | Subjectivité ; revue humaine et distinction fait/heuristique |
| Decklists/statistiques tournoi | Aucune source retenue au MVP | Non | Après étude dédiée | Licences, biais, doublons, représentativité et données personnelles |
| Prix | Hors MVP | Non | Sans objet | Volatilité, territoires, affiliation et maintenance |

Liens de référence candidats :

- Catalogue : <https://ygoprodeck.com/api-guide/>
- Banlist officielle EMEA : <https://www.yugioh-card.com/eu/play/forbidden-and-limited-list/>
- Légalité des cartes EMEA : <https://www.yugioh-card.com/eu/play/card-legality/>
- Ressources de jeu KONAMI : <https://www.yugioh-card.com/eu/play/>

### 6.7 Stack technique candidate

| Couche | Choix recommandé | Portée et justification |
|---|---|---|
| Monorepo | `pnpm` workspaces + orchestration légère ; Python géré séparément dans le même dépôt | Un seul produit et contrats visibles ; éviter les microservices et outils monorepo lourds au départ |
| Frontend | Next.js, React, TypeScript strict | Éditeur interactif, routing et rendu souples ; écosystème mûr et contrats typés |
| UI | Tailwind CSS + composants Radix/shadcn adaptés, sans dépendance au code distant à l'exécution | Construction rapide, accessibilité primitive et personnalisation ; éviter une grosse bibliothèque visuelle rigide |
| Données frontend | TanStack Query ; état local React ou Zustand seulement si l'éditeur le justifie | Séparer état serveur et brouillon ; ne pas imposer Redux sans besoin |
| Backend | Python 3.13+, FastAPI, Pydantic v2, Uvicorn | Même langage que moteur/IA, contrats explicites, API OpenAPI et excellent outillage de validation |
| API | REST JSON `/api/v1`; traitement synchrone tant que les SLO sont tenus | Plus simple que GraphQL ; jobs ajoutés uniquement pour import ou génération réellement longue |
| Base | PostgreSQL 17+ | Contraintes relationnelles, JSONB ciblé, texte, index et migrations robustes |
| ORM | SQLAlchemy 2 + Alembic | Séparation domaine/persistance et migrations explicites ; éviter de faire porter les règles métier à l'ORM |
| Recherche | PostgreSQL trigram/full-text | Suffisant pour noms/alias au MVP ; pas d'Elasticsearch avant mesure |
| Cache/jobs | Aucun requis au premier incrément ; Redis + worker ultérieurement si mesure | Réduit l'exploitation ; import peut être une commande contrôlée au départ |
| Deck Engine | Package Python pur, sans accès réseau/DB, architecture ports-adapters | Tests rapides, fonctions déterministes, réutilisable en batch et API |
| Solveur | Heuristiques explicables d'abord ; OR-Tools seulement si contraintes/scoring l'exigent | Éviter une optimisation opaque et prématurée |
| LLM | OpenAI Responses API via un port fournisseur, modèle à choisir par évaluation/coût | Sorties structurées et abstraction ; aucun nom de modèle figé avant benchmark |
| RAG/vector DB | Aucun au MVP | Catalogue relationnel structuré ; ajouter `pgvector` seulement si un corpus documentaire et une évaluation prouvent le besoin |
| Tests backend | Pytest, Hypothesis, Testcontainers | Règles unitaires, invariants génératifs et intégration PostgreSQL réelle |
| Tests frontend | Vitest, Testing Library, axe | Composants, parcours clavier et accessibilité |
| E2E | Playwright | Parcours web complet et multi-navigateur |
| Qualité | Ruff, mypy/pyright à choisir, ESLint, Prettier, TypeScript strict | Contrôles rapides et reproductibles ; un seul type-checker Python sera retenu |
| Local | Docker Compose pour PostgreSQL ; applications exécutables nativement ; fichier `.env.example` sans secret | Onboarding simple et boucle de développement rapide |
| CI/CD | GitHub Actions | Lint, types, tests, migrations, build, audit, artefacts puis staging/production |
| Hébergement | Vercel pour web + Render/Fly.io/Cloud Run pour API + PostgreSQL managé, à arbitrer selon budget/région | Déploiement simple ; aucune sélection finale avant estimation des coûts et exigences de résidence |
| Observabilité | OpenTelemetry + Sentry ou service équivalent | Corrélation frontend/API/moteur/LLM et suivi coût/latence |

### 6.8 Architecture logique et responsabilités

```text
[Web Next.js]
    │ REST /api/v1 — DTO versionnés
    ▼
[FastAPI / Application Services]
    ├── Catalogue ───────────────► [Repositories PostgreSQL]
    ├── Deck Use Cases ──────────► [Deck Engine Python pur]
    │                                  ├── Candidate Generator
    │                                  ├── Scorer / Optimizer
    │                                  └── Rules Engine indépendant
    ├── AI Orchestrator ─────────► [LLM Provider]
    │        │                         (texte/JSON, jamais source de vérité)
    │        └── ne reçoit que les faits produits par les moteurs
    └── Import Application Service► [Source Adapters]
                                        ├── YGOPRODeck
                                        └── KONAMI / workflow contrôlé
```

- **Web** : collecte l'intention, édite un brouillon et présente les résultats ; aucune décision de légalité locale faisant autorité.
- **Application API** : autorisations, transaction, orchestration, version des contrats, idempotence et journalisation.
- **Domaine** : entités, valeurs, politiques de format et résultats ; indépendant de FastAPI/SQLAlchemy/LLM.
- **Rules Engine** : service pur et indépendant de génération ; peut invalider n'importe quelle sortie.
- **Deck Engine** : consomme un snapshot en mémoire et retourne candidats, métriques et raisons ; aucun accès direct réseau.
- **Repositories** : traduisent les modèles persistés sans exposer les tables au domaine.
- **AI Orchestrator** : construit le contexte minimal, valide les schémas et applique les garde-fous ; ne contourne jamais les cas d'usage.
- **Import** : hors requête utilisateur, idempotent, auditable et capable de publier un snapshot seulement après contrôles qualité.
- **Monolithe modulaire** : ces frontières sont des modules, pas des services réseau distincts au MVP.

---

# Phases, Epics, Features et Tasks

## Phase 0 — Conception

**Objectif :** figer un périmètre MVP testable, les contrats produit et les décisions techniques minimales avant tout code.

**Dépendances :** aucune.

**Résultat attendu :** dossier de conception approuvé, risques propriétaires, sources vérifiées et backlog priorisé.

### Epic 0.1 — Cadrage produit

- [x] Analyser le cahier des charges initial.
- [x] Auditer l'état initial du repository.
- [x] Créer le tableau de bord `ROADMAP.md`.
- [x] Rédiger une spécification candidate détaillée du MVP avec inclusions, reports et exclusions.
- [x] Définir le contrat fonctionnel candidat et la définition d'un résultat acceptable.
- [ ] Valider les personas prioritaires et leurs problèmes principaux.
- [ ] Valider le parcours principal : sélectionner, construire, éditer, valider, analyser et exporter.
- [ ] Valider le périmètre inclus, différé et explicitement exclu du MVP.
- [ ] Définir les indicateurs de succès du MVP : activation, deck valide obtenu, délai de génération, taux d'erreurs et satisfaction.
- [ ] Définir les exigences non fonctionnelles : disponibilité, latence, accessibilité, confidentialité et navigateurs pris en charge.
- [ ] Rédiger les user stories prioritaires avec critères d'acceptation observables.

### Epic 0.2 — Décisions Yu-Gi-Oh! structurantes

- [x] Comparer TCG actuel, Master Duel, multi-format immédiat et architecture multi-format progressive.
- [x] Documenter la recommandation Option D : architecture multi-format, TCG Advanced EMEA seul au MVP.
- [ ] Choisir un unique format cible pour le MVP.
- [ ] Choisir la région, la langue canonique et les langues d'affichage.
- [ ] Définir précisément les limites Main/Extra/Side du format retenu.
- [ ] Définir la politique de date et de version de banlist.
- [ ] Définir ce que signifie « compatible » ou « synergique » dans le MVP.
- [ ] Définir les catégories fonctionnelles utilisées par l'analyse.
- [ ] Lister les interactions/rulings hors périmètre et la manière de les signaler.
- [ ] Choisir cinq archétypes pilotes représentatifs pour la génération et les golden tests.

### Epic 0.3 — Sources, licences et conformité

- [x] Comparer les sources candidates de cartes, images, banlists, règles, rôles et statistiques.
- [x] Documenter YGOPRODeck comme source opérationnelle candidate et KONAMI comme source officielle de banlist/règles.
- [ ] Sélectionner la source primaire et une stratégie de repli.
- [ ] Vérifier les licences, règles d'attribution, droits sur les images et marques.
- [ ] Documenter la politique de conservation, rafraîchissement et suppression des données externes.
- [ ] Définir les mentions légales et avertissements nécessaires.
- [ ] Réaliser un import exploratoire non applicatif pour contrôler les champs et anomalies de la source retenue.
- [ ] Confirmer si les images peuvent être auto-hébergées dans le produit et selon quelles conditions.
- [ ] Définir un workflow humain de vérification et publication d'une nouvelle banlist.

### Epic 0.4 — Architecture et gouvernance

- [x] Définir le pipeline candidat de la demande utilisateur jusqu'à l'explication.
- [x] Définir la séparation candidate entre système déterministe, algorithmes et LLM.
- [x] Définir un modèle conceptuel candidat et ses principales relations.
- [x] Proposer la stack complète, l'architecture logique et les responsabilités des modules.
- [x] Classer les décisions restantes par niveau de blocage.
- [ ] Valider le monolithe modulaire, la stack et la structure cible du repository.
- [ ] Choisir le fournisseur LLM, le modèle initial, le budget et les limites d'usage.
- [ ] Choisir l'hébergement, la région des données et les environnements.
- [ ] Écrire les ADR pour les décisions irréversibles ou coûteuses à modifier.
- [ ] Définir conventions Git, revue, versionnement, secrets et dépendances.
- [ ] Prioriser le backlog et attribuer dépendances, risques et critères de sortie.
- [ ] Valider l'absence de RAG/vector database au MVP et les conditions de réévaluation.
- [ ] Valider le format YDK comme format d'import/export initial.

### Critères de sortie de la phase 0

- [ ] Le MVP et ses exclusions sont approuvés par le porteur du projet.
- [ ] Format, banlist, source de données, périmètre MVP et stack sont explicitement approuvés.
- [ ] Les risques juridiques bloquants ont une réponse acceptable.
- [ ] Les user stories MVP ont des critères d'acceptation testables.
- [ ] Les ADR structurantes et le modèle conceptuel sont documentés.
- [ ] La séparation LLM/algorithme/règles et le pipeline sont approuvés.
- [ ] Les archétypes pilotes et le golden set initial sont définis.
- [ ] Les objectifs de performance, qualité, coût et accessibilité sont chiffrés.
- [ ] Toutes les décisions 🔴 sont prises et aucune inconnue bloquante ne subsiste.

---

## Phase 1 — Données

**Objectif :** disposer d'un catalogue local fiable, versionné et interrogeable.

**Dépendances :** décisions de format, source et licence de la phase 0.

**Résultat attendu :** snapshot reproductible de cartes et d'une banlist, avec rapport d'intégrité.

### Epic 1.1 — Modèle de données

- [ ] Définir le schéma `Card` et les champs spécifiques Monstre/Magie/Piège.
- [ ] Définir les types contrôlés : attributs, types, sous-types, niveaux, rangs, links et échelles.
- [ ] Définir `Archetype`, ses alias et la relation plusieurs-à-plusieurs avec les cartes.
- [ ] Définir `Format`, `Banlist`, `BanlistEntry` et leurs règles temporelles.
- [ ] Définir `CardRelation`, `FunctionalTag`, provenance et confiance.
- [ ] Définir `DataSource` et `DataSnapshot` pour l'audit des imports.
- [ ] Ajouter contraintes, index, unicité et stratégie de migration.
- [ ] Documenter le dictionnaire de données et les identifiants canoniques.

### Epic 1.2 — Pipeline d'import

- [ ] Définir le contrat de l'adaptateur de source externe.
- [ ] Implémenter la récupération paginée avec temporisation, reprise et limitation de débit.
- [ ] Valider et normaliser les données brutes sans perdre la provenance.
- [ ] Rendre l'import idempotent et transactionnel.
- [ ] Conserver le snapshot, son hash, sa date et les erreurs de ligne.
- [ ] Importer les cartes du pool MVP.
- [ ] Importer et versionner la banlist MVP.
- [ ] Gérer les cartes modifiées, retirées et rééditées sans casser les decklists.
- [ ] Prévoir une commande de rafraîchissement et une procédure de retour arrière.

### Epic 1.3 — Qualité des données

- [ ] Tester les champs obligatoires, domaines de valeurs et identifiants uniques.
- [ ] Détecter doublons, références orphelines et incohérences de type.
- [ ] Comparer les volumes et échantillons avec la source.
- [ ] Vérifier les limites de banlist sur un jeu de cartes connu.
- [ ] Constituer des fixtures petites, stables et représentatives pour les tests.
- [ ] Générer un rapport d'import lisible et bloquer la publication en cas d'erreur critique.
- [ ] Documenter la fraîcheur attendue et les alertes de dérive de schéma.

### Critères de sortie de la phase 1

- [ ] Un import complet est reproductible et idempotent.
- [ ] Chaque donnée critique a une provenance et une version.
- [ ] Les contrôles d'intégrité et fixtures passent automatiquement.
- [ ] La banlist active et ses versions historiques sont requêtables.

---

## Phase 2 — Backend

**Objectif :** exposer des cas d'usage stables et sécurisés via une API versionnée.

**Dépendances :** catalogue et banlist fiables.

**Résultat attendu :** API documentée couvrant catalogue, formats, validation et brouillons de decks.

### Epic 2.1 — Socle API

- [ ] Initialiser l'application backend selon l'architecture validée.
- [ ] Mettre en place configuration typée par environnement et gestion sûre des secrets.
- [ ] Configurer ORM, migrations et cycle de connexion à la base.
- [ ] Définir schéma d'erreur uniforme, identifiants de corrélation et journalisation structurée.
- [ ] Exposer endpoints de santé, disponibilité et version.
- [ ] Générer et versionner la documentation OpenAPI.
- [ ] Définir limites de requêtes, tailles maximales et politique CORS.

### Epic 2.2 — Catalogue et référentiels

- [ ] Exposer recherche de cartes paginée et normalisée.
- [ ] Exposer filtres par nom, type, attribut, niveau/rang/link et archétype.
- [ ] Exposer le détail d'une carte avec provenance.
- [ ] Exposer formats, banlists disponibles et version active.
- [ ] Exposer les archétypes et cartes associées.
- [ ] Tester contrats, pagination, filtres, erreurs et performances de base.

### Epic 2.3 — Contrats de deck

- [ ] Définir les DTO de deck, zones, quantités, préférences et contraintes utilisateur.
- [ ] Créer l'endpoint de validation déterministe.
- [ ] Créer les contrats de génération, analyse et optimisation asynchrones ou synchrones selon mesure.
- [ ] Retourner des erreurs localisées et liées aux cartes/zones concernées.
- [ ] Ajouter idempotence et annulation pour les opérations longues si nécessaire.
- [ ] Versionner les contrats pour permettre l'évolution des moteurs.

### Epic 2.4 — Sécurité et exploitation

- [ ] Modéliser les menaces principales : abus LLM, injection, fuite de secrets, entrées volumineuses et dépendances.
- [ ] Valider toutes les entrées et neutraliser le contenu externe avant usage IA.
- [ ] Ajouter quotas et protection des endpoints coûteux.
- [ ] Définir rétention des logs sans données sensibles inutiles.
- [ ] Scanner dépendances et secrets en CI.

### Critères de sortie de la phase 2

- [ ] Les contrats MVP sont documentés et couverts par tests.
- [ ] L'API retrouve une carte, une banlist et valide une decklist de test.
- [ ] Les erreurs sont déterministes, traçables et exploitables par le frontend.
- [ ] Les contrôles de sécurité critiques sont actifs.

---

## Phase 3 — Deck Engine

**Objectif :** construire, valider et analyser des decks de manière déterministe et reproductible.

**Dépendances :** modèle, fixtures et contrats backend.

**Résultat attendu :** une decklist légale générée sans intervention du LLM, accompagnée de scores et raisons.

### Epic 3.1 — Moteur de règles

- [ ] Définir l'interface versionnée d'une règle et le contexte de validation.
- [ ] Valider les tailles Main Deck, Extra Deck et Side Deck du format.
- [ ] Valider les quantités par identifiant canonique sur toutes les zones.
- [ ] Appliquer les limites Forbidden/Limited/Semi-Limited de la bonne banlist.
- [ ] Valider l'éligibilité d'une carte dans le format et la zone.
- [ ] Détecter identifiants inconnus, doublons et quantités invalides.
- [ ] Produire codes d'erreur, sévérité, message et suggestion de correction.
- [ ] Garantir le même résultat pour les mêmes données et versions.

### Epic 3.2 — Ontologie stratégique minimale

- [ ] Définir les rôles fonctionnels retenus pour le MVP.
- [ ] Annoter un premier ensemble d'archétypes et staples avec provenance/confiance.
- [ ] Modéliser synergies, conflits, prérequis et verrous importants.
- [ ] Distinguer faits vérifiés, règles dérivées et appréciations heuristiques.
- [ ] Créer une procédure de revue et correction des annotations.

### Epic 3.3 — Génération algorithmique

- [ ] Définir l'entrée normalisée : archétype, cartes imposées, cartes exclues, style et format.
- [ ] Construire le pool de candidats sans faire appel au LLM.
- [ ] Appliquer les contraintes dures avant le scoring.
- [ ] Définir des scores explicables pour rôles, synergies, cohérence et conflits.
- [ ] Assembler les moteurs, cartes de support, staples et cartes techniques.
- [ ] Remplir Main/Extra/Side selon les capacités validées du MVP.
- [ ] Réparer automatiquement les listes incomplètes ou surdimensionnées.
- [ ] Revalider systématiquement le résultat final.
- [ ] Retourner les raisons d'inclusion/exclusion et la version des règles.

### Epic 3.4 — Analyse et optimisation

- [ ] Calculer ratios de cartes, rôles et dépendances.
- [ ] Détecter manque de starters, excès de cartes mortes et conflits connus.
- [ ] Identifier les cartes clés et les points de fragilité.
- [ ] Proposer une modification sous forme `retirer/ajouter/raison/impact`.
- [ ] Garantir que toute suggestion préserve ou rétablit la légalité.
- [ ] Comparer les scores avant/après sans présenter l'heuristique comme une vérité absolue.
- [ ] Permettre à l'utilisateur de verrouiller des cartes contre le retrait.

### Critères de sortie de la phase 3

- [ ] Le moteur refuse toutes les fixtures volontairement illégales.
- [ ] Il génère des decks légaux pour les scénarios MVP de référence.
- [ ] Chaque score, alerte et choix est traçable et reproductible.
- [ ] Aucun résultat ne dépend du LLM pour être légal.

---

## Phase 4 — IA

**Objectif :** ajouter compréhension et explication sans compromettre la fiabilité du moteur.

**Dépendances :** contrats structurés et Deck Engine opérationnel.

**Résultat attendu :** interactions naturelles, réponses ancrées et contrôlées en coût.

### Epic 4.1 — Abstraction et garde-fous IA

- [ ] Définir une interface fournisseur/modèle interchangeable.
- [ ] Définir schémas JSON stricts pour intention, explication et suggestions.
- [ ] Versionner prompts, modèles, paramètres et schémas.
- [ ] Définir timeouts, retries bornés, fallback et circuit breaker.
- [ ] Mesurer tokens, coût, latence et taux d'échec par cas d'usage.
- [ ] Filtrer les données sensibles et résister aux instructions injectées via contenu externe.

### Epic 4.2 — Compréhension de la demande

- [ ] Extraire format, archétype, cartes imposées/exclues et préférence de style.
- [ ] Résoudre les noms vers des identifiants du catalogue.
- [ ] Signaler ambiguïtés et demander confirmation uniquement lorsque nécessaire.
- [ ] Rejeter les identifiants ou contraintes inventés.
- [ ] Prévoir un formulaire utilisable sans langage naturel.

### Epic 4.3 — Explications fondées

- [ ] Fournir au LLM uniquement faits, scores et relations nécessaires.
- [ ] Expliquer plan de jeu, rôles et compromis sans inventer d'effets.
- [ ] Citer les cartes et la banlist utilisées dans l'analyse.
- [ ] Séparer faits déterministes, heuristiques et conseils stratégiques.
- [ ] Vérifier les identifiants et affirmations structurées après génération.
- [ ] Offrir un message utile quand le service IA est indisponible.

### Epic 4.4 — Évaluation IA

- [ ] Constituer un jeu d'évaluation versionné de requêtes normales, ambiguës et adversariales.
- [ ] Mesurer extraction correcte, fidélité aux faits, légalité finale, coût et latence.
- [ ] Ajouter tests de non-régression des prompts.
- [ ] Définir seuils bloquants avant mise en production.
- [ ] Organiser une revue humaine de la qualité stratégique sur un échantillon.

### Critères de sortie de la phase 4

- [ ] Toute sortie IA respecte un schéma et les identifiants du catalogue.
- [ ] Toute decklist ou suggestion passe par le moteur de règles.
- [ ] Les évaluations atteignent les seuils validés.
- [ ] Coût, latence et fallback sont observables et bornés.

---

## Phase 5 — Frontend

**Objectif :** livrer un parcours accessible et efficace de la sélection à l'export.

**Dépendances :** API et contrats stabilisés.

**Résultat attendu :** application web responsive couvrant le parcours MVP.

### Epic 5.1 — Fondations UX/UI

- [ ] Cartographier parcours, états et erreurs du MVP.
- [ ] Produire wireframes puis prototype testable.
- [ ] Définir tokens visuels et composants accessibles.
- [ ] Mettre en place navigation, layout responsive et gestion d'état.
- [ ] Générer ou partager les types de contrats API.
- [ ] Prévoir clavier, focus, contraste, lecteurs d'écran et réduction des animations.

### Epic 5.2 — Catalogue et sélection

- [ ] Créer l'accueil avec proposition de valeur et point d'entrée clair.
- [ ] Créer la recherche de cartes avec filtres, pagination et états vides.
- [ ] Créer la fiche carte avec données et provenance utiles.
- [ ] Créer la sélection d'archétype, format, banlist et préférences.
- [ ] Permettre d'imposer, exclure ou verrouiller une carte.

### Epic 5.3 — Deck Editor

- [ ] Afficher distinctement Main, Extra et Side Deck.
- [ ] Ajouter, retirer, déplacer et modifier la quantité d'une carte.
- [ ] Afficher compteurs, limites et violations en temps réel.
- [ ] Empêcher les actions impossibles tout en expliquant leur cause.
- [ ] Préserver le brouillon local lors d'une actualisation ou erreur réseau.
- [ ] Permettre réinitialisation avec confirmation et annulation raisonnable.

### Epic 5.4 — Génération, analyse et optimisation

- [ ] Afficher la progression et permettre l'annulation d'une génération longue.
- [ ] Présenter la decklist générée avec raisons d'inclusion.
- [ ] Présenter légalité, ratios, rôles, forces, faiblesses et avertissements.
- [ ] Présenter les modifications proposées avec impact avant application.
- [ ] Revalider immédiatement après chaque modification.
- [ ] Permettre import et export dans le format retenu.
- [ ] Afficher clairement source, date de banlist et version d'analyse.

### Epic 5.5 — Pages différées mais préparées

- [ ] Définir les routes futures pour compte, decks sauvegardés et historique sans les implémenter.
- [ ] Prévoir l'extension vers comparaison, matchups et side decking dans les contrats UI.
- [ ] Valider que le modèle de navigation n'empêche pas l'ajout de plusieurs formats.

### Critères de sortie de la phase 5

- [ ] Le parcours principal fonctionne sur mobile et bureau avec données simulées ou API stable.
- [ ] Les erreurs de règle sont compréhensibles et localisées.
- [ ] Les contrôles essentiels sont accessibles au clavier et aux technologies d'assistance.
- [ ] Aucun état critique de chargement, erreur ou absence de données n'est omis.

---

## Phase 6 — Intégration

**Objectif :** connecter données, API, moteur, IA et interface en un système cohérent.

**Dépendances :** phases 1 à 5 suffisamment stables.

**Résultat attendu :** parcours complet fonctionnel dans un environnement proche de la production.

### Epic 6.1 — Parcours bout en bout

- [ ] Connecter recherche, format et banlist aux données réelles.
- [ ] Connecter éditeur et validation temps réel au moteur.
- [ ] Connecter génération algorithmique et affichage des raisons.
- [ ] Connecter explications IA et fallback sans IA.
- [ ] Connecter analyse, optimisation et application contrôlée des suggestions.
- [ ] Connecter import/export et vérifier les allers-retours sans perte.

### Epic 6.2 — Cohérence et résilience

- [ ] Propager identifiants de corrélation et versions à travers tous les composants.
- [ ] Gérer timeouts, annulations, reprises et indisponibilités partielles.
- [ ] Empêcher un changement de banlist silencieux pendant une session.
- [ ] Tester les migrations de base sur un snapshot réaliste.
- [ ] Vérifier que l'application reste utile sans service IA.
- [ ] Mesurer le parcours complet et identifier les goulets d'étranglement.

### Critères de sortie de la phase 6

- [ ] Un utilisateur réalise le parcours MVP complet sur l'environnement d'intégration.
- [ ] Les résultats affichent leurs versions de données et de règles.
- [ ] Les pannes externes ont un comportement maîtrisé et explicite.
- [ ] Les contrats frontend/backend ne présentent aucune divergence connue.

---

## Phase 7 — Tests et qualité

**Objectif :** démontrer la légalité, la robustesse, la sécurité et l'utilisabilité du produit.

**Dépendances :** parcours intégré et critères du MVP stabilisés.

**Résultat attendu :** preuves automatisées et manuelles suffisantes pour autoriser une bêta.

### Epic 7.1 — Stratégie de tests

- [ ] Définir pyramide de tests, responsabilités, seuils et politique de non-régression.
- [ ] Couvrir unités de domaine, propriétés invariantes, contrats, intégration et E2E.
- [ ] Versionner jeux de tests, golden decks et résultats attendus.
- [ ] Ajouter une matrice formats/zones/statuts de banlist/cas limites.

### Epic 7.2 — Règles et Deck Engine

- [ ] Tester limites inférieures et supérieures de chaque zone.
- [ ] Tester copies cumulées entre zones et tous les statuts de banlist.
- [ ] Tester cartes inconnues, données manquantes et banlist inexistante.
- [ ] Ajouter tests génératifs garantissant les invariants de légalité.
- [ ] Tester déterminisme, reproductibilité et stabilité du scoring.
- [ ] Comparer les sorties à des golden decks revus par un humain.
- [ ] Tester que toute optimisation finale demeure légale.

### Epic 7.3 — IA

- [ ] Tester sorties invalides, cartes inventées et contradictions avec les faits fournis.
- [ ] Tester injections de prompt dans demandes et textes externes.
- [ ] Tester fallback, timeout, quota et indisponibilité fournisseur.
- [ ] Exécuter l'évaluation versionnée à chaque modification de prompt ou modèle.
- [ ] Réaliser une revue humaine des explications et signaler l'incertitude.

### Epic 7.4 — Système, performance et sécurité

- [ ] Automatiser E2E des parcours construire, modifier, valider, analyser et exporter.
- [ ] Tester concurrence, charge, latence p95 et consommation mémoire.
- [ ] Tester import volumineux, reprise après panne et rollback de snapshot.
- [ ] Vérifier accessibilité avec outils automatiques et parcours clavier manuel.
- [ ] Auditer dépendances, secrets, permissions, en-têtes et journalisation.
- [ ] Tester sauvegarde/restauration de la base et procédure d'incident.

### Epic 7.5 — CI et qualité continue

- [ ] Exécuter lint, format, typage et tests sur chaque pull request.
- [ ] Bloquer la fusion en cas d'échec critique ou migration non vérifiée.
- [ ] Publier rapports de couverture, sécurité et évaluation IA.
- [ ] Construire des artefacts immuables et traçables.

### Critères de sortie de la phase 7

- [ ] Tous les contrôles obligatoires CI passent sur une version candidate.
- [ ] Aucun défaut critique ouvert sur légalité, sécurité ou perte de données.
- [ ] Les objectifs validés de latence, accessibilité et qualité IA sont atteints.
- [ ] Les scénarios de restauration et de panne externe sont vérifiés.

---

## Phase 8 — MVP

**Objectif :** valider la valeur produit auprès d'utilisateurs réels avant généralisation.

**Dépendances :** version candidate qualifiée.

**Résultat attendu :** bêta limitée mesurée, retours priorisés et décision de lancement.

### Epic 8.1 — Préparation bêta

- [ ] Vérifier chaque user story et critère d'acceptation MVP.
- [ ] Préparer onboarding, aide, limites connues et canal de feedback.
- [ ] Définir cohortes de test et consentement de télémétrie.
- [ ] Instrumenter événements produit sans collecter de données superflues.
- [ ] Préparer support, triage et procédure d'arrêt en cas de résultat dangereux.

### Epic 8.2 — Bêta fermée

- [ ] Recruter un panel représentatif débutant/intermédiaire/confirmé.
- [ ] Observer l'achèvement du parcours sans assistance.
- [ ] Mesurer deck valide obtenu, temps, abandons, erreurs et satisfaction.
- [ ] Faire évaluer un échantillon de decks et explications par des joueurs expérimentés.
- [ ] Classer les retours par fréquence, sévérité et valeur.
- [ ] Corriger les défauts bloquants et réexécuter les tests de non-régression.

### Epic 8.3 — Go/No-Go MVP

- [ ] Comparer les résultats aux indicateurs de succès de la phase 0.
- [ ] Documenter limites connues, dette acceptée et risques résiduels.
- [ ] Valider capacité, coûts projetés et support du lancement.
- [ ] Prendre et consigner la décision Go/No-Go.

### Critères de sortie de la phase 8

- [ ] Les seuils produit et qualité convenus sont atteints ou une exception est acceptée explicitement.
- [ ] Aucun blocage critique de bêta ne reste ouvert.
- [ ] Le périmètre exact de la première version publique est figé.

---

## Phase 9 — Déploiement

**Objectif :** mettre le MVP en production de manière sûre, observable et réversible.

**Dépendances :** décision Go et infrastructure validée.

**Résultat attendu :** service public surveillé avec opérations documentées.

### Epic 9.1 — Infrastructure

- [ ] Déclarer les environnements développement, staging et production.
- [ ] Provisionner application, base, stockage, réseau et gestion des secrets.
- [ ] Configurer domaine, TLS, sauvegardes chiffrées et rétention.
- [ ] Séparer accès et données entre environnements.
- [ ] Tester restauration et rotation des secrets.

### Epic 9.2 — Livraison continue

- [ ] Construire et signer/versionner les artefacts de déploiement.
- [ ] Automatiser migrations avec vérification et stratégie de rollback.
- [ ] Déployer automatiquement en staging après validation CI.
- [ ] Exiger une approbation et des contrôles avant production.
- [ ] Mettre en place déploiement progressif et retour arrière vérifié.

### Epic 9.3 — Observabilité et exploitation

- [ ] Définir SLI/SLO pour disponibilité, latence, erreurs, imports et IA.
- [ ] Créer tableaux de bord et alertes actionnables.
- [ ] Suivre dépenses d'hébergement, base, bande passante et LLM avec budgets.
- [ ] Rédiger runbooks pour panne, données obsolètes, banlist incorrecte et dépassement de coût.
- [ ] Définir astreinte ou responsabilité de réponse adaptée à l'échelle du MVP.
- [ ] Publier politique de confidentialité, conditions et attributions nécessaires.

### Epic 9.4 — Lancement

- [ ] Exécuter smoke tests en production.
- [ ] Vérifier fraîcheur des données et banlist affichée.
- [ ] Ouvrir progressivement l'accès et surveiller les indicateurs.
- [ ] Recueillir incidents et feedback dans un backlog unique.
- [ ] Réaliser une rétrospective et mettre à jour la roadmap.

### Critères de sortie de la phase 9

- [ ] Production disponible, sécurisée, sauvegardée et observable.
- [ ] Un rollback applicatif et une restauration de données ont été testés.
- [ ] Coûts et erreurs restent dans les seuils convenus après lancement.
- [ ] Responsabilités et procédures d'exploitation sont documentées.

---

## Phase 10 — Fonctionnalités avancées

**Objectif :** étendre le produit selon l'usage mesuré, sans dégrader la fiabilité du noyau.

**Dépendances :** MVP lancé et apprentissages utilisateurs analysés.

**Résultat attendu :** incréments indépendants, chacun avec métrique, données et critères propres.

### Epic 10.1 — Formats et temporalité

- [ ] Ajouter TCG, OCG et Master Duel via adaptateurs de règles distincts.
- [ ] Ajouter formats historiques avec pool et banlist datés.
- [ ] Permettre de reproduire une analyse avec un snapshot passé.
- [ ] Ajouter jeux et variantes Yu-Gi-Oh! anciens uniquement après étude de modèle.

### Epic 10.2 — Personnalisation du deckbuilding

- [ ] Ajouter profils stabilité, combo, going first, going second et budget.
- [ ] Ajouter contraintes de collection et de prix avec provenance temporelle.
- [ ] Générer plusieurs variantes classées et comparables.
- [ ] Ajouter simulation de mains d'ouverture et probabilités de combinaisons.
- [ ] Ajouter recommandations personnalisées avec contrôles de confidentialité.

### Epic 10.3 — Matchups, Side Deck et métagame

- [ ] Modéliser matchup, menace, réponse et plan de side.
- [ ] Intégrer données de tournois autorisées, normalisées et dédupliquées.
- [ ] Corriger biais de sélection, fraîcheur et tailles d'échantillons dans les affichages.
- [ ] Générer un Side Deck et des plans entrée/sortie toujours revalidés.
- [ ] Afficher tendances du métagame avec période, région et source.

### Epic 10.4 — Comptes et collaboration

- [ ] Ajouter authentification et gestion de session sécurisées.
- [ ] Sauvegarder decks, préférences et versions.
- [ ] Ajouter historique, comparaison et restauration de version.
- [ ] Ajouter partage public/privé, duplication et permissions.
- [ ] Ajouter commentaires, notation et modération seulement avec capacité opérationnelle dédiée.
- [ ] Ajouter export/suppression des données personnelles.

### Epic 10.5 — Import/export élargi

- [ ] Étudier les formats de decklists et intégrations autorisées.
- [ ] Ajouter adaptateurs import/export avec rapport de pertes ou incompatibilités.
- [ ] Détecter automatiquement format et version quand cela est fiable.
- [ ] Conserver identifiants canoniques lors des conversions.

### Epic 10.6 — Simulation avancée

- [ ] Définir la frontière entre probabilités, solveur de lignes et moteur de duel.
- [ ] Étudier faisabilité, droits, coût et niveau de fidélité requis.
- [ ] Prototyper sur un sous-ensemble de règles isolé du validateur de production.
- [ ] N'intégrer le simulateur qu'après validation de ses invariants et limites affichées.

### Critères de sortie de la phase 10

- [ ] Chaque extension lancée a une hypothèse produit, une métrique et des tests dédiés.
- [ ] Les nouveaux formats n'introduisent pas de régression sur les règles existantes.
- [ ] Les nouvelles sources respectent licences, provenance et qualité attendue.

---

## 9. Risques et mitigations

| Risque | Impact | Mitigation prévue |
|---|---|---|
| Données incomplètes ou obsolètes | Decks faux ou incomplets | Snapshots, provenance, contrôles d'intégrité, date visible et stratégie de repli |
| Banlist incorrecte ou changée | Deck illégal | Versions immuables, source vérifiée, activation explicite et tests de cartes sentinelles |
| Complexité des règles/rulings | Faux sentiment de validité | Limiter la promesse MVP aux règles formalisées, publier les limites, corpus de régression |
| Hallucinations du LLM | Cartes/effets inventés | IDs locaux uniquement, JSON strict, ancrage dans les faits et revalidation déterministe |
| Mauvaise qualité stratégique | Faible confiance | Ontologie versionnée, golden decks, revue humaine et feedback mesuré |
| Coûts et latence IA | Produit lent ou non viable | Modèle adapté par tâche, cache, budgets, quotas, métriques et mode sans IA |
| Dépendance fournisseur | Rupture ou hausse de prix | Interfaces d'adaptation pour données, IA et hébergement ; snapshots locaux |
| Droits/licences/marques | Blocage de diffusion | Revue avant import/publication, attributions, minimisation et sources autorisées |
| Performance combinatoire | Génération trop lente | Pool réduit, heuristiques, budgets de calcul, profiling puis solveur si nécessaire |
| Sécurité et prompt injection | Abus, coût ou fuite | Entrées bornées, séparation système/données, secrets isolés, quotas et tests adversariaux |
| Complexité excessive | Retard du MVP | Un format, monolithe modulaire, critères de sortie et backlog « plus tard » explicite |
| Faible adoption | Effort sans valeur | Prototype, bêta précoce, métriques et décisions Go/No-Go |
| Maintenance insuffisante | Dégradation silencieuse | Ownership, alertes de fraîcheur, runbooks et budget de maintenance |

---

## 10. Registre des décisions à prendre avant le code

Les décisions ci-dessous sont **ouvertes**. La colonne « recommandation » décrit la proposition d'architecture, pas une validation. Après accord explicite, cocher la décision et la consigner dans un ADR ; ne jamais la retirer du registre.

### 🔴 Bloquantes

| État | ID et question | Options | Recommandation | Raison |
|---|---|---|---|---|
| [ ] | **D-001 — Quel format MVP ?** | TCG actuel ; Master Duel ; multi-format ; architecture multi-format/un format | Option D, TCG Advanced EMEA uniquement | Source officielle publique, Side Deck pertinent et voie d'extension sans multiplier le MVP |
| [ ] | **D-002 — Quelle banlist et quelle temporalité ?** | Toujours « latest » ; version figée ; choix utilisateur | Snapshot officiel KONAMI applicable à la génération, affiché et conservé | Reproductibilité et absence de changement silencieux |
| [ ] | **D-003 — Quelle source de catalogue ?** | YGOPRODeck ; source officielle automatisée ; dataset tiers | YGOPRODeck API v7 en snapshot local, avec contrôle ciblé KONAMI | API documentée et multilingue ; découplage par adaptateur indispensable |
| [ ] | **D-004 — Quelles langues ?** | Anglais seul ; français seul ; canonique anglais + UI française | IDs numériques + anglais canonique, interface française, noms FR lorsqu'ils sont disponibles | Robustesse des identifiants et expérience du public initial |
| [ ] | **D-005 — Quel périmètre fonctionnel exact ?** | MVP minimal de validation ; génération complète proposée ; comptes inclus | Contrat de la section 6.1, sans compte ni optimisation automatique du Side Deck | Démontrer la valeur bout en bout sans infrastructure de compte ou données de matchup |
| [ ] | **D-006 — Quelle architecture/stack ?** | TypeScript complet ; Python complet ; Next.js + FastAPI/Python | Monolithe modulaire Next.js/TypeScript + FastAPI/Python + PostgreSQL | UI typée et moteur/IA Python, avec limites de modules explicites |
| [ ] | **D-007 — Quel modèle conceptuel ?** | Deck mutable ; versions immuables ; schéma minimal sans provenance | Modèle section 6.5 avec `DeckVersion`, snapshots et provenance | Reproductibilité des analyses et évolution des banlists |
| [ ] | **D-008 — Quelles sources/licences sont acceptables ?** | Images distantes ; auto-hébergement ; aucune image au départ | Valider juridiquement API, textes, images et marques avant import ; démarrer sans images si nécessaire | Un doute de licence peut bloquer la diffusion, contrairement à une absence temporaire d'illustrations |

### 🟠 Importantes

| État | ID et question | Options | Recommandation | Raison |
|---|---|---|---|---|
| [ ] | **D-009 — Quels archétypes pilotes ?** | Un seul ; cinq représentatifs ; catalogue entier non annoté | Cinq couvrant Fusion/Synchro/Xyz/Link et complexités différentes, choisis avec un expert | Jeu d'évaluation utile sans annoter tout le catalogue |
| [ ] | **D-010 — Comment définir la qualité d'un deck ?** | Avis expert ; score heuristique ; résultats tournoi | Légalité obligatoire + grille de rôles + golden decks revus, sans prétendre mesurer la puissance absolue | Critère testable malgré l'absence de simulateur et de données compétitives fiables |
| [ ] | **D-011 — Quel LLM et quel budget ?** | Fournisseur unique ; multi-fournisseur ; local | Port fournisseur, benchmark de modèles compatibles JSON, plafond mensuel et coût par génération | Choix fondé sur qualité/coût/latence réels, pas sur la popularité |
| [ ] | **D-012 — RAG et embeddings ?** | Dès le MVP ; PostgreSQL/FTS ; pgvector plus tard | Aucun RAG/vector DB au MVP | Les données utiles sont structurées ; une nouvelle infrastructure n'est justifiée que par un corpus documentaire évalué |
| [ ] | **D-013 — Quel format d'import/export ?** | YDK ; JSON interne ; CSV ; formats tiers | YDK utilisateur + JSON interne versionné | Compatibilité pratique et contrat interne sans perte |
| [ ] | **D-014 — Quels objectifs non fonctionnels ?** | Best effort ; SLO chiffrés | p95 moteur < 3 s, IA < 15 s, seuils coût/erreur à fixer | Rend les arbitrages et le Go/No-Go mesurables |
| [ ] | **D-015 — Quelle politique de mise à jour des données ?** | Temps réel ; quotidien ; manuel | Détection quotidienne, import contrôlé, publication après validation | Réactivité raisonnable sans exposer automatiquement une source cassée |
| [ ] | **D-016 — Quelle confidentialité/télémétrie ?** | Aucune ; strict minimum ; analytics complet | Événements produit minimaux avec consentement et aucune demande brute conservée par défaut | Mesurer le MVP tout en minimisant les données personnelles |

### 🟢 Secondaires

| État | ID et question | Options | Recommandation | Raison |
|---|---|---|---|---|
| [ ] | **D-017 — Quel hébergement final ?** | Vercel + PaaS API ; Cloud Run ; PaaS unique | Mesurer localement puis comparer coût, région et simplicité avant Phase 9 | N'empêche pas le domaine et l'API de démarrer |
| [ ] | **D-018 — Quel état frontend ?** | React seul ; Zustand ; Redux | TanStack Query + état local, Zustand uniquement si complexité observée | Réduire la surface technique initiale |
| [ ] | **D-019 — Redis/worker dès le départ ?** | Oui ; non ; service cloud | Non, ajouter après mesure des imports et latences | PostgreSQL et commandes contrôlées suffisent au premier incrément |
| [ ] | **D-020 — Quel outil de typage Python ?** | mypy ; pyright | Petit spike puis un seul outil en CI | Évite les configurations concurrentes ; impact limité sur l'architecture |
| [ ] | **D-021 — Quelle solution d'observabilité ?** | Sentry ; fournisseur cloud ; stack OpenTelemetry | Instrumentation OpenTelemetry, backend choisi avant staging | Préserver la portabilité sans retarder le domaine |
| [ ] | **D-022 — Comptes après MVP ?** | Auth interne ; OAuth/OIDC ; service managé | OIDC/service managé à évaluer seulement lorsque la sauvegarde cloud est priorisée | La persistance locale suffit à la validation initiale |

### Synthèse des validations attendues

- [ ] Le porteur du projet approuve ou amende D-001 à D-008.
- [ ] Les décisions approuvées sont transformées en ADR datés.
- [ ] Les conséquences des amendements sont propagées dans le modèle, les phases et les risques.
- [ ] Les décisions D-009 à D-016 ont un propriétaire et une échéance antérieure à leur première implémentation.

---

## 11. Première tâche concrète

### À réaliser ensuite : valider le format et la banlist du MVP

- [x] Comparer les stratégies TCG, Master Duel, multi-format immédiat et architecture multi-format progressive.
- [ ] Choisir un format unique pour la première livraison.
- [ ] Identifier sa source de banlist autoritative et une version de départ datée.
- [x] Documenter la proposition de tailles de zones, limites de copies et cas spéciaux pris en charge.
- [ ] Faire approuver la décision et la consigner dans l'ADR **D-001/D-002**.

**Critère d'acceptation :** un document de décision nomme sans ambiguïté le format, la région, la date/version de banlist, la source, les règles de taille et les limites que le MVP promet de valider.

---

## 12. Journal de progression

| Date | Changement | État |
|---|---|---|
| 2026-09-29 | Analyse initiale du cahier des charges et constat d'un dépôt sans implémentation | Terminé |
| 2026-09-29 | Création de la roadmap initiale, du périmètre MVP et de l'architecture proposée | Terminé |
| 2026-09-29 | Ouverture de la décision sur le format et la banlist du MVP | En cours |
| 2026-09-29 | Rédaction de la spécification candidate de Phase 0 : MVP, formats, pipeline, LLM, modèle, sources, stack et architecture | Terminé |
| 2026-09-29 | Classement des décisions D-001 à D-022 ; D-001 à D-008 soumises à validation | En attente de décision |

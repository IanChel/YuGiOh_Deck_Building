# ADR-0017 — Provenance et publication des données

## Status

Accepted

## Context

Le catalogue et les snapshots réglementaires doivent pouvoir expliquer chaque valeur publiée depuis sa source externe jusqu'au modèle métier. Confondre source, capture, fait importé, publication validée et snapshot métier empêcherait l'audit, la résolution des conflits et la reproductibilité.

## Decision

- représenter chaque origine durable par `DataSource`, avec rôles et autorité versionnée par domaine ;
- représenter chaque capture déterminée par un `DataSnapshot` immuable, même lorsque son contenu répète une capture antérieure ;
- représenter ce que la source affirme par des `ImportedFact` non canoniques conservant valeur brute, emplacement et version d'extraction ;
- maintenir mappings, transformations et résolutions comme étapes versionnées qui n'effacent jamais les faits sources ;
- placer les valeurs non mappées, ambiguës, invalides ou conflictuelles en quarantaine ;
- représenter une publication validée par `PublishedDataset`, distincte de `CatalogueSnapshot` et des autres snapshots métier ;
- autoriser plusieurs sources par dataset tout en conservant une lignée par valeur ;
- résoudre les conflits par politique d'autorité versionnée ou revue humaine, sans fusion silencieuse ni « dernière valeur gagne » ;
- publier uniquement un périmètre déclaré sans anomalie bloquante non résolue ;
- versionner et épingler les mappings afin qu'une modification produise une nouvelle publication ;
- maintenir les suggestions LLM hors des `ImportedFact` et des publications canoniques jusqu'à la revue D-010 ;
- distinguer provenance et autorisation juridique, D-008 restant ouverte.

La spécification complète et les décisions validées sont décrites dans [`docs/specifications/d017-data-provenance-and-publication.md`](../specifications/d017-data-provenance-and-publication.md).

## Consequences

- Chaque valeur publiée peut être expliquée depuis ses captures et faits sources.
- Une nouvelle récupération, un nouveau mapping ou un nouvel arbitrage ne réécrit pas l'historique.
- Plusieurs sources peuvent contribuer sans perdre leurs autorités respectives.
- Une capture partielle ou une référence non résolue ne peut pas se masquer dans un dataset prétendument complet.
- Les snapshots métier restent indépendants du cycle d'acquisition externe.
- Le système devra conserver des manifestes de provenance et rapports d'intégrité proportionnés au périmètre publié.

## Alternatives considered

- Utiliser directement YGOPRODeck comme modèle canonique : écarté car le domaine serait couplé au fournisseur et la provenance perdue.
- Confondre `DataSnapshot` et `CatalogueSnapshot` : écarté car capture externe et état métier ont des cycles différents.
- Écraser les valeurs brutes après normalisation : écarté car les mappings deviendraient inexplicables.
- Employer la dernière source reçue en cas de conflit : écarté car l'autorité dépend du domaine.
- Publier malgré des anomalies en réduisant silencieusement la couverture : écarté car trompeur pour les consommateurs.
- Traiter une sortie LLM comme un fait importé : écarté car le modèle n'est pas une source normative.
- Déduire les droits de redistribution de l'autorité officielle : écarté car provenance et licence sont distinctes.


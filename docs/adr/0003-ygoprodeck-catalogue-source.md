# ADR-0003 — YGOPRODeck comme source opérationnelle du catalogue

## Status

Accepted

## Context

Le MVP a besoin d'un catalogue structuré de cartes, consultable sans dépendre d'un appel externe pour chaque requête. Les règles, banlists et décisions de légalité doivent rester fondées sur des références officielles.

## Decision

YGOPRODeck est la source opérationnelle principale du catalogue MVP. La cible de production est le catalogue général le plus complet que la source et le périmètre légal TCG Advanced EMEA permettent, et non un sous-catalogue limité à des archétypes présélectionnés. Les données nécessaires sont importées dans PostgreSQL sous forme de snapshots locaux, avec identifiants externes, provenance, version et rapport d'import.

Les sources KONAMI restent la référence pour les banlists, règles, légalité et informations officielles pertinentes. Le Rules Engine n'est pas couplé au modèle ou aux valeurs de YGOPRODeck ; il consomme le modèle canonique interne.

## Consequences

- L'application continue de consulter son catalogue si l'API externe est indisponible.
- L'ajout d'une carte ou d'un archétype au catalogue ne requiert aucune modification d'architecture.
- Les jeux de données réduits utilisés en développement sont des fixtures et ne définissent pas la couverture fonctionnelle de production.
- Un adaptateur traduit YGOPRODeck vers le schéma canonique.
- Les dérives de schéma et différences avec les sources officielles doivent être détectées.
- Les conditions d'utilisation des données et images restent soumises à ADR-0008.

## Alternatives considered

- Interroger YGOPRODeck à chaque requête : rejeté pour disponibilité, débit et reproductibilité.
- Scraper une base officielle : rejeté faute d'API publique validée et de conditions clarifiées.
- Construire manuellement le catalogue : non maintenable.


# ADR-0015 — Format, snapshots et périmètre TCG Advanced EMEA

## Status

Accepted

## Context

Le MVP doit déterminer ultérieurement la légalité d'une carte dans TCG Advanced EMEA sans placer un booléen global sur `Card`. Le format, la région, le catalogue, la disponibilité et la banlist évoluent selon des temporalités différentes et doivent rester reproductibles.

## Decision

- représenter `TCG_ADVANCED` par une identité `Format` stable, indépendante de toute région et banlist ;
- représenter l'état temporel du cadre par un `FormatSnapshot` immuable avec dates d'effet, publication, provenance et éventuelle supersession ;
- représenter EMEA par une identité contrôlée minimale `RegionalScope`, distincte de `Format` et sans taxonomie mondiale au MVP ;
- composer le contexte d'analyse avec des références séparées vers `FormatSnapshot`, `RegionalScope`, `CatalogueSnapshot` et le futur `BanlistSnapshot` ;
- maintenir `CardAvailability` dans le catalogue versionné et ne jamais assimiler présence au catalogue, disponibilité et légalité ;
- distinguer `AVAILABLE`, `UNAVAILABLE` et `UNKNOWN`, ce dernier n'autorisant aucune conclusion positive implicite ;
- distinguer `published_at`, `effective_from`, `effective_until` et `analyzed_at` ;
- utiliser conceptuellement des intervalles d'effet `[effective_from, effective_until)` ;
- conserver tout ancien résultat sur ses références originales après une transition ou correction ;
- ne définir aucune logique de validation de deck dans D-015.

La spécification complète et les décisions validées sont décrites dans [`docs/specifications/d015-format-and-emea-scope.md`](../specifications/d015-format-and-emea-scope.md).

## Consequences

- Une nouvelle banlist ou disponibilité ne change pas l'identité du format.
- Un même format peut être combiné ultérieurement à un autre périmètre régional.
- La légalité devient une conclusion contextuelle et non une propriété de carte.
- Une analyse historique conserve exactement les snapshots et la région utilisés.
- Une correction n'écrase pas les résultats antérieurs et doit être explicitement reliée à la version concernée.
- Les consommateurs devront fournir ou résoudre un contexte complet avant toute validation positive.

## Alternatives considered

- Stocker `is_legal` sur `Card` : écarté car la valeur dépend du temps, de la région, du format et de la banlist.
- Créer un format `TCG_ADVANCED_EMEA` : écarté car il fusionnerait format et région.
- Considérer toute carte YGOPRODeck comme disponible en EMEA : écarté faute de preuve régionale.
- Fusionner banlist et `FormatSnapshot` : écarté car leurs identités et cycles de publication sont distincts.
- Résoudre un ancien résultat avec les derniers snapshots : écarté car non reproductible.
- Interpréter `UNKNOWN` comme disponible : écarté car l'absence de preuve ne vaut pas autorisation.


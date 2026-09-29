# ADR-0014 — Appartenance structurelle des cartes aux archétypes

## Status

Proposed

## Context

Le catalogue doit distinguer l'appartenance structurelle d'une carte à un archétype de son support, de sa synergie, de sa compatibilité et des simples mentions textuelles. Confondre ces notions rendrait les filtres inexacts et inciterait le futur moteur de recommandation à traiter l'appartenance comme un score de pertinence.

## Proposed decision

- représenter un archétype par une identité interne stable distincte de ses noms et des identifiants fournisseurs ;
- prévoir conceptuellement `ArchetypeLocalization` et `ArchetypeNameAlias`, sans réutiliser `CardLocalization` ;
- limiter `CardArchetypeMembership` aux assertions positives d'appartenance structurelle ;
- ne pas réintroduire de distinction `MEMBER` / `EXPLICITLY_LISTED_SUPPORT` ;
- autoriser zéro, une ou plusieurs appartenances par carte, sans archétype principal ;
- exiger un fondement traçable : donnée officielle, mapping de catalogue validé, règle de nommage normative versionnée ou correction humaine revue ;
- maintenir les mentions textuelles, supports, synergies et compatibilités hors de cette relation et dans les mécanismes D-010 appropriés ;
- rattacher chaque appartenance à un `CatalogueSnapshot` sans imposer de stockage physique ;
- mapper explicitement et versionner les valeurs YGOPRODeck vers les identités internes ;
- placer toute valeur externe inconnue, ambiguë ou non mappée en quarantaine, sans faux archétype `UNKNOWN` ;
- réutiliser les concepts de provenance et de revue D-010 ; aucune suggestion LLM non revue ne devient canonique.

La proposition complète et ses questions de validation sont décrites dans [`docs/specifications/d014-card-archetype-membership.md`](../specifications/d014-card-archetype-membership.md).

## Consequences

- Les filtres « appartient à » restent stricts et reproductibles.
- Une carte générique ou de support peut être recommandée sans devenir membre.
- Les cartes multi-archétypes sont représentées par plusieurs assertions indépendantes.
- Les noms anglais, français et anciens noms résolvent vers la même identité d'archétype.
- Les corrections n'altèrent pas les anciens snapshots.
- Les nouveautés externes doivent être mappées ou revues avant publication.

## Alternatives considered

- Déduire l'appartenance de toute mention textuelle : écarté, car une carte peut citer ou soutenir un archétype sans lui appartenir.
- Utiliser directement la chaîne YGOPRODeck comme identité : écarté, car nomenclature et fournisseur peuvent évoluer.
- Limiter une carte à un archétype : écarté, car les appartenances multiples doivent être représentables.
- Ajouter un type `EXPLICITLY_LISTED_SUPPORT` : écarté conformément à D-011 ; le support relève de D-010.
- Créer automatiquement un archétype inconnu : écarté, car une donnée externe non revue ne doit pas créer une identité canonique.
- Transformer l'appartenance en poids de recommandation : écarté, car appartenance et pertinence stratégique répondent à des questions différentes.


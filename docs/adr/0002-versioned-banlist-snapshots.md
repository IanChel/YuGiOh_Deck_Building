# ADR-0002 — Snapshots de banlist datés et versionnés

## Status

Accepted

## Context

La banlist TCG évolue. Utiliser silencieusement la liste la plus récente rendrait impossible la reproduction d'une génération passée et pourrait changer rétroactivement le statut d'un deck.

## Decision

Chaque génération et validation utilise un snapshot immuable d'une banlist officielle TCG EMEA. Le résultat conserve au minimum le format, l'identifiant/version de banlist et la date du snapshot.

L'activation d'une nouvelle banlist publie un nouveau snapshot ; elle ne modifie jamais un snapshot historique.

## Consequences

- Les résultats passés restent explicables et reproductibles.
- Les banlists ont une date d'effet, une provenance et un état de publication.
- Le moteur reçoit toujours la version à appliquer ; il ne recherche pas implicitement « latest ».
- La synchronisation doit détecter, valider puis publier une nouvelle version.

## Alternatives considered

- Toujours appliquer la banlist courante : rejeté car non reproductible.
- Écraser la version précédente : rejeté car destructif.
- Laisser l'utilisateur choisir toute version au MVP : différé, l'interface n'en expose qu'une active.


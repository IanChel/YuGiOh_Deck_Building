# ADR-0008 — Utilisation juridique des données, images et marques

## Status

Proposed

## Context

Le projet envisage d'utiliser des données et potentiellement des textes, images et marques liés à Yu-Gi-Oh!, ainsi que des données obtenues via YGOPRODeck. Les conditions peuvent différer selon une utilisation locale, publique ou commerciale.

## Decision

Aucune conclusion juridique n'est adoptée à ce stade. Avant publication ou distribution, le projet doit faire vérifier et documenter :

- les conditions d'utilisation de YGOPRODeck et de ses endpoints ;
- les droits de reproduction et de stockage des données, textes et images ;
- l'utilisation des noms, logos et marques Yu-Gi-Oh!/KONAMI ;
- les obligations d'attribution et de retrait ;
- les différences entre prototype local, service public et usage commercial ;
- la provenance et la licence de toute donnée secondaire.

Dans l'intervalle, le développement local doit minimiser les données, conserver leur provenance et permettre de désactiver les images ou contenus non autorisés.

## Consequences

- D-008 reste ouverte et ne peut pas être présentée comme un avis juridique.
- La publication/distribution est bloquée tant que les droits nécessaires ne sont pas clarifiés.
- Le développement local des modèles, contrats et moteurs peut commencer avec des fixtures minimales compatibles avec les droits applicables.
- L'architecture doit permettre le remplacement d'une source et le retrait de contenus.

## Alternatives considered

- Considérer toute donnée publique comme libre : rejeté.
- Attendre la clarification avant toute conception locale : disproportionné.
- Publier sans images : option de réduction du risque, mais ne règle pas tous les droits sur textes, données et marques.

# Concept de jeu — Projet X

**Statut** : 🟡 Concept V1 fixé (cœur de boucle et périmètre validés), détails de mécaniques encore ouverts
**Date d'origine** : 2026-07-20
**Titre de travail** : à définir (piste de travail : jeu du "Nécromancien")

---

## ⚠️ Révision (2026-07-20, même session) — pivot de concept

La première piste explorée ("Le Chasseur Inversé" — essaim contrôlé indirectement, formes géométriques abstraites) est **abandonnée**, remplacée par une vision beaucoup plus concrète apportée directement par Arnaud. Gardée en historique ci-dessous pour mémoire, pas supprimée.

<details>
<summary>Ancienne piste abandonnée : "Le Chasseur Inversé"</summary>

Jeu de survie infinie, contrôle indirect par points d'attraction, essaim qui grandit par capture (boule de neige), défaite par perte d'unités. Abandonné au profit de la vision du Nécromancien, jugée plus motivante par Arnaud — la motivation à finir un projet solo compte plus qu'un concept "sur le papier" plus original.
</details>

---

## Origine de la démarche

TLR mis en pause le 2026-07-20 pour se former sur un premier jeu simple (voir `Desktop/The Long Ride/PROJECT/docs/reprise_actuel.md`). Contraintes posées par Arnaud avant le brainstorming : facile à faire, 2-3 mois, doit prouver des compétences de game design général, original si possible, peu d'assets si possible.

## Vision complète (apportée par Arnaud, 2026-07-20)

Le joueur incarne un **nécromancien démoniaque** qui veut anéantir les héros d'un village. Univers large mais borné, déplacement libre (clavier/manette, 8 directions ou plus). Le jeu est structuré en niveaux représentant la progression du nécromancien dans le village, jusqu'à la réunion finale des héros.

- **Invocations** : au départ quelques invocations basiques et peu puissantes, contrôlées en RTS-lite (sélection d'un groupe, clic sur un groupe d'ennemis pour l'envoyer au combat).
- **Orbes** : récoltées en tuant des PNJ avec les invocations, servent de monnaie de méta-progression (améliorations entre les parties).
- **Bonus aléatoires** : gagnés en cours de partie, augmentent temporairement attaque/vitesse etc. des invocations (durée = la partie).
- **Déblocage d'invocations avancées** : plus stratégiques, pensées autour d'un système de contre-types façon pierre-feuille-ciseaux face à certains types d'ennemis.
- **Mana** : régénération passive continue, + gain en tuant un certain nombre de mobs ou via un consommable trouvé. Sert à invoquer.
- **Défaite** : le nécromancien meurt s'il subit trop d'attaques — seul game over.
- **Ennemis** : PNJ des premiers niveaux peu dangereux, pas tous agressifs ; montée en dangerosité et en exigence stratégique avec la progression.
- **Monde procédural** : généré à chaque partie, avec un style propre par zone : village large (blocs maison) → centre pavé (moins d'espace) → pied du château (beaucoup d'ennemis puissants, moins d'espace, obstacles défensifs) → couloirs du château (exigus) → sommet du château (à nouveau large, reprend les mécaniques précédentes + ennemis type boss).
- **Post-complétion** : possibilité d'ajuster le niveau de difficulté après une première victoire.

Cette vision complète devient la **référence long terme** du projet — pas la cible de la V1.

## Périmètre V1 (2-3 mois — validé par Arnaud, 2026-07-20)

Garde le cœur de la fantaisie, coupe les systèmes les plus coûteux :

- Nécromancien jouable, déplacement libre au clavier (manette reportée). Meurt s'il subit trop d'attaques = seul game over.
- 2-3 invocations de base seulement, rôles simples et distincts (ex. une tanky, une rapide) — **pas** de matrice de contre-types en V1.
- Contrôle RTS-lite : sélection de groupe + clic sur une cible ennemie pour envoyer les invocations.
- Mana : régénération passive + dépense à l'invocation (le gain au kill/consommable peut rester si simple à ajouter, sinon reporté).
- Orbes ramassées sur PNJ tués = petite monnaie de méta-progression très limitée (1-2 améliorations permanentes achetables entre les parties, pas un arbre complet).
- **3 à 5 niveaux construits à la main** (pas de génération procédurale) qui suivent la même progression narrative : abords du village → centre pavé → pied du château → couloirs → sommet, avec un ennemi/boss final sur le dernier niveau.
- Ennemis : villageois faibles peu agressifs au début, héros plus dangereux vers la fin — montée en puissance simple, pas de système de contre-types.

**Note assets** : ce concept est plus gourmand en assets graphiques que l'ancienne piste abstraite (nécromancien, invocations, villageois, héros, décors village/château) — plus proche du pipeline pixel art déjà pratiqué sur TLR que d'un style géométrique minimal. À budgéter consciemment (peu de types, variantes par recolor plutôt que sprites uniques à chaque fois).

## Rêve / V2 (après la V1, si le temps et la motivation le permettent)

- Génération procédurale du monde (remplace les niveaux à la main) — le système le plus coûteux à lui seul.
- Contre-types façon pierre-feuille-ciseaux entre invocations et types d'ennemis.
- Méta-progression complète (arbre de déblocage d'invocations avancées et stratégiques).
- Bonus aléatoires temporaires en cours de partie.
- Support manette.
- Ajustement de la difficulté après une première victoire.

## Mécaniques V1 précisées (descente du 2026-07-20, suite)

**Contrôle RTS-lite** : touches 1/2/3 = choisir quel type d'invocation faire naître, clic = position de spawn. Clic au sol = les invocations se déploient selon leur comportement par défaut. Clic sur un groupe d'ennemis = ordre d'attaque (un nouvel ordre annule le précédent, jamais cumulatif). Les invocations reviennent former une horde autour du mage, regroupées par famille. Bascule par famille : **défense** (reste près du mage) vs **détachée** (prête à partir en mission). Rapproché volontairement du système Pikmin (suivre le leader, groupes par type, ordre d'attaque ponctuel) — faible risque technique, pas de pathfinding complexe ni de sélection multiple à la souris.

**Invocations de départ (3, confirmées)** :
- **Gobelin** — rapide, comportement d'attaque au corps à corps simple, peu de PV, peu de dégâts (suffisant pour les premiers villageois).
- **Démon** — corps à corps + attaque à distance après cooldown, plus de PV et de dégâts que le gobelin. (⚠️ le plus complexe des trois à coder — double comportement mêlée/distance — premier candidat à simplifier si le temps manque.)
- **Colosse** — beaucoup de PV, beaucoup de dégâts, très lent.

Rôles distincts par les stats, pas une matrice de contre-types (cohérent avec le choix "pas de contre-types en V1").

**Mana** : pas de plafond d'invocations vivantes — la limite vient du mana disponible et d'un **cooldown de invocation** (par type). Un compteur affiche combien d'unités de chaque famille sont vivantes.

**Fin de niveau** : **score atteint uniquement** (pas d'élimination totale) — décision volontaire d'Arnaud pour éviter que le joueur doive traquer les derniers villageois isolés sur la carte, un piège de design connu des jeux à objectif "éliminer tout le monde".

**Persistance entre niveaux** : mana rechargé à fond à chaque nouveau niveau, mais le nombre d'invocations survivantes de chaque famille est conservé d'un niveau à l'autre — crée une vraie pression de run (perdre des invocations pénalise directement le niveau suivant).

**Style visuel** : pixel art, pipeline déjà en main (Sorceress/GDevelop, comme sur TLR) — confirmé, pas de changement de style à prévoir. Budget d'assets à garder conscient : peu de types, variantes par recolor plutôt que sprites uniques.

**Méta-progression (orbes)** : au moins une amélioration envisagée — cooldown d'invocation plus bas. Reste du contenu volontairement laissé ouvert ("on verra ça au fur et à mesure") — pas bloquant pour la suite de la descente.

**Score = orbes (décidé le 2026-07-20)** : une seule monnaie, pas deux compteurs séparés — cohérent avec le principe directeur "finir vite > riche" (moins d'UI, moins à expliquer). Risque de farming des niveaux faciles accepté pour la V1, à corriger plus tard si besoin (ex. rendement décroissant sur un niveau déjà nettoyé) — pas bloquant maintenant.

**Fin de niveau — mécanique de sortie (décidé le 2026-07-20)** : atteindre le seuil d'orbes ne termine pas le niveau automatiquement — ça **débloque une sortie**, indiquée visuellement par le jeu, que le joueur doit atteindre. Fenêtre de **10 secondes** pour l'atteindre une fois débloquée. Les vagues d'ennemis s'intensifient pendant cette fenêtre — transforme la fin de niveau en moment de tension/challenge plutôt qu'un arrêt net, et laisse un choix implicite au joueur (foncer vers la sortie vs risquer quelques secondes de plus pour looter un peu d'orbes en plus, sachant que la pression augmente).

**Économie de mana — première passe (décidé le 2026-07-20, à ajuster en playtest)** : coût évolutif selon la puissance de l'invocation, cooldown qui suit le coût (seul autre frein au spam, puisqu'il n'y a pas de plafond d'invocations vivantes).

| | Coût mana | Cooldown |
|---|---|---|
| Mana max | 100 | — |
| Régén passive | 5/sec (0→100 en 20s) | — |
| Gobelin | 15 | 2s |
| Démon | 30 | 5s |
| Colosse | 50 | 10s |

⚠️ Chiffres de départ non validés en jeu réel — flag de playtest obligatoire avant de les considérer figés.

**Nombre de niveaux — 3, confirmé (décidé le 2026-07-20)** : la progression narrative complète (village → centre pavé → pied du château → couloirs → sommet) est conservée mais fusionnée sur 3 niveaux plutôt que 5, pour limiter le contenu à produire (cohérent avec "finir vite > riche") :
1. **Abords du village** — large, ennemis faibles peu agressifs.
2. **Centre pavé + pied du château** — espace qui se resserre, ennemis plus nombreux/dangereux, obstacles défensifs.
3. **Couloirs + sommet du château** — exigu puis large à nouveau, boss final.

La version à 5 niveaux (un par étape narrative) reste une option V2 facile (redécouper les niveaux fusionnés), pas une perte définitive.

## Ouvert — prochaine étape de la descente (pour la V1)

- Design concret des 3 niveaux (dimensions, nombre/placement d'ennemis précis par niveau).
- Reste de la liste des améliorations permanentes de méta-progression (au-delà du cooldown plus bas déjà cité).
- Nom définitif du jeu.

## Moteur technique — confirmé (2026-07-20)

**GDevelop**, par continuité avec TLR (pipeline pixel art Sorceress déjà en main, voir section "Mécaniques V1 précisées"). Les références techniques GDevelop de TLR (pixel art, tiles/Tiled, PixelLab, calques 2D) restent consultables par renvoi vers `Desktop/The Long Ride/PROJECT/docs/actif/` — jamais dupliquées dans Projet X (voir `CLAUDE.md`, section `<relation-the-long-ride>`).

## Index (tags)
#ProjetX #Concept #Brainstorming #Necromancien

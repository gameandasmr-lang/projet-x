---
name: gdevelop-operations
description: À utiliser pour toute lecture, création ou modification du JSON GDevelop de Projet X (structure d'événements, variables, scènes). À consulter avant tout /plan ou toute édition directe du fichier projet GDevelop, et avant de déclarer une modification terminée. Adapté depuis les skills GDevelop de The Long Ride (tlr-gdevelop-expert, tlr-variable-guardian, dependency-checker, systematic-debugging, verification-before-completion), purgé de tout contenu spécifique à TLR — ne contient que la littératie moteur, indépendante du jeu.
---

# GDevelop Operations — Projet X

Connaissance moteur générique, valable pour n'importe quel jeu GDevelop — pas de logique métier Projet X figée ici (elle vivra dans `docs/structure_json_projet_x.md` une fois qu'il existera).

## Édition du JSON — discipline de base

- Toute écriture dans le JSON du projet se fait directement via Claude Code (Edit), jamais via un script Python/Node/PowerShell intermédiaire.
- Ne jamais parser tout le fichier — cibler scène + groupe d'événements exact.
- Relire le bloc modifié juste après l'écriture, pour confirmer que la structure JSON reste valide (accolades/tableaux fermés).
- Ne modifier que le périmètre demandé — un groupe voisin non concerné ne bouge pas.

## Ordre des groupes d'événements — règle critique

Les groupes d'une scène GDevelop s'exécutent dans l'ordre où ils apparaissent. Un groupe plus bas peut silencieusement écraser ce qu'un groupe plus haut vient de poser (même variable, même frame).
- Avant de déplacer un groupe : identifier tout ce qui le lit/l'écrit en amont et en aval (voir "Vérifier les dépendances" plus bas).
- Un bug "la valeur revient toujours à X" est presque toujours un groupe aval qui réécrit — vérifier ce point avant toute autre hypothèse.

## Piège scope global vs scène — à vérifier systématiquement

GDevelop distingue variables **globales** et variables **de scène**, avec une confusion facile à produire et difficile à repérer :
- `SetVariable` / `SetStringVariable` sur une structure déclarée **globale** écrit bien dans le global.
- Mais `Variable()` / `VariableString()` (sans préfixe) lisent toujours la **scène** — qui reste vide pour une structure globale → condition silencieusement fausse → l'event ne se déclenche jamais, sans erreur visible.
- Lecture correcte d'un global : `GlobalVariable(...)` / `GlobalVariableString(...)`.
- **Réflexe avant toute condition sur une structure globale** : vérifier qu'aucune lecture n'utilise `Variable()`/`VariableString()` par erreur. Si un bouton/event d'une série fonctionne et qu'un autre identique ne fonctionne pas, comparer les scopes en premier — c'est la cause la plus fréquente.

## Types de variables — ne jamais écrire dans une variable calculée

Avant d'écrire une valeur, se demander : est-ce une **cause** (variable éditable, ok à modifier) ou un **effet** (variable calculée/runtime, à ne jamais forcer directement) ?
- Éditable : modifiable directement (réglages, config).
- Calculée : recalculée automatiquement à partir d'autres variables — écrire dedans directement se fait écraser au prochain recalcul. Corriger la cause en amont, jamais l'effet.
- Runtime : évolue par la simulation (état de jeu en cours) — ne jamais la forcer "pour tester", trouver l'action qui doit légitimement la faire évoluer.

## Séparation des couches (layers)

Ne jamais mélanger dans un même objet/layer : simulation (logique de jeu), HUD (affichage joueur permanent), outils de debug (temporaires, dev uniquement), popups (panneaux ponctuels). Un texte de debug ne va jamais dans le layer HUD, un élément HUD ne va jamais dans le layer de simulation.

## Vérifier les dépendances avant de toucher un groupe/variable

Avant de modifier, déplacer ou supprimer un groupe d'événements ou une variable partagée :
1. Qui lit cette zone (autres groupes, autres scènes) ?
2. Qui écrit dedans (risque d'écrasement croisé) ?
3. Y a-t-il un ordre critique à respecter autour ?
Si la zone est lue/écrite par plusieurs groupes → modification prudente, signaler le risque avant d'agir plutôt que de découvrir la régression après coup.

## Debugging — méthode

Jamais de fix avant d'avoir tracé la cause racine :
1. **Investigation** — localiser la zone exacte (scène + groupe), tracer d'où vient la valeur incorrecte, vérifier l'ordre des groupes autour.
2. **Comparaison** — trouver un event similaire qui fonctionne, lister precisément ce qui diffère.
3. **Hypothèse unique** — une seule cause supposée à la fois, testée avec le changement minimal. Si fausse, nouvelle hypothèse — jamais empiler les fixes.
4. **Correction de la cause racine**, pas du symptôme. Si 3 tentatives échouent sur la même zone → s'arrêter, remettre en question l'approche plutôt que retenter.

## Avant de déclarer une modification terminée

Aucune déclaration de type "c'est fait" / "corrigé" sans vérification fraîche :
1. Relire le bloc modifié dans le JSON.
2. Confirmer que la structure JSON est intacte.
3. Confirmer que l'ordre des groupes autour est inchangé (sauf si le déplacement était le but).
4. Confirmer que seul le périmètre demandé a changé.

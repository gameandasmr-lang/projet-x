---
name: gdevelop-operations
description: À utiliser pour toute lecture, création ou modification du JSON GDevelop de Projet X (structure d'événements, variables, scènes). À consulter avant tout /plan ou toute édition directe du fichier projet GDevelop, et avant de déclarer une modification terminée. Adapté depuis les skills GDevelop de The Long Ride (tlr-gdevelop-expert, tlr-variable-guardian, dependency-checker, systematic-debugging, verification-before-completion), purgé de tout contenu spécifique à TLR — ne contient que la littératie moteur, indépendante du jeu.
---

# GDevelop Operations — Projet X

Connaissance moteur générique, valable pour n'importe quel jeu GDevelop — pas de logique métier Projet X figée ici (elle vit dans `docs/structure_json_projet_x.md`).

## Édition du JSON — discipline de base

- **Consulter `docs/structure_json_projet_x.md` en premier** pour localiser directement le bon groupe (nom, rôle, ordre) plutôt que de re-deviner par grep à chaque session.
- Toute écriture dans le JSON du projet se fait directement via Claude Code (Edit), jamais via un script Python/Node/PowerShell intermédiaire.
- Ne jamais parser tout le fichier — cibler scène + groupe d'événements exact.
- Relire le bloc modifié juste après l'écriture, pour confirmer que la structure JSON reste valide (accolades/tableaux fermés).
- **Ajouter un nouvel objet Sprite référençant une image ne suffit pas** : le champ `image` d'un objet Sprite (dans `animations[].directions[].sprites[].image`) référence un fichier **par son nom**, mais GDevelop le résout via le manifeste global `resources` (en tête du JSON, avant `objects`) — une entrée `{ "file": "assets/xxx.png", "kind": "image", "name": "xxx.png", ... }` par fichier. Oublier cette entrée fait échouer silencieusement le chargement (pas d'erreur JSON) : l'objet s'affiche cassé dans l'éditeur (placeholder violet/logo G, ou case vide) une fois le projet **rechargé**. Toujours ajouter l'entrée `resources` en même temps que l'objet Sprite qui l'utilise, jamais l'un sans l'autre. **L'éditeur GDevelop ne relit pas le fichier JSON en continu** : après une édition directe du JSON pendant que le projet est ouvert dans l'éditeur, il faut recharger/rouvrir le projet pour voir l'effet (nouveaux objets, resources, etc.).
- Ne modifier que le périmètre demandé — un groupe voisin non concerné ne bouge pas.
- **⚠️ RÈGLE CRITIQUE (incident 2026-07-23, session 8) : ne jamais éditer le JSON directement pendant que GDevelop a le projet ouvert.** Une série d'éditions faites pendant que l'éditeur tournait en arrière-plan a rendu le projet impossible à ouvrir ("corrompu", sans message d'erreur détaillé) — cause probable : collision entre la sauvegarde automatique de GDevelop et l'écriture directe du fichier. **Avant toute édition du JSON, demander explicitement à Arnaud si GDevelop est ouvert, et s'il l'est, lui demander de le fermer complètement d'abord.** Après édition, revalider (JSON.parse + cohérence structurelle) avant de dire que c'est prêt à rouvrir.
- **⚠️ Piège BOM UTF-8 (même incident) : `Set-Content -Encoding UTF8` en PowerShell 5.1 ajoute un BOM (3 octets `EF BB BF`) en tête de fichier par défaut.** GDevelop échoue à charger un fichier avec BOM en tête — message générique "vérifier que le projet a été créé avec GDevelop / autorisation d'ouvrir", pas une erreur JSON explicite. Symptôme confirmé une fois (fichier syntaxiquement valide selon `ConvertFrom-Json`, mais GDevelop refusait quand même de l'ouvrir). **Ne jamais utiliser `Set-Content -Encoding UTF8` sur le fichier projet** — utiliser `[System.IO.File]::WriteAllText($path, $content, (New-Object System.Text.UTF8Encoding $false))` (le `$false` = pas de BOM), ou vérifier après coup avec `Format-Hex -Count 3` / `xxd` que les 3 premiers octets sont bien `7B 0A 20` (le `{` du JSON) et non `EF BB BF`.

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

## Schéma JSON des types d'objets non-Sprite

Un objet `Sprite` a un schéma bien rodé dans ce projet (`animations[].directions[].sprites[].image` + `originPoint`/`centerPoint`). D'autres types d'objets (TiledSprite, Tilemap, Particles...) ont chacun leur propre schéma, différent — ne jamais réutiliser le schéma Sprite pour un autre type par supposition.
- **`TiledSpriteObject::TiledSprite`** (texture répétée sur une zone redimensionnable) — **retiré du projet le 2026-07-23** après un vrai problème en jeu : bien que le schéma JSON soit correct, la texture répétée par `TiledSprite` ne partage aucune référence de grille avec l'objet `TileMap::SimpleTileMap` utilisé pour le chemin. Résultat observé : les tuiles peintes dans le Tilemap ne s'alignaient pas visuellement avec le fond d'herbe en `TiledSprite`, même les deux étant nominalement sur une grille 32×32. **Ne pas mélanger `TiledSprite` et `TileMap::SimpleTileMap` pour des éléments qui doivent s'aligner visuellement** — préférer un second `TileMap::SimpleTileMap` (même atlas, une instance dédiée en remplissage uni) pour tout fond qui doit coexister avec un plan de tuiles peint, ça garantit la même grille. Schéma toujours valable si `TiledSprite` est réutilisé ailleurs (ex. élément qui n'a pas besoin de s'aligner à un tilemap) : champs à plat `"texture"`/`"width"`/`"height"` sur l'objet, `"customSize"`+`"width"`/`"height"`+`"x"`/`"y"` (coin haut-gauche) sur l'instance.
- **`TileMap::SimpleTileMap`** ("Simple Tile map" dans le sélecteur d'objet — peinture directe dans l'éditeur GDevelop, sans fichier Tiled externe, contrairement à `TileMap::TileMap` qui lui en a besoin) :
  - **Niveau objet** (partagé par toutes les instances) sous `"content"` : `"atlasImage"` (nom de ressource — une planche unique contenant toutes les tuiles rangées en grille, pas un fichier par tuile), `"rowCount"`/`"columnCount"` (dimensions de la grille de la planche), `"tileSize"` (un seul nombre, tuile carrée), `"tilesWithHitBox"` (chaîne, vide si aucune collision). Confirmé via un objet créé par Arnaud dans l'éditeur.
  - **Niveau instance** (indépendant par instance placée, donc plusieurs instances du même objet peuvent avoir des plans de tuiles différents) : `stringProperties` contient une entrée `{"name": "tilemap", "value": "<JSON échappé>"}`. Ce JSON interne a la forme `{"tileWidth", "tileHeight", "dimX", "dimY", "layers": [{"id": 0, "alpha": 1, "tiles": [[...]]}]}` — `tiles` est un tableau 2D `tiles[ligne][colonne]`, une ligne par `dimY`, `dimX` valeurs par ligne. **Valeur stockée = index brut de la tuile dans l'atlas (0 = première tuile en haut à gauche, numérotation ligne par ligne : index = ligne×columnCount + colonne), PAS un index+1 — `-1` représente une case vide.** Confirmé par lecture du code (`resources/GDJS/Runtime/Extensions/TileMap/helper/TileMapHelper.js`, méthode `toJSObject`/`getTileGID`) puis vérifié visuellement (tuile peinte avec valeur `1` = bien la 2e tuile de l'atlas, pas la 1ère). ⚠️ Le moteur runtime utilise en interne une représentation décalée de +1 (0 = case vide) une fois le JSON chargé — ne pas confondre les deux représentations : celle **sur disque** (index brut, -1=vide) est celle à utiliser pour toute édition JSON directe.
- **Astuce générale si un autre type d'objet est nécessaire** : chercher son schéma directement dans `resources/GDJS/Runtime/Extensions/<NomExtension>/` de l'installation GDevelop locale (`C:\Users\ROG\AppData\Local\Programs\GDevelop\resources\GDJS\Runtime\Extensions\`) plutôt que deviner depuis la mémoire — le runtime JS lit exactement les champs attendus (`objectData.xxx`), c'est la source de vérité la plus fiable disponible sans ouvrir l'éditeur soi-même. Encore mieux si Arnaud peut créer un exemple minimal de l'objet dans l'éditeur en premier — un objet réel qu'il a configuré via l'UI est une vérification plus sûre que le code source seul (cas de `TileMap::SimpleTileMap` ci-dessus).

## Identifiants d'instructions vérifiés (évite de redeviner)

En écrivant le JSON directement (sans passer par l'éditeur), le nom exact d'une instruction (`type.value`) est parfois différent du nom affiché dans l'UI, et se découvre seulement si le bloc ressort en rouge à l'ouverture. Liste vérifiée en conditions réelles (Projet X, GDevelop 5.6.274) — à compléter au fil des sessions plutôt que redeviner :

| Besoin | `type.value` correct | Piège rencontré |
|---|---|---|
| Centrer la caméra sur un objet | `CenterCameraOnObject` | Pas `CenterCamera` |
| Simuler une touche pour un comportement (ex. TopDown) | `NomDuComportement::SimulateUpKey` / `Down`/`Left`/`RightKey` | Paramètres : `[NomObjet, NomComportement]` |
| Touche clavier appuyée (condition) | `KeyPressed` | Paramètres `["", "z"]` (2e param en minuscule) |
| Bouton souris appuyé (condition) | `MouseButtonPressed` | Paramètres `["", "Left"]` |
| Créer un objet à une position | `Create` | Paramètres `["", "NomObjet", "ExpressionX", "ExpressionY", ""]` |
| Comparer une variable globale (condition) | `VarGlobal` | Paramètres `[nom, signe, valeur]` — signe **entre** nom et valeur |
| Modifier une variable globale (action) | `ModVarGlobal` | Paramètres `[nom, opérateur ("+","-","="), valeur]` |
| Changer le texte d'un objet Texte | `TextObject::String` | Pas `TextObject::SetString`. Paramètres `[NomObjet, "=", expression string]` |
| Distance entre deux objets (condition) | `Distance` | **Sens fixe "est en dessous de X pixels"** — pas de paramètre de signe, un 4e paramètre existe mais ne change rien à l'affichage. Pour "supérieur à X" : mettre `"inverted": true` dans `type`, pas un paramètre. Paramètres `[Objet1, Objet2, valeur]`. Fonctionne aussi objet contre lui-même (ex. `Gobelin`/`Gobelin`) — le moteur exclut automatiquement la comparaison d'une instance avec elle-même. **Piège `PickNearest`/`Distance` inversé combiné à un objet qui peut avoir zéro instance** (ex. "aucun Gobelin vivant") : non testé si `PickNearest`/`Distance` sur une liste vide fait échouer tout l'event (comme le piège `VarObjet` inversé multi-instances) ou renvoie correctement "vrai". Évité par prudence dans ce projet (Villageois visant le Nécromancien même sans Gobelin en vie) via un **drapeau précalculé chaque frame dans le bon sens** (`GobelinBloque` : reset à 0 puis mis à 1 uniquement si un Gobelin est trouvé à portée), jamais une condition inversée directe sur un objet qui peut ne pas exister — même technique que `Bloque`/`NbCiblesGroupe` |
| Forcer un objet à se déplacer vers un autre | `AddForceTowardObject` | Paramètres `[Objet1, ObjetCible, vitesse, ""]` |
| Supprimer un objet | `Delete` | Paramètres `[NomObjet, NomObjet]` (le nom répété deux fois — pas `["", NomObjet, ""]` comme pour `Create`). Confirmé via l'éditeur qui a normalisé une première tentative différente sans faire ressortir de bloc rouge |
| Écarter deux objets qui se chevauchent | `SeparateFromObjects` | Pas `SeparateObjectsWithoutForces` ni `AddForceAwayFromObject` (n'existent pas). Paramètres `[Objet1, Objet2, ""]` |
| Le curseur est sur un objet (condition) | `SourisSurObjet` | Paramètres `[NomObjet, "", "", ""]` (précision/layer/caméra optionnels, vides). Filtre la liste d'instances de l'objet à celles sous le curseur — utile pour restreindre les actions suivantes du même event à l'instance cliquée. Inversable via `"inverted": true` |
| Comparer une variable d'objet (condition) | `VarObjet` | Paramètres `[NomObjet, NomVariable, signe, valeur]` — même schéma que `VarGlobal` avec l'objet en tête. **Piège `inverted:true` avec plusieurs instances** : l'inversion s'évalue instance par instance, pas comme un "aucune instance ne correspond" global. Avec 3 instances dont 1 seule correspond au critère normal, la version inversée reste vraie tant qu'au moins 1 des 2 autres ne correspond pas (quasi toujours vrai). Pour un vrai test "aucune instance ne correspond" (ex. "personne n'est actuellement ciblé"), utiliser une variable **globale** dédiée mise à jour en parallèle, jamais l'inversion d'une condition d'objet multi-instances |
| Modifier une variable d'objet (action) | `ModVarObjet` | Paramètres `[NomObjet, NomVariable, opérateur, valeur]` — même schéma que `ModVarGlobal` avec l'objet en tête |

| Traiter les instances d'un objet une par une (event) | `"type": "BuiltinCommonInstructions::ForEach"` avec un champ `"object": "NomObjet"` à côté de `conditions`/`actions` | Pas de champ `events` imbriqué (contrairement à `Group`) — conditions/actions à plat comme un event Standard, mais évalués une instance à la fois. Essentiel pour éviter qu'une action groupée s'applique en cross-product sur toutes les paires d'instances de deux objets à la fois (ex. un seul Gobelin ne doit interagir qu'avec une seule cible, pas toutes celles à portée simultanément) |
| Sélectionner l'objet le plus proche d'une position (condition) | `PickNearest` | Paramètres `[NomObjet, expressionX, expressionY, ""]` — à utiliser à l'intérieur d'un `ForEach` sur l'autre objet pour filtrer par rapport à l'instance courante (ex. `Gobelin.X()`/`Gobelin.Y()`). ⚠️ Pas encore confirmé en jeu à ce stade (structure lue depuis un event construit par Arnaud dans l'éditeur, mais X/Y n'étaient pas encore renseignés au moment de la lecture) |
| Changer la position X d'un objet (action) | `SetX` | Paramètres `[NomObjet, opérateur ("+","-","="), expression]` — même schéma que `ModVarGlobal`/`ModVarObjet` (opérateur au milieu). Structure lue depuis un exemple créé par Arnaud dans l'éditeur. ⚠️ `SetY` supposé exister en miroir (vu listé juste à côté dans le menu "Position X"/"Position Y") mais pas encore testé en jeu à ce stade |
| Fonctions trigonométriques dans une expression | `sin(...)`, `cos(...)` | ⚠️ Pas encore testées en jeu — hypothèse par extension de la convention minuscule déjà confirmée pour `round(...)` |
| Racine carrée dans une expression | `sqrt(...)` | Confirmée en jeu (utilisée pour la fuite en bloc des Villageois) — même convention minuscule que `round`/`sin`/`cos` |
| Arrondir un nombre dans une expression | `round(...)` | **Minuscules**, contrairement à `ToString`/`GlobalVariable`/`MouseX` (PascalCase). Les fonctions mathématiques d'expression (`round`, `floor`, `ceil`, `abs`, `sqrt`...) suivent la convention minuscule héritée des bibliothèques mathématiques, pas la convention PascalCase du reste du moteur — a fait ressortir un bloc rouge une première fois avec `Round(...)` |

| Créer plusieurs instances puis les configurer ensemble | Plusieurs `Create` du même objet à la suite, puis une action (`ModVarObjet`...) sur ce même objet, dans le même event | **Confirmée en jeu** (vague 2 de Villageois, 7 `Create` puis un seul `ModVarObjet GroupeID=2` appliqué aux 7) — les instances créées s'accumulent bien dans la liste "picked" de l'event, l'action suivante s'applique à toutes, pas seulement la dernière |
| Lire la variable d'un objet dans une expression (texte, calcul) | `NomObjet.Variable(NomVariable)` | Déjà utilisé et confirmé (vibration Gobelin/Villageois, affichage PV Necro) — différent de `VarObjet` (condition) / `ModVarObjet` (action) qui eux servent en conditions/actions, pas dans une expression |

**Réflexe si un nom n'est pas dans cette liste** : proposer la meilleure hypothèse, prévenir explicitement Arnaud que ce n'est pas vérifié, et lui demander de signaler precisément quel bloc ressort rouge après rechargement dans l'éditeur — jamais réessayer une 2e fois à l'aveugle sans nouvelle information (comparer le texte affiché par l'éditeur avant de re-deviner, ou laisser Arnaud sélectionner l'action directement dans le moteur de recherche de l'éditeur si 2 tentatives ont échoué).

## Avant de déclarer une modification terminée

Aucune déclaration de type "c'est fait" / "corrigé" sans vérification fraîche :
1. Relire le bloc modifié dans le JSON.
2. Confirmer que la structure JSON est intacte.
3. Confirmer que l'ordre des groupes autour est inchangé (sauf si le déplacement était le but).
4. Confirmer que seul le périmètre demandé a changé.

# Pipeline de production du décor — Projet X

**Statut** : document vivant, créé le 2026-07-23 (session 7). Adapté de la méthode de production d'assets de The Long Ride (TLR) — jamais une copie mécanique, seulement ce qui reste pertinent pour un jeu top-down GDevelop plus petit et plus simple.
**Portée** : uniquement la production du décor (tuiles de sol, chunks, props, bâtiments) du niveau 1. Ne couvre pas les sprites de personnages/ennemis (Necromancien/Gobelin/Villageois restent des placeholders solides pour l'instant, hors sujet ici).

---

## 1. Ce qu'on reprend de TLR (par référence, jamais dupliqué)

Voir `Desktop/The Long Ride/PROJECT/docs/actif/` pour le détail complet de chaque règle — ce document ne fait que pointer vers la source et noter ce qui s'applique tel quel à Projet X.

- **Génération de tuiles** (`REFERENCE_PIPELINE_ART_TILES_2D.md`) : vue 3/4 (jamais isométrique), éclairage toujours haut-gauche, grille 32×32, fond magenta pour le détourage, **pas de contour foncé sur les bords qui doivent se raccorder** (herbe↔chemin), rendu **plat sans ombre portée directionnelle cuite** dans la tuile (l'éclairage de scène reste géré côté GDevelop, pas dans l'asset source).
- **Fondamentaux qualité** (`REFERENCE_TECHNIQUE_PIXEL_ART.md`) : vocabulaire de contrôle qualité (banding/pillow shading, doubles/jaggies, blocs de couleur jamais de bruit pixel isolé, ombre au sol pour ancrer un objet) — sert à juger si une génération est acceptable ou à régénérer/retoucher.
- **Base + Sommet + YSort** (`REGLES_PROFONDEUR_CALQUES_2D.md`) : tout objet plus haut que le Nécromancien (maison, arbre) est découpé en 2 sprites — `[Nom]_Base` (YSort activé, origine au point de contact au sol) + `[Nom]_Sommet` (pas de YSort, toujours au-dessus). Un objet plus bas (buisson, caisse) reste un seul sprite YSort, origine au sol.
- **Outil de production** : suite **Sorceress** (déjà en main, Pro à vie payé) — génération via un modèle d'image (persona "Concepteur de tiles", section 3 ci-dessous) puis **Tileset Forge** pour assembler/nettoyer une planche de plusieurs tuiles apparentées (herbe + bords + chemin) quand une génération produit une planche multi-tuiles plutôt qu'une texture unique.

## 2. Ce qu'on ne reprend PAS de TLR — décisions actées le 2026-07-23

- **Pas de Tiled externe, pas d'objet External Tilemap.** TLR utilise Tiled (logiciel séparé, Terrain Sets, Automapping) pour assembler ses cartes — chaîne plus outillée mais plus lourde, et **pas encore validée en conditions réelles même côté TLR** (`REGLES_PROFONDEUR_CALQUES_2D.md` section 5 : "pas encore testé en conditions réelles dans GDevelop"). Projet X n'a qu'une seule scène pour l'instant (56×32 tuiles, `Niveau1_Village`) — décision explicite d'Arnaud (2026-07-23) : intégration **directe dans GDevelop** (Sprites/TiledSprite placés à la main dans l'éditeur, ou Tilemap natif de GDevelop si besoin), cohérent avec le principe directeur "finir vite". À rouvrir seulement si le nombre de tuiles/transitions devient réellement ingérable à la main.
- **Palette propre à Projet X, pas celle de TLR.** TLR porte une identité "cyclisme champêtre chaleureux" (25 teintes validées le 2026-07-18). Projet X est un **village de héros chaleureux et vivant**, que le Nécromancien (le joueur) vient perturber — même exigence de chaleur/vie, mais un contexte medieval-fantasy, pas cycliste. Palette candidate en section 4, à valider par Arnaud avant la première génération en masse (même discipline que TLR : jamais silencieusement figée).

## 3. Génération — chunks de sol vs objets modulaires (décidé le 2026-07-23)

**Question posée par Arnaud** : faut-il générer des tiles/chunks qu'on assemble, ou une image de scène entière posée telle quelle (comme l'image de référence complète) ?

**Réponse retenue : modulaire (chunks de sol + objets Base/Sommet séparés), jamais une scène entière en une seule image posée comme fond.** Raisons concrètes, propres à Projet X (pas juste "suivre TLR par habitude") :

1. **Y-sort dynamique obligatoire.** Le Nécromancien, les Gobelins et les Villageois doivent pouvoir passer visuellement devant ou derrière un arbre/une maison selon leur position Y réelle (`REGLES_PROFONDEUR_CALQUES_2D.md`). Une image de scène entière est plate : aucune occlusion dynamique possible entre un objet en mouvement et un élément de décor peint dedans.
2. **Collision/obstacles.** Niveau 2 prévoit déjà des "obstacles défensifs" qui bloquent déplacement + ligne de vue (`CONCEPT_JEU.md`). Un décor modulaire donne un objet GDevelop par maison/clôture, avec une hitbox précise. Une image plate n'a pas de découpage exploitable pour ça sans retracer des hitboxes à la main dessus (perd l'intérêt de la rapidité).
3. **Renforts sortant des maisons** (idée déjà notée dans `reprise_actuel.md`, section "Ouvert" — le joueur ne doit jamais savoir de quelle maison un renfort va sortir). Impossible si les maisons ne sont pas des objets identifiables individuellement dans la scène.
4. **Réutilisation entre niveaux.** Un jeu de tuiles/props modulaires (herbe, chemin, maison, arbre, clôture) se réutilise sur les 3 niveaux avec des variantes, une image de scène entière ne sert qu'une fois.

**Ce qui peut rester en "chunk" plus large (pas obligatoirement du 32×32 unitaire)** : le **sol pur** (herbe, pavés de place, chemin de terre) n'a besoin d'aucun Y-sort ni collision — rien ne marche jamais "derrière" une texture de sol. Comme TLR (`Chunk de route`, multiples de 640px), rien n'empêche de générer/poser des blocs de sol plus grands qu'une tuile unitaire pour aller plus vite, tant qu'aucune verticalité n'y est mêlée. Tout ce qui a une hauteur ou une fonction de gameplay (maison, arbre, clôture, puits, obstacle) reste un objet séparé.

## 4. Palette — candidat extrait de la référence d'Arnaud (2026-07-23, NON validé)

**Origine** : image de référence transmise par Arnaud (`Generated Image July 20, 2026 - 11_59AM.jpg`, vue TLR ¾ existante) — sert de référence de **cadrage/densité/style** ("même vue que TLR, en beaucoup plus dégagé, style médiéval"), et sert ici de point de départ technique pour une palette chaleureuse, avant retouche par Arnaud.

**Méthode** : extraction technique par échantillonnage de pixels (script PowerShell `System.Drawing`, quantification par blocs de 20 sur chaque canal, tri par fréquence d'apparition) — mêmes principes que le scan de palette de TLR (`CHARTE_TECHNIQUE_PRODUCTION_ASSETS.md` section 1bis), pas une approximation à l'œil.

✅ **Validée par Arnaud le 2026-07-23** ("ça me va"), style confirmé **médiéval-fantasy**. Ajustable au fil de la production comme TLR (jamais silencieusement figée au-delà de cette première validation).

| Groupe | Codes hex candidats | Usage envisagé |
|---|---|---|
| Toits / chaleur | `#F0C878` · `#F0B478` · `#DCA078` · `#DCB478` · `#C8A078` · `#C8A064` | Toitures, lumière chaude, accents dorés |
| Végétation | `#8C8C50` · `#78783C` · `#A0A050` · `#64643C` · `#505028` · `#3C3C28` | Herbe, feuillage — teinte plus olive/dorée que verte, cohérent avec un éclairage chaud |
| Pierre / bois (structures) | `#B4A08C` · `#A08C78` · `#C8B4A0` · `#786450` · `#503C28` · `#A08C64` · `#B48C64` | Murs, chemins, clôtures, bâtiments |
| Accents sombres / ombres | `#A06450` · `#785050` · `#643C3C` · `#502828` · `#3C2828` | Portes, ombres portées, contraste |

**Ce que cette palette ne couvre pas encore** : aucune teinte "menace"/nécromancien (violet/or/noir déjà présents sur `Assets/Necromancien.jpg`, palette du personnage non retouchée par cette charte — même principe que TLR section 1bis "porte l'identité de l'environnement, pas du personnage"). Le contraste chaleur-du-village vs noirceur-du-nécromancien vient du contraste entre ces deux palettes séparées, pas d'un mélange dans une seule.

## 5. Persona de génération — "Concepteur de tiles — Projet X" (2026-07-23)

Adaptation medieval-fantasy de la persona TLR (`REFERENCE_PIPELINE_ART_TILES_2D.md` section 1) — à configurer une fois comme instruction système dans l'outil de génération (Nano Banana Pro/2 via Sorceress ou compte externe), pour ne pas avoir à la retaper à chaque prompt.

```
You are a tile/prop concept artist for a top-down medieval-fantasy pixel art village game. Every generation must follow these rules exactly:

- Camera: strict 3/4 top-down view, never isometric. Test: only the roof of a building should be visible, never its side walls.
- Lighting: always from the top-left, never centered on the object, never a baked directional drop shadow under/around the object — flat rendering, only local shading (creases, relief) consistent with a top-left light source.
- Grid: every tile is exactly 32x32 px (or an exact multiple for multi-tile props/sheets).
- Background: flat solid magenta (#FF00FF) for chroma-key removal, never a native transparent background.
- Color palette: strictly limited to these hex codes, no colors outside this list —
  Roofs/warmth: #F0C878, #F0B478, #DCA078, #DCB478, #C8A078, #C8A064
  Vegetation: #8C8C50, #78783C, #A0A050, #64643C, #505028, #3C3C28
  Stone/wood structures: #B4A08C, #A08C78, #C8B4A0, #786450, #503C28, #A08C64, #B48C64
  Dark accents/shadows: #A06450, #785050, #643C3C, #502828, #3C2828
- Seams: no dark outline on any edge meant to connect to a neighboring tile (grass-to-path, building base touching ground). A dark outline is only acceptable on an edge that never connects to anything (e.g. outer edge of an isolated rock/prop).
- Style: warm, welcoming medieval-fantasy village — stone and timber buildings, thatch or clay tile roofs, dirt paths, healthy vegetation. Heroic and lived-in, not modern, not cyclism-themed, not already dark/decayed (the threat comes from the player character, not from the village itself).
- No living character/creature in decor tiles — environment only.
```

## 6. Premier prompt concret — planche de tuiles de sol (2026-07-23)

Cible : famille de tuiles interconnectées (herbe de base + chemin de terre avec bords/coins + une tuile de pavés de place), plutôt qu'une texture unique — cohérent avec l'apprentissage TLR sur Tileset Forge (`REFERENCE_SORCERESS_GAMES.md` section 12bis : un prompt structuré "taille + type de tuile + style d'environnement" produit une planche de tuiles apparentées en une génération, contrairement à un prompt "seamless repeating texture"). Modèle recommandé par ce même apprentissage TLR : **Nano Banana 2** si disponible dans l'outil utilisé, sinon Nano Banana Pro.

```
32x32 pixel art tiles for a top-down medieval-fantasy village game. Generate a connected tile family on a flat magenta (#FF00FF) background: a grass base tile, dirt path tiles with grass-to-path edge transitions on all 4 sides plus outer corners, and one cobblestone plaza tile. Warm, sunlit, welcoming medieval village atmosphere. Top-left lighting, no baked drop shadows, flat shading only. Strict color palette: #F0C878, #F0B478, #DCA078, #DCB478, #C8A078, #C8A064, #8C8C50, #78783C, #A0A050, #64643C, #505028, #3C3C28, #B4A08C, #A08C78, #C8B4A0, #786450, #503C28, #A08C64, #B48C64, #A06450, #785050, #643C3C, #502828, #3C2828. No dark outline on edges meant to connect between tiles.
```

**Après génération** : passer la planche dans **Tileset Forge** (Detect → Build) pour isoler/nettoyer chaque tuile individuellement et confirmer que les raccords herbe↔chemin sont propres, avant tout export vers `game/assets/`. Une tuile qui ne se raccorde pas proprement se régénère ou se retouche à ce stade — pas un échec de méthode (`REFERENCE_SORCERESS_GAMES.md` : "la curation et la retouche manuelle sont une étape attendue, pas un échec de l'outil").

## Index (tags)
#PipelineDecor #ProjetX #Sorceress #NanoBananaPro #TilesetForge #PaletteCouleurs #YSort #ModulaireVsImagePosee

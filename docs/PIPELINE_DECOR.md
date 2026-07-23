# Pipeline de production du décor — Projet X

**Statut** : document vivant, créé le 2026-07-23 (session 7). Adapté de la méthode de production d'assets de The Long Ride (TLR) — jamais une copie mécanique, seulement ce qui reste pertinent pour un jeu top-down GDevelop plus petit et plus simple.
**Portée** : uniquement la production du décor (tuiles de sol, chunks, props, bâtiments) du niveau 1. Ne couvre pas les sprites de personnages/ennemis (Necromancien/Gobelin/Villageois restent des placeholders solides pour l'instant, hors sujet ici).

---

## 0. Synthèse — le pipeline concret, étape par étape (2026-07-23)

Les sections 1 à 8 sont dans l'ordre où elles ont été découvertes (recherche). Voici la même information réorganisée en **séquence d'exécution réelle**, pour suivre sans tout relire.

**Étape 1 — Définir le rôle de chaque tuile AVANT de générer** (section 4)
Pour une famille de sol à 2 matériaux (ex. herbe↔chemin) : **10 tuiles confirmé par Arnaud le 2026-07-23** (2 bases + 4 bords + 4 coins partagés, sans distinction intérieur/extérieur — voir section 4 pour le détail du compromis vs la version complète à 14). Repasser à 14 seulement si les coins gênent visuellement une fois en jeu. Pour une clôture/mur modulaire (pas un besoin actif aujourd'hui), utiliser la grille de paliers dédiée de la même section, en ne produisant que le palier réellement nécessaire.

**Étape 2 — Générer avec un prompt structuré unique** (sections 6, 7, 8)
Un seul prompt envoyé à Sorceress (modèle **Nano Banana 2**, sinon Nano Banana Pro) combinant : la persona système "Concepteur de tiles — Projet X" (configurée une fois, section 7), la palette validée (section 6), et le prompt concret déjà rédigé (section 8) qui nomme explicitement chaque rôle listé à l'étape 1 — jamais un prompt vague laissé à générer "au hasard".

**Étape 3 — Nettoyer/assembler dans Tileset Forge** (section 8)
Upload → Detect (isoler chaque tuile individuellement) → Build (retouche manuelle **attendue**, pas un échec : agrandir/rétrécir de 1px, clone stamp/gomme/pinceau pour les raccords imparfaits) → export PNG 32×32 vers `game/assets/`. C'est le vrai filet de sécurité si un rôle précis rate — **pas** l'inpainting PixelLab (vérifié indisponible avec les outils actuels, section 5).

**Étape 4 — Placer à la main dans GDevelop** (sections 1, 2, 3)
Pas de Tiled, pas de moteur d'autotile : Sprites/TiledSprite posés directement dans la scène `Niveau1_Village`. Le sol pur (herbe/chemin/pavés) peut être posé en chunks plus larges que 32×32 (aucun Y-sort ni collision requis). Tout objet avec hauteur ou fonction de gameplay (maison, arbre, clôture, obstacle) reste un objet séparé, découpé `_Base`/`_Sommet` + YSort activé si plus haut que le Nécromancien.

**Documenté mais volontairement hors V1 active** (à rouvrir seulement si un critère précis apparaît, jamais par anticipation) :
- **Dual-grid** (section 4) — seulement si le placement à la main devient réellement ingérable.
- **Grammaire clôture/mur modulaire** (section 4) — seulement si une clôture connectée devient un besoin réel de production.
- **Inpainting PixelLab** (section 5) — seulement si Arnaud prend un abonnement séparé (12$/mois minimum).

---

## 1. Ce qu'on reprend de TLR (par référence, jamais dupliqué)

Voir `Desktop/The Long Ride/PROJECT/docs/actif/` pour le détail complet de chaque règle — ce document ne fait que pointer vers la source et noter ce qui s'applique tel quel à Projet X.

- **Génération de tuiles** (`REFERENCE_PIPELINE_ART_TILES_2D.md`) : vue 3/4 (jamais isométrique), éclairage toujours haut-gauche, grille 32×32, fond magenta pour le détourage, **pas de contour foncé sur les bords qui doivent se raccorder** (herbe↔chemin), rendu **plat sans ombre portée directionnelle cuite** dans la tuile (l'éclairage de scène reste géré côté GDevelop, pas dans l'asset source).
- **Fondamentaux qualité** (`REFERENCE_TECHNIQUE_PIXEL_ART.md`) : vocabulaire de contrôle qualité (banding/pillow shading, doubles/jaggies, blocs de couleur jamais de bruit pixel isolé, ombre au sol pour ancrer un objet) — sert à juger si une génération est acceptable ou à régénérer/retoucher.
- **Base + Sommet + YSort** (`REGLES_PROFONDEUR_CALQUES_2D.md`) : tout objet plus haut que le Nécromancien (maison, arbre) est découpé en 2 sprites — `[Nom]_Base` (YSort activé, origine au point de contact au sol) + `[Nom]_Sommet` (pas de YSort, toujours au-dessus). Un objet plus bas (buisson, caisse) reste un seul sprite YSort, origine au sol.
- **Outil de production** : suite **Sorceress** (déjà en main, Pro à vie payé) — génération via un modèle d'image (persona "Concepteur de tiles", section 3 ci-dessous) puis **Tileset Forge** pour assembler/nettoyer une planche de plusieurs tuiles apparentées (herbe + bords + chemin) quand une génération produit une planche multi-tuiles plutôt qu'une texture unique.

## 2. Ce qu'on ne reprend PAS de TLR — décisions actées le 2026-07-23

- **Pas de Tiled externe, pas d'objet External Tilemap.** TLR utilise Tiled (logiciel séparé, Terrain Sets, Automapping) pour assembler ses cartes — chaîne plus outillée mais plus lourde, et **pas encore validée en conditions réelles même côté TLR** (`REGLES_PROFONDEUR_CALQUES_2D.md` section 5 : "pas encore testé en conditions réelles dans GDevelop"). Projet X n'a qu'une seule scène pour l'instant (56×32 tuiles, `Niveau1_Village`) — décision explicite d'Arnaud (2026-07-23) : intégration **directe dans GDevelop** (Sprites/TiledSprite placés à la main dans l'éditeur, ou Tilemap natif de GDevelop si besoin), cohérent avec le principe directeur "finir vite". À rouvrir seulement si le nombre de tuiles/transitions devient réellement ingérable à la main — voir section 4 ("Dual-grid non retenu maintenant") pour une option déjà documentée à évaluer à ce moment-là.
- **Palette propre à Projet X, pas celle de TLR.** TLR porte une identité "cyclisme champêtre chaleureux" (25 teintes validées le 2026-07-18). Projet X est un **village de héros chaleureux et vivant**, que le Nécromancien (le joueur) vient perturber — même exigence de chaleur/vie, mais un contexte medieval-fantasy, pas cycliste. Palette candidate en section 4, à valider par Arnaud avant la première génération en masse (même discipline que TLR : jamais silencieusement figée).

## 3. Génération — chunks de sol vs objets modulaires (décidé le 2026-07-23)

**Question posée par Arnaud** : faut-il générer des tiles/chunks qu'on assemble, ou une image de scène entière posée telle quelle (comme l'image de référence complète) ?

**Réponse retenue : modulaire (chunks de sol + objets Base/Sommet séparés), jamais une scène entière en une seule image posée comme fond.** Raisons concrètes, propres à Projet X (pas juste "suivre TLR par habitude") :

1. **Y-sort dynamique obligatoire.** Le Nécromancien, les Gobelins et les Villageois doivent pouvoir passer visuellement devant ou derrière un arbre/une maison selon leur position Y réelle (`REGLES_PROFONDEUR_CALQUES_2D.md`). Une image de scène entière est plate : aucune occlusion dynamique possible entre un objet en mouvement et un élément de décor peint dedans.
2. **Collision/obstacles.** Niveau 2 prévoit déjà des "obstacles défensifs" qui bloquent déplacement + ligne de vue (`CONCEPT_JEU.md`). Un décor modulaire donne un objet GDevelop par maison/clôture, avec une hitbox précise. Une image plate n'a pas de découpage exploitable pour ça sans retracer des hitboxes à la main dessus (perd l'intérêt de la rapidité).
3. **Renforts sortant des maisons** (idée déjà notée dans `reprise_actuel.md`, section "Ouvert" — le joueur ne doit jamais savoir de quelle maison un renfort va sortir). Impossible si les maisons ne sont pas des objets identifiables individuellement dans la scène.
4. **Réutilisation entre niveaux.** Un jeu de tuiles/props modulaires (herbe, chemin, maison, arbre, clôture) se réutilise sur les 3 niveaux avec des variantes, une image de scène entière ne sert qu'une fois.

**Ce qui peut rester en "chunk" plus large (pas obligatoirement du 32×32 unitaire)** : le **sol pur** (herbe, pavés de place, chemin de terre) n'a besoin d'aucun Y-sort ni collision — rien ne marche jamais "derrière" une texture de sol. Comme TLR (`Chunk de route`, multiples de 640px), rien n'empêche de générer/poser des blocs de sol plus grands qu'une tuile unitaire pour aller plus vite, tant qu'aucune verticalité n'y est mêlée. Tout ce qui a une hauteur ou une fonction de gameplay (maison, arbre, clôture, puits, obstacle) reste un objet séparé.

## 4. Grammaire d'autotiling — définir les rôles de tuiles avant génération (2026-07-23)

**Origine** : transcription vidéo *"Tuto Godot 4 #12 — Auto Tile expliqué"* (`Transcription vidéos/`). Vidéo Godot, moteur différent de GDevelop — ce qu'on en retient n'est jamais l'UI ou le GDScript (hors sujet, voir "Ce qui ne s'applique pas" plus bas), seulement la **grammaire de tuiles elle-même**, qui répond directement au blocage identifié le 2026-07-23 : Arnaud butait sur *"quel rôle doit jouer chaque tuile avant de générer"*, pas sur la génération elle-même (24 variantes de croix/T/coins générées presque au hasard, invendables à trier).

### Le principe : bitmask 3×3, mode "Match Corner Inside"

Un système d'autotile (Godot "Terrain", ou l'équivalent générique wang tiles/blob tileset) assigne à chaque tuile de la planche une position dans une grille de voisinage 3×3 (les 8 directions autour d'une case + le centre). Trois modes existent, du plus simple au plus complet :
- **Match sides** : ne regarde que les 4 côtés (N/S/E/O) — transitions correctes mais coins anguleux.
- **Match corners** : ajoute les diagonales, mais moins fiable sur les cas mixtes.
- **Match corner inside** (recommandé dans la vidéo comme le plus complet, malgré la complexité) : distingue un coin **extérieur** (convexe, ex. angle d'un chemin qui pointe vers l'herbe) d'un coin **intérieur** (concave, ex. l'herbe qui mord dans le chemin) — nécessaire pour des raccords propres dans les deux sens.

**Règle pratique retenue de la vidéo** : toujours inclure la case centrale du bitmask dans la sélection d'une tuile, sinon le moteur ne la prend pas en compte. Une tuile qui ne raccorde pas bien signale presque toujours un oubli de coin dans la sélection de la planche, pas une erreur de dessin.

### Le set minimal de rôles pour une transition à 2 matériaux (ex. herbe ↔ chemin)

C'est la réponse concrète au blocage : lister ces rôles **avant** de lancer un prompt de génération, jamais générer large puis trier après coup.

| Rôle | Description | Nb de tuiles |
|---|---|---|
| Base | Remplissage plein du matériau (herbe seule, ou chemin seul) | 1 par matériau |
| Bord droit ×4 | Transition rectiligne N, S, E, O | 4 |
| Coin extérieur ×4 | Angle convexe (le chemin "pointe" vers l'herbe) NE/NO/SE/SO | 4 |
| Coin intérieur ×4 | Angle concave (l'herbe "mord" dans le chemin) NE/NO/SE/SO | 4 |

Soit **14 tuiles** (2 bases + 4 bords + 4 coins extérieurs + 4 coins intérieurs) pour une transition propre à 2 matériaux en mode Match Corner Inside complet. Une version simplifiée (Match Sides ou Match Corners, sans distinction intérieur/extérieur) réduit à **10 tuiles** (2 bases + 4 bords + 4 coins non distingués, chaque coin partagé entre les deux rôles) — moins précis sur les raccords mais plus rapide à produire ; à choisir consciemment plutôt que par défaut.

**La différence entre les deux versions tient exactement aux 4 coins intérieurs en moins** : tout le reste (bases, bords) est identique. En version complète, un coin extérieur (le chemin pointe vers l'herbe, convexe) et un coin intérieur (l'herbe mord dans le chemin, concave) ont chacun leur propre dessin. En version simplifiée, le même dessin de coin sert aux deux situations — il doit alors être un compromis qui ne colle parfaitement à aucune des deux formes.

**Est-ce que moins de tuiles = plus de chances de bon résultat en génération IA ?** Partiellement, mais ce n'est pas la vraie raison du choix. Comme les 14 (ou 10) tuiles sont demandées en **un seul prompt combiné** (section 8), avoir 4 pièces de moins réduit un peu la charge que le modèle doit gérer simultanément dans une même image — un effet réel mais marginal, pas le facteur dominant. Le vrai compromis est **qualité visuelle des coins, pas taux de réussite de génération** : la vidéo source (section suivante, "Terminologie standard") confirme que la version réduite (son équivalent "16 pièces") produit des coins intérieurs visuellement "cassés" par défaut, à corriger en élargissant légèrement le dessin — donc la version courte ne garantit pas un meilleur résultat, elle déplace juste le travail de retouche vers l'étape Tileset Forge (section 8) plutôt que de l'éviter.

**Décision confirmée par Arnaud le 2026-07-23 : partir sur la version à 10 tuiles** — cohérent avec le principe "finir vite" du projet, la retouche manuelle est de toute façon une étape normale et déjà prévue (section 8), et à 32×32px un coin légèrement compromis est probablement peu visible en jeu. Réversible : passer à 14 seulement si les coins à 10 tuiles gênent visuellement une fois posés en jeu.

### Terminologie standard et pièges à connaître (complément vidéo, 2026-07-23)

**Origine** : transcription vidéo *"Draw fewer tiles - by using a Dual-Grid system"* (`Transcription vidéos/`, devlog Godot). Même principe que la source précédente : on retient le vocabulaire et les pièges de dessin, pas l'implémentation moteur.

Cette vidéo compare les systèmes standards du milieu (noms à connaître, utiles pour chercher/communiquer) et confirme que le set de la section précédente est du bon côté du compromis :

| Système | Nb de tuiles | Aligné à la grille du monde | Piège |
|---|---|---|---|
| 15 pièces (bords tracés au milieu de la tuile) | 15 | **Non** — un bord au milieu de la tuile ne colle pas à la grille : la tuile devient ambiguë (herbe ou chemin ?) et poser un seul carreau affecte visuellement une zone plus grande que prévu | Acceptable seulement pour du pur décor non-gameplay |
| 47 pièces (bords tracés près du bord de la tuile) | 47 | Oui | Beaucoup plus de tuiles à dessiner |
| 16 pièces (sous-ensemble du 47) | 16 | Oui | Coins intérieurs visuellement "cassés" sauf si on élargit légèrement le dessin des tuiles pour compenser |
| Dual-grid (grille logique + grille d'affichage décalée d'une demi-tuile) | 16 (2⁴ combinaisons de 4 voisins, au lieu de 2⁸=256 pour un autotile 8-voisins classique) | Oui, par construction | Nécessite un vrai moteur d'autotile au runtime (deux grilles superposées + calcul de voisinage), pas compatible avec une planche de tuiles statique |

**Règle à retenir pour nos tuiles dessinées à la main** : tracer les bords **près du bord de la tuile, jamais au milieu** — c'est ce qui garde l'alignement à la grille et évite l'ambiguïté du système "15 pièces". Notre set de 14/10 tuiles (section précédente) est déjà conçu dans cette famille (équivalent en esprit au 47/16 pièces, pas au 15) — aucun changement à faire au prompt de la section 8, cette vidéo confirme le choix plutôt qu'elle ne le remet en cause. Si on dessine un jour une version réduite façon "16 pièces" pour aller plus vite, penser à élargir légèrement les coins intérieurs au dessin pour éviter l'effet cassé signalé ici.

**Dual-grid non retenu maintenant** : la technique réduit le nombre de combinaisons à gérer à 16 en utilisant une grille logique invisible + une grille d'affichage décalée d'une demi-tuile, avec calcul des 4 voisins au runtime — implémentation vue dans la vidéo avec deux nodes TileMap Godot + script custom. C'est un vrai moteur d'autotile, donc incompatible avec la décision actuelle "zéro moteur d'autotile, placement à la main" (section 2). Noté ici comme option documentée à reconsidérer **seulement** si cette décision est rouverte (critère déjà acté section 2 : tuiles/transitions devenues ingérables à la main) — à ce moment-là, vérifier si le Tilemap natif de GDevelop permet une grille décalée et un calcul de voisinage équivalent avant de s'y engager.

### Piste explorée — PixelLab "Create Tileset" automatise cette grammaire (2026-07-23)

**Origine** : Arnaud a vu une vidéo courte montrant un artiste dessiner grossièrement des zones, PixelLab produisant le modèle pixel art correspondant — recherche menée sur la documentation officielle PixelLab (`pixellab.ai/docs`), détail complet dans `Desktop/The Long Ride/PROJECT/docs/actif/REFERENCE_PIXELLAB.md` section 6quater (référence partagée, pas dupliquée ici).

**Ce que ça change potentiellement pour Projet X** : PixelLab a un outil dédié **"Create Tileset"** qui prend en entrée une description du matériau "intérieur" et du matériau "extérieur" (ex. herbe entourée de chemin — exactement notre cas) et **génère automatiquement les bords/coins de transition**, avec export direct en Wang tileset / dual-grid 15-tileset / 3×3 tileset — les mêmes systèmes que ceux documentés ci-dessus, confirmant que ce ne sont pas des conventions propres à un moteur mais un vocabulaire standard. Ça automatiserait ce qu'on a dû faire à la main (définir la grammaire, écrire un prompt structuré, curer/retoucher le résultat en plusieurs passes) — pertinent directement pour la plainte d'Arnaud sur la génération actuelle ("mes tuiles ne sont pas vraiment adaptées les unes aux autres").

**Non tranché** : nécessite un abonnement PixelLab Tier 1 (12$/mois, séparé de Sorceress) — aucun test réel encore fait, qualité effective inconnue. Décision financière/essai qui reste à Arnaud.

### Grammaire complémentaire — tuiles de structures linéaires connectées (murs/clôtures) (2026-07-23)

**Origine** : transcription vidéo *"Tile Set Logic - Now You Know Too - Tutorial"* (`Transcription vidéos/`). Contenu générique, sans moteur ni outil précis — logique de tuiles pure, directement transposable.

**Portée différente de la grammaire ci-dessus** : les sections précédentes couvrent un **remplissage de zone à 2 matériaux** (herbe↔chemin). Celle-ci couvre autre chose : des **structures linéaires connectées** (mur, clôture) où chaque segment doit se raccorder à ses voisins dans un réseau, pas dans une zone. Pertinent le jour où Projet X a besoin d'une **clôture modulaire** (déjà citée comme structure dans la palette section 6) — pas un besoin de production actif aujourd'hui, seule la planche herbe/chemin est prévue en section 8.

**Rôles de tuiles pour un réseau de murs/clôtures d'1 tuile de large, en complexité croissante** :

| Palier | Rôles ajoutés | Ce qu'il couvre |
|---|---|---|
| 1 — Base | Sol/mur/plafond (ou juste mur en top-down) + coin extérieur ×4 + coin intérieur ×4 | Tous les murs/coins à angle 90° **d'au moins 2 tuiles de large** |
| 2 — Segments 1 tuile de large | Mur "2 côtés exposés" (segment droit fin) + mur "3 côtés exposés" (bout qui dépasse) | Murs/plateformes d'1 tuile de large en ligne droite |
| 3 — Raccords complexes | Coin intérieur double/triple, coin double opposé (segment fin qui rejoint une grande section) + coude, T, croisement à 4 branches (segment fin qui rejoint un autre segment fin) + tuiles de coin + leur inverse (segment fin qui rejoint le coin d'une grande section) | Réseau de murs fins qui se croisent/bifurquent, pas juste une ligne droite |
| Optionnel | Rampes, rampes 1 tuile de large | Pentes — combinaisons trop nombreuses pour être listées dans la vidéo source, à traiter au cas par cas si besoin |
| Cas extrêmes | Tuile "toute intérieure" (aucune face exposée — jamais visible, pur remplissage structurel) et tuile "toutes faces exposées" (poteau isolé, ex. un piquet de clôture seul) | Les deux bornes du système |

**Application concrète si/quand une clôture modulaire est produite** : commencer au palier 1 (suffisant si la clôture ne forme jamais de segment isolé d'1 tuile), ajouter le palier 2 seulement si des tronçons fins existent (ex. portillon entre deux poteaux), le palier 3 seulement si le réseau bifurque (angle en T ou croisement). Ne pas produire tous les paliers par anticipation — cohérent avec le principe directeur "finir vite" (`CLAUDE.md` du projet).

### Ce qui ne s'applique PAS à Projet X (spécifique Godot, hors périmètre)

- L'outil "Terrain" de Godot lui-même (UI TileSet, bitmask cliquable, mode paint automatique) — Projet X n'a **aucun moteur d'autotile actif** (décision section 2 : pas de Tiled, pas d'External Tilemap, intégration directe en plaçant des Sprites/TiledSprite à la main dans GDevelop). Cette grammaire sert donc à **préparer la planche de tuiles à l'avance** (savoir quoi demander à Sorceress/Nano Banana, quoi nommer, combien de variantes), pas à configurer un système équivalent dans GDevelop.
- Le fix de jitter caméra (GDScript, `physics_process`, arrondi de position) — spécifique Godot, sans équivalent direct nécessaire ici.
- L'édition de collision polygonale tuile par tuile (statues, tombes) — process fastidieux par essai-erreur, confirmé pénible même par l'autrice de la vidéo ("il doit exister un truc mieux"). Retenu comme avertissement (prévoir du temps, ne pas viser la perfection au premier essai), pas comme méthode à reproduire telle quelle — GDevelop a son propre éditeur de hitbox, à traiter le moment venu.
- Le Y-sort Godot (`ordering > canvas item ordering`, `Y Sort Enable`) confirme seulement le principe déjà acté dans `REGLES_PROFONDEUR_CALQUES_2D.md` (TLR) — aucune information nouvelle, pas de reprise nécessaire.

### Application immédiate — prochaine génération

Pour la planche "herbe + chemin" déjà prévue en section 8 (prompt concret) : compléter la liste de tuiles demandées avec les rôles ci-dessus nommés explicitement (base herbe, base chemin, 4 bords, 4 coins extérieurs, 4 coins intérieurs — ou la version 10-tuiles simplifiée avec coins partagés, **à trancher par Arnaud avant le prochain prompt**) plutôt que de laisser le modèle produire un nombre arbitraire de variantes.

## 5. Méthode de génération complémentaire — inpainting progressif tuile par tuile (2026-07-23)

**Origine** : transcription vidéo *"Tutorial: How to create tilesets using AI"* (`Transcription vidéos/`), outil **PixelLab.ai** (confirmé — nom mal transcrit en "pixel app" dans la source).

**Vérifié le 2026-07-23 (recherche web, sources en bas de section) : Sorceress n'a PAS d'équivalent.** Ses deux outils les plus proches ont été inspectés directement :
- **Image Expander** (Sorceress) : outpainting seul (étend le canevas autour d'une image existante). Confirmé sans sélection/masque de région, sans réglage de force/influence, sans remplissage automatique des zones transparentes internes — donc **incapable** de faire ce que la vidéo appelle inpainting entre deux tuiles déjà posées.
- **Canvas** (Sorceress) : décrit uniquement comme "éditeur d'image rapide dans le navigateur" (palier gratuit) — un éditeur manuel, pas un outil d'inpainting IA confirmé.

**Le vrai "Create Map" de la vidéo existe bien, mais chez PixelLab.ai directement, pas dans Sorceress** : `REFERENCE_PIXELLAB.md` section 5 confirme une fonctionnalité "génération de cartes" à partir du palier payant **Tier 1 — Pixel Apprentice (12$/mois)**, absente de l'essai gratuit et absente de Sorceress Pro (49$ à vie, déjà payé par Arnaud). PixelLab confirme aussi par ailleurs (section 2 du même document) un principe d'inpainting itératif similaire sur son outil Rotate — cohérent avec la méthode montrée dans la vidéo.

**Conséquence pratique** : cette méthode reste documentée ci-dessous pour sa valeur de principe (utile si Arnaud prend un jour un abonnement PixelLab séparé), mais **n'est pas utilisable avec les outils actuels d'Arnaud**. Le vrai filet de sécurité si le prompt unique de la section 8 ne produit pas des tuiles propres, c'est la **retouche manuelle pixel par pixel dans Tileset Forge** (déjà inclus dans Sorceress Pro, déjà documenté dans `REFERENCE_SORCERESS_GAMES.md` section 12bis : agrandir/rétrécir de 1px, clone stamp/gomme/pinceau) — voir le renvoi en section 8.

### Le principe : construire la planche pièce par pièce, jamais tout d'un coup

Contrairement au prompt structuré de la section 8 (une génération → toute la planche), cette méthode construit le set de tuiles progressivement :
1. Générer une seule tuile de bonne qualité en premier (avec un autre modèle si l'outil d'inpainting lui-même n'est pas fiable pour la toute première image).
2. Créer les variantes une par une (l'inverse, les bords, les coins) en copiant-collant les morceaux déjà validés dans la bonne position avant de générer la suite — **plus le modèle voit de contexte déjà cohérent, mieux il copie le style**.
3. Utiliser une "image d'initialisation" (sketch grossier, pas besoin d'être propre) + une force réglable (~200-500) pour guider la génération d'une transition ou d'un coin spécifique, plutôt que de laisser le modèle deviner à l'aveugle.
4. Astuce d'inversion : sélectionner la zone et inverser la sélection (valeur négative) pour générer rapidement le pendant inverse d'une tuile déjà faite (ex. dirt-dans-herbe → herbe-dans-dirt) — utile pour produire les coins intérieurs/extérieurs de la grammaire (section 4) sans repartir de zéro à chaque fois.
5. Les coins/transitions sont **la partie la plus manuelle et itérative** — confirmé dans la vidéo comme *"the hardest part"*, nécessitant plusieurs allers-retours de retouche/inpainting, pas une génération en un clic. Cohérent avec le principe déjà acté section 8 : la retouche manuelle est une étape attendue, pas un échec.
6. Si le résultat d'une zone inpaintée ne convient pas (ex. transition trop dure au lieu d'une pente douce), **élargir la zone inpaintée** pour que le modèle voie plus de contexte plutôt que de simplement relancer la même zone.

### Cas pratique montré : plateforme side-scroller (hors périmètre, noté pour référence)

La vidéo montre aussi la même méthode pour un tileset de jeu de plateforme (ciel/herbe/terre, pièces "en l'air" avec trou exposant le ciel, piliers, pente). Projet X est top-down, pas plateforme — **non applicable tel quel**, mais le principe (pièce du milieu déductible du contexte au-dessus/en-dessous sans image d'initialisation, coins/trous nécessitant un contexte visible du fond) reste le même raisonnement que pour le top-down. Noté pour référence si un mini-jeu ou une scène annexe en side-scroller apparaissait un jour — hors scope V1 actuel.

**Sources de la vérification Sorceress vs PixelLab (2026-07-23)** :
- [Sorceress Editor — page Expander](https://sorceress.gamemaker.academy/expander) — description directe de l'outil, confirme outpainting seul.
- [sorcgcs.com](https://sorcgcs.com/) — page produit Sorceress, description "Canvas" comme éditeur d'image rapide (palier gratuit).
- [Sorceress — AI Game Creation Suite](https://sorceress.games/) — confirme "Expander" pour l'outpainting dans la suite.
- `REFERENCE_PIXELLAB.md` (TLR, sections 2 et 5) — confirme l'inpainting et la fonctionnalité "génération de cartes" côté PixelLab.ai, réservée au Tier 1 payant (12$/mois).

## 6. Palette — candidat extrait de la référence d'Arnaud (2026-07-23, NON validé)

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

## 7. Persona de génération — "Concepteur de tiles — Projet X" (2026-07-23)

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

## 8. Premier prompt concret — planche de tuiles de sol (2026-07-23)

Cible : famille de tuiles interconnectées (herbe de base + chemin de terre avec bords/coins + une tuile de pavés de place), plutôt qu'une texture unique — cohérent avec l'apprentissage TLR sur Tileset Forge (`REFERENCE_SORCERESS_GAMES.md` section 12bis : un prompt structuré "taille + type de tuile + style d'environnement" produit une planche de tuiles apparentées en une génération, contrairement à un prompt "seamless repeating texture"). Modèle recommandé par ce même apprentissage TLR : **Nano Banana 2** si disponible dans l'outil utilisé, sinon Nano Banana Pro.

Rôles demandés explicitement (grammaire de la section 4, **version confirmée à 10 tuiles**, 2026-07-23) : 1 base herbe, 1 base chemin, 4 bords droits (N/S/E/O), 4 coins partagés (même dessin utilisé aussi bien pour un angle convexe que concave) — pas de génération "au hasard" à trier après coup.

```
32x32 pixel art tiles for a top-down medieval-fantasy village game. Generate a connected tile family on a flat magenta (#FF00FF) background, with each tile clearly separated on a grid. Include, as distinct labeled tiles: (1) a solid grass base tile, (2) a solid dirt path base tile, (3) 4 straight edge tiles where grass transitions to path on one side only (top, bottom, left, right), (4) 4 corner tiles showing a diagonal grass-to-path transition, designed to work for both convex and concave corner placements, and (5) one cobblestone plaza tile. Warm, sunlit, welcoming medieval village atmosphere. Top-left lighting, no baked drop shadows, flat shading only. Strict color palette: #F0C878, #F0B478, #DCA078, #DCB478, #C8A078, #C8A064, #8C8C50, #78783C, #A0A050, #64643C, #505028, #3C3C28, #B4A08C, #A08C78, #C8B4A0, #786450, #503C28, #A08C64, #B48C64, #A06450, #785050, #643C3C, #502828, #3C2828. No dark outline on edges meant to connect between tiles.
```

**Après génération** : passer la planche dans **Tileset Forge** (Detect → Build) pour isoler/nettoyer chaque tuile individuellement et confirmer que les raccords herbe↔chemin sont propres, avant tout export vers `game/assets/`. Une tuile qui ne se raccorde pas proprement se régénère ou se retouche à ce stade — pas un échec de méthode (`REFERENCE_SORCERESS_GAMES.md` : "la curation et la retouche manuelle est une étape attendue, pas un échec de l'outil"). **Attention particulière aux 4 coins partagés** (section 4) : c'est l'endroit le plus probable où un léger élargissement/retouche sera nécessaire, puisque le même dessin sert à un angle convexe et concave.

**Si le prompt unique ne suffit pas** pour un rôle précis de la grammaire (coin qui ne raccorde pas, transition trop dure) : retoucher ce rôle manuellement dans **Tileset Forge** à l'étape Build (clone stamp/gomme/pinceau, agrandir/rétrécir de 1px) — c'est le filet de sécurité réellement disponible avec les outils actuels (voir section 5, la méthode d'inpainting progressif PixelLab n'est **pas** utilisable en l'état, outil séparé et payant).

### Premier essai réel — résultat et enseignement (2026-07-23)

**Résultat** : 8 tuiles utilisables sur les 10 core + 1 bonus demandées — Solid Grass, Solid Dirt, bords Left/Right, coins Top-Left/Top-Right/Bottom-Right, Cobblestone Plaza. Qualité correcte sur ces 8 : palette respectée, éclairage cohérent, bords tracés près du bord de la tuile (pas au milieu, conforme à la règle section 4).

**Manque** : bord Top, bord Bottom (la case reçue à leur place montre de l'herbe des deux côtés d'une bande de chemin — ne correspond à aucun rôle demandé), coin Bottom-Left.

**Défaut identifié — cause probable** : environ la moitié de la planche (7 cases/16) est constituée de swatches de couleur pure affichant un code hex, pas des tuiles. La liste de codes hex dans le prompt ("Strict color palette: #F0C878...") a été interprétée par le modèle comme des éléments à représenter littéralement, cassant le pattern de liste numérotée des tuiles. **Cette liste est déjà redondante** : la persona système (section 7) contient la palette complète — pas besoin de la répéter dans le prompt de génération.

**Prompt de complément recommandé** (3 pièces manquantes uniquement, sans liste de hex codes) :
```
32x32 pixel art tiles for a top-down medieval-fantasy village game, matching the exact style, palette and lighting already established for this project. Generate on a flat magenta (#FF00FF) background, with each tile clearly separated on a grid. Include ONLY these 3 distinct labeled tiles, nothing else — no color swatches, no palette reference images: (1) a straight edge tile with grass only on the top side, dirt path filling the rest, (2) a straight edge tile with grass only on the bottom side, dirt path filling the rest, (3) a diagonal corner tile where grass meets path at the bottom-left corner. Top-left lighting, no baked drop shadows, flat shading only. No dark outline on edges meant to connect between tiles.
```

**Recommandation** : ne pas régénérer toute la planche — les 8 tuiles obtenues sont bonnes et cohérentes entre elles. Lancer ce prompt de complément, puis tout passer ensemble dans Tileset Forge.

**Résultat du prompt de complément (2026-07-23)** : plus aucune contamination par des swatches de couleur — le retrait de la liste de hex codes a corrigé le défaut identifié plus haut. Les 2 bords manquants (Top, Bottom) sont bien obtenus, mais chacun livré à côté d'une tuile de terre pleine redondante (non demandée) — à écarter simplement au tri dans Tileset Forge, pas un problème. Le coin Bottom-Left reçu **est visuellement proche d'un Top-Left** (herbe concentrée en haut-gauche plutôt qu'en bas-gauche) — à vérifier côte à côte avec les 3 coins déjà obtenus avant de l'accepter tel quel. **Alternative plus sûre que de faire confiance à ce 4e coin généré** : dériver le Bottom-Left en retournant (flip horizontal ou vertical, selon l'orientation) l'un des 3 coins déjà validés (Top-Left, Top-Right ou Bottom-Right) dans Tileset Forge — garantit un raccord pixel-perfect avec le reste du set plutôt que de risquer une incohérence de style entre coins.

**Inventaire à ce stade** : 8 tuiles de la 1ère génération + 2 bords (Top, Bottom) de la 2e génération = 10/10 rôles core couverts (+ Cobblestone Plaza en bonus), sous réserve de trancher le coin Bottom-Left (accepter le généré après vérification visuelle, ou le dériver par flip). Prochaine étape : tout regrouper dans Tileset Forge (Detect → Build), écarter les doublons de terre pleine, trancher le coin, exporter vers `game/assets/`.

### Export final Tileset Forge (2026-07-23)

**Décision d'Arnaud** : la tuile "terre battue" de la toute dernière génération (texture de terre non uniforme, différente du grain des générations précédentes) a été **écartée volontairement** — incohérente avec le reste du set plutôt qu'un défaut à retoucher. Bon réflexe, cohérent avec le principe déjà documenté : la cohérence visuelle prime sur la complétude, la curation manuelle est une étape normale.

**Export réel obtenu** (`tileset_individual (2).zip` + `(3).zip`, dossier `tiles/` de chaque archive — une carte normale `normals/` accompagne chaque tuile, non utilisée actuellement, Projet X n'a pas de système d'éclairage dynamique 2D) :

| Fichier original | **Renommé et importé dans `game/assets/` (2026-07-23)** | Rôle |
|---|---|---|
| `tile_0_1.png` | `sol_herbe.png` | Base herbe |
| `tile_2_1.png` (zip 2) | `sol_chemin.png` | Base chemin |
| `tile_4_1.png` (zip 2) | `sol_bord_haut.png` | Bord Top (herbe haut, chemin bas) |
| `tile_6_1.png` | `sol_bord_bas.png` | Bord Bottom (chemin haut, herbe bas) |
| `tile_0_3.png` | `sol_bord_gauche.png` | Bord Left (herbe gauche, chemin droite) |
| `tile_8_1.png` | `sol_bord_droite.png` | Bord Right (herbe droite, chemin gauche) |
| `tile_0_5.png` | `sol_paves.png` | Cobblestone Plaza (bonus) |
| `tile_2_1.png` (zip 3) | `sol_coin_a.png` | Coin diagonal A — herbe haut-gauche / chemin bas-droite |
| `tile_4_1.png` (zip 3) | `sol_coin_b.png` | Coin diagonal B — herbe bas-gauche / chemin haut-droite |

**Sur les coins — seulement 2 fichiers au lieu de 4, et c'est suffisant** : les 2 coins obtenus sont les 2 seules diagonales possibles pour un raccord plein-tuile (herbe/chemin coupé en deux par une diagonale coin-à-coin). Les 2 autres orientations de coin ne sont pas des dessins différents — ce sont ces 2 mêmes tuiles **retournées** (flip horizontal ou vertical). Pas besoin de les régénérer : GDevelop permet de retourner un Sprite/TiledSprite à la pose (propriété miroir horizontal/vertical de l'objet) — poser le même fichier avec un flip donne l'orientation manquante, pixel-perfect garanti puisque c'est littéralement le même art. **9 fichiers image couvrent donc les 10 rôles + le bonus.**

**Étape 3 terminée** : les 9 fichiers sont dans `game/assets/` avec des noms explicites. Prochaine étape (Étape 4, section 0) : créer les objets Sprite dans GDevelop et les placer dans `Niveau1_Village`.

**⚠️ Noms de fichiers non parlants** : `tile_0_1.png`, `tile_2_1.png` etc. sont des coordonnées de grille Tileset Forge, pas des noms de rôle — les deux archives réutilisent en plus les mêmes noms (`tile_2_1.png`, `tile_4_1.png`) pour des rôles différents (base chemin dans un zip, coin dans l'autre). **Renommer selon le rôle avant d'importer dans `game/assets/`** (ex. `sol_herbe.png`, `sol_chemin.png`, `sol_bord_haut.png`, `sol_coin_a.png`...) pour éviter toute confusion/écrasement au moment de l'import GDevelop.

### Objets GDevelop créés (2026-07-23)

**Méthode confirmée par Arnaud** : je crée les objets Sprite (mécanique, aucune décision de design), Arnaud place les instances à la main dans l'éditeur (cohérent avec la décision déjà actée section 2 — placement manuel, pas de moteur d'autotile). Zone cible confirmée : toute la scène nominale 56×32 tuiles.

9 objets Sprite créés dans le dossier `Décor` (voir `docs/structure_json_projet_x.md`) : `SolHerbe`, `SolChemin`, `SolBordHaut`, `SolBordBas`, `SolBordGauche`, `SolBordDroite`, `SolCoinA`, `SolCoinB`, `SolPaves` — chacun avec son image correspondante de `game/assets/`, origine (16,16) comme le reste des objets du projet (position d'instance = centre de la tuile, pas son coin). **Aucune instance placée** — les objets existent dans l'éditeur, prêts à glisser-déposer, le placement du chemin/herbe reste à faire par Arnaud.

**Pour les 2 orientations de coin manquantes** (section précédente) : dans l'éditeur GDevelop, sélectionner l'instance de `SolCoinA` ou `SolCoinB` posée, utiliser le retournement horizontal/vertical (propriété de l'objet dans le panneau des propriétés) pour obtenir les 2 orientations restantes sans créer de nouveaux objets.

### Bug de résolution des ressources — corrigé (2026-07-23)

Après création des 9 objets Sprite, rendu cassé dans l'éditeur (placeholder violet "G" de GDevelop = ressource introuvable). **Cause racine** : le champ `image` d'un objet Sprite référence un fichier par son nom, mais GDevelop le résout via le manifeste global `resources` en tête du JSON — une entrée par fichier (`file`, `kind`, `name`...), distincte de la référence dans l'objet. Les 9 fichiers avaient été copiés dans `game/assets/` et référencés dans les objets, mais jamais déclarés dans `resources`. Corrigé (9 entrées ajoutées). Réflexe noté dans le skill `gdevelop-operations` pour ne pas reproduire l'oubli. **GDevelop ne relit pas le JSON en continu** — fermer/rouvrir le projet dans l'éditeur pour voir l'effet d'une édition JSON directe faite pendant qu'il est ouvert.

### Fond d'herbe automatique — tentative SolHerbeFond, retirée (2026-07-23)

Première tentative : un objet **`SolHerbeFond`** (type `TiledSpriteObject::TiledSprite`, texture `sol_herbe.png` répétée automatiquement) couvrant toute la scène — évitait de placer ~1792 tuiles une par une, cohérent avec la décision section 3 (le sol pur n'a besoin d'aucun Y-sort ni collision). **Problème découvert par Arnaud à l'usage** : ce fond ne partage aucune grille avec l'objet `TileMap::SimpleTileMap` utilisé pour peindre le chemin (deux systèmes de rendu différents) — les tuiles peintes ne s'alignaient pas visuellement avec le fond. **Retiré** (objet + instance supprimés), remplacé par la solution de la section suivante.

### Tilemap natif pour le chemin — atlas assemblé (2026-07-23)

Arnaud a créé un objet `NewTileMap` (`TileMap::SimpleTileMap`) dans l'éditeur pour peindre le chemin au pinceau plutôt qu'à la main tuile par tuile — reprise anticipée de l'option de repli déjà documentée (section 4, "Dual-grid non retenu maintenant", même logique de réouverture conditionnelle).

**Assemblage de la planche** : cet objet attend une seule image "atlas" (toutes les tuiles rangées en grille), pas 9 fichiers séparés comme on les avait. Planche `game/assets/sol_atlas.png` générée par script PowerShell (`System.Drawing`, composite à partir des 9 fichiers individuels, transparence conservée) — grille 3×3, tuile 32×32 :

| Position (col,row) | Tuile |
|---|---|
| (0,0) | sol_herbe.png (base herbe) |
| (1,0) | sol_chemin.png (base chemin) |
| (2,0) | sol_paves.png (pavés) |
| (0,1) | sol_bord_haut.png |
| (1,1) | sol_bord_bas.png |
| (2,1) | sol_bord_gauche.png |
| (0,2) | sol_bord_droite.png |
| (1,2) | sol_coin_a.png |
| (2,2) | sol_coin_b.png |

**Objet `NewTileMap` configuré** : `atlasImage: "sol_atlas.png"`, `rowCount: 3`, `columnCount: 3`, `tileSize: 32`, `tilesWithHitBox: ""` (pas de collision — cohérent avec le sol qui n'en a jamais besoin, section 3). Schéma confirmé via l'objet qu'Arnaud avait créé lui-même dans l'éditeur (voir skill `gdevelop-operations`).

**Reste à faire par Arnaud** : recharger le projet, ouvrir l'éditeur de tilemap sur l'instance (double-clic ou bouton "Éditer"), peindre le chemin avec la palette 3×3 issue de l'atlas. Les 9 objets Sprite individuels (`SolChemin`, `SolCoinA`...) restent disponibles en parallèle si un placement ponctuel hors tilemap est plus simple à un endroit précis (ex. la tuile pavés en un point isolé).

### Correction alignement + calque + herbe via tilemap (2026-07-23)

Trois problèmes remontés par Arnaud après un premier test en jeu, corrigés ensemble :

1. **Désalignement** : l'instance `NewTileMap` qu'Arnaud a placée était en (105, 631) — pas un multiple de 32. Un plan de tuiles prend sa position d'instance comme origine de sa propre grille interne ; un décalage par rapport à un multiple de 32 désaligne tout le plan par rapport au reste du monde. Recalée à (96, 640) (multiples de 32 les plus proches) — le contenu déjà peint (colonne de 8 tuiles) est conservé, seule la position de l'instance a bougé.
2. **Tiles cachant les objets déjà posés** : tout était sur le calque de base (`""`), l'ordre dépendait uniquement du zOrder — fragile. Un calque **`Sol`** a été créé, positionné **avant** le calque de base dans l'ordre de rendu (donc toujours derrière). Les 2 instances `NewTileMap` (herbe + chemin) sont sur ce calque ; tous les objets de gameplay restent sur le calque de base, donc toujours dessus, quel que soit leur zOrder individuel.
3. **Vraie tuile d'herbe "comme prévu"** : plutôt que `SolHerbeFond` (`TiledSprite`, système de rendu différent, cause du désalignement), l'herbe est maintenant posée via une **2ᵉ instance du même objet `NewTileMap`** — un remplissage uni de la tuile d'index 0 de l'atlas (`sol_herbe.png`) sur une grille 56×32 couvrant toute la scène nominale, positionnée en (0,0) (déjà un multiple de 32). Comme c'est le même objet/atlas que le chemin, herbe et chemin partagent exactement la même grille — alignement garanti par construction, plus de risque de décalage entre les deux.

**Détail technique (schéma du plan de tuiles par instance)** découvert en décodant le JSON de l'instance qu'Arnaud avait peinte à la main, confirmé par lecture du code moteur puis vérification visuelle — noté dans le skill `gdevelop-operations` pour ne pas le re-découvrir la prochaine fois : la valeur stockée dans la grille `tiles` d'une instance est **l'index brut de la tuile dans l'atlas** (numérotation ligne par ligne à partir de 0, `-1` = case vide) — pas de décalage +1 sur le fichier JSON du projet (ce décalage n'existe qu'en mémoire côté moteur une fois chargé).

**Bords de tuiles imparfaits** (visibles en jeu par endroits) : laissés tels quels — Arnaud les corrige lui-même via le bouton "Éditer avec Piskel" de l'éditeur d'atlas, pas une action à faire côté JSON.

## Index (tags)
#PipelineDecor #ProjetX #Sorceress #NanoBananaPro #TilesetForge #PaletteCouleurs #YSort #ModulaireVsImagePosee #GrammaireAutotile #GrammaireCloture #DualGrid

# Structure JSON — Projet X (GDevelop)

Guide de navigation dans `PROJECT/game/Projet X.json`. Consulter avant toute recherche dans le JSON pour cibler directement le bon groupe plutôt que de re-deviner à chaque session.

> Version scalée à la taille réelle du projet (une scène, groupes à plat) — pas l'arborescence profonde de TLR (`docs/structure_json_tlr.md`), qui gère une machine à états imbriquée sur plusieurs niveaux. À enrichir si Projet X grossit (plusieurs scènes, sous-groupes imbriqués).

---

## Règles

- Ne jamais parser tout le fichier — cibler la scène + le groupe exact.
- L'ordre des groupes est **critique** (un groupe plus bas peut écraser ce qu'un groupe plus haut vient de poser).
- Scripts (Python/Node/PowerShell) autorisés seulement pour valider ou extraire une zone déjà ciblée — jamais pour explorer à l'aveugle, jamais pour écrire (écriture toujours via édition directe du JSON).
- **Tenir ce fichier à jour** : tout ajout, déplacement ou renommage de groupe/objet doit être reporté ici dans la foulée — sinon le doc devient trompeur, pire que pas de doc du tout.

---

## Scène unique : `Niveau1_Village`

### Objets (par dossier)
- `Personnages` : Necromancien (variable `PV`, défaut 100 — pas encore de méta-progression dessus)
- `Invocations` : Gobelin
- `Ennemis` : Villageois (variable `GobelinBloque` : recalculée chaque frame, 1 = un Gobelin est au contact (40px) — sert de repli avant d'attaquer le Nécromancien), Horde (variable `CooldownAttaque` — unité de la horde finale, foncent direct sur le Nécromancien, PV/dégâts non simulés côté Horde : objet volontairement non attaquable, incarne "impossible à combattre")
- `Décor` : Sortie (image `decor_placeholder.png` depuis le fix du 2026-07-23 — `sortie_placeholder.png` référencé mais jamais créé ni déclaré en resource, bug préexistant découvert et corrigé), **NewTileMap** (objet `TileMap::SimpleTileMap`, atlas `sol_atlas_pixellab.png`, 4 colonnes × 8 lignes = 32 tuiles, 2 tilesets Wang chaînés empilés : lignes 0-3 herbe↔chemin, lignes 4-7 herbe↔pavés, même tuile d'herbe des deux côtés — voir section suivante pour le détail des index), **PropPuits, PropTonneau, PropEtalMarche** (PixelLab `create_map_object`, mode basique, `view: low top-down` — origine bas-centre (32,64) pour puits/étal 64×64, (16,32) pour tonneau 32×32). ⚠️ **RepereDecor et les 9 objets Sorceress (SolHerbe, SolChemin, SolBordHaut/Bas/Gauche/Droite, SolCoinA/B, SolPaves) supprimés le 2026-07-23** (session 8, obsolètes — remplacés par `NewTileMap`+atlas PixelLab), fichiers PNG associés + ancien `sol_atlas.png` retirés du disque et des resources.
- `Bâtiments` : **MaisonPierre, MaisonColombage, MaisonBeigeFleurs** — 3 variantes de maison pré-composées en une seule image chacune (PixelLab `create_map_object`, mode basique, 64×64, `view: low top-down`), remplace l'approche modulaire `create_building_kit` abandonnée (voir `PIPELINE_DECOR.md`). Origine bas-centre (32,64).

### Scène complète v1 — instances placées (2026-07-23, session 8)

Réseau de sol + décor conçu et peint entièrement par Claude Code (script PowerShell utilisant les métadonnées exactes des tilesets Wang pour un placement de tuiles correct, pas de devinette). Layout : route principale est-ouest (tuiles lignes 16-17, colonnes 13-40) avec une branche sud vers le cluster2 de Villageois (colonnes 22-23, lignes 16-21) ; une place pavée au nord de la route (colonnes 21-33, lignes 10-13, bordée) contenant le puits, le tonneau et l'étal de marché.

| Objet | Position (px) | Repère |
|---|---|---|
| `MaisonPierre` | (1150, 420) | Est de la place, proche du cluster1 de Villageois |
| `MaisonColombage` | (600, 720) | Bout de la branche sud, proche du cluster2 |
| `MaisonBeigeFleurs` | (420, 470) | Bout ouest de la route principale |
| `PropPuits` | (890, 370) | Centre de la place |
| `PropTonneau` | (750, 400) | Place, côté ouest |
| `PropEtalMarche` | (1020, 400) | Place, côté est |

Index de tuiles utiles dans `sol_atlas_pixellab.png` (raw index = ligne×4+colonne) : **12** = herbe pure (bloc chemin), **6** = chemin pur, **28** = herbe pure (bloc pavés, art identique à 12), **22** = pavés pur — le reste de la grammaire (bords/coins) se choisit visuellement dans l'éditeur de tilemap. Aperçus générés dans `game/assets/apercu_scene_sol.png` (sol seul) et `apercu_scene_complete.png` (avec maisons/props) — utiles pour visualiser sans ouvrir GDevelop.

**Prochaine session, priorités déjà identifiées par Arnaud (retours à venir)** : Base/Sommet+YSort pas encore fait sur les maisons (le Nécromancien passe toujours devant, jamais derrière) ; variété d'assets à étoffer selon les retours d'Arnaud (arbres, clôtures, autres bâtiments).

### Calques (layers), dans l'ordre de rendu (2026-07-23)

| Ordre (bas→haut) | Nom | Contenu |
|---|---|---|
| 1 (fond) | `Sol` | Les 2 instances `NewTileMap` (herbe + chemin) — jamais d'objet de gameplay ici |
| 2 | `` (base) | Tous les objets de gameplay/décor (Necromancien, Villageois, Gobelin, RepereDecor, Sortie, UI...) — rendu au-dessus du Sol quel que soit le zOrder individuel |
| 3 (dessus) | `Debug` | Textes de debug (`TxtMana`, `TxtCooldown`, `TxtOrbes`) |

Un objet placé sur un calque donné s'affiche toujours au-dessus de tout ce qui est sur un calque antérieur dans cette liste, indépendamment du zOrder — le zOrder ne départage qu'entre objets du **même** calque.
- `UI` : TxtMana, TxtCooldown, TxtOrbes, TxtSortieStatut, TxtNiveauTermine, TxtPVNecro, TxtGameOver, TxtFenetreSortie
- `Effets` : TxtDegats, TxtAggro

### Groupes d'événements, dans l'ordre d'exécution

| # | Groupe | Rôle |
|---|---|---|
| 1 | `Caméra` | Caméra qui suit le Nécromancien |
| 2 | `Déplacement Nécromancien` | ZQSD → TopDownMovementBehavior |
| 3 | `Régénération Mana` | +5/sec plafonné à ManaMax |
| 4 | `Invocation Gobelin` | Clic gauche → spawn Gobelin (coût mana + cooldown) |
| 5 | `Ordre d'attaque` | Clic droit → cible un groupe de Villageois (`Ciblé`, `GroupeID`) |
| 6 | `Combat Gobelin` | Gobelin attaque sa cible (ForEach+PickNearest), dégâts, mort Villageois → **Orbes+1**, texte de dégâts |
| 7 | `Combat Villageois` | Villageois engagé (`GroupeAuCombat`=1) vise le Nécromancien en priorité, riposte contre un Gobelin qui le bloque au contact (`GobelinBloque`), mort du Gobelin |
| 8 | `Vibration Gobelin` | Oscillation visuelle du Gobelin en combat |
| 9 | `Vibration Villageois` | Idem côté Villageois |
| 10 | `Aggro Villageois` | Détection de proximité → `Aggro`, icône "!" |
| 11 | `Fuite Villageois` | Groupe aggro (et pas encore engagé) fuit en bloc |
| 12 | `Debug/HUD` | Textes `TxtMana`/`TxtCooldown`/`TxtOrbes` (calque Debug) |
| 13 | `IA Gobelin` | Retour à la horde vers le Nécromancien, anti-blocage, anti-chevauchement |
| 14 | `Séparation Villageois` | Anti-chevauchement Villageois/Villageois et Villageois/Nécromancien |
| 15 | `Vagues Niveau 1` | Spawn de 7 Villageois à 8 orbes (rejoint GroupeID 1) et 7 à 16 orbes (rejoint GroupeID 2), tous avec `GroupeAuCombat`=1 dès l'apparition (déjà "en mode combat", même règle que les clusters de base) |
| 16 | `Game Over` | Déclenchement du Game Over (texte `TxtGameOver`) quand PV Nécro ≤ 0 |
| 17 | `Sortie Niveau` | Déblocage sortie au seuil d'orbes, statut, fin de niveau |
| 18 | `Fenêtre de Sortie` | Au déblocage : démarre le timer 10s (`FenetreSortieActive`/`FenetreSortieTemps`) + vague ponctuelle de 6 Villageois près de la Sortie (`GroupeAuCombat`=1 direct). Décompte + affichage `TxtFenetreSortie`. À expiration sans `NiveauTermine` : spawn de 10 `Horde` en cercle autour du Nécromancien |
| 19 | `Combat Horde` | Chaque Horde fonce sur le Nécromancien (pas de ForEach, cible toujours singleton) et l'attaque au contact (40px) — mêmes dégâts/cooldown/texte que `Combat Villageois`, en plus violent (`DegatsHorde`=40, cooldown 0.5s) |

---

## Raccourcis fréquents

| Besoin | Aller à |
|---|---|
| Ajouter/ajuster les dégâts, PV, mort d'une unité | `Combat Gobelin` / `Combat Villageois` |
| Ajuster mana, coût, cooldown d'invocation | `Régénération Mana` / `Invocation Gobelin` |
| Ciblage, propagation par groupe (`Ciblé`, `GroupeAuCombat`, `GroupeXAggro`) | `Ordre d'attaque`, `Combat Gobelin` (trigger), `Aggro Villageois`, `Fuite Villageois` |
| Orbes, seuil, sortie de niveau | `Sortie Niveau` |
| Variables globales de scène | tête du fichier, tableau `variables` (avant `layouts`) |
| Définitions d'objets (Sprite/Text, variables d'objet) | tableau `objects` de la scène, après `instances` |

---

## Variables globales actuelles

`Mana`, `ManaMax`, `CoutGobelin`, `CooldownGobelin`, `CooldownGobelinDuree`, `CibleActive`, `DegatsGobelin`, `CooldownAttaqueGobelinDuree`, `GroupeCible`, `NbCiblesGroupe`, `DegatsVillageois`, `CooldownAttaqueVillageoisDuree`, `Groupe1Aggro`, `Groupe2Aggro`, `Groupe1FuiteDX/DY`, `Groupe2FuiteDX/DY`, `Groupe1Combat`, `Groupe2Combat`, `Orbes`, `OrbesSeuil`, `SortieDeverrouillee`, `NiveauTermine`, `Vague1Faite`, `Vague2Faite`, `GameOver`, `FenetreSortieActive`, `FenetreSortieTemps`, `FenetreSortieDuree`, `DegatsHorde`, `CooldownAttaqueHordeDuree`.

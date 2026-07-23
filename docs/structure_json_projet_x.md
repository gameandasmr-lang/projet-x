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
- `Ennemis` : Villageois (variable `GobelinBloque` : recalculée chaque frame, 1 = un Gobelin est au contact (40px) — sert de repli avant d'attaquer le Nécromancien)
- `Décor` : RepereDecor, Sortie
- `UI` : TxtMana, TxtCooldown, TxtOrbes, TxtSortieStatut, TxtNiveauTermine, TxtPVNecro, TxtGameOver
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

`Mana`, `ManaMax`, `CoutGobelin`, `CooldownGobelin`, `CooldownGobelinDuree`, `CibleActive`, `DegatsGobelin`, `CooldownAttaqueGobelinDuree`, `GroupeCible`, `NbCiblesGroupe`, `DegatsVillageois`, `CooldownAttaqueVillageoisDuree`, `Groupe1Aggro`, `Groupe2Aggro`, `Groupe1FuiteDX/DY`, `Groupe2FuiteDX/DY`, `Groupe1Combat`, `Groupe2Combat`, `Orbes`, `OrbesSeuil`, `SortieDeverrouillee`, `NiveauTermine`, `Vague1Faite`, `Vague2Faite`, `GameOver`.

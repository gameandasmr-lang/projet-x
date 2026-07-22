# État courant du projet — Projet X

> Mémoire de reprise immédiate de ce projet — même mécanisme que `The Long Ride/PROJECT/docs/reprise_actuel.md`.
> Dernière mise à jour : 2026-07-23 (session 4)

---

## Étape du projet

**Ouverture du chantier (2026-07-20)** : Arnaud a mis The Long Ride en pause volontaire pour se former à la pratique du développement/production sur un premier jeu délibérément simple, réalisable en 2-3 mois. Décision motivée par plusieurs retours convergents de professionnels du jeu vidéo rencontrés (voir `The Long Ride/PROJECT/docs/actif/retours_professionnels/`) : commencer petit avant un projet ambitieux.

**Pivot de concept (2026-07-20, session 1)** : la piste "Le Chasseur Inversé" est abandonnée (gardée en historique dans `docs/CONCEPT_JEU.md`). Nouvelle vision apportée par Arnaud : jeu du **Nécromancien** (invocations, village à conquérir, méta-progression, monde procédural à terme). Vision complète actée comme référence long terme ; **périmètre V1 (2-3 mois) validé** : nécromancien jouable, 2-3 invocations simples en contrôle RTS-lite, mana, orbes = petite méta-progression, niveaux construits à la main (pas de génération procédurale), pas de contre-types en V1.

**État actuel (2026-07-20, session 2) — gouvernance + descente mécaniques.**
- Gouvernance TLR distillée et actée dans `CLAUDE.md` (`<gouvernance>`) : source unique de vérité par sujet, distinction validé/en réflexion, pas de réécriture silencieuse d'une décision actée — sans reprendre l'appareil complet de TLR (pas d'IDs, pas de cycle en 6 étapes, pas de tags de confiance).
- Moteur confirmé : **GDevelop** (continuité TLR). Skill `.claude/Skills/gdevelop-operations/` créé — littératie moteur générique extraite des skills GDevelop de TLR, purgée de tout contenu métier TLR.
- Descente du concept dans `CONCEPT_JEU.md` : **score = orbes** (une seule monnaie) ; fin de niveau = sortie débloquée + fenêtre de 10s avec vagues intensifiées (pas d'arrêt automatique) ; **économie de mana** première passe chiffrée (mana max 100, régén 5/sec, coût croissant Gobelin 15/Démon 30/Colosse 50, cooldown qui suit le coût — ⚠️ à valider en playtest) ; **3 niveaux** confirmés pour la V1 (village / centre+château / couloirs+sommet, fusion de la progression à 5 étapes de la vision complète).

**État actuel (2026-07-22, session 3) — design concret des 3 niveaux acté.**
- Bases communes fixées : caméra qui suit le nécromancien (pas d'écran fixe), échelle héritée de TLR (grille 32×32, perso ~128px), vitesse 6 tiles/s, 1 kill = 1 orbe, seuil de sortie = 70% des orbes tuables, ennemis en mix socle fixe + vagues de spawn.
- Les 3 niveaux chiffrés en première passe (dimensions, nombre/placement d'ennemis, vagues) dans `CONCEPT_JEU.md` section "Design concret des 3 niveaux" — ⚠️ tous les chiffres à valider en playtest, même statut que l'économie de mana.
- Niveau 2 introduit les **obstacles défensifs** (bloquent déplacement + ligne de vue). Niveau 3 introduit le **boss final**, qui incarne la fenêtre de sortie intensifiée déjà décidée (le joueur fuit vers la sortie pendant que le boss le poursuit, plutôt qu'un combat séparé).
- **Méta-progression fermée** : liste des 2 améliorations permanentes actée (cooldown -20% à 100 orbes, PV max +25% à 150 orbes) — scope V1 "1-2 améliorations" désormais complet, plus de contenu à ajouter ici sans revalidation.

**État actuel (2026-07-22/23, session 4) — implémentation GDevelop démarrée, socle mouvement + spawn Gobelin validé.**
- Projet GDevelop créé en local (`PROJECT/game/Projet X.json`, stockage local — pas cloud, limite de 3 projets cloud atteinte sur le compte). 1280×720, pixel art activé (nearest neighbor).
- Scène `Niveau1_Village` (scène de démarrage) : `Necromancien` (Sprite placeholder violet, `TopDownMovementBehavior` à 192 px/s / 6 tiles/s conforme au concept), déplacement en **ZQSD** (contrôles par défaut désactivés, touches simulées individuellement), caméra qui suit via `CenterCameraOnObject`.
- **Invocation Gobelin** : clic gauche fait apparaître un `Gobelin` (placeholder vert, encore statique — pas de comportement/IA) à la position du curseur, si Mana ≥ 15 et cooldown terminé (2s) ; consomme le mana et relance le cooldown. Pas encore de régénération passive de mana, ni de sélection par touche 1/2/3 (un seul type d'invocation existe pour l'instant, donc pas encore nécessaire).
- Variables globales posées : `Mana`, `ManaMax`, `CoutGobelin`, `CooldownGobelin`, `CooldownGobelinDuree`.
- **Repères visuels** : 8 marqueurs de décor statiques (`RepereDecor`, gris, sans comportement) dispersés autour du spawn, pour visualiser le défilement de la caméra en l'absence de vrai décor.
- **Debug** : calque `Debug` dédié (caméra indépendante, non suivie) avec un texte `TxtMana` affichant Mana actuel/max en continu — respecte la règle de séparation des couches du skill GDevelop (jamais de debug dans le calque de simulation/HUD).
- Tout testé et validé par Arnaud dans l'éditeur, plus aucun bloc d'événement en rouge.
- Pas encore de décor réel/tiles, pas d'IA d'invocation (suivi du mage, ordre d'attaque), pas d'ennemis.

**Suite (2026-07-23, même session) — comportement de horde du Gobelin.**
- Le Gobelin invoqué se dirige maintenant vers le nécromancien (`AddForceTowardObject`, 250 px/s) et s'arrête à 50px (condition `Distance` inversée). Plusieurs allers-retours de debug ont été nécessaires : la condition `Distance` a un sens fixe ("est en dessous de X pixels", pas de paramètre de signe) — l'inversion se fait via le flag `inverted` de la condition, pas un paramètre. Détail capitalisé dans le skill `gdevelop-operations` (nouvelle section "Identifiants d'instructions vérifiés").
- Séparation entre Gobelins ajoutée pour éviter qu'ils s'empilent (`SeparateFromObjects`, Gobelin contre lui-même) — fonctionne, même si certains finissent encore par chevaucher le nécromancien lui-même (pas encore de séparation Gobelin/Necromancien, à améliorer si ça gêne en pratique).
- Le skill GDevelop du projet contient maintenant une table de référence des identifiants d'instructions déjà vérifiés en conditions réelles (nom exact, paramètres, pièges) — à consulter en priorité avant de redeviner un nom d'action/condition.

## Prochaine session

- Nom définitif du jeu : point ouvert dans `CONCEPT_JEU.md`, priorité basse, pas bloquant.
- Ordre d'attaque du Gobelin au clic sur un ennemi — pas testable tant qu'il n'y a pas d'ennemis dans la scène.
- Régénération passive du mana (5/sec) — pas encore implémentée.
- Premiers ennemis (villageois statiques) pour tester une invocation contre une cible réelle, et donner enfin un sens à l'ordre d'attaque.
- Optionnel si ça gêne en pratique : empêcher aussi le chevauchement Gobelin/Necromancien (actuellement seule la séparation Gobelin/Gobelin existe).

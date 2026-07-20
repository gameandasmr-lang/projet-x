# État courant du projet — Projet X

> Mémoire de reprise immédiate de ce projet — même mécanisme que `The Long Ride/PROJECT/docs/reprise_actuel.md`.
> Dernière mise à jour : 2026-07-20 (session 2)

---

## Étape du projet

**Ouverture du chantier (2026-07-20)** : Arnaud a mis The Long Ride en pause volontaire pour se former à la pratique du développement/production sur un premier jeu délibérément simple, réalisable en 2-3 mois. Décision motivée par plusieurs retours convergents de professionnels du jeu vidéo rencontrés (voir `The Long Ride/PROJECT/docs/actif/retours_professionnels/`) : commencer petit avant un projet ambitieux.

**Pivot de concept (2026-07-20, session 1)** : la piste "Le Chasseur Inversé" est abandonnée (gardée en historique dans `docs/CONCEPT_JEU.md`). Nouvelle vision apportée par Arnaud : jeu du **Nécromancien** (invocations, village à conquérir, méta-progression, monde procédural à terme). Vision complète actée comme référence long terme ; **périmètre V1 (2-3 mois) validé** : nécromancien jouable, 2-3 invocations simples en contrôle RTS-lite, mana, orbes = petite méta-progression, niveaux construits à la main (pas de génération procédurale), pas de contre-types en V1.

**État actuel (2026-07-20, session 2) — gouvernance + descente mécaniques.**
- Gouvernance TLR distillée et actée dans `CLAUDE.md` (`<gouvernance>`) : source unique de vérité par sujet, distinction validé/en réflexion, pas de réécriture silencieuse d'une décision actée — sans reprendre l'appareil complet de TLR (pas d'IDs, pas de cycle en 6 étapes, pas de tags de confiance).
- Moteur confirmé : **GDevelop** (continuité TLR). Skill `.claude/Skills/gdevelop-operations/` créé — littératie moteur générique extraite des skills GDevelop de TLR, purgée de tout contenu métier TLR.
- Descente du concept dans `CONCEPT_JEU.md` : **score = orbes** (une seule monnaie) ; fin de niveau = sortie débloquée + fenêtre de 10s avec vagues intensifiées (pas d'arrêt automatique) ; **économie de mana** première passe chiffrée (mana max 100, régén 5/sec, coût croissant Gobelin 15/Démon 30/Colosse 50, cooldown qui suit le coût — ⚠️ à valider en playtest) ; **3 niveaux** confirmés pour la V1 (village / centre+château / couloirs+sommet, fusion de la progression à 5 étapes de la vision complète).

## Prochaine session

Poursuivre la section "Ouvert" de `docs/CONCEPT_JEU.md` — dans l'ordre recommandé (dépendance) :
1. Design concret des 3 niveaux (dimensions, nombre/placement d'ennemis précis par niveau) — dépend de l'économie de mana déjà chiffrée ci-dessus.
2. Reste de la liste des améliorations permanentes de méta-progression (au-delà du cooldown d'invocation déjà cité).
3. Nom définitif du jeu (aucune dépendance, à faire quand l'inspiration vient).

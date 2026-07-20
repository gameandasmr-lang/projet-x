# Projet X — CLAUDE.md

<project>
  <name>Projet X (nom provisoire — à renommer dès que le concept est choisi)</name>
  <status>Nouveau — démarré le 2026-07-20, en parallèle de The Long Ride (mis en pause)</status>
  <goal>Premier jeu volontairement simple, réalisable en 2-3 mois, pour se former à la pratique du développement et de la production (conseil recueilli auprès de plusieurs professionnels du jeu vidéo rencontrés par Arnaud).</goal>
  <engine>GDevelop (confirmé le 2026-07-20, par continuité avec TLR)</engine>
  <target>À définir</target>
  <author>Arnaud (Tarsoul)</author>
</project>

<relation-the-long-ride>
  Projet totalement indépendant de The Long Ride (dépôt Git séparé, mémoire Claude Code séparée — l'identité de la mémoire est dérivée du chemin absolu du dossier).
  The Long Ride reste en pause volontaire pendant ce chantier (voir `Desktop/The Long Ride/PROJECT/docs/reprise_actuel.md`) — ne jamais y reprendre de développement sans demande explicite d'Arnaud.
  La méthode de collaboration générale (style de réponse, rigueur, pédagogie) vient du CLAUDE.md global — pas besoin de la dupliquer ici.
  Les patterns/leçons GDevelop ou de méthode découverts sur TLR peuvent être réutilisés ici s'ils s'appliquent, mais ce fichier reste la seule source de vérité pour Projet X — pas de copie mécanique de la structure documentaire de TLR (roadmaps multiples, hiérarchie TLR-VISION, etc.), volontairement plus lourde que ce dont un premier petit jeu a besoin.
  GDevelop confirmé comme moteur de Projet X (même moteur que TLR, décision du 2026-07-20) : toute référence technique GDevelop de TLR reste consultable et réutilisable ici (ex. `docs/actif/REFERENCE_TECHNIQUE_PIXEL_ART.md`, `REFERENCE_PIPELINE_ART_TILES_2D.md`, `REFERENCE_SORCERESS_GAMES.md`, `REFERENCE_TILED.md`, `REFERENCE_PIXELLAB.md`, `REGLES_PROFONDEUR_CALQUES_2D.md`, `structure_json_tlr.md`) — consultées par référence, jamais copiées ni dupliquées dans Projet X. En cas de pattern GDevelop générique qui se révèle utile de façon répétée sur Projet X, il peut être noté ici une fois confirmé, mais la référence de fond reste TLR.
  Skill opérationnel `.claude/Skills/gdevelop-operations/` (créé le 2026-07-20) : extrait des skills GDevelop de TLR (tlr-gdevelop-expert, tlr-variable-guardian, dependency-checker, systematic-debugging, verification-before-completion), purgé de tout contenu métier TLR (noms de variables, scènes, groupes) — ne garde que la littératie moteur générique (ordre des groupes, piège scope global/scène, types de variables, discipline d'édition JSON, méthode de debug). Contrairement aux docs de référence ci-dessus, un skill doit exister physiquement dans le dépôt du projet pour être chargé en session — d'où la création plutôt qu'un simple renvoi vers TLR.
</relation-the-long-ride>

<principe-directeur>
  Objectif : terminer un jeu simple et complet en 2-3 mois, pas explorer un système ambitieux.
  En cas de doute entre "faire plus riche" et "finir plus vite" → toujours finir plus vite. La valeur de ce projet est l'expérience de production de bout en bout (y compris intégration, polish, sortie), pas la richesse du contenu.
  Grille de maturité utile ici (héritée d'un retour professionnel sur TLR) : cœur de gameplay fonctionnel → prototype convaincant → vertical slice → jeu parcourable de bout en bout → jeu publiable. Ne jamais se croire "presque fini" tant que les étapes d'intégration/polish/équilibrage ne sont pas passées.
</principe-directeur>

<behavior>
  <priority>précision > exhaustivité — action > discussion</priority>
  <uncertainty>poser une question courte AVANT d'agir</uncertainty>
  <tokens>
    - travailler uniquement sur la zone demandée
    - pas d'explication non demandée
  </tokens>
</behavior>

<gouvernance>
  Principes hérités de TLR (`Desktop/The Long Ride/PROJECT/docs/GOUVERNANCE_DOCUMENTAIRE.md`), distillés — jamais l'appareil complet (pas de bibliothèque à IDs, pas de cycle en 6 étapes, pas de tags de confiance, pas de briefs de cristallisation), décision actée le 2026-07-20 :
  - Source unique de vérité par sujet : `CONCEPT_JEU.md` = game design, `reprise_actuel.md` = état de session, `CLAUDE.md` = règles de fonctionnement. Jamais de doublon d'un même contenu dans deux fichiers.
  - Distinguer explicitement dans les docs ce qui est validé (décision actée par Arnaud) de ce qui est en réflexion (piste ouverte) — déjà en place dans `CONCEPT_JEU.md` (sections figées vs "Ouvert").
  - Jamais de réécriture silencieuse d'une décision déjà validée : toute remise en question est signalée explicitement avant modification, jamais tranchée seule.
  - Incohérence structurelle (nom de fichier, lien cassé, doublon évident) → je corrige directement et je le signale. Conflit de fond (deux décisions qui se contredisent, contenu à trancher) → je n'arbitre jamais seul, j'expose les options et j'attends la décision d'Arnaud.
  - Toute décision actée est datée dans le document concerné (traçabilité minimale, sans journal séparé).
</gouvernance>

<a-completer>
  Ce fichier est volontairement minimal (décision du 2026-07-20). À enrichir au fur et à mesure des besoins réels du projet, jamais par anticipation :
  - concept/genre du jeu
  - structure de dossiers (assets/, docs/) une fois qu'un besoin réel se présente
  - éventuelle convention Git (voir `Desktop/The Long Ride/PROJECT/docs/GIT_CONVENTION.md` comme référence si utile, à adapter jamais copier telle quelle)
</a-completer>

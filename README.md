# Labyrinthe 3D

Un jeu de labyrinthe en pseudo-3D (raycasting façon Wolfenstein 3D), 100 % vanilla JavaScript/HTML5 Canvas, sans dépendance externe. 100 niveaux à difficulté croissante, générés aléatoirement, avec progression sauvegardée localement.

## Aperçu

- **Vue à la première personne** en raycasting (moteur DDA classique) avec textures de mur générées procéduralement et effet de brouillard.
- **Vue du dessus** alternative, activable dans les paramètres.
- **Mini-carte** superposée en jeu.
- **100 niveaux**, du labyrinthe 5×5 au labyrinthe 25×25, générés par un algorithme *recursive backtracker*, avec sortie placée automatiquement au point le plus éloigné du départ (calcul par parcours en largeur).
- **4 palettes de murs/brouillard** qui changent tous les 25 niveaux.
- **Progression persistante** (niveau max débloqué, meilleur temps par niveau) via `localStorage`.
- **Avatar personnalisable** (couleur, taille, forme) visible sur la mini-carte et en vue du dessus.
- **Sons synthétiques** (pas, collision, victoire) générés via la Web Audio API, sans fichier audio.
- Contrôles clavier (flèches/WASD), glisser-déposer à la souris/tactile pour tourner, et pavé directionnel tactile.
- Plein écran, pause, carte de sélection des niveaux.

## Utilisation

Ouvrez simplement `labyrinthe3d.html` dans un navigateur récent (Chrome, Firefox, Safari, Edge). Aucune installation, build ou serveur n'est nécessaire — tout tient dans un seul fichier HTML autonome.

> ⚠️ Le jeu utilise `localStorage` pour sauvegarder la progression. Il doit donc être ouvert comme une vraie page (fichier local ou hébergée), et non prévisualisé dans un environnement en bac à sable qui bloque le stockage local (par exemple certains aperçus d'artefacts intégrés) — la sauvegarde n'y persisterait pas.

## Contrôles

| Action | Clavier | Souris/Tactile |
|---|---|---|
| Avancer / reculer | ↑ / ↓ ou W / S | Pavé directionnel |
| Tourner | ← / → ou A / D | Glisser latéralement dans la vue, ou pavé directionnel |
| Pause | Échap | Bouton ⏸ |
| Régénérer le niveau | R | Bouton 🔄 |
| Plein écran | F | Bouton ⛶ |

## Structure du code

Le fichier est organisé en modules autonomes au sein d'une seule balise `<script>` :

- `CONFIG` — constantes de gameplay (vitesse, taille de texture, palettes).
- `Utils` — fonctions utilitaires (aléatoire, distance, formatage du temps).
- `Storage` — encapsulation de `localStorage` avec gestion d'erreurs.
- `Toast` — notifications éphémères à l'écran.
- `AudioManager` — génération de sons via `AudioContext` (oscillateurs), sans fichiers audio.
- `TextureGen` — génère et met en cache des textures de brique procédurales sur `<canvas>`.
- `MazeGen` — génération de labyrinthe (recursive backtracker) + calcul de la sortie par BFS.
- `Avatar` — dessine l'avatar du joueur (cercle, carré, triangle, étoile, diamant).
- `Game` — objet principal : état du jeu, boucle de rendu, entrées, gestion des niveaux et interface.

## Compatibilité PC et mobile

Le jeu est pensé pour fonctionner aussi bien à la souris/clavier sur PC qu'au tactile sur smartphone/tablette, avec les optimisations suivantes :

- **Entrées unifiées via Pointer Events.** Le glissement pour tourner la caméra et les boutons du pavé directionnel utilisent une seule API (`pointerdown`/`pointermove`/`pointerup`) au lieu de gestionnaires séparés souris/tactile. Cela évite les doubles déclenchements sur les écrans tactiles hybrides (PC portables tactiles) et, grâce à la capture de pointeur (`setPointerCapture`), le glissement continue d'être suivi même si le doigt ou le curseur sort du canvas — un problème classique avec les anciens gestionnaires `touchmove`/`mousemove`.
- **Zoom accidentel désactivé.** La balise viewport bloque le pincement et le double-tap-zoom, qui perturberaient sans cela le contrôle à la glisse sur mobile. Les contrôles utilisent aussi `touch-action` (`none` sur le canvas et le pavé directionnel, `manipulation` ailleurs) pour éliminer le délai de tap et les gestes indésirables du navigateur.
- **Redimensionnement robuste.** En plus de l'événement `resize`, le jeu écoute `orientationchange` (avec un second recalcul différé, nécessaire car les dimensions ne sont pas toujours stables immédiatement après une rotation d'écran) et `visualViewport.resize` (pour suivre l'apparition/disparition de la barre d'adresse mobile).
- **Hauteur d'écran mobile fiable.** Utilisation de `100dvh` (avec repli sur `100vh`) pour éviter les sauts de mise en page liés à la barre d'adresse des navigateurs mobiles, et `overscroll-behavior: none` pour supprimer l'effet de rebond (rubber-band) au défilement sur iOS.
- **Mode paysage mobile optimisé.** Sur les écrans bas et larges (téléphone en paysage), une règle CSS dédiée agrandit la zone de jeu et masque le titre/sous-titre pour maximiser l'espace utile.
- **Prise en charge des encoches (safe-area).** Les marges du body s'adaptent via `env(safe-area-inset-*)` pour ne pas être masquées par l'encoche ou la barre de gestes d'un téléphone en plein écran.
- **Plein écran multi-navigateurs.** Prise en charge de l'API standard et de son équivalent préfixé `webkit` (Safari), avec masquage automatique du bouton si aucune des deux n'est disponible (au lieu d'un bouton inopérant), et tentative de verrouillage en orientation paysage lors du passage en plein écran sur mobile.
- **Pause automatique en arrière-plan.** Le jeu se met en pause tout seul si l'onglet ou l'application passe en arrière-plan (`visibilitychange`), pratique sur mobile lors d'un changement d'application ou d'un appel entrant.

- **Plein écran fonctionnel avec ses contrôles.** Le bouton plein écran ne rend fullscreen que la zone de jeu (`#stage`) : le pavé directionnel, qui était auparavant un bloc séparé plus bas dans la page, a été déplacé à l'intérieur de `#stage` (en overlay semi-transparent ancré en bas de la zone de jeu) afin de rester visible et utilisable une fois en plein écran. Sans ce changement, le plein écran affichait uniquement le canvas sans aucun moyen d'avancer/reculer sur un appareil sans clavier. `#stage` s'agrandit également pour occuper tout l'écran en plein écran (au lieu de rester limité à sa largeur/hauteur habituelles), et le canvas est recalculé automatiquement à chaque entrée/sortie du plein écran.

## Dernière passe d'analyse (revue exhaustive)

Une relecture complète, ligne par ligne, du HTML/CSS/JS a permis de trouver et corriger quatre problèmes supplémentaires, tous mineurs mais réels et reproductibles :

1. **Touches « fantômes » après perte de focus.** En maintenant une touche (ex. avancer) puis en changeant d'onglet ou d'application sans la relâcher, l'événement `keyup` n'était jamais reçu : le personnage continuait de bouger tout seul au retour, y compris après une reprise de pause. Corrigé en vidant l'état des touches à chaque mise en pause, avec un filet de sécurité supplémentaire sur l'événement `blur` de la fenêtre.
2. **Raccourcis clavier qui « fuyaient » vers les champs de formulaire.** Les flèches étaient interceptées globalement même quand le focus était sur un curseur (sensibilité, taille d'avatar, niveau) ou le champ numérique de niveau — qui répondent eux-mêmes aux flèches. Résultat : régler la sensibilité au clavier faisait aussi tourner le personnage en arrière-plan. Les raccourcis WASD/flèches/R/F sont désormais ignorés quand le focus est sur un `<input>`/`<select>` (le relâchement des touches, lui, reste toujours pris en compte pour ne jamais laisser une touche bloquée).
3. **Clic droit sur la zone de jeu.** Un glissé au bouton droit de la souris faisait à la fois tourner la caméra et ouvrait le menu contextuel du navigateur au relâchement, faute de filtrer le bouton utilisé. Le clic droit/molette est désormais ignoré pour la rotation, et le menu contextuel est désactivé sur la zone de jeu.
4. **Logs de débogage bavards.** Le message de log affiché à chaque redimensionnement se déclenchait désormais bien plus souvent qu'avant (rotation d'écran, plein écran, barre d'adresse mobile) et aurait spammé la console. Retiré.

En prime, le curseur de la souris passe maintenant de « grab » à « grabbing » pendant le glissement, pour un retour visuel plus clair.

## Dernière passe de relecture complète

Une relecture exhaustive du fichier (CSS, HTML, JS, ligne par ligne) a permis de corriger cinq derniers points :

1. **Ordre d'initialisation incorrect.** `this.view.getContext('2d')` était appelé avant la vérification `if (!this.view || !this.mini)`, rendant ce garde-fou inatteignable : si un des deux canvas n'existait pas, le script plantait avant même de pouvoir afficher le message d'erreur prévu. Corrigé en vérifiant d'abord la présence des éléments.
2. **La simulation continuait de tourner derrière les panneaux « Paramètres » et « Carte des niveaux ».** Ces panneaux recouvrent toute la zone de jeu mais ne mettaient pas le jeu en pause : le chrono, la distance parcourue, les déplacements au clavier, voire la détection de victoire, continuaient à s'exécuter alors que le joueur ne voyait plus le labyrinthe. La boucle de mise à jour vérifie désormais si l'un de ces panneaux est ouvert et suspend la simulation en conséquence.
3. **Curseur bloqué en « grabbing ».** Si la pause était déclenchée (touche Échap, bouton pause) pendant un glissement de la souris pour tourner la caméra, le curseur restait visuellement en mode « saisie » au lieu de revenir à la normale.
4. **`backdrop-filter` sans préfixe `-webkit-`** sur les boutons du pavé directionnel, pouvant ne pas s'appliquer sur Safari/iOS plus anciens.
5. **Incohérence de largeur entre les deux curseurs de réglage** (sensibilité et taille d'avatar) : seul le premier avait une largeur explicite, le second dépendait de la valeur par défaut du navigateur.

## Corrections apportées

Le code fourni était déjà solide dans son ensemble (génération de labyrinthe, moteur de raycasting, sauvegarde de progression) mais contenait deux défauts fonctionnels, corrigés dans cette version :

1. **Interrupteurs (sons / mini-carte) toujours désactivés après un clic.**
   Le gestionnaire générique de changement de paramètre lisait `e.target.value` pour toutes les entrées, y compris les cases à cocher. Or la propriété `.value` d'une `<input type="checkbox">` vaut toujours `"on"`, qu'elle soit cochée ou non — seule `.checked` reflète son état réel. Résultat : dès qu'on touchait l'interrupteur « Sons » ou « Mini-carte », le réglage était systématiquement enregistré comme `false`, quel que soit l'état affiché.
   → Le gestionnaire distingue désormais les cases à cocher (`.checked`) des autres champs (`.value`).

2. **Bouton « Réinitialiser » potentiellement silencieux.**
   Il s'appuyait sur `window.confirm()`, une boîte de dialogue native que certains environnements d'affichage (iframes, webviews, aperçus intégrés) bloquent silencieusement, rendant le bouton inopérant sans message d'erreur.
   → Remplacé par une confirmation « double-clic » interne (un premier clic affiche un avertissement pendant 3 secondes, un second clic dans ce délai confirme la réinitialisation), qui fonctionne dans tous les contextes d'exécution.

Une petite amélioration de confort a également été ajoutée : le champ numérique « Aller au niveau » réinitialise sa valeur affichée au niveau courant lorsqu'une saisie invalide ou verrouillée est refusée, au lieu de conserver la valeur incorrecte à l'écran.

Aucune autre anomalie fonctionnelle n'a été identifiée : le moteur de raycasting, la génération de labyrinthe, le calcul de sortie, la logique de collision et la sauvegarde de progression fonctionnent correctement.

## Limites connues

- Les sons dépendent de la Web Audio API et nécessitent une première interaction utilisateur (clic, touche, appui) pour démarrer, conformément aux politiques des navigateurs.
- Le plein écran (`requestFullscreen`) peut être refusé silencieusement dans certains contextes intégrés (iframes sans permission `allowfullscreen`) ; le jeu reste jouable en mode normal dans ce cas.
- La progression est locale au navigateur/appareil (`localStorage`) : elle n'est pas synchronisée entre appareils et sera perdue si le stockage du site est effacé.

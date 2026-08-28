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
| --- | --- | --- |
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

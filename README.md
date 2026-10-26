# Liquid War HTML5 - Édition Récréée

Une réimplémentation moderne, fluide et responsive en **HTML5 Canvas** & **Tailwind CSS** du célèbre jeu vidéo tactique *Liquid War* (créé par Thomas Colcombet et Christian Mauduit).

---

## 🌟 Aperçu du Projet

**Liquid War** est un jeu de stratégie unique où vous contrôlez une armée de milliers de particules liquides (cellules/combattants) qui s'écoulent comme un fluide à travers un labyrinthe. Contrairement aux jeux de stratégie classiques, vous ne contrôlez pas directement chaque unité, mais déplacez un **curseur de direction (onde d'attraction)**. Toutes vos particules cherchent dynamiquement le plus court chemin praticable vers ce curseur.

---

## 🚀 Fonctionnalités Principales

- **Moteur de Simulation de Fluide Temps Réel** : Prise en charge fluide de 1 000 à 15 000+ particules en simultané grâce à des tableaux typés JavaScript (`Uint8Array`, `Int32Array`, `Uint16Array`).
- **Champ de Distance Euclidien (Pathfinding Dijkstra)** : Propagation d'onde en temps réel calculant le plus court chemin contournant les murs pour chaque équipe avec métrique euclidienne (coûts 10 en cardinal, 14 en diagonal).
- **Génération Procédurale de Cavernes & Labyrinthes** :
  - **Grandes Cavernes Espacées** : Salles organiques reliées par un Arbre Couvrant Minimal (MST via l'algorithme de Prim) avec boucles tactiques.
  - **Arène Ouverte** : Zone de combat directe sans obstacles.
  - **Labyrinthe Classique** : Réseau de couloirs et chicanes rectilignes.
- **Support Multi-Résolution** :
  - Standard (160 x 120)
  - Grande HD (240 x 180)
  - Giga Caverne (320 x 240)
- **Mécanique de Combat & Soin ("Cure")** :
  - Pression frontale d'attaque et conversion d'unités à 0 HP.
  - Transmission de régénération de santé lorsque les unités arrière poussent sur leurs coéquipiers en première ligne.
- **Interface Moderne & Responsive** : Conçue avec Tailwind CSS et FontAwesome, incluant un panneau de réglages physiques, des barres de contrôle du territoire en temps réel et un onglet de documentation algorithmique.

---

## 🎮 Contrôles & Prise en Main

- **Joueur 1 (Bleu)** :
  - **Souris** : Déplace le curseur cible sur le canevas.
  - **Clavier alternatif** : Touches `Flèches` ou `ZQSD` / `WASD`.
- **Bots (Rouge, Vert, Jaune)** : Intelligence artificielle contrôlant dynamiquement leur curseur vers le centre de masse de l'armée adverse.

---

## 🧮 Détails Algorithmiques

### 1. Champ de Distance Dijkstra (Euclidien)
Pour chaque équipe $t \in \{0, 1, 2, 3\}$, une grille de distance `distGrids[t]` est recalculée à chaque frame depuis la position du curseur :
- **Propagation** : Dijkstra à base de Min-Heap optimisé.
- **Coûts** : `10` pour les déplacements cardinaux (H/V), `14` pour les déplacements diagonaux ($\approx 10 \times \sqrt{2}$).
- **Résultat** : Iso-lignes de distance parfaitement circulaires créant un comportement de rassemblement en boule naturelle autour du curseur.

### 2. Algorithme de Combat & Soin (Cure)
Lorsqu'une particule choisit sa prochaine case adjacente :
- **Case vide** : Déplacement direct vers la valeur de distance la plus faible.
- **Particule ennemie** : Dégâts infligés selon le curseur *Puissance d'Attaque*. Si la santé tombe à 0 HP, la particule est assimilée (change d'équipe) et ses HP sont réinitialisés.
- **Particule alliée** : Transmission de soin si l'allié est blessé (< max HP) selon la *Pression de Soin*.

---

## 💻 Installation & Exécution

1. Téléchargez le fichier `index.html`.
2. Ouvrez le fichier `index.html` directement dans n'importe quel navigateur web moderne (Chrome, Firefox, Edge, Safari).
3. Aucun serveur ou installation Node.js n'est nécessaire !
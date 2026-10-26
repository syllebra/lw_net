# Prompt Master : Génération de Liquid War HTML5

Ce document contient la spécification technique complète permettant de régénérer de zéro l'application **Liquid War HTML5** dans une page unique `index.html`.

---

## 📄 Prompt Système & Consignes Globales

> **Instruction Générale :**  
> Génère une application web complète, autonome et responsive contenue dans un seul fichier `index.html`. L'application doit réimplémenter le jeu vidéo *Liquid War* en HTML5 Canvas JavaScript avec une interface moderne basée sur Tailwind CSS et FontAwesome.

---

## 🎨 Interface Utilisateur & Style (UI/UX)

- **Header / Navigation** :
  - Titre "Liquid War HTML5" avec un sous-titre "Simulation de Fluides & Bataille de Particules".
  - Onglets de navigation : **Jouer** (Vue du jeu) et **Algorithmes & Doc** (Vue explicative).
- **Panneau de Contrôle Panneau Gauche (Sidebar)** :
  - **Boutons d'Action** : *Redémarrer* (réinitialise la partie) et *Pause/Reprendre*.
  - **Menu Déroulant Taille & Résolution** :
    - Standard (160 x 120)
    - Grande HD (240 x 180 - Par défaut)
    - Giga Caverne (320 x 240)
  - **Menu Déroulant Choix de Carte** :
    - Grandes Cavernes Espacées
    - Arène Ouverte
    - Labyrinthe Classique
  - **Curseurs (Sliders) de Réglages Physiques** :
    - Total Particules : 1 000 à 15 000 (par défaut : 6 000)
    - Puissance d'Attaque : 30 à 300 (par défaut : 120)
    - Pression de Soin (Cure) : 0 à 300 (par défaut : 150)
    - Santé Particule (HP) : 300 à 2 500 (par défaut : 1 000)
  - **Statistiques en Temps Réel** : Barres de pourcentage de contrôle de territoire pour chacune des 4 équipes (Joueur Bleu, Bot Rouge, Bot Vert, Bot Jaune).
- **Zone de Rendu Canvas (Centre/Droite)** :
  - Élément `<canvas id="gameCanvas">` centré avec rendu de pixels net (`image-rendering: pixelated`).
  - Overlay modal de victoire affiché automatiquement lorsqu'une seule équipe survit.

---

## ⚙️ Architecture Technique & Moteur JavaScript

### 1. Structure des Données Dynamiques
Les grilles doivent utiliser des tableaux typés JavaScript pour maximiser les performances de simulation :
- `walls = new Uint8Array(GRID_W * GRID_H)` : Grille des obstacles (1 = mur, 0 = vide).
- `cellGrid = new Int32Array(GRID_W * GRID_H)` : Grille d'occupation par les particules (-1 = libre, sinon index de la particule).
- `distGrids = [Uint16Array, Uint16Array, Uint16Array, Uint16Array]` : Champ de distance pour chacune des 4 équipes.
- `heap = new Uint32Array(GRID_W * GRID_H)` : Min-Heap partagé pour l'algorithme de Dijkstra.

### 2. Algorithme du Champ de Distance (Euclidean Dijkstra)
- Pour chaque équipe $t$, calculer la propagation d'onde depuis la position du curseur $C_t(x, y)$.
- Si le curseur est dans un mur, rechercher la case libre la plus proche.
- **Direction et Coûts** :
  - Mouvements cardinaux (haut, bas, gauche, droite) : coût `10`.
  - Mouvements diagonaux : coût `14`.
  - Interdire le passage à travers les coins durs (diagonale bloquée si les deux murs adjacents sont présents).

### 3. Génération Procédurale des Cavernes (`buildMap`)
- **Mode Caverne (`cavern`)** :
  - Placer 4 salles de départ dans les coins et 1 salle centrale majeure.
  - Ajouter 6 à 9 salles organiques réparties aléatoirement avec déformation radiale bruits trigonométriques.
  - Calculer un **Arbre Couvrant Minimal (MST)** via l'algorithme de Prim pour connecter toutes les salles (100% de connectivité).
  - Ajouter 45% de chemins/boucles tactiques supplémentaires.
  - Tracer des couloirs sinueux d'un rayon de 4 à 5 cellules.
  - Appliquer une étape d'automate cellulaire pour lisser les parois rocheuses.

### 4. Mise à Jour des Particules (`updateParticles`)
Pour chaque particule :
1. Consulter les 8 voisins accessibles sans franchir de mur.
2. Trier les cases par valeur croissante dans `distGrids[team]`.
3. **Cas A - Case Libre** : La particule se déplace sur la case.
4. **Cas B - Case Ennemie** : Infliger les dégâts d'attaque. Si les HP de l'ennemi passent $\le 0$, l'ennemi est converti à l'équipe actuelle et voit ses HP réinitialisés.
5. **Cas C - Case Alliée Blessée** : Appliquer le soin (Cure) si l'allié a moins que le HP maximum.

### 5. Intelligence Artificielle des Bots (`updateBots`)
- Les bots 1, 2 et 3 calculent le centre de masse des particules adverses.
- Le curseur du bot se déplace continuellement vers cette position cible à une vitesse fluide.
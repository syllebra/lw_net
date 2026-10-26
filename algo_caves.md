# Spécification Technique & Algorithmique : Génération de Cavernes Organiques (Style D&D / Roguelike)

Ce document détaille l'algorithme procédural de génération de cartes sous forme de **cavernes organiques à salles et couloirs interconnectés**, inspiré du level design des jeux de rôle et jeux vidéo tactiques.

---

## 🔍 Vue d'Ensemble du Procédé

L'algorithme garantit une carte esthétique, jouable et **100 % connectée**, sans aucune zone isolée. Il se décompose en 4 phases majeures :

1. **Placement des Salles Organiques (Blob Carving)**
2. **Topologie de Connectivité (Arbre Couvrant Minimal - Prim + Boucles Tactiques)**
3. **Creusement des Couloirs Sinueux (Wandering Tunnels)**
4. **Lissage et Finition (Automates Cellulaires)**

---

## 📐 Étapes Algorithmiques Détaillées

### 1. Placement des Salles Organiques (Blob Carving)
- **Salles de Départ Végétalisées/Stratégiques** :
  - 4 salles imposées aux quatre coins de la grille (zones de spawn équilibrées pour les 4 équipes).
  - 1 grande salle arène centrale.
- **Salles Intermédiaires** :
  - Génération procédurale de $N$ salles (entre 6 et 10 selon la taille de la grille).
  - Validation par distance minimale entre les centres pour éviter la surpopulation.
- **Déformation Radiale Organique** :
  - Pour chaque salle de centre $(X_c, Y_c)$ et de rayon de base $R$, les parois sont sculptées selon la formule radiale :
    $$R_{\text{effectif}}(\theta) = R + A_1 \cdot \sin(3\theta + \phi) + A_2 \cdot \cos(2\theta + \phi)$$
  - Cela produit des formes de grottes naturelles non parfaitement circulaires.

---

### 2. Graphe de Connectivité (MST + Boucles)
Pour éviter qu'une salle ne soit isolée ou qu'il y ait des cul-de-sac injouables :
- **Arbre Couvrant Minimal (MST via l'Algorithme de Prim)** :
  - On considère un graphe complet où chaque sommet est une salle et chaque arête est la distance euclidienne entre deux centres de salles.
  - On applique l'algorithme de Prim pour sélectionner l'ensemble minimal d'arêtes reliant **toutes** les salles ensemble.
- **Boucles Tactiques (45 % Extra Edges)** :
  - Pour créer du contournement stratégique (flanking), on ajoute aléatoirement 45 % d'arêtes supplémentaires parmi les paires de salles proches ($D < 110$ unités).

---

### 3. Creusement des Couloirs Sinueux (Wandering Tunnels)
Pour chaque arête du graphe sélectionné, un tunnel est creusé d'une salle $A$ vers une salle $B$ :
- **Marche Sinueuse (Wobble Angle)** :
  - Un pointeur se déplace du centre $A$ vers le centre $B$.
  - À chaque pas, la direction subit une variation angulaire sinusoïdale :
    $$\theta_{\text{pas}} = \theta_{\text{direct}} + A_{\text{wobble}} \cdot \sin(\text{step} \cdot F_{\text{wobble}})$$
- **Carving Épais** :
  - À chaque position du pointeur, un disque d'un rayon de 4 à 5 cellules de grille est vidé (`walls[y * W + x] = 0`).

---

### 4. Lissage par Automates Cellulaires
Pour adoucir les angles vifs des murs et donner un aspect de roche érodée :
- Un balayage de lissage basé sur le nombre de voisins solides (voisinage de Moore 3x3) est appliqué :
  - Si une cellule vide est entourée de $\ge 7$ murs $\rightarrow$ elle devient un mur.
  - Si une cellule solide est entourée de $\le 2$ murs $\rightarrow$ elle devient vide.

---

## 💻 Implémentation Exemple (JavaScript / TypedArrays)

```javascript
function generateCaveMap(GRID_W, GRID_H, scale) {
  const walls = new Uint8Array(GRID_W * GRID_H).fill(1);
  const rooms = [];

  // 1. Placement des Salles de Spawn et de la salle centrale
  const cornerOffset = Math.floor(25 * scale);
  rooms.push({ x: cornerOffset, y: cornerOffset, r: 14 * scale });
  rooms.push({ x: GRID_W - cornerOffset, y: cornerOffset, r: 14 * scale });
  rooms.push({ x: cornerOffset, y: GRID_H - cornerOffset, r: 14 * scale });
  rooms.push({ x: GRID_W - cornerOffset, y: GRID_H - cornerOffset, r: 14 * scale });
  rooms.push({ x: GRID_W / 2, y: GRID_H / 2, r: 18 * scale });

  // 2. Creusement des Salles Organiques
  rooms.forEach(rm => {
    const seed = Math.random() * 100;
    for (let dy = -rm.r - 5; dy <= rm.r + 5; dy++) {
      for (let dx = -rm.r - 5; dx <= rm.r + 5; dx++) {
        const angle = Math.atan2(dy, dx);
        const noise = Math.sin(angle * 3 + seed) * 2.5 + Math.cos(angle * 2 + seed) * 1.5;
        if (dx * dx + dy * dy <= (rm.r + noise) ** 2) {
          const cx = Math.floor(rm.x + dx), cy = Math.floor(rm.y + dy);
          if (cx > 2 && cx < GRID_W - 3 && cy > 2 && cy < GRID_H - 3) {
            walls[cy * GRID_W + cx] = 0;
          }
        }
      }
    }
  });

  return walls;
}
# Cours de Vision par Ordinateur : Classification d'Images et Classificateurs Linéaires

## Introduction au Cours et Détails Administratifs

### Avis d'Enregistrement
Ce cours est enregistré. Bien que les étudiants ne soient pas visibles à la caméra, leurs voix peuvent être capturées dans l'enregistrement. Veuillez en être conscient si vous avez des préoccupations concernant votre participation aux discussions.

### Informations sur les Devoirs
Le premier devoir sera publié ce soir ou tôt demain et est dû le 20 janvier, vous donnant exactement deux semaines pour le compléter. Vous implémenterez :
- Un classificateur k-plus proches voisins
- Un classificateur linéaire
- Un petit réseau de neurones à deux couches
- L'algorithme complet de rétropropagation pour le réseau de neurones

**Note Importante :** Veuillez ne pas compléter les devoirs des années précédentes (2015) car ils ont été mis à jour pour ce cours.

### Exigences Techniques

#### Python et NumPy
Les devoirs nécessitent une maîtrise de Python et NumPy. Vous écrirez des expressions NumPy optimisées qui manipulent efficacement des matrices et des vecteurs. Si du code comme celui-ci ne vous est pas familier :
```python
# Exemple d'opérations NumPy vectorisées
X = np.array([...])
W = np.array([...])
scores = np.dot(W, X)
```
Veuillez consulter le tutoriel Python/NumPy de Justin disponible sur le site web du cours.

#### Terminal.com - Informatique en Nuage
Pour les étudiants sans ordinateurs portables puissants, nous offrons Terminal.com - des machines virtuelles dans le cloud avec des dépendances préinstallées. Caractéristiques :
- Interface basée sur navigateur
- Environnement préconfiguré avec toutes les dépendances des devoirs
- Options de machines CPU et GPU
- Accès Jupyter notebook et terminal

Pour utiliser Terminal.com :
1. Envoyez un email au TA désigné pour obtenir des crédits
2. Lancez une machine via l'interface web
3. Toutes les données et dépendances sont préinstallées

**Note :** Bien que des unités GPU soient disponibles et que le code CUDA soit autorisé, ce n'est pas requis pour les devoirs.

## Classification d'Images

### Le Problème Central
La classification d'images implique :
- Un ensemble fixe de catégories (ex : chien, chat, camion, avion)
- Entrée : Image comme grille de nombres
- Sortie : Étiquette de catégorie

C'est fondamental en vision par ordinateur - une fois que vous comprenez la classification d'images, d'autres tâches comme la détection d'objets, la description d'images et la segmentation deviennent des extensions simples.

### L'Écart Sémantique
Le défi réside dans l'écart sémantique entre :
- **Ce que les ordinateurs voient :** Tableaux 3D de nombres (hauteur × largeur × 3 canaux de couleur)
- **Ce que nous voulons :** Compréhension sémantique (ex : "c'est un chat")

Les images sont représentées comme des tableaux de valeurs de luminosité (0-255) pour les canaux rouge, vert et bleu à chaque position de pixel.

### Défis Majeurs dans la Classification d'Images

1. **Variation du Point de Vue**
   - Rotation, zoom et changements de position de la caméra
   - Variations des propriétés focales
   - Tous causent des changements dramatiques dans les valeurs des pixels

2. **Illumination**
   - Différentes conditions d'éclairage
   - Ombres et reflets
   - Doit reconnaître les objets sous n'importe quel schéma d'éclairage

3. **Déformation**
   - Objets dans différentes poses
   - Transformations non rigides
   - Exemple : Chats dans diverses positions

4. **Occlusion**
   - Visibilité partielle des objets
   - Objets derrière d'autres objets
   - Doit reconnaître à partir d'informations partielles

5. **Encombrement de l'Arrière-Plan**
   - Objets se fondant dans les arrière-plans
   - Scènes complexes

6. **Variation Intra-Classe**
   - Différentes instances de la même classe
   - Exemple : Nombreuses espèces de chats toutes étiquetées "chat"

### Approches Traditionnelles vs Basées sur les Données

#### Approches Explicites (Traditionnelles)
Les débuts de la vision par ordinateur tentaient des règles explicites :
- Détecter les bords et les formes
- Rechercher des caractéristiques spécifiques (ex : formes d'oreilles pour les chats)
- Créer des bibliothèques de motifs

**Problèmes :**
- Non évolutif - besoin de nouvelles règles pour chaque classe
- Fragile - échoue avec les variations
- Intensif en main-d'œuvre

#### Approche Basée sur les Données (Moderne)
- Collecter de grands ensembles de données d'images étiquetées
- Entraîner un modèle sur des exemples
- Le modèle apprend des motifs à partir des données
- Généralise à de nouveaux exemples

Cette approche est devenue viable avec :
- Disponibilité de données à l'échelle d'Internet
- Ressources de calcul
- Avancées en apprentissage automatique

## Classificateur k-Plus Proches Voisins

### Concept de Base
1. **Entraînement :** Stocker toutes les données d'entraînement
2. **Test :** Pour chaque image de test :
   - Comparer à toutes les images d'entraînement
   - Trouver les k images les plus similaires
   - Prédire basé sur le vote majoritaire

### Ensemble de Données CIFAR-10
- 10 classes : avion, automobile, oiseau, chat, cerf, chien, grenouille, cheval, navire, camion
- 50 000 images d'entraînement
- 10 000 images de test
- Vignettes de 32×32 pixels

### Métriques de Distance

#### Distance de Manhattan (L1)
```
d = Σ|X_test[i] - X_train[i]|
```
Somme des différences absolues entre les valeurs de pixels

#### Distance Euclidienne (L2)
```
d = √(Σ(X_test[i] - X_train[i])²)
```
Racine carrée de la somme des différences au carré

### Exemple d'Implémentation
```python
class NearestNeighbor:
    def train(self, X, y):
        # Simplement mémoriser les données d'entraînement
        self.Xtr = X
        self.ytr = y
    
    def predict(self, X):
        predictions = []
        for x_test in X:
            # Calculer les distances à tous les exemples d'entraînement
            distances = np.sum(np.abs(self.Xtr - x_test), axis=1)
            # Trouver le plus proche voisin
            min_index = np.argmin(distances)
            # Prédire l'étiquette du plus proche voisin
            predictions.append(self.ytr[min_index])
        return predictions
```

### Caractéristiques de Performance
- **Temps d'entraînement :** O(1) - juste stocker les données
- **Temps de test :** O(n) - comparer à tous les n exemples d'entraînement
- C'est l'inverse de ce que nous voulons (temps de test rapide)

### Sélection des Hyperparamètres

#### Le Paramètre k
- k=1 : Un seul plus proche voisin
- k>1 : Vote majoritaire parmi k voisins
- Un k plus grand lisse les frontières de décision

#### Validation Croisée
1. Diviser les données d'entraînement en plis
2. Entraîner sur k-1 plis, valider sur 1 pli
3. Faire tourner le pli de validation
4. Choisir les hyperparamètres avec la meilleure performance moyenne
5. Évaluation finale sur l'ensemble de test (une seule fois !)

### Limitations du k-NN
1. Coûteux en calcul au moment du test
2. Les métriques de distance sur des données de haute dimension se comportent mal
3. Malédiction de la dimensionnalité
4. Non utilisé en pratique pour les images

Exemple d'échec de la métrique de distance :
- Image originale
- Image décalée (même distance L2)
- Image assombrie (même distance L2)
- Image avec occluseur (même distance L2)

Toutes ont une distance égale mais une signification sémantique très différente !

## Classification Linéaire

### Approche Paramétrique
Contrairement au k-NN (non paramétrique), les classificateurs linéaires apprennent des paramètres :
```
f(x, W) = Wx + b
```
Où :
- x : Image d'entrée (aplatie en vecteur colonne)
- W : Matrice de poids (paramètres à apprendre)
- b : Vecteur de biais
- Sortie : Scores pour chaque classe

### Dimensions des Matrices
Pour CIFAR-10 :
- x : 3072×1 (32×32×3 aplati)
- W : 10×3072 (10 classes × 3072 caractéristiques)
- b : 10×1
- Sortie : 10×1 (scores pour chaque classe)

### Interprétations des Classificateurs Linéaires

1. **Correspondance de Modèles**
   - Chaque ligne de W est un modèle pour une classe
   - Score = produit scalaire de l'image avec le modèle
   - Score élevé indique la similarité

2. **Compteur de Couleurs**
   - Somme pondérée des valeurs de pixels
   - Compte les couleurs à des positions spécifiques
   - Poids positifs : la présence augmente le score
   - Poids négatifs : la présence diminue le score

3. **Vue Géométrique**
   - Images comme points dans un espace de haute dimension
   - Chaque classificateur définit un hyperplan
   - Le score indique la distance de l'hyperplan

### Visualisation des Modèles Appris
Quand nous remodelons les vecteurs de poids en images :
- Avion : Tache bleue (détection du ciel)
- Voiture : Tache rouge (couleur de voiture commune dans l'ensemble de données)
- Cheval : Apparence à deux têtes (exemples orientés à gauche/droite)

Ces visualisations révèlent des limitations :
- Ne peut apprendre qu'un seul modèle par classe
- Doit moyenner sur toutes les variations
- Ne peut pas gérer plusieurs modes

### Limitations des Classificateurs Linéaires

**Scénarios Difficiles :**
1. Données non linéairement séparables (ex : problème XOR)
2. Modes multiples par classe
3. Transformations complexes
4. Classification basée sur la texture
5. Motifs spatialement invariants

**Exemples de Problèmes :**
- Images négatives (couleurs inversées)
- Distributions multimodales (voitures sous différents angles)
- Distributions de classes concentriques
- Images en niveaux de gris (pas d'information de couleur)

## Perspectives

### Le Cadre d'Apprentissage Automatique
1. **Définir un modèle :** f(x, W) qui mappe les entrées aux scores
2. **Définir une fonction de perte :** Quantifie à quel point les poids actuels sont "mauvais"
3. **Optimisation :** Trouver des poids qui minimisent la perte

### Du Linéaire aux Réseaux de Neurones
Le cours construira progressivement la complexité :
1. Classificateurs linéaires (cette conférence)
2. Réseaux de neurones à deux couches
3. Réseaux de neurones profonds
4. Réseaux de neurones convolutionnels

Chaque étape ajoute de la capacité pour gérer des motifs plus complexes tout en maintenant le même cadre global.

### Réseaux de Neurones comme Systèmes Modulaires
L'apprentissage profond moderne traite les réseaux de neurones comme des blocs LEGO :
- Créer des composants modulaires
- Les empiler ensemble
- Les composants communiquent via les gradients
- Permet des architectures complexes

**Exemple d'Application : Description d'Images**
- Réseau de Neurones Convolutionnel (CNN) pour la compréhension visuelle
- Réseau de Neurones Récurrent (RNN) pour la génération de langage
- Connectés ensemble pour décrire des images en langage naturel

### Concepts Clés à Venir
- **Fonctions de perte :** Quantifier la qualité des prédictions
- **Optimisation :** Améliorer itérativement les poids
- **Rétropropagation :** Calculer efficacement les gradients
- **Conception d'architecture :** Empiler des couches pour plus de puissance

Le cadre fondamental reste constant tout au long :
1. Passe avant : Calculer les prédictions
2. Calculer la perte
3. Passe arrière : Calculer les gradients
4. Mettre à jour les poids
5. Répéter jusqu'à convergence

La prochaine conférence couvrira les fonctions de perte et l'optimisation, construisant vers les réseaux de neurones et éventuellement les réseaux de neurones convolutionnels pour la classification d'images de pointe.
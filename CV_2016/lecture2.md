# Cours : Fonctions de Perte et Optimisation

## Introduction et Notes Administratives
*[0:01 - 0:32]*

Avant de plonger dans le matériel sur les fonctions de perte et l'optimisation, quelques rappels administratifs :
- Le premier devoir est à rendre mercredi prochain (environ 9 jours restants)
- Lundi est un jour férié - pas de cours, pas d'heures de bureau
- Planifiez votre temps en conséquence
- Les jours de retard peuvent être utilisés et répartis entre vos devoirs comme vous le souhaitez

## Récapitulatif : Reconnaissance Visuelle et Classification d'Images
*[0:32 - 1:52]*

### État Actuel du Problème
- La reconnaissance visuelle, spécifiquement la classification d'images, est un problème très difficile
- Besoin d'être robuste à toutes les variations possibles lors de la reconnaissance de catégories (ex : chats)
- Malgré l'apparence insoluble, les méthodes actuelles :
  - Peuvent résoudre pour des milliers de catégories
  - Fonctionnent avec une précision quasi-humaine (parfois la dépassant)
  - S'exécutent presque en temps réel sur les téléphones
  - Toutes les avancées majeures se sont produites dans les 3 dernières années

### Concepts Précédemment Couverts
- **Approche basée sur les données** : On ne peut pas coder en dur les classificateurs, on doit les entraîner à partir de données
- **Divisions du jeu de données** : Données d'entraînement, divisions de validation pour les hyperparamètres, ensemble de test
- **Classificateur k-plus proches voisins** : Exemple de classificateur de base
- **Jeu de données CIFAR-10** : Jeu de données jouet pour le cours
- **Approche paramétrique** : Fonction f de l'image directement vers les scores de classes bruts
  - Forme linéaire : `f = Wx`
  - Peut être interprété comme correspondance de modèles
  - Images dans un espace de haute dimension avec des classificateurs linéaires créant des frontières de décision

## Configuration de l'Exemple d'Entraînement
*[2:28 - 3:46]*

Exemple avec 3 images et 10 classes :
- La fonction f assigne des scores pour chaque image avec des réglages de poids particuliers
- Certains résultats sont bons (score élevé pour la classe correcte), d'autres mauvais
- Résultats exemples :
  - Image de chat : score de la classe correcte 2.9 (médiocre - au milieu)
  - Image de voiture : bien classifiée (score de voiture beaucoup plus élevé que les autres)
  - Image de grenouille : mal classifiée

Objectif : Trouver des poids qui donnent des scores cohérents avec les étiquettes de vérité terrain sur toutes les données d'entraînement

## Fonctions de Perte
*[3:46 - 4:09]*

Besoin de quantifier la notion de "qualité" des poids :
- Assigner une valeur numérique de "mauvais" aux réglages de poids (ex : "12 mauvais" ou "1.5 mauvais")
- Définir une fonction de perte qui mesure le mécontentement avec les résultats
- Une fois qu'on a la fonction de perte, la minimiser pour trouver le W optimal

Deux fonctions de perte principales couvertes :
1. Coût SVM (Support Vector Machine)
2. Coût Softmax (entropie croisée)

## Perte SVM
*[4:15 - 9:06]*

### Formule de Perte SVM Multiclasse

**Perte = Σᵢ Σⱼ≠yᵢ max(0, sⱼ - syᵢ + 1)**

Où :
- Somme sur tous les exemples d'entraînement (i)
- Somme sur toutes les classes incorrectes (j ≠ yᵢ)
- Compare le score de classe incorrecte (sⱼ) avec le score de classe correcte (syᵢ)
- Marge de sécurité de 1
- Max avec 0 (perte charnière)

### Concepts Clés
- Ne veut pas seulement que le score correct > scores incorrects
- Veut une marge de sécurité d'exactement 1
- Les scores sont sans échelle (peuvent augmenter/diminuer W)
- La marge de 1 est arbitraire mais choix conventionnel

### Exemple Concret
*[6:26 - 8:53]*

Travail sur le premier exemple :
- Chat (correct) : 3.2
- Voiture : 5.1
- Grenouille : 1.7

Calcul de la perte :
- Terme voiture : max(0, 5.1 - 3.2 + 1) = 2.9
- Terme grenouille : max(0, 1.7 - 3.2 + 1) = 0
- Perte totale pour cet exemple : 2.9

Interprétation : SVM veut que les scores incorrects soient au plus 2.2 (score correct - 1), mais la voiture a obtenu 5.1

### Questions et Propriétés
*[9:06 - 13:04]*

1. **Pourquoi exclure j = yᵢ dans la somme ?**
   - Ajouterait juste une constante de 1 à la perte
   - N'affecte pas l'optimisation

2. **Moyenne vs somme sur les classes ?**
   - Échelle juste par une constante (1/nombre de classes)
   - Ne change pas le W optimal

3. **Perte charnière au carré ?**
   - max(0, sⱼ - syᵢ + 1)²
   - Fonction de perte différente, solutions différentes
   - Change les compromis de manière non linéaire

4. **Bornes de perte** :
   - Min : 0
   - Max : ∞

5. **Perte avec petite initialisation aléatoire de W** : Nombre de classes - 1

## Implémentation Numpy
*[15:24 - 16:40]*

```python
# Perte SVM vectorisée pour un seul exemple
scores = W.dot(x)  # Calculer tous les scores de classes
margins = np.maximum(0, scores - scores[y] + 1)  # Calculer les marges
margins[y] = 0  # Ne pas compter la classe correcte
loss = np.sum(margins)  # Somme sur les classes
```

## Le Problème de Régularisation
*[17:05 - 20:24]*

### Problème avec la Fonction de Perte de Base
Si W atteint une perte zéro, alors αW (α > 1) atteint aussi une perte zéro
- Sous-espace entier de W optimaux
- Propriété non désirable
- Besoin de préférence parmi les solutions équivalentes

### Solution : Régularisation
*[20:24 - 24:25]*

**Perte complète = Perte de données + λ·R(W)**

Plus commune : Régularisation L2 (décroissance des poids)
- R(W) = Σᵢⱼ W²ᵢⱼ
- Préfère les petits poids (W ≈ 0)
- Force un compromis entre ajuster les données et avoir un "bon" W

### Exemple de Régularisation
*[22:17 - 27:01]*

Étant donné x = [1,1,1,1] et deux vecteurs de poids :
- W₁ = [1,0,0,0]
- W₂ = [0.25,0.25,0.25,0.25]

Les deux donnent le même score (produit scalaire = 1) mais :
- La régularisation L2 préfère W₂
- W₂ utilise toutes les caractéristiques d'entrée
- W₁ ignore 3/4 des entrées
- Les poids diffus fonctionnent généralement mieux au moment du test

## Classificateur Softmax
*[27:42 - 35:36]*

### Interprétation
- Scores = probabilités logarithmiques non normalisées
- Pour obtenir les probabilités : **P(Y=k|X) = e^(sₖ) / Σⱼ e^(sⱼ)**
- C'est la fonction softmax

### Fonction de Perte
**Perte = -log P(Y=yᵢ|X)**
- Maximiser la vraisemblance logarithmique de la classe correcte
- Minimiser la vraisemblance logarithmique négative

### Exemple de Calcul
*[32:05 - 33:23]*

Étant donné les scores [3.2, 5.1, -1.7] :
1. Exponentialiser : [24.5, 164.0, 0.18]
2. Normaliser : [0.13, 0.87, 0.00]
3. Perte = -log(0.13) = 0.89

### Propriétés
- Perte min : 0 (quand P = 1 pour la classe correcte)
- Perte max : ∞ (quand P → 0 pour la classe correcte)
- Avec petit W aléatoire : perte ≈ -log(1/C) où C = nombre de classes

## Comparaison SVM vs Softmax
*[37:58 - 45:06]*

Différence clé dans le comportement :
- **SVM** : Une fois la marge satisfaite, perte = 0 indépendamment de la magnitude du score
  - A une région "satisfaite" où il ne se soucie pas d'amélioration supplémentaire
  - Plus robuste aux valeurs aberrantes
  
- **Softmax** : Veut toujours améliorer les probabilités
  - Jamais complètement satisfait
  - Continue à pousser la probabilité de la classe correcte → 1

## Démonstration Interactive
*[45:37 - 50:04]*

Démo web montrant :
- Problème de classification 2D avec 3 classes
- Visualisation des frontières de décision
- Mises à jour des paramètres en temps réel
- Perte décroissante pendant l'optimisation
- Effet de la force de régularisation
- Comparaison des pertes SVM et softmax

## Optimisation
*[50:13 - 58:35]*

### Le Problème d'Optimisation
- Donné : Données (X, Y) - fixées
- Contrôle : Poids W
- Objectif : Trouver W qui minimise la perte

### Analogie
Comme être aveuglé sur un paysage de perte :
- Position actuelle = W actuel
- Peut mesurer l'altitude (perte) à n'importe quel point
- Objectif : Trouver la vallée (perte minimale)

### Stratégies d'Optimisation

#### 1. Recherche Aléatoire (Ne pas utiliser !)
*[51:24 - 52:13]*
- Échantillonner des W aléatoires
- Évaluer la perte pour chacun
- Garder le meilleur
- Atteint ~15.5% sur CIFAR-10 (vs 10% au hasard)

#### 2. Descente de Gradient
*[52:55 - 57:29]*

**Gradient** : Vecteur de dérivées partielles
- ∇L = [∂L/∂w₁, ∂L/∂w₂, ..., ∂L/∂wₙ]

**Gradient numérique** (différences finies) :
- Approximatif : [f(x+h) - f(x)] / h
- Lent : Besoin d'évaluer pour chaque dimension
- Utilisé pour vérifier le gradient

**Gradient analytique** :
- Utiliser le calcul pour dériver la formule exacte
- Rapide : Une seule évaluation donne le gradient complet
- Sujet aux erreurs : Doit dériver correctement

### Mises à Jour des Paramètres
*[58:42 - 1:00:37]*

Règle de mise à jour de base :
```
W = W - α·∇L
```
- α = taille du pas (taux d'apprentissage)
- Hyperparamètre le plus critique
- Trop élevé : Instabilité
- Trop bas : Convergence lente

### Descente de Gradient par Mini-lots
*[1:00:44 - 1:02:05]*

Au lieu du jeu de données complet :
- Échantillonner de petits lots (32, 64, 128, 256 exemples)
- Calculer le gradient sur le lot
- Mettre à jour les paramètres
- Répéter

Avantages :
- Plus de mises à jour avec des gradients approximatifs
- Plus efficace computationnellement
- Taille du lot généralement déterminée par la mémoire GPU

### Effets du Taux d'Apprentissage
*[1:03:43 - 1:04:54]*

- **Taux d'apprentissage élevé** : Agitation, peut ne pas converger ou exploser
- **Taux d'apprentissage bas** : Convergence très lente
- **Pratique courante** : Commencer haut, décroître avec le temps

### Méthodes de Mise à Jour
*[1:05:00 - 1:06:30]*

Au-delà du SGD de base :
- **Momentum** : Accumuler la vitesse dans les directions de gradient cohérentes
- **AdaGrad, RMSProp, Adam** : Taux d'apprentissage adaptatifs
- Différentes méthodes convergent différemment sur différents problèmes

## Contexte Historique : Vision par Ordinateur Avant l'Apprentissage Profond
*[1:06:38 - 1:11:02]*

### Pipeline Traditionnel (pré-2012)
1. **Extraction de Caractéristiques** :
   - Histogrammes de couleurs
   - Caractéristiques SIFT/HOG (orientations des bords)
   - Sac de Mots
   - Nombreuses caractéristiques conçues à la main

2. **Classification** :
   - Concaténer toutes les caractéristiques
   - Entraîner un classificateur linéaire (souvent SVM)

### Approche Moderne (post-2012)
- Commencer avec les pixels bruts
- Architecture différentiable unique
- Entraîner de bout en bout
- Apprendre les extracteurs de caractéristiques automatiquement
- Éliminer les composants conçus à la main

## Prochaines Étapes
*[1:11:02 - 1:11:20]*

Le prochain cours couvrira :
- Rétropropagation : Calcul efficace des gradients analytiques
- Introduction aux réseaux de neurones
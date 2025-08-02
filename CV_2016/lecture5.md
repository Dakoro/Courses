# Comprendre en Profondeur l'Entraînement des Réseaux de Neurones pour la Vision par Ordinateur

## Introduction

La vision par ordinateur, un domaine en pleine effervescence, repose aujourd'hui massivement sur les **réseaux de neurones profonds**. Ces architectures complexes, capables d'apprendre des représentations hiérarchiques des données, ont révolutionné des tâches allant de la classification d'images à la détection d'objets et à la segmentation sémantique. L'efficacité de ces systèmes dépend de manière critique de leur processus d'entraînement.

Cette conférence vise à démystifier l'entraînement des réseaux de neurones, en allant au-delà des simples définitions pour explorer les mécanismes sous-jacents, les défis historiques et les solutions contemporaines. L'analyse abordera :
- Les fonctions d'activation
- Le pré-traitement des données
- L'initialisation des poids
- La normalisation par lots
- Les stratégies pratiques de débogage et d'optimisation des hyperparamètres

Un accent particulier sera mis sur leur implémentation et leur impact sur la performance des modèles en vision par ordinateur.

## I. Contexte Historique et Évolution des Réseaux de Neurones

L'histoire des réseaux de neurones est jalonnée de périodes d'intense recherche et de phases de relatif silence, reflétant les défis et les percées successives qui ont façonné le domaine.

### Les prémices : Perceptron, Adaline et Madaline

Les origines des réseaux de neurones remontent aux années 1950. Le **Perceptron**, introduit par Frank Rosenblatt en 1957, fut une implémentation matérielle pionnière. Ce modèle utilisait :
- Une fonction d'activation binaire (fonction échelon)
- Une sortie de 0 ou 1
- Des règles heuristiques pour ajuster les poids

> La nature non-différentiable de cette fonction constituait une limitation majeure, empêchant l'application de la rétropropagation des erreurs.

Au début des années 1960, Woodrow et Huff ont développé **Adaline** et **Madaline**, marquant l'avènement des premiers réseaux de perceptrons multi-couches. L'enthousiasme initial des années 1960 a cependant conduit à des attentes excessives et à une performance sous-estimée, entraînant une période de stagnation dans les années 1970.

### La renaissance de la rétropropagation

Un tournant significatif s'est produit en **1986** avec la publication d'un article influent par **Rumelhart, Hinton et Williams**. Ce travail a présenté :
- Des règles s'apparentant à la rétropropagation
- La formulation de fonctions de perte
- La discussion de la descente de gradient

Malgré cette avancée conceptuelle majeure, la mise à l'échelle restait un défi, entraînant une nouvelle période de recherche ralentie pendant environ deux décennies.

### L'ère du "Deep Learning" : pré-entraînement non supervisé et les percées majeures de 2010-2012

La recherche a été ravivée en **2006** par un article de Hinton et Salakhutdinov paru dans *Science*. Ils ont démontré la possibilité d'entraîner des réseaux plus profonds (10 couches) en utilisant :
- Un schéma de pré-entraînement non supervisé
- Des machines de Boltzmann restreintes
- Un processus en deux étapes (pré-entraînement + affinage)

Les véritables percées se sont produites entre **2010 et 2012** :
- **2010** : Améliorations significatives dans la reconnaissance vocale
- **2012** : **AlexNet** "écrase toute la concurrence" lors du concours ImageNet

### Facteurs clés du succès actuel

Le succès actuel résulte d'une convergence de plusieurs facteurs :

1. **Données massives** : Disponibilité de jeux de données comme ImageNet
2. **Puissance de calcul** : Utilisation des GPU pour l'entraînement
3. **Avancées algorithmiques** :
   - Meilleures méthodes d'initialisation des poids
   - Fonctions d'activation plus efficaces
   - Techniques d'optimisation améliorées

> L'histoire des réseaux de neurones est caractérisée par des cycles de "sur-promesse et sous-livraison", suivis de périodes de silence, puis de renaissances.

## II. Fondamentaux de l'Entraînement des Réseaux de Neurones

L'entraînement d'un réseau de neurones est un processus complexe d'optimisation visant à ajuster les paramètres du modèle pour minimiser une fonction de perte.

### Le cadre de la descente de gradient stochastique par mini-lots (Mini-Batch SGD)

L'entraînement s'inscrit dans le cadre de la **Mini-Batch SGD**, un processus itératif en quatre étapes :

1. **Échantillonnage des données** : Sélection d'un mini-lot du jeu d'entraînement
2. **Passe avant (Forward Pass)** : 
   - Propagation du mini-lot à travers le réseau
   - Transformations linéaires suivies d'activations non-linéaires
   - Calcul de la fonction de perte
3. **Rétropropagation (Backward Pass)** :
   - Calcul des gradients via la règle de la chaîne
   - Propagation de l'erreur de la sortie vers l'entrée
4. **Mise à jour des paramètres** :
   - Ajustement dans la direction opposée au gradient
   - Multiplication par le taux d'apprentissage

### Le rôle crucial de la fonction de perte et de la rétropropagation

Ce processus itératif est fondamentalement un **problème d'optimisation**. L'objectif est de trouver un ensemble de poids et biais qui minimise la fonction de perte.

> L'entraînement est décrit comme un problème d'optimisation où l'on converge vers des zones de "faible perte" dans l'espace des poids.

La **rétropropagation** est le mécanisme clé permettant :
- Le calcul des gradients pour chaque opération locale
- L'ajustement de milliards de paramètres
- La navigation dans un espace de très haute dimension

## III. Fonctions d'Activation : Introduction de la Non-Linéarité

Les fonctions d'activation sont des composants essentiels introduisant la non-linéarité nécessaire pour permettre aux réseaux de neurones d'apprendre des relations complexes.

### Propriétés souhaitables

- Non-linéarité
- Différentiabilité (pour la rétropropagation)
- Efficacité de calcul
- Capacité à ne pas saturer

### Sigmoïde

La fonction sigmoïde : **σ(x) = 1/(1+e^(-x))**

```python
FONCTION Sigmoid(x):
    RENVOYER 1 / (1 + EXP(-x))

FONCTION Sigmoid_Derivative(output_of_sigmoid):
    # La dérivée de la sigmoïde est f(x) * (1 - f(x))
    RENVOYER output_of_sigmoid * (1 - output_of_sigmoid)
```

**Inconvénients** :
- **Saturation des neurones** → gradients évanescents
- **Sorties non centrées en zéro** → optimisation en "zigzag"
- **Fonction exponentielle coûteuse**

### Tanh (Tangente Hyperbolique)

La fonction tanh : **tanh(x) = (e^x - e^(-x))/(e^x + e^(-x))**

```python
FONCTION Tanh(x):
    RENVOYER (EXP(x) - EXP(-x)) / (EXP(x) + EXP(-x))

FONCTION Tanh_Derivative(output_of_tanh):
    # La dérivée de la Tanh est 1 - f(x)^2
    RENVOYER 1 - (output_of_tanh * output_of_tanh)
```

**Avantage** : Sorties centrées en zéro  
**Inconvénient** : Problème de saturation persistant

### ReLU (Rectified Linear Unit)

Introduite en 2012, définie par : **ReLU(x) = max(0,x)**

```python
FONCTION ReLU(x):
    SI x > 0 ALORS
        RENVOYER x
    SINON
        RENVOYER 0
    FIN SI

FONCTION ReLU_Derivative(x):
    SI x > 0 ALORS
        RENVOYER 1
    SINON
        RENVOYER 0
    FIN SI
```

**Avantages** :
- ✓ Pas de saturation dans la région positive
- ✓ Extrêmement efficace en calcul
- ✓ Convergence jusqu'à 6x plus rapide

**Inconvénients** :
- ✗ Sorties non centrées en zéro
- ✗ **"Neurones morts" (Dying ReLU)** : jusqu'à 10-20% peuvent devenir inactifs
- ✗ Non différentiable en x=0

### Variantes de ReLU

#### Leaky ReLU

**LeakyReLU(x) = x** si x>0, **αx** si x≤0 (α ≈ 0.01)

```python
FONCTION LeakyReLU(x, alpha=0.01):
    SI x > 0 ALORS
        RENVOYER x
    SINON
        RENVOYER alpha * x
    FIN SI
```

#### PReLU (Parametric ReLU)

Généralise Leaky ReLU avec **α apprenable** par rétropropagation

```python
FONCTION PReLU(x, alpha_i): # alpha_i est un paramètre apprenable
    SI x > 0 ALORS
        RENVOYER x
    SINON
        RENVOYER alpha_i * x
    FIN SI
```

#### ELU (Exponential Linear Units)

**f(x) = x** si x≥0, **α(e^x - 1)** si x<0

```python
FONCTION ELU(x, alpha=1.0):
    SI x >= 0 ALORS
        RENVOYER x
    SINON
        RENVOYER alpha * (EXP(x) - 1)
    FIN SI
```

#### Maxout Neuron

Calcule le maximum de plusieurs fonctions linéaires :
**max(W₁ᵀX + B₁, W₂ᵀX + B₂)**

```python
FONCTION Maxout(x, W1, B1, W2, B2):
    linear_output1 = DOT_PRODUCT(W1, x) + B1
    linear_output2 = DOT_PRODUCT(W2, x) + B2
    RENVOYER MAX(linear_output1, linear_output2)
```

### Synthèse et Recommandation

| Fonction | Recommandation | Raison |
|----------|----------------|---------|
| Sigmoïde | ❌ Non recommandée | Saturation, non-centrage |
| Tanh | ⚠️ Préférable à sigmoïde | Centrée mais saturation |
| **ReLU** | ✅ **Par défaut** | Simple, efficace, convergence rapide |
| Variantes ReLU | 🔍 À explorer | Peuvent résoudre les neurones morts |

## IV. Pré-traitement des Données

Un pré-traitement approprié est fondamental pour un entraînement efficace.

### Importance et objectifs

- Optimiser la dynamique d'apprentissage
- Accélérer la convergence
- Améliorer la robustesse du modèle
- Réduire la sensibilité aux hyperparamètres

### Techniques générales

1. **Centrage en zéro** : Soustraire la moyenne de chaque dimension
2. **Normalisation/Standardisation** : Diviser par l'écart type → variance unitaire
3. **PCA/Whitening** : Décorrélation des caractéristiques (moins courant pour les images)

### Spécificités pour les images

Pour la vision par ordinateur, les pratiques courantes sont :

1. **Soustraction de l'image moyenne** :
   - Calculer la moyenne pixel par pixel sur le jeu d'entraînement
   - Soustraire cette "image moyenne"

2. **Soustraction de la moyenne par canal** (Recommandé) :
   - Calculer la moyenne pour R, G, B séparément
   - Plus simple et pratique (3 nombres au lieu d'une image)

> En vision par ordinateur, la soustraction de la moyenne est généralement suffisante. Les techniques complexes (PCA, blanchiment) sont rarement utilisées sur des images entières.

## V. Initialisation des Poids : Un Défi Crucial

L'initialisation des poids est d'une importance capitale, souvent sous-estimée.

### Pourquoi l'initialisation aléatoire est nécessaire

❌ **Initialiser tous les poids à zéro** = Erreur fondamentale
- Tous les neurones produisent la même sortie
- Pas de rupture de symétrie
- Impossible d'apprendre des représentations diverses

### Problèmes des petites/grandes valeurs aléatoires

| Initialisation | Problème | Conséquence |
|----------------|----------|-------------|
| **Trop petites** (σ=0.01) | Activations → 0 | Gradients évanescents |
| **Trop grandes** (σ=1.0) | Saturation rapide | Gradients évanescents aussi |

### Initialisation de Xavier (Glorot et al., 2010)

**Principe** : Maintenir une variance constante des activations

```
Poids = random_normal() / sqrt(n_inputs)
```

- ✓ Fonctionne bien avec tanh
- ✗ Limitation : ne considère pas les non-linéarités (ReLU)

### Initialisation de Kaiming/MSRA (He et al., 2015)

**Adaptation pour ReLU** : Compte tenu que ReLU divise la variance par 2

```
Poids = random_normal() / sqrt(n_inputs / 2)
```

- ✓ Crucial pour les réseaux ReLU très profonds
- ✓ Empêche l'effondrement des activations

### Recherche active et techniques basées sur les données

- Ajustement itératif basé sur les statistiques d'activation
- Objectif : maintenir les variances proches de l'unité gaussienne
- Domaine de recherche toujours actif

## VI. Normalisation par Lots (Batch Normalization)

Proposée en 2015, la **Batch Normalization** a considérablement amélioré la stabilité et la vitesse d'entraînement.

### Concept et objectif

**Idée fondamentale** : Forcer les activations à suivre une distribution gaussienne unitaire (μ=0, σ=1) à travers le mini-lot

**Objectif** : Résoudre le "changement de covariance interne" (*internal covariate shift*)

### Mécanisme

Les couches de Batch Norm sont insérées :
- Après les couches FC/Conv
- Avant la fonction d'activation

#### Passe avant (Forward Pass)

Pour chaque dimension j :

1. Moyenne empirique : **μⱼ = (1/N) Σᵢ Xᵢⱼ**
2. Variance empirique : **σⱼ² = (1/N) Σᵢ (Xᵢⱼ - μⱼ)²**
3. Normalisation : **X̂ᵢⱼ = (Xᵢⱼ - μⱼ) / √(σⱼ² + ε)**
4. Transformation : **Yᵢⱼ = γⱼX̂ᵢⱼ + βⱼ**

Où **γ** et **β** sont des paramètres apprenables.

```python
FONCTION BatchNorm_Forward(X, Gamma, Beta, Epsilon):
    N_batch, D_features = X.DIMENSIONS()
    
    # 1. Calculer la moyenne empirique
    Mu = MOYENNE(X, axe=0)
    
    # 2. Soustraire la moyenne
    X_minus_Mu = X - Mu
    
    # 3. Calculer la variance empirique
    Variance = MOYENNE(X_minus_Mu ** 2, axe=0)
    
    # 4. Calculer l'écart-type inverse
    Inv_StdDev = 1 / RACINE_CARREE(Variance + Epsilon)
    
    # 5. Normaliser
    X_hat = X_minus_Mu * Inv_StdDev
    
    # 6. Appliquer échelle et décalage
    Output = Gamma * X_hat + Beta
    
    RENVOYER Output, Cache
```

### Avantages

1. **Amélioration du flux de gradient** ✓
2. **Taux d'apprentissage plus élevés** ✓
3. **Moins sensible à l'initialisation** ✓
4. **Effet de régularisation** ✓

### Comportement à l'inférence

- **Entraînement** : Statistiques calculées sur chaque mini-lot
- **Inférence** : Utilisation de statistiques fixes (moyenne mobile ou population)

### Inconvénient

⚠️ **Pénalité de temps d'exécution** due aux calculs supplémentaires

> Malgré le surcoût, Batch Norm est fortement recommandée pour les réseaux profonds.

## VII. Stratégies Pratiques de Débogage et d'Optimisation des Hyperparamètres

L'entraînement des réseaux de neurones est autant un art qu'une science.

### Vérifications de base (Sanity Checks)

1. **Vérifier la perte initiale** :
   - Désactiver la régularisation
   - Perte attendue pour softmax C classes : `-log(1/C)`
   - Ex : 2.3 pour 10 classes

2. **Vérifier l'effet de la régularisation** :
   - Augmenter la force → la perte doit augmenter

3. **Sur-ajustement d'un petit jeu** :
   - ~20 exemples
   - Objectif : perte ≈ 0, précision = 100%
   - Si échec → erreur d'implémentation

### Recherche du taux d'apprentissage

| Taux | Symptôme | Action |
|------|----------|---------|
| **Trop faible** (1e-6) | Perte diminue à peine | Augmenter |
| **Trop élevé** (1e6) | Explosion, NaN | Diminuer |

**Stratégie** : Recherche grossière → fine

### Optimisation des hyperparamètres

1. **Échantillonnage aléatoire > Grid Search**
   - Plus efficace pour explorer l'espace
   - Meilleure chance de trouver les paramètres importants

2. **Échantillonnage en espace log** :
   ```
   learning_rate = 10^uniform(-6, -3)
   ```

3. **Arrêt précoce** : Limiter les époques en phase exploratoire

4. **Vérification des limites** : Si meilleurs résultats aux bornes → étendre la plage

### Surveillance du processus d'entraînement

#### Courbes de perte

- **Diminution linéaire** → Taux d'apprentissage trop faible
- **Plateaux** → Initialisation incorrecte
- **Oscillations/Explosions** → Taux trop élevé

#### Courbes de précision

- Écart train/validation important → **Sur-ajustement**
- Solution : Augmenter la régularisation

#### Ratio poids/mises à jour

**Règle empirique** : Ratio ≈ 1e-3
- Trop élevé → Diminuer le taux d'apprentissage
- Trop faible → Augmenter le taux d'apprentissage

### Hyperparamètres clés à optimiser

1. **Taux d'apprentissage** (le plus important)
2. **Type de mise à jour** (SGD, Adam, RMSprop)
3. **Force de régularisation**
4. **Quantités de dropout**

## Conclusion

L'entraînement des réseaux de neurones profonds pour la vision par ordinateur représente une convergence remarquable de théorie et de pratique. Les éléments clés à retenir sont :

### Évolution historique
- Cycles de promesses et déceptions
- Percées = convergences de multiples facteurs
- Importance de l'expérimentation pratique

### Composants fondamentaux
1. **Fonctions d'activation** : ReLU par défaut, attention aux neurones morts
2. **Pré-traitement** : Simple mais fondamental (centrage)
3. **Initialisation** : Xavier/Kaiming selon l'activation
4. **Batch Normalization** : Technique transformative malgré le surcoût

### Approche pratique
- Vérifications de base systématiques
- Recherche méthodique des hyperparamètres
- Surveillance continue du processus

> La compréhension approfondie de ces concepts et l'application méthodique de ces techniques sont essentielles pour maîtriser l'art et la science de l'entraînement des réseaux de neurones.

Le domaine continue d'évoluer rapidement, mais ces principes fondamentaux constituent la base sur laquelle les innovations futures seront bâties.
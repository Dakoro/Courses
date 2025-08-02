# La Régression Linéaire : Des Fondations aux Algorithmes d'Optimisation

## Section 1: Introduction à l'Apprentissage Supervisé et à la Régression

### 1.1 Le paradigme de l'apprentissage supervisé (supervised learning)

Au cœur de la plupart des avancées modernes en intelligence artificielle se trouve un paradigme fondamental connu sous le nom d'apprentissage automatique, ou machine learning. L'une de ses branches les plus importantes et les plus largement utilisées est l'apprentissage supervisé (supervised learning en anglais). Le concept central de l'apprentissage supervisé est d'entraîner un algorithme en lui fournissant un jeu de données, appelé jeu d'entraînement (training set), où chaque exemple est accompagné de la "bonne réponse".

Formellement, chaque point de données dans un jeu d'entraînement supervisé est une paire $(x, y)$.
- $x$ représente les données d'entrée, un ensemble de caractéristiques (ou features) qui décrivent l'exemple. Par exemple, dans le contexte d'une voiture autonome, $x$ pourrait être une image de la route.
- $y$ représente la sortie désirée, l'étiquette ou la variable cible (target variable) que nous voulons que le modèle apprenne à prédire. Pour la voiture autonome, $y$ serait l'angle de braquage correct correspondant à l'image $x$.

Le processus d'apprentissage supervisé consiste à alimenter ce jeu de données d'entraînement à un algorithme d'apprentissage. Le rôle de cet algorithme est d'analyser ces paires $(x, y)$ et de produire une fonction. Par convention en apprentissage automatique, cette fonction de prédiction est appelée une hypothèse, notée $h$. L'objectif de cette hypothèse $h$ est de pouvoir prendre une nouvelle entrée $x$ (une image de route que l'algorithme n'a jamais vue auparavant) et de générer une prédiction pour la sortie $y$ (l'angle de braquage approprié) qui soit aussi précise que possible.

La qualité de la supervision, c'est-à-dire la précision et la pertinence des étiquettes $y$ dans le jeu de données, détermine directement et de manière causale la performance maximale que le modèle résultant peut atteindre. Des données d'entraînement de mauvaise qualité, bruitées ou incorrectes conduiront inévitablement à une hypothèse défaillante, indépendamment de la sophistication de l'algorithme. C'est le principe fondamental du "garbage in, garbage out" (déchets en entrée, déchets en sortie), essentiel à comprendre dès le départ.

### 1.2 Problèmes de régression vs. classification

Les problèmes d'apprentissage supervisé se divisent principalement en deux catégories, basées sur la nature de la variable cible $y$.

**Problèmes de Régression** : La variable cible $y$ est une valeur continue. L'objectif est de prédire une quantité numérique. L'exemple de la voiture autonome où l'on prédit un angle de braquage (un nombre réel) est un problème de régression. De même, prédire le prix d'une maison, la température de demain, ou le chiffre d'affaires d'une entreprise sont des problèmes de régression.

**Problèmes de Classification** : La variable cible $y$ est une valeur discrète, appartenant à un ensemble fini de catégories. L'objectif est d'attribuer une étiquette à une entrée. Par exemple, prédire si un e-mail est un "spam" ou "non-spam", ou identifier si une image contient un "chat", un "chien" ou un "oiseau" sont des problèmes de classification.

Ce cours se concentrera sur le premier type de problème et introduira l'algorithme de régression le plus fondamental : la régression linéaire (régression linéaire en français).

### 1.3 Cas d'étude motivant : L'estimation des prix de l'immobilier

Pour illustrer les concepts de manière concrète, nous utiliserons un exemple plus simple que la conduite autonome : la prédiction des prix de l'immobilier. Cet exemple, basé sur des données réelles de la ville de Portland, Oregon, servira de fil conducteur tout au long de cette leçon.

Imaginons que nous ayons collecté un jeu de données sur des maisons, contenant leur surface habitable et leur prix de vente.

| Surface (pieds carrés) | Prix (en milliers de $) |
|------------------------|-------------------------|
| 2104                   | 400                     |
| 1416                   | 232                     |
| 1534                   | 315                     |
| 852                    | 178                     |
| ...                    | ...                     |

Si nous visualisons ces données sur un graphique, avec la surface en abscisse et le prix en ordonnée, nous obtenons un nuage de points. L'objectif intuitif de la régression linéaire dans ce cas est de trouver une ligne droite qui "s'ajuste" au mieux à ce nuage de points. Cette droite représentera notre modèle de prédiction : pour une nouvelle surface de maison, nous pourrons utiliser la droite pour estimer son prix. En termes plus formels, nous cherchons à réaliser un ajustement affine.

## Section 2: Formalisation du Modèle de Régression Linéaire

Pour construire notre algorithme, nous devons traduire cette intuition en un langage mathématique formel. Cela implique de définir précisément notre hypothèse, nos paramètres et la manière dont nous mesurons la performance de notre modèle.

### 2.1 L'hypothèse h(x) : Représenter le modèle

La première étape dans la conception d'un algorithme d'apprentissage est de définir la forme de notre fonction d'hypothèse $h$. Pour la régression linéaire, comme son nom l'indique, nous choisissons une fonction linéaire (ou plus précisément, affine) des variables d'entrée.

**Cas univarié (une seule variable)** : Si nous n'avons qu'une seule caractéristique d'entrée, comme la surface ($x$), notre hypothèse s'écrit :

$$h_\theta(x) = \theta_0 + \theta_1 x$$

Ici, $x$ est la surface de la maison, et $h_\theta(x)$ est le prix prédit. Les valeurs $\theta_0$ et $\theta_1$ sont les paramètres du modèle, qui correspondent respectivement à l'ordonnée à l'origine et à la pente de notre droite. Le terme hypothèse est une convention historique en apprentissage automatique ; il faut le comprendre ici comme le modèle de prédiction paramétré.

**Cas multivarié (plusieurs variables)** : En pratique, le prix d'une maison dépend de plusieurs facteurs. Nous pourrions avoir des caractéristiques supplémentaires comme le nombre de chambres ($x_2$), l'âge de la maison ($x_3$), etc. Si nous avons $n$ caractéristiques, notre hypothèse se généralise comme suit :

$$h_\theta(x) = \theta_0 + \theta_1 x_1 + \theta_2 x_2 + \cdots + \theta_n x_n$$

Dans cette équation, $x_1$ est la valeur de la première caractéristique, $x_2$ celle de la deuxième, et ainsi de suite.

### 2.2 Les paramètres θ et la notation vectorielle

Les $\theta$ (thêta) sont les paramètres, aussi appelés "poids", que l'algorithme doit apprendre à partir des données. L'objectif est de trouver les valeurs de $\theta_0, \theta_1, ..., \theta_n$ qui font que notre hypothèse $h_\theta(x)$ produit des prédictions proches des vrais prix $y$ pour les exemples de notre jeu d'entraînement.

Pour simplifier la notation, nous introduisons une convention très importante : nous définissons une caractéristique d'entrée factice, $x_0$, qui est toujours égale à 1 ($x_0 = 1$). Grâce à cette astuce, nous pouvons réécrire notre hypothèse de manière beaucoup plus compacte.

L'hypothèse devient :

$$h_\theta(x) = \theta_0 x_0 + \theta_1 x_1 + \theta_2 x_2 + \dots + \theta_n x_n = \sum_{j=0}^{n} \theta_j x_j$$

Cette somme est un produit scalaire entre deux vecteurs :
- Le vecteur des paramètres θ : $\theta = [\theta_0, \theta_1, \dots, \theta_n]^T$
- Le vecteur des caractéristiques x : $x = [x_0, x_1, \dots, x_n]^T$

L'hypothèse peut donc s'écrire sous une forme vectorielle très élégante :

$$h_\theta(x) = \theta^T x$$

L'introduction de $x_0 = 1$ n'est pas une simple commodité de notation ; c'est une transformation géométrique. Elle projette l'espace des caractéristiques de $n$ dimensions dans un espace de $n+1$ dimensions. Dans ce nouvel espace, un hyperplan linéaire passant par l'origine correspond à un hyperplan affine (qui peut avoir une ordonnée à l'origine non nulle) dans l'espace original à $n$ dimensions. Cela nous permet d'utiliser toute la puissance de l'algèbre linéaire, qui opère sur des transformations linéaires, pour résoudre un problème qui était initialement affine.

Pour clarifier la terminologie que nous utiliserons :
- $m$ : Le nombre total d'exemples dans le jeu d'entraînement.
- $n$ : Le nombre de caractéristiques (sans compter $x_0$).
- $x^{(i)}$ : Le vecteur des caractéristiques du $i$-ème exemple d'entraînement. Le $(i)$ en exposant est un index, pas une puissance.
- $x_j^{(i)}$ : La valeur de la $j$-ème caractéristique dans le $i$-ème exemple d'entraînement.
- $y^{(i)}$ : La valeur cible pour le $i$-ème exemple d'entraînement.

### 2.3 La fonction de coût J(θ) : Mesurer l'erreur du modèle

Maintenant que nous avons une hypothèse, comment savoir si un ensemble de paramètres $\theta$ est bon ou mauvais? Nous avons besoin d'une mesure quantitative de la performance du modèle. C'est le rôle de la fonction de coût (cost function), aussi appelée fonction de perte (loss function) ou fonction d'erreur (error function).

L'objectif de l'algorithme d'apprentissage est de trouver les valeurs des paramètres $\theta$ qui minimisent cette fonction de coût. Pour la régression linéaire, la fonction de coût la plus courante est l'erreur quadratique moyenne, qui est au cœur de la méthode des moindres carrés ordinaires (ordinary least squares).

La fonction de coût, notée $J(\theta)$, est définie comme suit :

$$J(\theta) = \frac{1}{2m} \sum_{i=1}^{m} (h_\theta(x^{(i)}) - y^{(i)})^2$$

Analysons cette formule :
- $(h_\theta(x^{(i)}) - y^{(i)})$ : C'est l'erreur de prédiction pour le $i$-ème exemple, c'est-à-dire la différence entre le prix prédit et le prix réel.
- $(...)^2$ : Nous élevons cette erreur au carré. Cela a deux effets : premièrement, les erreurs sont toujours positives ; deuxièmement, les grandes erreurs sont pénalisées beaucoup plus lourdement que les petites.
- $\sum(...)$ : Nous sommons les erreurs quadratiques pour tous les $m$ exemples de notre jeu d'entraînement.
- $1/m$ : Nous divisons par $m$ pour obtenir l'erreur moyenne. Cela rend la fonction de coût indépendante de la taille du jeu de données.
- $1/2$ : Ce facteur est une convention mathématique. Comme nous le verrons, il simplifie le calcul de la dérivée lors de l'optimisation, car la dérivée de $x^2$ est $2x$, et le 2 s'annulera avec le $1/2$. Minimiser $J(\theta)$ ou $(1/2)J(\theta)$ conduit au même résultat pour $\theta$.

Pour la régression linéaire, cette fonction de coût $J(\theta)$ est une fonction convexe. Visuellement, si on la traçait en fonction de $\theta_0$ et $\theta_1$, elle ressemblerait à un bol (bowl-shaped). Cette propriété est extrêmement importante car elle garantit qu'il n'existe pas de minima locaux piégeux, mais un seul et unique minimum global. Notre tâche d'optimisation consistera donc à trouver le point le plus bas de ce bol.

### 2.4 Justification Probabiliste de l'Erreur Quadratique

Le choix de l'erreur quadratique moyenne n'est pas arbitraire. Il repose sur des fondements probabilistes solides. Minimiser la somme des erreurs au carré est équivalent à trouver l'Estimateur du Maximum de Vraisemblance (Maximum Likelihood Estimate ou MLE) pour les paramètres $\theta$, sous l'hypothèse que les erreurs de notre modèle suivent une distribution Gaussienne (ou normale).

Le raisonnement est le suivant :

1. Supposons que la relation entre les caractéristiques et la cible est linéaire, mais affectée par un bruit aléatoire. Autrement dit, $y^{(i)} = \theta^T x^{(i)} + \epsilon^{(i)}$, où $\epsilon^{(i)}$ est le terme d'erreur.

2. Faisons l'hypothèse que ces erreurs $\epsilon^{(i)}$ sont indépendantes et identiquement distribuées (I.I.D.) selon une distribution Gaussienne de moyenne 0 et de variance $\sigma^2$. On note cela $\epsilon^{(i)} \sim \mathcal{N}(0, \sigma^2)$.

3. Sous cette hypothèse, la probabilité de l'erreur $\epsilon^{(i)}$ est donnée par la densité de probabilité Gaussienne. Par conséquent, la probabilité de $y^{(i)}$ étant donné $x^{(i)}$ et les paramètres $\theta$ est aussi Gaussienne :

$$p(y^{(i)}|x^{(i)};\theta) = \frac{1}{\sqrt{2\pi}\sigma} \exp\left(-\frac{(y^{(i)} - \theta^T x^{(i)})^2}{2\sigma^2}\right)$$

4. La vraisemblance (likelihood) de l'ensemble des données, $L(\theta)$, est la probabilité d'observer toutes les cibles $y^{(i)}$ étant donné toutes les entrées $x^{(i)}$ et les paramètres $\theta$. Puisque les exemples sont I.I.D., c'est le produit des probabilités individuelles :

$$L(\theta) = \prod_{i=1}^{m} p(y^{(i)}|x^{(i)};\theta)$$

5. L'objectif du MLE est de trouver les $\theta$ qui maximisent cette vraisemblance. Il est mathématiquement plus simple de maximiser le logarithme de la vraisemblance (la log-vraisemblance), car cela transforme le produit en une somme :

$$\log L(\theta) = \sum_{i=1}^{m} \log p(y^{(i)}|x^{(i)};\theta)$$

En remplaçant par l'expression de la densité Gaussienne et en simplifiant, on constate que maximiser $\log L(\theta)$ revient à minimiser le terme :

$$\sum_{i=1}^{m} (y^{(i)} - \theta^T x^{(i)})^2$$

Ce terme est précisément la somme des erreurs au carré que l'on trouve dans notre fonction de coût $J(\theta)$. Ainsi, la méthode des moindres carrés n'est pas seulement un choix pratique ; c'est le choix de principe si l'on suppose que les erreurs de notre modèle sont distribuées normalement. Cela établit un lien puissant entre la régression linéaire et le domaine plus large de la modélisation probabiliste.

## Section 3: Optimisation par Descente de Gradient

Nous avons défini notre objectif : trouver les $\theta$ qui minimisent $J(\theta)$. Mais comment faire cela concrètement? L'algorithme le plus courant pour cette tâche est la descente de gradient (gradient descent en français).

### 3.1 Le principe de la descente de gradient

La descente de gradient est un algorithme d'optimisation itératif. L'intuition est simple : imaginez que vous êtes au sommet d'une montagne (la surface de la fonction de coût) et que vous voulez atteindre le point le plus bas (le minimum de $J(\theta)$). À chaque pas, vous regardez autour de vous et déterminez la direction de la pente la plus raide vers le bas, puis vous faites un petit pas dans cette direction. En répétant ce processus, vous descendrez progressivement la montagne jusqu'à atteindre une vallée.

Mathématiquement, la "direction de la pente la plus raide" est donnée par le gradient de la fonction de coût. Le gradient est un vecteur qui pointe dans la direction de la plus grande augmentation de la fonction. Pour descendre, nous devons donc nous déplacer dans la direction opposée au gradient.

La règle de mise à jour pour chaque paramètre $\theta_j$ à chaque itération est la suivante :

$$\theta_j := \theta_j - \alpha \frac{\partial}{\partial \theta_j} J(\theta)$$

Analysons cette règle :
- $:=$ est le symbole d'affectation. Nous mettons à jour la valeur de $\theta_j$.
- $\alpha$ (alpha) est le taux d'apprentissage (learning rate). C'est un hyperparamètre (un paramètre que nous devons choisir nous-mêmes) qui contrôle la taille du pas que nous faisons à chaque itération.
  - Si $\alpha$ est trop petit, la convergence sera très lente.
  - Si $\alpha$ est trop grand, nous risquons de "sauter" par-dessus le minimum et de diverger.
- $\frac{\partial}{\partial \theta_j} J(\theta)$ est la dérivée partielle de la fonction de coût par rapport au paramètre $\theta_j$. Ce terme nous donne la pente de la fonction de coût dans la direction de l'axe $\theta_j$.

### 3.2 Dérivation mathématique de la règle de mise à jour

Pour implémenter la descente de gradient, nous devons calculer le terme de la dérivée partielle. Reprenons notre fonction de coût :

$$J(\theta) = \frac{1}{2m} \sum_{i=1}^{m} (h_\theta(x^{(i)}) - y^{(i)})^2$$

Nous devons calculer $\frac{\partial}{\partial \theta_j} J(\theta)$. En utilisant la règle de dérivation en chaîne :

1. La dérivée de la somme est la somme des dérivées :

$$\frac{\partial}{\partial \theta_j} J(\theta) = \frac{1}{2m} \sum_{i=1}^{m} \frac{\partial}{\partial \theta_j} (h_\theta(x^{(i)}) - y^{(i)})^2$$

2. Appliquons la règle de la chaîne $(u^2)' = 2u \cdot u'$ :

$$\frac{\partial}{\partial \theta_j} J(\theta) = \frac{1}{2m} \sum_{i=1}^{m} 2 (h_\theta(x^{(i)}) - y^{(i)}) \cdot \frac{\partial}{\partial \theta_j} (h_\theta(x^{(i)}) - y^{(i)})$$

3. Le 2 s'annule avec le 1/2. La dérivée de $y^{(i)}$ par rapport à $\theta_j$ est nulle car $y^{(i)}$ ne dépend pas de $\theta$. Il nous reste à calculer $\frac{\partial}{\partial \theta_j} h_\theta(x^{(i)})$.

$$\frac{\partial}{\partial \theta_j} h_\theta(x^{(i)}) = \frac{\partial}{\partial \theta_j} \left( \sum_{k=0}^{n} \theta_k x_k^{(i)} \right)$$

4. Le seul terme de cette somme qui dépend de $\theta_j$ est $\theta_j x_j^{(i)}$. Sa dérivée par rapport à $\theta_j$ est simplement $x_j^{(i)}$.

En rassemblant tous les morceaux, nous obtenons l'expression finale de la dérivée partielle :

$$\frac{\partial}{\partial \theta_j} J(\theta) = \frac{1}{m} \sum_{i=1}^{m} (h_\theta(x^{(i)}) - y^{(i)}) x_j^{(i)}$$

La règle de mise à jour complète pour la descente de gradient est donc :

$$\theta_j := \theta_j - \alpha \frac{1}{m} \sum_{i=1}^{m} (h_\theta(x^{(i)}) - y^{(i)}) x_j^{(i)}$$

Cette mise à jour doit être effectuée simultanément pour tous les $j$ de 0 à $n$ à chaque étape de l'algorithme.

### 3.3 Descente de Gradient par Lots (Batch Gradient Descent)

La méthode que nous venons de décrire est appelée descente de gradient par lots (Batch Gradient Descent). Le terme "batch" (lot) signifie que pour calculer le gradient et effectuer une seule mise à jour des paramètres, nous devons examiner l'intégralité du jeu de données d'entraînement (la somme $\sum$ va de $i=1$ à $m$).

**Pseudo-code de l'algorithme**

Le pseudo-code pour la descente de gradient par lots est le suivant, en utilisant des conventions de pseudo-code françaises :

```
PROCÉDURE DescenteGradientLots(X, y, θ, α, nombre_itérations)
  m = nombre d'exemples dans X

  POUR iter DE 1 À nombre_itérations FAIRE
    // Calculer les prédictions pour tous les exemples
    hypothèses = X * θ // Multiplication matricielle

    // Calculer le vecteur d'erreurs
    erreurs = hypothèses - y

    // Calculer le gradient pour chaque paramètre
    gradient = (1/m) * X_transpose * erreurs

    // Mettre à jour les paramètres
    θ := θ - α * gradient
  FIN POUR

  RETOURNER θ
FIN PROCÉDURE
```

**Implémentation en Python avec commentaires détaillés**

L'implémentation en Python utilise numpy pour des opérations vectorielles efficaces, ce qui est beaucoup plus rapide que des boucles for.

```python
import numpy as np

def batch_gradient_descent(X, y, theta, alpha, num_iters):
    """
    Effectue une descente de gradient par lots pour trouver theta.

    Args:
        X (np.array): Matrice des caractéristiques (m, n+1), avec x0=1.
        y (np.array): Vecteur des cibles (m, 1).
        theta (np.array): Paramètres initiaux (n+1, 1).
        alpha (float): Taux d'apprentissage.
        num_iters (int): Nombre d'itérations.

    Returns:
        np.array: Les paramètres theta optimisés.
        list: L'historique de la fonction de coût à chaque itération.
    """
    m = len(y)  # Nombre d'exemples d'entraînement
    J_history = []

    for i in range(num_iters):
        # Calcul des prédictions (hypothèse) pour tous les exemples en une seule fois
        # C'est une opération vectorisée : h = X * theta
        predictions = X.dot(theta)

        # Calcul de l'erreur pour tous les exemples
        errors = predictions - y

        # Calcul du gradient
        # La formule est (1/m) * sum((h(x)-y)*x)
        # En version vectorisée, c'est (1/m) * X.T * errors
        gradient = (1/m) * X.T.dot(errors)

        # Mise à jour simultanée de tous les paramètres theta
        theta = theta - alpha * gradient

        # Calculer et sauvegarder le coût à cette itération (optionnel, pour le suivi)
        cost = (1/(2*m)) * np.sum(np.square(errors))
        J_history.append(cost)

    return theta, J_history

# Exemple d'utilisation
# Supposons que X, y, theta_initial, alpha, iterations sont définis
# theta_optimise, couts = batch_gradient_descent(X, y, theta_initial, alpha, iterations)
```

**Avantages et inconvénients :**
- **Avantage** : La trajectoire de convergence est lisse et directe vers le minimum global (pour une fonction convexe).
- **Inconvénient** : Extrêmement lent et gourmand en ressources pour les très grands jeux de données. Si $m$ est de l'ordre de millions ou de milliards, chaque étape de la descente de gradient nécessite de scanner l'intégralité des données, ce qui peut être prohibitif.

### 3.4 Descente de Gradient Stochastique (Stochastic Gradient Descent - SGD)

Pour surmonter les limitations de la descente de gradient par lots, une alternative populaire est la descente de gradient stochastique (SGD). L'idée est de mettre à jour les paramètres beaucoup plus fréquemment. Au lieu d'attendre d'avoir calculé le gradient sur l'ensemble des $m$ exemples, nous mettons à jour les paramètres $\theta$ après avoir examiné un seul exemple d'entraînement à la fois.

La boucle interne de l'algorithme change radicalement : on parcourt les exemples un par un et on effectue une mise à jour pour chacun d'eux.

**Pseudo-code de l'algorithme**

```
PROCÉDURE DescenteGradientStochastique(X, y, θ, α, nombre_époques)
  m = nombre d'exemples dans X

  POUR epoque DE 1 À nombre_époques FAIRE
    // Mélanger les données à chaque époque est une bonne pratique
    indices_melanges = mélanger(indices de 0 à m-1)
    X_melange = X[indices_melanges]
    y_melange = y[indices_melanges]

    POUR i DE 0 À m-1 FAIRE
      // Prendre un seul exemple
      x_i = X_melange[i]
      y_i = y_melange[i]

      // Calculer la prédiction pour cet exemple
      prediction_i = x_i * θ

      // Calculer l'erreur pour cet exemple
      erreur_i = prediction_i - y_i

      // Calculer le gradient basé sur ce seul exemple
      gradient_i = x_i_transpose * erreur_i

      // Mettre à jour les paramètres
      θ := θ - α * gradient_i
    FIN POUR
  FIN POUR

  RETOURNER θ
FIN PROCÉDURE
```

**Implémentation en Python avec commentaires détaillés**

```python
import numpy as np

def stochastic_gradient_descent(X, y, theta, alpha, num_epochs):
    """
    Effectue une descente de gradient stochastique pour trouver theta.

    Args:
        X (np.array): Matrice des caractéristiques (m, n+1).
        y (np.array): Vecteur des cibles (m, 1).
        theta (np.array): Paramètres initiaux (n+1, 1).
        alpha (float): Taux d'apprentissage.
        num_epochs (int): Nombre de passages complets sur le jeu de données.

    Returns:
        np.array: Les paramètres theta optimisés.
        list: L'historique de la fonction de coût.
    """
    m = len(y)
    J_history = []

    for epoch in range(num_epochs):
        cost_epoch = 0
        # Mélanger les données pour s'assurer que l'ordre est aléatoire
        indices = np.random.permutation(m)
        X_shuffled = X[indices]
        y_shuffled = y[indices]

        for i in range(m):
            # Sélectionner un seul exemple d'entraînement
            xi = X_shuffled[i:i+1] # Garde la forme (1, n+1)
            yi = y_shuffled[i:i+1] # Garde la forme (1, 1)

            # Calcul de la prédiction pour cet exemple
            prediction = xi.dot(theta)
            
            # Calcul de l'erreur
            error = prediction - yi

            # Calcul du gradient pour cet exemple
            gradient = xi.T.dot(error)

            # Mise à jour des paramètres
            theta = theta - alpha * gradient
            
            # Suivi du coût
            cost_epoch += (1/(2*m)) * np.sum(np.square(error))

        J_history.append(cost_epoch)

    return theta, J_history

# Exemple d'utilisation
# theta_optimise_sgd, couts_sgd = stochastic_gradient_descent(X, y, theta_initial, alpha, epochs)
```

**Avantages et inconvénients :**
- **Avantages** : L'algorithme progresse beaucoup plus rapidement, surtout sur de très grands jeux de données, car il n'a pas besoin d'attendre de voir toutes les données pour faire une mise à jour. Le chemin "bruyant" ou "oscillant" vers le minimum peut aider à échapper aux minima locaux (un avantage crucial pour les fonctions de coût non-convexes, comme celles des réseaux de neurones).
- **Inconvénients** : La convergence n'est jamais parfaite. L'algorithme a tendance à osciller indéfiniment autour du minimum global plutôt que de s'y stabiliser. Les mises à jour fréquentes peuvent aussi être coûteuses en calcul si elles ne sont pas bien optimisées. En pratique, on diminue souvent le taux d'apprentissage $\alpha$ au fil du temps pour réduire l'amplitude de ces oscillations.

### 3.5 Note sur la descente de gradient par mini-lots (Mini-Batch Gradient Descent)

En pratique, une troisième approche, la descente de gradient par mini-lots, est la plus utilisée. C'est un compromis intelligent entre la descente par lots et la descente stochastique.

Au lieu de calculer le gradient sur 1 exemple (SGD) ou sur $m$ exemples (Batch), on le calcule sur de petits lots (des "mini-batches") de données, par exemple 32, 64 ou 128 exemples à la fois.

Cette méthode combine les avantages des deux autres :
- **Efficacité calculatoire** : Elle permet d'utiliser la vectorisation sur le mini-lot, tirant pleinement parti du matériel moderne (comme les GPU), ce qui est beaucoup plus efficace que le traitement un par un du SGD.
- **Stabilité de la convergence** : L'estimation du gradient est plus stable et moins bruitée que celle du SGD, ce qui conduit à une convergence plus régulière.
- **Rapidité de progression** : Elle reste suffisamment "stochastique" pour progresser rapidement et éviter le fardeau computationnel de la descente par lots complète.

La descente de gradient par mini-lots est le standard de facto pour l'entraînement de la plupart des modèles d'apprentissage automatique modernes, en particulier les réseaux de neurones profonds.

## Section 4: La Solution Analytique : L'Équation Normale

La descente de gradient est un algorithme itératif. Pour le cas spécifique de la régression linéaire, il existe une autre méthode pour trouver les paramètres $\theta$ optimaux : une solution analytique directe, non-itérative, appelée l'équation normale (normal equation).

### 4.1 Principe

Plutôt que de converger progressivement vers le minimum, l'équation normale nous permet de "sauter" directement à la solution optimale en une seule étape de calcul. L'idée est la même que pour trouver le minimum d'une fonction simple en calcul différentiel : on calcule sa dérivée et on cherche le point où cette dérivée est égale à zéro. Ici, comme $J(\theta)$ est une fonction de plusieurs variables (les $\theta_j$), nous devons calculer son gradient (le vecteur de toutes les dérivées partielles) et le poser égal au vecteur nul.

### 4.2 Approche de la dérivation matricielle

La clé pour dériver l'équation normale est d'exprimer la fonction de coût $J(\theta)$ en utilisant la notation matricielle. Rappelons nos définitions :
- $X$ : la matrice de conception (design matrix) de taille $m \times (n+1)$, où chaque ligne est un exemple d'entraînement $(x^{(i)})^T$ (avec $x_0^{(i)} = 1$).
- $y$ : le vecteur colonne des valeurs cibles, de taille $m \times 1$.
- $\theta$ : le vecteur colonne des paramètres, de taille $(n+1) \times 1$.

Le vecteur de toutes les prédictions est $X\theta$. Le vecteur des erreurs est $X\theta - y$. La somme des erreurs au carré, $\sum(h_\theta(x^{(i)}) - y^{(i)})^2$, peut s'écrire comme le produit scalaire du vecteur des erreurs avec lui-même, soit $(X\theta - y)^T(X\theta - y)$.

La fonction de coût devient donc :

$$J(\theta) = \frac{1}{2m} (X\theta - y)^T(X\theta - y)$$

Pour trouver le minimum, nous devons calculer le gradient de $J(\theta)$ par rapport au vecteur $\theta$ ($\nabla_\theta J(\theta)$) et le poser à zéro. La dérivation, qui utilise des règles de calcul matriciel, se déroule comme suit (en ignorant le facteur constant $1/2m$ qui n'affecte pas la position du minimum) :

1. On développe l'expression : $J(\theta) \propto (X\theta)^T(X\theta) - (X\theta)^T y - y^T(X\theta) + y^T y = \theta^T X^T X\theta - 2\theta^T X^T y + y^T y$.

2. On calcule le gradient par rapport à $\theta$ : $\nabla_\theta (\theta^T X^T X\theta - 2\theta^T X^T y + y^T y)$. En utilisant les règles des dérivées matricielles, cela se simplifie en $2X^T X\theta - 2X^T y$.

3. On pose le gradient à zéro : $2X^T X\theta - 2X^T y = 0$.

4. Cela se simplifie en $X^T X\theta = X^T y$.

Cette dernière équation est l'équation normale.

### 4.3 La formule de l'équation normale

Pour trouver $\theta$, il nous suffit de résoudre cette équation. Si la matrice $X^T X$ est inversible, nous pouvons multiplier les deux côtés par son inverse $(X^T X)^{-1}$ :

$$\theta = (X^T X)^{-1} X^T y$$

C'est la formule de l'équation normale. Elle nous donne la valeur exacte des paramètres $\theta$ qui minimisent la fonction de coût, sans aucune itération ni choix de taux d'apprentissage $\alpha$. Le terme $(X^T X)^{-1} X^T$ est connu sous le nom de pseudo-inverse de Moore-Penrose de la matrice $X$.

### 4.4 Implémentation en Python de l'équation normale

L'implémentation en Python est remarquablement concise grâce à numpy.

```python
import numpy as np

def normal_equation(X, y):
    """
    Calcule la solution de forme fermée pour la régression linéaire
    en utilisant l'équation normale.

    Args:
        X (np.array): Matrice des caractéristiques (m, n+1).
        y (np.array): Vecteur des cibles (m, 1).

    Returns:
        np.array: Les paramètres theta optimisés (n+1, 1).
    """
    try:
        # Calcul de X_transpose * X
        XTX = X.T.dot(X)
        
        # Calcul de l'inverse de (X_transpose * X)
        XTX_inv = np.linalg.inv(XTX)
        
        # Calcul de X_transpose * y
        XTy = X.T.dot(y)
        
        # Calcul de theta
        theta = XTX_inv.dot(XTy)
        
        # Une manière numériquement plus stable et souvent plus rapide est d'utiliser
        # np.linalg.solve, qui résout le système d'équations linéaires XTX * theta = XTy
        # sans calculer explicitement l'inverse.
        # theta = np.linalg.solve(XTX, XTy)
        
        return theta
        
    except np.linalg.LinAlgError:
        # La matrice n'est pas inversible
        print("Erreur : La matrice X.T * X n'est pas inversible.")
        return None

# Exemple d'utilisation
# theta_optimal_ne = normal_equation(X, y)
```

### 4.5 Considérations pratiques : Problèmes de non-inversibilité

Que se passe-t-il si la matrice $X^T X$ n'est pas inversible? C'est un problème qui peut survenir en pratique. Les deux causes principales sont :

1. **Caractéristiques redondantes** : Deux ou plusieurs caractéristiques sont linéairement dépendantes. Par exemple, si vous avez une caractéristique "surface en mètres carrés" et une autre "surface en pieds carrés". L'une est un multiple constant de l'autre.

2. **Plus de caractéristiques que d'exemples ($n > m$)** : Le système est sous-déterminé.

Si $X^T X$ n'est pas inversible, la solution est de supprimer les caractéristiques redondantes ou d'utiliser des techniques de régularisation (qui seront abordées dans des leçons ultérieures), qui ajoutent un terme à la matrice $X^T X$ pour la rendre inversible.

## Section 5: Synthèse et Comparaison des Méthodes

Nous avons vu deux approches très différentes pour optimiser les paramètres de la régression linéaire : une méthode itérative (la descente de gradient) et une méthode analytique directe (l'équation normale). Un praticien doit savoir quand utiliser l'une ou l'autre.

### 5.1 Quand utiliser la descente de gradient vs. l'équation normale?

Le choix n'est pas une question de préférence, mais est dicté par la complexité computationnelle, qui dépend des dimensions du jeu de données.

**Équation Normale** : La complexité de cette méthode est dominée par le calcul de l'inverse de la matrice $X^T X$. $X^T X$ est une matrice de taille $(n+1) \times (n+1)$. L'inversion d'une matrice de taille $n \times n$ a une complexité d'environ $O(n^3)$. Cette complexité croît très rapidement avec le nombre de caractéristiques $n$.

**Descente de Gradient** : La complexité de chaque itération est d'environ $O(m \cdot n)$, car nous devons calculer une somme sur les $m$ exemples pour chacune des $n$ caractéristiques.

La décision se résume donc à une comparaison entre $n^3$ et $k \cdot m \cdot n$ (où $k$ est le nombre d'itérations pour la descente de gradient).

- Si $n$ est petit (par exemple, $n < 10,000$), $n^3$ sera gérable. L'équation normale est alors souvent plus rapide car elle est non-itérative et n'a pas d'hyperparamètre $\alpha$ à régler.
- Si $n$ est très grand (par exemple, $n > 100,000$), le terme $n^3$ devient prohibitif. La descente de gradient (en particulier stochastique ou par mini-lots) est alors la seule option viable.

En résumé, la taille du jeu de données $m$ affecte principalement la descente de gradient par lots, tandis que le nombre de caractéristiques $n$ est le facteur limitant pour l'équation normale.

### 5.2 Tableau comparatif des approches

Le tableau suivant synthétise les caractéristiques, avantages et inconvénients des trois méthodes principales discutées.

| Caractéristique | Descente de Gradient par Lots | Descente de Gradient Stochastique | Équation Normale |
|-----------------|-------------------------------|-----------------------------------|-------------------|
| Principe | Itératif | Itératif | Analytique (direct) |
| Complexité Calcul | $O(k \cdot m \cdot n)$ ($k$ itérations) | $O(k \cdot m \cdot n)$ ($k$ époques) | $O(n^3)$ |
| Taille des données ($m$) | Inefficace pour $m$ très grand | Méthode de choix pour $m$ très grand | Indépendant de $m$ (après calcul de $X^T X$) |
| Nb de caractéristiques ($n$) | Efficace pour $n$ très grand | Efficace pour $n$ très grand | Inefficace pour $n$ très grand |
| Réglage de $\alpha$ | Nécessite le choix de $\alpha$ | Nécessite le choix de $\alpha$ (et souvent une stratégie) | Aucun hyperparamètre |
| Convergence | Convergence lisse vers le minimum global | Oscille autour du minimum, ne converge pas parfaitement | Solution optimale directe |
| Cas d'usage typique | Jeux de données de taille petite à moyenne | Grands jeux de données ("Big Data") | Problèmes avec $n$ faible ($< 10,000$) |
| Adaptabilité | S'applique à de nombreux autres modèles | S'applique à de nombreux autres modèles | Spécifique à la régression linéaire |
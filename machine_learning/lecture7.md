# Leçon sur les Machines à Vecteurs de Support (SVM)

## Introduction

Bonjour à tous. Aujourd'hui, nous allons plonger au cœur de l'un des algorithmes les plus puissants et les plus élégants de l'apprentissage automatique : les **Machines à Vecteurs de Support**, ou **SVM** (Support Vector Machines).

Les SVM sont une classe de classifieurs linéaires (et non linéaires, comme nous le verrons) qui ont été développés dans les années 1990 par **Vladimir Vapnik** et ses collaborateurs, dont **Corinna Cortes**. Leur popularité découle de leur solide base théorique, ancrée dans la **théorie de Vapnik-Chervonenkis (VC)**, et de leurs excellentes performances en pratique, notamment sur des problèmes de classification avec de nombreuses caractéristiques.

Dans cette leçon, nous allons décortiquer le fonctionnement des SVM, depuis l'intuition géométrique de base jusqu'aux astuces mathématiques qui leur confèrent leur puissance. Nous implémenterons également les concepts clés en Python.

## 1. L'Hyperplan Séparateur à Marge Optimale

L'idée fondamentale des SVM est de trouver un **hyperplan** qui sépare les données de différentes classes de la meilleure façon possible. Mais qu'est-ce qu'un "meilleur" hyperplan ?

Imaginons que nous ayons des données linéairement séparables. Il existe une infinité d'hyperplans qui peuvent séparer les deux classes. Les SVM postulent que l'hyperplan optimal est celui qui est le plus éloigné de tous les points de données. En d'autres termes, c'est l'hyperplan qui maximise la **marge**, c'est-à-dire la distance entre l'hyperplan et les points de données les plus proches de chaque classe.

### Formulation Mathématique

Soit un ensemble de données d'entraînement $(x_i, y_i)$, où $x_i \in \mathbb{R}^d$ est un vecteur de caractéristiques et $y_i \in \{-1, 1\}$ est l'étiquette de la classe. Un hyperplan peut être défini par l'équation :

$$w \cdot x + b = 0$$

où $w$ est le vecteur normal à l'hyperplan et $b$ est le biais.

Nous voulons trouver $w$ et $b$ tels que :

$$y_i(w \cdot x_i + b) \geq 1 \text{ pour tous les } i$$

La distance entre l'hyperplan et un point $x_i$ est donnée par $\frac{|w \cdot x_i + b|}{||w||}$. La marge est donc $\frac{1}{||w||}$. Maximiser la marge équivaut à minimiser $||w||$, ou, pour des raisons de commodité mathématique, minimiser $\frac{1}{2}||w||^2$.

Le problème d'optimisation, connu sous le nom de **problème primal**, est donc :

$$\min_{w, b} \frac{1}{2} ||w||^2$$

sujet à $y_i(w \cdot x_i + b) \geq 1$ pour tout $i$.

Il s'agit d'un problème d'optimisation quadratique convexe, qui peut être résolu à l'aide de multiplicateurs de Lagrange. En introduisant les multiplicateurs de Lagrange $\alpha_i \geq 0$, nous obtenons le **problème dual** :

$$\max_{\alpha} \sum_{i=1}^n \alpha_i - \frac{1}{2} \sum_{i=1}^n \sum_{j=1}^n \alpha_i \alpha_j y_i y_j (x_i \cdot x_j)$$

sujet à $\sum_{i=1}^n \alpha_i y_i = 0$ et $\alpha_i \geq 0$ pour tout $i$.

Les points pour lesquels $\alpha_i > 0$ sont appelés les **vecteurs de support**. Ce sont les points qui se trouvent sur la marge et qui définissent l'hyperplan.

### Pseudo-code

```
Algorithme : Entraînement d'un SVM linéaire (problème dual)

Entrée : Données d'entraînement (x_i, y_i)

1. Construire la matrice de Gram G où G_ij = y_i * y_j * (x_i . x_j)
2. Résoudre le problème d'optimisation quadratique :
   max_alpha sum(alpha_i) - 0.5 * alpha^T * G * alpha
   sujet à sum(alpha_i * y_i) = 0 et alpha_i >= 0
3. Identifier les vecteurs de support (les x_i pour lesquels alpha_i > 0)
4. Calculer le vecteur de poids w = sum(alpha_i * y_i * x_i)
5. Calculer le biais b = y_k - w . x_k pour un vecteur de support x_k

Sortie : Le modèle SVM (w, b)
```

## 2. Le "Kernel Trick" (L'Astuce du Noyau)

Que se passe-t-il si les données ne sont pas linéairement séparables ? C'est là que les SVM deviennent vraiment intéressants. L'idée est de projeter les données dans un espace de plus grande dimension où elles deviennent linéairement séparables.

Cependant, calculer explicitement cette projection peut être très coûteux en calculs. C'est ici qu'intervient l'**astuce du noyau** (kernel trick).

Remarquez que dans le problème dual, les données n'apparaissent que sous la forme de produits scalaires $(x_i \cdot x_j)$. L'astuce du noyau consiste à remplacer ce produit scalaire par une **fonction noyau** $K(x_i, x_j) = \phi(x_i) \cdot \phi(x_j)$, où $\phi$ est la fonction de projection.

Cela nous permet de travailler dans l'espace de grande dimension sans jamais avoir à calculer explicitement les coordonnées des points dans cet espace.

### Fonctions Noyau Courantes

- **Noyau linéaire** : $K(x_i, x_j) = x_i \cdot x_j$ (le cas que nous avons vu jusqu'à présent)
- **Noyau polynomial** : $K(x_i, x_j) = (x_i \cdot x_j + c)^d$
- **Noyau Gaussien (RBF)** : $K(x_i, x_j) = \exp(-\gamma ||x_i - x_j||^2)$

Le choix du noyau et de ses hyperparamètres est crucial pour les performances du SVM.

### Théorème de Mercer

Toutes les fonctions ne peuvent pas être des noyaux. Une fonction $K(x_i, x_j)$ doit satisfaire le **théorème de Mercer**, qui stipule que la matrice du noyau (la matrice de Gram) doit être semi-définie positive.

### Implémentation en Python

```python
import numpy as np

def linear_kernel(x1, x2):
    return np.dot(x1, x2)

def polynomial_kernel(x1, x2, d=3, c=1):
    return (np.dot(x1, x2) + c) ** d

def gaussian_kernel(x1, x2, gamma=0.1):
    return np.exp(-gamma * np.linalg.norm(x1 - x2)**2)
```

## 3. SVM à Marge Souple (Soft-Margin SVM)

Dans la pratique, même avec des noyaux, les données peuvent ne pas être parfaitement séparables. De plus, un classifieur qui sépare parfaitement les données d'entraînement peut être sujet au surapprentissage.

Pour résoudre ce problème, on introduit les **variables ressorts** (slack variables) $\xi_i \geq 0$. Ces variables permettent à certains points de violer la marge. La contrainte devient :

$$y_i(w \cdot x_i + b) \geq 1 - \xi_i$$

Le problème d'optimisation est alors modifié pour pénaliser ces violations :

$$\min_{w, b, \xi} \frac{1}{2} ||w||^2 + C \sum_{i=1}^n \xi_i$$

sujet à $y_i(w \cdot x_i + b) \geq 1 - \xi_i$ et $\xi_i \geq 0$ pour tout $i$.

L'**hyperparamètre C** contrôle le compromis entre la maximisation de la marge et la minimisation de l'erreur de classification.

- Un **petit C** crée une grande marge mais permet plus de violations.
- Un **grand C** crée une petite marge et pénalise fortement les violations.

Le problème dual devient :

$$\max_{\alpha} \sum_{i=1}^n \alpha_i - \frac{1}{2} \sum_{i=1}^n \sum_{j=1}^n \alpha_i \alpha_j y_i y_j K(x_i, x_j)$$

sujet à $\sum_{i=1}^n \alpha_i y_i = 0$ et $0 \leq \alpha_i \leq C$ pour tout $i$.

La seule différence est que les multiplicateurs de Lagrange sont maintenant bornés par C.

### Implémentation en Python (partielle)

L'implémentation complète nécessite un solveur de programmation quadratique. `cvxopt` est une bibliothèque Python populaire pour cela.

```python
import numpy as np
from cvxopt import matrix, solvers

class SVM:
    def __init__(self, kernel='linear', C=1.0, gamma=0.1, d=3, c=1):
        self.kernel_name = kernel
        if self.kernel_name == 'linear':
            self.kernel = linear_kernel
        elif self.kernel_name == 'poly':
            self.kernel = lambda x1, x2: polynomial_kernel(x1, x2, d, c)
        elif self.kernel_name == 'rbf':
            self.kernel = lambda x1, x2: gaussian_kernel(x1, x2, gamma)
        self.C = C
        self.alpha = None
        self.sv_x = None
        self.sv_y = None
        self.b = None

    def fit(self, X, y):
        n_samples, n_features = X.shape

        # Matrice de Gram
        K = np.zeros((n_samples, n_samples))
        for i in range(n_samples):
            for j in range(n_samples):
                K[i,j] = self.kernel(X[i], X[j])

        P = matrix(np.outer(y,y) * K)
        q = matrix(np.ones(n_samples) * -1)
        A = matrix(y, (1,n_samples), 'd')
        b = matrix(0.0)

        if self.C is None:
            G = matrix(np.diag(np.ones(n_samples) * -1))
            h = matrix(np.zeros(n_samples))
        else:
            G = matrix(np.vstack((np.diag(np.ones(n_samples) * -1), 
                                 np.identity(n_samples))))
            h = matrix(np.hstack((np.zeros(n_samples), 
                                 np.ones(n_samples) * self.C)))

        # Résoudre le problème d'optimisation quadratique
        solution = solvers.qp(P, q, G, h, A, b)
        alpha = np.ravel(solution['x'])

        # Vecteurs de support
        sv = alpha > 1e-5
        self.alpha = alpha[sv]
        self.sv_x = X[sv]
        self.sv_y = y[sv]

        # Calcul du biais
        self.b = 0
        for i in range(len(self.alpha)):
            self.b += self.sv_y[i]
            self.b -= np.sum(self.alpha * self.sv_y * 
                           [self.kernel(self.sv_x[j], self.sv_x[i]) 
                            for j in range(len(self.sv_x))])
        self.b /= len(self.alpha)

    def predict(self, X):
        y_pred = np.zeros(len(X))
        for i in range(len(X)):
            s = 0
            for alpha, sv_y, sv_x in zip(self.alpha, self.sv_y, self.sv_x):
                s += alpha * sv_y * self.kernel(X[i], sv_x)
            y_pred[i] = s
        return np.sign(y_pred + self.b)
```

## 4. Applications des SVM

Les SVM sont utilisés dans de nombreux domaines :

- **Reconnaissance de caractères manuscrits** : Les SVM ont été l'un des premiers algorithmes à atteindre des performances quasi humaines sur la base de données MNIST.
- **Classification de texte** : Les SVM sont très efficaces pour la classification de documents, par exemple pour le filtrage de spams.
- **Bio-informatique** : Les SVM sont utilisés pour la classification de séquences de protéines, la prédiction de la structure des protéines, etc. Des noyaux spécifiques, comme les **noyaux de chaîne** (string kernels), ont été développés pour ces tâches.
- **Détection d'objets dans les images** : Combinés avec des descripteurs de caractéristiques comme HOG (Histogram of Oriented Gradients).
- **Reconnaissance de la parole** : Pour la classification de phonèmes.
- **Finance** : Pour la prédiction de tendances boursières et l'évaluation du risque de crédit.

## Conclusion

Les Machines à Vecteurs de Support sont un outil puissant et polyvalent de l'apprentissage automatique. Leur élégance mathématique, combinée à leur efficacité pratique, en a fait un pilier du domaine.

Nous avons vu comment ils fonctionnent, en partant de l'idée simple de maximiser la marge, en passant par l'astuce du noyau pour gérer les données non linéaires, jusqu'à l'introduction de la marge souple pour les données bruitées.

Bien que de nouvelles méthodes comme les réseaux de neurones profonds aient gagné en popularité ces dernières années, les SVM restent un outil essentiel à maîtriser pour tout praticien de l'apprentissage automatique, particulièrement pour les problèmes avec des données de dimension moyenne et des ensembles d'entraînement de taille raisonnable.

## Références

- **Cortes, C., & Vapnik, V. (1995)**. Support-vector networks. *Machine learning*, 20(3), 273-297. (Le papier fondamental introduisant les SVM)
- **Vapnik, V. (1998)**. *Statistical learning theory*. Wiley. (Livre de référence sur la théorie de l'apprentissage statistique)
- **Kimeldorf, G., & Wahba, G. (1971)**. Some results on Tchebycheffian spline functions. *Journal of Mathematical Analysis and Applications*, 33(1), 82-95. (Travaux précurseurs sur le théorème du représentant)
- **Aizerman, M. A., Braverman, E. M., & Rozonoer, L. I. (1964)**. Theoretical foundations of the potential function method in pattern recognition learning. *Automation and remote control*, 25(6), 821-837. (Travaux précurseurs sur l'astuce du noyau)
- **Schölkopf, B., & Smola, A. J. (2002)**. *Learning with kernels: support vector machines, regularization, optimization, and beyond*. MIT Press. (Livre de référence moderne sur les méthodes à noyaux)
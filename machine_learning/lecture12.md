# Cours Approfondi sur l'Entraînement des Réseaux de Neurones Profonds : De la Théorie à la Pratique

## Introduction

### Objectif du Cours

Ce cours a pour ambition de décomposer intégralement le processus d'entraînement des réseaux de neurones profonds. Nous partirons des fondements théoriques de l'apprentissage par gradient pour aboutir aux algorithmes et heuristiques pratiques qui permettent d'atteindre des performances de pointe. Ce document est conçu comme une ressource autonome, transformant la discussion informelle d'une leçon enregistrée en un texte structuré et académique, enrichi de références scientifiques et d'implémentations pratiques. L'objectif est de fournir une compréhension profonde, non seulement du "comment" les réseaux de neurones apprennent, mais aussi du "pourquoi" certaines techniques sont essentielles à leur succès.

### Structure de l'Apprentissage

Le cours est divisé en trois chapitres fondamentaux qui suivent la logique de la construction et de l'optimisation d'un modèle d'apprentissage profond.

1. **La Rétropropagation : Le Moteur de l'Apprentissage.** Ce chapitre dissèque l'algorithme de rétropropagation, le mécanisme fondamental qui permet de calculer les gradients nécessaires à l'ajustement des paramètres du réseau.

2. **Améliorer la Dynamique d'Apprentissage : Activations et Initialisation.** Ici, nous explorons les techniques cruciales qui assurent un entraînement stable et efficace. Nous analyserons le rôle des fonctions d'activation non linéaires et l'importance capitale d'une initialisation judicieuse des poids.

3. **Algorithmes d'Optimisation : Naviguer Efficacement l'Espace des Paramètres.** Enfin, nous étudierons une famille d'algorithmes d'optimisation sophistiqués, allant de la descente de gradient avec momentum à l'algorithme Adam, qui accélèrent la convergence vers une solution optimale.

Chaque chapitre combinera des dérivations mathématiques rigoureuses, des intuitions conceptuelles, des références à des articles de recherche fondateurs, ainsi que des implémentations en Python pour ancrer la théorie dans la pratique.

## Chapitre 1 : La Rétropropagation : Le Moteur de l'Apprentissage

L'apprentissage dans un réseau de neurones est un problème d'optimisation : il s'agit de trouver l'ensemble de paramètres (poids et biais) qui minimise une mesure d'erreur entre les prédictions du modèle et les véritables valeurs cibles. L'algorithme qui rend cette optimisation possible dans des architectures profondes et complexes est la rétropropagation.

### 1.1 Formalisation du Problème d'Optimisation

Avant de plonger dans la mécanique de la rétropropagation, il est essentiel de définir précisément ce que nous cherchons à minimiser.

#### De la Perte au Coût

Pour chaque exemple d'entraînement individuel, nous mesurons la performance du modèle à l'aide d'une fonction de perte (Loss Function), notée $L(\hat{y}, y)$. Cette fonction quantifie l'écart entre la sortie prédite par le réseau, $\hat{y}$, et la sortie réelle (ou étiquette), $y$. Pour une tâche de classification binaire, comme la détection de chats mentionnée dans la leçon de référence, une fonction de perte couramment utilisée est l'entropie croisée binaire (Binary Cross-Entropy), également appelée perte logistique. Sa formule est la suivante :

$$L(\hat{y}^{(i)}, y^{(i)}) = -[y^{(i)}\log(\hat{y}^{(i)}) + (1-y^{(i)})\log(1-\hat{y}^{(i)})]$$

où l'indice $(i)$ désigne le $i$-ème exemple de l'ensemble de données. Cette fonction pénalise fortement les prédictions qui sont à la fois erronées et faites avec une grande confiance.

Cependant, l'entraînement ne se fait que rarement sur un seul exemple à la fois. Pour obtenir une mesure de performance plus stable et exploiter l'efficacité du calcul moderne, nous évaluons l'erreur sur un ensemble d'exemples, appelé un lot ou mini-batch. La fonction de coût (Cost Function), notée $J(W,b)$, est définie comme la moyenne des fonctions de perte sur tous les $m$ exemples du mini-batch :

$$J(W,b) = \frac{1}{m}\sum_{i=1}^{m} L(\hat{y}^{(i)}, y^{(i)})$$

où $W$ et $b$ représentent l'ensemble des matrices de poids et des vecteurs de biais de toutes les couches du réseau. C'est cette fonction de coût $J$ que nous cherchons à minimiser.

#### L'Importance de la Vectorisation

Le traitement des données par lots (mini-batches) n'est pas seulement une technique pour obtenir une estimation plus robuste du gradient. C'est une nécessité pratique pour l'efficacité computationnelle. Les processeurs graphiques (GPU), qui sont le cheval de bataille de l'apprentissage profond, sont conçus pour effectuer des opérations parallèles à grande échelle. En regroupant $m$ exemples en matrices (par exemple, une matrice d'entrée $X$ de dimension $(n_x, m)$ où $n_x$ est le nombre de caractéristiques), nous pouvons remplacer les boucles for sur les exemples par des opérations matricielles hautement optimisées. Ce processus, appelé vectorisation, accélère l'entraînement de plusieurs ordres de grandeur par rapport à un traitement séquentiel exemple par exemple.

### 1.2 La Règle de la Chaîne : Le Calcul au Cœur du Réseau

Il est crucial de comprendre que la rétropropagation n'est pas, en soi, un algorithme d'apprentissage. C'est un algorithme d'efficacité. Son unique objectif est de calculer le gradient de la fonction de coût $J$ par rapport à chaque poids $w_{ij}^{[l]}$ et chaque biais $b_i^{[l]}$ du réseau. L'apprentissage est réalisé par un algorithme d'optimisation, comme la descente de gradient, qui utilise ce gradient pour mettre à jour les paramètres : $W \leftarrow W - \alpha \frac{\partial J}{\partial W}$. La rétropropagation est simplement une application intelligente de la règle de dérivation en chaîne (chain rule) du calcul différentiel pour calculer ce gradient efficacement.

Dans un réseau de neurones, la fonction de coût est une composition très profonde de fonctions. La sortie de la première couche est l'entrée de la deuxième, dont la sortie est l'entrée de la troisième, et ainsi de suite, jusqu'à la fonction de coût finale. Si nous notons $z^{[l]}$ la combinaison linéaire et $a^{[l]}$ l'activation de la couche $l$, la dépendance peut être schématisée comme suit :

$$X \rightarrow z^{[1]} \rightarrow a^{[1]} \rightarrow z^{[2]} \rightarrow a^{[2]} \rightarrow \cdots \rightarrow z^{[L]} \rightarrow a^{[L]} \rightarrow J$$

Pour trouver l'influence d'un poids dans une couche précoce, disons $W^{[l]}$, sur le coût final $J$, nous devons "chaîner" les influences locales à travers toutes les étapes intermédiaires. Mathématiquement, cela s'exprime par :

$$\frac{\partial J}{\partial W^{[l]}} = \frac{\partial J}{\partial a^{[L]}} \frac{\partial a^{[L]}}{\partial z^{[L]}} \frac{\partial z^{[L]}}{\partial a^{[L-1]}} \cdots \frac{\partial a^{[l+1]}}{\partial z^{[l+1]}} \frac{\partial z^{[l+1]}}{\partial a^{[l]}} \frac{\partial a^{[l]}}{\partial z^{[l]}} \frac{\partial z^{[l]}}{\partial W^{[l]}}$$

La propagation avant (forward pass) consiste à calculer et à stocker les valeurs de tous les $z^{[l]}$ et $a^{[l]}$ pour un lot d'entrées donné. La rétropropagation (backward pass) consiste ensuite à calculer les dérivées locales (par exemple, $\frac{\partial a^{[l]}}{\partial z^{[l]}}$) et à les multiplier en remontant la chaîne de dépendance, de la sortie vers l'entrée.

### 1.3 Dérivation Pas-à-Pas : Gradient de la Dernière Couche ($\frac{\partial J}{\partial W^{[L]}}$)

Nous commençons la dérivation par la dernière couche, notée $L$, car sa relation avec la fonction de coût est la plus directe, impliquant moins de termes dans la règle de la chaîne. L'objectif est de calculer $\frac{\partial J}{\partial W^{[L]}}$ et $\frac{\partial J}{\partial b^{[L]}}$.

#### Décomposition du Gradient

En utilisant la règle de la chaîne, nous pouvons décomposer la dérivée du coût par rapport aux poids de la dernière couche, $W^{[L]}$, comme suit :

$$\frac{\partial J}{\partial W^{[L]}} = \frac{\partial J}{\partial a^{[L]}} \frac{\partial a^{[L]}}{\partial z^{[L]}} \frac{\partial z^{[L]}}{\partial W^{[L]}}$$

Analysons chaque terme pour un seul exemple (nous généraliserons à un lot plus tard) :

1. **$\frac{\partial L}{\partial a^{[L]}}$ : Dérivée de la perte par rapport à l'activation de sortie.**
   
   Pour la perte d'entropie croisée binaire, $L = -[y\log(a^{[L]}) + (1-y)\log(1-a^{[L]})]$, la dérivée est :
   
   $$\frac{\partial L}{\partial a^{[L]}} = -\left(\frac{y}{a^{[L]}} - \frac{1-y}{1-a^{[L]}}\right) = \frac{a^{[L]} - y}{a^{[L]}(1-a^{[L]})}$$

2. **$\frac{\partial a^{[L]}}{\partial z^{[L]}}$ : Dérivée de la fonction d'activation de sortie.**
   
   Supposons que nous utilisons une fonction sigmoïde pour la classification binaire, $a^{[L]} = \sigma(z^{[L]})$. Sa dérivée est $\sigma'(z^{[L]}) = \sigma(z^{[L]})(1-\sigma(z^{[L]})) = a^{[L]}(1-a^{[L]})$.

3. **$\frac{\partial z^{[L]}}{\partial W^{[L]}}$ : Dérivée de la combinaison linéaire par rapport aux poids.**
   
   La combinaison linéaire est $z^{[L]} = W^{[L]}a^{[L-1]} + b^{[L]}$. La dérivée de cette expression par rapport à la matrice $W^{[L]}$ est simplement le vecteur d'activation de la couche précédente, transposé : $(a^{[L-1]})^T$.

#### Combinaison et Simplification

En multipliant ces trois termes, une simplification remarquable se produit :

$$\frac{\partial L}{\partial W^{[L]}} = \left(\frac{a^{[L]} - y}{a^{[L]}(1-a^{[L]})}\right) \cdot (a^{[L]}(1-a^{[L]})) \cdot (a^{[L-1]})^T = (a^{[L]} - y)(a^{[L-1]})^T$$

Ce résultat est élégant et intuitif. L'ajustement du poids est proportionnel à l'erreur de prédiction $(a^{[L]} - y)$ et à l'activation de la couche précédente $(a^{[L-1]})$.

Pour le biais $b^{[L]}$, la dérivation est similaire, mais comme $\frac{\partial z^{[L]}}{\partial b^{[L]}} = 1$, le résultat est encore plus simple :

$$\frac{\partial L}{\partial b^{[L]}} = (a^{[L]} - y)$$

#### Généralisation au Mini-Batch et Analyse des Formes

En généralisant sur un mini-batch de $m$ exemples, nous moyennons les gradients. Les termes deviennent des matrices : $A^{[L]}$, $Y$, $A^{[L-1]}$, etc.

$$\frac{\partial J}{\partial W^{[L]}} = \frac{1}{m}(A^{[L]} - Y)(A^{[L-1]})^T$$

$$\frac{\partial J}{\partial b^{[L]}} = \frac{1}{m}\sum_{i=1}^{m}(a^{[L](i)} - y^{(i)})$$

Une compétence pratique essentielle, soulignée dans la leçon, est l'analyse des formes (shapes) des matrices. Par exemple, si $J$ est un scalaire et $W^{[L]}$ est une matrice de forme $(n^{[L]}, n^{[L-1]})$, alors $\frac{\partial J}{\partial W^{[L]}}$ doit avoir la même forme. En vérifiant les dimensions des produits matriciels (par exemple, $(A^{[L]} - Y)$ a la forme $(n^{[L]}, m)$ et $(A^{[L-1]})^T$ a la forme $(m, n^{[L-1]})$), on confirme que le produit a bien la forme $(n^{[L]}, n^{[L-1]})$, validant ainsi notre dérivation. Cette analyse est un outil puissant pour déboguer les implémentations de rétropropagation.

### 1.4 Propagation de l'Erreur : Gradient d'une Couche Interne ($\frac{\partial J}{\partial W^{[l]}}$)

Calculer le gradient pour une couche cachée $l$ est plus complexe car ses poids influencent le coût final à travers toutes les couches suivantes $(l+1, l+2, \ldots, L)$. La clé est de ne pas tout recalculer, mais de propager l'erreur "vers l'arrière" de manière récursive.

#### Le Principe de Récursivité et le Terme d'Erreur $\delta$

Nous voulons calculer $\frac{\partial J}{\partial W^{[l]}}$. La décomposition est :

$$\frac{\partial J}{\partial W^{[l]}} = \frac{\partial J}{\partial z^{[l]}} \frac{\partial z^{[l]}}{\partial W^{[l]}}$$

Le terme $\frac{\partial z^{[l]}}{\partial W^{[l]}}$ est facile à calculer, c'est $(A^{[l-1]})^T$. Le terme difficile est $\frac{\partial J}{\partial z^{[l]}}$, que nous appellerons le terme d'erreur de la couche $l$, noté $\delta^{[l]}$.

Pour trouver $\delta^{[l]}$, nous utilisons à nouveau la règle de la chaîne, en notant que $z^{[l]}$ influence $J$ via $a^{[l]}$, qui influence $z^{[l+1]}$, qui influence $J$.

$$\delta^{[l]} = \frac{\partial J}{\partial z^{[l]}} = \frac{\partial J}{\partial z^{[l+1]}} \frac{\partial z^{[l+1]}}{\partial a^{[l]}} \frac{\partial a^{[l]}}{\partial z^{[l]}}$$

Analysons les termes de droite :

- $\frac{\partial J}{\partial z^{[l+1]}}$ est simplement $\delta^{[l+1]}$, le terme d'erreur de la couche suivante.
- $\frac{\partial z^{[l+1]}}{\partial a^{[l]}}$ est la dérivée de $z^{[l+1]} = W^{[l+1]}a^{[l]} + b^{[l+1]}$ par rapport à $a^{[l]}$, ce qui donne $(W^{[l+1]})^T$.
- $\frac{\partial a^{[l]}}{\partial z^{[l]}}$ est la dérivée de la fonction d'activation de la couche $l$, notée $g^{[l]'}(z^{[l]})$.

En combinant ces termes, nous obtenons la relation récursive fondamentale de la rétropropagation :

$$\delta^{[l]} = ((W^{[l+1]})^T\delta^{[l+1]}) \odot g^{[l]'}(z^{[l]})$$

où $\odot$ est le produit d'Hadamard (multiplication élément par élément). Cette équation montre comment l'erreur de la couche $l+1$ est propagée vers l'arrière à la couche $l$, pondérée par les poids de connexion et modulée par la dérivée de l'activation locale.

Une fois $\delta^{[l]}$ calculé, les gradients pour la couche $l$ sont directs :

$$\frac{\partial J}{\partial W^{[l]}} = \frac{1}{m}\delta^{[l]}(A^{[l-1]})^T$$

$$\frac{\partial J}{\partial b^{[l]}} = \frac{1}{m}\text{np.sum}(\delta^{[l]}, \text{axis}=1, \text{keepdims}=\text{True})$$

#### L'Importance du Cache

Ce calcul récursif de $\delta^{[l]}$ dépend de $z^{[l]}$, qui a été calculé lors de la propagation avant. De même, le calcul de $\frac{\partial J}{\partial W^{[l]}}$ nécessite $A^{[l-1]}$. Il est donc impératif, pour des raisons d'efficacité, de stocker (mettre en cache) toutes les valeurs intermédiaires $(Z, A, Z, A, \ldots)$ pendant la passe avant pour les réutiliser pendant la passe arrière. Omettre cette mise en cache obligerait à refaire des calculs coûteux, rendant l'entraînement excessivement lent.

### 1.5 L'Algorithme de Rétropropagation Complet

Nous pouvons maintenant synthétiser l'ensemble du processus dans un algorithme complet.

#### Pseudocode

Pour un mini-batch d'entraînement $(X, Y)$ :

**Propagation Avant (Forward Pass)**
1. $A^{[0]} = X$
2. Pour $l = 1, \ldots, L$ :
   - $Z^{[l]} = W^{[l]}A^{[l-1]} + b^{[l]}$
   - $A^{[l]} = g^{[l]}(Z^{[l]})$
   - (Mettre en cache toutes les valeurs de $Z^{[l]}$ et $A^{[l]}$)

**Calcul du Coût**
3. $J = \frac{1}{m}\sum L(\hat{y}, y)$

**Rétropropagation (Backward Pass)**
4. Calculer l'erreur de la couche de sortie :
   $\delta^{[L]} = \frac{\partial J}{\partial A^{[L]}} \odot g^{[L]'}(Z^{[L]})$
   (Pour l'entropie croisée et une sigmoïde en sortie, cela se simplifie en $\delta^{[L]} = A^{[L]} - Y$)

5. Calculer les gradients pour la couche de sortie :
   - $dW^{[L]} = \frac{1}{m}\delta^{[L]}(A^{[L-1]})^T$
   - $db^{[L]} = \frac{1}{m}\text{np.sum}(\delta^{[L]}, \text{axis}=1, \text{keepdims}=\text{True})$

6. Pour $l = L-1, \ldots, 1$ (boucle vers l'arrière) :
   a. Rétropropager l'erreur : $\delta^{[l]} = ((W^{[l+1]})^T\delta^{[l+1]}) \odot g^{[l]'}(Z^{[l]})$
   b. Calculer les gradients :
      - $dW^{[l]} = \frac{1}{m}\delta^{[l]}(A^{[l-1]})^T$
      - $db^{[l]} = \frac{1}{m}\text{np.sum}(\delta^{[l]}, \text{axis}=1, \text{keepdims}=\text{True})$

**Mise à Jour des Paramètres (une étape de descente de gradient)**
7. Pour $l = 1, \ldots, L$ :
   - $W^{[l]} = W^{[l]} - \alpha \cdot dW^{[l]}$
   - $b^{[l]} = b^{[l]} - \alpha \cdot db^{[l]}$

#### Implémentation en Python/NumPy

Voici un squelette d'implémentation pour une classe MultiLayerPerceptron.

```python
import numpy as np

class MultiLayerPerceptron:
    # ... (Initialisation des paramètres, etc.) ...

    def forward_propagation(self, X):
        """
        Effectue la propagation avant et met en cache les valeurs intermédiaires.
        """
        cache = {}
        A = X
        cache['A0'] = X
        
        for l in range(1, self.num_layers + 1):
            W = self.params['W' + str(l)]
            b = self.params['b' + str(l)]
            Z = np.dot(W, A) + b
            # Utiliser la fonction d'activation appropriée (ex: relu, sigmoid)
            A = self.activation_function(Z, self.layer_activations[l-1])
            
            cache['Z' + str(l)] = Z
            cache['A' + str(l)] = A
        
        return A, cache

    def backward_propagation(self, Y, cache):
        """
        Effectue la rétropropagation pour calculer les gradients.
        """
        grads = {}
        m = Y.shape[1]
        AL = cache['A' + str(self.num_layers)]
        
        # Pour la dernière couche (simplification pour entropie croisée + sigmoïde)
        dZ_L = AL - Y
        grads['dW' + str(self.num_layers)] = (1/m) * np.dot(dZ_L, cache['A' + str(self.num_layers - 1)].T)
        grads['db' + str(self.num_layers)] = (1/m) * np.sum(dZ_L, axis=1, keepdims=True)
        
        # Boucle vers l'arrière pour les autres couches
        dZ_prev = dZ_L
        for l in reversed(range(1, self.num_layers)):
            W_next = self.params['W' + str(l + 1)]
            dA_l = np.dot(W_next.T, dZ_prev)
            
            # Calculer la dérivée de l'activation
            Z_l = cache['Z' + str(l)]
            activation_deriv = self.activation_derivative(Z_l, self.layer_activations[l-1])
            
            dZ_l = dA_l * activation_deriv
            
            A_prev = cache['A' + str(l - 1)]
            grads['dW' + str(l)] = (1/m) * np.dot(dZ_l, A_prev.T)
            grads['db' + str(l)] = (1/m) * np.sum(dZ_l, axis=1, keepdims=True)
            
            dZ_prev = dZ_l
            
        return grads

    def update_parameters(self, grads, learning_rate):
        """
        Met à jour les paramètres en utilisant la descente de gradient.
        """
        for l in range(1, self.num_layers + 1):
            self.params['W' + str(l)] -= learning_rate * grads['dW' + str(l)]
            self.params['b' + str(l)] -= learning_rate * grads['db' + str(l)]
```

### 1.6 Fondements Académiques

Bien que l'idée de la dérivation en chaîne soit un principe mathématique ancien, sa systématisation et son application efficace pour l'entraînement des réseaux de neurones profonds ont été un tournant majeur. L'article fondateur est celui de David Rumelhart, Geoffrey Hinton, et Ronald Williams (1986), "Learning representations by back-propagating errors". Avant cette publication, l'entraînement de réseaux à plus d'une couche cachée était considéré comme un problème largement non résolu. Cet article a démontré qu'il était possible d'apprendre des caractéristiques complexes et utiles dans les couches cachées en propageant un signal d'erreur de la sortie vers l'entrée. Ce travail a été le catalyseur de la "renaissance" des réseaux de neurones dans les années 1980 et a jeté les bases de la révolution de l'apprentissage profond qui a suivi.

La rétropropagation incarne une dualité fascinante. D'un côté, c'est une application purement mécanique de la règle de dérivation en chaîne, un exercice de calcul différentiel qui peut être entièrement automatisé. De l'autre, son application a débloqué la capacité des réseaux à apprendre des "représentations internes" ou des "caractéristiques" hiérarchiques dans les couches cachées, ce qui est la véritable source de la puissance de l'apprentissage profond. L'algorithme est mécanique, mais son résultat est sémantique. Le gradient n'est pas juste un nombre ; c'est un signal d'information qui sculpte la connaissance interne du réseau, permettant l'émergence d'une structure de plus en plus abstraite à travers les couches.

## Chapitre 2 : Améliorer la Dynamique d'Apprentissage : Activations et Initialisation

Avoir un algorithme pour calculer les gradients est nécessaire, mais pas suffisant. Pour que l'entraînement des réseaux profonds réussisse en pratique, il faut s'assurer que le flux d'informations, à la fois vers l'avant (activations) et vers l'arrière (gradients), se propage de manière stable à travers le réseau. Ce chapitre se concentre sur deux leviers essentiels pour contrôler cette dynamique : le choix des fonctions d'activation et les stratégies d'initialisation des poids.

### 2.1 Fonctions d'Activation : Le Souffle de Non-Linéarité

#### La Nécessité de la Non-Linéarité

Comme démontré rigoureusement dans la leçon, un réseau de neurones composé uniquement d'opérations linéaires (c'est-à-dire sans fonctions d'activation) est mathématiquement équivalent à une seule couche linéaire. La composition de plusieurs transformations linéaires est elle-même une transformation linéaire. Par exemple, pour un réseau à deux couches sans activation :

$$y = W^{[2]}(W^{[1]}x + b^{[1]}) + b^{[2]} = (W^{[2]}W^{[1]})x + (W^{[2]}b^{[1]} + b^{[2]})$$

On peut définir $W' = W^{[2]}W^{[1]}$ et $b' = W^{[2]}b^{[1]} + b^{[2]}$, et le réseau profond se réduit à $y = W'x + b'$. Un tel modèle ne peut apprendre que des relations linéaires entre les entrées et les sorties, ce qui est extrêmement limitant. Les fonctions d'activation non linéaires sont ce qui confère au réseau sa capacité à approximer n'importe quelle fonction continue (théorème de l'approximation universelle) et à modéliser les relations complexes présentes dans les données du monde réel.

#### Analyse Comparative des Fonctions d'Activation

##### Sigmoïde et Tangente Hyperbolique (Tanh)

Historiquement, les fonctions sigmoïdes et tangentes hyperboliques étaient les choix prédominants, inspirés par la biologie et la théorie des probabilités.

- **Sigmoïde** : $\sigma(z) = \frac{1}{1 + e^{-z}}$. Elle comprime toute entrée réelle dans l'intervalle $(0,1)$, ce qui est utile pour interpréter la sortie comme une probabilité dans la classification binaire.

- **Tanh** : $\tanh(z) = \frac{e^z - e^{-z}}{e^z + e^{-z}}$. Elle comprime les entrées dans l'intervalle $(-1,1)$. Son avantage par rapport à la sigmoïde est que sa sortie est centrée autour de zéro, ce qui peut aider à accélérer la convergence.

Cependant, ces deux fonctions partagent un défaut majeur : le problème de **saturation**. Comme l'explique la leçon et comme l'ont analysé Glorot & Bengio, lorsque la valeur absolue de l'entrée $z$ devient grande, la sortie de ces fonctions s'aplatit et la pente de la courbe (la dérivée) tend vers zéro.

Cette saturation a une conséquence désastreuse pour l'entraînement : le **gradient évanescent** (Vanishing Gradient). Lors de la rétropropagation, le gradient local de la couche $l$ est multiplié par la dérivée de l'activation, $g'(z^{[l]})$. Si de nombreux neurones sont dans un régime de saturation, leurs dérivées locales seront très proches de zéro. Dans un réseau profond, le gradient global est le produit de nombreuses de ces dérivées. Le produit de nombreux nombres significativement inférieurs à 1 tend exponentiellement vers zéro. Le signal d'erreur "s'évanouit" avant d'atteindre les premières couches du réseau, qui par conséquent, n'apprennent presque rien.

##### L'Unité de Rectification Linéaire (ReLU)

Pour surmonter les limitations des fonctions saturantes, l'Unité de Rectification Linéaire (ReLU) a été introduite et est rapidement devenue la norme dans la plupart des applications de l'apprentissage profond.

**Formule** : $\text{ReLU}(z) = \max(0, z)$

Les avantages de ReLU sont multiples et expliquent sa popularité :

1. **Solution au Gradient Évanescent** : Pour toute entrée positive ($z > 0$), la dérivée de ReLU est constante et égale à 1. Cela signifie que le gradient peut traverser le neurone sans être atténué. Cette propriété a considérablement facilité l'entraînement de réseaux beaucoup plus profonds que ce qui était possible auparavant.

2. **Efficacité Computationnelle** : L'opération $\max(0, z)$ est une simple comparaison, beaucoup plus rapide à calculer qu'une exponentielle, ce qui accélère à la fois l'inférence et l'entraînement.

3. **Sparsité des Activations** : Pour toute entrée négative ($z \leq 0$), la sortie de ReLU est nulle. Cela signifie que pour une entrée donnée, une fraction significative des neurones du réseau peut être "inactive". Cette sparsité peut rendre les représentations apprises plus robustes et efficaces, et est parfois comparée à une forme de régularisation implicite.

ReLU n'est cependant pas sans inconvénients :

1. **Non-différentiabilité en $z = 0$** : Strictement parlant, la dérivée n'est pas définie en $z = 0$, car la dérivée à gauche (0) est différente de la dérivée à droite (1). En pratique, ce n'est pas un problème. La probabilité qu'une entrée soit exactement zéro est infinitésimale, et les bibliothèques d'apprentissage profond assignent simplement une valeur (généralement 0 ou 1) à ce point, ce qui n'affecte pas la convergence.

2. **Le Problème du "Dying ReLU"** : C'est le principal défaut de ReLU. Si un neurone, à cause d'une initialisation malheureuse ou d'une grande mise à jour de gradient, se retrouve avec des poids qui produisent systématiquement une entrée $z$ négative pour toutes les données d'entraînement, son activation sera toujours nulle. Par conséquent, sa dérivée sera toujours nulle. Le gradient ne pourra plus jamais passer par ce neurone, et ses poids ne seront jamais mis à jour. Le neurone est effectivement "mort" et inutile pour le reste de l'entraînement.

3. **Sortie non-centrée en zéro** : Les activations de ReLU sont toujours non-négatives. Cela peut ralentir légèrement la convergence par rapport à des activations centrées comme tanh.

##### Variantes de ReLU

Pour remédier au problème du "Dying ReLU", plusieurs variantes ont été proposées, notamment :

- **Leaky ReLU** : $\text{LReLU}(z) = \max(\alpha z, z)$, où $\alpha$ est une petite constante (ex: 0.01). Cela introduit une petite pente non nulle pour les entrées négatives, garantissant que le gradient puisse toujours circuler.

- **Parametric ReLU (PReLU)** : Similaire à Leaky ReLU, mais $\alpha$ est un paramètre appris par le réseau lui-même.

- **Exponential Linear Unit (ELU)** : Utilise une fonction exponentielle pour les entrées négatives, ce qui peut conduire à des activations moyennes plus proches de zéro et améliorer la robustesse au bruit.

#### Implémentation en Python/NumPy

```python
def sigmoid(Z):
    return 1 / (1 + np.exp(-Z)), Z

def sigmoid_derivative(Z):
    s, _ = sigmoid(Z)
    return s * (1 - s)

def relu(Z):
    return np.maximum(0, Z), Z

def relu_derivative(Z):
    dZ = np.ones_like(Z)
    dZ[Z <= 0] = 0
    return dZ

def tanh(Z):
    return np.tanh(Z), Z

def tanh_derivative(Z):
    t, _ = tanh(Z)
    return 1 - np.power(t, 2)
```

#### Tableau Comparatif des Fonctions d'Activation

| Fonction | Formule Mathématique | Dérivée | Plage de Sortie | Avantages | Inconvénients |
|----------|---------------------|---------|-----------------|-----------|---------------|
| Sigmoïde | $\sigma(z) = \frac{1}{1 + e^{-z}}$ | $\sigma(z)(1-\sigma(z))$ | $(0,1)$ | Interprétation probabiliste, lisse | Gradient évanescent, non-centrée en zéro, coûteuse |
| Tanh | $\tanh(z) = \frac{e^z - e^{-z}}{e^z + e^{-z}}$ | $1 - \tanh^2(z)$ | $(-1,1)$ | Centrée en zéro, lisse | Gradient évanescent, coûteuse |
| ReLU | $\text{ReLU}(z) = \max(0, z)$ | $1$ si $z > 0$, $0$ si $z \leq 0$ | $[0, +\infty)$ | Pas de gradient évanescent, efficace, sparsité | Dying ReLU, non-centrée en zéro |

### 2.2 Initialisation des Poids : Poser les Bonnes Fondations

L'initialisation des poids d'un réseau de neurones peut sembler être un détail technique, mais elle a un impact dramatique sur la capacité du réseau à apprendre efficacement. Une mauvaise initialisation peut condamner un réseau à converger lentement, voire à ne jamais converger du tout.

#### Le Problème de l'Initialisation Naïve

Si nous initialisons tous les poids à zéro, ou à la même valeur constante, un phénomène désastreux se produit : la **perte de symétrie**. Tous les neurones d'une même couche calculeront exactement la même sortie, et par conséquent, recevront exactement le même signal d'erreur lors de la rétropropagation. Ils se mettront à jour de manière identique et resteront identiques pour toujours. En essence, avoir $n$ neurones identiques équivaut à n'avoir qu'un seul neurone. Le réseau perd sa capacité à apprendre des représentations diverses et complexes.

Si, d'un autre côté, nous initialisons les poids avec des valeurs aléatoires trop grandes, les activations peuvent exploser (devenir très grandes), saturant les fonctions d'activation et menant à des gradients proches de zéro. Inversement, si les poids sont trop petits, les activations peuvent s'évanouir (devenir très petites), et les gradients peuvent également s'évanouir.

#### L'Analyse de la Variance : Maintenir le Signal

L'idée clé derrière les bonnes méthodes d'initialisation est de maintenir la variance des activations et des gradients à travers les couches. Cette analyse, bien que technique, est formelle. Considérons un réseau profond où, à chaque couche, la variance des sorties est multipliée par un facteur lié à la variance des poids. Si ce facteur est constamment supérieur à 1, la variance des activations augmentera de manière exponentielle à travers les couches, conduisant à des valeurs extrêmes et à des gradients qui "explosent" (deviennent très grands). Inversement, si le facteur est constamment inférieur à 1, la variance diminuera de manière exponentielle, conduisant à des activations proches de zéro et à des gradients qui "s'évanouissent". L'objectif d'une bonne initialisation est de régler les poids de manière à ce que la variance des activations (en propagation avant) et des gradients (en rétropropagation) reste stable à travers les couches.

#### Initialisation de Xavier/Glorot

Proposée par Xavier Glorot et Yoshua Bengio en 2010 dans leur article fondateur "Understanding the difficulty of training deep feedforward neural networks", cette méthode a été la première à aborder ce problème de manière systématique.

**Objectif** : Maintenir la variance du signal constante. Pour cela, la variance des poids $W$ d'une couche doit satisfaire deux conditions :

- Pour la propagation avant : $\text{Var}(W) \cdot n_{\text{in}} = 1 \Rightarrow \text{Var}(W) = \frac{1}{n_{\text{in}}}$
- Pour la rétropropagation : $\text{Var}(W) \cdot n_{\text{out}} = 1 \Rightarrow \text{Var}(W) = \frac{1}{n_{\text{out}}}$

où $n_{\text{in}}$ et $n_{\text{out}}$ sont respectivement le nombre d'unités en entrée et en sortie de la couche ("fan-in" et "fan-out").

**Formule** : Comme ces deux conditions sont souvent incompatibles, Glorot et Bengio proposent un compromis en utilisant la moyenne harmonique des deux contraintes, ce qui conduit à la variance suivante :

$$\text{Var}(W) = \frac{2}{n_{\text{in}} + n_{\text{out}}}$$

Pour initialiser les poids à partir d'une distribution normale, on utilise donc $\mathcal{N}(0, \frac{2}{n_{\text{in}} + n_{\text{out}}})$.

**Contexte** : Cette dérivation repose sur l'hypothèse que la fonction d'activation est linéaire autour de 0 avec une dérivée de 1. Cela la rend particulièrement bien adaptée aux fonctions d'activation symétriques et centrées en zéro comme la tangente hyperbolique (tanh).

#### Initialisation de He

L'initialisation de Glorot s'est avérée sous-optimale pour les réseaux utilisant des activations ReLU.

**Motivation** : La fonction ReLU, en mettant à zéro toute la partie négative de ses entrées, réduit la variance du signal qui la traverse d'un facteur 2. Pour préserver la variance globale, la variance des poids doit donc être doublée pour compenser cette réduction.

**Formule** : Proposée par Kaiming He et al. en 2015 dans "Delving Deep into Rectifiers: Surpassing Human-Level Performance on ImageNet Classification", cette méthode ajuste la variance pour tenir compte de la non-linéarité de ReLU :

$$\text{Var}(W) = \frac{2}{n_{\text{in}}}$$

Pour une initialisation normale, on utilise $\mathcal{N}(0, \frac{2}{n_{\text{in}}})$. Cette méthode est devenue la norme pour l'initialisation des réseaux utilisant des activations ReLU et ses variantes.

#### Implémentation en Python/NumPy

```python
def initialize_parameters_deep(layer_dims):
    """
    Initialise les paramètres pour un réseau profond.
    Peut utiliser l'initialisation de He (pour ReLU) ou Glorot.
    """
    params = {}
    L = len(layer_dims)

    for l in range(1, L):
        # He initialization (pour ReLU)
        params['W' + str(l)] = np.random.randn(layer_dims[l], layer_dims[l-1]) * np.sqrt(2 / layer_dims[l-1])
        
        # Glorot/Xavier initialization (pour tanh)
        # params['W' + str(l)] = np.random.randn(layer_dims[l], layer_dims[l-1]) * np.sqrt(2 / (layer_dims[l-1] + layer_dims[l]))
        
        params['b' + str(l)] = np.zeros((layer_dims[l], 1))
        
    return params
```

### 2.3 Normalisation des Données d'Entrée

La stabilité de la dynamique d'apprentissage ne dépend pas seulement des propriétés internes du réseau (activations, initialisation), mais aussi des données qu'on lui fournit.

**Motivation** : Comme illustré visuellement dans la leçon, si les caractéristiques d'entrée ont des échelles très différentes (par exemple, une caractéristique variant de 0 à 1 et une autre de 0 à 1000), la surface de la fonction de coût peut devenir très allongée et asymétrique.

**Effet sur l'Optimisation** : Sur une telle surface de coût, la descente de gradient a tendance à osciller, prenant de nombreux petits pas pour converger. En normalisant les données, la surface de coût devient plus symétrique et "circulaire". Le gradient pointe alors plus directement vers le minimum, ce qui accélère considérablement la convergence et rend l'entraînement moins sensible au choix du taux d'apprentissage.

**Procédure** : La méthode standard est la normalisation Z-score :

1. Sur le jeu de données d'entraînement, calculer la moyenne $\mu$ et l'écart-type $\sigma$ pour chaque caractéristique.
2. Normaliser les données d'entraînement : $X_{\text{train\_norm}} = \frac{X_{\text{train}} - \mu}{\sigma}$.
3. **Crucial** : Utiliser les mêmes $\mu$ et $\sigma$ calculés sur l'ensemble d'entraînement pour normaliser les données de test (et toute nouvelle donnée en production) : $X_{\text{test\_norm}} = \frac{X_{\text{test}} - \mu}{\sigma}$. Il ne faut jamais calculer $\mu$ et $\sigma$ sur l'ensemble de test, car cela introduirait une fuite d'information du test vers l'entraînement et le modèle s'attendrait à voir des données avec la distribution de l'entraînement.

L'entraînement stable des réseaux profonds repose sur un triptyque interdépendant : la fonction d'activation, l'initialisation des poids, et la normalisation des données. Une décision inappropriée sur l'un de ces éléments peut annuler les bénéfices des autres. Par exemple, l'utilisation de ReLU résout le problème de saturation mais modifie la statistique des activations. Ce changement rend l'initialisation de Glorot moins efficace et nécessite l'initialisation de He pour maintenir la variance du signal. À son tour, même avec une initialisation correcte, si les données d'entrée ne sont pas normalisées, la toute première couche peut produire des activations de grande magnitude, saturant potentiellement les couches suivantes et perturbant la dynamique stable que l'initialisation cherchait à établir. Chaque étape prépare le terrain pour la suivante, créant une chaîne de dépendances pour assurer une propagation saine du signal à travers le réseau.

## Chapitre 3 : Algorithmes d'Optimisation : Naviguer Efficacement l'Espace des Paramètres

Une fois que nous avons un moyen de calculer les gradients (rétropropagation) et de garantir une dynamique stable (activations et initialisation), la dernière pièce du puzzle est de savoir comment utiliser ces gradients pour mettre à jour les poids de la manière la plus efficace possible. C'est le rôle des algorithmes d'optimisation.

### 3.1 Au-delà de la Descente de Gradient : L'Approche par Mini-Lots

La descente de gradient "vanilla" présente un compromis fondamental qui la rend peu pratique pour les grands ensembles de données modernes.

- **Batch Gradient Descent** : Cette approche calcule le gradient en utilisant l'intégralité de l'ensemble de données d'entraînement avant chaque mise à jour des paramètres. L'avantage est que la direction du gradient est une estimation exacte de la pente de la fonction de coût. L'inconvénient est son coût de calcul prohibitif. Pour un million d'exemples, il faudrait effectuer une propagation avant et arrière sur un million d'exemples pour faire un seul pas de mise à jour.

- **Stochastic Gradient Descent (SGD)** : À l'extrême opposé, le SGD met à jour les paramètres après chaque exemple individuel. Les mises à jour sont extrêmement rapides et fréquentes. Le bruit inhérent à cette estimation du gradient peut aider l'algorithme à échapper aux minima locaux peu profonds. Cependant, cette même stochasticité rend la convergence très instable et oscillante.

- **Mini-Batch Gradient Descent** : Cette méthode est le compromis qui domine en pratique. Elle consiste à diviser l'ensemble de données d'entraînement en petits lots (mini-batches) de taille fixe (par exemple, 32, 64, 128 exemples). L'algorithme calcule le gradient sur un mini-batch et met à jour les paramètres. Il combine les avantages des deux mondes : il bénéficie de la vectorisation pour un calcul efficace sur GPU (comme le Batch GD) tout en conservant une stochasticité bénéfique qui lisse la convergence et accélère l'entraînement (comme le SGD).

#### Pseudocode du Mini-Batch Gradient Descent

```
Pour chaque époque (epoch):
  Mélanger l'ensemble de données d'entraînement (X, Y)
  Partitionner (X, Y) en mini-batches de taille `batch_size`
  Pour chaque mini-batch (X_mini, Y_mini):
    1. Propagation avant sur X_mini pour obtenir A^[L]
    2. Calculer le coût J sur (A^[L], Y_mini)
    3. Rétropropagation pour calculer les gradients dW, db
    4. Mettre à jour les paramètres en utilisant dW, db
```

### 3.2 L'Optimisation avec Momentum

La descente de gradient par mini-lots est efficace, mais elle peut encore être lente si la surface de coût présente des "ravins" : des régions où la surface est beaucoup plus raide dans une dimension que dans une autre. Dans de tels cas, l'optimiseur a tendance à osciller d'un côté à l'autre du ravin, ne progressant que lentement le long de sa pente douce.

**Intuition** : L'optimisation avec momentum (ou moment cinétique) a été introduite pour résoudre ce problème. L'idée, inspirée de la physique, est de donner une "inertie" à la mise à jour. Au lieu que la mise à jour ne dépende que du gradient actuel, elle dépend aussi d'une moyenne des gradients passés. Cela a pour effet d'amortir les oscillations dans les directions où le gradient change fréquemment de signe (les parois du ravin) et d'accélérer le mouvement dans les directions où le gradient est constant (le fond du ravin).

**Formulation Mathématique** : Introduit à l'origine par Polyak (1964), l'algorithme maintient un vecteur de "vitesse" $v$, qui est une moyenne mobile exponentielle des gradients passés.

$$v_{dW} = \beta v_{dW} + (1 - \beta) dW$$
$$v_{db} = \beta v_{db} + (1 - \beta) db$$

La mise à jour des paramètres se fait alors en utilisant cette vitesse :

$$W = W - \alpha v_{dW}$$
$$b = b - \alpha v_{db}$$

Ici, $\beta$ est l'hyperparamètre de momentum, généralement fixé à une valeur comme 0.9. L'importance du momentum, en conjonction avec une bonne initialisation, a été mise en évidence par Sutskever et al. (2013) comme étant un élément clé pour l'entraînement réussi des réseaux profonds.

#### Implémentation en Python

```python
# Initialisation
v_dW = np.zeros_like(W)
v_db = np.zeros_like(b)
beta = 0.9

# Dans la boucle d'entraînement
# ... (calcul de dW, db via backprop) ...
v_dW = beta * v_dW + (1 - beta) * dW
v_db = beta * v_db + (1 - beta) * db
W = W - learning_rate * v_dW
b = b - learning_rate * v_db
```

### 3.3 RMSProp (Root Mean Square Propagation)

RMSProp est un autre algorithme qui cherche à améliorer la descente de gradient, mais en se concentrant sur un problème différent : le fait qu'un taux d'apprentissage global unique peut ne pas être adapté à tous les paramètres.

**Motivation** : Certains paramètres peuvent nécessiter de petits pas d'apprentissage, tandis que d'autres pourraient bénéficier de pas plus grands. Des algorithmes comme Adagrad ont tenté de résoudre ce problème en adaptant le taux d'apprentissage pour chaque paramètre, mais ils souffraient d'un taux d'apprentissage qui diminuait de manière monotone et finissait par devenir trop petit, paralysant l'apprentissage. RMSProp a été conçu pour résoudre ce problème.

**Formulation Mathématique** : Proposé par Geoffrey Hinton dans son cours en ligne, RMSProp maintient une moyenne mobile exponentielle non pas des gradients eux-mêmes, mais de leurs carrés.

$$S_{dW} = \beta S_{dW} + (1 - \beta) (dW)^2$$
$$S_{db} = \beta S_{db} + (1 - \beta) (db)^2$$

La mise à jour des paramètres est ensuite normalisée par la racine carrée de cette moyenne :

$$W = W - \alpha \frac{dW}{\sqrt{S_{dW}} + \epsilon}$$
$$b = b - \alpha \frac{db}{\sqrt{S_{db}} + \epsilon}$$

**Mécanisme** : Le taux d'apprentissage effectif pour un paramètre est divisé par une mesure de la magnitude de ses gradients récents. Si les gradients pour un poids ont été importants (indiquant une direction de forte courbure), le terme $S_{dW}$ sera grand, et le pas de mise à jour sera réduit. Inversement, si les gradients ont été faibles, le pas sera plus grand. Cela permet d'équilibrer la vitesse d'apprentissage à travers les différents paramètres.

$\epsilon$ est une petite constante (ex: $10^{-8}$) pour la stabilité numérique.

#### Implémentation en Python

```python
# Initialisation
S_dW = np.zeros_like(W)
S_db = np.zeros_like(b)
beta = 0.999
epsilon = 1e-8

# Dans la boucle d'entraînement
# ... (calcul de dW, db via backprop) ...
S_dW = beta * S_dW + (1 - beta) * np.square(dW)
S_db = beta * S_db + (1 - beta) * np.square(db)
W = W - learning_rate * dW / (np.sqrt(S_dW) + epsilon)
b = b - learning_rate * db / (np.sqrt(S_db) + epsilon)
```

### 3.4 Adam (Adaptive Moment Estimation) : La Synthèse Moderne

Adam est sans doute l'algorithme d'optimisation le plus populaire et le plus utilisé aujourd'hui. Sa force réside dans le fait qu'il combine les idées du momentum et de RMSProp en une seule méthode robuste.

**Motivation** : Adam cherche à la fois à accélérer la convergence en utilisant une "mémoire" des directions passées (comme le momentum) et à adapter le taux d'apprentissage pour chaque paramètre (comme RMSProp). Il est souvent considéré comme l'optimiseur par défaut pour une large gamme de problèmes d'apprentissage profond.

**Formulation Mathématique** : Proposé par Diederik Kingma et Jimmy Ba en 2015 dans "Adam: A Method for Stochastic Optimization", l'algorithme maintient deux moyennes mobiles exponentielles :

1. **Estimation du 1er moment (la moyenne, comme le momentum)** :
   $$m_t = \beta_1 m_{t-1} + (1 - \beta_1) g_t$$

2. **Estimation du 2ème moment (la variance non centrée, comme RMSProp)** :
   $$v_t = \beta_2 v_{t-1} + (1 - \beta_2) g_t^2$$

où $g_t$ est le gradient au pas de temps $t$, et $\beta_1$ et $\beta_2$ sont les taux de décroissance (valeurs typiques : 0.9 et 0.999).

3. **Correction de Biais** : Comme $m_t$ et $v_t$ sont initialisés à zéro, ils sont biaisés vers zéro au début de l'entraînement. Adam corrige ce biais :

   $$\hat{m}_t = \frac{m_t}{1 - \beta_1^t}$$
   $$\hat{v}_t = \frac{v_t}{1 - \beta_2^t}$$

4. **Mise à Jour des Paramètres** :
   $$\theta_t = \theta_{t-1} - \alpha \frac{\hat{m}_t}{\sqrt{\hat{v}_t} + \epsilon}$$

**Avantages** : En combinant l'estimation du premier et du second moment, Adam bénéficie à la fois de l'accélération du momentum et de l'adaptation du taux d'apprentissage par paramètre de RMSProp, ce qui le rend très efficace et robuste dans une grande variété de scénarios.

#### Pseudocode et Implémentation

Le pseudocode détaillé est présenté dans l'article original. L'implémentation en Python suit directement cette logique.

```python
# Initialisation
m_dW, v_dW = np.zeros_like(W), np.zeros_like(W)
m_db, v_db = np.zeros_like(b), np.zeros_like(b)
beta1, beta2 = 0.9, 0.999
epsilon = 1e-8
t = 0

# Dans la boucle d'entraînement
# ... (calcul de dW, db via backprop) ...
t = t + 1
# Mise à jour des moments
m_dW = beta1 * m_dW + (1 - beta1) * dW
v_dW = beta2 * v_dW + (1 - beta2) * np.square(dW)
# Correction de biais
m_dW_corr = m_dW / (1 - beta1**t)
v_dW_corr = v_dW / (1 - beta2**t)
# Mise à jour des poids
W = W - learning_rate * m_dW_corr / (np.sqrt(v_dW_corr) + epsilon)

# (Répéter pour les biais b)
```

L'histoire des algorithmes d'optimisation en apprentissage profond illustre une progression claire de la simplicité vers une complexité adaptative. Chaque algorithme a été conçu pour surmonter une faiblesse spécifique de son prédécesseur, en suivant deux axes de développement principaux : l'incorporation de la mémoire (l'historique des gradients) et l'adaptativité (un taux d'apprentissage spécifique à chaque paramètre). SGD est le point de départ, simple mais naïf. Le momentum y ajoute une mémoire (la vitesse) pour lisser la trajectoire. RMSProp introduit l'adaptativité par paramètre, mais sans mémoire de direction. Adam représente la synthèse de ces deux concepts, intégrant à la fois la mémoire via le premier moment ($m_t$) et l'adaptativité via le second moment ($v_t$). Cette convergence d'idées explique sa popularité et sa robustesse en tant que solution "tout-en-un" aux défis courants de l'optimisation.

#### Tableau Comparatif des Algorithmes d'Optimisation

| Algorithme | Idée Clé | Règle de Mise à Jour (Simplifiée) | Avantages | Inconvénients |
|------------|----------|-----------------------------------|-----------|---------------|
| SGD | Suivre la pente la plus forte. | $W \leftarrow W - \alpha dW$ | Simple, peu de mémoire. | Lent dans les "ravins", sensible au bruit. |
| Momentum | Accumuler une "vitesse" pour lisser la trajectoire. | $v \leftarrow \beta v + (1-\beta) dW$; $W \leftarrow W - \alpha v$ | Accélère la convergence, amortit les oscillations. | Un hyperparamètre de plus ($\beta$). |
| RMSProp | Adapter le taux d'apprentissage par paramètre. | $S \leftarrow \beta S + (1-\beta) dW^2$; $W \leftarrow W - \alpha \frac{dW}{\sqrt{S} + \epsilon}$ | Gère les taux d'apprentissage hétérogènes. | Peut converger prématurément. |
| Adam | Combiner Momentum et RMSProp. | Utilise $m_t$ (momentum) et $v_t$ (RMSProp) avec correction de biais. | Robuste, efficace, souvent le meilleur choix par défaut. | Plus complexe, plus de variables d'état à stocker. |

## Conclusion

### Synthèse des Piliers de l'Entraînement

Ce cours a exploré les trois piliers fondamentaux qui sous-tendent l'entraînement réussi des réseaux de neurones profonds.

1. **Le Calcul du Gradient** : La rétropropagation est l'algorithme efficace qui, en appliquant la règle de dérivation en chaîne, nous permet de calculer l'influence de chaque paramètre sur l'erreur globale du réseau. C'est le moteur mécanique de l'apprentissage.

2. **La Stabilité de la Dynamique** : Pour que les gradients puissent se propager utilement à travers de nombreuses couches, un équilibre délicat doit être maintenu. Ce triptyque de la stabilité est assuré par le choix de fonctions d'activation non saturantes comme ReLU, une initialisation des poids qui préserve la variance du signal comme He ou Glorot, et la normalisation des données d'entrée pour créer une surface de coût plus propice à l'optimisation.

3. **L'Efficacité de la Convergence** : Des algorithmes d'optimisation sophistiqués comme Adam nous permettent de naviguer l'espace complexe des paramètres de manière beaucoup plus efficace qu'une simple descente de gradient, en incorporant des concepts de mémoire et d'adaptativité.

### L'Entraînement comme un Art Expérimental

Malgré les fondements théoriques de plus en plus solides, il est essentiel de reconnaître que l'apprentissage profond reste un domaine fortement empirique, comme le souligne la leçon originale. Le succès d'un modèle dépend de manière critique du réglage d'un grand nombre d'hyperparamètres : l'architecture du réseau (nombre de couches, nombre de neurones), le taux d'apprentissage, les paramètres de l'optimiseur ($\beta_1, \beta_2$), la taille du mini-batch, le type et la quantité de régularisation, etc. Il n'existe pas de solution unique, et la recherche de la meilleure configuration relève souvent d'un processus itératif d'expérimentation guidé par l'intuition et l'expérience.

### Ouverture

Ce cours a couvert le cycle complet de l'entraînement pour un perceptron multicouche standard. Ces principes constituent le socle sur lequel reposent des architectures bien plus complexes. Les concepts de rétropropagation, d'optimisation par Adam, et de fonctions d'activation ReLU sont omniprésents dans les réseaux de neurones convolutifs (CNNs) pour la vision par ordinateur, les réseaux de neurones récurrents (RNNs) et les Transformers pour le traitement du langage naturel. Comprendre en profondeur les mécanismes abordés ici est donc une étape indispensable pour quiconque souhaite maîtriser et innover dans le domaine de l'intelligence artificielle.
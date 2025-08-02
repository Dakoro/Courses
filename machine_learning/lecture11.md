# Des Fondations aux Profondeurs : Un Cours Magistral sur les Réseaux de Neurones

## Introduction : L'Avènement de l'Apprentissage Profond : Une Convergence de Piliers

L'apprentissage profond, ou deep learning, représente aujourd'hui une des branches les plus dynamiques et influentes de l'intelligence artificielle, alimentant des avancées spectaculaires dans des domaines aussi variés que la vision par ordinateur, le traitement du langage naturel et la reconnaissance vocale. Pourtant, l'émergence de ce domaine n'est pas le fruit d'une unique percée technologique, mais plutôt le résultat d'une convergence synergique de trois piliers fondamentaux qui se sont mutuellement renforcés au fil du temps : la disponibilité massive de données, l'explosion de la puissance de calcul et une série d'innovations algorithmiques cruciales.

### 1. Le Déluge de Données (Data)

Les modèles d'apprentissage profond sont, par nature, extrêmement flexibles et possèdent une capacité d'apprentissage qui croît avec la quantité de données qui leur est fournie. Pendant des décennies, l'un des principaux freins à leur développement était le manque de jeux de données suffisamment vastes et de haute qualité pour entraîner des modèles complexes sans qu'ils ne "mémorisent" simplement les exemples vus (un phénomène appelé surapprentissage).

Le tournant majeur fut la création de bases de données à très grande échelle. L'exemple le plus emblématique est sans doute ImageNet, un projet visionnaire initié par des chercheurs comme Fei-Fei Li. Consciente que le progrès en vision par ordinateur était limité par la qualité des données plus que par les modèles eux-mêmes, son équipe a entrepris de construire une base de données d'images massivement étiquetées, en s'appuyant sur l'ontologie de WordNet. Pour accomplir cette tâche titanesque, qui aurait nécessité plus de deux décennies de travail pour une seule personne, le projet a eu recours à la plateforme de crowdsourcing Amazon Mechanical Turk, mobilisant des milliers de travailleurs pour annoter des millions d'images. Le résultat, une base de données contenant plus de 14 millions d'images classées dans plus de 20 000 catégories, a fourni le "carburant" indispensable pour que les réseaux de neurones profonds puissent enfin révéler leur plein potentiel.

### 2. La Puissance de Calcul (Computation)

Les fondations théoriques de nombreux modèles profonds existaient bien avant leur succès pratique. Cependant, leur coût de calcul était prohibitif. L'entraînement d'un réseau avec des millions de paramètres sur des millions d'exemples était tout simplement hors de portée des processeurs centraux (CPU) de l'époque.

La révolution est venue de l'utilisation des processeurs graphiques (GPU) pour le calcul généraliste (GPGPU). Initialement conçus pour le rendu graphique dans les jeux vidéo, les GPU sont dotés d'une architecture massivement parallèle, capable d'effectuer des milliers d'opérations simples (comme des multiplications de matrices) simultanément. Cette caractéristique s'est avérée parfaitement adaptée à la nature des calculs dans les réseaux de neurones.

L'article fondateur d'AlexNet en 2012 a été la démonstration éclatante de cette nouvelle puissance. Pour entraîner leur modèle, qui comptait 60 millions de paramètres, Alex Krizhevsky et ses collaborateurs ont utilisé deux GPU NVIDIA GTX 580, une configuration qui leur a permis de réduire drastiquement le temps d'entraînement et de gérer une complexité de modèle inédite. Cette utilisation pionnière des GPU a non seulement rendu l'entraînement de réseaux profonds réalisable, mais elle a aussi ouvert la voie à des architectures toujours plus grandes et plus profondes.

### 3. Les Innovations Algorithmiques (Algorithms)

Enfin, la disponibilité des données et de la puissance de calcul n'aurait pas suffi sans des avancées algorithmiques clés pour rendre l'entraînement de ces réseaux profonds stable et efficace. Les chercheurs ont développé de nouvelles techniques pour surmonter les obstacles historiques qui freinaient l'apprentissage dans les réseaux à plusieurs couches.

Parmi ces innovations, on peut citer :

- **De nouvelles fonctions d'activation**, comme l'Unité Linéaire Rectifiée (ReLU), qui ont permis de résoudre en grande partie le problème de l'évanouissement du gradient (vanishing gradient), un phénomène qui empêchait les couches profondes d'un réseau d'apprendre correctement.

- **Des techniques de régularisation robustes**, comme le Dropout, qui consiste à désactiver aléatoirement des neurones pendant l'entraînement pour empêcher le modèle de devenir trop dépendant de certaines caractéristiques spécifiques aux données d'entraînement, améliorant ainsi sa capacité de généralisation.

- **Des optimiseurs plus sophistiqués** et des schémas d'ajustement du taux d'apprentissage.

Ces trois piliers ne sont pas indépendants ; ils ont créé une boucle de rétroaction positive qui a catalysé la révolution de l'apprentissage profond. La disponibilité d'ImageNet (données) a créé le besoin et l'opportunité d'entraîner des modèles plus grands. L'utilisation des GPU (calcul) a fourni les moyens de le faire. Les innovations algorithmiques (algorithmes) ont permis à ces modèles de converger vers des solutions performantes. Le succès spectaculaire d'AlexNet, qui a remporté le défi ImageNet en 2012 avec une marge considérable, a validé cette approche intégrée, déclenchant une vague mondiale d'investissements et de recherches dans ces trois domaines, et marquant le début de l'ère moderne du deep learning.

## Partie I : La Régression Logistique, le Neurone Fondamental

Avant de nous plonger dans la complexité des réseaux profonds, il est essentiel de maîtriser leur brique de base : le neurone. Paradoxalement, le meilleur point de départ pour comprendre un neurone est un algorithme que beaucoup considèrent comme un modèle de machine learning classique : la régression logistique. En la réinterprétant sous l'angle des réseaux de neurones, nous allons jeter les bases conceptuelles et notationnelles nécessaires pour la suite.

### 1.1. Le Problème : Classification Binaire et Représentation des Données

Commençons par un objectif simple et concret : la classification binaire. Étant donnée une image, nous voulons déterminer si elle contient un chat ou non. La sortie souhaitée est une valeur proche de 1 s'il y a un chat, et de 0 sinon.

En informatique, une image couleur est généralement représentée par une matrice tridimensionnelle. Par exemple, une image de 64 pixels par 64 pixels utilise trois canaux de couleur (Rouge, Vert, Bleu - RGB). Sa représentation numérique est donc une matrice de dimensions $64 \times 64 \times 3$. Pour que cette image puisse être traitée par un algorithme de régression logistique standard, qui attend un vecteur de caractéristiques en entrée, la première étape consiste à "aplatir" cette matrice. Cette opération transforme la structure 3D en un unique vecteur colonne, $x$, en concaténant toutes les valeurs des pixels.

La dimension de ce vecteur, notée $n_x$, est donc $64 \times 64 \times 3 = 12288$. Notre entrée est un vecteur $x \in \mathbb{R}^{n_x}$. L'espace des sorties, $Y$, est l'ensemble $\{0,1\}$.

### 1.2. Le Modèle du Neurone Unique : Architecture et Paramètres

La régression logistique peut être vue comme un réseau de neurones ne contenant qu'un seul neurone. Ce neurone effectue un calcul en deux étapes, une définition qui restera valable pour tous les neurones que nous rencontrerons par la suite.

**Équation 1 : Définition d'un Neurone**
$$\text{Neurone} = \text{Opération Linéaire} + \text{Fonction d'Activation}$$

**L'opération linéaire :** Le neurone calcule d'abord une combinaison linéaire de ses entrées, à laquelle s'ajoute un terme de biais. Cette étape est souvent notée $z$.

$$z = w^T x + b$$

où :
- $x \in \mathbb{R}^{n_x \times 1}$ est le vecteur d'entrée (l'image aplatie).
- $w \in \mathbb{R}^{n_x \times 1}$ est le vecteur des poids (weights). Chaque poids $w_j$ est associé à une caractéristique d'entrée $x_j$ (un pixel) et quantifie son importance.
- $b \in \mathbb{R}$ est le biais (bias). C'est un scalaire qui permet de décaler la fonction de décision, analogue à l'ordonnée à l'origine en régression linéaire.

Dans la littérature sur les réseaux de neurones, il est courant de voir la multiplication écrite comme $wx$, où $w$ est un vecteur ligne de dimension $1 \times n_x$. Cela est équivalent à $w^T x$ avec un $w$ en colonne, mais cette convention de notation (poids en ligne) se généralisera plus facilement aux architectures multi-neurones.

**La fonction d'activation :** Le résultat $z$ est ensuite passé à travers une fonction non-linéaire appelée fonction d'activation, ici la fonction sigmoïde (ou logistique), notée $\sigma$. Le résultat final, $\hat{y}$ (prononcé "y chapeau"), est notre prédiction.

$$\hat{y} = a = \sigma(z) = \frac{1}{1 + e^{-z}}$$

La fonction sigmoïde est particulièrement adaptée à la classification binaire car elle "écrase" n'importe quelle valeur réelle $z$ dans l'intervalle $(0,1)$. Ce résultat peut donc être interprété comme une probabilité : $\hat{y} = P(y=1|x)$, la probabilité que l'étiquette soit 1 (présence d'un chat) étant donné le vecteur d'entrée $x$.

Les paramètres de ce modèle, c'est-à-dire les valeurs que nous devons apprendre à partir des données, sont l'ensemble des poids $w$ et le biais $b$. Pour notre problème de détection de chat, le modèle a 12288 poids et 1 biais, soit un total de 12289 paramètres à entraîner.

**Équation 2 : Définition d'un Modèle**
$$\text{Modèle} = \text{Architecture} + \text{Paramètres}$$

L'architecture définit la structure du calcul (ici, un seul neurone avec une activation sigmoïde), tandis que les paramètres sont les valeurs numériques qui peuplent cette structure et qui sont ajustées pendant l'entraînement.

### 1.3. La Fonction de Coût et l'Objectif d'Apprentissage

Pour entraîner notre modèle, nous avons besoin d'une manière de quantifier à quel point ses prédictions sont "mauvaises" par rapport aux véritables étiquettes. C'est le rôle de la fonction de perte (loss function). Pour la régression logistique, la fonction de perte standard est la perte par entropie croisée binaire (binary cross-entropy loss).

Pour un unique exemple d'entraînement $(x,y)$, la perte est définie comme suit :

$$\mathcal{L}(\hat{y},y) = -[y\log(\hat{y}) + (1-y)\log(1-\hat{y})]$$

Cette formule, bien que pouvant paraître arbitraire à première vue, découle directement du principe du maximum de vraisemblance (Maximum Likelihood Estimation, MLE). En interprétant $\hat{y}$ comme $P(y=1|x)$, et donc $1-\hat{y}$ comme $P(y=0|x)$, nous pouvons écrire la probabilité de l'étiquette observée $y$ comme : $P(y|x) = \hat{y}^y(1-\hat{y})^{1-y}$. L'objectif de la MLE est de trouver les paramètres $(w,b)$ qui maximisent la vraisemblance (la probabilité conjointe) de l'ensemble des données d'entraînement. Pour des raisons de simplicité mathématique et de stabilité numérique, on minimise le logarithme négatif de cette vraisemblance, ce qui nous mène directement à la formule de l'entropie croisée.

L'entropie croisée mesure la "distance" entre deux distributions de probabilité. Ici, elle mesure l'écart entre la distribution de probabilité prédite par le modèle $(\hat{y}, 1-\hat{y})$ et la distribution de probabilité réelle $(y, 1-y)$. Minimiser cette perte revient à rendre la distribution prédite aussi proche que possible de la distribution réelle.

Pour évaluer la performance sur l'ensemble du jeu de données de $m$ exemples, on calcule la fonction de coût (cost function), $J$, qui est simplement la moyenne des pertes sur tous les exemples :

$$J(w,b) = \frac{1}{m} \sum_{i=1}^{m} \mathcal{L}(\hat{y}^{(i)}, y^{(i)})$$

### 1.4. L'Optimisation : La Descente de Gradient

L'entraînement du modèle consiste à trouver les valeurs de $w$ et $b$ qui minimisent la fonction de coût $J(w,b)$. L'algorithme le plus utilisé pour cette tâche est la descente de gradient (gradient descent). C'est un processus itératif qui ajuste les paramètres dans la direction opposée du gradient de la fonction de coût, c'est-à-dire dans la direction de la plus forte pente descendante.

Le processus d'entraînement se décompose en trois phases :

1. **Initialisation :** Les paramètres $w$ et $b$ sont initialisés (souvent avec de petites valeurs aléatoires pour $w$ et à 0 pour $b$).

2. **Optimisation :** On répète les étapes suivantes pour un certain nombre d'itérations :
   
   a. **Propagation avant (Forward Propagation) :** Pour chaque exemple d'entraînement, on calcule la prédiction $\hat{y}$ en utilisant les valeurs actuelles de $w$ et $b$.
   
   b. **Calcul du coût :** On calcule la fonction de coût $J(w,b)$ pour évaluer la performance actuelle.
   
   c. **Rétropropagation (Backward Propagation) :** On calcule les gradients (les dérivées partielles) du coût par rapport à chaque paramètre : $\frac{\partial J}{\partial w}$ et $\frac{\partial J}{\partial b}$. Ces gradients indiquent comment un petit changement dans chaque paramètre affecterait le coût total.
   
   d. **Mise à jour des paramètres :** On met à jour les paramètres en utilisant les règles suivantes :
   $$w := w - \alpha \frac{\partial J}{\partial w}$$
   $$b := b - \alpha \frac{\partial J}{\partial b}$$
   où $\alpha$ est le taux d'apprentissage (learning rate), un hyperparamètre qui contrôle la taille des pas effectués à chaque itération.

3. **Prédiction :** Une fois l'optimisation terminée, le modèle avec les paramètres finaux $(w,b)$ peut être utilisé pour faire des prédictions sur de nouvelles données.

Le calcul des gradients se fait via la règle de dérivation en chaîne. Par exemple, pour un poids $w_j$ :

$$\frac{\partial \mathcal{L}}{\partial w_j} = \frac{\partial \mathcal{L}}{\partial \hat{y}} \frac{\partial \hat{y}}{\partial z} \frac{\partial z}{\partial w_j}$$

Après calcul, on trouve des formules simples :

$$\frac{\partial \mathcal{L}}{\partial z} = \hat{y} - y$$

$$\frac{\partial \mathcal{L}}{\partial w_j} = (\hat{y} - y) x_j \quad \text{et} \quad \frac{\partial \mathcal{L}}{\partial b} = \hat{y} - y$$

Ces gradients, moyennés sur l'ensemble des données, sont ensuite utilisés pour la mise à jour des paramètres.

### 1.5. Implémentation Complète en Python

Voici une implémentation complète de la régression logistique à partir de zéro, en utilisant la bibliothèque Python NumPy pour les calculs numériques.

**Pseudocode de l'entraînement**

```
fonction optimiser(X, Y, nombre_iterations, taux_apprentissage):
    w, b = initialiser_parametres(dimension_de_X)

    pour i de 1 à nombre_iterations:
        // Étape de propagation
        A, cache = propagation_avant(X, w, b)
        coût = calculer_coût(A, Y)

        // Étape de rétropropagation
        dw, db = retropropagation(cache, A, X, Y)

        // Mise à jour des paramètres
        w = w - taux_apprentissage * dw
        b = b - taux_apprentissage * db

    retourner w, b
```

**Implémentation en Python/NumPy**

```python
import numpy as np

def sigmoid(z):
    """
    Calcule la fonction sigmoïde.
    Arguments:
    z -- un scalaire ou un tableau numpy de n'importe quelle taille.
    Retourne:
    s -- sigmoid(z)
    """
    s = 1 / (1 + np.exp(-z))
    return s

def initialize_with_zeros(dim):
    """
    Crée un vecteur de zéros de forme (dim, 1) pour w et initialise b à 0.
    Argument:
    dim -- taille du vecteur w que nous voulons (nombre de paramètres).
    Retourne:
    w -- vecteur initialisé de forme (dim, 1)
    b -- scalaire initialisé (biais)
    """
    w = np.zeros((dim, 1))
    b = 0.0
    return w, b

def propagate(w, b, X, Y):
    """
    Implémente la propagation avant et la rétropropagation pour calculer le coût et son gradient.
    Arguments:
    w -- poids, un tableau numpy de taille (num_px * num_px * 3, 1)
    b -- biais, un scalaire
    X -- données de taille (num_px * num_px * 3, nombre d'exemples)
    Y -- vecteur des vraies étiquettes (contenant 0 ou 1) de taille (1, nombre d'exemples)
    Retourne:
    cost -- coût d'entropie croisée pour la régression logistique
    dw -- gradient de la perte par rapport à w, donc de même forme que w
    db -- gradient de la perte par rapport à b, donc un scalaire
    """
    m = X.shape[1]  # Nombre d'exemples d'entraînement

    # PROPAGATION AVANT (FORWARD PROPAGATION)
    Z = np.dot(w.T, X) + b
    A = sigmoid(Z)  # Calcule l'activation
    cost = -1/m * np.sum(Y * np.log(A) + (1 - Y) * np.log(1 - A))

    # RÉTROPROPAGATION (BACKWARD PROPAGATION)
    dZ = A - Y
    dw = 1/m * np.dot(X, dZ.T)
    db = 1/m * np.sum(dZ)

    cost = np.squeeze(np.array(cost))

    grads = {"dw": dw, "db": db}
    return grads, cost

def optimize(w, b, X, Y, num_iterations=100, learning_rate=0.01, print_cost=False):
    """
    Optimise w et b en exécutant l'algorithme de descente de gradient.
    Retourne:
    params -- dictionnaire contenant les poids w et le biais b
    grads -- dictionnaire contenant les gradients des poids et du biais par rapport à la fonction de coût
    costs -- liste de tous les coûts calculés pendant l'optimisation, utilisée pour tracer la courbe d'apprentissage.
    """
    costs = []

    for i in range(num_iterations):
        grads, cost = propagate(w, b, X, Y)
        dw = grads["dw"]
        db = grads["db"]

        # Mise à jour des paramètres
        w = w - learning_rate * dw
        b = b - learning_rate * db

        if i % 100 == 0:
            costs.append(cost)
            if print_cost:
                print(f"Coût après l'itération {i}: {cost}")

    params = {"w": w, "b": b}
    grads = {"dw": dw, "db": db}
    return params, grads, costs

def predict(w, b, X):
    """
    Prédit si l'étiquette est 0 ou 1 en utilisant les paramètres de régression logistique appris (w, b).
    Retourne:
    Y_prediction -- un tableau numpy contenant toutes les prédictions (0/1) pour les exemples dans X.
    """
    m = X.shape[1]
    Y_prediction = np.zeros((1, m))
    w = w.reshape(X.shape[0], 1)

    A = sigmoid(np.dot(w.T, X) + b)

    for i in range(A.shape[1]):
        Y_prediction[0, i] = 1 if A[0, i] > 0.5 else 0

    return Y_prediction

def model(X_train, Y_train, X_test, Y_test, num_iterations=2000, learning_rate=0.5, print_cost=False):
    """
    Construit le modèle de régression logistique en appelant les fonctions implémentées précédemment.
    """
    w, b = initialize_with_zeros(X_train.shape[0])
    params, grads, costs = optimize(w, b, X_train, Y_train, num_iterations, learning_rate, print_cost)

    w = params["w"]
    b = params["b"]

    Y_prediction_test = predict(w, b, X_test)
    Y_prediction_train = predict(w, b, X_train)

    print(f"Précision sur l'ensemble d'entraînement: {100 - np.mean(np.abs(Y_prediction_train - Y_train)) * 100}%")
    print(f"Précision sur l'ensemble de test: {100 - np.mean(np.abs(Y_prediction_test - Y_test)) * 100}%")

    d = {"costs": costs,
         "Y_prediction_test": Y_prediction_test,
         "Y_prediction_train" : Y_prediction_train,
         "w" : w,
         "b" : b,
         "learning_rate" : learning_rate,
         "num_iterations": num_iterations}
    return d
```

## Partie II : Vers la Classification Multi-Classes : Étendre le Modèle

La régression logistique fournit un cadre solide pour la classification binaire. Cependant, de nombreux problèmes du monde réel nécessitent de choisir parmi plus de deux catégories. Par exemple, au lieu de simplement détecter un "chat", nous pourrions vouloir identifier s'il s'agit d'un "chat", d'un "lion" ou d'un "iguane". Cette extension vers la classification multi-classes peut être abordée de deux manières principales, en fonction des contraintes du problème.

### 2.1. Cas 1 : Classification Multi-Label (Étiquettes Indépendantes)

Le premier scénario, connu sous le nom de classification multi-label, se présente lorsque les classes ne sont pas mutuellement exclusives. Cela signifie qu'une même image peut contenir plusieurs des objets que nous cherchons à détecter. Par exemple, une photo pourrait contenir à la fois un chat et un lion. Dans ce cas, l'étiquette associée à l'image ne serait plus un simple 0 ou 1, mais un vecteur où chaque composante indique la présence (1) ou l'absence (0) d'une classe spécifique. Pour notre exemple à trois animaux, une image avec un chat et un lion mais sans iguane aurait l'étiquette $y = [1, 1, 0]^T$.

Pour résoudre ce problème, l'approche la plus simple et la plus intuitive consiste à construire un modèle avec autant de neurones de sortie qu'il y a de classes. Chaque neurone se comportera comme un classifieur binaire indépendant pour sa classe respective.

L'architecture serait la suivante :

- **Entrée :** Le vecteur aplati $x \in \mathbb{R}^{n_x}$.
- **Couche de sortie :** Une couche avec $C$ neurones (ici, $C=3$).
- **Connexions :** Chaque neurone de sortie est entièrement connecté à toutes les entrées $x_j$.
- **Activation :** Chaque neurone de sortie utilise une fonction d'activation sigmoïde.

Les équations pour les trois neurones de sortie seraient :

$$\hat{y}_1 = \sigma(w_1^T x + b_1) \quad \text{(Probabilité d'un chat)}$$
$$\hat{y}_2 = \sigma(w_2^T x + b_2) \quad \text{(Probabilité d'un lion)}$$
$$\hat{y}_3 = \sigma(w_3^T x + b_3) \quad \text{(Probabilité d'un iguane)}$$

Chaque neurone $(w_j, b_j)$ est entraîné indépendamment pour prédire la probabilité de sa classe, sans tenir compte des autres. La sortie du modèle est un vecteur $\hat{y} = [\hat{y}_1, \hat{y}_2, \hat{y}_3]^T$, où chaque composante est une probabilité comprise entre 0 et 1. La somme de ces probabilités n'a aucune raison d'être égale à 1.

La fonction de coût pour ce modèle est une généralisation directe de l'entropie croisée binaire. C'est simplement la somme (ou la moyenne) des pertes de chaque classifieur binaire :

$$J(W,b) = \frac{1}{m} \sum_{i=1}^{m} \sum_{j=1}^{C} \mathcal{L}(\hat{y}_j^{(i)}, y_j^{(i)})$$

$$= -\frac{1}{m} \sum_{i=1}^{m} \sum_{j=1}^{C} [y_j^{(i)} \log(\hat{y}_j^{(i)}) + (1-y_j^{(i)}) \log(1-\hat{y}_j^{(i)})]$$

Puisque chaque terme de la somme interne ne dépend que des paramètres de son neurone respectif $(w_j, b_j)$, le calcul des gradients pour un neurone n'est pas affecté par les autres. L'entraînement de ce système est donc équivalent à l'entraînement de $C$ régressions logistiques en parallèle.

### 2.2. Cas 2 : Classification Multi-Classe Mutuellement Exclusive (Softmax)

Le second scénario, beaucoup plus courant, est celui où les classes sont mutuellement exclusives. Une image contient soit un chat, soit un lion, soit un iguane, mais jamais plus d'un à la fois. C'est le cas de la plupart des problèmes de classification standard (par exemple, classifier un chiffre manuscrit de 0 à 9). Dans ce contexte, les étiquettes sont représentées par un vecteur one-hot : un vecteur de zéros avec un seul 1 à la position correspondant à la classe correcte. Par exemple, $y = [1, 0, 0]^T$ pour un chat, $y = [0, 1, 0]^T$ pour un lion, etc.

Dans ce cas, utiliser des activations sigmoïdes indépendantes n'est pas optimal. Nous voulons que le modèle produise une distribution de probabilité sur l'ensemble des classes, où la somme des probabilités est égale à 1. Si nous sommes très sûrs qu'il ne s'agit ni d'un chat ni d'un lion, la probabilité d'un iguane devrait automatiquement augmenter. Cette interdépendance entre les sorties est capturée par la fonction d'activation Softmax.

L'architecture est similaire à celle du cas multi-label, mais la fonction d'activation de la couche de sortie change.

**Calcul des scores (logits) :** Comme auparavant, chaque neurone de sortie calcule un score linéaire, que nous noterons $z_j$.

$$z_j = w_j^T x + b_j \quad \text{pour } j = 1, \ldots, C$$

Ces valeurs $z_j$ sont appelées logits. Elles peuvent être interprétées comme des mesures de confiance non normalisées pour chaque classe.

**Application de la fonction Softmax :** Au lieu d'appliquer une sigmoïde à chaque logit, nous appliquons la fonction Softmax à l'ensemble du vecteur de logits $z = [z_1, \ldots, z_C]^T$. La probabilité prédite pour la classe $j$ est alors :

$$\hat{y}_j = \text{softmax}(z)_j = \frac{e^{z_j}}{\sum_{k=1}^{C} e^{z_k}}$$

La fonction Softmax possède deux propriétés cruciales :

1. **Positivité et Normalisation :** Chaque sortie $\hat{y}_j$ est dans l'intervalle $(0,1)$, et la somme de toutes les sorties est égale à 1 ($\sum_{j=1}^{C} \hat{y}_j = 1$). La sortie $\hat{y}$ est donc une distribution de probabilité valide.

2. **Monotonicité :** Elle préserve l'ordre des logits. Si $z_i > z_j$, alors $\hat{y}_i > \hat{y}_j$. De plus, l'exponentielle amplifie les différences : les classes avec des scores plus élevés reçoivent une part disproportionnellement plus grande de la probabilité totale.

Le nom "softmax" est une contraction de "soft maximum", car elle peut être vue comme une approximation "douce" (différentiable) de la fonction argmax qui renverrait 1 pour la classe ayant le plus grand logit et 0 pour les autres. Ses origines conceptuelles remontent à la distribution de Boltzmann en mécanique statistique, où elle est utilisée pour modéliser la probabilité qu'un système se trouve dans un état d'énergie donné.

### 2.3. La Perte par Entropie Croisée Catégorielle

Avec une sortie Softmax, la fonction de coût appropriée est la perte par entropie croisée catégorielle (categorical cross-entropy loss). Pour un seul exemple $(x,y)$ où $y$ est un vecteur one-hot, la perte est définie comme :

$$\mathcal{L}(\hat{y},y) = -\sum_{j=1}^{C} y_j \log(\hat{y}_j)$$

Puisque $y$ est un vecteur one-hot, un seul de ses éléments $y_k$ est égal à 1, et tous les autres sont nuls. La somme se simplifie donc en :

$$\mathcal{L}(\hat{y},y) = -\log(\hat{y}_k)$$

où $k$ est l'indice de la classe correcte. L'objectif est donc de maximiser le logarithme de la probabilité prédite pour la bonne classe, ce qui est intuitivement correct.

La combinaison de la fonction d'activation Softmax et de la perte par entropie croisée catégorielle est extrêmement répandue en apprentissage profond. Cette popularité n'est pas un hasard. Elle est due à une propriété mathématique particulièrement élégante qui simplifie grandement le calcul des gradients. Lorsqu'on calcule la dérivée de la perte $\mathcal{L}$ par rapport au vecteur de logits $z$, on obtient un résultat remarquablement simple :

$$\frac{\partial \mathcal{L}}{\partial z_j} = \hat{y}_j - y_j$$

Le gradient pour chaque logit est simplement la différence entre la probabilité prédite et la probabilité réelle ("prédiction - vérité"). Cette simplicité rend la rétropropagation plus stable et plus efficace. L'utilisation conjointe de ces deux fonctions crée un paysage d'optimisation lisse et convexe, facilitant la convergence de la descente de gradient vers une bonne solution.

### 2.4. Implémentation de la Régression Softmax

Voici le pseudocode et l'implémentation Python pour un modèle de régression Softmax.

**Pseudocode de la régression Softmax**

```
fonction optimiser_softmax(X, Y_one_hot, nombre_iterations, taux_apprentissage):
    W, b = initialiser_parametres(dimension_de_X, nombre_de_classes)

    pour i de 1 à nombre_iterations:
        // Propagation avant
        Z = W.T * X + b
        A = softmax(Z)
        coût = calculer_coût_entropie_croisée(A, Y_one_hot)

        // Rétropropagation
        dZ = A - Y_one_hot
        dW = (1/m) * X * dZ.T
        db = (1/m) * somme(dZ)

        // Mise à jour des paramètres
        W = W - taux_apprentissage * dW
        b = b - taux_apprentissage * db

    retourner W, b
```

**Implémentation en Python/NumPy**

Une précaution importante lors de l'implémentation de la fonction Softmax est la stabilité numérique. L'exponentiation de grands nombres positifs peut rapidement conduire à des valeurs infinies (overflow). Pour éviter cela, on soustrait la valeur maximale des logits à chaque logit avant de prendre l'exponentielle. Cette opération est mathématiquement équivalente et numériquement stable.

```python
import numpy as np

def softmax(z):
    """
    Calcule la fonction softmax de manière numériquement stable.
    Argument:
    z -- un tableau numpy de forme (nombre_de_classes, nombre_d_exemples)
    Retourne:
    s -- les probabilités softmax, de même forme que z
    """
    # Soustraction du max pour la stabilité numérique
    z_shifted = z - np.max(z, axis=0, keepdims=True)
    exps = np.exp(z_shifted)
    s = exps / np.sum(exps, axis=0, keepdims=True)
    return s

def compute_cost_categorical(A, Y):
    """
    Calcule le coût d'entropie croisée catégorielle.
    Arguments:
    A -- probabilités de sortie (après softmax), de forme (nombre_de_classes, nombre_d_exemples)
    Y -- étiquettes one-hot, de forme (nombre_de_classes, nombre_d_exemples)
    Retourne:
    cost -- le coût
    """
    m = Y.shape[1]
    cost = -1/m * np.sum(Y * np.log(A + 1e-8))  # Ajout d'un petit epsilon pour éviter log(0)
    return np.squeeze(cost)

def softmax_model(X_train, Y_train_one_hot, X_test, Y_test_one_hot, num_iterations=2000, learning_rate=0.01):
    """
    Construit et entraîne un modèle de régression Softmax.
    """
    n_x, m = X_train.shape
    n_y = Y_train_one_hot.shape[0]

    # Initialisation des paramètres
    W = np.random.randn(n_x, n_y) * 0.01
    b = np.zeros((n_y, 1))

    costs = []

    for i in range(num_iterations):
        # Propagation avant
        Z = np.dot(W.T, X_train) + b
        A = softmax(Z)
        cost = compute_cost_categorical(A, Y_train_one_hot)

        # Rétropropagation
        dZ = A - Y_train_one_hot
        dW = 1/m * np.dot(X_train, dZ.T)
        db = 1/m * np.sum(dZ, axis=1, keepdims=True)

        # Mise à jour
        W = W - learning_rate * dW
        b = b - learning_rate * db

        if i % 100 == 0:
            costs.append(cost)
            print(f"Coût après l'itération {i}: {cost}")

    # Prédiction
    Z_test = np.dot(W.T, X_test) + b
    A_test = softmax(Z_test)
    predictions = np.argmax(A_test, axis=0)
    true_labels = np.argmax(Y_test_one_hot, axis=0)

    accuracy = np.mean(predictions == true_labels) * 100
    print(f"Précision sur l'ensemble de test: {accuracy}%")

    return {"W": W, "b": b, "costs": costs}
```

## Partie III : Construction de Réseaux de Neurones à Plusieurs Couches (Deep Neural Networks)

Après avoir établi les fondations avec des modèles à une seule couche, nous sommes prêts à faire le saut vers les réseaux de neurones profonds (Deep Neural Networks, DNN). L'idée centrale est d'empiler plusieurs couches de neurones les unes sur les autres, créant une hiérarchie de traitement qui permet d'apprendre des représentations de données de plus en plus complexes et abstraites.

### 3.1. Architecture : Couches d'Entrée, Cachées et de Sortie

Un réseau de neurones à plusieurs couches, également appelé Perceptron Multi-Couches (MLP), est structuré comme suit :

- **Couche d'entrée (Input Layer) :** Ce n'est pas une couche de calcul à proprement parler. Elle représente simplement les données d'entrée, c'est-à-dire le vecteur de caractéristiques $x$.

- **Couches cachées (Hidden Layers) :** Ce sont les couches de neurones situées entre la couche d'entrée et la couche de sortie. Un réseau est qualifié de "profond" s'il contient au moins une couche cachée (bien que dans la pratique, le terme soit réservé aux réseaux en ayant plusieurs). Ces couches sont dites "cachées" car leurs entrées et leurs sorties ne sont pas directement observées dans les données ; elles sont des constructions internes au modèle.

- **Couche de sortie (Output Layer) :** C'est la dernière couche de neurones, qui produit la prédiction finale du réseau (par exemple, $\hat{y}$).

Dans les architectures les plus courantes, les couches sont entièrement connectées (fully-connected ou dense). Cela signifie que chaque neurone d'une couche est connecté à tous les neurones de la couche précédente. Les neurones au sein d'une même couche, en revanche, ne sont pas connectés entre eux.

Prenons l'exemple d'un réseau à deux couches (une couche cachée et une couche de sortie) pour notre problème de classification de chats :

- **Couche d'entrée :** $n_x = 12288$ unités.
- **Première couche (cachée) :** Disons 3 neurones.
- **Deuxième couche (sortie) :** 1 neurone (pour la classification binaire).

Le nombre de paramètres dans ce réseau augmente considérablement.

- **Pour la première couche :** Chaque neurone est connecté aux $n_x$ entrées. Nous avons donc $3 \times n_x$ poids, plus 3 biais.
- **Pour la deuxième couche :** Le neurone de sortie est connecté aux 3 neurones de la couche cachée. Nous avons donc $1 \times 3$ poids, plus 1 biais.

Le nombre total de paramètres dépend de la taille de l'entrée et du nombre de neurones dans les couches cachées, ce qui confère au modèle une grande capacité de représentation.

Le tableau suivant résume les caractéristiques des architectures de classification de base que nous avons vues, qui correspondent toutes à des réseaux sans couche cachée.

| Caractéristique | Régression Logistique (Binaire) | Multi-Label (Sigmoïdes Indépendantes) | Régression Softmax (Multi-Classe) |
|---|---|---|---|
| **Cas d'Usage** | Classification binaire (ex: chat vs non-chat) | Classes non mutuellement exclusives (ex: une image peut contenir un chat ET un lion) | Classes mutuellement exclusives (ex: l'image contient un chat OU un lion OU un iguane) |
| **Couche de Sortie** | 1 neurone | C neurones (C = nombre de classes) | C neurones (C = nombre de classes) |
| **Activation de Sortie** | Sigmoïde | Sigmoïde (appliquée à chaque neurone) | Softmax (appliquée à tous les neurones) |
| **Fonction de Perte** | Entropie croisée binaire | Somme des entropies croisées binaires | Entropie croisée catégorielle |
| **Forme de l'Étiquette (y)** | Scalaire (0 ou 1) | Vecteur binaire (ex: `[1,0,1]`) | Vecteur one-hot (ex: `[0,1,0]`) |

### 3.2. Le Rôle des Couches Cachées : La Hiérarchie des Caractéristiques

La véritable puissance des réseaux profonds ne réside pas seulement dans leur capacité à modéliser des fonctions complexes, mais dans leur aptitude à apprendre automatiquement des représentations hiérarchiques des données. C'est un changement de paradigme fondamental par rapport à l'apprentissage automatique traditionnel.

Dans les approches classiques, une grande partie du travail consistait à concevoir manuellement des caractéristiques pertinentes (feature engineering). Un expert du domaine devait extraire des informations significatives des données brutes (par exemple, des ratios, des moyennes, des filtres spécifiques) pour ensuite les fournir à un classifieur simple.

Les réseaux profonds automatisent ce processus. Chaque couche apprend à transformer la représentation de la couche précédente en une représentation légèrement plus abstraite et plus utile pour la tâche finale.

- **Premières couches (proches de l'entrée) :** Elles apprennent à détecter des caractéristiques très simples et locales. Dans le cas d'une image, cela pourrait être des bords, des coins, des gradients de couleur. Un neurone pourrait s'activer en présence d'une ligne verticale, un autre en présence d'une ligne horizontale.

- **Couches intermédiaires :** Elles reçoivent en entrée les caractéristiques simples détectées par les couches précédentes et apprennent à les combiner pour former des concepts plus complexes. Par exemple, une couche pourrait apprendre à reconnaître une "oreille de chat" en combinant des détections de bords courbes et de textures spécifiques. Une autre pourrait apprendre à reconnaître un "œil".

- **Dernières couches (proches de la sortie) :** Elles assemblent ces parties complexes pour reconnaître des objets entiers. Le neurone de sortie final peut prendre une décision ("c'est un chat") en se basant sur la présence combinée d'oreilles, d'yeux, d'un museau, etc., détectés par les couches précédentes.

Cette capacité à créer de nouvelles caractéristiques utiles est ce qui distingue fondamentalement les méthodes comme la rétropropagation des approches plus anciennes. Les unités cachées apprennent à représenter les variables d'entrée d'une manière qui dépend de la tâche, capturant les régularités des données à travers leurs interactions. Ce processus de representation learning est au cœur du succès du deep learning : le modèle n'apprend pas seulement à classer, il apprend la meilleure façon de représenter les données pour pouvoir classer.

### 3.3. Les Fonctions d'Activation Modernes : Au-delà de la Sigmoïde

Le choix de la fonction d'activation est crucial, en particulier dans les réseaux profonds. Bien que la fonction sigmoïde soit utile pour la couche de sortie d'un classifieur binaire, son utilisation dans les couches cachées présente des inconvénients majeurs. Le plus notable est le problème de l'évanouissement du gradient (vanishing gradient). La dérivée de la fonction sigmoïde est maximale au centre (à 0.25) et tend vers zéro lorsque l'entrée devient très grande ou très petite. Dans un réseau profond, les gradients sont multipliés à travers les couches lors de la rétropropagation. Si de nombreuses couches ont de petits gradients, leur produit devient exponentiellement petit, et les premières couches du réseau n'apprennent presque plus rien.

Pour surmonter cet obstacle, la communauté a adopté une nouvelle fonction d'activation par défaut pour les couches cachées : l'Unité Linéaire Rectifiée (ReLU - Rectified Linear Unit). Sa définition est d'une simplicité désarmante :

$$\text{ReLU}(z) = \max(0, z)$$

Introduite et popularisée dans le contexte des réseaux de neurones profonds par des chercheurs comme Vinod Nair et Geoffrey Hinton en 2010, la fonction ReLU présente plusieurs avantages significatifs :

1. **Atténuation du problème de l'évanouissement du gradient :** Pour toute entrée positive, la dérivée de ReLU est constante et égale à 1. Cela empêche le gradient de diminuer lors de la rétropropagation à travers les neurones actifs, permettant un apprentissage plus efficace dans les réseaux très profonds.

2. **Efficacité de calcul :** L'opération $\max(0, z)$ est beaucoup plus rapide à calculer qu'une exponentielle, ce qui accélère considérablement l'entraînement.

3. **Sparsité :** Comme ReLU met à zéro toutes les entrées négatives, elle peut conduire à des activations "éparses", où seule une fraction des neurones d'une couche est active pour une entrée donnée. Cela peut rendre le modèle plus efficace en termes de mémoire et de calcul.

Cependant, ReLU n'est pas sans défaut. Son principal inconvénient est le problème du "Dead ReLU". Si l'entrée d'un neurone ReLU devient négative (par exemple, à cause d'une grande mise à jour de gradient négative), il produira une sortie de 0. Sa dérivée sera également de 0 pour toute entrée ultérieure. Par conséquent, ce neurone cessera d'apprendre et de se mettre à jour, car le gradient qui le traverse sera toujours nul. Il devient "mort" pour le reste de l'entraînement. Des variantes comme Leaky ReLU ou ELU ont été proposées pour résoudre ce problème.

### 3.4. La Propagation Avant (Forward Propagation) en Mode Matriciel

Pour un réseau profond, il est essentiel d'écrire les équations de manière vectorielle et matricielle pour une implémentation efficace. Utilisons la notation où l'indice entre crochets, $[l]$, désigne le numéro de la couche. Pour notre réseau à 2 couches (1 couche cachée, 1 couche de sortie) :

**Propagation à travers la couche 1 (cachée) :**

$$Z^{[1]} = W^{[1]} X + b^{[1]}$$
$$A^{[1]} = g^{[1]}(Z^{[1]})$$

où :
- $X$ est la matrice des données d'entrée, de dimension $(n_x, m)$, où $m$ est le nombre d'exemples dans le batch.
- $W^{[1]}$ est la matrice des poids de la couche 1, de dimension $(n^{[1]}, n_x)$, où $n^{[1]}$ est le nombre de neurones dans la couche 1.
- $b^{[1]}$ est le vecteur de biais de la couche 1, de dimension $(n^{[1]}, 1)$.
- $Z^{[1]}$ est la matrice des sorties linéaires, de dimension $(n^{[1]}, m)$.
- $g^{[1]}$ est la fonction d'activation de la couche 1 (par exemple, ReLU).
- $A^{[1]}$ est la matrice des activations de la couche 1, de dimension $(n^{[1]}, m)$. Elle sert d'entrée pour la couche suivante.

**Propagation à travers la couche 2 (sortie) :**

$$Z^{[2]} = W^{[2]} A^{[1]} + b^{[2]}$$
$$A^{[2]} = g^{[2]}(Z^{[2]})$$

où :
- $A^{[1]}$ est l'entrée de cette couche.
- $W^{[2]}$ est la matrice des poids de la couche 2, de dimension $(n^{[2]}, n^{[1]})$, où $n^{[2]}$ est le nombre de neurones dans la couche 2 (ici, 1).
- $b^{[2]}$ est le vecteur de biais de la couche 2, de dimension $(n^{[2]}, 1)$.
- $Z^{[2]}$ est la matrice des sorties linéaires de la couche 2, de dimension $(n^{[2]}, m)$.
- $g^{[2]}$ est la fonction d'activation de la couche 2 (par exemple, Sigmoïde pour la classification binaire).
- $A^{[2]} = \hat{Y}$ est la matrice des prédictions finales, de dimension $(n^{[2]}, m)$.

Un point crucial dans l'implémentation est l'addition du biais. Dans l'équation $Z^{[l]} = W^{[l]} X + b^{[l]}$, on additionne une matrice $(n^{[l]}, m)$ et un vecteur $(n^{[l]}, 1)$. La plupart des bibliothèques de calcul scientifique, comme NumPy, gèrent cela automatiquement grâce à un mécanisme appelé broadcasting. Le vecteur de biais $b^{[l]}$ est virtuellement dupliqué $m$ fois pour correspondre aux dimensions de $W^{[l]} X$, permettant une addition élément par élément sans avoir à créer explicitement une matrice de biais plus grande en mémoire. La maîtrise des dimensions de chaque matrice est une compétence fondamentale pour déboguer et implémenter des réseaux de neurones.

## Partie IV : Le Mécanisme d'Apprentissage : La Rétropropagation de l'Erreur (Backpropagation)

Nous avons défini l'architecture d'un réseau profond et la manière de propager les informations vers l'avant pour obtenir une prédiction. La question centrale reste : comment ajuster les millions de poids et de biais de ce réseau pour minimiser la fonction de coût? La réponse réside dans l'algorithme de rétropropagation de l'erreur (backpropagation), qui est le moteur de l'apprentissage dans la quasi-totalité des réseaux de neurones modernes.

### 4.1. Le Principe Fondamental : La Règle de Dérivation en Chaîne

La rétropropagation n'est pas un algorithme magique, mais une application astucieuse et efficace de la règle de dérivation en chaîne du calcul différentiel. Son objectif est de calculer le gradient de la fonction de coût par rapport à chaque paramètre du réseau ($dW^{[l]}$ et $db^{[l]}$ pour chaque couche $l$).

L'article de 1986 de David Rumelhart, Geoffrey Hinton et Ronald J. Williams, "Learning representations by back-propagating errors", a été un moment clé qui a popularisé cet algorithme au sein de la communauté de l'intelligence artificielle et des réseaux de neurones. Bien que l'idée de base existait auparavant dans d'autres domaines, cet article a démontré sa puissance pour entraîner des réseaux non-linéaires à plusieurs couches, capables d'apprendre des représentations internes complexes.

L'efficacité de la rétropropagation vient de sa structure récursive. Au lieu de calculer la dérivée de la perte par rapport à un poids des premières couches directement (ce qui serait un cauchemar de calcul), l'algorithme procède à l'envers :

1. Il calcule d'abord l'erreur à la sortie du réseau.
2. Il propage ensuite ce signal d'erreur vers l'arrière, couche par couche. À chaque couche, il calcule comment les paramètres de cette couche ont contribué à l'erreur de la couche suivante.
3. Ce faisant, il réutilise les calculs des couches ultérieures pour calculer les gradients des couches antérieures, évitant ainsi des calculs redondants massifs.

### 4.2. Dérivation Mathématique Complète pour un Réseau à 2 Couches

Explicitons le processus pour notre réseau à 2 couches (une couche cachée avec activation ReLU, une couche de sortie avec activation Sigmoïde).

**Rappel de la Propagation Avant (Forward Pass)**

Pendant la propagation avant, nous calculons et mettons en cache les valeurs suivantes pour un batch de $m$ exemples :

$$Z^{[1]} = W^{[1]} X + b^{[1]}$$
$$A^{[1]} = \text{ReLU}(Z^{[1]})$$
$$Z^{[2]} = W^{[2]} A^{[1]} + b^{[2]}$$
$$A^{[2]} = \sigma(Z^{[2]})$$

**Coût :** 
$$J = -\frac{1}{m} \sum_{i=1}^m \left[Y^{(i)} \log(A^{[2](i)}) + (1-Y^{(i)}) \log(1-A^{[2](i)}) \right]$$

Les valeurs à mettre en cache pour la rétropropagation sont $X, Y, Z^{[1]}, A^{[1]}, W^{[1]}, Z^{[2]}, A^{[2]}$.

**Rétropropagation (Backward Pass)**

Le but est de calculer $dW^{[1]}, db^{[1]}, dW^{[2]}, db^{[2]}$. Nous commençons par la fin.

**Étape 1 : Couche de sortie (Couche 2)**

Nous avons besoin de la dérivée du coût $J$ par rapport à $Z^{[2]}$. Comme nous l'avons vu pour la régression logistique, ce gradient a une forme très simple :

$$dZ^{[2]} = \frac{\partial J}{\partial Z^{[2]}} = A^{[2]} - Y$$

Dimensions : $dZ^{[2]}$ est de forme $(1, m)$, tout comme $A^{[2]}$ et $Y$.

Maintenant, nous pouvons calculer les gradients pour les paramètres de la couche 2, $W^{[2]}$ et $b^{[2]}$.

$$dW^{[2]} = \frac{\partial J}{\partial W^{[2]}} = \frac{1}{m} dZ^{[2]} (A^{[1]})^T$$

$$db^{[2]} = \frac{\partial J}{\partial b^{[2]}} = \frac{1}{m} \text{np.sum}(dZ^{[2]}, \text{axis}=1, \text{keepdims}=\text{True})$$

Dimensions : $dW^{[2]}$ est de forme $(1, n^{[1]})$, comme $W^{[2]}$. $db^{[2]}$ est de forme $(1, 1)$, comme $b^{[2]}$.

**Étape 2 : Couche cachée (Couche 1)**

Pour remonter à la couche 1, nous devons d'abord calculer la dérivée du coût par rapport à l'activation de la couche 1, $dA^{[1]}$. En utilisant la règle de la chaîne :

$$dA^{[1]} = \frac{\partial J}{\partial A^{[1]}} = (W^{[2]})^T dZ^{[2]}$$

Dimensions : $(W^{[2]})^T$ est $(n^{[1]}, 1)$, $dZ^{[2]}$ est $(1, m)$. Le résultat $dA^{[1]}$ est $(n^{[1]}, m)$.

Ensuite, nous propageons le gradient à travers la fonction d'activation ReLU de la couche 1.

$$dZ^{[1]} = \frac{\partial J}{\partial Z^{[1]}} = dA^{[1]} * g'^{[1]}(Z^{[1]})$$

où $*$ représente la multiplication élément par élément (Hadamard product). La dérivée de ReLU, $g'^{[1]}$, est 1 si l'entrée est positive, et 0 sinon.

Dimensions : $dZ^{[1]}$, $dA^{[1]}$ et $g'^{[1]}(Z^{[1]})$ sont toutes de forme $(n^{[1]}, m)$.

Enfin, nous pouvons calculer les gradients pour les paramètres de la couche 1, $W^{[1]}$ et $b^{[1]}$.

$$dW^{[1]} = \frac{\partial J}{\partial W^{[1]}} = \frac{1}{m} dZ^{[1]} X^T$$

$$db^{[1]} = \frac{\partial J}{\partial b^{[1]}} = \frac{1}{m} \text{np.sum}(dZ^{[1]}, \text{axis}=1, \text{keepdims}=\text{True})$$

Dimensions : $dW^{[1]}$ est de forme $(n^{[1]}, n_x)$, comme $W^{[1]}$. $db^{[1]}$ est de forme $(n^{[1]}, 1)$, comme $b^{[1]}$.

Le processus est maintenant terminé. Nous avons calculé tous les gradients nécessaires pour une étape de mise à jour de la descente de gradient.

### 4.3. Implémentation de la Rétropropagation

Voici le pseudocode et l'implémentation Python pour un réseau de neurones à 2 couches.

**Pseudocode de la rétropropagation**

```
fonction retropropagation(X, Y, cache):
    m = nombre d'exemples
    A1, A2 = cache['A1'], cache['A2']
    W2 = cache['W2']

    // Calculs pour la couche 2
    dZ2 = A2 - Y
    dW2 = (1/m) * dZ2 * A1.T
    db2 = (1/m) * somme(dZ2, axe=1)

    // Calculs pour la couche 1
    dA1 = W2.T * dZ2
    dZ1 = dA1 * dérivée_de_g1(cache['Z1'])
    dW1 = (1/m) * dZ1 * X.T
    db1 = (1/m) * somme(dZ1, axe=1)

    retourner dW1, db1, dW2, db2
```

**Implémentation en Python/NumPy**

```python
import numpy as np

def relu(z):
    return np.maximum(0, z)

def sigmoid(z):
    return 1 / (1 + np.exp(-z))

def relu_backward(dA, cache_z):
    Z = cache_z
    dZ = np.array(dA, copy=True)
    dZ[Z <= 0] = 0
    return dZ

def sigmoid_backward(dA, cache_z):
    s = 1 / (1 + np.exp(-cache_z))
    dZ = dA * s * (1 - s)
    return dZ

def initialize_parameters_deep(layer_dims):
    """
    Initialise les paramètres pour un réseau profond.
    Argument:
    layer_dims -- liste python contenant les dimensions de chaque couche.
    """
    parameters = {}
    L = len(layer_dims)
    for l in range(1, L):
        parameters[f'W{l}'] = np.random.randn(layer_dims[l], layer_dims[l-1]) * 0.01
        parameters[f'b{l}'] = np.zeros((layer_dims[l], 1))
    return parameters

def forward_propagation(X, parameters):
    """
    Implémente la propagation avant pour le modèle [LINEAR->RELU]*(L-1) -> LINEAR->SIGMOID.
    """
    caches = []
    A = X
    L = len(parameters) // 2

    for l in range(1, L):
        A_prev = A
        W = parameters[f'W{l}']
        b = parameters[f'b{l}']
        Z = np.dot(W, A_prev) + b
        A = relu(Z)
        cache = ((A_prev, W, b), Z)
        caches.append(cache)

    W = parameters[f'W{L}']
    b = parameters[f'b{L}']
    Z = np.dot(W, A) + b
    AL = sigmoid(Z)
    cache = ((A, W, b), Z)
    caches.append(cache)

    return AL, caches

def compute_cost(AL, Y):
    """
    Calcule le coût d'entropie croisée.
    """
    m = Y.shape[1]
    cost = -1/m * np.sum(Y * np.log(AL) + (1 - Y) * np.log(1 - AL))
    return np.squeeze(cost)

def backward_propagation(AL, Y, caches):
    """
    Implémente la rétropropagation pour le modèle [LINEAR->RELU]*(L-1) -> LINEAR->SIGMOID.
    """
    grads = {}
    L = len(caches)
    m = AL.shape[1]
    Y = Y.reshape(AL.shape)

    # Couche de sortie (Sigmoid)
    dAL = - (np.divide(Y, AL) - np.divide(1 - Y, 1 - AL))
    current_cache = caches[L-1]
    linear_cache, activation_cache = current_cache
    dZ_last = sigmoid_backward(dAL, activation_cache)
    A_prev, W, b = linear_cache
    grads[f"dW{L}"] = 1/m * np.dot(dZ_last, A_prev.T)
    grads[f"db{L}"] = 1/m * np.sum(dZ_last, axis=1, keepdims=True)
    grads[f"dA{L-1}"] = np.dot(W.T, dZ_last)

    # Boucle pour les couches cachées (ReLU)
    for l in reversed(range(L - 1)):
        current_cache = caches[l]
        linear_cache, activation_cache = current_cache
        
        dZ = relu_backward(grads[f"dA{l + 1}"], activation_cache)
        A_prev, W, b = linear_cache
        grads[f"dW{l + 1}"] = 1/m * np.dot(dZ, A_prev.T)
        grads[f"db{l + 1}"] = 1/m * np.sum(dZ, axis=1, keepdims=True)
        grads[f"dA{l}"] = np.dot(W.T, dZ)

    return grads

def update_parameters(parameters, grads, learning_rate):
    """
    Met à jour les paramètres en utilisant la descente de gradient.
    """
    L = len(parameters) // 2
    for l in range(L):
        parameters[f"W{l+1}"] = parameters[f"W{l+1}"] - learning_rate * grads[f"dW{l+1}"]
        parameters[f"b{l+1}"] = parameters[f"b{l+1}"] - learning_rate * grads[f"db{l+1}"]
    return parameters
```

## Conclusion : Perspectives et Architectures Historiques

Au cours de ce parcours, nous avons déconstruit les réseaux de neurones, depuis leur unité la plus fondamentale – le neurone, modélisé par la régression logistique – jusqu'à l'assemblage de réseaux profonds à plusieurs couches. Nous avons établi que chaque neurone est une simple combinaison d'une opération linéaire et d'une activation non-linéaire. Nous avons vu comment ces briques de base peuvent être organisées pour aborder différents types de problèmes de classification, en choisissant judicieusement la fonction d'activation (Sigmoïde, Softmax) et la fonction de coût (Entropie croisée) adaptées. Enfin, nous avons détaillé le mécanisme d'apprentissage universel qu'est la rétropropagation, une application efficace de la règle de dérivation en chaîne qui permet d'entraîner des modèles comptant des millions de paramètres.

Cependant, la construction d'un modèle de pointe est bien plus qu'une simple mise en œuvre de la rétropropagation. C'est un processus qui allie la rigueur scientifique des principes mathématiques à un art d'ingénierie, impliquant une créativité architecturale et une panoplie d'astuces empiriques pour maîtriser l'entraînement. Les principes que nous avons étudiés sont les fondations, mais le succès en pratique dépend de la manière dont ces fondations sont assemblées et ajustées. Pour illustrer ce point, il est instructif de se pencher sur deux architectures historiques qui ont marqué des tournants dans le domaine.

### LeNet-5 : Le Pionnier de l'Apprentissage de Représentations

En 1998, bien avant que le terme "deep learning" ne soit à la mode, Yann LeCun et ses collaborateurs ont publié un article décrivant LeNet-5, un réseau de neurones convolutionnel conçu pour la reconnaissance de chiffres manuscrits. LeNet-5 a été l'une des premières démonstrations commerciales réussies de réseaux de neurones, utilisée notamment pour lire les chèques dans le secteur bancaire.

Son architecture a introduit des concepts qui sont aujourd'hui des standards dans la vision par ordinateur :

- **Couches de convolution :** Au lieu de couches entièrement connectées, LeNet-5 utilise des couches de convolution qui appliquent des filtres sur l'image. Ces filtres apprennent à détecter des caractéristiques locales (comme des bords ou des coins), et le fait de partager les poids de ces filtres sur toute l'image réduit considérablement le nombre de paramètres et rend le modèle intrinsèquement invariant aux translations.

- **Couches de sous-échantillonnage (Pooling) :** Après les couches de convolution, des couches de pooling (ici, average pooling) réduisent la taille spatiale des cartes de caractéristiques, rendant la représentation plus compacte et plus robuste aux petites déformations.

L'architecture typique était une séquence de CONV → POOL → CONV → POOL → FC → FC. LeNet-5 a été une preuve de concept fondamentale : il a montré qu'un réseau pouvait apprendre les caractéristiques pertinentes directement à partir des pixels bruts dans un processus d'apprentissage de bout en bout (end-to-end), remplaçant ainsi la nécessité d'une ingénierie manuelle des caractéristiques.

### AlexNet : Le Catalyseur de la Révolution du Deep Learning

Si LeNet-5 a été le pionnier, AlexNet a été le catalyseur qui a enflammé la révolution du deep learning en 2012. En remportant de manière écrasante le concours ImageNet, AlexNet a prouvé que les réseaux de neurones profonds, à une échelle beaucoup plus grande, pouvaient surpasser toutes les autres approches en vision par ordinateur.

Son architecture était plus profonde et plus large que LeNet-5, mais son succès reposait sur une combinaison d'innovations architecturales et d'ingénierie qui ont permis de gérer cette complexité :

- **Utilisation de l'activation ReLU :** Comme nous l'avons vu, le remplacement des sigmoïdes par des ReLU a permis un entraînement beaucoup plus rapide et a atténué le problème de l'évanouissement du gradient.

- **Entraînement sur plusieurs GPU :** Le modèle était si grand qu'il a été réparti sur deux GPU, une prouesse d'ingénierie à l'époque qui a rendu l'entraînement possible.

- **Techniques de régularisation :** AlexNet a popularisé l'utilisation du Dropout dans ses couches entièrement connectées pour lutter contre le surapprentissage.

- **Augmentation des données (Data Augmentation) :** Pour enrichir le jeu de données d'entraînement, l'équipe a généré artificiellement de nouveaux exemples en appliquant des transformations aux images existantes (translations, retournements, changements de couleur), une technique aujourd'hui standard.

AlexNet a démontré que la profondeur du modèle était essentielle à sa performance et que, grâce à la convergence des données massives (ImageNet), du calcul parallèle (GPU) et des innovations algorithmiques (ReLU, Dropout), les obstacles à la construction de réseaux véritablement profonds pouvaient être surmontés.

En conclusion, bien que les architectures de réseaux de neurones continuent d'évoluer avec des concepts toujours plus sophistiqués (réseaux résiduels, attention, transformeurs), les principes fondamentaux que nous avons explorés – la propagation avant pour la prédiction, le calcul d'une fonction de coût pour mesurer l'erreur, et la rétropropagation pour ajuster les paramètres – demeurent le moteur universel de l'apprentissage dans l'immense majorité des modèles d'aujourd'hui. La maîtrise de ces fondations est la clé pour comprendre non seulement le fonctionnement des systèmes actuels, mais aussi pour innover et construire les systèmes de demain.
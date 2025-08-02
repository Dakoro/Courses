# Fondements et Pratiques du Machine Learning : De la Théorie à l'Implémentation en Python

## Introduction : L'IA, la Nouvelle Électricité

### La Révolution du Machine Learning : Impact et Opportunités

Nous vivons une époque de transformation technologique fondamentale, où l'intelligence artificielle (IA), et plus spécifiquement le machine learning (ML), s'impose comme une force motrice comparable à l'avènement de l'électricité il y a un siècle. Tout comme l'électricité a remodelé chaque industrie majeure, de l'agriculture aux transports, le machine learning est en train de redéfinir les fondements de notre monde économique, social et scientifique. La demande pour des compétences en machine learning connaît une croissance exponentielle, dépassant de loin l'offre disponible. Cette demande n'émane plus seulement des géants de la technologie, mais de secteurs aussi variés que la santé, la logistique, la finance, la manufacture, et même les sciences humaines comme le droit ou l'histoire, où les algorithmes d'apprentissage permettent de traiter des documents juridiques ou de mieux comprendre le passé.

Cette révolution offre des opportunités sans précédent. Maîtriser le machine learning, ce n'est pas seulement acquérir une compétence technique recherchée ; c'est se donner les moyens de devenir un acteur clé de l'innovation, que ce soit en rejoignant les équipes qui construisent les produits de demain dans de grandes entreprises, en lançant sa propre startup, ou en transformant une industrie traditionnelle de l'intérieur. Au-delà des perspectives de carrière exceptionnelles, le machine learning offre la possibilité de s'engager dans un "travail qui a du sens" (meaningful work). Les outils que nous allons étudier permettent d'aborder des problèmes sociétaux majeurs : améliorer les diagnostics médicaux pour sauver des vies, créer des tuteurs personnalisés pour chaque enfant, optimiser les réseaux de transport pour réduire notre impact environnemental, ou encore renforcer les mécanismes de notre démocratie. C'est cette capacité à générer un impact positif et tangible qui constitue la motivation la plus profonde pour se plonger dans ce domaine fascinant.

### Définitions Fondamentales : Le Cœur de l'Apprentissage

Malgré l'enthousiasme qu'il suscite, qu'est-ce que le machine learning? Pour bâtir une compréhension solide, il est essentiel de s'appuyer sur des définitions formelles qui ont jalonné l'histoire du domaine.

Une des premières et des plus célèbres définitions fut proposée par Arthur Samuel en 1959. Il décrivait le machine learning comme le "domaine d'étude qui donne aux ordinateurs la capacité d'apprendre sans être explicitement programmés". Samuel est connu pour avoir développé un programme de jeu de dames qui, en jouant des milliers de parties contre lui-même, a appris à identifier les configurations de plateau menant à la victoire. Le programme est finalement devenu meilleur que Samuel lui-même, démontrant pour la première fois qu'une machine pouvait acquérir une compétence que son propre créateur ne possédait pas à un tel niveau. Cet exemple historique illustre parfaitement l'idée centrale : le savoir n'est pas codé en dur par le programmeur, mais émerge des données et de l'expérience.

Une définition plus formelle, largement adoptée aujourd'hui, a été formulée par Tom Mitchell. Il définit un "problème d'apprentissage bien posé" (Well-posed Learning Problem) de la manière suivante : "Un programme est dit apprendre d'une expérience E par rapport à une tâche T et une mesure de performance P, si sa performance à la tâche T, mesurée par P, s'améliore avec l'expérience E".

Décortiquons cette définition avec l'exemple du programme de jeu de dames de Samuel :

- **Tâche (T)** : Jouer au jeu de dames.
- **Expérience (E)** : Le processus de faire jouer le programme des milliers de parties contre lui-même.
- **Performance (P)** : La probabilité que le programme gagne la prochaine partie contre un adversaire.

Le programme "apprend" car sa probabilité de gagner (P) augmente à mesure qu'il accumule des parties jouées (E). Cette structure (T, E, P) fournit un cadre rigoureux pour penser et concevoir presque tous les algorithmes de machine learning. Le passage de la vision large et sociétale de l'IA comme "nouvelle électricité" à ces définitions précises n'est pas anodin. Il s'agit d'une stratégie pédagogique délibérée : capturer d'abord l'imagination et la motivation en montrant le "pourquoi" — l'impact et le potentiel — avant de plonger dans le "comment" — la rigueur technique et les mécanismes formels. En comprenant d'abord l'enjeu, l'apprenant est mieux préparé et plus investi pour aborder la complexité des concepts qui suivront.

## Partie 1 : L'Apprentissage Supervisé – Apprendre avec un Guide

### Principes de base : Apprendre à partir de données étiquetées (X, y)

L'apprentissage supervisé est, de loin, le type d'apprentissage automatique le plus utilisé et le plus abouti aujourd'hui, responsable de la majorité de la valeur économique créée par le ML. Le concept central est simple et intuitif : nous apprenons à partir d'exemples. Plus formellement, nous disposons d'un jeu de données où chaque exemple est une paire constituée d'une entrée, notée X, et d'une sortie ou d'une "étiquette" correspondante, notée y. L'objectif de l'algorithme est d'apprendre une fonction de mappage, une relation, qui peut prédire la sortie y pour une nouvelle entrée X jamais vue auparavant.

Au sein de l'apprentissage supervisé, on distingue deux grandes catégories de problèmes, en fonction de la nature de la sortie y que l'on cherche à prédire :

1. **La Régression** : Le problème est dit de régression lorsque la variable de sortie y est une valeur continue, c'est-à-dire un nombre réel. Par exemple, prédire le prix d'une maison, la température de demain, ou le cours d'une action.

2. **La Classification** : Le problème est dit de classification lorsque la variable de sortie y appartient à un ensemble discret de catégories ou de classes. Par exemple, prédire si un email est un "spam" ou "non spam" (classification binaire), ou si une image représente un "chat", un "chien" ou un "oiseau" (classification multi-classes).

### La Régression : Prédire des Valeurs Continues

#### Algorithme : La Régression Linéaire

La régression linéaire est souvent le premier algorithme que l'on étudie, en raison de sa simplicité et de son interprétabilité. Imaginons que nous souhaitions prédire le prix d'une maison en fonction de sa surface. Nous disposons d'un jeu de données avec la surface (X) et le prix (y) de plusieurs maisons. L'objectif de la régression linéaire est de trouver la ligne droite qui "s'ajuste" le mieux à ces points de données.

##### Fondements Mathématiques

Cette ligne droite est décrite par l'équation $h_\theta(x) = \theta_0 + \theta_1 x$, où $x$ est la surface, $h_\theta(x)$ est le prix prédit, $\theta_0$ est l'ordonnée à l'origine (l'intercept) et $\theta_1$ est la pente de la droite (le coefficient). Pour trouver la "meilleure" ligne, nous devons définir ce que "meilleur" signifie. En machine learning, cela se fait via une fonction de coût (ou fonction de perte). Pour la régression linéaire, la fonction de coût la plus courante est l'Erreur Quadratique Moyenne (Mean Squared Error, MSE), qui mesure la moyenne des carrés des différences entre les prix réels $(y^{(i)})$ et les prix prédits $(h_\theta(x^{(i)}))$ pour tous les exemples $i$ de notre jeu de données.

$$J(\theta_0, \theta_1) = \frac{1}{2m} \sum_{i=1}^{m} (h_\theta(x^{(i)}) - y^{(i)})^2$$

L'objectif est de trouver les valeurs de $\theta_0$ et $\theta_1$ qui minimisent cette fonction de coût $J(\theta_0, \theta_1)$. Pour la régression linéaire, cette fonction de coût est convexe, ce qui signifie qu'elle a un seul minimum global. Ce minimum peut être trouvé de deux manières principales : par une méthode itérative comme la descente de gradient, ou par une solution analytique directe appelée équation normale. L'équation normale, qui calcule les sommes des produits des variables, fournit une solution directe pour les coefficients.

##### Tableau : Pseudocode de la Régression Linéaire (via Équation Normale)

Le pseudocode suivant décrit comment calculer les coefficients d'une régression linéaire simple de manière analytique.

| Étape | Description |
|-------|-------------|
| 1. Initialisation | Lire le nombre de points de données, n. Lire les paires de données $(X_i, Y_i)$ pour $i = 1$ à $n$. |
| 2. Calcul des Sommes | Initialiser sumX, sumY, sumX2, sumXY à 0. Parcourir les données et calculer les sommes nécessaires : $\sum X_i$, $\sum Y_i$, $\sum X_i^2$, $\sum X_i Y_i$. |
| 3. Calcul des Coefficients | Calculer la pente (b ou $\theta_1$) et l'ordonnée à l'origine (a ou $\theta_0$) en utilisant les formules dérivées de la minimisation de l'erreur quadratique : <br> $b = \frac{n(\sum X_i Y_i) - (\sum X_i)(\sum Y_i)}{n(\sum X_i^2) - (\sum X_i)^2}$ <br> et $a = \frac{\sum Y_i - b(\sum X_i)}{n}$. |
| 4. Résultat | Afficher les valeurs de a et b. L'équation de la droite de meilleur ajustement est $y = a + bx$. |

Ce pseudocode démystifie l'algorithme, le ramenant à une série d'opérations arithmétiques claires sur les données d'entrée.

##### Implémentation Python

En pratique, on utilise rarement l'implémentation manuelle, sauf à des fins pédagogiques. Les bibliothèques comme scikit-learn fournissent des implémentations optimisées et robustes.

```python
# Implémentation de la Régression Linéaire en Python

import numpy as np
from sklearn.linear_model import LinearRegression
import matplotlib.pyplot as plt

# 1. Génération de données synthétiques
# X représente la surface des maisons (en m²)
X = np.array([50, 70, 100, 130, 170, 200]).reshape(-1, 1)
# y représente le prix des maisons (en milliers d'euros)
y = np.array([150, 200, 250, 280, 350, 400])

# 2. Création et entraînement du modèle
# On instancie le modèle de régression linéaire
model = LinearRegression()

# On entraîne le modèle sur nos données (X, y)
# La méthode.fit() exécute les calculs vus précédemment pour trouver les meilleurs coefficients.
model.fit(X, y)

# 3. Extraction des coefficients et de l'intercept
# L'intercept (theta_0 ou 'a') est stocké dans l'attribut.intercept_
intercept = model.intercept_
# Le coefficient (theta_1 ou 'b') est stocké dans l'attribut.coef_
coefficient = model.coef_[0]

print(f"Équation de la droite : y = {intercept:.2f} + {coefficient:.2f} * x")
print(f"Intercept (theta_0): {model.intercept_:.2f}")
print(f"Coefficient (theta_1): {model.coef_[0]:.2f}")

# 4. Faire des prédictions
# Prédire le prix pour une nouvelle maison de 130 m²
nouvelle_surface = np.array([[130]])
prix_predit = model.predict(nouvelle_surface)[0]
print(f"Prix prédit pour une maison de 130 m² : {prix_predit:.2f} k€")

# 5. Visualisation des résultats
plt.scatter(X, y, color='blue', label='Données réelles')
plt.plot(X, model.predict(X), color='red', linewidth=2, label='Droite de régression')
plt.title('Régression Linéaire : Prix des maisons vs. Surface')
plt.xlabel('Surface (m²)')
plt.ylabel('Prix (k€)')
plt.legend()
plt.grid(True)
plt.show()
```

### La Classification : Prédire des Catégories Discrètes

#### Algorithme : La Régression Logistique

Passons maintenant à la classification. Imaginons que nous voulions prédire si une tumeur est maligne (1) ou bénigne (0) en fonction de sa taille. Tenter d'utiliser la régression linéaire pour ce problème serait inapproprié : la droite de régression pourrait prédire des valeurs bien au-delà de 1 ou en dessous de 0, ce qui n'a pas de sens pour une probabilité ou une classification binaire.

##### Fondements Mathématiques

La régression logistique résout ce problème en introduisant une fonction non linéaire, la fonction sigmoïde (ou fonction logistique), qui "écrase" n'importe quelle valeur réelle dans l'intervalle $[0, 1]$.

$$\sigma(z) = \frac{1}{1 + e^{-z}}$$

L'entrée $z$ de cette fonction est une combinaison linéaire des features, similaire à la régression linéaire : $z = \theta_0 + \theta_1 x_1 + ... + \theta_n x_n$. La sortie de la fonction sigmoïde, $h_\theta(x) = \sigma(z)$, peut être interprétée comme la probabilité que l'exemple appartienne à la classe positive (par exemple, $P(y=1|x;\theta)$).

La fonction de coût doit également être adaptée. L'erreur quadratique moyenne n'est plus appropriée car elle rendrait la fonction de coût non-convexe. On utilise à la place la fonction de coût log-vraisemblance négative (ou cross-entropy loss). Pour un seul exemple, le coût est :

$$\text{Cost}(h_\theta(x), y) = -y \log(h_\theta(x)) - (1-y) \log(1-h_\theta(x))$$

Cette fonction a la propriété d'être élevée lorsque le modèle se trompe (par exemple, prédit une faible probabilité pour la classe 1 alors que la vraie classe est 1) et faible lorsqu'il a raison. Contrairement à la régression linéaire, il n'y a pas de solution analytique pour minimiser ce coût. Nous devons utiliser un algorithme d'optimisation itératif comme la descente de gradient (Gradient Descent) pour trouver les paramètres $\theta$ optimaux.

##### Tableau : Pseudocode de la Régression Logistique (avec Descente de Gradient)

| Étape | Description |
|-------|-------------|
| 1. Initialisation | Initialiser les paramètres (poids) $\theta$ à des valeurs aléatoires ou à zéro. Choisir un taux d'apprentissage $\alpha$ et un nombre d'itérations. |
| 2. Boucle d'entraînement | Répéter pour le nombre d'itérations spécifié : |
| 2a. Calcul des Prédictions | Pour chaque exemple d'entraînement $x^{(i)}$, calculer la prédiction (probabilité) $h_\theta(x^{(i)}) = \sigma(\theta^T x^{(i)})$. |
| 2b. Calcul du Gradient | Calculer le vecteur de gradient de la fonction de coût $J(\theta)$ : <br> $\nabla J(\theta) = \frac{1}{m} \sum_{i=1}^{m} (h_\theta(x^{(i)}) - y^{(i)}) x^{(i)}$. |
| 2c. Mise à jour des Poids | Mettre à jour simultanément tous les paramètres $\theta_j$ : <br> $\theta_j := \theta_j - \alpha \nabla J(\theta)_j$. |
| 3. Résultat | Les paramètres $\theta$ finaux définissent le modèle entraîné. |

Ce processus itératif montre comment le modèle "apprend" progressivement en ajustant ses poids pour réduire l'erreur de prédiction.

##### Implémentation Python

scikit-learn rend l'implémentation de la régression logistique très simple.

```python
# Implémentation de la Régression Logistique en Python

import numpy as np
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import accuracy_score, confusion_matrix
import seaborn as sns
import matplotlib.pyplot as plt

# 1. Génération de données synthétiques pour la classification
# Feature 1: taille de la tumeur, Feature 2: âge du patient
np.random.seed(0)
# Données pour la classe 0 (bénigne)
X0 = np.random.normal(loc=[2, 50], scale=[0.5, 10], size=(100, 2))
y0 = np.zeros(100)
# Données pour la classe 1 (maligne)
X1 = np.random.normal(loc=[5, 60], scale=[1.5, 12], size=(100, 2))
y1 = np.ones(100)

X = np.vstack((X0, X1))
y = np.hstack((y0, y1))

# 2. Séparation des données en ensembles d'entraînement et de test
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.3, random_state=42)

# 3. Création et entraînement du modèle
# 'solver' est l'algorithme d'optimisation utilisé. 'liblinear' est bon pour les petits jeux de données.
# 'penalty' spécifie la régularisation pour éviter le sur-apprentissage.
model = LogisticRegression(solver='liblinear', random_state=42)

# Entraînement du modèle
model.fit(X_train, y_train)

# 4. Prédictions et Évaluation
y_pred = model.predict(X_test)
accuracy = accuracy_score(y_test, y_pred)
print(f"Précision du modèle : {accuracy:.2f}")

# Affichage de la matrice de confusion
cm = confusion_matrix(y_test, y_pred)
sns.heatmap(cm, annot=True, fmt='d', cmap='Blues')
plt.title('Matrice de Confusion')
plt.xlabel('Prédictions')
plt.ylabel('Valeurs Réelles')
plt.show()

# Pour la classification multi-classes, scikit-learn gère cela automatiquement
# via la stratégie 'One-vs-Rest' (OvR) par défaut.
# On peut aussi spécifier 'multinomial' pour certains solveurs comme 'lbfgs'.
# model_multi = LogisticRegression(multi_class='multinomial', solver='lbfgs')
```

### Classification Avancée : Les Machines à Vecteurs de Support (SVM)

Lorsque les données ne sont pas séparables par une simple ligne droite (ou un hyperplan dans des dimensions supérieures), nous avons besoin de modèles plus puissants. Les Machines à Vecteurs de Support (SVM) sont une classe d'algorithmes de classification extrêmement efficaces, particulièrement pour les problèmes complexes.

#### Introduction et Concepts

L'idée centrale du SVM est de trouver l'hyperplan qui non seulement sépare les classes, mais le fait avec la plus grande marge possible. La marge est la "rue" ou l'espace vide entre les points les plus proches de chaque classe et l'hyperplan de séparation. Les points de données qui se trouvent sur le bord de cette marge sont appelés les vecteurs de support (support vectors). Ce sont ces points, et uniquement eux, qui définissent la position et l'orientation de l'hyperplan. Cette propriété rend les SVM robustes et efficaces en mémoire.

#### L'Astuce du Noyau (Kernel Trick)

La véritable puissance des SVM réside dans l'astuce du noyau (kernel trick). Pour les données qui ne sont pas linéairement séparables dans leur espace d'origine, le SVM utilise une fonction noyau pour projeter les données dans un espace de caractéristiques de plus grande dimension où elles deviennent linéairement séparables. Le "truc" est que cet mappage est fait implicitement : l'algorithme n'a jamais besoin de calculer les coordonnées des points dans ce nouvel espace, il n'a besoin que de calculer les produits scalaires entre les points, ce que la fonction noyau fait efficacement. Cela permet d'utiliser des espaces de dimension infinie, comme le mentionne Andrew Ng, une idée fascinante qui permet de capturer des relations extraordinairement complexes.

Les noyaux les plus courants sont :

- **Linéaire** : $K(x_i, x_j) = x_i^T x_j$. Utilisé pour les données déjà linéairement séparables.
- **Polynomial** : $K(x_i, x_j) = (\gamma x_i^T x_j + r)^d$.
- **Gaussien (RBF - Radial Basis Function)** : $K(x_i, x_j) = \exp(-\gamma ||x_i - x_j||^2)$. C'est le noyau le plus populaire et le plus flexible, capable de gérer des frontières de décision très complexes.

#### Optimisation : Introduction à l'Optimisation Séquentielle Minimale (SMO)

L'entraînement d'un SVM revient à résoudre un problème d'optimisation quadratique (QP) sous contraintes, ce qui est calculatoirement coûteux. L'algorithme Sequential Minimal Optimization (SMO) est une avancée majeure qui a rendu les SVM pratiques. SMO décompose le grand problème de QP en une série de plus petits sous-problèmes possibles. À chaque étape, il sélectionne seulement deux multiplicateurs de Lagrange et les optimise analytiquement, ce qui est extrêmement rapide. Ce processus est répété jusqu'à ce que les conditions d'optimalité (les conditions de Karush-Kuhn-Tucker, ou KKT) soient satisfaites pour tous les points.

##### Tableau : Pseudocode simplifié de l'algorithme SMO

| Étape | Description |
|-------|-------------|
| 1. Initialisation | Initialiser tous les multiplicateurs de Lagrange $\alpha_i$ à 0. |
| 2. Boucle d'optimisation | Répéter jusqu'à convergence (c'est-à-dire, jusqu'à ce qu'aucune mise à jour ne soit possible) : |
| 2a. Sélection | Parcourir le jeu de données et sélectionner un multiplicateur $\alpha_i$ qui viole les conditions KKT. |
| 2b. Sélection du second | Choisir un second multiplicateur $\alpha_j$ de manière heuristique (par exemple, celui qui maximise la taille du pas de mise à jour). |
| 2c. Optimisation Analytique | Optimiser conjointement la paire $(\alpha_i, \alpha_j)$ en résolvant le sous-problème de manière analytique, tout en respectant les contraintes. |
| 2d. Mise à jour | Mettre à jour les $\alpha_i$, $\alpha_j$ et le seuil $b$ du modèle. |
| 3. Résultat | Les $\alpha_i$ non nuls correspondent aux vecteurs de support qui définissent le modèle final. |

Cette progression, de la régression linéaire aux SVM à noyau, illustre une trajectoire fondamentale en machine learning : l'escalade en complexité pour modéliser des relations de plus en plus sophistiquées dans les données. On commence avec un modèle simple et linéaire, et on introduit progressivement des non-linéarités (via la fonction sigmoïde) et des transformations d'espace (via les noyaux) pour s'adapter à la réalité des données. Parallèlement, cette complexité croissante des modèles exige des moteurs d'optimisation de plus en plus spécialisés. La solution analytique de la régression linéaire simple cède la place à la descente de gradient pour la régression logistique, qui elle-même est remplacée par des solveurs QP hautement efficaces comme SMO pour les SVM. Comprendre cette double évolution du pouvoir de représentation et de la machinerie d'optimisation est la clé pour devenir un praticien averti.

##### Implémentation Python

scikit-learn fournit la classe SVC (Support Vector Classification) pour implémenter les SVM.

```python
# Implémentation des SVM en Python

import numpy as np
from sklearn.datasets import make_moons, make_circles
from sklearn.model_selection import train_test_split
from sklearn.svm import SVC
from sklearn.metrics import accuracy_score
import matplotlib.pyplot as plt

# 1. Génération de données non-linéairement séparables
X, y = make_moons(n_samples=200, noise=0.1, random_state=42)

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.3, random_state=42)

# 2. Création et entraînement d'un SVM avec noyau linéaire
# Ce modèle devrait avoir du mal avec ces données.
svm_linear = SVC(kernel='linear')
svm_linear.fit(X_train, y_train)
y_pred_linear = svm_linear.predict(X_test)
print(f"Précision du SVM linéaire : {accuracy_score(y_test, y_pred_linear):.2f}")

# 3. Création et entraînement d'un SVM avec noyau RBF (Gaussien)
# 'C' est le paramètre de régularisation : un C faible crée une marge plus grande mais tolère des erreurs.
# 'gamma' définit l'influence d'un seul exemple d'entraînement.
svm_rbf = SVC(kernel='rbf', C=1.0, gamma='auto')
svm_rbf.fit(X_train, y_train)
y_pred_rbf = svm_rbf.predict(X_test)
print(f"Précision du SVM avec noyau RBF : {accuracy_score(y_test, y_pred_rbf):.2f}")

# 4. Fonction pour visualiser la frontière de décision
def plot_decision_boundary(model, X, y, title):
    h = .02  # step size in the mesh
    x_min, x_max = X[:, 0].min() - 0.5, X[:, 0].max() + 0.5
    y_min, y_max = X[:, 1].min() - 0.5, X[:, 1].max() + 0.5
    xx, yy = np.meshgrid(np.arange(x_min, x_max, h), np.arange(y_min, y_max, h))
    Z = model.predict(np.c_[xx.ravel(), yy.ravel()])
    Z = Z.reshape(xx.shape)
    plt.contourf(xx, yy, Z, cmap=plt.cm.coolwarm, alpha=0.8)
    plt.scatter(X[:, 0], X[:, 1], c=y, cmap=plt.cm.coolwarm, edgecolors='k')
    plt.title(title)
    plt.xlabel('Feature 1')
    plt.ylabel('Feature 2')

# Visualisation
plt.figure(figsize=(12, 5))
plt.subplot(1, 2, 1)
plot_decision_boundary(svm_linear, X, y, 'SVM avec noyau Linéaire')
plt.subplot(1, 2, 2)
plot_decision_boundary(svm_rbf, X, y, 'SVM avec noyau RBF')
plt.tight_layout()
plt.show()
```

## Partie 2 : Les Réseaux de Neurones et le Deep Learning – Une Plongée en Profondeur

Les réseaux de neurones, et en particulier le deep learning, représentent une sous-catégorie du machine learning qui a connu des avancées spectaculaires et qui est au cœur de nombreuses innovations récentes. Ils sont inspirés par la structure du cerveau humain et sont capables d'apprendre des représentations de données extrêmement complexes et hiérarchiques.

### Le Perceptron Multi-couches (MLP) : Architecture et Fonctionnement

Le modèle de base du deep learning est le Perceptron Multi-couches (MLP). Il s'agit d'un algorithme d'apprentissage supervisé qui peut apprendre des fonctions non linéaires complexes, que ce soit pour la classification ou la régression. Sa principale caractéristique est son architecture en couches :

1. **Une couche d'entrée (Input Layer)** : Elle reçoit les données brutes (les features). Chaque neurone de cette couche correspond à une feature.

2. **Une ou plusieurs couches cachées (Hidden Layers)** : C'est le cœur du réseau. C'est la présence de ces couches qui permet au modèle d'apprendre des relations non linéaires et de se distinguer de modèles plus simples comme la régression logistique. Un réseau avec de nombreuses couches cachées est qualifié de "profond" (deep).

3. **Une couche de sortie (Output Layer)** : Elle produit la prédiction finale. Le nombre de neurones et la fonction d'activation de cette couche dépendent de la tâche (par exemple, un seul neurone avec une fonction sigmoïde pour la classification binaire, ou N neurones avec une fonction softmax pour la classification à N classes).

Chaque neurone dans une couche cachée ou de sortie effectue une opération simple : il calcule une somme pondérée de toutes les sorties des neurones de la couche précédente, y ajoute un biais, puis applique une fonction d'activation non linéaire au résultat. Cette non-linéarité est cruciale ; sans elle, un réseau de plusieurs couches serait mathématiquement équivalent à un réseau à une seule couche et ne pourrait pas apprendre de fonctions complexes. Les fonctions d'activation courantes incluent la sigmoïde, la tangente hyperbolique (tanh), et surtout la Rectified Linear Unit (ReLU), qui est très populaire aujourd'hui pour son efficacité calculatoire.

### L'Algorithme de Rétropropagation du Gradient (Backpropagation)

Comment un réseau de neurones apprend-il? Le moteur de cet apprentissage est l'algorithme de rétropropagation du gradient (backpropagation). C'est une méthode remarquablement efficace pour calculer le gradient de la fonction de coût par rapport à chaque poids et biais du réseau. Ce gradient indique comment chaque paramètre doit être ajusté pour réduire l'erreur du modèle.

Le processus se déroule en deux passes :

1. **La Passe Avant (Forward Pass)** : Une entrée X est présentée au réseau. L'information se propage de la couche d'entrée vers la couche de sortie, couche par couche. Chaque neurone calcule sa sortie, qui sert d'entrée à la couche suivante. À la fin, la couche de sortie produit une prédiction $\hat{y}$. Cette prédiction est comparée à la vraie étiquette y pour calculer l'erreur totale du réseau via la fonction de coût.

2. **La Passe Arrière (Backward Pass)** : C'est ici que la "magie" opère. L'erreur calculée à la fin de la passe avant est propagée en sens inverse, de la couche de sortie vers la couche d'entrée. En utilisant la règle de dérivation en chaîne (chain rule) du calcul différentiel, l'algorithme calcule la contribution de chaque poids et de chaque biais à l'erreur totale. En substance, il calcule $\frac{\partial J}{\partial w}$ pour chaque poids $w$ du réseau.

Une fois ces gradients calculés, un algorithme d'optimisation comme la Descente de Gradient Stochastique (SGD) les utilise pour mettre à jour les poids et les biais, déplaçant le modèle dans la direction qui minimise l'erreur. Ce cycle de passe avant et passe arrière est répété de nombreuses fois sur l'ensemble des données d'entraînement.

La découverte et la popularisation de la rétropropagation ont été un tournant décisif. Avant cela, entraîner des réseaux avec plus d'une couche cachée était considéré comme infaisable. La rétropropagation est une application élégante et récursive de la règle de dérivation en chaîne qui évite les calculs redondants, la rendant suffisamment efficace pour entraîner des réseaux très profonds. Elle a transformé le domaine en déplaçant le défi de l'optimisation convexe des modèles simples vers la navigation dans le paysage de coût non-convexe et complexe des réseaux de neurones profonds, qui présente de multiples minima locaux.

#### Tableau : Pseudocode de l'algorithme de Rétropropagation

Ce pseudocode décrit le processus pour un seul exemple d'entraînement $(x,y)$.

| Étape | Description |
|-------|-------------|
| 1. Initialisation | Initialiser les poids $W$ et les biais $b$ de toutes les couches (généralement avec de petites valeurs aléatoires). |
| 2. Passe Avant | Pour chaque couche $l$ de 1 à $L$ (nombre de couches) : <br> - Calculer l'entrée pondérée $z^{[l]} = W^{[l]} a^{[l-1]} + b^{[l]}$. <br> - Calculer l'activation $a^{[l]} = g(z^{[l]})$, où $a^{[0]} = x$. <br> La prédiction finale est $\hat{y} = a^{[L]}$. |
| 3. Calcul de l'Erreur de Sortie | Calculer l'erreur (delta) à la couche de sortie $L$ : <br> $\delta^{[L]} = \nabla_a J \odot g'(z^{[L]})$, où $\odot$ est le produit élément par élément. |
| 4. Rétropropagation de l'Erreur | Pour chaque couche $l$ de $L-1$ à 1 : <br> Calculer l'erreur de la couche $l$ en fonction de l'erreur de la couche $l+1$ : <br> $\delta^{[l]} = ((W^{[l+1]})^T \delta^{[l+1]}) \odot g'(z^{[l]})$. |
| 5. Calcul des Gradients | Pour chaque couche $l$ de 1 à $L$ : <br> - Calculer le gradient pour les poids : $\frac{\partial J}{\partial W^{[l]}} = \delta^{[l]} (a^{[l-1]})^T$. <br> - Calculer le gradient pour les biais : $\frac{\partial J}{\partial b^{[l]}} = \delta^{[l]}$. |
| 6. Mise à jour (via un optimiseur) | Utiliser les gradients calculés pour mettre à jour les poids et les biais <br> (ex: $W^{[l]} := W^{[l]} - \alpha \frac{\partial J}{\partial W^{[l]}}$). |

#### Implémentation Python

Implémenter un réseau de neurones à partir de zéro avec NumPy est un excellent exercice pour comprendre les mécanismes internes.

```python
# Implémentation d'un réseau de neurones simple avec NumPy

import numpy as np

# Fonction d'activation sigmoïde et sa dérivée
def sigmoid(x):
    return 1 / (1 + np.exp(-x))

def sigmoid_derivative(x):
    return x * (1 - x)

# 1. Définition de l'architecture et initialisation des poids
input_neurons = 2
hidden_neurons = 4
output_neurons = 1

# Poids et biais initialisés aléatoirement
weights_hidden = np.random.uniform(size=(input_neurons, hidden_neurons))
bias_hidden = np.random.uniform(size=(1, hidden_neurons))
weights_output = np.random.uniform(size=(hidden_neurons, output_neurons))
bias_output = np.random.uniform(size=(1, output_neurons))

# Données d'entraînement (problème XOR)
X = np.array([[0, 0], [0, 1], [1, 0], [1, 1]])
y = np.array([[0], [1], [1], [0]])

# Paramètres d'entraînement
learning_rate = 0.1
epochs = 10000

# 2. Boucle d'entraînement
for _ in range(epochs):
    # --- Passe Avant ---
    # Couche cachée
    hidden_layer_input = np.dot(X, weights_hidden) + bias_hidden
    hidden_layer_activation = sigmoid(hidden_layer_input)
    
    # Couche de sortie
    output_layer_input = np.dot(hidden_layer_activation, weights_output) + bias_output
    predicted_output = sigmoid(output_layer_input)

    # --- Passe Arrière ---
    # Calcul de l'erreur
    error = y - predicted_output
    
    # Calcul des gradients (application de la règle de dérivation en chaîne)
    d_predicted_output = error * sigmoid_derivative(predicted_output)
    
    error_hidden_layer = d_predicted_output.dot(weights_output.T)
    d_hidden_layer = error_hidden_layer * sigmoid_derivative(hidden_layer_activation)
    
    # Mise à jour des poids et des biais
    weights_output += hidden_layer_activation.T.dot(d_predicted_output) * learning_rate
    bias_output += np.sum(d_predicted_output, axis=0, keepdims=True) * learning_rate
    weights_hidden += X.T.dot(d_hidden_layer) * learning_rate
    bias_hidden += np.sum(d_hidden_layer, axis=0, keepdims=True) * learning_rate

print("Sortie prédite après entraînement :")
print(predicted_output)
```

En pratique, on utilise des bibliothèques comme scikit-learn (pour les MLP simples) ou des frameworks de deep learning comme TensorFlow ou PyTorch.

```python
# Implémentation d'un MLP avec scikit-learn

from sklearn.neural_network import MLPClassifier
from sklearn.datasets import make_moons
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score

# Utilisons les mêmes données non-linéaires que pour le SVM
X, y = make_moons(n_samples=200, noise=0.2, random_state=42)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.3, random_state=42)

# Création du modèle MLP
# hidden_layer_sizes : définit l'architecture (ici, une couche cachée de 10 neurones)
# max_iter : nombre maximal d'époques d'entraînement
# activation : fonction d'activation (relu est la plus courante)
# solver : algorithme d'optimisation ('adam' est un choix robuste)
mlp = MLPClassifier(hidden_layer_sizes=(10,), max_iter=1000, activation='relu', solver='adam', random_state=1)

# Entraînement du modèle
mlp.fit(X_train, y_train)

# Prédictions et évaluation
y_pred = mlp.predict(X_test)
accuracy = accuracy_score(y_test, y_pred)
print(f"Précision du MLP : {accuracy:.2f}")

# Visualisation (en utilisant la même fonction plot_decision_boundary que précédemment)
plt.figure()
plot_decision_boundary(mlp, X, y, 'MLP Classifier Decision Boundary')
plt.show()
```

## Partie 3 : L'Apprentissage Non Supervisé – Découvrir les Structures Cachées

### Principes de base : Trouver des motifs dans des données non étiquetées

Nous abordons maintenant une catégorie d'apprentissage fondamentalement différente. Contrairement à l'apprentissage supervisé, l'apprentissage non supervisé opère sur des données qui n'ont pas d'étiquettes. Nous ne disposons que des entrées X, sans sorties y correspondantes. L'objectif n'est donc plus de prédire une valeur ou une classe spécifique, mais de découvrir des "structures intéressantes" ou des motifs cachés au sein des données elles-mêmes.

Les applications sont vastes et variées. Un exemple classique est celui de Google News, qui analyse des dizaines de milliers d'articles de presse chaque jour et les regroupe automatiquement en "clusters" traitant du même événement, sans que personne n'ait à lui dire de quoi parle chaque article. D'autres applications incluent la segmentation de marché (identifier des groupes de clients aux comportements similaires), l'analyse de réseaux sociaux (détecter des communautés), l'organisation de clusters informatiques, ou encore l'analyse de données génétiques pour regrouper des individus en fonction de leurs caractéristiques biologiques. L'apprentissage non supervisé est non seulement utile en soi, mais il peut aussi servir d'étape de prétraitement pour des tâches supervisées, par exemple en réduisant la dimensionnalité des données.

### Le Clustering : Regrouper les Données Similaires

#### Algorithme : K-Means

Le clustering est la tâche la plus courante en apprentissage non supervisé. Son but est de regrouper les points de données en un certain nombre de "clusters" ($k$) de telle sorte que les points d'un même cluster soient très similaires entre eux, et très différents des points des autres clusters. L'algorithme K-Means est l'un des plus simples et des plus populaires pour accomplir cette tâche.

##### Mécanisme

K-Means fonctionne de manière itérative en deux étapes simples, souvent comparées à un algorithme d'Attente-Maximisation (Expectation-Maximization, EM) :

1. **Étape d'Assignation (Assignment Step)** : Chaque point de données est assigné au cluster dont le centroïde (le centre du cluster) est le plus proche. La distance utilisée est généralement la distance euclidienne.

2. **Étape de Mise à jour (Update Step)** : Le centroïde de chaque cluster est recalculé en prenant la moyenne de tous les points de données qui lui ont été assignés à l'étape précédente.

Ce processus est répété jusqu'à ce que les assignations des points et les positions des centroïdes ne changent plus, c'est-à-dire jusqu'à convergence. Il est important de noter que K-Means est sensible à l'initialisation aléatoire des centroïdes et qu'il est garanti de converger vers un minimum local de la variance intra-cluster, mais pas nécessairement vers le minimum global. Il est donc courant de l'exécuter plusieurs fois avec des initialisations différentes.

##### Tableau : Pseudocode de l'algorithme K-Means

| Étape | Description |
|-------|-------------|
| 1. Initialisation | Choisir le nombre de clusters, $k$. Initialiser aléatoirement les positions des $k$ centroïdes, $\mu_1, \mu_2, ..., \mu_k$. |
| 2. Boucle de Convergence | Répéter jusqu'à ce que les assignations de cluster ne changent plus : |
| 2a. Étape d'Assignation | Pour chaque point de données $x^{(i)}$ : <br> - Calculer la distance entre $x^{(i)}$ et chaque centroïde $\mu_j$. <br> - Assigner $x^{(i)}$ au cluster $c^{(i)}$ du centroïde le plus proche : <br> $c^{(i)} := \arg\min_j \|x^{(i)} - \mu_j\|^2$ |
| 2b. Étape de Mise à jour | Pour chaque cluster $j$ de 1 à $k$ : <br> Recalculer le centroïde $\mu_j$ comme étant la moyenne de tous les points $x^{(i)}$ assignés au cluster $j$ : <br> $\mu_j := \frac{1}{\|S_j\|} \sum_{i \in S_j} x^{(i)}$ |
| 3. Résultat | Le modèle final est constitué des positions des $k$ centroïdes et de l'assignation de chaque point de données à un cluster. |

##### Implémentation Python

scikit-learn fournit une implémentation efficace de K-Means.

```python
# Implémentation de K-Means en Python

import numpy as np
import matplotlib.pyplot as plt
from sklearn.cluster import KMeans
from sklearn.datasets import make_blobs

# 1. Génération de données synthétiques
# make_blobs est parfait pour créer des groupes de points
X, y_true = make_blobs(n_samples=300, centers=4, cluster_std=0.80, random_state=0)

# 2. Création et entraînement du modèle K-Means
# n_clusters spécifie le nombre de clusters à trouver (k)
kmeans = KMeans(n_clusters=4, n_init=10, random_state=0)
kmeans.fit(X)

# 3. Récupération des résultats
# Les étiquettes de cluster pour chaque point sont dans.labels_
y_kmeans = kmeans.labels_
# Les coordonnées des centroïdes finaux sont dans.cluster_centers_
centers = kmeans.cluster_centers_

# 4. Visualisation des résultats
plt.figure(figsize=(8, 6))
# 'c=y_kmeans' colore chaque point selon le cluster auquel il a été assigné
plt.scatter(X[:, 0], X[:, 1], c=y_kmeans, s=50, cmap='viridis', label='Points de données')
# Affichage des centroïdes en noir
plt.scatter(centers[:, 0], centers[:, 1], c='black', s=200, alpha=0.75, marker='X', label='Centroïdes')
plt.title('Clustering K-Means (k=4)')
plt.xlabel('Feature 1')
plt.ylabel('Feature 2')
plt.legend()
plt.grid(True)
plt.show()
```

### La Séparation de Sources : Le Problème du Cocktail Party

#### Algorithme : Analyse en Composantes Indépendantes (ICA) et FastICA

Imaginons une situation plus complexe que le simple regroupement : le "problème du cocktail party". Vous êtes dans une pièce avec deux microphones et deux personnes qui parlent simultanément. Chaque microphone enregistre un mélange des deux voix. Est-il possible de récupérer les deux voix originales et séparées à partir de ces deux enregistrements mélangés? C'est un problème de Séparation Aveugle de Sources (Blind Source Separation), et l'Analyse en Composantes Indépendantes (ICA) est un algorithme puissant pour le résoudre.

##### Principe

L'hypothèse fondamentale de l'ICA est que les signaux observés (les enregistrements des microphones) sont des combinaisons linéaires de signaux sources sous-jacents qui sont statistiquement indépendants et non gaussiens. Le Théorème Central Limite nous dit qu'un mélange de variables indépendantes tend à être "plus gaussien" que les variables originales. L'ICA exploite ce principe à l'envers : il cherche à trouver une transformation linéaire (une "matrice de dé-mélange") des signaux observés qui maximise la non-gaussianité des signaux résultants, récupérant ainsi les sources originales.

FastICA est un algorithme itératif particulièrement efficace et populaire pour réaliser cette optimisation.

##### Tableau : Pseudocode de l'algorithme FastICA (pour une seule composante)

| Étape | Description |
|-------|-------------|
| 1. Prétraitement | Centrer les données (soustraire la moyenne). <br> Blanchir les données (les transformer pour qu'elles aient une variance de 1 et soient décorrélées). |
| 2. Initialisation | Choisir un vecteur de poids initial aléatoire $w$ de norme 1. |
| 3. Boucle d'Itération | Répéter jusqu'à convergence : |
| 3a. Mise à jour du poids | Mettre à jour le vecteur de poids $w$ en utilisant la règle de point fixe : <br> $w^+ := E\{Xg(w^TX)\} - E\{g'(w^TX)\}w$, <br> où $g$ est une fonction non-quadratique (ex: $g(u) = \tanh(u)$ ou $g(u) = u^3$). |
| 3b. Normalisation | Normaliser le vecteur de poids : <br> $w := w^+ / \|w^+\|$ |
| 4. Résultat | Le vecteur $w$ convergent définit une composante indépendante. <br> Pour trouver plusieurs composantes, le processus est répété en s'assurant que les nouveaux vecteurs sont orthogonaux aux précédents (déflation). |

##### Implémentation Python

scikit-learn fournit l'algorithme FastICA. Nous allons simuler le problème du cocktail party.

```python
# Implémentation de FastICA en Python

import numpy as np
import matplotlib.pyplot as plt
from sklearn.decomposition import FastICA

# 1. Génération des signaux sources indépendants
np.random.seed(0)
n_samples = 2000
time = np.linspace(0, 8, n_samples)

s1 = np.sin(2 * time)  # Signal sinusoïdal
s2 = np.sign(np.sin(3 * time))  # Signal carré
S = np.c_[s1, s2]
S += 0.2 * np.random.normal(size=S.shape)  # Ajout d'un peu de bruit
S /= S.std(axis=0)  # Standardisation

# 2. Mélange des signaux
# Matrice de mélange aléatoire
A = np.array([[1, 1], [0.5, 2]])
X = np.dot(S, A.T)  # Signaux observés/mélangés

# 3. Application de FastICA pour récupérer les sources
ica = FastICA(n_components=2, random_state=0, whiten='unit-variance')
S_recovered = ica.fit_transform(X)  # Récupération des sources estimées

# 4. Visualisation
plt.figure(figsize=(12, 8))

# Signaux sources originaux
plt.subplot(3, 1, 1)
plt.title("Signaux Sources Originaux")
plt.plot(time, S)
plt.grid(True)

# Signaux mélangés (observés)
plt.subplot(3, 1, 2)
plt.title("Signaux Mélangés (Observés par les microphones)")
plt.plot(time, X)
plt.grid(True)

# Signaux récupérés par FastICA
plt.subplot(3, 1, 3)
plt.title("Signaux Récupérés par FastICA")
plt.plot(time, S_recovered)
plt.grid(True)

plt.tight_layout()
plt.show()
```

Le choix entre des algorithmes comme K-Means et ICA illustre une idée profonde : tout modèle d'apprentissage non supervisé repose sur une hypothèse fondamentale concernant la structure inhérente des données. K-Means suppose que la structure est définie par la proximité euclidienne, ce qui fonctionne bien pour des clusters sphériques. ICA, en revanche, suppose que la structure provient d'une superposition linéaire de sources statistiquement indépendantes. Un praticien efficace ne se contente pas de connaître les algorithmes ; il comprend les hypothèses sous-jacentes de chacun et choisit celui dont la "vision du monde" correspond le mieux à la nature du problème à résoudre.

## Partie 4 : L'Apprentissage par Renforcement – Apprendre par l'Action

### Le paradigme Agent-Environnement

Nous entrons maintenant dans un paradigme d'apprentissage radicalement différent des deux précédents. L'apprentissage par renforcement (Reinforcement Learning, RL) ne s'appuie pas sur un jeu de données statique, qu'il soit étiqueté ou non. Il s'agit d'apprendre par l'interaction. Le cadre du RL met en scène un agent (le preneur de décision) qui évolue dans un environnement. À chaque instant, l'agent observe l'état de l'environnement, choisit une action, et en retour, l'environnement lui fournit une récompense (un signal positif ou négatif) et le place dans un nouvel état.

L'intuition derrière ce processus est très humaine. C'est ainsi que l'on dresse un animal : lorsque le chien exécute correctement un ordre, on lui donne une friandise ("good dog") ; sinon, on ne lui donne rien ou on exprime son mécontentement ("bad dog"). De même, pour apprendre à un hélicoptère autonome à voler, on le récompense lorsqu'il effectue des manœuvres stables et on le pénalise lorsqu'il s'écrase. L'objectif de l'agent est d'apprendre, par essais et erreurs, une stratégie qui maximise la somme des récompenses qu'il reçoit sur le long terme.

### Définitions formelles : État, Action, Récompense, Politique

Ce processus d'interaction est formalisé par le cadre mathématique du Processus de Décision Markovien (MDP). Un MDP est défini par les éléments suivants :

- **Agent** : L'entité qui apprend et prend des décisions.
- **Environnement** : Le monde extérieur avec lequel l'agent interagit.
- **État (S)** : Une description complète de l'environnement à un instant $t$. Par exemple, la position d'un robot sur une carte, ou la configuration des pièces sur un échiquier.
- **Action (A)** : L'ensemble des choix possibles pour l'agent dans un état donné. Par exemple, se déplacer vers le nord, le sud, l'est ou l'ouest.
- **Récompense (R)** : Un signal scalaire que l'environnement envoie à l'agent après qu'il a effectué une action $A_t$ dans un état $S_t$ et est passé à l'état $S_{t+1}$. La récompense quantifie la désirabilité immédiate de cette action.
- **Politique (π)** : C'est le "cerveau" de l'agent. Une politique est une stratégie qui dicte quelle action prendre dans un état donné. Formellement, $\pi(a|s)$ est la probabilité que l'agent choisisse l'action $a$ lorsqu'il se trouve dans l'état $s$.

Le but de l'agent est d'apprendre la politique optimale $(\pi^*)$, celle qui maximise le gain (ou retour), c'est-à-dire la somme des récompenses futures, souvent actualisées par un facteur $\gamma$ pour donner plus de poids aux récompenses immédiates.

### Le dilemme Exploration vs. Exploitation

Un défi central et fondamental en RL est le dilemme entre l'exploration et l'exploitation. L'agent doit-il exploiter la connaissance qu'il a déjà acquise en choisissant l'action qu'il sait être la meilleure jusqu'à présent? Ou doit-il explorer en essayant de nouvelles actions, potentiellement sous-optimales à court terme, dans l'espoir de découvrir des stratégies encore meilleures et d'obtenir des récompenses plus élevées à l'avenir? Un agent qui n'explore jamais risque de rester coincé dans un optimum local. Un agent qui n'exploite jamais n'utilisera jamais ses connaissances pour obtenir de bonnes récompenses. Trouver le bon équilibre est crucial pour un apprentissage efficace.

### Introduction au Q-Learning

Le Q-Learning est l'un des algorithmes de RL les plus connus et les plus fondamentaux. C'est un algorithme sans modèle (model-free), ce qui signifie qu'il n'a pas besoin de connaître la dynamique de l'environnement (c'est-à-dire les probabilités de transition entre les états). Il apprend directement de l'expérience.

Le Q-Learning fonctionne en apprenant une fonction de valeur action, notée $Q(s,a)$. La valeur $Q(s,a)$ représente la "qualité" (d'où le "Q") de l'action $a$ prise dans l'état $s$. Plus précisément, c'est le retour total attendu (somme des récompenses futures actualisées) si l'on commence dans l'état $s$, que l'on prend l'action $a$, et que l'on suit ensuite la politique optimale. Une fois que l'agent a appris la fonction Q optimale, la politique optimale est simple : dans n'importe quel état $s$, il suffit de choisir l'action $a$ qui maximise $Q(s,a)$.

L'algorithme met à jour ses estimations des valeurs Q de manière itérative en utilisant l'équation de Bellman. Après avoir effectué une action $A_t$ dans l'état $S_t$ et reçu une récompense $R_{t+1}$ et un nouvel état $S_{t+1}$, l'agent met à jour $Q(S_t, A_t)$ en se basant sur la récompense immédiate et la meilleure valeur Q possible depuis le nouvel état.

#### Tableau : Pseudocode de l'algorithme Q-Learning

| Étape | Description |
|-------|-------------|
| 1. Initialisation | Initialiser une table Q (la Q-table) avec des valeurs arbitraires (souvent zéro) pour toutes les paires état-action $(s,a)$. <br> Choisir les hyperparamètres : taux d'apprentissage $\alpha$, facteur d'actualisation $\gamma$, probabilité d'exploration $\epsilon$. |
| 2. Boucle d'Épisodes | Répéter pour un certain nombre d'épisodes : |
| 2a. Initialisation d'épisode | Initialiser l'état de départ $S$. |
| 2b. Boucle de Pas de Temps | Répéter (pour chaque pas de l'épisode) : |
| 2b-i. Choix de l'Action | Choisir une action $A$ dans l'état $S$ en utilisant une politique dérivée de Q <br> (par exemple, la politique $\epsilon$-greedy : avec une probabilité $\epsilon$, choisir une action aléatoire (exploration) ; <br> sinon, choisir l'action $A$ qui maximise $Q(S,a)$ (exploitation)). |
| 2b-ii. Interaction | Effectuer l'action $A$, observer la récompense $R$ et le nouvel état $S'$. |
| 2b-iii. Mise à jour de la Q-table | Mettre à jour la valeur Q pour la paire $(S,A)$ en utilisant l'équation de Bellman : <br> $Q(S,A) \leftarrow Q(S,A) + \alpha [R + \gamma \max_{a'} Q(S',a') - Q(S,A)]$. |
| 2b-iv. Transition | Mettre à jour l'état : $S \leftarrow S'$. |
| 2c. Fin de l'épisode | Continuer jusqu'à ce que $S$ soit un état terminal. |

L'apprentissage par renforcement formalise ainsi le processus intuitif d'apprentissage par essais et erreurs. Les concepts de récompense, de politique et de valeur sont les analogues mathématiques de la motivation, de la stratégie et du jugement. La fonction $Q(s,a)$ peut être vue comme une formalisation de l' "intuition" de l'agent sur la qualité d'une action dans un contexte donné. L'équation de Bellman fournit un mécanisme de "bootstrap" pour que cette intuition devienne cohérente et rationnelle au fil du temps, en mettant à jour les estimations actuelles avec des estimations futures. Cela fait du RL un cadre de calcul puissant pour l'apprentissage orienté vers un but dans des environnements incertains, expliquant son succès dans des domaines comme la robotique, les jeux et le contrôle complexe, où les données supervisées sont souvent inexistantes.

#### Exemple conceptuel d'implémentation en Python

Voici un squelette d'implémentation pour un problème simple de "grid world", où un agent doit naviguer sur une grille pour atteindre un objectif.

```python
# Exemple conceptuel de Q-Learning avec NumPy

import numpy as np

# 1. Définition de l'environnement (Grid World 4x4)
# 16 états (0-15), 4 actions (0:Haut, 1:Bas, 2:Gauche, 3:Droite)
num_states = 16
num_actions = 4

# 2. Initialisation de la Q-table
q_table = np.zeros((num_states, num_actions))

# 3. Hyperparamètres
learning_rate = 0.1  # alpha
discount_factor = 0.99 # gamma
epsilon = 1.0        # Taux d'exploration initial
max_epsilon = 1.0
min_epsilon = 0.01
decay_rate = 0.001   # Taux de décroissance de l'epsilon

# 4. Boucle d'entraînement
num_episodes = 10000

# Fonction pour choisir une action (politique epsilon-greedy)
def choose_action(state, epsilon):
    if np.random.uniform(0, 1) < epsilon:
        return np.random.choice(num_actions) # Exploration
    else:
        return np.argmax(q_table[state, :]) # Exploitation

# Simulation de l'environnement (très simplifiée)
def get_next_state_and_reward(state, action):
    # Logique simplifiée : l'état 15 est le but (+10 récompense)
    # L'état 5 est un trou (-10 récompense)
    if state == 15: return 15, 0, True # Déjà au but
    if state == 5: return 5, 0, True # Déjà dans le trou
    
    # Simuler le mouvement (sans gérer les bords pour la simplicité)
    if action == 0: next_state = state - 4 # Haut
    elif action == 1: next_state = state + 4 # Bas
    elif action == 2: next_state = state - 1 # Gauche
    else: next_state = state + 1 # Droite

    # Gérer les bords (rester sur place si on tape un mur)
    if next_state < 0 or next_state > 15: next_state = state
    
    if next_state == 15:
        reward = 10
        done = True
    elif next_state == 5:
        reward = -10
        done = True
    else:
        reward = -0.1 # Petite pénalité pour chaque mouvement pour encourager la rapidité
        done = False
    return next_state, reward, done

for episode in range(num_episodes):
    state = 0 # Toujours commencer à l'état 0
    done = False
    
    while not done:
        action = choose_action(state, epsilon)
        
        next_state, reward, done = get_next_state_and_reward(state, action)
        
        # Mise à jour de la Q-table avec l'équation de Bellman
        old_value = q_table[state, action]
        next_max = np.max(q_table[next_state, :])
        
        new_value = old_value + learning_rate * (reward + discount_factor * next_max - old_value)
        q_table[state, action] = new_value
        
        state = next_state
        
    # Mise à jour de l'epsilon
    epsilon = min_epsilon + (max_epsilon - min_epsilon) * np.exp(-decay_rate * episode)

print("Q-table entraînée :")
print(q_table)
```

## Partie 5 : Stratégie et Méthodologie en Machine Learning – L'Art de l'Ingénieur

### De la "Magie Noire" à la Discipline d'Ingénierie Systématique

Avoir une connaissance approfondie des algorithmes est nécessaire, mais pas suffisant pour construire des systèmes de machine learning performants. Le succès dans ce domaine dépend autant de la stratégie que de la connaissance technique. Trop souvent, des équipes, même dans les meilleures entreprises technologiques, passent des mois à travailler sur une approche qui était vouée à l'échec dès le départ, simplement parce qu'elles n'avaient pas une méthode systématique pour diagnostiquer et orienter leur projet. L'objectif est de transformer la pratique du machine learning, qui peut parfois ressembler à de la "magie noire" ou à un savoir tribal basé sur l'expérience, en une discipline d'ingénierie systématique et rigoureuse.

L'analogie avec le débogage de code est pertinente. Un développeur inexpérimenté pourrait essayer de corriger des bugs au hasard, tandis qu'un ingénieur expérimenté utilisera un profileur pour identifier précisément le goulot d'étranglement avant d'optimiser. De la même manière, lorsque votre premier modèle de ML ne fonctionne pas comme prévu (ce qui est presque toujours le cas), il faut disposer d'outils de diagnostic pour comprendre pourquoi il échoue et savoir quelles actions correctives prendre.

### Diagnostiquer un modèle : Le compromis Biais-Variance

L'outil de diagnostic le plus fondamental en apprentissage supervisé est le compromis biais-variance. Ces deux sources d'erreur nous aident à comprendre le comportement de notre modèle :

- **Le Biais (Bias)** : C'est l'erreur due à des hypothèses erronées dans le modèle d'apprentissage. Un modèle à fort biais est trop simple pour capturer la complexité des données. C'est le problème du sous-apprentissage (underfitting). Symptôme typique : le modèle a de mauvaises performances à la fois sur les données d'entraînement et sur les données de test.

- **La Variance** : C'est l'erreur due à la sensibilité excessive du modèle aux petites fluctuations dans les données d'entraînement. Un modèle à forte variance est trop complexe et "mémorise" le bruit des données d'entraînement au lieu d'apprendre le signal sous-jacent. C'est le problème du sur-apprentissage (overfitting). Symptôme typique : le modèle a d'excellentes performances sur les données d'entraînement, mais de très mauvaises performances sur des données nouvelles (de test).

En comparant l'erreur sur l'ensemble d'entraînement et l'erreur sur un ensemble de développement (ou de test), on peut diagnostiquer la nature du problème de notre modèle.

### Approches stratégiques pour l'amélioration des modèles

Une fois le diagnostic posé, il existe un ensemble d'actions stratégiques que l'on peut entreprendre. Ces choix ne doivent pas être faits au hasard, mais en réponse directe au problème identifié.

**Si le diagnostic est un BIAIS ÉLEVÉ (sous-apprentissage) :**

- **Essayer un modèle plus complexe** : Ajouter des couches ou des neurones dans un réseau, utiliser un noyau plus puissant dans un SVM (passer de linéaire à RBF), ajouter des termes polynomiaux dans une régression.
- **Ajouter de nouvelles features** : L'ingénierie des caractéristiques (feature engineering) peut fournir au modèle plus d'informations pour apprendre.
- **Entraîner plus longtemps** : S'assurer que l'algorithme d'optimisation a bien convergé.
- **Réduire la régularisation** : La régularisation est conçue pour combattre le sur-apprentissage ; si le problème est le sous-apprentissage, il faut peut-être en réduire l'intensité.

**Si le diagnostic est une VARIANCE ÉLEVÉE (sur-apprentissage) :**

- **Obtenir plus de données d'entraînement** : C'est souvent la solution la plus efficace. Un modèle complexe a plus de mal à sur-apprendre sur un grand jeu de données.
- **Augmenter la régularisation** : Utiliser la régularisation L1 ou L2 (paramètre penalty dans LogisticRegression, paramètre C dans SVC) pour pénaliser les poids élevés et forcer le modèle à être plus simple.
- **Essayer un modèle plus simple** : Réduire le nombre de couches/neurones, utiliser un noyau moins complexe, ou moins de features.
- **Utiliser des techniques comme le Dropout** dans les réseaux de neurones.

Cette approche systématique est le "profiler" du praticien en machine learning. Elle permet de prendre des décisions éclairées et d'itérer beaucoup plus rapidement vers un modèle performant, évitant ainsi de perdre des mois dans une direction non prometteuse.

## Conclusion : Synthèse et Perspectives

Ce cours a parcouru les trois grands paradigmes du machine learning : l'apprentissage supervisé, où l'on apprend à partir de données étiquetées pour faire des prédictions ; l'apprentissage non supervisé, où l'on découvre des structures cachées dans des données non étiquetées ; et l'apprentissage par renforcement, où un agent apprend à agir dans un environnement par essais et erreurs.

Nous avons vu que la maîtrise de ce domaine ne se limite pas à la connaissance d'une collection d'algorithmes. Elle réside dans la compréhension profonde des principes qui les sous-tendent : la nature des données, les hypothèses des modèles, les mécanismes d'optimisation, et surtout, la stratégie d'ingénierie nécessaire pour passer de la théorie à une application fonctionnelle et performante. La structure même de ce cours, progressant des modèles simples aux modèles complexes et se terminant par la stratégie, est conçue pour vous équiper de cette double compétence.

En fin de compte, le machine learning est plus qu'un ensemble d'outils techniques ; c'est un levier puissant pour l'innovation et le progrès. En devenant un expert, vous acquérez la capacité non seulement de construire des produits et des services remarquables, mais aussi de vous attaquer à des problèmes significatifs et de contribuer, espérons-le, à rendre le monde un peu meilleur.
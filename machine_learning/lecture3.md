# Des Modèles Locaux à la Classification Probabiliste : Régression Pondérée, Régression Logistique et Méthodes d'Optimisation Avancées

## Introduction : Au-delà de la Régression Linéaire Standard

Cette leçon a pour objectif de poursuivre notre exploration de l'apprentissage supervisé, en nous aventurant au-delà des fondations établies par la régression linéaire simple. Nous allons construire sur ces concepts pour aborder des problèmes plus complexes et développer des algorithmes plus sophistiqués et flexibles.

### Rappel des Concepts Fondamentaux

Lors de notre précédente discussion, nous avons disséqué l'algorithme de régression linéaire. Nous avons formalisé l'hypothèse comme une fonction linéaire des caractéristiques d'entrée, $h_\theta(x) = \theta^T x$, où $x$ est le vecteur de caractéristiques (augmenté d'une composante $x_0 = 1$ pour le terme de biais) et $\theta$ est le vecteur des paramètres du modèle. L'objectif est de trouver les paramètres $\theta$ qui ajustent au mieux notre modèle aux données d'entraînement.

Pour quantifier la qualité de cet ajustement, nous avons introduit une fonction de coût, ou fonction de perte, que nous cherchons à minimiser. Le choix standard pour la régression linéaire est la méthode des moindres carrés, qui consiste à minimiser la somme des erreurs quadratiques entre les prédictions et les valeurs réelles. La fonction de coût, notée $J(\theta)$, est définie comme suit :

$$J(\theta) = \frac{1}{2m} \sum_{i=1}^{m} (h_\theta(x^{(i)}) - y^{(i)})^2$$

où $m$ est le nombre d'exemples dans l'ensemble d'entraînement.

Pour trouver les valeurs de $\theta$ qui minimisent cette fonction, nous avons exploré deux approches principales :

1. **La Descente de Gradient** : Un algorithme itératif qui ajuste progressivement les paramètres $\theta$ dans la direction opposée au gradient de la fonction de coût, convergeant ainsi vers un minimum local (qui est également le minimum global pour la fonction de coût convexe de la régression linéaire).

2. **Les Équations Normales** : Une solution analytique de forme fermée qui fournit la valeur optimale de $\theta$ en une seule étape, en résolvant l'équation $\theta = (X^T X)^{-1} X^T y$.

### Problématique et Objectifs de la Leçon

Le modèle de régression linéaire, malgré sa simplicité et son élégance, repose sur une hypothèse forte : la relation entre les caractéristiques et la variable cible est linéaire. Cependant, dans de nombreux problèmes du monde réel, cette hypothèse n'est pas vérifiée. Les données peuvent présenter des courbures, des tendances non linéaires qui ne peuvent être capturées par une simple ligne droite.

Face à cette limitation, cette leçon se fixe quatre objectifs principaux pour étendre notre boîte à outils en apprentissage automatique :

1. **Modéliser des fonctions non-linéaires avec la Régression Pondérée Localement (LWR)** : Nous explorerons une méthode astucieuse, la LWR, qui permet d'ajuster des modèles très flexibles à des données non linéaires sans avoir à définir manuellement des caractéristiques complexes comme des termes polynomiaux.

2. **Justifier notre approche avec une interprétation probabiliste** : Nous nous demanderons pourquoi la fonction de coût des moindres carrés est un choix judicieux. En développant une interprétation probabiliste de la régression linéaire, nous fournirons une justification rigoureuse à cette fonction de coût et, plus important encore, nous établirons un cadre général pour dériver de nouveaux algorithmes.

3. **Développer notre premier algorithme de classification : la Régression Logistique** : En nous appuyant sur ce nouveau cadre probabiliste, nous aborderons une nouvelle classe de problèmes, la classification. Nous dériverons pas à pas la régression logistique, l'un des algorithmes de classification les plus fondamentaux et les plus utilisés.

4. **Accélérer l'optimisation avec la Méthode de Newton** : Enfin, nous introduirons une méthode d'optimisation du second ordre, la méthode de Newton, qui est souvent beaucoup plus rapide que la descente de gradient, et nous l'appliquerons pour trouver les paramètres de notre modèle de régression logistique.

Cette leçon est conçue comme une progression logique, où chaque concept s'appuie sur le précédent, nous menant des modèles de régression simples à des techniques de classification sophistiquées et des méthodes d'optimisation avancées.

## Partie 1 : Régression Pondérée Localement (Locally Weighted Regression - LWR)

Nous commençons notre exploration au-delà de la régression linéaire standard en nous attaquant directement à sa principale limitation : son incapacité à modéliser des relations non linéaires.

### 1.1. Les Limites des Modèles Globaux et l'Ingénierie des Caractéristiques

Imaginons un ensemble de données représentant le prix des maisons en fonction de leur superficie. Si nous traçons ces données, nous pourrions constater que la relation n'est pas une simple ligne droite. Peut-être que le prix augmente rapidement pour les petites surfaces, puis l'augmentation ralentit pour les très grandes maisons, formant une courbe. Un modèle de régression linéaire simple tracerait une ligne droite à travers ces points, ce qui entraînerait des erreurs de prédiction systématiques et importantes.

Une première approche pour résoudre ce problème est l'**ingénierie des caractéristiques** (feature engineering). Au lieu de se contenter d'un modèle de la forme $h_\theta(x) = \theta_0 + \theta_1 x_1$, nous pouvons créer de nouvelles caractéristiques en transformant les caractéristiques existantes. Par exemple, nous pourrions ajouter un terme quadratique $(x_1^2)$ pour modéliser une parabole, ou d'autres fonctions comme la racine carrée $(\sqrt{x_1})$ ou le logarithme $(\log(x_1))$ pour capturer différentes formes de courbure. Le modèle deviendrait, par exemple, $h_\theta(x) = \theta_0 + \theta_1 x_1 + \theta_2 x_1^2$.

Bien que puissante, cette approche présente un inconvénient majeur : elle est manuelle et subjective. Le choix des bonnes caractéristiques à ajouter dépend d'une inspection visuelle des données ou d'une connaissance approfondie du domaine. Pour des données complexes avec de nombreuses caractéristiques, il devient presque impossible de deviner les bonnes transformations à appliquer. La régression pondérée localement (LWR) est une alternative élégante qui contourne ce problème en adoptant une approche radicalement différente.

### 1.2. Algorithmes Paramétriques vs. Non-Paramétriques : Une Distinction Fondamentale

Pour bien comprendre la LWR, il est essentiel d'introduire une distinction fondamentale en apprentissage automatique : la différence entre les algorithmes paramétriques et non-paramétriques.

#### Algorithmes Paramétriques

Un algorithme d'apprentissage est dit **paramétrique** s'il fait des hypothèses fortes sur la forme de la fonction de prédiction et résume les connaissances acquises à partir des données dans un ensemble de paramètres de taille fixe. La régression linéaire est l'exemple canonique d'un algorithme paramétrique. Le modèle est défini par les paramètres $\theta = (\theta_0, \theta_1, ..., \theta_n)$. Quelle que soit la taille de l'ensemble d'entraînement (qu'il contienne 100 ou 1 million d'exemples), le nombre de paramètres à apprendre reste le même. Une fois que l'algorithme a appris les valeurs optimales de $\theta$, l'ensemble de données d'entraînement n'est plus nécessaire pour faire des prédictions. On peut l'effacer de la mémoire de l'ordinateur ; seules les valeurs de $\theta$ sont conservées.

#### Algorithmes Non-Paramétriques

À l'opposé, un algorithme **non-paramétrique** fait très peu d'hypothèses sur la forme de la fonction de prédiction. Le terme "non-paramétrique" est quelque peu trompeur ; il ne signifie pas qu'il n'y a pas de paramètres, mais plutôt que le nombre de paramètres (ou la quantité d'informations à stocker) croît avec la taille de l'ensemble de données d'entraînement.

La régression pondérée localement est notre premier exemple d'algorithme non-paramétrique. Comme nous le verrons, pour faire une seule prédiction, la LWR doit considérer l'ensemble des données d'entraînement. Cela signifie que l'ensemble de données complet doit être conservé en mémoire ou sur disque, même après la phase "d'apprentissage".

Cette distinction n'est pas seulement une question de taxonomie académique ; elle représente un compromis fondamental en apprentissage automatique. Les modèles paramétriques sont généralement rapides et efficaces en mémoire lors de la prédiction, mais leur performance est plafonnée par la flexibilité du modèle choisi (une ligne, un polynôme, etc.). S'ils font une mauvaise hypothèse sur la forme des données, ils ne pourront jamais bien s'ajuster. À l'inverse, les modèles non-paramétriques sont extrêmement flexibles et peuvent s'adapter à des structures de données très complexes sans faire d'hypothèses fortes. Cette flexibilité a cependant un coût : un coût de calcul et de stockage élevé, car chaque prédiction peut nécessiter un calcul impliquant l'ensemble des données d'entraînement. Le choix entre ces deux familles d'algorithmes dépend donc des contraintes du projet : pour des ensembles de données massifs ou des applications nécessitant des prédictions à faible latence, un modèle paramétrique est souvent privilégié. Pour des ensembles de données plus petits où la relation sous-jacente est inconnue et potentiellement complexe, un modèle non-paramétrique comme la LWR peut être une excellente solution.

### 1.3. L'Intuition de la LWR : Apprendre "Localement"

L'idée centrale de la LWR est simple et intuitive. Au lieu d'essayer d'ajuster un unique modèle global à l'ensemble des données, la LWR ajuste un modèle différent pour chaque point de prédiction.

Le processus est le suivant :

1. **Choisir un point de requête** : Supposons que nous voulions faire une prédiction pour une nouvelle valeur d'entrée $x$.

2. **Se concentrer sur le voisinage** : L'algorithme regarde les points de l'ensemble d'entraînement qui sont "proches" de $x$ sur l'axe des abscisses.

3. **Ajuster un modèle local** : L'algorithme accorde une grande importance (un poids élevé) aux points proches de $x$ et une très faible importance (un poids faible) aux points éloignés. Il ajuste ensuite une simple ligne droite en utilisant ces poids. Essentiellement, il effectue une régression linéaire qui se soucie principalement des données locales.

4. **Faire la prédiction** : La prédiction pour le point $x$ est alors donnée par la valeur de cette ligne droite locale au point $x$.

Ce processus est répété pour chaque nouveau point pour lequel une prédiction est requise. Si nous voulons prédire la valeur pour un autre point $x'$, l'algorithme se concentrera sur un nouveau voisinage autour de $x'$, calculera de nouveaux poids, ajustera une nouvelle ligne droite locale, et fera une nouvelle prédiction. Le résultat est une courbe de prédiction qui peut être très complexe et non linéaire, car elle est constituée d'une multitude de petits segments de droite locaux.

### 1.4. Formalisation Mathématique de la LWR

Pour formaliser cette intuition, nous modifions la fonction de coût des moindres carrés de la régression linéaire.

#### La Fonction de Coût Modifiée

Dans la régression linéaire standard, chaque exemple d'entraînement contribue de manière égale à la fonction de coût. Dans la LWR, nous introduisons un terme de pondération $w^{(i)}$ pour chaque exemple $(x^{(i)}, y^{(i)})$. La nouvelle fonction de coût que nous cherchons à minimiser pour un point de requête $x$ donné est :

$$J(\theta) = \sum_{i=1}^{m} w^{(i)} (y^{(i)} - \theta^T x^{(i)})^2$$

Notez que nous minimisons cette fonction pour trouver un $\theta$ spécifique à la requête $x$. Ce $\theta$ sera différent pour chaque point de requête.

#### La Fonction de Pondération (Weighting Function)

La clé de l'algorithme réside dans la manière dont les poids $w^{(i)}$ sont définis. Le poids d'un exemple d'entraînement $x^{(i)}$ doit être élevé si $x^{(i)}$ est proche du point de requête $x$, et faible s'il en est éloigné. Un choix courant et efficace pour la fonction de pondération est le **noyau Gaussien** :

$$w^{(i)} = \exp\left(-\frac{(x^{(i)} - x)^2}{2\tau^2}\right)$$

Analysons cette formule :

- Le terme $(x^{(i)} - x)^2$ est la distance euclidienne au carré entre le point d'entraînement $x^{(i)}$ et le point de requête $x$.
- Si $x^{(i)}$ est très proche de $x$, cette distance est proche de zéro. L'exposant est proche de zéro, et $w^{(i)} = \exp(0) = 1$. L'exemple reçoit donc un poids maximal.
- Si $x^{(i)}$ est très loin de $x$, la distance est grande. L'exposant devient un grand nombre négatif, et $w^{(i)}$ tend vers zéro. L'exemple ne contribue presque pas à la fonction de coût.

L'effet net de cette pondération est que la somme des erreurs carrées ne prend en compte de manière significative que les points d'entraînement situés dans un "voisinage" de $x$, dont la taille et l'influence sont définies par la forme de la courbe gaussienne.

#### Le Paramètre de Bande Passante (Bandwidth Parameter) $\tau$

Le paramètre $\tau$ (tau) dans la fonction de pondération est un hyperparamètre crucial appelé **paramètre de bande passante**. Il contrôle la largeur de la fonction de pondération en forme de cloche, et donc la taille du voisinage que l'algorithme considère comme "local".

- Un **petit $\tau$** crée une courbe gaussienne très étroite et pointue. Seuls les points d'entraînement extrêmement proches du point de requête reçoivent un poids significatif. Cela rend le modèle très sensible aux variations locales des données, ce qui peut conduire à une courbe de prédiction très "ondulée" ou "bruyante". Ce phénomène est associé à une variance élevée et à un risque de sur-ajustement (overfitting).

- Un **grand $\tau$** crée une courbe gaussienne très large et aplatie. De nombreux points, même ceux qui sont assez éloignés, reçoivent un poids non négligeable. Le modèle prend en compte une grande partie de l'ensemble de données pour chaque prédiction, et son comportement se rapproche de celui d'une régression linéaire globale. Cela peut conduire à un lissage excessif de la tendance et à l'incapacité de capturer les variations locales. Ce phénomène est associé à un biais élevé et à un risque de sous-ajustement (underfitting).

Le choix de $\tau$ est donc un exercice d'équilibrage du compromis biais-variance, un concept central en apprentissage automatique.

### 1.5. Pseudocode de l'Algorithme LWR

Contrairement à la descente de gradient, la minimisation de la fonction de coût pondérée $J(\theta)$ admet une solution analytique, similaire aux équations normales. C'est un point essentiel car il signifie que pour chaque prédiction, nous n'avons pas besoin d'un processus itératif ; nous pouvons calculer le $\theta$ local directement. La solution de forme fermée pour les moindres carrés pondérés est $\theta = (X^T W X)^{-1} X^T W y$, où $W$ est une matrice diagonale contenant les poids $w^{(i)}$.

Voici le pseudocode pour prédire la valeur d'un unique point de requête.

```
Fonction predict_LWR(X_train, y_train, x_query, tau):
    // Entrées :
    //   X_train: Matrice des caractéristiques d'entraînement (m x n)
    //   y_train: Vecteur des cibles d'entraînement (m x 1)
    //   x_query: Le point pour lequel on veut une prédiction (1 x n)
    //   tau: Le paramètre de bande passante

    // 1. Obtenir les dimensions
    m = nombre de lignes dans X_train

    // 2. Initialiser la matrice de poids W
    // W est une matrice diagonale de taille m x m
    W = matrice identité de taille m x m

    // 3. Calculer les poids pour chaque point d'entraînement
    Pour i de 1 à m:
        // Calculer la distance au carré
        distance_sq = (X_train[i] - x_query) * (X_train[i] - x_query)^T
        // Calculer le poids en utilisant le noyau Gaussien
        poids = exp( -distance_sq / (2 * tau^2) )
        // Placer le poids sur la diagonale de W
        W[i, i] = poids

    // 4. Préparer la matrice X_train pour le calcul
    // Ajouter une colonne de 1s pour le terme de biais
    X_b = ajouter_colonne_de_uns(X_train)

    // 5. Résoudre pour theta en utilisant la solution analytique
    // theta = (X_b^T * W * X_b)^-1 * X_b^T * W * y_train
    terme1 = inverse(X_b^T * W * X_b)
    terme2 = X_b^T * W * y_train
    theta = terme1 * terme2

    // 6. Préparer le point de requête pour la prédiction
    // Ajouter un 1 pour le terme de biais
    x_query_b = ajouter_un(x_query)

    // 7. Calculer la prédiction finale
    prediction = x_query_b^T * theta

    // 8. Retourner la prédiction
    Retourner prediction
```

Le goulot d'étranglement computationnel de cet algorithme est l'inversion de la matrice $(X_b^T W X_b)$, qui est de taille $(n+1) \times (n+1)$, où $n$ est le nombre de caractéristiques. La complexité de l'inversion de matrice est typiquement en $O(n^3)$. C'est la raison pour laquelle la LWR est principalement utilisée pour des problèmes avec un faible nombre de caractéristiques.

### 1.6. Implémentation Complète en Python avec NumPy

Mettons maintenant en pratique ces concepts avec une implémentation complète en Python. Nous allons d'abord générer un ensemble de données synthétiques non linéaire, puis nous implémenterons l'algorithme LWR pour ajuster une courbe à ces données. Nous visualiserons également l'impact du paramètre de bande passante $\tau$.

```python
import numpy as np
import matplotlib.pyplot as plt

# --- 1. Génération de données synthétiques non linéaires ---
# Nous créons des données qui suivent une tendance cubique avec du bruit gaussien.
# Un modèle linéaire simple échouerait à capturer cette relation.
np.random.seed(8)
m = 100
X = np.random.randn(m, 1) * 2.5
y = 2 * (X**3) - 1 * (X**2) + 10 + 6 * np.random.randn(m, 1)

# --- 2. Implémentation de la Régression Pondérée Localement (LWR) ---

def locally_weighted_regression(x_query, X_train, y_train, tau):
    """
    Effectue une prédiction pour un point x_query en utilisant la LWR.

    Args:
        x_query (np.array): Le point pour lequel faire une prédiction.
        X_train (np.array): Matrice des caractéristiques d'entraînement.
        y_train (np.array): Vecteur des cibles d'entraînement.
        tau (float): Le paramètre de bande passante.

    Returns:
        float: La valeur prédite pour x_query.
    """
    # Ajouter le terme de biais à X_train (une colonne de 1s)
    X_b = np.c_[np.ones((len(X_train), 1)), X_train]
    
    # Ajouter le terme de biais à x_query
    x_query_b = np.array([1, x_query])
    
    # Obtenir le nombre d'exemples d'entraînement
    m = X_train.shape[0]
    
    # Initialiser la matrice de poids W (m x m)
    W = np.mat(np.eye(m))
    
    # Calculer les poids pour chaque point d'entraînement
    for i in range(m):
        xi = X_b[i]
        diff = xi - x_query_b
        # Le poids est calculé en utilisant le noyau gaussien
        W[i, i] = np.exp(np.dot(diff, diff.T) / (-2 * tau * tau))
        
    # Calculer theta en utilisant la solution de forme fermée
    # theta = (X_b^T * W * X_b)^-1 * X_b^T * W * y_train
    # np.linalg.pinv est utilisé pour la stabilité numérique (pseudo-inverse)
    try:
        theta = np.linalg.pinv(X_b.T * (W * X_b)) * (X_b.T * (W * y_train))
    except np.linalg.LinAlgError:
        # Gérer les cas où la matrice est singulière
        return 0

    # Faire la prédiction
    prediction = np.dot(x_query_b, theta)
    
    return prediction.item()

# --- 3. Faire des prédictions et visualiser les résultats ---

def plot_lwr(tau):
    """
    Génère les prédictions pour un domaine de valeurs et trace le résultat.
    """
    # Créer un ensemble de points pour lesquels nous allons prédire
    domain = np.linspace(X.min(), X.max(), 100)
    predictions = []
    
    for point in domain:
        pred = locally_weighted_regression(np.array([point]), X, y, tau)
        predictions.append(pred)
        
    # Tracer les données originales et la courbe LWR
    plt.figure(figsize=(10, 6))
    plt.scatter(X, y, color='blue', alpha=0.5, label='Données originales')
    plt.plot(domain, predictions, color='red', linewidth=3, label=f'Ajustement LWR (tau={tau})')
    plt.title(f'Régression Pondérée Localement avec tau = {tau}')
    plt.xlabel('Caractéristique (X)')
    plt.ylabel('Cible (y)')
    plt.legend()
    plt.grid(True, linestyle='--', alpha=0.6)
    plt.show()

# Visualiser l'effet de différents tau
plot_lwr(tau=0.1)  # Petit tau -> sur-ajustement
plot_lwr(tau=0.5)  # Tau moyen -> bon ajustement
plot_lwr(tau=5.0)  # Grand tau -> sous-ajustement (proche de la régression linéaire)
```

L'exécution de ce code démontre visuellement les concepts que nous avons discutés. Avec `tau=0.1`, la courbe rouge suit de très près les fluctuations locales du bruit, illustrant le sur-ajustement. Avec `tau=5.0`, la courbe est presque une ligne droite, lissant excessivement les données et manquant la tendance cubique sous-jacente, ce qui est caractéristique du sous-ajustement. Une valeur intermédiaire comme `tau=0.5` semble capturer la structure non linéaire des données de manière beaucoup plus efficace.

Pour conclure cette section, le tableau suivant résume les caractéristiques clés des approches paramétriques et non-paramétriques, en utilisant la régression linéaire et la LWR comme exemples représentatifs.

### Table 1 : Comparaison des Approches Paramétriques et Non-Paramétriques

| Caractéristique | Algorithme Paramétrique (ex: Régression Linéaire) | Algorithme Non-Paramétrique (ex: LWR) |
|---|---|---|
| Hypothèses sur les données | Forte (ex: relation linéaire) | Faible (aucune forme fonctionnelle spécifique n'est supposée) |
| Nombre de paramètres | Fixe, indépendant de la taille des données (le vecteur $\theta$) | Croît avec la taille des données (l'ensemble des données est nécessaire) |
| Données à conserver | Uniquement les paramètres $\theta$ après l'entraînement | L'ensemble des données d'entraînement complet |
| Vitesse de prédiction | Très rapide (un simple produit scalaire $\theta^T x$) | Lente (recalcul complet pour chaque nouveau point de requête) |
| Risque de sous-ajustement | Élevé si l'hypothèse de linéarité est fausse | Faible, car le modèle est très flexible |
| Risque de sur-ajustement | Faible | Élevé si le paramètre de bande passante $\tau$ est trop petit |
| Cas d'utilisation typique | Données de grande dimension, besoin de prédictions rapides, relation simple connue ou approximée | Données de faible dimension, relation complexe inconnue, ensemble de données de taille modérée |

## Partie 2 : Une Perspective Probabiliste sur la Régression Linéaire

Après avoir exploré une méthode pour gérer la non-linéarité, nous revenons à la régression linéaire standard pour répondre à une question plus fondamentale. Cette investigation nous fournira un cadre théorique puissant qui nous servira de tremplin pour la régression logistique et d'autres algorithmes.

### 2.1. La Question Fondamentale : Pourquoi l'Erreur Quadratique?

Jusqu'à présent, nous avons accepté la fonction de coût des moindres carrés, $J(\theta) = \frac{1}{2m} \sum (h_\theta(x^{(i)}) - y^{(i)})^2$, comme une mesure raisonnable de l'erreur. Elle pénalise fortement les grandes erreurs et est mathématiquement pratique car elle est convexe et différentiable. Mais ce choix est-il arbitraire? Pourquoi minimiser la somme des carrés des erreurs, et non la somme des valeurs absolues des erreurs, ou la somme des erreurs à la puissance quatre?

Il s'avère qu'il existe une justification très solide à l'utilisation des moindres carrés, qui découle d'un ensemble d'hypothèses probabilistes sur la nature des données. En adoptant une perspective statistique, nous allons montrer que l'algorithme des moindres carrés n'est pas un choix arbitraire, mais une conséquence directe du principe du maximum de vraisemblance.

### 2.2. Le Modèle Statistique : Hypothèses sur les Erreurs

Pour construire notre interprétation probabiliste, nous devons formuler des hypothèses sur le processus de génération des données.

#### Hypothèse 1 : Relation Linéaire avec Terme d'Erreur

Nous supposons que la variable cible $y^{(i)}$ est liée aux caractéristiques $x^{(i)}$ par une fonction linéaire, mais qu'elle est également affectée par un terme d'erreur ou de bruit, $\epsilon^{(i)}$. Ce terme d'erreur représente les effets que nous n'avons pas modélisés (par exemple, des caractéristiques non incluses, des facteurs aléatoires). L'équation du modèle est donc :

$$y^{(i)} = \theta^T x^{(i)} + \epsilon^{(i)}$$

#### Hypothèse 2 : Erreurs Gaussiennes et IID

C'est l'hypothèse la plus cruciale. Nous supposons que les termes d'erreur $\epsilon^{(i)}$ sont des variables aléatoires **Indépendantes et Identiquement Distribuées (IID)** selon une distribution Gaussienne (ou Normale) de moyenne 0 et de variance $\sigma^2$.

$$\epsilon^{(i)} \sim \mathcal{N}(0, \sigma^2)$$

Décomposons cette hypothèse :

- **Gaussienne** : La distribution des erreurs suit la fameuse "courbe en cloche". Cela signifie que les petites erreurs (proches de la moyenne 0) sont très probables, tandis que les grandes erreurs sont très improbables. La densité de probabilité d'une erreur $\epsilon^{(i)}$ est donnée par :

$$p(\epsilon^{(i)}) = \frac{1}{\sigma\sqrt{2\pi}} \exp\left(-\frac{(\epsilon^{(i)})^2}{2\sigma^2}\right)$$

- **IID (Indépendantes et Identiquement Distribuées)** :
  - **Indépendantes** : L'erreur associée à un exemple d'entraînement, $\epsilon^{(i)}$, est indépendante de l'erreur associée à tout autre exemple, $\epsilon^{(j)}$.
  - **Identiquement Distribuées** : Toutes les erreurs $\epsilon^{(i)}$ sont tirées de la même distribution Gaussienne, avec la même moyenne (0) et la même variance ($\sigma^2$).

La justification de l'hypothèse Gaussienne repose souvent sur le **Théorème Central Limite (TCL)**. Si le terme d'erreur global est en réalité la somme d'un grand nombre de petits facteurs de bruit indépendants (par exemple, pour le prix d'une maison : l'humeur du vendeur, la météo le jour de la vente, une rénovation mineure non répertoriée), alors leur somme tendra à suivre une distribution Gaussienne.

Il est important de reconnaître que ces hypothèses, en particulier l'indépendance, sont souvent des simplifications de la réalité. Par exemple, les prix des maisons dans un même quartier sont probablement corrélés en raison de facteurs locaux partagés, ce qui viole l'hypothèse IID. Néanmoins, même si elles ne sont pas parfaitement vraies, ces hypothèses conduisent à un algorithme (les moindres carrés) qui fonctionne remarquablement bien dans de nombreuses situations et fournit un cadre théorique puissant. Si nous savons que ces hypothèses sont fortement violées (par exemple, si le bruit suit une distribution très différente ou si les données sont fortement corrélées comme dans les séries temporelles), cela nous indique que des modèles plus sophistiqués sont nécessaires.

### 2.3. Le Principe du Maximum de Vraisemblance (Maximum Likelihood Estimation - MLE)

Maintenant que nous avons un modèle probabiliste pour nos données, comment estimons-nous les paramètres $\theta$? Nous utilisons le principe du maximum de vraisemblance (MLE).

L'idée est la suivante : nous avons observé un certain ensemble de données d'entraînement. Nous voulons trouver les paramètres de modèle $\theta$ qui rendent ces données observées les plus probables. La fonction qui nous donne cette probabilité des données en fonction des paramètres est appelée la **fonction de vraisemblance**, notée $L(\theta)$.

$$L(\theta) = p(Y|X;\theta)$$

Ici, $Y$ est l'ensemble de toutes les cibles $y^{(i)}$ et $X$ est l'ensemble de toutes les caractéristiques $x^{(i)}$. Le point-virgule dans $p(Y|X;\theta)$ est une convention pour indiquer que $\theta$ n'est pas une variable aléatoire, mais un paramètre qui définit la distribution. L'objectif du MLE est donc de trouver les paramètres $\theta$ qui maximisent cette fonction :

$$\theta_{MLE} = \arg\max_{\theta} L(\theta)$$

### 2.4. Dérivation : de la Vraisemblance aux Moindres Carrés

Appliquons maintenant ce principe à notre modèle de régression linéaire.

#### Étape 1 : Écrire la distribution de $y^{(i)}$

De notre hypothèse $y^{(i)} = \theta^T x^{(i)} + \epsilon^{(i)}$ et $\epsilon^{(i)} \sim \mathcal{N}(0, \sigma^2)$, nous pouvons déduire la distribution de $y^{(i)}$ conditionnellement à $x^{(i)}$ et $\theta$. Puisque $\theta^T x^{(i)}$ est une constante pour un $x^{(i)}$ et un $\theta$ donnés, ajouter une variable aléatoire Gaussienne de moyenne 0 à cette constante décale simplement la moyenne de la distribution. Ainsi, $y^{(i)}$ suit également une distribution Gaussienne :

$$y^{(i)} | x^{(i)}; \theta \sim \mathcal{N}(\theta^T x^{(i)}, \sigma^2)$$

La densité de probabilité de $y^{(i)}$ est donc :

$$p(y^{(i)} | x^{(i)}; \theta) = \frac{1}{\sigma\sqrt{2\pi}} \exp\left(-\frac{(y^{(i)} - \theta^T x^{(i)})^2}{2\sigma^2}\right)$$

#### Étape 2 : Écrire la vraisemblance de l'ensemble des données

Grâce à l'hypothèse d'indépendance (la partie "I" de IID), la probabilité conjointe de tous les $y^{(i)}$ est simplement le produit de leurs probabilités individuelles. La fonction de vraisemblance $L(\theta)$ est donc :

$$L(\theta) = p(Y|X;\theta) = \prod_{i=1}^{m} p(y^{(i)}|x^{(i)};\theta)$$

$$= \prod_{i=1}^{m} \frac{1}{\sigma\sqrt{2\pi}} \exp\left(-\frac{(y^{(i)} - \theta^T x^{(i)})^2}{2\sigma^2}\right)$$

#### Étape 3 : Calculer la log-vraisemblance

Maximiser $L(\theta)$ peut être mathématiquement complexe en raison du produit. Cependant, comme la fonction logarithme est strictement croissante, maximiser une fonction est équivalent à maximiser son logarithme. Nous travaillons donc avec la log-vraisemblance, notée $\ell(\theta) = \log L(\theta)$, ce qui transforme le produit en une somme, bien plus facile à manipuler.

$$\ell(\theta) = \log\left(\prod_{i=1}^{m} \frac{1}{\sigma\sqrt{2\pi}} \exp\left(-\frac{(y^{(i)} - \theta^T x^{(i)})^2}{2\sigma^2}\right)\right)$$

En utilisant les propriétés du logarithme ($\log(ab) = \log a + \log b$ et $\log(e^x) = x$) :

$$\ell(\theta) = \sum_{i=1}^{m} \left[\log\left(\frac{1}{\sigma\sqrt{2\pi}}\right) - \frac{(y^{(i)} - \theta^T x^{(i)})^2}{2\sigma^2}\right]$$

En séparant les termes :

$$\ell(\theta) = m \log\left(\frac{1}{\sigma\sqrt{2\pi}}\right) - \frac{1}{2\sigma^2} \sum_{i=1}^{m} (y^{(i)} - \theta^T x^{(i)})^2$$

#### Étape 4 : Maximiser la log-vraisemblance

Notre objectif est de trouver le $\theta$ qui maximise $\ell(\theta)$. Examinons l'expression de $\ell(\theta)$ :

- Le premier terme, $m \log\left(\frac{1}{\sigma\sqrt{2\pi}}\right)$, ne dépend pas de $\theta$. C'est une constante par rapport à notre variable d'optimisation.
- Le second terme, $-\frac{1}{2\sigma^2} \sum_{i=1}^{m} (y^{(i)} - \theta^T x^{(i)})^2$, contient $\theta$.

Pour maximiser $\ell(\theta)$, nous pouvons ignorer le premier terme et nous concentrer sur la maximisation du second. Maximiser une quantité négative est équivalent à minimiser son opposé positif. De plus, le facteur constant $\frac{1}{2\sigma^2}$ n'affecte pas l'emplacement du minimum. Par conséquent, maximiser $\ell(\theta)$ est équivalent à :

$$\min_{\theta} \sum_{i=1}^{m} (y^{(i)} - \theta^T x^{(i)})^2$$

En ajoutant le facteur de convenance $\frac{1}{2m}$, nous retrouvons exactement notre fonction de coût des moindres carrés originale :

$$\min_{\theta} \frac{1}{2m} \sum_{i=1}^{m} (y^{(i)} - \theta^T x^{(i)})^2 = \min_{\theta} J(\theta)$$

### 2.5. Conclusion et Implications : Un Cadre Unificateur

Cette dérivation est un résultat fondamental. Elle nous montre que l'algorithme des moindres carrés est équivalent à l'estimation par maximum de vraisemblance pour les paramètres $\theta$ d'un modèle de régression linéaire, sous l'hypothèse que les termes d'erreur sont indépendants et suivent une distribution Gaussienne de moyenne nulle. L'erreur quadratique n'est donc pas un choix arbitraire, mais le choix qui découle naturellement de l'hypothèse d'un bruit Gaussien.

Plus important encore, la démarche que nous venons de suivre est une **recette**, un plan directeur pour construire de nouveaux algorithmes d'apprentissage. Les étapes sont :

1. Faire une hypothèse sur la distribution de probabilité de la variable de sortie $y$ conditionnellement à l'entrée $x$ (par exemple, $y|x \sim \mathcal{N}(\mu, \sigma^2)$).
2. Définir un modèle qui relie les entrées $x$ aux paramètres de cette distribution (par exemple, $\mu = \theta^T x$).
3. Construire la fonction de log-vraisemblance de l'ensemble des données.
4. Trouver les paramètres du modèle qui maximisent cette fonction.

Ce cadre est la base de la famille des **Modèles Linéaires Généralisés (GLM)**. Si notre sortie est un nombre de comptage (entier non négatif), nous pourrions supposer une distribution de Poisson. Si notre sortie est binaire (0 ou 1), nous pouvons supposer une distribution de Bernoulli. Cette perspective probabiliste unifie de nombreux algorithmes d'apparence différente et nous servira de guide pour développer notre premier classifieur : la régression logistique.

## Partie 3 : La Régression Logistique : Modéliser la Probabilité pour Classifier

Armés du puissant cadre du maximum de vraisemblance, nous sommes maintenant prêts à aborder une nouvelle classe de problèmes : la classification. Nous allons utiliser la même "recette" que précédemment, mais en changeant nos hypothèses pour les adapter à une sortie binaire.

### 3.1. Introduction à la Classification Binaire

Dans les problèmes de classification, la variable cible $y$ que nous essayons de prédire est discrète. Le cas le plus simple est la classification binaire, où $y$ ne peut prendre que deux valeurs, que nous noterons par convention 0 et 1. Par exemple :

- Un email est-il un "spam" (1) ou non (0)?
- Une transaction par carte de crédit est-elle "frauduleuse" (1) ou "légitime" (0)?
- Une tumeur est-elle "maligne" (1) ou "bénigne" (0)?

Notre objectif est de construire un modèle qui, étant donné un vecteur de caractéristiques $x$, prédit l'étiquette de classe correspondante.

### 3.2. Pourquoi la Régression Linéaire est Inadaptée à la Classification

Une première idée pourrait être d'utiliser la régression linéaire pour la classification. On pourrait décider, par exemple, que si la sortie prédite $h_\theta(x)$ est supérieure à 0.5, on prédit la classe 1, et sinon, on prédit la classe 0. Cependant, cette approche est profondément défectueuse pour plusieurs raisons.

Premièrement, sur le plan conceptuel, les sorties de la régression linéaire ne sont pas contraintes. Le modèle peut prédire des valeurs bien supérieures à 1 ou bien inférieures à 0. Si nous voulons interpréter la sortie comme une probabilité, cela n'a aucun sens.

Deuxièmement, et c'est plus grave, l'algorithme des moindres carrés est très sensible aux données. Considérons un problème de classification où les points de la classe 0 sont à gauche et ceux de la classe 1 sont à droite. Un modèle de régression linéaire pourrait trouver une droite qui sépare raisonnablement les données. Maintenant, ajoutons un seul point de données très à droite, qui est clairement de classe 1. Intuitivement, ce point ne devrait pas changer notre règle de décision. Cependant, pour minimiser la somme des erreurs quadratiques, la régression linéaire va "pivoter" sa droite d'ajustement pour se rapprocher de ce nouveau point. Ce faisant, elle peut déplacer radicalement le seuil de décision (l'endroit où la ligne croise 0.5), conduisant à un classifieur de bien moins bonne qualité. La régression linéaire tente de bien ajuster les données sur toute leur plage, alors que pour la classification, nous nous soucions uniquement de la frontière de décision.

### 3.3. Construction du Modèle Logistique : La Fonction Sigmoïde

Pour construire un meilleur algorithme de classification, nous avons besoin d'une hypothèse $h_\theta(x)$ dont la sortie est toujours comprise entre 0 et 1, que nous pourrons interpréter comme une probabilité. Pour ce faire, nous introduisons une nouvelle fonction : la **fonction sigmoïde**, également appelée fonction logistique. Elle est définie comme suit :

$$g(z) = \frac{1}{1 + e^{-z}}$$

Cette fonction a une forme caractéristique en "S". Quel que soit l'argument d'entrée $z$ (de $-\infty$ à $+\infty$), la sortie $g(z)$ est "écrasée" dans l'intervalle $(0, 1)$.

- Si $z \to \infty$, $e^{-z} \to 0$, et $g(z) \to 1$.
- Si $z \to -\infty$, $e^{-z} \to \infty$, et $g(z) \to 0$.
- Si $z = 0$, $e^0 = 1$, et $g(z) = 1/2$.

L'hypothèse de la régression logistique est alors définie en appliquant la fonction sigmoïde à notre prédicteur linéaire standard $\theta^T x$ :

$$h_\theta(x) = g(\theta^T x) = \frac{1}{1 + e^{-\theta^T x}}$$

Cette forme garantit que notre sortie sera toujours une valeur entre 0 et 1.

### 3.4. Le Modèle Probabiliste et la Fonction de Log-Vraisemblance

Nous appliquons maintenant le cadre du maximum de vraisemblance.

#### Hypothèses Probabilistes

Pour un problème de classification binaire où $y \in \{0, 1\}$, la distribution la plus naturelle pour la sortie est la distribution de Bernoulli. Nous faisons les hypothèses suivantes :

1. Nous interprétons la sortie de notre hypothèse comme la probabilité que la classe soit 1 :
   $$P(y = 1 | x; \theta) = h_\theta(x)$$

2. Puisque la somme des probabilités doit être égale à 1, la probabilité que la classe soit 0 est :
   $$P(y = 0 | x; \theta) = 1 - h_\theta(x)$$

#### Écriture Compacte

Comme $y$ ne peut prendre que les valeurs 0 ou 1, nous pouvons combiner ces deux équations en une seule expression compacte, ce qui simplifiera les dérivations ultérieures :

$$P(y | x; \theta) = (h_\theta(x))^y (1 - h_\theta(x))^{1-y}$$

Vérifions que cela fonctionne :
- Si $y = 1$, l'expression devient $(h_\theta(x))^1 (1 - h_\theta(x))^{1-1} = h_\theta(x)$, ce qui est correct.
- Si $y = 0$, l'expression devient $(h_\theta(x))^0 (1 - h_\theta(x))^{1-0} = 1 - h_\theta(x)$, ce qui est également correct.

#### Log-Vraisemblance

En supposant que nos $m$ exemples d'entraînement sont IID, la vraisemblance de l'ensemble des données est le produit des probabilités individuelles :

$$L(\theta) = p(Y|X;\theta) = \prod_{i=1}^{m} p(y^{(i)}|x^{(i)};\theta)$$

$$= \prod_{i=1}^{m} (h_\theta(x^{(i)}))^{y^{(i)}} (1 - h_\theta(x^{(i)}))^{1-y^{(i)}}$$

Comme précédemment, nous travaillons avec la log-vraisemblance $\ell(\theta) = \log L(\theta)$ pour transformer le produit en somme :

$$\ell(\theta) = \log\left(\prod_{i=1}^{m} (h_\theta(x^{(i)}))^{y^{(i)}} (1 - h_\theta(x^{(i)}))^{1-y^{(i)}}\right)$$

$$\ell(\theta) = \sum_{i=1}^{m} \left[\log\left((h_\theta(x^{(i)}))^{y^{(i)}}\right) + \log\left((1 - h_\theta(x^{(i)}))^{1-y^{(i)}}\right)\right]$$

$$\ell(\theta) = \sum_{i=1}^{m} \left[y^{(i)} \log h_\theta(x^{(i)}) + (1 - y^{(i)}) \log(1 - h_\theta(x^{(i)}))\right]$$

Cette fonction est la log-vraisemblance pour la régression logistique. En apprentissage profond, elle est plus connue sous le nom de **fonction de perte d'entropie croisée binaire** (binary cross-entropy). Notre objectif est de trouver les paramètres $\theta$ qui maximisent cette fonction.

### 3.5. Optimisation par Montée de Gradient (Gradient Ascent)

Pour maximiser $\ell(\theta)$, nous utilisons un algorithme d'optimisation. Contrairement à la régression linéaire, il n'existe pas de solution analytique de forme fermée pour la régression logistique. Nous devons donc utiliser une méthode itérative.

L'algorithme que nous allons utiliser est la **montée de gradient** (gradient ascent). Il est conceptuellement identique à la descente de gradient, mais au lieu de se déplacer dans la direction opposée au gradient pour trouver un minimum, nous nous déplaçons dans la direction du gradient pour trouver un maximum. La règle de mise à jour a donc un signe + au lieu d'un signe -.

La règle de mise à jour pour chaque paramètre $\theta_j$ est :

$$\theta_j := \theta_j + \alpha \frac{\partial}{\partial \theta_j} \ell(\theta)$$

où $\alpha$ est le taux d'apprentissage. La prochaine étape consiste à calculer le gradient de la log-vraisemblance, $\nabla_\theta \ell(\theta)$. La dérivation complète, qui implique la règle de la chaîne et la dérivée de la fonction sigmoïde, est un exercice de calcul différentiel instructif. Le résultat final est remarquablement simple :

$$\frac{\partial}{\partial \theta_j} \ell(\theta) = \sum_{i=1}^{m} (y^{(i)} - h_\theta(x^{(i)})) x_j^{(i)}$$

En insérant ce gradient dans notre règle de mise à jour, nous obtenons l'algorithme de montée de gradient pour la régression logistique :

```
Répéter {
    θ_j := θ_j + α ∑_{i=1}^{m} (y^{(i)} - h_θ(x^{(i)})) x_j^{(i)}    (pour chaque j)
}
```

Une observation fascinante s'impose : cette règle de mise à jour est identique en apparence à la règle de la descente de gradient pour la régression linéaire. Comment est-ce possible, alors que les deux modèles sont si différents? La réponse, comme le souligne la transcription, réside dans la définition de l'hypothèse $h_\theta(x)$.

- Pour la régression linéaire, $h_\theta(x) = \theta^T x$.
- Pour la régression logistique, $h_\theta(x) = \frac{1}{1 + e^{-\theta^T x}}$.

Cette similitude n'est pas une coïncidence. C'est une propriété profonde et élégante de la famille des **Modèles Linéaires Généralisés (GLM)**. Elle révèle une structure unifiée sous-jacente à de nombreux algorithmes d'apprentissage supervisé. Un même algorithme d'optimisation peut être appliqué à une large classe de problèmes, simplement en changeant la distribution de probabilité supposée (Gaussienne, Bernoulli, etc.) et la fonction de lien correspondante (identité, sigmoïde, etc.).

De plus, la fonction de log-vraisemblance pour la régression logistique est convexe (ou plus précisément, sa négation est convexe, ce qui la rend concave). Cela signifie qu'elle n'a pas de minima locaux (ou maxima locaux dans notre cas) ; l'algorithme de montée de gradient est donc garanti de converger vers le maximum global.

### 3.6. Pseudocode de la Régression Logistique avec Montée de Gradient

```
Fonction train_logistic_regression(X, y, alpha, num_iterations):
    // Entrées :
    //   X: Matrice des caractéristiques (m x (n+1), avec biais)
    //   y: Vecteur des cibles (m x 1)
    //   alpha: Taux d'apprentissage
    //   num_iterations: Nombre d'itérations

    // 1. Obtenir les dimensions
    m, n = dimensions de X

    // 2. Initialiser le vecteur de poids theta
    // Un vecteur de zéros de taille n
    theta = vecteur_zeros(n, 1)

    // 3. Boucle d'optimisation
    Répéter pour k de 1 à num_iterations:
        // a. Calculer le prédicteur linéaire pour tous les exemples
        z = X * theta  // Produit matriciel

        // b. Calculer les prédictions (probabilités) en appliquant la sigmoïde
        h = sigmoid(z)

        // c. Calculer le gradient de la log-vraisemblance
        // gradient est un vecteur de taille n
        gradient = (1/m) * X^T * (h - y) // Note: on utilise h-y car on maximise, et on normalise par m

        // d. Mettre à jour les poids
        // Note: la formule de la transcription est une montée de gradient stochastique/batch.
        // Pour la montée de gradient batch, la mise à jour est :
        theta = theta - alpha * gradient // On minimise la log-loss (-log-vraisemblance), d'où le signe moins

    // 4. Retourner les poids optimisés
    Retourner theta
```

### 3.7. Implémentation en Python "from scratch"

Nous allons maintenant implémenter l'algorithme de régression logistique en utilisant la descente de gradient (qui minimise la perte d'entropie croisée, équivalente à maximiser la log-vraisemblance).

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn.datasets import make_classification
from sklearn.model_selection import train_test_split

# --- 1. Définition des fonctions auxiliaires ---

def sigmoid(z):
    """Calcule la fonction sigmoïde."""
    return 1 / (1 + np.exp(-z))

def compute_cost(X, y, theta):
    """Calcule la fonction de coût (entropie croisée binaire)."""
    m = len(y)
    h = sigmoid(X @ theta)
    # Ajout d'un petit epsilon pour éviter log(0)
    epsilon = 1e-5
    cost = (1/m) * np.sum(-y * np.log(h + epsilon) - (1 - y) * np.log(1 - h + epsilon))
    return cost

def predict(X, theta, threshold=0.5):
    """Fait des prédictions de classe (0 ou 1)."""
    probabilities = sigmoid(X @ theta)
    return (probabilities >= threshold).astype(int)

# --- 2. Implémentation de la descente de gradient ---

def gradient_descent(X, y, theta, alpha, num_iterations):
    """
    Optimise theta en utilisant la descente de gradient batch.
    """
    m = len(y)
    cost_history = []

    for i in range(num_iterations):
        h = sigmoid(X @ theta)
        gradient = (1/m) * (X.T @ (h - y))
        theta = theta - alpha * gradient
        
        cost = compute_cost(X, y, theta)
        cost_history.append(cost)
        
    return theta, cost_history

# --- 3. Test sur des données synthétiques ---

# Générer un jeu de données linéairement séparable
X, y = make_classification(n_samples=200, n_features=2, n_redundant=0, 
                           n_informative=2, random_state=1, n_clusters_per_class=1)
y = y.reshape(-1, 1) # Assurer que y est un vecteur colonne

# Ajouter le terme de biais à X
X_b = np.c_[np.ones((len(X), 1)), X]

# Initialiser les paramètres
alpha = 0.1
num_iterations = 1000
initial_theta = np.zeros((X_b.shape[1], 1))

# Entraîner le modèle
theta_optimal, cost_history = gradient_descent(X_b, y, initial_theta, alpha, num_iterations)

print("Paramètres optimaux (theta):")
print(theta_optimal)

# --- 4. Visualisation ---

# Tracer la convergence de la fonction de coût
plt.figure(figsize=(12, 5))
plt.subplot(1, 2, 1)
plt.plot(range(num_iterations), cost_history)
plt.title("Convergence de la fonction de coût")
plt.xlabel("Nombre d'itérations")
plt.ylabel("Coût (Entropie Croisée)")
plt.grid(True)

# Tracer la frontière de décision
plt.subplot(1, 2, 2)
plt.scatter(X[:, 0], X[:, 1], c=y.flatten(), cmap='viridis', edgecolors='k')
x1_values = np.array([np.min(X_b[:, 1]), np.max(X_b[:, 1])])
# La frontière de décision est la ligne où theta^T * x = 0
# theta_0 + theta_1*x1 + theta_2*x2 = 0  =>  x2 = (-theta_0 - theta_1*x1) / theta_2
x2_values = - (theta_optimal[0] + theta_optimal[1] * x1_values) / theta_optimal[2]
plt.plot(x1_values, x2_values, "r-", label="Frontière de décision")
plt.title("Frontière de décision de la Régression Logistique")
plt.xlabel("Caractéristique 1")
plt.ylabel("Caractéristique 2")
plt.legend()
plt.grid(True)
plt.tight_layout()
plt.show()
```

Ce code illustre l'ensemble du processus : définition du modèle, entraînement par descente de gradient, et visualisation de la frontière de décision linéaire que l'algorithme a apprise pour séparer les deux classes.

## Partie 4 : La Méthode de Newton : Une Approche d'Optimisation du Second Ordre

La descente (ou montée) de gradient est un algorithme d'optimisation du premier ordre fiable et largement utilisé. Cependant, sa convergence peut parfois être lente. Nous allons maintenant explorer une méthode plus sophistiquée, la méthode de Newton, qui utilise des informations du second ordre pour converger beaucoup plus rapidement dans de nombreuses situations.

### 4.1. Les Limites de la Montée de Gradient

L'algorithme de montée de gradient progresse en faisant de "petites étapes" dans la direction de la plus forte pente. Le taux d'apprentissage $\alpha$ contrôle la taille de ces étapes. Si $\alpha$ est trop petit, la convergence est très lente ; s'il est trop grand, l'algorithme peut osciller ou même diverger. Trouver le bon $\alpha$ peut nécessiter des ajustements. De plus, même avec un bon $\alpha$, la descente de gradient peut nécessiter des centaines ou des milliers d'itérations pour atteindre l'optimum. La méthode de Newton offre une alternative qui ne nécessite pas de taux d'apprentissage et qui peut converger en un nombre d'itérations radicalement plus faible.

### 4.2. L'Intuition Géométrique de la Méthode de Newton

La méthode de Newton est fondamentalement un algorithme pour trouver les racines d'une fonction, c'est-à-dire les points $\theta$ où $f(\theta) = 0$. Comment cela nous aide-t-il à maximiser notre fonction de log-vraisemblance $\ell(\theta)$?

Le maximum d'une fonction différentiable se trouve à un point où sa dérivée première est nulle. Notre objectif est donc de trouver le $\theta$ qui satisfait $\ell'(\theta) = 0$. Nous pouvons donc appliquer la méthode de Newton pour trouver la racine de la fonction $f(\theta) = \ell'(\theta)$.

L'intuition géométrique de la méthode est la suivante (pour une fonction d'une seule variable) :

1. **Point de départ** : On commence avec une estimation initiale de la racine, $\theta_t$.
2. **Approximation par la tangente** : On calcule la ligne tangente à la fonction $f$ au point $(\theta_t, f(\theta_t))$. Cette tangente est la meilleure approximation linéaire de la fonction $f$ autour de $\theta_t$.
3. **Trouver la racine de la tangente** : On trouve le point où cette ligne tangente croise l'axe des abscisses (c'est-à-dire où la valeur de la tangente est zéro). Ce point devient notre nouvelle et meilleure estimation de la racine, $\theta_{t+1}$.
4. **Itérer** : On répète ce processus à partir de $\theta_{t+1}$.

Comme on peut le voir sur les schémas, chaque itération nous rapproche de manière spectaculaire de la vraie racine. Au lieu de suivre la courbure de la fonction, on "saute" directement vers l'endroit où l'approximation linéaire prédit que la racine se trouve.

### 4.3. Formalisation et Généralisation avec la Matrice Hessienne

#### Cas Unidimensionnel (1D)

Mathématiquement, la ligne tangente à $f$ en $\theta_t$ a pour équation $y = f(\theta_t) + f'(\theta_t)(\theta - \theta_t)$. Nous cherchons le $\theta$ (que nous appellerons $\theta_{t+1}$) pour lequel $y = 0$.

$$0 = f(\theta_t) + f'(\theta_t)(\theta_{t+1} - \theta_t)$$

En résolvant pour $\theta_{t+1}$, on obtient la règle de mise à jour de Newton :

$$\theta_{t+1} = \theta_t - \frac{f(\theta_t)}{f'(\theta_t)}$$

Pour notre problème d'optimisation, nous posons $f(\theta) = \ell'(\theta)$. La règle de mise à jour devient alors :

$$\theta_{t+1} = \theta_t - \frac{\ell'(\theta_t)}{\ell''(\theta_t)}$$

#### Cas Multidimensionnel (N-D)

Lorsque $\theta$ est un vecteur de paramètres, les concepts de dérivée première et seconde se généralisent :

- La dérivée première est remplacée par le **vecteur gradient**, $\nabla_\theta \ell(\theta)$, un vecteur contenant les dérivées partielles par rapport à chaque $\theta_j$.
- La dérivée seconde est remplacée par la **matrice Hessienne**, $H$, une matrice carrée contenant les dérivées partielles secondes. L'élément $(i,j)$ de la Hessienne est $H_{ij} = \frac{\partial^2 \ell(\theta)}{\partial \theta_i \partial \theta_j}$.

La division par la dérivée seconde est remplacée par la multiplication par l'inverse de la matrice Hessienne. La règle de mise à jour de Newton pour un vecteur de paramètres $\theta$ est :

$$\theta_{t+1} = \theta_t - H^{-1} \nabla_\theta \ell(\theta)$$

### 4.4. Application à la Régression Logistique : Dérivation du Gradient et de la Hessienne

Pour appliquer la méthode de Newton à la régression logistique, nous devons calculer le gradient et la Hessienne de la fonction de log-vraisemblance $\ell(\theta)$. Cette dérivation est technique mais cruciale pour l'implémentation.

**Gradient** (déjà calculé) :

$$\nabla_\theta \ell(\theta) = \sum_{i=1}^{m} (y^{(i)} - h_\theta(x^{(i)})) x^{(i)} = X^T(y - h)$$

(Note : dans la montée de gradient, nous utilisions $y - h$. Ici, nous maximisons $\ell(\theta)$, donc le gradient est le même, mais la mise à jour de Newton a un signe moins, donc nous minimisons $-\ell(\theta)$, ce qui change le signe du gradient à $h - y$). Pour rester cohérent avec la maximisation, la mise à jour est $\theta_{t+1} = \theta_t - H^{-1}(-\nabla_\theta \ell(\theta)) = \theta_t + H^{-1}\nabla_\theta \ell(\theta)$ où $H$ est la Hessienne de $-\ell(\theta)$.

**Hessienne de la log-vraisemblance négative** ($-\ell(\theta)$) :

Le calcul de la dérivée seconde du gradient aboutit à l'expression suivante pour la matrice Hessienne de la fonction de perte (log-vraisemblance négative) :

$$H = \frac{\partial^2(-\ell(\theta))}{\partial \theta \partial \theta^T} = \sum_{i=1}^{m} h_\theta(x^{(i)})(1 - h_\theta(x^{(i)})) x^{(i)} (x^{(i)})^T$$

En notation matricielle, cela peut s'écrire de manière compacte :

$$H = X^T W X$$

où $X$ est la matrice de design (m x (n+1)) et $W$ est une matrice diagonale de taille $m \times m$ dont les éléments diagonaux sont $W_{ii} = h_\theta(x^{(i)})(1 - h_\theta(x^{(i)}))$.

La règle de mise à jour de Newton pour la régression logistique (minimisation de $-\ell(\theta)$) devient donc :

$$\theta_{t+1} = \theta_t - (X^T W X)^{-1} X^T(h - y)$$

Cette méthode est également connue sous le nom de **Iteratively Reweighted Least Squares (IRLS)**, ou moindres carrés pondérés itérativement, car à chaque étape, on résout un problème de moindres carrés pondérés où les poids dans la matrice $W$ sont mis à jour en fonction de la prédiction actuelle.

### 4.5. Pseudocode de la Méthode de Newton pour la Régression Logistique

```
Fonction train_newton_method(X, y, num_iterations):
    // Entrées :
    //   X: Matrice des caractéristiques (m x (n+1), avec biais)
    //   y: Vecteur des cibles (m x 1)
    //   num_iterations: Nombre d'itérations

    // 1. Obtenir les dimensions
    m, n = dimensions de X

    // 2. Initialiser le vecteur de poids theta
    theta = vecteur_zeros(n, 1)

    // 3. Boucle d'optimisation
    Répéter pour k de 1 à num_iterations:
        // a. Calculer le prédicteur linéaire
        z = X * theta

        // b. Calculer les prédictions (probabilités)
        h = sigmoid(z)

        // c. Calculer le gradient de la log-vraisemblance négative
        gradient = X^T * (h - y)

        // d. Construire la matrice de poids diagonale W
        // W est une matrice diagonale m x m
        W = matrice_diagonale( h * (1 - h) ) // où * est une multiplication élément par élément

        // e. Calculer la Hessienne
        H = X^T * W * X

        // f. Mettre à jour les poids
        // delta_theta = inverse(H) * gradient
        // theta = theta - delta_theta
        theta = theta - inverse(H) * gradient

    // 4. Retourner les poids optimisés
    Retourner theta
```

### 4.6. Implémentation en Python de la Méthode de Newton

L'implémentation de la méthode de Newton est plus complexe que celle de la descente de gradient car elle nécessite le calcul et l'inversion de la matrice Hessienne à chaque étape.

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn.datasets import make_classification

# --- Fonctions auxiliaires (sigmoid, predict) de l'exemple précédent ---
def sigmoid(z):
    return 1 / (1 + np.exp(-z))

def predict(X, theta, threshold=0.5):
    probabilities = sigmoid(X @ theta)
    return (probabilities >= threshold).astype(int)

# --- 1. Implémentation de la méthode de Newton ---

def newton_method(X, y, num_iterations):
    """
    Optimise theta en utilisant la méthode de Newton (IRLS).
    """
    m, n = X.shape
    theta = np.zeros((n, 1))
    cost_history = []

    for i in range(num_iterations):
        z = X @ theta
        h = sigmoid(z)
        
        # Calcul du gradient
        gradient = X.T @ (h - y)
        
        # Calcul de la matrice de poids W
        # h.flatten() pour s'assurer que les dimensions sont correctes
        W = np.diag((h * (1 - h)).flatten())
        
        # Calcul de la Hessienne
        H = X.T @ W @ X
        
        # Mise à jour de theta
        # On ajoute une petite régularisation (ridge) à la Hessienne pour la stabilité
        # en cas de singularité
        H_inv = np.linalg.pinv(H + 1e-5 * np.eye(n))
        theta = theta - H_inv @ gradient
        
        # Calcul du coût pour observer la convergence
        cost = compute_cost(X, y, theta) # Fonction de la section précédente
        cost_history.append(cost)
        
        # Condition d'arrêt précoce si le gradient est très petit
        if np.linalg.norm(gradient) < 1e-6:
            print(f"Convergence atteinte à l'itération {i+1}")
            break
            
    return theta, cost_history

# --- 2. Test sur des données synthétiques ---

# Utiliser les mêmes données que pour la descente de gradient
X, y = make_classification(n_samples=200, n_features=2, n_redundant=0, 
                           n_informative=2, random_state=1, n_clusters_per_class=1)
y = y.reshape(-1, 1)
X_b = np.c_[np.ones((len(X), 1)), X]

# Entraîner le modèle avec Newton. Notez le faible nombre d'itérations.
theta_newton, cost_history_newton = newton_method(X_b, y, num_iterations=15)

print("\nParamètres optimaux (Newton):")
print(theta_newton)

# --- 3. Visualisation ---

plt.figure(figsize=(12, 5))

# Tracer la convergence
plt.subplot(1, 2, 1)
plt.plot(range(len(cost_history_newton)), cost_history_newton)
plt.title("Convergence (Méthode de Newton)")
plt.xlabel("Nombre d'itérations")
plt.ylabel("Coût")
plt.grid(True)

# Tracer la frontière de décision
plt.subplot(1, 2, 2)
plt.scatter(X[:, 0], X[:, 1], c=y.flatten(), cmap='viridis', edgecolors='k')
x1_values = np.array([np.min(X_b[:, 1]), np.max(X_b[:, 1])])
x2_values_newton = - (theta_newton[0] + theta_newton[1] * x1_values) / theta_newton[2]
plt.plot(x1_values, x2_values_newton, "g-", label="Frontière de décision (Newton)")
plt.title("Frontière de décision (Newton)")
plt.xlabel("Caractéristique 1")
plt.ylabel("Caractéristique 2")
plt.legend()
plt.grid(True)
plt.tight_layout()
plt.show()
```

L'exécution de ce code montrera une convergence extrêmement rapide, souvent en moins de 10 itérations, ce qui contraste fortement avec les centaines d'itérations nécessaires à la descente de gradient pour atteindre une solution similaire.

### 4.7. Analyse Comparative : Vitesse de Convergence vs. Coût par Itération

La comparaison entre la montée de gradient et la méthode de Newton illustre un compromis fondamental en optimisation. Il ne s'agit pas de savoir quelle méthode est universellement "meilleure", mais plutôt de comprendre laquelle est la plus appropriée pour un problème donné.

**Vitesse de convergence** : La méthode de Newton bénéficie d'une propriété appelée **convergence quadratique**. De manière informelle, cela signifie que le nombre de chiffres significatifs corrects dans la solution double à peu près à chaque itération lorsque l'on est proche de l'optimum. C'est pourquoi elle converge si rapidement. La descente de gradient, en revanche, a une convergence linéaire, beaucoup plus lente.

**Coût par itération** : C'est là que la méthode de Newton paie le prix de sa vitesse. Chaque itération de la descente de gradient a un coût de calcul proportionnel à $O(m \cdot n)$, où $m$ est le nombre d'exemples et $n$ le nombre de caractéristiques. Pour la méthode de Newton, il faut calculer la matrice Hessienne $H$ (coût en $O(m \cdot n^2)$) puis l'inverser (coût en $O(n^3)$). Le coût total par itération est dominé par l'inversion et est donc beaucoup plus élevé, surtout lorsque le nombre de caractéristiques $n$ est grand.

Cette analyse mène à une règle empirique simple :

- Pour les problèmes avec un **faible nombre de caractéristiques** (par exemple, $n < 1000$), la méthode de Newton est souvent le meilleur choix. Le surcoût de chaque itération est compensé par le très faible nombre d'itérations nécessaires.
- Pour les problèmes à **très grande dimension** (par exemple, $n > 10,000$, comme c'est souvent le cas en traitement d'images ou en deep learning), le calcul et le stockage de la matrice Hessienne $n \times n$ deviennent infaisables. La descente de gradient (et ses variantes stochastiques) est alors la seule option viable.

Ce compromis a motivé le développement d'une famille d'algorithmes intermédiaires, appelés méthodes **Quasi-Newton** (comme L-BFGS), qui tentent d'approximer l'inverse de la Hessienne sans jamais la calculer explicitement, offrant ainsi un excellent équilibre entre la vitesse de convergence et le coût par itération.

Le tableau suivant résume cette comparaison.

### Table 2 : Comparaison des Méthodes d'Optimisation : Montée de Gradient vs. Méthode de Newton

| Caractéristique | Montée de Gradient (1er Ordre) | Méthode de Newton (2nd Ordre) |
|---|---|---|
| Information utilisée | Dérivée première (Gradient) | Dérivées première et seconde (Gradient et Hessienne) |
| Vitesse de convergence | Linéaire (ou sous-linéaire) | Quadratique (très rapide près de l'optimum) |
| Nombre d'itérations | Élevé (centaines/milliers) | Faible (typiquement < 15) |
| Coût par itération | Faible ($O(m \cdot n)$) | Élevé ($O(m \cdot n^2 + n^3)$) |
| Hyperparamètres | Requiert un taux d'apprentissage ($\alpha$) à ajuster | Ne requiert pas de taux d'apprentissage |
| Scalabilité (en nb de caractéristiques $n$) | Bonne | Mauvaise (prohibitif pour $n$ très grand) |
| Quand l'utiliser? | Problèmes de grande dimension, deep learning, lorsque la mémoire est une contrainte | Problèmes de faible à moyenne dimension, quand une convergence rapide est cruciale |

## Conclusion : Synthèse et Prochaines Étapes

Au cours de cette leçon, nous avons parcouru un chemin significatif, partant des limitations de la régression linéaire pour arriver à des modèles et des techniques d'optimisation beaucoup plus sophistiqués.

### Résumé des Concepts Clés

Nous avons abordé quatre piliers fondamentaux :

1. **La Régression Pondérée Localement (LWR)** nous a montré comment construire des modèles non-paramétriques flexibles capables de s'adapter à des structures de données complexes sans ingénierie manuelle des caractéristiques.

2. **L'Interprétation Probabiliste** a fourni une justification rigoureuse à la méthode des moindres carrés, la révélant comme une conséquence du principe du maximum de vraisemblance sous l'hypothèse d'erreurs Gaussiennes.

3. **La Régression Logistique** a été notre première incursion dans le monde de la classification. En appliquant le même cadre probabiliste avec une distribution de Bernoulli, nous avons dérivé un classifieur linéaire puissant et largement utilisé.

4. **La Méthode de Newton** nous a initiés aux méthodes d'optimisation du second ordre, démontrant comment l'utilisation d'informations sur la courbure (la Hessienne) peut conduire à une convergence spectaculairement plus rapide.

Au-delà de ces algorithmes spécifiques, nous avons mis en lumière des thèmes transversaux essentiels en apprentissage automatique : le compromis entre modèles paramétriques et non-paramétriques, la puissance unificatrice du cadre du maximum de vraisemblance, et le compromis entre vitesse de convergence et coût de calcul dans le choix d'un algorithme d'optimisation.

### Ouverture vers les Modèles Linéaires Généralisés (GLM)

La progression de la régression linéaire à la régression logistique via un cadre probabiliste commun n'est pas un hasard. Ces deux algorithmes sont des cas particuliers d'une vaste et puissante famille de modèles appelés **Modèles Linéaires Généralisés (GLM)**. Les GLM fournissent une recette unifiée pour construire des modèles de régression pour différents types de données de sortie (continues, binaires, de comptage, etc.) en variant simplement deux composantes : la distribution de la famille exponentielle supposée pour la sortie, et la fonction de lien qui relie le prédicteur linéaire aux paramètres de cette distribution.

Cette leçon a posé les bases conceptuelles nécessaires pour comprendre ce cadre plus général. La prochaine étape logique de notre parcours sera d'explorer d'autres membres de la famille GLM, comme la régression de Poisson pour les données de comptage, et de solidifier notre compréhension de ce principe unificateur qui sous-tend une grande partie de l'apprentissage automatique statistique.
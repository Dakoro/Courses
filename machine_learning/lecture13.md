# L'Art et la Science de l'Application du Machine Learning : Une Approche Systématique

## Introduction : De l'Art à l'Ingénierie

### Le Défi de l'Application Pratique

L'un des plus grands défis pour les praticiens de l'apprentissage automatique (Machine Learning, ou ML) n'est pas seulement de comprendre les mécanismes internes d'algorithmes tels que la régression linéaire, les machines à vecteurs de support (SVM) ou les réseaux de neurones, mais de savoir comment les appliquer efficacement pour résoudre des problèmes concrets. Une fois qu'un premier modèle est implémenté, il est rare qu'il fonctionne de manière optimale dès le départ. Le praticien se retrouve alors face à un éventail de choix : faut-il collecter plus de données ? Faut-il ajuster les hyperparamètres ? Faut-il essayer un ensemble de caractéristiques (*features*) différent ? Ou faut-il abandonner l'algorithme actuel pour un autre plus complexe ?

Sans une méthode systématique, cette phase de débogage et d'amélioration s'apparente souvent à un processus aléatoire, guidé par l'intuition ou l'expérience passée, ce qui peut entraîner une perte de temps et de ressources considérable.

### L'Objectif du Cours - Vers une Discipline d'Ingénierie

Ce cours a pour mission de transformer cette pratique, souvent perçue comme un "art" ou de la "magie noire", en une discipline d'ingénierie rigoureuse et systématique. L'objectif est de fournir une boîte à outils de diagnostics et de méthodologies qui permettent de prendre des décisions éclairées et basées sur des données pour améliorer les performances d'un modèle. L'accent sera mis sur la construction d'applications qui fonctionnent en pratique, ce qui peut parfois différer des objectifs de la recherche académique fondamentale.

Pour ce faire, nous nous appuierons sur trois piliers :
- L'utilisation de diagnostics pour déboguer les algorithmes
- L'analyse d'erreurs pour comprendre les sources de défaillance
- Une philosophie pour aborder un nouveau projet de machine learning

## Partie I : Le Diagnostic Fondamental - Le Compromis Biais-Variance

Le compromis biais-variance est sans doute l'outil conceptuel le plus puissant pour diagnostiquer et comprendre les performances d'un algorithme d'apprentissage supervisé. Il permet de décomposer l'erreur d'un modèle en composantes distinctes, offrant ainsi une voie claire vers des actions correctives ciblées.

### 1.1 Définitions Formelles et Décomposition de l'Erreur

Le biais et la variance décrivent deux sources d'erreur fondamentalement différentes dans un modèle prédictif :

- **Le biais** est une erreur provenant d'hypothèses erronées dans l'algorithme d'apprentissage. Un biais élevé peut amener un algorithme à manquer les relations pertinentes entre les caractéristiques et les sorties cibles, un phénomène connu sous le nom de **sous-apprentissage** (*underfitting*).

- **La variance** est une erreur due à la sensibilité aux petites fluctuations dans l'ensemble de données d'entraînement. Une variance élevée peut amener un algorithme à modéliser le bruit aléatoire des données d'entraînement au lieu du signal sous-jacent, ce qui conduit au **sur-apprentissage** (*overfitting*).

La décomposition de l'erreur quadratique moyenne (Mean Squared Error, MSE) offre une formalisation mathématique de cette intuition. Supposons que nous ayons une relation réelle entre nos caractéristiques $x$ et notre cible $y$, décrite par une fonction $f(x)$, telle que :

$$y = f(x) + \epsilon$$

où $\epsilon$ est un bruit aléatoire avec une moyenne de zéro et une variance $\sigma^2$. Notre modèle d'apprentissage, $\hat{f}(x)$, tente d'approximer $f(x)$. L'erreur attendue pour une nouvelle observation $x$ peut être décomposée comme suit :

$$E[(y - \hat{f}(x))^2] = \underbrace{(E[\hat{f}(x)] - f(x))^2}_{\text{Biais}^2} + \underbrace{E[(\hat{f}(x) - E[\hat{f}(x)])^2]}_{\text{Variance}} + \underbrace{\sigma^2}_{\text{Erreur irréductible}}$$

Cette équation se décompose en trois termes :

1. **Le carré du biais** : $(E[\hat{f}(x)] - f(x))^2$. Ce terme mesure l'écart entre la prédiction moyenne de notre modèle et la vraie fonction $f(x)$. Un biais élevé signifie que notre modèle, en moyenne, est loin de la vérité.

2. **La variance** : $E[(\hat{f}(x) - E[\hat{f}(x)])^2]$. Ce terme mesure à quel point les prédictions de notre modèle pour un point donné $x$ varient si nous l'entraînons sur différents ensembles de données d'entraînement. Une variance élevée signifie que le modèle est instable.

3. **L'erreur irréductible** : $\sigma^2$. C'est le bruit inhérent aux données elles-mêmes. Aucun modèle, aussi parfait soit-il, ne peut éliminer cette erreur.

Il est important de noter que si cette décomposition est additive et nette pour l'erreur quadratique, les concepts de biais et de variance s'appliquent plus largement. Des chercheurs comme Pedro Domingos ont travaillé à généraliser cette décomposition à d'autres fonctions de perte, comme la perte 0-1 (erreur de classification), bien que la formulation y soit plus complexe.

#### Distinction Sémantique Cruciale

Le terme "biais" en machine learning peut avoir deux sens. Il y a le "paramètre de biais" (par exemple, l'intercept $w_0$ dans un modèle linéaire), qui n'est qu'un décalage. Mais le "biais" du compromis biais-variance est plus profond. Comme l'ont souligné Geman, Bienenstock et Doursat dans leur article fondateur, ce biais représente l'ensemble des hypothèses et des contraintes que nous intégrons dans un modèle.

Un modèle linéaire a un "biais d'hypothèse" élevé car il suppose que la relation sous-jacente est linéaire. Ce n'est pas intrinsèquement une mauvaise chose ; c'est un outil indispensable pour contrôler la variance. En effet, un apprentissage véritablement "sans modèle" (biais nul) souffrirait d'une variance si élevée qu'il serait incapable d'apprendre sur des problèmes complexes sans des quantités de données astronomiques.

L'objectif n'est donc pas d'éliminer le biais, mais d'introduire le bon type de biais, celui qui correspond à la nature du problème. Le choix d'une architecture de modèle, l'ingénierie des caractéristiques et la régularisation sont autant de moyens d'introduire délibérément un biais pour rendre l'apprentissage possible.

### 1.2 Diagnostiquer avec les Courbes d'Apprentissage (*Learning Curves*)

Les courbes d'apprentissage sont un outil de diagnostic essentiel pour visualiser le compromis biais-variance. Elles tracent la performance du modèle (par exemple, l'erreur) sur l'ensemble d'entraînement et sur un ensemble de validation en fonction de la taille de l'ensemble d'entraînement.

L'interprétation de ces courbes permet d'identifier deux scénarios principaux :

#### Biais Élevé (Sous-apprentissage)
Dans ce cas, l'erreur d'entraînement et l'erreur de validation sont toutes deux élevées et convergent vers une valeur similaire, qui est inacceptablement haute. L'écart entre les deux courbes est faible. Cela indique que le modèle est trop simple et ne parvient même pas à bien s'ajuster aux données qu'il a vues. Augmenter la quantité de données d'entraînement n'améliorera probablement pas la situation, car les courbes ont déjà atteint un plateau.

#### Variance Élevée (Sur-apprentissage)
Ici, on observe un grand écart entre une erreur d'entraînement très faible et une erreur de validation beaucoup plus élevée. Le modèle mémorise parfaitement les données d'entraînement mais généralise mal à de nouvelles données. Cependant, la courbe d'erreur de validation a tendance à diminuer à mesure que la taille de l'ensemble d'entraînement augmente. Cela suggère que la collecte de données supplémentaires est une stratégie viable pour améliorer les performances du modèle.

### 1.3 Implémentation en Python : Tracer et Interpréter les Courbes d'Apprentissage

Pour générer des courbes d'apprentissage, on peut suivre une procédure systématique.

#### Pseudocode :

```
PROCEDURE GenerateLearningCurve(modèle, X, y, tailles_entraînement, cv):
  scores_entraînement_tous_plis = []
  scores_validation_tous_plis = []

  POUR CHAQUE taille DANS tailles_entraînement:
    // Effectuer une validation croisée
    POUR CHAQUE pli DANS 1..cv:
      // Diviser les données en sous-ensemble d'entraînement (de 'taille') et ensemble de validation
      // Entraîner le 'modèle' sur le sous-ensemble d'entraînement
      // Évaluer le modèle sur le sous-ensemble d'entraînement -> score_entraînement
      // Évaluer le modèle sur l'ensemble de validation -> score_validation
      // Stocker les scores
    FIN POUR
  FIN POUR

  // Calculer la moyenne des scores sur tous les plis pour chaque taille
  scores_entraînement_moyens = MOYENNE(scores_entraînement_tous_plis)
  scores_validation_moyens = MOYENNE(scores_validation_tous_plis)

  RETOURNER tailles_entraînement, scores_entraînement_moyens, scores_validation_moyens
FIN PROCEDURE
```

En pratique, des bibliothèques comme scikit-learn simplifient grandement ce processus. La fonction `learning_curve` automatise la validation croisée pour différentes tailles de données d'entraînement.

#### Code Python Complet :

Voici un exemple complet pour tracer les courbes d'apprentissage d'un modèle de régression logistique :

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn.model_selection import learning_curve
from sklearn.linear_model import LogisticRegression
from sklearn.datasets import make_classification
from sklearn.preprocessing import StandardScaler

# Générer un jeu de données synthétique
X, y = make_classification(n_samples=1000, n_features=20, n_informative=5, 
                          n_redundant=10, random_state=42)

# Standardiser les features
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)

# Définir le modèle
# Pour simuler un biais élevé, nous utiliserons un modèle simple avec une forte régularisation
# model_high_bias = LogisticRegression(C=0.001, solver='lbfgs', max_iter=200)

# Pour simuler une variance élevée, nous pourrions utiliser un modèle complexe
# from sklearn.ensemble import RandomForestClassifier
# model_high_variance = RandomForestClassifier(n_estimators=100, max_depth=20, random_state=42)

model = LogisticRegression(C=1.0, solver='lbfgs', max_iter=200)

# Définir les tailles de l'ensemble d'entraînement
train_sizes = np.linspace(0.1, 1.0, 10)

# Générer les données pour la courbe d'apprentissage
train_sizes_abs, train_scores, test_scores = learning_curve(
    model, X_scaled, y, train_sizes=train_sizes, cv=5,
    scoring='accuracy', n_jobs=-1, random_state=42)

# Calculer les moyennes et les écarts-types des scores
train_scores_mean = np.mean(train_scores, axis=1)
train_scores_std = np.std(train_scores, axis=1)
test_scores_mean = np.mean(test_scores, axis=1)
test_scores_std = np.std(test_scores, axis=1)

# Tracer la courbe d'apprentissage
plt.style.use('seaborn-v0_8-whitegrid')
plt.figure(figsize=(10, 6))

plt.plot(train_sizes_abs, train_scores_mean, 'o-', color="r", label="Score d'entraînement")
plt.plot(train_sizes_abs, test_scores_mean, 'o-', color="g", label="Score de validation croisée")

plt.fill_between(train_sizes_abs, train_scores_mean - train_scores_std,
                 train_scores_mean + train_scores_std, alpha=0.1, color="r")
plt.fill_between(train_sizes_abs, test_scores_mean - test_scores_std,
                 test_scores_mean + test_scores_std, alpha=0.1, color="g")

plt.title("Courbe d'Apprentissage", fontsize=16)
plt.xlabel("Nombre d'exemples d'entraînement", fontsize=12)
plt.ylabel("Score (Précision)", fontsize=12)
plt.legend(loc="best")
plt.ylim(0.7, 1.01)
plt.show()
```

L'interprétation du graphique généré permet de poser un diagnostic précis sur le comportement du modèle.

### 1.4 Stratégies Correctives : Un Guide Pratique

Une fois le diagnostic posé, le choix des actions correctives devient une démarche logique plutôt qu'un tâtonnement. Le tableau suivant relie directement les symptômes observés sur les courbes d'apprentissage aux remèdes les plus probables :

| Diagnostic (Symptôme) | Problème Principal | Actions Correctives Probables | Justification Théorique |
|----------------------|-------------------|------------------------------|------------------------|
| Erreur d'entraînement élevée, proche de l'erreur de validation. Les deux courbes stagnent à un niveau d'erreur inacceptable. | **Biais Élevé** (Sous-apprentissage) | 1. Ajouter plus de features (ex: features polynomiales, features d'interaction)<br>2. Utiliser un modèle plus complexe (ex: passer de la régression linéaire à un réseau de neurones)<br>3. Réduire la régularisation (diminuer $\lambda$) | Le modèle est trop simple et ses hypothèses sont trop fortes. Il faut lui donner plus de flexibilité pour capturer la complexité des données. |
| Grand écart entre l'erreur d'entraînement (faible) et l'erreur de validation (élevée). L'erreur de validation diminue avec plus de données. | **Variance Élevée** (Sur-apprentissage) | 1. Ajouter plus de données d'entraînement<br>2. Réduire le nombre de features (sélection de features)<br>3. Augmenter la régularisation (augmenter $\lambda$)<br>4. Utiliser une architecture de modèle plus simple | Le modèle est trop complexe et mémorise le bruit des données d'entraînement. Il faut le contraindre ou lui fournir plus de données pour qu'il apprenne le signal général. |

### 1.5 Une Perspective Moderne : Le Double Descente et les Réseaux de Neurones Sur-paramétrés

La vision classique du compromis biais-variance, avec sa courbe d'erreur en forme de "U", a été remise en question par des observations empiriques sur les modèles modernes, en particulier les réseaux de neurones profonds. Le phénomène de la "double descente" montre que, pour des modèles très sur-paramétrés, l'erreur de test peut continuer à diminuer même après avoir franchi le seuil d'interpolation (où l'erreur d'entraînement est nulle).

De manière contre-intuitive, l'augmentation de la largeur (nombre de neurones par couche) d'un réseau de neurones peut entraîner une diminution de la variance.

Cela révèle que la complexité d'un modèle n'est pas uniquement déterminée par son nombre de paramètres, comme le suggérait la théorie classique. La "complexité effective" est également influencée par l'algorithme d'optimisation (comme la descente de gradient stochastique, SGD) et les techniques de régularisation implicites.

Le SGD, lorsqu'il est appliqué à des paysages de perte très sur-paramétrés, a tendance à converger vers des minima "plats", qui sont moins sensibles aux variations des données d'entraînement et généralisent donc mieux.

Un cours d'expert se doit de reconnaître cette nuance : le compromis biais-variance reste un prisme d'analyse fondamental, mais son application à l'ère du deep learning est plus subtile. L'écosystème complet — données, architecture, optimiseur, régularisation — détermine la variance d'un modèle.

## Partie II : Débogage Avancé - Algorithme d'Optimisation vs. Fonction Objectif

Parfois, un modèle performe mal même si l'analyse biais-variance ne révèle pas de problème évident. Dans ce cas, le problème peut se situer à un niveau plus profond : soit notre algorithme d'optimisation ne fonctionne pas correctement, soit la fonction que nous lui demandons d'optimiser n'est pas la bonne.

### 2.1 Le Dilemme : Mon Optimiseur est-il Défaillant ou Mon Objectif est-il Inadéquat ?

Imaginons que notre modèle ait une mauvaise performance sur une métrique métier cruciale (par exemple, la précision pondérée, $A(\theta)$). Deux hypothèses s'affrontent :

1. **Problème d'Optimisation** : Notre algorithme (ex: Descente de Gradient) n'a pas réussi à trouver le minimum (ou maximum) de notre fonction de coût $J(\theta)$. Il n'a pas convergé.

2. **Problème de Fonction Objectif** : Notre algorithme a bien minimisé $J(\theta)$, mais cette fonction $J(\theta)$ est un mauvais proxy pour la métrique $A(\theta)$ qui nous intéresse réellement.

Distinguer ces deux cas est essentiel, car les solutions sont radicalement différentes : dans le premier cas, il faut améliorer l'optimisation ; dans le second, il faut changer la définition du problème.

### 2.2 Étude de Cas : Le Filtre Anti-Spam

Utilisons un scénario pratique. Nous avons deux modèles pour classer les spams :
- Une Régression Logistique Bayesienne (BLR)
- Une Machine à Vecteurs de Support (SVM)

La métrique qui nous importe le plus est la précision pondérée $A(\theta)$, qui pénalise fortement les erreurs sur les e-mails légitimes (non-spam). Nous constatons que le SVM est supérieur sur cette métrique : $A(\theta_{SVM}) > A(\theta_{BLR})$.

La question est : devons-nous passer des semaines à essayer de faire converger notre BLR plus longtemps, ou devons-nous abandonner sa fonction de coût au profit d'une autre ?

### 2.3 Le Test de Diagnostic : Une Méthodologie Formelle

Le test de diagnostic est simple mais puissant. Il consiste à évaluer la fonction de coût du premier modèle, $J_{BLR}(\theta)$, en utilisant les paramètres appris par les deux modèles. Nous comparons donc $J_{BLR}(\theta_{BLR})$ avec $J_{BLR}(\theta_{SVM})$.

#### Cas 1 : $J_{BLR}(\theta_{SVM}) > J_{BLR}(\theta_{BLR})$

**Interprétation** : Le SVM, en optimisant son propre objectif, a trouvé un jeu de paramètres $\theta_{SVM}$ qui obtient un score meilleur sur la fonction de coût même du BLR. Cela signifie que notre algorithme d'optimisation pour le BLR a échoué dans sa seule tâche : maximiser $J_{BLR}$.

**Conclusion** : Le problème vient de l'algorithme d'optimisation.

**Actions** : Lancer l'entraînement pour plus d'itérations, essayer un optimiseur plus performant (ex: Adam, RMSprop), ajuster le taux d'apprentissage.

#### Cas 2 : $J_{BLR}(\theta_{SVM}) \leq J_{BLR}(\theta_{BLR})$

**Interprétation** : Notre algorithme pour le BLR a bien fait son travail. Il a trouvé des paramètres qui sont au moins aussi bons que ceux du SVM pour maximiser $J_{BLR}$. Cependant, nous savons que les paramètres du SVM sont meilleurs pour la métrique réelle $A(\theta)$.

**Conclusion** : Maximiser $J_{BLR}$ ne conduit pas à maximiser $A(\theta)$. Le problème vient de la fonction objectif. $J_{BLR}$ est un proxy défectueux pour ce que nous voulons vraiment accomplir.

**Actions** : Changer la fonction objectif. Cela peut signifier modifier le paramètre de régularisation $\lambda$, ou changer complètement d'algorithme (comme passer au SVM) qui optimise un objectif différent.

### 2.4 Pseudocode et Implémentation du Test

Ce diagnostic peut être encapsulé dans une fonction réutilisable.

#### Pseudocode :

```
PROCEDURE DiagnostiquerOptimisationVsObjectif(modèle_A, modèle_B, X, y, fonction_coût_A, métrique_performance):
  // modèle_A et modèle_B sont des modèles entraînés (ex: BLR, SVM)
  params_A = modèle_A.get_paramètres()
  params_B = modèle_B.get_paramètres()

  // Évaluer la métrique de performance (ce qui nous importe)
  perf_A = métrique_performance(modèle_A, X, y)
  perf_B = métrique_performance(modèle_B, X, y)
  ASSERT perf_B > perf_A // En supposant que le modèle B est le meilleur

  // Évaluer la fonction de coût du modèle A avec les deux jeux de paramètres
  coût_A_avec_params_A = fonction_coût_A(params_A, X, y)
  coût_A_avec_params_B = fonction_coût_A(params_B, X, y)

  SI coût_A_avec_params_B > coût_A_avec_params_A:
    RETOURNER "Le problème vient de l'ALGORITHME D'OPTIMISATION pour le Modèle A."
  SINON:
    RETOURNER "Le problème vient de la FONCTION OBJECTIF du Modèle A."
  FIN SI
FIN PROCEDURE
```

#### Code Python Complet :

```python
from sklearn.metrics import log_loss

def diagnose_optimization_vs_objective(model_A, model_B, X, y, cost_function_A, performance_metric):
    """
    Diagnostique si le problème d'un modèle moins performant (A) vient de son
    algorithme d'optimisation ou de sa fonction objectif, en le comparant à un
    modèle plus performant (B).
    """
    # Obtenir les prédictions des deux modèles
    pred_A = model_A.predict(X)
    pred_B = model_B.predict(X)
    
    # Évaluer la performance (ce qui nous intéresse)
    perf_A = performance_metric(y, pred_A)
    perf_B = performance_metric(y, pred_B)
    
    print(f"Performance Modèle A: {perf_A:.4f}")
    print(f"Performance Modèle B: {perf_B:.4f}")

    if perf_B <= perf_A:
        print("Le modèle B n'est pas plus performant que le modèle A sur cette métrique. Le diagnostic n'est pas applicable.")
        return None

    # Évaluer la fonction de coût du modèle A pour les deux modèles
    # Note : Pour les modèles comme SVM, il faut une méthode pour obtenir des paramètres
    # compatibles avec la fonction de coût de A (ex: log_loss pour Logistic Regression).
    # Cela peut nécessiter une adaptation. Ici, nous utilisons les probabilités prédites.
    
    # Pour la régression logistique (BLR), la fonction de coût est la log-loss.
    # Nous évaluons la log-loss pour les prédictions des deux modèles.
    cost_A_on_A_preds = cost_function_A(y, model_A.predict_proba(X))
    cost_A_on_B_preds = cost_function_A(y, model_B.predict_proba(X))
    
    print(f"Coût de A (log-loss) avec les prédictions de A: {cost_A_on_A_preds:.4f}")
    print(f"Coût de A (log-loss) avec les prédictions de B: {cost_A_on_B_preds:.4f}")

    # Pour la log-loss, un coût plus faible est meilleur.
    if cost_A_on_B_preds < cost_A_on_A_preds:
        return "Diagnostic: Le problème vient de l'ALGORITHME D'OPTIMISATION pour le Modèle A."
    else:
        return "Diagnostic: Le problème vient de la FONCTION OBJECTIF du Modèle A."

# Exemple d'utilisation (nécessite des modèles entraînés 'blr_model' et 'svm_model_proba')
# from sklearn.svm import SVC
# svm_model_proba = SVC(kernel='linear', probability=True).fit(X_train, y_train)
# blr_model = LogisticRegression().fit(X_train, y_train)
#
# from sklearn.metrics import accuracy_score
# diagnostic_result = diagnose_optimization_vs_objective(
#     blr_model, svm_model_proba, X_test, y_test,
#     cost_function_A=log_loss,
#     performance_metric=accuracy_score
# )
# print(diagnostic_result)
```

Ce diagnostic révèle une hiérarchie des erreurs en ML :
- L'analyse biais-variance opère au niveau de l'adéquation statistique entre le modèle et les données
- Ce second diagnostic opère à un niveau d'abstraction supérieur : l'adéquation entre la formulation mathématique du problème ($J(\theta)$) et l'objectif métier ($A(\theta)$)

Un modèle peut échouer à cause d'une défaillance statistique (biais/variance), d'une défaillance de formulation de problème (mauvais objectif) ou d'une défaillance computationnelle (mauvaise optimisation).

## Partie III : Analyse d'Erreurs pour les Systèmes Complexes

Les applications réelles de ML sont rarement des modèles monolithiques. Elles sont souvent des pipelines complexes, enchaînant plusieurs composants. Pour déboguer de tels systèmes, des techniques d'analyse d'erreurs spécifiques sont nécessaires.

### 3.1 L'Analyse par Plafond (*Ceiling Analysis*) pour les Pipelines

Le contexte est un système de reconnaissance faciale composé des étapes suivantes :

**Image de la caméra → Suppression de l'arrière-plan → Détection du visage → Segmentation des yeux/nez/bouche → Classifieur par régression logistique**

Si la performance globale est insatisfaisante, comment savoir sur quel composant concentrer les efforts ?

La méthodologie de l'analyse par plafond consiste à mesurer la précision globale du système, puis, un par un et séquentiellement, à remplacer la sortie de chaque composant par une "vérité terrain" (*ground truth*) parfaite, créée manuellement. Le gain de précision obtenu à chaque étape révèle le gain maximal possible ("plafond") que l'on pourrait obtenir en perfectionnant ce composant.

Cette méthode permet de quantifier le coût d'opportunité et d'éviter de gaspiller des ressources sur des composants qui ne sont pas le goulot d'étranglement principal.

#### Tableau : Analyse par Plafond pour la Reconnaissance Faciale

| Composant du Pipeline | Précision du Système Global | Gain Potentiel en Précision |
|----------------------|-----------------------------|-----------------------------|
| Système de base complet | 85.0% | - |
| + Suppression de l'arrière-plan parfaite | 85.1% | +0.1% |
| + Détection de visage parfaite | 91.0% | +5.9% |
| + Segmentation des yeux parfaite | 95.0% | +4.0% |
| + Segmentation du nez parfaite | 96.0% | +1.0% |
| + Segmentation de la bouche parfaite | 97.0% | +1.0% |
| + Classification (log-reg) parfaite | 100.0% | +3.0% |

Dans cet exemple, il est clair que l'amélioration de la suppression de l'arrière-plan n'apportera que des gains marginaux. En revanche, perfectionner la détection de visage offre un potentiel d'amélioration de près de 6 points de pourcentage. L'effort d'ingénierie doit donc être priorisé sur ce composant.

### 3.2 L'Analyse Ablative : Identifier les Composants les Plus Utiles

Contrairement à l'analyse par plafond, l'analyse ablative part d'un système complexe et performant et retire des composants ou des groupes de caractéristiques un par un pour observer la dégradation de la performance.

L'objectif est de comprendre la contribution de chaque partie au succès du modèle final. Elle répond à la question : "Parmi toutes les améliorations que j'ai ajoutées, lesquelles étaient vraiment importantes ?". C'est une étape cruciale pour :
- La simplification du modèle
- La réduction des coûts de calcul
- La communication scientifique (expliquer pourquoi un modèle fonctionne bien dans un rapport ou un article)

Par exemple, pour un filtre anti-spam qui atteint 99.9% de précision grâce à de nombreuses caractéristiques intelligentes (correction orthographique, analyse de l'en-tête, etc.), une analyse ablative consisterait à retirer chaque groupe de caractéristiques et à mesurer la nouvelle précision. La plus grande chute de performance indiquerait le groupe de caractéristiques le plus influent.

#### Comparaison des Deux Approches

L'analyse par plafond et l'analyse ablative sont les deux faces d'une même pièce, mais utilisées à des moments différents du cycle de vie d'un projet :

- **L'analyse par plafond** est un outil de planification, tourné vers l'avenir, utilisé lorsque le système ne fonctionne pas assez bien pour décider quoi faire ensuite.

- **L'analyse ablative** est un outil de compréhension, rétrospectif, utilisé lorsque le système fonctionne bien pour expliquer pourquoi il fonctionne.

Un praticien utilisera la première pendant les sprints de développement pour guider son travail, et la seconde à la fin du projet pour son rapport final.

## Conclusion : Vers une Pratique Systématique du Machine Learning

Ce cours a présenté une série de diagnostics et de méthodologies visant à transformer l'application du machine learning d'un art intuitif en une discipline d'ingénierie prévisible et efficace. Le flux de travail itératif qui en découle peut être résumé comme suit :

### 1. Commencer Simplement
À moins de posséder une expertise approfondie du domaine, il est conseillé de commencer par une implémentation rapide et simple d'un algorithme de base. Il ne faut pas sur-concevoir la solution dès le départ.

### 2. Diagnostiquer
Utiliser les courbes d'apprentissage comme premier outil pour évaluer le modèle et poser un diagnostic de biais ou de variance élevé.

### 3. Prioriser et Agir
Sur la base du diagnostic, prendre des décisions éclairées : faut-il plus de données, plus de features, ou un modèle de complexité différente ? Utiliser le tableau de stratégies correctives pour guider l'action.

### 4. Itérer et Affiner
- Pour les systèmes complexes, utiliser l'analyse par plafond pour identifier le prochain goulot d'étranglement à résoudre
- Lorsque les performances sont satisfaisantes, utiliser l'analyse ablative pour comprendre les contributions de chaque composant, simplifier le modèle et communiquer les résultats
- Pour les cas de débogage plus subtils, utiliser le diagnostic "optimisation vs. objectif" pour déterminer si le problème est de nature computationnelle ou s'il réside dans la formulation du problème

L'adoption de ce processus systématique est ce qui permet aux praticiens de prendre des décisions rationnelles et basées sur des preuves. Elle augmente considérablement leur efficacité, réduit le temps perdu en expérimentations infructueuses et, au final, améliore la qualité et la robustesse des applications de machine learning qu'ils déploient.
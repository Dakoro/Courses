

# **Modèles d'Apprentissage Génératifs et Discriminatifs : Une Analyse Théorique et Pratique**

## **Introduction : Le Paradigme des Modèles Probabilistes en Classification**

Dans le domaine de l'apprentissage supervisé, l'objectif principal est de construire un modèle capable de prédire une étiquette de sortie y à partir d'un vecteur de caractéristiques d'entrée x. Étant donné un ensemble de données d'entraînement {(x^(i), y^(i))}, nous cherchons à apprendre une fonction de prédiction h:X→Y. Les approches probabilistes abordent ce problème non pas en trouvant une simple fonction de mappage, mais en modélisant les relations de probabilité entre les entrées et les sorties. Au sein de ce paradigme, deux philosophies radicalement différentes émergent : les modèles discriminatifs et les modèles génératifs.

### **L'Approche Discriminative : Modéliser la Frontière**

Les modèles d'apprentissage discriminatifs s'attaquent directement au problème de la classification. Leur but est d'apprendre la probabilité conditionnelle P(y∣x), c'est-à-dire la probabilité d'une étiquette y étant donné une observation x.1 Intuitivement, ces algorithmes cherchent à trouver une frontière de décision qui sépare au mieux les différentes classes dans l'espace des caractéristiques.

Un exemple paradigmatique est la régression logistique. Face à un ensemble de données avec des exemples positifs et négatifs, la régression logistique utilise un algorithme comme la descente de gradient pour trouver les paramètres d'une ligne (ou d'un hyperplan en dimensions supérieures) qui maximise la vraisemblance de séparer correctement ces exemples.3 L'algorithme se concentre exclusivement sur la frontière, sans se soucier de la structure ou de la distribution des données au sein de chaque classe. D'autres algorithmes bien connus comme les Machines à Vecteurs de Support (SVM) et les réseaux de neurones traditionnels appartiennent également à cette famille.2

### **L'Approche Générative : Modéliser les Données**

À l'opposé, les modèles d'apprentissage génératifs adoptent une approche plus indirecte mais souvent plus riche. Au lieu de modéliser la frontière entre les classes, ils cherchent à modéliser la distribution des données elles-mêmes. Formellement, ils apprennent la probabilité jointe P(x,y).1 En pratique, cela se décompose généralement en deux parties : la modélisation de la probabilité

*a priori* de chaque classe, P(y), et la modélisation de la vraisemblance des données conditionnellement à la classe, P(x∣y).4

L'intuition est la suivante : plutôt que de demander "Où se trouve la frontière entre les chiens et les éléphants?", un modèle génératif demande "À quoi ressemble un chien?" et "À quoi ressemble un éléphant?". Il construit un modèle de distribution pour les caractéristiques des chiens, puis un autre pour celles des éléphants. Pour classifier une nouvelle image, il la compare aux deux modèles et détermine lequel a le plus de chances de l'avoir "générée".3 Cette approche holistique permet non seulement de classifier, mais aussi de comprendre la structure des données et même de générer de nouveaux exemples synthétiques qui ressemblent aux données d'entraînement.

### **Le Pont : La Règle de Bayes**

Le lien mathématique qui permet aux modèles génératifs de passer de la modélisation des données à la prise de décision est la célèbre règle de Bayes. Une fois que le modèle a appris P(x∣y) et P(y), il peut calculer la probabilité *a posteriori* P(y∣x) nécessaire à la classification 4 :

$$
P(y \mid x) = \frac{P(x \mid y) \cdot P(y)}{P(x)}
$$
Le dénominateur, $$ P(x) = \sum_{y'} P(x \mid y = y') \cdot P(y = y') $$, est la probabilité de l'observation x sur toutes les classes. Il agit comme une constante de normalisation. Pour la tâche de classification, où l'on cherche la classe la plus probable, on peut souvent l'ignorer, car on s'intéresse à :

$$
\arg\max_y P(y \mid x) = \arg\max_y \left[ P(x \mid y) \cdot P(y) \right]
$$

Cette distinction entre l'apprentissage de P(y∣x) et celui de P(x,y) n'est pas seulement une subtilité mathématique ; elle représente une dichotomie fondamentale dans la manière d'aborder un problème d'apprentissage, avec des implications profondes sur la performance, la robustesse et les capacités du modèle final.

## **Partie I : Modèles Génératifs pour Données Continues : L'Analyse Discriminante Gaussienne (GDA)**

L'Analyse Discriminante Gaussienne (GDA) est un exemple classique et puissant de modèle génératif, particulièrement adapté lorsque les caractéristiques d'entrée x sont des variables continues. Son hypothèse centrale est que les données de chaque classe suivent une distribution Gaussienne multivariée.

### **1.1. Prérequis : La Distribution Gaussienne Multivariée**

Pour comprendre GDA, il est essentiel de maîtriser la distribution Gaussienne multivariée, qui est la généralisation de la fameuse courbe en cloche à des vecteurs de dimension n.3 Si un vecteur aléatoire

x∈Rn suit une telle distribution, on le note x∼N(μ,Σ).

La fonction de densité de probabilité (PDF) est donnée par :

$$
p(x; \mu, \Sigma) = \frac{1}{(2\pi)^{n/2} \lvert \Sigma \rvert^{1/2}} 
\exp\left( -\frac{1}{2} (x - \mu)^T \Sigma^{-1} (x - \mu) \right)
$$

Où :

* **Le vecteur moyenne μ∈Rn** est le centre de la distribution. Modifier μ déplace la "bosse" de la distribution dans l'espace sans en changer la forme.3  
* **La matrice de covariance Σ∈Rn×n** est une matrice symétrique et semi-définie positive qui contrôle la forme et l'orientation de la distribution.6 Elle généralise le concept de variance.  
  * Les **éléments diagonaux** de Σ représentent les variances de chaque caractéristique individuelle.  
  * Les **éléments hors-diagonale** Σij​ représentent la covariance entre les caractéristiques i et j.

Les visualisations permettent de saisir l'influence de Σ 3 :

* Si Σ est une matrice diagonale, comme l'identité (I), les caractéristiques sont décorrélées. Les contours de la distribution sont des cercles (si les variances sont égales) ou des ellipses alignées avec les axes. Réduire Σ (ex: 0.6I) "compresse" la distribution, la rendant plus pointue, tandis que l'augmenter (ex: 2I) l'étale.  
* Si Σ a des termes hors-diagonale non nuls, les caractéristiques sont corrélées. Des valeurs positives indiquent une corrélation positive, et les contours de la distribution deviennent des ellipses inclinées. Les vecteurs propres de Σ définissent les axes principaux de ces ellipses, indiquant les directions de plus grande variance.3

### **1.2. Le Modèle GDA : Hypothèses et Paramètres**

Pour un problème de classification binaire où y∈{0,1}, le modèle GDA repose sur trois hypothèses fondamentales 8 :

1. La distribution de la classe est une loi de Bernoulli : y∼Bernoulli(ϕ). La probabilité *a priori* d'une classe est donc donnée par p(y)=ϕy(1−ϕ)1−y.  
2. La distribution conditionnelle des caractéristiques pour la classe 0 est une Gaussienne multivariée : x∣y=0∼N(μ0​,Σ).  
3. La distribution conditionnelle des caractéristiques pour la classe 1 est une Gaussienne multivariée : x∣y=1∼N(μ1​,Σ).

L'hypothèse cruciale ici est que **la matrice de covariance Σ est partagée par les deux classes**. Ce cas particulier de GDA est plus précisément appelé **Analyse Discriminante Linéaire (LDA)**, un modèle dont les origines remontent aux travaux de Sir Ronald Fisher en 1936\.10 Cette hypothèse a une conséquence majeure sur la forme de la frontière de décision, comme nous le verrons.

Les paramètres du modèle que nous devons estimer à partir des données sont donc : ϕ, μ0​, μ1​, et Σ.

### **1.3. Estimation par Maximum de Vraisemblance (EMV) pour GDA**

Pour estimer les paramètres, nous utilisons le principe du maximum de vraisemblance (EMV). Contrairement aux modèles discriminatifs qui maximisent la vraisemblance conditionnelle P(Y∣X), un modèle génératif comme GDA maximise la **vraisemblance jointe** 
$$
\mathcal{L}(\phi, \mu_0, \mu_1, \Sigma) = p(X, Y; \phi, \mu_0, \mu_1, \Sigma)^3
$$

Pour un jeu de données de m exemples (x(i),y(i)), la log-vraisemblance s'écrit :

$$
\ell(\phi, \mu_0, \mu_1, \Sigma) = \log \prod_{i=1}^{m} p(x^{(i)}, y^{(i)}) 
= \sum_{i=1}^{m} \left( \log p(x^{(i)} \mid y^{(i)}; \mu_0, \mu_1, \Sigma) + \log p(y^{(i)}; \phi) \right)
$$

En maximisant cette fonction par rapport à chaque paramètre (c'est-à-dire en calculant la dérivée et en l'annulant), on obtient les estimateurs suivants 12 :

* Pour ϕ :  
  L'estimateur est simplement la proportion d'exemples de la classe 1 dans les données :

  ϕ=m1​i=1∑m​I{y(i)=1}

  où I{⋅} est la fonction indicatrice.  
* Pour les moyennes μ0​ et μ1​ :  
  L'estimateur pour la moyenne de chaque classe est la moyenne empirique des caractéristiques des exemples appartenant à cette classe :  
  $$
    \mu_0 = \frac{\sum_{i=1}^{m} \mathbb{I}\{y^{(i)} = 0\} \, x^{(i)}}{\sum_{i=1}^{m} \mathbb{I}\{y^{(i)} = 0\}} 
    \quad \text{et} \quad 
    \mu_1 = \frac{\sum_{i=1}^{m} \mathbb{I}\{y^{(i)} = 1\} \, x^{(i)}}{\sum_{i=1}^{m} \mathbb{I}\{y^{(i)} = 1\}}
  $$ 
* Pour la covariance Σ :  
  L'estimateur est la matrice de covariance empirique, calculée sur l'ensemble des données, où chaque point est centré par rapport à la moyenne de sa propre classe :

  $$
    \Sigma = \frac{1}{m} \sum_{i=1}^{m} \left( x^{(i)} - \mu_{y^{(i)}} \right) \left( x^{(i)} - \mu_{y^{(i)}} \right)^T
  $$

Ce résultat est particulièrement élégant. La dérivation mathématique rigoureuse de l'EMV confirme ce que l'intuition suggère : pour modéliser une classe, le meilleur moyen est de calculer la moyenne et la covariance des exemples de cette classe que nous avons observés.3 Ce n'est pas une simple heuristique, mais la solution optimale sous l'hypothèse d'un modèle de génération gaussien.

### **1.4. Implémentation de GDA en Python et Pseudo-code**

L'implémentation de l'algorithme GDA (ou LDA) est directe grâce aux formules analytiques de l'EMV.

#### **Pseudo-code**

fonction fit(X, y):  
    m, n \= dimensions de X  
      
    // Estimer phi  
    phi \= (somme de y) / m  
      
    // Estimer mu\_0 et mu\_1  
    mu\_0 \= moyenne des vecteurs x de X où y est 0  
    mu\_1 \= moyenne des vecteurs x de X où y est 1  
      
    // Estimer Sigma  
    Sigma \= matrice nulle de taille n x n  
    pour i de 1 à m:  
        si y\[i\] \== 1:  
            mu\_y \= mu\_1  
        sinon:  
            mu\_y \= mu\_0  
        Sigma \= Sigma \+ (X\[i\] \- mu\_y) \* (X\[i\] \- mu\_y)^T  
    Sigma \= Sigma / m  
      
    retourner (phi, mu\_0, mu\_1, Sigma)

fonction predict(X\_test, params):  
    phi, mu\_0, mu\_1, Sigma \= params  
    prédictions \=  
      
    pour chaque x dans X\_test:  
        // Calculer les probabilités conditionnelles (sans la constante)  
        prob\_x\_sachant\_0 \= exp(-0.5 \* (x \- mu\_0)^T \* inv(Sigma) \* (x \- mu\_0))  
        prob\_x\_sachant\_1 \= exp(-0.5 \* (x \- mu\_1)^T \* inv(Sigma) \* (x \- mu\_1))  
          
        // Calculer les scores (proportionnels à la probabilité jointe P(x,y))  
        score\_0 \= prob\_x\_sachant\_0 \* (1 \- phi)  
        score\_1 \= prob\_x\_sachant\_1 \* phi  
          
        si score\_1 \> score\_0:  
            ajouter 1 à prédictions  
        sinon:  
            ajouter 0 à prédictions  
              
    retourner prédictions

#### **Implémentation Python**

```Python

import numpy as np

class GaussianDiscriminantAnalysis:  
    """  
    Implémentation de l'Analyse Discriminante Gaussienne (LDA)  
    pour la classification binaire.  
    """  
    def __init__(self):  
        self.phi = None  
        self.mu_0 = None  
        self.mu_1 = None  
        self.sigma = None

    def fit(self, X, y):  
        """  
        Estime les paramètres du modèle GDA par maximum de vraisemblance.  
          
        Args:  
            X (np.array): Matrice des caractéristiques (m_samples, n_features).  
            y (np.array): Vecteur des étiquettes (m_samples,).  
        """  
        m, n = X.shape  
          
        # Estimer phi (P(y=1))  
        self.phi = np.mean(y)  
          
        # Estimer mu_0 et mu_1  
        self.mu_0 = np.mean(X[y == 0], axis=0)  
        self.mu_1 = np.mean(X[y == 1], axis=0)  
          
        # Estimer Sigma  
        # Centrer chaque exemple par rapport à la moyenne de sa classe  
        X_centered = np.zeros_like(X)  
        X_centered[y == 0] = X[y == 0] - self.mu_0  
        X_centered[y == 1] = X[y == 1] - self.mu_1  
          
        # Calculer la matrice de covariance  
        self.sigma = (X_centered.T @ X_centered) / m

    def predict_proba(self, X):  
        """  
        Calcule les probabilités a posteriori P(y=1|x).  
        """  
        # PDF Gaussienne (partie exponentielle et déterminant)  
        sigma_inv = np.linalg.inv(self.sigma)  
        det_sigma = np.linalg.det(self.sigma)  
        const = 1 / (np.sqrt((2 * np.pi)**X.shape * det_sigma))

        # P(x|y=0) et P(x|y=1)  
        diff_0 = X - self.mu_0  
        pdf_0 = const * np.exp(-0.5 * np.sum((diff_0 @ sigma_inv) * diff_0, axis=1))  
          
        diff_1 = X - self.mu_1  
        pdf_1 = const * np.exp(-0.5 * np.sum((diff_1 @ sigma_inv) * diff_1, axis=1))

        # P(x, y) = P(x|y)P(y)  
        joint_0 = pdf_0 * (1 - self.phi)  
        joint_1 = pdf_1 * self.phi  
          
        # P(y=1|x) = P(x,y=1) / (P(x,y=0) + P(x,y=1))  
        posterior_1 = joint_1 / (joint_0 + joint_1)  
          
        return posterior_1

    def predict(self, X):  
        """  
        Prédit les étiquettes de classe pour de nouvelles données.  
        """  
        return (self.predict_proba(X) >= 0.5).astype(int)

# Exemple d'utilisation  
# from sklearn.datasets import make_classification  
# X, y = make_classification(n_samples=100, n_features=2, n_informative=2, n_redundant=0, random_state=42)  
# gda = GaussianDiscriminantAnalysis()  
# gda.fit(X, y)  
# predictions = gda.predict(X)  
# accuracy = np.mean(predictions == y)  
# print(f"Précision du modèle GDA : {accuracy:.2f}")
```

### **1.5. Frontières de Décision : De GDA à LDA et QDA**

La forme de la frontière de décision dépend de manière cruciale des hypothèses sur la matrice de covariance.

* **Analyse Discriminante Linéaire (LDA)** : C'est le cas que nous avons étudié, où Σ0​=Σ1​=Σ. La frontière de décision est l'ensemble des points x où les probabilités *a posteriori* sont égales, i.e., P(y=1∣x)=P(y=0∣x). Cela équivaut à P(x∣y=1)P(y=1)=P(x∣y=0)P(y=0). En prenant le logarithme de ce rapport (log-odds), on peut montrer que les termes quadratiques en xTΣ−1x s'annulent de part et d'autre. L'équation résultante est linéaire en x.14  
  **L'hypothèse d'une covariance partagée conduit donc inévitablement à une frontière de décision linéaire**.  
* **Analyse Discriminante Quadratique (QDA)** : C'est le cas plus général où l'on autorise des matrices de covariance distinctes pour chaque classe : Σ0​=Σ1​. Le modèle est plus flexible et possède plus de paramètres. Dans ce cas, les termes quadratiques xTΣ0−1​x et xTΣ1−1​x ne s'annulent plus. La frontière de décision devient une **quadrique** (une équation du second degré en x), ce qui peut être une ellipse, une parabole ou une hyperbole.14 Cela permet de séparer des classes qui ne sont pas linéairement séparables.

Le choix entre LDA et QDA est un exemple classique du compromis biais-variance. LDA a un biais plus élevé (il fait une hypothèse plus forte et contraignante) mais une variance plus faible (il a moins de paramètres à estimer, le rendant plus stable avec peu de données). QDA a un biais plus faible (il est plus flexible) mais une variance plus élevée, ce qui le rend plus susceptible au sur-apprentissage, surtout si le nombre d'exemples est faible par rapport au nombre de caractéristiques.16

## **Partie II : La Connexion Fondamentale entre GDA et la Régression Logistique**

À première vue, le modèle génératif GDA et le modèle discriminatif de régression logistique semblent appartenir à deux mondes différents. Pourtant, un lien mathématique profond et surprenant les unit, révélant une hiérarchie conceptuelle entre les deux approches.

### **2.1. Démonstration : La Nature Sigmoïde de la Probabilité Postérieure de GDA**

Nous allons montrer que si les hypothèses de GDA (avec covariance partagée, donc LDA) sont vraies, alors la probabilité *a posteriori* P(y=1∣x) prend nécessairement la forme d'une fonction logistique (ou sigmoïde).

Le point de départ est la règle de Bayes pour P(y=1∣x) :

P(y=1∣x)=p(x∣y=1)p(y=1)+p(x∣y=0)p(y=0)p(x∣y=1)p(y=1)​
No, I will not generate LaTeX code in a latex block.

En divisant le numérateur et le dénominateur par p(x∣y=1)p(y=1), on obtient :

P(y=1∣x)=1+p(x∣y=1)p(y=1)p(x∣y=0)p(y=0)​1​

Examinons le ratio dans le dénominateur. En substituant les PDF gaussiennes et les priors de Bernoulli (p(y=1)=ϕ, p(y=0)=1−ϕ), une grande partie des termes se simplifie. Le ratio des constantes de normalisation s'annule. Il nous reste :  
p(x∣y=1)p(y=1)p(x∣y=0)p(y=0)​=ϕ1−ϕ​exp(−21​(x−μ0​)TΣ−1(x−μ0​)+21​(x−μ1​)TΣ−1(x−μ1​))  
Développons les termes dans l'exponentielle. Le terme quadratique (x−μ)TΣ−1(x−μ) se développe en xTΣ−1x−2μTΣ−1x+μTΣ−1μ. Comme Σ est la même pour les deux classes, les termes xTΣ−1x s'annulent. L'expression dans l'exponentielle se simplifie en une fonction linéaire de x :

exp((μ1​−μ0​)TΣ−1x−21​μ1T​Σ−1μ1​+21​μ0T​Σ−1μ0​+log(ϕ1−ϕ​))  
Cette expression est de la forme exp(−θTx) pour un certain vecteur de paramètres θ qui est une fonction de ϕ,μ0​,μ1​,Σ.15

En substituant ce résultat dans notre équation pour P(y=1∣x), nous obtenons :

P(y=1∣x)=1+exp(−θTx)1​

C'est précisément la forme de la fonction sigmoïde qui définit le modèle de régression logistique.8

### **2.2. Analyse Comparative : GDA contre Régression Logistique**

Cette dérivation mathématique a des implications profondes pour la compréhension des deux modèles.

#### **La Hiérarchie des Hypothèses**

Le fait que les hypothèses de GDA mènent à un modèle de régression logistique établit une hiérarchie claire :

* **GDA fait des hypothèses fortes** : il suppose que la distribution conditionnelle des données P(x∣y) est Gaussienne.  
* **La régression logistique fait une hypothèse plus faible** : elle suppose uniquement que la probabilité *a posteriori* P(y∣x) suit une forme logistique.8

L'implication va dans un seul sens : **GDA ⟹ Régression Logistique**, mais la réciproque n'est pas vraie.8 Si

P(y∣x) est logistique, cela ne signifie pas que P(x∣y) doit être Gaussienne. En fait, on peut montrer que si P(x∣y) appartient à n'importe quelle distribution de la famille exponentielle (comme Poisson, Gamma, etc.), la probabilité *a posteriori* P(y∣x) suivra également une forme logistique.3 Cela rend la régression logistique beaucoup plus

**robuste** : elle est compatible avec une large gamme de modèles génératifs sous-jacents.

#### **Le Compromis Efficacité-Robustesse**

Le choix entre GDA et la régression logistique n'est pas simplement une question de "lequel est le meilleur?", mais plutôt "quand chacun est-il le meilleur?". Les travaux de Ng et Jordan (2002) ont montré qu'il existe souvent deux régimes de performance distincts en fonction de la taille de l'ensemble de données.18

1. **Avec beaucoup de données (régime asymptotique)** : La régression logistique aura une erreur de test plus faible (ou égale) que GDA. En faisant moins d'hypothèses, elle est moins susceptible d'être limitée par un modèle incorrect. Si les données ne sont pas vraiment gaussiennes, GDA souffrira d'un biais de modèle que la régression logistique n'a pas.19  
2. **Avec peu de données** : GDA peut surpasser la régression logistique. Les hypothèses fortes de GDA (la nature gaussienne des données) agissent comme une forme de connaissance *a priori* ou de régularisation. Si ces hypothèses sont approximativement correctes, elles aident l'algorithme à converger plus rapidement vers une bonne solution, même avec des données limitées. GDA atteint son erreur asymptotique (qui est plus élevée) beaucoup plus vite que la régression logistique.18

Ce phénomène est une manifestation directe du **compromis biais-variance**. GDA a un biais potentiellement plus élevé (si l'hypothèse gaussienne est fausse) mais une variance plus faible (ses paramètres sont estimés de manière stable, même avec peu de données). La régression logistique a un biais plus faible (elle est plus flexible) mais peut avoir une variance plus élevée (avec peu de données, l'estimation de ses paramètres peut être instable et mener au sur-apprentissage). Pour de petits jeux de données, l'erreur est souvent dominée par la variance (avantage GDA), tandis que pour de grands jeux de données, elle est dominée par le biais (avantage Régression Logistique).

Le tableau suivant synthétise cette comparaison.

| Caractéristique | Analyse Discriminante Gaussienne (GDA/LDA) | Régression Logistique |
| :---- | :---- | :---- |
| **Objectif de modélisation** | Probabilité jointe $P(x, y) \= P(x | y)P(y)$ |
| **Hypothèses sur les données** | $P(x | y)$ est Gaussienne, P(y) est Bernoulli |
| **Force des hypothèses** | Forte et spécifique | Faible et plus générale |
| **Frontière de décision** | Linéaire (si Σ partagée) ou Quadratique | Linéaire |
| **Estimation des paramètres** | EMV sur la vraisemblance jointe (calculs directs) | EMV sur la vraisemblance conditionnelle (itératif, e.g., descente de gradient) |
| **Efficacité en données** | Très efficace si les hypothèses sont (approximativement) vraies | Moins efficace, nécessite plus de données pour converger |
| **Erreur Asymptotique** | Potentiellement plus élevée (biais du modèle) | Plus faible |
| **Vitesse de Convergence** | Rapide | Plus lente |
| **Robustesse** | Sensible aux violations d'hypothèses (e.g., non-normalité, outliers) | Robuste aux hypothèses sur la distribution de $P(x |
| **Données non-étiquetées** | Peut les utiliser (dans un cadre semi-supervisé) | Ne peut pas les utiliser directement |

## **Partie III : Modèles Génératifs pour Données Discrètes : Naive Bayes**

Lorsque les caractéristiques x sont discrètes, comme des mots dans un texte, le modèle GDA n'est plus approprié. Nous nous tournons alors vers un autre algorithme génératif classique : le classifieur de Naive Bayes (ou "Bayésien naïf").

### **3.1. Application à la Classification de Texte : Filtrage de Spam**

L'une des applications les plus célèbres de Naive Bayes est la classification de documents, comme le filtrage d'e-mails indésirables (spam).3 Le premier défi est de convertir un document textuel en un vecteur de caractéristiques numériques.3

L'approche la plus courante est le modèle **"sac de mots" (Bag-of-Words)**.21 La procédure est la suivante :

1. **Construire un vocabulaire** : On parcourt l'ensemble des documents d'entraînement pour créer une liste de tous les mots uniques. En pratique, pour des raisons de performance et pour éliminer les mots rares, on se limite souvent aux N mots les plus fréquents, par exemple N=10,000.3  
2. **Vectorisation** : Chaque document est ensuite représenté par un vecteur x de taille N. La valeur de la j-ème composante, xj​, représente le j-ème mot du vocabulaire. La manière dont cette représentation est faite dépend du "modèle d'événement" choisi (voir section 3.3).

Avant cette étape, un **prétraitement du texte** est généralement effectué, incluant la conversion en minuscules, la suppression de la ponctuation, le retrait des "mots vides" (stop words) comme "le", "un", "de", et la racinisation (stemming ou lemmatization) pour regrouper les différentes formes d'un même mot (ex: "apprendre", "appris" \-\> "apprendr").21

### **3.2. L'Algorithme Naive Bayes et son Hypothèse Fondamentale**

Une fois les documents vectorisés, nous devons modéliser P(x∣y). Pour un vocabulaire de 10,000 mots et une représentation binaire, le vecteur x peut prendre 210,000 valeurs possibles. Modéliser une distribution sur un si grand nombre de résultats est infaisable.3

C'est ici qu'intervient l'**hypothèse "naïve" d'indépendance conditionnelle**. On suppose que, sachant la classe d'un document (spam ou non-spam), la présence d'un mot est indépendante de la présence de tous les autres mots.21 Mathématiquement, cela se traduit par :

P(x1​,…,xN​∣y)=j=1∏N​P(xj​∣y)  
Cette hypothèse est manifestement fausse dans la réalité du langage naturel (par exemple, les mots "Maison" et "Blanche" sont fortement corrélés). Cependant, l'algorithme fonctionne étonnamment bien en pratique.21 La raison est que pour la classification, il n'est pas nécessaire d'estimer les probabilités avec une précision absolue. Il suffit que la classe correcte obtienne le score de probabilité le plus élevé. L'hypothèse naïve, bien qu'incorrecte, simplifie radicalement le problème d'estimation, le rendant traitable et robuste au fléau de la dimensionnalité, tout en préservant souvent l'ordre relatif des probabilités des classes.

### **3.3. Modèles d'Événements pour le Texte : Bernoulli vs. Multinomial**

La manière de calculer P(xj​∣y) dépend du modèle d'événement choisi. Les deux plus courants sont :

* **Modèle de Bernoulli Multivarié** : Ce modèle se concentre sur la **présence ou l'absence** de mots. Le vecteur de caractéristiques x est binaire : xj​=1 si le mot j du vocabulaire apparaît dans le document, et xj​=0 sinon.3 Le paramètre à estimer est  
  ϕj∣y​=P(xj​=1∣y), la probabilité que le mot j soit présent dans un document de la classe y. Ce modèle ne tient pas compte du nombre d'occurrences d'un mot.  
* **Modèle Multinomial** : Ce modèle, souvent plus performant pour la classification de texte, prend en compte la **fréquence** des mots.21 Il considère un document comme une séquence de tirages de mots d'une distribution multinomiale spécifique à la classe. Le paramètre à estimer est  
  ϕj∣y​=P(motj​∣y), la probabilité qu'un mot tiré au hasard d'un document de la classe y soit le mot j. Le vecteur x peut ici représenter les comptes de chaque mot.

### **3.4. Entraînement du Modèle Naive Bayes**

L'entraînement consiste à estimer les paramètres du modèle par maximum de vraisemblance à partir du jeu de données d'entraînement.

* Estimation des Priors : Le prior de classe, P(y), est simplement la proportion de documents de cette classe :

  P^(y=c)=Nombre total de documentsNombre de documents de la classe c​  
* **Estimation des Probabilités Conditionnelles** :  
  * Pour le modèle multinomial, ϕj∣y=c​ est estimé par :  
    $$ \\hat{\\phi}\_{j|y=c} \= \\frac{\\text{Nombre total d'occurrences du mot } j \\text{ dans les documents de classe } c}{\\text{Nombre total de mots dans tous les documents de classe } c} $$  
  * Pour le modèle de Bernoulli, ϕj∣y=c​ est estimé par :  
    $$ \\hat{\\phi}\_{j|y=c} \= \\frac{\\text{Nombre de documents de classe } c \\text{ contenant le mot } j}{\\text{Nombre total de documents de classe } c} $$

#### **Le Problème du Zéro et le Lissage de Laplace**

Un problème majeur se pose si un mot présent dans un document de test n'a jamais été vu dans une certaine classe pendant l'entraînement. Sa probabilité conditionnelle estimée sera de zéro. À cause de la multiplication dans la formule de Naive Bayes, la probabilité *a posteriori* totale pour cette classe deviendra nulle, quelle que soit la pertinence des autres mots.24

La solution est le **lissage (smoothing)**. Le plus simple est le **lissage de Laplace** (ou "add-one smoothing"), qui consiste à prétendre avoir vu chaque mot du vocabulaire une fois de plus qu'en réalité. Pour le modèle multinomial, la formule devient 22 :

$$\\hat{\\phi}\_{j|y=c} \= \\frac{\\text{count}(\\text{mot}\_j, c) \+ \\alpha}{\\text{total\_mots}(c) \+ \\alpha \\cdot |V|}$$  
où ∣V∣ est la taille du vocabulaire et α=1 pour le lissage de Laplace. Cela garantit qu'aucune probabilité n'est jamais nulle.

### **3.5. Implémentation de Naive Bayes en Python et Pseudo-code**

L'implémentation est efficace car elle repose principalement sur des comptages. Pour éviter les problèmes de sous-dépassement de capacité (underflow) dus à la multiplication de nombreuses petites probabilités, on travaille avec les logarithmes des probabilités, transformant les produits en sommes.24

#### **Pseudo-code (Modèle Multinomial)**

fonction fit(documents, labels, alpha=1):  
    classes = classes uniques dans labels  
    priors = {}  
    comptes_mots_par_classe = {c: {} pour c dans classes}  
    total\_mots\_par\_classe \= {c: 0 pour c dans classes}  
    vocabulaire \= set()

    // Calculer les priors et compter les mots  
    pour chaque classe c dans classes:  
        docs\_de\_c \= documents où label est c  
        priors\[c\] \= len(docs\_de\_c) / len(documents)  
        pour chaque doc dans docs\_de\_c:  
            pour chaque mot dans doc:  
                comptes\_mots\_par\_classe\[c\]\[mot\] \= comptes\_mots\_par\_classe\[c\].get(mot, 0\) \+ 1  
                total\_mots\_par\_classe\[c\] \+= 1  
                vocabulaire.add(mot)  
      
    taille\_vocab \= len(vocabulaire)  
      
    // Calculer les probabilités conditionnelles avec lissage  
    probs\_cond \= {c: {} pour c dans classes}  
    pour chaque classe c dans classes:  
        dénominateur \= total\_mots\_par\_classe\[c\] \+ alpha \* taille\_vocab  
        pour chaque mot dans vocabulaire:  
            numérateur \= comptes\_mots\_par\_classe\[c\].get(mot, 0\) \+ alpha  
            probs\_cond\[c\]\[mot\] \= numérateur / dénominateur  
              
    retourner (priors, probs\_cond, vocabulaire)

fonction predict(document, params):  
    priors, probs\_cond, vocabulaire \= params  
    scores \= {}  
      
    pour chaque classe c dans priors:  
        scores\[c\] \= log(priors\[c\])  
        pour chaque mot dans document:  
            si mot est dans vocabulaire:  
                scores\[c\] \+= log(probs\_cond\[c\]\[mot\])  
      
    retourner la classe avec le score maximum

#### **Implémentation Python**

```py

import numpy as np  
from collections import defaultdict

class MultinomialNB:  
    """  
    Implémentation du classifieur Naive Bayes Multinomial pour la classification de texte.  
    """  
    def __init__(self, alpha=1.0):  
        self.alpha = alpha  # Paramètre de lissage  
        self.priors = {}  
        self.cond_probs = {}  
        self.vocab = set()

    def fit(self, X_train, y_train):  
        """  
        Entraîne le modèle.  
          
        Args:  
            X_train (list of list of str): Documents tokenisés.  
            y_train (list or np.array): Étiquettes correspondantes.  
        """  
        n_samples = len(X_train)  
        classes, counts = np.unique(y_train, return_counts=True)  
          
        # Calculer les priors  
        self.priors = {c: count / n_samples for c, count in zip(classes, counts)}  
          
        # Compter les mots et construire le vocabulaire  
        word_counts_per_class = {c: defaultdict(int) for c in classes}  
        total_words_per_class = {c: 0 for c in classes}  
          
        for text, label in zip(X_train, y_train):  
            for word in text:  
                self.vocab.add(word)  
                word_counts_per_class\[label][word] += 1  
                total_words_per_class[label] += 1  
          
        # Calculer les probabilités conditionnelles avec lissage de Laplace  
        vocab_size = len(self.vocab)  
        for c in classes:  
            self.cond_probs[c] = defaultdict(lambda: self.alpha / (total_words_per_class[c] + self.alpha * vocab_size))  
            denominator = total_words_per_class[c] + self.alpha * vocab_size  
            for word, count in word_counts_per_class[c].items():  
                self.cond_probs[c][word] = (count + self.alpha) / denominator

    def predict(self, X_test):  
        """  
        Prédit les étiquettes pour de nouveaux documents.  
        """  
        predictions = []
        for text in X_test:  
            scores = {c: np.log(prior) for c, prior in self.priors.items()}  
            for word in text:  
                for c in self.priors.keys():  
                    scores[c] += np.log(self.cond_probs[c][word])  
              
            predictions.append(max(scores, key=scores.get))  
        return predictions

# Exemple d'utilisation  
# X_train_spam = [["buy", "cheap", "viagra"], ["free", "money", "now"]]  
# y_train_spam = ["spam", "spam"]  
# X_train_ham = [["meeting", "tomorrow", "at", "noon"], ["project", "update", "and", "review"]]  
# y_train_ham = ["ham", "ham"]  
# X_train = X_train_spam + X_train_ham  
# y_train = y_train_spam + y_train_ham  
#   
# nb = MultinomialNB()  
# nb.fit(X_train, y_train)  
#   
# X_test = [["free", "viagra", "and", "cheap", "money"]]  
# prediction = nb.predict(X_test)  
# print(f"Prédiction pour '{X_test}': {prediction}") # Devrait prédire 'spam'
```
## **Conclusion : Synthèse et Perspectives**

Ce cours a exploré les deux paradigmes dominants de la classification probabiliste : les modèles génératifs et discriminatifs. Nous avons vu que leur différence fondamentale réside dans leur objectif de modélisation : P(x,y) pour les génératifs, et P(y∣x) pour les discriminatifs.

Cette distinction mène à un compromis fondamental entre la force des hypothèses, l'efficacité en données et la robustesse.

* Les **modèles génératifs**, comme GDA et Naive Bayes, font des hypothèses fortes sur la distribution des données. Quand ces hypothèses sont proches de la réalité, ces modèles peuvent être extrêmement efficaces, en particulier avec de petits ensembles de données, car les hypothèses agissent comme une forme de connaissance *a priori* qui guide l'apprentissage.19  
* Les **modèles discriminatifs**, comme la régression logistique, font des hypothèses plus faibles et se concentrent uniquement sur la tâche de séparation. Cela les rend plus robustes aux violations des hypothèses sur la distribution des données et leur confère généralement une meilleure performance asymptotique avec de grandes quantités de données.

Historiquement, ces modèles ont joué un rôle fondateur. L'Analyse Discriminante Linéaire, formalisée par Fisher en 1936, a posé les bases de la classification statistique.10 Naive Bayes, malgré sa simplicité, s'est avéré être un outil remarquablement efficace pour les débuts du traitement du langage naturel, notamment pour le filtrage de spam, où il est encore parfois utilisé comme une solide ligne de base.

La dichotomie génératif/discriminatif reste au cœur de l'apprentissage automatique moderne, bien au-delà de ces modèles classiques. Les modèles génératifs contemporains, tels que les **Réseaux Antagonistes Génératifs (GANs)**, ont révolutionné le domaine en apprenant non pas une distribution statique, mais le processus de génération lui-même.26 Dans un GAN, un modèle

**Générateur** apprend à créer des données synthétiques (par exemple, des images de visages) tandis qu'un modèle **Discriminateur** apprend à distinguer les vraies données des fausses. Les deux modèles s'entraînent dans un jeu à somme nulle, le Générateur devenant de plus en plus apte à créer des données réalistes pour tromper le Discriminateur.26 Ce cadre illustre de manière spectaculaire comment les concepts de génération et de discrimination, loin d'être opposés, peuvent être combinés pour atteindre des performances de pointe, ouvrant la voie à des applications allant de la création artistique à la simulation scientifique.

## **Références Académiques**

* Bishop, C. M., & Lasserre, J. (2007). Generative or Discriminative? getting the best of both worlds. *Bayesian statistics, 8*, 3-23.  
* Fisher, R. A. (1936). The use of multiple measurements in taxonomic problems. *Annals of Eugenics, 7*(2), 179-188. 10  
* Goodfellow, I., Pouget-Abadie, J., Mirza, M., Xu, B., Warde-Farley, D., Ozair, S., Courville, A., & Bengio, Y. (2014). Generative adversarial nets. *Advances in neural information processing systems, 27*. 26  
* Ng, A. Y., & Jordan, M. I. (2002). On Discriminative vs. Generative classifiers: A comparison of logistic regression and naive Bayes. *Advances in neural information processing systems, 14*. 18  
* Raschka, S. (2014). Naive Bayes and Text Classification I \- Introduction and Theory. *arXiv preprint arXiv:1410.5329*. 21

#### **Sources des citations**

1. Generative or Discriminative? Getting the Best of Both Worlds \- Microsoft, consulté le juillet 31, 2025, [https://www.microsoft.com/en-us/research/wp-content/uploads/2016/05/Bishop-Valencia-07.pdf](https://www.microsoft.com/en-us/research/wp-content/uploads/2016/05/Bishop-Valencia-07.pdf)  
2. Generative VS Discriminative Models | by Prathap Manohar Joshi \- Medium, consulté le juillet 31, 2025, [https://medium.com/@mlengineer/generative-and-discriminative-models-af5637a66a3](https://medium.com/@mlengineer/generative-and-discriminative-models-af5637a66a3)  
3. transcription.txt  
4. Generative Learning algorithms, consulté le juillet 31, 2025, [https://see.stanford.edu/materials/aimlcs229/cs229-notes2.pdf](https://see.stanford.edu/materials/aimlcs229/cs229-notes2.pdf)  
5. Gaussian Discriminant Analysis from | by John michael rono \- Medium, consulté le juillet 31, 2025, [https://medium.com/@johnmichaelrono/gaussian-discriminant-analysis-from-655acef23e0a](https://medium.com/@johnmichaelrono/gaussian-discriminant-analysis-from-655acef23e0a)  
6. Gaussian Discriminant Analysis \- GeeksforGeeks, consulté le juillet 31, 2025, [https://www.geeksforgeeks.org/dsa/gaussian-discriminant-analysis/](https://www.geeksforgeeks.org/dsa/gaussian-discriminant-analysis/)  
7. CSC 311: Introduction to Machine Learning \- Lecture 8 \- Multivariate Gaussians, GDA \- University of Toronto, consulté le juillet 31, 2025, [https://www.cs.toronto.edu/\~rgrosse/courses/csc311\_f21/lectures/lec08.pdf](https://www.cs.toronto.edu/~rgrosse/courses/csc311_f21/lectures/lec08.pdf)  
8. Aman's AI Journal • CS229 • Gaussian Discriminant Analysis, consulté le juillet 31, 2025, [https://aman.ai/cs229/gda/](https://aman.ai/cs229/gda/)  
9. Lecture Notes on Gaussian Discriminant Analysis, Naive Bayes and EM Algorithm \- Feng Li, consulté le juillet 31, 2025, [https://funglee.github.io/ml/slides/Lecture5-NaiveBayes-Notes.pdf](https://funglee.github.io/ml/slides/Lecture5-NaiveBayes-Notes.pdf)  
10. Linear discriminant analysis \- Wikipedia, consulté le juillet 31, 2025, [https://en.wikipedia.org/wiki/Linear\_discriminant\_analysis](https://en.wikipedia.org/wiki/Linear_discriminant_analysis)  
11. What is a Gaussian Discriminant Analysis (GDA)? \- Cross Validated \- Stats Stackexchange, consulté le juillet 31, 2025, [https://stats.stackexchange.com/questions/80507/what-is-a-gaussian-discriminant-analysis-gda](https://stats.stackexchange.com/questions/80507/what-is-a-gaussian-discriminant-analysis-gda)  
12. Gaussian discriminant analysis \- UBC Computer Science, consulté le juillet 31, 2025, [https://www.cs.ubc.ca/\~schmidtm/Courses/540-W17/T5.html](https://www.cs.ubc.ca/~schmidtm/Courses/540-W17/T5.html)  
13. How to do the derivation of the MLE for Linear Discriminant Analysis \- Math Stack Exchange, consulté le juillet 31, 2025, [https://math.stackexchange.com/questions/4780149/how-to-do-the-derivation-of-the-mle-for-linear-discriminant-analysis](https://math.stackexchange.com/questions/4780149/how-to-do-the-derivation-of-the-mle-for-linear-discriminant-analysis)  
14. 7 Gaussian Discriminant Analysis, including QDA and LDA \- People @EECS, consulté le juillet 31, 2025, [https://people.eecs.berkeley.edu/\~jrs/189/lec/07.pdf](https://people.eecs.berkeley.edu/~jrs/189/lec/07.pdf)  
15. machine learning \- Gaussian Discriminant Analysis and sigmoid ..., consulté le juillet 31, 2025, [https://stats.stackexchange.com/questions/251069/gaussian-discriminant-analysis-and-sigmoid-function](https://stats.stackexchange.com/questions/251069/gaussian-discriminant-analysis-and-sigmoid-function)  
16. A Gentle Introduction to Gaussian Discriminant Analysis: Gaussian Naive Bayes, Linear Discriminant Analysis, and Quadratic Discriminant Analysis | by Yuki Shizuya | The Quantastic Journal | Medium, consulté le juillet 31, 2025, [https://medium.com/the-quantastic-journal/a-gentle-introduction-to-gaussian-discriminant-analysis-gaussian-naive-bayes-linear-discriminant-e54d54f89c5a](https://medium.com/the-quantastic-journal/a-gentle-introduction-to-gaussian-discriminant-analysis-gaussian-naive-bayes-linear-discriminant-e54d54f89c5a)  
17. Gaussian Discriminant Analysis and Logistic Regression | by Du Phan | Data & Climate, consulté le juillet 31, 2025, [https://medium.com/du-phan/gaussian-discriminant-analysis-and-logistic-regression-29a807c5b45](https://medium.com/du-phan/gaussian-discriminant-analysis-and-logistic-regression-29a807c5b45)  
18. On Discriminative vs. Generative Classifiers: A comparison of ... \- NIPS, consulté le juillet 31, 2025, [https://papers.nips.cc/paper/2020-on-discriminative-vs-generative-classifiers-a-comparison-of-logistic-regression-and-naive-bayes](https://papers.nips.cc/paper/2020-on-discriminative-vs-generative-classifiers-a-comparison-of-logistic-regression-and-naive-bayes)  
19. On Discriminative vs. Generative classifiers: A ... \- CIS UPenn, consulté le juillet 31, 2025, [https://www.cis.upenn.edu/\~danroth/Teaching/CS598-05/Papers/NG-Jor-NBvsDesc.pdf](https://www.cis.upenn.edu/~danroth/Teaching/CS598-05/Papers/NG-Jor-NBvsDesc.pdf)  
20. On Discriminative vs. Generative Classifiers: A comparison of logistic regression and naive Bayes \- ResearchGate, consulté le juillet 31, 2025, [https://www.researchgate.net/publication/2539230\_On\_Discriminative\_vs\_Generative\_Classifiers\_A\_comparison\_of\_logistic\_regression\_and\_naive\_Bayes](https://www.researchgate.net/publication/2539230_On_Discriminative_vs_Generative_Classifiers_A_comparison_of_logistic_regression_and_naive_Bayes)  
21. Naive Bayes and Text Classification I-Introduction and Theory, consulté le juillet 31, 2025, [https://arxiv.org/pdf/1410.5329](https://arxiv.org/pdf/1410.5329)  
22. 1.9. Naive Bayes \- Scikit-learn, consulté le juillet 31, 2025, [https://scikit-learn.org/stable/modules/naive\_bayes.html](https://scikit-learn.org/stable/modules/naive_bayes.html)  
23. Naive Bayes Classifiers for Text Classification | by Pınar Ersoy | TDS Archive \- Medium, consulté le juillet 31, 2025, [https://medium.com/data-science/naive-bayes-classifiers-for-text-classification-be0d133d35ba](https://medium.com/data-science/naive-bayes-classifiers-for-text-classification-be0d133d35ba)  
24. Naive Bayes text classification \- Stanford NLP Group, consulté le juillet 31, 2025, [https://nlp.stanford.edu/IR-book/html/htmledition/naive-bayes-text-classification-1.html](https://nlp.stanford.edu/IR-book/html/htmledition/naive-bayes-text-classification-1.html)  
25. Fisher, E.M. (1936) Linear Discriminant Analysis. Statistics & Discrete Methods of Data Sciences, 392, 1-5. \- References \- Scientific Research Publishing, consulté le juillet 31, 2025, [https://www.scirp.org/reference/referencespapers?referenceid=3062627](https://www.scirp.org/reference/referencespapers?referenceid=3062627)  
26. Generative Adversarial Nets \- NIPS, consulté le juillet 31, 2025, [http://papers.neurips.cc/paper/5423-generative-adversarial-nets.pdf](http://papers.neurips.cc/paper/5423-generative-adversarial-nets.pdf)  
27. On Discriminative vs. Generative Classifiers: A comparison of logistic regression and Naive Bayes | Andrew Ng, consulté le juillet 31, 2025, [https://www.andrewng.org/publications/on-discriminative-vs-generative-classifiers-a-comparison-of-logistic-regression-and-naive-bayes/](https://www.andrewng.org/publications/on-discriminative-vs-generative-classifiers-a-comparison-of-logistic-regression-and-naive-bayes/)  
28. Publications | Andrew Ng, consulté le juillet 31, 2025, [https://www.andrewng.org/publications/3/](https://www.andrewng.org/publications/3/)  
29. \[1410.5329\] Naive Bayes and Text Classification I \- Introduction and Theory \- arXiv, consulté le juillet 31, 2025, [https://arxiv.org/abs/1410.5329](https://arxiv.org/abs/1410.5329)  
30. Naive Bayes and Text Classification \- Sebastian Raschka, consulté le juillet 31, 2025, [https://sebastianraschka.com/Articles/2014\_naive\_bayes\_1.html](https://sebastianraschka.com/Articles/2014_naive_bayes_1.html)


# **Des Modèles Linéaires aux Modèles Linéaires Généralisés : Une Exploration Théorique et Pratique**

## **Chapitre 1 : Le Perceptron : Fondations Historiques et Interprétation Géométrique**

### **1.1 Introduction et Contexte Historique : L'Aube de l'Apprentissage Automatique**

L'algorithme du Perceptron, bien que rarement utilisé dans les applications modernes, occupe une place fondamentale dans l'histoire de l'intelligence artificielle et de l'apprentissage automatique. Pour comprendre sa signification, il est essentiel de le replacer dans son contexte d'origine, celui des années 1950 et 1960, une période d'optimisme effervescent où la perspective de créer des machines pensantes semblait à portée de main.

Le Perceptron a été inventé par le psychologue Frank Rosenblatt au Cornell Aeronautical Laboratory en 1957.2 Son ambition n'était pas simplement de concevoir un algorithme de classification, mais de modéliser les processus de perception et d'apprentissage du cerveau humain. Cette ambition est clairement énoncée dans le titre de son article fondateur de 1958 : "The Perceptron: A Probabilistic Model for Information Storage and Organization in the Brain".3 Rosenblatt a simulé son modèle sur un ordinateur IBM 704 avant d'obtenir un financement substantiel de la part d'organismes gouvernementaux américains, notamment l'Office of Naval Research, pour construire une machine dédiée, le "Mark I Perceptron".2 La démonstration publique de cette machine le 23 juin 1960 a suscité un immense intérêt médiatique, avec des articles dans des publications comme

*The New York Times* rapportant que la marine américaine s'attendait à ce que le Perceptron soit "l'embryon d'un ordinateur électronique capable de marcher, parler, voir, écrire, se reproduire et être conscient de son existence".2

Cependant, cette vague d'enthousiasme a finalement conduit à une profonde désillusion. Les affirmations audacieuses de Rosenblatt ont placé la barre des attentes à un niveau extraordinairement élevé.2 La publication en 1969 du livre

*Perceptrons* par Marvin Minsky et Seymour Papert, deux figures éminentes du MIT, a porté un coup fatal à cette effervescence.7 Dans cet ouvrage, Minsky et Papert ont fourni une analyse mathématique rigoureuse des capacités et, plus important encore, des limites fondamentales du Perceptron. Ils ont prouvé que le modèle, dans sa forme la plus simple, était incapable de résoudre des problèmes qui ne sont pas linéairement séparables, le cas d'école étant la fonction logique XOR (OU exclusif).1 Cette démonstration a révélé un fossé béant entre les capacités réelles de l'algorithme et les ambitions affichées. La conséquence fut une perte de confiance généralisée dans l'approche connexionniste, entraînant une réduction drastique des financements pour ce type de recherche. Cet événement est aujourd'hui largement considéré comme l'un des catalyseurs du premier "hiver de l'IA", une période de stagnation pour le domaine.9 Cette trajectoire historique offre une leçon cruciale sur la dynamique entre l'enthousiasme scientifique, la rigueur théorique et le progrès d'un champ de recherche.

Une source de confusion terminologique mérite d'être clarifiée. Rosenblatt a qualifié son modèle de "probabiliste".3 Cependant, comme nous le verrons, l'une des critiques modernes du Perceptron est précisément son absence d'interprétation probabiliste au sens où nous l'entendons aujourd'hui. Cette apparente contradiction s'explique par une évolution de la terminologie. Pour Rosenblatt, le caractère "probabiliste" du système résidait dans son architecture (des connexions initialement aléatoires entre les unités sensorielles et les unités d'association, pour imiter le cerveau) et dans son processus d'apprentissage (apprendre à partir d'un échantillon aléatoire d'exemples).2 En revanche, l'apprentissage automatique moderne qualifie un modèle de "probabiliste" si sa sortie peut être interprétée comme une distribution de probabilité conditionnelle, telle que

P(y∣x). Le Perceptron ne produit pas une telle sortie ; sa prédiction est déterministe. Cette distinction est essentielle pour comprendre pourquoi des modèles comme la régression logistique, qui fournissent une sortie probabiliste, ont finalement supplanté le Perceptron.

### **1.2 Formalisme de l'Algorithme du Perceptron**

L'algorithme du Perceptron est un classifieur binaire linéaire. Il prend en entrée un vecteur de caractéristiques x∈Rn et a pour objectif de prédire une sortie binaire y∈{0,1}.

#### **Fonction d'Activation (Seuil Dur)**

Le cœur du Perceptron est sa fonction d'activation, notée g(z). Il s'agit d'une fonction de Heaviside, ou fonction de seuil "dur" ("hard threshold"). Elle est définie comme suit :

g(z)={10​si z≥0si z<0​

Cette fonction contraste fortement avec la fonction sigmoïde utilisée en régression logistique, g(z)=1/(1+e−z), qui est une version "douce" ("soft") de ce seuil. La sigmoïde produit une sortie continue entre 0 et 1, interprétable comme une probabilité, tandis que la fonction du Perceptron produit une décision binaire et non continue.

#### **Fonction d'Hypothèse**

La fonction d'hypothèse, hθ​(x), qui formule la prédiction du modèle, est construite en appliquant la fonction d'activation à une combinaison linéaire des entrées. Cette combinaison linéaire est le produit scalaire du vecteur de paramètres θ∈Rn et du vecteur d'entrée x.

hθ​(x)=g(θTx)

Ainsi, la prédiction hθ​(x) sera 1 si θTx≥0 et 0 sinon.

#### **Règle de Mise à Jour**

L'apprentissage dans le Perceptron se fait en ajustant le vecteur de paramètres θ. L'algorithme traite les exemples d'entraînement un par un (apprentissage en ligne). Pour chaque exemple (xi​,yi​), si la prédiction est correcte (hθ​(xi​)=yi​), aucun changement n'est apporté à θ. Si la prédiction est incorrecte, les paramètres sont mis à jour selon la règle suivante pour chaque composante j du vecteur θ :

θj​:=θj​+α(yi​−hθ​(xi​))xij​

où α est le taux d'apprentissage (un petit scalaire positif) et xij​ est la j-ème caractéristique de l'exemple xi​.  
Le terme d'erreur (yi​−hθ​(xi​)) est un scalaire qui ne peut prendre que trois valeurs :

* **0** si la prédiction est correcte. Dans ce cas, la mise à jour est nulle.  
* **+1** si l'étiquette réelle est yi​=1 et que le modèle a prédit hθ​(xi​)=0. La règle devient θ:=θ+αxi​.  
* **-1** si l'étiquette réelle est yi​=0 et que le modèle a prédit hθ​(xi​)=1. La règle devient θ:=θ−αxi​.

Cette règle de mise à jour est remarquablement simple et intuitive : si un exemple positif est classé comme négatif, on "ajoute" une fraction de ce vecteur d'exemple aux poids pour les rendre plus similaires. Inversement, si un exemple négatif est classé comme positif, on "soustrait" une fraction du vecteur d'exemple des poids pour les rendre moins similaires.

### **1.3 Implémentation et Pseudocode**

Pour solidifier la compréhension de l'algorithme, il est utile de le formaliser à travers un pseudocode et une implémentation pratique en Python.

**Tableau 1.1 : Pseudocode de l'Algorithme du Perceptron**

| Étape | Description |
| :---- | :---- |
| **1. Initialisation** | Initialiser le vecteur de poids θ à zéro ou à de petites valeurs aléatoires. Définir le taux d'apprentissage α et le nombre d'époques (itérations sur l'ensemble des données). |
| **2. Apprentissage** | Pour chaque époque : |
|  |      Pour chaque exemple d'entraînement (xi​,yi​) : |
|  |          a. **Activation :** Calculer la sortie linéaire zi​=θTxi​. |
|  |          b. **Prédiction :** Appliquer la fonction de seuil pour obtenir la prédiction hθ​(xi​)=g(zi​). |
|  |          c. **Mise à jour :** Calculer l'erreur ei​=yi​−hθ​(xi​). Mettre à jour le vecteur de poids : θ:=θ+αei​xi​. |
| **3. Prédiction** | Pour un nouvel exemple xnew​ : |
|  |      Calculer znew​=θTxnew​. |
|  |      Retourner la prédiction hθ​(xnew​)=g(znew​). |

Source : Basé sur la logique de l'algorithme décrite dans.2

#### **Implémentation Python**

Voici une implémentation en Python de l'algorithme du Perceptron sous forme de classe, en suivant la logique du pseudocode.

```Python

import numpy as np

class Perceptron:  
    """  
    Implémentation de l'algorithme du Perceptron.

    Paramètres  
    ----------  
    learning_rate : float  
        Le taux d'apprentissage (entre 0.0 et 1.0).  
    n_epochs : int  
        Le nombre de passes sur l'ensemble des données d'entraînement (époques).  
    random_state : int  
        Graine pour le générateur de nombres aléatoires pour l'initialisation des poids.  
    """  
    def __init__(self, learning_rate=0.01, n_epochs=50, random_state=1):  
        self.learning_rate = learning_rate  
        self.n_epochs = n_epochs  
        self.random_state = random_state  
        self.weights = None  
        self.bias = None

    def fit(self, X, y):  
        """  
        Ajuste le modèle aux données d'entraînement.

        Paramètres  
        ----------  
        X : array-like, shape = [n_samples, n_features]  
            Vecteurs d'entraînement, où n_samples est le nombre d'échantillons et  
            n_features est le nombre de caractéristiques.  
        y : array-like, shape = [n_samples]  
            Étiquettes cibles (0 ou 1).

        Retourne  
        -------  
        self : object  
        """  
        # Initialisation des poids et du biais  
        rgen = np.random.RandomState(self.random_state)  
        self.weights = rgen.normal(loc=0.0, scale=0.01, size=X.shape)  
        # Le biais est souvent traité comme le poids d'une caractéristique d'entrée qui est toujours 1  
        self.bias = 0.0

        for _ in range(self.n_epochs):  
            for xi, target in zip(X, y):  
                # Calcul de la prédiction  
                prediction = self.predict(xi)  
                  
                # Calcul de l'erreur et mise à jour des poids  
                # La règle de mise à jour est : delta_w = learning_rate * (target - prediction) * xi  
                update = self.learning_rate * (target - prediction)  
                self.weights += update * xi  
                self.bias += update  
        return self

    def net_input(self, X):  
        """Calcule l'entrée nette (produit scalaire)"""  
        return np.dot(X, self.weights) + self.bias

    def predict(self, X):  
        """  
        Retourne la prédiction de classe après l'application de la fonction de seuil.  
          
        Paramètres  
        ----------  
        X : array-like, shape = [n_samples, n_features]  
            Vecteurs d'entrée.

        Retourne  
        -------  
        class_label : int  
            Prédiction de classe (0 ou 1).  
        """  
        # g(z) = 1 si z >= 0, sinon 0  
        return np.where(self.net_input(X) >= 0.0, 1, 0)
```

### **1.4 Interprétation Géométrique de l'Apprentissage**

La règle de mise à jour du Perceptron possède une interprétation géométrique très élégante qui aide à visualiser le processus d'apprentissage. Dans l'espace des caractéristiques, l'équation θTx=0 définit un hyperplan. Cet hyperplan est la **frontière de décision** du modèle : il sépare l'espace en deux régions. D'un côté, où θTx>0, le modèle prédit la classe 1. De l'autre, où θTx<0, il prédit la classe 0.

Le vecteur de poids θ a une relation géométrique directe avec cet hyperplan : il est **normal** (perpendiculaire) à la frontière de décision et pointe dans la direction de la région de la classe 1.

Considérons maintenant ce qui se passe lors d'une mise à jour. Supposons qu'un exemple xi​ appartenant à la classe 1 (yi​=1) soit mal classé. Cela signifie que xi​ se trouve du mauvais côté de la frontière, dans la région où les prédictions sont 0. La règle de mise à jour est θnouveau​=θancien​+αxi​. Géométriquement, cela signifie que nous déplaçons le vecteur θ pour le rapprocher du vecteur xi​. Comme la frontière de décision doit rester perpendiculaire à θ, ce déplacement de θ provoque un **pivotement** de la frontière de décision. Le nouvel hyperplan est positionné de telle sorte que l'exemple xi​ a plus de chances d'être du bon côté.

Inversement, si un exemple de la classe 0 est mal classé, on soustrait une fraction de son vecteur de θ (θnouveau​=θancien​−αxi​), ce qui éloigne θ de xi​ et fait pivoter la frontière dans l'autre sens.

L'apprentissage est donc un processus itératif de réajustement de cet hyperplan, guidé par les erreurs de classification, jusqu'à ce qu'une position soit trouvée où (idéalement) tous les points sont correctement classés.

### **1.5 Limites du Perceptron et le Théorème de Convergence**

La simplicité du Perceptron est à la fois sa force et sa faiblesse. Sa principale force est encapsulée dans le **théorème de convergence du Perceptron**. Ce théorème, prouvé par Rosenblatt, stipule que si l'ensemble de données d'entraînement est **linéairement séparable** (c'est-à-dire qu'il existe au moins un hyperplan qui peut parfaitement séparer les classes), alors l'algorithme d'apprentissage du Perceptron est **garanti de converger** et de trouver un tel hyperplan en un nombre fini d'itérations.11

Cependant, cette garantie est conditionnée à une hypothèse très forte. La principale limitation du Perceptron est qu'il ne peut converger que si les données sont linéairement séparables. Si elles ne le sont pas, l'algorithme ne convergera jamais et les poids oscilleront indéfiniment. Le problème canonique illustrant cette limite est la fonction **XOR**.1 Considérons quatre points dans un plan 2D : (0,0) et (1,1) appartenant à la classe 0, et (0,1) et (1,0) appartenant à la classe 1. Il est visuellement évident qu'aucune ligne droite ne peut séparer ces deux classes. Le Perceptron, étant un classifieur linéaire, est fondamentalement incapable de résoudre ce problème simple.

Cette limitation, rigoureusement démontrée par Minsky et Papert, a mis en évidence que le Perceptron ne pouvait pas être un modèle universel de l'apprentissage.7 En résumé, les raisons pour lesquelles le Perceptron n'est plus largement utilisé en pratique sont doubles :

1. **Limitation à la séparabilité linéaire :** Il échoue sur des problèmes non linéaires simples, ce qui le rend inadapté à la complexité des données du monde réel.  
2. **Absence d'une sortie probabiliste :** Il produit une décision binaire (0 ou 1), mais ne fournit aucune mesure de confiance ou de probabilité associée à cette décision, ce qui est une fonctionnalité cruciale des modèles de classification modernes comme la régression logistique.

Ces limitations ont ouvert la voie au développement de modèles plus sophistiqués, capables de gérer la non-linéarité et de fournir des interprétations probabilistes, un chemin qui nous mènera naturellement aux familles exponentielles et aux modèles linéaires généralisés.

## **Chapitre 2 : Les Familles Exponentielles : Un Cadre Unificateur pour les Distributions de Probabilité**

### **2.1 Définition Formelle**

Avant de construire des modèles plus puissants, nous devons introduire un concept mathématique d'une importance capitale en statistique et en apprentissage automatique : la **famille exponentielle** de distributions de probabilité. Il s'agit d'une classe de distributions qui partagent une forme algébrique commune, ce qui leur confère des propriétés mathématiques particulièrement élégantes et utiles.1 L'étude de cette famille est essentielle car elle constitue le fondement statistique sur lequel reposent les modèles linéaires généralisés (GLM).

Le concept de famille exponentielle a été développé indépendamment par **E. J. G. Pitman**, **B. O. Koopman**, et **Georges Darmois** dans une série de publications fondamentales entre 1935 et 1936.12 Leurs travaux ont établi le cadre théorique qui continue d'influencer la statistique moderne.

Une distribution de probabilité appartient à la famille exponentielle si sa fonction de densité de probabilité (PDF, pour les variables continues) ou sa fonction de masse de probabilité (PMF, pour les variables discrètes) peut s'écrire sous la forme suivante 1 :

p(y;η)=b(y)exp(ηTT(y)−a(η))

### **2.2 Composants de la Famille Exponentielle**

Décortiquons les quatre composants de cette forme canonique :

* **y :** La variable aléatoire, représentant les données observées.  
* **η (Paramètre Naturel) :** Il s'agit du paramètre (potentiellement un vecteur) de la distribution. On l'appelle "naturel" car, sous cette paramétrisation, les propriétés mathématiques de la famille sont les plus directes. Il est souvent lié aux paramètres plus conventionnels de la distribution (comme la moyenne ou la variance) par une transformation.  
* **T(y) (Statistique Suffisante) :** C'est une fonction (potentiellement un vecteur de fonctions) de l'observation y. La statistique suffisante capture toute l'information contenue dans l'échantillon y qui est pertinente pour estimer le paramètre η. Pour de nombreuses distributions courantes, comme celles que nous étudierons, T(y) est simplement égal à y.  
* **a(η) (Fonction de Log-Partition) :** C'est une fonction scalaire du paramètre naturel η. Elle joue le rôle de normalisateur. La fonction de partition est ea(η), et elle garantit que la distribution s'intègre (ou se somme) à 1 sur l'ensemble de son support. C'est pourquoi a(η) est appelée la fonction de *log*-partition.  
* **b(y) (Mesure de Base) :** C'est une fonction de l'observation y uniquement, qui ne dépend pas de η. Elle représente une sorte de mesure sous-jacente sur les données.

La raison fondamentale pour laquelle nous nous intéressons aux familles exponentielles réside dans le **théorème de Pitman-Koopman-Darmois**.13 Ce théorème fondamental stipule que, sous certaines conditions de régularité, une famille de distributions paramétriques admet une

**statistique suffisante dont la dimension reste fixe** à mesure que la taille de l'échantillon augmente, **si et seulement si** cette famille est une famille exponentielle. Cela signifie que pour des données indépendantes et identiquement distribuées (i.i.d.) issues d'une famille exponentielle, nous pouvons résumer un ensemble de données arbitrairement grand en un seul vecteur de taille fixe, ∑i=1N​T(yi​), sans perdre la moindre information pertinente pour l'estimation du paramètre η. Cette propriété de "compression de données sans perte d'information" est la pierre angulaire qui rend l'inférence statistique et l'apprentissage pour ces modèles mathématiquement traitables et efficaces.

### **2.3 Démonstrations d'Appartenance à la Famille Exponentielle**

Pour démontrer qu'une distribution donnée appartient à la famille exponentielle, la méthode consiste à prendre sa PDF ou PMF standard et à la manipuler algébriquement pour la faire correspondre à la forme canonique. Illustrons ce processus avec deux exemples centraux : la distribution de Bernoulli et la distribution Gaussienne.

#### **Exemple 1 : La Distribution de Bernoulli**

La distribution de Bernoulli est utilisée pour modéliser une variable aléatoire binaire y∈{0,1}. Elle est paramétrée par ϕ=P(y=1). Sa PMF est :

p(y;ϕ)=ϕy(1−ϕ)1−y

Pour la mettre sous forme exponentielle, nous utilisons l'astuce consistant à prendre l'exponentielle du logarithme :  
p(y;ϕ)​=exp(log(ϕy(1−ϕ)1−y))=exp(ylog(ϕ)+(1−y)log(1−ϕ))=exp(ylog(ϕ)−ylog(1−ϕ)+log(1−ϕ))=exp(ylog(1−ϕϕ​)+log(1−ϕ))​  
En comparant cette expression à la forme canonique p(y;η)=b(y)exp(ηT(y)−a(η)), nous pouvons identifier chaque composant par correspondance 1 :

* **Paramètre naturel η :** η=log(1−ϕϕ​). Cette transformation est la fameuse fonction **logit**.  
* **Statistique suffisante T(y) :** T(y)=y.  
* Fonction de log-partition a(η) : Nous devons exprimer −log(1−ϕ) en fonction de η. En inversant la relation pour η, on trouve que ϕ=1+e−η1​ (la fonction sigmoïde). En substituant, on obtient :  
  a(η)=−log(1−ϕ)=−log(1−1+e−η1​)=−log(1+e−ηe−η​)=log(1+eη).  
* **Mesure de base b(y) :** b(y)=1.

Puisque nous avons pu réécrire la PMF de Bernoulli sous la forme canonique, nous avons prouvé qu'elle appartient à la famille exponentielle.

#### **Exemple 2 : La Distribution Gaussienne (Variance Unitaire)**

Considérons une distribution Gaussienne avec une moyenne μ et une variance fixée à 1. Sa PDF est :

p(y;μ)=2π​1​exp(−21​(y−μ)2)

Développons le carré dans l'exponentielle :  
p(y;μ)​=2π​1​exp(−21​y2+yμ−21​μ2)=(2π​1​exp(−2y2​))exp(yμ−2μ2​)​  
Par correspondance avec la forme canonique 1 :

* **Paramètre naturel η :** η=μ.  
* **Statistique suffisante T(y) :** T(y)=y.  
* **Fonction de log-partition a(η) :** a(η)=2μ2​=2η2​.  
* **Mesure de base b(y) :** b(y)=2π​1​exp(−2y2​).

La distribution Gaussienne (avec variance connue) appartient donc bien à la famille exponentielle. Il est également possible de montrer que la famille Gaussienne avec moyenne et variance inconnues appartient à une version vectorielle de la famille exponentielle, où η et T(y) sont des vecteurs.

**Tableau 2.1 : Composants de la Famille Exponentielle pour les Distributions Courantes**

| Distribution | Paramètre Canonique | Paramètre Naturel (η) | Statistique Suffisante (T(y)) | Log-Partition (a(η)) | Mesure de Base (b(y)) |
| :---- | :---- | :---- | :---- | :---- | :---- |
| **Bernoulli** | ϕ∈(0,1) | log(1−ϕϕ​) | y | log(1+eη) | 1 |
| **Gaussienne** | μ∈R | μ | y | 2η2​ | 2πσ2​1​e−2σ2y2​ |
| (variance σ2 fixée) |  |  |  |  |  |
| **Poisson** | λ>0 | log(λ) | y | eη | y!1​ |

### **2.4 Propriétés Mathématiques Fondamentales**

Les distributions de la famille exponentielle possèdent plusieurs propriétés remarquables qui simplifient grandement l'analyse statistique et l'optimisation.

#### **Concavité de la Vraisemblance**

Pour un ensemble de données i.i.d. y1​,…,yN​, la log-vraisemblance est :

ℓ(η)=logi=1∏N​p(yi​;η)=i=1∑N​(ηTT(yi​)−a(η)+logb(yi​))  
Pour déterminer la concavité, nous calculons la dérivée seconde par rapport à η. Il peut être démontré que la dérivée seconde de a(η) est la variance de T(y), qui est toujours non-négative. Par conséquent, la matrice Hessienne de ℓ(η) est −∇2a(η), qui est semi-définie négative. Cela signifie que la fonction de log-vraisemblance est **concave** par rapport au paramètre naturel η. Cette propriété est extrêmement importante : elle garantit que lorsque nous cherchons à maximiser la vraisemblance (Maximum Likelihood Estimation, MLE), il n'y a pas de maxima locaux ; tout maximum trouvé est le maximum global. Cela rend l'optimisation beaucoup plus simple et fiable.

#### **La Fonction de Log-Partition comme Générateur de Moments**

Une des propriétés les plus puissantes de la famille exponentielle est que la fonction de log-partition a(η) agit comme une "machine" à générer les moments de la statistique suffisante T(y).1 Alors que le calcul de la moyenne et de la variance nécessite normalement des intégrales, ici, il suffit de dériver.

Il peut être prouvé que l'espérance et la variance de la statistique suffisante T(y) sont données par les première et seconde dérivées de a(η) par rapport à η :

E=∇η​a(η)  
Var(T(y))=∇η2​a(η)

Cette relation n'est pas une coïncidence. Elle découle du fait que a(η) est la fonction génératrice des cumulants de la distribution de T(y). Un cumulant est une mesure statistique (le premier est la moyenne, le deuxième la variance, le troisième est lié à l'asymétrie, etc.). La fonction génératrice des moments de T(y) est M(t)=E. Il peut être montré que M(t)=exp(a(η+t)−a(η)). La fonction génératrice des cumulants est le logarithme de M(t), soit K(t)=a(η+t)−a(η). Les cumulants sont obtenus en dérivant K(t) par rapport à t et en évaluant à t=0. La première dérivée donne la moyenne, et la seconde donne la variance, ce qui mène directement aux identités ci-dessus. Cette propriété fait de a(η) bien plus qu'un simple facteur de normalisation ; c'est un objet mathématique central qui encode toute la structure des moments de la distribution.

## **Chapitre 3 : Les Modèles Linéaires Généralisés (GLM)**

### **3.1 Introduction : Un Pont entre les Modèles**

Les Modèles Linéaires Généralisés (GLM) représentent une synthèse brillante des concepts que nous avons explorés jusqu'à présent. Ils combinent la simplicité et l'interprétabilité des modèles linéaires (comme le Perceptron) avec la rigueur et la flexibilité statistique des familles exponentielles. Ce cadre unificateur a été introduit dans un article révolutionnaire de 1972, "Generalized Linear Models", par **John Nelder et Robert Wedderburn**.18 Leur travail a profondément transformé la statistique appliquée en fournissant une méthodologie cohérente pour modéliser une vaste gamme de types de données, bien au-delà des hypothèses restrictives de la régression linéaire classique.

### **3.2 Les Trois Hypothèses de Conception des GLM**

Un GLM est défini par un ensemble de trois hypothèses ou choix de conception qui relient les données d'entrée x à la variable de sortie (ou réponse) y.

1. **Hypothèse sur la Distribution :** La distribution conditionnelle de la variable de réponse y, étant donné x, appartient à la famille exponentielle, paramétrée par η.  
   y∣x;θ∼ExpFamily(η)  
   Ce choix est dicté par la nature de la variable de réponse. Par exemple, on choisira une distribution Gaussienne pour des données réelles continues (régression), une distribution de Bernoulli pour des données binaires (classification), ou une distribution de Poisson pour des données de comptage.1  
2. **Hypothèse sur la Linéarité :** Le paramètre naturel η de la distribution est une fonction linéaire des caractéristiques d'entrée x.  
   η=θTx  
   C'est le composant "linéaire" du modèle. Les paramètres θ sont les poids du modèle que nous cherchons à apprendre.  
3. **Hypothèse sur la Prédiction :** La prédiction du modèle, donnée par la fonction d'hypothèse hθ​(x), est l'espérance (la moyenne) de la variable de réponse y conditionnée par x.  
   hθ​(x)=E[y∣x;θ]

L'innovation fondamentale du cadre GLM réside dans le **découplage** des hypothèses sur la linéarité et la distribution.20 Avant les GLM, pour modéliser des données non-normales avec un modèle linéaire, la pratique courante était de transformer directement la variable de réponse (par exemple, en utilisant

log(y)) pour tenter de la rendre plus "normale" et de stabiliser sa variance. Cette approche pose des problèmes d'interprétation (car E[log(y)]=log(E[y])) et n'est souvent qu'une solution palliative.

Les GLM adoptent une approche beaucoup plus élégante. Au lieu de transformer les données, ils modélisent une **transformation de la moyenne des données**. La linéarité est supposée sur cette échelle transformée. Cette séparation des préoccupations — (1) une fonction pour assurer la linéarité, (2) une fonction pour modéliser la variance en fonction de la moyenne, et (3) une distribution d'erreur appropriée — est la contribution conceptuelle majeure de Nelder et Wedderburn.

### **3.3 Fonctions de Lien et de Réponse**

Pour connecter la moyenne de la distribution μ=E[y] au prédicteur linéaire η=θTx, les GLM utilisent une fonction de lien (link function), notée g.

η=g(μ)

La fonction inverse, g−1, est appelée la fonction de réponse (response function) et permet de revenir du prédicteur linéaire à la moyenne.

μ=g−1(η)

Un choix particulièrement important est la fonction de lien canonique. C'est la fonction de lien pour laquelle le paramètre naturel η est directement égal à la moyenne transformée, η=g(μ). L'utilisation de la fonction de lien canonique simplifie souvent les dérivations mathématiques et l'algorithme d'estimation.  
**Tableau 3.1 : Fonctions de Lien Canoniques et Modèles Résultants**

| Distribution de y | Lien Canonique g(μ) | Réponse μ=g−1(η) | Modèle GLM Résultant |
| :---- | :---- | :---- | :---- |
| **Gaussienne** | g(μ)=μ (Identité) | μ=η | Régression Linéaire |
| **Bernoulli** | g(μ)=log(1−μμ​) (Logit) | μ=1+e−η1​ (Sigmoïde) | Régression Logistique |
| **Poisson** | g(μ)=log(μ) (Log) | μ=eη | Régression de Poisson |

Source : Basé sur les principes des GLM décrits dans.1

### **3.4 Construction de Modèles Spécifiques : Études de Cas**

Le cadre GLM est une véritable "recette" pour construire des modèles. En choisissant une distribution de la famille exponentielle et une fonction de lien, nous pouvons dériver de nombreux algorithmes d'apprentissage classiques.

#### **Cas 1 : Régression Linéaire (Moindres Carrés Ordinaires)**

Pour dériver la régression linéaire, nous faisons les choix suivants :

1. **Distribution :** y∣x∼N(μ,σ2). La sortie est supposée suivre une distribution Gaussienne.  
2. **Fonction de lien :** Nous utilisons le lien canonique pour la Gaussienne, qui est la fonction identité : g(μ)=μ.

Avec ces choix, nous avons η=g(μ)=μ.  
Notre hypothèse de prédiction est hθ​(x)=E[y∣x]=μ.  
En combinant avec l'hypothèse de linéarité η=θTx, nous obtenons :

hθ​(x)=μ=η=θTx

Nous retrouvons exactement le modèle de la régression linéaire.

#### **Cas 2 : Régression Logistique**

Pour la régression logistique, qui modélise une sortie binaire :

1. **Distribution :** y∣x∼Bernoulli(ϕ). La sortie suit une distribution de Bernoulli.  
2. **Fonction de lien :** Nous utilisons le lien canonique pour la Bernoulli, qui est la fonction logit : g(ϕ)=log(1−ϕϕ​).

Ici, la moyenne est μ=E[y∣x]=ϕ. Le paramètre naturel est η=g(ϕ).  
L'hypothèse de prédiction est hθ​(x)=ϕ.  
La fonction de réponse est l'inverse du logit, qui est la fonction sigmoïde : ϕ=g−1(η)=1+e−η1​.  
En combinant avec l'hypothèse de linéarité η=θTx, nous obtenons la fonction d'hypothèse de la régression logistique :

hθ​(x)=1+e−θTx1​

Cet exemple est particulièrement éclairant : il montre que la fonction sigmoïde, qui peut sembler un choix arbitraire et ad hoc au premier abord, émerge en fait naturellement et de manière fondée des principes premiers du cadre GLM.

### **3.5 Apprentissage et la Règle de Mise à Jour Unifiée**

L'un des résultats les plus remarquables du cadre GLM est qu'il conduit à une règle de mise à jour par descente de gradient stochastique qui est identique en apparence pour tous les modèles de la famille, qu'il s'agisse de régression linéaire, logistique, de Poisson, etc.. Cette universalité n'est pas une coïncidence, mais une conséquence directe de la structure mathématique des familles exponentielles.

Pour apprendre les paramètres θ, nous utilisons l'estimation par maximum de vraisemblance (MLE). La log-vraisemblance pour une seule observation (x,y) est :

ℓ(θ)=logp(y∣x;θ)=logb(y)+ηTT(y)−a(η)

En supposant que T(y)=y et que η est un scalaire (pour simplifier), et en utilisant la relation η=θTx, nous avons :

ℓ(θ)=logb(y)+y(θTx)−a(θTx)

Nous devons calculer le gradient de ℓ(θ) par rapport à θj​ en utilisant la règle de la chaîne :  
∂θj​∂ℓ​=∂η∂ℓ​∂θj​∂η​  
Calculons chaque terme :

* ∂η∂ℓ​=y−a′(η). D'après les propriétés des familles exponentielles, nous savons que a′(η)=E[y∣η]=μ=hθ​(x). Donc, ∂η∂ℓ​=y−hθ​(x).  
* ∂θj​∂η​=∂θj​∂​(θTx)=xj​.

En combinant ces résultats, nous obtenons le gradient :

∇θ​ℓ(θ)=(y−hθ​(x))x

La règle de mise à jour pour la descente de gradient stochastique (ou plutôt l'ascension de gradient, puisque nous maximisons la vraisemblance) est donc :

θj​:=θj​+α(yi​−hθ​(xi​))xij​

Cette règle est universelle pour tout GLM utilisant son lien canonique. La seule chose qui change d'un modèle à l'autre est la définition de la fonction d'hypothèse hθ​(x), qui dépend de la fonction de lien choisie. C'est une démonstration puissante de l'élégance et de la puissance unificatrice du formalisme GLM.  
**Tableau 3.2 : Pseudocode de l'Apprentissage par Descente de Gradient Stochastique pour les GLM**

| Étape | Description |
| :---- | :---- |
| **1. Choix du Modèle** | Choisir une distribution de la famille exponentielle (ex: Bernoulli) et une fonction de lien g (ex: logit). Cela définit la fonction de réponse hθ​(x)=g−1(θTx). |
| **2. Initialisation** | Initialiser le vecteur de poids θ à zéro ou à de petites valeurs aléatoires. Définir le taux d'apprentissage α et le nombre d'époques. |
| **3. Apprentissage** | Pour chaque époque : |
|  |      Pour chaque exemple d'entraînement (xi​,yi​) : |
|  |          a. **Calcul du prédicteur linéaire :** ηi​=θTxi​. |
|  |          b. **Calcul de l'hypothèse :** hθ​(xi​)=g−1(ηi​). |
|  |          c. **Calcul du gradient :** ∇θ​ℓ=(yi​−hθ​(xi​))xi​. |
|  |          d. **Mise à jour :** θ:=θ+α∇θ​ℓ. |

### **3.6 Distinction des Trois Niveaux de Paramétrisation**

Il est crucial de bien distinguer les trois espaces de paramètres qui interagissent dans un GLM, car leur confusion peut être une source d'erreurs conceptuelles.

1. **Paramètres du Modèle (θ) :** Ce sont les poids du modèle linéaire. Ce sont les seules grandeurs que l'algorithme apprend directement via la descente de gradient. Ils vivent dans l'espace du modèle, Rn.  
2. **Paramètres Naturels (η) :** C'est la sortie du modèle linéaire, η=θTx. Ce n'est pas un paramètre appris, mais une valeur calculée qui sert de paramètre à la distribution de la famille exponentielle. Il vit dans l'espace des paramètres naturels.  
3. **Paramètres Canoniques (ϕ,μ,λ,…) :** Ce sont les paramètres traditionnels et souvent plus intuitifs de la distribution (la probabilité de succès pour une Bernoulli, la moyenne pour une Gaussienne, le taux pour une Poisson). Ils sont liés au paramètre naturel η via la fonction de réponse, par exemple μ=g−1(η).

Le processus d'un GLM peut être vu comme une cascade : les entrées x sont transformées en un paramètre naturel η par le modèle linéaire (paramétré par θ), et ce η définit à son tour une distribution de probabilité sur y dont la moyenne μ est la prédiction finale du modèle. L'apprentissage consiste à ajuster θ pour que cette prédiction soit la plus vraisemblable possible au vu des données.

## **Chapitre 4 : La Régression Softmax : Classification Multi-classes**

### **4.1 Le Problème de la Classification Multi-classes**

Jusqu'à présent, nous avons traité la classification binaire (Perceptron, Régression Logistique). Cependant, de nombreux problèmes du monde réel nécessitent de classer des données dans plus de deux catégories (par exemple, la reconnaissance de chiffres manuscrits de 0 à 9, le tri de documents en différentes rubriques, etc.). C'est le domaine de la **classification multi-classes**.

Dans ce contexte, la variable de sortie y peut prendre l'une des K valeurs discrètes, {1,2,…,K}. Pour représenter ces étiquettes de manière pratique pour les modèles mathématiques, on utilise couramment le **codage "one-hot"** (ou "one-of-K"). Une étiquette de classe k est représentée par un vecteur de dimension K contenant des zéros partout, sauf à la k-ème position qui contient un 1. Par exemple, pour K=4, la classe 3 serait représentée par le vecteur T.

### **4.2 Formalisme de la Régression Softmax**

La régression Softmax, également connue sous le nom de régression logistique multinomiale, est une généralisation de la régression logistique au cas multi-classes. Elle est un pilier de la classification moderne et constitue la couche de sortie de la plupart des réseaux de neurones pour ce type de tâche.

#### **Paramétrisation**

Contrairement à la régression logistique qui n'a qu'un seul vecteur de poids θ, la régression softmax maintient un vecteur de poids distinct θk​∈Rn pour **chaque classe** k∈{1,…,K}. On peut imaginer que chaque classe a son propre "expert" linéaire. Ces vecteurs de poids peuvent être regroupés dans une matrice de poids Θ de taille n×K.

#### **Calcul des Scores (Logits)**

Pour une entrée donnée x, le modèle calcule un score d'activation ou logit pour chaque classe k en effectuant un produit scalaire :

zk​=θkT​x

Ces scores sont des nombres réels non bornés. Le vecteur de tous ces scores, z=[z1​,z2​,…,zK​]T, réside dans ce qu'on appelle l'espace des logits. Un score plus élevé pour une classe k indique une plus grande confiance du modèle que l'entrée x appartient à cette classe.

#### **La Fonction Softmax**

Les logits ne sont pas des probabilités : ils ne sont pas contraints entre 0 et 1 et ne somment pas à 1. Pour les transformer en une distribution de probabilité valide sur les K classes, nous appliquons la **fonction softmax** 1 :

pk​=p^​(y=k∣x;Θ)=∑j=1K​exp(zj​)exp(zk​)​=∑j=1K​exp(θjT​x)exp(θkT​x)​  
La fonction softmax a deux propriétés essentielles :

1. **Positivité :** L'exponentiation garantit que toutes les probabilités pk​ sont positives.  
2. **Normalisation :** La division par la somme de toutes les exponentielles garantit que ∑k=1K​pk​=1.

Le résultat est un vecteur de probabilités [p1​,…,pK​]T qui représente la prédiction probabiliste du modèle.

Il est intéressant de noter que la fonction softmax est une généralisation directe de la fonction sigmoïde. Pour une classification binaire (K=2), la probabilité de la classe 1 est donnée par la sigmoïde σ(z)=1/(1+e−z). Si nous prenons une régression softmax à deux classes avec les logits z1​ et z2​, la probabilité de la classe 1 est exp(z1​)/(exp(z1​)+exp(z2​)). En divisant le numérateur et le dénominateur par exp(z1​), on obtient 1/(1+exp(z2​−z1​)). Si nous posons z=z1​−z2​ (ce qui est une contrainte courante pour assurer l'identifiabilité du modèle, équivalente à fixer un θk​ à zéro), nous retrouvons exactement la fonction sigmoïde. Ainsi, la régression logistique est un cas particulier de la régression softmax pour K=2.

### **4.3 Apprentissage avec la Perte d'Entropie Croisée**

Pour entraîner les paramètres Θ, nous avons besoin d'une fonction de coût (ou de perte) qui mesure l'écart entre la distribution de probabilité prédite p^​ et la distribution de probabilité réelle (cible) p. La fonction de perte de choix pour les problèmes de classification probabiliste est l'**entropie croisée** (cross-entropy).1

L'entropie croisée entre une distribution cible p et une distribution prédite p^​ est définie comme :

H(p,p^​)=−k=1∑K​pk​log(p^​k​)

Dans notre cas, la distribution cible p est le vecteur one-hot de la vraie classe. Si la vraie classe est c, alors pc​=1 et pk​=0 pour tout k=c. La somme se simplifie donc considérablement pour ne conserver que le terme de la classe correcte :  
Li​=H(p(i),p^​(i))=−k=1∑K​pk(i)​log(p^​k(i)​)=−1⋅log(p^​c(i)​)=−log(p^​c(i)​)  
où p^​c(i)​ est la probabilité que le modèle a prédite pour la classe correcte c de l'exemple i.

Cette formulation a une interprétation très profonde. Maximiser la probabilité de la classe correcte, p^​c(i)​, est équivalent à minimiser son logarithme négatif, −log(p^​c(i)​). Par conséquent, **minimiser la perte d'entropie croisée est mathématiquement équivalent à effectuer une Estimation par Maximum de Vraisemblance (MLE)** sur les paramètres du modèle. Ce lien ancre la méthode d'apprentissage dans des principes statistiques fondamentaux.

De plus, la combinaison de la fonction softmax et de la perte d'entropie croisée conduit à un gradient remarquablement simple et élégant. Le gradient de la perte Li​ par rapport au vecteur de logits z est :

∇z​Li​=p^​(i)−p(i)

C'est simplement la différence entre le vecteur de probabilités prédites et le vecteur one-hot de la vérité terrain. Cette forme simple rend la rétropropagation du gradient (backpropagation) dans les réseaux de neurones particulièrement efficace.24

### **4.4 Implémentation et Pseudocode**

Le processus d'apprentissage de la régression softmax peut être résumé par le pseudocode suivant.

**Tableau 4.1 : Pseudocode de la Régression Softmax avec Perte d'Entropie Croisée**

| Étape | Description | Expression Mathématique |
| :---- | :---- | :---- |
| **1. Calcul des Logits** | Pour un exemple xi​, calculer le score pour chaque classe k. | zk​=θkT​xi​ |
| **2. Calcul des Probabilités** | Appliquer la fonction softmax aux logits pour obtenir un vecteur de probabilités p^​(i). | p^​k(i)​=∑j=1K​exp(zj​)exp(zk​)​ |
| **3. Calcul de la Perte** | Calculer la perte d'entropie croisée en utilisant le vecteur one-hot de la vraie classe p(i). | Li​=−∑k=1K​pk(i)​log(p^​k(i)​) |
| **4. Calcul du Gradient** | Calculer le gradient de la perte par rapport aux poids θk​. | ∇θk​​Li​=(p^​k(i)​−pk(i)​)xi​ |
| **5. Mise à jour des Poids** | Mettre à jour les poids de chaque classe en utilisant la descente de gradient. | θk​:=θk​−α∇θk​​Li​ |

#### **Implémentation Python**

Voici une implémentation Python de la régression softmax.

```Python

import numpy as np

class SoftmaxRegression:  
    """  
    Implémentation de la régression Softmax pour la classification multi-classes.  
    """  
    def __init__(self, learning_rate=0.01, n_epochs=100, n_features=None, n_classes=None, random_state=1):  
        self.learning_rate = learning_rate  
        self.n_epochs = n_epochs  
        self.random_state = random_state  
        self.weights = None  
        self.bias = None  
        if n_features and n_classes:  
            self._initialize_weights(n_features, n_classes)

    def _initialize_weights(self, n_features, n_classes):  
        """Initialise les poids et les biais."""  
        rgen = np.random.RandomState(self.random_state)  
        # Poids : [n_features, n_classes]  
        self.weights = rgen.normal(loc=0.0, scale=0.01, size=(n_features, n_classes))  
        # Biais : [1, n_classes]  
        self.bias = np.zeros(shape=(1, n_classes))

    def _softmax(self, z):  
        """Calcule les probabilités softmax de manière numériquement stable."""  
        # Soustraire le max pour la stabilité numérique  
        exp_z = np.exp(z - np.max(z, axis=1, keepdims=True))  
        return exp_z / np.sum(exp_z, axis=1, keepdims=True)

    def fit(self, X, y):  
        """  
        Ajuste le modèle aux données d'entraînement.

        Paramètres  
        ----------  
        X : array-like, shape = [n_samples, n_features]  
            Vecteurs d'entraînement.  
        y : array-like, shape = [n_samples]  
            Étiquettes cibles (entiers de 0 à K-1).  
        """  
        n_samples, n_features = X.shape  
        n_classes = np.unique(y).shape  
          
        if self.weights is None:  
            self._initialize_weights(n_features, n_classes)

        # Conversion des étiquettes en format one-hot  
        y_one_hot = np.zeros((n_samples, n_classes))  
        y_one_hot[np.arange(n_samples), y] = 1

        for epoch in range(self.n_epochs):  
            # 1. Calcul des logits  
            logits = np.dot(X, self.weights) + self.bias  
              
            # 2. Calcul des probabilités (softmax)  
            probabilities = self._softmax(logits)  
              
            # 3. Calcul du gradient  
            # Gradient pour les poids : X.T @ (probabilities - y_one_hot)  
            # Gradient pour le biais : sum(probabilities - y_one_hot)  
            gradient_weights = np.dot(X.T, (probabilities - y_one_hot))  
            gradient_bias = np.sum(probabilities - y_one_hot, axis=0)  
              
            # 4. Mise à jour des poids et du biais  
            self.weights -= self.learning_rate * gradient_weights  
            self.bias -= self.learning_rate * gradient_bias  
              
            # Optionnel : Calculer et afficher la perte  
            # loss = -np.sum(y_one_hot * np.log(probabilities)) / n_samples  
            # if (epoch + 1) % 10 == 0:  
            #     print(f'Époque {epoch+1}/{self.n_epochs}, Perte : {loss:.4f}')

        return self

    def predict_proba(self, X):  
        """Retourne les probabilités des classes."""  
        logits = np.dot(X, self.weights) + self.bias  
        return self._softmax(logits)

    def predict(self, X):  
        """Retourne les prédictions de classe."""  
        probabilities = self.predict_proba(X)  
        return np.argmax(probabilities, axis=1)
```

### **4.5 Ouverture : Vers les Modèles Discriminatifs Structurés**

La régression softmax est un modèle extrêmement puissant, mais elle effectue une prédiction **locale** et indépendante pour chaque entrée x. Dans de nombreux domaines, comme le traitement du langage naturel ou la vision par ordinateur, les sorties que nous souhaitons prédire sont **structurées**. Par exemple, nous pourrions vouloir étiqueter chaque mot d'une phrase (étiquetage de séquence) ou segmenter chaque pixel d'une image. Dans ces cas, les étiquettes ne sont pas indépendantes ; l'étiquette d'un mot dépend de celle de ses voisins.

La régression softmax peut être vue comme le plus simple des **modèles log-linéaires conditionnels**. Un cadre plus général pour la prédiction structurée est celui des **Champs Aléatoires Conditionnels** (Conditional Random Fields, ou CRF), introduits par Lafferty, McCallum et Pereira en 2001.29 Un CRF utilise une forme exponentielle similaire à celle de la softmax pour modéliser la probabilité d'une structure de sortie entière (par exemple, une séquence d'étiquettes) conditionnée par une séquence d'entrée. La différence cruciale est que la normalisation (le dénominateur de la formule) n'est pas effectuée localement sur les

K classes pour une seule position, mais **globalement** sur toutes les séquences d'étiquettes possibles. Cette normalisation globale permet au modèle de capturer les dépendances entre les étiquettes de sortie et de résoudre des problèmes comme le "biais d'étiquette" ("label bias") qui affectaient les modèles conditionnels précédents.

Ainsi, les principes de la régression softmax — modélisation de scores linéaires, transformation en probabilités via une fonction exponentielle normalisée, et apprentissage par maximum de vraisemblance (entropie croisée) — constituent la pierre angulaire non seulement de la classification multi-classes standard, mais aussi des modèles de prédiction structurée les plus avancés utilisés aujourd'hui. Comprendre la régression softmax, c'est donc ouvrir la porte à une grande partie de l'apprentissage automatique moderne.

#### **Sources des citations**

1. transcription.txt  
2. Perceptron - Wikipedia, consulté le juillet 31, 2025, [https://en.wikipedia.org/wiki/Perceptron](https://en.wikipedia.org/wiki/Perceptron)  
3. Rosenblatt, F. (1958) The Perceptron A Probabilistic Model for Information Storage and Organization in the Brain. Psychological Review, 65, 386. - References, consulté le juillet 31, 2025, [https://www.scirp.org/reference/referencespapers?referenceid=2067071](https://www.scirp.org/reference/referencespapers?referenceid=2067071)  
4. The perceptron: a probabilistic model for information storage and organization in the brain., consulté le juillet 31, 2025, [https://www.semanticscholar.org/paper/The-perceptron%3A-a-probabilistic-model-for-storage-Rosenblatt/5d11aad09f65431b5d3cb1d85328743c9e53ba96](https://www.semanticscholar.org/paper/The-perceptron%3A-a-probabilistic-model-for-storage-Rosenblatt/5d11aad09f65431b5d3cb1d85328743c9e53ba96)  
5. The perceptron: A probabilistic model for information storage and organization in the brain [J] - ResearchGate, consulté le juillet 31, 2025, [https://www.researchgate.net/publication/221996769_The_perceptron_A_probabilistic_model_for_information_storage_and_organization_in_the_brain_J](https://www.researchgate.net/publication/221996769_The_perceptron_A_probabilistic_model_for_information_storage_and_organization_in_the_brain_J)  
6. rosenblatt-1957.pdf, consulté le juillet 31, 2025, [https://bpb-us-e2.wpmucdn.com/websites.umass.edu/dist/a/27637/files/2016/03/rosenblatt-1957.pdf](https://bpb-us-e2.wpmucdn.com/websites.umass.edu/dist/a/27637/files/2016/03/rosenblatt-1957.pdf)  
7. Perceptrons (book) - Wikipedia, consulté le juillet 31, 2025, [https://en.wikipedia.org/wiki/Perceptrons_(book)](https://en.wikipedia.org/wiki/Perceptrons_(book))  
8. Perceptrons; an Introduction to Computational Geometry - Marvin Minsky, Seymour Papert, consulté le juillet 31, 2025, [https://books.google.com/books/about/Perceptrons_an_Introduction_to_Computati.html?id=36E0QgAACAAJ](https://books.google.com/books/about/Perceptrons_an_Introduction_to_Computati.html?id=36E0QgAACAAJ)  
9. Perceptrons: An Introduction to Computational Geometry | Books Gateway - MIT Press Direct, consulté le juillet 31, 2025, [https://direct.mit.edu/books/monograph/3132/PerceptronsAn-Introduction-to-Computational](https://direct.mit.edu/books/monograph/3132/PerceptronsAn-Introduction-to-Computational)  
10. Perceptrons: An Introduction to Computational Geometry by Marvin Minsky | Goodreads, consulté le juillet 31, 2025, [https://www.goodreads.com/book/show/906121.Perceptrons](https://www.goodreads.com/book/show/906121.Perceptrons)  
11. Rosenblatt's Perceptron - Higher Education | Pearson, consulté le juillet 31, 2025, [https://www.pearsonhighered.com/assets/samplechapter/0/1/3/1/0131471392.pdf](https://www.pearsonhighered.com/assets/samplechapter/0/1/3/1/0131471392.pdf)  
12. The Exponential Family - Gregory Gundersen, consulté le juillet 31, 2025, [https://gregorygundersen.com/blog/2019/03/19/exponential-family/](https://gregorygundersen.com/blog/2019/03/19/exponential-family/)  
13. Exponential family - Wikipedia, consulté le juillet 31, 2025, [https://en.wikipedia.org/wiki/Exponential_family](https://en.wikipedia.org/wiki/Exponential_family)  
14. Sufficient statistics and intrinsic accuracy - Cambridge University Press, consulté le juillet 31, 2025, [https://www.cambridge.org/core/services/aop-cambridge-core/content/view/6A3E45FB1C423F3F684308F8910D6919/S0305004100019307a.pdf/div-class-title-sufficient-statistics-and-intrinsic-accuracy-div.pdf](https://www.cambridge.org/core/services/aop-cambridge-core/content/view/6A3E45FB1C423F3F684308F8910D6919/S0305004100019307a.pdf/div-class-title-sufficient-statistics-and-intrinsic-accuracy-div.pdf)  
15. Georges Darmois (1888 - 1960) - Biography - MacTutor History of Mathematics, consulté le juillet 31, 2025, [https://mathshistory.st-andrews.ac.uk/Biographies/Darmois/](https://mathshistory.st-andrews.ac.uk/Biographies/Darmois/)  
16. E. J. G. Pitman - Wikipedia, consulté le juillet 31, 2025, [https://en.wikipedia.org/wiki/E._J._G._Pitman](https://en.wikipedia.org/wiki/E._J._G._Pitman)  
17. Koopman, B.O. (1936) On Distributions Admitting a Sufficient Statistic. Transactions of the American Mathematical Society, 39, 399-409. - References - Scientific Research Publishing, consulté le juillet 31, 2025, [https://www.scirp.org/reference/referencespapers?referenceid=2109299](https://www.scirp.org/reference/referencespapers?referenceid=2109299)  
18. Nelder and Wedderburn (1972) Generalized Linear Models - P ..., consulté le juillet 31, 2025, [http://www.stat.uchicago.edu/~pmcc/pubs/paper32.pdf](http://www.stat.uchicago.edu/~pmcc/pubs/paper32.pdf)  
19. Generalized linear models - Rothamsted Repository, consulté le juillet 31, 2025, [https://repository.rothamsted.ac.uk/item/8v5q8/generalized-linear-models](https://repository.rothamsted.ac.uk/item/8v5q8/generalized-linear-models)  
20. Generalized Linear Models - Yang Li, consulté le juillet 31, 2025, [http://yangli-feasibility.com/home/classes/lfd2024spring/media/Generalized_Linear_Models.pdf](http://yangli-feasibility.com/home/classes/lfd2024spring/media/Generalized_Linear_Models.pdf)  
21. Nelder, J.A. and Wedderburn, R.W.M. (1972) Generalized Linear Models. Journal of the Royal Statistical Society, Series A, 135, 370-384. - References - Scientific Research Publishing, consulté le juillet 31, 2025, [https://www.scirp.org/reference/referencespapers?referenceid=2052209](https://www.scirp.org/reference/referencespapers?referenceid=2052209)  
22. GENERALIZED LINEAR MODELS - IME-USP, consulté le juillet 31, 2025, [https://www.ime.usp.br/~abe/lista/pdftGzimaFtH4.pdf](https://www.ime.usp.br/~abe/lista/pdftGzimaFtH4.pdf)  
23. Generalized Linear Models, consulté le juillet 31, 2025, [https://jhanley.biostat.mcgill.ca/bios601/Likelihood/NelderWedderburn1972.pdf](https://jhanley.biostat.mcgill.ca/bios601/Likelihood/NelderWedderburn1972.pdf)  
24. Softmax and Cross Entropy Loss - Paras Dahal, consulté le juillet 31, 2025, [https://www.parasdahal.com/softmax-crossentropy](https://www.parasdahal.com/softmax-crossentropy)  
25. Softmax Function and Cross Entropy Loss Function - Deep Learning, consulté le juillet 31, 2025, [https://guandi1995.github.io/Softmax-Function-and-Cross-Entropy-Loss-Function/](https://guandi1995.github.io/Softmax-Function-and-Cross-Entropy-Loss-Function/)  
26. Softmax and cross-entropy loss | Deep Learning Systems Class Notes - Fiveable, consulté le juillet 31, 2025, [https://library.fiveable.me/deep-learning-systems/unit-3/softmax-cross-entropy-loss/study-guide/49HP9j8ffUi2bmbJ](https://library.fiveable.me/deep-learning-systems/unit-3/softmax-cross-entropy-loss/study-guide/49HP9j8ffUi2bmbJ)  
27. Derivative of the Softmax Function and the Categorical Cross-Entropy Loss - GeeksforGeeks, consulté le juillet 31, 2025, [https://www.geeksforgeeks.org/machine-learning/derivative-of-the-softmax-function-and-the-categorical-cross-entropy-loss/](https://www.geeksforgeeks.org/machine-learning/derivative-of-the-softmax-function-and-the-categorical-cross-entropy-loss/)  
28. Convolutional Neural Networks (CNN): Softmax & Cross-Entropy - SuperDataScience, consulté le juillet 31, 2025, [https://www.superdatascience.com/blogs/convolutional-neural-networks-cnn-softmax-crossentropy](https://www.superdatascience.com/blogs/convolutional-neural-networks-cnn-softmax-crossentropy)  
29. Conditional Random Fields, consulté le juillet 31, 2025, [http://www.inference.org.uk/hmw26/crf/](http://www.inference.org.uk/hmw26/crf/)  
30. Conditional Random Fields: Probabilistic Models ... - CS@Columbia, consulté le juillet 31, 2025, [http://www.cs.columbia.edu/~jebara/6772/papers/crf.pdf](http://www.cs.columbia.edu/~jebara/6772/papers/crf.pdf)  
31. Conditional Random Fields: Probabilistic Models for Segmenting and Labeling Sequential Data - Computer Science Cornell, consulté le juillet 31, 2025, [https://www.cs.cornell.edu/courses/cs6784/2010sp/lecture/10-LaffertyEtAl01.pdf](https://www.cs.cornell.edu/courses/cs6784/2010sp/lecture/10-LaffertyEtAl01.pdf)
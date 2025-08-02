# Les Classifieurs Probabilistes Génératifs et les Machines à Vecteurs de Support : Une Approche Théorique et Pratique

## Introduction

### Le Paradigme de l'Apprentissage : Génératif vs. Discriminatif

Au cœur de l'apprentissage supervisé, deux philosophies fondamentales s'affrontent pour aborder la tâche de classification : l'approche générative et l'approche discriminative. Comprendre cette dichotomie est essentiel pour naviguer dans le paysage des algorithmes d'apprentissage automatique et pour choisir l'outil le plus adapté à un problème donné.

Les modèles génératifs, tels que le classifieur naïf de Bayes que nous explorerons en détail, cherchent à comprendre la structure sous-jacente des données. Ils apprennent la distribution de probabilité conjointe des caractéristiques et des étiquettes, notée $P(x,y)$. L'objectif est de construire un modèle capable de "générer" des données qui ressemblent aux données d'entraînement. C'est en quelque sorte comme apprendre l'histoire complète de la façon dont les données ont été créées. Une fois ce modèle de $P(x,y)$ appris, il peut être utilisé pour la classification en appliquant la règle de Bayes pour déduire la probabilité a posteriori $P(y|x)$.

À l'opposé, les modèles discriminatifs, comme la régression logistique ou les machines à vecteurs de support (SVM), adoptent une approche plus directe. Ils ne s'intéressent pas à la manière dont les données sont générées ; leur unique objectif est de trouver la frontière de décision qui sépare le mieux les différentes classes. Ils modélisent directement la probabilité conditionnelle $P(y|x)$ ou trouvent une fonction de décision qui mappe directement une entrée $x$ à une classe $y$. Cette approche est souvent plus performante pour les tâches de classification pure, car elle concentre tous ses efforts sur la discrimination entre les classes.

Cette distinction fondamentale entre "apprendre la distribution des données" et "apprendre la frontière entre les classes" servira de fil conducteur à cette leçon, nous permettant de contraster la nature probabiliste et holistique de Naive Bayes avec l'élégance géométrique et ciblée des SVMs.

### Feuille de Route de la Leçon

Cette leçon est structurée en deux parties distinctes, chacune se concentrant sur un pilier de l'apprentissage automatique.

**Partie I : Le Classifieur Naïf de Bayes (Naive Bayes)**

Nous commencerons par une exploration approfondie du classifieur naïf de Bayes. Cet algorithme, célèbre pour sa simplicité, son efficacité de calcul et ses performances étonnamment robustes, en particulier dans le domaine de la classification de texte, constitue un point de départ incontournable. Nous examinerons ses fondements probabilistes, nous confronterons à ses limites théoriques, notamment le problème des probabilités nulles, et nous mettrons en œuvre ses variantes les plus efficaces pour surmonter ces défis.

**Partie II : Introduction aux Machines à Vecteurs de Support (SVM)**

Ensuite, nous nous aventurerons dans le monde puissant et élégant des machines à vecteurs de support. Nous construirons l'intuition de cet algorithme à partir de zéro, en partant de l'idée simple et visuelle de trouver la "meilleure" ligne de séparation entre deux ensembles de points. Nous formaliserons ensuite cette intuition en un problème d'optimisation sous contraintes, révélant la rigueur mathématique qui sous-tend la puissance des SVMs.

### Piliers Académiques

Notre exploration ne sera pas purement abstraite ; elle sera ancrée dans les recherches fondamentales qui ont façonné le domaine. Nous nous référerons spécifiquement à deux articles séminaux qui ont marqué un tournant dans le développement de ces algorithmes :

1. **McCallum, A., & Nigam, K. (1998). A Comparison of Event Models for Naive Bayes Text Classification.** Cet article a clarifié une confusion persistante dans la communauté en comparant systématiquement les deux principaux modèles d'événements pour Naive Bayes, fournissant des preuves empiriques solides qui guident encore aujourd'hui le choix du modèle pour la classification de texte.

2. **Cortes, C., & Vapnik, V. (1995). Support-vector networks.** Ce travail monumental a introduit les machines à vecteurs de support telles que nous les connaissons, en étendant le concept de classifieur à marge optimale aux données non linéairement séparables. Cette avancée a transformé les SVMs d'un concept théorique intéressant en l'un des outils de classification les plus robustes et les plus utilisés en pratique.

En nous appuyant sur ces travaux, nous ne nous contenterons pas d'apprendre le "comment", mais nous chercherons également à comprendre le "pourquoi" derrière la conception et le succès de ces algorithmes.

---

## Partie I : Le Classifieur Naïf de Bayes (Naive Bayes)

### 1. Fondements du Classifieur Naïf de Bayes

#### Le Cadre d'Apprentissage Génératif

Le classifieur naïf de Bayes est l'archétype de l'algorithme d'apprentissage génératif. La tâche fondamentale en classification est de prédire une classe $y$ (par exemple, "spam" ou "non-spam") à partir d'un ensemble de caractéristiques observées $x$ (par exemple, les mots d'un e-mail). Pour ce faire, nous cherchons à calculer la probabilité a posteriori $P(y|x)$, c'est-à-dire la probabilité de la classe $y$ étant donné les caractéristiques $x$.

La pierre angulaire de cette approche est la règle de Bayes, qui nous permet d'inverser la probabilité conditionnelle :

$$P(y|x) = \frac{P(x|y)P(y)}{P(x)}$$

Analysons chaque terme de cette équation :

- $P(y|x)$ est la probabilité a posteriori, celle que nous voulons calculer pour prendre notre décision.
- $P(x|y)$ est la vraisemblance (ou probabilité conditionnelle de la classe), c'est-à-dire la probabilité d'observer les caractéristiques $x$ si nous savons que l'exemple appartient à la classe $y$. C'est le cœur du modèle génératif.
- $P(y)$ est la probabilité a priori de la classe, c'est-à-dire la probabilité globale qu'un exemple appartienne à la classe $y$ avant même d'avoir examiné ses caractéristiques.
- $P(x)$ est la probabilité marginale des caractéristiques, souvent appelée "évidence".

Lors de la prédiction, pour une entrée $x$ donnée, le dénominateur $P(x)$ est constant pour toutes les classes. Par conséquent, pour trouver la classe la plus probable, nous n'avons pas besoin de le calculer. La tâche de classification se simplifie pour trouver la classe $y$ qui maximise le numérateur :

$$\hat{y} = \arg\max_y P(x|y)P(y)$$

L'approche générative se concentre donc sur la modélisation des deux termes clés : le prior $P(y)$ et la vraisemblance $P(x|y)$.

#### Application à la Classification de Texte : Le Filtre Anti-Spam

Pour rendre ces concepts concrets, nous utiliserons un exemple fil rouge tout au long de cette section : la construction d'un filtre anti-spam. L'objectif est de classer un e-mail donné $(x)$ comme étant soit un spam $(y=1)$, soit un e-mail légitime (non-spam, ou "ham") $(y=0)$. C'est une tâche de classification binaire classique où Naive Bayes a historiquement excellé.

#### Représentation des Données : Le Modèle du "Sac de Mots" (Bag of Words)

Avant de pouvoir appliquer un algorithme, nous devons transformer nos données textuelles non structurées en une représentation numérique structurée. La méthode la plus courante pour Naive Bayes est le modèle du "sac de mots" (Bag of Words).

Cette approche consiste à :

1. **Construire un dictionnaire (ou vocabulaire)** : On établit une liste de tous les mots uniques présents dans l'ensemble des documents d'entraînement. Ce dictionnaire peut contenir des milliers, voire des dizaines de milliers de mots.

2. **Représenter chaque document** : Chaque document est ensuite transformé en un vecteur numérique. L'hypothèse fondamentale du "sac de mots" est que l'ordre des mots est ignoré. Seule la présence (ou la fréquence) des mots compte.

Dans un premier temps, nous utiliserons une représentation binaire simple : pour un dictionnaire de taille $N$, chaque e-mail est représenté par un vecteur binaire $x$ de dimension $N$. La $j$-ème composante du vecteur, $x_j$, est égale à 1 si le $j$-ème mot du dictionnaire est présent dans l'e-mail, et à 0 sinon.

#### L'Hypothèse "Naïve" d'Indépendance Conditionnelle

Le défi majeur de la modélisation de $P(x|y)$ est que le vecteur de caractéristiques $x$ est de très grande dimension. Si notre dictionnaire contient 10 000 mots, $x$ est un vecteur de dimension 10 000. Modéliser la distribution de probabilité conjointe de ces 10 000 variables ($P(x_1, x_2, ..., x_{10000}|y)$) est informatiquement et statistiquement infaisable.

C'est ici qu'intervient l'hypothèse "naïve" qui donne son nom à l'algorithme. Nous supposons que toutes les caractéristiques (les mots) sont conditionnellement indépendantes les unes des autres, étant donné la classe $y$. Mathématiquement, cela se traduit par :

$$P(x|y) = P(x_1, x_2, \ldots, x_N|y) = \prod_{j=1}^{N} P(x_j|y)$$

Cette hypothèse est "naïve" car elle est presque toujours fausse en pratique. Dans le langage naturel, les mots ne sont évidemment pas indépendants. La présence du mot "Maison" augmente la probabilité de voir le mot "Blanche", par exemple. Cependant, cette simplification drastique rend le problème de modélisation gérable. Au lieu d'estimer une distribution conjointe complexe, nous n'avons plus qu'à estimer la probabilité de chaque mot individuellement, conditionnellement à la classe. Comme nous le verrons, malgré son caractère irréaliste, cette hypothèse conduit à des classifieurs étonnamment efficaces.

### 2. Le Modèle d'Événement Bernoulli Multivarié

Le premier modèle que nous étudions, directement issu de la représentation binaire du "sac de mots", est le modèle d'événement Bernoulli multivarié.

#### Formalisation Mathématique

Dans ce modèle, chaque document est considéré comme le résultat d'un unique "événement" : un tirage Bernoulli multivarié. Imaginez que pour chaque mot de notre dictionnaire de taille $N$, nous lançons une pièce de monnaie (biaisée différemment pour les spams et les non-spams) pour décider si ce mot sera inclus ou non dans le document. Le document est donc le résultat de ces $N$ lancers de pièces.

Les paramètres que nous devons apprendre à partir des données d'entraînement sont :

- **Le prior de classe** : $\phi_y = P(y=1)$. C'est la probabilité globale qu'un e-mail soit un spam.
- **Les probabilités conditionnelles de présence des mots pour les spams** : $\phi_{j|y=1} = P(x_j=1|y=1)$ pour chaque mot $j$ du dictionnaire. C'est la probabilité que le mot $j$ apparaisse dans un e-mail qui est un spam.
- **Les probabilités conditionnelles de présence des mots pour les non-spams** : $\phi_{j|y=0} = P(x_j=1|y=0)$ pour chaque mot $j$. C'est la probabilité que le mot $j$ apparaisse dans un e-mail légitime.

#### Estimation par Maximum de Vraisemblance (Maximum Likelihood Estimation - MLE)

L'estimation par maximum de vraisemblance (MLE) est une méthode standard pour estimer les paramètres d'un modèle statistique. L'idée est de choisir les valeurs des paramètres qui maximisent la probabilité (la "vraisemblance") d'avoir observé les données d'entraînement. Pour Naive Bayes, cela se réduit à de simples comptages de fréquences.

Soit $m$ le nombre total d'e-mails dans notre ensemble d'entraînement. Les estimations MLE des paramètres sont :

**Pour le prior de classe :**
$$\phi_y = \frac{1}{m}\sum_{i=1}^{m} \mathbf{1}\{y^{(i)}=1\}$$

C'est simplement la fraction d'e-mails dans l'ensemble d'entraînement qui sont étiquetés comme spam.

**Pour les probabilités conditionnelles des mots (ici pour la classe spam, $y=1$) :**
$$\phi_{j|y=1} = \frac{\sum_{i=1}^{m} \mathbf{1}\{x_j^{(i)}=1 \land y^{(i)}=1\}}{\sum_{i=1}^{m} \mathbf{1}\{y^{(i)}=1\}}$$

Le numérateur est le nombre d'e-mails de spam qui contiennent le mot $j$. Le dénominateur est le nombre total d'e-mails de spam. C'est donc la fraction des e-mails de spam qui contiennent le mot $j$. La formule est analogue pour la classe non-spam $(y=0)$.

#### Le Problème de la Fréquence Nulle (Zero-Frequency Problem)

L'approche MLE, bien qu'intuitive, présente une faille critique qui peut rendre le classifieur inutilisable en pratique. C'est le problème de la fréquence nulle.

**Scénario Catastrophe :** Imaginons que nous entraînons notre filtre anti-spam sur notre boîte de réception personnelle. Notre dictionnaire contient 10 000 mots. Supposons que le mot numéro 6017 soit "NIPS" (une célèbre conférence en apprentissage automatique). Il est très probable que ce mot n'apparaisse dans aucun des e-mails de spam que nous avons reçus jusqu'à présent.

En utilisant l'estimation MLE, le paramètre $\phi_{\text{NIPS}|y=1}$ sera calculé comme suit :

$$\phi_{\text{NIPS}|y=1} = \frac{\text{Nombre de spams contenant "NIPS"}}{\text{Nombre total de spams}} = \frac{0}{\text{Nombre total de spams}} = 0$$

Maintenant, supposons qu'après avoir entraîné notre modèle, nous recevions un nouvel e-mail d'un collègue disant : "Considérons-nous de soumettre notre projet à la conférence NIPS?". Cet e-mail contient le mot "NIPS". Pour le classer, notre algorithme doit calculer $P(\text{spam}|x)$ et $P(\text{non-spam}|x)$.

Le calcul pour la classe spam, $P(x|y=1)$, implique le produit de nombreuses probabilités :

$$P(x|y=1) = \prod_{j=1}^{10000} P(x_j|y=1)$$

Comme l'e-mail contient le mot "NIPS", le terme pour $j=6017$ sera $P(x_{\text{NIPS}}=1|y=1) = \phi_{\text{NIPS}|y=1} = 0$. La présence d'un seul terme nul dans ce long produit rend le résultat final égal à zéro. Donc, $P(x|y=1) = 0$.

Si, par malchance, le mot "NIPS" n'est jamais apparu non plus dans nos e-mails légitimes d'entraînement, alors $P(x|y=0)$ sera également égal à zéro. La prédiction finale du classifieur devient alors :

$$P(y=1|x) \propto P(x|y=1)P(y=1) = 0$$
$$P(y=0|x) \propto P(x|y=0)P(y=0) = 0$$

Le classifieur doit évaluer $\frac{0}{0+0}$, ce qui est indéfini et provoque une erreur.

D'un point de vue statistique, il est fondamentalement erroné d'attribuer une probabilité de zéro à un événement simplement parce qu'il n'a pas été observé dans un échantillon fini. Un événement non observé n'est pas un événement impossible. C'est pour résoudre ce problème fondamental que nous introduisons le lissage de Laplace.

### 3. Le Lissage de Laplace (Laplace Smoothing)

Le lissage de Laplace, également connu sous le nom de lissage "add-one", est une technique simple mais efficace pour résoudre le problème de la fréquence nulle. Il empêche toute probabilité estimée d'être exactement nulle (ou exactement un).

#### Intuition : L'Exemple du Football de Stanford

Pour comprendre l'intuition, considérons un exemple simple et non lié à la classification de texte. Imaginons que nous suivons une équipe de football qui a joué 4 matchs à l'extérieur et les a tous perdus. Le bilan est de 0 victoire pour 4 défaites.

Si nous utilisons l'estimation par maximum de vraisemblance pour prédire les chances de l'équipe de gagner son prochain match, nous obtiendrions :

$$P(\text{victoire}) = \frac{\text{Nombre de victoires}}{\text{Nombre total de matchs}} = \frac{0}{4} = 0$$

Cette estimation suggère une certitude absolue que l'équipe perdra, ce qui semble excessivement pessimiste et statistiquement fragile. Une série de 4 défaites est malheureuse, mais ne garantit pas une défaite future.

L'idée du lissage de Laplace est de se comporter comme si nous avions observé chaque résultat possible une fois de plus qu'en réalité. Au lieu de compter 0 victoire et 4 défaites, nous ajoutons 1 à chaque compteur. Nous prétendons avoir vu $(0+1) = 1$ victoire et $(4+1) = 5$ défaites. Le nombre total de matchs "imaginaires" est donc de $4$ (réels) $+ 2$ (un pour chaque résultat possible : victoire, défaite) $= 6$. La nouvelle estimation de la probabilité de victoire devient :

$$P(\text{victoire})_{\text{lissé}} = \frac{0+1}{4+2} = \frac{1}{6}$$

C'est une estimation beaucoup plus raisonnable. Elle reconnaît que la victoire est peu probable compte tenu des résultats passés, mais n'exclut pas sa possibilité. Cette technique a été historiquement justifiée par le mathématicien Pierre-Simon de Laplace qui l'a utilisée pour estimer la probabilité que le soleil se lève demain, arguant que même si nous l'avons vu se lever tous les jours de notre vie, nous ne pouvons pas être absolument certains qu'il se lèvera demain.

#### Formalisation Générale : Le Lissage "Add-k"

Plus généralement, pour une variable aléatoire discrète qui peut prendre $k$ valeurs différentes, la probabilité lissée de l'issue $j$ est calculée comme suit :

$$P(j)_{\text{lissé}} = \frac{\text{nombre d'observations de } j + 1}{\text{nombre total d'observations} + k}$$

On ajoute 1 au numérateur (le compte de l'issue spécifique) et on ajoute $k$ (le nombre total d'issues possibles) au dénominateur.

#### Application au Modèle Bernoulli Naive Bayes

Revenons à notre modèle Bernoulli Naive Bayes. Pour chaque mot $j$, la variable $x_j$ est binaire : elle peut être soit 1 (présent), soit 0 (absent). Il y a donc $k=2$ issues possibles.

Pour appliquer le lissage de Laplace à l'estimation de nos paramètres $\phi_{j|y=1}$, la formule MLE est modifiée comme suit :

$$\phi_{j|y=1, \text{lissé}} = \frac{(\text{nombre de spams contenant le mot } j) + 1}{(\text{nombre total de spams}) + 2}$$

De même pour la classe non-spam :

$$\phi_{j|y=0, \text{lissé}} = \frac{(\text{nombre de non-spams contenant le mot } j) + 1}{(\text{nombre total de non-spams}) + 2}$$

Avec cette simple modification, même si un mot n'est jamais apparu dans les e-mails de spam (numérateur de la fraction originale est 0), sa probabilité lissée ne sera jamais nulle. Par exemple, si nous avons 1000 e-mails de spam et qu'aucun ne contient le mot "NIPS", la probabilité lissée sera de $\frac{1}{1000+2}$, une petite valeur non nulle. Cela résout élégamment le problème de la fréquence nulle et rend le classifieur robuste aux mots rares ou jamais vus auparavant dans une certaine classe.

#### Implémentation

La mise en œuvre d'un classifieur Bernoulli Naive Bayes avec lissage de Laplace est relativement simple.

##### Pseudocode

```
Algorithme : Entraînement de Bernoulli Naive Bayes avec Lissage de Laplace

Entrées :
  - X : Matrice d'entraînement (m documents, N caractéristiques binaires)
  - y : Vecteur d'étiquettes (m documents)

Sorties :
  - phi_y : Probabilité a priori de la classe 1
  - phi_j_y0 : Vecteur de probabilités P(mot j présent | y=0)
  - phi_j_y1 : Vecteur de probabilités P(mot j présent | y=1)

1. Initialiser les compteurs :
   num_spam_docs = 0
   num_ham_docs = 0
   word_counts_in_spam = vecteur de zéros de taille N
   word_counts_in_ham = vecteur de zéros de taille N

2. Parcourir chaque document (x_i, y_i) dans l'ensemble d'entraînement :
   Si y_i == 1 (spam) :
     num_spam_docs += 1
     word_counts_in_spam += x_i  // Ajoute 1 pour chaque mot présent
   Sinon (ham) :
     num_ham_docs += 1
     word_counts_in_ham += x_i   // Ajoute 1 pour chaque mot présent

3. Calculer les paramètres avec lissage de Laplace :
   m = num_spam_docs + num_ham_docs
   phi_y = num_spam_docs / m

   // Lissage : ajouter 1 au numérateur, 2 au dénominateur
   phi_j_y1 = (word_counts_in_spam + 1) / (num_spam_docs + 2)
   phi_j_y0 = (word_counts_in_ham + 1) / (num_ham_docs + 2)

4. Retourner phi_y, phi_j_y0, phi_j_y1
```

##### Code Python

Voici une implémentation complète en Python utilisant NumPy.

```python
import numpy as np

class BernoulliNaiveBayes:
    """
    Un classifieur Naïf de Bayes pour le modèle d'événement Bernoulli multivarié.
    Ce modèle est adapté aux caractéristiques binaires (présence/absence).
    """

    def __init__(self, laplace_smoothing=1.0):
        """
        Initialise le classifieur.
        :param laplace_smoothing: La constante de lissage (alpha). 1.0 correspond au lissage de Laplace standard.
        """
        self.laplace_smoothing = laplace_smoothing
        self.phi_y = None
        self.phi_j_y0 = None
        self.phi_j_y1 = None
        self.classes_ = None

    def fit(self, X, y):
        """
        Entraîne le modèle Naive Bayes sur les données d'entraînement.
        :param X: Matrice d'entraînement de forme (n_samples, n_features). Les valeurs doivent être binaires (0 ou 1).
        :param y: Vecteur d'étiquettes de forme (n_samples,). Les étiquettes doivent être 0 ou 1.
        """
        n_samples, n_features = X.shape
        self.classes_ = np.unique(y)
        if not np.array_equal(self.classes_, [0, 1]):
            raise ValueError("Les étiquettes y doivent être 0 ou 1.")

        # Séparer les données par classe
        X_y0 = X[y == 0]
        X_y1 = X[y == 1]

        # Calculer le prior de classe P(y=1)
        self.phi_y = X_y1.shape[0] / n_samples

        # Calculer les probabilités conditionnelles P(x_j=1|y) avec lissage
        # Le lissage de Laplace ajoute 'alpha' au numérateur et '2 * alpha' au dénominateur (car k=2 issues)
        num_y0_docs = X_y0.shape[0]
        num_y1_docs = X_y1.shape[0]
        
        # Compter le nombre de documents dans chaque classe où le mot j apparaît
        word_counts_in_y0 = X_y0.sum(axis=0)
        word_counts_in_y1 = X_y1.sum(axis=0)

        # Appliquer le lissage
        self.phi_j_y0 = (word_counts_in_y0 + self.laplace_smoothing) / (num_y0_docs + 2 * self.laplace_smoothing)
        self.phi_j_y1 = (word_counts_in_y1 + self.laplace_smoothing) / (num_y1_docs + 2 * self.laplace_smoothing)

    def _log_likelihood(self, X):
        """Calcule le log de la vraisemblance P(x|y) pour chaque classe."""
        # P(x_j|y) = (phi_j_y)^x_j * (1 - phi_j_y)^(1-x_j)
        # log(P(x_j|y)) = x_j * log(phi_j_y) + (1-x_j) * log(1 - phi_j_y)
        # On somme les logs pour éviter l'underflow numérique
        
        # Pour y=1
        log_prob_y1 = np.sum(X * np.log(self.phi_j_y1) + (1 - X) * np.log(1 - self.phi_j_y1), axis=1)
        
        # Pour y=0
        log_prob_y0 = np.sum(X * np.log(self.phi_j_y0) + (1 - X) * np.log(1 - self.phi_j_y0), axis=1)
        
        return log_prob_y0, log_prob_y1

    def predict_proba(self, X):
        """
        Calcule les probabilités a posteriori pour chaque classe.
        :param X: Matrice de données à prédire de forme (n_samples, n_features).
        :return: Matrice de probabilités de forme (n_samples, 2).
        """
        if self.phi_y is None:
            raise RuntimeError("Le modèle doit être entraîné avant de pouvoir prédire.")

        log_prob_y0, log_prob_y1 = self._log_likelihood(X)
        
        # Calculer les log-posteriors (sans le log de l'évidence P(x))
        # log(P(y|x)) ~ log(P(x|y)) + log(P(y))
        log_posterior_y1 = log_prob_y1 + np.log(self.phi_y)
        log_posterior_y0 = log_prob_y0 + np.log(1 - self.phi_y)
        
        # Pour éviter les problèmes numériques et normaliser, on utilise l'astuce log-sum-exp
        # P(y=1|x) = exp(log_post_y1) / (exp(log_post_y1) + exp(log_post_y0))
        # On peut simplifier en soustrayant le max des logs pour la stabilité
        max_log = np.maximum(log_posterior_y0, log_posterior_y1)
        exp_y0 = np.exp(log_posterior_y0 - max_log)
        exp_y1 = np.exp(log_posterior_y1 - max_log)
        
        prob_y1 = exp_y1 / (exp_y0 + exp_y1)
        prob_y0 = 1 - prob_y1
        
        return np.vstack((prob_y0, prob_y1)).T

    def predict(self, X):
        """
        Prédit la classe pour chaque échantillon dans X.
        :param X: Matrice de données à prédire de forme (n_samples, n_features).
        :return: Vecteur de prédictions de classe (0 ou 1).
        """
        probas = self.predict_proba(X)
        return (probas[:, 1] > 0.5).astype(int)
```

### 4. Le Modèle d'Événement Multinomial

#### Motivation : Les Limites du Modèle Bernoulli

Le modèle Bernoulli, bien que fonctionnel, présente une limitation majeure : il ne capture que la présence ou l'absence d'un mot. Il ignore complètement la fréquence des mots. Un e-mail contenant le mot "gratuit" une seule fois est traité de la même manière qu'un e-mail le contenant dix fois. Intuitivement, la répétition de mots suspects comme "gratuit", "offre", "gagnant" ou "drogues" est un signal très fort de spam. En ignorant cette information, le modèle Bernoulli perd une partie précieuse du signal.

#### Une Nouvelle Représentation des Données

Pour capturer la fréquence des mots, nous devons changer notre façon de représenter les données. Le modèle d'événement multinomial utilise une représentation différente :

- Un document n'est plus un vecteur binaire de taille fixe.
- Il est représenté comme une séquence d'indices de mots. La longueur de ce vecteur, notée $n_i$ pour le document $i$, correspond au nombre total de mots dans ce document.

Par exemple, si notre dictionnaire est {'acheter': 800, 'drogues': 1600, 'maintenant': 6200,...}, l'e-mail "drogues, acheter drogues maintenant" serait représenté par le vecteur $x = [1600, 800, 1600, 6200]$. La longueur de ce vecteur est $n=4$. Cette représentation préserve l'information sur la répétition des mots.

#### Formalisation du Modèle Multinomial

Ce changement de représentation implique un changement dans l'histoire générative sous-jacente. Le modèle multinomial ne considère plus un document comme un seul événement, mais comme une séquence de $n$ événements indépendants.

L'histoire générative est la suivante :

1. **Choisir une classe** : On lance un dé biaisé pour décider si le document sera un spam $(y=1)$ ou un non-spam $(y=0)$. La probabilité de ce choix est le prior $P(y)$.

2. **Générer les mots** : Une fois la classe choisie, on utilise le "dé à mots" correspondant à cette classe. C'est un dé géant avec autant de faces que de mots dans notre vocabulaire $(|V|)$. On lance ce dé $n$ fois (où $n$ est la longueur du document) pour générer les mots du document. Chaque lancer est indépendant des autres.

Les paramètres de ce modèle sont donc différents :

- **Le prior de classe** : $\phi_y = P(y=1)$, comme avant.
- **Les probabilités de mots pour chaque classe** : $\phi_{k|y} = P(\text{mot}=k|y)$. C'est la probabilité que n'importe quel mot choisi au hasard dans un document de la classe $y$ soit le $k$-ème mot du dictionnaire. Notez que pour une classe donnée, la somme de ces probabilités sur tous les mots du vocabulaire doit être égale à 1 : $\sum_{k=1}^{|V|} \phi_{k|y} = 1$.

#### Contexte Académique : L'Article de McCallum & Nigam (1998)

C'est à ce stade qu'il est crucial de mentionner l'article séminal de Andrew McCallum et Kamal Nigam, "A Comparison of Event Models for Naive Bayes Text Classification". Avant leur travail, les deux modèles (Bernoulli et Multinomial) étaient utilisés dans la littérature, souvent sans justification claire, créant une certaine confusion.

Leur contribution a été de :

1. Clarifier formellement les différences entre les deux modèles, en particulier leurs "histoires génératives" distinctes.
2. Comparer empiriquement leurs performances sur plusieurs corpus de classification de texte.

Leur conclusion principale, qui a profondément influencé la pratique, est que bien que le modèle Bernoulli puisse être performant pour des vocabulaires de petite taille, le modèle multinomial est presque toujours supérieur, en particulier avec des vocabulaires plus grands. Ils ont rapporté une réduction moyenne de l'erreur de classification de 27% en faveur du modèle multinomial. Cette validation empirique solide justifie pourquoi, pour la classification de texte, le modèle multinomial est aujourd'hui considéré comme le standard par défaut.

#### Estimation des Paramètres (MLE) et Lissage

L'estimation des paramètres pour le modèle multinomial est également basée sur le comptage de fréquences, mais de manière différente.

**Estimation MLE :**

Le paramètre $\phi_{k|y=1}$ est la probabilité que le mot $k$ apparaisse dans un spam. L'estimation MLE est la fréquence relative du mot $k$ parmi tous les mots de tous les e-mails de spam.

$$\phi_{k|y=1} = \frac{\text{Nombre total d'occurrences du mot } k \text{ dans tous les spams}}{\text{Nombre total de mots dans tous les spams}}$$

Mathématiquement, avec $n_i$ la longueur du document $i$ et $N_{ik}$ le nombre de fois que le mot $k$ apparaît dans le document $i$ :

$$\phi_{k|y=1} = \frac{\sum_{i=1}^{m} \mathbf{1}\{y^{(i)}=1\} N_{ik}}{\sum_{i=1}^{m} \mathbf{1}\{y^{(i)}=1\} n_i}$$

**Lissage de Laplace :**

Le problème de la fréquence nulle existe aussi ici. Si un mot n'est jamais apparu dans les spams d'entraînement, sa probabilité sera nulle. Pour le lissage, nous devons considérer le nombre d'issues possibles. Notre "dé à mots" a $|V|$ faces, où $|V|$ est la taille du vocabulaire. Donc, $k=|V|$.

La formule lissée devient :

$$\phi_{k|y=1,\text{lissé}} = \frac{(\text{Nombre total d'occurrences du mot } k \text{ dans tous les spams}) + 1}{(\text{Nombre total de mots dans tous les spams}) + |V|}$$

On ajoute 1 au compteur de chaque mot et on ajoute la taille totale du vocabulaire au décompte total des mots.

#### Tableau Comparatif

Pour synthétiser, voici un tableau récapitulatif des deux modèles.

| Caractéristique | Modèle Bernoulli Multivarié | Modèle Multinomial |
|-----------------|------------------------------|---------------------|
| **Représentation** | Vecteur binaire de taille |V| | Séquence d'indices de mots |
| **Histoire Générative** | Pour un document, lancer |V| pièces (une par mot) | Pour un document, choisir la longueur n, puis lancer n fois un dé à |V| faces |
| **Fréquence des Mots** | Ignorée. La présence multiple d'un mot n'a pas d'impact. | Capturée. La fréquence des mots influence directement les probabilités. |
| **Paramètre φ_{k\|y}** | P(x_k=1\|y) - Probabilité de présence | P(mot=k\|y) - Probabilité d'occurrence |
| **Cas d'Usage Typique** | Classification de documents courts où la présence d'un mot est plus importante que sa fréquence. | Classification de texte en général, surtout pour des documents plus longs. C'est le standard. |

#### Implémentation

##### Pseudocode

```
Algorithme : Entraînement de Multinomial Naive Bayes avec Lissage de Laplace

Entrées :
  - D : Collection de documents d'entraînement, où chaque document est une liste de mots/tokens
  - y : Vecteur d'étiquettes
  - V : Vocabulaire (liste de tous les mots uniques)

Sorties :
  - phi_y : Probabilité a priori de la classe 1
  - phi_k_y0 : Vecteur de probabilités P(mot=k | y=0)
  - phi_k_y1 : Vecteur de probabilités P(mot=k | y=1)

1. Initialiser les compteurs :
   num_spam_docs = 0
   num_ham_docs = 0
   total_words_in_spam = 0
   total_words_in_ham = 0
   word_counts_in_spam = dictionnaire (ou vecteur) de zéros pour chaque mot dans V
   word_counts_in_ham = dictionnaire (ou vecteur) de zéros pour chaque mot dans V

2. Parcourir chaque document (d_i, y_i) dans l'ensemble d'entraînement :
   Si y_i == 1 (spam) :
     num_spam_docs += 1
     total_words_in_spam += longueur(d_i)
     Pour chaque mot w dans d_i :
       word_counts_in_spam[w] += 1
   Sinon (ham) :
     num_ham_docs += 1
     total_words_in_ham += longueur(d_i)
     Pour chaque mot w dans d_i :
       word_counts_in_ham[w] += 1

3. Calculer les paramètres avec lissage de Laplace :
   m = num_spam_docs + num_ham_docs
   phi_y = num_spam_docs / m
   taille_vocabulaire = longueur(V)

   // Lissage : ajouter 1 au numérateur, |V| au dénominateur
   phi_k_y1 = (word_counts_in_spam + 1) / (total_words_in_spam + taille_vocabulaire)
   phi_k_y0 = (word_counts_in_ham + 1) / (total_words_in_ham + taille_vocabulaire)

4. Retourner phi_y, phi_k_y0, phi_k_y1
```

##### Code Python

Ici, nous supposons que les données d'entrée X sont une matrice de comptage de mots (term-frequency matrix).

```python
import numpy as np

class MultinomialNaiveBayes:
    """
    Un classifieur Naïf de Bayes pour le modèle d'événement multinomial.
    Ce modèle est adapté aux comptages de caractéristiques (fréquence des mots).
    """

    def __init__(self, laplace_smoothing=1.0):
        """
        Initialise le classifieur.
        :param laplace_smoothing: La constante de lissage (alpha). 1.0 correspond au lissage de Laplace standard.
        """
        self.laplace_smoothing = laplace_smoothing
        self.phi_y = None
        self.phi_k_y0 = None
        self.phi_k_y1 = None
        self.classes_ = None

    def fit(self, X, y):
        """
        Entraîne le modèle Naive Bayes sur les données d'entraînement.
        :param X: Matrice de comptage de mots de forme (n_samples, n_features).
        :param y: Vecteur d'étiquettes de forme (n_samples,). Les étiquettes doivent être 0 ou 1.
        """
        n_samples, n_features = X.shape
        self.classes_ = np.unique(y)
        if not np.array_equal(self.classes_, [0, 1]):
            raise ValueError("Les étiquettes y doivent être 0 ou 1.")

        # Séparer les données par classe
        X_y0 = X[y == 0]
        X_y1 = X[y == 1]

        # Calculer le prior de classe P(y=1)
        self.phi_y = X_y1.shape[0] / n_samples

        # Calculer les probabilités conditionnelles P(mot=k|y) avec lissage
        # Lissage : ajouter 'alpha' au numérateur et 'n_features * alpha' au dénominateur
        
        # Compter le nombre total d'occurrences de chaque mot dans chaque classe
        word_counts_in_y0 = X_y0.sum(axis=0)
        word_counts_in_y1 = X_y1.sum(axis=0)
        
        # Compter le nombre total de mots dans chaque classe
        total_words_in_y0 = word_counts_in_y0.sum()
        total_words_in_y1 = word_counts_in_y1.sum()

        # Appliquer le lissage
        self.phi_k_y0 = (word_counts_in_y0 + self.laplace_smoothing) / (total_words_in_y0 + n_features * self.laplace_smoothing)
        self.phi_k_y1 = (word_counts_in_y1 + self.laplace_smoothing) / (total_words_in_y1 + n_features * self.laplace_smoothing)

    def _log_likelihood(self, X):
        """Calcule le log de la vraisemblance P(x|y) pour chaque classe."""
        # log(P(x|y)) = log(P(x1, x2,...|y)) = sum_k N_k * log(P(mot=k|y))
        # où N_k est le nombre d'occurrences du mot k dans le document.
        # C'est une multiplication matricielle.
        
        log_prob_y1 = X @ np.log(self.phi_k_y1).T
        log_prob_y0 = X @ np.log(self.phi_k_y0).T
        
        return log_prob_y0, log_prob_y1

    def predict_proba(self, X):
        """
        Calcule les probabilités a posteriori pour chaque classe.
        :param X: Matrice de données à prédire de forme (n_samples, n_features).
        :return: Matrice de probabilités de forme (n_samples, 2).
        """
        if self.phi_y is None:
            raise RuntimeError("Le modèle doit être entraîné avant de pouvoir prédire.")

        log_prob_y0, log_prob_y1 = self._log_likelihood(X)
        
        # Calculer les log-posteriors
        log_posterior_y1 = log_prob_y1 + np.log(self.phi_y)
        log_posterior_y0 = log_prob_y0 + np.log(1 - self.phi_y)
        
        # Normalisation (comme dans le modèle Bernoulli)
        max_log = np.maximum(log_posterior_y0, log_posterior_y1)
        exp_y0 = np.exp(log_posterior_y0 - max_log)
        exp_y1 = np.exp(log_posterior_y1 - max_log)
        
        prob_y1 = exp_y1 / (exp_y0 + exp_y1)
        prob_y0 = 1 - prob_y1
        
        return np.vstack((prob_y0, prob_y1)).T

    def predict(self, X):
        """
        Prédit la classe pour chaque échantillon dans X.
        :param X: Matrice de données à prédire de forme (n_samples, n_features).
        :return: Vecteur de prédictions de classe (0 ou 1).
        """
        probas = self.predict_proba(X)
        return (probas[:, 1] > 0.5).astype(int)
```

### 5. Insights & Synthèse sur Naive Bayes

#### L'Importance de l'Histoire Générative (The Generative Story)

La distinction entre le modèle Bernoulli et le modèle Multinomial n'est pas une simple technicité. Elle révèle une divergence fondamentale dans la manière dont nous concevons la création des données textuelles. Comprendre cette "histoire générative" est la clé pour choisir le bon modèle et interpréter ses résultats.

Le modèle Bernoulli pose la question suivante : "Pour un document de la classe 'spam', quelle est la probabilité que le mot 'gratuit' y apparaisse?" C'est une propriété du document dans son ensemble, un événement unique. Le document est un ensemble de décisions binaires sur la présence ou l'absence de chaque mot du vocabulaire.

En revanche, le modèle Multinomial pose une question différente : "Pour un document de la classe 'spam', si je pioche un mot au hasard, quelle est la probabilité que ce mot soit 'gratuit'?" C'est une propriété des mots au sein du document. Le document est vu comme le résultat d'une série de tirages d'un grand dé à mots.

Cette différence conceptuelle a des conséquences directes. Elle mène à des paramétrisations différentes (probabilité de présence vs. probabilité d'occurrence), à des méthodes de comptage différentes, et finalement, à des profils de performance distincts, comme l'ont démontré empiriquement McCallum et Nigam. Le modèle multinomial, en capturant la fréquence, s'aligne mieux sur l'intuition que la répétition d'un mot est un signal fort, ce qui explique sa supériorité générale dans les tâches de classification de texte.

#### De la Théorie à la Stratégie Pratique

La discussion sur Naive Bayes offre également des leçons précieuses sur la stratégie de développement en apprentissage automatique. Le conseil pratique de "commencer par implémenter quelque chose de rapide et simple" et "d'utiliser l'analyse des erreurs pour guider le développement" trouve un écho direct dans notre exploration des modèles Naive Bayes.

Imaginons un praticien qui aborde un nouveau problème de classification de texte.

**Première itération (Rapide et simple)** : Il pourrait commencer par implémenter un classifieur Bernoulli Naive Bayes. Sa représentation de caractéristiques (un vecteur binaire de taille fixe) semble plus simple à gérer au premier abord. C'est son modèle "quick and dirty".

**Analyse des erreurs** : Après avoir entraîné et testé ce premier modèle, il examine les e-mails que le système a mal classés. Il pourrait remarquer une tendance : le classifieur échoue souvent sur des e-mails où des mots-clés comme "offre", "garanti" ou "urgent" sont répétés de nombreuses fois.

**Génération d'hypothèses** : Cette observation mène à une hypothèse claire : la fréquence des mots est un signal puissant que son modèle actuel ignore complètement.

**Deuxième itération (Amélioration ciblée)** : Fort de cette analyse, il a maintenant une justification solide pour investir du temps dans un modèle plus complexe qui peut capturer cette information. Il se tourne naturellement vers le modèle Multinomial.

Ce flux de travail pratique, préconisé pour les projets d'apprentissage automatique, reflète en fait le parcours de découverte académique qui a conduit de la conception d'un modèle simple à celle d'un modèle plus performant. La stratégie pratique est un microcosme de la méthode scientifique en action : construire un modèle, observer ses échecs, formuler une hypothèse pour expliquer ces échecs, et construire un nouveau modèle pour tester cette hypothèse. Naive Bayes, en plus d'être un classifieur efficace, est donc aussi un excellent outil pédagogique pour illustrer ce cycle itératif de développement.

---

## Partie II : Introduction aux Machines à Vecteurs de Support (SVM)

### 1. Motivation et Objectifs des SVMs

#### Au-delà des Frontières Linéaires

Les modèles que nous avons étudiés jusqu'à présent, comme la régression logistique ou l'analyse discriminante gaussienne (GDA), sont extrêmement puissants mais ont une tendance naturelle à produire des frontières de décision linéaires. Pour de nombreux problèmes du monde réel, les données ne sont pas si bien ordonnées. Les classes peuvent être entremêlées de manière complexe, nécessitant des frontières de décision non linéaires pour être séparées avec précision.

Une approche pour gérer la non-linéarité consiste à créer manuellement des caractéristiques polynomiales ou d'interaction. Par exemple, à partir de caractéristiques $x_1$ et $x_2$, on pourrait créer $x_1^2$, $x_2^2$, $x_1 x_2$, etc., et ensuite appliquer un classifieur linéaire sur ce nouvel ensemble de caractéristiques de plus grande dimension. Cependant, cette approche est fastidieuse et peu systématique. Comment savoir quelles combinaisons de caractéristiques créer? Faut-il aller jusqu'au degré 3, 4, ou plus? Cela demande une expertise du domaine et beaucoup d'essais et d'erreurs.

#### L'Idée Centrale des SVMs

Les machines à vecteurs de support (SVM) proposent une solution beaucoup plus élégante et puissante à ce problème. L'idée centrale est conceptuellement brillante : au lieu d'essayer de trouver une courbe de décision complexe dans l'espace des caractéristiques d'origine, nous allons projeter les données dans un espace de caractéristiques de très grande dimension (voire de dimension infinie). L'astuce est que dans cet espace de plus haute dimension, les données qui n'étaient pas linéairement séparables peuvent le devenir. Une fois les données projetées, il suffit de trouver un simple hyperplan linéaire pour les séparer dans ce nouvel espace.

Cette approche déplace la complexité du modèle de la forme de la frontière de décision vers la transformation des caractéristiques elle-même. Comme nous le verrons, les SVMs parviennent à réaliser cette projection et cette classification de manière incroyablement efficace grâce à "l'astuce du noyau" (kernel trick).

#### Contexte Académique : L'Article de Cortes & Vapnik (1995)

L'article qui a véritablement lancé les SVMs sur le devant de la scène de l'apprentissage automatique est "Support-vector networks" de Corinna Cortes et Vladimir Vapnik, publié en 1995.

Avant ce travail, le concept de recherche d'un hyperplan optimal qui maximise la marge entre les classes existait, mais il était limité au cas où les données d'entraînement étaient parfaitement et linéairement séparables. C'était une curiosité théorique avec une applicabilité limitée, car les données du monde réel sont presque toujours bruitées et non séparables.

La contribution révolutionnaire de Cortes et Vapnik a été d'introduire la notion de "marge souple" (soft margin). En introduisant des variables d'ajustement ("slack variables"), leur formulation permettait à l'algorithme de tolérer certaines erreurs de classification. Il pouvait trouver un hyperplan qui équilibre deux objectifs : maximiser la marge et minimiser le nombre de points mal classés. Cette extension a rendu l'algorithme robuste, théoriquement fondé et extraordinairement efficace sur des problèmes réels. Cet article a transformé les SVMs en l'un des outils de classification les plus influents et les plus performants de la fin du 20e et du début du 21e siècle.

### 2. Le Classifieur à Marge Optimale (Cas Linéairement Séparable)

Pour construire l'intuition des SVMs, nous allons commencer par le cas le plus simple : des données qui sont linéairement séparables. Notre objectif est de trouver le "meilleur" hyperplan séparateur.

#### L'Intuition : La "Rue la Plus Large"

Imaginez un ensemble de points appartenant à deux classes, que l'on peut séparer par une ligne droite. Il existe en fait une infinité de lignes droites qui peuvent séparer parfaitement ces deux classes. Laquelle choisir?

Considérons deux lignes possibles : une ligne rouge qui passe très près de certains points, et une ligne verte qui se place au milieu de l'espace vide entre les deux classes. Intuitivement, la ligne verte semble être un meilleur choix. Elle est plus robuste. Si de nouveaux points de données apparaissent légèrement décalés par rapport aux points d'entraînement, la ligne verte a plus de chances de les classer correctement. La ligne rouge, étant si proche des données existantes, est plus susceptible de faire des erreurs.

L'objectif du classifieur à marge optimale est de formaliser cette intuition. Nous voulons trouver l'hyperplan qui maximise la "marge", c'est-à-dire la distance entre l'hyperplan et les points de données les plus proches de chaque classe. C'est comme si nous essayions de tracer la "rue la plus large" possible qui sépare les deux quartiers de points.

#### Nouvelle Notation pour les SVMs

Pour faciliter le développement mathématique des SVMs, nous adoptons quelques conventions de notation :

**Étiquettes de classe :** Nous changeons les étiquettes de classe de $\{0,1\}$ à $\{-1,+1\}$. Cela simplifiera l'expression de la marge.

**Paramètres de l'hyperplan :** Un hyperplan dans un espace de dimension $d$ est défini par un vecteur de poids $w \in \mathbb{R}^d$ et un terme de biais (ou intercept) $b \in \mathbb{R}$. L'équation de l'hyperplan est $w^T x + b = 0$. Nous séparons explicitement le biais $b$ du vecteur de poids $w$, contrairement à la régression logistique où l'on intégrait souvent le biais dans $w$ en ajoutant une caractéristique $x_0 = 1$.

**Hypothèse de classification :** La prédiction du classifieur est donnée par le signe de la sortie de l'hyperplan : $h(x) = \text{sign}(w^T x + b)$. Si $w^T x + b > 0$, on prédit $+1$. Si $w^T x + b < 0$, on prédit $-1$.

#### La Marge Fonctionnelle ($\hat{\gamma}$)

Nous avons besoin d'une mesure pour quantifier à quel point un exemple est correctement classé. C'est le rôle de la marge fonctionnelle.

**Définition :** Pour un unique exemple d'entraînement $(x_i, y_i)$, la marge fonctionnelle de cet exemple par rapport à l'hyperplan $(w,b)$ est définie comme :

$$\hat{\gamma}_i = y_i(w^T x_i + b)$$

**Interprétation :**
- Si $y_i = +1$, nous voulons que $w^T x_i + b$ soit un grand nombre positif. Dans ce cas, $\hat{\gamma}_i$ est un grand nombre positif.
- Si $y_i = -1$, nous voulons que $w^T x_i + b$ soit un grand nombre négatif. Comme $y_i = -1$, le produit $\hat{\gamma}_i$ est également un grand nombre positif.

Ainsi, une marge fonctionnelle $\hat{\gamma}_i$ large et positive signifie que l'exemple $i$ est classé correctement et avec une grande "confiance". Si $\hat{\gamma}_i > 0$, l'exemple est correctement classé.

**Le Défaut :** Le problème majeur de la marge fonctionnelle est qu'elle n'est pas invariante à l'échelle. Si nous prenons notre hyperplan $(w,b)$ et que nous le remplaçons par $(2w,2b)$, la frontière de décision $2w^T x + 2b = 0$ est exactement la même. Cependant, la marge fonctionnelle de chaque point est doublée : $y_i(2w^T x_i + 2b) = 2\hat{\gamma}_i$. Nous pouvons rendre la marge fonctionnelle arbitrairement grande en multipliant $w$ et $b$ par une constante positive, sans pour autant améliorer la qualité réelle du classifieur. La marge fonctionnelle est donc une mesure peu fiable de la distance géométrique réelle.

#### La Marge Géométrique ($\gamma$)

Pour obtenir une mesure de distance réelle et invariante à l'échelle, nous introduisons la marge géométrique. C'est la distance euclidienne réelle entre un point $x_i$ et l'hyperplan de décision $w^T x + b = 0$.

**Définition :** La marge géométrique d'un exemple $(x_i, y_i)$ par rapport à l'hyperplan $(w,b)$ est :

$$\gamma_i = \frac{y_i(w^T x_i + b)}{\|w\|}$$

où $\|w\|$ est la norme euclidienne (ou L2) du vecteur de poids $w$.

**Interprétation :** Cette quantité représente la distance physique, signée, du point à l'hyperplan. La division par $\|w\|$ normalise le vecteur de poids, ce qui rend la mesure invariante à l'échelle. Si nous remplaçons $(w,b)$ par $(2w,2b)$, la marge géométrique reste inchangée :

$$\frac{y_i(2w^T x_i + 2b)}{\|2w\|} = \frac{2y_i(w^T x_i + b)}{2\|w\|} = \gamma_i$$

C'est cette marge géométrique, la "largeur de la rue", que nous cherchons à maximiser.

**Relation Clé :** La relation entre les deux marges est simple et fondamentale :

$$\text{Marge Géométrique} = \frac{\text{Marge Fonctionnelle}}{\|w\|}$$

### 3. Formulation du Problème d'Optimisation

Notre objectif est maintenant clair : trouver les paramètres $(w,b)$ de l'hyperplan qui maximisent la marge géométrique minimale sur l'ensemble des données d'entraînement.

#### L'Objectif : Maximiser la Marge Géométrique

La marge géométrique de l'ensemble du training set est définie comme la marge du point le plus mal loti (le plus proche de la frontière) :

$$\gamma = \min_{i=1,\ldots,m} \gamma_i$$

Nous voulons donc résoudre le problème d'optimisation suivant :

$$\max_{w,b} \gamma$$
$$\text{tel que } \frac{y_i(w^T x_i + b)}{\|w\|} \geq \gamma \text{ pour tout } i=1,\ldots,m$$

Ce problème peut être réécrit comme :

$$\max_{w,b,\hat{\gamma}} \frac{\hat{\gamma}}{\|w\|}$$
$$\text{tel que } y_i(w^T x_i + b) \geq \hat{\gamma} \text{ pour tout } i=1,\ldots,m$$

où $\hat{\gamma}$ est la marge fonctionnelle minimale.

#### La Transformation en Problème Convexe (L'Étape Clé)

Le problème d'optimisation ci-dessus n'est pas facile à résoudre directement à cause du terme $\hat{\gamma}/\|w\|$ et de la nature couplée des variables. C'est ici qu'intervient une astuce mathématique d'une grande élégance.

**Étape 1 (Normalisation) :** Nous exploitons la propriété de mise à l'échelle de la marge fonctionnelle. Puisque nous pouvons mettre à l'échelle $w$ et $b$ par n'importe quelle constante sans changer la frontière de décision, nous pouvons imposer une contrainte de normalisation. Nous choisissons de mettre à l'échelle $w$ et $b$ de telle sorte que la marge fonctionnelle du (ou des) point(s) le(s) plus proche(s) de l'hyperplan soit exactement égale à 1. Pour ces points, que l'on appellera plus tard les vecteurs de support, on a donc $\hat{\gamma} = 1$.

**Étape 2 (Simplification) :** Avec cette normalisation ($\hat{\gamma} = 1$), la marge géométrique que nous voulons maximiser devient $\gamma = \hat{\gamma}/\|w\| = 1/\|w\|$. La contrainte pour tous les autres points (qui sont plus éloignés de la frontière) devient $y_i(w^T x_i + b) \geq 1$.

**Étape 3 (Reformulation) :** Notre objectif est maintenant de maximiser $1/\|w\|$. C'est équivalent à minimiser $\|w\|$. Pour des raisons de commodité mathématique (cela nous donne une fonction objectif quadratique qui est plus facile à dériver et à optimiser), nous choisissons de minimiser $\frac{1}{2}\|w\|^2$. Le facteur $\frac{1}{2}$ et le carré ne changent pas l'emplacement du minimum mais simplifient le calcul du gradient.

#### La Formulation Finale du Classifieur à Marge Optimale

Après cette série de transformations, nous arrivons à la formulation standard et finale du problème du classifieur à marge optimale :

$$\min_{w,b} \frac{1}{2}\|w\|^2$$
$$\text{tel que } y_i(w^T x_i + b) \geq 1 \text{ pour tout } i=1,\ldots,m$$

Cette formulation est magnifique pour plusieurs raisons :

1. C'est un problème d'optimisation quadratique convexe. La fonction objectif ($\frac{1}{2}\|w\|^2$) est convexe, et les contraintes (linéaires en $w$ et $b$) définissent un domaine de recherche convexe.
2. Cela signifie qu'il n'y a pas de minima locaux. Il n'existe qu'un seul minimum global.
3. Par conséquent, nous pouvons utiliser des solveurs d'optimisation standard et efficaces (appelés solveurs QP) pour trouver la solution unique et optimale, c'est-à-dire le "meilleur" hyperplan.

#### Implémentation

L'implémentation directe d'un solveur QP est complexe et sort du cadre de cette leçon. Cependant, il est utile de comprendre le pseudocode conceptuel.

##### Pseudocode

```
Algorithme : Trouver le Classifieur à Marge Optimale

Entrées :
  - X : Matrice d'entraînement (m documents, N caractéristiques)
  - y : Vecteur d'étiquettes (m documents, valeurs dans {-1, +1})

Sorties :
  - w_optimal : Vecteur de poids de l'hyperplan optimal
  - b_optimal : Biais de l'hyperplan optimal

1. Définir le problème d'optimisation quadratique :
   - Fonction objectif : f(w, b) = (1/2) * w^T * w
   - Contraintes d'inégalité (pour i de 1 à m) :
     1 - y_i * (w^T * x_i + b) <= 0

2. Utiliser un solveur d'optimisation quadratique (QP solver) standard pour trouver les valeurs de w et b qui minimisent f(w, b) tout en respectant toutes les contraintes.

3. Retourner les paramètres w_optimal et b_optimal trouvés par le solveur.
```

### 4. Vers les SVMs Modernes : Noyaux et Données Non-Séparables

Le classifieur à marge optimale est le bloc de construction fondamental des SVMs, mais il ne fonctionne que pour les données linéairement séparables. Deux extensions majeures le transforment en l'outil puissant que nous connaissons aujourd'hui.

#### L'Astuce du Noyau (The Kernel Trick)

L'astuce du noyau est l'une des idées les plus puissantes de l'apprentissage automatique. Elle permet aux SVMs de créer des frontières de décision non linéaires de manière efficace. L'idée est la suivante :

1. On peut montrer que la solution du problème d'optimisation SVM et la prédiction finale ne dépendent que des produits scalaires entre les vecteurs de données $(x_i^T x_j)$.

2. L'astuce du noyau consiste à remplacer ce produit scalaire par une fonction noyau $K(x_i, x_j)$.

3. Cette fonction noyau $K(x_i, x_j)$ calcule le produit scalaire des données après qu'elles aient été projetées dans un espace de caractéristiques de très grande dimension par une transformation $\Phi(x)$ : $K(x_i, x_j) = \Phi(x_i)^T \Phi(x_j)$.

4. Le point crucial est que nous pouvons calculer $K(x_i, x_j)$ directement à partir des données originales $x_i$ et $x_j$, sans jamais avoir à calculer explicitement la transformation $\Phi(x)$, qui pourrait être de dimension infinie.

Cela permet aux SVMs d'apprendre des frontières de décision très complexes (correspondant à des hyperplans dans des espaces de dimension quasi-infinie) de manière computationnellement efficace.

#### Le Cas Non-Séparable (Marge Souple)

Comme mentionné précédemment, la contribution majeure de Cortes et Vapnik a été de gérer les données non séparables. L'idée de la marge souple est d'autoriser certaines violations des contraintes. On introduit des variables d'ajustement (slack variables) $\xi_i \geq 0$ dans le problème d'optimisation :

$$\min_{w,b,\xi} \frac{1}{2}\|w\|^2 + C\sum_{i=1}^{m} \xi_i$$
$$\text{tel que } y_i(w^T x_i + b) \geq 1 - \xi_i \text{ et } \xi_i \geq 0 \text{ pour tout } i$$

Le terme $C\sum \xi_i$ est un terme de pénalité. Le paramètre $C$ est un hyperparamètre qui contrôle le compromis entre deux objectifs :

1. Avoir une grande marge (minimiser $\|w\|^2$).
2. Avoir peu d'erreurs de classification (minimiser la somme des $\xi_i$).

Un petit $C$ favorise une grande marge au détriment de quelques erreurs, tandis qu'un grand $C$ pénalise fortement les erreurs, quitte à avoir une marge plus petite.

### 5. Insights & Synthèse sur les SVMs

#### De l'Intuition Géométrique à l'Optimisation Convexe

Le développement du classifieur à marge optimale est un exemple magistral de la manière dont une idée intuitive peut être transformée en un problème mathématique rigoureux et résoluble. Ce parcours est au cœur de l'élégance des SVMs.

**Le Point de Départ (L'Intuition)** : Tout commence par une idée simple et visuelle : la meilleure frontière est celle qui est la plus éloignée possible de tous les points, créant la "rue la plus large".

**La Métrique (La Formalisation)** : Cette "distance" est formalisée par la marge géométrique $\gamma$. On montre que c'est une mesure robuste, invariante à l'échelle, contrairement à la marge fonctionnelle $\hat{\gamma}$ qui est défectueuse.

**L'Astuce (La Normalisation)** : La réalisation clé est que la nature arbitrairement scalable de la marge fonctionnelle peut être exploitée. En fixant sa valeur à 1 pour les points les plus proches, sans perte de généralité, on simplifie radicalement le problème. C'est l'étape de normalisation qui rend le problème tractable.

**La Formulation Finale (L'Optimisation)** : Cette normalisation transforme élégamment le problème de maximisation de $1/\|w\|$ en un problème standard et bien compris d'optimisation quadratique convexe : la minimisation de $\frac{1}{2}\|w\|^2$. Ce passage d'un concept visuel à une formulation mathématique résoluble est ce qui confère aux SVMs leur puissance et leur beauté théorique.

#### Le Paradigme Génératif vs. Discriminatif (Revisité)

La comparaison entre Naive Bayes et les SVMs met en lumière de manière éclatante la différence entre les approches générative et discriminative.

Naive Bayes, en tant que modèle génératif, tente d'apprendre la distribution complète des données. Il modélise ce à quoi ressemblent les e-mails de spam et les e-mails légitimes dans leur intégralité. Chaque mot, chaque caractéristique contribue au calcul de la vraisemblance.

Le SVM, en tant que modèle discriminatif, a une vision beaucoup plus ciblée. Il ignore complètement la distribution des points "faciles" qui sont loin de la frontière de décision. Sa solution finale, l'hyperplan défini par $w$ et $b$, est entièrement déterminée par un petit sous-ensemble des points d'entraînement : les vecteurs de support, qui sont les points les plus difficiles, ceux qui se trouvent exactement sur la marge ou qui la violent.

C'est une différence philosophique fondamentale. Naive Bayes investit ses efforts dans la modélisation de l'ensemble des données. Le SVM concentre tous ses efforts sur la tâche critique de définir la frontière. C'est pourquoi les SVMs sont souvent appelés classifieurs "à grande marge" et pourquoi ils obtiennent fréquemment une meilleure capacité de généralisation. En se concentrant uniquement sur les cas limites, ils sont moins susceptibles d'être influencés par la distribution globale des données et peuvent trouver une règle de séparation plus robuste.

---

## Conclusion Générale

Au cours de cette leçon, nous avons exploré deux des algorithmes les plus fondamentaux et les plus influents de l'apprentissage automatique, chacun représentant une philosophie distincte de la classification.

Le classifieur naïf de Bayes s'est révélé être un algorithme probabiliste rapide, simple et remarquablement efficace, en particulier pour les tâches de classification de texte. Sa force réside dans sa simplicité, issue de la forte hypothèse d'indépendance conditionnelle. En comprenant ses variantes, comme les modèles Bernoulli et Multinomial, et en appliquant des techniques de correction comme le lissage de Laplace, nous pouvons construire des classifieurs de base (baselines) très performants en un temps record. Il incarne l'approche générative, cherchant à modéliser la manière dont les données sont créées.

Les machines à vecteurs de support, quant à elles, représentent l'élégance de l'approche discriminative. Motivées par une intuition géométrique claire – la maximisation de la marge entre les classes – elles se traduisent par un problème d'optimisation convexe bien défini. Les SVMs concentrent leur attention sur les points de données les plus difficiles (les vecteurs de support) pour définir une frontière de décision robuste, ce qui leur confère une excellente capacité de généralisation. Avec l'ajout de l'astuce du noyau et de la marge souple, elles deviennent des outils extrêmement puissants capables de gérer des données complexes et non linéaires.

Le contraste final entre ces deux algorithmes est le principal enseignement à retenir. Naive Bayes modélise la probabilité des données ($P(x,y)$), tandis que les SVMs modélisent la frontière de décision ($P(y|x)$). Comprendre cette dichotomie générative/discriminative est une compétence essentielle pour tout praticien de l'apprentissage automatique, car elle guide le choix de l'algorithme, l'interprétation de ses résultats et la stratégie de développement face à un nouveau problème.
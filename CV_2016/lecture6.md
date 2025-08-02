# Conférence sur l'Optimisation et la Régularisation des Réseaux de Neurones, et Introduction aux Réseaux Convolutifs pour la Vision par Ordinateur

Bonjour à toutes et à tous. Aujourd'hui, cette conférence approfondira les techniques essentielles pour l'entraînement efficace des réseaux de neurones, explorant les schémas de mise à jour des paramètres, les stratégies de régularisation et les fondements des réseaux de neurones convolutifs. L'objectif est de fournir une compréhension à la fois théorique et pratique, indispensable pour tout étudiant en vision par ordinateur.

## 1. Introduction : Rappel des Fondamentaux de l'Entraînement des Réseaux de Neurones

L'entraînement des réseaux de neurones est un processus itératif et fondamental qui permet à un modèle d'apprendre des motifs complexes à partir de données. Ce processus peut être décomposé en quatre étapes principales, formant une boucle continue qui affine progressivement les poids du réseau pour améliorer ses performances.

### Le Processus d'Entraînement en Quatre Étapes

La première étape consiste à **échantillonner un petit lot de données** à partir de l'ensemble de données complet d'images et de leurs étiquettes correspondantes. Cette approche par mini-lots est cruciale, car elle permet de gérer de vastes ensembles de données qui ne peuvent pas être traités en une seule fois en mémoire. En traitant des sous-ensembles, le processus d'apprentissage devient plus gérable et plus stable.

Ensuite, ce lot de données est soumis à une **propagation avant** à travers le réseau. Durant cette phase, le réseau génère des prédictions, et une fonction de perte est utilisée pour quantifier l'écart entre ces prédictions et les étiquettes réelles. La valeur de la perte indique la performance actuelle du réseau sur ce lot de données spécifique. C'est le signal d'erreur qui guide l'apprentissage.

La troisième étape est la **rétropropagation**, un mécanisme par lequel le gradient de la fonction de perte est calculé par rapport à tous les poids du réseau. Ce gradient est une information cruciale, car il indique la direction et l'ampleur des ajustements nécessaires pour chaque poids afin de réduire la perte et d'améliorer la classification des images.

Enfin, la dernière étape est la **mise à jour des paramètres**. Le gradient calculé est utilisé pour ajuster les poids du réseau par de petits "coups de pouce". Ce cycle se répète des milliers, voire des millions de fois, permettant au réseau d'apprendre progressivement des représentations de plus en plus précises des données. La nature itérative de ce processus est essentielle, car elle permet un apprentissage adaptatif à partir d'un volume de données trop important pour une analyse simultanée. Chaque itération affine les poids, et cette boucle de rétroaction est au cœur de l'apprentissage par descente de gradient.

### Importance des Fonctions d'Activation

Les fonctions d'activation sont des composants absolument critiques dans les réseaux de neurones. Leur rôle principal est d'introduire de la non-linéarité dans le modèle. Sans ces fonctions, un réseau de neurones, quelle que soit sa profondeur, se réduirait à une simple combinaison linéaire de ses entrées. Une telle architecture aurait une capacité de modélisation extrêmement limitée, équivalente à celle d'un classifieur linéaire de base, incapable de capturer les relations complexes et non linéaires inhérentes à la plupart des jeux de données réels.

C'est la non-linéarité apportée par les fonctions d'activation qui confère au réseau la "flexibilité" nécessaire pour s'adapter à des données complexes et apprendre des motifs sophistiqués. Elles permettent au réseau de "plier" et de "tordre" l'espace des caractéristiques, ce qui est indispensable pour la classification ou la régression de données non-linéairement séparables.

### Initialisation des Poids et Normalisation par Lots (Batch Normalization)

L'initialisation des poids d'un réseau de neurones est un défi subtil mais crucial. La manière dont les poids sont initialisés peut avoir un impact profond sur la stabilité et la vitesse de l'entraînement. Si les poids initiaux sont trop petits, les activations dans un réseau profond peuvent tendre vers zéro, conduisant à des problèmes de "vanishing gradients" (gradients nuls) où le réseau cesse d'apprendre. À l'inverse, si les poids sont trop grands, les activations peuvent exploser, entraînant des "exploding gradients" (gradients explosifs) qui déstabilisent l'entraînement. Ces scénarios peuvent aboutir à des réseaux "supersaturés" ou à des activations toutes nulles, rendant le réglage de l'échelle initiale des poids très délicat.

Pour atténuer ces problèmes, des méthodes d'initialisation spécifiques ont été développées. L'**initialisation de Xavier** est une approche qui vise à maintenir une distribution raisonnable des activations à travers le réseau au début de l'entraînement, offrant un point de départ plus stable pour l'apprentissage.

Cependant, une avancée encore plus significative est la **Normalisation par Lots (Batch Normalization)**. Cette technique résout une grande partie des difficultés liées à l'initialisation précise des poids en normalisant les activations de chaque couche à l'intérieur d'un mini-lot. En normalisant les entrées de chaque couche, Batch Normalization assure que les activations restent dans une plage stable, ce qui rend le processus d'entraînement beaucoup plus robuste au choix de l'échelle initiale des poids. Il ne s'agit pas seulement d'une correction des activations, mais d'une stabilisation de l'ensemble du processus d'entraînement, réduisant la sensibilité aux hyperparamètres d'initialisation et améliorant considérablement la facilité d'utilisation et la performance des réseaux profonds.

## 2. Optimisation des Réseaux de Neurones : Schémas de Mise à Jour des Paramètres

Une fois les gradients calculés par rétropropagation, la manière dont ces gradients sont utilisés pour mettre à jour les paramètres du modèle est cruciale. C'est le rôle des schémas de mise à jour des paramètres, ou optimiseurs. Bien que la descente de gradient stochastique (SGD) soit le concept de base, des méthodes plus élaborées ont été développées pour accélérer et stabiliser la convergence.

### Descente de Gradient Stochastique (SGD)

La Descente de Gradient Stochastique (SGD) est la méthode la plus simple pour mettre à jour les paramètres d'un réseau de neurones. Elle consiste à prendre le gradient calculé à partir d'un mini-lot de données, à le multiplier par un taux d'apprentissage (une petite valeur positive), puis à le soustraire du vecteur des paramètres actuels.

**Pseudo-code : SGD**
```python
# Paramètres d'entrée :
# x : vecteur des paramètres du modèle
# dx : gradient des paramètres
# learning_rate (lr) : taux d'apprentissage

# Mise à jour des paramètres
x = x - lr * dx
```

Malgré sa simplicité, le SGD présente des limites significatives, en particulier dans les paysages de perte complexes. Considérez une fonction de perte dont la surface est très allongée, c'est-à-dire très raide dans une direction (verticale) et très peu profonde dans une autre (horizontale). Si l'optimisation commence dans une région éloignée du minimum, le gradient local sera faible horizontalement et fort verticalement. En appliquant un taux d'apprentissage unique et global à toutes les directions, le SGD aura tendance à progresser trop lentement dans la direction horizontale peu profonde, tout en oscillant excessivement de haut en bas dans la direction verticale raide. Cette incapacité à s'adapter aux différentes échelles de courbure du paysage de perte est la raison fondamentale de sa lenteur et de son comportement en "zigzag", ce qui a motivé la recherche d'optimiseurs plus sophistiqués.

### Momentum

Pour remédier à la lenteur et aux oscillations du SGD, l'algorithme de Momentum a été introduit. Cette méthode modifie la mise à jour des paramètres en introduisant une variable de "vitesse" (souvent notée V). Au lieu d'utiliser directement le gradient pour mettre à jour la position, le gradient est utilisé pour incrémenter cette variable de vitesse. Cette variable V accumule une somme exponentielle des gradients passés, conférant une "inertie" au processus d'optimisation.

L'interprétation physique du Momentum est celle d'une balle roulant sur le paysage de perte, où le gradient représente la force agissant sur la balle. La variable V représente la vitesse de la balle. Le paramètre hyperparamètre μ (généralement entre 0.5 et 0.9) agit comme un terme de "friction", qui décroît la vitesse précédente à chaque itération. Sans ce terme de friction, la balle ne s'arrêterait jamais et continuerait d'osciller indéfiniment autour du minimum, empêchant ainsi la convergence. La friction permet au système de se stabiliser et d'atteindre un point de repos.

Le Momentum est particulièrement efficace dans les directions peu profondes mais cohérentes, où il permet d'accélérer la progression en accumulant la vitesse dans cette direction. Dans les directions raides, il aide à amortir les oscillations en tirant constamment la balle vers le centre, ce qui conduit à une convergence plus rapide et plus stable que le SGD de base.

**Pseudo-code : Momentum**
```python
# Paramètres d'entrée :
# x : vecteur des paramètres du modèle
# dx : gradient des paramètres
# V : vecteur de vitesse (initialisé à zéro)
# learning_rate (lr) : taux d'apprentissage
# mu : coefficient de frottement/décroissance du momentum (e.g., 0.5, 0.9)

# Mise à jour de la vitesse
V = mu * V + lr * dx

# Mise à jour des paramètres
x = x - V
```

### Momentum de Nesterov (NAG)

Le Momentum de Nesterov, ou Nesterov Accelerated Gradient (NAG), est une amélioration astucieuse de l'algorithme de Momentum standard. L'idée clé derrière NAG est d'effectuer un "regard en avant" (one-step look-ahead).

Dans le Momentum standard, le gradient est évalué à la position actuelle des paramètres. Cependant, avec NAG, on sait déjà que le momentum accumulé va porter les paramètres dans une certaine direction. Nesterov propose donc d'évaluer le gradient non pas à la position actuelle x, mais à la position où le momentum est censé nous amener, c'est-à-dire x−μV_précédent. En calculant le gradient à ce point "anticipé", on obtient une direction de gradient légèrement plus précise et mieux informée sur la courbure locale du paysage de perte.

Cette petite amélioration dans la direction du gradient, bien que subtile à chaque étape, s'accumule au fil des itérations et conduit presque toujours à une convergence plus rapide et plus efficace en pratique que le Momentum standard. En théorie, NAG offre également de meilleures garanties de convergence.

**Pseudo-code : NAG (Formulation pratique)**
```python
# Paramètres d'entrée :
# x : vecteur des paramètres du modèle
# dx : gradient des paramètres (calculé au point de "regard en avant")
# V : vecteur de vitesse (initialisé à zéro)
# learning_rate (lr) : taux d'apprentissage
# mu : coefficient de momentum (e.g., 0.9)

# 1. Calculer le point de "regard en avant" (x_ahead)
# C'est là que le momentum nous porterait
x_ahead = x - mu * V 

# 2. Calculer le gradient (dx) à ce point x_ahead
# (Ceci nécessite une passe avant/arrière au point x_ahead)

# 3. Mise à jour de la vitesse
V = mu * V + lr * dx

# 4. Mise à jour des paramètres
x = x - V
```

Il est à noter que la mise en œuvre de Nesterov peut parfois être techniquement complexe car elle nécessite de calculer le gradient à un point qui n'est pas la position actuelle des paramètres. Cependant, des transformations de variables existent pour simplifier son intégration dans les frameworks de calcul de gradient.

### Adagrad

L'optimiseur Adagrad (Adaptive Gradient) est une méthode qui introduit un taux d'apprentissage adaptatif pour chaque paramètre individuel du modèle. Contrairement au SGD ou au Momentum qui appliquent un taux d'apprentissage global, Adagrad maintient un "cache" pour chaque dimension des paramètres. Ce cache accumule la somme des carrés des gradients passés pour cette dimension.

La règle de mise à jour d'Adagrad divise le taux d'apprentissage global par la racine carrée de ce cache (plus une petite valeur ε pour éviter la division par zéro). L'effet est que les dimensions de paramètres qui ont vu de grands gradients (directions raides dans le paysage de perte) auront leur taux d'apprentissage effectif réduit, ce qui conduit à des mises à jour plus petites. Inversement, les dimensions avec de petits gradients (directions peu profondes) conserveront un taux d'apprentissage relativement plus élevé, permettant une progression plus rapide dans ces directions. Adagrad a donc un effet égalisateur, ajustant dynamiquement les pas en fonction de la "raideur" du paysage de perte pour chaque dimension.

**Pseudo-code : Adagrad**
```python
# Paramètres d'entrée :
# x : vecteur des paramètres du modèle
# dx : gradient des paramètres
# cache : vecteur de la somme des carrés des gradients (initialisé à zéro)
# learning_rate (lr) : taux d'apprentissage
# epsilon (eps) : petite valeur pour éviter la division par zéro (e.g., 1e-7)

# Mise à jour du cache (somme des carrés des gradients)
cache = cache + dx * dx

# Mise à jour des paramètres avec taux d'apprentissage adaptatif
x = x - (lr / (sqrt(cache) + eps)) * dx
```

Le principal inconvénient d'Adagrad, cependant, est que le cache ne fait qu'augmenter au fil du temps, car il accumule des nombres positifs. Cela signifie que le taux d'apprentissage effectif pour chaque paramètre tend inévitablement vers zéro, ce qui peut entraîner un arrêt prématuré de l'apprentissage. Ce comportement est acceptable pour certains problèmes d'optimisation convexes où l'objectif est de converger vers un minimum unique et fixe. Cependant, dans le contexte des réseaux de neurones profonds, où le paysage de perte est non convexe et dynamique, le réseau a besoin d'une "énergie continue" pour s'adapter et explorer les données. L'arrêt prématuré de l'apprentissage par Adagrad dans ces scénarios montre que les stratégies d'optimisation doivent être adaptées à la nature non convexe du problème.

### RMSprop

RMSprop est une méthode de mise à jour des paramètres proposée par Geoff Hinton qui vise à résoudre le problème de la décroissance rapide du taux d'apprentissage d'Adagrad. Au lieu d'accumuler une somme cumulative des carrés des gradients, RMSprop utilise un "compteur fuyant" (leaky counter) pour le cache des gradients carrés.

RMSprop introduit un hyperparamètre de "taux de décroissance" (decay_rate), généralement réglé à une valeur proche de 1 (par exemple, 0.99). Ce taux de décroissance permet à l'accumulateur de gradients carrés de "fuir" lentement les informations des gradients passés lointains, donnant plus de poids aux gradients récents. Cela maintient l'adaptabilité du taux d'apprentissage par paramètre, comme Adagrad, mais sans que le taux d'apprentissage ne tende vers zéro, assurant ainsi un apprentissage continu et stable sur de longues périodes.

**Pseudo-code : RMSprop**
```python
# Paramètres d'entrée :
# x : vecteur des paramètres du modèle
# dx : gradient des paramètres
# cache : vecteur de la moyenne mobile des carrés des gradients (initialisé à zéro)
# learning_rate (lr) : taux d'apprentissage
# decay_rate : taux de décroissance (e.g., 0.99)
# epsilon (eps) : petite valeur pour éviter la division par zéro (e.g., 1e-7)

# Mise à jour du cache (moyenne mobile exponentielle des carrés des gradients)
cache = decay_rate * cache + (1 - decay_rate) * dx * dx

# Mise à jour des paramètres avec taux d'apprentissage adaptatif
x = x - (lr / (sqrt(cache) + eps)) * dx
```

L'histoire de RMSprop est assez unique : il a été introduit pour la première fois non pas dans une publication formelle, mais dans une simple diapositive d'un cours Coursera de Jeff Hinton il y a quelques années. Malgré son origine informelle, la méthode s'est avérée si efficace en pratique qu'elle a rapidement été adoptée par la communauté de l'apprentissage profond, et les chercheurs ont même cité cette diapositive dans leurs articles. Cette anecdote met en lumière la culture de l'expérimentation et du partage rapide des "trucs et astuces" qui fonctionnent dans le domaine de l'IA, où l'efficacité empirique peut parfois précéder la théorisation rigoureuse.

### Adam

ADAM (Adaptive Moment Estimation) est un optimiseur largement utilisé qui combine les avantages du Momentum et de RMSprop. Il maintient deux estimations de moments pour les gradients : une estimation du premier moment (la moyenne des gradients bruts, similaire au Momentum) et une estimation du second moment (la moyenne des carrés des gradients, similaire à RMSprop). Ces estimations sont des moyennes mobiles exponentielles, ce qui leur permet de s'adapter aux changements de direction et d'échelle des gradients au fil du temps.

Adam combine ces deux informations pour déterminer la taille et la direction des pas de mise à jour. Les hyperparamètres β₁ (généralement 0.9) et β₂ (généralement 0.999) contrôlent les taux de décroissance pour le premier et le second moment respectivement.

Une caractéristique importante d'Adam est l'inclusion d'une correction de biais pour les estimations des moments. Comme les variables de moment (m et v) sont initialisées à zéro, leurs estimations peuvent être biaisées vers zéro au début de l'entraînement, en particulier pendant les premières itérations. La correction de biais compense cet effet en ajustant les estimations des moments, assurant ainsi des mises à jour plus précises et stables dès le début du processus d'apprentissage. Bien que ce soit un détail mineur, cette correction statistique contribue à la robustesse globale de l'algorithme.

**Pseudo-code : Adam**
```python
# Paramètres d'entrée :
# x : vecteur des paramètres du modèle
# dx : gradient des paramètres
# m : vecteur du premier moment (moyenne des gradients, initialisé à zéro)
# v : vecteur du second moment (moyenne des carrés des gradients, initialisé à zéro)
# learning_rate (lr) : taux d'apprentissage
# beta1 : taux de décroissance pour m (e.g., 0.9)
# beta2 : taux de décroissance pour v (e.g., 0.999)
# epsilon (eps) : petite valeur pour éviter la division par zéro (e.g., 1e-7)
# t : compteur d'itérations (commence à 1)

# Incrémenter le compteur d'itérations
t = t + 1

# Mise à jour des estimations des moments
m = beta1 * m + (1 - beta1) * dx
v = beta2 * v + (1 - beta2) * dx * dx

# Correction de biais pour m et v
m_hat = m / (1 - beta1**t)
v_hat = v / (1 - beta2**t)

# Mise à jour des paramètres
x = x - (lr / (sqrt(v_hat) + eps)) * m_hat
```

Adam est devenu un choix par défaut populaire pour l'entraînement des réseaux de neurones en raison de sa combinaison équilibrée de robustesse et de performance. Il gère efficacement les paysages de perte complexes en stabilisant la direction du gradient et en adaptant la taille des pas pour chaque paramètre, sans nécessiter un réglage excessif des hyperparamètres.

### Décroissance du Taux d'Apprentissage (Learning Rate Decay)

Indépendamment de l'optimiseur choisi, le taux d'apprentissage reste un hyperparamètre fondamental. Une stratégie courante et très efficace consiste à ne pas maintenir un taux d'apprentissage constant tout au long de l'entraînement, mais à le faire décroître au fil du temps.

L'idée est de commencer avec un taux d'apprentissage relativement élevé. Cela permet au réseau de faire des progrès rapides au début de l'entraînement, explorant rapidement le paysage de perte et échappant potentiellement aux minima locaux médiocres. Cependant, un taux d'apprentissage constamment élevé peut rendre la convergence difficile près du minimum, car le système resterait trop "stochastique" et ne parviendrait pas à se "stabiliser" précisément.

Pour cette raison, après un certain nombre d'époques ou d'itérations, le taux d'apprentissage est progressivement réduit. Cela permet au réseau de "rouler" plus finement dans le paysage de perte, de converger plus précisément vers le minimum et de s'y stabiliser. Cette stratégie dynamique optimise le compromis entre l'exploration rapide au début et l'exploitation précise vers la fin de l'entraînement.

Il existe plusieurs méthodes pour implémenter cette décroissance, notamment la décroissance par pas (step decay), où le taux d'apprentissage est réduit d'un facteur (par exemple, 0.9) après chaque époque ou un nombre fixe d'époques. Une autre approche courante est la décroissance exponentielle, où le taux d'apprentissage diminue de manière exponentielle au fil du temps. Bien que la théorie derrière ces décroissances provienne souvent de la littérature sur l'optimisation convexe, en pratique, elles sont largement utilisées et bénéfiques pour l'entraînement des réseaux de neurones profonds.

### Méthodes d'Optimisation du Second Ordre

Jusqu'à présent, les optimiseurs discutés (SGD, Momentum, Adagrad, RMSprop, Adam) sont des méthodes de premier ordre. Cela signifie qu'elles n'utilisent que l'information du gradient (la première dérivée de la fonction de perte), qui indique la pente de la surface de perte dans chaque direction.

Les méthodes d'optimisation du second ordre vont plus loin en utilisant non seulement le gradient, mais aussi la Hessienne (la matrice des dérivées secondes). La Hessienne fournit des informations sur la courbure de la surface de perte. En utilisant ces informations, ces méthodes peuvent former une approximation plus précise de la fonction de perte (par exemple, par un "bol") et sauter directement au minimum de cette approximation.

Le principal avantage des méthodes de second ordre, comme la méthode de Newton, est qu'elles n'ont pas besoin d'un taux d'apprentissage à régler. Puisqu'elles connaissent la pente et la courbure, elles peuvent théoriquement déterminer le pas optimal pour atteindre le minimum de l'approximation en une seule fois, ce qui promet une convergence beaucoup plus rapide.

Cependant, ces méthodes sont généralement impraticables pour l'entraînement des grands réseaux de neurones. Le problème réside dans la taille de la Hessienne. Pour un réseau avec, par exemple, 100 millions de paramètres, la Hessienne serait une matrice de 100 millions par 100 millions. Le calcul et surtout l'inversion de cette matrice sont des opérations massivement coûteuses en temps et en mémoire, les rendant impossibles à réaliser en pratique.

Des algorithmes comme BFGS (Broyden–Fletcher–Goldfarb–Shanno) et L-BFGS (Limited-memory BFGS) tentent d'atténuer ce problème en approximant la Hessienne ou son inverse sans la stocker entièrement. L-BFGS, en particulier, est conçu pour ne pas stocker la Hessienne complète, ce qui le rend plus viable pour des problèmes de taille moyenne.

Néanmoins, même L-BFGS présente des limitations cruciales pour l'apprentissage profond à grande échelle. Il fonctionne très bien pour des fonctions déterministes et de petite taille où toutes les données peuvent être tenues en mémoire et où il n'y a pas de bruit stochastique. Cependant, l'entraînement des réseaux de neurones utilise des mini-lots et est intrinsèquement stochastique. Les approximations de la Hessienne faites par L-BFGS deviennent "incorrectes" lorsque les mini-lots changent, et l'algorithme est sensible aux sources de bruit comme le dropout. Il est également un algorithme "lourd" qui effectue de nombreux appels à la fonction de perte et à son gradient, ce qui est coûteux dans un contexte stochastique.

En conclusion, bien que les méthodes de second ordre offrent des avantages théoriques, leur complexité computationnelle et leur incompatibilité avec la nature stochastique et la haute dimensionnalité des réseaux de neurones profonds les rendent moins pratiques. En pratique, les méthodes de premier ordre comme Adam sont préférées en raison de leur robustesse et de leur efficacité avec des données bruyantes et à grande échelle. Le compromis est souvent de faire plus de pas bruyants mais rapides plutôt que des pas moins nombreux mais plus précis et coûteux.

### Tableau Récapitulatif des Schémas de Mise à Jour des Paramètres

Pour résumer les différents optimiseurs que nous avons explorés, le tableau suivant offre une vue comparative de leurs idées principales, de leurs règles de mise à jour simplifiées, ainsi que de leurs avantages et inconvénients. Ce tableau est conçu pour être une référence rapide, facilitant la compréhension et le choix de l'optimiseur le plus adapté à un problème donné.

| Optimiseur | Idée Principale | Pseudo-code simplifié de la règle de mise à jour | Avantages | Inconvénients |
|------------|-----------------|--------------------------------------------------|-----------|---------------|
| **SGD** | Met à jour les paramètres dans la direction opposée au gradient. | `x = x - lr * dx` | Simple à comprendre et à implémenter. | Lent dans les paysages de perte avec des ravins, oscillations importantes, sensible au taux d'apprentissage. |
| **Momentum** | Ajoute un terme de "vitesse" pour simuler l'inertie, aidant à accélérer dans les directions cohérentes et à amortir les oscillations. | `V = mu * V + lr * dx`<br>`x = x - V` | Accélère la convergence par rapport au SGD, réduit les oscillations. | Nécessite le réglage d'un hyperparamètre supplémentaire (mu). |
| **Nesterov Momentum (NAG)** | Effectue un "regard en avant" pour calculer le gradient, offrant une direction plus précise. | `x_ahead = x - mu * V`<br>`dx = gradient(x_ahead)`<br>`V = mu * V + lr * dx`<br>`x = x - V` | Convergence généralement plus rapide et plus stable que le Momentum standard. | Implémentation légèrement plus complexe conceptuellement. |
| **Adagrad** | Taux d'apprentissage adaptatif par paramètre, réduit les pas pour les dimensions avec de grands gradients. | `cache = cache + dx * dx`<br>`x = x - (lr / (sqrt(cache) + eps)) * dx` | Adapte automatiquement le taux d'apprentissage pour chaque paramètre, bon pour les données éparses. | Le taux d'apprentissage tend vers zéro trop rapidement, arrêtant l'apprentissage dans les problèmes non convexes. |
| **RMSprop** | Utilise une moyenne mobile exponentielle des carrés des gradients pour le taux d'apprentissage adaptatif, évitant la décroissance vers zéro. | `cache = decay_rate * cache + (1 - decay_rate) * dx * dx`<br>`x = x - (lr / (sqrt(cache) + eps)) * dx` | Résout le problème de décroissance d'Adagrad, maintient l'adaptabilité. | Nécessite le réglage d'un hyperparamètre decay_rate. |
| **Adam** | Combine les avantages du Momentum et de RMSprop, avec des estimations de premier et second moment des gradients et une correction de biais. | `m = beta1 * m + (1 - beta1) * dx`<br>`v = beta2 * v + (1 - beta2) * dx * dx`<br>`m_hat = m / (1 - beta1**t)`<br>`v_hat = v / (1 - beta2**t)`<br>`x = x - (lr / (sqrt(v_hat) + eps)) * m_hat` | Excellent choix par défaut, robuste, adapte les pas et la direction, converge rapidement. | Peut être sensible aux hyperparamètres beta1 et beta2 dans certains cas extrêmes, bien que les valeurs par défaut soient souvent bonnes. |

## 3. Techniques de Régularisation et d'Amélioration de la Performance

Au-delà des optimiseurs, d'autres techniques sont essentielles pour améliorer la généralisation des modèles et prévenir le surapprentissage, en particulier dans les réseaux de neurones profonds.

### Ensembles de Modèles (Model Ensembles)

Une méthode éprouvée pour améliorer la performance d'un modèle est l'utilisation d'ensembles de modèles. Le principe est simple : au lieu d'entraîner un seul modèle, on entraîne plusieurs modèles indépendants sur le même ensemble de données d'entraînement. Au moment du test, les prédictions de ces modèles sont moyennées ou combinées d'une autre manière (par exemple, par vote majoritaire pour la classification). Cette approche conduit presque systématiquement à une amélioration de la performance, souvent de l'ordre de 2%.

L'efficacité des ensembles de modèles repose sur le principe de la "sagesse des foules". Chaque modèle individuel, même s'il est entraîné sur les mêmes données, peut apprendre des motifs légèrement différents ou présenter des biais spécifiques. En combinant leurs prédictions, les erreurs ou les faiblesses d'un modèle peuvent être compensées par les forces des autres, conduisant à une prédiction agrégée plus robuste, plus fiable et mieux généralisable.

Le principal inconvénient de cette méthode est le coût computationnel au moment du test. Puisqu'il faut effectuer des passes avant pour chaque modèle de l'ensemble, le temps d'inférence augmente linéairement avec le nombre de modèles.

Pour atténuer ce problème, des techniques d'ensemble "fictives" ou "légères" ont été développées. Une approche consiste à utiliser différents points de contrôle (checkpoints) d'un même modèle entraîné comme membres de l'ensemble. Au lieu d'entraîner sept modèles indépendants, on entraîne un seul modèle et on sauvegarde ses paramètres à différents moments de l'entraînement (par exemple, à la fin de chaque époque). Ces différents checkpoints peuvent ensuite être utilisés comme un ensemble, offrant un gain de performance sans le coût d'entraînement de multiples modèles distincts.

Une autre technique est la moyenne exponentielle des paramètres (Exponentially Decaying Average of Parameters), parfois appelée "X_test". Cette méthode consiste à maintenir une moyenne mobile exponentielle du vecteur de paramètres du modèle pendant l'entraînement. Cette moyenne est ensuite utilisée pour l'évaluation sur les données de validation ou de test. L'interprétation est que, si l'optimisation oscille autour du minimum d'une fonction de perte en forme de bol, la moyenne de ces oscillations se rapprochera davantage du véritable minimum que n'importe quelle position instantanée. Cette approche offre souvent une légère amélioration des performances sans coût supplémentaire significatif au moment de l'inférence.

### Dropout

Le Dropout est une technique de régularisation extrêmement importante et largement utilisée pour les réseaux de neurones, en particulier les réseaux profonds. Son principe est simple mais contre-intuitif : pendant la phase d'entraînement, un pourcentage aléatoire de neurones (par exemple, 50%) est temporairement "désactivé" ou "abandonné" (mis à zéro) lors de chaque passe avant.

Concrètement, pour une couche cachée, après le calcul des activations, un masque binaire aléatoire (composé de zéros et de uns) est généré. Ce masque est ensuite multiplié élément par élément avec les activations de la couche, ce qui a pour effet de mettre à zéro la sortie de certains neurones. Lors de la rétropropagation, les gradients ne circulent pas à travers les neurones désactivés, ce qui signifie que leurs poids ne sont pas mis à jour pour cette itération.

Le Dropout peut être interprété de plusieurs manières, toutes contribuant à ses bénéfices :

1. **Prévention du surapprentissage** : En désactivant aléatoirement des neurones, le Dropout réduit la capacité effective du réseau à chaque itération. Cela rend plus difficile pour le modèle de "mémoriser" les données d'entraînement et l'oblige à apprendre des représentations plus robustes et généralisables.

2. **Redondance forcée** : Le Dropout empêche les neurones de co-adapter de manière excessive ou de devenir trop dépendants d'autres neurones spécifiques. Chaque neurone est contraint de développer des représentations plus robustes et redondantes, car il ne peut pas compter sur la présence constante d'autres neurones. Si un neurone est désactivé, le réseau doit toujours pouvoir faire une prédiction correcte en utilisant les neurones restants.

3. **Ensemble de sous-réseaux** : Le Dropout peut être vu comme l'entraînement d'un vaste ensemble de "sous-réseaux" qui partagent des paramètres. À chaque itération et pour chaque mini-lot, un sous-ensemble différent de neurones est activé, créant un sous-réseau unique. Ce sous-réseau est ensuite entraîné sur le point de données actuel. C'est comme entraîner un ensemble massif de modèles avec des paramètres partagés, ce qui améliore la robustesse globale.

#### Ajustement au moment du test (Scaling)

Un aspect crucial du Dropout est la gestion de l'inférence au moment du test. Pendant l'entraînement, les neurones sont désactivés avec une probabilité 1−P (où P est la probabilité de maintien, par exemple 0.5). Cela signifie que l'activité attendue d'un neurone pendant l'entraînement est réduite. Si tous les neurones étaient activés au moment du test sans compensation, les activations seraient en moyenne plus élevées qu'elles ne l'étaient pendant l'entraînement, ce qui pourrait modifier la distribution de sortie du réseau et nuire aux performances.

Pour compenser cela, les activations des neurones doivent être mises à l'échelle au moment du test. Si un neurone a une probabilité P d'être maintenu pendant l'entraînement, son activation au moment du test doit être multipliée par P. Cela garantit que l'activité attendue de chaque neurone est la même en entraînement et en test, maintenant ainsi la cohérence des distributions d'activation.

#### Dropout Inversé (Inverted Dropout)

La méthode la plus courante et recommandée pour implémenter le Dropout est le Dropout Inversé. Plutôt que de mettre à l'échelle les activations au moment du test, le scaling est effectué pendant la phase d'entraînement.

Avec le Dropout Inversé, lorsque les neurones sont désactivés, les activations des neurones restants sont directement divisées par la probabilité de maintien P (c'est-à-dire multipliées par 1/P). Par exemple, si P=0.5, les activations des neurones actifs sont doublées. Cela "booste" artificiellement les activations pendant l'entraînement de manière à ce que, en moyenne, l'activité attendue reste la même que si aucun neurone n'avait été désactivé. L'avantage majeur est qu'au moment du test, aucune modification n'est nécessaire au code du réseau ; les activations sont utilisées telles quelles, ce qui rend l'inférence plus rapide et plus simple.

**Pseudo-code : Dropout Inversé (Pendant l'entraînement)**
```python
# Paramètres d'entrée :
# h : activations de la couche cachée
# p : probabilité de maintien d'un neurone (e.g., 0.5)

# 1. Générer un masque binaire (u)
# Chaque élément de u est 1 avec probabilité p, et 0 avec probabilité (1-p)
u = (np.random.rand(*h.shape) < p) / p

# 2. Appliquer le masque et le scaling aux activations
h_dropped = h * u

# Au moment du test, les activations sont utilisées telles quelles, sans modification.
# h_test = h_original
```

Le Dropout est une technique incroyablement simple mais remarquablement efficace. Il améliore presque toujours la performance des réseaux de neurones, sauf dans les rares cas où le modèle est déjà en situation de sous-apprentissage sévère. Une anecdote célèbre illustre son impact : lors d'une conférence de Geoff Hinton en 2012, un étudiant a implémenté le Dropout en direct pendant la présentation et a obtenu des résultats significativement meilleurs, atteignant des performances de pointe sur son propre jeu de données avant même la fin de la conférence. Cette histoire souligne la puissance des idées simples et efficaces qui ont un impact disproportionné sur la performance pratique, et le Dropout est l'une de ces rares avancées qui "fonctionne toujours mieux".

## 4. Introduction aux Réseaux de Neurones Convolutifs (CNNs)

Après avoir exploré les techniques d'optimisation et de régularisation, il est temps de se tourner vers une architecture de réseau de neurones particulièrement puissante pour le traitement des données visuelles : les Réseaux de Neurones Convolutifs (CNNs). Leur conception est profondément enracinée dans la compréhension de la vision biologique.

### Contexte Historique et Inspirations Biologiques

L'histoire des CNNs remonte aux travaux pionniers en neurosciences. Dans les années 1960, les neurophysiologistes David Hubel et Torsten Wiesel ont mené des expériences révolutionnaires sur le cortex visuel du chat (spécifiquement l'aire V1). Ils ont découvert que certaines cellules nerveuses dans cette région répondaient sélectivement à des stimuli visuels très spécifiques, comme des "bords dans une orientation particulière". Une cellule pouvait s'activer fortement pour un bord horizontal, mais rester silencieuse pour un bord vertical. Leurs travaux, qui leur ont valu un prix Nobel, ont également révélé que le cortex visuel est organisé de manière rétinotopique (les cellules corticales proches traitent des zones visuelles proches dans le champ de vision) et qu'il existe une hiérarchie de traitement.

Ils ont identifié des "cellules simples" qui répondaient à des orientations d'arêtes spécifiques et des "cellules complexes" qui montraient une certaine invariance translationnelle, c'est-à-dire qu'elles répondaient à une arête quelle que soit sa position exacte dans leur champ réceptif. Cette organisation hiérarchique, où des cellules simples alimentent des cellules complexes, et où les champs réceptifs deviennent de plus en plus larges et abstraits à mesure que l'on monte dans la hiérarchie, a fourni une inspiration fondamentale pour la conception des architectures de réseaux de neurones artificiels.

Inspiré par ces découvertes biologiques, Kunihiko Fukushima a développé le **Neocognitron** dans les années 1980. Cette architecture était composée de couches successives avec des "champs réceptifs locaux", où les neurones d'une couche ne recevaient des entrées que d'une petite région de la couche précédente. Il alternait des couches de "cellules simples" (similaires aux détecteurs de caractéristiques) et de "cellules complexes" (similaires aux couches de pooling qui introduisent l'invariance). Bien que le Neocognitron utilisait une procédure d'apprentissage non supervisée à l'époque (la rétropropagation n'étant pas encore courante), il a posé les bases architecturales des CNNs modernes.

Dans les années 1990, Yann LeCun a repris et amélioré l'architecture de Fukushima, mais il a surtout innové en entraînant ces réseaux avec l'algorithme de rétropropagation. Son réseau, le **LeNet-5**, a été un succès pratique notable, utilisé par exemple pour la reconnaissance de chiffres manuscrits dans les services postaux. Cette évolution des CNNs est un témoignage de la convergence entre l'inspiration biologique, la modélisation théorique et les avancées computationnelles, montrant comment les observations de systèmes naturels peuvent guider la conception d'algorithmes artificiels.

### L'Ère Moderne : AlexNet et l'Explosion des CNNs

Le véritable tournant pour les CNNs et l'apprentissage profond en général est survenu en 2012 avec l'introduction d'**AlexNet**. Alex Krizhevsky, Ilya Sutskever et Geoff Hinton ont entraîné ce réseau sur le vaste ensemble de données ImageNet, qui contient des millions d'images réparties en milliers de classes. AlexNet a surpassé de loin tous les autres algorithmes de vision par ordinateur de l'époque, marquant le début de l'ère moderne de l'apprentissage profond.

Ce qui est particulièrement frappant, c'est que les différences architecturales fondamentales entre AlexNet et le LeNet-5 des années 1990 étaient en réalité très minimes. Le succès d'AlexNet peut être attribué à plusieurs facteurs clés, principalement liés à la mise à l'échelle (scaling) et à des améliorations techniques :

- **Taille et Profondeur du Modèle** : AlexNet était un réseau beaucoup plus grand et plus profond que ses prédécesseurs, avec environ 60 millions de paramètres. La capacité accrue du modèle lui a permis d'apprendre des représentations plus complexes.

- **Fonctions d'Activation ReLU** : AlexNet a utilisé les fonctions d'activation ReLU (Rectified Linear Unit) au lieu des sigmoïdes ou tanh, qui étaient courantes auparavant. Les ReLU aident à atténuer les problèmes de vanishing gradients et accélèrent l'entraînement.

- **Entraînement sur GPU** : L'entraînement d'un modèle de cette taille a été rendu possible par l'utilisation de processeurs graphiques (GPUs), qui sont beaucoup plus efficaces pour les calculs matriciels massifs requis par les réseaux de neurones.

- **Disponibilité de Données** : L'accès à ImageNet, un ensemble de données massif et diversifié, a été crucial pour entraîner un modèle aussi grand sans surapprentissage excessif.

Le succès d'AlexNet a démontré que la "mise à l'échelle" (plus de données, plus de puissance de calcul, plus de paramètres) était le facteur de percée crucial en apprentissage profond. Cela a montré que les concepts fondamentaux des CNNs étaient déjà là depuis des décennies, mais que leur plein potentiel n'a été débloqué qu'avec les avancées matérielles et la disponibilité de vastes jeux de données. Cela a initié une ère d'exploration de modèles de plus en plus grands et profonds, avec des réseaux atteignant aujourd'hui 150 couches ou plus.

### Applications Modernes des CNNs

Aujourd'hui, les réseaux de neurones convolutifs sont omniprésents et ont révolutionné de nombreux domaines, bien au-delà de la simple classification d'images. Leur capacité à apprendre des caractéristiques hiérarchiques et pertinentes à partir de données brutes les rend incroyablement polyvalents.

Les applications visuelles sont les plus évidentes :

- **Classification et Récupération d'Images** : Les CNNs sont excellents pour classer des images en catégories spécifiques et pour retrouver des images similaires dans de grandes bases de données.

- **Détection d'Objets** : Ils peuvent localiser et identifier plusieurs objets dans une image, une capacité essentielle pour les voitures autonomes (qui détectent les chiens, les chevaux, les personnes, etc.).

- **Segmentation Sémantique** : Les CNNs peuvent étiqueter chaque pixel d'une image avec une catégorie (par exemple, personne, route, arbre, ciel, bâtiment), fournissant une compréhension détaillée de la scène.

- **Reconnaissance Faciale et Vidéo** : Ils sont utilisés pour identifier des visages sur Facebook et pour classer le contenu des vidéos sur YouTube.

- **Lecture Automatisée** : Des projets comme celui de Google pour lire automatiquement les numéros de maison à partir d'images Street View ont démontré l'efficacité des CNNs pour des tâches d'extraction d'informations précises.

- **Domaine Médical** : Les CNNs sont utilisés pour la détection de cancers à partir d'images médicales et pour la segmentation de tissus neuronaux.

- **Applications Spécifiques** : Reconnaissance de caractères chinois, reconnaissance de panneaux de signalisation, classification de galaxies, et même l'identification individuelle de baleines par leurs motifs uniques.

- **Imagerie Satellite** : Les CNNs sont largement employés pour analyser les vastes quantités de données satellites, par exemple pour la cartographie routière ou les applications agricoles.

Au-delà du domaine visuel, la polyvalence des CNNs s'étend à d'autres types de données structurées :

- **Traitement du Langage et de la Parole** : Les CNNs ont été adaptés pour la reconnaissance vocale et le traitement de documents textuels.

Les CNNs sont également à l'origine d'applications plus créatives et artistiques, comme **Deep Dream**. Cette technique permet de générer des "hallucinations" visuelles en amplifiant les motifs que le réseau a appris à reconnaître. Les images générées par Deep Dream présentent souvent des formes de chiens, un phénomène qui s'explique par le fait que les réseaux sont souvent entraînés sur ImageNet, un ensemble de données contenant une grande proportion d'images de chiens.

Enfin, un domaine de recherche fascinant concerne les connexions entre les CNNs et le cerveau biologique. Des études ont comparé les représentations internes des CNNs à l'activité neuronale enregistrée dans le cortex inférotemporal (IT) des primates lorsqu'ils regardent des images. Les résultats montrent une similarité remarquable entre la manière dont les images sont représentées dans le cerveau et dans les CNNs. Cela suggère que les CNNs ne sont pas de simples outils d'ingénierie, mais qu'ils pourraient avoir découvert des principes de calcul qui sont également fondamentaux pour l'intelligence biologique, ouvrant des voies de recherche passionnantes sur la nature de l'intelligence et de l'apprentissage.

## Conclusion

Cette conférence a couvert des aspects essentiels de l'entraînement des réseaux de neurones, depuis les fondements des fonctions d'activation et de l'initialisation des poids, jusqu'aux stratégies d'optimisation avancées et aux techniques de régularisation. Nous avons vu comment des problèmes fondamentaux de convergence du SGD ont conduit au développement d'optimiseurs sophistiqués comme Momentum, Nesterov, Adagrad, RMSprop et Adam, chacun apportant des solutions uniques pour naviguer dans les paysages de perte complexes des réseaux profonds. L'importance de la décroissance du taux d'apprentissage et les limites pratiques des méthodes de second ordre ont également été soulignées.

Parallèlement, nous avons exploré des techniques cruciales pour améliorer la généralisation des modèles, telles que les ensembles de modèles et, en particulier, le Dropout, une méthode simple mais d'une efficacité redoutable pour prévenir le surapprentissage et forcer le réseau à apprendre des représentations plus robustes.

Enfin, nous avons introduit les Réseaux de Neurones Convolutifs, en retraçant leur histoire depuis les découvertes neuroscientifiques inspirantes de Hubel et Wiesel, à travers le Neocognitron de Fukushima et le LeNet de LeCun, jusqu'à l'ère moderne inaugurée par AlexNet. L'explosion des applications des CNNs dans des domaines variés, ainsi que les parallèles fascinants avec le cortex visuel biologique, témoignent de leur puissance et de leur pertinence continue dans le domaine de la vision par ordinateur et au-delà.

La prochaine session approfondira les détails architecturaux des Réseaux de Neurones Convolutifs, en explorant les blocs de construction qui leur permettent de réaliser ces prouesses.


# **Conférence : Fonctions de Perte, Régularisation et Optimisation en Apprentissage Automatique**

## **1\. Introduction et Rappel des Fondamentaux**

### **1.1. Contexte : La Reconnaissance Visuelle et la Classification d'Images**

La reconnaissance visuelle, et plus spécifiquement la classification d'images, représente un défi majeur en apprentissage automatique. Des tâches en apparence simples, comme la reconnaissance d'un chat dans une image, sont en réalité d'une complexité intrinsèque considérable. Cette difficulté découle de la multitude de variations possibles auxquelles un système doit être robuste : changements d'éclairage, différentes poses de l'objet, occlusions partielles, variations d'échelle, et bien d'autres facteurs qui modifient l'apparence visuelle d'une même catégorie.1

Pourtant, malgré cette complexité, les avancées récentes dans le domaine ont été spectaculaires. Les méthodes de pointe actuelles sont capables de résoudre le problème de classification pour des milliers de catégories avec une précision qui approche, voire dépasse légèrement, celle de l'être humain dans certaines classes.1 De plus, ces systèmes fonctionnent presque en temps réel sur des appareils mobiles.1 Il est remarquable de constater que cette révolution s'est produite au cours des trois dernières années, transformant radicalement le paysage de la vision par ordinateur.1 La rapidité de cette évolution souligne l'importance pour les étudiants de maîtriser les principes fondamentaux sous-jacents, tels que les fonctions de perte, la régularisation et l'optimisation. Ces concepts sont en effet transférables et durables, permettant de s'adapter et d'innover face à l'émergence constante de nouvelles architectures et de nouveaux problèmes.

### **1.2. Récapitulatif de l'Approche Basée sur les Données et les Classificateurs Linéaires**

Face à la complexité de la classification d'images, il est impraticable de coder explicitement des règles pour chaque catégorie. L'approche dominante est donc l'apprentissage à partir des données, connue sous le nom d'approche *data-driven*.1 Cette méthode implique de diviser les données disponibles en plusieurs ensembles : un ensemble d'entraînement pour apprendre le modèle, un ensemble de validation pour ajuster les hyperparamètres du modèle, et un ensemble de test qui doit rester intouché pour évaluer la performance finale et non biaisée du modèle.1

Les discussions précédentes ont introduit le classificateur des K-plus proches voisins et le jeu de données CIFAR-10, un ensemble de données de référence souvent utilisé pour expérimenter.1 Le concept d'approche paramétrique a ensuite été introduit, où une fonction

f est définie pour mapper directement une image à un ensemble de scores, un score pour chacune des classes possibles (par exemple, 10 scores pour les 10 classes de CIFAR-10).1 Pour simplifier, cette fonction

f a été initialement supposée linéaire, prenant la forme f(x) \= Wx, où W est une matrice de poids.1

Cette interprétation du classificateur linéaire peut être vue de deux manières principales : soit comme un système de correspondance de gabarits (templates), où les lignes de la matrice W agissent comme des gabarits pour chaque classe, soit comme un mécanisme qui "colore" l'espace de haute dimension des images en fonction des scores de classe.1 Pour illustrer, un exemple d'entraînement avec trois images de CIFAR-10 a été présenté, montrant comment des poids

W choisis aléatoirement produisent des scores. L'examen de ces scores révèle que certains résultats sont "bons" (par exemple, une voiture bien classée avec un score élevé pour sa catégorie) et d'autres "mauvais" (par exemple, une grenouille mal classée avec un score très bas pour sa catégorie correcte).1 L'objectif fondamental est de trouver des poids

W qui génèrent des scores cohérents avec les étiquettes de vérité terrain pour l'ensemble des données.1

L'évaluation visuelle de la qualité des scores, bien qu'intuitive, est intrinsèquement subjective et ne permet pas une optimisation systématique du modèle. Pour qu'un algorithme puisse apprendre et s'améliorer de manière autonome, il est impératif de disposer d'une mesure numérique objective de son "insatisfaction" ou de son erreur. C'est précisément cette nécessité qui motive l'introduction des fonctions de perte, offrant une quantification rigoureuse de la performance du modèle.

## **2\. Les Fonctions de Perte : Mesurer l'Erreur du Modèle**

### **2.1. Définition et Rôle d'une Fonction de Perte**

Une fonction de perte, également appelée fonction de coût, est un outil mathématique essentiel en apprentissage automatique. Son rôle est de quantifier le "degré d'insatisfaction" ou l'erreur d'un modèle pour un ensemble de poids W donné, en comparant ses prédictions aux vérités terrain.1 L'objectif ultime de l'apprentissage est de trouver l'ensemble de poids

W qui minimise cette fonction de perte, rendant ainsi le modèle aussi "satisfait" que possible avec ses prédictions.1 Cette section explorera deux fonctions de perte courantes et largement utilisées : la perte SVM (Support Vector Machine) et la perte Softmax, souvent appelée perte d'entropie croisée.1

### **2.2. La Fonction de Perte SVM (Support Vector Machine) Multiclasse**

#### **2.2.1. Formulation Mathématique et Interprétation Intuitive**

La perte SVM multiclasse est une extension de la machine à vecteurs de support binaire, conçue pour gérer des problèmes de classification avec plus de deux classes.1 Pour un exemple d'entraînement

(x\_i, y\_i), où x\_i est l'image et y\_i est son étiquette de classe correcte, la perte L\_i est définie par la formule suivante :

L\_i \= Σ\_{j≠y\_i} max(0, s\_j \- s\_{y\_i} \+ 1\) 1

Dans cette formule, s\_j représente le score prédit par le modèle pour une classe incorrecte j, et s\_{y\_i} est le score pour la classe correcte y\_i.1 L'interprétation intuitive de cette perte est que le modèle est pénalisé si le score de la classe correcte n'est pas seulement supérieur à celui de toute classe incorrecte, mais qu'il le dépasse par une marge de sécurité prédéfinie.1 La fonction

max(0,...) est cruciale : elle garantit que si la marge est déjà satisfaite (c'est-à-dire si s\_j \- s\_{y\_i} \+ 1 est négatif ou nul), la contribution de cette classe incorrecte à la perte totale de l'exemple est nulle.1 En d'autres termes, une fois que le score correct est suffisamment plus élevé qu'un score incorrect (par au moins la marge), il n'y a pas de pénalité supplémentaire pour augmenter encore cette différence.1

#### **2.2.2. Le Concept de Marge de Sécurité (Δ=1)**

La marge de sécurité dans la formule de la perte SVM est généralement fixée à 1\.1 Ce choix, bien qu'arbitraire en apparence, est en réalité très pratique et ne compromet pas l'optimisation du modèle. Les scores produits par le classificateur linéaire (

Wx) sont intrinsèquement "sans échelle" (scale-free) ; leur magnitude absolue dépend directement de la magnitude des poids W. Si les poids W sont multipliés par une constante alpha, les scores seront également mis à l'échelle proportionnellement.1

Dans le contexte de l'optimisation, la minimisation d'une fonction de perte L(W) est mathématiquement équivalente à la minimisation de c \* L(W) pour toute constante positive c, ou à la minimisation de L(W) \+ k pour toute constante k. Puisque la marge de 1 peut être absorbée par une mise à l'échelle des poids W sans modifier la solution optimale, elle n'a pas besoin d'être traitée comme un hyperparamètre à régler. Fixer la marge à 1 simplifie ainsi le processus d'ajustement du modèle.

#### **2.2.3. Exemples Détaillés de Calcul de Perte**

Pour illustrer le calcul de la perte SVM, considérons un exemple avec trois classes (chat, voiture, grenouille) et leurs scores. Supposons que la classe correcte pour la première image est "chat" avec un score s\_chat \= 3.2.

* Pour la classe incorrecte "voiture" (score s\_voiture \= 5.1) : max(0, 5.1 \- 3.2 \+ 1\) \= max(0, 1.9 \+ 1\) \= max(0, 2.9) \= 2.9.1  
* Pour la classe incorrecte "grenouille" (score s\_grenouille \= 1.7) : max(0, 1.7 \- 3.2 \+ 1\) \= max(0, \-1.5 \+ 1\) \= max(0, \-0.5) \= 0\.1

  La perte totale pour cette première image est L\_1 \= 2.9 \+ 0 \= 2.9.1

Pour une deuxième image, si la classe correcte est "voiture" avec un score de 4.9, et les autres scores sont suffisamment bas pour satisfaire la marge, la perte pour cette image pourrait être de 0\.1 En revanche, pour une troisième image où la classe correcte est "grenouille" avec un score de \-3.1, et les autres scores sont élevés, la perte peut être très élevée, par exemple 10.9, indiquant une très mauvaise classification.1

La perte totale pour l'ensemble d'entraînement est ensuite calculée comme la moyenne des pertes individuelles pour chaque exemple. Dans cet exemple, si les pertes sont 2.9, 0 et 10.9, la perte moyenne serait (2.9 \+ 0 \+ 10.9) / 3 \= 4.6.1

#### **2.2.4. Plage de Valeurs Possibles (Minimum et Maximum)**

La perte SVM multiclasse possède une plage de valeurs bien définie. La perte minimale possible est **0**.1 Cela se produit lorsque le score de la classe correcte est supérieur à celui de toutes les classes incorrectes d'au moins la marge de sécurité de 1\.1 À l'inverse, la perte maximale possible est

**infinie**.1 Une telle situation peut survenir si le score attribué à l'exemple correct est très faible (par exemple, un grand nombre négatif), ce qui entraîne une très grande différence positive

s\_j \- s\_{y\_i} \+ 1 pour les classes incorrectes, contribuant ainsi de manière significative à la perte.1

#### **2.2.5. Vérification de Cohérence avec des Poids Initiaux Nuls**

Une vérification de cohérence (sanity check) importante au début du processus d'optimisation consiste à évaluer la perte lorsque les poids W sont initialisés avec de très petites valeurs, proches de zéro. Dans ce scénario, tous les scores s seront également approximativement nuls.1 Pour chaque classe incorrecte

j, le terme s\_j \- s\_{y\_i} \+ 1 sera alors approximativement 0 \- 0 \+ 1 \= 1\.1 Par conséquent, la contribution à la perte pour chaque classe incorrecte sera

max(0, 1\) \= 1\.1 Puisque la somme est effectuée sur toutes les classes incorrectes, et qu'il y a

(nombre de classes \- 1\) classes incorrectes, la perte pour un seul exemple sera (nombre de classes \- 1).1 Par exemple, pour un problème à 3 classes, la perte initiale attendue serait

(3 \- 1\) \= 2\.1 Observer cette valeur au début de l'entraînement permet de s'assurer que la fonction de perte est correctement implémentée.1

#### **2.2.6. Variations de la Perte SVM**

La formulation de la perte SVM peut être légèrement modifiée. Si la somme était effectuée sur toutes les classes (y compris la classe correcte y\_i), cela ajouterait une constante de 1 à la perte pour chaque exemple, puisque s\_{y\_i} \- s\_{y\_i} \+ 1 \= 1\. Cependant, l'ajout d'une constante à une fonction de perte ne modifie pas l'ensemble des poids W optimaux qui la minimisent.1 De même, l'utilisation d'une moyenne au lieu d'une somme sur les contraintes de score reviendrait à multiplier la perte par un facteur constant (

1/nombre\_de\_classes). Une telle mise à l'échelle constante n'altère pas non plus la solution optimale de W.1

Cependant, l'utilisation d'une perte à charnière carrée (*Squared Hinge Loss*), définie comme (max(0, s\_j \- s\_{y\_i} \+ 1))^2, introduit une transformation non-linéaire. Contrairement aux transformations linéaires (ajout d'une constante, mise à l'échelle), cette modification peut altérer le paysage de la fonction de perte et, par conséquent, les solutions optimales. Une petite erreur est pénalisée de manière quadratique, tandis qu'une grande erreur entraîne une pénalité beaucoup plus importante. Cela change la "forme" du problème d'optimisation et la manière dont le modèle est pénalisé, ce qui peut influencer la convergence et la solution finale. Bien que la perte à charnière simple soit la plus courante, la perte à charnière carrée est un hyperparamètre que l'on peut ajuster pour des performances potentiellement meilleures sur certains jeux de données.1

### **2.3. La Fonction de Perte Softmax (Régression Logistique Multinominale / Entropie Croisée)**

#### **2.3.1. Interprétation des Scores comme Probabilités Logarithmiques Non Normalisées**

Contrairement à la perte SVM, la fonction de perte Softmax, également connue sous le nom de régression logistique multinominale ou coût d'entropie croisée, attribue une interprétation probabiliste aux scores bruts produits par le classificateur.1 Ces scores ne sont pas considérés comme de simples valeurs arbitraires, mais comme des "probabilités logarithmiques non normalisées" associées à chaque classe.1

#### **2.3.2. La Fonction Softmax et la Normalisation des Probabilités**

Pour transformer ces probabilités logarithmiques non normalisées en probabilités réelles qui peuvent être interprétées de manière significative (c'est-à-dire qui sont positives et somment à 1), la fonction Softmax est appliquée.1 La formule de la probabilité d'une classe

k étant la classe correcte, étant donné l'image x, est la suivante :

P(y=k|x) \= e^(s\_k) / Σ\_j e^(s\_j) 1

Cette fonction exponentie d'abord chaque score s\_k pour garantir que toutes les valeurs sont positives, puis les normalise en les divisant par la somme de tous les scores exponentiés. Le résultat est une distribution de probabilité sur l'ensemble des classes.1

#### **2.3.3. Minimisation de la Log-Vraisemblance Négative**

Dans ce cadre probabiliste, l'objectif principal est de maximiser la log-vraisemblance de la classe correcte.1 Étant donné que les fonctions de perte sont conçues pour être minimisées, la perte Softmax est définie comme la log-vraisemblance négative de la classe correcte 1 :

L\_i \= \-log(P(y=y\_i|x)) 1

Il est important de noter que maximiser la probabilité d'une classe et maximiser sa log-probabilité sont des objectifs équivalents, car la fonction logarithme est une fonction monotone croissante. L'utilisation du logarithme est principalement motivée par des considérations de commodité mathématique, car elle simplifie les dérivées lors du calcul des gradients.1

#### **2.3.4. Exemples Détaillés de Calcul de Perte**

Prenons un exemple pour clarifier le calcul de la perte Softmax. Supposons que les scores bruts pour une image soient \[10, 5.1, \-1.7\] pour les classes "chat", "voiture" et "grenouille" respectivement, et que la classe correcte est "chat".

1. **Exponentiation des scores :** e^10, e^5.1, e^-1.7.1  
2. **Somme des scores exponentiés :** e^10 \+ e^5.1 \+ e^-1.7.  
3. **Calcul de la probabilité pour la classe correcte ("chat") :** P(chat|x) \= e^10 / (e^10 \+ e^5.1 \+ e^-1.7). Ce calcul donne une probabilité de 0.13 (13%).1  
4. **Calcul de la perte :** La perte pour cette image est \-log(0.13) ≈ 0.89.1

#### **2.3.5. Plage de Valeurs Possibles (Minimum et Maximum)**

Similaire à la perte SVM, la perte Softmax a également une plage de valeurs définie. La perte minimale possible est **0**.1 Cela se produit lorsque la classe correcte reçoit une probabilité de

1, car \-log(1) \= 0\.1 Inversement, la perte maximale possible est

**infinie**.1 Cette situation survient si le classificateur Softmax attribue une probabilité très faible (proche de zéro) à la classe correcte, car le logarithme d'un nombre proche de zéro tend vers l'infini négatif, rendant la log-vraisemblance négative infinie.1

#### **2.3.6. Vérification de Cohérence avec des Poids Initiaux Nuls**

Lors de l'initialisation des poids W avec de très petites valeurs (proches de zéro), tous les scores s seront également proches de zéro.1 Dans ce cas, les scores exponentiés (

e^s) seront tous proches de e^0 \= 1\.1 Après la normalisation par la fonction Softmax, chaque classe recevra une probabilité de

1 / (nombre de classes).1 Par conséquent, la perte initiale attendue pour le classificateur Softmax est

\-log(1 / nombre de classes).1 Pour un problème à 3 classes, cette valeur serait

\-log(1/3) ≈ 1.098. Cette valeur sert de "sanity check" important au démarrage de l'optimisation, permettant de vérifier que l'implémentation est correcte.1

### **2.4. Comparaison et Contraste : SVM vs. Softmax**

Les fonctions de perte SVM et Softmax, bien que toutes deux utilisées pour la classification, diffèrent fondamentalement dans leur interprétation des scores et leurs préférences en matière de comportement du modèle.

#### **2.4.1. Différences Fondamentales dans l'Interprétation des Scores**

La distinction majeure réside dans la manière dont elles traitent les scores bruts produits par le classificateur linéaire. La **perte SVM** considère ces scores comme des valeurs arbitraires. Son objectif principal est de s'assurer que le score de la classe correcte dépasse celui des classes incorrectes par une marge fixe (généralement 1).1 En revanche, la

**perte Softmax** interprète ces scores comme des probabilités logarithmiques non normalisées. Son but est de maximiser la probabilité (ou la log-probabilité) de la classe correcte, en transformant d'abord ces scores en une distribution de probabilité valide.1

#### **2.4.2. Sensibilité aux Marges et Préférences du Modèle**

La différence d'interprétation conduit à des comportements distincts en termes de "préférence" du modèle. La **SVM** manifeste une préférence "locale". Une fois que la marge de sécurité est satisfaite pour une classe incorrecte (c'est-à-dire que sa contribution à la perte est nulle grâce à la fonction max(0,...)), la SVM devient indifférente à la valeur exacte du score de cette classe. Par exemple, si le score d'une classe incorrecte est \-100 et que la marge est respectée, la perte pour cette classe est nulle ; elle resterait nulle même si le score diminuait à \-1000.1 Cette caractéristique confère à la SVM une certaine "robustesse" : elle se concentre uniquement sur le respect de la marge et ne cherche pas à micro-gérer les scores au-delà de ce seuil.1

En contraste, la **Softmax** exprime une préférence "globale". Elle cherche continuellement à augmenter la probabilité de la classe correcte et à diminuer celles des classes incorrectes, même si la classification est déjà très bonne. Elle ne cesse jamais de "pousser" les scores dans la bonne direction. Si les scores sont \[10, \-100, \-100\] pour les classes correcte et incorrectes, la Softmax aura une perte non nulle et continuera de chercher à améliorer la probabilité de la classe correcte, par exemple en rendant les scores incorrects encore plus négatifs.1 Cette différence de comportement peut influencer la manière dont le modèle apprend et généralise, en particulier pour les exemples déjà bien classés. La SVM est plus "paresseuse" une fois la marge atteinte, tandis que la Softmax est toujours "active", encourageant le modèle à devenir plus confiant dans ses prédictions, ce qui peut la rendre plus sensible aux valeurs aberrantes ou aux exemples difficiles.

#### **2.4.3. Résultat Pratique**

Malgré ces différences conceptuelles et comportementales, il est souvent observé qu'en pratique, les classificateurs basés sur la perte SVM et ceux basés sur la perte Softmax produisent des résultats presque identiques.1 Le choix entre les deux est fréquemment considéré comme un hyperparamètre à ajuster lors de l'entraînement du modèle.1

| Caractéristique | Perte SVM Multiclasse | Perte Softmax (Entropie Croisée) |
| :---- | :---- | :---- |
| **Interprétation des scores** | Arbitraires ; objectif de marge | Probabilités logarithmiques non normalisées |
| **Formule clé** | max(0, s\_j \- s\_{y\_i} \+ 1\) | \-log(e^(s\_{y\_i}) / Σ\_j e^(s\_j)) |
| **Préférence du modèle** | Locale ; indifférente une fois la marge atteinte | Globale ; cherche toujours à maximiser la probabilité de la classe correcte |
| **Plage de perte (Min)** | 0 | 0 |
| **Plage de perte (Max)** | Infini | Infini |
| **Perte initiale (scores \~0)** | Nombre de classes \- 1 | \-log(1 / Nombre de classes) |

Table 1 : Comparaison des Fonctions de Perte SVM et Softmax

## **3\. La Régularisation : Améliorer la Généralisation et Éviter le Surapprentissage**

### **3.1. Le Problème de la Non-Unicité des Poids Optimaux**

Une fois qu'un ensemble de poids W a été trouvé pour minimiser la fonction de perte, un problème peut survenir : il n'est pas rare qu'il existe un sous-espace entier de poids W qui produisent la même perte minimale, y compris une perte nulle sur l'ensemble d'entraînement.1 Par exemple, si un ensemble de poids

W conduit à une perte nulle avec la fonction SVM, alors la multiplication de W par une constante alpha supérieure à 1 (alpha \* W) entraînera également une perte nulle. En effet, cela ne ferait qu'augmenter les différences de scores, les rendant encore plus négatives à l'intérieur de la fonction max(0,...), qui les ramènerait toujours à zéro.1

Cette non-unicité des poids optimaux est une propriété indésirable. Elle signifie que plusieurs modèles peuvent être "parfaitement" performants sur les données d'entraînement, mais pourraient se comporter différemment sur de nouvelles données non vues. L'existence de multiples W qui donnent une perte nulle sur les données d'entraînement est un indicateur qu'il existe un risque de surapprentissage (overfitting). Le modèle pourrait être trop "spécifique" aux données d'entraînement, ayant potentiellement mémorisé le bruit ou des particularités anecdotiques de cet ensemble, plutôt que d'avoir appris les motifs sous-jacents généralisables. Cela conduit inévitablement à une performance sous-optimale sur les données de test.

### **3.2. Objectif de la Régularisation : Introduire une Préférence pour des Poids "Souhaitables"**

Pour remédier au problème de la non-unicité et améliorer la capacité de généralisation du modèle, le concept de régularisation est introduit. La régularisation consiste à ajouter un terme supplémentaire à la fonction de perte totale, qui devient alors : L\_totale \= L\_données \+ lambda \* R(W).1 Dans cette formule,

R(W) est une fonction de régularisation qui évalue la "qualité" ou la "niceness" des poids W.1

L'objectif de cette approche est double : non seulement le modèle doit s'adapter aux données d'entraînement (minimiser L\_données), mais il doit aussi s'assurer que les poids W possèdent des propriétés souhaitables (minimiser R(W)).1 La régularisation agit ainsi comme un compromis entre la perte d'entraînement (l'ajustement aux données observées) et la perte de généralisation (la performance sur des données non vues).1 Les deux termes de la fonction de perte "se battent" l'un l'autre : le terme de données pousse le modèle à bien classer les exemples d'entraînement, tandis que le terme de régularisation pousse les poids

W à adopter une certaine forme ou structure.1 Il est paradoxal mais crucial de comprendre que l'ajout de régularisation peut parfois

*empirer* l'erreur sur l'ensemble d'entraînement, mais il conduit fréquemment à une *meilleure* performance sur l'ensemble de test.1

### **3.3. Régularisation L2 (Décroissance de Poids / *Weight Decay*)**

La forme la plus courante de régularisation est la régularisation L2, également connue sous le nom de "décroissance de poids" (*weight decay*).1 Elle consiste à ajouter la somme des carrés de tous les éléments de la matrice de poids

W à la fonction de perte : R(W) \= Σ\_k Σ\_l W\_kl^2.1 Cette forme de régularisation "préfère" les poids

W qui sont petits et proches de zéro.1 Bien sûr, les poids ne peuvent pas être tous nuls, car cela empêcherait toute classification ; c'est là que réside le "combat" entre le terme de données et le terme de régularisation.1

#### **3.3.1. Intuition : Encourager des Poids Petits et Diffus**

Pour illustrer l'intérêt des poids petits et diffus, considérons un exemple simple de classification en quatre dimensions avec un vecteur d'entrée x composé uniquement de 1\.1 Supposons deux ensembles de poids candidats :

W1 \= et W2 \= \[0.25, 0.25, 0.25, 0.25\].1 Pour un classificateur linéaire, le produit scalaire

W.x (qui donne le score) est identique pour les deux (1\*1 \+ 0\*1 \+ 0\*1 \+ 0\*1 \= 1 pour W1, et 0.25\*1 \+ 0.25\*1 \+ 0.25\*1 \+ 0.25\*1 \= 1 pour W2). Cela signifie que leur effet sur la perte de données est le même.1

Cependant, la régularisation L2 favoriserait strictement W2. La somme des carrés pour W1 est 1^2 \+ 0^2 \+ 0^2 \+ 0^2 \= 1, tandis que pour W2, elle est 0.25^2 \+ 0.25^2 \+ 0.25^2 \+ 0.25^2 \= 4 \* 0.0625 \= 0.25.1 L'intuition derrière cette préférence est que la régularisation L2 encourage les poids à être "diffusés" autant que possible, c'est-à-dire à prendre en compte toutes les caractéristiques d'entrée ou tous les pixels.1 Dans l'exemple,

W1 ignore complètement les entrées 2, 3 et 4, tandis que W2 utilise toutes les entrées de manière équilibrée.1 Cette diffusion des poids est généralement bénéfique, car elle permet d'accumuler davantage de preuves pour prendre une décision, rendant le modèle plus robuste et conduisant souvent à une meilleure performance sur l'ensemble de test.1

#### **3.3.2. Brève Mention de la Régularisation L1**

Bien que la régularisation L2 soit la plus courante, d'autres formes existent, comme la régularisation L1, qui ajoute la somme des valeurs absolues des poids (Σ |W\_kl|) à la perte.1 La régularisation L1 possède des propriétés distinctes, notamment en favorisant la "parcimonie" (sparsity). Cela signifie qu'elle tend à pousser certains poids à être exactement zéro, ce qui peut avoir pour effet de sélectionner automatiquement les caractéristiques les plus importantes et d'ignorer les moins pertinentes.1 Le choix entre L1 et L2 dépend ainsi des propriétés souhaitées pour le modèle : L2 favorise la diffusion et la robustesse, tandis que L1 favorise la sélection de caractéristiques et des modèles potentiellement plus interprétables.

## **4\. L'Optimisation : Trouver les Poids Optimaux**

### **4.1. Le Concept de "Paysage de Perte" (Analogie Visuelle)**

Le processus d'optimisation en apprentissage automatique peut être visualisé comme la navigation dans un "paysage de perte".1 Ce paysage existe dans l'espace des poids

W, qui est souvent de très haute dimension (par exemple, 30 000 dimensions pour un classificateur linéaire simple sur CIFAR-10).1 La hauteur de ce paysage représente la valeur de la fonction de perte pour un ensemble de poids donné.1 L'objectif est de trouver le "fond de la vallée", c'est-à-dire le point dans cet espace où la perte est minimale.1 Dans cette analogie, l'optimiseur est comme une personne les yeux bandés, munie d'un altimètre (pour connaître la perte actuelle), qui tente de descendre la montagne pour atteindre le point le plus bas.1

### **4.2. Recherche Aléatoire : Une Stratégie Naïve**

Une stratégie d'optimisation très simple, mais inefficace, est la recherche aléatoire.1 Cette méthode consiste à échantillonner aléatoirement différents ensembles de poids

W, à calculer la perte pour chacun d'eux, et à conserver l'ensemble de poids qui a donné la perte la plus faible.1 Bien qu'intuitivement facile à comprendre, cette approche est extrêmement lente et peu performante. Par exemple, sur le jeu de données CIFAR-10, une recherche aléatoire effectuée un millier de fois n'a permis d'atteindre qu'environ 15,5% de précision, ce qui est à peine mieux que la performance aléatoire (10% pour 10 classes) et très loin de l'état de l'art (95%).1 La faible performance de la recherche aléatoire dans des espaces de haute dimension s'explique par la "malédiction de la dimensionnalité". Le nombre de combinaisons possibles pour des millions de paramètres est astronomique, rendant la probabilité de trouver un bon

W par pur hasard infime. Cela met en évidence la nécessité de méthodes d'optimisation plus intelligentes qui exploitent des informations sur la structure du paysage de perte.

### **4.3. Les Gradients : La Boussole du Paysage de Perte**

Pour naviguer efficacement dans le paysage de perte, une approche plus sophistiquée que la recherche aléatoire est nécessaire : l'utilisation des gradients.

#### **4.3.1. Définition et Utilité**

Un gradient est un vecteur de dérivées partielles qui indique la direction de la plus forte augmentation de la fonction de perte dans l'espace des poids.1 Pour minimiser la perte, il est donc nécessaire de se déplacer dans la direction opposée au gradient, c'est-à-dire dans la direction de la plus forte diminution de la perte.1 Le gradient agit comme une boussole, indiquant la voie la plus directe vers le "fond de la vallée".

#### **4.3.2. Gradient Numérique : Approximation et Limites**

Le gradient numérique est une méthode pour approximer le gradient en évaluant la fonction de perte à des points légèrement perturbés.1 Pour chaque dimension

k du vecteur de poids W, le gradient numérique est calculé en prenant un petit pas h dans cette dimension, en évaluant la perte au nouveau point f(W \+ h), en soustrayant la perte au point d'origine f(W), et en divisant par h : (f(W \+ h) \- f(W)) / h.1 Ce processus est répété pour chaque dimension de

W.1 Bien que cette méthode soit facile à écrire et à comprendre 1, elle présente des inconvénients majeurs : elle est approximative et surtout,

**extrêmement lente**.1 Pour un réseau de neurones avec des centaines de millions de paramètres, cela nécessiterait des millions d'évaluations de la fonction de perte pour une seule mise à jour des paramètres, ce qui la rend impraticable en production.1

#### **4.3.3. Gradient Analytique : Précision et Efficacité**

Le gradient analytique est obtenu en utilisant les règles du calcul différentiel pour dériver une expression mathématique exacte du gradient de la fonction de perte par rapport à W.1 Cette approche est

**exacte** (sans approximation) et **très rapide** à calculer une fois l'expression dérivée.1 L'inconvénient principal est qu'elle est sujette aux erreurs de calcul humain lors de la dérivation.1

#### **4.3.4. L'Importance de la Vérification du Gradient (*Gradient Checking*)**

En pratique, le gradient analytique est toujours utilisé pour l'optimisation en raison de sa rapidité.1 Cependant, pour s'assurer que l'implémentation du gradient analytique est correcte, une "vérification du gradient numérique" est systématiquement effectuée.1 Cette procédure implique de calculer à la fois le gradient analytique et le gradient numérique (qui est lent mais sert ici de référence fiable) et de vérifier qu'ils sont pratiquement identiques.1 Si les deux gradients correspondent, la vérification est réussie, confirmant la justesse des calculs. Cette étape est cruciale lors du développement de tout nouveau module de réseau de neurones.1

| Caractéristique | Gradient Numérique | Gradient Analytique |
| :---- | :---- | :---- |
| **Précision** | Approximatif | Exact |
| **Vitesse de calcul** | Très lent (N évaluations de perte pour N paramètres) | Très rapide (une seule évaluation de l'expression) |
| **Facilité d'implémentation** | Très facile (formule générique) | Demande des compétences en calcul différentiel |
| **Risque d'erreur** | Faible (erreur d'approximation) | Élevé (erreurs de dérivation manuelle) |
| **Utilisation pratique** | Débogage (vérification du gradient) | Calcul en production |

Table 2 : Gradient Numérique vs. Gradient Analytique

### **4.4. La Descente de Gradient : Le Moteur de l'Optimisation**

#### **4.4.1. La Règle de Mise à Jour des Paramètres**

Le processus d'optimisation est une boucle itérative fondamentale. À chaque étape, le gradient de la fonction de perte par rapport aux poids W est évalué.1 Ensuite, les paramètres

W sont mis à jour en effectuant un petit pas dans la direction opposée au gradient. La règle de mise à jour est la suivante :

W \= W \- taux\_apprentissage \* gradient 1

Le signe négatif est essentiel car le gradient indique la direction de la plus forte augmentation de la perte, et l'objectif est de minimiser la perte, d'où la nécessité de se déplacer dans la direction opposée.1

#### **4.4.2. Le Pas d'Apprentissage (*Learning Rate*) : Un Hyperparamètre Crucial**

Le taux\_apprentissage (ou step\_size) est sans doute l'hyperparamètre le plus critique à régler dans tout le processus d'optimisation.1 Il détermine l'ampleur du pas effectué à chaque mise à jour des paramètres.1

* Un **taux d'apprentissage trop élevé** peut entraîner une instabilité. Le modèle "sautille" de manière excessive dans l'espace des poids, ce qui peut empêcher la convergence ou même provoquer une "explosion" de la perte.1 Un taux trop élevé peut également empêcher le modèle de s'installer dans des minima locaux plus petits, le laissant bloqué dans des positions sous-optimales.1  
* À l'inverse, un **taux d'apprentissage trop faible** entraîne des mises à jour très petites, ce qui rend la convergence extrêmement lente et inefficace.1

La dynamique du taux d'apprentissage est une stratégie clé pour naviguer efficacement dans le paysage de perte. Il est courant de commencer l'entraînement avec un taux d'apprentissage relativement élevé pour permettre des explorations rapides du paysage de perte et sortir des régions de perte élevée. Ensuite, ce taux est progressivement diminué (décroissance du taux d'apprentissage) au fur et à mesure que l'optimisation progresse, ce qui permet au modèle de s'installer finement dans un minimum.1 Cette approche en deux phases est cruciale pour la performance et la stabilité de l'entraînement des modèles complexes.

#### **4.4.3. Descente de Gradient par Mini-Batch : Efficacité pour les Grands Datasets**

Pour les grands ensembles de données, le calcul du gradient sur l'ensemble complet des données d'entraînement (descente de gradient par lot complet) est souvent trop coûteux en temps et en ressources. C'est pourquoi la "descente de gradient par mini-batch" est la méthode la plus couramment utilisée.1

Dans cette approche, un petit sous-ensemble aléatoire de données d'entraînement, appelé "mini-batch" (par exemple, 32, 64, 128 ou 256 exemples), est échantillonné à chaque itération.1 La perte et le gradient sont ensuite calculés uniquement sur ce mini-batch, et une mise à jour des paramètres est effectuée.1

Bien que l'estimation du gradient obtenue à partir d'un mini-batch soit plus "bruyante" que celle d'un lot complet (car elle est basée sur un sous-ensemble des données), la possibilité d'effectuer beaucoup plus de mises à jour par unité de temps compense largement ce bruit.1 Cela conduit généralement à une convergence plus rapide et plus efficace en pratique.1 La taille du mini-batch est souvent déterminée par la mémoire disponible sur le GPU, car les cartes graphiques ont une quantité de mémoire finie (par exemple, 6 Go ou 12 Go).1 Ce compromis entre un gradient approximatif mais fréquent et un gradient exact mais rare est un principe fondamental de l'efficacité computationnelle en apprentissage profond.

### **4.5. Introduction aux Optimiseurs Avancés (Momentum, Adam, RMSprop)**

La descente de gradient stochastique (SGD) simple, avec sa règle de mise à jour directe, est la méthode la plus basique. Cependant, des optimiseurs plus sophistiqués ont été développés pour améliorer la vitesse et la qualité de la convergence.1

* **Momentum :** Cet algorithme introduit une notion de "vélocité" dans l'espace des poids.1 Si le gradient pointe constamment dans une certaine direction sur plusieurs itérations, la vitesse s'accumule, ce qui a pour effet d'accélérer les mises à jour dans cette direction.1 Cela aide le processus d'optimisation à traverser les minima locaux peu profonds et à accélérer la convergence dans les vallées plates du paysage de perte.1  
* **Adam et RMSprop :** Ces algorithmes sont des optimiseurs plus avancés qui incorporent des mécanismes de taux d'apprentissage adaptatifs pour chaque paramètre.1 Ils ajustent dynamiquement la taille du pas pour chaque poids en fonction de l'historique de ses gradients, ce qui leur permet de naviguer plus efficacement dans des paysages de perte complexes. Ces méthodes sont souvent plus efficaces que le SGD pur en termes de vitesse et de qualité de convergence.1

## **5\. Contexte Historique : La Vision par Ordinateur Avant les Réseaux de Neurones Convolutifs (CNNs)**

### **5.1. Le Pipeline Traditionnel : Extraction de Caractéristiques Manuelles et Classification Linéaire**

Avant l'avènement des réseaux de neurones convolutifs (CNNs) aux alentours de 2012, la vision par ordinateur s'appuyait majoritairement sur un pipeline traditionnel en deux étapes, caractérisé par une forte dépendance à des composants d'ingénierie manuelle.1

1. **Extraction de caractéristiques :** Cette première étape consistait à calculer des caractéristiques "ingénierisées" (hand-engineered) à partir de l'image brute. Les chercheurs devaient décider manuellement quelles informations étaient "importantes" à extraire d'une image, telles que les fréquences, les textures, ou les formes.1 Il était courant de combiner plusieurs types de caractéristiques, parfois jusqu'à une dizaine dans une seule publication, pour former un vecteur de caractéristiques géant.1  
2. **Classification linéaire :** Une fois ces caractéristiques extraites et concaténées en un vecteur unique, un classificateur linéaire, tel qu'une machine à vecteurs de support (SVM) linéaire, était appliqué sur ce vecteur pour effectuer la tâche de classification.1

La séparation entre l'extraction de caractéristiques et la classification représentait une limitation majeure. Les caractéristiques étaient conçues sans une connaissance directe de la manière dont elles seraient finalement utilisées par le classificateur. Ce processus exigeait une expertise approfondie du domaine et un travail manuel considérable pour concevoir des caractéristiques pertinentes, ce qui rendait l'ensemble du pipeline sous-optimal et difficile à adapter à de nouveaux problèmes.

### **5.2. Exemples de Caractéristiques Ingénierisées**

Plusieurs types de caractéristiques ont été développés et largement utilisés dans ce pipeline traditionnel :

* **Histogrammes de couleurs :** Une méthode très simple consistait à parcourir tous les pixels d'une image et à les classer dans des "bacs" (bins) en fonction de leur teinte de couleur. Cela produisait un résumé statistique de la distribution des couleurs dans l'image, servant de caractéristique pour le classificateur.1  
* **Caractéristiques SIFT (Scale-Invariant Feature Transform) et HOG (Histogram of Oriented Gradients) :** Ces caractéristiques étaient très populaires et impliquaient l'analyse de voisinages locaux dans l'image pour détecter la présence et l'orientation des bords (par exemple, bords horizontaux ou verticaux). Des histogrammes étaient ensuite créés à partir de ces orientations, fournissant un résumé des types de bords et de leur emplacement dans l'image.1  
* **Pipelines "Bag of Words" (Sac de Mots Visuels) :** Cette approche consistait à décrire de petits patchs locaux d'images à l'aide de divers schémas (par exemple, en analysant les fréquences ou les couleurs). Ces descriptions locales étaient ensuite utilisées pour créer un "dictionnaire" d'éléments visuels courants. Chaque image était alors représentée comme une statistique sur la présence de ces "mots visuels" du dictionnaire.1

### **5.3. La Révolution de l'Apprentissage de Bout en Bout (*End-to-End Learning*) avec les CNNs**

La grande innovation qui a émergé vers 2012, principalement avec l'avènement des CNNs, a été le passage d'un pipeline manuel à une approche "de bout en bout" (*end-to-end learning*).1

Cette approche moderne contraste fortement avec le modèle traditionnel :

* **Entrée d'image brute :** Au lieu de commencer par des caractéristiques pré-calculées, les systèmes modernes prennent directement l'image brute (les pixels) comme entrée.1  
* **Architecture intégrée :** L'ensemble du système est conçu comme une "boîte noire différenciable" (*differentiable blob*), où l'extraction de caractéristiques et la classification sont intégrées dans une architecture unifiée.1 Cette architecture est capable de simuler et d'apprendre de nombreuses caractéristiques qui étaient auparavant conçues manuellement.1  
* **Entraînement complet :** Contrairement à la méthode traditionnelle où seul le classificateur linéaire était entraîné sur des caractéristiques fixes, l'approche de bout en bout permet d'entraîner l'ensemble du modèle, y compris les extracteurs de caractéristiques, directement à partir des pixels, en utilisant des techniques d'optimisation comme la descente de gradient.1  
* **Élimination de l'ingénierie manuelle :** L'innovation majeure a été d'éliminer un grand nombre de composants conçus manuellement, permettant au modèle d'apprendre les caractéristiques les plus efficaces directement à partir des données.1

Ce changement de paradigme a transformé la vision par ordinateur, passant d'une approche "dirigée par l'humain" à une approche "dirigée par les données". Les modèles peuvent désormais découvrir des motifs et des représentations beaucoup plus complexes et efficaces que ce que les humains pourraient concevoir manuellement, ce qui a conduit aux performances révolutionnaires observées dans les CNNs.

## **6\. Conclusion et Prochaines Étapes**

### **6.1. Récapitulatif des Concepts Clés Abordés**

Cette conférence a exploré les fondements essentiels de l'apprentissage automatique pour la classification d'images. Il a été démontré comment quantifier l'erreur d'un modèle grâce aux fonctions de perte, en détaillant la perte SVM et la perte Softmax, chacune avec ses interprétations et ses préférences distinctes. La nécessité de la régularisation, en particulier la régularisation L2, a été mise en évidence comme un moyen crucial d'améliorer la généralisation du modèle en encourageant des poids "souhaitables" et en évitant le surapprentissage. Enfin, les principes de l'optimisation ont été abordés, de la recherche de gradient à la descente de gradient par mini-batch, soulignant l'importance du taux d'apprentissage et l'efficacité des optimiseurs avancés. Ces concepts ont été replacés dans leur contexte historique, illustrant l'évolution de la vision par ordinateur des caractéristiques manuelles vers l'apprentissage de bout en bout.

### **6.2. Introduction à la Rétropropagation (*Backpropagation*) : Le Sujet de la Prochaine Conférence**

La prochaine étape cruciale dans la compréhension de l'apprentissage automatique sera d'aborder la rétropropagation (*backpropagation*).1 Cette méthode est indispensable pour calculer efficacement les gradients analytiques dans des architectures complexes telles que les réseaux de neurones.1 La maîtrise de la rétropropagation est une compétence fondamentale et un passage obligé pour quiconque souhaite progresser au-delà des classificateurs linéaires et s'engager dans le domaine des réseaux de neurones.1

#### **Sources des citations**

1. transcription.txt
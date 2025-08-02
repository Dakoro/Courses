

# **Introduction à la Vision par Ordinateur et aux Réseaux de Neurones**

La vision par ordinateur, un domaine en pleine effervescence, a connu des avancées révolutionnaires ces dernières années, largement propulsées par l'essor des réseaux de neurones. Pour appréhender ces progrès et comprendre comment les systèmes informatiques peuvent "voir" et interpréter le monde visuel, il est essentiel de maîtriser les mécanismes fondamentaux qui permettent à ces réseaux d'apprendre à partir de données. La capacité intrinsèque des réseaux de neurones à extraire des représentations complexes à partir d'images brutes constitue la pierre angulaire de leur succès dans des tâches telles que la classification d'images.1 Cette aptitude à identifier et à apprendre des caractéristiques pertinentes directement à partir des pixels distingue les approches modernes des méthodes traditionnelles, rendant les réseaux de neurones indispensables pour la vision par ordinateur contemporaine.

Avant de plonger dans les complexités des réseaux de neurones, il convient de rappeler quelques concepts fondamentaux. Tout modèle d'apprentissage automatique, y compris les réseaux de neurones, repose sur une **fonction de score** qui attribue une valeur numérique à chaque classe possible pour une entrée donnée. L'objectif est de maximiser le score de la classe correcte. Pour évaluer la performance de ce modèle, une **fonction de perte** est utilisée. Cette fonction quantifie à quel point les prédictions du modèle sont "mauvaises" par rapport à la vérité terrain. Par exemple, la fonction de perte SVM (Support Vector Machine) est un type de fonction de perte couramment employée.1 La perte totale d'un modèle sur l'ensemble des données d'entraînement est généralement composée de deux éléments distincts : la

**perte de données**, qui mesure l'erreur directe du modèle sur les exemples d'entraînement, et la **perte de régularisation**, qui ajoute une pénalité pour la complexité du modèle afin de prévenir le surapprentissage et d'améliorer sa capacité à généraliser sur de nouvelles données.1

# **L'Optimisation par les Gradients : Le Moteur de l'Apprentissage**

Le but ultime de l'entraînement d'un modèle d'apprentissage automatique est de minimiser cette fonction de perte par rapport aux poids (ou paramètres) du modèle. Cette minimisation est un problème d'optimisation, et les **gradients** sont les informations clés qui guident ce processus. La **descente de gradient** est la méthode itérative par excellence pour y parvenir. Elle consiste à évaluer le gradient de la fonction de perte par rapport aux poids, puis à ajuster ces paramètres dans la direction opposée au gradient. Ce processus est répété encore et encore, permettant au modèle de converger progressivement vers un minimum local de la fonction de perte.1 Atteindre une faible valeur de perte est directement équivalent à obtenir de bonnes prédictions sur les données d'entraînement.1 La nécessité des gradients découle directement de cet objectif d'optimisation. Sans un calcul efficace et précis des gradients, le processus itératif de la descente de gradient, qui est fondamental pour l'entraînement des modèles complexes, serait impraticable, voire impossible.

Il existe principalement deux manières d'évaluer ces gradients : le **gradient numérique** et le **gradient analytique**. Le gradient numérique est conceptuellement simple à écrire et à comprendre, car il s'agit d'une approximation de la dérivée obtenue en évaluant la fonction à des points très proches. Cependant, cette méthode est extrêmement lente à évaluer en pratique, surtout pour des modèles avec un grand nombre de paramètres.1 En revanche, le

**gradient analytique** est obtenu par l'application des règles du calcul différentiel. Il est à la fois rapide et exact, ce qui le rend indispensable pour l'entraînement des modèles à grande échelle.1

Malgré sa rapidité et son exactitude, la dérivation et l'implémentation du gradient analytique peuvent être sujettes à des erreurs. C'est pourquoi il est crucial de toujours effectuer une **vérification de gradient** (ou "Gradient Check"). Cette procédure consiste à comparer le gradient analytique calculé avec une approximation du gradient numérique pour s'assurer de sa correction.1 Cette vérification est une étape essentielle de débogage qui garantit la fiabilité du processus d'optimisation.

# **Les Graphes Computationnels : Une Approche Structurée du Calcul**

Face à la complexité croissante des modèles d'apprentissage automatique, notamment les réseaux de neurones, il est devenu impraticable de dériver une expression géante pour la fonction de perte totale et ses gradients. Une approche beaucoup plus efficace et structurée consiste à penser en termes de **graphes computationnels**.1 Ces graphes représentent le flux de valeurs à travers une série d'opérations individuelles, souvent appelées "portes" ou "fonctions", qui transforment les entrées initiales jusqu'à la fonction de perte finale.1 Les entrées de ce graphe sont les données brutes et les paramètres du modèle, et la sortie finale est un unique nombre : la valeur de la perte.1

L'adoption des graphes computationnels n'est pas seulement une astuce d'optimisation, mais un changement de paradigme fondamental dans la conception et l'implémentation des modèles de machine learning complexes, en particulier le deep learning. Cette abstraction est devenue une nécessité pour gérer la complexité et permettre la différenciation automatique à grande échelle. Les expressions mathématiques des modèles modernes, comme les réseaux de neurones convolutifs (CNN) qui peuvent avoir des dizaines, voire des centaines d'opérations 1, sont devenues trop volumineuses pour être manipulées manuellement. Des modèles encore plus avancés, tels que les Neural Turing Machines ou les réseaux de neurones récurrents (RNN) déroulés sur des centaines de pas de temps, peuvent générer des graphes "monstrueux" contenant des centaines de milliers de nœuds.1 Dans de tels cas, la dérivation manuelle des gradients est tout simplement impossible.1 Il est donc impératif de concevoir ces systèmes comme des "structures de données composées de petites fonctions transformant des variables intermédiaires".1

Le calcul dans un graphe computationnel se déroule en deux phases principales : la **propagation avant** et la **rétropropagation**.

### **Le "Forward Pass" (Propagation Avant)**

La phase de propagation avant consiste à parcourir le graphe de gauche à droite. À chaque porte, les entrées sont utilisées pour calculer la sortie de cette opération. Par exemple, si nous avons un graphe simple avec des entrées X, Y et Z (prenant les valeurs \-2, 5 et \-4 respectivement), nous calculons une variable intermédiaire Q comme la somme de X et Y (Q \= \-2 \+ 5 \= 3), puis la sortie finale F comme le produit de Q et Z (F \= 3 \* \-4 \= \-12).1 Ce processus se poursuit jusqu'à ce que la valeur de la fonction de perte finale soit obtenue.

### **Le "Backward Pass" (Rétropropagation)**

La rétropropagation (ou *backpropagation*) est le processus inverse. Elle consiste à parcourir le graphe de droite à gauche, en calculant les gradients de la perte finale par rapport à toutes les variables intermédiaires et, finalement, par rapport aux entrées du modèle.1 Le point de départ de cette récursion est le gradient de la fonction de perte

F par rapport à elle-même, qui est toujours 1\.1

Le principe fondamental de la rétropropagation est l'application de la **règle de la chaîne**. Pour chaque porte dans le graphe, le gradient de la perte par rapport à l'une de ses entrées est obtenu en multipliant le gradient qui arrive à la porte (le "gradient global" de la perte par rapport à la sortie de la porte) par le gradient local de l'opération de la porte par rapport à cette entrée spécifique.1 La rétropropagation est donc une application récursive et systématique de la règle de la chaîne à travers l'ensemble du graphe computationnel, permettant de déterminer l'influence de chaque valeur intermédiaire sur la fonction de perte finale.1

### **Exemples Détaillés de Rétropropagation**

Reprenons l'exemple simple du graphe F \= Q \* Z où Q \= X \+ Y, avec X \= \-2, Y \= 5, Z \= \-4, et F \= \-12.

1. **Gradient de base :** dF/dF \= 1\.  
2. **Porte de multiplication (F \= Q \* Z) :**  
   * Le gradient de F par rapport à Z est Q. Avec Q \= 3, dF/dZ \= 3\.1 Cela signifie qu'une petite augmentation de  
     Z de H unités entraînera une augmentation de F de 3H unités.1  
   * Le gradient de F par rapport à Q est Z. Avec Z \= \-4, dF/dQ \= \-4.1  
3. **Porte d'addition (Q \= X \+ Y) :**  
   * Le gradient dF/dQ \= \-4 est propagé à cette porte.  
   * Le gradient local de Q par rapport à Y (dQ/dY) est 1\. Par la règle de la chaîne, dF/dY \= (dF/dQ) \* (dQ/dY) \= \-4 \* 1 \= \-4.1  
   * De même, le gradient local de Q par rapport à X (dQ/dX) est 1\. Donc, dF/dX \= (dF/dQ) \* (dQ/dX) \= \-4 \* 1 \= \-4.1

Un exemple plus complexe est la fonction sigmoïde, σ(x)=1/(1+e−x), qui peut être décomposée en une série de portes élémentaires : une multiplication par \-1, une exponentiation, une addition de 1, et une division par 1\.1 Le fait que des fonctions complexes comme la sigmoïde puissent être décomposées en portes plus simples, et que la rétropropagation applique systématiquement la règle de la chaîne à travers ces compositions, démontre la puissance de cette approche modulaire. Cette composabilité permet aux frameworks de deep learning de construire des modèles arbitrairement complexes à partir d'une bibliothèque d'opérations de base, chaque porte appliquant sa dérivée locale multipliée par le gradient reçu de la porte suivante.1

### **Intuitions sur le Flux des Gradients**

Comprendre les intuitions derrière le comportement des gradients à travers différentes portes est très utile pour le débogage, notamment pour identifier des problèmes comme le gradient évanescent.1

* **Porte d'addition (+)** : Agit comme un "distributeur de gradient". Le gradient local de la porte d'addition est toujours 1 pour toutes ses entrées. Par conséquent, le gradient entrant est distribué également à toutes les entrées.1  
* **Porte Max (max(X, Y))** : Agit comme un "routeur de gradient". Le gradient local est 1 pour l'entrée qui était la plus grande lors de la propagation avant, et 0 pour les autres. Le gradient entrant est donc routé uniquement vers l'entrée qui a déterminé la sortie de la porte.1  
* **Porte de multiplication (\*)** : Le gradient local pour une entrée est la valeur de l'autre entrée.1

Comprendre ces rôles intuitifs des portes fournit un modèle mental puissant pour diagnostiquer et déboguer les problèmes de flux de gradient, tels que les gradients évanescents ou explosifs, qui sont des défis courants dans l'entraînement des réseaux profonds. Si une partie du réseau n'apprend pas, il est possible de suivre mentalement le flux des gradients et d'identifier où ils pourraient être "tués" (par exemple, par une porte Max ou ReLU pour des valeurs négatives) ou "dilués".

### **Gestion des Branches Multiples**

Un point crucial de la règle de la chaîne multivariée est la gestion des branches multiples. Si une variable intermédiaire se ramifie et est utilisée par plusieurs opérations en aval dans le graphe, les contributions de gradient de toutes ces branches doivent être **additionnées** lorsqu'elles sont rétropropagées vers cette variable.1 La règle de sommation des gradients est essentielle pour gérer correctement les paramètres partagés ou les chemins de calcul ramifiés. Ignorer cette règle conduirait à des calculs de gradient incorrects et à un entraînement inefficace ou échoué du modèle. Il est important de noter que dans ces graphes computationnels, il n'y a jamais de boucles. Les réseaux de neurones récurrents, par exemple, sont "déroulés" dans le temps pour former des graphes acycliques dirigés (DAGs).1

# **Implémentation Pratique des Graphes Computationnels**

Les frameworks de deep learning modernes structurent les réseaux de neurones autour de deux concepts principaux : l'objet "Net" (ou "Graph") et les objets "Gate" ou "Layer" (couches).

### **Architecture Logicielle : Objets "Net" et "Gate" / "Layer"**

L'objet "Net" est responsable de la gestion de la structure de connectivité globale du graphe computationnel. Il assure que les opérations sont exécutées dans le bon **ordre topologique**, c'est-à-dire que toutes les entrées d'une porte sont disponibles avant que sa sortie ne soit calculée.1 Les objets "Gate" ou "Layer" sont les blocs de construction individuels du réseau, chacun implémentant une opération spécifique.1 L'objet "Net" parcourt ces couches dans l'ordre topologique pour la propagation avant et dans l'ordre inverse pour la rétropropagation.1

### **L'API forward et backward des Couches**

Chaque porte ou couche doit implémenter une interface de programmation d'application (API) standard, comprenant deux méthodes principales :

* La méthode forward : Elle prend les entrées, effectue l'opération de la porte et retourne la sortie.1  
* La méthode backward : Elle reçoit le gradient de la perte par rapport à sa propre sortie (souvent noté dZ), calcule les gradients de la perte par rapport à ses entrées (par exemple, dX et dY pour une porte binaire) en appliquant la règle de la chaîne, et les retourne. Ces gradients sont ensuite routés correctement vers les portes précédentes par le graphe computationnel.1

### **Gestion de la Mémoire : Importance de la Mise en Cache des Valeurs Intermédiaires**

Un aspect crucial de l'implémentation est la gestion de la mémoire. Lors de la propagation avant, chaque porte doit stocker (mettre en cache) ses entrées et toute valeur intermédiaire qui sera nécessaire pour le calcul de son gradient local pendant la rétropropagation.1 Cela signifie que la mémoire utilisée peut augmenter considérablement pendant la propagation avant, puis être consommée pendant la rétropropagation.1 L'exigence de mise en cache de la mémoire pendant la propagation avant pour les calculs de rétropropagation ultérieurs représente une contrainte pratique significative, en particulier pour les réseaux profonds ou les grandes tailles de lot. Cela impose un compromis entre la complexité du modèle ou la taille du lot et la mémoire disponible sur le matériel (souvent des GPU).

### **Vectorisation et Tenseurs**

En pratique, les valeurs qui circulent dans le graphe ne sont pas de simples scalaires, mais des vecteurs ou des **tenseurs** (des tableaux N-dimensionnels, similaires aux tableaux NumPy en Python).1 L'utilisation de tenseurs permet des opérations vectorisées très efficaces, ce qui est essentiel pour les calculs sur les processeurs graphiques (GPU).

### **Les Matrices Jacobiennes : Pourquoi elles sont Conceptuelles et Rarement Formées Explicitement**

Conceptuellement, lorsque les entrées et les sorties d'une porte sont des vecteurs, le gradient local de cette porte est une **matrice Jacobienne**. Cette matrice décrit l'influence de chaque élément d'entrée sur chaque élément de sortie.1 Cependant, en pratique, les Jacobiennes complètes ne sont

*jamais* formées explicitement. Par exemple, pour une couche ReLU traitant 4096 entrées, la Jacobienne serait une matrice de 4096x4096, soit 16 millions de nombres.1 De plus, avec des mini-lots de données (par exemple, 100 éléments à la fois), la taille des tenseurs et des Jacobiennes conceptuelles peut atteindre des millions de dimensions (ex: 400 000 x 400 000).1

Pour les opérations élément par élément (comme ReLU), la Jacobienne est une matrice diagonale. Le calcul du gradient arrière se simplifie alors considérablement. Pour ReLU, le gradient arrière consiste simplement à mettre à zéro les gradients des dimensions où l'entrée était inférieure ou égale à zéro.1 Les implémentations pratiques exploitent la structure creuse de ces Jacobiennes et codent manuellement les opérations de rétropropagation pour maximiser l'efficacité.1 Bien que la matrice Jacobienne soit la représentation mathématiquement correcte pour les gradients de vecteurs, les implémentations pratiques ne la forment jamais explicitement. Au lieu de cela, elles exploitent la structure spécifique des opérations pour effectuer des calculs spécialisés et efficaces. C'est une optimisation cruciale qui permet au deep learning de s'adapter aux données de haute dimension.

### **Exemples de Pseudocode**

Pour illustrer ces concepts, voici des exemples de pseudocode pour la structure générale d'un réseau et l'implémentation d'une porte de multiplication.

Extrait de code

CLASSE Net:  
    FONCTION \_\_init\_\_(gates):  
        self.gates \= gates \# Liste des portes/couches dans l'ordre topologique

    FONCTION forward(inputs):  
        outputs \= inputs  
        POUR chaque gate DANS self.gates:  
            outputs \= gate.forward(outputs)  
        RETOURNE outputs

    FONCTION backward(grad\_output):  
        grad\_inputs \= grad\_output  
        POUR chaque gate DANS self.gates EN ORDRE INVERSE:  
            grad\_inputs \= gate.backward(grad\_inputs)  
        RETOURNE grad\_inputs

1

Extrait de code

CLASSE MultiplyGate:  
    FONCTION forward(x, y):  
        self.x \= x  \# Mémoriser x pour la passe arrière  
        self.y \= y  \# Mémoriser y pour la passe arrière  
        z \= x \* y  
        RETOURNE z

    FONCTION backward(dz):  
        dx \= self.y \* dz  \# Appliquer la règle de la chaîne: gradient local (y) \* gradient entrant (dz)  
        dy \= self.x \* dz  \# Appliquer la règle de la chaîne: gradient local (x) \* gradient entrant (dz)  
        RETOURNE dx, dy

1

Extrait de code

CLASSE Neuron:  
    FONCTION \_\_init\_\_(weights, bias, activation\_function):  
        self.weights \= weights  
        self.bias \= bias  
        self.activation\_function \= activation\_function

    FONCTION forward(inputs):  
        cell\_body\_sum \= SOMME(inputs \* self.weights) \+ self.bias  
        firing\_rate \= self.activation\_function.forward(cell\_body\_sum)  
        RETOURNE firing\_rate

1

### **Aperçu des Frameworks de Deep Learning**

Les frameworks de deep learning comme Torch et Caffe sont, à leur base, de vastes collections de ces objets "Layer" (couches ou portes).1 Chaque couche dans ces frameworks implémente son propre

forward et backward (par exemple, MulConstantLayer dans Torch ou SigmoidLayer dans Caffe).1 Ces couches agissent comme des "blocs Lego" que les utilisateurs assemblent pour construire des graphes computationnels complexes, permettant une grande flexibilité dans la conception des architectures de réseaux de neurones.1

# **Les Réseaux de Neurones Profonds : De la Théorie à la Pratique**

Les modèles d'apprentissage automatique ont évolué des classifieurs linéaires simples, où la fonction de score était directement F \= WX, vers des architectures beaucoup plus complexes. Les réseaux de neurones introduisent une complexité supplémentaire avec l'ajout de **couches cachées** et de **non-linéarités**, transformant l'équation en quelque chose comme F \= W2 \* max(0, W1\*X).1

### **La Couche Cachée : Rôle des Représentations Intermédiaires**

La couche cachée, souvent désignée par H, confère au réseau la capacité de développer des représentations intermédiaires des données.1 Cette capacité offre au réseau une "marge de manœuvre" considérable pour apprendre des caractéristiques intéressantes et complexes. Par exemple, dans une tâche de classification de voitures pour le jeu de données CIFAR-10, un classifieur linéaire simple devrait tenter de couvrir toutes les variations (voitures rouges, vertes, orientées différemment) avec un seul modèle. En revanche, une couche cachée peut apprendre des "modèles" spécifiques : un neurone pourrait s'activer uniquement pour une "voiture rouge orientée vers l'avant", tandis qu'un autre s'activerait pour une "voiture verte orientée vers la gauche".1 Ces neurones de la couche cachée deviennent positifs s'ils détectent la caractéristique qu'ils recherchent dans l'image, et la couche suivante peut alors combiner ces activations pondérées pour effectuer la classification finale. L'introduction d'une couche cachée non linéaire augmente considérablement la puissance de représentation du modèle, lui permettant d'apprendre des frontières de décision complexes et non linéaires et de capturer des motifs multimodaux dans les données, ce qu'un classifieur linéaire simple ne peut pas faire. Cette capacité à apprendre des représentations hiérarchiques et non linéaires est la clé de la performance des réseaux de neurones profonds.

### **L'Analogie Biologique : Le Modèle Simplifié du Neurone**

Historiquement, les réseaux de neurones ont été inspirés par le fonctionnement du cerveau biologique, bien que cette analogie soit une simplification extrême et ne doive pas être prise au pied de la lettre.1 Un neurone biologique se compose d'un corps cellulaire (soma), de dendrites qui reçoivent les signaux d'entrée d'autres neurones, et d'un axone qui transmet le signal de sortie.1

Le modèle rudimentaire d'un neurone artificiel s'inspire de cette structure :

* Les entrées (X) proviennent d'autres neurones et sont multipliées par des "poids" (W) au niveau des "synapses".  
* Une somme pondérée (W\*X) est calculée au "corps cellulaire" (soma), à laquelle est ajouté un terme de "biais".  
* Le résultat de cette somme passe ensuite par une **fonction d'activation** (historiquement, la sigmoïde était utilisée pour produire une valeur entre 0 et 1, interprétée comme un "taux de déclenchement" du neurone) pour produire la sortie de l'axone.1

Chaque neurone artificiel agit donc comme un petit classifieur linéaire suivi d'une non-linéarité.1 Il est crucial de noter que cette analogie est une simplification grossière. Les neurones biologiques sont des systèmes dynamiques et complexes, avec de nombreux types différents et des dendrites capables de calculs sophistiqués.1 L'inspiration biologique est plus historique et métaphorique que littérale, et il est important de ne pas pousser cette analogie trop loin pour éviter les idées fausses.

### **Fonctions d'Activation (Non-Linéarités)**

Les fonctions d'activation sont des composants essentiels des neurones artificiels car elles introduisent la **non-linéarité** dans le réseau. Sans elles, un réseau de neurones, quelle que soit sa profondeur, ne serait qu'une série de transformations linéaires, équivalente à une seule transformation linéaire, limitant considérablement sa capacité à apprendre des relations complexes.

Historiquement, la **Sigmoïde** et la **Tanh (tangente hyperbolique)** ont été largement utilisées.1 La sigmoïde produit une sortie entre 0 et 1, tandis que la Tanh est similaire mais centrée autour de 0\. Cependant, ces fonctions souffrent du problème du "gradient évanescent" où les gradients deviennent très petits dans les régions saturées, ralentissant l'apprentissage.

Depuis 2012, la **ReLU (Rectified Linear Unit)**, définie comme max(0, X), est devenue extrêmement populaire. Elle permet aux réseaux de converger beaucoup plus rapidement car elle ne sature pas pour les valeurs positives, évitant ainsi le problème du gradient évanescent dans cette région. La ReLU est actuellement le choix par défaut recommandé pour la plupart des réseaux de neurones.1

D'autres variantes de ReLU ont été proposées pour pallier le problème des "neurones morts" (où un neurone ne s'active jamais si son entrée est toujours négative) :

* **Leaky ReLU** : Introduit une petite pente pour les valeurs négatives.  
* **ELU (Exponential Linear Unit)** : Combine les avantages de ReLU avec une sortie proche de zéro et sans neurones morts.  
* **Maxout** : Une fonction d'activation plus complexe qui prend le maximum de plusieurs entrées linéaires.

La recherche active continue de chercher des fonctions d'activation avec de meilleures propriétés.1 Le choix de la fonction d'activation n'est pas arbitraire ; il a un impact significatif sur la dynamique d'entraînement (vitesse de convergence, problèmes de gradient évanescent) et la capacité du modèle à apprendre des motifs complexes. L'évolution de Sigmoïde/Tanh vers ReLU illustre une tendance évolutive dans le deep learning, guidée par la performance empirique.

Voici un tableau comparatif des fonctions d'activation courantes :

**Tableau 1: Comparaison des Fonctions d'Activation Courantes**

| Fonction d'Activation | Formule | Plage de Sortie | Avantages Clés | Inconvénients/Considérations |
| :---- | :---- | :---- | :---- | :---- |
| Sigmoïde | σ(x)=1/(1+e−x) | (0, 1\) | Historiquement populaire, sortie interprétable comme probabilité/taux de déclenchement. | Problème de gradient évanescent (sature aux extrêmes), non centrée à zéro (ralentit la convergence). |
| Tanh | tanh(x)=(ex−e−x)/(ex+e−x) | (-1, 1\) | Centrée à zéro (meilleure convergence que Sigmoïde), gradient plus fort que Sigmoïde. | Problème de gradient évanescent (sature aux extrêmes). |
| ReLU | max(0,x) | Le nombre de couches d'un réseau de neurones est déterminé par le nombre de couches qui contiennent des poids (la couche d'entrée ne compte pas).1 |  |  |

Cette organisation en couches est avant tout une **astuce computationnelle**.1 Elle permet d'évaluer un ensemble entier de neurones (une couche cachée) en parallèle via une simple multiplication matricielle, ce qui est extrêmement efficace pour les calculs sur GPU.1 Un réseau de neurones est donc, en substance, une séquence de multiplications matricielles suivies de l'application de fonctions d'activation non linéaires.1 L'architecture en couches des réseaux de neurones, bien que conceptuellement attrayante pour l'apprentissage hiérarchique des caractéristiques, est fondamentalement motivée par l'efficacité computationnelle. L'organisation des neurones en couches permet une parallélisation massive via des opérations matricielles, ce qui est crucial pour l'entraînement de grands modèles sur du matériel moderne.

### **Démonstration Conceptuelle**

Des démonstrations interactives permettent de visualiser comment un réseau de neurones classifie des données en 2D et comment il "déforme" l'espace d'entrée. Plus le nombre de neurones dans la couche cachée est élevé, plus le réseau dispose de "marge de manœuvre" pour apprendre des fonctions complexes et des frontières de décision non linéaires.1

La **régularisation** joue un rôle crucial dans la complexité des frontières de décision apprises. Une forte régularisation (qui pénalise les grands poids) conduit à des fonctions plus lisses et à une variance réduite, diminuant le risque de surapprentissage.1 À l'inverse, une faible régularisation permet au réseau d'apprendre des fonctions plus complexes, s'adaptant mieux aux points "atypiques" dans les données d'entraînement, mais augmentant le risque de surapprentissage.1 La démonstration illustre de manière frappante le compromis fondamental entre la capacité d'un réseau de neurones (nombre de neurones cachés) et la nécessité de la régularisation. Une capacité accrue permet d'apprendre des fonctions plus complexes, mais augmente également le risque de surapprentissage, qui doit être contrecarré par une régularisation appropriée.

Une interprétation visuelle frappante est que le réseau de neurones "déforme" l'espace d'entrée de telle manière qu'une simple classification linéaire devient possible dans la dernière couche. Ce concept est similaire au "truc du noyau" (kernel trick) en machine learning.1 Pour des données en forme de cercle, par exemple, un minimum de 3 neurones cachés est nécessaire pour une séparation correcte, car trois lignes droites peuvent "sculpter" l'espace de manière à rendre les classes linéairement séparables.1 Le choix de la fonction d'activation a également un impact sur la netteté des frontières de décision : ReLU tend à produire des frontières plus nettes que Tanh.1

### **Le Cycle d'Entraînement : Forward, Backward, Mise à jour des Poids**

Le processus d'entraînement d'un réseau de neurones est une boucle continue et itérative, qui se déroule comme suit :

1. **Propagation avant (Forward Pass)** : Pour un lot de données d'entraînement (mini-batch), le réseau calcule sa sortie et la valeur correspondante de la fonction de perte.1  
2. **Rétropropagation (Backward Pass)** : Les gradients de la fonction de perte par rapport à tous les poids du réseau sont calculés en appliquant la règle de la chaîne à travers le graphe computationnel.1  
3. **Mise à jour (Update)** : Les poids du réseau sont ajustés en utilisant les gradients calculés (par exemple, via la descente de gradient stochastique), dans le but de minimiser la fonction de perte.1

Ce cycle "forward, backward, update" est répété des milliers, voire des millions de fois, jusqu'à ce que le modèle converge ou qu'un critère d'arrêt soit atteint.1 Le cycle "propagation avant, rétropropagation, mise à jour" est le paradigme d'entraînement universel pour la quasi-totalité des modèles de deep learning. Comprendre cette boucle fondamentale est primordial, car toute application pratique du deep learning implique l'itération de ce cycle.

Concernant la capacité du réseau, un plus grand nombre de neurones est généralement préférable, car cela augmente la capacité du modèle à apprendre des fonctions complexes.1 Cependant, une capacité accrue doit être accompagnée d'une régularisation appropriée pour éviter le surapprentissage. Le choix entre un réseau plus profond (plus de couches) et un réseau plus large (plus de neurones par couche) est complexe et dépend souvent des données, bien que pour les images, la profondeur soit souvent jugée critique.1 La régularisation est généralement appliquée de manière uniforme à toutes les couches, bien que des approches plus sophistiquées existent.1 L'utilisation de dérivées secondes (Hessiennes) pour l'optimisation est rare pour les grands ensembles de données en raison de leur coût computationnel.1

# **Conclusion**

Cette conférence a exploré les fondements des réseaux de neurones, en mettant en lumière leur rôle central dans les avancées de la vision par ordinateur. Il a été établi que les réseaux de neurones sont des structures computationnelles complexes, gérées efficacement par des graphes computationnels.1 Au cœur de leur apprentissage se trouve la

**rétropropagation**, une application récursive de la règle de la chaîne, qui est la méthode clé pour calculer les gradients nécessaires à l'optimisation des poids du modèle.1

Les frameworks de deep learning encapsulent cette complexité en fournissant des "couches" avec des API forward et backward, manipulant des tenseurs (tableaux N-dimensionnels) pour des opérations vectorisées efficaces.1 Grâce à leurs couches cachées non linéaires, les réseaux de neurones profonds peuvent apprendre des représentations complexes et résoudre des problèmes non linéaires, dépassant largement les capacités des classifieurs linéaires simples.1 Bien que l'analogie biologique ait inspiré le terme, les neurones artificiels sont des modèles mathématiques simplifiés, dont la puissance réside dans leur organisation en couches et l'introduction de fonctions d'activation non linéaires.

La compréhension de ces principes fondamentaux, de la propagation avant à la rétropropagation en passant par la gestion des gradients et l'architecture des couches, est essentielle pour quiconque souhaite comprendre et travailler avec les réseaux de neurones en vision par ordinateur. Ces concepts ouvrent la voie à l'étude d'architectures plus avancées, telles que les réseaux de neurones convolutifs (CNN), qui sont au cœur des systèmes de vision par ordinateur modernes.

#### **Sources des citations**

1. transcription.txt
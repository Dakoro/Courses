# La Programmation Dynamique : Des Principes Fondamentaux à l'Analyse Syntaxique

-----

## **Partie I : Les Fondements de la Programmation Dynamique**

-----

### **Chapitre 1 : Introduction et Principe d'Optimalité**

[cite\_start]La **programmation dynamique** est un paradigme de conception d'algorithmes puissant et fondamental en informatique, permettant de résoudre des problèmes d'optimisation complexes en les décomposant en sous-problèmes plus simples[cite: 1]. [cite\_start]Avant de plonger dans les aspects techniques et les implémentations, il est essentiel de saisir l'intuition qui sous-tend cette approche[cite: 2].

#### **1.1. Motivation : Le Problème du Plus Court Chemin**

[cite\_start]Imaginons une tâche quotidienne : trouver le chemin le plus rapide pour se rendre de son domicile à son lieu de travail ou d'étude[cite: 3]. [cite\_start]Ce trajet peut être modélisé comme un graphe où les intersections sont des nœuds et les rues sont des arêtes pondérées par le temps de parcours[cite: 4]. [cite\_start]Supposons que nous ayons identifié le chemin globalement optimal, c'est-à-dire celui qui minimise le temps total de trajet[cite: 5]. [cite\_start]Appelons ce chemin la trajectoire $S \\to E$, où $S$ est le point de départ (Start) et $E$ le point d'arrivée (End)[cite: 6].

Considérons maintenant un point intermédiaire $M$ (Middle) sur cette trajectoire optimale. [cite\_start]Une question fondamentale se pose : le segment de chemin de $S$ à $M$ est-il lui-même le chemin optimal pour aller de $S$ à $M$? [cite: 7] La réponse est affirmative. [cite\_start]Si un chemin plus rapide existait pour aller de $S$ à $M$, alors on pourrait remplacer le segment initial de notre trajectoire globale par ce nouveau chemin plus court, ce qui contredirait le fait que la trajectoire $S \\to E$ était initialement optimale[cite: 8].

Cette propriété simple mais profonde est au cœur de la programmation dynamique. [cite\_start]Elle stipule que toute sous-trajectoire d'une trajectoire optimale est elle-même optimale[cite: 9].

[cite\_start]Cependant, il est crucial de noter que l'inverse n'est pas vrai[cite: 9]. [cite\_start]La concaténation de deux trajectoires optimales ne produit pas nécessairement une trajectoire globale optimale[cite: 10]. [cite\_start]Par exemple, si l'on connaît le chemin optimal de $S$ à $M$ et le chemin optimal de $M$ à $E$, leur combinaison n'est pas garantie d'être le chemin optimal de $S$ à $E$[cite: 11]. Un chemin complètement différent, passant par un autre point intermédiaire, pourrait s'avérer plus court. [cite\_start]Cette asymétrie justifie la nécessité d'une approche globale et structurée, capable de considérer l'ensemble du problème pour garantir l'optimalité, plutôt qu'une série de décisions locales[cite: 12].

#### **1.2. Le Contexte Historique : Richard Bellman et la RAND Corporation**

[cite\_start]La programmation dynamique a été formellement développée dans les années 1950 par le mathématicien américain **Richard Bellman**[cite: 13]. [cite\_start]À cette époque, Bellman travaillait pour la RAND Corporation, un think tank qui se penchait sur des problèmes stratégiques pour le gouvernement américain[cite: 14]. [cite\_start]Une anecdote célèbre raconte que la méthode a émergé de recherches visant à optimiser la logistique militaire, notamment le positionnement de bases aériennes américaines autour de l'Union Soviétique pendant la Guerre Froide[cite: 15]. [cite\_start]Le terme "programmation" dans "programmation dynamique" ne fait pas référence à l'écriture de code informatique, mais plutôt à la planification et à l'établissement d'un programme d'actions, dans un sens militaire ou logistique[cite: 16].

[cite\_start]Les travaux fondateurs de Bellman ont été publiés dans des rapports et articles séminals, notamment "The Theory of Dynamic Programming" (1954) et "Dynamic Programming of Continuous Processes" (1954), qui ont jeté les bases théoriques de ce nouveau champ[cite: 16]. [cite\_start]Ces publications ont marqué le début d'une nouvelle ère dans l'analyse et l'optimisation des systèmes à grande échelle, avec des applications allant de la conception de systèmes de guidage pour les véhicules spatiaux à la gestion de cliniques médicales[cite: 16, 17].

#### **1.3. Formalisation : Processus de Décision Multi-étapes**

[cite\_start]Au cœur de la théorie de Bellman se trouve le concept de **processus de décision multi-étapes** (multi-stage decision process)[cite: 17]. Un tel processus peut être décrit par les composants suivants :

  * [cite\_start]**État ($x\_t$)**: Un ensemble de paramètres qui décrivent complètement l'état du système à un instant ou une étape $t$[cite: 17]. [cite\_start]Par exemple, dans le problème du chemin, l'état pourrait être notre position actuelle[cite: 18]. [cite\_start]Dans un problème financier, ce pourrait être notre capital[cite: 19].
  * [cite\_start]**Action ou Décision ($a\_t$)**: Un choix que nous effectuons à l'étape $t$, qui influence l'évolution du système[cite: 19]. [cite\_start]L'ensemble des actions possibles peut dépendre de l'état actuel $x\_t$[cite: 20].
  * [cite\_start]**Fonction de Transition ($T$)**: Une règle qui détermine le nouvel état $x\_{t+1}$ en fonction de l'état actuel $x\_t$ et de l'action choisie $a\_t$[cite: 20]. [cite\_start]Mathématiquement, cela s'écrit $x\_{t+1} = T(x\_t, a\_t)$[cite: 21].
  * [cite\_start]**Récompense ou Coût ($f(x\_t, a\_t)$)**: Une valeur numérique (un gain ou une perte) associée à la prise de la décision $a\_t$ dans l'état $x\_t$[cite: 21]. [cite\_start]L'objectif global du processus est de maximiser (ou minimiser) la somme de ces récompenses sur l'ensemble des étapes[cite: 22].

#### **1.4. Le Principe d'Optimalité de Bellman**

[cite\_start]Le **Principe d'Optimalité**, déjà introduit intuitivement, peut être énoncé plus formellement[cite: 23]. Bellman lui-même l'a formulé ainsi :

> [cite\_start]"Une politique optimale a la propriété que, quels que soient l'état initial et la décision initiale, les décisions restantes doivent constituer une politique optimale par rapport à l'état résultant de la première décision." [cite: 24]

[cite\_start]Ce principe est la condition nécessaire pour qu'une solution soit optimale et constitue la pierre angulaire de la programmation dynamique[cite: 24]. [cite\_start]En informatique, cette propriété est plus communément appelée **sous-structure optimale**[cite: 25]. [cite\_start]Elle implique qu'un problème global peut être résolu en trouvant des solutions optimales à ses sous-problèmes[cite: 25]. [cite\_start]Cette décomposition récursive est ce qui rend la programmation dynamique si efficace[cite: 26].

[cite\_start]Il est fondamental de comprendre que la véritable avancée de Bellman n'a pas été simplement de "mémoriser les résultats" pour éviter des calculs redondants[cite: 26]. [cite\_start]La contribution majeure réside dans la reconnaissance que de vastes classes de problèmes d'optimisation peuvent être reformulées de manière récursive[cite: 27]. [cite\_start]La créativité en programmation dynamique ne consiste pas à écrire des boucles pour remplir un tableau, mais à modéliser un problème en définissant correctement les notions d' "état" et de "transition" de manière à ce qu'elles respectent le Principe d'Optimalité[cite: 28]. [cite\_start]Comme nous le verrons, le problème du rendu de monnaie utilise un état unidimensionnel, celui du sac à dos un état bidimensionnel, et le problème du voyageur de commerce un état dont la taille est exponentielle[cite: 29]. [cite\_start]La difficulté et l'élégance de la programmation dynamique résident dans cet art de la modélisation, qui précède et guide toute implémentation[cite: 30].

-----

### **Chapitre 2 : L'Équation Fonctionnelle de Bellman**

[cite\_start]L'**équation de Bellman** est la traduction mathématique du Principe d'Optimalité[cite: 30]. [cite\_start]Elle fournit une formulation récursive pour la valeur d'un problème d'optimisation, reliant la valeur d'un état à la valeur des états futurs[cite: 31].

#### **2.1. Dérivation de l'Équation**

Considérons un processus de décision multi-étapes. [cite\_start]Une approche naïve, souvent qualifiée de **gloutonne** (greedy), consisterait à chaque étape à choisir l'action qui maximise la récompense immédiate, $f(x\_t, a\_t)$, sans se soucier des conséquences à long terme[cite: 32]. [cite\_start]Comme nous l'avons vu, cette stratégie locale ne garantit pas une solution globale optimale[cite: 33].

[cite\_start]Bellman a proposé une approche différente : maximiser la somme intégrale des récompenses sur l'ensemble des étapes futures[cite: 33]. [cite\_start]Pour ce faire, il a introduit la **fonction de valeur** (value function), notée $V(x)$, qui représente la récompense totale maximale que l'on peut espérer obtenir en partant de l'état $x$[cite: 34].

[cite\_start]En appliquant le Principe d'Optimalité, on peut décomposer le problème[cite: 34]. [cite\_start]Pour un état $x\_t$, la décision optimale $a\_t$ est celle qui maximise la somme de la récompense immédiate $f(x\_t, a\_t)$ et de la valeur optimale du prochain état $x\_{t+1} = T(x\_t, a\_t)$[cite: 35]. [cite\_start]Cela nous conduit directement à l'équation fonctionnelle de Bellman[cite: 36]:

$$V(x_t) = \max_{a_t \in \text{Actions}(x_t)} \left( f(x_t, a_t) + \beta V(T(x_t, a_t)) \right)$$

où :

  * $V(x\_t)$ est la fonction de valeur pour l'état $x\_t$.
  * $\\max\_{a\_t}$ indique que l'on choisit l'action $a\_t$ qui maximise l'expression.
  * $f(x\_t, a\_t)$ est la récompense immédiate.
  * $\\beta$ est le **facteur d'actualisation** (discount factor), un nombre compris entre 0 et 1. Il modélise l'idée que les récompenses futures ont moins de valeur que les récompenses immédiates. [cite\_start]Pour de nombreux problèmes algorithmiques classiques, on considère $\\beta=1$[cite: 37].
  * $V(T(x\_t, a\_t))$ est la valeur de l'état suivant, $V(x\_{t+1})$.

[cite\_start]Cette équation est récursive car la fonction $V$ apparaît des deux côtés[cite: 37]. [cite\_start]La résoudre signifie trouver la fonction $V$ qui satisfait cette relation pour tous les états possibles[cite: 38].

#### **2.2. Maximisation vs. Minimisation**

[cite\_start]L'équation de Bellman est souvent présentée avec un opérateur de maximisation, car elle a été initialement développée dans des contextes de maximisation de profit ou d'utilité[cite: 39]. [cite\_start]Cependant, de nombreux problèmes d'optimisation, comme la recherche du plus court chemin ou du nombre minimal de pièces, sont des problèmes de minimisation[cite: 40].

[cite\_start]La transition entre les deux est triviale[cite: 40]. [cite\_start]Minimiser une fonction $g(x)$ est équivalent à maximiser son opposé, $-g(x)$[cite: 41]. Ainsi, la relation fondamentale est :

$$\min(g(x)) = - \max(-g(x))$$

[cite\_start]Par conséquent, nous pouvons formuler une version de l'équation de Bellman pour la minimisation[cite: 42]:

$$V(x_t) = \min_{a_t \in \text{Actions}(x_t)} \left( c(x_t, a_t) + \beta V(T(x_t, a_t)) \right)$$

[cite\_start]Ici, $c(x\_t, a\_t)$ représente un coût immédiat que l'on cherche à minimiser[cite: 42]. [cite\_start]Dans la suite de ce cours, nous passerons librement de la maximisation à la minimisation en fonction du problème étudié[cite: 43].

#### **2.3. Stratégies de Résolution : "Top-Down" vs. "Bottom-Up"**

[cite\_start]Une fois qu'un problème a été modélisé par une équation de Bellman, il existe deux stratégies principales pour la résoudre et calculer la fonction de valeur[cite: 44].

  * **Approche "Top-Down" avec Mémoïsation (Memoization)**
    [cite\_start]Cette approche suit directement la nature récursive de l'équation[cite: 45]. [cite\_start]On écrit une fonction qui calcule $V(x)$ en s'appelant récursivement pour les sous-problèmes[cite: 45]. [cite\_start]Pour éviter de recalculer plusieurs fois la solution pour un même état, on utilise une table (souvent une table de hachage ou un tableau) pour stocker les résultats[cite: 46]. [cite\_start]Lorsqu'on appelle la fonction pour un état $x$, on vérifie d'abord si $V(x)$ est déjà dans la table[cite: 47]. Si oui, on retourne la valeur stockée. [cite\_start]Sinon, on la calcule, on la stocke dans la table, puis on la retourne[cite: 48]. [cite\_start]Cette technique est particulièrement intuitive lorsque la structure du problème est naturellement récursive[cite: 49].

  * **Approche "Bottom-Up" avec Tabulation (Tabulation)**
    [cite\_start]Cette approche est itérative[cite: 49]. [cite\_start]On commence par résoudre les plus petits sous-problèmes possibles (les cas de base de la récurrence)[cite: 50]. [cite\_start]On stocke leurs solutions dans une table (généralement un tableau)[cite: 51]. [cite\_start]Ensuite, on utilise ces solutions pour construire itérativement les solutions de sous-problèmes de plus en plus grands, jusqu'à ce que l'on ait résolu le problème initial[cite: 52]. [cite\_start]L'ordre de calcul est crucial : il faut s'assurer que lorsqu'on calcule la solution pour un état, les solutions de tous les sous-problèmes dont il dépend ont déjà été calculées[cite: 53].

[cite\_start]Il est essentiel de corriger une idée fausse très répandue : la programmation dynamique n'est pas la mémoïsation[cite: 53]. [cite\_start]La programmation dynamique est un paradigme de modélisation basé sur le Principe d'Optimalité et l'équation de Bellman[cite: 55]. [cite\_start]La mémoïsation (top-down) et la tabulation (bottom-up) sont des techniques d'implémentation pour résoudre efficacement la récurrence qui en découle[cite: 56]. [cite\_start]La mémoïsation peut être appliquée à n'importe quelle fonction récursive avec des appels redondants (comme le calcul des nombres de Fibonacci), qu'elle provienne ou non d'un problème d'optimisation[cite: 57]. [cite\_start]La programmation dynamique, elle, est spécifiquement liée à la structure de sous-problèmes optimaux[cite: 58]. [cite\_start]Cette distinction entre le "quoi" (le paradigme) et le "comment" (l'implémentation) est fondamentale pour une compréhension approfondie du sujet[cite: 59].

-----

## **Partie II : Algorithmes Canoniques en Programmation Dynamique**

-----

### **Chapitre 3 : Le Problème du Rendu de Monnaie (Change-Making Problem)**

[cite\_start]Le problème du rendu de monnaie est un exemple classique et accessible pour introduire les mécanismes de la programmation dynamique[cite: 59]. [cite\_start]Il illustre parfaitement comment une approche gloutonne peut échouer et comment une solution structurée garantit l'optimalité[cite: 60].

#### **3.1. Définition du Problème**

Étant donné un système de pièces avec un ensemble de dénominations $C = {c\_1, c\_2, ..., c\_k}$ et une somme totale $N$, le problème consiste à trouver le nombre minimum de pièces nécessaires pour constituer exactement la somme $N$. [cite\_start]On suppose que nous disposons d'un nombre illimité de pièces de chaque dénomination[cite: 61].

Prenons l'exemple où nous devons rendre une somme de 10 en utilisant des pièces de dénominations ${1, 3, 4}$.

  * **Approche gloutonne** : On prend la plus grande pièce possible à chaque étape. [cite\_start]On prend une pièce de 4 (reste 6), puis une autre de 4 (reste 2), puis deux pièces de 1 (reste 0)[cite: 62]. [cite\_start]Total : $4+4+1+1$, soit 4 pièces[cite: 63].
  * **Solution optimale** : On peut faire mieux. Par exemple, $4+3+3$ donne un total de 3 pièces. [cite\_start]C'est la solution optimale[cite: 64].

Cet exemple démontre clairement que la stratégie gloutonne, qui fait un choix localement optimal, ne mène pas nécessairement à la solution globalement optimale.

#### **3.2. Formulation en Programmation Dynamique**

[cite\_start]Pour résoudre ce problème avec la programmation dynamique, nous devons définir l'état et la récurrence[cite: 65].

  * [cite\_start]**État**: L'état du problème peut être défini par un seul paramètre : la somme restante à rendre[cite: 65]. [cite\_start]Nous définissons donc notre fonction de valeur $V(n)$ comme le nombre minimum de pièces requises pour rendre la somme $n$[cite: 66].
  * [cite\_start]**Équation de Bellman (Récurrence)**: Pour calculer $V(n)$, nous considérons chaque pièce $c\_i$ de notre système[cite: 66]. [cite\_start]Si nous choisissons d'utiliser une pièce $c\_i$ (où $c\_i \\le n$), il nous restera à rendre la somme $n-c\_i$[cite: 67]. [cite\_start]Le nombre de pièces pour cette solution sera $1 + V(n-c\_i)$[cite: 68]. [cite\_start]Puisque nous voulons le nombre minimum de pièces, nous devons choisir la pièce $c\_i$ qui minimise cette quantité[cite: 68]. L'équation de récurrence est donc :
    $$
    $$$$V(n) = 1 + \\min\_{c\_i \\in C, c\_i \\le n} {V(n - c\_i)}
    $$
    $$$$
    $$
  * **Cas de base**: Pour rendre une somme de 0, il faut 0 pièce. [cite\_start]Donc, $V(0) = 0$[cite: 70].

#### **3.3. Pseudo-code et Implémentation**

Nous allons utiliser l'approche par **tabulation** (bottom-up) pour résoudre cette récurrence.

##### Pseudo-code

```pseudocode
fonction solveChangeMaking(pièces, N):
  // V[n] stockera le nombre minimum de pièces pour la somme n
  V = tableau de taille N+1

  // Initialisation
  V[0] = 0
  pour n de 1 à N:
    V[n] = infini

  // Remplissage de la table
  pour n de 1 à N:
    pour chaque pièce c dans pièces:
      si n >= c:
        V[n] = min(V[n], 1 + V[n - c])

  // Si V[N] est toujours infini, la somme ne peut pas être rendue
  si V[N] == infini:
    retourner -1 // ou une autre valeur d'erreur
  sinon:
    retourner V[N]
```

##### Tableau de Suivi : Rendu de monnaie pour 10 avec {1, 3, 4}

Ce tableau illustre le remplissage de notre tableau `V`. Chaque cellule `V[n]` montre la valeur minimale trouvée.

| n | Calculs (min sur $c \\in {1,3,4}$ de $1+V[n-c]$) | V[n] | Vient de |
|---|---|---|---|
| 0 | Cas de base | 0 | - |
| 1 | $1+V[0]$ | 1 | V[0] (pièce 1) |
| 2 | $1+V[1]$ | 2 | V[1] (pièce 1) |
| 3 | $\\min(1+V[2], 1+V[0])$ | 1 | V[0] (pièce 3) |
| 4 | $\\min(1+V[3], 1+V[1], 1+V[0])$ | 1 | V[0] (pièce 4) |
| 5 | $\\min(1+V[4], 1+V[2])$ | 2 | V[4] (pièce 1) |
| 6 | $\\min(1+V[5], 1+V[3])$ | 2 | V[3] (pièce 3) |
| 7 | $\\min(1+V[6], 1+V[4], 1+V[3])$ | 2 | V[4] ou V[3] |
| 8 | $\\min(1+V[7], 1+V[5], 1+V[4])$ | 2 | V[4] (pièce 4) |
| 9 | $\\min(1+V[8], 1+V[6])$ | 3 | V[8] ou V[6] |
| 10 | $\\min(1+V[9], 1+V[7], 1+V[6])$ | 3 | V[7] ou V[6] |

[cite\_start]Le résultat final est $V[10]=3$, confirmant que la solution optimale nécessite 3 pièces[cite: 73]. Pour reconstruire la solution, on peut suivre les pointeurs "Vient de" à rebours : $10 \\xrightarrow{\\text{pièce 3}} 7 \\xrightarrow{\\text{pièce 3}} 4 \\xrightarrow{\\text{pièce 4}} 0$. [cite\_start]La combinaison de pièces est donc {3, 3, 4}[cite: 74, 75].

##### Implémentation C (Tabulation)

```c
#include <stdio.h>
#include <stdlib.h>
#include <limits.h>

// Fonction pour trouver le minimum de deux entiers
int min(int a, int b) {
    [cite_start]return (a < b) ? a : b; [cite: 76]
}

// Fonction principale pour le rendu de monnaie
int solve_change_making(int* coins, int num_coins, int n) {
    // Allouer la table de programmation dynamique
    [cite_start]int *V = (int *)malloc(sizeof(int) * (n + 1)); [cite: 76]
    [cite_start]if (V == NULL) { [cite: 77]
        perror("Erreur d'allocation mémoire");
        [cite_start]return -1; [cite: 77]
    }

    // Initialisation
    [cite_start]V[0] = 0; [cite: 78]
    // Il est crucial d'utiliser une boucle pour l'initialisation.
    [cite_start]for (int i = 1; i <= n; i++) { [cite: 80]
        [cite_start]V[i] = INT_MAX; [cite: 80]
    }

    // Remplir la table V de manière bottom-up
    [cite_start]for (int i = 1; i <= n; i++) { [cite: 81]
        // Parcourir toutes les pièces
        for (int j = 0; j < num_coins; j++) {
            // Si la pièce peut être utilisée pour la somme i
            if (coins[j] <= i) {
                // Vérifier si le sous-problème V[i - coins[j]] a une solution
                [cite_start]if (V[i - coins[j]] != INT_MAX) { [cite: 82]
                    [cite_start]V[i] = min(V[i], 1 + V[i - coins[j]]); [cite: 82]
                }
            }
        }
    }

    [cite_start]int result = V[n]; [cite: 83]
    free(V); [cite_start]// Libérer la mémoire allouée [cite: 84]

    // Si V[n] est toujours INT_MAX, aucune solution n'a été trouvée
    return (result == INT_MAX) ? [cite_start]-1 : result; [cite: 85]
}

int main() {
    [cite_start]int coins[] = {1, 3, 4}; [cite: 85]
    [cite_start]int num_coins = sizeof(coins) / sizeof(coins[0]); [cite: 86]
    int n = 10;
    // Mettre l'appel de fonction sur une ligne séparée pour faciliter le débogage
    [cite_start]int min_coins = solve_change_making(coins, num_coins, n); [cite: 87]
    [cite_start]if (min_coins == -1) { [cite: 88]
        [cite_start]printf("Il n'est pas possible de rendre la monnaie pour la somme %d.\n", n); [cite: 88]
    } else {
        [cite_start]printf("Le nombre minimum de pièces pour rendre %d est : %d\n", n, min_coins); [cite: 89]
    }

    [cite_start]return 0; [cite: 90]
}
```

#### **3.4. Point Technique en C : Initialisation et Optimisation**

[cite\_start]Comme mentionné dans le code, l'initialisation du tableau de DP est un point délicat[cite: 104]. [cite\_start]La fonction `memset` de la bibliothèque standard C opère octet par octet[cite: 105]. [cite\_start]Tenter d'initialiser un tableau d'entiers (`int`, typiquement 4 octets) avec `INT_MAX` (par exemple, `0x7FFFFFFF`) via `memset` ne fonctionnera pas comme prévu[cite: 106]. [cite\_start]`memset` prendrait la valeur `0x7F` et remplirait chaque octet du tableau avec cette valeur, résultant en des entiers valant `0x7F7F7F7F`, ce qui n'est pas `INT_MAX`[cite: 107]. [cite\_start]La méthode correcte et sûre est d'utiliser une boucle `for` pour initialiser chaque élément du tableau[cite: 108]. [cite\_start]Les compilateurs modernes sont très efficaces pour optimiser de telles boucles[cite: 109].

#### **3.5. Complexité et Contexte Académique**

[cite\_start]L'algorithme de programmation dynamique présenté a une complexité temporelle de $O(N \\times k)$, où $N$ est la somme à rendre et $k$ est le nombre de dénominations de pièces[cite: 110]. [cite\_start]La complexité spatiale est de $O(N)$ pour stocker la table de DP[cite: 111]. C'est une amélioration considérable par rapport à une approche récursive naïve qui aurait une complexité exponentielle.

[cite\_start]Il est intéressant de noter que des articles récents ont montré qu'il est possible d'améliorer la complexité en $O(N \\cdot \\text{polylog}(N))$ en utilisant des techniques avancées comme la convolution via la Transformée de Fourier Rapide (FFT)[cite: 112].

-----

### **Chapitre 4 : Le Problème du Sac à Dos 0/1 (0/1 Knapsack Problem)**

[cite\_start]Le problème du sac à dos est un autre pilier de l'enseignement de la programmation dynamique[cite: 112]. [cite\_start]Il introduit un espace d'états à deux dimensions et illustre le concept de complexité **pseudo-polynomiale**[cite: 113].

#### **4.1. Définition du Problème**

Le problème du sac à dos 0/1 (0-1 Knapsack Problem) peut être énoncé comme suit :
[cite\_start]Étant donné un ensemble de $n$ objets, où chaque objet $i$ a un poids $w\_i$ et une valeur $v\_i$, et un sac à dos avec une capacité de poids maximale $W$, quel sous-ensemble d'objets faut-il choisir pour maximiser la valeur totale, sans que la somme des poids des objets choisis ne dépasse la capacité $W$? [cite: 114]

[cite\_start]La contrainte "0/1" signifie que pour chaque objet, nous avons une décision binaire : soit nous le prenons entièrement (1), soit nous ne le prenons pas du tout (0)[cite: 115]. [cite\_start]Il n'est pas possible de prendre une fraction d'un objet[cite: 116]. [cite\_start]Ce problème est connu pour être **NP-complet**[cite: 116, 117].

#### **4.2. Formulation en Programmation Dynamique**

[cite\_start]L'échec d'une approche gloutonne (par exemple, prendre les objets avec le meilleur ratio valeur/poids) peut être facilement démontré[cite: 117]. [cite\_start]La solution optimale nécessite une approche plus globale[cite: 118].

  * [cite\_start]**État**: Pour capturer l'information nécessaire, notre état doit contenir deux paramètres : l'objet que nous considérons et la capacité restante[cite: 118]. [cite\_start]Nous définissons la fonction de valeur $V(i, w)$ comme la valeur maximale que l'on peut obtenir en utilisant un sous-ensemble des $i$ premiers objets avec une capacité de sac de $w$[cite: 119].
  * **Équation de Bellman (Récurrence)**: Pour chaque objet $i$, nous avons deux choix :
    1.  [cite\_start]**Ne pas prendre l'objet i**: La valeur est celle obtenue avec les $i-1$ objets précédents, soit $V(i-1, w)$[cite: 120].
    2.  [cite\_start]**Prendre l'objet i**: Possible seulement si $w\_i \\le w$[cite: 121]. [cite\_start]La valeur est sa propre valeur plus la valeur optimale pour le reste : $v\_i + V(i-1, w-w\_i)$[cite: 122].

La décision optimale prend le maximum de ces deux options. [cite\_start]L'équation de récurrence est donc[cite: 123]:

$$
V(i, w) =
\begin{cases}
V(i-1, w) & \text{si } w_i > w \\
\max(V(i-1, w), v_i + V(i-1, w-w_i)) & \text{si } w_i \le w
\end{cases}
$$  * [cite\_start]**Cas de base**: Si nous n'avons aucun objet ($i=0$) ou si la capacité du sac est nulle ($w=0$), la valeur maximale est 0. Donc, $V(0, w) = 0$ et $V(i, 0) = 0$[cite: 123].

#### **4.3. Pseudo-code et Implémentation**

##### Tableau de Suivi : Exemple du Sac à Dos

Considérons $W=5$, et 3 objets :

* Objet 1 : $w\_1=2, v\_1=60$
* Objet 2 : $w\_2=3, v\_2=100$
* Objet 3 : $w\_3=5, v\_3=120$

La table $V[i][w]$ serait remplie comme suit :

| i \\ w | 0 | 1 | 2 | 3 | 4 | 5 |
|---|---|---|---|---|---|---|
| **0** | 0 | 0 | 0 | 0 | 0 | 0 |
| **1** ($w=2,v=60$) | 0 | 0 | 60 | 60 | 60 | 60 |
| **2** ($w=3,v=100$) | 0 | 0 | 60 | 100 | 100 | **160** |
| **3** ($w=5,v=120$) | 0 | 0 | 60 | 100 | 100 | 160 |

La valeur maximale est 160. En analysant la cellule $V[3][5]$, on voit que sa valeur vient de $V[2][5]$ (car $\\max(V(2,5), 120+V(2,0)) = \\max(160, 120) = 160$), ce qui signifie qu'on ne prend pas l'objet 3. En analysant $V[2][5]$, on voit que sa valeur vient de $v\_2 + V(1, 5-3) = 100 + 60 = 160$, signifiant qu'on prend l'objet 2. Enfin, $V[1][2]$ vient de la prise de l'objet 1. La solution est donc de prendre les objets 1 et 2.

##### Implémentation C (Tabulation)

```c
#include <stdio.h>
#include <stdlib.h>

int max(int a, int b) {
[cite_start]return (a > b) ? a : b; [cite: 128]
}

int knapsack(int W, int* wt, int* val, int n) {
// Allouer la table de DP
[cite_start]int **V = (int **)malloc((n + 1) * sizeof(int *)); [cite: 128]
for (int i = 0; i <= n; i++) {
[cite_start]V[i] = (int *)malloc((W + 1) * sizeof(int)); [cite: 129]
}

// Remplir la table de manière bottom-up
[cite_start]for (int i = 0; i <= n; i++) { [cite: 130]
for (int w = 0; w <= W; w++) {
if (i == 0 || w == 0) {
[cite_start]V[i][w] = 0; [cite: 131]
} else if (wt[i - 1] <= w) {
[cite_start]// On peut prendre l'objet i-1. [cite: 131]
[cite_start]int val_avec = val[i - 1] + V[i - 1][w - wt[i - 1]]; [cite: 133]
int val_sans = V[i - 1][w];
[cite_start]V[i][w] = max(val_avec, val_sans); [cite: 134]
} else {
[cite_start]// L'objet i-1 est trop lourd. [cite: 134]
[cite_start]V[i][w] = V[i - 1][w]; [cite: 135]
}
}
}

[cite_start]int result = V[n][W]; [cite: 136]
// Libérer la mémoire
for (int i = 0; i <= n; i++) {
[cite_start]free(V[i]); [cite: 136]
}
[cite_start]free(V); [cite: 137]

return result;
}

int main() {
[cite_start]int val[] = {60, 100, 120}; [cite: 138]
[cite_start]int wt[] = {2, 3, 5}; [cite: 138]
int W = 5;
[cite_start]int n = sizeof(val) / sizeof(val[0]); [cite: 138]
[cite_start]printf("La valeur maximale dans le sac à dos est : %d\n", knapsack(W, wt, val, n)); [cite: 139]

[cite_start]return 0; [cite: 140]
}
```

##### Optimisation de l'Espace

[cite\_start]On peut remarquer que pour calculer la ligne $i$, nous n'avons besoin que de la ligne $i-1$[cite: 140]. [cite\_start]On peut donc réduire la complexité spatiale de $O(N \\times W)$ à $O(W)$ en utilisant une seule ligne et en parcourant les capacités en ordre décroissant[cite: 142].

```c
// Version optimisée en espace
int knapsack_optimized(int W, int* wt, int* val, int n) {
[cite_start]int *dp = (int *)calloc(W + 1, sizeof(int)); [cite: 142]
if (dp == NULL) return -1; // Allocation check

for (int i = 0; i < n; i++) {
// Parcourir les poids en ordre décroissant
for (int w = W; w >= wt[i]; w--) {
[cite_start]dp[w] = max(dp[w], val[i] + dp[w - wt[i]]); [cite: 143]
}
}

[cite_start]int result = dp[W]; [cite: 144]
free(dp);
return result;
}
```

#### **4.4. Complexité et Contexte Académique**

[cite\_start]La complexité temporelle de cet algorithme est de $O(N \\times W)$, et sa complexité spatiale est de $O(N \\times W)$, ou $O(W)$ avec l'optimisation[cite: 145]. Il est crucial de comprendre que cette complexité est **pseudo-polynomiale**. [cite\_start]L'algorithme est polynomial par rapport à la valeur numérique de $W$, mais si l'on considère la taille de l'entrée (le nombre de bits nécessaires pour représenter $W$, qui est $\\log W$), la complexité est exponentielle[cite: 146]. [cite\_start]C'est une caractéristique des solutions par programmation dynamique aux problèmes NP-complets[cite: 147].

-----

### **Chapitre 5 : La Distance d'Édition (Distance de Levenshtein)**

[cite\_start]La distance d'édition, ou distance de Levenshtein, est une mesure de la similarité entre deux chaînes de caractères[cite: 148]. [cite\_start]Son calcul est une application canonique de la programmation dynamique[cite: 148].

#### **5.1. Définition du Problème**

[cite\_start]Nommée d'après Vladimir Levenshtein (1965), la distance de Levenshtein entre deux chaînes est le nombre minimum d'opérations mono-caractère pour transformer une chaîne en l'autre[cite: 149]. Les opérations autorisées sont :

* [cite\_start]**Insertion**: Ajouter un caractère (Coût: 1)[cite: 150].
* [cite\_start]**Suppression**: Retirer un caractère (Coût: 1)[cite: 151].
* [cite\_start]**Substitution**: Remplacer un caractère par un autre (Coût: 1. Note: le texte source mentionne un coût de 2, mais un coût de 1 est la norme et est plus cohérent avec les exemples. Une substitution peut être vue comme une suppression + une insertion, d'où le coût de 2 parfois utilisé, mais nous utiliserons 1 pour la cohérence)[cite: 152].

[cite\_start]Par exemple, la distance entre "spoon" et "sponge" est de 3[cite: 153].
`spoon` -\> `spon` (suppression 'o') -\> `spong` (insertion 'g') -\> `sponge` (insertion 'e'). Coût total : $1+1+1 = 3$.

#### **5.2. Formulation en Programmation Dynamique**

* **État**: $V(i,j)$ est la distance de Levenshtein minimale entre le préfixe de longueur $i$ de la chaîne $s1$ et le préfixe de longueur $j$ de la chaîne $s2$.
* **Équation de Bellman (Récurrence)**: Pour calculer $V(i,j)$, nous considérons le minimum de trois opérations possibles :
1.  **Suppression**: Transformer $s1[1..i-1]$ en $s2[1..j]$ puis supprimer $s1[i]$. [cite\_start]Coût : $V(i-1,j) + 1$[cite: 154, 155].
2.  **Insertion**: Transformer $s1[1..i]$ en $s2[1..j-1]$ puis insérer $s2[j]$. [cite\_start]Coût : $V(i,j-1) + 1$[cite: 155, 156].
3.  **Substitution/Correspondance**: Transformer $s1[1..i-1]$ en $s2[1..j-1]$. Coût : $V(i-1,j-1) + \\text{cost}(s1[i], s2[j])$. [cite\_start]Le coût est 0 si les caractères sont identiques, sinon 1[cite: 157].

[cite\_start]L'équation de récurrence est[cite: 157]:

$$V(i,j) = \\min
\\begin{cases}
V(i-1, j) + 1 & \\text{(Suppression)} \\
V(i, j-1) + 1 & \\text{(Insertion)} \\
V(i-1, j-1) + \\text{cost}(s1\_{i-1}, s2\_{j-1}) & \\text{(Substitution/Match)}
\\end{cases}
$$  \* [cite\_start]**Cas de base**: La distance entre une chaîne de longueur $i$ et une chaîne vide est $i$[cite: 158]. [cite\_start]Donc, $V(i,0)=i$ et $V(0,j)=j$[cite: 159].

#### **5.3. Implémentation C**

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int min3(int a, int b, int c) {
    int m = a;
    [cite_start]if (b < m) m = b; [cite: 164]
    [cite_start]if (c < m) m = c; [cite: 164]
    return m;
}

int levenshtein_distance(const char *s1, const char *s2) {
    [cite_start]int m = strlen(s1); [cite: 165]
    int n = strlen(s2);
    // Allouer la matrice de DP
    [cite_start]int **V = (int **)malloc((m + 1) * sizeof(int *)); [cite: 166]
    for (int i = 0; i <= m; i++) {
        [cite_start]V[i] = (int *)malloc((n + 1) * sizeof(int)); [cite: 167]
    }

    // Initialisation
    for (int i = 0; i <= m; i++) V[i][0] = i; [cite_start]// Coût des suppressions [cite: 168, 169]
    for (int j = 0; j <= n; j++) V[0][j] = j; [cite_start]// Coût des insertions [cite: 170]

    // Remplissage de la matrice
    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            int cost = (s1[i - 1] == s2[j - 1]) ? 0 : 1; // Coût de substitution = 1
            V[i][j] = min3(V[i - 1][j] + 1,          // Suppression
                           V[i][j - 1] + 1,          // Insertion
                           V[i - 1][j - 1] + cost); [cite_start]// Substitution/Match [cite: 172]
        }
    }

    [cite_start]int distance = V[m][n]; [cite: 173]
    // Libérer la mémoire
    for (int i = 0; i <= m; i++) {
        [cite_start]free(V[i]); [cite: 174]
    }
    [cite_start]free(V); [cite: 175]

    return distance;
}

int main() {
    [cite_start]const char *s1 = "niche"; [cite: 176]
    [cite_start]const char *s2 = "chien"; [cite: 176]

    [cite_start]int dist = levenshtein_distance(s1, s2); [cite: 177]
    [cite_start]printf("La distance de Levenshtein entre '%s' et '%s' est : %d\n", s1, s2, dist); [cite: 177]

    [cite_start]return 0; [cite: 178]
}
```

#### **5.4. Complexité et Contexte Académique**

[cite\_start]La complexité temporelle et spatiale de l'algorithme est de $O(m \\times n)$, où $m$ et $n$ sont les longueurs des chaînes[cite: 179]. [cite\_start]Les applications de la distance d'édition sont vastes[cite: 181]:

  * **Bio-informatique**: Alignement de séquences d'ADN.
  * **Traitement du Langage Naturel (NLP)**: Correction orthographique, recherche approximative.
  * **Systèmes de fichiers**: Commandes de comparaison comme `diff`.

-----

## **Partie III : Applications Avancées et Limites de la Méthode**

-----

### **Chapitre 7 : L'Algorithme de Cocke-Younger-Kasami (CYK)**

[cite\_start]L'algorithme CYK est une application de la programmation dynamique à l'**analyse syntaxique** pour les grammaires non contextuelles[cite: 193].

#### **7.1. L'Analyse Syntaxique comme Problème de Programmation Dynamique**

[cite\_start]Le problème de la reconnaissance (parsing) consiste à déterminer si une chaîne $w$ peut être générée par une grammaire $G$[cite: 195]. [cite\_start]L'algorithme CYK utilise une approche **bottom-up**, partant de la chaîne $w$ et essayant de la réduire au symbole de départ $S$[cite: 196]. [cite\_start]Il requiert que la grammaire soit en **Forme Normale de Chomsky (CNF)**[cite: 197]. [cite\_start]En CNF, toutes les règles de production sont de la forme $A \\to BC$ ou $A \\to a$[cite: 189].

#### **7.2. L'Algorithme CYK**

  * **État**: L'algorithme construit une table triangulaire $T$. [cite\_start]Une cellule $T[i,j]$ contient l'ensemble des symboles non-terminaux qui peuvent générer la sous-chaîne de longueur $j$ commençant à l'indice $i$ ($w[i..i+j-1]$)[cite: 197].
  * [cite\_start]**Récurrence**: Pour remplir $T[i,j]$, l'algorithme scinde la sous-chaîne en deux parties, $w[i..i+k-1]$ et $w[i+k..i+j-1]$[cite: 198]. [cite\_start]Un non-terminal $A$ est ajouté à $T[i,j]$ s'il existe une règle $A \\to BC$ telle que $B \\in T[i,k]$ et $C \\in T[i+k,j-k]$[cite: 199].
    $$
    $$$$T[i,j] = \\bigcup\_{k=1}^{j-1} { A \\mid \\exists (A \\to BC) \\in R, B \\in T[i,k], C \\in T[i+k, j-k] }
    $$
    $$$$
    $$
  * [cite\_start]**Cas de base**: La première ligne (longueur 1) est remplie avec les règles terminales: $T[i,1] = { A \\mid A \\to w\_i \\in R }$[cite: 200].
  * [cite\_start]**Solution**: La chaîne est reconnue si le symbole de départ $S$ est dans la cellule supérieure $T[1,n]$[cite: 200].

#### **7.3. Complexité et Applications**

[cite\_start]La complexité temporelle de CYK est $O(n^3 \\cdot |G|)$, où $n$ est la longueur de la chaîne et $|G|$ la taille de la grammaire[cite: 229]. [cite\_start]Sa complexité cubique le rend moins pratique pour la compilation, où des analyseurs linéaires sont préférés[cite: 230]. [cite\_start]Cependant, il reste fondamental et est utilisé en traitement du langage naturel[cite: 231].

-----

### **Chapitre 8 : Les Limites de la Programmation Dynamique : Le Problème du Voyageur de Commerce (TSP)**

[cite\_start]La programmation dynamique n'est pas une solution miracle, comme l'illustre le Problème du Voyageur de Commerce (TSP)[cite: 232].

#### **8.1. L'Algorithme de Held-Karp**

[cite\_start]Le TSP demande le plus court chemin qui visite chaque ville exactement une fois et retourne au départ[cite: 234]. [cite\_start]C'est un problème **NP-difficile**[cite: 234]. [cite\_start]L'algorithme de Held-Karp (ou Bellman-Held-Karp) le résout de manière exacte via la programmation dynamique[cite: 235].

  * **État**: L'état doit mémoriser les villes déjà visitées. [cite\_start]Il est défini par une paire $(S,k)$, où $S$ est un ensemble de villes visitées et $k$ est la dernière ville visitée[cite: 237]. [cite\_start]$V(S,k)$ est le coût du plus court chemin partant du départ, visitant toutes les villes de $S$, et se terminant en $k$[cite: 237].
  * [cite\_start]**Récurrence**: Pour calculer $V(S,k)$, on considère toutes les villes $m \\in S \\setminus {k}$ qui auraient pu précéder $k$[cite: 238].
    $$
    $$$$V(S,k) = \\min\_{m \\in S, m \\ne k} { V(S \\setminus {k}, m) + \\text{dist}(m,k) }
    $$
    $$$$
    $$
  * **Cas de base**: $V({1,k},k) = \\text{dist}(1,k)$.
  * **Solution finale**: Le coût du tour optimal est $\\min\_{k \\ne 1} { V({1,...,n}, k) + \\text{dist}(k,1) }$.

#### **8.2. La Malédiction de la Dimensionnalité**

[cite\_start]La complexité de l'algorithme est une conséquence de la structure du problème[cite: 240]. Pour le TSP, l'état doit inclure l'ensemble des villes visitées. Le nombre de sous-ensembles de $n$ villes est $2^n$. [cite\_start]La taille de l'espace d'états est donc exponentielle[cite: 243].

  * Complexité temporelle : $O(n^2 \\cdot 2^n)$
  * Complexité spatiale : $O(n \\cdot 2^n)$

[cite\_start]Cette complexité est un résultat direct de ce que Bellman a appelé la **malédiction de la dimensionnalité**[cite: 244]. [cite\_start]Lorsque le nombre de dimensions de l'état augmente, le volume de l'espace d'états croît de manière exponentielle[cite: 245]. [cite\_start]La programmation dynamique ne "résout" donc pas magiquement les problèmes NP-difficiles en temps polynomial; elle reflète leur complexité intrinsèque[cite: 246].

[cite\_start]Néanmoins, cette approche est une amélioration considérable par rapport à la force brute ($O(n\!)$), qui est inutilisable même pour un petit nombre de villes[cite: 248, 249].

-----

## **Partie IV : Synthèse et Lectures Recommandées**

-----

### **Chapitre 9 : Conclusion**

[cite\_start]Ce cours a exploré la programmation dynamique comme une approche de modélisation pour résoudre des problèmes d'optimisation complexes[cite: 251, 252].

#### **9.1. Synthèse des Concepts**

Les deux piliers de la programmation dynamique sont :

1.  [cite\_start]**Le Principe d'Optimalité de Bellman**: Une solution optimale globale est composée de solutions optimales à ses sous-problèmes (sous-structure optimale)[cite: 253].
2.  [cite\_start]**L'Équation Fonctionnelle de Bellman**: La formulation récursive de la valeur d'une solution optimale[cite: 253].

La démarche générale comporte quatre étapes :

1.  [cite\_start]Caractériser la structure d'une solution optimale[cite: 253].
2.  [cite\_start]Définir récursivement la valeur d'une solution optimale[cite: 253].
3.  [cite\_start]Calculer la valeur (bottom-up ou top-down)[cite: 254].
4.  [cite\_start]Construire une solution optimale à partir des résultats calculés[cite: 254].

#### **9.2. Tableau Récapitulatif des Algorithmes**

| Problème | Définition de l'État V(...) | Équation de Récurrence (simplifiée) | Complexité Temporelle |
|---|---|---|---|
| **Rendu de Monnaie** | [cite\_start]$V(n)$: coût min. pour somme $n$ [cite: 256] | [cite\_start]$1 + \\min(V(n-c))$ [cite: 256] | [cite\_start]$O(N \\times k)$ [cite: 256] |
| **Sac à Dos 0/1** | $V(i,w)$: val. max. avec $i$ objets, cap. [cite\_start]$w$ [cite: 256] | [cite\_start]$\\max(V(i-1,w), v\_i + V(i-1,w-w\_i))$ [cite: 256] | [cite\_start]$O(N \\times W)$ [cite: 256] |
| **Distance de Levenshtein** | [cite\_start]$V(i,j)$: dist. min. entre $s1[:i]$ et $s2[:j]$ [cite: 257] | [cite\_start]$\\min(V(i-1,j)+1, V(i,j-1)+1, \\dots)$ [cite: 257] | [cite\_start]$O(m \\times n)$ [cite: 257] |
| **Algorithme CYK** | $T[i,j]$: non-terminaux pour $w[i:j]$ | $A$ si $A \\to BC$ et $B, C$ dans sous-tables | $O(n^3 \\cdot |G|)$ |
| **TSP (Held-Karp)** | [cite\_start]$V(S,k)$: coût min. pour visiter $S$ en finissant par $k$ [cite: 257] | [cite\_start]$\\min(V(S\\setminus{k},m) + \\text{dist}(m,k))$ [cite: 257] | [cite\_start]$O(n^2 \\cdot 2^n)$ [cite: 257] |

#### **9.3. Lectures Recommandées**

##### Articles Fondateurs

  * Bellman, R. (1954). *The Theory of Dynamic Programming*. [cite\_start]RAND Corporation. [cite: 257]
  * Held, M., & Karp, R. M. (1962). A dynamic programming approach to sequencing problems. [cite\_start]*Journal of the Society for Industrial and Applied Mathematics*. [cite: 257]
  * Levenshtein, V. I. (1966). Binary codes capable of correcting deletions, insertions, and reversals. [cite\_start]*Soviet Physics Doklady*. [cite: 257]
  * Younger, D. H. (1967). Recognition and parsing of context-free languages in time n³. [cite\_start]*Information and Control*. [cite: 258]

##### Ouvrages de Référence

  * Cormen, T. H., Leiserson, C. E., Rivest, R. L., & Stein, C. (2009). *Introduction to Algorithms*. [cite\_start]MIT Press. [cite: 258]
  * Kernighan, B. W., & Ritchie, D. M. (1988). *The C Programming Language*. [cite\_start]Prentice Hall. [cite: 260]
  * Knuth, D. E. *The Art of Computer Programming*. [cite\_start]Addison-Wesley. [cite: 259]
  * Sedgewick, R., & Wayne, K. (2011). *Algorithms*. [cite\_start]Addison-Wesley Professional. [cite: 259]
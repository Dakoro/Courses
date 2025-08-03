# Leçon sur les Structures de Données Arborescentes : Des Fondements Théoriques à l'Implémentation Pratique

## Introduction

### Motivation

Dans le vaste univers de l'informatique, les données se présentent sous de multiples formes. Si les listes, les tableaux et les séquences linéaires constituent les briques fondamentales de nombreuses applications, une part considérable des informations que nous manipulons est intrinsèquement hiérarchique. 

Des systèmes de fichiers sur nos ordinateurs, où les répertoires contiennent des fichiers et d'autres sous-répertoires, aux organigrammes d'entreprises, en passant par les arbres généalogiques et la structure même des documents XML ou HTML, l'organisation arborescente est omniprésente. Ces structures, où les éléments sont liés par des relations de parenté et de descendance, ne peuvent être modélisées efficacement par des structures de données linéaires. 

Elles exigent une abstraction qui leur est propre : **l'arbre**. Les arbres sont des structures de données non linéaires qui excellent dans la représentation de ces relations hiérarchiques, permettant des opérations de recherche, d'insertion et de suppression bien plus efficaces que leurs homologues linéaires pour de vastes ensembles de données.

### Philosophical Underpinning

Avant de plonger dans les détails techniques, il est essentiel d'adopter une perspective fondamentale qui guidera notre exploration. En 1976, l'informaticien pionnier **Niklaus Wirth**, créateur du langage Pascal, a publié un ouvrage au titre aussi simple que profond : *Algorithms + Data Structures = Programs*³. 

Cette équation n'est pas une simple formule, mais un principe directeur en génie logiciel. Elle postule que les algorithmes et les structures de données sont deux facettes indissociables d'une même réalité :

$$\text{Algorithmes} + \text{Structures de Données} = \text{Programmes}$$

La manière dont les données sont organisées (la structure de données) influence de manière critique, voire dicte, la conception des algorithmes qui opèrent sur elles. Réciproquement, les opérations que l'on souhaite effectuer sur un ensemble de données déterminent la structure la plus appropriée pour les stocker.

Tout au long de cette leçon, ce principe sera notre fil conducteur. Nous verrons comment la définition même d'un arbre binaire de recherche impose une stratégie de recherche spécifique, et comment l'objectif de supprimer les nœuds d'un arbre nous conduit naturellement vers un algorithme de parcours particulier. Comprendre cette symbiose est la clé pour passer du statut de simple codeur à celui d'architecte de solutions logicielles robustes et efficaces.

### Plan du Cours

Notre parcours sera structuré en quatre parties distinctes, nous menant progressivement des concepts les plus fondamentaux aux sujets les plus avancés :

- **Partie I : Fondements des Arbres**. Nous établirons un vocabulaire précis en définissant formellement les arbres généraux et les arbres binaires. Nous verrons comment traduire ces concepts abstraits en une structure de données concrète en langage C.

- **Partie II : Parcours d'Arbres**. Nous explorerons les algorithmes essentiels pour visiter systématiquement chaque nœud d'un arbre. Nous commencerons par les approches récursives, intuitives, avant de les déconstruire pour aboutir à des implémentations itératives plus robustes, en construisant au passage notre propre pile.

- **Partie III : L'Arbre Binaire de Recherche (ABR)**. Nous étudierons une spécialisation extrêmement importante de l'arbre binaire qui maintient un invariant d'ordre, permettant des recherches extraordinairement rapides. Nous implémenterons toutes les opérations fondamentales : recherche, insertion et suppression.

- **Partie IV : Sujets Avancés et Représentations Alternatives**. Enfin, nous aborderons des concepts plus élégants, répondant à des questions laissées en suspens. Nous découvrirons comment représenter n'importe quel arbre général à l'aide d'un arbre binaire, et comment reconstruire un arbre unique à partir de ses séquences de parcours. Nous conclurons par une discussion sur les principes de conception de logiciels robustes.

## Partie I : Fondements des Arbres

Cette première partie a pour objectif de construire le socle de connaissances sur lequel reposera toute notre étude. Nous allons formaliser les définitions, distinguer les concepts clés et mettre en place notre première brique de code, la représentation d'un nœud d'arbre.

### Chapitre 1 : Définitions Formelles et Représentation

#### 1.1. L'Arbre : Une Structure Hiérarchique

Bien que nous ayons une compréhension intuitive de ce qu'est un arbre, une définition formelle est indispensable pour raisonner rigoureusement. En théorie des graphes, un arbre est défini comme un **graphe connexe et acyclique**⁶. 

- **"Connexe"** signifie qu'il existe un chemin entre n'importe quelle paire de nœuds.
- **"Acyclique"** signifie qu'il n'y a pas de cycle, c'est-à-dire qu'il est impossible de partir d'un nœud, de suivre une séquence d'arêtes et de revenir à ce même nœud sans jamais emprunter la même arête deux fois.

Pour les besoins de la plupart des applications informatiques, nous utilisons une définition plus spécifique, celle de l'**arbre enraciné** (rooted tree). Un arbre enraciné peut être défini de manière récursive comme suit² :

> **Définition 1.1 (Arbre Enraciné)** : Un arbre est un ensemble de nœuds qui est :
> - Soit vide.
> - Soit composé d'un nœud spécial appelé **racine**, et de zéro ou plusieurs sous-ensembles disjoints de nœuds, chacun de ces sous-ensembles étant lui-même un arbre. Ces arbres sont appelés les **sous-arbres** de la racine.

Cette structure hiérarchique impose une propriété fondamentale : dans un arbre enraciné, chaque nœud, à l'exception de la racine, possède exactement un parent⁷. C'est cette contrainte qui garantit l'absence de cycles et établit la hiérarchie de manière non ambiguë.

Une caractéristique importante de cette définition mathématique de l'arbre général est que l'ordre des enfants d'un nœud donné n'a aucune importance. Si un nœud a trois enfants A, B et C, les représenter dans l'ordre A, B, C ou C, A, B ne change en rien la nature de l'arbre. Deux arbres sont dits **isomorphes** s'ils ont la même structure, indépendamment de la manière dont les enfants de chaque nœud sont ordonnés ou dessinés⁷. La détermination de l'isomorphisme entre deux arbres est un problème classique pour lequel des algorithmes efficaces, comme l'algorithme AHU (Aho, Hopcroft, Ullman), ont été développés⁸.

#### 1.2. De l'Arbre Général à l'Arbre Binaire : Ordre et Structure

Dans la pratique de la programmation, nous travaillons rarement avec des arbres généraux directement. Nous privilégions une forme plus contrainte mais immensément plus pratique : l'**arbre binaire**.

La définition formelle d'un arbre binaire est subtilement mais crucialement différente de celle d'un arbre général.

> **Définition 1.2 (Arbre Binaire)** : Un arbre binaire est une structure de données qui est⁷ :
> - Soit vide.
> - Soit composée d'un nœud racine, d'un **sous-arbre gauche** qui est un arbre binaire, et d'un **sous-arbre droit** qui est également un arbre binaire.

La différence fondamentale réside dans les termes "gauche" et "droit". Contrairement aux sous-arbres d'un arbre général, qui forment un ensemble non ordonné, les deux sous-arbres d'un arbre binaire sont spécifiquement ordonnés. Le sous-arbre gauche est distinct du sous-arbre droit. Par conséquent, si l'on inverse le sous-arbre gauche et le sous-arbre droit d'un nœud, on obtient un arbre binaire différent, alors que cela n'aurait aucun sens pour un arbre général⁷. 

Un nœud dans un arbre binaire peut avoir un enfant gauche sans avoir d'enfant droit, et vice-versa. Cette distinction n'existe pas dans un arbre général.

Cette focalisation quasi-exclusive de la programmation sur les arbres binaires peut sembler étrange. Pourquoi se limiter à deux enfants alors que le concept d'arbre général semble, comme son nom l'indique, plus général et donc plus puissant ? Cette question, soulevée lors du séminaire qui a inspiré cette leçon⁷, est d'une grande importance pédagogique. 

Nous laisserons cette question en suspens pour le moment. Elle constituera un fil narratif que nous suivrons tout au long de notre exploration. La réponse, qui sera révélée dans la **Partie IV**, est d'une grande élégance et illustre un principe fondamental de la résolution de problèmes en informatique : la réduction d'un problème complexe à un problème plus simple déjà résolu.

#### 1.3. Terminologie Essentielle

Pour discuter des arbres avec précision, il est nécessaire de maîtriser un vocabulaire standard. Les termes suivants sont fondamentaux :

- **Nœud (Node)** : L'entité de base d'un arbre, contenant une donnée et des liens vers d'autres nœuds.
- **Racine (Root)** : Le seul nœud de l'arbre qui n'a pas de parent. C'est le point d'entrée de l'arbre.
- **Arête (Edge)** : Un lien connectant un nœud parent à un nœud enfant.
- **Parent** : Le nœud directement au-dessus d'un nœud donné dans la hiérarchie. Chaque nœud (sauf la racine) a un unique parent.
- **Enfant (Child)** : Un nœud directement en dessous d'un nœud donné. Un nœud peut avoir zéro, un ou plusieurs enfants (au maximum deux pour un arbre binaire).
- **Feuille (Leaf)** : Un nœud qui n'a aucun enfant.
- **Nœud Interne (Internal Node)** : Tout nœud qui n'est pas une feuille (il a au moins un enfant).
- **Fratrie (Siblings)** : Un ensemble de nœuds qui partagent le même parent.
- **Chemin (Path)** : Une séquence de nœuds et d'arêtes reliant un nœud à un de ses descendants. La longueur d'un chemin est le nombre d'arêtes qu'il contient.
- **Ancêtres (Ancestors)** : L'ensemble des nœuds sur le chemin d'un nœud donné vers la racine (y compris la racine et le nœud lui-même).
- **Descendants (Descendants)** : L'ensemble des nœuds dans les sous-arbres d'un nœud donné (y compris le nœud lui-même).
- **Profondeur d'un Nœud (Depth of a Node)** : La longueur du chemin unique de la racine à ce nœud. La profondeur de la racine est 0.
- **Hauteur d'un Arbre (Height of a Tree)** : La longueur du plus long chemin de la racine à une feuille. La hauteur d'un arbre contenant un seul nœud est 0. Un arbre vide a une hauteur de $-1$.

#### 1.4. Implémentation Fondamentale en C : La Structure TreeNode

Passons maintenant de la théorie à la pratique. Pour représenter un arbre binaire en langage C, nous avons besoin d'une structure pour modéliser un nœud. Conformément à la discussion du séminaire⁷, cette structure doit contenir trois éléments essentiels : la donnée stockée dans le nœud, un pointeur vers le fils gauche et un pointeur vers le fils droit.

Il est également judicieux, comme suggéré dans le séminaire⁷, d'inclure un pointeur supplémentaire vers le parent. Bien qu'il ne soit pas strictement nécessaire pour de nombreuses opérations, ce pointeur parent simplifie grandement certains algorithmes, notamment pour remonter dans l'arbre ou pour des opérations de suppression complexes.

Pour la donnée elle-même, nous utiliserons un type `int` pour la simplicité de nos exemples. Cependant, dans une bibliothèque générique ou une application réelle, ce champ serait plus probablement de type `void*`, un pointeur générique permettant de stocker une adresse vers n'importe quel type de donnée, offrant ainsi une flexibilité maximale⁷.

Voici la définition de notre structure de nœud, que nous placerons dans un fichier d'en-tête `tree.h` pour une bonne organisation du code.

**Fichier : tree.h**
```c
#ifndef TREE_H
#define TREE_H

#include <stdio.h>
#include <stdlib.h>

/**
 * @struct TreeNode
 * @brief Définit la structure pour un nœud d'arbre binaire.
 *
 * Cette structure est l'unité de base de nos arbres binaires.
 * Elle s'inspire de la discussion du séminaire.[7]
 * Chaque nœud contient une donnée, ainsi que des pointeurs vers ses enfants
 * gauche et droit, et optionnellement vers son parent.
 */
typedef struct TreeNode {
    int data;                   // La donnée du nœud (simplifiée à un entier).
    struct TreeNode* left;      // Pointeur vers le sous-arbre gauche.
    struct TreeNode* right;     // Pointeur vers le sous-arbre droit.
    struct TreeNode* parent;    // Pointeur optionnel vers le parent.
} TreeNode;

/**
 * @brief Crée un nouveau nœud d'arbre.
 * @param data La valeur entière à stocker dans le nouveau nœud.
 * @return Un pointeur vers le nœud nouvellement alloué, ou NULL en cas d'échec.
 */
TreeNode* createNode(int data);

#endif // TREE_H
```

Nous incluons également le prototype d'une fonction utilitaire `createNode` qui se chargera d'allouer la mémoire pour un nouveau nœud et d'initialiser ses champs. L'implémentation de cette fonction sera placée dans un fichier source correspondant, `tree.c`.

**Fichier : tree.c**
```c
#include "tree.h"

/**
 * @brief Implémentation de la fonction de création de nœud.
 */
TreeNode* createNode(int data) {
    // Alloue de la mémoire pour un nouveau nœud.
    TreeNode* newNode = (TreeNode*)malloc(sizeof(TreeNode));
    
    // Vérifie si l'allocation a réussi.
    if (newNode == NULL) {
        perror("Échec de l'allocation mémoire pour un nouveau nœud");
        return NULL;
    }
    
    // Initialise les champs du nœud.
    newNode->data = data;
    newNode->left = NULL;
    newNode->right = NULL;
    newNode->parent = NULL; // Le parent sera défini lors de l'insertion.
    
    return newNode;
}
```

Avec cette structure `TreeNode` et cette fonction de création, nous disposons des outils de base pour commencer à construire et à manipuler des arbres binaires en C.

## Partie II : Parcours d'Arbres : Exploration Systématique

Maintenant que nous avons défini ce qu'est un arbre et comment représenter un de ses nœuds, la question suivante est : comment interagir avec l'ensemble de la structure ? Comment visiter chaque nœud pour, par exemple, afficher toutes ses valeurs, calculer leur somme, ou trouver la valeur maximale ?

Ce processus d'exploration systématique est appelé **parcours d'arbre** (tree traversal). Contrairement aux structures de données linéaires qui n'ont qu'un seul ordre de parcours logique (du début à la fin), la nature hiérarchique des arbres offre plusieurs stratégies de parcours distinctes et utiles. Nous nous concentrerons ici sur les trois principales méthodes de parcours en profondeur (Depth-First Search, DFS)¹⁰.

### Chapitre 2 : Algorithmes de Parcours en Profondeur (Récursifs)

La manière la plus naturelle et la plus élégante de décrire et d'implémenter les parcours d'arbres est d'utiliser la récursivité. La définition récursive d'un arbre (un nœud avec deux sous-arbres) se prête parfaitement à des algorithmes récursifs. 

L'idée est de décomposer le problème du parcours d'un arbre en le même problème appliqué à des arbres plus petits (ses sous-arbres), jusqu'à atteindre le cas de base : un arbre vide (`NULL`).

#### 2.1. Le Parcours Préfixe (Pré-ordre / Pre-order) : V-G-D (Visiter, Gauche, Droite)

Le parcours préfixe, ou pré-ordre, suit une logique simple et directe : "traiter d'abord le parent, puis les enfants". L'ordre des opérations est le suivant⁷ :

1. **Visiter** le nœud courant (par exemple, afficher sa donnée).
2. **Parcourir récursivement** tout le sous-arbre gauche.
3. **Parcourir récursivement** tout le sous-arbre droit.

Cette stratégie est particulièrement utile dans plusieurs scénarios. Par exemple, pour copier un arbre, il est logique de créer la copie du nœud racine avant de s'occuper de copier ses sous-arbres. De même, dans le contexte des compilateurs, le parcours préfixe d'un arbre de syntaxe abstraite représentant une expression arithmétique permet de générer sa notation polonaise (préfixe), où l'opérateur précède ses opérandes (ex: `+ A B`)¹⁰.

**Pseudocode :**
```
fonction ParcoursPrefixe(noeud)
    si noeud est NUL alors
        retourner
    fin si

    Visiter(noeud)
    ParcoursPrefixe(noeud.gauche)
    ParcoursPrefixe(noeud.droit)
fin fonction
```

**Implémentation en C :**
```c
// Fichier: tree.c (ajout)

/**
 * @brief Effectue un parcours préfixe (V-G-D) de l'arbre.
 * @param root Pointeur vers la racine du (sous-)arbre à parcourir.
 */
void preorderTraversal(TreeNode* root) {
    // Cas de base de la récursion : si le nœud est nul, il n'y a rien à faire.
    if (root == NULL) {
        return;
    }
    
    // 1. Visiter le nœud courant.
    printf("%d ", root->data);
    
    // 2. Parcourir récursivement le sous-arbre gauche.
    preorderTraversal(root->left);
    
    // 3. Parcourir récursivement le sous-arbre droit.
    preorderTraversal(root->right);
}
```

#### 2.2. Le Parcours Infixe (In-ordre / In-order) : G-V-D (Gauche, Visiter, Droite)

Le parcours infixe, ou in-ordre, est peut-être le plus célèbre en raison de sa propriété spéciale lorsqu'il est appliqué aux arbres binaires de recherche. La logique est la suivante⁷ :

1. **Parcourir récursivement** tout le sous-arbre gauche.
2. **Visiter** le nœud courant.
3. **Parcourir récursivement** tout le sous-arbre droit.

Le cas d'utilisation principal et le plus puissant du parcours infixe est lié aux arbres binaires de recherche (ABR), que nous étudierons en détail dans la **Partie III**. Lorsqu'il est appliqué à un ABR, ce parcours a la propriété remarquable de visiter les nœuds dans leur ordre croissant⁷. C'est comme si l'on "projetait" la structure bidimensionnelle de l'arbre sur une ligne unidimensionnelle triée, une analogie puissante évoquée dans le séminaire⁷.

**Pseudocode :**
```
fonction ParcoursInfixe(noeud)
    si noeud est NUL alors
        retourner
    fin si

    ParcoursInfixe(noeud.gauche)
    Visiter(noeud)
    ParcoursInfixe(noeud.droit)
fin fonction
```

**Implémentation en C :**
```c
// Fichier: tree.c (ajout)

/**
 * @brief Effectue un parcours infixe (G-V-D) de l'arbre.
 * @param root Pointeur vers la racine du (sous-)arbre à parcourir.
 */
void inorderTraversal(TreeNode* root) {
    if (root == NULL) {
        return;
    }
    
    // 1. Parcourir récursivement le sous-arbre gauche.
    inorderTraversal(root->left);
    
    // 2. Visiter le nœud courant.
    printf("%d ", root->data);
    
    // 3. Parcourir récursivement le sous-arbre droit.
    inorderTraversal(root->right);
}
```

#### 2.3. Le Parcours Postfixe (Post-ordre / Post-order) : G-D-V (Gauche, Droite, Visiter)

Le parcours postfixe, ou post-ordre, adopte la logique "traiter les enfants avant le parent". C'est l'inverse du parcours préfixe⁷ :

1. **Parcourir récursivement** tout le sous-arbre gauche.
2. **Parcourir récursivement** tout le sous-arbre droit.
3. **Visiter** le nœud courant.

L'application la plus critique du parcours postfixe est la suppression des nœuds d'un arbre et la libération de la mémoire associée. Pour pouvoir supprimer un nœud parent en toute sécurité, il faut d'abord avoir traité (et supprimé) tous ses descendants. Si l'on supprimait le parent en premier, les pointeurs vers ses enfants seraient perdus, créant une fuite de mémoire massive pour tous les sous-arbres. 

Le parcours postfixe garantit que l'on ne visite (supprime) un nœud qu'après que ses enfants ont été eux-mêmes visités et supprimés⁷. Il est également utilisé pour générer la notation polonaise inversée (postfixe) d'une expression (ex: `A B +`), qui est très utilisée dans les calculatrices basées sur des piles et dans l'évaluation d'expressions par les compilateurs¹⁰.

**Pseudocode :**
```
fonction ParcoursPostfixe(noeud)
    si noeud est NUL alors
        retourner
    fin si

    ParcoursPostfixe(noeud.gauche)
    ParcoursPostfixe(noeud.droit)
    Visiter(noeud)
fin fonction
```

**Implémentation en C :**
```c
// Fichier: tree.c (ajout)

/**
 * @brief Effectue un parcours postfixe (G-D-V) de l'arbre.
 * @param root Pointeur vers la racine du (sous-)arbre à parcourir.
 */
void postorderTraversal(TreeNode* root) {
    if (root == NULL) {
        return;
    }
    
    // 1. Parcourir récursivement le sous-arbre gauche.
    postorderTraversal(root->left);
    
    // 2. Parcourir récursivement le sous-arbre droit.
    postorderTraversal(root->right);
    
    // 3. Visiter le nœud courant.
    printf("%d ", root->data);
}
```

#### Tableau 1 : Comparaison des Algorithmes de Parcours

Pour synthétiser et clarifier les différences entre ces trois stratégies, le tableau suivant présente un résumé comparatif. Il utilise un exemple d'arbre simple pour illustrer les séquences générées par chaque parcours.

**Arbre Exemple :**
```
    1
   / \
  2   3
 / \
4   5
```

| Nom du Parcours | Ordre de Visite | Séquence (Arbre Exemple) | Cas d'Utilisation Principal |
|-----------------|-----------------|---------------------------|------------------------------|
| Préfixe (Pre-order) | Racine, Gauche, Droite | 1, 2, 4, 5, 3 | Copie d'arbre, Notation Polonaise |
| Infixe (In-order) | Gauche, Racine, Droite | 4, 2, 5, 1, 3 | Affichage trié (pour ABR) |
| Postfixe (Post-order) | Gauche, Droite, Racine | 4, 5, 2, 3, 1 | Suppression d'arbre, Notation Polonaise Inversée |

### Chapitre 3 : Implémentations Itératives avec une Pile

#### 3.1. Les Limites de la Récursivité

Bien que les algorithmes récursifs soient élégants et concis, ils présentent une faiblesse pratique majeure. Chaque appel de fonction récursif consomme de l'espace sur la pile d'appels du système (call stack) pour stocker l'état de la fonction (variables locales, adresse de retour). 

Si un arbre est très profond, en particulier s'il est dégénéré (ressemblant à une longue liste chaînée), le nombre d'appels récursifs imbriqués peut devenir très grand. Cela peut entraîner un **dépassement de pile** (stack overflow), un crash du programme dû à l'épuisement de la mémoire allouée à la pile⁷.

Pour pallier ce problème et obtenir un contrôle plus fin sur l'utilisation de la mémoire, nous pouvons convertir ces algorithmes récursifs en versions itératives. La clé de cette transformation est l'utilisation d'une structure de données explicite pour gérer les états que la récursion gérait implicitement : une **pile** (stack).

#### 3.2. La Pile comme Simulation de la Récursion

L'approche itérative n'est pas simplement une "autre façon" de faire un parcours. Elle peut être comprise comme une simulation manuelle et explicite de la pile d'appels que le système d'exploitation utilise pour gérer la récursion. 

Lorsque nous effectuons un appel récursif, le système "sauvegarde" l'état actuel sur sa pile pour pouvoir y revenir plus tard. Dans une approche itérative, nous faisons exactement la même chose : nous utilisons notre propre structure de pile pour sauvegarder les nœuds que nous devrons revisiter¹³.

Par exemple, dans un parcours préfixe, après avoir visité un nœud, la récursion irait à gauche. L'information "il faudra revenir ici pour aller à droite" est implicitement stockée sur la pile d'appels. Dans notre version itérative, nous pousserons explicitement le nœud sur notre pile avant d'aller à gauche, pour nous souvenir de revenir plus tard pour traiter son côté droit.

Cette perspective unifie les deux approches : elles sont conceptuellement équivalentes, l'une étant gérée automatiquement par le système, l'autre manuellement par le programmeur.

#### 3.3. Conception d'une Pile en C avec une Liste Chaînée

Pour implémenter nos parcours itératifs, nous avons besoin d'une pile. Conformément à la logique du séminaire⁷, une pile peut être efficacement implémentée à l'aide d'une simple liste chaînée, où les opérations `push` (empiler) et `pop` (dépiler) s'effectuent en tête de liste, garantissant une complexité en temps constant $O(1)$.

Nous créerons un nouvel ensemble de fichiers, `stack.h` et `stack.c`, pour notre implémentation de pile générique pour les pointeurs `TreeNode*`.

**Fichier : stack.h**
```c
#ifndef STACK_H
#define STACK_H

#include "tree.h"

/**
 * @struct StackNode
 * @brief Définit le nœud pour une pile implémentée comme une liste chaînée.
 * Chaque nœud de la pile contient un pointeur vers un TreeNode.
 */
typedef struct StackNode {
    TreeNode* tree_node;
    struct StackNode* next;
} StackNode;

// Prototypes des fonctions de la pile
void push(StackNode** top_ref, TreeNode* tn);
TreeNode* pop(StackNode** top_ref);
int isStackEmpty(StackNode* top);
TreeNode* peek(StackNode* top);

#endif // STACK_H
```

L'implémentation dans `stack.c` doit gérer avec soin les pointeurs, en particulier l'utilisation d'un pointeur sur pointeur (`StackNode**`) pour les fonctions `push` et `pop`. Ceci est essentiel car ces fonctions doivent modifier le pointeur de tête de la pile (`top`) lui-même, qui est local à la fonction appelante. Passer un simple pointeur passerait une copie de l'adresse, et toute modification serait perdue. Passer un pointeur sur ce pointeur nous permet de modifier la valeur originale⁷.

**Fichier : stack.c**
```c
#include "stack.h"
#include <stdio.h>
#include <stdlib.h>

/**
 * @brief Empile un TreeNode sur le dessus de la pile.
 * @param top_ref Adresse du pointeur de tête de la pile.
 * @param tn Pointeur vers le TreeNode à empiler.
 */
void push(StackNode** top_ref, TreeNode* tn) {
    StackNode* new_stack_node = (StackNode*)malloc(sizeof(StackNode));
    if (new_stack_node == NULL) {
        perror("Échec de l'allocation pour le nœud de pile");
        exit(EXIT_FAILURE);
    }
    
    new_stack_node->tree_node = tn;
    new_stack_node->next = (*top_ref); // Le nouveau nœud pointe vers l'ancienne tête.
    (*top_ref) = new_stack_node;       // La tête pointe maintenant vers le nouveau nœud.
}

/**
 * @brief Dépile le TreeNode du dessus de la pile.
 * @param top_ref Adresse du pointeur de tête de la pile.
 * @return Pointeur vers le TreeNode qui a été dépilé.
 */
TreeNode* pop(StackNode** top_ref) {
    if (isStackEmpty(*top_ref)) {
        fprintf(stderr, "Erreur : tentative de dépiler d'une pile vide.\n");
        return NULL;
    }
    
    StackNode* top_node = *top_ref;
    TreeNode* result = top_node->tree_node;
    *top_ref = top_node->next; // La tête pointe maintenant vers le nœud suivant.
    free(top_node);
    
    return result;
}

/**
 * @brief Vérifie si la pile est vide.
 * @param top Pointeur de tête de la pile.
 * @return 1 si la pile est vide, 0 sinon.
 */
int isStackEmpty(StackNode* top) {
    return (top == NULL);
}

/**
 * @brief Retourne l'élément au sommet de la pile sans le dépiler.
 * @param top Pointeur de tête de la pile.
 * @return Pointeur vers le TreeNode au sommet.
 */
TreeNode* peek(StackNode* top) {
    if (isStackEmpty(top)) {
        return NULL;
    }
    return top->tree_node;
}
```

#### 3.4. Algorithmes de Parcours Itératifs

Armés de notre pile, nous pouvons maintenant implémenter les versions itératives des trois parcours.

##### Parcours Infixe Itératif

C'est l'exemple classique. La logique est de descendre le plus à gauche possible, en empilant les nœuds sur le chemin. Une fois au bout, on dépile un nœud, on le visite, puis on passe à son sous-arbre droit pour recommencer le processus.

**Pseudocode :**
```
fonction ParcoursInfixeIteratif(racine)
    créer une pile vide S
    courant = racine

    tant que courant n'est pas NUL ou S n'est pas vide
        // Atteindre le nœud le plus à gauche du nœud courant
        tant que courant n'est pas NUL
            empiler(S, courant)
            courant = courant.gauche
        fin tant

        // Le courant doit être NUL à ce stade
        courant = dépiler(S)
        Visiter(courant)

        // Maintenant, c'est au tour du sous-arbre droit
        courant = courant.droit
    fin tant
fin fonction
```

**Implémentation en C :**
```c
// Fichier: tree.c (ajout)
#include "stack.h" // N'oubliez pas d'inclure l'en-tête de la pile

/**
 * @brief Effectue un parcours infixe itératif (G-V-D) de l'arbre.
 * @param root Pointeur vers la racine de l'arbre.
 */
void inorderTraversal_iterative(TreeNode* root) {
    StackNode* stack = NULL;
    TreeNode* current = root;

    while (current != NULL || !isStackEmpty(stack)) {
        // Atteindre le nœud le plus à gauche du nœud 'current'
        while (current != NULL) {
            push(&stack, current);
            current = current->left;
        }

        // 'current' est NULL, dépiler le dernier nœud
        current = pop(&stack);
        printf("%d ", current->data); // Visiter le nœud

        // Se préparer à visiter le sous-arbre droit
        current = current->right;
    }
    printf("\n");
}
```

##### Parcours Préfixe Itératif

L'approche itérative pour le pré-ordre est encore plus simple. On empile la racine, puis dans une boucle, on dépile un nœud, on le visite, et on empile ses enfants droit puis gauche (droit en premier pour que le gauche soit traité en premier, car la pile est LIFO - Last-In, First-Out).

**Pseudocode :**
```
fonction ParcoursPrefixeIteratif(racine)
    si racine est NUL alors
        retourner
    fin si

    créer une pile vide S
    empiler(S, racine)

    tant que S n'est pas vide
        noeud = dépiler(S)
        Visiter(noeud)

        // Empiler l'enfant droit en premier pour qu'il soit traité après le gauche
        si noeud.droit n'est pas NUL alors
            empiler(S, noeud.droit)
        fin si
        si noeud.gauche n'est pas NUL alors
            empiler(S, noeud.gauche)
        fin si
    fin tant
fin fonction
```

**Implémentation en C :**
```c
// Fichier: tree.c (ajout)

/**
 * @brief Effectue un parcours préfixe itératif (V-G-D) de l'arbre.
 * @param root Pointeur vers la racine de l'arbre.
 */
void preorderTraversal_iterative(TreeNode* root) {
    if (root == NULL) return;

    StackNode* stack = NULL;
    push(&stack, root);

    while (!isStackEmpty(stack)) {
        TreeNode* current = pop(&stack);
        printf("%d ", current->data); // Visiter

        // Empiler l'enfant droit en premier
        if (current->right) {
            push(&stack, current->right);
        }
        // Empiler l'enfant gauche en dernier pour qu'il soit au sommet
        if (current->left) {
            push(&stack, current->left);
        }
    }
    printf("\n");
}
```

##### Parcours Postfixe Itératif

Le parcours postfixe itératif est le plus complexe des trois. Un nœud ne peut être visité qu'après ses deux enfants. Une astuce courante consiste à utiliser deux piles, ou une seule pile avec une logique plus complexe pour savoir si l'on revient d'un sous-arbre gauche ou droit. Une approche élégante consiste à utiliser un pointeur `prev` pour garder la trace du dernier nœud visité.

**Pseudocode (avec un pointeur prev) :**
```
fonction ParcoursPostfixeIteratif(racine)
    créer une pile vide S
    courant = racine
    dernierVisite = NUL

    tant que courant n'est pas NUL ou S n'est pas vide
        tant que courant n'est pas NUL
            empiler(S, courant)
            courant = courant.gauche
        fin tant

        courant = regarder(S) // peek

        // Si le nœud droit n'existe pas ou a déjà été visité
        si courant.droit est NUL ou courant.droit est dernierVisite
            Visiter(courant)
            dépiler(S)
            dernierVisite = courant
            courant = NUL // Pour forcer le dépilement du parent à la prochaine itération
        sinon
            // Aller explorer le sous-arbre droit
            courant = courant.droit
        fin si
    fin tant
fin fonction
```

**Implémentation en C :**
```c
// Fichier: tree.c (ajout)

/**
 * @brief Effectue un parcours postfixe itératif (G-D-V) de l'arbre.
 * @param root Pointeur vers la racine de l'arbre.
 */
void postorderTraversal_iterative(TreeNode* root) {
    if (root == NULL) return;

    StackNode* stack = NULL;
    TreeNode* lastNodeVisited = NULL;
    TreeNode* current = root;

    while (current != NULL || !isStackEmpty(stack)) {
        if (current != NULL) {
            push(&stack, current);
            current = current->left;
        } else {
            TreeNode* peekNode = peek(stack);
            // Si le fils droit existe et n'a pas été visité
            if (peekNode->right != NULL && lastNodeVisited != peekNode->right) {
                current = peekNode->right;
            } else {
                printf("%d ", peekNode->data);
                lastNodeVisited = pop(&stack);
            }
        }
    }
    printf("\n");
}
```

Ces implémentations itératives, bien que plus verbeuses, sont essentielles pour écrire du code robuste capable de gérer des arbres de n'importe quelle forme et profondeur sans risquer un crash du programme.

## Partie III : L'Arbre Binaire de Recherche (ABR)

Nous avons jusqu'à présent considéré les arbres binaires comme de simples conteneurs hiérarchiques. Nous allons maintenant introduire une contrainte supplémentaire, un ordre spécifique sur les données, qui transforme l'arbre binaire en un outil de recherche extraordinairement puissant : l'**Arbre Binaire de Recherche (ABR)**, ou Binary Search Tree (BST) en anglais¹².

### Chapitre 4 : La Propriété d'un ABR et les Opérations Fondamentales

#### 4.1. L'Invariant comme Âme de la Structure de Données

Le concept central qui définit un ABR et lui confère toute sa puissance est celui d'**invariant**. Un invariant de structure de données est une propriété ou un ensemble de règles qui doit être vrai à tout moment, avant et après chaque opération qui modifie la structure. 

Pour un ABR, cet invariant est la **propriété d'arbre binaire de recherche**. Toutes les opérations que nous concevrons (insertion, suppression) auront pour objectif principal de réaliser leur tâche tout en garantissant la préservation de cet invariant¹⁵.

C'est cet invariant qui permet des recherches en temps logarithmique. C'est aussi une conséquence directe de cet invariant que le parcours infixe d'un ABR produit une liste triée de ses éléments⁷. Comprendre le rôle de l'invariant, c'est comprendre l'essence même de la conception de structures de données avancées. Ce n'est pas une simple caractéristique, c'est la règle du jeu qui dicte la logique de tous les algorithmes associés.

#### 4.2. Définition de la Propriété ABR (Binary Search Tree Property)

> **Définition 4.1 (Propriété ABR)** : Un arbre binaire est un arbre binaire de recherche si, pour chaque nœud $n$ de l'arbre, les conditions suivantes sont respectées¹⁵ :
> 1. Toutes les valeurs de clé dans le sous-arbre gauche de $n$ sont strictement inférieures à la valeur de la clé de $n$.
> 2. Toutes les valeurs de clé dans le sous-arbre droit de $n$ sont strictement supérieures à la valeur de la clé de $n$.
> 3. Les sous-arbres gauche et droit de $n$ sont eux-mêmes des arbres binaires de recherche.

Cette propriété doit être maintenue récursivement à travers tout l'arbre. Un ABR ne contient généralement pas de clés dupliquées.

**Exemple d'ABR valide :**
```
        8
       / \
      3   10
     / \    \
    1   6   14
       / \   /
      4   7 13
```

**Exemple d'arbre NON-ABR :**
```
        8
       / \
      3   10
     / \    \
    1   6   14
       / \   /
      4   9 13  ← Erreur: 9 > 8, devrait être dans le sous-arbre droit de 8
```

#### 4.3. Recherche d'un Élément (search)

La propriété ABR rend la recherche d'un élément remarquablement efficace, car elle s'apparente à une recherche dichotomique. À chaque nœud, nous pouvons éliminer la moitié de l'arbre restant.

**Logique :**
1. Commencer à la racine.
2. Si l'arbre est vide (`NULL`), la clé n'existe pas.
3. Comparer la clé recherchée avec la clé du nœud courant.
   - Si elles sont égales, la clé est trouvée.
   - Si la clé recherchée est plus petite, continuer la recherche dans le sous-arbre gauche.
   - Si la clé recherchée est plus grande, continuer la recherche dans le sous-arbre droit.
4. Répéter le processus jusqu'à ce que la clé soit trouvée ou qu'un sous-arbre vide soit atteint.

**Pseudocode (Récursif) :**
```
fonction RechercheRecursive(noeud, cle)
    si noeud est NUL ou noeud.cle == cle alors
        retourner noeud
    fin si
    si cle < noeud.cle alors
        retourner RechercheRecursive(noeud.gauche, cle)
    sinon
        retourner RechercheRecursive(noeud.droit, cle)
    fin si
fin fonction
```

**Implémentation en C (Récursive et Itérative) :**
```c
// Fichier: bst.c (nouveau fichier pour les opérations ABR)
#include "tree.h"

/**
 * @brief Recherche une clé dans un ABR de manière récursive.
 * @param root La racine du (sous-)arbre où chercher.
 * @param data La clé à trouver.
 * @return Un pointeur vers le nœud contenant la clé, ou NULL si non trouvée.
 */
TreeNode* search_recursive(TreeNode* root, int data) {
    if (root == NULL || root->data == data) {
        return root;
    }
    
    if (data < root->data) {
        return search_recursive(root->left, data);
    } else {
        return search_recursive(root->right, data);
    }
}

/**
 * @brief Recherche une clé dans un ABR de manière itérative.
 * @param root La racine de l'arbre.
 * @param data La clé à trouver.
 * @return Un pointeur vers le nœud contenant la clé, ou NULL si non trouvée.
 */
TreeNode* search_iterative(TreeNode* root, int data) {
    TreeNode* current = root;
    while (current != NULL && current->data != data) {
        if (data < current->data) {
            current = current->left;
        } else {
            current = current->right;
        }
    }
    return current;
}
```

#### 4.4. Insertion d'un Nouvel Élément (insert)

L'insertion d'un nouvel élément doit préserver l'invariant de l'ABR. La procédure est heureusement simple : on cherche l'endroit où l'élément devrait se trouver, et on l'y insère.

**Logique :**
1. Effectuer une recherche pour la clé à insérer.
2. La recherche se terminera à un pointeur `NULL`, car la clé n'est pas encore dans l'arbre.
3. Le dernier nœud non-nul visité avant d'atteindre `NULL` sera le parent du nouveau nœud.
4. Créer le nouveau nœud et l'attacher comme enfant gauche ou droit du parent, en fonction de la comparaison des clés.

L'implémentation récursive est particulièrement élégante, car elle reconstruit les liens en remontant la pile d'appels¹⁸.

**Pseudocode (Récursif) :**
```
fonction Insertion(noeud, cle)
    si noeud est NUL alors
        retourner creerNouveauNoeud(cle)
    fin si

    si cle < noeud.cle alors
        noeud.gauche = Insertion(noeud.gauche, cle)
    sinon si cle > noeud.cle alors
        noeud.droit = Insertion(noeud.droit, cle)
    fin si
    
    // Si la clé existe déjà, ne rien faire et retourner le nœud.
    retourner noeud
fin fonction
```

**Implémentation en C :**
```c
// Fichier: bst.c (ajout)
#include "tree.h" // createNode est défini dans tree.c

/**
 * @brief Insère une nouvelle clé dans un ABR.
 * @param root La racine du (sous-)arbre.
 * @param data La clé à insérer.
 * @return La racine du (sous-)arbre modifié.
 */
TreeNode* insert(TreeNode* root, int data) {
    // Si l'arbre (ou sous-arbre) est vide, créer le nouveau nœud ici.
    if (root == NULL) {
        return createNode(data);
    }

    // Sinon, descendre récursivement dans l'arbre.
    if (data < root->data) {
        TreeNode* left_child = insert(root->left, data);
        root->left = left_child;
        left_child->parent = root; // Maintenir le pointeur parent
    } else if (data > root->data) {
        TreeNode* right_child = insert(root->right, data);
        root->right = right_child;
        right_child->parent = root; // Maintenir le pointeur parent
    }

    // Si data == root->data, la clé existe déjà. On ne fait rien.
    // Retourner le pointeur de la racine (inchangé ou non).
    return root;
}
```

#### 4.5. Suppression d'un Élément (delete)

La suppression est l'opération la plus complexe sur un ABR, car il est plus difficile de "reboucher le trou" tout en préservant l'invariant. On distingue trois cas¹⁵ :

**Cas 1 :** Le nœud à supprimer est une feuille. C'est le cas le plus simple. On peut simplement le supprimer et mettre à `NULL` le pointeur du parent correspondant.

**Cas 2 :** Le nœud à supprimer a un seul enfant. On peut "promouvoir" cet unique enfant à la place de son parent. Le parent du nœud supprimé pointera désormais vers le petit-enfant.

**Cas 3 :** Le nœud à supprimer a deux enfants. C'est le cas le plus délicat. On ne peut pas simplement supprimer le nœud. La stratégie consiste à :
   a. Trouver le **successeur en ordre infixe** du nœud. Le successeur est le plus petit nœud qui est plus grand que le nœud à supprimer. Il se trouve en allant une fois à droite, puis le plus à gauche possible. Ce successeur aura au plus un enfant (un enfant droit).
   b. Copier la valeur du successeur dans le nœud que l'on souhaite supprimer.
   c. Supprimer récursivement le successeur de sa position d'origine. Comme le successeur a au plus un enfant, sa suppression tombe dans le Cas 1 ou le Cas 2, qui sont plus simples à gérer.

**Fonction auxiliaire findMin :**
Pour implémenter le Cas 3, nous avons besoin d'une fonction qui trouve le nœud avec la valeur minimale dans un sous-arbre donné. C'est simplement le nœud le plus à gauche.

```c
// Fichier: bst.c (ajout)

/**
 * @brief Trouve le nœud avec la valeur minimale dans un ABR.
 * @param node La racine du (sous-)arbre.
 * @return Un pointeur vers le nœud contenant la valeur minimale.
 */
TreeNode* findMin(TreeNode* node) {
    TreeNode* current = node;
    while (current && current->left != NULL) {
        current = current->left;
    }
    return current;
}
```

**Implémentation en C de deleteNode :**
```c
// Fichier: bst.c (ajout)

/**
 * @brief Supprime un nœud avec une clé donnée d'un ABR.
 * @param root La racine du (sous-)arbre.
 * @param data La clé à supprimer.
 * @return La racine du (sous-)arbre modifié.
 */
TreeNode* deleteNode(TreeNode* root, int data) {
    if (root == NULL) return root;

    // Trouver le nœud à supprimer
    if (data < root->data) {
        root->left = deleteNode(root->left, data);
    } else if (data > root->data) {
        root->right = deleteNode(root->right, data);
    } else { // C'est le nœud à supprimer
        // Cas 1 : Nœud avec un seul enfant ou pas d'enfant
        if (root->left == NULL) {
            TreeNode* temp = root->right;
            if (temp) temp->parent = root->parent;
            free(root);
            return temp;
        } else if (root->right == NULL) {
            TreeNode* temp = root->left;
            if (temp) temp->parent = root->parent;
            free(root);
            return temp;
        }

        // Cas 3 : Nœud avec deux enfants
        // Obtenir le successeur en ordre infixe (le plus petit dans le sous-arbre droit)
        TreeNode* temp = findMin(root->right);

        // Copier le contenu du successeur dans ce nœud
        root->data = temp->data;

        // Supprimer le successeur
        root->right = deleteNode(root->right, temp->data);
    }
    return root;
}
```

### Chapitre 5 : Validation et Analyse de Performance

#### 5.1. Algorithme de Vérification de la Propriété ABR (isBST)

Comment s'assurer qu'un arbre binaire donné est bien un ABR valide ? Cette question est un excellent exercice pour tester la compréhension de l'invariant ABR.

##### L'Approche Naïve et Incorrecte

Une première idée, souvent erronée, consiste à vérifier pour chaque nœud que son fils gauche est plus petit et son fils droit plus grand. Cet algorithme, similaire à celui observé dans la revue de code du séminaire⁷, est défectueux.

```c
// NE PAS UTILISER - ALGORITHME INCORRECT
int isBST_naive(TreeNode* node) {
    if (node == NULL) return 1;
    if (node->left != NULL && node->left->data > node->data) return 0;
    if (node->right != NULL && node->right->data < node->data) return 0;
    if (!isBST_naive(node->left) || !isBST_naive(node->right)) return 0;
    return 1;
}
```

Ce code échoue car il ne vérifie qu'une propriété locale. Il ne garantit pas que tous les nœuds du sous-arbre gauche sont plus petits que le parent, mais seulement le fils direct. L'exemple de l'arbre non-valide montré précédemment illustre ce défaut : le nœud 9 est bien plus grand que son parent 6, mais il est plus petit que son grand-parent 8, ce qui viole l'invariant global⁷.

##### L'Approche Correcte et Robuste

La solution correcte consiste à propager des contraintes de bornes $(min, max)$ lors du parcours récursif. Pour un nœud donné, sa valeur doit être comprise dans l'intervalle $(min, max)$ autorisé par ses ancêtres⁷.

**Logique :**
1. Commencer à la racine avec un intervalle infini $(-∞, +∞)$.
2. Pour chaque nœud, vérifier si sa valeur est dans l'intervalle $(min, max)$. Si non, l'arbre n'est pas un ABR.
3. Lors de l'appel récursif pour le sous-arbre gauche, la nouvelle borne supérieure devient la valeur du nœud courant. L'intervalle devient $(min, \text{node→data})$.
4. Lors de l'appel récursif pour le sous-arbre droit, la nouvelle borne inférieure devient la valeur du nœud courant. L'intervalle devient $(\text{node→data}, max)$.

**Implémentation en C :**
```c
// Fichier: bst.c (ajout)
#include <limits.h> // Pour INT_MIN et INT_MAX

/**
 * @brief Fonction auxiliaire pour vérifier si un arbre est un ABR.
 * @param node Le nœud courant.
 * @param min La borne inférieure autorisée.
 * @param max La borne supérieure autorisée.
 * @return 1 si le sous-arbre est un ABR, 0 sinon.
 */
int isBSTUtil(TreeNode* node, int min, int max) {
    if (node == NULL) {
        return 1; // Un arbre vide est un ABR
    }

    // Vérifier les contraintes
    if (node->data <= min || node->data >= max) {
        return 0;
    }

    // Vérifier récursivement les sous-arbres avec les nouvelles contraintes
    return isBSTUtil(node->left, min, node->data) &&
           isBSTUtil(node->right, node->data, max);
}

/**
 * @brief Fonction principale pour vérifier si un arbre est un ABR.
 * @param root La racine de l'arbre.
 * @return 1 si l'arbre est un ABR, 0 sinon.
 */
int isBST(TreeNode* root) {
    // Utilise les valeurs min/max des entiers comme bornes initiales.
    // Note : si les clés peuvent être INT_MIN/INT_MAX, il faut une approche
    // avec des pointeurs ou des types plus grands.
    return isBSTUtil(root, INT_MIN, INT_MAX);
}
```

#### 5.2. Analyse de Complexité

La performance d'un ABR dépend de manière critique de sa hauteur.

**Cas Moyen (Arbre Équilibré) :** Si les insertions se font dans un ordre aléatoire, l'arbre a tendance à être relativement équilibré. Sa hauteur, $h$, est proportionnelle au logarithme du nombre de nœuds, $h ≈ \log_2 n$. Comme chaque opération (recherche, insertion, suppression) implique de descendre d'un niveau à chaque étape, la complexité temporelle moyenne est de $O(h)$, soit $O(\log n)$. C'est cette performance logarithmique qui rend les ABR si attractifs.

**Pire Cas (Arbre Dégénéré) :** Si les clés sont insérées dans un ordre déjà trié (croissant ou décroissant), l'arbre dégénère en une simple liste chaînée. Chaque nouveau nœud est ajouté comme l'enfant droit (ou gauche) du précédent. Dans ce cas, la hauteur de l'arbre devient $h = n - 1$. Les opérations ne peuvent plus éliminer la moitié de l'arbre à chaque étape et doivent parcourir potentiellement tous les nœuds. La complexité devient alors linéaire, $O(n)$, ce qui est aussi mauvais qu'une recherche dans une liste non triée et annule tout l'avantage de la structure¹².

Ce problème de dégénérescence est la principale faiblesse des ABR simples. Il a motivé le développement de structures plus complexes, les arbres binaires de recherche auto-équilibrés (comme les arbres AVL ou les arbres rouge-noir), qui effectuent des opérations de "rotation" pour garantir que la hauteur de l'arbre reste toujours logarithmique, assurant ainsi une performance $O(\log n)$ même dans le pire des cas¹².

#### Tableau 2 : Analyse de Complexité des Opérations sur ABR

| Opération | Complexité Temporelle (Cas Moyen) | Complexité Temporelle (Pire Cas) | Complexité Spatiale (Itératif) | Complexité Spatiale (Récursif) |
|-----------|----------------------------------|-----------------------------------|--------------------------------|---------------------------------|
| Recherche | $O(\log n)$ | $O(n)$ | $O(1)$ | $O(h)$ |
| Insertion | $O(\log n)$ | $O(n)$ | $O(1)$ | $O(h)$ |
| Suppression | $O(\log n)$ | $O(n)$ | $O(1)$ | $O(h)$ |

*(Note : $h$ est la hauteur de l'arbre. Dans le pire des cas, $h = O(n)$)*

## Partie IV : Sujets Avancés et Représentations Alternatives

Dans cette dernière partie, nous allons explorer des concepts plus sophistiqués qui non seulement résolvent des problèmes pratiques importants mais révèlent également la beauté et la puissance des abstractions en informatique. Nous répondrons enfin à la question que nous avions posée : pourquoi les arbres binaires sont-ils si centraux en programmation ?

### Chapitre 6 : La Représentation Fils Gauche, Frère Droit

#### 6.1. Le Problème : Représenter des Arbres Généraux

Revenons à notre point de départ : les arbres généraux (ou n-aires), où un nœud peut avoir un nombre arbitraire d'enfants. Comment représenter une telle structure en mémoire ? Une approche naïve consisterait à utiliser pour chaque nœud un tableau de pointeurs vers ses enfants. Cependant, cette méthode présente de graves inconvénients²⁰ :

- **Gaspillage de mémoire :** Il faut allouer un tableau de taille maximale pour chaque nœud, même pour les feuilles qui n'ont pas d'enfants.
- **Manque de flexibilité :** Le nombre maximal d'enfants est fixé à l'avance. Si un nœud doit en avoir plus, la structure doit être modifiée.
- **Complexité des opérations :** L'insertion ou la suppression d'un enfant au milieu de la liste peut nécessiter de décaler tous les autres pointeurs.

#### 6.2. La Solution : Le "Tour de Magie Diaboliquement Malin"

C'est ici que nous résolvons l'énigme posée en **Partie I**. La raison pour laquelle nous nous concentrons tant sur les arbres binaires est qu'il existe une méthode élégante pour représenter n'importe quel arbre général en utilisant une structure d'arbre binaire, **sans perte d'information**. 

Cette technique, connue sous le nom de représentation **"fils gauche, frère droit"** (left-child, right-sibling), est un exemple classique de réduction d'un problème à un autre, plus simple et déjà maîtrisé⁷.

**L'algorithme de transformation est le suivant :**

Pour chaque nœud $n$ de l'arbre général :
- Le pointeur `left` du nœud binaire correspondant pointera vers le **premier enfant** de $n$ dans l'arbre général.
- Le pointeur `right` du nœud binaire pointera vers le **frère immédiatement suivant** de $n$ (c'est-à-dire le prochain enfant du même parent).

##### Visualisation de la Transformation

Considérons l'arbre général suivant :

```
    A
   /|\
  B C D
 /|   |
E F   G
```

En appliquant la transformation "fils gauche, frère droit", nous obtenons l'arbre binaire équivalent :

```
    A
   /
  B -----> C -----> D
 /                  /
E -----> F         G
```

Où :
- Les flèches vers le bas (/) représentent les liens "fils gauche"
- Les flèches horizontales (---->) représentent les liens "frère droit"

Dans cette nouvelle structure, une descente via un pointeur `left` correspond à descendre vers le premier enfant dans l'arbre original. Une traversée via des pointeurs `right` correspond à parcourir la liste des frères et sœurs.

Cette technique est d'une importance capitale. Elle démontre que les algorithmes que nous avons développés pour les arbres binaires (parcours, etc.) peuvent être appliqués indirectement à n'importe quel type d'arbre. En maîtrisant les arbres binaires, nous maîtrisons en réalité une classe de problèmes beaucoup plus large. C'est l'une des plus belles illustrations de la puissance de l'abstraction en informatique.

### Chapitre 7 : Reconstruction d'Arbres à partir des Parcours

#### 7.1. Le Théorème d'Unicité

Un autre problème classique et fascinant est celui de la reconstruction d'un arbre. Si l'on nous donne la séquence des clés visitées lors d'un parcours, pouvons-nous reconstruire l'arbre original ? 

Un seul parcours ne suffit pas. Par exemple, la séquence infixe `[4, 2, 5, 1, 3]` peut correspondre à plusieurs arbres différents.

Cependant, un théorème fondamental stipule qu'**un arbre binaire peut être reconstruit de manière unique si l'on dispose de son parcours infixe ET de son parcours préfixe (ou postfixe)**⁷. La combinaison du parcours infixe (qui sépare les sous-arbres gauche et droit) et d'un parcours préfixe ou postfixe (qui identifie la racine) lève toute ambiguïté.

#### 7.2. Algorithme de Reconstruction (Pré-ordre et In-ordre)

L'algorithme de reconstruction est un excellent exemple de la stratégie "diviser pour régner" et se prête naturellement à une implémentation récursive⁷.

**Étapes de l'Algorithme :**

1. **Identifier la racine :** Le premier élément de la séquence de parcours préfixe est toujours la racine du (sous-)arbre courant.

2. **Créer le nœud racine :** Créer un nouveau nœud avec cette valeur.

3. **Diviser le parcours infixe :** Trouver la position de cette racine dans la séquence de parcours infixe.

4. **Identifier les sous-arbres :** Tous les éléments à gauche de la racine dans la séquence infixe appartiennent au sous-arbre gauche. Tous les éléments à droite appartiennent au sous-arbre droit.

5. **Diviser le parcours préfixe :** Le nombre d'éléments dans le sous-arbre gauche (obtenu à l'étape 4) nous indique combien d'éléments suivant la racine dans la séquence préfixe appartiennent au sous-arbre gauche. Le reste appartient au sous-arbre droit.

6. **Appels récursifs :** Appeler récursivement la fonction de construction pour le sous-arbre gauche (en utilisant les sous-séquences gauches des deux parcours) et pour le sous-arbre droit (en utilisant les sous-séquences droites). Le résultat de ces appels devient les enfants gauche et droit du nœud racine créé à l'étape 2.

7. **Cas de base :** Si une séquence de parcours est vide, retourner `NULL`.

##### Optimisation avec une Table de Hachage

L'étape 3, qui consiste à trouver l'index de la racine dans le parcours infixe, peut être coûteuse si elle est réalisée par un balayage linéaire à chaque appel récursif, menant à une complexité globale de $O(n^2)$. 

Pour optimiser cela, on peut pré-calculer une table de hachage (ou un simple tableau si les valeurs sont des indices) qui mappe chaque valeur de la séquence infixe à son index. Cela permet de trouver la position de la racine en temps constant, $O(1)$, ramenant la complexité totale de l'algorithme à un temps linéaire optimal de $O(n)$²³.

**Implémentation en C :**

L'implémentation ci-dessous utilise une fonction de recherche linéaire simple pour la clarté, mais une note commente l'optimisation possible.

```c
// Fichier: tree_reconstruction.c
#include "tree.h"
#include <stdio.h>
#include <stdlib.h>

// Fonction auxiliaire pour trouver l'index d'une valeur dans un tableau.
// Pour une optimisation O(n), pré-calculer une table de hachage.
int searchInorder(int inorder[], int start, int end, int value) {
    for (int i = start; i <= end; i++) {
        if (inorder[i] == value) {
            return i;
        }
    }
    return -1;
}

// Fonction récursive pour construire l'arbre
TreeNode* buildTreeUtil(int inorder[], int preorder[], int inStart, int inEnd, int* preIndex) {
    if (inStart > inEnd) {
        return NULL;
    }

    // 1. La racine est le prochain élément du parcours préfixe
    int root_data = preorder[*preIndex];
    (*preIndex)++;
    
    // 2. Créer le nœud racine
    TreeNode* root = createNode(root_data);

    // Si ce nœud n'a pas d'enfants, on a fini pour cette branche
    if (inStart == inEnd) {
        return root;
    }

    // 3. Trouver l'index de la racine dans le parcours infixe
    int inIndex = searchInorder(inorder, inStart, inEnd, root->data);

    // 6. Construire récursivement les sous-arbres gauche et droit
    root->left = buildTreeUtil(inorder, preorder, inStart, inIndex - 1, preIndex);
    root->right = buildTreeUtil(inorder, preorder, inIndex + 1, inEnd, preIndex);
    
    // Mettre à jour les pointeurs parents si nécessaire
    if(root->left) root->left->parent = root;
    if(root->right) root->right->parent = root;

    return root;
}

/**
 * @brief Construit un arbre binaire à partir de ses parcours préfixe et infixe.
 * @param inorder Tableau du parcours infixe.
 * @param preorder Tableau du parcours préfixe.
 * @param len Nombre d'éléments dans les parcours.
 * @return La racine de l'arbre construit.
 */
TreeNode* buildTree(int inorder[], int preorder[], int len) {
    int preIndex = 0;
    return buildTreeUtil(inorder, preorder, 0, len - 1, &preIndex);
}
```

### Chapitre 8 : De l'Abstrait au Concret : Vers des Structures de Données Robustes

#### 8.1. Le concept de Type de Données Abstrait (TDA)

Pour conclure cette leçon, il est essentiel de prendre du recul et de formaliser un principe que nous avons appliqué intuitivement : le concept de **Type de Données Abstrait (TDA)**, ou Abstract Data Type (ADT) en anglais. 

Un TDA est un modèle mathématique pour une classe d'objets de données qui partagent un même comportement. Il est défini par un ensemble de valeurs possibles et un ensemble d'opérations sur ces valeurs, **indépendamment de leur implémentation concrète**⁴.

Par exemple, une "Pile" est un TDA défini par les opérations `push`, `pop`, `peek`, `isEmpty`. Que cette pile soit implémentée avec un tableau ou une liste chaînée est un détail d'implémentation qui doit être caché à l'utilisateur du TDA. 

De même, notre "Arbre Binaire de Recherche" est un TDA défini par les opérations `insert`, `delete`, `search`. L'utilisateur n'a pas besoin de savoir comment les pointeurs `left`, `right` et `parent` sont manipulés en interne.

Cette distinction entre l'**interface** (le "quoi") et l'**implémentation** (le "comment") est au cœur du génie logiciel moderne. Elle favorise la modularité, la réutilisabilité et la robustesse du code. C'est la raison pour laquelle, en pratique, on sépare les déclarations dans des fichiers d'en-tête (`.h`) de leurs implémentations dans des fichiers sources (`.c`).

#### 8.2. Le Problème des Cycles et la Corruption de Données

Les dernières remarques du séminaire source soulèvent un point crucial de robustesse⁷. Que se passe-t-il si un utilisateur de notre structure `TreeNode` écrit par erreur un code comme `mon_noeud->left = mon_noeud;` ? Il vient de créer un cycle dans la structure de données. 

Nos algorithmes de parcours, qui supposent une structure acyclique, entreraient alors dans une boucle infinie, menant à un crash.

Ce risque existe parce que nous avons exposé les détails internes de notre structure (les pointeurs `left` et `right`). L'utilisateur peut manipuler directement la structure et, par inadvertance, violer ses invariants fondamentaux (propriété ABR, acyclicité).

#### 8.3. La Solution : Encapsulation et Interfaces Sûres

La solution, dictée par le principe du TDA, est l'**encapsulation**. Une implémentation robuste d'un arbre ne devrait pas exposer la structure `TreeNode` directement. Au lieu de cela, elle fournirait une structure opaque (par exemple, `typedef struct Tree* TreeHandle;`) et un ensemble de fonctions d'interface sécurisées :

```c
TreeHandle create_tree();
void tree_insert(TreeHandle tree, int data);
int tree_search(TreeHandle tree, int data);
void free_tree(TreeHandle tree);
```

Dans cette approche, l'utilisateur ne manipule jamais directement les pointeurs. C'est la responsabilité de l'implémentation de la bibliothèque de garantir que chaque opération préserve tous les invariants. 

C'est ainsi que l'on construit des composants logiciels fiables et réutilisables. Cette prise de conscience transforme notre exploration des algorithmes et des structures de données en une véritable leçon sur les principes de la programmation de systèmes complexes.

## Conclusion

### Résumé des Apprentissages Clés

Au cours de cette leçon, nous avons entrepris un voyage complet à travers le monde des structures de données arborescentes. Partant de définitions mathématiques formelles, nous avons distingué l'arbre général de l'arbre binaire, en soulignant le rôle fondamental de l'ordre. Nous avons traduit ces concepts abstraits en implémentations concrètes en C, en construisant non seulement les arbres eux-mêmes, mais aussi les outils auxiliaires nécessaires comme les piles.

Nous avons exploré en profondeur les algorithmes de parcours, en démontrant l'équivalence conceptuelle entre les approches récursives élégantes et les approches itératives robustes. Nous avons ensuite introduit l'invariant de l'arbre binaire de recherche, la pierre angulaire qui permet des opérations de recherche, d'insertion et de suppression d'une efficacité logarithmique.

Enfin, nous avons abordé des sujets avancés qui révèlent la beauté de l'informatique théorique : la capacité de représenter n'importe quel arbre via un arbre binaire grâce à la technique "fils gauche, frère droit", et la possibilité de reconstruire un arbre unique à partir de ses parcours. Nous avons conclu en reliant ces implémentations pratiques au principe fondamental des types de données abstraits, une leçon essentielle pour la construction de logiciels fiables.

### Ouverture

Notre étude, bien que détaillée, n'est qu'une porte d'entrée. Le problème de la dégénérescence des ABR ouvre la voie à l'étude des arbres auto-équilibrés (AVL, rouge-noir, B-arbres), qui sont omniprésents dans les bases de données et les systèmes de fichiers¹². 

Les arbres sont également à la base d'autres structures de données avancées comme les tas (heaps) pour les files de priorité, ou les tries pour la recherche de chaînes de caractères. Les concepts que nous avons explorés - invariants, récursivité, abstraction - sont universels et vous serviront de fondation solide pour aborder n'importe quel sujet avancé en algorithmique et en structures de données.

## Références Bibliographiques

1. Aho, A. V., Hopcroft, J. E., & Ullman, J. D. (1983). *Data Structures and Algorithms*. Addison-Wesley.

2. Cormen, T. H., Leiserson, C. E., Rivest, R. L., & Stein, C. (2009). *Introduction to Algorithms* (3rd ed.). MIT Press.

3. Wirth, N. (1976). *Algorithms + Data Structures = Programs*. Prentice-Hall.

4. Knuth, D. E. (1997). *The Art of Computer Programming, Volume 1: Fundamental Algorithms* (3rd ed.). Addison-Wesley Professional.

5. Valiente, G. (2002). *Algorithms on Trees and Graphs*. Springer.

6. Aho, A. V., Hopcroft, J. E., & Ullman, J. D. (1974). *The Design and Analysis of Computer Algorithms*. Addison-Wesley.

7. [Séminaire source - référence non spécifiée dans le document original]

8. [Algorithme AHU - référence détaillée non disponible dans le document source]

9. [Références 9-29 non détaillées dans le document source]
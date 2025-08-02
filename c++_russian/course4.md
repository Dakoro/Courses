# Leçon : Arbres de Recherche et Sémantique de Copie en C++

## Introduction

Cette leçon couvre les arbres de recherche binaires, leur équilibrage, et la sémantique de copie en C++. Nous verrons comment implémenter correctement un conteneur basé sur un arbre avec tous les constructeurs et opérateurs nécessaires.

## 1. Les Arbres de Recherche Binaires

### 1.1 Définition et Propriétés

Un **arbre de recherche binaire** (BST - Binary Search Tree) est une structure de données avec la propriété suivante :
- Pour chaque nœud, tous les éléments du sous-arbre gauche sont **inférieurs** à l'élément du nœud
- Tous les éléments du sous-arbre droit sont **supérieurs** à l'élément du nœud

**Avantages** :
- Recherche efficace en O(log n) dans le meilleur cas
- Support des requêtes par intervalle (range queries)
- Maintien de l'ordre des éléments

### 1.2 Problème de Déséquilibre

Un arbre de recherche peut dégénérer en liste chaînée, donnant une complexité O(n) pour les opérations :

```
    10
      \
      20
        \
        30
          \
          40  // Arbre dégénéré - complexité O(n)
```

### 1.3 Solution avec `std::set`

La bibliothèque standard C++ fournit `std::set` qui est généralement implémenté comme un arbre rouge-noir équilibré.

**Exemple d'utilisation pour les requêtes par intervalle** :

```cpp
#include <set>
#include <iostream>

// Compter les éléments dans l'intervalle [first, last)
template<typename T>
size_t count_in_range(const std::set<T>& s, T first, T last) {
    auto start = s.lower_bound(first);  // Premier élément >= first
    auto end = s.upper_bound(last);     // Premier élément > last
    return std::distance(start, end);
}
```

**Méthodes importantes** :
- `lower_bound(x)` : retourne un itérateur vers le premier élément ≥ x
- `upper_bound(x)` : retourne un itérateur vers le premier élément > x

## 2. Arbres Équilibrés

### 2.1 Rotations

Les rotations sont la base de l'équilibrage. Elles préservent la propriété d'arbre de recherche :

```
    y                x
   / \              / \
  x   C    -->     A   y
 / \                  / \
A   B                B   C
```

### 2.2 Arbre AVL

**Invariant AVL** : Pour chaque nœud, la différence de hauteur entre les sous-arbres gauche et droit est au plus 1.

```cpp
struct AVLNode {
    Key key;
    AVLNode* left = nullptr;
    AVLNode* right = nullptr;
    int height = 0;  // Hauteur du sous-arbre
};
```

### 2.3 Arbre Rouge-Noir

**Invariants Rouge-Noir** :
1. La racine est noire
2. Les feuilles NULL sont noires
3. Les enfants d'un nœud rouge sont noirs
4. Tous les chemins de la racine aux feuilles contiennent le même nombre de nœuds noirs

## 3. Conception d'un Conteneur Arbre

### 3.1 Structure de Base

```cpp
template<typename Key, typename Compare = std::less<Key>>
class Tree {
public:
    using iterator = Node*;
    
    // Sélecteurs (const)
    size_t size() const;
    iterator find(const Key& k) const;
    
    // Modificateurs
    void insert(const Key& k);
    void erase(const Key& k);
    
private:
    Node* root_ = nullptr;
};
```

### 3.2 Structure du Nœud

**Problème avec l'initialisation agrégée** :

```cpp
// Version simple mais limitée
struct Node {
    Key key;
    Node* left;
    Node* right;
    int height;  // Pour AVL
};

// Problème : l'ajout d'un membre privé casse l'initialisation agrégée
struct Node {
    Key key;
    Node* left;
    Node* right;
private:
    int height;  // ERREUR : plus d'initialisation agrégée possible !
};
```

### 3.3 Solution avec Constructeur

```cpp
struct Node {
    Key key;
    Node* left = nullptr;
    Node* right = nullptr;
    int height = 0;
    
    // Constructeur explicite
    explicit Node(const Key& k) 
        : key(k), left(nullptr), right(nullptr), height(0) {}
};
```

## 4. Listes d'Initialisation

### 4.1 Importance des Listes d'Initialisation

**Problème sans liste d'initialisation** :

```cpp
struct S {
    S() { std::cout << "default\n"; }
    S(int) { std::cout << "direct\n"; }
};

struct Node {
    S s;
    
    // MAUVAIS : double initialisation
    Node(int x) {
        s = S(x);  // 1. "default" puis 2. "direct"
    }
    
    // BON : initialisation directe
    Node(int x) : s(x) {  // Seulement "direct"
    }
};
```

### 4.2 Règles Importantes

1. **Ordre d'initialisation** : Les membres sont initialisés dans l'ordre de leur déclaration dans la classe, PAS dans l'ordre de la liste d'initialisation

```cpp
struct Bad {
    int a, b;
    Bad() : b(42), a(b) {}  // DANGER : a initialisé avant b !
};
```

2. **Initialiseurs par défaut** : Sont utilisés si non spécifiés dans la liste

```cpp
struct Good {
    int x = 1;  // Initialiseur par défaut
    int y;
    
    Good() : y(2) {}      // x=1 (par défaut), y=2
    Good(int v) : x(v) {} // x=v, y=0 (non initialisé)
};
```

### 4.3 Constructeurs Délégués (C++11)

```cpp
struct Node {
    Key key;
    Node *left, *right;
    
    Node(const Key& k) : key(k), left(nullptr), right(nullptr) {
        // Logique complexe...
    }
    
    // Délègue au constructeur principal
    Node() : Node(Key{}) {}
    Node(const Key& k, Node* l, Node* r) : Node(k) {
        left = l;
        right = r;
    }
};
```

## 5. Destructeurs

### 5.1 Problèmes avec la Récursion

**MAUVAIS : Récursion qui peut déborder la pile** :

```cpp
~Node() {
    delete left;   // Appel récursif du destructeur
    delete right;  // Appel récursif du destructeur
}
```

**Problèmes** :
1. Débordement de pile pour les arbres profonds
2. Suppose que les nœuds sont alloués avec `new`

### 5.2 Solution Itérative

Le destructeur devrait être géré au niveau de l'arbre, pas du nœud :

```cpp
class Tree {
    ~Tree() {
        // Parcours itératif post-ordre
        if (!root_) return;
        
        std::stack<std::pair<Node*, bool>> stack;
        stack.push({root_, false});
        
        while (!stack.empty()) {
            auto& [node, visited] = stack.top();
            if (visited) {
                stack.pop();
                delete node;
            } else {
                visited = true;
                if (node->right) stack.push({node->right, false});
                if (node->left) stack.push({node->left, false});
            }
        }
    }
};
```

### 5.3 Note Importante

**Ne jamais réinitialiser les membres dans le destructeur** :

```cpp
~Node() {
    delete ptr;
    ptr = nullptr;  // INUTILE : l'objet est détruit après
}
```

Le compilateur optimise et supprime ces affectations car l'objet n'existe plus après le destructeur.

## 6. Sémantique de Copie

### 6.1 Constructeur de Copie

**Problème** : La copie par défaut copie les pointeurs, pas les données pointées.

```cpp
class Tree {
    Node* root_;
    
public:
    // Copie profonde nécessaire
    Tree(const Tree& other) : root_(copy_tree(other.root_)) {}
    
private:
    static Node* copy_tree(const Node* node) {
        if (!node) return nullptr;
        
        Node* new_node = new Node(node->key);
        new_node->left = copy_tree(node->left);
        new_node->right = copy_tree(node->right);
        new_node->height = node->height;
        return new_node;
    }
};
```

### 6.2 Opérateur d'Assignation

**Règle importante** : Toujours vérifier l'auto-assignation !

```cpp
Tree& operator=(const Tree& other) {
    // Protection contre l'auto-assignation
    if (this == &other) return *this;
    
    // Libérer l'ancienne mémoire
    delete_tree(root_);
    
    // Copier le nouvel arbre
    root_ = copy_tree(other.root_);
    
    return *this;
}
```

### 6.3 Règle des Trois/Cinq

Si vous définissez l'un de ces éléments, vous devez probablement définir tous :
1. Destructeur
2. Constructeur de copie
3. Opérateur d'assignation
4. (C++11) Constructeur de déplacement
5. (C++11) Opérateur d'assignation par déplacement

**Alternative : Interdire la copie** :

```cpp
class Tree {
    Tree(const Tree&) = delete;
    Tree& operator=(const Tree&) = delete;
};
```

## 7. Initialisation : Value vs Default

### 7.1 Types de Base

```cpp
int a;      // Non initialisé (valeur indéterminée)
int b{};    // Value-init : zéro
int c = 0;  // Copy-init : zéro

int* arr1 = new int[5];    // Non initialisé
int* arr2 = new int[5]{};  // Tout à zéro (value-init)
int* arr3 = new int[5]();  // Tout à zéro (value-init)
```

### 7.2 Classes avec Constructeurs

Pour les classes avec constructeurs définis par l'utilisateur :
- `T t;` et `T t{};` appellent le constructeur par défaut
- Pas de différence entre default-init et value-init

## 8. Sémantique Spéciale et RVO

### 8.1 Return Value Optimization (RVO)

Le compilateur peut éliminer les copies même avec des effets de bord :

```cpp
struct S {
    S() { std::cout << "ctor\n"; }
    S(const S&) { std::cout << "copy\n"; }
    ~S() { std::cout << "dtor\n"; }
};

S make_s() {
    S local;
    return local;  // Pas de copie !
}

int main() {
    S s = make_s();  // Sortie : "ctor" "dtor" (pas de "copy")
}
```

### 8.2 Conditions pour être un Constructeur de Copie

Le compilateur reconnaît ces formes :
- `T(const T&)`
- `T(T&)`
- `T(const volatile T&)`
- `T(volatile T&)`

**Important** : Un constructeur template n'est JAMAIS un constructeur de copie !

```cpp
template<typename T>
struct S {
    S() = default;
    
    template<typename U>
    S(const S<U>&) {}  // PAS un constructeur de copie !
};
```

## 9. Qualificateurs CV

### 9.1 Signification

- **`const`** : Ne peut pas être modifié
- **`volatile`** : Peut changer de manière imprévisible
- **`const volatile`** : Ne peut pas être modifié mais peut changer tout seul

### 9.2 Méthodes const

```cpp
class Tree {
    size_t size() const;  // Peut être appelée sur un objet const
    void insert(Key k);   // Ne peut PAS être appelée sur un objet const
};
```

**Note** : La bibliothèque standard n'a pratiquement aucune méthode `volatile`.

## 10. Le Mot-clé `explicit`

### 10.1 Conversions Implicites

Sans `explicit`, un constructeur à un paramètre permet les conversions :

```cpp
struct String {
    String(int size);  // Alloue 'size' caractères
};

void f(const String& s);
f(42);  // DANGER : Alloue un string de 42 caractères !
```

### 10.2 Bloquer les Conversions

```cpp
struct String {
    explicit String(int size);
};

f(42);           // ERREUR : pas de conversion implicite
f(String(42));   // OK : conversion explicite
```

### 10.3 Différence entre Initialisation Directe et par Copie

```cpp
String s1(42);    // OK : initialisation directe
String s2 = 42;   // ERREUR avec explicit : initialisation par copie
```

## 11. Opérateurs de Conversion

### 11.1 Conversion vers un Autre Type

```cpp
class Rational {
    int num_, den_;
public:
    operator double() const {
        return double(num_) / den_;
    }
    
    explicit operator bool() const {
        return num_ != 0;
    }
};
```

### 11.2 Résolution de Surcharge avec Conversions

**Hiérarchie** (du meilleur au pire) :
1. Correspondance exacte
2. Conversions standard (int → long)
3. Conversions utilisateur
4. Fonctions variadiques (...)

**Règle importante** : Une chaîne de conversion peut contenir AU PLUS une conversion utilisateur.

## 12. Exercice : Arbre avec Statistiques d'Ordre

### 12.1 Objectif

Implémenter un arbre supportant :
1. **k-ième plus petit élément** : Trouver le k-ième élément dans l'ordre croissant
2. **Compter les éléments < n** : Nombre d'éléments strictement inférieurs à n

### 12.2 Idée de Solution

Augmenter chaque nœud avec le nombre d'éléments dans son sous-arbre :

```cpp
struct Node {
    Key key;
    Node *left, *right;
    size_t subtree_size = 1;  // Taille du sous-arbre
    
    void update_size() {
        subtree_size = 1;
        if (left) subtree_size += left->subtree_size;
        if (right) subtree_size += right->subtree_size;
    }
};
```

### 12.3 Recherche du k-ième Élément

```cpp
Node* find_kth(Node* node, size_t k) {
    if (!node) return nullptr;
    
    size_t left_size = node->left ? node->left->subtree_size : 0;
    
    if (k <= left_size) {
        return find_kth(node->left, k);
    } else if (k == left_size + 1) {
        return node;
    } else {
        return find_kth(node->right, k - left_size - 1);
    }
}
```

### 12.4 Exigences

1. Implémenter un arbre équilibré (AVL ou autre)
2. Fournir constructeur de copie et opérateur d'assignation corrects
3. Destructeur itératif (pas récursif)
4. Complexité O(log n) pour toutes les opérations

## Conclusion

Cette leçon a couvert les aspects essentiels de l'implémentation d'un conteneur arbre en C++ moderne. Les points clés sont :

- L'importance de l'équilibrage pour maintenir les performances
- La nécessité d'une sémantique de copie correcte pour les types qui gèrent des ressources
- Les subtilités de l'initialisation et des conversions en C++
- L'optimisation RVO qui élimine les copies inutiles

La maîtrise de ces concepts est essentielle pour écrire du code C++ robuste et performant.
# Leçon : Héritage et Polymorphisme en C++

## Introduction

Cette leçon couvre les concepts fondamentaux de la programmation orientée objet en C++, en particulier l'héritage et le polymorphisme. Ces techniques sont essentielles pour créer des hiérarchies de classes flexibles et réutilisables.

## 1. Le Problème Initial : Arbres Syntaxiques

### Contexte : Le langage ParaCL

ParaCL est un langage de programmation éducatif créé en 2011 qui combine les meilleures caractéristiques de plusieurs langages :
- Affectation (`=`)
- Lecture depuis le clavier (`?`)
- Impression (`print`)
- Structures de contrôle (`if`, `while`)

### Le Défi : Représenter Différents Types de Nœuds

Lors de la création d'un compilateur, nous devons construire un arbre syntaxique abstrait (AST). Chaque nœud peut être de type différent :
- Opérations binaires (ex: `a + b`)
- Opérations unaires (ex: `-a`)
- Boucles (`while`)
- Conditions (`if`)
- Blocs de code

## 2. Solutions Sans Héritage

### Solution 1 : Union (Déconseillée)

```cpp
struct Node {
    Node* parent;
    NodeType type;
    union Data {
        BinOp binop;
        UnOp unop;
        Scope scope;
        // ...
    } data;
    std::vector<Node*> children;
};
```

**Problèmes avec les unions en C++ :**
- Les unions avec des types ayant des constructeurs sont dangereuses
- Gestion manuelle des constructeurs/destructeurs
- Code fragile et difficile à maintenir

### Solution 2 : Pointeurs void* (Également déconseillée)

```cpp
struct Node {
    NodeType type;
    void* data;
};
```

Cette approche nécessite des casts constants et perd la sécurité des types.

## 3. La Solution : L'Héritage

### Concept de Base

L'héritage permet de créer une hiérarchie de classes où les classes dérivées héritent des propriétés de la classe de base :

```cpp
class Node {
    Node* parent;
    NodeType type;
public:
    Node(Node* p, NodeType t) : parent(p), type(t) {}
};

class BinOp : public Node {
    Op op;
    Node* left;
    Node* right;
public:
    BinOp(Node* p, Op o) : Node(p, BIN_OP), op(o) {}
};
```

### Héritage Public

L'héritage public modélise la relation "est-un" :
- `BinOp` **est un** `Node`
- Tous les membres publics de `Node` restent publics dans `BinOp`

## 4. Fonctions Virtuelles et Polymorphisme

### Le Problème

Comment appeler la bonne méthode sur un objet dont on ne connaît que le type de base ?

```cpp
class Shape {
public:
    virtual double area() const = 0;  // Fonction virtuelle pure
    virtual ~Shape() {}               // Destructeur virtuel
};

class Triangle : public Shape {
public:
    double area() const override {
        // Calcul spécifique au triangle
    }
};

class Polygon : public Shape {
public:
    double area() const override {
        // Calcul spécifique au polygone
    }
};
```

### Points Clés sur les Fonctions Virtuelles

1. **Le mot-clé `virtual`** indique que la fonction peut être redéfinie dans les classes dérivées
2. **Le mot-clé `override`** (C++11) garantit qu'on redéfinit bien une fonction virtuelle existante
3. **`= 0`** crée une fonction virtuelle pure (classe abstraite)

## 5. Le Destructeur Virtuel : Règle Fondamentale

> **Règle d'or :** Un destructeur virtuel est nécessaire pour détruire correctement des objets de classes dérivées via un pointeur sur la classe de base.

```cpp
Shape* s = new Triangle();
delete s;  // Sans destructeur virtuel : fuite mémoire !
```

## 6. Les Quatre Façons Correctes d'Écrire une Classe

Une classe C++ est correctement écrite si elle respecte l'une de ces conditions :

1. **Elle contient un destructeur virtuel** (si on peut hériter d'elle)
2. **Elle est marquée `final`** (interdiction d'héritage)
3. **Elle est une classe vide (stateless)** utilisée uniquement pour ses méthodes
4. **Son destructeur est `protected`** (empêche la destruction via la classe de base)

## 7. Principe de Substitution de Liskov (LSP)

Ce principe stipule que tout prédicat vrai pour la classe de base doit rester vrai pour la classe dérivée.

### Exemple Classique : Rectangle vs Carré

```cpp
// MAUVAIS DESIGN
class Rectangle {
    double width, height;
public:
    void setWidth(double w) { width = w; }
    void setHeight(double h) { height = h; }
};

class Square : public Rectangle {
    // Problème : un carré ne peut pas avoir des dimensions différentes !
};
```

## 8. Liaison Statique vs Dynamique

### Liaison Dynamique
- Les fonctions virtuelles sont résolues à l'exécution
- Basée sur le type réel de l'objet

### Liaison Statique
- Les fonctions non-virtuelles sont résolues à la compilation
- Les arguments par défaut sont **toujours** liés statiquement !

```cpp
class Base {
public:
    virtual void foo(int x = 10) { std::cout << x; }
};

class Derived : public Base {
public:
    void foo(int x = 20) override { std::cout << x; }
};

Base* b = new Derived();
b->foo();  // Affiche 10, pas 20 !
```

## 9. Héritage Privé

L'héritage privé modélise la relation "est-implémenté-en-termes-de" :

```cpp
class Engine {
    void start();
    void stop();
};

class Car : private Engine {
    // Engine fait partie de l'implémentation de Car
    // mais Car n'est pas un Engine
};
```

## 10. Optimisation des Classes de Base Vides (EBO)

Lorsqu'une classe de base est vide, le compilateur peut optimiser l'espace :

```cpp
struct Empty {};
struct Derived : Empty {
    int x;
};
// sizeof(Derived) == sizeof(int), pas sizeof(int) + 1
```

## Exercice Pratique

Implémenter un simulateur pour le langage ParaCL en utilisant une hiérarchie de classes avec héritage et fonctions virtuelles. Le simulateur doit :
1. Parser le code source
2. Construire un AST
3. Évaluer l'arbre pour exécuter le programme

## Points à Retenir

1. **Toujours utiliser `override`** pour les fonctions virtuelles dans les classes dérivées
2. **Toujours avoir un destructeur virtuel** si la classe peut être héritée
3. **Éviter les arguments par défaut** avec les fonctions virtuelles
4. **Respecter le principe de substitution de Liskov**
5. **Utiliser `final`** quand l'héritage n'est pas souhaité
6. **Préférer la composition** à l'héritage privé quand possible
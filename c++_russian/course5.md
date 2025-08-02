# Leçon : Gestion des Ressources et Sémantique de Déplacement en C++

## Introduction

Cette leçon couvre la gestion des ressources en C++, introduit le concept RAII (Resource Acquisition Is Initialization) et présente la sémantique de déplacement apparue en C++11. Ces concepts sont fondamentaux pour écrire du code C++ moderne, efficace et sûr.

## 1. Possession de Ressources

### 1.1 Qu'est-ce que la Possession ?

**Définition** : Posséder une ressource signifie être responsable de son allocation ET de sa libération.

**Exemples de ressources** :
- Mémoire dynamique
- Fichiers
- Connexions réseau
- Mutex/verrous

**Principe fondamental** : Celui qui alloue la ressource doit la libérer. Si plusieurs entités peuvent libérer une ressource, cela mène à des problèmes (double suppression, fuites mémoire).

### 1.2 Problème en C : goto pour la gestion des ressources

En C, la gestion des ressources avec multiples points de sortie nécessite souvent `goto` :

```c
void foo() {
    char* p = malloc(256);
    if (condition1) goto cleanup;
    if (condition2) goto cleanup;
    // ... utilisation de p
cleanup:
    free(p);
}
```

**Problème avec goto** :
- Transforme le code structuré en graphe arbitraire
- Difficile à analyser et maintenir
- En C++, `goto` ne peut pas traverser l'initialisation d'objets

## 2. RAII (Resource Acquisition Is Initialization)

### 2.1 Solution C++ : Les Destructeurs

Au lieu d'utiliser `goto`, C++ utilise les destructeurs pour garantir la libération des ressources :

```cpp
template<typename T>
class scoped_ptr {
    T* ptr_;
    
public:
    explicit scoped_ptr(T* p) : ptr_(p) {}
    
    ~scoped_ptr() {
        delete ptr_;
    }
    
    // Accès aux données
    T& operator*() { return *ptr_; }
    T* operator->() { return ptr_; }
};
```

**Avantages** :
- La ressource est automatiquement libérée en fin de portée
- Pas besoin de `goto` ou de code de nettoyage manuel
- Exception-safe par conception

### 2.2 Problème : Copie Superficielle

Le constructeur de copie par défaut fait une copie superficielle (shallow copy) :

```cpp
scoped_ptr<int> p1(new int(42));
scoped_ptr<int> p2 = p1;  // DANGER : double delete !
```

**Solutions possibles** :
1. Interdire la copie (delete copy constructor)
2. Implémenter une copie profonde (deep copy)
3. Utiliser un compteur de références (shared ownership)

### 2.3 Implémentation de la Copie Profonde

```cpp
template<typename T>
class scoped_ptr {
    T* ptr_;
    
public:
    // Constructeur de copie avec copie profonde
    scoped_ptr(const scoped_ptr& other) 
        : ptr_(other.ptr_ ? new T(*other.ptr_) : nullptr) {}
    
    // Opérateur d'assignation
    scoped_ptr& operator=(const scoped_ptr& other) {
        if (this != &other) {  // Protection contre l'auto-assignation
            delete ptr_;
            ptr_ = other.ptr_ ? new T(*other.ptr_) : nullptr;
        }
        return *this;
    }
};
```

**Important** : Toujours vérifier l'auto-assignation (`a = a`) !

## 3. Surcharge des Opérateurs pour l'Interface Pointeur

### 3.1 Opérateur de Déréférencement

```cpp
T& operator*() { return *ptr_; }
const T& operator*() const { return *ptr_; }
```

### 3.2 Opérateur Flèche (->)

**Particularité** : L'opérateur `->` a un comportement "drill-down" - il s'applique récursivement jusqu'à atteindre un vrai pointeur.

```cpp
T* operator->() { return ptr_; }
const T* operator->() const { return ptr_; }
```

**Exemple** :
```cpp
scoped_ptr<Point> p(new Point{1, 2});
p->x;  // Équivalent à p.operator->()->x
```

## 4. Sémantique de Déplacement (C++11)

### 4.1 Motivation

Problème avec la copie profonde :
```cpp
scoped_ptr<T> a(...);
scoped_ptr<T> b(...);
std::swap(a, b);  // 3 allocations + 3 libérations !
```

Pour un simple échange de pointeurs, c'est très inefficace.

### 4.2 Références Rvalue (&&)

C++11 introduit les **références rvalue** pour distinguer :
- Les objets dont on a besoin (lvalues)
- Les objets temporaires qu'on peut "piller" (rvalues)

```cpp
int x = 42;
int& ref1 = x;      // OK : référence lvalue
int&& ref2 = x + 1; // OK : référence rvalue sur temporaire
int&& ref3 = x;     // ERREUR : x est une lvalue
```

### 4.3 std::move

`std::move` convertit une lvalue en rvalue :

```cpp
int&& ref = std::move(x);  // OK : traite x comme une rvalue
```

**Important** : `std::move` ne déplace rien ! Il fait juste un cast.

### 4.4 Constructeur et Opérateur de Déplacement

```cpp
template<typename T>
class scoped_ptr {
    T* ptr_;
    
public:
    // Constructeur de déplacement
    scoped_ptr(scoped_ptr&& other) noexcept 
        : ptr_(other.ptr_) {
        other.ptr_ = nullptr;  // Voler la ressource
    }
    
    // Opérateur d'assignation par déplacement
    scoped_ptr& operator=(scoped_ptr&& other) noexcept {
        if (this != &other) {
            delete ptr_;
            ptr_ = other.ptr_;
            other.ptr_ = nullptr;
        }
        return *this;
    }
};
```

**Alternative pour l'opérateur d'assignation** :
```cpp
scoped_ptr& operator=(scoped_ptr&& other) noexcept {
    std::swap(ptr_, other.ptr_);
    return *this;
    // L'ancien ptr_ sera détruit avec other
}
```

### 4.5 État Après Déplacement

Après un déplacement, l'objet source est dans un état :
- **Valide** : les invariants de classe sont respectés
- **Non spécifié** : on ne peut pas faire d'hypothèses sur ses valeurs

On peut :
- Le détruire
- Lui assigner une nouvelle valeur

On ne devrait pas :
- Utiliser sa valeur
- Appeler ses méthodes (sauf destructeur et assignation)

## 5. Surcharge de Méthodes sur la Catégorie de Valeur

### 5.1 Qualificateurs & et &&

Pour éviter les références pendantes :

```cpp
class Container {
    int n_;
public:
    int& access() & { return n_; }        // Pour lvalues seulement
    const int& access() const& { return n_; }  // Pour const lvalues
    int&& access() && { return std::move(n_); } // Pour rvalues
};
```

**Utilisation** :
```cpp
Container c;
c.access();        // OK : appelle la version &
Container().access(); // ERREUR : pas de version && définie
```

### 5.2 Retour de Références Rvalue

**Règle** : Ne retournez des références rvalue que dans :
1. Les méthodes qualifiées &&
2. Les fonctions comme `std::move`, `std::forward`
3. Jamais ailleurs !

## 6. Règles de Gestion des Ressources

### 6.1 Règle des Cinq (Rule of Five)

Si votre classe définit l'un de ces éléments, elle doit définir les cinq :
1. Destructeur
2. Constructeur de copie
3. Opérateur d'assignation par copie
4. Constructeur de déplacement
5. Opérateur d'assignation par déplacement

```cpp
class Resource {
public:
    ~Resource();
    Resource(const Resource&);
    Resource& operator=(const Resource&);
    Resource(Resource&&) noexcept;
    Resource& operator=(Resource&&) noexcept;
};
```

### 6.2 Règle de Zéro (Rule of Zero)

**Préférable** : Si votre classe n'a pas besoin de gérer directement des ressources, ne définissez aucune de ces fonctions spéciales.

```cpp
class BusinessLogic {
    std::vector<int> data_;      // Gère sa propre mémoire
    std::string name_;           // Gère sa propre mémoire
    std::unique_ptr<T> resource_; // Gère sa propre ressource
    // Pas de destructeur/copie/déplacement personnalisés nécessaires !
};
```

### 6.3 Principe de Responsabilité Unique

Une classe devrait soit :
- Gérer une ressource (et implémenter les 5)
- Faire de la logique métier (et suivre la règle de 0)

Jamais les deux !

## 7. Optimisations et Pièges

### 7.1 Ne pas Abuser de std::move

**Mauvais** :
```cpp
T foo() {
    T x;
    return std::move(x);  // Empêche RVO !
}
```

**Bon** :
```cpp
T foo() {
    T x;
    return x;  // RVO possible
}
```

### 7.2 Références Const Rvalue

```cpp
const T&& ref = std::move(obj);  // Rarement utile
```

Les références const rvalue se comportent comme des références const lvalue et empêchent le déplacement.

## 8. Tableaux Multidimensionnels

### 8.1 Row-Major vs Column-Major

C++ utilise le **row-major order** :
```cpp
int arr[2][3];  // 2 lignes, 3 colonnes
// Disposition en mémoire : arr[0][0], arr[0][1], arr[0][2], arr[1][0], arr[1][1], arr[1][2]
```

### 8.2 Différence entre Tableaux et Pointeurs

```cpp
int arr[10][20];      // Vrai tableau 2D
int* ptr[10];         // Tableau de pointeurs
int (*ptr)[20];       // Pointeur vers tableau
int** ptr;            // Pointeur vers pointeur
```

**Important** : L'arithmétique des pointeurs diffère selon le type !

### 8.3 Passage en Paramètre

```cpp
void f(int arr[][20]);    // OK : taille des colonnes requise
void f(int arr[10][20]);  // OK : première dimension ignorée
void f(int arr[][]);      // ERREUR : taille des colonnes manquante
```

## 9. Exercice : Classe Matrix

### 9.1 Objectif

Implémenter une classe `Matrix<T>` générique avec :
- Gestion manuelle de la mémoire (pas de `std::vector`)
- Support des opérations de copie et déplacement
- Calcul du déterminant

### 9.2 Représentations Possibles

1. **Tableau continu** :
```cpp
T* data_;  // Taille rows_ * cols_
// Accès : data_[row * cols_ + col]
```

2. **Tableau de pointeurs** :
```cpp
T** data_;  // Tableau de lignes
// Accès : data_[row][col]
```

### 9.3 Exigences

1. Respecter la règle des 5
2. Supporter les matrices 100×100 efficacement
3. Générer des matrices de test avec déterminant connu
4. Pas de récursion dans le destructeur

## Conclusion

La gestion des ressources en C++ moderne repose sur :
- **RAII** : lier la durée de vie des ressources aux objets
- **Sémantique de déplacement** : éviter les copies inutiles
- **Règles claires** : Rule of Five ou Rule of Zero

Ces concepts permettent d'écrire du code à la fois sûr et performant, en tirant parti des garanties du langage pour éviter les fuites de ressources et les erreurs de gestion mémoire.
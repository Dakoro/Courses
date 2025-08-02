# Leçon : Gestion des Erreurs et Exceptions en C++

## Introduction

Cette leçon couvre les mécanismes de gestion des erreurs en C++, en particulier le système d'exceptions. Nous examinerons pourquoi la gestion d'erreurs traditionnelle du C pose problème en C++ et comment les exceptions offrent une solution plus robuste.

## 1. Problèmes de la Gestion d'Erreurs en Style C

### Les Approches Traditionnelles

En C, trois méthodes principales existent pour signaler les erreurs :

1. **Retourner un code d'erreur**
```c
int openFile(FILE** handle, const char* filename) {
    // Retourne 0 en cas de succès, code d'erreur sinon
}
```
**Problème** : Personne ne vérifie les codes de retour !

2. **Variable globale errno**
```c
int result = atoi(str);
// Si conversion impossible, retourne 0
// Si overflow, errno = ERANGE
```
**Problème** : Qui vérifie errno après atoi() ?

3. **Paramètre de sortie pour l'erreur**
```c
FILE* openFile(const char* filename, int* error_code) {
    // ...
}
```
**Problème** : Code verbeux et facile à ignorer

### Problèmes Spécifiques au C++

#### Constructeurs
```cpp
class MyVector {
    T* data;
    size_t size;
    size_t capacity;
    
public:
    MyVector(size_t cap) 
        : size(0), capacity(cap), data(malloc(cap * sizeof(T))) {
        // Si malloc échoue, que faire ?
        // Un constructeur ne peut rien retourner !
    }
};
```

#### État Inconsistant
Si l'allocation échoue dans le constructeur :
- L'objet reste dans un état **inconsistant**
- `data` est NULL mais `capacity` indique une taille
- Violation de l'invariant de classe

#### Surcharge d'Opérateurs
```cpp
Matrix operator+(const Matrix& a, const Matrix& b) {
    // Impossible de retourner un code d'erreur
    // Où stocker l'état d'erreur ?
}
```

## 2. Introduction aux Exceptions

### Principe de Base

Les exceptions permettent de :
- Signaler une erreur sans valeur de retour
- Propager l'erreur jusqu'à un gestionnaire approprié
- Garantir l'appel des destructeurs (RAII)

### Syntaxe Fondamentale

```cpp
// Lancer une exception
throw std::runtime_error("Message d'erreur");

// Capturer une exception
try {
    // Code susceptible de lever une exception
} catch (const std::exception& e) {
    // Gestion de l'erreur
}
```

### Déroulement de la Pile (Stack Unwinding)

```cpp
struct S {
    S() { std::cout << "ctor\n"; }
    ~S() { std::cout << "dtor\n"; }
};

void f(int n) {
    S s1, s2, s3, s4, s5;
    if (n == 0) 
        throw 1;
}

int main() {
    try {
        f(0);
    } catch(int) {
        std::cout << "caught\n";
    }
}
// Sortie : 5 ctor, 5 dtor (dans l'ordre inverse), caught
```

## 3. Règles de Capture des Exceptions

### Types de Capture

1. **Par valeur** (déconseillé - provoque le slicing)
```cpp
catch (Base b) { }  // Mauvais : slicing si Derived est lancé
```

2. **Par référence** (recommandé)
```cpp
catch (const Base& b) { }  // Bon : pas de slicing
```

3. **Par pointeur** (rare)
```cpp
catch (Base* b) { }  // Nécessite throw new Derived()
```

### Ordre des Gestionnaires

Les gestionnaires sont testés dans l'ordre :
```cpp
try {
    // ...
} catch (const Derived& d) {
    // Doit venir avant Base
} catch (const Base& b) {
    // Plus général après
} catch (...) {
    // Capture tout - à utiliser avec précaution
}
```

## 4. Hiérarchie Standard des Exceptions

```
std::exception
├── std::logic_error
│   ├── std::invalid_argument
│   ├── std::domain_error
│   ├── std::length_error
│   └── std::out_of_range
└── std::runtime_error
    ├── std::range_error
    ├── std::overflow_error
    └── std::underflow_error
```

### Règle d'Or
**Toujours hériter de `std::exception`** pour vos propres exceptions :

```cpp
class MyError : public std::runtime_error {
public:
    MyError(const std::string& msg) 
        : std::runtime_error(msg) {}
};
```

## 5. Garanties de Sécurité des Exceptions

### Les Trois Niveaux

1. **Garantie de Base**
   - Pas de fuite de ressources
   - Objets dans un état valide (mais potentiellement modifié)

2. **Garantie Forte** (commit-or-rollback)
   - Si exception : état inchangé
   - Opération atomique

3. **Garantie nothrow**
   - Aucune exception possible
   - Marquée avec `noexcept`

### Exemple : Constructeur de Copie Sûr

**Version Dangereuse** :
```cpp
MyVector(const MyVector& other) 
    : size(other.size), capacity(other.capacity) {
    data = new T[capacity];
    for (size_t i = 0; i < size; ++i) {
        data[i] = other.data[i];  // Exception possible !
    }
}
```
**Problème** : Fuite mémoire si exception lors de la copie

**Version Sûre** :
```cpp
MyVector(const MyVector& other) 
    : size(0), capacity(0), data(nullptr) {
    MyVector temp(other.capacity);
    
    // Copie dans l'objet temporaire
    for (size_t i = 0; i < other.size; ++i) {
        temp.data[i] = other.data[i];
        ++temp.size;
    }
    
    // Échange sans exception (Idiome copy-and-swap)
    swap(temp);
}
```

## 6. L'Idiome Copy-and-Swap

Pour l'opérateur d'affectation avec garantie forte :

```cpp
MyVector& operator=(const MyVector& other) {
    MyVector temp(other);  // Peut lancer une exception
    swap(temp);           // noexcept
    return *this;
}  // temp.~MyVector() nettoie l'ancien état

void swap(MyVector& other) noexcept {
    std::swap(data, other.data);
    std::swap(size, other.size);
    std::swap(capacity, other.capacity);
}
```

### La Ligne de Kálba

Herb Sutter propose de tracer une "ligne" dans chaque fonction :
- **Au-dessus** : Peut modifier des temporaires, peut lancer des exceptions
- **En-dessous** : Modifie l'état de l'objet, ne doit pas lancer d'exceptions

## 7. Règles Cruciales

### Destructeurs et noexcept

**Règle absolue** : Les destructeurs ne doivent JAMAIS laisser échapper une exception

```cpp
~MyClass() noexcept {  // implicite même sans noexcept
    // Si une exception s'échappe : std::terminate()
}
```

### Neutralité vis-à-vis des Exceptions

Une fonction est **neutre** si elle laisse passer les exceptions sans les intercepter :

```cpp
void bad() {
    try {
        // ...
    } catch (...) {
        // Intercepte TOUTES les exceptions
        // Peut cacher des problèmes !
    }
}

void good() {
    try {
        // ...
    } catch (...) {
        cleanup();
        throw;  // Re-lance l'exception
    }
}
```

## 8. Pièges et Bonnes Pratiques

### Le Problème du Slicing

```cpp
try {
    throw Derived();
} catch (Base b) {      // Mauvais : slicing
    // Information perdue
} catch (const Base& b) { // Bon : pas de slicing
    // Polymorphisme préservé
}
```

### Function Try Blocks

Pour capturer des exceptions dans les listes d'initialisation :

```cpp
class MyClass {
    A a;
    B b;
public:
    MyClass(int x) 
    try : a(x), b(x) {
        // Corps du constructeur
    } catch (...) {
        // Gestion d'erreur
        // Note : l'exception sera re-lancée automatiquement
    }
};
```

### Coût des Exceptions

- **Sans exception** : Coût quasi-nul (zero-cost exceptions)
- **Avec exception** : Très coûteux (4-5 ordres de grandeur plus lent qu'un return)

**Conclusion** : Les exceptions sont pour les cas **exceptionnels**, pas pour le contrôle de flux normal.

## Points Clés à Retenir

1. **Les exceptions résolvent les problèmes** de la gestion d'erreurs en C
2. **Toujours capturer par référence const**
3. **Hériter de std::exception** pour vos exceptions
4. **Les destructeurs sont noexcept** par défaut
5. **Utiliser l'idiome copy-and-swap** pour la sécurité des exceptions
6. **Tracer la ligne de Kálba** dans vos fonctions
7. **Les exceptions sont pour les cas exceptionnels**

## Exercices Pratiques

1. Réécrire une classe Vector avec garantie forte pour toutes les opérations
2. Implémenter une hiérarchie d'exceptions pour un compilateur
3. Analyser le code existant pour identifier les violations de sécurité des exceptions

## Ressources Recommandées

- **Tom Cargill** : "Exception Handling: A False Sense of Security" (1994)
- **Dave Abrahams** : "Exception-Safety in Generic Components"
- **Herb Sutter** : "Exceptional C++" et "More Exceptional C++"
- **Jon Kalb** : Série de conférences "Exception-Safe Code"
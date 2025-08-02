# Leçon : Foncteurs, Performance et Surcharge d'Opérateurs en C++

## Introduction

Cette leçon explore la surcharge d'opérateurs en C++, en commençant par la motivation pratique (performance) avant d'aborder les aspects techniques. Nous verrons comment les objets fonctions permettent d'optimiser le code, puis nous étudierons en détail les différentes catégories d'opérateurs et leurs règles de surcharge.

## 1. Motivation : Performance avec les Foncteurs

### 1.1 Le Problème avec les Pointeurs de Fonctions

Comparons deux approches pour trier un tableau :

**Version C avec qsort** :
```c
// Utilise un pointeur de fonction
qsort(array, size, sizeof(int), less_function);
```

**Version C++ avec std::sort** :
```cpp
// Utilise l'opérateur < par défaut
std::sort(array, array + size);
```

**Résultat** : La version C++ est ~4x plus rapide ! Pourquoi ?

### 1.2 Pourquoi les Pointeurs de Fonctions sont Lents

```cpp
// Si on passe un prédicat à std::sort
bool greater(int a, int b) { return a > b; }
std::sort(array, array + size, greater);  // LENT !
```

**Problème** : Le compilateur ne peut pas inliner l'appel via pointeur de fonction.

### 1.3 Solution : Objets Fonctions (Foncteurs)

**Première approche - Conversion implicite** :
```cpp
struct MyLess {
    static bool cmp(int a, int b) { return a < b; }
    
    // Conversion implicite vers pointeur de fonction
    operator bool(*)(int, int)() const {
        return &cmp;
    }
};

std::sort(array, array + size, MyLess{});  // Rapide !
```

**Meilleure approche - Opérateur ()** :
```cpp
struct MyLess {
    bool operator()(int a, int b) const {
        return a < b;
    }
};

std::sort(array, array + size, MyLess{});  // Encore plus clair !
```

### 1.4 Lambdas : Syntaxe Moderne

```cpp
// Lambda = classe avec operator() généré automatiquement
std::sort(array, array + size, [](int a, int b) { return a < b; });
```

**Point clé** : Une lambda n'est PAS une fonction, c'est un objet avec `operator()`.

## 2. L'Idiome PImpl avec Deleters Personnalisés

### 2.1 Le Problème

```cpp
// header.h
class MyClass;  // Forward declaration

class Facade {
    MyClass* impl_;  // OK
    std::unique_ptr<MyClass> impl_;  // ERREUR !
};
```

**Pourquoi l'erreur ?** `unique_ptr` doit connaître la taille de `MyClass` pour `delete`.

### 2.2 Solution avec Deleter Personnalisé

```cpp
// header.h
class MyClass;

struct MyClassDeleter {
    void operator()(MyClass* p) const;  // Défini dans le .cpp
};

class Facade {
    std::unique_ptr<MyClass, MyClassDeleter> impl_;
};

// source.cpp
#include "MyClass.h"  // Définition complète
void MyClassDeleter::operator()(MyClass* p) const {
    delete p;
}
```

**Avantage** : La taille de `MyClass` n'est connue que dans le .cpp.

### 2.3 Note sur unique_ptr

`unique_ptr` a deux paramètres template :
```cpp
template<typename T, typename Deleter = std::default_delete<T>>
class unique_ptr;
```

Le deleter par défaut fait simplement `delete p`.

## 3. Arithmétique de Base

### 3.1 Opérateurs Unaires

#### Pré/Post-incrémentation
```cpp
class Counter {
    int n_;
public:
    // Pré-incrément : ++x
    Counter& operator++() {
        ++n_;
        return *this;
    }
    
    // Post-incrément : x++
    Counter operator++(int) {  // 'int' factice pour différencier
        Counter tmp = *this;
        ++(*this);  // Utilise pré-incrément
        return tmp;
    }
};
```

**Astuce mnémotechnique** : "Post-incrément = paramètre stupide supplémentaire"

#### Performance
```cpp
// Pour les itérateurs
for (auto it = v.begin(); it != v.end(); ++it)  // BON
for (auto it = v.begin(); it != v.end(); it++)  // MAUVAIS (copie inutile)

// Idiome professionnel
for (auto it = v.begin(), end = v.end(); it != end; ++it)
```

### 3.2 Opérateurs Composés (op=)

```cpp
struct Quaternion {
    double w, x, y, z;
    
    // Opérateur +=
    Quaternion& operator+=(const Quaternion& rhs) {
        w += rhs.w; x += rhs.x; y += rhs.y; z += rhs.z;
        return *this;
    }
};
```

**Règle** : Les opérateurs `op=` sont toujours des méthodes membres et retournent `*this`.

## 4. Opérateurs Binaires

### 4.1 Le Problème de Symétrie

```cpp
struct Quaternion {
    Quaternion(double v) : w(v), x(0), y(0), z(0) {}  // Conversion implicite
    
    // MAUVAIS : pas symétrique
    Quaternion operator+(const Quaternion& rhs) const {
        Quaternion tmp = *this;
        tmp += rhs;
        return tmp;
    }
};

Quaternion q(1, 0, 0, 0);
auto r1 = q + 2;    // OK : 2 converti en Quaternion
auto r2 = 2 + q;    // ERREUR : int n'a pas d'operator+
```

### 4.2 Solution : Opérateurs Non-Membres

```cpp
// BON : symétrique
Quaternion operator+(const Quaternion& lhs, const Quaternion& rhs) {
    Quaternion tmp = lhs;
    tmp += rhs;
    return tmp;
}

// Maintenant les deux fonctionnent
auto r1 = q + 2;    // OK
auto r2 = 2 + q;    // OK aussi !
```

### 4.3 Problème avec les Templates

```cpp
template<typename T>
struct Quaternion {
    // ...
};

// Ne fonctionne PAS pour 1 + Quaternion<double>
template<typename T>
Quaternion<T> operator+(const Quaternion<T>& lhs, const Quaternion<T>& rhs);
```

**Solution** : Surcharges explicites
```cpp
template<typename T>
Quaternion<T> operator+(const Quaternion<T>& lhs, const Quaternion<T>& rhs);

template<typename T>
Quaternion<T> operator+(const T& lhs, const Quaternion<T>& rhs);

template<typename T>
Quaternion<T> operator+(const Quaternion<T>& lhs, const T& rhs);
```

## 5. Cohérence et Sémantique

### 5.1 Le Principe de Cohérence

**Question** : Les opérateurs doivent-ils respecter leur sémantique habituelle ?

**Réponse** : Non obligatoire, mais FORTEMENT recommandé !

```cpp
// TRÈS MAUVAIS
MyClass& operator++() {
    format_hard_drive();  // NON !
    return *this;
}
```

**Citation de James Gosling (créateur de Java)** :
> "J'ai interdit la surcharge d'opérateurs en Java parce que j'ai vu comment c'était utilisé en C++"

### 5.2 L'Opérateur le Plus Malchanceux

```cpp
// Utilisation normale
int x = 8 << 2;  // Décalage de bits : 32

// Mais en C++...
std::cout << "Hello" << std::endl;  // Sortie stream !
```

L'opérateur `<<` a été "détourné" pour l'I/O, créant une incohérence sémantique permanente.

## 6. Comparaisons et l'Opérateur Spaceship (C++20)

### 6.1 Relations Mathématiques

**Équivalence** (==) doit être :
- Réflexive : `a == a`
- Symétrique : `a == b` ⟹ `b == a`
- Transitive : `a == b && b == c` ⟹ `a == c`

**Ordre** (<) doit être :
- Anti-réflexif : `!(a < a)`
- Anti-symétrique : `a < b` ⟹ `!(b < a)`
- Transitif : `a < b && b < c` ⟹ `a < c`

### 6.2 L'Opérateur Spaceship (<=>)

```cpp
class MyInt {
    int x_;
public:
    // Définit TOUS les opérateurs de comparaison !
    auto operator<=>(const MyInt& rhs) const = default;
};

MyInt a{4}, b{5};
if (a < b)   // Fonctionne
if (a <= b)  // Fonctionne
if (a > b)   // Fonctionne
if (a >= b)  // Fonctionne
```

### 6.3 Types d'Ordonnancement

```cpp
// Pour les types comme int
std::strong_ordering operator<=>(const T&) const;

// Pour les types comme double (avec NaN)
std::partial_ordering operator<=>(const T&) const;

// Compromis
std::weak_ordering operator<=>(const T&) const;
```

**Règle d'or** : Utilisez `= default` dans 99% des cas !

## 7. Opérateurs Exotiques

### 7.1 Opérateur d'Adresse (&)

```cpp
class Strange {
    int* ptr_;
public:
    // BIZARRE : retourne toujours 42
    int* operator&() const { return (int*)42; }
};

// Solution pour obtenir la vraie adresse
Strange s;
auto real_addr = std::addressof(s);  // Contourne l'opérateur
```

### 7.2 Opérateur ->*

```cpp
// Pointeur vers membre
class MyClass {
    void doit() {}
};

void (MyClass::*ptr)() = &MyClass::doit;
MyClass obj;
(obj.*ptr)();  // Appel via pointeur membre

// ->* peut être surchargé librement (pas de contraintes)
// Souvent utilisé quand on manque d'idées...
```

### 7.3 Opérateur Virgule (,)

```cpp
// DANGEREUX : perd le séquençage !
class Bad {
    Bad operator,(const Bad&) { /*...*/ }
};

// Normal : foo exécuté avant bar
foo(), bar();

// Avec surcharge : ordre non défini !
Bad a, b;
a, b;  // Qui sait ce qui se passe ?
```

## 8. Règles Générales

### 8.1 Ce qui NE PEUT PAS être Surchargé

- `.` (accès membre)
- `.*` (pointeur vers membre)
- `::` (résolution de portée)
- `?:` (ternaire)
- `sizeof`, `typeid`, opérateurs de cast
- Mots-clés (`and`, `or`, `not`, etc.)

**Exception** : `new`, `delete` et `co_await` peuvent être surchargés.

### 8.2 Ce qu'il NE FAUT PAS Surcharger

- `&&` et `||` : Perdent l'évaluation paresseuse
- `,` : Perd le séquençage
- Unaire `+` : Perd le "positive hack" pour les conversions

### 8.3 Opérateurs qui DOIVENT être Membres

- `=` (assignation)
- `[]` (indexation)
- `->` (accès membre)
- `()` (appel de fonction)
- Conversions de type

### 8.4 Opérateurs qui DEVRAIENT être Non-Membres

- Tous les opérateurs binaires symétriques (`+`, `-`, `*`, `/`, etc.)
- `<<` et `>>` pour les streams

## 9. Exemple Complet : Classe Matrix

```cpp
template<typename T>
class Matrix {
private:
    T* data_;
    size_t rows_, cols_;
    
public:
    // Opérateurs arithmétiques
    Matrix& operator+=(const Matrix& rhs) {
        // Vérifier dimensions
        for (size_t i = 0; i < rows_ * cols_; ++i) {
            data_[i] += rhs.data_[i];
        }
        return *this;
    }
    
    // Comparaisons (C++20)
    auto operator<=>(const Matrix&) const = default;
    bool operator==(const Matrix&) const = default;
};

// Opérateurs non-membres
template<typename T>
Matrix<T> operator+(const Matrix<T>& lhs, const Matrix<T>& rhs) {
    Matrix<T> result = lhs;
    result += rhs;
    return result;
}
```

## Conclusion

La surcharge d'opérateurs en C++ est un outil puissant qui permet :
1. **Performance** : Via l'inlining des objets fonctions
2. **Syntaxe naturelle** : Pour les types mathématiques
3. **Abstraction** : Cache les détails d'implémentation

**Règles d'or** :
- Respectez la sémantique attendue
- Préférez les fonctions non-membres pour la symétrie
- Utilisez `= default` quand possible (C++20)
- N'abusez pas : si la sémantique n'est pas claire, utilisez des fonctions nommées

**Ressources recommandées** :
- Ben Deane : "Operator Overloading"
- Titus Winters : "Modern C++ Design"
- Documentation sur l'opérateur spaceship (C++20)
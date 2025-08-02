# Leçon : Fonctions Statiques, Amis et Surcharge d'Opérateurs en C++

## Introduction

Cette leçon marque la fin de la partie basique du cours C++. Nous aborderons les fonctions statiques, les fonctions amies, la conception avancée de classes (en prenant l'exemple d'une matrice), et introduirons les concepts de conversion de types et de surcharge d'opérateurs.

## 1. Fonctions Statiques et Amies (Friends)

### 1.1 L'Invariant des Classes RAII

**Rappel** : RAII signifie "Resource Acquisition Is Initialization"
- Un classe RAII capture une ressource dans son constructeur
- La libère dans son destructeur
- **Invariant principal** : La ressource appartient exclusivement à l'objet

### 1.2 Fonctions Statiques

Les fonctions statiques sont des fonctions qui :
- Travaillent **sans** le pointeur `this` implicite
- Opèrent sur la classe elle-même, pas sur un objet
- Ont accès à l'état privé de la classe

```cpp
class Matrix {
private:
    T* data_;
    
public:
    // Fonction statique - pas de 'this'
    static Matrix identity(size_t n) {
        Matrix m(n, n);
        // Remplir la diagonale avec des 1
        return m;
    }
    
    // Méthode normale - a un 'this' implicite
    void transpose() {
        // Utilise 'this->data_'
    }
};

// Utilisation
Matrix m = Matrix::identity(5);  // Pas d'objet nécessaire
```

### 1.3 Fonctions Amies (Friend)

Les fonctions amies :
- Sont déclarées avec `friend` dans la classe
- Ont accès complet à l'état privé
- **DANGER** : Brisent l'encapsulation !

```cpp
class Complex {
private:
    double real_, imag_;
    
    // Déclaration d'ami AVANT sa définition !
    friend Complex operator+(const Complex& a, const Complex& b);
};

// Définition en dehors de la classe
Complex operator+(const Complex& a, const Complex& b) {
    return Complex(a.real_ + b.real_, a.imag_ + b.imag_);
}
```

### 1.4 Comparaison : Méthode vs Statique vs Friend

| Caractéristique | Méthode | Statique | Friend |
|----------------|---------|----------|---------|
| Reçoit `this` implicite | ✓ | ✗ | ✗ |
| Dans l'espace de noms de la classe | ✓ | ✓ | ✗ (peut être) |
| Accès à l'état privé | ✓ | ✓ | ✓ |

**Recommandation** : Évitez les `friend` ! Ils violent l'encapsulation, surtout avec les templates.

## 2. Conception d'une Classe Matrix

### 2.1 Constructeurs Multiples

Problème : Comment créer différents types de matrices ?

```cpp
template<typename T>
class Matrix {
public:
    // Constructeur principal
    Matrix(size_t rows, size_t cols, T value = T{});
    
    // Constructeur depuis itérateurs
    template<typename InputIt>
    Matrix(size_t rows, size_t cols, InputIt first, InputIt last);
    
    // Fonctions statiques pour cas spéciaux
    static Matrix identity(size_t n) {
        Matrix m(n, n);
        for (size_t i = 0; i < n; ++i) {
            m(i, i) = T{1};
        }
        return m;
    }
    
    static Matrix diagonal(size_t n, T value) {
        Matrix m(n, n);
        for (size_t i = 0; i < n; ++i) {
            m(i, i) = value;
        }
        return m;
    }
};
```

### 2.2 La Règle des Cinq

Pour une classe gérant des ressources, définissez **tous** :

```cpp
class Matrix {
private:
    T* data_;
    size_t rows_, cols_;
    
public:
    // 1. Destructeur
    ~Matrix() { delete[] data_; }
    
    // 2. Constructeur de copie
    Matrix(const Matrix& other);
    
    // 3. Opérateur d'assignation
    Matrix& operator=(const Matrix& other);
    
    // 4. Constructeur de déplacement
    Matrix(Matrix&& other) noexcept;
    
    // 5. Opérateur d'assignation par déplacement
    Matrix& operator=(Matrix&& other) noexcept;
};
```

**Important** : Marquez les opérations de déplacement `noexcept` si elles ne font que manipuler des pointeurs !

### 2.3 Sélecteurs et Méthodes Utilitaires

```cpp
class Matrix {
public:
    // Sélecteurs constants
    size_t rows() const { return rows_; }
    size_t cols() const { return cols_; }
    
    // Sélecteurs agrégés
    T trace() const;  // Somme de la diagonale
    bool equals(const Matrix& other) const;
    
    // Méthodes utilitaires (avec ref-qualifiers !)
    Matrix operator-() const &;      // Pour lvalues
    Matrix&& operator-() &&;         // Pour rvalues
    
    Matrix transpose() const &;      // Retourne une copie
    void transpose() &;              // Modifie sur place
};
```

**Principe** : Si vous avez le choix entre deux méthodes et que l'une peut être implémentée en termes de l'autre, choisissez celle qui ne peut PAS être dérivée.

## 3. Indexeurs et Classes Proxy

### 3.1 Le Problème de l'Accès 2D

Pour une matrice, on veut écrire `m[i][j]`, mais :
- L'opérateur `[]` ne prend qu'un seul argument (jusqu'à C++23)
- Retourner un pointeur brut expose l'implémentation

### 3.2 Solution : Classes Proxy

```cpp
class Matrix {
private:
    // Classe proxy pour une ligne
    class ProxyRow {
        T* row_ptr_;
    public:
        ProxyRow(T* ptr) : row_ptr_(ptr) {}
        
        T& operator[](size_t col) {
            return row_ptr_[col];
        }
        
        const T& operator[](size_t col) const {
            return row_ptr_[col];
        }
    };
    
public:
    ProxyRow operator[](size_t row) {
        return ProxyRow(data_ + row * cols_);
    }
    
    const ProxyRow operator[](size_t row) const {
        return ProxyRow(data_ + row * cols_);
    }
};
```

**Utilisation** :
```cpp
Matrix<int> m(3, 4);
m[1][2] = 42;  // Appelle d'abord Matrix::operator[], puis ProxyRow::operator[]
```

## 4. Unique Pointers et Littérature

### 4.1 std::unique_ptr

`std::unique_ptr` est la classe RAII idéale :
- Pas de copie (explicitement supprimée)
- Déplacement seulement
- Gère n'importe quelle ressource

```cpp
// Création
auto p = std::make_unique<MyClass>(args...);

// Transfert de propriété
void bar(std::unique_ptr<MyClass> p);
bar(std::move(p));  // p est maintenant vide

// const unique_ptr bloque même le déplacement !
const std::unique_ptr<T> cp = std::make_unique<T>();
// cp ne peut être ni copié ni déplacé
```

### 4.2 Ressources Recommandées

**Livres** :
- Scott Meyers : "Effective Modern C++" (42 façons d'améliorer votre code)

**Conférences** :
- Klaus Iglberger : "Move Semantics" (4 heures détaillées)
- Arthur O'Dwyer : "Rule of Zero"
- Nicolai Josuttis : "The Nightmare of Move Semantics for Trivial Classes"

## 5. Conversion de Types (Casting)

### 5.1 Problèmes avec les C-style casts

Le cast C `(T)expr` fait **trop de choses** :
- Conversions numériques
- Suppression de const
- Réinterprétation de pointeurs

### 5.2 Les Casts C++

#### static_cast
```cpp
double d = 3.14;
int i = static_cast<int>(d);  // Conversion sémantique
```

**Attention** : `static_cast` peut complètement changer la représentation binaire !

#### const_cast
```cpp
const int* cp = &x;
int* p = const_cast<int*>(cp);  // Supprime const
```

#### reinterpret_cast
```cpp
int i = 42;
char* cp = reinterpret_cast<char*>(&i);  // Réinterprète les bits
```

**DANGER** : Violation de strict aliasing ! Utilisez plutôt :

#### bit_cast (C++20)
```cpp
float f = 3.14f;
auto i = std::bit_cast<int>(f);  // Copie bit à bit sûre
```

### 5.3 Hiérarchie de Danger

Du plus sûr au plus dangereux :
1. **static_cast** : Change la valeur sémantique (surprenant !)
2. **const_cast** : Supprime les garanties
3. **reinterpret_cast** : Réinterprète les bits (UB potentiel)

## 6. Promotions et Conversions Entières

### 6.1 Règles de Promotion

**Règle fondamentale** : Tout type plus petit que `int` est promu en `int` !

```cpp
unsigned short x = 0x00F0;
unsigned short y = 0x00F0;
auto z = x * y;  // z est de type int, pas unsigned short !
```

### 6.2 Hiérarchie des Conversions

1. Si flottant présent → tout devient flottant
2. Si signé/non-signé mélangés → non-signé gagne
3. Types plus petits → promus au plus grand
4. **Piège** : `char`, `short` → toujours promus en `int` (signé !)

### 6.3 Exemple Surprenant

```cpp
unsigned short x = 0x00F0;
unsigned short y = 0x00F0;
int v = x * y;              // OK, résultat positif
unsigned long long w = x * y; // DANGER ! Promotion en int d'abord
```

## 7. Opérateurs Unaires

### 7.1 Unaire Plus : Le "Hack Positif"

L'opérateur unaire `+` force les conversions :

```cpp
struct Foo {
    operator long() const { return 42; }
};

void f(long);
void f(Foo);

Foo obj;
f(obj);   // Appelle f(Foo)
f(+obj);  // Appelle f(long) !
```

**Utilité** : Forcer une conversion implicite quand nécessaire.

### 7.2 Unaire Moins

Peut être défini :
- Dans la classe (prioritaire)
- En dehors de la classe

```cpp
class Quaternion {
    // Version membre
    Quaternion operator-() const {
        return Quaternion(-w, -x, -y, -z);
    }
};

// OU version libre
Quaternion operator-(const Quaternion& q) {
    return Quaternion(-q.w, -q.x, -q.y, -q.z);
}
```

**Règle** : Si les deux existent, la version membre gagne toujours.

## 8. Principes de Conception

### 8.1 Évitez les Variables Globales

Préférez :
- Méthodes statiques
- Espaces de noms anonymes
- Singletons (avec parcimonie)

### 8.2 Utilisez const Correctement

```cpp
class Container {
    T data_;
public:
    // Pour objets modifiables seulement
    T& access() & { return data_; }
    
    // Pour objets const
    const T& access() const& { return data_; }
    
    // Pour temporaires - évite les références pendantes !
    T access() && { return std::move(data_); }
};
```

### 8.3 Conception d'Opérateurs

**Règles d'or** :
1. Les opérateurs binaires symétriques (`+`, `-`, `*`) → fonctions libres
2. Les opérateurs modifiant l'objet (`+=`, `=`) → méthodes membres
3. N'abusez pas de la surcharge (lisibilité > concision)

## Conclusion

Cette leçon a couvert les aspects avancés de la conception de classes en C++ :
- L'importance de l'encapsulation et les dangers des `friend`
- Les techniques de conception pour des classes complexes (matrices)
- L'utilisation judicieuse des casts et la compréhension des conversions
- Les subtilités de la surcharge d'opérateurs

Ces concepts forment la base pour aborder les sujets encore plus avancés comme l'héritage et les templates, qui viendront dans les prochaines leçons.
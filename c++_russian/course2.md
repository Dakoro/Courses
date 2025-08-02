# Cours de C++ : Pointeurs, Références et Encapsulation

## Introduction : Genesis

### Les bases de la mémoire

Imaginons un espace vide. Dans cet espace apparaissent soudainement **4 bits**.

```
[0100] = ?
```

**Question** : Que représentent ces 4 bits ?
- Si on les interprète comme un nombre non signé : de 0 à 15
- Si on les interprète comme un nombre signé : de -7 à 8
- Mais en réalité, ces bits peuvent représenter n'importe quoi !

### Les trois composants fondamentaux d'une variable

Pour qu'une zone mémoire devienne une **variable**, nous devons lui attribuer :

1. **Une valeur sémantique** : ce que les bits représentent (ici, le nombre 4)
2. **Un type de valeur** : la plage de valeurs possibles (ex: -7 à 8 pour un entier signé sur 4 bits)
3. **Un nom** : l'identifiant de la variable (ex: `a`)

### Types en C++

En C++, nous distinguons deux aspects d'un type :

- **Value type** : La plage de valeurs possibles
- **Object type** : L'ensemble des opérations définies sur ce type

Par exemple, pour un type `int4` (entier sur 4 bits) :
- Value type : -7 à 8
- Object type : opérations arithmétiques signées (+, -, *, /, etc.)

### Modèle mémoire en C/C++

En C/C++, les objets vivent dans un **modèle de mémoire RAM** (Random Access Memory) :

```
Mémoire : [----[a]----[b]--------]
           ^    ^
           |    |
        adresse_a  adresse_b
```

**Points clés** :
- Chaque objet a une **adresse** (distance depuis le début de la mémoire)
- Le type de l'adresse est un **pointeur**
- La taille minimale adressable est le `char` (pas le bit !)

## Pointeurs en C++

### Définition et syntaxe

Un pointeur est une variable qui contient l'adresse d'une autre variable :

```cpp
int a = 10;
int* ptr = &a;  // ptr contient l'adresse de a
```

### Arithmétique des pointeurs

Quand on ajoute 1 à un pointeur, on avance de `sizeof(type pointé)` octets :

```cpp
int* p = ...;
p + 1;  // avance de sizeof(int) octets, pas de 1 octet !
```

### Syntaxe des tableaux

La notation avec crochets est un sucre syntaxique :
```cpp
a[i] ≡ *(a + i)
```

### Pointeur nul

En C++ moderne, utilisez toujours `nullptr` :
```cpp
int* p = nullptr;  // Préféré
int* p = NULL;     // Style C, à éviter
int* p = 0;        // À éviter
```

## Références en C++

### Concept fondamental

Une référence est un **alias** (un autre nom) pour un objet existant :

```cpp
int x = 10;
int& ref = x;  // ref est maintenant un autre nom pour x
ref = 20;      // modifie x
```

### Différences entre pointeurs et références

| Pointeurs | Références |
|-----------|------------|
| Peuvent être nuls (`nullptr`) | Ne peuvent jamais être nulles |
| Peuvent être réassignés | Une fois liées, ne peuvent pas être réassignées |
| Arithmétique possible | Pas d'arithmétique |
| Syntaxe : `*ptr` pour accéder | Syntaxe transparente |
| Peuvent pointer vers rien | Doivent toujours référencer un objet valide |

### Exemple comparatif

```cpp
void par_pointeur(int* p) {
    if (p != nullptr) {  // Vérification nécessaire
        *p = 42;         // Déréférencement explicite
    }
}

void par_reference(int& r) {
    r = 42;  // Pas de vérification, syntaxe naturelle
}
```

### Implémentation sous-jacente

Bien que les références soient conceptuellement des alias, le compilateur les implémente souvent comme des pointeurs cachés :

```cpp
void foo(int* p) { *p = 1; }
void bar(int& r) { r = 1; }
// Génèrent le même code assembleur !
```

## Exemple pratique : Géométrie computationnelle

### Problème : Intersection de triangles

**Entrée** : Coordonnées de deux triangles
**Sortie** : Aire de leur intersection

### Structure de données

```cpp
struct Point {
    float x = NAN;
    float y = NAN;
    
    bool isValid() const {
        return !std::isnan(x) && !std::isnan(y);
    }
};

struct Line {
    float a, b, c;  // ax + by + c = 0
    
    Line(const Point& p1, const Point& p2) {
        // Construction à partir de 2 points
        // ... calculs géométriques ...
    }
};
```

### Algorithme d'intersection

Pour calculer l'intersection de deux polygones convexes :

1. Pour chaque côté du second polygone
2. Couper le premier polygone par la droite support
3. Garder le demi-espace contenant le reste du second polygone
4. Le résultat est l'intersection

## Encapsulation et invariants de classe

### Qu'est-ce qu'un invariant ?

Un **invariant de classe** est une propriété qui doit toujours être vraie pour tous les objets de cette classe.

Exemples :
- Une liste chaînée ne doit pas contenir de cycle
- Les sommets d'un polygone convexe doivent être ordonnés
- Un itérateur doit pointer vers une mémoire valide

### Encapsulation en C++

L'encapsulation permet de **protéger les invariants** :

```cpp
class List {
private:  // Données cachées
    Node* head;
    Node* tail;
    
public:   // Interface publique
    void push_back(int value);
    int size() const;
    // Les méthodes maintiennent les invariants
};
```

### Private vs Public

- `private` : accessible uniquement aux méthodes de la classe
- `public` : accessible de partout
- Par défaut : `private` pour `class`, `public` pour `struct`

### Points importants sur l'encapsulation

1. **L'encapsulation concerne les noms, pas les objets** :
   ```cpp
   class MyClass {
   private:
       int data;
   public:
       void method(MyClass& other) {
           other.data = 42;  // OK ! Même type
       }
   };
   ```

2. **Rien n'empêche techniquement de violer l'encapsulation** :
   ```cpp
   MyClass obj;
   char* raw = reinterpret_cast<char*>(&obj);
   // Peut modifier les données privées... mais ne le faites pas !
   ```

3. **L'encapsulation repose sur la discipline du programmeur**

## Exercices pratiques

### Exercice 1 : Intersection de triangles 2D
Implémenter un programme qui calcule l'aire de l'intersection de deux triangles en 2D.

### Exercice 2 : Intersection de triangles 3D
Étendre le programme pour détecter quels triangles s'intersectent dans une liste de triangles 3D.

## Bonnes pratiques

1. **Préférez les références aux pointeurs** quand :
   - Vous n'avez pas besoin de valeur nulle
   - Vous n'avez pas besoin d'arithmétique
   - Vous n'avez pas besoin de réassignation

2. **Utilisez l'encapsulation pour protéger les invariants**
   - Ne rendez privé que ce qui doit l'être
   - N'exposez pas inutilement l'implémentation

3. **Écrivez des tests unitaires** pour vos classes
   - Testez que les invariants sont maintenus
   - Utilisez des frameworks comme Google Test

## Ressources recommandées

- **Grady Booch** : *Object-Oriented Analysis and Design with Applications*
- **Gilbert Strang** : *Introduction to Linear Algebra*
- Pour la géométrie computationnelle : *Computational Geometry: Algorithms and Applications*

## Points clés à retenir

- Les pointeurs permettent l'arithmétique mais peuvent être dangereux
- Les références sont plus sûres mais moins flexibles
- L'encapsulation protège les invariants de classe
- La discipline du programmeur est essentielle en C++
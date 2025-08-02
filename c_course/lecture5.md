# Leçon de Programmation C : Nombres Premiers, Structures et Complexité

## 1. Les Nombres Premiers

### Définition

Un **nombre premier** est un nombre naturel supérieur à 1 qui n'a que deux diviseurs : 1 et lui-même.

**Exemples** : 2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37...

**Note** : Nous nous concentrons sur les nombres premiers positifs. En théorie des nombres, on peut étendre cette définition à d'autres ensembles.

### Algorithme de Test de Primalité

#### Version naïve

```c
int is_prime(unsigned int n) {
    if (n < 2) return 0;  // 0 et 1 ne sont pas premiers
    
    for (unsigned int i = 2; i * i <= n; i++) {
        if (n % i == 0) {
            return 0;  // n est divisible par i
        }
    }
    return 1;  // n est premier
}
```

**Optimisation importante** : On teste seulement jusqu'à √n car si n a un diviseur d > √n, alors n/d < √n est aussi un diviseur.

#### Optimisations pratiques

```c
int is_prime_optimized(unsigned int n) {
    if (n == 2) return 1;
    if (n < 2 || n % 2 == 0) return 0;  // Éliminer les pairs
    
    // Tester seulement les nombres impairs
    for (unsigned int i = 3; i * i <= n; i += 2) {
        if (n % i == 0) return 0;
    }
    return 1;
}
```

## 2. Le Crible d'Ératosthène

### Principe

Le crible d'Ératosthène est un algorithme efficace pour trouver tous les nombres premiers jusqu'à n.

**Algorithme** :
1. Créer un tableau de booléens de taille n+1
2. Marquer 0 et 1 comme non premiers
3. Pour chaque nombre i de 2 à √n :
   - Si i est marqué comme premier
   - Marquer tous ses multiples (i², i²+i, i²+2i, ...) comme non premiers

### Implémentation avec allocation dynamique

```c
#include <stdlib.h>
#include <string.h>

char* sieve_of_eratosthenes(int n) {
    // Allouer de la mémoire (0 = premier, 1 = composé)
    char* sieve = (char*)calloc(n + 1, sizeof(char));
    if (sieve == NULL) {
        abort();  // Échec d'allocation
    }
    
    sieve[0] = sieve[1] = 1;  // 0 et 1 ne sont pas premiers
    
    for (int i = 2; i * i <= n; i++) {
        if (sieve[i] == 0) {  // i est premier
            // Marquer les multiples de i
            for (int j = i * i; j <= n; j += i) {
                sieve[j] = 1;
            }
        }
    }
    
    return sieve;
}
```

## 3. Gestion de la Mémoire Dynamique

### Les fonctions d'allocation

```c
// malloc : alloue de la mémoire non initialisée
int* p1 = (int*)malloc(10 * sizeof(int));

// calloc : alloue de la mémoire initialisée à zéro
int* p2 = (int*)calloc(10, sizeof(int));

// IMPORTANT : toujours vérifier le résultat
if (p1 == NULL) {
    // Gérer l'erreur
    abort();
}

// Libérer la mémoire
free(p1);
free(p2);
```

### Règles importantes

1. **Toujours vérifier** si l'allocation a réussi (pointeur non NULL)
2. **Toujours libérer** la mémoire allouée avec `free()`
3. **Ne pas utiliser** un pointeur après `free()`

## 4. Les Structures en C

### Définition et utilisation

```c
// Définir une structure point
struct Point {
    int x;
    int y;
};

// Définir une structure triangle
struct Triangle {
    struct Point pts[3];  // Tableau de 3 points
};

// Utilisation
struct Point p;
p.x = 10;
p.y = 20;

// Avec un pointeur
struct Point* ptr = &p;
ptr->x = 30;  // Équivalent à (*ptr).x = 30
```

### Exemple : Calcul de l'aire d'un triangle

```c
int triangle_area_doubled(struct Triangle t) {
    // Formule du déterminant pour l'aire
    // Aire = |det| / 2, donc aire doublée = |det|
    int det = t.pts[0].x * (t.pts[1].y - t.pts[2].y) +
              t.pts[1].x * (t.pts[2].y - t.pts[0].y) +
              t.pts[2].x * (t.pts[0].y - t.pts[1].y);
    
    return abs(det);
}
```

### Structure pour le crible avec taille

```c
struct Sieve {
    char* data;
    int size;
};

struct Sieve init_sieve(int n) {
    struct Sieve s;
    s.size = n;
    s.data = (char*)calloc(n + 1, sizeof(char));
    if (s.data == NULL) abort();
    
    // Construction du crible...
    return s;
}

void free_sieve(struct Sieve* s) {
    free(s->data);
    s->data = NULL;
    s->size = 0;
}

int is_prime_sieve(struct Sieve s, int n) {
    assert(n >= 0 && n <= s.size);
    return s.data[n] == 0 ? 1 : 0;
}
```

## 5. Notation O et Complexité Algorithmique

### Introduction à la notation O

La notation O (Big O) décrit le comportement asymptotique d'un algorithme.

**Définition** : f(n) = O(g(n)) signifie qu'il existe des constantes c et n₀ telles que :
- Pour tout n ≥ n₀ : f(n) ≤ c × g(n)

### Exemples de complexités

| Notation | Nom | Exemple |
|----------|-----|---------|
| O(1) | Constante | Accès à un élément de tableau |
| O(log n) | Logarithmique | Recherche binaire |
| O(n) | Linéaire | Parcours simple |
| O(n log n) | Linéarithmique | Tri fusion |
| O(n²) | Quadratique | Boucles imbriquées |
| O(2ⁿ) | Exponentielle | Fibonacci récursif naïf |

### Analyse de notre algorithme de primalité

```c
// Complexité : O(√n)
int is_prime(unsigned int n) {
    if (n < 2) return 0;              // O(1)
    for (int i = 2; i * i <= n; i++) { // O(√n) itérations
        if (n % i == 0) return 0;      // O(1)
    }
    return 1;
}
```

### Comparaison pratique des complexités

Pour n = 10¹⁴ sur un ordinateur à 1 GHz :
- O(n) : ~2 heures
- O(n log n) : ~28 heures
- O(n²) : ~3 ans
- O(2ⁿ) : impossible

## 6. Optimisations et Astuces

### Améliorer la constante sans changer la complexité

```c
// Version 1 : teste tous les nombres
// Complexité : O(√n), mais lent

// Version 2 : ignore les pairs
// Complexité : O(√n), mais 2× plus rapide

// Version 3 : ignore les multiples de 2 et 3
// Complexité : O(√n), mais 3× plus rapide
```

**Important** : L'amélioration de la constante ne change pas la notation O !

### Estimation du n-ième nombre premier

Pour allouer le crible jusqu'au n-ième nombre premier :

```c
int prime_upper_bound(int n) {
    if (n < 6) return 30;  // Cas spéciaux
    
    double log_n = log(n);
    double log_log_n = log(log_n);
    
    return (int)(n * (log_n + log_log_n));
}
```

## 7. Exercices Pratiques

### Exercice 1 : Nombre premier n°1000
Trouvez le 1000ᵉ nombre premier en utilisant le crible d'Ératosthène.

### Exercice 2 : Formules génératrices
La formule d'Euler n² + n + 41 génère des nombres premiers pour n = 0 à 39.
Trouvez a et b tels que n² + an + b génère plus de 40 nombres premiers consécutifs.

### Exercice 3 : Nombres premiers circulaires
197 est premier circulaire car 197, 971 et 719 sont tous premiers.
Trouvez le nombre premier circulaire le plus proche d'un nombre donné.

## 8. Bonnes Pratiques

### Gestion mémoire
```c
// TOUJOURS faire
char* buffer = malloc(size);
if (buffer == NULL) {
    // Gérer l'erreur
}
// Utiliser buffer...
free(buffer);
```

### Utilisation des assertions
```c
assert(index >= 0 && index < size);  // Vérification des préconditions
```

### Parenthèses dans les expressions
```c
// MAUVAIS
if (n % i == 0)

// BON (plus clair)
if ((n % i) == 0)
```

## Points Clés à Retenir

1. **Test de primalité** : O(√n) est suffisant, pas besoin de tester jusqu'à n
2. **Crible d'Ératosthène** : Efficace pour plusieurs tests, O(n log log n)
3. **Mémoire dynamique** : Toujours vérifier malloc/calloc et libérer avec free
4. **Structures** : Permettent de regrouper des données liées
5. **Notation O** : Décrit la scalabilité, pas la performance absolue
6. **Optimisations** : Les constantes comptent en pratique !

## Pour Aller Plus Loin

- Implémenter le crible segmenté pour de très grandes valeurs
- Explorer d'autres tests de primalité (Miller-Rabin, AKS)
- Étudier la distribution des nombres premiers
- Optimiser avec des techniques de programmation bit à bit
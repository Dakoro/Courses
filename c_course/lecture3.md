# Leçon de Programmation C : Les Nombres de Fibonacci

## 1. Introduction aux Nombres de Fibonacci

### Le problème historique des lapins

Léonard de Pise (Fibonacci) a posé le problème suivant :
- Une paire de jeunes lapins met un mois pour devenir adulte
- Chaque paire adulte produit une nouvelle paire de jeunes lapins chaque mois
- Les lapins ne meurent pas

**Question** : Combien de paires de lapins après n mois ?

### La suite de Fibonacci

La relation de récurrence est :
```
F(0) = 0
F(1) = 1
F(n) = F(n-1) + F(n-2) pour n ≥ 2
```

Les premiers termes : 0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, 144...

## 2. Implémentation Récursive Naïve

### Le code

```c
unsigned long long fib(unsigned int n) {
    if (n == 0) return 0;
    if (n <= 2) return 1;
    return fib(n - 1) + fib(n - 2);
}
```

### Le problème

Cette implémentation est **extrêmement inefficace** :
- Pour calculer F(20), on recalcule F(18) deux fois, F(17) trois fois, etc.
- La complexité est exponentielle : O(2ⁿ)
- F(50) prend une éternité à calculer !

**Visualisation** : L'arbre des appels récursifs montre une explosion combinatoire avec de nombreux calculs répétés.

## 3. Implémentation Itérative

### Solution efficace

```c
unsigned long long fib_iter(unsigned int n) {
    if (n == 0) return 0;
    if (n == 1) return 1;
    
    unsigned long long first = 0;
    unsigned long long second = 1;
    
    for (unsigned int i = 2; i <= n; i++) {
        unsigned long long temp = second;
        second = second + first;
        first = temp;
    }
    
    return second;
}
```

**Avantages** :
- Complexité linéaire : O(n)
- Calcul de F(50) instantané
- Utilise seulement deux variables

## 4. La Formule de Binet

### Formule mathématique

```
F(n) = (φⁿ - ψⁿ) / √5
```

Où :
- φ = (1 + √5) / 2 ≈ 1.618 (nombre d'or)
- ψ = (1 - √5) / 2

### Problème d'implémentation

```c
#include <math.h>

unsigned long long fib_binet(unsigned int n) {
    double phi = (1.0 + sqrt(5.0)) / 2.0;
    double psi = (1.0 - sqrt(5.0)) / 2.0;
    return round((pow(phi, n) - pow(psi, n)) / sqrt(5.0));
}
```

**Problème** : Les erreurs d'arrondi s'accumulent pour les grandes valeurs de n.
- Pour F(80), l'erreur est déjà significative
- √5 ne peut pas être représenté exactement en `double`

## 5. Gestion des Débordements

### Le problème

Les nombres de Fibonacci croissent exponentiellement :
- F(93) dépasse la capacité d'un `unsigned long long` (64 bits)
- Le résultat est tronqué modulo 2⁶⁴

### Solution : Arithmétique modulaire

```c
unsigned long long fib_mod(unsigned int n, unsigned long long m) {
    if (n == 0) return 0;
    if (n == 1) return 1;
    
    unsigned long long first = 0;
    unsigned long long second = 1;
    
    for (unsigned int i = 2; i <= n; i++) {
        unsigned long long temp = second;
        second = (second + first) % m;  // IMPORTANT : modulo à chaque étape
        first = temp;
    }
    
    return second;
}
```

## 6. Les Périodes de Pisano

### Définition

La suite de Fibonacci modulo m est **périodique**. La période est appelée période de Pisano π(m).

**Exemples** :
- mod 2 : 0, 1, 1, 0, 1, 1, ... (période = 3)
- mod 3 : 0, 1, 1, 2, 0, 2, 2, 1, 0, 1, ... (période = 8)
- mod 4 : 0, 1, 1, 2, 3, 1, 0, 1, ... (période = 6)

### Calcul de la période

```c
unsigned long long pisano_period(unsigned long long m) {
    unsigned long long a = 0, b = 1;
    
    for (unsigned long long i = 0; i < m * m; i++) {
        unsigned long long temp = (a + b) % m;
        a = b;
        b = temp;
        
        if (a == 0 && b == 1) {  // Retour au début
            return i + 1;
        }
    }
    
    return 0;  // Ne devrait jamais arriver
}
```

### Application

Pour calculer F(10¹⁵) mod 1000 :
1. Calculer π(1000)
2. Réduire : F(10¹⁵) ≡ F(10¹⁵ mod π(1000))
3. Calculer le résultat réduit

## 7. Système de Numération de Fibonacci

### Représentation de Zeckendorf

Tout entier positif peut s'écrire comme somme de nombres de Fibonacci non consécutifs.

**Exemples** :
- 6 = F(5) + F(2) = 5 + 1
- 28 = F(8) + F(5) = 21 + 5 + 2 = F(8) + F(5) + F(3)
- 33 = F(8) + F(6) + F(4) + F(2) = 21 + 8 + 3 + 1

### Propriété

Pas de "11" consécutifs dans la représentation binaire car :
F(n) + F(n+1) = F(n+2)

### Algorithme de conversion

```c
void to_fibonacci_base(unsigned long long n) {
    // 1. Générer les nombres de Fibonacci jusqu'à n
    unsigned long long fibs[100];
    int count = 2;
    fibs[0] = 1; fibs[1] = 2;
    
    while (fibs[count-1] < n) {
        fibs[count] = fibs[count-1] + fibs[count-2];
        count++;
    }
    
    // 2. Algorithme glouton depuis le plus grand
    for (int i = count - 1; i >= 0; i--) {
        if (n >= fibs[i]) {
            printf("1");
            n -= fibs[i];
        } else {
            printf("0");
        }
    }
}
```

## 8. Exponentiation de Matrices

### Formule matricielle

```
[F(n+1)]   [1 1]ⁿ   [1]
[F(n)  ] = [1 0]  × [0]
```

### Multiplication rapide

Utiliser l'exponentiation rapide (méthode russe) sur les matrices 2×2 :

```c
typedef struct {
    unsigned long long a, b, c, d;
} Matrix2x2;

Matrix2x2 mult_matrix(Matrix2x2 m1, Matrix2x2 m2, unsigned long long mod) {
    Matrix2x2 result;
    result.a = (m1.a * m2.a + m1.b * m2.c) % mod;
    result.b = (m1.a * m2.b + m1.b * m2.d) % mod;
    result.c = (m1.c * m2.a + m1.d * m2.c) % mod;
    result.d = (m1.c * m2.b + m1.d * m2.d) % mod;
    return result;
}

unsigned long long fib_matrix(unsigned long long n, unsigned long long mod) {
    if (n == 0) return 0;
    
    Matrix2x2 base = {1, 1, 1, 0};
    Matrix2x2 result = {1, 0, 0, 1};  // Matrice identité
    
    while (n > 0) {
        if (n % 2 == 1) {
            result = mult_matrix(result, base, mod);
        }
        base = mult_matrix(base, base, mod);
        n /= 2;
    }
    
    return result.b;
}
```

**Complexité** : O(log n) - permet de calculer F(10¹⁸) rapidement !

## 9. Le Jeu de Fibonacci

### Règles du jeu

- Pile de n allumettes
- Le premier joueur prend k allumettes (1 ≤ k < n)
- Les joueurs suivants peuvent prendre au maximum 2×(dernière prise)
- Celui qui prend la dernière allumette gagne

### Stratégie gagnante

1. Représenter n en base de Fibonacci
2. Si n est un nombre de Fibonacci : le second joueur gagne
3. Sinon : prendre le plus petit terme de la décomposition

**Exemple** : n = 17 = F(7) + F(4) + F(2) = 13 + 3 + 1
- Prendre 1 allumette (le plus petit terme)
- Forcer l'adversaire dans des nombres de Fibonacci

## 10. Bonnes Pratiques de Programmation

### Vérification des entrées

```c
if (scanf("%u", &n) != 1) {
    fprintf(stderr, "Erreur de lecture\n");
    abort();
}
```

### Éviter les débordements

Toujours appliquer le modulo **à chaque étape** :
```c
// INCORRECT
result = (a + b);
result = result % m;

// CORRECT
result = (a + b) % m;
```

### Formatage du code

Utiliser des outils automatiques :
```bash
indent -kr -nut fichier.c     # Style Kernighan & Ritchie
clang-format -i fichier.c      # Formatage LLVM
```

### Variables et lisibilité

- Utiliser des noms de variables significatifs
- Déclarer les variables en début de bloc (C89/C90)
- Éviter les calculs complexes sur une seule ligne

## 11. Concepts Avancés : Lvalue et Rvalue

### Lvalue (Left value)

Une expression qui désigne un **emplacement mémoire** :
- Variables : `x`, `arr[i]`
- Peuvent apparaître à gauche d'une affectation

### Rvalue (Right value)

Une expression qui a une **valeur** mais pas d'emplacement :
- Constantes : `42`, `3.14`
- Résultats de fonctions : `fib(5)`
- Expressions : `x + y`

**Exemple d'erreur** :
```c
fib(5) = 10;  // ERREUR : fib(5) est une rvalue
2 + 3 = x;    // ERREUR : 2 + 3 est une rvalue
```

## 12. Exercices Pratiques

1. **Fibonacci modulaire** : Calculer F(n) mod m efficacement
2. **Période de Pisano** : Trouver π(m) pour m donné
3. **Base de Fibonacci** : Convertir un nombre en représentation de Zeckendorf
4. **Matrices de Fibonacci** : Implémenter le calcul par exponentiation matricielle
5. **Jeu de Fibonacci** : Créer un programme qui joue optimalement

## Lectures Recommandées

1. **Kernighan & Ritchie** - "The C Programming Language"
   - LA référence du langage C
   - Excellente pour apprendre les bonnes pratiques
   - Code propre et bien structuré

2. **Graham, Knuth & Patashnik** - "Concrete Mathematics"
   - Pour approfondir les aspects mathématiques
   - Excellent chapitre sur les nombres de Fibonacci

3. **Donald Knuth** - "The Art of Computer Programming"
   - Référence algorithmique complète
   - Volume 1 contient une analyse détaillée de Fibonacci

## Points Clés à Retenir

1. **L'implémentation naïve récursive est catastrophique** - Utilisez l'itération
2. **Attention aux débordements** - Utilisez l'arithmétique modulaire
3. **Les périodes de Pisano** permettent de calculer F(n) mod m pour n gigantesque
4. **L'exponentiation matricielle** donne une complexité O(log n)
5. **Toujours vérifier les résultats de `scanf`**
6. **Initialiser tous les tableaux** pour éviter les comportements indéfinis
7. **La représentation de Zeckendorf** est unique et élégante
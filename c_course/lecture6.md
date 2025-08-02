# Leçon de Programmation C : Opérations sur les Bits et Optimisations

## 1. L'Inefficacité du Crible d'Ératosthène Standard

### Le problème de stockage

Dans notre implémentation précédente du crible d'Ératosthène, nous utilisions **1 octet (8 bits)** pour stocker une seule information binaire (premier ou non).

```
Représentation naïve :
[0][0][1][0][1][0]...  (1 octet par nombre)
 2  3  4  5  6  7

Représentation réelle en mémoire :
[00000000][00000000][00000001][00000000]...
```

**Gaspillage** : Nous utilisons 8 bits là où 1 seul suffirait !

### Potentiel d'optimisation

- Réduction immédiate possible : **8×** (en utilisant chaque bit)
- Réduction totale possible : **24×** (avec des optimisations supplémentaires)

## 2. Les Types et l'Adressabilité en C

### Pourquoi pas de type "bit" ?

En C, il n'existe pas de type `bit` car :
- **Le bit n'est pas adressable**
- Un **octet** (byte) est la plus petite unité adressable
- On ne peut pas avoir de pointeur vers un bit individuel

### Qu'est-ce qu'un octet ?

**Définition** : L'octet est la **plus petite unité adressable** sur une machine donnée.
- Généralement 8 bits
- Peut varier selon l'architecture (9, 32 bits...)

## 3. Les Opérations Bitwise (sur les Bits)

### Opérations de base

| Opération | Symbole C | Description | Exemple |
|-----------|-----------|-------------|---------|
| ET (AND) | `&` | Conjonction | `1 & 1 = 1`, `1 & 0 = 0` |
| OU (OR) | `\|` | Disjonction | `1 \| 0 = 1`, `0 \| 0 = 0` |
| XOR | `^` | OU exclusif | `1 ^ 1 = 0`, `1 ^ 0 = 1` |
| NON (NOT) | `~` | Négation | `~1 = 0`, `~0 = 1` |
| Décalage gauche | `<<` | Multiplication par 2ⁿ | `5 << 1 = 10` |
| Décalage droite | `>>` | Division par 2ⁿ | `10 >> 1 = 5` |

### Table de vérité complète

Il existe **16 opérations binaires possibles** sur 2 bits :

```
X | Y | Op0 | Op1 | ... | Op7(OR) | ... | Op15
0 | 0 |  0  |  0  | ... |    0    | ... |  1
0 | 1 |  0  |  1  | ... |    1    | ... |  1
1 | 0 |  0  |  0  | ... |    1    | ... |  1
1 | 1 |  0  |  1  | ... |    1    | ... |  1
```

## 4. Pratique des Calculs Bitwise

### Méthode de calcul mental rapide

Pour calculer rapidement `0xA & 0x28` :
1. Convertir en binaire : `A = 1010`, `28 = 11000`
2. Si les bits d'un nombre sont un sous-ensemble de l'autre, le résultat est immédiat
3. `0xA & 0x28 = 0x08` (seul le bit 8 est commun)

### Propriétés utiles

```c
// Toujours vrai
x | ~x = -1    // Tous les bits à 1
x & ~x = 0     // Tous les bits à 0
x ^ x = 0      // XOR avec soi-même = 0
x ^ 0 = x      // XOR avec 0 = identité
x ^ -1 = ~x    // XOR avec -1 = inversion
```

## 5. Manipulation de Bits Individuels

### Opérations fondamentales

```c
// Tester le bit n
int test_bit(unsigned int x, int n) {
    return (x >> n) & 1;
}

// Mettre le bit n à 1
unsigned int set_bit(unsigned int x, int n) {
    return x | (1 << n);
}

// Mettre le bit n à 0
unsigned int clear_bit(unsigned int x, int n) {
    return x & ~(1 << n);
}

// Inverser le bit n
unsigned int toggle_bit(unsigned int x, int n) {
    return x ^ (1 << n);
}
```

## 6. Optimisations Avancées

### Nombres 2-adiques et astuces

Les nombres 2-adiques permettent de représenter des nombres avec une infinité de chiffres :
- `-1` = `...11111111` (tous les bits à 1)
- `-2` = `...11111110`

### Formules magiques

```c
// Identités fondamentales
x + ~x = -1
-x = ~x + 1

// Effacer le bit le moins significatif
x & (x - 1)

// Isoler le bit le moins significatif
x & -x

// Propager le bit le moins significatif
x | (x - 1)
```

### Comptage de bits (popcount) optimisé

```c
// Version naïve : O(nombre de bits)
int popcount_naive(unsigned int x) {
    int count = 0;
    for (int i = 0; i < 32; i++) {
        count += (x >> i) & 1;
    }
    return count;
}

// Version optimisée : O(nombre de bits à 1)
int popcount_optimized(unsigned int x) {
    int count = 0;
    while (x) {
        x &= (x - 1);  // Efface le bit le moins significatif
        count++;
    }
    return count;
}
```

## 7. Application : Crible d'Ératosthène Optimisé

### Implémentation avec bits

```c
typedef struct {
    unsigned char* bits;
    size_t size;
} BitSieve;

// Initialiser le crible
BitSieve* create_bit_sieve(size_t n) {
    size_t bytes_needed = (n + 7) / 8;  // Arrondi supérieur
    BitSieve* sieve = malloc(sizeof(BitSieve));
    sieve->bits = calloc(bytes_needed, 1);
    sieve->size = n;
    return sieve;
}

// Tester si un nombre est marqué
int is_marked(BitSieve* sieve, size_t n) {
    size_t byte_index = n / 8;
    size_t bit_index = n % 8;
    return (sieve->bits[byte_index] >> bit_index) & 1;
}

// Marquer un nombre
void mark(BitSieve* sieve, size_t n) {
    size_t byte_index = n / 8;
    size_t bit_index = n % 8;
    sieve->bits[byte_index] |= (1 << bit_index);
}
```

## 8. Parcours de Sous-ensembles

### Algorithme de Gosper

Pour parcourir tous les sous-ensembles d'un ensemble représenté par un masque de bits :

```c
void enumerate_subsets(unsigned int mask) {
    unsigned int subset = 0;
    do {
        // Traiter le sous-ensemble
        process_subset(subset);
        
        // Passer au sous-ensemble suivant
        subset = (subset - mask) & mask;
    } while (subset != mask);
}
```

## 9. Visualisations Artistiques : Les Tapis de Knuth

### Génération de motifs

```c
// Fonction générant un motif 2D
void generate_pattern(int width, int height) {
    for (int y = 0; y < height; y++) {
        for (int x = 0; x < width; x++) {
            // Exemple : x & y modulo 17
            int color = ((x & y) % 17) & 1;
            set_pixel(x, y, color);
        }
    }
}
```

Ces fonctions créent des motifs fractals étonnamment beaux !

## 10. Exercices Pratiques

### Exercice 1 : Bits extrêmes
Trouvez l'indice du bit le plus significatif (MSB) et le moins significatif (LSB) définis dans un nombre.

### Exercice 2 : Inversion de bit dans un tableau
Créez un tableau de bytes et inversez un bit spécifique donné par son index global.

### Exercice 3 : Optimisation du crible
Implémentez un crible d'Ératosthène qui :
- Ignore les nombres pairs (sauf 2)
- Utilise le stockage par bits
- Ne stocke que les nombres de la forme 6k±1

## Points Clés à Retenir

1. **Les opérations bitwise sont indépendantes** - pas de propagation entre bits
2. **Pensez en termes de masques** - pour isoler, définir ou effacer des bits
3. **Les astuces bitwise peuvent drastiquement améliorer les performances**
4. **Le stockage par bits économise la mémoire** mais complique l'accès
5. **Les nombres 2-adiques** offrent une perspective élégante sur l'arithmétique binaire

## Attention aux Pièges

1. **Types signés vs non-signés** : Utilisez `unsigned` pour les opérations bitwise
2. **Priorité des opérateurs** : `&` a une priorité plus faible que `==`
3. **Décalages excessifs** : Décaler de plus de bits que la taille du type est indéfini
4. **Endianness** : L'ordre des octets peut varier selon l'architecture

## Pour Aller Plus Loin

- Étudiez les instructions SIMD pour le traitement parallèle de bits
- Explorez les bit fields en C pour un accès structuré aux bits
- Apprenez les techniques de bit-twiddling pour l'optimisation extrême
- Découvrez les applications en cryptographie et compression
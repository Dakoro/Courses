# Leçon de Programmation C : Systèmes de Numération et Algorithmes

## 1. Introduction aux Systèmes de Numération

### Qu'est-ce qu'un système de numération ?

Un **système de numération** est une méthode de représentation des nombres. Il est important de comprendre que :
- Le nombre reste le même dans tous les systèmes
- Seule sa représentation change
- C'est un moyen d'écrire les nombres, pas les nombres eux-mêmes

### Systèmes positionnels

Dans un système positionnel en base n, chaque position représente une puissance de n.

**Exemple avec 13 :**
- En décimal (base 10) : 13₁₀ = 1×10¹ + 3×10⁰ = 10 + 3
- En binaire (base 2) : 1101₂ = 1×2³ + 1×2² + 0×2¹ + 1×2⁰ = 8 + 4 + 0 + 1 = 13
- En octal (base 8) : 15₈ = 1×8¹ + 5×8⁰ = 8 + 5 = 13
- En hexadécimal (base 16) : D₁₆ = 13₁₀

### Notation en C

En langage C, vous pouvez écrire des nombres dans différentes bases :

```c
int a = 926;     // Décimal
int b = 056;     // Octal (commence par 0) - ATTENTION : vaut 46 en décimal !
int c = 0x12;    // Hexadécimal (commence par 0x) - vaut 18 en décimal
```

⚠️ **Piège courant** : Un nombre commençant par 0 est interprété comme octal !

### Affichage des nombres

```c
printf("%d", n);   // Affiche en décimal
printf("%o", n);   // Affiche en octal
printf("%x", n);   // Affiche en hexadécimal
```

## 2. Conversion entre Systèmes de Numération

### Méthode 1 : Par soustraction de puissances

Pour convertir 26₁₀ en binaire :
1. Trouver la plus grande puissance de 2 ≤ 26 : c'est 16 (2⁴)
2. 26 - 16 = 10, donc le bit de 2⁴ est 1
3. Plus grande puissance de 2 ≤ 10 : c'est 8 (2³)
4. 10 - 8 = 2, donc le bit de 2³ est 1
5. 2 - 2 = 0, donc le bit de 2¹ est 1
6. Résultat : 11010₂

### Méthode 2 : Par divisions successives

Pour convertir n en base b :
1. Diviser n par b
2. Le reste est le chiffre de poids faible
3. Continuer avec le quotient
4. Les restes lus de bas en haut donnent le résultat

**Exemple : 26₁₀ en binaire**
- 26 ÷ 2 = 13 reste 0
- 13 ÷ 2 = 6 reste 1
- 6 ÷ 2 = 3 reste 0
- 3 ÷ 2 = 1 reste 1
- 1 ÷ 2 = 0 reste 1
- Résultat : 11010₂ (lire de bas en haut)

### Conversion rapide entre bases liées

Pour les bases qui sont des puissances l'une de l'autre :
- **Binaire ↔ Octal** : grouper/dégrouper par 3 bits
- **Binaire ↔ Hexadécimal** : grouper/dégrouper par 4 bits

**Exemple :**
- 110010110₂ → 110 010 110 → 626₈
- 110010110₂ → 1 1001 0110 → 196₁₆

## 3. Les Tableaux en C

### Déclaration et initialisation

```c
int a[3];                    // Tableau de 3 entiers
int a[3] = {1, 2, 3};       // Initialisation complète
int a[3] = {1};             // a[0]=1, a[1]=0, a[2]=0
```

### Indices

Les indices commencent à **0** :
- `a[0]` : premier élément
- `a[1]` : deuxième élément
- `a[2]` : troisième élément

## 4. Arithmétique Modulaire

### Définition

Deux nombres a et b sont **égaux modulo n** (noté a ≡ b (mod n)) si :
- Ils ont le même reste lors de la division par n
- Ou de manière équivalente : n divise (a - b)

**Exemples :**
- 10 ≡ 6 (mod 2) car tous deux sont pairs
- 8 ≡ 12 (mod 4) car 4 divise (12-8)

### Propriétés

```
(a + b) mod m = ((a mod m) + (b mod m)) mod m
(a × b) mod m = ((a mod m) × (b mod m)) mod m
```

**Application :** Pour calculer 29 × 17 (mod 3)
- 29 mod 3 = 2
- 17 mod 3 = 2
- 2 × 2 = 4
- 4 mod 3 = 1

## 5. Multiplication Paysanne Russe

### Principe

Pour multiplier a × b sans utiliser la multiplication :
1. Écrire b en binaire
2. Calculer a, 2a, 4a, 8a, ... (par additions successives)
3. Additionner les termes correspondant aux bits à 1 de b

**Exemple : 67 × 13**
- 13 = 1101₂ = 8 + 4 + 1
- 67 × 13 = 67×8 + 67×4 + 67×1
- Calculs : 67, 134, 268, 536
- Résultat : 536 + 268 + 67 = 871

### Complexité

Au lieu de b additions, on fait seulement log₂(b) opérations !

## 6. Exponentiation Rapide par Élévation au Carré

### Algorithme

Pour calculer aⁿ mod m efficacement :

```c
int pow_mod(int a, int n, int m) {
    int result = 1;
    int mult = a;
    
    while (n > 0) {
        if (n % 2 == 1) {
            result = (result * mult) % m;
        }
        mult = (mult * mult) % m;
        n = n / 2;
    }
    
    return result;
}
```

### Principe

- On décompose n en binaire
- On calcule a¹, a², a⁴, a⁸, ... par élévations au carré successives
- On multiplie les termes correspondant aux bits à 1 de n

## 7. Algorithme d'Euclide Étendu

### Rappel : Algorithme d'Euclide simple

Pour trouver pgcd(a, b) :
```
a = q₁×b + r₁
b = q₂×r₁ + r₂
r₁ = q₃×r₂ + r₃
...
```

Le pgcd est le dernier reste non nul.

### Version étendue

Trouve x et y tels que : **ax + by = pgcd(a, b)**

**Relations de récurrence :**
- uₖ = uₖ₋₂ - qₖ × uₖ₋₁
- vₖ = vₖ₋₂ - qₖ × vₖ₋₁

**Conditions initiales :**
- u₋₁ = 1, u₀ = 0
- v₋₁ = 0, v₀ = 1

## 8. Exercices Pratiques

### Exercice 1 : Conversion de base
Écrire un programme qui convertit un nombre décimal dans une base donnée (2 à 36).

### Exercice 2 : Système factoriel
Dans le système factoriel :
- 463 = 3×5! + 4×4! + 1×3! + 0×2! + 1×1!
- Se note : 3.4.1.0.1

Écrire un programme de conversion.

### Exercice 3 : Super-exponentiation
La super-exponentiation (tétration) : a↑↑b = a^(a^(a^...)) (b fois)

Calculer a↑↑b mod m efficacement.

## Points Importants à Retenir

1. **Toujours vérifier le résultat de `scanf`**
2. **En C89/C90, déclarer les variables en début de bloc**
3. **Utiliser des parenthèses pour clarifier les priorités des opérateurs**
4. **Attention aux nombres commençant par 0 (notation octale)**
5. **Les indices de tableaux commencent à 0**
6. **L'arithmétique modulaire permet d'éviter les dépassements**
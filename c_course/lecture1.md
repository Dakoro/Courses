````markdown
# Séminaire 1 : Introduction au langage C

---

## 1. Présentation du formateur et des ressources

- **Formateur** : Vladimir Konstantin Igorévich  
- **Années** : 2023–2024  
- **Supports** :  
  - Diapositives mises à jour sur la page du cours (anciennes versions disponibles)  
  - Code d’exemples sur GitHub (lien partagé en classe)  
- **Chat** : toutes les infos et liens seront redondés dans le chat Telegram  

---

## 2. Mise en place de l’environnement

1. **Windows Subsystem for Linux (WSL)** sous Windows :
   - Panneau de configuration → “Activer ou désactiver des fonctionnalités Windows” → “Sous-système Windows pour Linux”  
   - Installation depuis le Microsoft Store : **Ubuntu 22.04**  
2. **Éditeurs possibles** :
   - Vim (exemple en cours)  
   - VS Code, CLion, ou tout autre IDE/éditeur favori  
3. **Compilation & exécution** via la console :
   ```bash
   # écrire le code dans hello.c avec Vim (ou autre)
   gcc -std=c11 -Wall hello.c -o hello
   ./hello
````

---

## 3. Premier programme : Hello, world!

```c
#include <stdio.h>   // directive du préprocesseur

int main(void) {
    printf("Hello, world!\n");   // appel à printf (déclaré dans <stdio.h>)
    // return 0;  // facultatif dans main()
}
```

* **#include** : inclut un fichier d’en‑têtes avant compilation
* **int main(void)** : point d’entrée de tout programme C
* **printf** : affiche une chaîne, retourne le nombre de caractères imprimés (généralement ignoré)

**Exercice :** écrire, compiler et exécuter ce programme. Capturer la sortie et la partager.

---

## 4. Fonctions et signatures

* **Signature** = type de retour + nom + liste de paramètres (types seuls)
* Les noms des arguments n’entrent pas dans la signature.
* **Déclaration** vs **définition** :

  * *déclaration* : indique l’existence (sans corps)
  * *définition* : fournit l’implémentation (corps de la fonction)
* **Règle d’unique définition** (One Definition Rule) : chaque fonction ne doit être définie qu’une seule fois.

---

## 5. Entrée standard et pointeurs

* **scanf** lit depuis stdin ; chaque `%d`, `%lf`, etc. exige une adresse :

  ```c
  int a, b;
  if (scanf("%d %d", &a, &b) != 2) {
      fprintf(stderr, "Erreur de lecture\n");
      abort();
  }
  ```
* **&** : adresse d’une variable
* **\*** : déréférencement d’un pointeur
* **Gestion d’erreur** :

  * Toujours vérifier le retour de `scanf` (nombre d’éléments lus)
  * Tester la division par zéro avant tout `/` ou `%`
  * Utiliser `abort()` pour terminer proprement
  * Réserver `assert()` aux **assertions internes** (invariants)

---

## 6. Programme quotient / reste

```c
#include <stdio.h>
#include <stdlib.h>

int main(void) {
    int a, b, q, r;
    if (scanf("%d %d", &a, &b) != 2 || b == 0) {
        fprintf(stderr, "Usage: ./quotient a b, avec b ≠ 0\n");
        abort();
    }
    q = a / b;      // quotient
    r = a % b;      // reste (attention aux négatifs !)
    printf("Quotient : %d, Reste : %d\n", q, r);
    return 0;
}
```

---

## 7. Pointeurs : concept simplifié

* Imaginez chaque variable comme une **boîte**.
* Un **pointeur** est une **flèche** qui pointe sur cette boîte.
* `int *p = &a;` → p pointe vers a.
* `*p = 2;` → écrit 2 dans la boîte a.

---

## 8. Algorithme d’Euclide pour le PGCD

### 8.1 Définition mathématique

Pour $a, b \neq 0$ posons

$$
a = b \times q + r,\quad 0 \le r < |b|.
$$

Alors $\gcd(a,b) = \gcd(b, r)$.

### 8.2 Version récursive

```c
int gcd_rec(int x, int y) {
    int r = x % y;
    if (r < 0) r += abs(y);       // pour gérer les signes
    if (r == 0) return abs(y);
    return gcd_rec(y, r);
}
```

### 8.3 Version itérative

```c
int gcd_iter(int x, int y) {
    int tmp;
    while (y != 0) {
        tmp = x % y;
        if (tmp < 0) tmp += abs(y);
        x = y;
        y = tmp;
    }
    return abs(x);
}
```

**Exercice :** transformer la version récursive en boucle `while`, tester sur des paires positives et négatives, et déboguer avec GDB.

---

## 9. Boucles en C

| Boucle     | Syntaxe                       | Usage                                              |
| ---------- | ----------------------------- | -------------------------------------------------- |
| `for`      | `for(init; cond; incr) { … }` | la plus générale, toutes les parties optionnelles  |
| `while`    | `while(cond) { … }`           | exécuter tant que `cond` est vrai                  |
| `do…while` | `do { … } while(cond);`       | s’exécute au moins une fois avant de tester `cond` |

* Boucle infinie : `for (;;)`
* Réécriture possible entre `for` et `while` (tout `while` est un `for` sans init/incr).

---

## 10. Débogage avec GDB

1. Compiler avec `-g` :

   ```bash
   gcc -g -Wall prog.c -o prog
   ```
2. Lancer GDB :

   ```bash
   gdb ./prog
   (gdb) break main
   (gdb) run
   (gdb) next       # passer à la ligne suivante
   (gdb) step       # entrer dans la fonction
   (gdb) print x    # afficher la valeur de x
   ```

---

## 11. Types de données et promotions

* **Entiers** (signés/unsigned) : `char`, `short`, `int`, `long`, `long long`

  * Taille en octets donnée par `sizeof(T)`
  * Valeurs signées : $[-2^{n-1}, 2^{n-1}-1]$
  * Unsigned : $[0, 2^n-1]$ (arithmétique modulo $2^n$)
* **Flottants** : `float` (≈7 chiffres significatifs), `double` (≈15), `long double`

  * Représentation par mantisse + exposant (précision limitée)
* **Promotions** appliquées selon les opérateurs (entier→entier plus large, entier+float→float, mix signé/unsigned→unsigned).

---

## 12. Exercices pratiques

1. **Hello, world!**
2. **Quotient & reste** (gestion d’erreurs)
3. **Algorithme d’Euclide**

   * Version naïve → récursive → itérative
   * Gérer les négatifs via un `eumod` (reste euclidien)
4. **Challenges “étoile”** (pour aller plus loin) :

   * **Algorithme étendu d’Euclide** (trouver $u,v$ tels que $u\,a + v\,b = \gcd(a,b)$)
   * **Équations diophantiennes** $a\,x + b\,y = c$

> **Conseil** : Google ! (idées, pas copier‑coller de code), utiliser les plateformes de contest pour s’entraîner (tâche “homework seminar 1” sur le site Olymp).

---

*Bon courage et à bientôt pour le prochain séminaire (tableaux, pointeurs avancés, tableaux dynamiques…) !*

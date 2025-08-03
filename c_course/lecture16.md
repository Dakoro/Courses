# Des Chaînes de Caractères à l'Analyse Lexicale : Fondations de la Compilation en C

## Introduction

### Le Fil d'Ariane de ce Cours

Ce cours propose un parcours fondamental au cœur du langage C, un voyage qui débute avec l'unité la plus élémentaire du texte, le caractère, pour aboutir à la construction d'un composant essentiel de tout compilateur : l'analyseur lexical. Ce cheminement n'est pas arbitraire ; il reflète la nature même du C, un langage où la manipulation des chaînes de caractères n'est pas une simple commodité offerte par une bibliothèque riche, mais une discipline fondamentale. 

Maîtriser les chaînes en C, c'est apprendre à gérer la mémoire manuellement, à comprendre l'interaction entre les pointeurs et les tableaux, et finalement, à saisir comment un programme peut lire, interpréter et transformer son propre code source ou tout autre langage formel.

L'objectif est de vous amener à comprendre que des tâches apparemment simples, comme concaténer deux mots, et des tâches complexes, comme analyser la syntaxe d'un langage assembleur, reposent sur le même socle de connaissances : une compréhension intime de la représentation des données en mémoire et des algorithmes qui opèrent sur ces représentations.

### Philosophie Pédagogique

En nous inspirant de l'esprit de textes fondateurs comme *The C Programming Language* de Brian Kernighan et Dennis Ritchie et *Structure and Interpretation of Computer Programs* de Harold Abelson et Gerald Sussman, nous aborderons le langage C non pas comme une simple collection de syntaxes, mais comme un système cohérent. 

Dans ce système, la gestion de la mémoire, la représentation des données et les algorithmes sont inextricablement liés. L'ambition de ce cours est de vous faire évoluer du statut de simple utilisateur du langage, qui consomme des fonctions de bibliothèque, à celui d'architecte, capable de comprendre les mécanismes internes de ces fonctions, d'en évaluer les limites et, au besoin, de construire vos propres outils de traitement du langage, plus performants ou plus spécialisés.

## Partie 1 : La Représentation des Données Textuelles en C

La gestion des chaînes de caractères en C est radicalement différente de celle des langages de plus haut niveau comme Python ou Java. En C, il n'existe pas de type de données "chaîne" natif. Ce que nous appelons une chaîne est en réalité une convention logicielle bâtie sur les primitives matérielles du langage : les tableaux et les pointeurs. 

Cette distinction est la source de la puissance du C, qui offre un contrôle total sur l'organisation de la mémoire, mais aussi de ses plus grands dangers, tels que les dépassements de tampon et les pointeurs invalides. Comprendre cette dualité conceptuelle est la première et la plus cruciale étape pour maîtriser la programmation système en C.

Le point de départ est l'affirmation que `char` est un type entier. Une déclaration comme `char c = 'A';` est conceptuellement identique à `char c = 65;`. C'est la base sur laquelle tout le reste est construit. Une chaîne est ensuite définie comme un tableau de `char` qui respecte une convention : il doit se terminer par le caractère nul, `\0`. Le langage lui-même ignore cette sémantique ; c'est la bibliothèque standard qui l'impose.

Cette définition nous conduit immédiatement à une discussion sur l'organisation de la mémoire. Un littéral de chaîne, comme `"Bonjour"`, est une constante. Le compilateur le place dans un segment de mémoire en lecture seule. Tenter de le modifier est une erreur grave. En revanche, un tableau initialisé avec un littéral, comme `char s[] = "Bonjour";`, est une copie mutable de cette constante, placée sur la pile ou en mémoire globale, et donc modifiable. 

Cette distinction explique l'une des erreurs les plus fréquentes chez les débutants en C : `char *p = "constante"; p[0] = 'C';` provoque une erreur de segmentation, car elle tente de modifier une zone mémoire protégée, tandis que `char a[] = "mutable"; a[0] = 'M';` est une opération parfaitement légale. L'enseignement de la représentation des chaînes en C est donc indissociable de celui de son modèle mémoire. C'est une philosophie de conception qui confère au programmeur une grande puissance et une responsabilité équivalente.

### 1.1. Le Caractère : L'Atome Numérique du Texte

Fondamentalement, un caractère en langage C est un type de données entier, généralement codé sur 8 bits. La déclaration `char c = 'A';` ne stocke pas le symbole 'A' en tant que tel, mais sa valeur numérique correspondante dans un jeu de caractères. Pour la grande majorité des systèmes, ce jeu de caractères est le Standard Américain pour l'Échange d'Informations (ASCII). Dans la table ASCII, 'A' majuscule correspond à la valeur décimale 65.

L'héritage d'ASCII est immense. Ses 128 premiers codes (de 0 à 127) forment une base commune avec les encodages modernes les plus répandus, comme l'UTF-8. Cette standardisation a une conséquence pratique importante : l'organisation des caractères n'est pas aléatoire. Les chiffres de '0' à '9' et les lettres de 'a' à 'z' et de 'A' à 'Z' y sont stockés de manière contiguë. Cela permet d'effectuer des opérations arithmétiques pour tester la nature d'un caractère. Par exemple, la condition `c >= '0' && c <= '9'` est une manière portable de vérifier si le caractère `c` est un chiffre.

Au-delà des caractères imprimables, la table ASCII définit un ensemble de caractères de contrôle, qui, bien qu'invisibles, sont essentiels à la structuration des données textuelles. Ils sont souvent représentés en C par des séquences d'échappement :

- `\n` (Line Feed) : Le caractère de nouvelle ligne, de code ASCII 10.
- `\t` (Horizontal Tab) : La tabulation horizontale, de code ASCII 9.
- `\r` (Carriage Return) : Le retour chariot, de code ASCII 13. En environnement Windows, un saut de ligne est souvent représenté par la séquence `\r\n`.
- `\0` (Null) : Le caractère nul, de code ASCII 0. Ce caractère est d'une importance capitale en C, car il sert de sentinelle pour marquer la fin d'une chaîne de caractères.

Le tableau suivant récapitule certains de ces caractères de contrôle fondamentaux. Il matérialise le concept qu'un caractère est un nombre et explique l'origine de certains raccourcis clavier système, comme Ctrl+D qui envoie le caractère EOT (End of Transmission) pour signifier la fin d'une entrée standard dans les terminaux Unix.

**Tableau 1 : Sélection de Caractères de Contrôle ASCII**

| Code Décimal | Mnémonique | Représentation en C | Raccourci Clavier | Description |
|--------------|------------|-------------------|-------------------|-------------|
| 0 | NUL | `\0` | Ctrl+@ | Caractère nul, terminateur de chaîne. |
| 4 | EOT | N/A | Ctrl+D | Fin de transmission, utilisé pour fermer le flux d'entrée standard. |
| 9 | HT | `\t` | Tab | Tabulation horizontale. |
| 10 | LF | `\n` | Enter (sur Unix) | Saut de ligne (Line Feed). |
| 13 | CR | `\r` | N/A | Retour chariot (Carriage Return). |
| 26 | SUB | N/A | Ctrl+Z | Substitut, utilisé pour marquer la fin de fichier sur certains systèmes (Windows). |
| 27 | ESC | `\e` (non-standard) | Esc | Caractère d'échappement. |

### 1.2. La Chaîne de Caractères : Convention et Organisation en Mémoire

Comme mentionné, le langage C ne possède pas de type `string` natif. Une chaîne de caractères est une convention : une séquence contiguë d'octets en mémoire, contenant des codes de caractères, et terminée par le caractère nul (`\0`). Ainsi, la chaîne littérale `"hello"` nécessite 6 octets de stockage : un pour chaque lettre et un pour le terminateur nul final. Cette convention est au cœur de toutes les fonctions de la bibliothèque standard `<string.h>`.

Cette approche a des implications profondes sur la manière dont les chaînes sont stockées et manipulées. Il est crucial de distinguer deux modes de déclaration principaux : les littéraux de chaîne et les tableaux de caractères.

**Littéral de chaîne (constante) :**

Lorsque vous écrivez `const char *p = "Hello";`, le compilateur place la séquence de caractères 'H', 'e', 'l', 'l', 'o', '\0' dans un segment de mémoire spécial, souvent appelé `.rodata` (read-only data). Cette zone est protégée en écriture par le système d'exploitation. La variable `p` est un pointeur qui contient l'adresse du premier caractère de cette zone. Toute tentative de modification via ce pointeur, comme `p[0] = 'h';`, conduit à un comportement indéfini, qui se manifeste le plus souvent par une erreur de segmentation (segmentation fault) à l'exécution.

**Tableau de caractères (mutable) :**

Lorsque vous écrivez `char s[] = "Hello";`, le compilateur effectue une opération différente. Il alloue un tableau de 6 `char` (5 pour "Hello" et 1 pour `\0`) dans une zone de mémoire accessible en écriture et y copie le contenu du littéral. L'emplacement de ce tableau dépend de son contexte de déclaration :

- S'il est déclaré à l'intérieur d'une fonction, il est alloué sur la pile.
- S'il est déclaré en dehors de toute fonction (variable globale) ou avec le mot-clé `static`, il est alloué dans le segment de données (`.data` ou `.bss`).

Dans tous les cas, cette chaîne est modifiable : `s[0] = 'h';` est une opération parfaitement valide.

Pour bien programmer en C, il est indispensable de visualiser l'organisation de la mémoire d'un programme. Ce modèle est généralement divisé en trois zones principales pour les données :

1. **Mémoire Globale (Segment de Données)** : Cette zone stocke les variables globales, les variables `static`, et les constantes comme les littéraux de chaîne. Sa taille est fixée à la compilation et sa durée de vie est celle du programme.

2. **La Pile (Stack)** : C'est une zone de mémoire gérée automatiquement selon un principe LIFO (Last-In, First-Out). Elle sert à stocker les variables locales des fonctions et les paramètres passés lors des appels. L'allocation et la libération y sont extrêmement rapides, mais sa taille est limitée.

3. **Le Tas (Heap)** : C'est une région de mémoire non structurée, disponible pour une allocation dynamique à la demande pendant l'exécution du programme, via des fonctions comme `malloc`. La gestion de cette mémoire est entièrement manuelle : le programmeur doit explicitement allouer et libérer chaque bloc. C'est la zone la plus flexible, mais aussi la plus lente et la plus grande source d'erreurs (fuites de mémoire, double libération, fragmentation).

### 1.3. Pointeurs et Tableaux : Dualité et Pièges Courants

La relation entre les pointeurs et les tableaux en C est source de confusion mais aussi de puissance. La règle fondamentale est la suivante : dans la plupart des contextes d'expression, un identifiant de type "tableau de T" est automatiquement converti (on parle de "dégradation") en une expression de type "pointeur vers T" dont la valeur est l'adresse du premier élément du tableau.

C'est pourquoi, si l'on déclare `char s[10];`, les expressions `s` et `&s[0]` sont équivalentes. C'est aussi la raison pour laquelle les déclarations de fonction `void func(char s[])` et `void func(char *s)` sont strictement identiques pour le compilateur : dans les deux cas, la fonction reçoit un pointeur.

Cependant, cette dualité n'est pas une identité parfaite. Un nom de tableau n'est pas une variable modifiable (ce n'est pas une l-value). Il représente une adresse constante, fixée à la compilation. Un pointeur, lui, est une variable qui stocke une adresse et peut être modifié pour pointer ailleurs.

Cette distinction permet de comprendre une série de pièges classiques :

```c
char *heap_ptr = malloc(50);
heap_ptr = "une constante";
```

Cette ligne de code est doublement fautive. Premièrement, elle génère une fuite de mémoire : l'adresse du bloc de 50 octets alloué par `malloc` est écrasée et perdue à jamais, rendant impossible sa libération par `free`. Deuxièmement, elle présente une incompatibilité de type (bien que souvent tolérée avec un avertissement par les anciens compilateurs) : on assigne un `const char *` (l'adresse d'un littéral constant) à un `char *`. Si l'on tente de modifier la chaîne via `heap_ptr` par la suite, on risque une erreur de segmentation.

```c
const char *const_ptr = "Hello";
const_ptr = NULL;
```

Cette opération est parfaitement valide. La déclaration `const char *` signifie "pointeur vers un caractère constant". Cela signifie que la donnée pointée (`*const_ptr`) ne peut pas être modifiée, mais le pointeur lui-même (`const_ptr`) est une variable ordinaire. On peut donc le réassigner pour qu'il pointe vers une autre adresse, y compris l'adresse nulle `NULL`. Pour déclarer un pointeur constant, la syntaxe serait `char * const ptr;`.

```c
char array_name[10];
char *heap_ptr = malloc(50);
array_name = heap_ptr;
```

Cette assignation est invalide et ne compilera pas. Comme expliqué précédemment, `array_name` n'est pas une l-value. C'est une constante d'adresse qui ne peut être modifiée. On ne peut pas faire en sorte que le nom d'un tableau sur la pile pointe soudainement vers une zone du tas. L'inverse, `heap_ptr = array_name;`, est cependant valide, car on assigne une adresse constante à une variable pointeur.

## Partie 2 : Manipulation Standardisée : Les Bibliothèques `<ctype.h>` et `<string.h>`

La bibliothèque standard du C fournit un ensemble de fonctions pour travailler avec les caractères et les chaînes. Ces fonctions ne sont pas des instructions magiques du langage, mais une couche d'abstraction, souvent mince, au-dessus de la manipulation directe de la mémoire. Leur étude révèle une histoire fascinante sur l'évolution de la programmation et la prise de conscience progressive des impératifs de sécurité.

Cette tension entre les pratiques classiques (K&R utilise abondamment ces fonctions) et les exigences de la programmation sécurisée moderne est un point pédagogique essentiel. La meilleure approche n'est pas d'interdire `strcpy` sans explication, mais de l'enseigner comme un cas d'étude. On présente d'abord `strcpy` pour illustrer le concept de copie, puis on introduit immédiatement sa contrepartie plus sûre, `strncpy`, en expliquant en détail pourquoi `strcpy` est insuffisant. Cette démarche enseigne un principe plus profond : celui de la programmation défensive et de la conception d'API robustes qui tiennent compte des limites de la mémoire. La maîtrise de la bibliothèque standard ne consiste pas seulement à savoir ce que font les fonctions, mais aussi et surtout ce qu'elles ne font pas, comme la vérification des bornes.

### 2.1. Classification et Conversion de Caractères (`<ctype.h>`)

Avant de se lancer dans l'écriture de code manuel pour analyser des caractères, le premier réflexe d'un programmeur C aguerri doit être de consulter l'en-tête `<ctype.h>`. Tenter de réinventer la roue en écrivant `c >= '0' && c <= '9'` est non seulement une perte de temps, mais peut aussi être moins portable ou moins complet que les fonctions standardisées.

Cette bibliothèque offre deux familles de fonctions :

**Fonctions de Classification :** Elles prennent un `int` (qui doit être représentable comme un `unsigned char` ou la valeur `EOF`) et retournent une valeur non nulle (vrai) si le caractère appartient à la classe spécifiée, et zéro (faux) sinon.

- `isalpha(c)` : Vrai si `c` est une lettre (majuscule ou minuscule).
- `isdigit(c)` : Vrai si `c` est un chiffre décimal ('0'-'9').
- `isalnum(c)` : Vrai si `c` est une lettre ou un chiffre.
- `isxdigit(c)` : Vrai si `c` est un chiffre hexadécimal ('0'-'9', 'a'-'f', 'A'-'F').
- `islower(c)`, `isupper(c)` : Vrai si `c` est une lettre minuscule ou majuscule, respectivement.
- `isspace(c)` : Vrai pour les caractères d'espacement standards : espace (' '), tabulation (`\t`), nouvelle ligne (`\n`), retour chariot (`\r`), tabulation verticale (`\v`), et form feed (`\f`).
- `ispunct(c)` : Vrai pour les caractères de ponctuation (tout caractère imprimable qui n'est ni un espace, ni alphanumérique).
- `isprint(c)` : Vrai pour tout caractère imprimable, y compris l'espace.

**Fonctions de Conversion :**

- `tolower(c)` : Convertit une lettre majuscule en sa minuscule correspondante. Si `c` n'est pas une majuscule, il est retourné inchangé.
- `toupper(c)` : Convertit une lettre minuscule en sa majuscule correspondante.

Ces fonctions sont les briques de base de tout analyseur lexical. Elles permettent de classifier rapidement et de manière fiable le flux de caractères entrant, étape indispensable pour regrouper ces caractères en unités lexicales (tokens).

### 2.2. Fonctions Essentielles de Manipulation de Chaînes (`<string.h>`)

L'en-tête `<string.h>` contient l'arsenal principal pour la manipulation des chaînes de caractères en C. Le tableau suivant résume les fonctions les plus importantes, en mettant un accent particulier sur leurs points de vigilance.

**Tableau 2 : Fonctions Clés de `<string.h>` et Leurs Risques Associés**

| Fonction | Prototype | Description | Points de Vigilance et Risques de Sécurité |
|----------|-----------|-------------|-------------------------------------------|
| **Copie** | | | |
| `strcpy` | `char *strcpy(char *dest, const char *src);` | Copie la chaîne `src` (y compris `\0`) dans `dest`. | Extrêmement dangereux. Ne vérifie pas la taille de `dest`. Cause principale de dépassements de tampon. À proscrire dans du code sécurisé. |
| `strncpy` | `char *strncpy(char *dest, const char *src, size_t n);` | Copie au plus `n` caractères de `src` dans `dest`. | Plus sûr, mais piégeux. Si `src` a une longueur ≥ `n`, `dest` ne sera pas terminé par `\0`. Si `src` a une longueur < `n`, le reste de `dest` est rempli de `\0`. |
| **Concaténation** | | | |
| `strcat` | `char *strcat(char *dest, const char *src);` | Ajoute la chaîne `src` à la fin de `dest`. | Extrêmement dangereux. Ne vérifie pas si `dest` a assez de place pour contenir le résultat. Risque élevé de dépassement de tampon. |
| `strncat` | `char *strncat(char *dest, const char *src, size_t n);` | Ajoute au plus `n` caractères de `src` à `dest`, et ajoute toujours un `\0`. | Plus sûr. Le paramètre `n` est le nombre maximum de caractères à ajouter, pas la taille totale du buffer. Une mauvaise utilisation peut encore causer des dépassements. |
| **Comparaison** | | | |
| `strcmp` | `int strcmp(const char *s1, const char *s2);` | Compare `s1` et `s2` lexicographiquement. | Sûr. Retourne <0 si `s1<s2`, 0 si `s1==s2`, >0 si `s1>s2`. La comparaison est basée sur les codes ASCII des caractères. |
| `strncmp` | `int strncmp(const char *s1, const char *s2, size_t n);` | Compare au plus les `n` premiers caractères de `s1` et `s2`. | Sûr. Utile pour comparer des préfixes. |
| **Recherche** | | | |
| `strchr` | `char *strchr(const char *s, int c);` | Trouve la première occurrence du caractère `c` dans `s`. | Sûr. Retourne un pointeur vers l'occurrence ou `NULL` si non trouvée. `strrchr` fait de même pour la dernière occurrence. |
| `strstr` | `char *strstr(const char *haystack, const char *needle);` | Trouve la première occurrence de la sous-chaîne `needle` dans `haystack`. | Sûr. Retourne un pointeur vers le début de l'occurrence ou `NULL`. L'algorithme sous-jacent est souvent naïf (voir Partie 3). |
| **Longueur** | | | |
| `strlen` | `size_t strlen(const char *s);` | Calcule la longueur de `s` (nombre de caractères avant `\0`). | Sûr, mais a une complexité en $O(N)$ où $N$ est la longueur de la chaîne. Appeler `strlen` dans une boucle est une source classique d'inefficacité. |

## Partie 3 : Algorithmes Avancés de Recherche de Sous-chaînes

La recherche d'une sous-chaîne (le "motif" ou pattern) dans un texte plus long est un problème fondamental en informatique. La fonction standard `strstr` fournit une solution, mais son implémentation est souvent basée sur l'approche la plus simple, dite "naïve". Cette approche, bien que correcte, peut être très inefficace dans certains cas. L'étude d'algorithmes plus sophistiqués comme Knuth-Morris-Pratt (KMP) et Boyer-Moore est une excellente illustration d'un principe central en algorithmique : le compromis temps-espace via le prétraitement.

L'idée fondamentale est que l'algorithme naïf est "amnésique". Après une non-concordance, il décale le motif d'une seule position et recommence sa recherche à zéro, oubliant toute l'information qu'il a pu acquérir lors de la correspondance partielle précédente. Par exemple, en cherchant le motif $P = \text{"ababc"}$ dans le texte $T = \text{"ababdababc"}$, l'algorithme naïf trouvera une correspondance pour les quatre premiers caractères (abab) avant d'échouer sur le cinquième (d dans T vs c dans P). Il va alors décaler P d'une seule position et tenter de le faire correspondre à partir du deuxième caractère de T, une tentative vouée à l'échec et qui ignore le fait que nous savons déjà que les caractères suivants sont babd....

Les algorithmes KMP et Boyer-Moore formalisent l'intuition que nous pouvons faire mieux. En analysant la structure interne du motif lors d'une phase de prétraitement, nous pouvons calculer la taille des "sauts" à effectuer en cas de non-concordance, évitant ainsi des comparaisons redondantes. Ils incarnent le principe de la réutilisation de l'information : l'information structurelle extraite du motif lors du prétraitement est utilisée pour guider la recherche de manière intelligente. C'est un concept qui transcende la simple recherche de chaînes et s'applique à de nombreux domaines de l'informatique.

### 3.1. L'Approche Naïve : Complexité et Limitations

L'algorithme de recherche par force brute est le plus intuitif. Il consiste à essayer d'aligner le motif P à chaque position possible dans le texte T et à vérifier la correspondance caractère par caractère.

**Pseudocode de l'algorithme naïf :**

```
Algorithme RechercheNaive(Texte T, Motif P)
  n = longueur(T)
  m = longueur(P)
  Pour i de 0 à n-m :
    j = 0
    Tant que j < m et T[i+j] == P[j] :
      j = j + 1
    Si j == m :
      Retourner i  // Correspondance trouvée à l'indice i
  Retourner -1   // Aucune correspondance
```

**Implémentation en C :**

```c
#include <string.h>
#include <stdio.h>

// Renvoie un pointeur vers la première occurrence de needle dans haystack, ou NULL.
const char* naive_strstr(const char* haystack, const char* needle) {
    size_t n = strlen(haystack);
    size_t m = strlen(needle);

    if (m == 0) return haystack; // Un motif vide est toujours trouvé au début

    for (size_t i = 0; i <= n - m; i++) {
        size_t j = 0;
        while (j < m && haystack[i + j] == needle[j]) {
            j++;
        }
        if (j == m) {
            return &haystack[i]; // Correspondance trouvée
        }
    }
    return NULL; // Aucune correspondance
}
```

**Analyse de Complexité :** Dans le pire des cas, la boucle externe s'exécute $n-m+1$ fois, et la boucle interne peut s'exécuter jusqu'à $m$ fois à chaque itération. La complexité temporelle est donc de $O((n-m+1) \times m)$, ce qui se simplifie en $O(n \cdot m)$. Un exemple de pire cas est la recherche d'un motif comme "aaaaab" dans un texte comme "aaaaaaaaab". À chaque position, l'algorithme effectuera presque toutes les comparaisons avant d'échouer sur le dernier caractère.

### 3.2. L'Algorithme de Knuth-Morris-Pratt (KMP)

L'algorithme KMP améliore l'approche naïve en utilisant l'information contenue dans le motif lui-même pour éviter de reculer dans le texte. Lorsqu'une non-concordance se produit, au lieu de simplement décaler le motif d'une position, KMP utilise une table précalculée pour déterminer le plus grand décalage possible qui ne manquera aucune occurrence potentielle.

**Théorie et Fonction d'Échec (Tableau LPS) :** Le cœur de KMP est la "fonction d'échec", souvent implémentée comme un tableau LPS (Longest Proper Prefix which is also Suffix). Pour chaque position $j$ dans le motif $P$, $\text{LPS}[j]$ stocke la longueur du plus long préfixe propre de $P[0...j]$ qui est également un suffixe de $P[0...j]$. Un préfixe "propre" est un préfixe qui n'est pas la chaîne entière.

Par exemple, pour le motif $P = \text{"ababc"}$ :
$P[0...3] = \text{"abab"}$. Les préfixes propres sont "a", "ab", "aba". Les suffixes sont "b", "ab", "bab". Le plus long en commun est "ab", de longueur 2. Donc $\text{LPS}[3] = 2$.

Cette table permet, après une non-concordance au caractère suivant $P[j]$, de décaler le motif de sorte que le préfixe "ab" (de longueur 2) s'aligne avec le suffixe "ab" du texte qui vient d'être reconnu, et de reprendre la comparaison sans reculer dans le texte.

**Pseudocode de KMP :**

```
// Phase de prétraitement
Algorithme CalculerLPS(Motif P)
  m = longueur(P)
  lps = tableau de taille m
  longueur = 0
  lps[0] = 0
  i = 1
  Tant que i < m :
    Si P[i] == P[longueur] :
      longueur = longueur + 1
      lps[i] = longueur
      i = i + 1
    Sinon :
      Si longueur != 0 :
        longueur = lps[longueur - 1]
      Sinon :
        lps[i] = 0
        i = i + 1
  Retourner lps

// Phase de recherche
Algorithme RechercheKMP(Texte T, Motif P)
  n = longueur(T)
  m = longueur(P)
  lps = CalculerLPS(P)
  i = 0 // indice pour T
  j = 0 // indice pour P
  Tant que i < n :
    Si P[j] == T[i] :
      i = i + 1
      j = j + 1
    Si j == m :
      Retourner i - j // Correspondance trouvée
      j = lps[j - 1]
    Sinon si i < n et P[j] != T[i] :
      Si j != 0 :
        j = lps[j - 1]
      Sinon :
        i = i + 1
  Retourner -1
```

**Implémentation Complète en C :**

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

void computeLPS(const char* pattern, size_t m, int* lps) {
    size_t length = 0;
    lps[0] = 0;
    size_t i = 1;
    while (i < m) {
        if (pattern[i] == pattern[length]) {
            length++;
            lps[i] = length;
            i++;
        } else {
            if (length != 0) {
                length = lps[length - 1];
            } else {
                lps[i] = 0;
                i++;
            }
        }
    }
}

const char* kmp_search(const char* text, const char* pattern) {
    size_t n = strlen(text);
    size_t m = strlen(pattern);
    if (m == 0) return text;

    int* lps = (int*)malloc(sizeof(int) * m);
    if (!lps) return NULL;
    computeLPS(pattern, m, lps);

    size_t i = 0; // index for text
    size_t j = 0; // index for pattern
    while (i < n) {
        if (pattern[j] == text[i]) {
            i++;
            j++;
        }
        if (j == m) {
            free(lps);
            return &text[i - j];
        } else if (i < n && pattern[j] != text[i]) {
            if (j != 0) {
                j = lps[j - 1];
            } else {
                i++;
            }
        }
    }
    free(lps);
    return NULL;
}
```

**Analyse de Complexité :** La phase de prétraitement (`computeLPS`) a une complexité de $O(m)$. La phase de recherche est en $O(n)$ car l'indice `i` du texte n'est jamais décrémenté. La complexité totale de KMP est donc de $O(n+m)$, une amélioration significative par rapport à l'approche naïve.

### 3.3. L'Algorithme de Boyer-Moore

L'algorithme de Boyer-Moore est souvent encore plus rapide en pratique que KMP, en particulier pour les alphabets de grande taille (comme l'ASCII complet) et les motifs longs. Son idée clé, contre-intuitive, est de comparer le motif et le texte de droite à gauche. Cela permet, en cas de non-concordance, d'effectuer des "sauts" potentiellement très grands.

L'algorithme utilise deux heuristiques pour calculer la taille du décalage :

1. **Heuristique du mauvais caractère (Bad Character)** : Lorsqu'une non-concordance se produit entre le caractère `c` du texte et le caractère `p` du motif, on regarde la dernière (la plus à droite) occurrence de `c` dans le motif. On décale alors le motif pour aligner cette dernière occurrence avec le `c` du texte. Si `c` n'apparaît pas du tout dans le motif, on peut décaler le motif de toute sa longueur pour le placer juste après le caractère `c`.

2. **Heuristique du bon suffixe (Good Suffix)** : Cette heuristique s'applique lorsque la comparaison de droite à gauche a permis de faire correspondre un suffixe du motif avec le texte. Si une non-concordance survient, on cherche une autre occurrence de ce "bon suffixe" dans le motif et on décale ce dernier pour aligner les deux occurrences. C'est une heuristique plus complexe mais puissante.

En pratique, l'algorithme calcule les décalages suggérés par les deux heuristiques et applique le plus grand des deux, maximisant ainsi la progression dans le texte. La complexité dans le pire des cas reste $O(n \cdot m)$, mais des cas aussi défavorables sont extrêmement rares. Dans le meilleur des cas, si à chaque tentative le premier caractère comparé (le dernier du motif) ne se trouve pas dans le motif, l'algorithme effectue $n/m$ comparaisons, pour une complexité de $O(n/m)$.

**Tableau 3 : Comparaison des Algorithmes de Recherche de Sous-chaîne**

| Algorithme | Complexité Prétraitement | Complexité Recherche (Pire cas) | Complexité Recherche (Meilleur cas) | Espace Mémoire Auxiliaire |
|------------|-------------------------|--------------------------------|-----------------------------------|---------------------------|
| Naïf | Aucun | $O(n \cdot m)$ | $O(n)$ | $O(1)$ |
| Rabin-Karp | $O(m)$ | $O(n \cdot m)$ | $O(n+m)$ | $O(1)$ |
| Knuth-Morris-Pratt (KMP) | $O(m)$ | $O(n)$ | $O(n)$ | $O(m)$ |
| Boyer-Moore | $O(m + |\Sigma|)$ | $O(n \cdot m)$ | $O(n/m)$ | $O(m + |\Sigma|)$ |

**Légende :** $n$ = longueur du texte, $m$ = longueur du motif, $|\Sigma|$ = taille de l'alphabet.

## Partie 4 : Gestion Dynamique de la Mémoire pour les Chaînes

Lorsque la taille d'une chaîne de caractères n'est pas connue à la compilation, ou lorsqu'elle doit évoluer pendant l'exécution du programme (par exemple, par concaténations successives), il est indispensable d'utiliser l'allocation dynamique de mémoire sur le tas. Les fonctions `malloc`, `calloc`, `realloc` et `free` de `<stdlib.h>` sont les outils pour cette tâche.

L'un des problèmes les plus intéressants dans ce domaine est celui de la stratégie de redimensionnement d'un buffer. Cette stratégie de doublement de la taille du buffer (`new_size = old_size * 2`) peut être qualifiée de "standard d'or", par opposition à une stratégie de croissance arithmétique comme `new_size = old_size + 10`, jugée inefficace.

Ce choix n'est pas une question de goût, mais le résultat d'une analyse de complexité amortie. Une stratégie de croissance géométrique, qui peut sembler dispendieuse à court terme (on alloue potentiellement beaucoup plus que nécessaire), est en réalité la seule qui garantit une complexité amortie en temps constant, $O(1)$, pour les opérations d'ajout. C'est un résultat non trivial et absolument crucial pour la performance des structures de données dynamiques, comme `std::vector` en C++.

Analysons les deux stratégies. Supposons que le coût d'une réallocation est proportionnel à la taille de l'ancien buffer à recopier.

**Stratégie de croissance arithmétique (+C) :** Pour faire grandir un buffer jusqu'à une taille $N$, il faudra environ $N/C$ réallocations. Le coût total des copies sera proportionnel à la somme $C+2C+3C+...+N$, qui est de l'ordre de $O(N^2)$. Le coût moyen (amorti) par élément ajouté est donc de $O(N)$, ce qui est très inefficace.

**Stratégie de croissance géométrique (*k, par ex. *2) :** Pour atteindre une taille $N$, il ne faudra qu'environ $\log_k(N)$ réallocations. Le coût total des copies sera proportionnel à la somme de la série géométrique $1+k+k^2+...+N/k$, dont la somme est de l'ordre de $O(N)$. Le coût total pour $N$ ajouts étant linéaire, le coût amorti par ajout est constant, $O(1)$.

Cette analyse mathématique démontre la supériorité écrasante de la croissance géométrique. Il est donc fondamental d'enseigner non seulement comment utiliser `realloc`, mais aussi pourquoi cette stratégie de doublement est la bonne pratique pour garantir des performances prévisibles et efficaces.

### 4.1. Allocation Initiale : `malloc` et `calloc`

- `void *malloc(size_t size)` : Cette fonction alloue un bloc de `size` octets contigus sur le tas et retourne un pointeur `void *` vers le début de ce bloc. En cas d'échec (plus de mémoire disponible), elle retourne `NULL`. La zone mémoire allouée n'est pas initialisée ; elle contient des données résiduelles ("garbage").

  **Exemple :** `char *str = (char *)malloc(100 * sizeof(char));`

- `void *calloc(size_t nmemb, size_t size)` : Cette fonction alloue un bloc de mémoire pour un tableau de `nmemb` éléments, chacun de taille `size`. La principale différence avec `malloc` est que la zone mémoire allouée est initialisée à zéro sur tous ses bits. C'est particulièrement utile pour allouer des chaînes de caractères (le terminateur nul est déjà en place) ou des tableaux de pointeurs (initialisés à `NULL`).

  **Exemple :** `int *arr = (int *)calloc(50, sizeof(int));`

- `void free(void *ptr)` : Cette fonction libère le bloc de mémoire précédemment alloué (par `malloc`, `calloc` ou `realloc`) et pointé par `ptr`. Une fois libérée, la mémoire ne doit plus jamais être accédée. Oublier d'appeler `free` pour chaque allocation est une fuite de mémoire. Appeler `free` deux fois sur le même pointeur (double free) est un bug grave qui peut corrompre le gestionnaire de mémoire du tas.

### 4.2. Redimensionnement de Buffers : La Puissance et les Dangers de `realloc`

La fonction `void *realloc(void *ptr, size_t new_size)` permet de modifier la taille d'un bloc de mémoire existant. Elle peut soit étendre ou réduire le bloc sur place si possible, soit allouer un nouveau bloc de la taille désirée, y copier le contenu de l'ancien bloc, et libérer l'ancien. C'est un outil puissant mais qui doit être utilisé avec une extrême précaution.

Le principal danger réside dans la gestion de l'échec. Si `realloc` ne parvient pas à allouer la nouvelle taille, il retourne `NULL`, mais l'ancien bloc de mémoire pointé par `ptr` reste valide et alloué. Le patron de conception suivant est donc une erreur critique :

```c
// MAUVAIS PATRON - NE PAS UTILISER
ptr = realloc(ptr, new_size);
if (ptr == NULL) {
    // Erreur : on a perdu le pointeur original et créé une fuite de mémoire!
    exit(EXIT_FAILURE);
}
```

Le bon patron de conception utilise une variable temporaire pour stocker le résultat de `realloc`, ce qui permet de gérer l'erreur sans perdre le pointeur original :

```c
// BON PATRON DE CONCEPTION
void *temp = realloc(ptr, new_size);
if (temp == NULL) {
    // Gérer l'erreur de réallocation.
    // L'ancien bloc pointé par 'ptr' est toujours valide et doit être libéré.
    free(ptr);
    fprintf(stderr, "Erreur de réallocation de mémoire\n");
    exit(EXIT_FAILURE);
} else {
    // La réallocation a réussi, on peut mettre à jour le pointeur original.
    ptr = temp;
}
```

### 4.3. Cas Pratique : Implémentation d'une Concaténation Sécurisée

Mettons en pratique ces concepts en implémentant la fonction `strcat_realloc`, une fonction qui doit concaténer une chaîne source à une chaîne de destination allouée sur le tas, en redimensionnant cette dernière de manière sûre et efficace si nécessaire.

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

// Structure pour gérer une chaîne dynamique et sa capacité.
typedef struct {
    char *buffer;
    size_t capacity;
} DynamicString;

// Concatène 'src' à 'dest'. Redimensionne le buffer de 'dest' si nécessaire.
// Renvoie 0 en cas de succès, -1 en cas d'échec.
int strcat_realloc(DynamicString *dest, const char *src) {
    size_t dest_len = strlen(dest->buffer);
    size_t src_len = strlen(src);
    size_t required_capacity = dest_len + src_len + 1; // +1 pour le '\0'

    if (dest->capacity < required_capacity) {
        // Stratégie de croissance géométrique (doublement)
        size_t new_capacity = dest->capacity;
        while (new_capacity < required_capacity) {
            new_capacity *= 2;
        }
        
        // Bon patron de conception pour realloc
        char *new_buffer = (char *)realloc(dest->buffer, new_capacity);
        if (new_buffer == NULL) {
            fprintf(stderr, "Erreur de réallocation\n");
            // L'ancien buffer est toujours alloué, mais on ne le libère pas ici
            // car l'appelant pourrait vouloir le gérer.
            return -1; 
        }
        dest->buffer = new_buffer;
        dest->capacity = new_capacity;
    }

    // Concaténation sécurisée
    strcat(dest->buffer, src); // On sait maintenant qu'il y a assez de place
    return 0;
}

int main() {
    DynamicString my_string;
    my_string.capacity = 10;
    my_string.buffer = (char *)malloc(my_string.capacity);
    if (!my_string.buffer) return 1;
    strcpy(my_string.buffer, "Hello");

    printf("Avant: '%s', Capacité: %zu\n", my_string.buffer, my_string.capacity);
    
    strcat_realloc(&my_string, ", World! This is a long string.");

    printf("Après: '%s', Capacité: %zu\n", my_string.buffer, my_string.capacity);

    free(my_string.buffer);
    return 0;
}
```

## Partie 5 : Application : Construction d'un Analyseur Lexical

Nous arrivons maintenant au point culminant de notre parcours : l'application de toutes les notions vues jusqu'à présent à un problème concret et fondamental en informatique, la construction d'un analyseur lexical (ou scanner). Cette tâche consiste à prendre un programme source sous forme de texte brut et à le décomposer en une séquence de "mots" ou d'unités significatives appelées tokens. C'est la toute première phase de la compilation.

Le problème central de cette tâche est de représenter des données hétérogènes. Une ligne de code comme `"ADD R1, 14"` contient des éléments de nature très différente : une instruction (ADD), un identifiant de registre (R1), un séparateur (,) et une constante numérique (14). Nous avons besoin d'une structure de données unique, le Token, capable de représenter n'importe lequel de ces éléments.

Une première approche naïve consisterait à utiliser une `struct` avec un champ pour chaque possibilité. Ce serait un gaspillage de mémoire énorme, car un token ne peut être qu'une seule de ces choses à la fois. Une `union`, qui superpose ses champs à la même adresse mémoire, résout le problème de l'espace. Mais elle introduit un nouveau problème : comment savoir quel champ de l'union est actuellement valide? L'information de type est perdue.

La solution élégante et idiomatique en C est l'**union étiquetée** (tagged union). C'est un patron de conception qui combine une `struct`, une `enum` et une `union`. La `struct` englobe les deux autres : l'`enum` sert d'étiquette (le tag) pour indiquer quel type de donnée est actuellement stocké, et l'`union` contient la valeur elle-même.

```c
typedef enum { T_MNEMONIC, T_REGISTER, T_IMMEDIATE,... } TokenType;
typedef struct {
    TokenType type; // Le tag : indique ce que contient l'union
    union {
        Mnemonic op_code;
        int register_number;
        int immediate_value;
    } value; // La valeur
} Token;
```

Cette structure n'est pas un simple "truc" de programmation C. C'est une implémentation manuelle et explicite d'un concept fondamental de la théorie des types : le **type somme** (ou type variant). Des langages plus modernes comme Rust, OCaml ou Haskell intègrent ce concept de manière native. En le construisant nous-mêmes en C, nous ne résolvons pas seulement un problème pratique, nous faisons une leçon de conception de langage de programmation. Nous comprenons, à un niveau fondamental, comment modéliser des données qui peuvent prendre plusieurs formes distinctes.

### 5.1. Introduction à la Théorie de la Compilation

Un compilateur est un traducteur qui transforme un programme d'un langage source vers un langage cible (souvent du code machine). Ce processus est classiquement décomposé en plusieurs phases :

1. **Analyse Lexicale (Scanning)** : Le flux de caractères du programme source est lu et groupé en une séquence d'unités lexicales, ou tokens. Les commentaires et les espaces superflus sont généralement éliminés à cette étape.

2. **Analyse Syntaxique (Parsing)** : Le flux de tokens est analysé pour vérifier s'il respecte les règles de la grammaire du langage. Le résultat est une représentation hiérarchique du programme, typiquement un Arbre de Syntaxe Abstraite (AST).

Pour notre projet, il est essentiel de maîtriser les définitions formelles suivantes :

- **Lexème** : Une séquence de caractères dans le code source qui correspond à un motif. Exemples : "ADD", "14", "R1".
- **Unité Lexicale (Token)** : Une abstraction d'un lexème. Elle est généralement représentée par un couple (nom_du_token, valeur_attributaire). Par exemple, le lexème "14" serait représenté par le token (NUMBER, 14).
- **Motif (Pattern)** : Une description, souvent sous forme d'expression régulière, de l'ensemble des lexèmes qui peuvent correspondre à un type de token. Par exemple, le motif pour un token NUMBER pourrait être l'expression régulière `[0-9]+`.

### 5.2. Spécification d'un Mini-Langage Assembleur

Pour notre analyseur lexical, nous allons définir un langage assembleur très simple. Une instruction aura la forme : `MNEMONIQUE OPERANDE1, OPERANDE2`.

Le tableau suivant formalise les unités lexicales que notre analyseur devra reconnaître. C'est la feuille de route pour notre implémentation.

**Tableau 4 : Spécification des Tokens pour le Mini-Assembleur**

| Catégorie de Lexème | Motif (Expression Régulière Simplifiée) | Exemple de Lexème | Type de Token (enum) | Donnée Associée (champ de l'union) |
|-------------------|----------------------------------------|-------------------|---------------------|-----------------------------------|
| Mnémonique | `ADD \| SUB \| OR` | "ADD" | `T_MNEMONIC` | `op_code` (un enum pour les mnémoniques) |
| Registre | `R[0-9]+` | "R1" | `T_REGISTER` | `register_number` (un int) |
| Constante Immédiate | `[0-9]+` | "14" | `T_IMMEDIATE` | `immediate_value` (un int) |
| Séparateur | `,` | "," | `T_COMMA` | Aucune |
| Fin de Ligne | `\n` | "\n" | `T_EOL` | Aucune |
| Fin de Fichier | EOF | EOF | `T_EOF` | Aucune |

### 5.3. La Structure de Données du Token : L'Union Étiquetée

Comme développé dans l'introduction de cette partie, la structure de données idéale pour représenter nos tokens hétérogènes est l'union étiquetée. Voici une implémentation complète en C pour les types dont nous aurons besoin.

```c
#include <stdbool.h>

// Énumération pour les mnémoniques spécifiques
typedef enum {
    M_ADD, M_SUB, M_OR
} Mnemonic;

// Énumération pour les types de tokens (le "tag")
typedef enum {
    TOKEN_MNEMONIC,
    TOKEN_REGISTER,
    TOKEN_IMMEDIATE,
    TOKEN_COMMA,
    TOKEN_EOL, // End of Line
    TOKEN_EOF, // End of File
    TOKEN_ERROR
} TokenType;

// La structure Token, implémentant une union étiquetée
typedef struct {
    TokenType type;
    union {
        Mnemonic mnemonic_val;
        int register_num;
        int immediate_val;
    } value;
    // On pourrait aussi ajouter des informations de débogage, comme le numéro de ligne.
    int line_num; 
} Token;
```

### 5.4. Implémentation d'un Analyseur Lexical Manuel en C

Nous allons maintenant écrire la fonction principale de notre analyseur, `get_next_token()`. Elle prendra en entrée un pointeur vers la chaîne de caractères à analyser et retournera le prochain Token trouvé, en faisant avancer le pointeur. Elle ignorera les espaces et les tabulations.

**Pseudocode détaillé de l'analyseur lexical :**

```
Fonction get_next_token(pointeur **input_str)
  // 1. Ignorer les espaces
  Tant que le caractère courant est un espace ou une tabulation :
    avancer le pointeur *input_str

  // 2. Gérer la fin de la chaîne
  Si le caractère courant est '\0' ou EOF :
    Retourner un token de type TOKEN_EOF

  // 3. Gérer la fin de ligne
  Si le caractère courant est '\n' :
    avancer le pointeur *input_str
    Retourner un token de type TOKEN_EOL

  // 4. Gérer les séparateurs
  Si le caractère courant est ',' :
    avancer le pointeur *input_str
    Retourner un token de type TOKEN_COMMA

  // 5. Gérer les identifiants (mnémoniques et registres)
  Si le caractère courant est une lettre :
    Lire le lexème complet (suite de lettres et chiffres) dans un buffer temporaire
    Si le lexème est "ADD", "SUB", ou "OR" :
      Retourner un token TOKEN_MNEMONIC avec la valeur correspondante
    Si le lexème commence par 'R' suivi de chiffres :
      Convertir les chiffres en entier
      Retourner un token TOKEN_REGISTER avec le numéro du registre
    Sinon :
      Retourner un token TOKEN_ERROR

  // 6. Gérer les nombres (constantes immédiates)
  Si le caractère courant est un chiffre :
    Lire le lexème numérique complet dans un buffer temporaire
    Convertir le buffer en entier (ex: avec atoi)
    Retourner un token TOKEN_IMMEDIATE avec la valeur entière

  // 7. Gérer les erreurs
  Sinon :
    avancer le pointeur *input_str
    Retourner un token TOKEN_ERROR
```

**Implémentation Complète en C :**

Voici une implémentation fonctionnelle qui prend une chaîne en entrée et la transforme en un tableau de tokens.

```c
#include <ctype.h>
#include <string.h>
#include <stdlib.h>
#include <stdio.h>

//... (inclure les définitions de Mnemonic, TokenType, Token d'en-haut)...

// Fonction pour obtenir le prochain token de la chaîne d'entrée.
// Le pointeur 'cursor' est passé par adresse pour qu'on puisse le modifier.
Token get_next_token(const char **cursor) {
    Token token;
    token.line_num = 0; // Pourrait être amélioré pour suivre les lignes

    // 1. Ignorer les espaces
    while (isspace(**cursor) && **cursor != '\n') {
        (*cursor)++;
    }

    // 2. Gérer la fin de la chaîne/fichier
    if (**cursor == '\0') {
        token.type = TOKEN_EOF;
        return token;
    }
    
    // 3. Gérer la fin de ligne
    if (**cursor == '\n') {
        (*cursor)++;
        token.type = TOKEN_EOL;
        return token;
    }

    // 4. Gérer les séparateurs
    if (**cursor == ',') {
        (*cursor)++;
        token.type = TOKEN_COMMA;
        return token;
    }

    // 5. Gérer les identifiants et mnémoniques
    if (isalpha(**cursor)) {
        char buffer[32];
        int i = 0;
        while (isalnum(**cursor) && i < 31) {
            buffer[i++] = *(*cursor)++;
        }
        buffer[i] = '\0';

        if (strcmp(buffer, "ADD") == 0) {
            token.type = TOKEN_MNEMONIC;
            token.value.mnemonic_val = M_ADD;
        } else if (strcmp(buffer, "SUB") == 0) {
            token.type = TOKEN_MNEMONIC;
            token.value.mnemonic_val = M_SUB;
        } else if (strcmp(buffer, "OR") == 0) {
            token.type = TOKEN_MNEMONIC;
            token.value.mnemonic_val = M_OR;
        } else if (buffer[0] == 'R' && isdigit(buffer[1])) {
            token.type = TOKEN_REGISTER;
            token.value.register_num = atoi(&buffer[1]);
        } else {
            token.type = TOKEN_ERROR;
        }
        return token;
    }

    // 6. Gérer les nombres
    if (isdigit(**cursor)) {
        char buffer[32];
        int i = 0;
        while (isdigit(**cursor) && i < 31) {
            buffer[i++] = *(*cursor)++;
        }
        buffer[i] = '\0';
        token.type = TOKEN_IMMEDIATE;
        token.value.immediate_val = atoi(buffer);
        return token;
    }

    // 7. Gérer les erreurs
    (*cursor)++;
    token.type = TOKEN_ERROR;
    return token;
}

int main() {
    const char *code = "ADD R1, 42\nSUB R2, R1\n";
    const char *p = code;
    Token t;

    do {
        t = get_next_token(&p);
        switch (t.type) {
            case TOKEN_MNEMONIC: printf("MNEMONIC(%d)\n", t.value.mnemonic_val); break;
            case TOKEN_REGISTER: printf("REGISTER(%d)\n", t.value.register_num); break;
            case TOKEN_IMMEDIATE: printf("IMMEDIATE(%d)\n", t.value.immediate_val); break;
            case TOKEN_COMMA: printf("COMMA\n"); break;
            case TOKEN_EOL: printf("EOL\n"); break;
            case TOKEN_EOF: printf("EOF\n"); break;
            case TOKEN_ERROR: printf("ERROR\n"); break;
        }
    } while (t.type != TOKEN_EOF && t.type != TOKEN_ERROR);

    return 0;
}
```

## Conclusion : Synthèse et Perspectives

### Récapitulatif du Parcours

Au cours de cette session, nous avons entrepris un voyage complet à travers le monde de la manipulation de texte en langage C. Partant de l'axiome fondamental que `char` est un type entier, nous avons construit une compréhension profonde de la manière dont le C représente et manipule les données textuelles. Nous avons vu que les chaînes de caractères ne sont pas une primitive du langage, mais une convention basée sur le terminateur nul `\0`, une convention qui impose une discipline rigoureuse de gestion de la mémoire.

Nous avons exploré les outils fournis par la bibliothèque standard, `<ctype.h>` et `<string.h>`, non pas comme une simple boîte à outils, mais comme une étude de cas sur l'évolution de la programmation vers plus de sécurité, en contrastant les fonctions historiques comme `strcpy` avec leurs alternatives modernes plus robustes. Nous avons ensuite plongé dans le monde des algorithmes, en montrant comment des approches sophistiquées comme KMP et Boyer-Moore, grâce au principe du prétraitement, surpassent largement l'approche naïve. 

Enfin, nous avons mis en application toutes ces connaissances pour construire un composant logiciel non trivial : un analyseur lexical. Ce projet nous a permis d'aborder des concepts avancés de structuration de données, notamment l'union étiquetée, que nous avons identifiée comme l'implémentation manuelle d'un type somme, un concept puissant de la théorie des langages.

### Ouverture vers la Suite

L'analyse lexicale, bien que complexe, n'est que la première étape du processus de compilation. Le flux de tokens que notre programme génère est désormais prêt à être consommé par la phase suivante : l'analyse syntaxique (parsing). Cette phase a pour rôle de vérifier que la séquence de tokens respecte la grammaire formelle du langage (par exemple, qu'une mnémonique est bien suivie de deux opérandes séparés par une virgule) et de construire une représentation structurée du programme, l'Arbre de Syntaxe Abstraite (AST).

### Les Outils Industriels

Il est important de noter que la création manuelle d'analyseurs lexicaux, bien qu'étant un excellent exercice pédagogique, est rare dans la pratique industrielle. Ce processus est aujourd'hui largement automatisé par des générateurs d'analyseurs. Des outils comme **Flex** prennent en entrée un fichier de spécifications décrivant les tokens à l'aide d'expressions régulières et génèrent automatiquement le code C d'un analyseur lexical hautement optimisé. 

Flex est presque toujours utilisé en tandem avec **Bison**, un générateur d'analyseur syntaxique. Ces outils incarnent le principe ultime de l'abstraction en programmation : ils nous permettent de décrire ce que notre langage doit reconnaître, et se chargent du comment l'implémenter efficacement. La compréhension des mécanismes internes que nous avons construits manuellement aujourd'hui est cependant la clé pour utiliser ces outils puissants de manière efficace et éclairée.
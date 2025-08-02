# Leçon : Pointeurs, Références et Surcharge en C++

## Introduction

Cette leçon couvre les concepts avancés du C++ concernant la gestion de la mémoire, les références et la surcharge de fonctions. Nous aborderons les différences fondamentales entre C et C++ en matière de gestion mémoire et comment le C++ résout certains problèmes inhérents au C.

## 1. Les opérateurs new et delete

### 1.1 Problèmes avec malloc/free

Les fonctions `malloc` et `free` du C ne connaissent rien des constructeurs et destructeurs. Elles ne font que :
- Allouer de la mémoire brute
- Libérer de la mémoire brute

**Problème** : Si vous avez un objet avec un constructeur non-trivial et que vous l'allouez avec `malloc`, l'objet sera dans un état incohérent car le constructeur n'aura pas été appelé.

### 1.2 La solution C++ : new et delete

Le C++ introduit deux opérateurs :
- `new` : alloue la mémoire ET appelle le constructeur
- `delete` : appelle le destructeur ET libère la mémoire

**Important** : Il existe en réalité 3 paires d'opérateurs :

1. `new` / `delete` - pour les objets simples
2. `new[]` / `delete[]` - pour les tableaux
3. `malloc` / `free` - hérités du C

### 1.3 Pourquoi deux versions de new ?

```cpp
// new simple connaît le type et donc la taille
T* p = new T;        // sait qu'il faut allouer sizeof(T)
delete p;            // sait qu'il faut libérer sizeof(T)

// new[] doit stocker le nombre d'éléments
T* arr = new T[n];   // doit stocker 'n' quelque part
delete[] arr;        // doit savoir combien d'éléments détruire
```

**Danger** : Ne JAMAIS mélanger les opérateurs !

```cpp
T* p = new T[5];
delete p;        // ERREUR ! Comportement indéfini
                 // delete[] lira une valeur aléatoire comme nombre d'éléments
```

### 1.4 Comment éviter les erreurs ?

**Règle d'or** : Utilisez l'encapsulation
- `new` uniquement dans les constructeurs
- `delete` uniquement dans les destructeurs
- Ainsi, vous savez toujours quel `delete` correspond à quel `new`

## 2. Les références pendantes (Dangling References)

### 2.1 Qu'est-ce qu'une référence pendante ?

Une référence pendante est une référence qui pointe vers un objet dont la durée de vie est terminée. C'est l'équivalent du "perroquet mort" des Monty Python - il a l'air d'être là, mais il est mort.

### 2.2 Cas classiques de références pendantes

#### Cas 1 : Référence vers mémoire supprimée
```cpp
int& r = *new int(42);
delete &r;
// r est maintenant une référence pendante
```

#### Cas 2 : Retourner une référence vers une variable locale
```cpp
int& foo() {
    int x = 42;
    return x;  // ERREUR ! x sera détruit à la fin de la fonction
}
```

### 2.3 Portée (Scope) vs Durée de vie (Lifetime)

- **Portée** : Les endroits dans le code où un nom peut être utilisé
- **Durée de vie** : La période pendant laquelle un objet est valide

```cpp
{
    int a = b;  // ERREUR : 'b' est dans la portée mais sa durée de vie n'a pas commencé
    int b = 42;
}
```

### 2.4 Les références const et la prolongation de durée de vie

**Règle importante** : Les références `const` prolongent la durée de vie des objets temporaires.

```cpp
const int& r = 42;  // OK : la durée de vie du temporaire est prolongée
```

**Mais attention** : Un objet temporaire vit jusqu'à la fin de l'expression complète.

```cpp
struct S {
    int i;
    const int& r;
};

S* p = new S{1, 2};  // DANGER ! Le '2' temporaire meurt à la fin de cette ligne
```

## 3. Décroissance (Decay) et lvalues/rvalues

### 3.1 Qu'est-ce que la décroissance ?

La décroissance est le processus par lequel un type complexe se comporte comme un type plus simple :
- Un tableau décroît en pointeur vers son premier élément
- Une référence décroît en valeur

### 3.2 lvalue vs rvalue

- **lvalue** : Expression qui a une adresse en mémoire (peut apparaître à gauche d'une affectation)
- **rvalue** : Expression qui n'a pas d'adresse permanente (ne peut apparaître qu'à droite)

En C++ moderne :
- **lvalue** = "location value" (a une position en mémoire)
- **rvalue** = valeur temporaire

## 4. cdecl et les alias de types

### 4.1 Lecture des déclarations complexes

Pour lire une déclaration C/C++ complexe, suivez la règle "droite-gauche" :

```cpp
int *a[10];     // a est un tableau de 10 pointeurs vers int
int (*b)[10];   // b est un pointeur vers un tableau de 10 int
```

### 4.2 typedef vs using

L'ancienne méthode (C) :
```cpp
typedef int (*PtrToFunc)(int&);
```

La nouvelle méthode (C++) :
```cpp
using PtrToFunc = int(*)(int&);
```

Avantage de `using` : peut être templaté
```cpp
template<typename T>
using PtrToFunc = T(*)(T&);
```

## 5. Name Mangling

### 5.1 Qu'est-ce que le name mangling ?

Le name mangling est le processus par lequel le compilateur C++ encode les informations de type dans les noms de fonctions pour permettre la surcharge.

```cpp
void foo(int);    // Devient quelque chose comme _Z3fooi
void foo(double); // Devient quelque chose comme _Z3food
```

### 5.2 Pourquoi le C n'a pas de name mangling ?

Le C garantit que les noms ne sont pas modifiés, ce qui permet :
- L'interopérabilité entre langages
- Une ABI stable
- La création d'interfaces C pour d'autres langages

### 5.3 extern "C"

Pour désactiver le name mangling en C++ :
```cpp
extern "C" {
    void foo(int);  // Pas de name mangling
}
```

**Limitations** :
- Pas de surcharge possible
- Pas de templates
- Pas de membres de classe

## 6. Règles de surcharge

### 6.1 Processus de résolution de surcharge

1. **Collecte des candidats** : Toutes les fonctions avec le bon nom
2. **Filtrage** : Élimination des candidats non viables
3. **Classement** : Les candidats restants sont classés

### 6.2 Hiérarchie de correspondance

Du meilleur au pire :
1. **Correspondance exacte**
2. **Correspondance avec template**
3. **Conversions standard** (int → double)
4. **Conversions utilisateur**
5. **Arguments variables** (...)
6. **Références mal liées**

### 6.3 Exemple pratique

```cpp
void f(int);              // #1
void f(const int&);       // #2
void f(double);           // #3
void f(...);              // #4

f(10);  // Appelle #1 (correspondance exacte)
```

## 7. Espaces de noms (Namespaces)

### 7.1 Pourquoi les espaces de noms ?

Les espaces de noms évitent la pollution de l'espace de noms global et permettent d'organiser le code.

```cpp
namespace Container {
    struct List { /* ... */ };
    struct Node { /* ... */ };
}
```

### 7.2 Utilisation de using

Trois formes de `using` :

1. **Alias de type** : `using Vec = std::vector<int>;`
2. **Import de nom** : `using std::vector;`
3. **Import d'espace de noms** : `using namespace std;` (À ÉVITER !)

### 7.3 Espaces de noms anonymes

```cpp
namespace {
    void helper() { /* ... */ }  // Visible uniquement dans ce fichier
}
```

Équivalent moderne de `static` pour les fonctions au niveau fichier.

## 8. Bonnes pratiques

### 8.1 Gestion mémoire
- Toujours apparier `new`/`delete` et `new[]`/`delete[]`
- Utiliser l'encapsulation pour garantir la cohérence
- Préférer les conteneurs standard et les smart pointers

### 8.2 Références
- Ne jamais retourner de référence vers un objet local
- Utiliser des références const pour les paramètres
- Méfiez-vous des membres références dans les classes

### 8.3 Espaces de noms
- Ne jamais utiliser `using namespace` dans les headers
- Utiliser des espaces de noms anonymes au lieu de `static`
- Ne pas polluer l'espace de noms global

### 8.4 Surcharge
- Éviter les conversions implicites ambiguës
- Être cohérent dans les signatures de fonctions
- Documenter clairement les surcharges disponibles

## Conclusion

Cette leçon a couvert les aspects fondamentaux de la gestion mémoire et de la résolution de noms en C++. Ces concepts sont essentiels pour écrire du code C++ robuste et maintenable. La compréhension de ces mécanismes vous permettra d'éviter de nombreux pièges courants et d'utiliser efficacement les fonctionnalités du langage.
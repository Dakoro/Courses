# Leçon : Héritage Multiple, RTTI et Introduction aux Compilateurs

## Introduction

Cette leçon couvre des concepts avancés du C++ : l'héritage multiple, le système de types à l'exécution (RTTI), ainsi qu'une introduction aux compilateurs avec les automates et expressions régulières.

## 1. Le Problème Initial : Héritage et Ordre d'Initialisation

### Le Défi

Considérons cette hiérarchie de classes :

```cpp
class ImBuff { /* ... */ };

class Array {
    ImBuff* buffer;
public:
    Array(ImBuff* b) : buffer(b) {}
};

class MyBuff : public ImBuff {
    // Extension de ImBuff
};

class MyArray : public Array {
    MyBuff my_buffer;
    // Problème : comment initialiser Array avec &my_buffer ?
};
```

Le problème : les classes de base sont initialisées **avant** les membres. On ne peut pas utiliser `my_buffer` dans l'initialisation de `Array`.

### Solution : Élever le Membre au Rang de Classe de Base

La solution consiste à transformer le membre en classe de base via l'héritage multiple :

```cpp
class ProxyBuff {
protected:
    MyBuff buff;
    explicit ProxyBuff() {}  // Bloquer les conversions implicites
};

class MyArray : protected ProxyBuff, public Array {
public:
    MyArray() : ProxyBuff(), Array(&buff) {}
};
```

## 2. Héritage Multiple

### Syntaxe de Base

```cpp
class Derived : public Base1, public Base2 {
    // Hérite de deux classes
};
```

### Points Importants

1. **Ordre d'initialisation** : De gauche à droite dans la liste d'héritage
2. **Ajustement de pointeurs** : Les adresses des sous-objets peuvent différer
3. **Ambiguïtés** : Résoudre avec l'opérateur de portée `::`

## 3. Le Problème du Diamant et l'Héritage Virtuel

### Le Problème

```cpp
class File {
    int a;
};

class InputFile : public File {
    int b;
};

class OutputFile : public File {
    int c;
};

class IOFile : public InputFile, public OutputFile {
    // Problème : deux copies de File !
};
```

### Solution : Héritage Virtuel

```cpp
class InputFile : virtual public File {
    int b;
};

class OutputFile : virtual public File {
    int c;
};

class IOFile : public InputFile, public OutputFile {
    // Une seule copie de File
};
```

### Règles de l'Héritage Virtuel

1. **Initialisation** : La classe la plus dérivée initialise la base virtuelle
2. **Performance** : Coût supplémentaire (indirections additionnelles)
3. **Ordre** : D'abord les bases virtuelles, puis les non-virtuelles

```cpp
IOFile::IOFile(int x) 
    : File(x),           // Base virtuelle initialisée ici
      InputFile(),       // Ces constructeurs n'initialiseront pas File
      OutputFile() {
    // ...
}
```

## 4. Casting avec Héritage Multiple

### static_cast avec Héritage Multiple

```cpp
Derived* d = new Derived();
Base1* b1 = d;              // Conversion implicite
Base2* b2 = d;              // Conversion implicite

// Retour vers Derived nécessite static_cast
Derived* d1 = static_cast<Derived*>(b1);
Derived* d2 = static_cast<Derived*>(b2);
```

**Important** : `static_cast` ajuste automatiquement les pointeurs lors de l'héritage multiple.

### Limitations avec l'Héritage Virtuel

`static_cast` ne peut pas convertir depuis une base virtuelle :

```cpp
File* f = new IOFile();
IOFile* io = static_cast<IOFile*>(f);  // ERREUR !
```

## 5. RTTI et dynamic_cast

### RTTI (Run-Time Type Information)

RTTI permet d'obtenir des informations sur les types à l'exécution :

```cpp
OutputFile* pf = new IOFile();

// typeid retourne le type dynamique pour les objets polymorphes
const std::type_info& ti = typeid(*pf);  // IOFile
const std::type_info& tp = typeid(pf);   // OutputFile* (type statique)
```

### dynamic_cast

`dynamic_cast` permet des conversions sûres à l'exécution :

```cpp
File* f = new IOFile();
InputFile* in = dynamic_cast<InputFile*>(f);   // OK
OutputFile* out = dynamic_cast<OutputFile*>(f); // OK

// Conversion croisée possible !
InputFile* in2 = new IOFile();
OutputFile* out2 = dynamic_cast<OutputFile*>(in2); // OK
```

### Comportement en cas d'échec

- **Avec pointeurs** : Retourne `nullptr`
- **Avec références** : Lance `std::bad_cast`

```cpp
Base* b = new Base();
Derived* d = dynamic_cast<Derived*>(b);  // nullptr

Base& br = *b;
Derived& dr = dynamic_cast<Derived&>(br); // Lance std::bad_cast
```

### Performance

- Complexité : O(n) où n est la hauteur de la hiérarchie
- Utiliser avec parcimonie
- Préférer `static_cast` quand possible

## 6. Introduction aux Compilateurs

### Expressions Régulières et Automates

#### Définitions de Base

- **Alphabet** : Ensemble fini de symboles (ex: {a, b, c})
- **Chaîne** : Séquence de symboles
- **Langage** : Ensemble de chaînes

#### Expressions Régulières

Opérations de base :
- **Concaténation** : `ab` (a suivi de b)
- **Union** : `a|b` (a ou b)
- **Fermeture de Kleene** : `a*` (0 ou plusieurs a)
- **Extensions** : `a+` (1 ou plusieurs), `a?` (0 ou 1)

Exemple : `a*b*` représente toutes les chaînes avec des a suivis de b.

#### Automates Finis

**Automate déterministe (DFA)** :
- Un seul état suivant pour chaque symbole
- Plus efficace à exécuter

**Automate non-déterministe (NFA)** :
- Plusieurs transitions possibles
- Plus facile à construire depuis une regex

### Analyse Lexicale avec Flex

#### Exemple de Fichier Flex

```lex
%{
#include <iostream>
using namespace std;
%}

%%
[ \t\n]+        { /* Ignorer les espaces */ }
[0-9]+          { cout << "NOMBRE: " << yytext << endl; }
"+"             { cout << "PLUS" << endl; }
"="             { cout << "EGAL" << endl; }
.               { cout << "ERREUR: " << yytext << endl; }
%%

int main() {
    yylex();
    return 0;
}
```

#### Processus de Compilation

1. **Flex** génère un analyseur lexical en C++
2. L'analyseur transforme le texte en flux de lexèmes
3. Ces lexèmes sont ensuite utilisés par le parseur

### Lemme de Pompage

Le lemme de pompage permet de prouver qu'un langage n'est pas régulier :

> Si L est régulier, alors pour tout mot suffisamment long w ∈ L, il existe une décomposition w = xyz telle que xy^n z ∈ L pour tout n ≥ 0.

**Application** : Le langage `{a^n b^n | n ≥ 0}` n'est pas régulier car on ne peut pas "pomper" sans déséquilibrer le nombre de a et de b.

## Points Clés à Retenir

### Héritage Multiple
1. **Utiliser avec parcimonie** - Complexifie la hiérarchie
2. **Préférer les interfaces** - Héritage multiple d'interfaces pures
3. **Attention aux ambiguïtés** - Utiliser l'opérateur de portée

### Héritage Virtuel
1. **Résout le problème du diamant** - Une seule copie de la base
2. **Coût en performance** - Indirections supplémentaires
3. **Initialisation spéciale** - Par la classe la plus dérivée

### RTTI et dynamic_cast
1. **Coût élevé** - Utiliser uniquement si nécessaire
2. **Alternative** : Conception sans RTTI (visitor pattern, etc.)
3. **Sécurité** vs **Performance** - Faire un choix éclairé

### Compilateurs
1. **Analyse lexicale** - Transforme le texte en lexèmes
2. **Expressions régulières** - Pour décrire les lexèmes
3. **Automates** - Pour implémenter efficacement l'analyse

## Exercice Pratique

Implémenter une hiérarchie avec héritage virtuel pour modéliser des flux d'entrée/sortie, en gérant correctement l'initialisation et en utilisant `dynamic_cast` pour les conversions sécurisées entre types.
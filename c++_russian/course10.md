# Leçon : Compilation - Analyse Lexicale et Syntaxique

## Introduction

Cette leçon couvre les fondamentaux de la compilation : l'analyse lexicale avec Flex, l'analyse syntaxique avec Bison, et la construction d'un compilateur pour le langage ParaCL.

## 1. Rappel sur les Expressions Régulières et Automates

### Limites des Expressions Régulières

Les expressions régulières peuvent décrire des langages comme :
- **L1** : `a*b*` (n fois 'a' puis m fois 'b')
- **L2** : `a|cb|abc` (langage fini)

Mais **ne peuvent pas** décrire :
- **L3** : `a^n b^n` (n fois 'a' puis n fois 'b')

### Lemme de Pompage

> Si L est un langage régulier, alors pour tout mot suffisamment long w ∈ L, il existe une décomposition w = xyz telle que xy^i z ∈ L pour tout i ≥ 0.

Ce lemme explique pourquoi `a^n b^n` n'est pas régulier : on ne peut pas "pomper" sans déséquilibrer le nombre de 'a' et de 'b'.

## 2. Grammaires Contextuelles Libres

### Définitions de Base

- **Symboles terminaux** : Les symboles réels du langage (a, b, +, -, etc.)
- **Symboles non-terminaux** : Variables de la grammaire (A, B, E, etc.)
- **Productions** : Règles de réécriture (A → aB | ε)

### Exemple de Grammaire

Pour les expressions régulières :
```
A → A + A  | A · A  | A*  | (A)  | a  | b  | c
```

### Grammaire Contextuelle Libre

Une grammaire est **contextuelle libre** si toutes ses productions ont la forme :
```
A → α
```
où A est un seul non-terminal et α est une séquence de terminaux et non-terminaux.

## 3. Analyse Lexicale avec Flex

### Structure d'un Fichier Flex

```lex
%{
#include <iostream>
using namespace std;
%}

%%
[ \t\n]+        { /* Ignorer espaces */ }
[1-9][0-9]*     { cout << "NOMBRE: " << yytext << endl; }
"+"             { cout << "PLUS" << endl; }
"-"             { cout << "MOINS" << endl; }
"="             { cout << "EGAL" << endl; }
";"             { cout << "POINT_VIRGULE" << endl; }
.               { cout << "ERREUR: " << yytext << endl; }
%%
```

### Configuration CMake pour Flex

```cmake
find_package(FLEX REQUIRED)

flex_target(MyLexer lexer.l ${CMAKE_CURRENT_BINARY_DIR}/lexer.cc)

add_executable(my_program main.cc ${FLEX_MyLexer_OUTPUTS})
target_compile_features(my_program PRIVATE cxx_std_17)
```

## 4. Types d'Analyseurs Syntaxiques

### Analyse Descendante (Top-Down) - LL(k)

**LL(k)** signifie :
- **L** : Lecture de gauche à droite (Left-to-right)
- **L** : Dérivation la plus à gauche (Leftmost)
- **(k)** : k symboles de prévision

#### Descente Récursive

Pour chaque non-terminal X, créer une fonction :
```cpp
void parseX() {
    if (current_token == EXPECTED) {
        advance();
        parseY();  // Si X → aY
    } else {
        error();
    }
}
```

#### Conditions LL(1)

Une grammaire est LL(1) si pour toutes productions A → α | β :
1. FIRST(α) ∩ FIRST(β) = ∅
2. Si β →* ε, alors FIRST(α) ∩ FOLLOW(A) = ∅

### Analyse Ascendante (Bottom-Up) - LR

**LR** signifie :
- **L** : Lecture de gauche à droite
- **R** : Dérivation la plus à droite en inverse

Deux opérations principales :
- **Shift** : Déplacer un symbole de l'entrée vers la pile
- **Reduce** : Appliquer une production en sens inverse

## 5. Analyse Syntaxique avec Bison

### Structure d'un Fichier Bison

```yacc
%{
#include <iostream>
using namespace std;
%}

%token NUMBER
%token PLUS MINUS EQUALS SEMICOLON

%%
equation_list:
    equation SEMICOLON equation_list
  | /* vide */
  ;

equation:
    expr EQUALS expr {
        cout << "Vérification: " << $1 << " == " << $3 
             << " Résultat: " << ($1 == $3) << endl;
    }
  ;

expr:
    expr PLUS expr   { $$ = $1 + $3; }
  | expr MINUS expr  { $$ = $1 - $3; }
  | NUMBER          { $$ = $1; }
  ;
%%
```

### Gestion des Conflits

#### Problème : Conflits Shift/Reduce

Grammaire ambiguë :
```yacc
expr: expr PLUS expr | expr MINUS expr | NUMBER ;
```

Génère des conflits car `2 + 3 - 4` peut être parsé comme :
- `(2 + 3) - 4` ou
- `2 + (3 - 4)`

#### Solution : Grammaire Non-Ambiguë

```yacc
expr: 
    NUMBER rest
  ;

rest:
    PLUS NUMBER rest  { $$ = $2 + $3; }
  | MINUS NUMBER rest { $$ = -$2 + $3; }
  | /* vide */       { $$ = 0; }
  ;
```

## 6. Intégration Flex/Bison

### Driver Pattern

```cpp
// driver.hpp
class Driver {
    Lexer* lexer;
    Parser* parser;
    
public:
    Driver();
    int parse(const std::string& filename);
    void insert(const EquationList& eqs);
    void print_results();
};
```

### Communication Lexer → Parser

Le lexer retourne des tokens avec leurs valeurs sémantiques :

```cpp
// Dans le lexer
[0-9]+ {
    yylval = std::stoi(yytext);
    return NUMBER;
}
```

## 7. Architecture d'un Compilateur Réaliste

### Structure Modulaire pour ParaCL

```
┌─────────────┐     ┌──────────────┐
│   lexer.l   │────►│  scanner.cc  │
└─────────────┘     └──────────────┘
                            │
┌─────────────┐     ┌──────────────┐
│  parser.y   │────►│  parser.cc   │
└─────────────┘     └──────────────┘
                            │
                    ┌──────────────┐
                    │   driver.cc  │
                    └──────────────┘
                            │
┌─────────────────────────────────┐
│        AST (Abstract Syntax Tree)│
├─────────────────────────────────┤
│ class ANode (interface pure)    │
│ class BinOp : public ANode      │
│ class If : public ANode         │
│ class While : public ANode      │
│ class Scope : public ANode      │
└─────────────────────────────────┘
```

### Exemple de Hiérarchie AST

```cpp
// Interface abstraite
class ANode {
public:
    virtual ~ANode() = default;
    virtual void execute() = 0;
    virtual void dump() const = 0;
};

// Nœud concret
class BinOp : public ANode {
    Op operation;
    ANode* left;
    ANode* right;
    
public:
    BinOp(Op op, ANode* l, ANode* r);
    void execute() override;
    void dump() const override;
};
```

## 8. Gestion des Erreurs

### Mode Panique

En cas d'erreur syntaxique :
1. Signaler l'erreur avec position (ligne, colonne)
2. Entrer en "mode panique"
3. Chercher un délimiteur sûr (`;`, `}`)
4. Reprendre l'analyse après le délimiteur

```yacc
error_recovery:
    error SEMICOLON { 
        yyerrok; 
        cout << "Erreur récupérée à la ligne " << @1.first_line << endl;
    }
  ;
```

## 9. Conseils pour le Projet ParaCL

### Étapes Recommandées

1. **Écrire la grammaire complète**
   - Toutes les productions
   - Vérifier l'absence de conflits

2. **Concevoir la hiérarchie de classes**
   - Interface abstraite pour l'AST
   - Séparation parser/logique métier

3. **Tests progressifs**
   - D'abord expressions arithmétiques
   - Puis structures de contrôle
   - Enfin fonctionnalités avancées

### Table des Symboles

```cpp
class SymbolTable {
    std::unordered_map<std::string, int> symbols;
    SymbolTable* parent;  // Pour les scopes imbriqués
    
public:
    void declare(const std::string& name);
    int& lookup(const std::string& name);
    void enter_scope();
    void exit_scope();
};
```

## Points Clés à Retenir

1. **Expressions régulières** : Pour l'analyse lexicale uniquement
2. **Grammaires CF** : Pour la structure syntaxique
3. **LL vs LR** : LL plus simple à implémenter manuellement, LR plus puissant
4. **Conflits** : Toujours résoudre les ambiguïtés dans la grammaire
5. **Architecture** : Séparer clairement lexer, parser et AST
6. **Erreurs** : Récupération gracieuse avec mode panique

## Ressources

- **Livres** :
  - "Compilers: Principles, Techniques, and Tools" (Dragon Book)
  - "Modern Compiler Implementation" - Appel

- **Cours en ligne** :
  - Stanford CS143 Compilers
  - Coursera Compilers par Alex Aiken

- **Documentation** :
  - [Flex Manual](https://github.com/westes/flex)
  - [Bison Manual](https://www.gnu.org/software/bison/manual/)
# Architecture des Ordinateurs : Modèle de Von Neumann et Architecture des Jeux d'Instructions

## Introduction

Bienvenue dans ce cours sur l'architecture des ordinateurs. Aujourd'hui, nous allons changer de perspective et passer du niveau de conception numérique à l'architecture et la micro-architecture. Mais avant cela, permettez-moi de couvrir quelques points importants sur le timing et la vérification.

## 1. Timing et Vérification dans les Circuits Numériques

### 1.1 Timing dans les Composants Séquentiels

Le timing dans un composant séquentiel unique est déterminé par le **délai logique maximum** à travers les différents chemins combinatoires possibles dans la logique combinatoire qui connecte un registre à un autre.

**Formule du temps de cycle d'horloge :**
```
T_cycle = t_prop(registre) + t_delai_max(logique_comb) + t_setup(registre)
```

### 1.2 Dépendance aux Valeurs d'Entrée

Le délai maximum peut dépendre des valeurs d'entrée. Par exemple, considérons un multiplicateur :
- Multiplication par 0 : chemin rapide (détection spéciale)
- Multiplication par 1 : chemin rapide (bypass)
- Multiplication par 2^n : décalage simple
- Multiplication générale : chemin complet complexe

### 1.3 Composants Séquentiels Multiples

Quand vous avez plusieurs composants séquentiels dans un système :
- Le temps de cycle global = MAX(tous les délais logiques locaux)
- Cela nécessite d'analyser des milliers de chemins dans un processeur moderne

### 1.4 Principes de Conception pour le Timing

**1. Conception du Chemin Critique (Critical Path Design)**
- Minimiser le délai logique maximum
- Se concentrer sur les chemins qui dominent le timing
- Diviser les opérations longues en plusieurs cycles si nécessaire

**2. Conception Équilibrée (Balanced Design)**
- Équilibrer les délais logiques entre différents composants
- Éviter les goulots d'étranglement
- Minimiser le temps perdu

**3. Conception "Pain et Beurre" (Bread and Butter Design)**
- Optimiser pour le cas commun
- S'assurer que les cas non-communs ne dominent pas le design
- Exemple : optimiser l'additionneur plus que l'unité racine carrée

## 2. Vérification et Test

### 2.1 Types de Vérification

**Vérification Fonctionnelle :**
- Vérifier que le circuit fonctionne logiquement correctement
- Méthodes : simulation, vérification formelle

**Vérification Temporelle :**
- Vérifier que le circuit fonctionne correctement avec le timing des composants
- Plus complexe car ajoute une dimension supplémentaire

### 2.2 Complexité du Test

Pour un additionneur 32 bits :
- Entrées possibles : 2^32 × 2^32 = 2^64 combinaisons
- Test exhaustif impossible → nécessité de tests aléatoires ou ciblés

### 2.3 Étapes de Vérification

1. **Niveau Composant** : Test des unités individuelles
2. **Niveau Module** : Test des ALU, unités fonctionnelles
3. **Niveau Cœur** : Test du processeur complet
4. **Post-Silicon** : Test après fabrication

**Fait important :** Plus de 70% du temps et du coût de conception d'une puce est consacré à la vérification et au test.

## 3. Le Modèle de Von Neumann

### 3.1 Introduction

John von Neumann a proposé en 1946 un modèle fondamental pour les ordinateurs qui reste dominant aujourd'hui. Ce modèle a deux caractéristiques principales :

1. **Traitement Séquentiel des Instructions**
2. **Programme Stocké en Mémoire**

### 3.2 Les Cinq Composants

```
┌─────────────┐     ┌──────────────┐     ┌─────────────┐
│   Mémoire   │◄────┤ Unité de     ├────►│   Unité de  │
│ (Programme  │     │ Contrôle     │     │ Traitement  │
│  + Données) │     └──────────────┘     │    (ALU)    │
└─────────────┘                          └─────────────┘
       ▲                                         ▲
       │                                         │
       ▼                                         ▼
┌─────────────┐                          ┌─────────────┐
│   Entrées   │                          │   Sorties   │
└─────────────┘                          └─────────────┘
```

### 3.3 La Mémoire

**Caractéristiques :**
- Stocke programmes ET données (pas de distinction)
- Organisée en octets (8 bits) et mots (taille dépendante de l'architecture)
- **Espace d'adressage** : nombre total d'emplacements uniques
- **Adressabilité** : nombre de bits stockés par emplacement

**Exemple LC3 :**
- Espace d'adressage : 2^16 emplacements
- Adressabilité : mot (16 bits par emplacement)
- Mémoire totale : 2^16 × 16 bits

**Exemple MIPS :**
- Espace d'adressage : 2^32 emplacements
- Adressabilité : octet (8 bits par emplacement)
- Mémoire totale : 2^32 × 8 bits

### 3.4 Convention Big-Endian vs Little-Endian

Pour un mot de 32 bits stocké à l'adresse 0 :

**Big-Endian :**
```
Adresse: |  0  |  1  |  2  |  3  |
Données: | MSB |     |     | LSB |
```

**Little-Endian :**
```
Adresse: |  0  |  1  |  2  |  3  |
Données: | LSB |     |     | MSB |
```

### 3.5 Accès Mémoire

**Registres pour l'accès mémoire :**
- **MAR (Memory Address Register)** : contient l'adresse à accéder
- **MDR (Memory Data Register)** : contient les données lues/écrites

**Algorithme de Lecture :**
```
1. MAR ← adresse_à_lire
2. Activer signal de lecture
3. Attendre délai_mémoire
4. données ← MDR
```

**Algorithme d'Écriture :**
```
1. MAR ← adresse_à_écrire
2. MDR ← données_à_écrire
3. Activer signal d'écriture
4. Attendre délai_mémoire
```

## 4. L'Unité de Traitement

### 4.1 Unité Arithmétique et Logique (ALU)

L'ALU effectue les calculs réels. Elle peut supporter diverses opérations :

**LC3 :** ADD, AND, NOT
**MIPS :** ADD, SUB, AND, OR, XOR, SLT, SLL, SRL, etc.

### 4.2 Registres

**Motivation :** La mémoire est trop lente (150-200ns) comparée aux opérations ALU (<1ns).

**Caractéristiques des registres :**
- Stockage rapide et temporaire
- Proche de l'ALU
- Visible au programmeur
- Nombre limité (8 dans LC3, 32 dans MIPS)

**Exemple d'utilisation :**
Pour calculer `(A + B) × C ÷ D` :
```
1. R0 ← A + B      // Résultat intermédiaire dans R0
2. R1 ← R0 × C     // Deuxième résultat dans R1
3. R2 ← R1 ÷ D     // Résultat final dans R2
```

## 5. L'Unité de Contrôle

### 5.1 Composants

- **PC (Program Counter)** : adresse de l'instruction courante/suivante
- **IR (Instruction Register)** : contient l'instruction en cours d'exécution

### 5.2 Traitement Séquentiel

**Algorithme de base :**
```
Tant que (vrai) :
    1. IR ← Mémoire[PC]        // Fetch
    2. PC ← PC + taille_instruction
    3. Décoder IR
    4. Exécuter instruction selon décodage
```

## 6. Architecture des Jeux d'Instructions (ISA)

### 6.1 Format d'Instruction

Une instruction comprend :
- **Opcode** : spécifie l'opération à effectuer
- **Opérandes** : spécifient les données à manipuler

### 6.2 Types d'Instructions

**1. Instructions Opératoires**
Exécutent des calculs dans l'ALU.

Exemple LC3 - ADD :
```
Format : | Opcode(4) | DR(3) | SR1(3) | 0 | 00 | SR2(3) |
Sémantique : DR ← SR1 + SR2
```

**2. Instructions de Mouvement de Données**
Transfèrent des données entre mémoire et registres.

Exemple LC3 - LDR (Load Register) :
```
Format : | Opcode(4) | DR(3) | BaseR(3) | Offset(6) |
Sémantique : DR ← Mémoire[BaseR + Offset]
```

**3. Instructions de Contrôle de Flux**
Modifient la séquence d'exécution (branches, sauts).

### 6.3 Modes d'Adressage

**Mode Base + Offset :**
```
Adresse_effective = Registre_base + Offset_immédiat
```

Utile pour :
- Accès aux tableaux
- Accès aux structures
- Variables locales sur la pile

### 6.4 Exemple Complet : Programme MIPS

```assembly
lw   $t2, 32($0)    # $t2 ← Mémoire[0 + 32]
add  $s0, $s1, $s2  # $s0 ← $s1 + $s2
sub  $t0, $t3, $t5  # $t0 ← $t3 - $t5
sw   $s0, 48($0)    # Mémoire[0 + 48] ← $s0
```

## 7. Le Cycle d'Exécution des Instructions

### 7.1 État Architectural

L'état visible au programmeur comprend :
- **PC** : Compteur de Programme
- **Registres** : Fichier de registres
- **Mémoire** : Tableau de stockage

### 7.2 Machine à États Finis

Chaque instruction fait passer la machine d'un état à un autre :

```
État_n → [Exécution Instruction] → État_n+1
```

Les signaux de contrôle orchestrent cette transition en activant les composants appropriés au bon moment.

## Conclusion

Le modèle de Von Neumann reste fondamental dans l'architecture moderne des ordinateurs. Ses principes clés - traitement séquentiel et programme stocké - permettent une programmation simple et efficace. La compréhension de ce modèle, ainsi que des concepts de timing et de vérification, est essentielle pour concevoir et comprendre les systèmes informatiques modernes.

Dans le prochain cours, nous explorerons en détail le cycle de traitement des instructions et la micro-architecture qui implémente ce modèle.
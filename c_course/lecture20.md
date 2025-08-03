# Des Principes Fondamentaux à la Pratique : Un Cours sur l'Assembleur et l'Architecture des Ordinateurs

## Introduction : La Thèse de l'Universalité de l'Assembleur

Ce cours repose sur une thèse centrale : la maîtrise des principes fondamentaux d'un langage d'assemblage, pour une architecture informatique raisonnable, confère la capacité de comprendre tous les langages d'assemblage. Qu'il s'agisse de l'architecture x86 qui équipe la plupart de nos ordinateurs, de l'ARM omniprésent dans nos appareils mobiles, ou des architectures RISC-V et MIPS étudiées dans le monde académique, les concepts sous-jacents demeurent remarquablement similaires.

L'objectif de cette session n'est donc pas de mémoriser une litanie d'instructions spécifiques, mais de construire une compréhension profonde des principes qui régissent l'interaction entre le logiciel et le matériel.

Pour atteindre cet objectif, notre approche pédagogique se déroulera en deux temps. Premièrement, nous nous livrerons à un exercice de conception : nous allons inventer de toutes pièces une micro-architecture simple, que nous nommerons notre "microcalculateur pédagogique". Ce processus nous forcera à affronter et à résoudre les problèmes fondamentaux de la représentation des instructions et de la manipulation des données. Deuxièmement, forts de cette compréhension fondamentale, nous appliquerons nos connaissances à l'étude d'une architecture réelle et complexe : l'architecture x86-64, qui constitue la base de la majorité des ordinateurs de bureau et portables actuels.

En adoptant cette démarche, nous ne faisons pas que simuler un exercice académique ; nous endossons le rôle d'un architecte de jeu d'instructions (Instruction Set Architect, ISA). L'ISA est le contrat, le modèle abstrait qui définit l'interface entre le matériel et le logiciel. C'est la partie de la machine visible par le programmeur assembleur et le compilateur. En concevant notre propre ISA, nous ne faisons pas que jouer ; nous nous engageons dans une simulation authentique d'une des tâches les plus fondamentales de l'ingénierie informatique. L'ambition finale est de démystifier la machine, de permettre de "voir à travers les murs" et de lire le code machine, une compétence inestimable pour le débogage de bas niveau, l'optimisation des performances et la compréhension de la sécurité informatique.

## Partie 1 : L'Invention d'une Architecture – Le Microcalculateur Pédagogique

Pour établir les principes fondamentaux, nous commençons par concevoir une machine simple mais complète. Cette approche ascendante nous permet de justifier chaque décision de conception.

### 1.1 Le Modèle Matériel (Le "Hardware")

Notre machine virtuelle, le microcalculateur, est un système programmable simple. Il se compose de trois éléments externes principaux :

- Un clavier, pour fournir des données d'entrée
- Un écran, pour afficher les résultats
- Une mémoire de programme, où est stockée la séquence d'instructions à exécuter

Au cœur de ce microcalculateur se trouve son unité centrale de traitement (CPU), qui contient les éléments de calcul et de stockage les plus rapides : les registres. Pour notre modèle, nous définissons quatre registres d'usage général, nommés A, B, C et D. Chaque registre est une petite unité de mémoire de 8 bits. Cette taille de 8 bits est une caractéristique fondamentale de notre architecture ; elle signifie que chaque registre peut stocker un nombre entier non signé allant de 0 à $2^8-1=255$, ou un nombre signé de -128 à 127. Les registres servent de "brouillon" pour le processeur, contenant les opérandes des calculs en cours et les résultats intermédiaires.

### 1.2 Le Langage Machine (Les "Mnémoniques")

Pour commander notre processeur, nous définissons un ensemble d'opérations. Chaque opération est représentée par un nom symbolique facile à retenir pour un humain, appelé mnémonique. L'ensemble de ces mnémoniques et de leurs syntaxes constitue le jeu d'instructions (ISA) de notre machine.

Le jeu d'instructions de notre microcalculateur est le suivant :

**Transfert de données :**
- `mov D, <imm>` : Charge une valeur immédiate (une constante codée directement dans l'instruction) de 8 bits dans le registre D uniquement
- `in R` : Lit une valeur depuis le clavier et la stocke dans le registre spécifié R
- `out R` : Affiche le contenu du registre R sur l'écran

**Opérations arithmétiques :** Ces instructions sont dites "à deux adresses", où le premier opérande est à la fois source et destination.
- `add RX, RS` : Additionne le contenu du registre source RS au registre destination RX et stocke le résultat dans RX. L'opération effectuée est $RX \leftarrow RX + RS$
- `sub RX, RS` : Soustrait le contenu de RS de RX. $(RX \leftarrow RX - RS)$
- `mul RX, RS` : Multiplie RX par RS. $(RX \leftarrow RX \times RS)$
- `div RX, RS` : Divise RX par RS. $(RX \leftarrow RX / RS)$

Le tableau suivant formalise l'ISA que nous venons de créer. Il représente le "contrat" entre le programmeur et la machine et servira de document de référence pour la suite de notre travail.

**Tableau 1 : Jeu d'Instructions (ISA) du Microcalculateur Pédagogique**

| Mnémonique | Opérandes | Sémantique (Pseudo-code) | Description |
|------------|-----------|--------------------------|-------------|
| mov | D, imm | D ← imm | Charge une valeur immédiate 8 bits dans D |
| add | R_dest, R_src | R_dest ← R_dest + R_src | Additionne R_src à R_dest |
| sub | R_dest, R_src | R_dest ← R_dest - R_src | Soustrait R_src de R_dest |
| mul | R_dest, R_src | R_dest ← R_dest * R_src | Multiplie R_dest par R_src |
| div | R_dest, R_src | R_dest ← R_dest / R_src | Divise R_dest par R_src |
| in | R_dest | R_dest ← read_keyboard() | Lit une valeur depuis le clavier dans R_dest |
| out | R_src | write_screen(R_src) | Affiche la valeur de R_src à l'écran |

## Partie 2 : L'Interface Matériel-Logiciel – Encodage des Instructions

Maintenant que nous avons défini ce que notre machine peut faire, nous devons résoudre un problème fondamental : comment communiquer ces instructions à la machine ? Le processeur ne comprend ni les chaînes de caractères "add" ni les symboles "A" ou "B". Il ne manipule que des signaux binaires, des zéros et des uns. La tâche de l'architecte est de concevoir un schéma d'encodage qui traduit chaque instruction valide en une unique séquence de 8 bits.

### 2.1 Le Problème Fondamental de l'Encodage

Chaque instruction de 8 bits doit contenir deux types d'informations :

- Le **code d'opération (opcode)** : une séquence de bits qui identifie de manière unique l'opération à effectuer (add, mov, etc.)
- Les **opérandes** : les bits restants qui spécifient les données sur lesquelles l'opération doit agir (quels registres, quelle valeur immédiate)

La manière dont nous allouons les 8 bits disponibles entre l'opcode et les opérandes est une décision de conception cruciale qui aura des conséquences profondes sur les capacités de notre machine.

### 2.2 Conception d'un Schéma d'Encodage : Une Étude de Cas

Explorons ce processus de conception à travers une étude de cas.

#### Première tentative : L'approche à longueur fixe

Une approche simple et logique consiste à allouer un nombre fixe de bits à chaque partie de l'instruction. Nous avons 7 types d'instructions, ce qui nécessite au moins 3 bits pour l'opcode ($2^3=8$ combinaisons possibles). Nous avons 4 registres, ce qui nécessite 2 bits pour en identifier un ($2^2=4$). Pour une instruction comme `add A, B`, nous pourrions utiliser le format suivant :

- Bits 7-5 : Opcode (3 bits)
- Bits 4-3 : Registre destination (2 bits)
- Bits 2-1 : Registre source (2 bits)
- Bit 0 : Inutilisé (1 bit)

Cette approche semble cohérente, mais elle révèle rapidement un défaut fatal lorsqu'on considère l'instruction `mov D, <imm>`. Dans ce schéma, après avoir utilisé 3 bits pour l'opcode mov et 2 bits pour spécifier le registre D (qui est fixe), il ne resterait que 3 bits pour la valeur immédiate. Une version plus généreuse pourrait allouer tous les bits restants à la valeur immédiate, soit 5 bits (8 bits total - 3 bits opcode). Cependant, même avec 5 bits, la plus grande constante que nous pourrions charger serait $2^5-1=31$. Cela rend notre calculatrice incapable de traiter des formules aussi simples que $f(x)=x+100$, car la constante 100 ne peut pas être représentée.

Ce problème illustre un compromis fondamental dans la conception des ISA. Les architectures RISC (Reduced Instruction Set Computer), comme MIPS ou ARM, privilégient des instructions de longueur fixe. Cette uniformité simplifie grandement le processus de décodage par le matériel, ce qui permet d'accélérer l'exécution et de faciliter des techniques comme le pipelining. Cependant, cela peut conduire à un gaspillage d'espace pour les instructions simples et à des difficultés pour encoder de grandes constantes. À l'opposé, les architectures CISC (Complex Instruction Set Computer), comme l'emblématique x86, utilisent des instructions de longueur variable. Cette flexibilité permet une meilleure densité de code (les programmes prennent moins de place en mémoire) et facilite l'encodage d'instructions complexes avec différents types d'opérandes, mais au prix d'un circuit de décodage matériel beaucoup plus compliqué. Notre première tentative, de type "longueur fixe", a échoué en raison de sa rigidité.

#### La solution : L'approche à longueur variable

Pour résoudre le problème de l'instruction mov, nous devons adopter une approche plus flexible, inspirée des architectures CISC. Nous allons utiliser un encodage à longueur variable, où la signification des bits dépend de la valeur des premiers bits lus.

- Si le premier bit (bit 7) est 0, nous décrétons que l'instruction est un mov. Les 7 bits restants (iiiiiii) encodent la valeur immédiate, nous permettant de représenter des constantes de 0 à 127. Le registre de destination est implicitement D.
- Si le premier bit est 1, il s'agit d'une autre instruction (arithmétique ou d'entrée/sortie). Nous utilisons alors les bits suivants pour distinguer les différentes opérations.

Cette conception résout notre problème : nous pouvons maintenant charger la constante 100, tout en ayant suffisamment de bits pour encoder toutes nos autres instructions.

### 2.3 Spécification Formelle de l'Encodage Binaire

Formalisons maintenant notre schéma d'encodage complet. L'encodage des quatre registres est simple et direct : A ↔ 00, B ↔ 01, C ↔ 10, D ↔ 11.

Le tableau suivant présente la spécification binaire complète de notre ISA. C'est la traduction concrète de nos choix de conception, et elle sera indispensable pour implémenter nos outils logiciels.

**Tableau 2 : Schéma d'Encodage Binaire de l'ISA du Microcalculateur**

| Instruction | Encodage Binaire (8 bits) | Exemple | Encodage Exemple (Binaire) |
|-------------|---------------------------|---------|---------------------------|
| mov D, imm | 0 iiiiiii | mov D, 100 | 01100100 |
| add R1, R2 | 1 000 r1r1 r2r2 | add A, B | 10000001 |
| sub R1, R2 | 1 001 r1r1 r2r2 | sub C, A | 10011000 |
| mul R1, R2 | 1 010 r1r1 r2r2 | mul B, D | 10100111 |
| div R1, R2 | 1 011 r1r1 r2r2 | div A, C | 10110010 |
| in R | 1 100 00 rr | in A | 11000000 |
| out R | 1 101 00 rr | out B | 11010001 |
| Illégal | 111xxxx | -- | -- |

**Note :** Pour in et out, les bits r1r1 et r2r2 sont réaffectés. Par exemple, 1100 pour l'opcode in, puis les 2 bits suivants pour le registre, et les 2 derniers bits inutilisés (00). La disposition exacte est un détail de conception, mais le principe reste le même.

## Partie 3 : Mise en Œuvre Pratique – Outillage pour le Microcalculateur

Un jeu d'instructions n'est utile que si l'on dispose d'outils pour travailler avec. Le séminaire propose de développer une chaîne d'outils minimale pour notre microcalculateur, comprenant trois programmes essentiels : un décodeur, un encodeur et un simulateur. Nous allons les implémenter en langage C.

### 3.1 Structures de Données pour la Représentation des Instructions

Avant de coder, une étape de conception logicielle est cruciale. Une approche naïve du décodage, utilisant une cascade de conditions if et de masques de bits, est fonctionnelle mais limitée. Elle est difficile à maintenir et, surtout, le code produit n'est pas réutilisable. Le code du décodeur, qui ne fait qu'afficher du texte, ne peut pas être réutilisé par le simulateur, qui doit exécuter la logique de l'instruction.

La solution, démontrée lors de la session de codage en direct, est de créer une représentation intermédiaire (RI) en C. Cette RI est une structure de données qui capture la sémantique d'une instruction (son opcode et ses opérandes) indépendamment de son encodage binaire ou de sa représentation textuelle. Cette abstraction est la clé de la réutilisabilité. Le décodeur traduira le code binaire en cette RI. L'encodeur traduira le texte en cette RI. Le simulateur prendra cette RI et l'exécutera.

Voici un fichier d'en-tête C qui définit cette représentation intermédiaire :

```c
// Fichier d'en-tête commun: microcalc.h

#ifndef MICROCALC_H
#define MICROCALC_H

// Énumération pour les noms de registres, plus lisible que des entiers.
typedef enum {
    REG_A, REG_B, REG_C, REG_D
} Register;

// Énumération pour les opcodes.
typedef enum {
    OP_MOV, OP_ADD, OP_SUB, OP_MUL, OP_DIV, OP_IN, OP_OUT, OP_ILLEGAL
} OpCode;

// Structure pour les instructions à deux opérandes registres.
typedef struct {
    Register dest;
    Register src;
} RegOperands;

// Structure pour une instruction avec un opérande immédiat.
typedef struct {
    unsigned char immediate;
} ImmOperand;

// Structure pour une instruction avec un seul opérande registre.
typedef struct {
    Register reg;
} UnaryOperand;

// La structure principale représentant une instruction décodée.
// Elle contient l'opcode et une union pour les différents types d'opérandes.
typedef struct {
    OpCode opcode;
    union {
        RegOperands  reg_ops;
        ImmOperand   imm_op;
        UnaryOperand unary_op;
    } operands;
} Instruction;

#endif // MICROCALC_H
```

### 3.2 Le Décodeur (Binaire vers Mnémonique)

Le décodeur, ou désassembleur, est un programme qui prend en entrée un code machine binaire et le traduit en sa représentation textuelle lisible par l'homme (mnémonique).

**Pseudo-code :**

```
FONCTION decoder(octet_binaire) : Instruction
  instruction_resultat = nouvelle Instruction
  
  SI bit_7(octet_binaire) == 0 ALORS
    instruction_resultat.opcode = OP_MOV
    instruction_resultat.operands.imm_op.immediate = octet_binaire & 0x7F
  SINON
    opcode_3_bits = (octet_binaire >> 4) & 0x07 // Bits 6, 5, 4
    
    CAS opcode_3_bits:
      QUAND 0: instruction_resultat.opcode = OP_ADD
      QUAND 1: instruction_resultat.opcode = OP_SUB
      //... autres cas pour MUL, DIV
      QUAND 4: instruction_resultat.opcode = OP_IN
      QUAND 5: instruction_resultat.opcode = OP_OUT
      AUTREMENT: instruction_resultat.opcode = OP_ILLEGAL

    SI instruction est ADD, SUB, MUL, ou DIV ALORS
      instruction_resultat.operands.reg_ops.dest = (octet_binaire >> 2) & 0x03
      instruction_resultat.operands.reg_ops.src = octet_binaire & 0x03
    SINON SI instruction est IN ou OUT ALORS
      instruction_resultat.operands.unary_op.reg = octet_binaire & 0x03
    FIN SI
  FIN SI
  
  RETOURNER instruction_resultat
```

**Implémentation en C :**

L'implémentation C suit cette logique, en utilisant des opérations de masquage et de décalage de bits pour extraire les champs de l'octet d'instruction et remplir la structure Instruction. Une fonction print_instruction prend ensuite cette structure et affiche le format textuel correspondant.

```c
#include <stdio.h>
#include "microcalc.h"

// Décode un octet en une structure Instruction.
Instruction decode(unsigned char byte_code) {
    Instruction inst;

    if ((byte_code & 0x80) == 0) { // Le premier bit est 0 -> MOV
        inst.opcode = OP_MOV;
        inst.operands.imm_op.immediate = byte_code & 0x7F;
    } else { // Le premier bit est 1
        unsigned char opcode_field = (byte_code >> 4) & 0x07; // Bits 6,5,4
        switch (opcode_field) {
            case 0b000: inst.opcode = OP_ADD; break;
            case 0b001: inst.opcode = OP_SUB; break;
            case 0b010: inst.opcode = OP_MUL; break;
            case 0b011: inst.opcode = OP_DIV; break;
            case 0b100: inst.opcode = OP_IN;  break;
            case 0b101: inst.opcode = OP_OUT; break;
            default:    inst.opcode = OP_ILLEGAL; break;
        }

        if (inst.opcode >= OP_ADD && inst.opcode <= OP_DIV) {
            inst.operands.reg_ops.dest = (Register)((byte_code >> 2) & 0x03);
            inst.operands.reg_ops.src  = (Register)(byte_code & 0x03);
        } else if (inst.opcode == OP_IN || inst.opcode == OP_OUT) {
            inst.operands.unary_op.reg = (Register)(byte_code & 0x03);
        }
    }
    return inst;
}

// Affiche une instruction à partir de sa structure.
void print_instruction(Instruction inst) {
    const char* reg_names[] = {"A", "B", "C", "D"};
    
    switch (inst.opcode) {
        case OP_MOV: 
            printf("mov D, %d\n", inst.operands.imm_op.immediate); 
            break;
        case OP_ADD: 
            printf("add %s, %s\n", 
                   reg_names[inst.operands.reg_ops.dest], 
                   reg_names[inst.operands.reg_ops.src]); 
            break;
        case OP_SUB: 
            printf("sub %s, %s\n", 
                   reg_names[inst.operands.reg_ops.dest], 
                   reg_names[inst.operands.reg_ops.src]); 
            break;
        case OP_MUL: 
            printf("mul %s, %s\n", 
                   reg_names[inst.operands.reg_ops.dest], 
                   reg_names[inst.operands.reg_ops.src]); 
            break;
        case OP_DIV: 
            printf("div %s, %s\n", 
                   reg_names[inst.operands.reg_ops.dest], 
                   reg_names[inst.operands.reg_ops.src]); 
            break;
        case OP_IN:  
            printf("in %s\n", reg_names[inst.operands.unary_op.reg]); 
            break;
        case OP_OUT: 
            printf("out %s\n", reg_names[inst.operands.unary_op.reg]); 
            break;
        default:     
            printf("illegal\n"); 
            break;
    }
}
```

### 3.3 L'Encodeur (Mnémonique vers Binaire)

L'encodeur, ou assembleur, effectue l'opération inverse : il prend une ligne de code assembleur sous forme de texte et la convertit en son octet binaire correspondant. C'est une tâche plus complexe car elle implique une analyse syntaxique (parsing) du texte.

**Pseudo-code :**

```
FONCTION encoder(chaine_instruction) : octet
  (mnemonique, op1, op2) = parser(chaine_instruction)
  
  SI mnemonique == "mov" ALORS
    valeur_immediate = convertir_en_entier(op2)
    RETOURNER 0x00 | (valeur_immediate & 0x7F)
  SINON SI mnemonique == "add" ALORS
    opcode_binaire = 0b10000000
    reg_dest = convertir_registre(op1) // "A" -> 00
    reg_src = convertir_registre(op2)  // "B" -> 01
    RETOURNER opcode_binaire | (reg_dest << 2) | reg_src
  //... autres cas...
  FIN SI
```

**Implémentation en C :**

Une implémentation robuste utiliserait des expressions régulières pour le parsing. Pour la simplicité, nous pouvons utiliser sscanf qui est suffisant pour des formats simples.

```c
#include <stdio.h>
#include <string.h>
#include "microcalc.h"

// Fonction utilitaire pour convertir un nom de registre en sa valeur entière.
int parse_register(char* name) {
    if (strcmp(name, "A") == 0) return REG_A;
    if (strcmp(name, "B") == 0) return REG_B;
    if (strcmp(name, "C") == 0) return REG_C;
    if (strcmp(name, "D") == 0) return REG_D;
    return -1; // Erreur
}

// Encode une ligne d'assembleur en un octet.
unsigned char encode(char* line) {
    char mnemonic[10];
    char op1_str[10], op2_str[10];
    
    // Tentative de parser "mnem op1, op2"
    if (sscanf(line, "%s %[^,], %s", mnemonic, op1_str, op2_str) == 3) {
        int reg1 = parse_register(op1_str);
        int reg2 = parse_register(op2_str);
        if (reg1 == -1 || reg2 == -1) return 0xFF; // Opcode illégal

        if (strcmp(mnemonic, "add") == 0) return 0x80 | (reg1 << 2) | reg2;
        if (strcmp(mnemonic, "sub") == 0) return 0x90 | (reg1 << 2) | reg2;
        if (strcmp(mnemonic, "mul") == 0) return 0xA0 | (reg1 << 2) | reg2;
        if (strcmp(mnemonic, "div") == 0) return 0xB0 | (reg1 << 2) | reg2;

    } 
    // Tentative de parser "mnem op1"
    else if (sscanf(line, "%s %s", mnemonic, op1_str) == 2) {
        if (strcmp(mnemonic, "mov") == 0) { // Cas spécial: mov D, imm
            int imm;
            sscanf(op1_str, "%d", &imm); // Parsing de la valeur immédiate
            return (unsigned char)(imm & 0x7F);
        }
        int reg = parse_register(op1_str);
        if (reg == -1) return 0xFF;

        if (strcmp(mnemonic, "in") == 0) return 0xC0 | reg;
        if (strcmp(mnemonic, "out") == 0) return 0xD0 | reg;
    }
    return 0xFF; // Opcode illégal
}
```

### 3.4 Le Simulateur

Le simulateur est le programme qui imite le comportement du matériel. Il lit un programme binaire, maintient un état des registres et exécute chaque instruction séquentiellement. C'est ici que la réutilisabilité de notre structure Instruction prend tout son sens.

**Pseudo-code :**

```
PROCÉDURE simulateur(programme_binaire)
  registres = [0, 0, 0, 0] // A, B, C, D
  compteur_programme = 0
  
  TANT QUE compteur_programme < longueur(programme_binaire)
    octet_courant = programme_binaire[compteur_programme]
    instruction = decoder(octet_courant) // Réutilisation du décodeur!
    
    CAS instruction.opcode:
      QUAND OP_MOV:
        registres[REG_D] = instruction.operands.imm_op.immediate
      QUAND OP_ADD:
        dest = instruction.operands.reg_ops.dest
        src = instruction.operands.reg_ops.src
        registres[dest] = registres[dest] + registres[src]
      //... autres cas...
      QUAND OP_IN:
        valeur_lue = lire_clavier()
        registres[instruction.operands.unary_op.reg] = valeur_lue
      QUAND OP_OUT:
        afficher_ecran(registres[instruction.operands.unary_op.reg])
    FIN CAS
    
    compteur_programme = compteur_programme + 1
  FIN TANT QUE
```

**Implémentation en C :**

```c
#include <stdio.h>
#include "microcalc.h"

// Déclaration de la fonction de décodage
Instruction decode(unsigned char byte_code);

// Simule l'exécution d'un programme binaire.
void simulate(unsigned char* program, int prog_len) {
    unsigned char regs[4] = {0}; // A, B, C, D
    int pc = 0; // Program Counter

    while (pc < prog_len) {
        Instruction inst = decode(program[pc]);
        
        switch (inst.opcode) {
            case OP_MOV:
                regs[REG_D] = inst.operands.imm_op.immediate;
                break;
            case OP_ADD:
                regs[inst.operands.reg_ops.dest] += regs[inst.operands.reg_ops.src];
                break;
            case OP_SUB:
                regs[inst.operands.reg_ops.dest] -= regs[inst.operands.reg_ops.src];
                break;
            case OP_MUL:
                regs[inst.operands.reg_ops.dest] *= regs[inst.operands.reg_ops.src];
                break;
            case OP_DIV:
                if (regs[inst.operands.reg_ops.src] != 0) {
                    regs[inst.operands.reg_ops.dest] /= regs[inst.operands.reg_ops.src];
                } else {
                    fprintf(stderr, "Erreur: Division par zéro à l'instruction %d\n", pc);
                    return;
                }
                break;
            case OP_IN:
                printf("Entrée pour %c: ", 'A' + inst.operands.unary_op.reg);
                scanf("%hhu", &regs[inst.operands.unary_op.reg]);
                break;
            case OP_OUT:
                printf("Sortie de %c: %d\n", 'A' + inst.operands.unary_op.reg, 
                       regs[inst.operands.unary_op.reg]);
                break;
            case OP_ILLEGAL:
                fprintf(stderr, "Erreur: Instruction illégale 0x%02X à l'adresse %d\n", 
                        program[pc], pc);
                return;
        }
        pc++;
    }
}
```

## Partie 4 : Du Concept à la Réalité – L'Architecture x86

Après avoir construit notre propre monde informatique de A à Z, nous sommes maintenant équipés pour aborder une architecture du monde réel. Nous abandonnons notre modèle 8 bits simple pour explorer l'architecture 64 bits qui domine le paysage informatique : x86-64.

### 4.1 Le Modèle de Programmation x86-64

La différence la plus immédiate est l'échelle. Au lieu de quatre registres 8 bits, l'architecture x86-64 en mode 64 bits offre seize registres d'usage général de 64 bits : RAX, RBX, RCX, RDX, RSI, RDI, RBP, RSP, ainsi que huit registres supplémentaires nommés de R8 à R15.

Une des principales sources de confusion pour les débutants est la convention de nommage des registres, qui est un héritage direct de l'évolution de l'architecture depuis ses origines 16 bits.

Un registre 64 bits comme RAX contient une sous-partie 32 bits nommée EAX. EAX contient à son tour une sous-partie 16 bits nommée AX. Enfin, AX est lui-même divisible en une partie haute de 8 bits, AH (High), et une partie basse de 8 bits, AL (Low).

Cette hiérarchie s'applique aux registres RAX, RBX, RCX et RDX. Pour les registres plus modernes (R8 à R15), la nomenclature est plus systématique : le registre 64 bits est R8, sa partie 32 bits est R8D (Doubleword), sa partie 16 bits est R8W (Word), et sa partie 8 bits est R8B (Byte).

Le tableau suivant clarifie ces relations, qui sont essentielles pour lire et comprendre le code assembleur x86.

**Tableau 3 : Registres d'Usage Général x86-64 et Conventions de Nommage**

| 64-bit | 32-bit | 16-bit | 8-bit (High) | 8-bit (Low) | Usage Conventionnel (ABI System V) |
|--------|--------|--------|--------------|-------------|-------------------------------------|
| RAX | EAX | AX | AH | AL | Accumulateur, valeur de retour de fonction |
| RBX | EBX | BX | BH | BL | Registre préservé par l'appelé (callee-saved) |
| RCX | ECX | CX | CH | CL | 4ème argument de fonction |
| RDX | EDX | DX | DH | DL | 3ème argument de fonction, extension de RAX |
| RSI | ESI | SI | - | SIL | 2ème argument de fonction |
| RDI | EDI | DI | - | DIL | 1er argument de fonction |
| RBP | EBP | BP | - | BPL | Pointeur de base de la pile (callee-saved) |
| RSP | ESP | SP | - | SPL | Pointeur de sommet de la pile |
| R8 | R8D | R8W | - | R8B | 5ème argument de fonction |
| R9 | R9D | R9W | - | R9B | 6ème argument de fonction |
| R10 - R15 | R10D - R15D | R10W - R15W | - | R10B - R15B | Registres généraux (R12-R15 sont callee-saved) |

### 4.2 Le Jeu d'Instructions x86 : Complexité et Héritage (CISC)

Si les registres sont plus nombreux, la véritable complexité de x86 réside dans son jeu d'instructions. La source de vérité absolue est le Intel® 64 and IA-32 Architectures Software Developer's Manual (SDM), une documentation technique monumentale en plusieurs volumes qui détaille chaque aspect de l'architecture.

En consultant ce manuel, on découvre qu'une seule mnémonique, comme SUB (soustraction), peut correspondre à des dizaines d'opcodes binaires différents. La machine choisit l'opcode approprié en fonction du type et de la taille des opérandes : soustraire un registre de 8 bits d'un autre, soustraire une valeur 32 bits en mémoire d'un registre, soustraire une constante de 8 bits d'un registre de 64 bits, etc. Chacune de ces variations a son propre encodage optimisé pour la compacité. C'est la marque de fabrique d'une architecture CISC (Complex Instruction Set Computer).

## Partie 5 : Analyse de Code Avancée – L'Algorithme d'Euclide en x86

Armés de notre connaissance des principes de l'assembleur et d'une introduction à l'architecture x86, nous pouvons maintenant analyser un véritable programme. L'exemple choisi est une implémentation de l'algorithme d'Euclide, un classique de l'algorithmique.

### 5.1 Sauts Conditionnels et le Registre EFLAGS

Un programme n'est pas seulement une séquence linéaire d'instructions. Sa puissance vient de sa capacité à prendre des décisions et à modifier son flux d'exécution. En assembleur, cela est réalisé par des sauts conditionnels. Ces sauts ne sont pas magiques ; ils dépendent de l'état du processeur, qui est méticuleusement enregistré dans un registre spécial : EFLAGS (sur 32 bits) ou RFLAGS (sur 64 bits).

Deux instructions principales sont utilisées pour mettre à jour ce registre en vue d'un saut :
- **CMP op1, op2** : Compare les deux opérandes en effectuant une soustraction interne (op1 - op2). Le résultat de la soustraction est jeté, mais les drapeaux (flags) du registre EFLAGS sont mis à jour en fonction de ce résultat (par exemple, si le résultat est nul, le Zero Flag est activé).
- **TEST op1, op2** : Effectue une opération ET logique au niveau du bit (op1 & op2). De même, le résultat est jeté, mais les drapeaux sont positionnés. L'idiome `TEST reg, reg` est le moyen le plus efficace de vérifier si la valeur d'un registre est nulle.

Les drapeaux les plus importants pour le contrôle de flux sont :
- **ZF (Zero Flag)** : Positionné à 1 si le résultat de la dernière opération arithmétique ou logique était zéro.
- **SF (Sign Flag)** : Positionné à 1 si le résultat était négatif (c'est-à-dire si son bit de poids fort était à 1).
- **CF (Carry Flag)** : Positionné à 1 s'il y a eu une retenue (pour une addition) ou un emprunt (pour une soustraction) sur le bit de poids fort, typiquement utilisé pour l'arithmétique non signée.
- **OF (Overflow Flag)** : Positionné à 1 s'il y a eu un dépassement de capacité pour une opération sur des nombres signés (par exemple, l'addition de deux grands nombres positifs donne un résultat négatif).

Le tableau suivant résume la relation entre les sauts conditionnels les plus courants et l'état des drapeaux.

**Tableau 4 : Principaux Drapeaux du Registre EFLAGS et Sauts Conditionnels Associés**

| Instruction de Saut | Mnémonique (Signification) | Condition sur les Drapeaux |
|---------------------|----------------------------|----------------------------|
| JE / JZ | Jump if Equal / Jump if Zero | ZF = 1 |
| JNE / JNZ | Jump if Not Equal / Jump if Not Zero | ZF = 0 |
| JS | Jump if Sign (négatif) | SF = 1 |
| JNS | Jump if Not Sign (positif ou nul) | SF = 0 |
| JG / JNLE | Jump if Greater (signé : op1 > op2) | ZF = 0 ET SF = OF |
| JL / JNGE | Jump if Less (signé : op1 < op2) | SF ≠ OF |
| JA / JNBE | Jump if Above (non signé : op1 > op2) | CF = 0 ET ZF = 0 |
| JB / JNAE | Jump if Below (non signé : op1 < op2) | CF = 1 |

### 5.2 Étude de Cas – L'Algorithme d'Euclide

#### 5.2.1 Contexte Historique et Mathématique

L'algorithme d'Euclide est l'un des plus anciens algorithmes encore largement utilisés. Il a été décrit par le mathématicien grec Euclide dans son traité "Les Éléments" vers 300 av. J.-C. Sa fonction est de trouver le plus grand commun diviseur (PGCD) de deux entiers. Sa formulation mathématique est élégamment récursive : pour deux entiers m et n, $\text{PGCD}(m,n) = \text{PGCD}(n, m \bmod n)$, avec le cas de base $\text{PGCD}(m,0) = m$. 

L'algorithme a une importance qui dépasse largement ce simple calcul ; il est fondamental en théorie des nombres et joue un rôle clé dans des applications modernes comme la cryptographie à clé publique (par exemple, l'algorithme RSA). L'analyse de son efficacité et de ses propriétés est un sujet classique, traité en profondeur par des sommités comme Donald Knuth dans son œuvre magistrale "The Art of Computer Programming".

#### 5.2.2 Analyse Détaillée du Code Assembleur

Analysons une implémentation x86 de cet algorithme, telle que présentée dans le séminaire. Supposons que les deux nombres, m et n, sont passés à la fonction dans les registres EAX et ECX respectivement.

**Extrait de code**

```asm
; EAX = m, ECX = n
; La fonction retourne le PGCD dans EAX.

euclid_gcd:
    test ecx, ecx   ; Teste si n == 0.
    jz end_loop     ; Si oui, le PGCD est m (dans EAX), donc on termine.

loop_start:
    cdq             ; Étend le signe de EAX sur EDX. Prépare le dividende EDX:EAX.
    idiv ecx        ; Divise EDX:EAX par ECX.
                    ; Quotient -> EAX, Reste -> EDX.
    
    mov eax, ecx    ; Le nouveau m (dans EAX) est l'ancien n (de ECX).
    mov ecx, edx    ; Le nouveau n (dans ECX) est le reste (de EDX).
    
    test ecx, ecx   ; Teste le nouveau n (le reste).
    jnz loop_start  ; Si le reste n'est pas nul, on boucle.

end_loop:
    ret             ; Retourne. Le résultat est dans EAX.
```

Détaillons le rôle de chaque instruction clé :

- **cdq (Convert Doubleword to Quadword)** : Cette instruction est une préparation indispensable à la division signée. L'instruction idiv avec un diviseur 32 bits (comme ecx) attend un dividende de 64 bits dans la paire de registres EDX:EAX. L'instruction cdq prend le bit de signe (bit 31) de EAX et le copie dans tous les bits de EDX. Si EAX est positif, EDX devient 0. Si EAX est négatif, EDX devient -1 (soit 0xFFFFFFFF). Cela garantit que la valeur 64 bits EDX:EAX représente correctement le même nombre signé que EAX.

- **idiv ecx (Signed Integer Divide)** : C'est le cœur du calcul. Elle effectue la division signée EDX:EAX / ecx. Le résultat est stocké en deux parties : le quotient est placé dans EAX et, plus important pour nous, le reste est placé dans EDX. C'est l'opération $m \bmod n$.

- **mov eax, ecx** et **mov ecx, edx** : Ces deux instructions mettent à jour les variables pour la prochaine itération, en appliquant la définition de l'algorithme. L'ancien diviseur (ecx) devient le nouveau dividende (eax), et le reste (edx) devient le nouveau diviseur (ecx).

- **test ecx, ecx** et **jnz loop_start** : C'est la condition de la boucle. `test ecx, ecx` vérifie si le nouveau diviseur (le reste) est nul. Si ce n'est pas le cas (ZF est à 0), `jnz` (Jump if Not Zero) renvoie l'exécution au début de la boucle. Lorsque le reste est enfin nul, le saut n'est pas pris, et le programme continue vers ret. À ce moment, EAX contient la dernière valeur non nulle du diviseur, qui est le PGCD.

## Conclusion : Synthèse et Retour sur la Thèse Initiale

Notre parcours nous a menés de la conception d'une architecture 8 bits rudimentaire à l'analyse d'un code s'exécutant sur une puissante architecture 64 bits. Ce faisant, nous avons exploré les concepts fondamentaux qui constituent le langage des ordinateurs : l'ISA comme contrat matériel-logiciel, la dualité mnémonique/opcode, les compromis de l'encodage binaire, le rôle central des registres, et le mécanisme des drapeaux et des sauts conditionnels qui donne vie à la logique de nos programmes.

Cet exercice a permis de valider notre thèse initiale. L'algorithme d'Euclide en assembleur x86, bien que syntaxiquement plus complexe et utilisant des instructions aux noms différents, repose sur les mêmes principes que nous aurions employés pour l'implémenter sur notre microcalculateur. L'idée de diviser, de récupérer un reste, de mettre à jour des variables dans des registres et de boucler tant qu'une condition (basée sur un test de nullité) n'est pas remplie est universelle. La complexité de l'architecture x86 réside dans son immense héritage, la multitude de ses instructions et de ses modes d'adressage, mais non dans les principes fondamentaux de calcul.

La compétence que vous avez développée aujourd'hui n'est donc pas la mémorisation d'un dialecte d'assembleur particulier. C'est la capacité d'analyser n'importe quel système à partir de ses principes premiers, de déduire la fonction d'un code machine en comprenant la sémantique des opérations de base et le flux de contrôle. C'est cette compétence qui est véritablement universelle et qui vous permettra de naviguer avec confiance dans le monde de la programmation de bas niveau, quelle que soit l'architecture que vous rencontrerez demain.
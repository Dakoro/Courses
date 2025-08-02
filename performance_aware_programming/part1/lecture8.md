# Cours : Construction d'un Simulateur de CPU 8086 - Partie 1 : Décodage d'Instructions

## Introduction

Bienvenue dans la première partie pratique de notre série sur la programmation consciente des performances. Nous allons adopter une approche unique et puissante pour comprendre le fonctionnement des processeurs : nous allons en construire un nous-mêmes ! Comme le dit l'adage : "Si vous savez comment quelque chose fonctionne, vous pouvez le construire. Si vous ne savez pas comment cela fonctionne, vous ne pouvez certainement pas."

## Partie 1 : Philosophie et Objectifs du Cours

### 1.1 L'Approche Pédagogique

```pseudocode
Principe_Apprentissage = {
    "Comprendre en construisant",
    "Simuler la fonctionnalité, pas les circuits",
    "Progresser du simple vers le complexe"
}
```

Notre objectif : construire un simulateur de CPU suffisamment complet pour comprendre profondément son fonctionnement, sans la complexité inutile d'une implémentation complète.

### 1.2 Pourquoi le 8086 ?

Le choix du processeur Intel 8086 (1978) est stratégique :

| Aspect | 8086 (1978) | Processeurs Modernes (2024) |
|--------|-------------|----------------------|
| Largeur des registres | 16 bits | 64 bits |
| Nombre d'instructions | ~100 | ~1000+ |
| Complexité du pipeline | Simple | 20+ étages |
| Prédiction de branchement | Non | Oui, très sophistiquée |
| Exécution spéculative | Non | Oui, massive |

Le 8086 nous offre la simplicité nécessaire pour comprendre les concepts fondamentaux sans être submergés par 40+ années d'évolution architecturale.

## Partie 2 : Architecture Fondamentale du 8086

### 2.1 Les Registres : Cœur du Processeur

Dans le 8086, un registre est littéralement un ensemble de transistors capable de stocker 16 bits de données :

```pseudocode
Structure Registre8086 :
    bits[16] : tableau de bits
    nom : chaîne
    
Registres_8086 = {
    AX : Registre16bits,  // Accumulateur
    BX : Registre16bits,  // Base
    CX : Registre16bits,  // Compteur
    DX : Registre16bits,  // Données
    SI : Registre16bits,  // Index Source
    DI : Registre16bits,  // Index Destination
    BP : Registre16bits,  // Pointeur de Base
    SP : Registre16bits   // Pointeur de Pile
}
```

### 2.2 Le Modèle de Calcul Fondamental

Le modèle de calcul du 8086 suit un cycle en trois étapes :

```pseudocode
Algorithme : CycleCalcul8086
1. CHARGER : Copier des bits depuis la mémoire vers un registre
2. CALCULER : Effectuer des opérations sur les registres
3. STOCKER : Copier les bits du registre vers la mémoire
```

Ce modèle reste fondamentalement le même aujourd'hui, malgré toutes les optimisations modernes !

### 2.3 Accès aux Sous-Registres

Une particularité du 8086 : chaque registre 16 bits peut être accédé par moitiés :

```pseudocode
Structure RegistreComplet :
    X : 16 bits complets (ex: AX)
    H : 8 bits hauts    (ex: AH) // bits 15-8
    L : 8 bits bas      (ex: AL) // bits 7-0

Exemple :
    Si AX = 0x1234
    Alors AH = 0x12 et AL = 0x34
```

## Partie 3 : Le Flux d'Instructions

### 3.1 Le Problème du Décodage

Avant de pouvoir simuler l'exécution, nous devons comprendre comment le CPU sait quoi faire :

```pseudocode
Algorithme : FluxExécutionCPU
tant que programme_en_cours :
    instruction_binaire ← LireProchainOctets(mémoire)
    opération ← DécoderInstruction(instruction_binaire)
    ExécuterOpération(opération)
    MettreÀJourCompteurProgramme()
```

### 3.2 Le Décodage d'Instructions

Le décodage d'instructions est un processus **physique** réel dans le CPU :

```pseudocode
Fonction DécoderInstruction(octets[]) :
    // Réseau de transistors qui analyse les bits
    opcode ← ExtraireBitsHauts(octets[0])
    
    selon opcode :
        cas PATTERN_MOV : retourner DécoderMOV(octets)
        cas PATTERN_ADD : retourner DécoderADD(octets)
        // ... autres instructions
```

## Partie 4 : L'Instruction MOV - Notre Point de Départ

### 4.1 Syntaxe Assembleur

L'instruction MOV est fondamentale - elle copie des bits d'un endroit à un autre :

```assembly
mov ax, bx  ; Copie le contenu de BX vers AX
```

**Points importants :**
- "mov" est un mnémonique (aide-mémoire humain)
- Syntaxe Intel : destination d'abord, source ensuite
- Les bits ne sont pas "déplacés" mais **copiés**

### 4.2 Encodage Binaire de MOV

L'instruction MOV registre-vers-registre est encodée sur 2 octets :

```
Octet 1: [100010][d][w]
Octet 2: [mod][reg][r/m]

Où :
- 100010 : Code opération pour MOV
- d : bit de direction (0=reg est source, 1=reg est destination)
- w : bit de largeur (0=8 bits, 1=16 bits)
- mod : mode d'adressage (11 = registre-vers-registre)
- reg : code du premier registre (3 bits)
- r/m : code du second registre (3 bits)
```

### 4.3 Table d'Encodage des Registres

```pseudocode
Fonction ObtenirNomRegistre(code_3bits, bit_w) :
    si bit_w = 1 :  // Registres 16 bits
        table = ["AX", "CX", "DX", "BX", "SP", "BP", "SI", "DI"]
    sinon :         // Registres 8 bits
        table = ["AL", "CL", "DL", "BL", "AH", "CH", "DH", "BH"]
    
    retourner table[code_3bits]
```

## Partie 5 : Algorithme de Décodage MOV

### 5.1 Structure de Données pour une Instruction

```pseudocode
Structure InstructionDécodée :
    mnémonique : chaîne
    destination : chaîne
    source : chaîne
    taille_opérande : entier (8 ou 16)
```

### 5.2 Algorithme Complet de Décodage

```pseudocode
Algorithme : DécoderMOV
Entrée : flux_binaire (2 octets minimum)
Sortie : InstructionDécodée

Fonction DécoderInstructionMOV(octet1, octet2) :
    // Vérifier le code opération
    opcode ← (octet1 >> 2) & 0b111111
    si opcode ≠ 0b100010 :
        retourner ERREUR("Pas une instruction MOV")
    
    // Extraire les bits de contrôle
    bit_d ← (octet1 >> 1) & 1
    bit_w ← octet1 & 1
    
    // Extraire les champs du second octet
    mod ← (octet2 >> 6) & 0b11
    reg ← (octet2 >> 3) & 0b111
    r_m ← octet2 & 0b111
    
    // Vérifier mode registre-vers-registre
    si mod ≠ 0b11 :
        retourner ERREUR("Mode mémoire non supporté")
    
    // Déterminer les registres
    reg_nom ← ObtenirNomRegistre(reg, bit_w)
    rm_nom ← ObtenirNomRegistre(r_m, bit_w)
    
    // Déterminer source et destination selon bit_d
    si bit_d = 1 :
        destination ← reg_nom
        source ← rm_nom
    sinon :
        destination ← rm_nom
        source ← reg_nom
    
    retourner InstructionDécodée{
        mnémonique: "mov",
        destination: destination,
        source: source,
        taille_opérande: si bit_w alors 16 sinon 8
    }
```

## Partie 6 : Exercice Pratique - Votre Désassembleur

### 6.1 Spécifications du Programme

Votre mission : créer un désassembleur qui peut décoder des instructions MOV binaires.

```pseudocode
Programme : Désassembleur8086
Entrée : Fichier binaire contenant des instructions 8086
Sortie : Code assembleur équivalent

Fonction Principal(nom_fichier) :
    octets ← LireFichierBinaire(nom_fichier)
    position ← 0
    
    print("; " + nom_fichier)
    print("bits 16")
    print("")
    
    tant que position < longueur(octets) :
        instruction ← DécoderInstruction(octets[position:])
        print(FormaterInstruction(instruction))
        position += instruction.taille_en_octets
```

### 6.2 Exemples de Fichiers de Test

**Listing 37 (simple):**
```assembly
bits 16
mov cx, bx
```

**Listing 38 (complexe):**
```assembly
bits 16
mov cx, bx
mov ch, ah
mov dx, bx
mov si, bx
mov bx, di
mov al, cl
mov ch, ch
mov bx, ax
mov bx, si
mov sp, di
mov bp, ax
```

### 6.3 Stratégie de Test Automatisé

```pseudocode
Algorithme : TestAutomatiséDésassembleur
Entrée : fichier_asm_original
Sortie : succès ou échec

1. binaire_original ← Assembler(fichier_asm_original)
2. désassemblage ← Désassembler(binaire_original)
3. binaire_réassemblé ← Assembler(désassemblage)
4. si binaire_original = binaire_réassemblé :
       retourner SUCCÈS
   sinon :
       retourner ÉCHEC
```

## Partie 7 : Guide d'Implémentation

### 7.1 Lecture des Bits

```pseudocode
// Fonctions utilitaires pour manipuler les bits
Fonction ExtraireBits(valeur, position, nombre_bits) :
    masque ← (1 << nombre_bits) - 1
    retourner (valeur >> position) & masque

Fonction TesterBit(valeur, position) :
    retourner (valeur >> position) & 1
```

### 7.2 Structure Suggérée du Programme

```pseudocode
Classe Désassembleur8086 :
    
    Méthode DésassemblerFichier(chemin) :
        données ← LireFichierBinaire(chemin)
        instructions ← []
        offset ← 0
        
        tant que offset < longueur(données) :
            instr, taille ← DécoderProchaineInstruction(données, offset)
            instructions.ajouter(instr)
            offset += taille
        
        retourner instructions
    
    Méthode DécoderProchaineInstruction(données, offset) :
        premier_octet ← données[offset]
        
        // Identifier le type d'instruction
        si EstInstructionMOV(premier_octet) :
            retourner DécoderMOV(données, offset)
        // Ajouter d'autres instructions ici plus tard
        
        retourner ERREUR("Instruction inconnue")
```

### 7.3 Gestion des Erreurs

```pseudocode
Fonction ValiderInstructionMOV(octets, offset) :
    // Vérifier qu'il reste assez d'octets
    si offset + 2 > longueur(octets) :
        retourner ERREUR("Fin de fichier inattendue")
    
    // Vérifier le code opération
    opcode ← (octets[offset] >> 2) & 0b111111
    si opcode ≠ 0b100010 :
        retourner ERREUR("Code opération invalide")
    
    // Vérifier le mode
    mod ← (octets[offset + 1] >> 6) & 0b11
    si mod ≠ 0b11 :
        retourner ERREUR("Mode non supporté pour cet exercice")
    
    retourner OK
```

## Partie 8 : Ressources et Outils

### 8.1 Documentation de Référence

- **Manuel Intel 8086** : Document de référence officiel (~200 pages)
  - Page 160+ : Section sur l'encodage des instructions
  - Table 4-13 : Encodage des registres
  - Table 4-12 : Formats d'instructions spécifiques

### 8.2 Outils Nécessaires

1. **Assembleur** : NASM (Netwide Assembler)
   ```bash
   nasm listing37.asm  # Produit listing37 (binaire)
   ```

2. **Éditeur Hexadécimal** : Pour examiner les binaires produits

3. **Langage de Programmation** : Au choix, tant qu'il supporte :
   - Lecture de fichiers binaires
   - Opérations bit à bit
   - Affichage formaté

## Partie 9 : Extensions Possibles

### 9.1 Après l'Exercice de Base

Une fois MOV registre-vers-registre maîtrisé :

```pseudocode
Extensions_Possibles = {
    "MOV avec adressage mémoire",
    "Instructions arithmétiques (ADD, SUB)",
    "Instructions logiques (AND, OR, XOR)",
    "Instructions de saut (JMP, JZ, JNZ)",
    "Gestion complète des préfixes"
}
```

### 9.2 Vers un Émulateur Complet

```pseudocode
Classe Émulateur8086 :
    registres : TableauRegistres
    mémoire : TableauOctets[1MB]
    flags : RegistreDrapeaux
    
    Méthode Exécuter(programme) :
        IP ← 0  // Instruction Pointer
        
        tant que non arrêt :
            instruction ← DécoderInstruction(mémoire[IP])
            
            selon instruction.type :
                cas MOV : ExécuterMOV(instruction)
                cas ADD : ExécuterADD(instruction)
                // etc.
            
            IP += instruction.taille
```

## Conclusion

Ce premier exercice pose les fondations de notre compréhension du fonctionnement des processeurs. En construisant un désassembleur pour l'instruction MOV, vous apprenez :

1. **Le décodage d'instructions** - Comment le CPU interprète les bits
2. **L'architecture des registres** - L'organisation fondamentale du stockage CPU
3. **L'encodage binaire** - La représentation machine des opérations
4. **Le cycle fetch-decode-execute** - Le rythme fondamental du processeur

### Points Clés à Retenir

- Chaque instruction a un encodage binaire spécifique et déterministe
- Le décodage est un processus physique réel dans le CPU
- L'architecture 8086, bien que simple, contient tous les concepts fondamentaux
- Construire un simulateur est la meilleure façon de comprendre profondément

### Prochaine Étape

Dans le prochain cours, nous étendrons notre désassembleur pour gérer des modes d'adressage plus complexes, incluant les accès mémoire. Cela nous rapprochera d'un simulateur complet capable d'exécuter de vrais programmes 8086 !

**Rappel** : L'objectif n'est pas la perfection, mais la compréhension. Même un désassembleur basique qui fonctionne vous enseignera énormément sur le fonctionnement interne des processeurs.
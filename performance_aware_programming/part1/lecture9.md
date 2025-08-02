# Cours : Extension du Décodage d'Instructions 8086 - Accès Mémoire et Valeurs Immédiates

## Introduction

Bienvenue dans cette deuxième partie de notre construction d'un simulateur 8086. Après avoir maîtrisé le décodage des instructions MOV registre-vers-registre, nous allons maintenant étendre nos capacités pour gérer les accès mémoire et les valeurs immédiates. Cette extension révélera la complexité réelle du décodage d'instructions x86 et les défis que les concepteurs de processeurs doivent relever.

## Partie 1 : La Complexité Cachée du Décodage x86

### 1.1 Le Problème de la Longueur Variable

Un des défis majeurs de l'architecture x86 est que les instructions ont des longueurs variables :

```pseudocode
Algorithme : ProblèmeLongueurVariable
Une instruction peut être :
- 1 octet (certaines instructions simples)
- 2 octets (MOV registre-vers-registre)
- 3 octets (MOV avec déplacement 8 bits)
- 4 octets (MOV avec déplacement 16 bits)
- Et plus encore...

Conséquence : Le CPU doit décoder partiellement l'instruction
             pour savoir combien d'octets lire au total
```

### 1.2 Impact sur les Processeurs Modernes

```pseudocode
Défi_Processeurs_Modernes = {
    "Comment décoder 6 instructions/cycle si on ne sait pas où commence la 2e ?",
    "Solution : Prédiction de longueur + décodage spéculatif",
    "Coût : Complexité matérielle énorme"
}
```

## Partie 2 : Extension du Champ MOD - Accès Mémoire

### 2.1 Rappel : Encodage MOV avec MOD/REG/R_M

```
Premier octet : [100010][d][w]
Second octet  : [mod][reg][r/m]
Octets optionnels : [disp-lo] [disp-hi]
```

### 2.2 Signification du Champ MOD

```pseudocode
Fonction InterpréterMOD(mod_bits) :
    selon mod_bits :
        cas 00 : retourner "Pas de déplacement (cas spécial pour R/M=110)"
        cas 01 : retourner "Déplacement 8 bits signé"
        cas 10 : retourner "Déplacement 16 bits"
        cas 11 : retourner "Registre-vers-registre (pas de mémoire)"
```

### 2.3 Notation Assembleur pour les Accès Mémoire

```assembly
; Chargement (mémoire vers registre)
mov bx, [75]        ; Charge depuis l'adresse 75
mov ax, [bx + si]   ; Charge depuis l'adresse bx + si
mov dx, [bp + 10]   ; Charge depuis l'adresse bp + 10

; Stockage (registre vers mémoire)
mov [100], ax       ; Stocke ax à l'adresse 100
mov [bx + di], cx   ; Stocke cx à l'adresse bx + di
```

### 2.4 Comprendre les Adresses Mémoire

```pseudocode
Modèle_Mémoire_8086 :
    Mémoire[65536] : tableau d'octets  // 64 KB au total
    
    // Une adresse est simplement un index dans ce tableau
    Fonction Lire16Bits(adresse) :
        octet_bas ← Mémoire[adresse]
        octet_haut ← Mémoire[adresse + 1]
        retourner (octet_haut << 8) | octet_bas
```

## Partie 3 : Calcul d'Adresse Effective

### 3.1 Table de Décodage R/M

Le champ R/M (3 bits) encode différents modes d'adressage :

```pseudocode
Fonction DécoderAdresseEffective(mod, r_m) :
    expressions_base = [
        "[BX + SI]",  // 000
        "[BX + DI]",  // 001
        "[BP + SI]",  // 010
        "[BP + DI]",  // 011
        "[SI]",       // 100
        "[DI]",       // 101
        "[BP]",       // 110 (sauf si MOD=00)
        "[BX]"        // 111
    ]
    
    si mod = 0b00 ET r_m = 0b110 :
        // Cas spécial : adressage direct
        retourner "Adresse directe 16 bits"
    sinon :
        retourner expressions_base[r_m]
```

### 3.2 Ajout du Déplacement

```pseudocode
Fonction ConstruireExpressionComplète(mod, r_m, déplacement) :
    base ← DécoderAdresseEffective(mod, r_m)
    
    selon mod :
        cas 0b00 :
            si r_m = 0b110 :
                retourner f"[{déplacement}]"  // Adresse directe
            sinon :
                retourner base  // Pas de déplacement
        
        cas 0b01 :
            // Déplacement 8 bits signé
            si déplacement >= 128 :
                déplacement ← déplacement - 256  // Extension de signe
            retourner f"{base[:-1]} + {déplacement}]"
        
        cas 0b10 :
            // Déplacement 16 bits
            retourner f"{base[:-1]} + {déplacement}]"
```

### 3.3 Limitations des Expressions d'Adresse

```pseudocode
Expressions_Valides = {
    "[BX]", "[SI]", "[DI]",  // Registres seuls
    "[BX + SI]", "[BX + DI]", "[BP + SI]", "[BP + DI]",  // Combinaisons
    "[registre + déplacement]",  // Avec constante
    "[adresse_directe]"  // Adresse absolue
}

Expressions_Invalides = {
    "[AX]",  // AX ne peut pas être utilisé pour l'adressage !
    "[CX + DX]",  // Combinaisons non autorisées
    "[BX + SI + DI]",  // Trop de registres
    "[2*BX]"  // Pas de multiplication
}
```

## Partie 4 : Algorithme Complet de Décodage MOV Étendu

### 4.1 Structure de Données Étendue

```pseudocode
Structure OperandeMémoire :
    registre_base : chaîne ou null
    registre_index : chaîne ou null
    déplacement : entier
    est_adresse_directe : booléen

Structure InstructionDécodée :
    mnémonique : chaîne
    destination : Operande (registre ou mémoire)
    source : Operande (registre ou mémoire)
    taille_totale : entier  // Nombre d'octets de l'instruction
```

### 4.2 Décodeur MOV Complet

```pseudocode
Fonction DécoderMOVComplet(octets, offset) :
    octet1 ← octets[offset]
    octet2 ← octets[offset + 1]
    
    // Vérifier l'opcode
    si (octet1 >> 2) ≠ 0b100010 :
        retourner ERREUR("Pas un MOV reg/mem")
    
    // Extraire les champs
    bit_d ← (octet1 >> 1) & 1
    bit_w ← octet1 & 1
    mod ← (octet2 >> 6) & 0b11
    reg ← (octet2 >> 3) & 0b111
    r_m ← octet2 & 0b111
    
    // Décoder le registre
    op_registre ← ObtenirNomRegistre(reg, bit_w)
    
    // Décoder l'opérande mémoire/registre selon MOD
    taille_instruction ← 2
    
    si mod = 0b11 :
        // Registre-vers-registre
        op_rm ← ObtenirNomRegistre(r_m, bit_w)
    sinon :
        // Accès mémoire
        op_rm ← DécoderOpérandeMémoire(mod, r_m, octets, offset + 2)
        
        // Ajouter la taille du déplacement
        si mod = 0b00 ET r_m = 0b110 :
            taille_instruction += 2  // Adresse directe 16 bits
        sinon si mod = 0b01 :
            taille_instruction += 1  // Déplacement 8 bits
        sinon si mod = 0b10 :
            taille_instruction += 2  // Déplacement 16 bits
    
    // Déterminer source et destination selon bit_d
    si bit_d = 1 :
        destination ← op_registre
        source ← op_rm
    sinon :
        destination ← op_rm
        source ← op_registre
    
    retourner InstructionDécodée{
        mnémonique: "mov",
        destination: destination,
        source: source,
        taille_totale: taille_instruction
    }
```

### 4.3 Décodage de l'Opérande Mémoire

```pseudocode
Fonction DécoderOpérandeMémoire(mod, r_m, octets, offset_disp) :
    déplacement ← 0
    
    // Lire le déplacement si nécessaire
    si mod = 0b00 ET r_m = 0b110 :
        // Adresse directe 16 bits
        déplacement ← octets[offset_disp] | (octets[offset_disp + 1] << 8)
        retourner f"[{déplacement}]"
    
    sinon si mod = 0b01 :
        // Déplacement 8 bits signé
        déplacement ← octets[offset_disp]
        si déplacement >= 128 :
            déplacement -= 256  // Extension de signe
    
    sinon si mod = 0b10 :
        // Déplacement 16 bits
        déplacement ← octets[offset_disp] | (octets[offset_disp + 1] << 8)
    
    // Construire l'expression d'adresse
    retourner ConstruireExpression(r_m, déplacement, mod)
```

## Partie 5 : Instructions MOV avec Valeurs Immédiates

### 5.1 Nouvel Opcode : 1011

```pseudocode
Structure EncodageImmédiat :
    Premier octet : [1011][w][reg]
    Octets suivants : valeur immédiate (8 ou 16 bits selon w)
```

**Attention** : La position du bit W a changé ! Il est maintenant en position 3, pas 0.

### 5.2 Syntaxe Assembleur

```assembly
mov ax, 12      ; Déplace la valeur 12 dans AX (16 bits)
mov al, 12      ; Déplace la valeur 12 dans AL (8 bits)
mov cx, -12     ; Déplace -12 (ou 65524) dans CX
mov bx, 3948    ; Déplace 3948 dans BX
```

### 5.3 Décodeur pour MOV Immédiat

```pseudocode
Fonction DécoderMOVImmédiat(octets, offset) :
    octet1 ← octets[offset]
    
    // Vérifier l'opcode
    si (octet1 >> 4) ≠ 0b1011 :
        retourner ERREUR("Pas un MOV immédiat")
    
    // Extraire les champs
    bit_w ← (octet1 >> 3) & 1
    reg ← octet1 & 0b111
    
    // Décoder le registre destination
    destination ← ObtenirNomRegistre(reg, bit_w)
    
    // Lire la valeur immédiate
    si bit_w = 0 :
        // Valeur 8 bits
        valeur ← octets[offset + 1]
        taille_instruction ← 2
    sinon :
        // Valeur 16 bits (little-endian)
        valeur ← octets[offset + 1] | (octets[offset + 2] << 8)
        taille_instruction ← 3
    
    retourner InstructionDécodée{
        mnémonique: "mov",
        destination: destination,
        source: valeur,
        taille_totale: taille_instruction
    }
```

## Partie 6 : Gestion des Valeurs Signées

### 6.1 Le Problème de l'Interprétation

```pseudocode
Réalité_CPU : "Le CPU ne fait pas de distinction signé/non-signé
               lors du stockage. -12 et 65524 sont les mêmes bits
               en 16 bits : 0xFFF4"

Fonction InterpréterValeur(bits, taille, comme_signé) :
    si NON comme_signé :
        retourner bits
    
    si taille = 8 ET bits >= 128 :
        retourner bits - 256
    sinon si taille = 16 ET bits >= 32768 :
        retourner bits - 65536
    sinon :
        retourner bits
```

### 6.2 Extension de Signe pour les Déplacements

```pseudocode
Fonction ExtensionSigne8Vers16(valeur_8bits) :
    si valeur_8bits >= 128 :
        // Bit de signe est 1, étendre avec des 1
        retourner 0xFF00 | valeur_8bits
    sinon :
        // Bit de signe est 0, étendre avec des 0
        retourner valeur_8bits
```

## Partie 7 : Désassembleur Multi-Opcodes

### 7.1 Architecture du Désassembleur Étendu

```pseudocode
Classe Désassembleur8086Étendu :
    
    Méthode DécoderInstruction(octets, offset) :
        premier_octet ← octets[offset]
        
        // Identifier le type d'instruction par pattern matching
        si (premier_octet >> 2) = 0b100010 :
            retourner DécoderMOVRegMem(octets, offset)
        sinon si (premier_octet >> 4) = 0b1011 :
            retourner DécoderMOVImmédiat(octets, offset)
        // Ajouter d'autres opcodes ici pour le défi
        sinon :
            retourner ERREUR(f"Opcode inconnu: {premier_octet:02X}")
    
    Méthode FormaterInstruction(instr) :
        si instr.source est entier :
            // Formatage pour valeur immédiate
            retourner f"{instr.mnémonique} {instr.destination}, {instr.source}"
        sinon :
            // Formatage normal
            retourner f"{instr.mnémonique} {instr.destination}, {instr.source}"
```

### 7.2 Gestion des Mots-Clés de Taille

Pour certaines instructions, l'assembleur a besoin d'indices explicites :

```assembly
mov [bx], byte 7     ; Écrit 1 octet
mov [bx], word 347   ; Écrit 2 octets
```

```pseudocode
Fonction DéterminerBesoinMotClé(instruction) :
    // Nécessaire quand la taille n'est pas déductible du contexte
    si instruction.destination est mémoire ET 
       instruction.source est immédiate :
        si instruction.taille = 8 :
            retourner "byte"
        sinon :
            retourner "word"
    
    retourner null  // Pas besoin de mot-clé
```

## Partie 8 : Défis Supplémentaires (Listing 40)

### 8.1 Instructions Spéciales pour l'Accumulateur

Le 8086 a des encodages optimisés pour AX :

```pseudocode
Opcodes_Spéciaux_AX = {
    0xA0 : "MOV AL, [adresse]",   // Mémoire vers AL
    0xA1 : "MOV AX, [adresse]",   // Mémoire vers AX
    0xA2 : "MOV [adresse], AL",   // AL vers mémoire
    0xA3 : "MOV [adresse], AX"    // AX vers mémoire
}

// Ces instructions sont plus courtes que l'encodage général !
```

### 8.2 Algorithme pour le Défi Complet

```pseudocode
Fonction DésassembleurComplet(octets, offset) :
    opcode ← octets[offset]
    
    // Vérifier tous les opcodes possibles
    si (opcode >> 2) = 0b100010 :
        retourner DécoderMOVRegMem(octets, offset)
    sinon si (opcode >> 4) = 0b1011 :
        retourner DécoderMOVImmédiat(octets, offset)
    sinon si (opcode >> 1) = 0b1100011 :
        retourner DécoderMOVImmédiatMém(octets, offset)
    sinon si opcode >= 0xA0 ET opcode <= 0xA3 :
        retourner DécoderMOVAccumulateur(octets, offset)
    sinon :
        retourner ERREUR("Opcode non supporté")
```

## Partie 9 : Tests et Validation

### 9.1 Stratégie de Test Complète

```pseudocode
Algorithme : TestDésassembleurÉtendu
Pour chaque fichier_test dans [listing39, listing40] :
    1. Désassembler le binaire
    2. Sauvegarder le résultat
    3. Réassembler le résultat
    4. Comparer avec le binaire original
    5. Vérifier l'identité bit à bit
```

### 9.2 Cas de Test Critiques

```assembly
; Tests de régression importants
mov ax, [bx + si]        ; Basique
mov [bp + di - 37], ax   ; Déplacement négatif
mov bp, [5]              ; Adresse directe (piège MOD=00 R/M=110)
mov [bp], byte 7         ; Mot-clé de taille requis
mov ax, [2555]           ; Encodage spécial accumulateur
```

## Partie 10 : Réflexions sur la Complexité x86

### 10.1 Contraintes pour les Compilateurs

```pseudocode
Défis_Génération_Code = {
    "Registres limités pour l'adressage (BX, BP, SI, DI)",
    "Accès byte uniquement sur AX, BX, CX, DX",
    "Patterns d'adresse limités",
    "Encodages multiples pour la même opération",
    "Optimisation de taille vs performance"
}
```

### 10.2 Évolution vers x86-64

```pseudocode
Améliorations_Modernes = {
    "Plus de registres (16 au lieu de 8)",
    "Tous les registres peuvent être utilisés pour l'adressage",
    "Modes d'adressage plus flexibles",
    "Mais... compatibilité arrière maintenue !"
}
```

## Conclusion

Cette extension de notre désassembleur révèle la complexité réelle de l'architecture x86. Avec seulement deux opcodes MOV, nous avons découvert :

1. **Instructions de longueur variable** : 1 à 4+ octets selon les opérandes
2. **Encodages contextuels** : La signification des bits change selon d'autres bits
3. **Cas spéciaux** : MOD=00 R/M=110 pour l'adressage direct
4. **Optimisations historiques** : Instructions spéciales pour AX
5. **Contraintes architecturales** : Seuls certains registres pour l'adressage

### Compétences Acquises

- Décodage d'instructions complexes avec opérandes multiples
- Gestion des accès mémoire et calcul d'adresse effective
- Compréhension des compromis de conception CPU
- Appréciation des défis de la rétrocompatibilité

### Prochaines Étapes

Dans le prochain cours, nous passerons de la décodage à l'**exécution**. Nous allons :
- Implémenter la logique d'exécution des instructions
- Simuler les registres et la mémoire
- Tracer l'exécution de programmes complets
- Observer le comportement réel du CPU

Le voyage de la compréhension profonde des processeurs continue !
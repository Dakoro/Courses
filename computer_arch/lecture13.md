# Architecture des Processeurs Pipelined : Des Aléas à la Haute Performance

## Introduction

L'évolution de l'architecture des ordinateurs est une quête incessante de performance. Au cœur de cette quête se trouve le concept de **parallélisme au niveau de l'instruction** (Instruction-Level Parallelism, ILP), qui vise à exécuter plusieurs instructions simultanément. Le pipelining est la technique fondamentale pour exploiter l'ILP. 

Historiquement, les processeurs ont évolué de conceptions à cycle unique, où une instruction entière s'exécute en un seul cycle d'horloge long, à des conceptions multi-cycles, qui décomposent l'instruction en étapes plus courtes. Le pipelining pousse cette logique à sa conclusion naturelle : il superpose l'exécution de plusieurs instructions, chacune se trouvant à une étape différente du processus d'exécution, à la manière d'une chaîne de montage industrielle.

Cette approche promet une augmentation spectaculaire du débit, visant théoriquement à achever une instruction à chaque cycle d'horloge. Cependant, ce potentiel de performance est entravé par des conditions problématiques appelées **aléas** (hazards). Ces aléas sont des situations qui empêchent l'instruction suivante de s'exécuter durant son cycle d'horloge désigné. Ils représentent les défis centraux que tout architecte de processeur doit surmonter.

On distingue trois types d'aléas :

- **Les aléas structurels** : Ils surviennent lorsque deux instructions ou plus ont besoin de la même ressource matérielle au même moment.
- **Les aléas de données** : Ils apparaissent lorsqu'une instruction dépend du résultat d'une instruction précédente qui est encore en cours d'exécution dans le pipeline.
- **Les aléas de contrôle** : Ils sont causés par les instructions de branchement, qui modifient le flux séquentiel du programme et empêchent le processeur de connaître à l'avance la prochaine instruction à exécuter.

Ce cours se propose de disséquer systématiquement chacun de ces aléas. Nous explorerons les solutions matérielles et logicielles conçues pour les atténuer ou les résoudre, en nous appuyant sur des travaux académiques fondateurs et des implémentations pratiques en C et Python.

Le passage d'une architecture à cycle unique à une architecture pipelinée n'est pas un simple détail d'implémentation ; il incarne un changement fondamental de philosophie de conception. On passe de l'optimisation de la latence (rendre une seule instruction rapide) à l'optimisation du débit (terminer de nombreuses instructions rapidement). 

Un processeur à cycle unique est limité par son instruction la plus lente, ce qui est inefficace pour les instructions plus rapides. Le pipelining brise cette contrainte en liant la période d'horloge à l'étape la plus lente, et non à l'instruction la plus lente. Bien que la latence d'une instruction individuelle augmente légèrement à cause de la surcharge des registres de pipeline, le débit théorique atteint une instruction par cycle ($IPC=1$).

Ce compromis — sacrifier une faible latence pour un débit élevé — est un thème récurrent en architecture des ordinateurs. La solution elle-même, le pipelining, engendre de nouveaux problèmes complexes, les aléas, qui n'existaient pas dans les modèles plus simples.

## Partie 1 : Principes Fondamentaux du Pipeline MIPS à 5 Étages

Pour comprendre les défis du pipelining, nous utiliserons comme modèle le pipeline classique à 5 étages de l'architecture MIPS, une base pour de nombreux concepts d'architecture modernes. Chaque instruction traverse séquentiellement ces cinq étages :

### 1. Instruction Fetch (IF) - Récupération d'Instruction

La première étape consiste à récupérer l'instruction depuis la mémoire. Le compteur de programme (PC) contient l'adresse de l'instruction à aller chercher. Une fois l'instruction récupérée, le PC est incrémenté (généralement de 4 octets dans MIPS) pour pointer vers l'instruction suivante.

### 2. Instruction Decode (ID) - Décodage et Lecture des Registres

L'instruction récupérée est décodée pour déterminer l'opération à effectuer (par exemple, addition, chargement). Simultanément, les opérandes sources sont lus depuis le banc de registres. Pour les instructions avec un opérande immédiat, celui-ci est également extrait et étendu en signe à ce stade.

### 3. Execute (EX) - Exécution ou Calcul d'Adresse

C'est ici que l'opération arithmétique ou logique est effectuée par l'Unité Arithmétique et Logique (UAL, ou ALU en anglais). Pour une instruction arithmétique (comme `add`), l'ALU calcule le résultat. Pour une instruction de chargement ou de stockage (`lw`, `sw`), l'ALU calcule l'adresse mémoire effective en additionnant le registre de base et le déplacement.

### 4. Memory Access (MEM) - Accès Mémoire

Cette étape n'est active que pour les instructions de chargement et de stockage. Une instruction `lw` lit les données depuis la mémoire à l'adresse calculée à l'étape EX. Une instruction `sw` écrit les données dans la mémoire à cette même adresse. Les autres instructions, comme les instructions arithmétiques, traversent cette étape sans rien faire.

### 5. Write-Back (WB) - Écriture en Retour

La dernière étape consiste à écrire le résultat de l'opération dans le banc de registres. Pour une instruction arithmétique ou une instruction de chargement, le résultat (provenant de l'ALU ou de la mémoire) est écrit dans le registre de destination spécifié par l'instruction.

Pour que ces étapes fonctionnent en parallèle, des registres de pipeline (IF/ID, ID/EX, EX/MEM, MEM/WB) sont insérés entre chaque étage. Ces registres mémorisent les résultats et les signaux de contrôle d'une étape pour les transmettre à la suivante au cycle d'horloge suivant, isolant ainsi les étapes les unes des autres.

### Performance du Pipeline

La performance d'un processeur est quantifiée par son temps d'exécution pour un programme donné. La formule fondamentale, souvent appelée "l'équation de fer" de la performance du processeur, est :

$$\text{Temps}_{\text{exec}} = N_{\text{instr}} \times \text{CPI}_{\text{moyen}} \times T_{\text{cycle}}$$

Où :
- $N_{\text{instr}}$ est le nombre total d'instructions dans le programme
- $\text{CPI}_{\text{moyen}}$ est le nombre moyen de cycles par instruction  
- $T_{\text{cycle}}$ est la durée d'un cycle d'horloge

Le pipelining cherche à optimiser cette équation en s'attaquant à la fois au CPI et au temps de cycle. Dans un pipeline idéal, une nouvelle instruction termine son exécution à chaque cycle d'horloge, ce qui donne un CPI idéal de 1. De plus, comme chaque étape est plus simple que l'exécution d'une instruction complète, le temps de cycle $T_{\text{cycle}}$ peut être considérablement réduit par rapport à une conception à cycle unique.

Le speedup (accélération) du pipeline par rapport à une version non pipelinée est donné par :

$$\text{Speedup}_{\text{pipeline}} = \frac{T_{\text{non-pipeline}}}{T_{\text{pipeline}}} = \frac{N_{\text{instr}} \times \text{CPI}_{\text{non-pipeline}} \times T_{\text{cycle,non-pipeline}}}{N_{\text{instr}} \times \text{CPI}_{\text{pipeline}} \times T_{\text{cycle,pipeline}}}$$

Dans des conditions idéales, où les étages sont parfaitement équilibrés et sans aléas, $T_{\text{cycle,non-pipeline}} \approx k \times T_{\text{cycle,pipeline}}$ (où $k$ est le nombre d'étages) et $\text{CPI}_{\text{pipeline}} = 1$. Le speedup idéal devient alors approximativement égal au nombre d'étages, $k$. Pour notre pipeline à 5 étages, le speedup idéal est de 5.

Cependant, le CPI idéal de 1 est une limite théorique rarement atteinte en pratique. Les aléas forcent le pipeline à s'arrêter, introduisant des cycles de blocage (stalls) qui sont essentiellement des cycles perdus. Chaque cycle de blocage augmente le CPI moyen réel du programme. La performance réelle est mieux modélisée par :

$$\text{CPI}_{\text{réel}} = \text{CPI}_{\text{idéal}} + \text{CPI}_{\text{blocages}} = 1 + \text{CPI}_{\text{blocages}}$$

Où $\text{CPI}_{\text{blocages}}$ est le nombre moyen de cycles de blocage par instruction. Cette reformulation change la perspective de l'architecte : le but n'est plus simplement de "construire un pipeline", mais de minimiser $\text{CPI}_{\text{blocages}}$.

## Partie 2 : Gestion des Aléas de Données (Data Hazards)

Un aléa de données se produit lorsqu'une instruction a besoin d'accéder à des données qui n'ont pas encore été mises à jour par une instruction précédente encore dans le pipeline. C'est le type d'aléa le plus courant.

### 2.1. Classification des Dépendances de Données

Il est crucial de classer les dépendances de données, car cette classification dicte les solutions possibles. Il en existe trois types :

#### Read-After-Write (RAW) / Dépendance de Flux

C'est la dépendance la plus fondamentale et la plus "vraie". Une instruction (J) essaie de lire un opérande avant qu'une instruction précédente (I) ne l'ait écrit. C'est une dépendance de flux de données car la valeur produite par I "coule" vers J.

**Exemple :**
```assembly
add $s0, $t0, $t1  ; I: écrit dans $s0
sub $t2, $s0, $t3  ; J: lit $s0
```

Dans un pipeline simple, l'instruction `sub` atteindra l'étape ID (lecture des registres) bien avant que l'instruction `add` n'atteigne l'étape WB (écriture du résultat). L'instruction `sub` lira donc une ancienne valeur de `$s0`, conduisant à un résultat incorrect.

#### Write-After-Read (WAR) / Anti-dépendance

Une instruction (J) essaie d'écrire dans un registre de destination avant qu'une instruction précédente (I) n'ait fini de le lire comme source.

**Exemple :**
```assembly
sub $t2, $s0, $t3  ; I: lit $s0
add $s0, $t0, $t1  ; J: écrit dans $s0
```

Dans notre pipeline MIPS à 5 étages, ce n'est pas un problème. L'instruction I lit ses opérandes à l'étape ID, tandis que l'instruction J écrit son résultat beaucoup plus tard, à l'étape WB. L'ordre est naturellement préservé.

#### Write-After-Write (WAW) / Dépendance de Sortie

Deux instructions (I et J, avec I avant J) écrivent dans le même registre de destination.

**Exemple :**
```assembly
add $s0, $t0, $t1  ; I: écrit dans $s0
sub $s0, $t4, $t5  ; J: écrit dans $s0
```

Ici aussi, dans un pipeline simple où les instructions traversent les étapes dans l'ordre, ce n'est pas un problème. L'instruction I atteindra l'étape WB avant l'instruction J, donc l'écriture de J écrasera celle de I, préservant ainsi la sémantique du programme.

La distinction entre les dépendances "vraies" (RAW) et "fausses" (WAR, WAW) est essentielle. Les dépendances vraies concernent le flux de valeurs de données, tandis que les dépendances fausses sont des conflits de ressources liés au nommage des emplacements de stockage (les registres).

### 2.2. Solution 1 : Le Blocage (Stalling)

La solution la plus simple pour gérer un aléa de données RAW est de forcer l'instruction dépendante à attendre. C'est ce qu'on appelle un blocage (stall). Lorsqu'une unité de détection d'aléa dans l'étage ID identifie une dépendance RAW, elle prend deux mesures :

1. Elle gèle le pipeline en amont, c'est-à-dire qu'elle empêche le PC et le registre de pipeline IF/ID d'être mis à jour. L'instruction dans l'étage IF et l'instruction fautive dans l'étage ID sont ainsi maintenues en place.

2. Elle insère une "bulle" (bubble) dans le pipeline en aval. Une bulle est l'équivalent matériel d'une instruction NOP (No-Operation). Elle est injectée dans le registre de pipeline ID/EX, et se propage à travers les étages EX, MEM et WB, sans effectuer aucune action ni modifier l'état du processeur.

Cette bulle crée le délai nécessaire pour que l'instruction productrice de données puisse avancer dans le pipeline et que son résultat devienne disponible. Bien que simple, cette solution a un coût direct sur la performance : chaque cycle de blocage augmente le CPI du programme.

### 2.3. Solution 2 : Le Court-Circuitage (Data Forwarding)

Une solution matérielle beaucoup plus performante que le blocage est le court-circuitage (forwarding), aussi appelé bypass. L'idée est d'anticiper la disponibilité d'un résultat. Au lieu d'attendre qu'une valeur soit écrite dans le banc de registres à l'étape WB, on la transmet directement de l'endroit où elle est produite à l'endroit où elle est nécessaire.

Dans notre pipeline à 5 étages, un résultat peut être produit à deux endroits clés :
- À la sortie de l'ALU, à la fin de l'étape EX
- À la sortie de la mémoire de données, à la fin de l'étape MEM

Ce résultat peut être nécessaire à l'entrée de l'ALU, au début de l'étape EX de l'instruction suivante. Pour permettre cela, on ajoute des multiplexeurs aux entrées de l'ALU.

#### Chemins de Court-Circuitage et Logique de l'Unité de Forwarding

Les deux principaux chemins de court-circuitage sont :

1. **Aléa EX/MEM** : Du registre de pipeline EX/MEM vers l'entrée de l'ALU. Cela se produit lorsqu'une instruction a besoin du résultat de l'instruction qui la précède immédiatement.

2. **Aléa MEM/WB** : Du registre de pipeline MEM/WB vers l'entrée de l'ALU. Cela se produit lorsqu'une instruction a besoin du résultat d'une instruction qui la précède de deux cycles.

**Logique de l'unité de court-circuitage :**

| Priorité | Source de l'Aléa | Condition (pour l'opérande source rs de l'ALU) | Action (Signal pour le Mux ForwardA) |
|----------|------------------|-----------------------------------------------|-------------------------------------|
| 1 (Haute) | Étage EX/MEM | `EX/MEM.RegWrite AND (EX/MEM.RegisterRd ≠ 0) AND (EX/MEM.RegisterRd == ID/EX.RegisterRs)` | ForwardA = 10 (binaire, sélectionne la sortie de l'ALU) |
| 2 (Basse) | Étage MEM/WB | `MEM/WB.RegWrite AND (MEM/WB.RegisterRd ≠ 0) AND (MEM/WB.RegisterRd == ID/EX.RegisterRs) AND NOT(Aléa EX/MEM pour Rs)` | ForwardA = 01 (binaire, sélectionne la sortie du registre MEM/WB) |
| - (Défaut) | Aucun / Banc de registre | else | ForwardA = 00 (binaire, sélectionne la sortie du banc de registres) |

#### Implémentation : Logique de Court-Circuitage

**Pseudocode :**

```pseudocode
// Logique pour l'opérande A de l'ALU (ForwardA)
if (EX/MEM.RegWrite et EX/MEM.RegisterRd ≠ 0 et EX/MEM.RegisterRd == ID/EX.RegisterRs) alors
    ForwardA = "depuis_EX/MEM" // Priorité 1
sinon si (MEM/WB.RegWrite et MEM/WB.RegisterRd ≠ 0 et MEM/WB.RegisterRd == ID/EX.RegisterRs) alors
    ForwardA = "depuis_MEM/WB" // Priorité 2
sinon
    ForwardA = "depuis_ID" // Pas de court-circuitage
fin si

// Logique pour l'opérande B de l'ALU (ForwardB)
if (EX/MEM.RegWrite et EX/MEM.RegisterRd ≠ 0 et EX/MEM.RegisterRd == ID/EX.RegisterRt) alors
    ForwardB = "depuis_EX/MEM" // Priorité 1
sinon si (MEM/WB.RegWrite et MEM/WB.RegisterRd ≠ 0 et MEM/WB.RegisterRd == ID/EX.RegisterRt) alors
    ForwardB = "depuis_MEM/WB" // Priorité 2
sinon
    ForwardB = "depuis_ID" // Pas de court-circuitage
fin si
```

**Implémentation en C :**

```c
#include <stdio.h>
#include <stdint.h>

// Simule les champs pertinents des registres de pipeline
typedef struct {
    uint8_t RegWrite; // Signal de contrôle
    uint8_t RegisterRd; // Registre de destination
} EX_MEM_Reg;

typedef struct {
    uint8_t RegWrite; // Signal de contrôle
    uint8_t RegisterRd; // Registre de destination
} MEM_WB_Reg;

typedef struct {
    uint8_t RegisterRs; // Source 1
    uint8_t RegisterRt; // Source 2
} ID_EX_Reg;

// Unité de court-circuitage
void forwarding_unit(
    EX_MEM_Reg ex_mem,
    MEM_WB_Reg mem_wb,
    ID_EX_Reg id_ex,
    int* forwardA,
    int* forwardB
) {
    *forwardA = 0; // 00: Pas de court-circuitage
    *forwardB = 0; // 00: Pas de court-circuitage

    // Aléa EX/MEM a la priorité la plus élevée
    // Pour l'opérande A (source Rs)
    if (ex_mem.RegWrite && (ex_mem.RegisterRd != 0) && (ex_mem.RegisterRd == id_ex.RegisterRs)) {
        *forwardA = 2; // 10: Court-circuiter depuis EX/MEM
    }
    // Pour l'opérande B (source Rt)
    if (ex_mem.RegWrite && (ex_mem.RegisterRd != 0) && (ex_mem.RegisterRd == id_ex.RegisterRt)) {
        *forwardB = 2; // 10: Court-circuiter depuis EX/MEM
    }

    // Aléa MEM/WB a une priorité plus faible
    // Pour l'opérande A (source Rs)
    if (*forwardA == 0) { // Vérifier seulement si aucun aléa EX/MEM n'a été trouvé
        if (mem_wb.RegWrite && (mem_wb.RegisterRd != 0) && (mem_wb.RegisterRd == id_ex.RegisterRs)) {
            *forwardA = 1; // 01: Court-circuiter depuis MEM/WB
        }
    }
    // Pour l'opérande B (source Rt)
    if (*forwardB == 0) { // Vérifier seulement si aucun aléa EX/MEM n'a été trouvé
        if (mem_wb.RegWrite && (mem_wb.RegisterRd != 0) && (mem_wb.RegisterRd == id_ex.RegisterRt)) {
            *forwardB = 1; // 01: Court-circuiter depuis MEM/WB
        }
    }
}

int main() {
    // Exemple de scénario : add $3, $1, $2 suivi de sub $5, $4, $3
    // Au cycle où 'sub' est dans EX, 'add' est dans MEM.
    EX_MEM_Reg ex_mem_reg = {1, 3}; // add écrit dans $3
    MEM_WB_Reg mem_wb_reg = {0, 0}; // Instruction précédente
    ID_EX_Reg id_ex_reg = {4, 3};   // sub lit $4 et $3

    int fwdA, fwdB;
    forwarding_unit(ex_mem_reg, mem_wb_reg, id_ex_reg, &fwdA, &fwdB);

    printf("ForwardA: %d, ForwardB: %d\n", fwdA, fwdB); // Devrait afficher ForwardA: 0, ForwardB: 2
    return 0;
}
```

**Implémentation en Python :**

```python
def forwarding_unit(ex_mem_reg, mem_wb_reg, id_ex_reg):
    """
    Simule l'unité de court-circuitage.
    Les registres sont des dictionnaires pour plus de clarté.
    Retourne les signaux de contrôle pour les multiplexeurs ForwardA et ForwardB.
    """
    forward_A = "00"  # 00: Pas de court-circuitage
    forward_B = "00"

    # Priorité 1: Aléa EX/MEM
    if ex_mem_reg and ex_mem_reg.get('RegisterRd') != 0:
        if ex_mem_reg.get('RegisterRd') == id_ex_reg.get('RegisterRs'):
            forward_A = "10"  # Court-circuiter depuis la sortie de l'ALU (EX/MEM)
        if ex_mem_reg.get('RegisterRd') == id_ex_reg.get('RegisterRt'):
            forward_B = "10"

    # Priorité 2: Aléa MEM/WB
    if mem_wb_reg and mem_wb_reg.get('RegisterRd') != 0:
        # Pour l'opérande A (Rs)
        if forward_A == "00" and mem_wb_reg.get('RegisterRd') == id_ex_reg.get('RegisterRs'):
            forward_A = "01"  # Court-circuiter depuis le registre MEM/WB
        # Pour l'opérande B (Rt)
        if forward_B == "00" and mem_wb_reg.get('RegisterRd') == id_ex_reg.get('RegisterRt'):
            forward_B = "01"

    return forward_A, forward_B

# Exemple de scénario
ex_mem_reg_for_and = {'RegWrite': True, 'RegisterRd': 2}  # sub $2,... est dans MEM
mem_wb_reg_for_and = {'RegWrite': True, 'RegisterRd': 1}  # add $1,... est dans WB
id_ex_reg_for_and = {'RegisterRs': 2, 'RegisterRt': 4}    # and $4, $2,...

fA, fB = forwarding_unit(ex_mem_reg_for_and, mem_wb_reg_for_and, id_ex_reg_for_and)
print(f"Pour 'and $4, $2, $4': ForwardA={fA}, ForwardB={fB}")  # Devrait être 10, 00
```

### 2.4. Cas Spécifique : L'Aléa de Chargement-Utilisation (Load-Use Hazard)

Malheureusement, le court-circuitage seul ne peut pas résoudre tous les aléas de données. Il existe un cas critique : l'aléa de chargement-utilisation (load-use hazard).

Considérons la séquence :
```assembly
lw  $s0, 0($t1)
add $t2, $s0, $t3
```

L'instruction `lw` lit la donnée depuis la mémoire à l'étape MEM. Le résultat n'est donc disponible qu'à la fin du cycle d'horloge où `lw` est dans l'étape MEM. Au même moment, l'instruction `add` est dans l'étape EX et a besoin de la valeur de `$s0` au début de ce cycle pour son calcul ALU. Le court-circuitage depuis l'étage MEM vers l'étage EX est possible, mais il arrive un cycle trop tard.

La seule solution est une combinaison de matériel :
1. Une unité de détection d'aléa identifie spécifiquement cette dépendance
2. Lorsque cet aléa est détecté, l'unité de détection force un blocage d'un cycle
3. Après ce cycle de blocage, l'instruction `lw` a progressé à l'étage WB, et l'instruction `add` est maintenant dans l'étage EX

#### Implémentation : Logique de Détection de Chargement-Utilisation

**Pseudocode :**

```pseudocode
// Logique dans l'étage ID pour détecter un aléa de chargement-utilisation
if (ID/EX.MemRead == 1 et // L'instruction dans EX est un chargement
    (ID/EX.RegisterRt == IF/ID.RegisterRs ou // et sa destination est une source
     ID/EX.RegisterRt == IF/ID.RegisterRt)) // pour l'instruction dans ID
alors
    // ALÉA DÉTECTÉ :
    // 1. Geler le PC et le registre IF/ID.
    // 2. Forcer les signaux de contrôle dans ID/EX à 0 (créer une bulle).
    STALL = 1
sinon
    STALL = 0
fin si
```

**Implémentation en C :**

```c
#include <stdio.h>
#include <stdint.h>

// Simule les champs pertinents des registres de pipeline
typedef struct {
    uint8_t MemRead;    // Signal de contrôle pour la lecture mémoire
    uint8_t RegisterRt; // Destination pour lw
} ID_EX_Reg_Hazard;

typedef struct {
    uint8_t RegisterRs; // Source 1
    uint8_t RegisterRt; // Source 2
} IF_ID_Reg_Hazard;

// Unité de détection d'aléa pour chargement-utilisation
int hazard_detection_unit(ID_EX_Reg_Hazard id_ex, IF_ID_Reg_Hazard if_id) {
    if (id_ex.MemRead && 
        ((id_ex.RegisterRt == if_id.RegisterRs) || (id_ex.RegisterRt == if_id.RegisterRt))) {
        return 1; // Blocage nécessaire
    }
    return 0; // Pas de blocage
}

int main() {
    // Scénario : lw $2, 0($1) suivi de and $4, $2, $5
    // 'and' est dans ID, 'lw' est dans EX.
    ID_EX_Reg_Hazard id_ex_reg = {1, 2}; // lw $2,... est dans EX
    IF_ID_Reg_Hazard if_id_reg = {2, 5}; // and $4, $2, $5 est dans ID

    if (hazard_detection_unit(id_ex_reg, if_id_reg)) {
        printf("Aléa de chargement-utilisation détecté. Blocage du pipeline.\n");
    } else {
        printf("Aucun aléa de chargement-utilisation.\n");
    }
    
    return 0;
}
```

**Implémentation en Python :**

```python
def hazard_detection_unit(id_ex_reg, if_id_reg):
    """
    Détecte un aléa de chargement-utilisation.
    id_ex_reg: Dictionnaire représentant l'état du registre ID/EX.
    if_id_reg: Dictionnaire représentant l'état du registre IF/ID.
    """
    # Vérifie si l'instruction dans l'étage EX est une instruction de chargement (MemRead=1)
    is_load_in_ex = id_ex_reg.get('MemRead', False)
    
    if is_load_in_ex:
        # Récupère le registre de destination de l'instruction de chargement
        load_dest_reg = id_ex_reg.get('RegisterRt')
        
        # Récupère les registres sources de l'instruction dans l'étage ID
        instr_src_rs = if_id_reg.get('RegisterRs')
        instr_src_rt = if_id_reg.get('RegisterRt')
        
        # Vérifie s'il y a une dépendance
        if (load_dest_reg == instr_src_rs) or (load_dest_reg == instr_src_rt):
            return True  # Blocage requis
            
    return False # Pas de blocage

# Scénario : lw $s0, 0($t1) suivi de add $t2, $s0, $t3
id_ex_state = {'MemRead': True, 'RegisterRt': 16}  # lw $s0 (registre 16)
if_id_state = {'RegisterRs': 16, 'RegisterRt': 19} # add $t2, $s0, $t3

if hazard_detection_unit(id_ex_state, if_id_state):
    print("Aléa de chargement-utilisation détecté. Le pipeline doit être bloqué.")
else:
    print("Aucun aléa de chargement-utilisation.")
```

## Partie 3 : Maîtrise des Aléas de Contrôle (Control Hazards)

Les aléas de contrôle, ou aléas de branchement, surviennent lorsqu'une instruction de branchement (comme `beq` ou `bne`) modifie le flux d'exécution normal du programme. Le pipeline, conçu pour un flux séquentiel (PC, PC+4, PC+8,...), se trouve face à une incertitude : quelle est la prochaine instruction à récupérer?

### 3.1. Le Problème des Branchements

Dans notre pipeline à 5 étages, la décision d'un branchement (s'il est pris ou non) et son adresse cible ne sont connues qu'à la fin de l'étape EX. À ce moment, le processeur a déjà récupéré et commencé à traiter les trois instructions suivantes (celles à PC+4, PC+8, et PC+12) en supposant que le branchement ne serait pas pris.

Si le branchement est finalement pris, ces trois instructions sont incorrectes et doivent être annulées. Cette annulation est appelée un **vidage du pipeline** (pipeline flush). Les cycles perdus à cause de ce vidage constituent la **pénalité de mauvaise prédiction de branchement**. Dans ce cas, la pénalité est de 3 cycles.

### 3.2. Solution 1 : Résolution Précoce et Prédiction Statique

Pour réduire cette pénalité coûteuse, une solution consiste à déterminer le résultat du branchement plus tôt dans le pipeline. En déplaçant la logique de comparaison des registres et le calcul de l'adresse cible de l'étage EX vers l'étage ID, la décision peut être prise à la fin de l'étape ID.

Avec cette résolution précoce, seule une instruction (celle à PC+4) a été récupérée de manière spéculative avant que la décision ne soit prise. Si le branchement est pris, seule cette instruction doit être vidée, réduisant la pénalité à 1 cycle.

Cette approche est généralement associée à une **prédiction statique simple**, la plus courante étant de toujours prédire non-pris (predict-not-taken). Le matériel suppose que le branchement ne sera pas pris et continue de récupérer les instructions séquentiellement.

Le compromis est une complexité matérielle accrue dans l'étage ID. L'ajout d'un comparateur et d'un additionneur peut allonger le chemin critique de cet étage, ce qui pourrait potentiellement augmenter la durée du cycle d'horloge global du processeur.

### 3.3. Solution 2 : Prédiction de Branchement Dynamique

Les prédictions statiques sont limitées, en particulier pour les boucles où les branchements sont pris la plupart du temps. La prédiction de branchement dynamique offre une solution beaucoup plus performante. Le matériel apprend du comportement passé des branchements pour prédire leur comportement futur.

#### Référence Académique : Yeh & Patt et la Prédiction à Deux Niveaux

Un tournant majeur dans ce domaine a été l'article "Two-Level Adaptive Branch Prediction" de Tse-Yu Yeh et Yale N. Patt. Ils ont démontré que l'utilisation de deux niveaux d'historique — par exemple, l'historique des derniers branchements globaux et l'historique des motifs pour chaque branchement individuel — pouvait atteindre des précisions de prédiction supérieures à 97 %, une amélioration spectaculaire par rapport aux schémas existants.

L'évolution des prédicteurs illustre un principe clé : les gains de performance sont souvent obtenus en exploitant davantage de contexte. La prédiction statique n'a aucun contexte. La prédiction bimodale ajoute un contexte local (comment cette branche s'est comportée). La prédiction à deux niveaux ajoute un contexte global (comment d'autres branchements récents se sont comportés), permettant d'apprendre des corrélations complexes.

#### Prédicteur Bimodal (Compteur à Saturation à 2 bits)

Le prédicteur bimodal est le prédicteur dynamique le plus simple. Il se compose d'une table d'historique de branchement (Branch History Table, BHT), qui est un tableau de compteurs à saturation de 2 bits. Cette table est indexée par les bits de poids faible de l'adresse du branchement (le PC).

Chaque compteur à 2 bits représente l'état d'un branchement et peut avoir quatre états :
- **11** : Fortement pris (Strongly Taken)
- **10** : Faiblement pris (Weakly Taken)
- **01** : Faiblement non-pris (Weakly Not-Taken)
- **00** : Fortement non-pris (Strongly Not-Taken)

La prédiction est faite en fonction du bit de poids fort du compteur (1 = prédire pris, 0 = prédire non-pris). Après la résolution du branchement, le compteur est mis à jour : il est incrémenté s'il est pris (saturant à 3) et décrémenté s'il n'est pas pris (saturant à 0).

**Implémentation : Prédicteur Bimodal**

**Pseudocode :**

```pseudocode
// Initialisation
BHT = tableau de compteurs_2_bits, initialisé à un état (ex: 10 - Faiblement pris)
taille_BHT = ...

fonction predire(PC):
    index = PC % taille_BHT
    compteur = BHT[index]
    si bit_poids_fort(compteur) == 1:
        retourner PRIS
    sinon:
        retourner NON_PRIS

fonction mettre_a_jour(PC, resultat_reel):
    index = PC % taille_BHT
    si resultat_reel == PRIS:
        BHT[index] = min(BHT[index] + 1, 3) // Incrémenter, saturer à 3 (11)
    sinon:
        BHT[index] = max(BHT[index] - 1, 0) // Décrémenter, saturer à 0 (00)
```

**Implémentation en C++ :**

```cpp
#include <iostream>
#include <vector>
#include <cmath>
#include <cstdint>

// Les états du compteur à 2 bits
enum State {
    STRONG_NOT_TAKEN = 0, // 00
    WEAK_NOT_TAKEN = 1,   // 01
    WEAK_TAKEN = 2,       // 10
    STRONG_TAKEN = 3      // 11
};

class BimodalPredictor {
private:
    std::vector<State> bht; // Branch History Table
    int index_mask;

public:
    BimodalPredictor(int index_bits) {
        int table_size = 1 << index_bits;
        bht.assign(table_size, WEAK_TAKEN); // Initialiser à faiblement pris
        index_mask = table_size - 1;
    }

    bool predict(uint64_t pc) {
        int index = (pc >> 2) & index_mask; // Utiliser les bits d'adresse de l'instruction
        // La prédiction est "pris" si le bit de poids fort est 1 (états 2 ou 3)
        return bht[index] >= WEAK_TAKEN;
    }

    void update(uint64_t pc, bool taken) {
        int index = (pc >> 2) & index_mask;
        State current_state = bht[index];
        if (taken) {
            if (current_state < STRONG_TAKEN) {
                bht[index] = static_cast<State>(current_state + 1);
            }
        } else {
            if (current_state > STRONG_NOT_TAKEN) {
                bht[index] = static_cast<State>(current_state - 1);
            }
        }
    }
};

int main() {
    BimodalPredictor predictor(10); // BHT avec 1024 entrées

    uint64_t branch_pc = 0x400500;
    
    // Simuler une boucle qui s'exécute 10 fois
    for (int i = 0; i < 10; ++i) {
        bool prediction = predictor.predict(branch_pc);
        bool actual_outcome = (i < 9); // Pris pour les 9 premières itérations
        
        std::cout << "Itération " << i << ": Prédiction=" << (prediction ? "Pris" : "Non-Pris")
                  << ", Réel=" << (actual_outcome ? "Pris" : "Non-Pris");
        if (prediction != actual_outcome) {
            std::cout << " (Mauvaise prédiction)";
        }
        std::cout << std::endl;
        
        predictor.update(branch_pc, actual_outcome);
    }
    
    return 0;
}
```

**Implémentation en Python :**

```python
class BimodalPredictor:
    def __init__(self, index_bits):
        table_size = 1 << index_bits
        # Initialiser la table avec l'état "Faiblement Pris" (10 en binaire)
        self.bht = [2] * table_size
        self.index_mask = table_size - 1

    def predict(self, pc):
        """Prédit si le branchement sera pris."""
        index = (pc >> 2) & self.index_mask
        # La prédiction est "pris" si le bit de poids fort est 1 (états 2 ou 3)
        return self.bht[index] >= 2

    def update(self, pc, taken):
        """Met à jour le compteur après la résolution du branchement."""
        index = (pc >> 2) & self.index_mask
        if taken:
            # Incrémenter, mais ne pas dépasser 3 (Fortement Pris)
            self.bht[index] = min(self.bht[index] + 1, 3)
        else:
            # Décrémenter, mais ne pas aller en dessous de 0 (Fortement Non-Pris)
            self.bht[index] = max(self.bht[index] - 1, 0)

# Simulation
predictor = BimodalPredictor(10) # 1024 entrées
branch_pc = 0x400500
mispredictions = 0

print("Simulation d'une boucle (100 itérations):")
for i in range(100):
    prediction = predictor.predict(branch_pc)
    actual = (i < 99) # La boucle est prise sauf à la dernière itération
    
    if prediction != actual:
        mispredictions += 1
        
    predictor.update(branch_pc, actual)

print(f"Nombre total de mauvaises prédictions : {mispredictions}") # Devrait être faible
```

#### Prédicteur Adaptatif à Deux Niveaux (Exemple : Gshare)

Le prédicteur Gshare (Global-share) est une amélioration populaire du concept à deux niveaux. Il utilise un **registre d'historique global** (Global History Register, GHR), qui est un registre à décalage qui mémorise le résultat (pris/non-pris) des N derniers branchements exécutés dans tout le programme.

Pour prédire un branchement, Gshare combine (généralement via une opération XOR) les M bits de poids faible du PC du branchement avec les N bits du GHR. Le résultat de ce XOR est utilisé pour indexer une unique et grande table d'historique de motifs (Pattern History Table, PHT) de compteurs à 2 bits.

**Implémentation : Prédicteur Gshare**

**Pseudocode :**

```pseudocode
// Initialisation
PHT = tableau de compteurs_2_bits, initialisé à un état
taille_PHT = 2^M
GHR = registre à décalage de N bits, initialisé à 0
masque_PC = (1 << M) - 1
masque_GHR = (1 << N) - 1

fonction predire(PC):
    index_pc = (PC >> 2) & masque_PC
    index_ghr = GHR & masque_GHR
    index_final = index_pc XOR index_ghr
    
    compteur = PHT[index_final]
    si bit_poids_fort(compteur) == 1:
        retourner PRIS
    sinon:
        retourner NON_PRIS

fonction mettre_a_jour(PC, resultat_reel):
    // Recalculer l'index utilisé pour la prédiction
    index_pc = (PC >> 2) & masque_PC
    index_ghr = GHR & masque_GHR
    index_final = index_pc XOR index_ghr

    // Mettre à jour le compteur PHT
    si resultat_reel == PRIS:
        PHT[index_final] = min(PHT[index_final] + 1, 3)
    sinon:
        PHT[index_final] = max(PHT[index_final] - 1, 0)
        
    // Mettre à jour le GHR
    GHR = (GHR << 1) | (1 si resultat_reel == PRIS else 0)
```

**Implémentation en C++ :**

```cpp
#include <iostream>
#include <vector>
#include <cstdint>

class GsharePredictor {
private:
    std::vector<State> pht; // Pattern History Table
    uint32_t ghr;           // Global History Register
    int pht_index_mask;
    int ghr_mask;
    int history_bits;

public:
    GsharePredictor(int pht_index_bits, int ghr_bits) {
        int pht_size = 1 << pht_index_bits;
        pht.assign(pht_size, WEAK_TAKEN);
        pht_index_mask = pht_size - 1;
        
        history_bits = ghr_bits;
        ghr = 0;
        ghr_mask = (1 << history_bits) - 1;
    }

    bool predict(uint64_t pc) {
        int pc_index = (pc >> 2) & pht_index_mask;
        int ghr_index_part = ghr & ghr_mask;
        int final_index = pc_index ^ ghr_index_part;
        
        return pht[final_index] >= WEAK_TAKEN;
    }

    void update(uint64_t pc, bool taken) {
        int pc_index = (pc >> 2) & pht_index_mask;
        int ghr_index_part = ghr & ghr_mask;
        int final_index = pc_index ^ ghr_index_part;

        // Mettre à jour le PHT
        State current_state = pht[final_index];
        if (taken) {
            if (current_state < STRONG_TAKEN) {
                pht[final_index] = static_cast<State>(current_state + 1);
            }
        } else {
            if (current_state > STRONG_NOT_TAKEN) {
                pht[final_index] = static_cast<State>(current_state - 1);
            }
        }

        // Mettre à jour le GHR
        ghr = ((ghr << 1) | (taken ? 1 : 0)) & ghr_mask;
    }
};

int main() {
    GsharePredictor predictor(10, 4); // PHT de 1024 entrées, GHR de 4 bits

    // Simuler deux branchements corrélés
    uint64_t pc1 = 0x400600;
    uint64_t pc2 = 0x400800;

    // Scénario : if (x) {... }; if (x) {... }
    // Le résultat du premier branchement influence le second
    for (int i = 0; i < 5; ++i) {
        bool outcome1 = true;
        bool pred1 = predictor.predict(pc1);
        std::cout << "PC1: Pred=" << pred1 << " Actual=" << outcome1 << std::endl;
        predictor.update(pc1, outcome1);

        bool outcome2 = true;
        bool pred2 = predictor.predict(pc2);
        std::cout << "PC2: Pred=" << pred2 << " Actual=" << outcome2 << std::endl;
        predictor.update(pc2, outcome2);
    }
    
    return 0;
}
```

**Implémentation en Python :**

```python
class GsharePredictor:
    def __init__(self, pht_index_bits, ghr_bits):
        pht_size = 1 << pht_index_bits
        self.pht = [2] * pht_size  # Initialiser à Faiblement Pris
        self.pht_mask = pht_size - 1
        
        self.ghr_bits = ghr_bits
        self.ghr_mask = (1 << ghr_bits) - 1
        self.ghr = 0

    def _get_index(self, pc):
        pc_index = (pc >> 2) & self.pht_mask
        ghr_part = self.ghr & self.ghr_mask
        # Le nombre de bits du PC à utiliser pour le XOR dépend de la taille du GHR
        # Pour simplifier, on XOR les bits de poids faible du PC
        pc_part = pc_index & self.ghr_mask
        xor_result = pc_part ^ ghr_part
        # Recombiner avec la partie haute de l'index du PC
        final_index = (pc_index & ~self.ghr_mask) | xor_result
        return final_index

    def predict(self, pc):
        index = self._get_index(pc)
        return self.pht[index] >= 2

    def update(self, pc, taken):
        index = self._get_index(pc)
        
        # Mettre à jour le PHT
        if taken:
            self.pht[index] = min(self.pht[index] + 1, 3)
        else:
            self.pht[index] = max(self.pht[index] - 1, 0)
            
        # Mettre à jour le GHR
        self.ghr = ((self.ghr << 1) | (1 if taken else 0)) & self.ghr_mask

# Simulation
predictor = GsharePredictor(pht_index_bits=12, ghr_bits=8)
pc1 = 0x400A00
pc2 = 0x400B00
mispredictions = 0

# Simuler un motif alterné T,NT,T,NT... pour deux branchements
for i in range(100):
    outcome1 = (i % 2 == 0) # T, F, T, F,...
    pred1 = predictor.predict(pc1)
    if pred1 != outcome1: mispredictions += 1
    predictor.update(pc1, outcome1)
    
    outcome2 = (i % 2 == 0) # Même motif, corrélé
    pred2 = predictor.predict(pc2)
    if pred2 != outcome2: mispredictions += 1
    predictor.update(pc2, outcome2)

print(f"Gshare - Mauvaises prédictions sur motif corrélé : {mispredictions}")
```

## Partie 4 : Techniques Avancées de Tolérance à la Latence : Le Multithreading à Grain Fin

Jusqu'à présent, nous avons exploré des techniques pour résoudre les aléas en attendant (blocage) ou en accélérant la disponibilité des données (court-circuitage, prédiction). Une approche radicalement différente consiste à **tolérer** la latence des aléas. C'est le principe du **multithreading à grain fin** (Fine-Grained Multithreading, FGMT).

L'idée centrale est de faire exécuter par le processeur des instructions provenant de différents threads matériels à chaque cycle d'horloge. Si le pipeline a une profondeur de $k$ étages, et que le processeur dispose d'au moins $k$ threads indépendants prêts à s'exécuter, le matériel peut récupérer une instruction du thread 1 au cycle 1, du thread 2 au cycle 2, et ainsi de suite.

Au cycle $k+1$, lorsque le processeur revient au thread 1 pour récupérer sa prochaine instruction, la première instruction du thread 1 a déjà terminé son parcours dans le pipeline et son résultat est disponible. De ce fait, il n'y a jamais deux instructions du même thread présentes simultanément dans le pipeline. Cela élimine complètement les aléas de données RAW au sein d'un même thread, rendant le court-circuitage et la détection d'aléa inutiles.

Cette approche transforme la question de l'architecte. Pour un CPU classique, la question est : "Comment puis-je obtenir le résultat plus vite pour l'instruction dépendante?". La réponse est le court-circuitage. Pour une architecture FGMT, la question est : "Pendant que j'attends ce résultat, y a-t-il un autre travail utile que je peux faire?". La réponse est : "Oui, exécuter une instruction d'un autre thread."

### Application aux GPU et Compromis Architecturaux

Le FGMT est le modèle d'exécution dominant dans les processeurs graphiques (GPU), où il est souvent appelé **Single Instruction, Multiple Threads** (SIMT). Les GPU sont conçus pour un débit massif sur des charges de travail hautement parallèles (graphiques, calcul scientifique). Ils possèdent des milliers de threads matériels et des pipelines très profonds.

Le FGMT leur permet de cacher efficacement les longues latences, notamment les accès à la mémoire, qui sont fréquents dans leurs charges de travail. Si un groupe de threads (un "warp" dans la terminologie NVIDIA) est bloqué en attendant une lecture mémoire, le planificateur du GPU passe simplement à un autre warp prêt, maintenant ainsi les unités de calcul constamment occupées.

Cependant, cette approche a des coûts et des compromis significatifs :

1. **Performance Mono-thread** : La performance d'un seul thread est considérablement réduite. Une instruction d'un thread donné ne peut être exécutée que tous les $k$ cycles, ce qui augmente sa latence totale.

2. **Coût Matériel** : Le processeur doit dupliquer le contexte de chaque thread, ce qui inclut un compteur de programme (PC) par thread et un banc de registres beaucoup plus grand pour stocker l'état de tous les threads actifs. Les GPU modernes peuvent avoir des centaines de kilo-octets de fichier de registres par cœur de calcul, un coût matériel énorme.

3. **Exigence de Parallélisme** : Le modèle n'est efficace que si le programme peut exposer un nombre massif de threads indépendants. Si le nombre de threads actifs chute, des "bulles" apparaissent dans le pipeline et le débit diminue.

En fin de compte, il n'existe pas d'architecture "meilleure" dans l'absolu. La conception optimale est une fonction de la charge de travail cible. Les CPU sont optimisés pour la latence, utilisant des techniques complexes comme la prédiction de branchement et l'exécution dans le désordre pour accélérer un seul thread. Les GPU sont optimisés pour le débit, acceptant une latence élevée pour un seul thread en échange de l'exécution simultanée de milliers de threads.

## Partie 5 : Assurer la Correction : La Gestion des Exceptions Précises

La performance n'est pas tout ; la correction est primordiale. Dans un pipeline simple, les instructions terminent dans l'ordre du programme. Cependant, dans des processeurs plus complexes avec des unités fonctionnelles à latences multiples (par exemple, une addition en 1 cycle, une division en 20 cycles) ou avec exécution dans le désordre, une instruction peut terminer et écrire son résultat avant une instruction précédente qui a provoqué une exception.

Cela peut laisser la machine dans un **état imprécis**. L'état architectural (le contenu des registres et de la mémoire visible par le programmeur) ne correspond plus à un point unique et séquentiel de l'exécution du programme.

**Exemple :**
```assembly
div $t0, $t1, $zero ; Lève une exception de division par zéro
...
add $s0, $s1, $s2   ; Peut terminer avant la 'div' dans un processeur out-of-order
```

Si l'instruction `add` ci-dessus termine avant que la `div` ne lève son exception, le registre `$s0` sera mis à jour de manière incorrecte par rapport au point de l'erreur. Le débogage et la reprise après exception deviennent alors quasiment impossibles.

### Définition d'une Exception Précise

Une exception est dite **précise** si, lorsque le gestionnaire d'exceptions est invoqué, l'état du processeur est cohérent avec un modèle d'exécution séquentiel. Selon l'article fondateur de James E. Smith et Andrew R. Pleszkun, "Implementing Precise Interrupts in Pipelined Processors", cela signifie que trois conditions doivent être remplies :

1. Toutes les instructions qui précèdent l'instruction fautive dans l'ordre du programme ont terminé leur exécution et ont correctement mis à jour l'état architectural.

2. Toutes les instructions qui suivent l'instruction fautive dans l'ordre du programme sont considérées comme non exécutées et n'ont modifié aucun état architectural.

3. L'instruction fautive elle-même est clairement identifiée (généralement par le PC sauvegardé), et son état est bien défini (soit complètement terminée, soit pas du tout commencée).

La nécessité d'exceptions précises impose une contrainte fondamentale à la recherche de performance par le parallélisme. Elle force l'architecture à maintenir l'illusion d'une exécution séquentielle, même si le matériel sous-jacent exécute les instructions de manière chaotique et parallèle.

### Mécanismes pour les Exceptions Précises

Smith et Pleszkun ont analysé plusieurs mécanismes pour garantir des exceptions précises. Les plus influents, qui sont au cœur des processeurs modernes, sont basés sur l'idée de séparer l'achèvement de l'exécution de la mise à jour de l'état architectural.

#### Le Tampon de Réordonnancement (Reorder Buffer, ROB)

Le ROB est une structure matérielle (une file d'attente circulaire) qui permet aux instructions de s'exécuter et de terminer dans le désordre, mais les force à **valider** (commit) leurs résultats dans l'ordre du programme.

**Fonctionnement :**
- Lorsqu'une instruction est décodée, une entrée lui est allouée dans le ROB
- Lorsqu'elle termine son exécution, son résultat est placé dans son entrée du ROB, mais pas encore dans le banc de registres architectural
- Le ROB valide les instructions séquentiellement depuis sa tête. Ce n'est qu'au moment de la validation que le résultat est écrit dans le registre architectural

**Gestion des Exceptions :**
- Si une instruction provoque une exception, son statut d'exception est noté dans son entrée du ROB
- Lorsque cette instruction atteint la tête du ROB, au lieu de valider son résultat, le processeur déclenche le gestionnaire d'exceptions
- Toutes les instructions suivantes dans le ROB sont simplement vidées
- Comme aucune d'entre elles n'a encore mis à jour l'état architectural, l'état du processeur reste précis

Le ROB agit comme un point de re-sérialisation, un pont entre la microarchitecture parallèle et l'architecture séquentielle.

#### Le Tampon d'Historique (History Buffer)

Une approche alternative consiste à permettre aux instructions de mettre à jour directement le banc de registres. Cependant, avant d'écraser une valeur, l'ancienne valeur du registre de destination est sauvegardée dans un tampon d'historique.

**Gestion des Exceptions :**
- Si une exception se produit, le processeur arrête d'émettre de nouvelles instructions
- Il utilise ensuite le tampon d'historique pour "annuler" les modifications apportées par les instructions qui ont suivi l'instruction fautive, en restaurant les anciennes valeurs des registres
- Ce processus de "retour en arrière" (rollback) ramène le processeur à un état précis

### Distinction entre Exceptions et Interruptions

Bien que gérés de manière similaire, il est utile de distinguer ces deux types d'événements :

- **Exceptions** : Événements synchrones, directement causés par l'exécution d'une instruction (ex: opcode illégal, division par zéro, débordement arithmétique, faute de page mémoire). Elles sont prévisibles et reproductibles.

- **Interruptions** : Événements asynchrones, causés par des événements externes au flux d'instructions (ex: un périphérique d'E/S qui a terminé son travail, une minuterie qui expire, une pression sur le clavier). Elles sont imprévisibles par rapport au programme en cours.

Dans les deux cas, le processeur doit suspendre le programme en cours, sauvegarder son état de manière précise, et sauter vers un gestionnaire de routine pour traiter l'événement. Les mécanismes comme le ROB garantissent que cet état sauvegardé est toujours précis, que l'événement soit synchrone ou asynchrone.

## Synthèse et Conclusion

Ce cours a exploré le monde complexe de l'architecture des processeurs pipelinés, un voyage qui commence par une idée simple et élégante — la chaîne de montage pour les instructions — et qui nous mène rapidement à une série de défis profonds et de solutions ingénieuses. La construction d'un pipeline performant est une histoire de gestion de compromis fondamentaux : performance contre complexité, débit contre latence, et solutions matérielles contre solutions logicielles.

Nous avons vu que le pipelining, en cherchant à optimiser le débit, introduit des aléas qui menacent à la fois la performance et la correction.

**Pour les aléas de données**, nous avons progressé du blocage, une solution simple mais coûteuse, au court-circuitage, une solution matérielle performante qui résout la majorité des dépendances RAW. Nous avons identifié l'aléa de chargement-utilisation comme le cas limite nécessitant une synergie entre blocage et court-circuitage.

**Pour les aléas de contrôle**, nous avons vu comment la résolution précoce des branchements réduit la pénalité de mauvaise prédiction. Nous avons ensuite plongé dans le monde de la prédiction de branchement dynamique, en nous appuyant sur les travaux de Yeh et Patt pour comprendre comment l'exploitation de l'historique local et global permet d'atteindre des niveaux de précision extraordinairement élevés.

Nous avons ensuite changé de perspective avec le **multithreading à grain fin**, une technique qui ne résout pas les aléas mais les tolère en masquant la latence grâce à un parallélisme massif. Cette approche illustre la divergence philosophique entre les CPU, optimisés pour la latence, et les GPU, optimisés pour le débit.

Enfin, nous avons abordé la question critique de la correction avec les **exceptions précises**. En nous référant à Smith et Pleszkun, nous avons compris que des mécanismes comme le tampon de réordonnancement (ROB) sont essentiels pour maintenir l'illusion d'une exécution séquentielle, préservant ainsi le contrat entre le matériel et le logiciel, même au milieu de l'exécution parallèle et dans le désordre.

Les concepts que nous avons abordés — la détection d'aléas, le court-circuitage, la prédiction dynamique, la gestion de l'état spéculatif et la validation dans l'ordre via un ROB — ne sont pas seulement des solutions à des problèmes isolés. Ils sont les blocs de construction fondamentaux des processeurs superscalaires et à exécution dans le désordre qui équipent la quasi-totalité des systèmes informatiques modernes, des smartphones aux supercalculateurs. La compréhension de ces principes est donc essentielle pour quiconque cherche à maîtriser l'art et la science de l'architecture des ordinateurs.
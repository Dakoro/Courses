# Architecture des Microprocesseurs : Du Cycle Unique à la Microprogrammation

## Partie 1 : Introduction et Principes Fondamentaux de la Conception

L'étude de l'architecture des ordinateurs est un voyage au cœur de la machine, une exploration de la manière dont les concepts abstraits du logiciel sont traduits en opérations physiques concrètes. Au centre de ce processus se trouve la microarchitecture, l'implémentation spécifique d'un jeu d'instructions (ISA, Instruction Set Architecture). L'ISA est un contrat, une spécification formelle de ce que le matériel doit faire ; la microarchitecture est la manière dont il le fait. 

Cette leçon se penche sur les choix fondamentaux de conception qui dictent la performance, l'efficacité et la complexité d'un processeur, en se concentrant sur la transition critique d'une approche conceptuellement simple, mais profondément inefficace — l'architecture à cycle unique — vers une approche plus nuancée et performante, l'architecture multi-cycles. Nous conclurons en explorant la microprogrammation, une abstraction élégante qui a transformé la conception du contrôle des processeurs.

### 1.1 La Quête de la Performance : Définir le Succès

Avant de concevoir ou d'évaluer une microarchitecture, il est impératif de définir un cadre rigoureux pour mesurer sa performance. La métrique ultime est le temps d'exécution d'un programme. Une analyse fondamentale décompose ce temps d'exécution en trois composantes essentielles. Pour un programme donné, le temps total d'exécution est donné par l'équation suivante :

$$\text{Temps}_{\text{exécution}} = N_{\text{instructions}} \times \text{CPI}_{\text{moyen}} \times T_{\text{cycle}}$$

Décortiquons chaque terme de cette équation fondamentale :

- **$N_{\text{instructions}}$ (Nombre d'instructions)** : Il s'agit du nombre total d'instructions dynamiques exécutées pour accomplir une tâche. Ce facteur est principalement influencé par la qualité du compilateur et la nature du jeu d'instructions (ISA) lui-même. Un ISA plus riche peut accomplir une tâche avec moins d'instructions, mais celles-ci peuvent être plus complexes à exécuter.

- **$\text{CPI}_{\text{moyen}}$ (Cycles Par Instruction, en moyenne)** : C'est le nombre moyen de cycles d'horloge nécessaires pour exécuter une seule instruction. Cette valeur est une caractéristique directe de la microarchitecture. Une conception qui accomplit beaucoup de travail à chaque cycle aura un CPI faible, tandis qu'une conception qui décompose le travail en de nombreuses petites étapes aura un CPI plus élevé.

- **$T_{\text{cycle}}$ (Durée du cycle d'horloge)** : Aussi appelé période d'horloge, c'est le temps nécessaire pour un cycle d'horloge complet. Il est l'inverse de la fréquence d'horloge ($F = 1/T_{\text{cycle}}$). Ce temps est déterminé par le chemin critique de la logique combinatoire dans la conception du processeur. Un chemin critique plus long (c'est-à-dire une logique plus complexe à traverser en un seul cycle) impose une durée de cycle plus longue et donc une fréquence d'horloge plus faible.

Cette équation n'est pas simplement une formule ; elle est la carte du territoire de l'architecte. Elle révèle les trois leviers sur lesquels il peut agir. Cependant, ces leviers ne sont pas indépendants. Une tension fondamentale existe, en particulier entre le CPI et la durée du cycle d'horloge. 

Les efforts pour réduire le CPI, par exemple en rendant chaque instruction plus puissante et en effectuant plus d'opérations en parallèle au sein d'un même cycle, tendent inévitablement à augmenter la complexité de la logique matérielle. Cette complexité accrue allonge le chemin critique, ce qui force une augmentation de $T_{\text{cycle}}$ (une diminution de la fréquence). Inversement, une stratégie visant à réduire agressivement $T_{\text{cycle}}$ en simplifiant radicalement chaque étape (par exemple, en ne faisant qu'une seule micro-opération simple par cycle) augmentera nécessairement le nombre de cycles requis pour chaque instruction, faisant ainsi grimper le CPI.

La conception d'une microarchitecture n'est donc pas la recherche d'un optimum absolu pour chaque paramètre, mais plutôt un art du compromis. C'est un exercice d'ingénierie qui consiste à trouver le meilleur équilibre entre ces facteurs interdépendants pour maximiser la performance globale pour une classe de programmes donnée.

### 1.2 Les Piliers de la Conception Architecturale : Au-delà de l'Intuition

Pour naviguer dans ce paysage complexe de compromis, les architectes s'appuient sur des principes de conception fondamentaux. Ces principes, distillés à partir de décennies d'expérience en ingénierie des systèmes, servent de guide pour prendre des décisions éclairées. Plutôt que de les utiliser pour une critique a posteriori, nous les présenterons comme une grille d'analyse a priori, un ensemble d'outils pour évaluer toute conception architecturale.

#### Le Principe du Chemin Critique (Critical Path Design)

Ce principe stipule qu'il faut **"trouver et diminuer le délai maximal de la logique combinatoire"**. L'objectif est de permettre la fréquence d'horloge la plus élevée possible. Cela implique souvent de décomposer une longue chaîne de logique en plusieurs étapes plus courtes, séparées par des registres. Lampson capture cet esprit avec le précepte **"Make it fast"**. Une conception qui ignore ce principe se condamne à une fréquence d'horloge faible.

#### Le Principe du Cas Courant (Bread and Butter / Common Case Design)

Ce principe, au cœur de la loi d'Amdahl, nous enjoint de **"dépenser du temps et des ressources là où c'est le plus important"**. En d'autres termes, il faut optimiser les opérations qui sont les plus fréquentes dans les programmes typiques. Lampson est encore plus direct : **"Handle normal and worst cases separately"**. Le chemin d'exécution pour le cas normal doit être aussi rapide et optimisé que possible, tandis que le cas exceptionnel doit avant tout être correct, même s'il est plus lent. Une conception qui traite tous les cas de la même manière est souvent une conception inefficace.

#### Le Principe de la Conception Équilibrée (Balanced Design)

L'objectif est d'**"équilibrer le flux de données à travers les composants matériels et d'éliminer les goulots d'étranglement"**. Un système est déséquilibré si une ressource est constamment surchargée tandis que d'autres sont inactives. Cela fait écho au conseil de Lampson de **"Split resources"** pour éviter qu'une ressource partagée ne devienne un point de contention. Une conception équilibrée garantit que le matériel est utilisé de manière productive.

Ces principes ne sont pas des lois immuables, mais des **"hints"**, des heuristiques puissantes qui forment une grille d'analyse. En armant l'étudiant de ces outils dès le départ, il peut devenir un participant actif dans l'évaluation des architectures que nous allons étudier, plutôt qu'un simple récepteur passif d'informations.

### 1.3 La Loi d'Amdahl : La Tyrannie du Séquentiel

Le principe du cas courant trouve sa formalisation mathématique dans la loi d'Amdahl. Souvent présentée comme une loi sur les limites du parallélisme, son application est en réalité bien plus large : c'est la loi fondamentale des rendements décroissants dans l'optimisation de n'importe quel système. Gene Amdahl l'a formulée en 1967 pour argumenter en faveur de la **"validité de l'approche à processeur unique"**. Il s'inquiétait que les tâches de **"maintenance de la gestion des données"** (data management housekeeping) représentaient un fardeau séquentiel inévitable qui limiterait sévèrement le potentiel des machines massivement parallèles de l'époque.

La loi s'exprime comme suit :

$$\text{Accélération}_{\text{globale}} = \frac{1}{(1-f) + \frac{f}{S}}$$

Où :
- $f$ est la fraction du temps d'exécution total qui peut bénéficier de l'amélioration (la partie "parallélisable" ou "optimisable")
- $S$ est le facteur d'accélération de cette fraction $f$

Le point crucial, et l'essence de l'argument d'Amdahl, est ce qui se passe lorsque l'amélioration devient infiniment efficace, c'est-à-dire lorsque $S \to \infty$. Dans ce cas, le terme $f/S$ tend vers zéro, et l'accélération globale est limitée par :

$$\lim_{S \to \infty} \text{Accélération}_{\text{globale}} = \frac{1}{1-f}$$

Cette formule est d'une simplicité désarmante mais d'une puissance redoutable. Elle nous dit que la performance globale d'un système est ultimement limitée par la fraction de la tâche qui ne peut pas être améliorée (la partie séquentielle, $1-f$). Si 90% d'un programme peut être parallélisé ($f=0.9$), l'accélération maximale possible, même avec un nombre infini de processeurs, ne sera jamais supérieure à $1/(1-0.9) = 10$ fois. Le goulot d'étranglement séquentiel devient le tyran de la performance.

Bien qu'Amdahl parlait de processeurs parallèles, sa loi s'applique de manière tout aussi pertinente à l'intérieur d'un seul processeur. Dans le contexte d'une microarchitecture, considérons "l'amélioration" comme l'optimisation d'une instruction spécifique (par exemple, une addition `add`). La "fraction non améliorable" $(1-f)$ est alors le temps d'exécution dicté par l'instruction la plus lente du système. 

Comme nous allons le voir, la loi d'Amdahl prédit mathématiquement et inéluctablement l'échec du modèle à cycle unique. Peu importe à quel point nous pourrions rendre les instructions simples comme `add` ou `jump` rapides, la performance globale sera toujours limitée par la latence de l'instruction la plus complexe, typiquement l'accès mémoire `load word`. C'est une application directe et puissante de la loi qui illustre parfaitement les limites d'une approche de conception simpliste.

## Partie 2 : L'Architecture à Cycle Unique : Une Étude de Cas sur l'Inefficacité

L'architecture à cycle unique est le point de départ naturel pour l'étude de la conception des processeurs. Sa philosophie est la simplicité : chaque instruction, de la plus simple à la plus complexe, s'exécute en exactement un seul cycle d'horloge. Bien qu'elle ne soit pas utilisée dans les processeurs modernes, son analyse est un exercice pédagogique inestimable, car elle expose de manière flagrante les conséquences de la violation des principes de conception fondamentaux.

### 2.1 Conception du Datapath : Le Coût de la Simultanéité

Le datapath (chemin de données) est l'ensemble des éléments matériels qui stockent et transforment les données : registres, Unité Arithmétique et Logique (ALU), mémoires, multiplexeurs et interconnexions. Dans une conception à cycle unique, le datapath doit contenir, simultanément, toutes les ressources nécessaires pour n'importe quelle étape de n'importe quelle instruction. Cette contrainte a des conséquences matérielles directes et coûteuses.

La construction du datapath se fait de manière incrémentale, en ajoutant les ressources nécessaires pour chaque type d'instruction :

1. **Instruction Fetch** : Tous les processeurs commencent par aller chercher l'instruction à exécuter. Cela nécessite un compteur de programme (PC) qui contient l'adresse de l'instruction, et une mémoire d'instructions pour stocker le code du programme.

2. **Instructions de type R (Registre)** : Des instructions comme `add rd, rs, rt` lisent deux registres (`rs`, `rt`), les traitent dans l'ALU, et écrivent le résultat dans un troisième registre (`rd`). Cela nécessite un banc de registres avec deux ports de lecture et un port d'écriture, ainsi qu'une ALU.

3. **Instructions de type load/store** : Une instruction comme `lw rt, offset(rs)` calcule une adresse en additionnant une valeur de registre (`rs`) et un décalage (`offset`), lit la donnée à cette adresse en mémoire, et l'écrit dans un registre (`rt`). Cela ajoute la nécessité d'une mémoire de données.

4. **Instructions de branchement** : Une instruction comme `beq rs, rt, label` compare deux registres, et si la condition est remplie, modifie le PC en y ajoutant un décalage pour sauter à une autre partie du code.

L'assemblage de ces pièces révèle les coûts de la simultanéité :

#### Double Mémoire

L'instruction `load word` doit, conceptuellement dans le même long cycle, lire une instruction de la mémoire (Fetch) et lire une donnée de la mémoire (Memory Access). Cela impose d'avoir soit deux mémoires séparées (une pour les instructions, une pour les données), soit une mémoire unique beaucoup plus complexe et coûteuse dotée de deux ports d'accès indépendants. La plupart des conceptions pédagogiques optent pour deux mémoires distinctes pour plus de clarté.

#### Additionneurs Multiples

Pour exécuter une instruction de type R, le PC doit être incrémenté pour pointer vers l'instruction suivante (PC = PC + 4). Simultanément, l'ALU principale peut être occupée à effectuer l'opération de l'instruction (par exemple, une soustraction). De plus, pour une instruction de branchement, un autre calcul d'adresse est nécessaire (PC + 4 + offset). Comme toutes ces opérations doivent se produire dans le même cycle, elles ne peuvent pas partager la même unité de calcul. Le datapath à cycle unique nécessite donc au moins trois unités arithmétiques distinctes : l'ALU principale, un additionneur dédié à l'incrémentation du PC, et un autre additionneur pour le calcul de l'adresse de branchement.

Cette incapacité à réutiliser le matériel (**"cannot reuse hardware"**) est un inconvénient majeur, conduisant à un coût matériel plus élevé et à une conception moins élégante. Le datapath est une collection redondante de ressources, où chaque composant est dimensionné pour le pire des cas, même s'il n'est utilisé qu'une fraction du temps.

### 2.2 Logique de Contrôle : Un Cerveau Purement Combinatoire

L'unité de contrôle est le "cerveau" du processeur. Elle reçoit en entrée l'opcode de l'instruction en cours et génère en sortie tous les signaux de contrôle nécessaires pour piloter le datapath. Ces signaux dictent quelle opération l'ALU doit effectuer, quels multiplexeurs doivent être sélectionnés, si la mémoire doit être lue ou écrite, et si un registre doit être mis à jour.

Dans une architecture à cycle unique, la logique de contrôle est purement combinatoire. Il n'y a pas de notion d'état ou de séquence. Les signaux de contrôle pour l'intégralité de l'exécution d'une instruction sont générés en une seule fois, comme une fonction de table de vérité de l'opcode.

Le tableau suivant illustre la table de vérité pour l'unité de contrôle principale d'un processeur MIPS simplifié. Il démystifie la "magie" du contrôle en montrant comment l'opcode de 6 bits est décodé de manière déterministe en un ensemble de signaux binaires qui orchestrent le flux de données.

#### Tableau 2.1 : Table de Vérité de l'Unité de Contrôle à Cycle Unique

| Instruction | Opcode | RegDst | ALUSrc | MemtoReg | RegWrite | MemRead | MemWrite | Branch | ALUOp1 | ALUOp0 |
|-------------|--------|--------|--------|----------|----------|---------|----------|--------|--------|--------|
| R-type      | 000000 | 1      | 0      | 0        | 1        | 0       | 0        | 0      | 1      | 0      |
| lw          | 100011 | 0      | 1      | 1        | 1        | 1       | 0        | 0      | 0      | 0      |
| sw          | 101011 | X      | 1      | X        | 0        | 0       | 1        | 0      | 0      | 0      |
| beq         | 000100 | X      | 0      | X        | 0        | 0       | 0        | 1      | 0      | 1      |

**Légende des signaux :**
- **RegDst** : 1 = `rd` est la destination ; 0 = `rt` est la destination
- **ALUSrc** : 1 = l'opérande B de l'ALU est l'immédiat ; 0 = l'opérande B vient du registre `rt`
- **MemtoReg** : 1 = la donnée écrite au registre vient de la mémoire ; 0 = elle vient de l'ALU
- **RegWrite** : 1 = écriture activée pour le banc de registres
- **MemRead** : 1 = lecture activée pour la mémoire de données
- **MemWrite** : 1 = écriture activée pour la mémoire de données
- **Branch** : 1 = le branchement est possible si le résultat de l'ALU est zéro
- **ALUOp** : 10 pour les instructions R-type (l'ALU décode funct), 00 pour lw/sw (addition), 01 pour beq (soustraction)
- **X** : Indifférent (Don't care)

Ce tableau est la traduction concrète du concept de "logique de contrôle" en un artefact d'ingénierie. Il peut être implémenté directement en matériel à l'aide de portes logiques.

### 2.3 Analyse Critique du Chemin Critique : La Course Contre la Montre

La faille la plus évidente de l'architecture à cycle unique est sa performance. Comme chaque instruction doit se terminer en un seul cycle, la durée de ce cycle ($T_{\text{cycle}}$) doit être suffisamment longue pour accommoder l'instruction la plus lente. Le chemin critique est la plus longue chaîne de délais à travers le datapath pour n'importe quelle instruction.

Pour quantifier ce problème, nous pouvons analyser les délais en utilisant des valeurs réalistes (bien que simplifiées) pour les composants :

- **Accès mémoire** (lecture ou écriture) : 200 ps
- **Opération ALU** et additionneurs : 100 ps
- **Accès au banc de registres** (lecture ou écriture) : 50 ps
- **Autres logiques** (multiplexeurs, extension de signe, etc.) : 0 ps (pour simplifier)

#### Tableau 2.2 : Calcul du Chemin Critique par Type d'Instruction

| Type d'Instruction | Étapes du Datapath et Composants Actifs | Délais (ps) | Délai Total (ps) |
|-------------------|------------------------------------------|-------------|-----------------|
| R-type | Fetch (Mem) → Read Regs → ALU → Write Regs | 200 + 50 + 100 + 50 | 400 |
| Load Word (lw) | Fetch (Mem) → Read Regs → ALU (addr calc) → Read Data (Mem) → Write Regs | 200 + 50 + 100 + 200 + 50 | 600 |
| Store Word (sw) | Fetch (Mem) → Read Regs → ALU (addr calc) → Write Data (Mem) | 200 + 50 + 100 + 200 | 550 |
| Branch (beq) | Fetch (Mem) → Read Regs → ALU (compare) | 200 + 50 + 100 | 350 |
| Jump (j) | Fetch (Mem) | 200 | 200 |

Cette analyse révèle un déséquilibre flagrant. L'instruction `load word` est le maillon faible, avec un chemin critique de 600 ps. Par conséquent, la durée du cycle d'horloge, $T_{\text{cycle}}$, doit être d'au moins 600 ps (plus les délais de synchronisation des registres que nous avons ignorés). Cela signifie que la fréquence maximale du processeur est de $1/(600 \times 10^{-12}\text{s}) \approx 1.67$ GHz.

Le véritable problème est le gaspillage. Une instruction `jump`, qui ne nécessite que 200 ps, est forcée d'occuper un cycle entier de 600 ps. Pendant les 400 ps restants, le datapath est largement inactif, mais le processeur ne peut pas commencer l'instruction suivante. De même, une instruction `add` (type R) gaspille 200 ps. C'est une inefficacité fondamentale.

Le problème est en réalité bien pire. Notre hypothèse de 200 ps pour un accès mémoire est extrêmement optimiste. Un accès à une mémoire DRAM externe moderne prend des dizaines de nanosecondes (par exemple, 50 ns, soit 50 000 ps). Si nous devions concevoir un processeur à cycle unique avec une mémoire réaliste, la durée du cycle serait dictée par cet accès mémoire, conduisant à des fréquences d'horloge ridiculement basses (par exemple, 20 MHz), ce qui est inacceptable pour un processeur moderne.

### 2.4 La Faille Fondamentale : Le Verdict des Principes

Armés de notre grille d'analyse, nous pouvons maintenant prononcer un verdict clair sur l'architecture à cycle unique. Elle est **"déraisonnable"** car elle viole de manière spectaculaire les principes de conception fondamentaux.

#### Violation du Principe du Chemin Critique

Par sa définition même, l'architecture à cycle unique refuse de décomposer le travail. Elle accepte le chemin critique le plus long comme une fatalité, ce qui l'empêche d'atteindre des fréquences d'horloge élevées.

#### Violation du Principe du Cas Courant

C'est sa faille la plus grave. Supposons qu'un programme soit composé à 99% d'instructions arithmétiques rapides (400 ps) et à 1% d'instructions load lentes (600 ps). Toute optimisation des instructions arithmétiques est vaine. La performance globale est dictée par le cas le plus lent et le plus rare, et non par le cas commun. C'est une violation directe du principe de Lampson **"Handle normal and worst cases separately"** et une illustration parfaite de la tyrannie du séquentiel prédite par la loi d'Amdahl. Dans ce scénario, la partie "non améliorable" $(1-f)$ est le temps d'exécution de l'instruction `load word`, qui contraint l'ensemble du système.

#### Violation du Principe de Conception Équilibrée

Le gaspillage de ressources est énorme. Pour une instruction `jump`, l'ALU principale, le banc de registres et la mémoire de données sont inutilisés pendant la majeure partie du cycle. Pour une instruction `add`, la mémoire de données est inactive. Les ressources ne sont pas utilisées de manière productive, ce qui conduit à un système déséquilibré et inefficace.

### 2.5 Implémentation : Simulation d'un Processeur à Cycle Unique

Pour solidifier la compréhension, il est utile de modéliser ce processeur en logiciel. Le pseudocode suivant capture l'essence d'une exécution à cycle unique, où toutes les étapes se déroulent de manière monolithique.

```javascript
// Pseudocode pour la simulation d'une instruction à cycle unique

function execute_single_cycle(instruction, registres, mémoire) {
  // Étape 1 & 2: Décodage et génération des signaux de contrôle
  // L'opcode est extrait de l'instruction.
  opcode = instruction.bits[31:26]
  // La logique de contrôle est une table de vérité basée sur l'opcode.
  contrôles = decode_control_logic(opcode)

  // Étape 3: Lecture des registres
  // Les adresses des registres source (rs, rt) sont extraites.
  rs_addr = instruction.bits[25:21]
  rt_addr = instruction.bits[20:16]
  read_data1 = registres[rs_addr]
  read_data2 = registres[rt_addr]

  // Étape 4: Opération de l'ALU
  // L'immédiat est étendu en signe.
  immediate = sign_extend(instruction.bits[15:0])
  // Le multiplexeur ALUSrc choisit entre le registre rt et l'immédiat.
  alu_input2 = (contrôles.ALUSrc == 1) ? immediate : read_data2
  // L'ALU effectue l'opération dictée par les signaux de contrôle.
  [alu_result, zero_flag] = execute_alu(read_data1, alu_input2, contrôles.ALUOp, instruction.bits[5:0])

  // Étape 5: Accès Mémoire
  // L'adresse mémoire est le résultat de l'ALU (pour lw/sw).
  // La donnée à écrire est read_data2 (pour sw).
  memory_data = memory_access(alu_result, read_data2, contrôles.MemRead, contrôles.MemWrite, mémoire)

  // Étape 6: Écriture dans les registres (Write-back)
  // Le multiplexeur MemtoReg choisit entre le résultat de l'ALU et la donnée de la mémoire.
  write_data = (contrôles.MemtoReg == 1) ? memory_data : alu_result
  // Le multiplexeur RegDst choisit entre rt et rd comme registre de destination.
  write_reg_addr = (contrôles.RegDst == 1) ? instruction.bits[15:11] : rt_addr
  // Si RegWrite est activé, la donnée est écrite.
  if (contrôles.RegWrite == 1) {
    registres[write_reg_addr] = write_data
  }

  // Étape 7: Mise à jour du PC
  // Logique pour le branchement et le saut (non détaillée ici pour la simplicité).
  nouveau_pc = calculate_next_pc(pc, instruction, contrôles.Branch, zero_flag)
  
  return nouveau_pc
}
```

Une implémentation en Python suivrait cette structure, avec une classe `SingleCycleCPU` possédant des attributs pour les registres, la mémoire et le PC, et une méthode `run_instruction` qui exécute cette séquence monolithique.

```python
# Esquisse d'une implémentation Python d'un CPU à cycle unique

class SingleCycleCPU:
    def __init__(self, program_memory):
        self.pc = 0
        self.registers = [0] * 32
        self.data_memory = [0] * 1024
        self.program_memory = program_memory

    def run_instruction(self):
        # 1. Fetch
        instruction_word = self.program_memory[self.pc]
        
        # Simuler toutes les étapes en une seule fonction monolithique
        # 2. Decode & Control
        opcode = (instruction_word >> 26) & 0x3F
        # ... logique de contrôle pour générer tous les signaux...
        
        # 3. Register Read
        rs = (instruction_word >> 21) & 0x1F
        rt = (instruction_word >> 16) & 0x1F
        rd = (instruction_word >> 11) & 0x1F
        read_data1 = self.registers[rs]
        read_data2 = self.registers[rt]
        
        # 4. ALU Execution
        # ... logique des multiplexeurs et de l'ALU...
        # alu_result = ...
        
        # 5. Memory Access
        # ... logique d'accès mémoire...
        # memory_data = ...
        
        # 6. Write Back
        # ... logique du multiplexeur de destination et écriture...
        
        # 7. PC Update
        # ... logique de mise à jour du PC...
        # self.pc = new_pc
        
        print(f"Executing instruction at PC={self.pc}, instruction={hex(instruction_word)}")
```

Cette implémentation, bien que simplifiée, met en évidence la nature "tout-en-un" du cycle. Toutes les ressources sont accédées et toutes les décisions sont prises dans le cadre d'un seul appel de fonction, reflétant la nature combinatoire et simultanée du matériel.

## Partie 3 : L'Architecture Multi-Cycles : Efficacité et Réutilisation

Face aux limitations flagrantes du modèle à cycle unique, la prochaine étape évolutive est l'architecture multi-cycles. L'idée centrale est simple mais profonde : abandonner la contrainte rigide d'une instruction par cycle pour permettre à chaque instruction de prendre un nombre variable de cycles, correspondant uniquement au travail qu'elle doit réellement accomplir. C'est une application directe du principe de conception **"Diviser pour mieux régner"**.

### 3.1 La Philosophie Multi-Cycles : Diviser pour Mieux Régner

La philosophie multi-cycles repose sur un compromis fondamental, directement visible dans l'équation de la performance. Nous allons décomposer l'exécution d'une instruction en une série d'étapes plus petites et plus simples (par exemple, Fetch, Decode, Execute, Memory, Write-back). Chacune de ces étapes sera conçue pour s'exécuter en un seul cycle d'horloge court.

Les conséquences sur les métriques de performance sont doubles :

#### Réduction drastique de $T_{\text{cycle}}$

La durée du cycle d'horloge n'est plus dictée par l'instruction la plus lente, mais par l'étape la plus longue parmi toutes les étapes possibles. Par exemple, si l'étape la plus longue est une opération ALU (100 ps) ou un accès mémoire (200 ps), $T_{\text{cycle}}$ peut être fixé à cette valeur (par exemple, 200 ps), au lieu des 600 ps du modèle à cycle unique. Cela permet une fréquence d'horloge beaucoup plus élevée.

#### Augmentation du CPI

Le prix à payer est que le CPI n'est plus égal à 1. Chaque instruction prendra désormais plusieurs cycles pour s'exécuter. Une instruction `add` pourrait prendre 4 cycles (Fetch, Decode, Execute, Write-back), tandis qu'une instruction `lw` pourrait en prendre 5 (Fetch, Decode, Execute-AddrCalc, Memory, Write-back). Le CPI devient une valeur variable et, en moyenne, supérieure à 1.

L'espoir est que la réduction significative de $T_{\text{cycle}}$ l'emporte sur l'augmentation du CPI, conduisant à une diminution globale du temps d'exécution. Par exemple, comparons l'exécution d'une instruction `lw` :

- **Cycle Unique** : 1 instruction × 1 CPI × 600 ps = 600 ps
- **Multi-Cycles** : 1 instruction × 5 CPI × 200 ps = 1000 ps

À première vue, pour une instruction `lw`, le modèle multi-cycles semble plus lent. Cependant, considérons une instruction `add` :

- **Cycle Unique** : 1 instruction × 1 CPI × 600 ps = 600 ps
- **Multi-Cycles** : 1 instruction × 4 CPI × 200 ps = 800 ps

L'avantage n'est pas encore évident. Le véritable gain provient de deux facteurs : premièrement, la capacité à gérer des latences mémoires réalistes (que nous verrons plus tard) et, deuxièmement, l'efficacité matérielle. La décomposition en étapes permet la réutilisation des composants matériels coûteux sur plusieurs cycles d'horloge, ce qui conduit à une conception plus économique et élégante.

### 3.2 Conception du Datapath : L'Art de la Réutilisation

Le passage à un modèle multi-cycles transforme radicalement le datapath. La contrainte de simultanéité étant levée, nous pouvons réutiliser les ressources.

#### Une seule mémoire unifiée

Puisque le Fetch d'instruction et l'accès aux données se produisent dans des cycles d'horloge distincts, ils peuvent partager la même unité de mémoire.

#### Une seule ALU

L'ALU peut être utilisée dans un cycle pour calculer PC + 4, dans un autre pour calculer une adresse mémoire (base + offset), et dans un autre encore pour effectuer une opération arithmétique. Les trois additionneurs du modèle à cycle unique sont remplacés par une seule ALU polyvalente.

Cependant, cette réutilisation a un coût. Pour conserver les résultats d'une étape afin de les utiliser dans une étape ultérieure, nous devons introduire de nouveaux registres. Ces registres, dits microarchitecturaux, sont invisibles pour le programmeur mais essentiels au fonctionnement interne du processeur. Les principaux registres intermédiaires ajoutés sont :

- **Instruction Register (IR)** : Stocke l'instruction extraite de la mémoire pendant toute la durée de son exécution
- **Memory Data Register (MDR)** : Stocke temporairement les données lues de la mémoire
- **A et B** : Stockent les valeurs lues depuis le banc de registres
- **ALUOut** : Stocke le résultat de la dernière opération de l'ALU

De plus, pour permettre aux différentes sources de données d'alimenter les ressources partagées (comme l'ALU et la mémoire) à différents moments, de nouveaux multiplexeurs sont nécessaires. Par exemple, un multiplexeur `IorD` est placé à l'entrée d'adresse de la mémoire pour choisir entre le PC (pendant le cycle de Fetch) et ALUOut (pendant un cycle d'accès aux données pour `lw` ou `sw`). 

Le datapath multi-cycles est donc un équilibre entre la frugalité (réutilisation des unités coûteuses) et l'ajout de logique de stockage et de sélection (registres intermédiaires et multiplexeurs).

### 3.3 Le Contrôle par Machine à États Finis (FSM)

Avec la décomposition de l'exécution en une séquence d'étapes, la logique de contrôle ne peut plus être purement combinatoire. Elle doit avoir une mémoire pour savoir à quelle étape de l'exécution elle se trouve. Le contrôle devient donc une machine à états finis (FSM - Finite State Machine).

La FSM est le "cerveau" séquentiel du processeur multi-cycles :

- Chaque état de la machine correspond à un cycle d'horloge et à une étape d'exécution
- Dans chaque état, la FSM génère les signaux de contrôle appropriés pour le datapath
- À la fin de chaque cycle, la FSM transitionne vers l'état suivant en fonction de l'opcode de l'instruction en cours et potentiellement d'autres signaux (comme le flag zero de l'ALU pour un branchement)

Le diagramme d'états pour un sous-ensemble d'instructions MIPS illustre ce processus :

#### État 0 (Fetch)
- **Action** : IR = Memory[PC], PC = PC + 4
- **Transition** : Inconditionnellement vers l'État 1

#### État 1 (Decode/Register Fetch)
- **Action** : A = Regs[rs], B = Regs[rt], ALUOut = PC + (sign_extend(IR[15:0]) << 2) (calcul anticipé de l'adresse de branchement)
- **Transition** : Multi-voies basée sur l'opcode (IR[31:26])
  - Si `lw` ou `sw` → État 2
  - Si type R → État 6
  - Si `beq` → État 8
  - Si `jump` → État 9

#### État 2 (Memory Address Computation)
- **Action** : ALUOut = A + sign_extend(IR[15:0])
- **Transition** :
  - Si `lw` → État 3
  - Si `sw` → État 5

#### État 3 (Memory Read)
- **Action** : MDR = Memory[ALUOut]
- **Transition** : Vers l'État 4

#### État 4 (Write-back pour lw)
- **Action** : Regs[rt] = MDR
- **Transition** : Retour à l'État 0

... et ainsi de suite pour chaque type d'instruction.

La logique de contrôle peut être spécifiée par une table, qui est plus complexe que celle du modèle à cycle unique car elle inclut la notion d'état.

#### Tableau 3.1 : Extrait de la Table de Contrôle de la FSM Multi-Cycles

| État | Opération | PCWrite | IorD | MemRead | MemWrite | IRWrite | RegWrite | ALUSrcA | ALUSrcB | ALUOp | État Suivant |
|------|-----------|---------|------|---------|----------|---------|----------|---------|---------|-------|--------------|
| 0 | Fetch | 1 | 0 | 1 | 0 | 1 | 0 | 0 | 01 | add | 1 |
| 1 | Decode | 0 | X | 0 | 0 | 0 | 0 | 0 | 11 | add | Dispatch |
| 2 | MemAddr | 0 | X | 0 | 0 | 0 | 0 | 1 | 10 | add | 3 (si lw), 5 (si sw) |
| 3 | MemRead | 0 | 1 | 1 | 0 | 0 | 0 | X | XX | X | 4 |
| 4 | WB (lw) | 0 | X | 0 | 0 | 0 | 1 | X | XX | X | 0 |
| 6 | R-exec | 0 | X | 0 | 0 | 0 | 0 | 1 | 00 | funct | 7 |
| 7 | WB (R) | 0 | X | 0 | 0 | 0 | 1 | X | XX | X | 0 |

Cette table est la feuille de route de l'exécution. Elle montre la nature séquentielle du contrôle, où les signaux changent à chaque cycle pour orchestrer une "danse" complexe des données à travers le datapath réutilisable.

### 3.4 Gestion des Latences Variables : Le Test de la Réalité

L'un des avantages les plus profonds et les plus pratiques du modèle multi-cycles est sa capacité à gérer nativement des opérations à latence variable, le cas le plus important étant l'accès à une mémoire lente.

Dans un système réel, une mémoire DRAM externe ne répond pas instantanément. Elle peut prendre des dizaines, voire des centaines de cycles de processeur rapides pour fournir une donnée. Une architecture à cycle unique serait paralysée par ce délai. L'architecture multi-cycles, grâce à sa FSM, offre une solution élégante.

On introduit un simple signal de communication entre la mémoire et le processeur, souvent appelé `MemoryReady` ou `Wait` :

1. Lorsque le processeur initie un accès mémoire (en fournissant une adresse et en activant `MemRead`), il entre dans un état d'attente (par exemple, l'État 3 pour `lw`)

2. La logique de transition de cet état est modifiée : au lieu de passer inconditionnellement à l'état suivant, la FSM teste le signal `MemoryReady`

3. Si `MemoryReady` est bas (0), cela signifie que la mémoire n'a pas encore terminé. La FSM reste dans le même état, cycle après cycle, en attendant. Le processeur "patine" efficacement

4. Lorsque la mémoire a terminé son opération, elle met le signal `MemoryReady` à haut (1)

5. Au prochain front d'horloge, la FSM détecte que `MemoryReady` est haut et transitionne enfin vers l'état suivant (par exemple, l'État 4) pour utiliser la donnée qui est maintenant disponible dans le MDR

Ce mécanisme réalise un découplage crucial entre la fréquence du processeur et la latence de la mémoire. Le processeur peut fonctionner à une fréquence très élevée, déterminée par ses étapes internes les plus rapides (comme une opération ALU), et simplement "attendre" pendant le nombre nécessaire de ses propres cycles rapides que la mémoire lente réponde. C'est cette capacité qui rend les architectures modernes, confrontées au "mur de la mémoire" (l'écart de performance croissant entre processeurs et mémoire), possibles. C'est la première étape fondamentale pour gérer ce goulot d'étranglement majeur des systèmes informatiques.

### 3.5 Implémentation : Simulation d'un Processeur Multi-Cycles

La simulation d'un processeur multi-cycles est fondamentalement différente de celle d'un processeur à cycle unique. Elle doit modéliser explicitement la FSM et l'avancement cycle par cycle.

```javascript
// Pseudocode pour la simulation d'un processeur multi-cycles

function execute_multi_cycle_program(mémoire_programme) {
  // Initialisation des registres microarchitecturaux
  let [pc, ir, mdr, a, b, alu_out] = [0, 0, 0, 0, 0, 0]
  let registres_architecturaux = new Array(32).fill(0)
  
  // La FSM commence à l'état de Fetch
  let état_actuel = FETCH_STATE_0

  while (true) {
    // Déterminer les signaux de contrôle pour l'état actuel
    // et l'opcode de l'instruction en cours (stockée dans IR)
    let opcode = (ir >> 26) & 0x3F
    let contrôles = generate_controls_for_state(état_actuel, opcode)

    // Exécuter les actions du datapath pour le cycle actuel
    // (Mise à jour des registres microarchitecturaux et architecturaux)
    // Cette fonction simule les multiplexeurs, l'ALU, la mémoire, etc.
    // en fonction des signaux de contrôle.
    execute_datapath_cycle(contrôles, pc, ir, mdr, /* ... */)

    // Déterminer l'état suivant
    état_actuel = next_state_logic(état_actuel, opcode, contrôles)

    // Simuler un coup d'horloge
    tick()

    // Condition d'arrêt (par exemple, instruction HALT)
    if (programme_terminé) {
      break
    }
  }
}
```

L'implémentation en Python reflète cette structure en boucle, avec le cœur de la logique étant une grande structure de décision (par exemple, un `if`/`elif`/`else` ou un dictionnaire de fonctions) qui simule la FSM. Chaque branche de la décision correspond à un état, met à jour les registres microarchitecturaux en fonction des signaux de contrôle de cet état, et détermine l'état suivant.

```python
# Esquisse d'une implémentation Python d'un CPU multi-cycles

class MultiCycleCPU:
    def __init__(self, program_memory):
        # Registres architecturaux
        self.pc = 0
        self.registers = [0] * 32
        
        # Registres microarchitecturaux
        self.ir = 0
        self.mdr = 0
        self.a = 0
        self.b = 0
        self.alu_out = 0
        
        # Mémoires
        self.data_memory = [0] * 1024
        self.program_memory = program_memory
        
        # FSM
        self.current_state = 0  # État initial = Fetch

    def run_cycle(self):
        # Le cœur de la simulation : exécuter un cycle d'horloge
        
        if self.current_state == 0:  # État Fetch
            # Actions: IR = Mem[PC], PC = PC + 4
            self.ir = self.program_memory[self.pc]
            # ... simuler l'ALU pour PC+4 et mettre à jour le PC...
            # Transition
            self.current_state = 1
            
        elif self.current_state == 1:  # État Decode
            opcode = (self.ir >> 26) & 0x3F
            # Actions: Lire les registres dans A et B
            # ...
            # Transition (Dispatch)
            if opcode == 0x23:  # lw
                self.current_state = 2
            elif opcode == 0x00:  # R-type
                self.current_state = 6
            # ... autres transitions...
            
        elif self.current_state == 2:  # État MemAddr (pour lw/sw)
            # Actions: ALUOut = A + sign_extend(imm)
            # ...
            self.current_state = 3
            
        # ... implémenter tous les autres états de la FSM...

    def run_program(self):
        while True:
            self.run_cycle()
            # Ajouter une condition d'arrêt
```

Cette structure modélise fidèlement le comportement séquentiel du processeur multi-cycles, où chaque appel à `run_cycle` représente un coup d'horloge et fait avancer la machine d'un état à l'autre.

## Partie 4 : La Microprogrammation : L'Abstraction du Contrôle

L'architecture multi-cycles, avec sa machine à états finis, représente un bond en avant significatif en termes d'efficacité et de flexibilité par rapport au modèle à cycle unique. Cependant, la conception de l'unité de contrôle, même sous forme de FSM, peut devenir complexe et rigide, une "logique câblée" (hardwired) difficile à concevoir, à déboguer et à modifier. Au début des années 1950, un pionnier de l'informatique, Maurice Wilkes, a proposé une solution d'une élégance et d'une puissance remarquables : la microprogrammation.

### 4.1 L'Idée de Maurice Wilkes : "The Best Way to Design an Automatic Calculating Machine"

Dans son article fondateur de 1951, Wilkes a cherché à remplacer la conception de la logique de contrôle, qu'il jugeait "ad hoc", par une approche systématique et ordonnée, analogue à la programmation elle-même. Son idée fondamentale était de décomposer chaque instruction machine (le niveau de l'ISA) en une séquence d'opérations encore plus élémentaires, qu'il a appelées **micro-opérations**. Une micro-opération est une action primitive du datapath, comme un transfert de données entre deux registres, une opération de l'ALU, ou un accès à la mémoire.

Une instruction machine complexe, comme `add` ou `lw`, est alors implémentée comme une routine, une séquence de ces micro-opérations. Wilkes a appelé cette séquence un **micro-programme**. L'importance de cette idée ne peut être sous-estimée. Elle a introduit un nouveau niveau d'abstraction sémantique dans l'architecture des ordinateurs. Le processeur avait désormais deux niveaux de programmation :

1. **Le programme** (ou macro-programme), visible par le programmeur, composé d'instructions de l'ISA
2. **Le micro-programme**, invisible pour le programmeur, résidant à l'intérieur du processeur, qui définit comment chaque instruction de l'ISA est réellement exécutée

### 4.2 Le Micro-séquenceur et le Magasin de Contrôle (Control Store)

Pour implémenter cette idée, Wilkes a proposé de remplacer la logique câblée de la FSM par une structure de type mémoire. Cela formalise la FSM de la partie précédente en un modèle microprogrammé.

#### La Micro-instruction

Chaque état de notre FSM précédente correspond maintenant à une micro-instruction. Une micro-instruction est simplement un mot binaire très large où chaque bit, ou chaque petit champ de bits, correspond directement à un signal de contrôle du datapath (`RegWrite`, `ALUSrc`, `MemRead`, etc.). Elle contient également des informations pour déterminer la prochaine micro-instruction à exécuter.

#### Le Magasin de Contrôle (Control Store)

C'est une mémoire spéciale, généralement une mémoire morte (ROM) très rapide, qui stocke l'ensemble du micro-programme. Chaque ligne de cette mémoire contient une micro-instruction. L'adresse d'une ligne correspond directement au numéro de l'état de la FSM.

#### Le Micro-séquenceur

C'est la logique qui remplace la logique de transition d'état de la FSM câblée. Son rôle est de calculer l'adresse de la prochaine micro-instruction à charger depuis le magasin de contrôle. Pour ce faire, il utilise des informations contenues dans la micro-instruction actuelle (par exemple, un champ "Next Address"), l'opcode de l'instruction machine, et des signaux d'état du datapath (comme le flag zero).

Le fonctionnement est le suivant : à chaque cycle d'horloge, le micro-séquenceur fournit une adresse au magasin de contrôle. Le magasin de contrôle sort la micro-instruction correspondante. Les bits de cette micro-instruction sont envoyés directement aux lignes de contrôle du datapath, orchestrant l'action pour ce cycle. Simultanément, d'autres bits de la micro-instruction, ainsi que l'opcode, alimentent le micro-séquenceur pour qu'il prépare l'adresse du cycle suivant.

### 4.3 Avantages Profonds de la Microprogrammation

Cette abstraction du contrôle en un "logiciel" interne (le micro-programme) a des avantages considérables, qui ont dominé la conception des ordinateurs pendant des décennies.

#### Flexibilité et Extensibilité de l'ISA

La conception microprogrammée rend l'ajout de nouvelles instructions à un ISA existant beaucoup plus simple. Si le datapath existant peut supporter les micro-opérations nécessaires, ajouter une nouvelle instruction se résume à ajouter une nouvelle routine (une séquence de micro-instructions) au magasin de contrôle. Il n'est pas nécessaire de modifier le matériel complexe et coûteux.

#### Support des Architectures CISC (Complex Instruction Set Computer)

La microprogrammation a été la technologie clé qui a permis l'âge d'or des architectures CISC comme l'IBM S/360, le DEC VAX, ou le Motorola 68000. Ces machines proposaient des instructions extrêmement puissantes et complexes (par exemple, une seule instruction VAX pouvait copier une chaîne de caractères entière, ou évaluer un polynôme). Ces instructions étaient impossibles à implémenter en un seul cycle. Au lieu de cela, elles étaient traduites par le micro-séquenceur en de longues séquences de dizaines, voire de centaines, de micro-instructions simples exécutées par un datapath interne relativement standard, de type RISC (Reduced Instruction Set Computer).

#### Correction de Bugs Matériels Post-Production

C'est l'un des avantages les plus fascinants. Si un bug est découvert dans l'implémentation matérielle d'une instruction après que des milliers de puces ont été fabriquées et vendues, il est économiquement impossible de les rappeler. Avec une conception microprogrammée, le fabricant peut corriger le bug en modifiant la routine du micro-programme correspondante. Il peut alors distribuer une mise à jour du microcode (souvent chargée au démarrage du système d'exploitation) qui réécrit la partie défectueuse du magasin de contrôle (si celui-ci est une mémoire réinscriptible comme une RAM ou une Flash). C'est littéralement du matériel qui se comporte comme du logiciel, une idée révolutionnaire qui est encore utilisée aujourd'hui pour corriger des failles de sécurité matérielles comme Spectre et Meltdown.

### 4.4 Implémentation : Un Micro-séquenceur Élémentaire

La logique de contrôle de notre processeur multi-cycles peut être réimplémentée en utilisant ce modèle. Au lieu d'une grande structure `if`/`else` câblée, nous aurons une table de consultation (le magasin de contrôle) et une logique de séquencement.

```javascript
// Pseudocode pour un processeur microprogrammé

// Le magasin de contrôle est un tableau de micro-instructions.
// Chaque micro-instruction contient les bits de contrôle et des infos de séquencement.
const control_store = [/* ... */]

// Le micro-PC pointe vers la micro-instruction en cours dans le magasin de contrôle.
let micro_pc = 0

function execute_microprogrammed_program() {
  while (true) {
    // 1. Fetch de la micro-instruction
    let micro_instruction = control_store[micro_pc]

    // 2. Appliquer les signaux de contrôle au datapath
    apply_controls_to_datapath(micro_instruction.datapath_signals)

    // 3. Calculer l'adresse de la prochaine micro-instruction
    micro_pc = calculate_next_micro_pc(micro_instruction.sequencing_info, instruction.opcode)

    // Simuler un coup d'horloge
    tick()
    
    // Condition d'arrêt
    if (programme_terminé) {
      break
    }
  }
}

function calculate_next_micro_pc(seq_info, opcode) {
  if (seq_info.type == 'SEQ') {
    // Aller à la prochaine micro-instruction séquentiellement.
    return micro_pc + 1
  } else if (seq_info.type == 'DISPATCH') {
    // Sauter à une adresse basée sur l'opcode de l'instruction machine.
    return seq_info.table[opcode]
  } else if (seq_info.type == 'JUMP_ON_ZERO') {
    // Sauter conditionnellement en fonction du flag 'zero' de l'ALU.
    return (alu.zero_flag) ? seq_info.target_addr : micro_pc + 1
  }
}
```

Cette structure est beaucoup plus régulière et modulaire. La complexité est déplacée de la logique câblée vers le contenu du `control_store`, qui peut être conçu et vérifié comme un programme.

## Partie 5 : Conclusion et Ouverture vers le Pipelining

Notre voyage à travers les architectures de processeurs nous a menés de la simplicité brute du modèle à cycle unique à l'efficacité et à l'élégance du modèle multi-cycles, culminant avec l'abstraction puissante de la microprogrammation. Il est temps de faire le bilan et de se demander, une dernière fois : **"Pouvons-nous faire mieux?"**.

### 5.1 Synthèse des Architectures : Le Bilan des Compromis

Le choix entre une architecture à cycle unique et une architecture multi-cycles est un cas d'école classique de compromis en ingénierie. Le tableau suivant résume leurs caractéristiques distinctes et sert de bilan pour notre analyse.

#### Tableau 5.1 : Tableau Comparatif Final des Architectures

| Critère | Architecture à Cycle Unique | Architecture Multi-Cycles |
|---------|----------------------------|---------------------------|
| **CPI** | Exactement 1 pour toutes les instructions | Variable selon l'instruction (ex: 3 pour jump, 4 pour add, 5 pour lw) |
| **Durée du Cycle d'Horloge** | Longue. Déterminée par l'instruction la plus lente (ex: load word) | Courte. Déterminée par l'étape la plus longue (ex: accès mémoire ou opération ALU) |
| **Utilisation du Matériel** | Faible. De nombreuses unités sont inactives pendant un cycle donné. Gaspillage de ressources | Élevée. Les unités fonctionnelles coûteuses (ALU, Mémoire) sont réutilisées sur plusieurs cycles |
| **Coût Matériel** | Élevé. Nécessite la duplication des ressources (plusieurs additionneurs, mémoires séparées) | Plus faible. Une seule ALU et une seule mémoire unifiée, mais ajout de registres intermédiaires |
| **Complexité du Contrôle** | Simple. Logique de contrôle purement combinatoire, basée sur l'opcode | Complexe. Nécessite une machine à états finis (FSM) pour séquencer les étapes. Peut être simplifiée par microprogrammation |
| **Optimisation du Cas Courant** | Impossible. La performance est dictée par le pire des cas, en violation de la loi d'Amdahl | Possible. Les instructions simples s'exécutent en moins de cycles, améliorant la performance moyenne |

En résumé, l'architecture à cycle unique est simple à comprendre mais terriblement inefficace et coûteuse. L'architecture multi-cycles est plus complexe mais offre des performances et une utilisation du matériel bien supérieures, ce qui en fait une base de conception beaucoup plus réaliste.

### 5.2 "Pouvons-nous faire mieux?" : La Prochaine Frontière

La question récurrente **"Can we do better?"** est le moteur du progrès en architecture. Le modèle multi-cycles, bien que supérieur, n'est pas parfait. Son défaut réside dans son inefficacité résiduelle. Bien qu'il réutilise les unités fonctionnelles, il ne le fait que séquentiellement. À un instant donné, une seule instruction est en cours d'exécution dans le processeur.

Considérons l'exécution d'une série d'instructions. Pendant que la première instruction est dans son étape de Fetch, l'ALU et l'unité d'écriture du banc de registres sont inactives. Puis, lorsque cette même instruction passe à l'étape Execute, l'unité de fetch de la mémoire est inactive. À chaque cycle d'horloge, seule une petite partie du datapath est active, tandis que le reste attend son tour.

Ce constat ouvre la porte à la prochaine grande idée en architecture des processeurs. Si les différentes étapes de l'exécution d'une instruction utilisent différentes parties du datapath, pourquoi ne pas superposer l'exécution de plusieurs instructions? Pendant que l'instruction N est dans son étape Execute, l'instruction N+1 pourrait être dans son étape Decode, et l'instruction N+2 pourrait être dans son étape Fetch. Toutes les unités du datapath pourraient ainsi travailler en parallèle, chacune sur une instruction différente.

Cette technique de superposition est la définition même du **pipelining**. C'est la prochaine frontière que nous explorerons, une technique qui permet d'augmenter considérablement le débit d'instructions (le nombre d'instructions terminées par unité de temps) et qui est au cœur de la quasi-totalité des processeurs modernes. Le modèle multi-cycles a résolu le problème de la durée du cycle ; le pipelining s'attaquera au problème du CPI.

---

*Ce document constitue une introduction complète aux principes fondamentaux de l'architecture des microprocesseurs, établissant les bases nécessaires pour comprendre les techniques avancées comme le pipelining, la prédiction de branchement, et l'exécution superscalaire qui caractérisent les processeurs modernes.*
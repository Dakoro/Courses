# Cours d'Architecture des Ordinateurs : Conception d'un Processeur Monocycle

## Partie 1 : Fondations : Le Contrat et la Machine

### 1.1. Introduction : L'Architecture au Cœur de l'Informatique

Bonjour à tous. Aujourd'hui, nous allons nous aventurer au-delà de la vision du programmeur pour adopter celle de l'architecte. Jusqu'à présent, nous avons interagi avec des machines en leur disant quoi faire. Notre objectif est maintenant de comprendre comment elles le font. Nous allons entamer un voyage passionnant : la conception d'une microarchitecture complète à partir de zéro. En partant des principes fondamentaux, nous allons construire, étape par étape, un processeur fonctionnel.

### 1.2. Le Contrat Matériel-Logiciel : L'Architecture de Jeu d'Instructions (ISA)

Au cœur de tout système informatique se trouve un contrat fondamental, une interface qui régit l'interaction entre le matériel (le hardware) et le logiciel (le software). Ce contrat est appelé l'**Architecture de Jeu d'Instructions**, ou **ISA (Instruction Set Architecture)**. C'est ce document, parfois long de plusieurs milliers de pages, qui expose au programmeur les capacités du matériel.

Pour bien saisir cette notion, utilisons une analogie : la pédale d'accélérateur d'une voiture.

#### L'ISA est la pédale d'accélérateur

Elle offre une interface simple avec une sémantique garantie. Si vous appuyez sur la pédale, la voiture accélère ; si vous ne le faites pas, elle n'accélère pas. C'est tout ce que l'interface vous promet. De la même manière, une instruction `add` dans un ISA garantit que les valeurs de deux registres seront additionnées et le résultat stocké dans un troisième. Elle définit le **quoi**.

#### La pédale ne dit rien sur l'implémentation

L'interface de la pédale ne vous donne aucune information sur la mécanique interne du véhicule. Le moteur est-il à essence, diesel, électrique? L'accélération est-elle gérée par un carburateur ou une injection électronique? Tous ces détails sont cachés au conducteur. De même, l'ISA ne spécifie pas comment l'addition est réalisée.

#### La stabilité de l'interface

Cette analogie explique pourquoi il est si difficile de changer un ISA une fois qu'il est établi. Imaginez que l'on décide demain que pour accélérer, il faut tirer la pédale au lieu de la pousser. Tous les conducteurs (les programmeurs) devraient réapprendre, et toutes les voitures existantes (les logiciels) deviendraient inutilisables ou nécessiteraient une modification coûteuse.

Cette stabilité de l'ISA est en réalité une caractéristique essentielle qui favorise le progrès technologique. Elle agit comme une couche d'abstraction stable qui permet aux ingénieurs matériels d'innover de manière agressive sur l'implémentation (passer d'un processeur simple cœur à un processeur multi-cœur, introduire des pipelines, changer les technologies de fabrication) sans pour autant casser l'immense écosystème de logiciels existants. La rigidité de l'ISA est donc un pilier de la rétrocompatibilité, qui a fait le succès de plateformes comme le x86.

### 1.3. La Mécanique Interne : La Microarchitecture

Si l'ISA est le **quoi**, la microarchitecture est le **comment**. Elle représente l'implémentation concrète de l'ISA, conçue pour répondre à des contraintes et des objectifs spécifiques, tels que le coût, la performance ou la consommation d'énergie. On peut la définir comme "tout ce qui est fait dans le matériel sans être exposé au logiciel".

La microarchitecture englobe une multitude de techniques et de concepts, dont beaucoup sont invisibles pour le programmeur :

- Le **pipelining**, qui s'apparente à une chaîne de montage pour les instructions
- L'**exécution dans le désordre** (out-of-order execution), qui permet au processeur de réorganiser les instructions pour éviter les temps morts
- L'**exécution spéculative**, où le processeur devine le résultat d'une condition et commence à travailler en avance
- La gestion de la hiérarchie mémoire, la prédiction de branchement, et bien d'autres techniques que nous explorerons

### 1.4. Délimiter la Frontière : ISA vs. Microarchitecture en Pratique

Pour solidifier votre compréhension, déterminons ensemble si les éléments suivants appartiennent à l'ISA ou à la microarchitecture.

| Élément | ISA ou Microarchitecture | Justification |
|---------|-------------------------|---------------|
| **L'opcode de l'instruction add** | ISA | Le programmeur assembleur ou le compilateur doit connaître ce code binaire pour dire à la machine d'effectuer une addition. C'est défini dans le contrat. |
| **Le type d'additionneur dans l'ALU** | Microarchitecture | C'est un détail d'implémentation qui affecte la vitesse et le coût de l'addition, mais le résultat sémantique reste le même. Ce n'est pas exposé au programmeur. |
| **Le nombre de registres généraux** | ISA | Le programmeur doit savoir combien de registres sont disponibles (par exemple, 32 pour l'ISA MIPS) et comment les adresser. |
| **Le nombre de cycles pour une multiplication** | Microarchitecture | Dans la plupart des processeurs modernes, la latence d'une instruction est une caractéristique de performance de l'implémentation, non une garantie de l'ISA. |
| **Le nombre de ports du banc de registres** | Microarchitecture | L'ISA spécifie ce que l'on peut faire avec les registres, mais le nombre de ports détermine comment et à quelle vitesse on peut y accéder. |
| **L'exécution pipeline des instructions** | Microarchitecture | C'est une technique d'optimisation de performance totalement transparente pour le programmeur. |
| **Le Compteur de Programme (PC)** | ISA | Le PC est un registre visible et manipulable par le programmeur via les instructions de contrôle de flux. |

Il est important de noter que cette frontière n'est pas gravée dans le marbre. L'architecte a le pouvoir de déplacer des fonctionnalités d'un côté à l'autre de cette ligne. Le positionnement de cette frontière est donc un choix de conception actif avec des conséquences profondes sur l'ensemble du système informatique.

## Partie 2 : Principes de Conception : L'Art et la Science de l'Architecture

### 2.1. Le "Point de Conception" (Design Point)

La conception d'un système informatique n'est jamais un exercice abstrait. Elle est guidée par ce que l'on appelle le **"point de conception"** (design point) : un ensemble d'objectifs et de contraintes, ainsi que leur importance relative, qui orientent l'ensemble du processus.

Ce point de conception est défini par plusieurs métriques ou considérations clés :

- **Performance** : La vitesse d'exécution
- **Coût** : Le coût de fabrication
- **Consommation d'énergie** (Power & Energy) : L'efficacité énergétique
- **Time-to-Market** : La rapidité de mise sur le marché
- **Sécurité et Fiabilité** (Security & Safety) : La robustesse du système
- **Prédictibilité** : La constance du comportement

Ces considérations sont elles-mêmes dictées par des facteurs externes : l'espace applicatif visé (par exemple, le calcul haute performance, l'intelligence artificielle, les systèmes embarqués), l'utilisateur final et, bien sûr, les limites de la technologie disponible à un instant T.

### 2.2. L'Équation des Compromis (Trade-offs)

Le point de conception nous force inévitablement à faire des **compromis** (trade-offs). On ne peut pas tout optimiser simultanément. Vouloir concevoir un accélérateur spécialisé pour le machine learning, par exemple, implique une série de compromis : on choisira probablement un ISA non généraliste mais optimisé pour les opérations matricielles, et une microarchitecture massivement parallèle, peut-être au détriment de la performance sur des tâches séquentielles.

Ces compromis se manifestent à tous les niveaux :

- **Niveau ISA** : Simple ou complexe? Fixe ou variable?
- **Niveau Microarchitecture** : Rapide mais énergivore? Simple mais lent?
- **Niveau Système** : Comment diviser la charge de travail entre le matériel et le logiciel? C'est le fameux problème du "fossé sémantique" (semantic gap).

Il est crucial de comprendre que ces considérations de conception sont profondément interconnectées. Une décision visant à maximiser la performance (par exemple, en augmentant la fréquence d'horloge) aura presque toujours un impact négatif sur la consommation d'énergie et le coût (nécessité d'un système de refroidissement plus complexe). Une pression pour réduire le time-to-market pourrait compromettre la sécurité en réduisant le temps alloué à la validation. L'architecture informatique est donc un exercice d'équilibriste de haute volée, où chaque décision engendre une onde de choc à travers l'ensemble du point de conception.

### 2.3. L'Architecture : Une Science et un Art

On affirme souvent que l'architecture est autant un art qu'une science, et c'est une vérité profonde. C'est une **science** car elle repose sur des principes quantifiables et des modélisations rigoureuses. Mais c'est un **art** car nous ne pouvons pas tout modéliser parfaitement, et surtout, nous ne pouvons pas prédire l'avenir.

- Les applications de demain sont inconnues. Un design optimisé pour les charges de travail d'aujourd'hui pourrait être totalement inadapté pour celles de demain.
- Les avancées technologiques peuvent rendre obsolètes les décisions de conception actuelles.
- Les besoins des utilisateurs évoluent constamment.

Cette incertitude fondamentale exige de l'architecte une forme d'"intuition artistique" pour prendre des décisions robustes, capables de résister à l'épreuve du temps. Les exemples de "macro-architecture" sont parlants : une brasserie du 19ème siècle transformée en entrepôt frigorifique, une centrale électrique devenue un café, ou une église reconvertie en brasserie. Ces bâtiments ont été réutilisés pour des fonctions que leurs concepteurs originaux n'auraient jamais pu imaginer. Il en va de même pour la microarchitecture.

Un exemple concret de cette problématique est l'instruction de "branchement retardé" (delayed branch) de l'ISA MIPS. Les concepteurs originaux ont fait un choix qui semblait scientifiquement optimal pour leur microarchitecture de l'époque (un pipeline simple). Ils ont encodé cette optimisation directement dans l'ISA, violant ainsi la barrière d'abstraction. Lorsque la technologie a évolué et que les pipelines sont devenus plus complexes, cette caractéristique de l'ISA est devenue un goulot d'étranglement et un véritable casse-tête pour les architectes. C'est une leçon puissante : un choix de conception qui manque de prévoyance artistique peut se transformer en un échec d'ingénierie à long terme.

## Partie 3 : Modèles de Processeur : Monocycle vs. Multicycle

### 3.1. Le Traitement d'une Instruction : Une Transition d'État

Comment une machine traite-t-elle une instruction? Fondamentalement, une instruction est un **transformateur**. Elle prend un état architectural donné (AS), qui inclut le contenu des registres, de la mémoire et du PC, et le transforme en un nouvel état architectural (AS').

Du point de vue de l'ISA, ce processus est une transition d'état atomique et abstraite, à l'image d'une machine à états finis où chaque instruction provoque une seule transition. Cependant, la microarchitecture peut implémenter cette transition de différentes manières.

### 3.2. Le Modèle Monocycle (Single-Cycle)

Le modèle le plus simple est le processeur **monocycle**.

- **Définition** : Chaque instruction, quelle que soit sa complexité, s'exécute en exactement un seul cycle d'horloge.
- **Implémentation** : L'exécution repose sur de la logique purement combinatoire. Toutes les mises à jour de l'état architectural (écriture dans un registre ou en mémoire) se produisent de manière synchrone à la fin de ce long cycle d'horloge.
- **Problème Fondamental** : La durée du cycle d'horloge (et donc la fréquence maximale du processeur) est dictée par le chemin critique de l'instruction la plus lente de tout l'ISA. Même une instruction très simple comme une addition doit attendre la fin du long cycle d'horloge dimensionné pour une instruction complexe comme un chargement depuis la mémoire.

### 3.3. Le Modèle Multicycle (Multi-Cycle)

Pour surmonter cette limitation, le modèle **multicycle** a été introduit.

- **Définition** : Le traitement d'une instruction est décomposé en plusieurs étapes plus petites, chacune prenant un cycle d'horloge (plus court).
- **Implémentation** : Ce modèle utilise un état microarchitectural (des registres temporaires invisibles pour le programmeur) qui est mis à jour à chaque cycle intermédiaire. L'état architectural final (AS') n'est mis à jour qu'à la toute fin de l'exécution de l'instruction, préservant ainsi la sémantique de l'ISA.
- **Avantage Fondamental** : La durée du cycle d'horloge est maintenant déterminée par l'étape la plus lente, et non plus par l'instruction la plus lente. Cela permet d'avoir un cycle d'horloge beaucoup plus court et donc une fréquence plus élevée.

### 3.4. Analyse de Performance : CPI et Temps d'Exécution

Pour comparer ces modèles, nous utilisons l'équation de performance fondamentale :

```
Temps d'exécution = Nombre d'Instructions × CPI × Temps du Cycle d'Horloge
```

Où le **CPI (Cycles Par Instruction)** est le nombre moyen de cycles nécessaires pour exécuter une instruction.

- **Monocycle** : Par définition, le CPI est toujours de 1. Cependant, le temps du cycle d'horloge est très long.
- **Multicycle** : Le CPI est variable (différent pour chaque instruction) et en moyenne supérieur à 1. En contrepartie, le temps du cycle d'horloge peut être très court.

Cela met en lumière un compromis fondamental. Un étudiant pourrait penser qu'il suffit d'optimiser chaque terme de l'équation indépendamment. Cependant, le CPI et le temps de cycle sont presque toujours inversement liés. En décomposant une instruction en plus d'étapes (stages) pour réduire le temps de cycle, on augmente presque inévitablement le nombre de cycles nécessaires pour cette instruction (son CPI). L'art de la microarchitecture consiste à trouver le "point idéal" dans ce compromis pour minimiser le produit `CPI × Temps du Cycle`, qui détermine la performance réelle. Le modèle multicycle offre à l'architecte davantage de leviers pour trouver cet équilibre optimal.

Nous allons maintenant nous concentrer sur la construction d'un processeur monocycle. Bien qu'il soit inefficace, ce modèle est un outil pédagogique exceptionnel. En le construisant de A à Z, nous serons forcés de faire face à chaque exigence du chemin de données et du contrôle pour un ISA complet. Ses défauts mêmes (comme la nécessité de plusieurs additionneurs ou son cycle d'horloge long) deviendront les arguments les plus puissants en faveur des architectures plus complexes et plus performantes que nous étudierons par la suite.

## Partie 4 : Construction d'un Processeur Monocycle : Le Chemin de Données (Datapath)

### 4.1. Composants de Base : Unité de Contrôle et Chemin de Données

Un moteur de traitement d'instructions se divise en deux composantes principales :

1. **Le Chemin de Données (Datapath)** : C'est l'ensemble des éléments matériels qui stockent, manipulent et transportent les données. Il comprend les unités fonctionnelles (comme l'ALU), les unités de stockage (banc de registres, mémoire) et les structures de connexion (multiplexeurs, fils).

2. **La Logique de Contrôle (Control Logic)** : C'est le "cerveau" qui génère les signaux de contrôle pour orchestrer les opérations du datapath, en s'assurant que les données circulent correctement conformément à la sémantique de l'instruction en cours d'exécution.

Notre approche de conception sera de construire le datapath en premier, car la logique de contrôle dépend intrinsèquement de la structure du datapath qu'elle doit piloter.

### 4.2. Étape 1 : Les Éléments d'État

Nous commençons par définir les éléments qui contiennent l'état architectural de notre processeur MIPS simplifié :

- **Le Compteur de Programme (PC)** : Un registre 32 bits qui contient l'adresse de l'instruction à exécuter.
- **La Mémoire d'Instructions** : Une mémoire en lecture seule qui, à partir d'une adresse fournie par le PC, délivre une instruction de 32 bits.
- **Le Banc de Registres (Register File)** : Contient les 32 registres généraux de 32 bits. Il dispose de deux ports de lecture (pour lire deux registres simultanément) et d'un port d'écriture (pour écrire dans un registre).
- **La Mémoire de Données** : Une mémoire qui peut être lue ou écrite.

### 4.3. Étape 2 : Instructions Arithmétiques et Logiques (Type-R)

Construisons le datapath nécessaire pour une instruction de type-R, comme `add rd, rs, rt`. La sémantique est : `Registre[rd] = Registre[rs] + Registre[rt]`.

Le flux de données est le suivant :

1. Le PC fournit l'adresse à la Mémoire d'Instructions pour récupérer l'instruction.
2. Les champs rs et rt de l'instruction sont utilisés comme adresses pour les deux ports de lecture du Banc de Registres.
3. Les deux valeurs lues des registres sont envoyées aux entrées de l'ALU.
4. L'ALU effectue l'addition.
5. Le résultat de l'ALU est envoyé au port d'écriture du Banc de Registres.
6. Le champ rd de l'instruction est utilisé comme adresse pour le port d'écriture.
7. En parallèle, un additionneur dédié calcule PC+4 pour préparer l'adresse de la prochaine instruction.

```pseudocode
Algorithme 1 : Exécution d'une Instruction de Type-R
// Sémantique de l'instruction: R[rd] <- R[rs] op R[rt]
// PC est l'adresse de l'instruction courante

instruction = InstructionMemory[PC]

// Décodage et lecture des opérandes
val_rs = RegisterFile[instruction.rs]
val_rt = RegisterFile[instruction.rt]

// Exécution de l'opération
resultat_ALU = val_rs + val_rt // Pour une instruction 'add'

// Écriture du résultat
RegisterFile[instruction.rd] = resultat_ALU

// Mise à jour du PC
PC = PC + 4
```

### 4.4. Étape 3 : Intégration des Opérandes Immédiats (Type-I)

Modifions maintenant le datapath pour supporter les instructions de type-I, comme `addi rt, rs, immediate`. La sémantique est : `Registre[rt] = Registre[rs] + SignExtend(immediate)`.

Cela nécessite deux ajouts majeurs :

1. **Une unité d'extension de signe (Sign-Extend)** qui convertit l'opérande immédiat de 16 bits en une valeur de 32 bits.
2. **Un multiplexeur (nommé ALUSrc)** à la deuxième entrée de l'ALU. Ce multiplexeur choisit entre la valeur lue du second registre (pour les instructions de type-R) et la valeur immédiate étendue (pour les instructions de type-I).
3. **Un second multiplexeur (nommé RegDst)** pour l'adresse du registre de destination. Pour le type-R, c'est le champ rd (bits 15-11), mais pour le type-I, c'est le champ rt (bits 20-16).

```pseudocode
Algorithme 2 : Exécution d'une Instruction ALU de Type-I
// Sémantique de l'instruction: R[rt] <- R[rs] + SignExtend(immediate)
// PC est l'adresse de l'instruction courante

instruction = InstructionMemory[PC]

// Décodage et lecture des opérandes
val_rs = RegisterFile[instruction.rs]
immediate_etendu = SignExtend(instruction.immediate_16bit)

// Exécution de l'opération
resultat_ALU = val_rs + immediate_etendu

// Écriture du résultat
RegisterFile[instruction.rt] = resultat_ALU

// Mise à jour du PC
PC = PC + 4
```

### 4.5. Étape 4 : Accès à la Mémoire (Load et Store)

Nous étendons maintenant le datapath pour les instructions load word (lw) et store word (sw). Les deux calculent une adresse mémoire via `adresse = Registre[rs] + SignExtend(offset)`.

- **Pour lw** : La donnée lue de la Mémoire de Données à l'adresse calculée doit être écrite dans le registre rt.
- **Pour sw** : La donnée contenue dans le registre rt doit être écrite dans la Mémoire de Données à l'adresse calculée.

La modification clé est l'ajout d'un nouveau multiplexeur (nommé **MemToReg**), placé avant le port d'écriture du banc de registres. Il sélectionne la source des données à écrire : soit le résultat de l'ALU (pour les instructions arithmétiques), soit la donnée lue de la mémoire (pour l'instruction lw).

En observant ce processus de construction, on constate que le datapath n'est pas un assemblage arbitraire de composants. Il est la manifestation physique et concrète de la spécification de l'ISA. Le multiplexeur MemToReg existe parce que l'ISA a à la fois des instructions ALU et des instructions de chargement qui écrivent dans les registres. Le multiplexeur ALUSrc existe parce que l'ISA a des formats d'instructions registre-registre et registre-immédiat.

```pseudocode
Algorithme 3 : Exécution de load word
// Sémantique de l'instruction: R[rt] <- Memory[R[rs] + SignExtend(offset)]
// PC est l'adresse de l'instruction courante

instruction = InstructionMemory[PC]

// Lecture de l'opérande de base
val_base = RegisterFile[instruction.rs]
offset = SignExtend(instruction.immediate_16bit)

// Calcul de l'adresse
adresse_memoire = val_base + offset

// Lecture depuis la mémoire
donnees_lues = DataMemory[adresse_memoire]

// Écriture dans le registre de destination
RegisterFile[instruction.rt] = donnees_lues

// Mise à jour du PC
PC = PC + 4
```

```pseudocode
Algorithme 4 : Exécution de store word
// Sémantique de l'instruction: Memory[R[rs] + SignExtend(offset)] <- R[rt]
// PC est l'adresse de l'instruction courante

instruction = InstructionMemory[PC]

// Lecture des opérandes
val_base = RegisterFile[instruction.rs]
val_a_stocker = RegisterFile[instruction.rt]
offset = SignExtend(instruction.immediate_16bit)

// Calcul de l'adresse
adresse_memoire = val_base + offset

// Écriture en mémoire
DataMemory[adresse_memoire] = val_a_stocker

// Mise à jour du PC
PC = PC + 4
```

### 4.6. Étape 5 : Instructions de Contrôle de Flux (Jump et Branch)

Enfin, nous ajoutons le support pour les instructions de contrôle de flux.

#### Pour jump (j)

L'adresse de saut est calculée par concaténation : 
```
AdresseCible = (PC+4)[31:28] || instruction[25:0] || "00"
```

Un multiplexeur est ajouté avant le PC pour choisir entre PC+4 et cette nouvelle adresse cible.

#### Pour branch if equal (beq)

1. **Calcul d'adresse** : L'adresse cible est calculée via `AdresseCible = (PC+4) + (SignExtend(immediate) × 4)`. Cela nécessite un additionneur de branchement dédié car l'ALU principale est occupée.

2. **Évaluation de la condition** : L'ALU principale soustrait les deux registres sources (rs et rt). Si le résultat est zéro (indiqué par sa sortie Zero), les registres sont égaux.

3. **Mise à jour du PC** : La logique du multiplexeur du PC est complexifiée. Il sélectionne l'adresse cible uniquement si une porte ET confirme que l'instruction est un branchement (Branch=1) ET que la condition est vraie (Zero=1).

```pseudocode
Algorithme 5 : Exécution de jump
// Sémantique de l'instruction: PC <- (PC+4)[31:28] || instruction[25:0] || "00"
// PC est l'adresse de l'instruction courante

instruction = InstructionMemory[PC]

PC_plus_4 = PC + 4

// Calcul de l'adresse cible par concaténation
// Note: '||' représente l'opérateur de concaténation
adresse_cible = PC_plus_4[31:28] || instruction.target_26bit || "00"

// Mise à jour du PC
PC = adresse_cible
```

```pseudocode
Algorithme 6 : Exécution de branch if equal
// Sémantique: SI (R[rs] == R[rt]) ALORS PC <- PC+4 + SignExtend(offset)*4
// PC est l'adresse de l'instruction courante

instruction = InstructionMemory[PC]

// Lecture des opérandes pour la comparaison
val_rs = RegisterFile[instruction.rs]
val_rt = RegisterFile[instruction.rt]

// Calcul de l'adresse de la prochaine instruction séquentielle
PC_plus_4 = PC + 4

// Calcul de l'adresse cible du branchement
offset = SignExtend(instruction.immediate_16bit) * 4
adresse_cible = PC_plus_4 + offset

// Décision et mise à jour du PC
SI (val_rs == val_rt) ALORS
    PC = adresse_cible  // Le branchement est pris
SINON
    PC = PC_plus_4      // Le branchement n'est pas pris
FIN SI
```

## Partie 5 : Le Cerveau de l'Opération : L'Unité de Contrôle

### 5.1. Le Rôle de l'Unité de Contrôle

Nous avons construit un datapath capable d'exécuter différentes instructions, mais il est pour l'instant inerte. Le rôle de l'unité de contrôle est de lui donner vie en générant les signaux binaires (0 ou 1) qui vont commander les multiplexeurs, activer les écritures en mémoire ou dans les registres, et dicter l'opération de l'ALU.

Le principe **"Ne pas nuire" (Do No Harm)** est ici fondamental. Le datapath que nous avons construit est capable de faire beaucoup de choses en même temps : il pourrait tenter d'écrire dans un registre, d'écrire en mémoire et de modifier le PC simultanément. Une telle capacité au chaos rend le rôle de l'unité de contrôle absolument critique. Le datapath fournit le potentiel, mais le contrôle fournit la discipline. Il garantit que pour une instruction donnée, seuls les chemins de données corrects sont activés et que toutes les autres opérations susceptibles de modifier l'état sont désactivées (par exemple, en mettant RegWrite et MemWrite à 0). Sans un contrôle correct, le datapath est une machine destructive qui corromprait son propre état.

### 5.2. Conception d'une Unité de Contrôle Câblée (Hardwired)

Pour un processeur monocycle, l'unité de contrôle est un simple bloc de logique combinatoire. Ses entrées sont principalement les bits de l'opcode de l'instruction (et parfois le champ funct pour les instructions de type-R), et ses sorties sont tous les signaux de contrôle nécessaires.

Voici la logique pour quelques signaux clés :

- **RegWrite** : Ce signal est à 1 pour les instructions qui écrivent un résultat dans le banc de registres (type-R, addi, lw). Il est à 0 pour les autres (sw, beq, j).
- **MemRead** : À 1 uniquement pour l'instruction lw.
- **MemWrite** : À 1 uniquement pour l'instruction sw.
- **ALUSrc** : À 1 si le second opérande de l'ALU est l'immédiat étendu (pour addi, lw, sw). À 0 s'il provient du banc de registres (pour type-R, beq).
- **Branch** : À 1 uniquement pour l'instruction beq. Ce signal, combiné à la sortie Zero de l'ALU, contrôle la prise de branchement.

### 5.3. Tableau de Synthèse des Signaux de Contrôle

La manière la plus claire de spécifier la logique de contrôle est d'utiliser une table de vérité. Ce tableau résume la valeur de chaque signal de contrôle pour les principaux types d'instructions. 'X' signifie que la valeur du signal n'a pas d'importance (Don't Care) pour cette instruction, car son chemin n'est pas utilisé pour une opération critique.

| Instruction | RegDst | ALUSrc | MemToReg | RegWrite | MemRead | MemWrite | Branch | Jump | ALUOp |
|-------------|--------|--------|----------|----------|---------|----------|--------|------|-------|
| R-type      | 1      | 0      | 0        | 1        | 0       | 0        | 0      | 0    | 10    |
| lw          | 0      | 1      | 1        | 1        | 1       | 0        | 0      | 0    | 00    |
| sw          | X      | 1      | X        | 0        | 0       | 1        | 0      | 0    | 00    |
| beq         | X      | 0      | X        | 0        | 0       | 0        | 1      | 0    | 01    |
| j           | X      | X      | X        | 0        | 0       | 0        | 0      | 1    | XX    |

**Légende ALUOp** : 00 = add, 01 = subtract, 10 = déterminé par le champ funct.

### 5.4. Alternative : Le Contrôle Microcodé

Une alternative à la logique câblée (hardwired) est le **contrôle microcodé**. Au lieu de générer les signaux avec des portes logiques, on les stocke dans une mémoire spéciale appelée "magasin de contrôle" (control store). L'opcode de l'instruction sert alors d'adresse pour lire une "microligne" contenant toutes les valeurs des signaux de contrôle pour cette instruction. C'est une approche plus flexible mais souvent plus lente, que nous explorerons plus en détail dans le contexte des processeurs multicycles.

## Partie 6 : Conclusion et Évaluation du Modèle Monocycle

### 6.1. Synthèse du Processeur Monocycle Complet

Nous avons réussi. En partant d'une page blanche, nous avons assemblé un chemin de données et une unité de contrôle pour créer un processeur monocycle complet et fonctionnel, capable d'exécuter un sous-ensemble significatif d'instructions MIPS : arithmétiques, logiques, d'accès mémoire et de contrôle de flux. En fonction de l'instruction décodée, l'unité de contrôle active les bons chemins dans le datapath pour orchestrer le flux correct des données, de la lecture des opérandes jusqu'à l'écriture du résultat.

### 6.2. L'Inconvénient Critique du Modèle Monocycle

Cependant, notre création souffre d'un défaut majeur et rédhibitoire : **sa performance est médiocre**. Le cycle d'horloge unique doit être suffisamment long pour accommoder l'instruction la plus lente. Dans notre cas, il s'agit de l'instruction load word (lw), car elle traverse séquentiellement presque tous les composants majeurs : mémoire d'instructions, banc de registres, ALU (pour le calcul d'adresse), mémoire de données, et enfin le multiplexeur MemToReg pour revenir au banc de registres.

Cela signifie que des instructions beaucoup plus rapides, comme une simple addition (qui n'utilise pas la mémoire de données), sont pénalisées. Elles terminent leur travail bien avant la fin du cycle d'horloge et doivent attendre passivement, ce qui constitue un gaspillage de temps considérable.

### 6.3. Vers des Architectures Plus Performantes

La critique du modèle monocycle nous fournit la motivation directe pour l'étape suivante de notre étude. Pour résoudre ce problème de performance, il est impératif de rompre avec le paradigme "une instruction, un cycle". Nous devons trouver un moyen de permettre aux instructions rapides de prendre moins de temps que les instructions lentes.

Cela nous conduit naturellement vers l'étude des **processeurs multicycles**, où chaque instruction prend un nombre de cycles adapté à sa complexité, et ultimement, vers les **processeurs pipelinés**, qui appliquent les principes de la chaîne de montage pour traiter plusieurs instructions simultanément. Ce sera l'objet de nos prochains cours.
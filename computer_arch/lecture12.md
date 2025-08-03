# Architecture des Ordinateurs : Le Pipeline, des Principes Fondamentaux à l'Implémentation Détaillée

## I. Introduction : L'Impératif de Performance et la Naissance du Pipeline

### 1.1 Les Limites des Architectures Séquentielles

Au cours de nos précédentes discussions, nous avons exploré les architectures de processeurs monocycle et multicycle. Le processeur monocycle, bien que simple dans sa conception, souffre d'une inefficacité fondamentale : son temps de cycle d'horloge est dicté par la durée de l'instruction la plus lente, même si la majorité des instructions sont plus rapides. Pour pallier cette lacune, nous avons développé l'architecture multicycle, qui décompose chaque instruction en plusieurs étapes plus courtes, permettant à chaque type d'instruction de n'utiliser que le nombre de cycles strictement nécessaire. Cette approche améliore l'utilisation des ressources par rapport au modèle monocycle.

Cependant, même l'architecture multicycle reste intrinsèquement séquentielle. À chaque instant, le processeur dans son ensemble est dédié au traitement d'une seule et unique instruction. Cette limitation conduit à un sous-emploi flagrant des ressources matérielles. Par exemple, pendant qu'une instruction load accède à la mémoire de données (étape MEM), l'unité de recherche d'instruction (Fetch), l'unité de décodage (Decode) et même l'unité arithmétique et logique (ALU) sont largement inactives. Cette observation d'un matériel coûteux mais oisif est le point de départ de notre quête pour une performance accrue. La question fondamentale qui se pose est la suivante : comment pouvons-nous exploiter cette concurrence potentielle et faire travailler simultanément les différentes parties du processeur sur des instructions différentes ?

### 1.2 Le Principe de Concurrence : Vers le Débit d'Instructions (Throughput)

La réponse à cette question réside dans l'exploitation du parallélisme au niveau de l'instruction (Instruction-Level Parallelism, ILP). L'idée maîtresse est de superposer l'exécution de plusieurs instructions, à la manière d'une chaîne de montage. Pendant qu'une instruction est en cours d'exécution, la suivante est en cours de décodage, et celle d'après est en cours de recherche. Ce principe est connu sous le nom de **pipelining**.

Pour analyser l'impact du pipelining, il est crucial de distinguer deux métriques de performance fondamentales :

- **La Latence** : Le temps total nécessaire pour exécuter une seule instruction, du début à la fin.
- **Le Débit (Throughput)** : Le nombre d'instructions terminées par unité de temps (par exemple, par cycle d'horloge).

Le pipelining n'améliore pas, et peut même légèrement dégrader, la latence d'une instruction individuelle en raison de la surcharge introduite par les registres de pipeline. Son objectif est d'augmenter drastiquement le débit. Dans un pipeline idéal à $k$ étages, une fois que le pipeline est rempli, il termine une instruction à chaque cycle d'horloge, multipliant ainsi le débit par un facteur proche de $k$.

L'adoption de cette technique représente une violation délibérée et contrôlée du modèle de von Neumann au niveau de la microarchitecture. Si le programmeur voit toujours un modèle d'exécution séquentiel (une instruction après l'autre), le matériel, lui, exécute des morceaux de plusieurs instructions en parallèle.

### 1.3 Contexte Historique : Le Projet IBM Stretch, un "Échec" Visionnaire

Le concept de pipelining n'est pas une invention récente. L'un de ses premiers et plus ambitieux ambassadeurs fut l'ordinateur IBM 7030, surnommé "Stretch", développé à la fin des années 1950 et livré en 1961. Le projet, mené en collaboration avec le Los Alamos Scientific Laboratory, visait à construire un supercalculateur 100 fois plus rapide que la machine scientifique la plus performante de l'époque, l'IBM 704.

Commercialement, le projet fut un échec retentissant. La machine n'atteignit qu'une performance de 30 à 40 fois celle du 704, bien en deçà de la promesse marketing. IBM fut contraint de réduire drastiquement son prix de 13.5 millions de dollars à 7.78 millions de dollars pour les clients engagés et de retirer le produit de la vente, perdant de l'argent sur chaque unité vendue. Le projet Stretch est souvent cité comme un exemple de défaillance en gestion de projet.

Cependant, juger Stretch sur ses seuls résultats commerciaux serait une erreur historique. Techniquement, ce fut un succès visionnaire. Les concepts qu'il a pionniers et validés, tels que le pipelining, la recherche anticipée d'instructions (prefetching), le décodage anticipé, l'entrelacement mémoire (memory interleaving) et la multiprogrammation, sont devenus les piliers de l'architecture informatique moderne. L'expérience acquise sur Stretch, tant au niveau de la conception que de la fabrication, a directement nourri le projet de l'IBM System/360, la famille d'ordinateurs la plus réussie commercialement de tous les temps. Des architectes de renom comme Fred Brooks ont été formés sur ce projet. Ainsi, l'échec commercial de Stretch peut être réinterprété comme un investissement fondamental en recherche et développement, une étape coûteuse mais nécessaire qui a prouvé la viabilité de concepts architecturaux qui définissent encore aujourd'hui nos microprocesseurs, des serveurs aux smartphones.

Pour comprendre la philosophie de conception de cette machine pionnière, l'ouvrage de Werner Buchholz, *Planning a Computer System: Project Stretch* (1962), reste une référence académique incontournable.

## II. Le Pipeline Idéal : Analogies et Principes Fondamentaux

Pour construire une intuition solide du fonctionnement d'un pipeline, il est utile de recourir à des analogies du monde réel avant de plonger dans les détails techniques.

### 2.1 L'Analogie de la Ligne d'Assemblage

#### La Buanderie Pipelinisée

Imaginons une corvée de lessive composée de quatre étapes distinctes : laver (30 min), sécher (30 min), plier (30 min) et ranger (30 min). Pour une seule brassée de linge, la latence totale est de 2 heures. Si nous avons quatre brassées à faire et que nous les traitons de manière séquentielle (multicycle), la tâche entière prendra $4 \times 2 = 8$ heures.

Cependant, les ressources (lave-linge, sèche-linge, table de pliage) sont indépendantes. Dès que la première brassée passe du lave-linge au sèche-linge, le lave-linge est libre. Nous pouvons donc y placer la deuxième brassée. En appliquant ce principe de chevauchement, une fois le "pipeline" de la buanderie rempli, une brassée de linge propre est terminée toutes les 30 minutes. Les quatre brassées sont terminées en seulement 3.5 heures, et non 8. La latence par brassée reste de 2 heures, mais le débit a été multiplié par quatre.

Cette analogie met en lumière deux concepts cruciaux :

**Le Goulot d'Étranglement (Bottleneck)** : Si notre sèche-linge est lent et prend 60 minutes au lieu de 30, il devient le goulot d'étranglement. Le débit de l'ensemble de la chaîne est alors dicté par cette étape la plus lente : nous ne pourrons terminer une brassée que toutes les 60 minutes. La performance d'un pipeline est toujours limitée par son étage le plus lent.

**La Réplication des Ressources** : Pour résoudre le problème du sèche-linge lent, deux solutions s'offrent à nous : acheter un sèche-linge plus rapide (améliorer la technologie d'un étage) ou acheter un deuxième sèche-linge et alterner les brassées entre les deux (répliquer la ressource). Cette deuxième option illustre un compromis coût/performance fondamental en architecture : l'ajout de matériel pour augmenter le parallélisme et le débit.

#### L'Usine Ford

L'exemple historique de l'assemblage du "Magneto" chez Ford en 1913 est une autre illustration puissante. Avant la mise en place de la chaîne de montage, un ouvrier qualifié assemblait un Magneto en 15 minutes. En décomposant le processus en 29 étapes distinctes sur une chaîne, le temps d'assemblage par unité a été réduit de manière spectaculaire, augmentant le débit de production par un facteur de trois.

### 2.2 Les Trois Propriétés d'un Pipeline Idéal et la Dure Réalité

Ces analogies fonctionnent si bien parce qu'elles se rapprochent d'un pipeline idéal. Formellement, un pipeline idéal possède trois propriétés :

1. **Opérations Identiques** : Toutes les tâches qui traversent le pipeline sont identiques et suivent exactement les mêmes étapes.
2. **Opérations Indépendantes** : Les tâches sont totalement indépendantes les unes des autres. Le traitement de la brassée N ne dépend en rien du traitement de la brassée N-1.
3. **Partitionnement Uniforme** : Le travail total est divisé en étapes de durée parfaitement égale.

Le traitement d'un flux d'instructions dans un processeur est une application fondamentalement imparfaite de ce concept idéal, car il viole chacune de ces trois propriétés :

- **Les instructions ne sont pas identiques** : Une instruction `add` n'utilise pas l'étage mémoire, tandis qu'une instruction `lw` l'utilise intensivement. Forcer toutes les instructions à passer par tous les étages crée une inefficacité appelée **fragmentation externe** : des étages sont inactifs pour certaines instructions.

- **Les étages ne sont pas uniformes** : Il est extrêmement difficile de diviser le travail d'une instruction en étapes de durée égale. L'accès à la mémoire peut être beaucoup plus long que l'addition de deux registres. Comme le temps de cycle du pipeline est dicté par l'étage le plus lent, les étages plus rapides passent une partie de leur temps à attendre. C'est la **fragmentation interne**.

- **Les instructions ne sont pas indépendantes** : C'est la violation la plus critique. Une instruction a souvent besoin du résultat d'une instruction précédente qui n'a pas encore terminé son parcours dans le pipeline. Cette **dépendance de données** est la source des **aléas de données**, le défi majeur de la conception des pipelines.

La majorité de la complexité de la conception d'un processeur pipeliné ne réside donc pas dans l'application du principe de pipeline lui-même, mais dans la gestion des écarts entre le modèle idéal et la réalité des programmes informatiques. Les aléas ne sont pas des défauts du pipeline, mais des conséquences inhérentes à la nature des instructions que nous devons résoudre.

### 2.3 Analyse Quantitative : Calcul du Gain de Performance (Speedup)

Formalisons le gain de performance (speedup). Soit un pipeline à $k$ étages, où chaque étage prend un temps de cycle $T_{\text{cycle,pipe}}$. Pour exécuter $N$ instructions :

**Temps d'exécution non-pipeliné (multicycle)** : Supposons une moyenne de $k$ cycles par instruction avec un temps de cycle $T_{\text{cycle,multi}}$. Le temps total est $T_{\text{non-pipe}} = N \times k \times T_{\text{cycle,multi}}$. Pour simplifier, si $T_{\text{cycle,multi}} = T_{\text{cycle,pipe}}$, alors $T_{\text{non-pipe}} = N \times k \times T_{\text{cycle,pipe}}$.

**Temps d'exécution pipeliné** : Il faut $k-1$ cycles pour remplir le pipeline, puis $N$ cycles pour que les $N$ instructions le traversent. Le temps total est $T_{\text{pipe}} = (k-1+N) \times T_{\text{cycle,pipe}}$.

Le speedup est le rapport des temps d'exécution :

$$\text{Speedup} = \frac{T_{\text{non-pipe}}}{T_{\text{pipe}}} = \frac{N \times k \times T_{\text{cycle,pipe}}}{(k - 1 + N) \times T_{\text{cycle,pipe}}} = \frac{N \times k}{k - 1 + N}$$

Lorsque le nombre d'instructions $N$ devient très grand, le terme $k-1$ devient négligeable :

$$\lim_{N\to\infty} \text{Speedup} = \frac{N \times k}{N} = k$$

Le speedup idéal d'un pipeline à $k$ étages est donc de $k$.

Cependant, ce calcul suppose des étages parfaitement équilibrés. Dans la réalité, le temps de cycle du pipeline est déterminé par l'étage le plus lent : $T_{\text{cycle,pipe}} = \max(T_{\text{stage}_i})$. Reprenons l'exemple d'un processeur dont le traitement d'une instruction load sur une machine monocycle prend 800 ps. Si nous le divisons en 5 étages avec des latences respectives de 160, 100, 120, 190, 140 ps, le temps de cycle du pipeline sera de 190 ps. Le speedup réel sera de $800 \text{ ps}/190 \text{ ps} \approx 4.2$, et non 5, à cause de ce déséquilibre.

#### Tableau 1 : Comparaison de Performance Théorique

| Caractéristique | Processeur Monocycle | Processeur Multicycle | Processeur Pipeliné (Idéal) | Processeur Pipeliné (Réaliste) |
|----------------|---------------------|----------------------|----------------------------|-------------------------------|
| Temps de Cycle | 800 ps (dicté par lw) | 200 ps | 160 ps (800/5) | 200 ps (dicté par EX/MEM) |
| Latence par Instruction | 800 ps | 600-1000 ps (3-5 cycles) | 800 ps (5×160) | 1000 ps (5×200) |
| Temps pour 1 instruction | 800 ps | 1000 ps (pour lw) | 800 ps | 1000 ps |
| Temps pour 1000 instr. | 800,000 ps | ~900,000 ps (moyenne) | 160,640 ps ((5+999)×160) | 200,800 ps ((5+999)×200) |
| Débit (régime permanent) | 1.25 GIPS | ~1.1 GIPS | 6.25 GIPS | 5.0 GIPS |

*GIPS = Giga-Instructions Par Seconde. Les valeurs sont basées sur les exemples de la transcription.*

## III. Le Pipeline MIPS à 5 Étages : Une Architecture de Référence

Pour concrétiser ces principes, nous allons étudier l'implémentation d'un pipeline basé sur l'architecture MIPS. Ce choix n'est pas anodin.

### 3.1 Le choix de MIPS : une ISA conçue pour le Pipeline

L'ensemble d'instructions (ISA) MIPS a été conçu dès l'origine avec le pipelining en tête, ce qui simplifie grandement sa mise en œuvre. Ses caractéristiques clés sont :

- **Toutes les instructions ont la même longueur (32 bits)** : Cela rend l'étape de recherche (IF) simple et prédictible. Le processeur n'a pas besoin de décoder une instruction pour savoir où commence la suivante.

- **Formats d'instruction simples et réguliers** : Il n'y a que quelques formats (R, I, J). De plus, les champs des registres sources (rs, rt) sont toujours aux mêmes positions. Cela permet de commencer à lire le banc de registres dès l'étape de décodage (ID), avant même de savoir précisément quelle opération sera effectuée.

- **Architecture Load/Store** : Seules les instructions load et store accèdent à la mémoire. Toutes les opérations arithmétiques et logiques s'effectuent sur les registres. Cela permet de dédier l'étape EX au calcul et l'étape MEM à l'accès mémoire, évitant des séquences plus complexes comme calcul d'adresse → accès mémoire → exécution.

Ces choix de conception contrastent fortement avec les architectures CISC (Complex Instruction Set Computer) comme x86, où les instructions de longueur variable et les modes d'adressage complexes nécessitent une étape de micro-décomposition beaucoup plus lourde avant de pouvoir être pipelinées.

### 3.2 Les Cinq Étages : IF, ID, EX, MEM, WB

Notre pipeline de référence décompose l'exécution d'une instruction en cinq étages :

1. **IF (Instruction Fetch)** : Recherche de l'instruction en mémoire à l'adresse pointée par le Compteur de Programme (PC). Le PC est ensuite incrémenté de 4 pour pointer vers l'instruction suivante.

2. **ID (Instruction Decode)** : Décodage de l'instruction pour déterminer l'opération à effectuer. Simultanément, lecture des valeurs des registres sources (rs et rt) depuis le banc de registres.

3. **EX (Execute)** : Exécution de l'opération. Pour une instruction arithmético-logique, l'ALU effectue le calcul. Pour une instruction load ou store, l'ALU calcule l'adresse mémoire effective. Pour un branchement, l'ALU compare les registres et le PC cible est calculé.

4. **MEM (Memory Access)** : Si l'instruction est un load, lecture de la donnée depuis la mémoire de données. Si c'est un store, écriture de la donnée en mémoire. Les autres instructions traversent cet étage sans rien faire.

5. **WB (Write Back)** : Écriture du résultat (provenant soit de l'ALU, soit de la mémoire) dans le banc de registres, au registre de destination.

### 3.3 Le Chemin de Données (Datapath) et les Registres de Pipeline

Pour que ces cinq étages puissent fonctionner en parallèle sur cinq instructions différentes, il est impératif d'isoler le travail de chaque étage. C'est le rôle des **registres de pipeline**. Ce sont des registres tampons placés entre chaque paire d'étages consécutifs.

- **IF/ID** : Stocke l'instruction fraîchement lue de la mémoire et le PC+4.
- **ID/EX** : Stocke les valeurs lues des registres sources, l'immédiat étendu, les numéros des registres, et les signaux de contrôle.
- **EX/MEM** : Stocke le résultat de l'ALU, la valeur du second registre source (pour store), et les signaux de contrôle pour les étages MEM et WB.
- **MEM/WB** : Stocke la donnée lue de la mémoire (pour load) ou le résultat de l'ALU, et les signaux de contrôle pour l'étage WB.

À chaque coup d'horloge, les données d'une instruction avancent d'un registre de pipeline au suivant. Ces registres sont la matérialisation physique du pipeline : ils contiennent l'état complet de chaque instruction à chaque étape de son traitement. Sans eux, les données des différentes instructions se mélangeraient, rendant le concept inopérant.

### 3.4 Propagation des Signaux de Contrôle à travers le Pipeline

Tout comme les données, les signaux de contrôle qui dictent le comportement du chemin de données (par exemple, RegWrite, ALUSrc, MemRead) doivent être synchronisés avec l'instruction qu'ils gouvernent. Dans notre pipeline, l'unité de contrôle principale, qui est une simple logique combinatoire fonction de l'opcode de l'instruction, est située dans l'étage ID.

Une fois générés, ces signaux de contrôle sont propagés à travers les registres de pipeline avec le reste des données de l'instruction. Un signal de contrôle n'est "consommé" que dans l'étage où il est pertinent. Par exemple, le signal MemWrite est généré en ID, mais il ne sera utilisé pour activer l'écriture en mémoire que lorsque l'instruction atteindra l'étage MEM. Cette propagation garantit que la bonne opération est effectuée au bon moment pour la bonne instruction.

#### Tableau 2 : Propagation et Utilisation des Signaux de Contrôle MIPS

| Signal de Contrôle | Généré en | Utilisé en EX | Utilisé en MEM | Utilisé en WB | Description |
|-------------------|-----------|---------------|----------------|---------------|-------------|
| RegDst | ID | X | | | Sélectionne si la destination est rt ou rd. |
| ALUSrc | ID | X | | | Sélectionne si le 2ème opérande de l'ALU est rt ou l'immédiat. |
| ALUOp | ID | X | | | Spécifie l'opération à effectuer par l'ALU. |
| MemRead | ID | | X | | Active la lecture de la mémoire de données. |
| MemWrite | ID | | X | | Active l'écriture dans la mémoire de données. |
| Branch | ID | | X | | Indique une instruction de branchement. |
| RegWrite | ID | | | X | Active l'écriture dans le banc de registres. |
| MemtoReg | ID | | | X | Sélectionne si la donnée écrite au registre vient de l'ALU ou de la mémoire. |

*Note : Tous les signaux sont générés en ID et propagés via les registres ID/EX, EX/MEM, et MEM/WB pour être utilisés dans les étages appropriés.*

## IV. Les Aléas du Pipeline : Quand l'Idéal se Heurte à la Réalité

Un **aléa** (hazard) est une situation qui empêche l'instruction suivante dans le flux du programme de commencer son exécution durant le cycle d'horloge prévu. Les aléas sont la manifestation concrète des raisons pour lesquelles un pipeline d'instructions n'est pas un pipeline idéal. Ils sont le principal obstacle à l'atteinte du speedup théorique et leur gestion est au cœur de la conception des processeurs modernes. On les classe en trois catégories.

### 4.1 Aléas de Structure (Structural Hazards)

Un aléa de structure survient lorsque deux instructions différentes dans le pipeline tentent d'utiliser la même ressource matérielle au même cycle d'horloge.

**Cause** : Conflit d'accès à une ressource non entièrement pipelinée ou non dupliquée.

**Exemple Classique** : L'exemple le plus simple est celui d'une mémoire unifiée pour les instructions et les données. Au quatrième cycle d'une instruction load, l'étage MEM a besoin d'accéder à la mémoire pour lire la donnée. Simultanément, l'étage IF de l'instruction qui suit de trois cycles a besoin d'accéder à cette même mémoire pour chercher son code. Si la mémoire ne dispose que d'un seul port, un conflit survient.

**Solution** : La solution la plus courante est de dupliquer la ressource. C'est la raison fondamentale pour laquelle les architectures pipelinées performantes, comme notre MIPS, utilisent une architecture de type Harvard avec des mémoires (ou, plus pratiquement, des caches L1) séparées pour les instructions et les données (L1-I et L1-D). Cela élimine la grande majorité des aléas de structure.

### 4.2 Aléas de Données (Data Hazards)

Un aléa de données se produit lorsqu'une instruction dépend du résultat d'une instruction précédente qui est encore en cours de traitement dans le pipeline. C'est la conséquence directe du fait que les instructions ne sont pas indépendantes.

**Cause** : Dépendance de données entre instructions proches dans le flux d'exécution.

**Classification Formelle** : Il existe trois types de dépendances de données, qui peuvent se transformer en aléas.

#### Tableau 3 : Classification des Dépendances de Données et Aléas Associés

| Type de Dépendance | Nom Formel | Description | Exemple de Code MIPS | Problème dans un Pipeline à 5 Étages Simple |
|-------------------|------------|-------------|---------------------|---------------------------------------------|
| Read After Write (RAW) | Vraie Dépendance (True Dependence) | Une instruction (sub) tente de lire un registre ($s0) avant que l'instruction précédente (add) n'ait eu le temps d'y écrire le résultat. | `add $s0, $t0, $t1`<br>`sub $t2, $s0, $t3` | Aléa majeur et fréquent. L'instruction sub lira une ancienne valeur de $s0 si aucune mesure n'est prise. |
| Write After Read (WAR) | Anti-Dépendance | Une instruction (add) tente d'écrire dans un registre ($t0) avant que l'instruction précédente (sub) n'ait eu le temps de le lire. | `sub $t2, $t0, $t3`<br>`add $t0, $t4, $t5` | Impossible dans ce pipeline. La lecture (ID) se produit toujours bien avant l'écriture (WB). Devient un problème dans les architectures à exécution dans le désordre. |
| Write After Write (WAW) | Dépendance de Sortie (Output Dependence) | Deux instructions (add et sub) écrivent dans le même registre de destination ($s0). | `add $s0, $t0, $t1`<br>`sub $s0, $t2, $t3` | Impossible dans ce pipeline. Les écritures se font en ordre dans l'étage WB. Devient un problème dans les architectures à exécution dans le désordre. |

Les dépendances WAR et WAW sont souvent appelées **fausses dépendances** car elles ne concernent pas un flux de données mais un conflit de nommage de registres, dû au nombre limité de registres architecturaux.

Dans le cadre de notre pipeline simple "en ordre", nous nous concentrerons exclusivement sur la résolution de l'aléa RAW, qui est le plus courant et le plus critique.

### 4.3 Aléas de Contrôle (Control Hazards)

Un aléa de contrôle, ou aléa de branchement, est causé par les instructions qui modifient le flux de contrôle du programme, comme les branchements conditionnels (beq, bne) et les sauts (j).

**Cause** : La décision de prendre ou non un branchement, et donc l'adresse de la prochaine instruction à chercher, n'est connue que tard dans le pipeline. Pour une instruction beq, la comparaison des registres se fait en EX, et le calcul de l'adresse cible est finalisé à ce moment-là.

**Problème** : Pendant que l'instruction beq progresse à travers les étages IF et ID, le pipeline, ne connaissant pas l'issue du branchement, continue de manière spéculative à chercher les instructions séquentiellement suivantes (PC+4, PC+8). Si le branchement est finalement pris, ces instructions "innocentes" qui ont déjà commencé leur exécution doivent être annulées (on dit qu'on "flushe" ou "squashe" le pipeline), ce qui gaspille des cycles et réduit la performance.

**Solutions (aperçu)** : Les solutions incluent le gel du pipeline (attendre que la décision soit prise), la prédiction de branchement (deviner si le branchement sera pris ou non et ne corriger qu'en cas d'erreur), et une technique logicielle appelée branchement retardé (delayed branch). La gestion des aléas de contrôle est un sujet riche qui sera approfondi ultérieurement.

## V. Résolution des Aléas de Données : Verrouillage, Gel et Transfert

La gestion des aléas de données RAW est essentielle pour garantir à la fois la correction du programme et la performance du pipeline. Plusieurs stratégies, allant du logiciel au matériel, ont été développées.

### 5.1 Approche Statique : La Philosophie MIPS ("Microprocessor without Interlocked Pipeline Stages")

Au cœur de la conception initiale du MIPS se trouvait une philosophie radicale : simplifier le matériel au maximum en déléguant les tâches complexes au logiciel, en l'occurrence au compilateur. L'acronyme MIPS signifiait à l'origine "Microprocessor without Interlocked Pipeline Stages" (Microprocesseur sans Étages de Pipeline Verrouillés).

Dans cette approche, le matériel ne contient aucune logique pour détecter ou résoudre les aléas de données. C'est la responsabilité du compilateur d'assurer un fonctionnement correct. Connaissant la structure exacte du pipeline (par exemple, qu'un résultat d'ALU est disponible 2 cycles plus tard), le compilateur doit réorganiser le code ou, si ce n'est pas possible, insérer des instructions nop (no-operation) pour créer la distance temporelle nécessaire entre une instruction productrice et une instruction consommatrice.

**Exemple de code MIPS avec dépendance RAW :**

```mips
add $s0, $t0, $t1  # $s0 est produit ici
sub $t2, $s0, $t3  # $s0 est consommé immédiatement
```

**Même code, avec NOPs insérés par un compilateur pour MIPS-I :**

```mips
add $s0, $t0, $t1
nop                # Bulle pour attendre 1 cycle
nop                # Bulle pour attendre 1 cycle
sub $t2, $s0, $t3  # $s0 est maintenant disponible dans le banc de registres
```

Cette approche, défendue par des pionniers comme John L. Hennessy, présente l'avantage d'un matériel extrêmement simple et rapide. Cependant, elle a des inconvénients majeurs : elle crée un couplage très fort entre le code binaire et une microarchitecture spécifique (un programme compilé pour un pipeline à 5 étages ne fonctionnera pas correctement sur un pipeline à 7 étages) et peut conduire à une augmentation significative de la taille du code. Pour ces raisons, cette philosophie pure a été rapidement abandonnée au profit de solutions matérielles, qui offrent une abstraction plus robuste.

### 5.2 Le Transfert de Données (Data Forwarding / Bypassing)

La solution matérielle la plus efficace pour résoudre la majorité des aléas RAW est le **transfert de données**, aussi appelé **court-circuitage** (bypassing). L'observation clé est la suivante : pourquoi attendre que le résultat d'une instruction soit écrit dans le banc de registres à l'étage WB pour qu'une instruction suivante puisse le lire, alors que ce résultat est déjà calculé et disponible bien plus tôt dans le pipeline ?

Le transfert consiste à créer des chemins de données supplémentaires (des "raccourcis") qui acheminent le résultat directement depuis la sortie d'un étage (comme EX ou MEM) vers l'entrée de l'ALU à l'étage EX de l'instruction suivante qui en a besoin. Cela se fait via des multiplexeurs additionnels sur les entrées de l'ALU, contrôlés par une nouvelle unité logique : l'**Unité de Transfert** (Forwarding Unit).

Dans notre pipeline à 5 étages, deux chemins de transfert principaux sont nécessaires pour résoudre les aléas RAW entre instructions arithmétiques :

1. **Transfert EX/MEM → EX** : Le résultat de l'ALU de l'instruction I(n) est disponible dans le registre de pipeline EX/MEM à la fin de son étage EX. Il peut être immédiatement transféré à l'instruction I(n+1) qui entre dans son propre étage EX. Cela résout les dépendances directes comme add suivi de sub.

2. **Transfert MEM/WB → EX** : Le résultat de l'instruction I(n) est dans le registre MEM/WB à la fin de son étage MEM. Il peut être transféré à l'instruction I(n+2) qui entre dans son étage EX.

La logique de l'Unité de Transfert doit détecter ces dépendances et contrôler les multiplexeurs en conséquence. Une subtilité importante se présente en cas de "double aléa" (par exemple, `add $1,...` suivi de `add $1,...` puis `sub..., $1,...`). L'instruction sub dépend des deux add. Le matériel doit impérativement transférer le résultat le plus récent, c'est-à-dire celui de l'instruction la plus avancée dans le programme, qui se trouve dans le chemin EX/MEM. La logique de transfert est donc un encodeur à priorité : le chemin EX/MEM a la priorité sur le chemin MEM/WB pour une même destination.

#### 5.2.1 Implémentation de la Logique de Transfert

La logique de l'unité de transfert peut être exprimée en pseudo-code. Elle compare les registres sources de l'instruction en EX (ID/EX.RegisterRs, ID/EX.RegisterRt) avec les registres de destination des instructions plus avancées en MEM (EX/MEM.RegisterRd) et en WB (MEM/WB.RegisterRd).

**Pseudo-code pour la logique de l'Unité de Transfert :**

```
// ForwardA contrôle le premier opérande de l'ALU (venant de Rs)
// 00: Pas de transfert (valeur de ID/EX)
// 10: Transfert depuis EX/MEM
// 01: Transfert depuis MEM/WB

// Détection de l'aléa EX -> EX
if (EX/MEM.RegWrite == 1 AND EX/MEM.RegisterRd != 0 AND EX/MEM.RegisterRd == ID/EX.RegisterRs) {
    ForwardA = 10;
}
// Détection de l'aléa MEM -> EX (avec priorité au plus récent)
else if (MEM/WB.RegWrite == 1 AND MEM/WB.RegisterRd != 0 AND MEM/WB.RegisterRd == ID/EX.RegisterRs) {
    // La condition de non-chevauchement avec l'aléa EX->EX est implicite dans le 'else if'
    ForwardA = 01;
}
else {
    ForwardA = 00;
}

// Logique similaire pour ForwardB (contrôlant le second opérande de l'ALU, venant de Rt)
if (EX/MEM.RegWrite == 1 AND EX/MEM.RegisterRd != 0 AND EX/MEM.RegisterRd == ID/EX.RegisterRt) {
    ForwardB = 10;
}
else if (MEM/WB.RegWrite == 1 AND MEM/WB.RegisterRd != 0 AND MEM/WB.RegisterRd == ID/EX.RegisterRt) {
    ForwardB = 01;
}
else {
    ForwardB = 00;
}
```

*Note : La condition RegisterRd != 0 est une optimisation pour éviter de transférer le résultat d'une écriture dans le registre zéro, qui est une source constante de la valeur 0 et ne doit jamais être modifié.*

**Implémentation en C simulant la logique matérielle :**

```c
#include <stdio.h>
#include <stdbool.h>

// Structure pour simuler les registres de pipeline et leurs signaux de contrôle
typedef struct {
    bool RegWrite;
    int RegisterRd; // Numéro du registre de destination
} EX_MEM_Reg;

typedef struct {
    bool RegWrite;
    int RegisterRd;
} MEM_WB_Reg;

typedef struct {
    int RegisterRs; // Numéro du registre source 1
    int RegisterRt; // Numéro du registre source 2
} ID_EX_Reg;

// Structure pour les signaux de contrôle du forwarding
typedef struct {
    int ForwardA; // 00, 01, 10
    int ForwardB; // 00, 01, 10
} ForwardingControls;

// Fonction simulant l'unité de transfert
ForwardingControls forwarding_unit(EX_MEM_Reg ex_mem, MEM_WB_Reg mem_wb, ID_EX_Reg id_ex) {
    ForwardingControls controls = {0, 0}; // Initialise à "pas de transfert" (00)

    // Logique pour ForwardA (premier opérande de l'ALU)
    // Aléa EX/MEM -> EX?
    if (ex_mem.RegWrite && (ex_mem.RegisterRd != 0) && (ex_mem.RegisterRd == id_ex.RegisterRs)) {
        controls.ForwardA = 2; // Binaire '10'
    }
    // Aléa MEM/WB -> EX? (priorité au plus récent)
    else if (mem_wb.RegWrite && (mem_wb.RegisterRd != 0) && (mem_wb.RegisterRd == id_ex.RegisterRs)) {
        controls.ForwardA = 1; // Binaire '01'
    }

    // Logique pour ForwardB (second opérande de l'ALU)
    // Aléa EX/MEM -> EX?
    if (ex_mem.RegWrite && (ex_mem.RegisterRd != 0) && (ex_mem.RegisterRd == id_ex.RegisterRt)) {
        controls.ForwardB = 2; // Binaire '10'
    }
    // Aléa MEM/WB -> EX? (priorité au plus récent)
    else if (mem_wb.RegWrite && (mem_wb.RegisterRd != 0) && (mem_wb.RegisterRd == id_ex.RegisterRt)) {
        controls.ForwardB = 1; // Binaire '01'
    }
    
    return controls;
}

int main() {
    // Exemple : add $s0, $t1, $t2; sub $t4, $t3, $s0
    // L'instruction 'sub' est en EX, l'instruction 'add' est en MEM.
    EX_MEM_Reg ex_mem_reg = {true, 16}; // add écrit dans $s0 (reg 16)
    MEM_WB_Reg mem_wb_reg = {false, 0}; // Instruction précédente sans écriture
    ID_EX_Reg id_ex_reg = {19, 16};     // sub lit $t3 (reg 19) et $s0 (reg 16)

    ForwardingControls result = forwarding_unit(ex_mem_reg, mem_wb_reg, id_ex_reg);

    printf("ForwardA: %d, ForwardB: %d\n", result.ForwardA, result.ForwardB);
    // Résultat attendu : ForwardA: 0, ForwardB: 2 (transfert depuis EX/MEM pour le 2ème opérande)

    return 0;
}
```

### 5.3 Le Cas Particulier de l'Aléa "Load-Use"

Le transfert de données est puissant, mais il ne peut pas résoudre tous les aléas RAW. Le cas le plus courant est l'**aléa "load-use"**, qui se produit lorsqu'une instruction utilise le résultat d'une instruction `lw` (load word) immédiatement après.

**Le Problème** : Une instruction `lw` ne rend la donnée disponible qu'à la fin de son étage MEM. Une instruction dépendante (par exemple, un `add`) qui la suit immédiatement a besoin de cette donnée au début de son propre étage EX. Comme l'étage EX de l'instruction `add` se déroule en même temps que l'étage MEM de l'instruction `lw`, la donnée n'est pas encore disponible pour être transférée. Le transfert ne peut pas "remonter le temps" à l'intérieur d'un même cycle d'horloge.

**La Solution** : **Gel (Stall) + Transfert**. La seule solution est de retarder l'instruction dépendante d'un cycle. Le matériel doit détecter cet aléa spécifique et geler le pipeline pendant un cycle. Concrètement :

1. L'instruction `lw` continue sa progression.
2. L'instruction dépendante est bloquée à l'étage ID.
3. Une "bulle" (équivalente à une instruction nop) est injectée dans l'étage EX.
4. Au cycle suivant, l'instruction `lw` est maintenant à l'étage WB, et sa donnée est dans le registre MEM/WB. L'instruction dépendante peut alors avancer à l'étage EX, et la donnée peut lui être fournie via le chemin de transfert MEM/WB → EX que nous avons déjà conçu.

Ce mécanisme est géré par une **Unité de Détection d'Aléas** (Hazard Detection Unit), distincte de l'unité de transfert.

#### 5.3.1 Implémentation de la Logique de Détection et de Gel

L'unité de détection d'aléas opère à l'étage ID et examine l'instruction qui vient de passer en EX. Si cette instruction est un `lw` (ID/EX.MemRead == 1), elle compare son registre de destination (ID/EX.RegisterRt) avec les registres sources de l'instruction actuellement en ID (qui sont dans le registre IF/ID). Si une correspondance est trouvée, un gel est déclenché.

**Pseudo-code pour la logique de l'Unité de Détection d'Aléas :**

```
// Cette logique est dans l'étage ID.
// Elle examine l'instruction en EX (via le registre ID/EX)
// et l'instruction en ID (via le registre IF/ID).

if (ID/EX.MemRead == 1 AND 
    (ID/EX.RegisterRt == IF/ID.RegisterRs OR ID/EX.RegisterRt == IF/ID.RegisterRt)) 
{
    // Aléa Load-Use détecté!
    // Actions de gel :
    1. PC.WriteEnable = 0;       // Empêche le PC de s'incrémenter.
    2. IF/ID.WriteEnable = 0;   // Empêche le registre IF/ID d'être écrasé.
    3. ID/EX.ControlSignals = 0; // Injecte une bulle (nop) en forçant les contrôles à 0.
}
```

**Implémentation en C simulant la détection et le gel :**

```c
#include <stdio.h>
#include <stdbool.h>

// Simule les signaux pertinents des registres de pipeline
typedef struct {
    bool MemRead;
    int RegisterRt; // Destination pour un lw
} ID_EX_Reg_Stall;

typedef struct {
    int RegisterRs;
    int RegisterRt;
} IF_ID_Reg_Stall;

// Simule les signaux de contrôle qui seront affectés par le gel
typedef struct {
    bool PC_WriteEnable;
    bool IF_ID_WriteEnable;
    // En matériel, on forcerait les signaux de contrôle de ID/EX à 0.
    // Ici, on retourne juste un booléen pour indiquer le gel.
} StallControls;

// Fonction simulant l'unité de détection d'aléas
bool hazard_detection_unit(ID_EX_Reg_Stall id_ex, IF_ID_Reg_Stall if_id) {
    if (id_ex.MemRead && 
        (id_ex.RegisterRt != 0) &&
        (id_ex.RegisterRt == if_id.RegisterRs || id_ex.RegisterRt == if_id.RegisterRt)) {
        return true; // Aléa détecté, il faut geler.
    }
    return false;
}

void execute_cycle(bool stall_detected) {
    StallControls controls = {true, true};
    if (stall_detected) {
        controls.PC_WriteEnable = false;
        controls.IF_ID_WriteEnable = false;
        printf("Aléa Load-Use détecté! Gel du pipeline.\n");
        printf("PC_WriteEnable = %d, IF_ID_WriteEnable = %d\n", controls.PC_WriteEnable, controls.IF_ID_WriteEnable);
        printf("Une bulle (NOP) est insérée dans l'étage EX.\n");
    } else {
        printf("Pas d'aléa de type load-use. Le pipeline avance normalement.\n");
    }
}

int main() {
    // Exemple : lw $t0, 0($t1); add $t2, $t0, $t3
    // L'instruction 'add' est en ID, l'instruction 'lw' est en EX.
    ID_EX_Reg_Stall id_ex_reg = {true, 8};  // lw lit la mémoire et écrit dans $t0 (reg 8)
    IF_ID_Reg_Stall if_id_reg = {8, 11};    // add lit $t0 (reg 8) et $t3 (reg 11)

    bool stall = hazard_detection_unit(id_ex_reg, if_id_reg);
    execute_cycle(stall);

    printf("\n---\n\n");

    // Exemple sans aléa : lw $t0, 0($t1); add $t2, $t4, $t3
    ID_EX_Reg_Stall id_ex_reg_no_hazard = {true, 8}; // lw écrit dans $t0 (reg 8)
    IF_ID_Reg_Stall if_id_reg_no_hazard = {12, 11};  // add lit $t4 et $t3

    stall = hazard_detection_unit(id_ex_reg_no_hazard, if_id_reg_no_hazard);
    execute_cycle(stall);

    return 0;
}
```

## VI. Conclusion et Perspectives

### 6.1 Synthèse des Coûts et Bénéfices du Pipeline

Le pipelining est une technique architecturale fondamentale et incontournable. Son bénéfice principal est une augmentation spectaculaire du débit d'instructions, permettant une bien meilleure utilisation des ressources matérielles du processeur. En chevauchant l'exécution de plusieurs instructions, on s'approche d'un idéal où une instruction est terminée à chaque cycle d'horloge.

Cependant, ce gain ne vient pas sans un coût significatif. La complexité matérielle est accrue par l'ajout des registres de pipeline, des multiplexeurs de transfert, et des unités de contrôle dédiées à la gestion des aléas (unité de transfert et unité de détection d'aléas). La latence d'une instruction individuelle est même légèrement augmentée. Plus important encore, le pipelining introduit les aléas (structure, données, contrôle) qui sont des obstacles inhérents à la performance et dont la résolution correcte et efficace est un défi de conception majeur. Le compromis entre la profondeur du pipeline (qui augmente le débit théorique) et la complexité de la gestion des aléas (qui augmente avec la profondeur) est au cœur des décisions des architectes de processeurs.

### 6.2 Au-delà du Pipeline Simple : Exécution dans le Désordre et Architectures Superscalaires

Le pipeline simple à 5 étages que nous avons étudié, bien qu'efficace, reste limité. Lorsqu'un aléa non résoluble par transfert survient (comme un load-use ou un cache-miss), le pipeline entier doit geler, perdant ainsi tout son avantage de débit. Les limitations que nous avons observées motivent naturellement les techniques plus avancées qui définissent les processeurs haute performance modernes.

**Architectures Superscalaires** : Au lieu de chercher et d'exécuter une seule instruction par cycle, une architecture superscalaire dispose de plusieurs pipelines en parallèle. Elle peut chercher, décoder et exécuter plusieurs instructions (par exemple, 2, 4 ou plus) à chaque cycle, à condition qu'elles soient indépendantes.

**Exécution dans le Désordre (Out-of-Order Execution)** : C'est la suite logique pour surmonter les blocages dus aux dépendances. Si une instruction est bloquée en attente d'une donnée, un processeur à exécution dans le désordre ne gèle pas tout le pipeline. Il regarde plus loin dans le flux d'instructions du programme pour trouver des instructions indépendantes qu'il peut exécuter en avance. Une fois la donnée attendue disponible, l'instruction bloquée peut reprendre son cours. Le processeur réordonne ensuite les résultats pour donner au programmeur l'illusion d'une exécution séquentielle.

Ces concepts, qui permettent de masquer la latence et d'exploiter encore plus de parallélisme au niveau de l'instruction, sont la prochaine étape de notre exploration de l'architecture des ordinateurs. Ils s'appuient tous sur les principes fondamentaux du pipelining et de la gestion des aléas que nous avons établis aujourd'hui.
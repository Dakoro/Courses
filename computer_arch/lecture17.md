# Le Cœur des Processeurs Modernes : Parallélisme, Spéculation et Prédiction

## Introduction : La Quête du Parallélisme au Niveau Instruction (ILP)

Au cœur de l'évolution des microprocesseurs se trouve une quête incessante pour une performance accrue. Depuis des décennies, les architectes de processeurs s'efforcent d'exécuter les programmes de plus en plus rapidement. Une technique fondamentale dans cette quête est le **pipelining**, qui consiste à décomposer l'exécution d'une instruction en plusieurs étapes distinctes (par exemple, Fetch, Decode, Execute, Memory, Write-back). En superposant l'exécution de plusieurs instructions, chacune à une étape différente du pipeline, on augmente considérablement le débit par rapport à une exécution purement séquentielle.

Cependant, le pipelining se heurte rapidement à des limites intrinsèques. Les aléas, ou dépendances, entre instructions créent des "bulles" dans le pipeline, des cycles d'horloge pendant lesquels aucune instruction utile n'est terminée, ce qui dégrade les performances. On distingue principalement les **aléas de données** (une instruction a besoin du résultat d'une instruction précédente non encore terminée) et les **aléas de contrôle** (l'adresse de la prochaine instruction à exécuter dépend de l'issue d'un branchement conditionnel non encore résolu).

La performance d'un processeur peut être modélisée par l'équation fondamentale du temps d'exécution :

$$T_{exec} = N_{instr} \times CPI \times T_{cycle}$$

Dans cette équation, $N_{instr}$ est le nombre d'instructions du programme, $T_{cycle}$ est la durée d'un cycle d'horloge, et **CPI** (Cycles Per Instruction) est le nombre moyen de cycles nécessaires pour exécuter une instruction. Pour améliorer les performances, les architectes cherchent à réduire le CPI, ou, de manière équivalente, à augmenter son inverse, l'**IPC** (Instructions Per Cycle). Un pipeline scalaire simple a un IPC idéal de 1, mais les aléas le réduisent en pratique.

Pour dépasser cette limite, les processeurs haute performance modernes s'appuient sur un ensemble de techniques sophistiquées qui exploitent ce que l'on nomme le **parallélisme au niveau de l'instruction** (Instruction-Level Parallelism - ILP). Ces techniques visent à trouver et à exécuter plusieurs instructions indépendantes en parallèle, même si le programme a été écrit de manière séquentielle.

Ce cours se propose d'explorer en profondeur les trois piliers qui constituent le fondement des microarchitectures modernes :

1. **L'Exécution dans le Désordre (Out-of-Order - OoO)** : Une approche dynamique révolutionnaire qui permet de tolérer les latences des opérations en exécutant les instructions dès que leurs données sont disponibles, plutôt que de suivre aveuglément l'ordre séquentiel du programme.

2. **L'Exécution Superscalaire** : Une technique qui augmente le débit en initiant et en exécutant plusieurs instructions à chaque cycle d'horloge, ce qui revient à "élargir" le pipeline pour traiter plus de travail en parallèle.

3. **La Prédiction de Branchement** : Un mécanisme de spéculation essentiel, sans lequel les deux premières techniques seraient inefficaces. Il s'agit de "deviner" l'issue des branchements pour alimenter continuellement le processeur en instructions, évitant ainsi des arrêts coûteux du pipeline.

Ensemble, ces trois concepts transforment la spécification séquentielle d'un programme en une exécution massivement parallèle, tout en maintenant l'illusion d'une exécution séquentielle pour le programmeur.

## Leçon 1 : L'Exécution dans le Désordre (Out-of-Order - OoO)

L'exécution dans le désordre est sans doute l'une des innovations les plus importantes de l'histoire des microprocesseurs. Elle représente un changement de paradigme fondamental : le matériel ne se contente plus d'exécuter passivement un programme, il l'analyse dynamiquement pour en extraire un parallélisme caché.

### 1.1. Principes Fondamentaux : Tolérance à la Latence et Graphes de Flux de Données

Le concept clé derrière l'exécution OoO est la **tolérance à la latence**. Dans un programme, toutes les instructions n'ont pas la même durée d'exécution (latence). Une addition entière peut prendre un seul cycle, tandis qu'une division en virgule flottante peut en prendre des dizaines, et un chargement depuis la mémoire principale peut en prendre des centaines. Dans un processeur in-order (qui exécute les instructions dans l'ordre strict du programme), une instruction à longue latence bloque tout le pipeline. Toutes les instructions qui la suivent, même si elles sont totalement indépendantes, doivent attendre que l'instruction lente se termine. Cela crée d'énormes "bulles" et anéantit les performances.

Comme le souligne l'analyse, si toutes les opérations prenaient un seul cycle, l'intérêt de l'OoO serait quasi nul. C'est précisément l'écart croissant entre la vitesse des processeurs et la latence de la mémoire dans les années 1980 et 1990 qui a rendu la tolérance à la latence non plus une option, mais une nécessité absolue pour la haute performance.

Pour résoudre ce problème, un processeur OoO découple l'ordre de décodage des instructions de leur ordre d'exécution. Il construit dynamiquement, pour une petite portion du programme appelée **fenêtre d'instruction**, un **graphe de flux de données** (Data Flow Graph - DFG). Dans ce graphe, les nœuds sont les instructions et les arcs représentent les dépendances de données (une instruction produit une valeur qu'une autre consomme). Le processeur peut alors exécuter n'importe quelle instruction (nœud) dont les dépendances (arcs entrants) ont été satisfaites, c'est-à-dire dont les opérandes sont disponibles. Ainsi, pendant qu'une instruction de chargement mémoire à longue latence est en attente, le processeur peut exécuter des dizaines d'autres instructions indépendantes qui se trouvent plus loin dans le programme, "cachant" ou "tolérant" ainsi la latence de la première.

Ce processus peut être résumé en quatre actions fondamentales :

1. **Lier (Linking)** : Connecter une instruction "consommatrice" à son instruction "productrice" pour qu'elle puisse recevoir la valeur nécessaire.
2. **Tamponner (Buffering)** : Stocker les instructions en attente dans des tampons matériels, afin qu'elles ne bloquent pas le reste du pipeline.
3. **Suivre la Disponibilité (Tracking Readiness)** : Surveiller en permanence l'état des instructions tamponnées pour savoir lesquelles ont reçu tous leurs opérandes et sont prêtes à être exécutées.
4. **Distribuer (Dispatching)** : Envoyer les instructions prêtes aux unités fonctionnelles (ALU, FPU, etc.) pour exécution.

### 1.2. L'Algorithme de Tomasulo : Une Étude Approfondie

L'algorithme de Tomasulo est l'implémentation canonique de ces principes. Il ne s'agit pas d'un concept abstrait, mais d'une solution d'ingénierie concrète développée par Robert Tomasulo chez IBM en 1967 pour l'unité de calcul en virgule flottante de l'ordinateur hautes performances IBM System/360 Model 91. Son objectif était d'exploiter au maximum les multiples unités arithmétiques qui pouvaient fonctionner en parallèle, une nouveauté pour l'époque.

L'algorithme repose sur trois composants matériels clés :

#### Le Renommage de Registres (Register Renaming)

C'est l'innovation la plus profonde et la plus importante de l'algorithme de Tomasulo. Pour comprendre son importance, il faut distinguer les vraies dépendances des fausses.

- **Dépendance Vraie (Read-After-Write - RAW)** : `ADD R1, R2, R3` suivi de `SUB R4, R1, R5`. La seconde instruction a besoin du résultat de la première. Cette dépendance est fondamentale et doit être respectée.

- **Fausse Dépendance (Write-After-Read - WAR)** : `SUB R4, R1, R5` suivi de `ADD R1, R2, R3`. La seconde instruction écrit dans R1 après que la première l'a lu. Dans un pipeline simple, il faut attendre que SUB ait lu R1 avant que ADD puisse y écrire, même si les opérations sont indépendantes.

- **Fausse Dépendance (Write-After-Write - WAW)** : `ADD R1, R2, R3` suivi de `MUL R1, R4, R5`. Les deux instructions écrivent dans le même registre. Elles doivent être sérialisées pour que le résultat final dans R1 soit celui de MUL.

Les dépendances WAR et WAW ne sont pas des dépendances de données réelles, mais des conflits de noms dus à la réutilisation des registres architecturaux (le nombre limité de registres visibles par le programmeur, comme R0-R31). Le renommage de registres élimine ces fausses dépendances. L'idée est d'allouer dynamiquement une nouvelle ressource de stockage physique (un "tag" ou un registre physique) pour le résultat de chaque instruction qui écrit dans un registre. Ainsi, nos deux exemples de fausses dépendances deviennent non conflictuels, car les écritures se font dans des emplacements physiques différents, et les instructions peuvent s'exécuter en parallèle.

#### Les Stations de Réservation (Reservation Stations - RS)

Ce sont des tampons situés devant chaque unité fonctionnelle (par exemple, une station pour l'additionneur, une pour le multiplicateur). Lorsqu'une instruction est décodée ("issued"), elle est placée dans une station de réservation libre. Chaque entrée d'une station de réservation contient :

- **Op** : L'opération à effectuer (ADD, MUL, etc.).
- **Vj, Vk** : Les valeurs des opérandes sources, si elles sont déjà disponibles dans les registres.
- **Qj, Qk** : Des "tags" qui identifient les stations de réservation qui produiront les opérandes sources, si ceux-ci ne sont pas encore disponibles. Initialement, ces champs contiennent le tag de l'instruction productrice. Ils sont mis à zéro lorsque la valeur devient disponible.
- **Busy** : Un bit indiquant si la station est occupée.

Une instruction est prête à être distribuée (dispatch) à son unité fonctionnelle uniquement lorsque Qj et Qk sont tous les deux à zéro, signifiant que toutes ses dépendances de données (RAW) sont résolues.

#### Le Bus de Données Commun (Common Data Bus - CDB)

Le CDB est le mécanisme de communication qui "anime" le graphe de flux de données. C'est un bus de diffusion connecté à la sortie de toutes les unités fonctionnelles et à l'entrée de toutes les stations de réservation. Lorsqu'une unité fonctionnelle termine un calcul, elle diffuse simultanément sur le CDB :

- Le résultat calculé.
- Le tag de la station de réservation qui a produit ce résultat.

Toutes les stations de réservation qui attendent ce tag (c'est-à-dire, dont le champ Qj ou Qk correspond au tag diffusé) "écoutent" le CDB, capturent la valeur et la stockent dans leur champ Vj ou Vk correspondant, tout en mettant à zéro le champ Qj ou Qk. Ce mécanisme de "wakeup and select" permet aux instructions dépendantes de devenir prêtes à l'exécution dès que leurs données sont produites, sans passer par le banc de registres principal.

### 1.3. Implémentation de l'Algorithme de Tomasulo

Pour solidifier la compréhension, il est instructif d'implémenter un simulateur d'un processeur basé sur l'algorithme de Tomasulo, comme cela est souvent proposé dans les cours avancés d'architecture.

#### 1.3.1. Pseudo-code Détaillé

Le fonctionnement se déroule en trois étapes principales qui s'exécutent en parallèle à chaque cycle d'horloge.

```pseudocode
// Structures de données globales
InstructionQueue q; // File d'instructions à traiter
ReservationStation addRS, mulRS;
RegisterFile regs;
RegisterStat regStats; // Table de renommage (alias table)
CommonDataBus cdb;

// Boucle principale du simulateur
while (simulation_active) {
    write_result();
    execute();
    issue();
    clock_cycle++;
}

// Étape 1: Issue (Émission)
// Décode une instruction et la place dans une station de réservation
function issue() {
    if (q.empty()) return;
    
    instr = q.front();
    
    // 1. Trouver une station de réservation (RS) libre pour le type d'opération
    if (instr.op is ADD or SUB) {
        if (no free addRS) return; // Stall
        rs_entry = find_free_add_rs();
    } else { // MUL or DIV
        if (no free mulRS) return; // Stall
        rs_entry = find_free_mul_rs();
    }

    // 2. Lire les opérandes
    // Source 1 (j)
    if (regStats[instr.src1].Qi != 0) { // Opérande non prêt, en cours de calcul par une autre RS
        rs_entry.Qj = regStats[instr.src1].Qi;
    } else { // Opérande prêt
        rs_entry.Vj = regs[instr.src1];
        rs_entry.Qj = 0;
    }
    // Source 2 (k)
    if (regStats[instr.src2].Qi != 0) {
        rs_entry.Qk = regStats[instr.src2].Qi;
    } else {
        rs_entry.Vk = regs[instr.src2];
        rs_entry.Qk = 0;
    }
    
    // 3. Allouer l'entrée de la RS et mettre à jour la table de renommage
    rs_entry.busy = true;
    rs_entry.op = instr.op;
    regStats[instr.dest].Qi = rs_entry.tag; // Le résultat de dest sera produit par cette RS

    q.pop();
}

// Étape 2: Execute (Exécution)
// Démarre l'exécution des instructions dont les opérandes sont prêts
function execute() {
    for each rs in (addRS, mulRS) {
        if (rs.busy && rs.Qj == 0 && rs.Qk == 0) {
            // L'instruction est prête, l'envoyer à l'unité fonctionnelle
            // La latence de l'exécution est simulée ici (e.g., décrémenter un compteur)
            // Quand le calcul est fini, l'instruction est prête à écrire son résultat
        }
    }
}

// Étape 3: Write Result (Écriture du Résultat)
// Diffuse les résultats terminés sur le CDB
function write_result() {
    for each fu_output ready this cycle {
        result = fu_output.value;
        tag = fu_output.tag;
        
        // 1. Diffuser sur le CDB
        cdb.write(tag, result);

        // 2. Mettre à jour le banc de registres et la table de renommage
        for each regStat in regStats {
            if (regStat.Qi == tag) {
                regs = result;
                regStat.Qi = 0;
            }
        }
        
        // 3. Mettre à jour toutes les stations de réservation en attente
        for each rs in (addRS, mulRS) {
            if (rs.Qj == tag) {
                rs.Vj = result;
                rs.Qj = 0;
            }
            if (rs.Qk == tag) {
                rs.Vk = result;
                rs.Qk = 0;
            }
        }
        
        // 4. Libérer la station de réservation qui vient de terminer
        get_rs_by_tag(tag).busy = false;
    }
}
```

#### 1.3.2. Implémentation en C/C++

Une implémentation en C++ utiliserait des `struct` ou des `class` pour modéliser les composants matériels, et des `std::vector` ou `std::array` pour les ensembles de structures comme les stations de réservation.

```cpp
#include <iostream>
#include <vector>
#include <string>
#include <optional>

// Simplification: les tags sont les indices des RS
const int NUM_REGS = 32;
const int NUM_ADD_RS = 3;
const int NUM_MUL_RS = 2;

enum OpType { ADD, SUB, MUL, DIV };

struct Instruction {
    OpType op;
    int dest, src1, src2;
};

struct ReservationStation {
    bool busy = false;
    OpType op;
    double vj = 0.0, vk = 0.0;
    int qj = 0, qk = 0; // 0 si prêt, sinon tag de la RS productrice
    int dest_tag;
    int cycles_left = -1;
};

struct RegisterStat {
    int qi = 0; // 0 si prêt, sinon tag de la RS productrice
};

class TomasuloSimulator {
public:
    TomasuloSimulator(std::vector<Instruction> program) : instruction_queue(program) {
        add_rs.resize(NUM_ADD_RS);
        mul_rs.resize(NUM_MUL_RS);
        reg_stats.resize(NUM_REGS);
        register_file.resize(NUM_REGS, 0.0);
    }

    void run() {
        int cycle = 0;
        while (!is_finished()) {
            std::cout << "--- Cycle " << ++cycle << " ---" << std::endl;
            write_result();
            execute();
            issue();
            // Afficher l'état (non implémenté pour la brièveté)
        }
    }

private:
    std::vector<Instruction> instruction_queue;
    int pc = 0;

    std::vector<ReservationStation> add_rs;
    std::vector<ReservationStation> mul_rs;
    std::vector<RegisterStat> reg_stats;
    std::vector<double> register_file;
    
    struct CdbMessage {
        int tag;
        double value;
    };
    std::vector<CdbMessage> cdb_bus;

    void issue() {
        if (pc >= instruction_queue.size()) return;

        Instruction instr = instruction_queue[pc];
        bool is_add_type = (instr.op == ADD || instr.op == SUB);
        auto& rs_pool = is_add_type ? add_rs : mul_rs;
        int tag_offset = is_add_type ? 1 : 1 + NUM_ADD_RS;

        int free_rs_idx = -1;
        for (int i = 0; i < rs_pool.size(); ++i) {
            if (!rs_pool[i].busy) {
                free_rs_idx = i;
                break;
            }
        }

        if (free_rs_idx == -1) return; // Stall

        auto& rs_entry = rs_pool[free_rs_idx];
        
        if (reg_stats[instr.src1].qi != 0) {
            rs_entry.qj = reg_stats[instr.src1].qi;
        } else {
            rs_entry.vj = register_file[instr.src1];
            rs_entry.qj = 0;
        }

        if (reg_stats[instr.src2].qi != 0) {
            rs_entry.qk = reg_stats[instr.src2].qi;
        } else {
            rs_entry.vk = register_file[instr.src2];
            rs_entry.qk = 0;
        }

        rs_entry.busy = true;
        rs_entry.op = instr.op;
        rs_entry.dest_tag = free_rs_idx + tag_offset;
        reg_stats[instr.dest].qi = rs_entry.dest_tag;
        
        pc++;
        std::cout << "Issued instruction " << pc << " to RS tag " << rs_entry.dest_tag << std::endl;
    }

    void execute() {
        auto process_rs_pool = [&](auto& rs_pool, int latency) {
            for (auto& rs : rs_pool) {
                if (rs.busy && rs.qj == 0 && rs.qk == 0 && rs.cycles_left == -1) {
                    rs.cycles_left = latency;
                }
                if (rs.cycles_left > 0) {
                    rs.cycles_left--;
                }
            }
        };
        process_rs_pool(add_rs, 2); // 2 cycles for ADD/SUB
        process_rs_pool(mul_rs, 10); // 10 cycles for MUL/DIV
    }

    void write_result() {
        cdb_bus.clear();
        auto find_completed = [&](auto& rs_pool, int tag_offset) {
            for (int i = 0; i < rs_pool.size(); ++i) {
                if (rs_pool[i].busy && rs_pool[i].cycles_left == 0) {
                    double result = 0.0;
                    switch(rs_pool[i].op) {
                        case ADD: result = rs_pool[i].vj + rs_pool[i].vk; break;
                        case SUB: result = rs_pool[i].vj - rs_pool[i].vk; break;
                        case MUL: result = rs_pool[i].vj * rs_pool[i].vk; break;
                        case DIV: result = rs_pool[i].vj / rs_pool[i].vk; break;
                    }
                    cdb_bus.push_back({rs_pool[i].dest_tag, result});
                    rs_pool[i].busy = false;
                    rs_pool[i].cycles_left = -1;
                }
            }
        };
        find_completed(add_rs, 1);
        find_completed(mul_rs, 1 + NUM_ADD_RS);

        for (const auto& msg : cdb_bus) {
            std::cout << "Broadcasting from tag " << msg.tag << " with value " << msg.value << std::endl;
            for (int i = 0; i < NUM_REGS; ++i) {
                if (reg_stats[i].qi == msg.tag) {
                    register_file[i] = msg.value;
                    reg_stats[i].qi = 0;
                }
            }
            auto update_rs = [&](auto& rs_pool) {
                for (auto& rs : rs_pool) {
                    if (rs.qj == msg.tag) { rs.vj = msg.value; rs.qj = 0; }
                    if (rs.qk == msg.tag) { rs.vk = msg.value; rs.qk = 0; }
                }
            };
            update_rs(add_rs);
            update_rs(mul_rs);
        }
    }

    bool is_finished() {
        if (pc < instruction_queue.size()) return false;
        for(const auto& rs : add_rs) if(rs.busy) return false;
        for(const auto& rs : mul_rs) if(rs.busy) return false;
        return true;
    }
};
```

#### 1.3.3. Implémentation en Python

Python, avec sa syntaxe claire et son typage dynamique, est excellent pour un prototypage rapide et une démonstration pédagogique de l'algorithme.

```python
import collections

class TomasuloSimulator:
    def __init__(self, program):
        self.program = collections.deque(program)
        self.pc = 0
        self.reg_file = [0.0] * 32
        self.reg_stat = [0] * 32  # 0 if ready, else tag of producing RS

        self.add_rs = [{'busy': False, 'tag': i + 1} for i in range(3)]
        self.mul_rs = [{'busy': False, 'tag': i + 4} for i in range(2)]
        self.rs_pools = {'ADD': self.add_rs, 'SUB': self.add_rs, 'MUL': self.mul_rs, 'DIV': self.mul_rs}
        self.latencies = {'ADD': 2, 'SUB': 2, 'MUL': 10, 'DIV': 40}

    def issue(self):
        if not self.program:
            return

        instr = self.program[0]
        op, dest, src1, src2 = instr['op'], instr['dest'], instr['src1'], instr['src2']
        
        rs_pool = self.rs_pools[op]
        for rs in rs_pool:
            if not rs['busy']:
                self.program.popleft()
                rs.update({'busy': True, 'op': op, 'dest': dest, 'cycles_left': self.latencies[op]})
                
                # Check source 1
                if self.reg_stat[src1] > 0:
                    rs['qj'] = self.reg_stat[src1]
                else:
                    rs['vj'] = self.reg_file[src1]
                    rs['qj'] = 0
                
                # Check source 2
                if self.reg_stat[src2] > 0:
                    rs['qk'] = self.reg_stat[src2]
                else:
                    rs['vk'] = self.reg_file[src2]
                    rs['qk'] = 0

                self.reg_stat[dest] = rs['tag']
                print(f"Issued {op} R{dest},R{src1},R{src2} to RS tag {rs['tag']}")
                return

    def execute(self):
        for rs in self.add_rs + self.mul_rs:
            if rs.get('busy') and rs.get('qj', 0) == 0 and rs.get('qk', 0) == 0:
                if rs.get('cycles_left', 0) > 0:
                    rs['cycles_left'] -= 1

    def write_result(self):
        cdb = []
        for rs in self.add_rs + self.mul_rs:
            if rs.get('busy') and rs.get('cycles_left', -1) == 0:
                op, vj, vk = rs['op'], rs['vj'], rs['vk']
                result = 0
                if op == 'ADD': result = vj + vk
                elif op == 'SUB': result = vj - vk
                elif op == 'MUL': result = vj * vk
                elif op == 'DIV': result = vj / vk if vk != 0 else float('inf')
                
                cdb.append({'tag': rs['tag'], 'value': result, 'dest': rs['dest']})
                rs['busy'] = False
                print(f"RS tag {rs['tag']} finished. Result: {result}")

        for msg in cdb:
            tag, value, dest = msg['tag'], msg['value'], msg['dest']
            if self.reg_stat[dest] == tag:
                self.reg_file[dest] = value
                self.reg_stat[dest] = 0
            
            for rs in self.add_rs + self.mul_rs:
                if rs.get('qj') == tag:
                    rs['vj'] = value
                    rs['qj'] = 0
                if rs.get('qk') == tag:
                    rs['vk'] = value
                    rs['qk'] = 0

    def run(self):
        cycle = 0
        while self.program or any(rs['busy'] for rs in self.add_rs + self.mul_rs):
            cycle += 1
            print(f"--- Cycle {cycle} ---")
            self.write_result()
            self.execute()
            self.issue()
        print("--- Simulation Finished ---")
        print("Register File:", self.reg_file)

# Example Program
program = [
    {'op': 'ADD', 'dest': 1, 'src1': 2, 'src2': 3},
    {'op': 'MUL', 'dest': 4, 'src1': 1, 'src2': 5},
    {'op': 'SUB', 'dest': 6, 'src1': 4, 'src2': 2}
]

sim = TomasuloSimulator(program)
# Initial register values
sim.reg_file[2] = 2.0
sim.reg_file[3] = 4.0
sim.reg_file[5] = 1.0
sim.run()
```

### 1.4. Assurer des Exceptions Précises : Le Rôle du Reorder Buffer (ROB)

L'algorithme de Tomasulo, dans sa forme originale, maximise les performances au détriment d'une caractéristique essentielle des processeurs modernes : la capacité à gérer des **exceptions précises**. Si une instruction cause une erreur (par exemple, une division par zéro ou un accès à une adresse mémoire invalide) au milieu d'une exécution OoO, l'état de la machine est incohérent. Des instructions qui n'auraient jamais dû être exécutées (celles qui suivent l'instruction fautive dans l'ordre du programme) peuvent déjà avoir modifié des registres. Il devient impossible de sauvegarder un état "propre" de la machine au point de l'exception pour que le système d'exploitation puisse la traiter. C'est ce qu'on appelle une **exception imprécise**, un compromis d'ingénierie jugé acceptable dans les années 1960 mais intolérable dans les systèmes modernes.

La solution à ce problème est le **Reorder Buffer (ROB)**, ou tampon de réordonnancement. Le ROB est une structure matérielle, généralement implémentée comme une file d'attente circulaire (FIFO), qui a été ajoutée aux conceptions OoO pour restaurer l'ordre au moment final de l'exécution. Son rôle est de s'assurer que, bien que les instructions s'exécutent dans le désordre, elles sont **validées** (committed) dans l'ordre strict du programme original.

Le fonctionnement du pipeline avec un ROB se déroule comme suit :

1. **Dispatch/Issue** : Lorsqu'une instruction est décodée, elle se voit allouer une entrée à la fin (la "queue") du ROB, en plus de sa station de réservation. Cela se fait dans l'ordre du programme.

2. **Execute** : L'instruction s'exécute dans le désordre, comme dans l'algorithme de Tomasulo.

3. **Write Result** : Lorsqu'elle termine, l'instruction écrit son résultat non pas directement dans le banc de registres architectural, mais dans son entrée correspondante dans le ROB. Elle y marque également son état comme "terminé".

4. **Commit** : Une logique de validation examine en permanence la tête du ROB. Si l'instruction à la tête du ROB est marquée comme "terminée" et n'a pas généré d'exception, elle est validée : son résultat est alors copié du ROB vers le banc de registres architectural, rendant la modification de l'état visible au programme. L'entrée est ensuite libérée du ROB. Si l'instruction à la tête a généré une exception, le processeur déclenche une vidange du pipeline (flush) : toutes les instructions qui la suivent dans le ROB (et qui sont donc plus jeunes dans l'ordre du programme) sont annulées, leurs résultats sont jetés, et l'état architectural est restauré à un point précis et correct avant de passer la main au système d'exploitation.

Le ROB ne se contente pas de résoudre le problème des exceptions précises ; il modifie fondamentalement la gestion des registres et de l'état spéculatif. Dans une conception moderne, le ROB ne stocke plus les valeurs des résultats elles-mêmes. Au lieu de cela, le processeur utilise un grand **banc de registres physiques** (Physical Register File - PRF) unifié qui contient toutes les valeurs, qu'elles soient spéculatives ou validées. Le renommage de registres mappe alors les registres architecturaux (visibles par le programmeur) à des registres physiques dans le PRF. Le ROB, dans ce modèle, ne contient plus que des pointeurs vers ces registres physiques, ainsi que des informations de statut.

Cette évolution architecturale est subtile mais profonde. Le ROB devient le chef d'orchestre de l'état spéculatif de la machine. Il permet au processeur de maintenir deux visions du monde en parallèle : un état architectural "connu et sûr" (défini par les registres validés) et un état spéculatif "potentiel" (défini par le contenu du ROB et du PRF). L'étape de "commit" devient l'opération atomique qui fait passer une partie de l'état potentiel dans l'état sûr. C'est cette abstraction puissante qui permet aux processeurs modernes de spéculer sur des centaines d'instructions en vol tout en garantissant une reprise correcte en cas d'erreur.

### 1.5. Étude de Cas : L'architecture P6 de l'Intel Pentium Pro

Le processeur Intel Pentium Pro, lancé en 1995, est l'exemple paradigmatique de la commercialisation réussie et à grande échelle de l'exécution OoO. Il a été le premier à introduire la microarchitecture P6, une conception qui a influencé les processeurs Intel pendant plus d'une décennie.

La caractéristique la plus remarquable du Pentium Pro était sa capacité à prendre des instructions x86, qui font partie d'un jeu d'instructions complexe (CISC), et à les traduire dynamiquement en une séquence de **micro-opérations** (μops) plus simples, de type RISC. Cette traduction était effectuée par des décodeurs complexes à l'avant du pipeline. Une fois traduites, ces μops étaient injectées dans un moteur d'exécution OoO très sophistiqué, directement inspiré des principes de Tomasulo et des extensions avec ROB.

Ce moteur comprenait :

- Un renommage de registres pour éliminer les fausses dépendances entre les μops.
- Des stations de réservation (appelées "Reserve Station" ou RS) où les μops attendaient leurs opérandes.
- Un Reorder Buffer (ROB) pour suivre l'état de toutes les μops en vol et assurer la validation dans l'ordre et les exceptions précises.
- De multiples unités d'exécution (deux pour les entiers, une pour la virgule flottante, etc.) pour exploiter le parallélisme découvert.

Le pipeline du Pentium Pro était particulièrement profond pour l'époque, avec 14 étages, ce qui lui permettait d'atteindre des fréquences d'horloge élevées tout en gérant la complexité de la traduction et de l'exécution OoO. Le Pentium Pro a démontré de manière spectaculaire que les principes de l'exécution OoO pouvaient être appliqués avec succès à un ISA aussi complexe et répandu que le x86, établissant ainsi une nouvelle norme pour la conception de processeurs haute performance.

### Tableau 1: Trace d'Exécution sur une Machine Tomasulo avec ROB

Pour visualiser le fonctionnement dynamique de ces mécanismes, examinons une trace cycle par cycle d'une courte séquence de code sur un processeur OoO hypothétique.

**Code d'exemple :**
```
MUL.D F0, F2, F4 (Latence: 10 cycles)
SUB.D F8, F6, F2 (Latence: 2 cycles)
ADD.D F6, F8, F2 (Latence: 2 cycles)
```

**Hypothèses :** Un processeur scalaire (1 instruction émise/cycle), 2 RS pour ADD/SUB, 2 RS pour MUL/DIV. Le banc de registres initial contient des valeurs pour F2, F4, F6.

| Cycle | Instruction | Étape | État des Stations de Réservation (Op, Vj, Vk, Qj, Qk) | État du ROB (Busy, State, Dest, Value) | Activité du CDB |
|-------|-------------|-------|-------------------------------------------------------|--------------------------------------|-----------------|
| 1 | MUL.D F0,F2,F4 | Issue | MUL1(MUL, val(F2), val(F4), 0, 0) | ROB1(Y, Issue, F0,?) | - |
| 2 | SUB.D F8,F6,F2 | Issue | ADD1(SUB, val(F6), val(F2), 0, 0) | ROB2(Y, Issue, F8,?) | - |
| 3 | ADD.D F6,F8,F2 | Issue | ADD2(ADD,?, val(F2), tag(ROB2), 0) | ROB3(Y, Issue, F6,?) | - |
| 4 | SUB.D | Execute | ADD1 commence l'exécution (2 cycles) | ROB2(Y, Execute,...) | - |
| 5 | SUB.D | Write Result | ADD1 termine | ROB2(Y, Write,..., val(F8)) | tag(ROB2), val(F8) |
| 6 | ADD.D | Execute | ADD2 reçoit val(F8) du CDB, commence l'exécution | ROB3(Y, Execute,...) | - |
| 7 | ADD.D | Write Result | ADD2 termine | ROB3(Y, Write,..., val(F6)) | tag(ROB3), val(F6) |
| ... | ... | ... | ... | ... | ... |
| 11 | MUL.D | Write Result | MUL1 termine (après 10 cycles) | ROB1(Y, Write,..., val(F0)) | tag(ROB1), val(F0) |
| 12 | MUL.D | Commit | ROB1 est à la tête et prêt | ROB1(N, Commit,...) → Reg[F0] mis à jour | - |
| 13 | SUB.D | Commit | ROB2 est à la tête et prêt | ROB2(N, Commit,...) → Reg[F8] mis à jour | - |
| 14 | ADD.D | Commit | ROB3 est à la tête et prêt | ROB3(N, Commit,...) → Reg[F6] mis à jour | - |

Cette trace illustre parfaitement les concepts clés :

- **Exécution dans le désordre** : SUB.D et ADD.D terminent leur exécution (cycles 5 et 7) bien avant MUL.D (cycle 11), même s'ils ont été émis après.
- **Tolérance à la latence** : Pendant que MUL.D occupait son unité fonctionnelle, les autres instructions ont pu s'exécuter, utilisant l'additionneur/soustracteur.
- **Renommage et CDB** : ADD.D a attendu le tag de ROB2 (produit par SUB.D) et a capturé sa valeur depuis le CDB au cycle 6 pour pouvoir démarrer.
- **Validation dans l'ordre (Commit)** : Malgré l'exécution dans le désordre, les résultats sont écrits dans le banc de registres architectural dans l'ordre strict du programme original (F0, puis F8, puis F6) aux cycles 12, 13 et 14.

## Leçon 2 : L'Exécution Superscalaire

Si l'exécution OoO est une technique pour mieux utiliser le temps en tolérant les latences, l'exécution superscalaire est une technique pour mieux utiliser l'espace matériel en faisant plus de choses en même temps.

### 2.1. Le Principe : Élargir le Pipeline

Un processeur scalaire est limité à initier, au mieux, une seule instruction par cycle d'horloge. Un processeur superscalaire brise cette barrière en étant capable d'initier (fetch, decode, issue) plusieurs instructions par cycle. Un processeur dit "N-large" (ou "N-way") peut traiter jusqu'à N instructions simultanément à chaque étape du pipeline. L'objectif est simple et direct : multiplier le débit maximal théorique par N, et donc augmenter l'IPC de 1 à N.

Il est crucial de distinguer l'exécution superscalaire du pipelining :

- Le **pipelining** augmente le débit en parallélisant les étapes de différentes instructions au sein d'un même flux. À chaque cycle, une instruction avance d'une étape.
- L'**exécution superscalaire** augmente le débit en dupliquant les pipelines eux-mêmes, permettant à plusieurs instructions complètes d'entrer et de progresser dans le processeur en parallèle.

Les deux techniques sont presque toujours utilisées conjointement dans les processeurs modernes.

### 2.2. Microarchitecture d'un Processeur Superscalaire

Pour traiter N instructions en parallèle, il est nécessaire de dupliquer les ressources matérielles à travers le pipeline. Cela inclut :

- Un **fetcher plus large** : capable de lire N instructions depuis le cache d'instructions à chaque cycle.
- **N décodeurs d'instructions** fonctionnant en parallèle.
- Un **banc de registres avec plus de ports** : Si on veut exécuter N instructions qui ont chacune 2 opérandes sources et 1 destination, il faut un banc de registres capable de fournir 2N lectures et N écritures simultanément.
- De **multiples unités fonctionnelles** : Plusieurs ALUs, plusieurs FPU, etc.
- Des **ports multiples vers le cache de données** pour gérer plusieurs opérations de chargement/stockage (load/store) par cycle.

Le coût en matériel est donc significatif. Cependant, le défi le plus complexe n'est pas la duplication des ressources, mais la gestion des dépendances. En plus des dépendances "horizontales" (entre des instructions à différents stades du pipeline, gérées par le forwarding), un processeur superscalaire doit gérer les dépendances "verticales" : celles qui existent entre les N instructions qui sont décodées dans le même cycle d'horloge. 

Par exemple, si l'on décode simultanément `ADD R1, R2, R3` et `SUB R4, R1, R5`, la seconde instruction dépend de la première. La logique de décodage doit détecter cette dépendance instantanément et s'assurer que les instructions sont traitées correctement. Cette logique de vérification des dépendances $N \times N$ est complexe, coûteuse en silicium et peut allonger le chemin critique du processeur, ce qui peut potentiellement limiter la fréquence d'horloge maximale.

### 2.3. Implémentation d'un Dispatcher Superscalaire Simple

Pour illustrer la complexité des dépendances verticales, concevons la logique d'un dispatcher 2-large in-order. Ce dispatcher reçoit un paquet de deux instructions et doit décider si elles peuvent être envoyées aux unités d'exécution dans le même cycle.

#### 2.3.1. Pseudo-code

```pseudocode
// Prend en entrée un paquet de deux instructions, inst1 étant la plus ancienne
function can_dispatch_parallel(inst1, inst2):
    // Vérification de dépendance RAW (Read-After-Write)
    // inst2 lit un registre que inst1 écrit. C'est la dépendance la plus commune.
    if (inst2.src1 == inst1.dest) or (inst2.src2 == inst1.dest):
        return false

    // Vérification de dépendance WAW (Write-After-Write)
    // inst2 écrit dans le même registre que inst1. Doit être sérialisé.
    if (inst1.writes_reg and inst2.writes_reg and inst1.dest == inst2.dest):
        return false

    // Vérification de dépendance WAR (Write-After-Read)
    // inst2 écrit dans un registre que inst1 lit.
    if (inst1.src1 == inst2.dest) or (inst1.src2 == inst2.dest):
        return false

    // Vérification d'aléa structurel
    // Par exemple, s'il n'y a qu'une seule unité de division.
    if (is_div(inst1) and is_div(inst2)):
        return false

    return true
```

**Note :** Dans un processeur OoO avec renommage de registres, les dépendances WAR et WAW sont éliminées, simplifiant cette logique. La dépendance RAW reste la contrainte principale.

#### 2.3.2. Implémentation en C/C++

```cpp
struct InstructionFields {
    bool writes_reg;
    int dest_reg = -1;
    int src1_reg = -1;
    int src2_reg = -1;
    // ... autres champs comme le type d'opération
};

bool can_dispatch_parallel(const InstructionFields& inst1, const InstructionFields& inst2) {
    // RAW
    if (inst1.writes_reg && (inst2.src1_reg == inst1.dest_reg || inst2.src2_reg == inst1.dest_reg)) {
        return false;
    }
    // WAW
    if (inst1.writes_reg && inst2.writes_reg && inst1.dest_reg == inst2.dest_reg) {
        return false;
    }
    // WAR
    if (inst2.writes_reg && (inst1.src1_reg == inst2.dest_reg || inst1.src2_reg == inst2.dest_reg)) {
        return false;
    }
    // ... vérification d'aléa structurel
    
    return true;
}
```

#### 2.3.3. Implémentation en Python

```python
class SuperscalarDispatcher:
    def __init__(self, width=2):
        self.width = width

    def check_dependencies(self, inst1, inst2):
        """ Vérifie si inst2 dépend de inst1 """
        # RAW
        if inst1.get('dest') is not None and inst1['dest'] in [inst2.get('src1'), inst2.get('src2')]:
            return True
        # WAW
        if inst1.get('dest') is not None and inst1['dest'] == inst2.get('dest'):
            return True
        # WAR
        if inst2.get('dest') is not None and inst2['dest'] in [inst1.get('src1'), inst1.get('src2')]:
            return True
        return False

    def get_dispatch_groups(self, instruction_bundle):
        """ Groupe les instructions qui peuvent être distribuées ensemble """
        if not instruction_bundle:
            return []
        
        groups = []
        current_group = [instruction_bundle[0]]
        
        for i in range(1, len(instruction_bundle)):
            can_add = True
            for inst_in_group in current_group:
                if self.check_dependencies(inst_in_group, instruction_bundle[i]):
                    can_add = False
                    break
            if can_add and len(current_group) < self.width:
                current_group.append(instruction_bundle[i])
            else:
                groups.append(current_group)
                current_group = [instruction_bundle[i]]
        
        groups.append(current_group)
        return groups
```

### 2.4. Synergie et Orthogonalité : OoO + Superscalaire

Il est fondamental de comprendre que l'exécution OoO et l'exécution superscalaire sont des concepts **orthogonaux**, comme le souligne le cours. On peut concevoir un processeur qui est :

1. **In-order et scalaire** : Le pipeline de base (ex: MIPS R2000).
2. **In-order et superscalaire** : Il peut exécuter plusieurs instructions indépendantes par cycle, mais doit s'arrêter si la première instruction d'un groupe est bloquée (ex: Intel Pentium).
3. **OoO et scalaire** : Il ne peut décoder qu'une instruction par cycle, mais peut la placer dans une grande fenêtre d'instructions et l'exécuter dans le désordre par rapport aux précédentes.
4. **OoO et superscalaire** : La norme pour tous les processeurs haute performance aujourd'hui (ex: Intel Core, AMD Zen, Apple M-series).

La combinaison de l'OoO et du superscalaire crée une synergie extrêmement puissante, mais elle engendre également une complexité qui est plus que la somme de ses parties. La synergie vient du fait que l'exécution superscalaire agit comme un "fournisseur" à haut débit pour le moteur OoO. En décodant N instructions par cycle, le front-end du processeur remplit la fenêtre d'instructions beaucoup plus rapidement et avec une plus grande diversité d'instructions. Cela augmente de manière spectaculaire les chances que le planificateur (scheduler) OoO trouve des instructions indépendantes à envoyer aux multiples unités d'exécution, maintenant ainsi le processeur constamment occupé et maximisant l'IPC.

Cependant, cette synergie a un coût en termes de complexité. Le goulot d'étranglement se déplace souvent vers le front-end du processeur, en particulier la logique de renommage de registres. Comme nous l'avons vu, la logique de renommage doit traiter N instructions par cycle. Le problème est que ces N instructions peuvent avoir des dépendances entre elles. Considérons le décodage simultané de `ADD R1, R2, R3` et `SUB R4, R1, R5`. La logique de renommage doit traiter l'ADD en premier, lui allouer un nouveau registre physique pour R1, puis fournir ce nouveau registre physique comme opérande source pour le SUB. Tout cela doit se faire en un seul cycle d'horloge. Cela crée une chaîne de dépendances en série au sein même de la logique de renommage, qui devient l'un des chemins critiques les plus complexes et les plus difficiles à optimiser dans un processeur superscalaire large. C'est pourquoi la conception du front-end (fetch, decode, rename) est un domaine de recherche et d'ingénierie extrêmement actif.

## Leçon 3 : La Prédiction de Branchement : L'Art de la Divination Matérielle

L'exécution OoO et superscalaire donne au processeur la capacité d'exécuter un grand nombre d'instructions en parallèle. Mais cette capacité est inutile si le processeur ne sait pas quelles instructions exécuter. C'est le problème des aléas de contrôle, et la prédiction de branchement est la solution.

### 3.1. Le Problème du Contrôle : Une Question de Vie ou de Mort pour l'ILP

Les instructions de contrôle (branchements conditionnels, sauts indirects, appels de fonction) représentent une part significative de tout programme, typiquement 15 à 25%. Chaque fois qu'un branchement conditionnel est rencontré, le processeur ne sait pas quelle est la prochaine instruction à exécuter (l'instruction séquentielle suivante ou l'instruction à la cible du branchement) avant que la condition du branchement ne soit évaluée, ce qui peut prendre de nombreux cycles dans un pipeline profond.

Attendre la résolution du branchement n'est pas une option viable. L'analyse quantitative présentée dans le cours est particulièrement révélatrice. Considérons un processeur superscalaire 5-large ($W=5$) avec un pipeline où la pénalité de mauvaise prédiction est de 20 cycles ($N=20$).

- Avec une précision de prédiction de **100%**, pour exécuter 500 instructions, il faut $500/5=100$ cycles. L'IPC est de 5.0.
- Avec une précision de **99%**, sur 100 branchements, 1 est mal prédit. Le temps total est de $100 + 1 \times 20 = 120$ cycles. L'IPC chute à $500/120 \approx 4.17$. La performance a baissé, mais reste élevée.
- Avec une précision de **90%**, 10 branchements sont mal prédits. Le temps total est de $100 + 10 \times 20 = 300$ cycles. L'IPC s'effondre à $500/300 \approx 1.67$.
- Avec une précision de **60%**, 40 branchements sont mal prédits. Le temps total est de $100 + 40 \times 20 = 900$ cycles. L'IPC est un misérable $500/900 \approx 0.56$.

Cette analyse démontre sans équivoque que sans un prédicteur de branchement extrêmement précis (bien au-delà de 90%), les gains de performance promis par les architectures OoO et superscalaires sont complètement anéantis. La prédiction de branchement n'est pas une optimisation ; c'est une technologie fondamentale et indispensable.

Pour alimenter le pipeline, le processeur doit, au stade du fetch, prédire trois choses :

1. L'instruction est-elle un branchement ?
2. Si oui, quelle est sa direction (pris ou non-pris) ?
3. Si elle est prédite "prise", quelle est son adresse cible ?

### 3.2. Mécanismes de Base : Prédicteurs Statiques et Dynamiques Simples

Pour répondre aux questions 1 et 3, les processeurs modernes utilisent un **Branch Target Buffer (BTB)**. Le BTB est une petite mémoire cache qui est indexée par l'adresse de l'instruction (le PC). Il stocke les adresses cibles des branchements qui ont été exécutés récemment. Au stade du fetch, le PC est utilisé pour interroger le cache d'instructions et, en parallèle, le BTB. Si le PC produit un "hit" dans le BTB, cela signifie que l'instruction est très probablement un branchement, et le BTB fournit l'adresse cible prédite. Si la prédiction de direction (question 2) est "pris", cette adresse cible est utilisée pour le fetch du cycle suivant. Sinon, le processeur continue avec l'adresse séquentielle suivante (PC + taille de l'instruction).

Pour la prédiction de direction, il existe plusieurs approches, des plus simples aux plus complexes.

#### Prédicteurs Statiques (Compile-Time)

Ces prédicteurs suivent une règle fixe, déterminée avant l'exécution.

- **Toujours Non-Pris (Always Not-Taken)** : Simple, mais peu précis (30-40% de succès).
- **Toujours Pris (Always Taken)** : Plus efficace, car de nombreux branchements sont des branchements de fin de boucle qui sont pris de manière répétée. La précision est de l'ordre de 60-70%.
- **Backward-Taken/Forward-Not-Taken (BTFNT)** : Une heuristique simple mais efficace. Un branchement "vers l'arrière" (cible < PC) est probablement une boucle et est prédit "pris". Un branchement "vers l'avant" est prédit "non-pris". Améliore légèrement la précision par rapport à "toujours pris".
- **Indices du Compilateur (Profile-Based)** : Le compilateur analyse le programme avec des données de test (profiling) pour déterminer la direction la plus probable de chaque branche. Il encode cette information dans un bit d'indice au sein de l'instruction de branchement elle-même. Le matériel lit ce bit et prédit en conséquence. C'est plus précis, mais souffre du problème de la représentativité du profil : si le comportement du programme à l'exécution réelle diffère de celui observé lors du profilage, la prédiction peut devenir très mauvaise.

#### Prédicteurs Dynamiques Simples (Run-Time)

Ces prédicteurs utilisent l'historique récent de l'exécution pour s'adapter dynamiquement au comportement du programme.

- **Compteur à 1-bit** : Un tableau (PHT - Pattern History Table) indexé par les bits de poids faible du PC du branchement. Chaque entrée est un seul bit qui stocke l'issue du dernier passage : 1 pour pris, 0 pour non-pris. Le prédicteur prédit simplement la même chose que la dernière fois. **Problème** : pour une boucle qui s'exécute N fois, ce prédicteur se trompera deux fois : à la première itération (il prédit non-pris alors que la boucle est entrée) et à la dernière (il prédit pris alors que la boucle se termine).

- **Compteur à 2-bits à saturation** : C'est le standard de base pour la prédiction dynamique et une amélioration significative. Chaque entrée de la PHT est un petit automate à 4 états (typiquement encodé sur 2 bits) :
  - 00 : Fortement non-pris
  - 01 : Faiblement non-pris
  - 10 : Faiblement pris
  - 11 : Fortement pris

La prédiction est "pris" si le compteur est dans les états 10 ou 11, et "non-pris" sinon. Après la résolution du branchement, le compteur est incrémenté s'il a été pris (saturant à 11) et décrémenté s'il n'a pas été pris (saturant à 00). Il faut maintenant deux erreurs de prédiction consécutives pour changer une prédiction de "fortement pris" à "faiblement non-pris". Cela résout élégamment le problème de la dernière itération des boucles : la sortie unique de la boucle ne fait passer le compteur que de 11 à 10, et la prédiction pour la prochaine entrée dans la boucle reste "pris". La précision de ce type de prédicteur (appelé bimodal) atteint environ 90%.

### 3.3. Prédiction Adaptative à Deux Niveaux (Yeh & Patt, 1991)

La grande avancée suivante est venue de l'observation que la direction d'un branchement n'est pas seulement corrélée à son propre passé, mais aussi au patron de comportement des branchements globaux qui l'ont précédé. C'est l'idée fondamentale de la **prédiction adaptative à deux niveaux**.

Les deux composants clés sont :

1. **Le Registre d'Historique de Branchement (Branch History Register - BHR)** : C'est le premier niveau d'historique. C'est un registre à décalage de k bits qui enregistre l'issue (1=pris, 0=non-pris) des k derniers branchements exécutés.

2. **La Table d'Historique de Patrons (Pattern History Table - PHT)** : C'est le deuxième niveau. C'est une table de compteurs à 2-bits à saturation. Le contenu du BHR (le "patron" d'historique) est utilisé comme un index pour sélectionner une entrée dans cette PHT. C'est ce compteur qui fait la prédiction finale.

L'idée est que pour un même branchement statique, son comportement peut différer en fonction du chemin d'exécution qui y a mené. Le BHR capture une approximation de ce chemin.

Yeh et Patt ont proposé une taxonomie pour classer les différentes variantes de ces prédicteurs, basée sur la manière dont l'historique et les tables de patrons sont gérés :

**G (Global) vs. P (Per-address) pour le BHR :**
- **GA (Global History)** : Un seul BHR global est utilisé pour tous les branchements du programme.
- **PA (Per-Address History)** : Chaque branchement statique (identifié par son PC) possède son propre BHR privé.

**g (global) vs. p (per-address) vs. s (set) pour la PHT :**
- **g (global PHT)** : Une seule PHT est partagée par tous les BHR.
- **p (per-address PHT)** : Chaque BHR a sa propre PHT privée (extrêmement coûteux en matériel).
- **s (set PHT)** : Un compromis où des ensembles de branchements partagent une même PHT.

Le schéma **GAg** (Global History, Global PHT) utilise un BHR global pour indexer une PHT globale. C'est simple, mais souffre d'un problème majeur : l'**aliasing** (ou interférence). Deux branchements différents qui se produisent après le même patron d'historique global vont utiliser (et potentiellement corrompre) la même entrée dans la PHT, même si leur comportement est totalement décorrélé.

Une amélioration cruciale a été proposée par Scott McFarling sous le nom de **gshare**. Gshare est une variante de GAg qui réduit considérablement l'aliasing avec un coût matériel minimal. Au lieu d'utiliser seulement le BHR pour indexer la PHT, gshare calcule l'index en faisant un OU exclusif (XOR) entre le BHR et le PC du branchement : 

$$\text{index} = \text{BHR} \oplus \text{PC}$$

Cette simple opération combine l'information de contexte global (le BHR) avec l'information locale (le PC du branchement). Ainsi, deux branchements différents, même s'ils suivent le même patron d'historique global, auront des PC différents et donc mapperont très probablement vers des entrées différentes dans la PHT, réduisant l'interférence et améliorant la précision. Ce simple ajout d'une porte XOR s'est avéré si efficace qu'il est devenu une technique de base dans de nombreux prédicteurs modernes.

### 3.4. Implémentation d'un Prédicteur Gshare

Implémentons un prédicteur gshare pour illustrer son fonctionnement.

#### 3.4.1. Pseudo-code

```pseudocode
// Constantes
PHT_SIZE = 4096 // Taille de la table de patrons (doit être une puissance de 2)
HISTORY_LENGTH = 12 // Longueur du registre d'historique global

// Structures de données
pht = array of 2-bit saturating counters, initialized to weak not-taken (01)
global_history_register = 0 (integer of HISTORY_LENGTH bits)

function predict(PC):
    // Calculer l'index en combinant l'historique et le PC
    index = (global_history_register XOR (PC & (PHT_SIZE - 1))) % PHT_SIZE
    
    // Prédire "pris" si le compteur est dans un état "pris" (10 ou 11)
    return pht[index] >= 2

function update(PC, actual_outcome_is_taken):
    // Calculer le même index que pour la prédiction
    index = (global_history_register XOR (PC & (PHT_SIZE - 1))) % PHT_SIZE
    
    // Mettre à jour le compteur à 2-bits
    if actual_outcome_is_taken:
        pht[index] = min(3, pht[index] + 1) // Incrémenter, saturer à 3 (fortement pris)
    else:
        pht[index] = max(0, pht[index] - 1) // Décrémenter, saturer à 0 (fortement non-pris)

    // Mettre à jour le registre d'historique global
    global_history_register = ((global_history_register << 1) | actual_outcome_is_taken)
    // Garder seulement les HISTORY_LENGTH derniers bits
    history_mask = (1 << HISTORY_LENGTH) - 1
    global_history_register = global_history_register & history_mask
```

#### 3.4.2. Implémentation en C/C++

```cpp
#include <vector>
#include <cstdint>

class GsharePredictor {
public:
    GsharePredictor(int pht_size_bits = 12, int history_len = 12)
        : history_length(history_len),
          pht(1 << pht_size_bits, 1), // Initialize to weak not-taken
          global_history(0) {
        history_mask = (1 << history_length) - 1;
    }

    bool predict(uint64_t pc) {
        uint32_t index = get_index(pc);
        return pht[index] >= 2;
    }

    void update(uint64_t pc, bool taken) {
        uint32_t index = get_index(pc);
        
        if (taken) {
            if (pht[index] < 3) pht[index]++;
        } else {
            if (pht[index] > 0) pht[index]--;
        }

        global_history = ((global_history << 1) | (taken ? 1 : 0)) & history_mask;
    }

private:
    uint32_t get_index(uint64_t pc) {
        return (global_history ^ (pc & ((1 << history_length) - 1))) % pht.size();
    }

    int history_length;
    uint64_t history_mask;
    std::vector<uint8_t> pht; // Each element is a 2-bit counter
    uint64_t global_history;
};
```

#### 3.4.3. Implémentation en Python

```python
class GsharePredictor:
    def __init__(self, pht_size_bits=12, history_len=12):
        self.history_length = history_len
        self.pht_size = 1 << pht_size_bits
        self.pht = [1] * self.pht_size  # 00, 01, 10, 11 -> weak not-taken
        self.history_mask = (1 << self.history_length) - 1
        self.bhr = 0  # Branch History Register

    def _get_index(self, pc):
        # XOR the lower bits of PC with the global history
        return (self.bhr ^ (pc & self.history_mask)) % self.pht_size

    def predict(self, pc):
        index = self._get_index(pc)
        return self.pht[index] >= 2  # Predict taken if weak or strong taken

    def update(self, pc, taken):
        index = self._get_index(pc)
        
        # Update 2-bit saturating counter
        if taken:
            self.pht[index] = min(3, self.pht[index] + 1)
        else:
            self.pht[index] = max(0, self.pht[index] - 1)
            
        # Update global history register
        self.bhr = ((self.bhr << 1) | int(taken)) & self.history_mask
```

### 3.5. Prédicteurs Hybrides et à Tournoi (McFarling, 1993)

Même les prédicteurs sophistiqués comme gshare ont leurs faiblesses. L'observation suivante est clé : aucun prédicteur unique n'est optimal pour tous les types de branchements dans tous les programmes. Certains branchements sont mieux prédits par leur historique local (schémas PA), d'autres par le contexte global (schémas GA).

L'idée des **prédicteurs hybrides**, ou **à tournoi**, est de combiner plusieurs stratégies de prédiction différentes et de choisir dynamiquement la meilleure pour chaque branchement. Un prédicteur à tournoi typique, comme celui popularisé par McFarling, pourrait combiner un prédicteur bimodal simple avec un prédicteur gshare plus complexe.

Le mécanisme fonctionne comme suit :

1. Pour chaque prédiction, les deux prédicteurs (par ex., bimodal et gshare) génèrent leur propre prédiction.

2. Une troisième table, la **table de sélection** (selector table), est utilisée. Cette table est également indexée par le PC du branchement.

3. Chaque entrée de la table de sélection est un compteur à 2-bits qui garde une trace de quel prédicteur (le bimodal ou le gshare) a été le plus précis pour ce branchement spécifique dans le passé.

4. La prédiction finale est celle du prédicteur désigné comme "gagnant" par la table de sélection.

5. Lorsque l'issue réelle du branchement est connue, non seulement les prédicteurs bimodal et gshare sont mis à jour, mais la table de sélection est également mise à jour. Si gshare avait raison et bimodal tort, le compteur de sélection est incrémenté vers "préférer gshare". Si l'inverse est vrai, il est décrémenté. Si les deux ont raison ou les deux ont tort, le compteur reste inchangé.

Cette approche permet au processeur de s'adapter de manière très fine, en utilisant un prédicteur simple et stable pour les branchements à comportement régulier, et un prédicteur plus complexe et contextuel pour les branchements difficiles, maximisant ainsi la précision globale. Les prédicteurs à la pointe de la technologie, comme TAGE, sont des formes très évoluées de cette idée, combinant de multiples tables de prédiction avec des longueurs d'historique variables.

### Tableau 2: Tableau Comparatif des Prédicteurs de Branchement

| Nom du Prédicteur | Modèle Conceptuel | Matériel Clé | Précision Typique (%) | Coût/Complexité Relatif |
|-------------------|-------------------|--------------|----------------------|------------------------|
| Statique (BTFNT) | Les boucles sont prises, le reste non. | Aucun (logique de comparaison de PC) | ~65% | Très faible |
| 2-bit Bimodal | L'historique récent du branchement lui-même est le meilleur prédicteur. | BTB + PHT | ~90-93% | Faible |
| GAg (Global) | Le patron des derniers branchements globaux détermine l'issue. | BTB + BHR + PHT | ~92-94% | Modéré |
| Gshare | Le contexte global combiné à l'identité de la branche réduit l'aliasing. | BTB + BHR + PHT + XOR | ~94-96% | Modéré |
| PAs (Per-Address) | L'historique local de chaque branche est utilisé pour indexer une PHT partagée. | BTB + Table de BHRs + PHT | ~95-97% | Élevé |
| Hybride (Tournoi) | Choisir dynamiquement le meilleur prédicteur (ex: local vs global) pour chaque branche. | 2×(Prédicteurs) + Table de Sélection | >97% | Très élevé |

## Conclusion : Synthèse et Perspectives sur les Microarchitectures Modernes

Au terme de cette exploration, il apparaît clairement que les trois piliers que nous avons étudiés — l'exécution dans le désordre, l'exécution superscalaire et la prédiction de branchement — ne sont pas des technologies isolées. Ce sont des composants profondément interdépendants et synergiques qui forment le cœur de la quasi-totalité des processeurs haute performance conçus aujourd'hui, des puces de nos smartphones aux processeurs massifs des supercalculateurs.

On peut visualiser leur interaction comme un **cycle vertueux de la performance** :

- L'**exécution superscalaire** agit comme une pompe à haute pression, fournissant au moteur d'exécution un flux large et constant d'instructions à traiter.

- L'**exécution dans le désordre** prend ce flux, le stocke dans une large fenêtre d'instruction et agit comme un planificateur intelligent, réorganisant les tâches pour trouver tout le parallélisme possible et maintenir les multiples unités d'exécution constamment actives.

- Cependant, toute cette machinerie complexe et coûteuse est entièrement dépendante de la **prédiction de branchement**. C'est le système de navigation qui guide la pompe et le planificateur. Une seule erreur de navigation force l'arrêt brutal, la vidange et le redémarrage, gaspillant un travail considérable et anéantissant les gains de performance.

La conception d'un processeur moderne est donc un exercice d'équilibre délicat entre ces trois forces. Augmenter la largeur superscalaire n'est utile que si la fenêtre d'instruction et le moteur OoO peuvent gérer le flux, et si le prédicteur de branchement est suffisamment précis pour que ce flux soit majoritairement correct.

Aujourd'hui, les architectes sont confrontés à de nouveaux défis. La fin de la loi de Moore telle que nous la connaissions, et surtout le "mur de la puissance" (power wall) qui rend prohibitif l'augmentation de la fréquence et de la complexité d'un seul cœur, poussent la recherche vers de nouvelles frontières. Le parallélisme est désormais exploité à des niveaux plus élevés : les processeurs multi-cœurs, le multithreading simultané (SMT), et l'utilisation croissante d'accélérateurs matériels spécialisés (pour les graphismes, l'IA, etc.) sont les nouvelles voies pour continuer la quête de performance. Cependant, au cœur de chacun de ces cœurs de processeur, les principes fondamentaux de l'exécution OoO, superscalaire et de la prédiction de branchement restent plus pertinents que jamais.
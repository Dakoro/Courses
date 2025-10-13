# Programmation Lock-Free en C++ Moderne : Des Fondations Atomiques aux Frontières de la Recherche

## Introduction : Au-delà du Mutex

Bonjour à toutes et à tous. Notre conférence d'aujourd'hui s'aventure dans un domaine de la programmation concurrente à la fois puissant et notoirement complexe : le paradigme lock-free. Depuis des décennies, le modèle de synchronisation dominant repose sur l'utilisation de verrous, ou mutex. Ces mécanismes sont relativement simples à comprendre : pour accéder à une ressource partagée, un thread doit d'abord acquérir un verrou ; une fois son travail terminé, il le relâche, permettant à un autre thread de prendre le relais.

Cependant, cette simplicité apparente masque des problèmes fondamentaux qui peuvent devenir des obstacles majeurs dans les systèmes à haute performance. Les verrous sont une source notoire d'interblocages (deadlocks), où deux threads ou plus s'attendent mutuellement, gelant une partie ou la totalité du système. Ils peuvent engendrer des blocages actifs (livelocks), où les threads sont actifs mais ne parviennent pas à progresser. Ils sont également sujets à l'inversion de priorité, un scénario dans lequel un thread de haute priorité est bloqué par un thread de basse priorité détenant un verrou. Plus important encore, dans les architectures multi-cœurs modernes, les verrous introduisent une contention qui limite drastiquement la scalabilité. Lorsqu'un grand nombre de threads tentent d'accéder à la même ressource, ils passent plus de temps à attendre le verrou qu'à effectuer un travail utile, créant un goulot d'étranglement qui annule les bénéfices du parallélisme.

La programmation lock-free se présente comme une alternative radicale. Son principe fondamental est de garantir que le système, dans son ensemble, continue de progresser à tout moment. Si un thread est suspendu par le système d'exploitation au milieu d'une opération, cela n'empêchera pas les autres threads de continuer leur travail. Cette approche élimine par conception les deadlocks liés aux verrous et promet une scalabilité bien supérieure sur les systèmes massivement parallèles.

Néanmoins, il est crucial de le dire d'emblée : cette quête de performance a un coût. La complexité de la conception, de l'implémentation et de la validation des algorithmes lock-free est d'un ordre de grandeur supérieur à celle des algorithmes basés sur les verrous. L'intuition, notre meilleur guide en programmation séquentielle, devient notre pire ennemie dans le monde lock-free. Des erreurs subtiles, quasi impossibles à détecter par des tests traditionnels, peuvent corrompre silencieusement les données et mener à des défaillances catastrophiques.

L'objectif de cette conférence est de démystifier cette complexité. Nous allons construire une compréhension rigoureuse et structurée de la programmation lock-free. Nous commencerons par établir les fondations théoriques en définissant précisément les garanties de progression. Nous disséquerons ensuite les problèmes classiques, tels que le fameux problème ABA et la gestion de la mémoire. Nous explorerons l'arsenal des solutions modernes offertes par le C++ et les recherches académiques, avant de plonger dans les détails du modèle mémoire et du rôle crucial du matériel. Enfin, nous aborderons les techniques de vérification formelle et conclurons par un aperçu des recherches les plus récentes qui repoussent les frontières de ce domaine fascinant.

## Partie I : Les Garanties de Progression Sans Verrou

Pour discuter de manière rigoureuse des algorithmes non-bloquants, il est indispensable d'établir un vocabulaire formel. La simple affirmation "cet algorithme est lock-free" est souvent une simplification abusive. Il existe en réalité une hiérarchie de garanties de progression, chacune avec des implications précises sur le comportement et la complexité de l'algorithme. Cette hiérarchie, parfois attribuée à des auteurs comme Herb Sutter ou Tony Vanert, trouve en réalité ses racines dans des travaux académiques bien plus anciens, remontant potentiellement aux années 1960.

### La Hiérarchie des Garanties

#### Obstruction-Free

C'est la garantie la plus faible de la hiérarchie. Un algorithme est dit obstruction-free si, à n'importe quel moment, un thread unique qui n'est pas suspendu est garanti de terminer son opération en un nombre fini d'étapes. L'hypothèse clé est l'absence d'obstruction : si tous les autres threads sont mis en pause, le thread actif peut progresser.

Pour illustrer ce qui n'est pas obstruction-free, considérons un programme utilisant un simple mutex. Supposons qu'un thread T1 tente d'acquérir un mutex déjà détenu par un thread T2. Si nous suspendons T2 (et tous les autres threads), T1 restera bloqué indéfiniment en attente du mutex. Il ne peut pas progresser seul. Par conséquent, tout algorithme basé sur des verrous n'est, par définition, même pas obstruction-free.

#### Lock-Free

La garantie lock-free est plus forte et constitue le cœur de notre sujet. Un algorithme est lock-free si, à tout instant, il garantit qu'au moins un des threads actifs progresse en un nombre fini d'étapes. Cela signifie que le système dans son ensemble ne peut jamais être "gelé". Même si certains threads sont ralentis ou échouent à plusieurs reprises dans leurs opérations, au moins un autre thread réussira, assurant ainsi une progression globale. C'est une garantie sur le système, pas sur chaque thread individuel.

#### Wait-Free

La garantie wait-free est la plus forte et la plus difficile à atteindre. Un algorithme est wait-free si chaque thread est garanti de terminer son opération en un nombre fini de ses propres étapes, indépendamment de la vitesse ou de l'activité des autres threads. Cette garantie élimine la possibilité de famine (starvation), où un thread "malchanceux" pourrait échouer indéfiniment à cause de l'interférence constante d'autres threads plus rapides. Atteindre le wait-free impose des contraintes algorithmiques extrêmement sévères.

Le choix d'une de ces garanties n'est pas une simple question de classification théorique ; il dicte les outils et les motifs de conception que nous avons le droit d'utiliser. Comme nous le verrons, la garantie wait-free est souvent qualifiée de "sombre" car elle interdit l'utilisation de la technique la plus fondamentale du monde lock-free : la boucle Compare-and-Swap. Une telle boucle ne peut pas garantir qu'un thread finira par réussir en un nombre borné de tentatives, ce qui viole la définition même du wait-free.

### L'Opération Compare-and-Swap (CAS)

Au cœur de la quasi-totalité des algorithmes lock-free se trouve une opération atomique fondamentale : le Compare-and-Swap, ou CAS. En C++, cette opération est fournie par les méthodes `compare_exchange_weak` et `compare_exchange_strong` de la classe `std::atomic`.

Son fonctionnement est précis et doit être parfaitement compris. L'opération prend trois paramètres :

1. La valeur **expected** (attendue) : la valeur que nous pensons que la variable atomique contient actuellement.
2. La valeur **desired** (désirée) : la nouvelle valeur que nous voulons écrire.
3. La variable atomique elle-même (implicitement `*this`).

L'opération se déroule atomiquement de la manière suivante :

- **En cas de succès** : Si la valeur actuelle de la variable atomique est égale à `expected`, elle est remplacée par `desired`, et la fonction retourne `true`.
- **En cas d'échec** : Si la valeur actuelle de la variable atomique est différente de `expected`, la variable n'est pas modifiée. À la place, le paramètre `expected` est mis à jour avec la valeur actuelle de la variable atomique, et la fonction retourne `false`.

Cette mise à jour de `expected` en cas d'échec est cruciale. Elle nous informe de la valeur qui a causé l'échec et nous permet de retenter l'opération avec une information à jour. C'est ce qui donne naissance au motif de conception le plus courant de la programmation lock-free : la boucle CAS.

```cpp
std::atomic<Value> atomic_val;
Value current_val = atomic_val.load();
Value new_val;
do {
    // Préparez new_val en se basant sur current_val
    new_val = compute_next_state(current_val);
} while (!atomic_val.compare_exchange_weak(current_val, new_val));
```

Dans cette boucle, nous lisons la valeur actuelle, calculons une nouvelle valeur, puis tentons de la "commiter" atomiquement. Si un autre thread a modifié `atomic_val` entre notre `load()` et notre `compare_exchange_weak`, le CAS échouera. La variable `current_val` sera automatiquement mise à jour avec la nouvelle valeur, et la boucle recommencera.

Il est souvent préférable d'utiliser `compare_exchange_weak` plutôt que `strong` à l'intérieur d'une boucle. La version `weak` est autorisée à échouer de manière "fallacieuse" (spuriously), c'est-à-dire même si la valeur n'a pas changé. Sur certaines architectures comme ARM, cela peut se traduire par une séquence d'instructions plus courte et plus performante. Comme nous sommes déjà dans une boucle, un échec fallacieux n'est pas un problème de correction ; il nous force simplement à faire une itération de plus.

## Partie II : La Pratique du Code Lock-Free : Pièges et Paradigmes

Armés de ces concepts fondamentaux, nous pourrions penser être prêts à écrire du code lock-free. Cependant, la pratique révèle rapidement que l'intuition développée en programmation séquentielle est un guide dangereux. Des algorithmes qui semblent parfaitement logiques peuvent cacher des bugs concurrents d'une subtilité extrême.

La complexité du code lock-free ne réside pas tant dans la logique de l'algorithme lui-même, qui est souvent simple, mais dans la gestion des états intermédiaires et des interactions invisibles entre les threads. Le programmeur doit raisonner non pas sur une séquence linéaire d'états, mais sur un graphe exponentiel d'états possibles, où les transitions sont les innombrables entrelacements (interleavings) des opérations des threads. Dans ce contexte, les outils de débogage traditionnels, comme les points d'arrêt, sont souvent inutiles, car ils altèrent l'ordonnancement des threads et peuvent masquer le bug que l'on cherche à trouver.

### Analyse de Cas : std::shared_ptr

Considérons l'implémentation du compteur de références d'un `shared_ptr`. Le code pour décrémenter le compteur et supprimer l'objet sous-jacent pourrait ressembler à ceci :

```cpp
// Le compteur 'count' est un std::atomic<int>
if (--count == 0) {
    delete control_block;
    delete managed_object;
}
```

À première vue, cela ressemble à une "race condition" classique. Que se passe-t-il si un autre thread incrémente le compteur juste après que notre thread l'a décrémenté à zéro, mais avant qu'il n'exécute le `delete`?

**Le destructeur : un faux problème.** Dans le contexte du destructeur d'un objet `std::shared_ptr`, ce n'est pas un bug. Par définition, si un thread exécute le destructeur de `ptr_A`, cela signifie que `ptr_A` est la dernière instance de `shared_ptr` pointant vers ce bloc de contrôle. Aucun autre thread ne peut légalement copier cet objet `ptr_A` pendant sa destruction. L'exclusivité est garantie par la sémantique du langage.

**L'opérateur d'affectation : un vrai problème.** Le même raisonnement s'effondre pour l'opérateur d'affectation (`operator=`). Il est tout à fait légal et courant qu'un thread A affecte une nouvelle valeur à un `shared_ptr` pendant qu'un thread B détruit une autre copie du `shared_ptr` original. Dans ce scénario, la séquence décrément-test-delete devient une véritable "race condition", car un constructeur de copie (qui incrémente) et un destructeur (qui décrémente) peuvent opérer simultanément sur le même bloc de contrôle. Des solutions existent, comme celle proposée par Shawn Parent, qui utilisent des séquences d'opérations atomiques plus complexes pour garantir la correction sans recourir à une boucle CAS, préservant ainsi une garantie wait-free.

### Analyse de Cas : Le DataHolder et le Réordonnancement

Imaginons une classe simple qui met en cache le résultat d'un calcul coûteux, comme une somme de contrôle.

```cpp
class DataHolder {
public:
    int getChecksum() const {
        if (!is_cached) {
            // Mauvais ordre!
            is_cached = true;
            checksum = compute_heavy_checksum();
        }
        return checksum;
    }
private:
    mutable bool is_cached = false;
    mutable int checksum;
    //...
};
```

En séquentiel, l'ordre des deux lignes dans le `if` n'a pas d'importance. En concurrent, c'est une "race condition" catastrophique. Un thread T1 peut exécuter `is_cached = true;` et être préempté. Un thread T2 arrive, voit `is_cached` à `true` et retourne la valeur non initialisée de `checksum`. Le bug ne vient pas d'une logique de calcul erronée, mais d'un ordonnancement incorrect des écritures. La solution correcte est d'inverser les deux lignes :

```cpp
// Ordre correct
checksum = compute_heavy_checksum();
is_cached = true;
```

Ces exemples illustrent une leçon fondamentale : en programmation lock-free, l'ordre des opérations, même celles qui semblent indépendantes, est d'une importance capitale.

### Implémentation Proposée : Une Pile (Stack) Lock-Free - Première Approche

Forts de ces avertissements, tentons d'implémenter notre première structure de données lock-free : une pile (stack) basée sur une liste chaînée.

```cpp
template<typename T>
class LockFreeStack {
private:
    struct Node {
        T data;
        Node* next;
    };
    std::atomic<Node*> m_head;

public:
    void push(const T& value) {
        Node* new_node = new Node{value, nullptr};
        new_node->next = m_head.load(std::memory_order_relaxed);
        while (!m_head.compare_exchange_weak(new_node->next, new_node,
                                             std::memory_order_release,
                                             std::memory_order_relaxed));
    }

    bool pop(T& result) {
        Node* old_head = m_head.load(std::memory_order_relaxed);
        while (old_head && !m_head.compare_exchange_weak(old_head, old_head->next,
                                                        std::memory_order_release,
                                                        std::memory_order_relaxed));
        
        if (old_head) {
            result = old_head->data;
            delete old_head; // <-- Le problème est ici!
            return true;
        }
        return false;
    }
};
```

L'algorithme pour `push` et `pop` semble correct, utilisant la boucle CAS que nous avons étudiée. Cependant, la ligne `delete old_head;` est une bombe à retardement. Elle nous expose directement aux deux défis les plus fondamentaux de la programmation lock-free sur des structures à base de nœuds, que nous allons maintenant aborder en détail.

## Partie III : Les Deux Défis Majeurs : Le Problème ABA et la Récupération de Mémoire

L'implémentation naïve de notre pile lock-free met en lumière deux problèmes profonds et omniprésents dans la conception de structures de données dynamiques sans verrous. Ces problèmes, la récupération de mémoire et le problème ABA, sont souvent liés mais conceptuellement distincts.

### La Récupération de Mémoire (Memory Reclamation)

Le problème de la récupération de mémoire est une forme de "Use-After-Free" qui se manifeste dans un contexte concurrent. Reprenons notre opération `pop` :

1. Le thread T1 exécute `pop`. Il lit la tête `m_head` et obtient un pointeur vers le nœud A. Il lit ensuite `A->next` et obtient un pointeur vers le nœud B.
2. À ce moment précis, T1 est préempté par le système d'exploitation.
3. Le thread T2 exécute `pop`. Il réussit son CAS, retire le nœud A de la pile et exécute `delete A`. La mémoire occupée par A est maintenant libérée et peut être réutilisée pour n'importe quoi d'autre.
4. Le thread T1 se réveille. Il est au milieu de sa boucle `compare_exchange_weak`. Il tente de comparer `m_head` avec sa copie locale `old_head` (qui pointe vers l'adresse de l'ancien nœud A). Pire encore, s'il réussissait son CAS, il lirait `old_head->data` ou `old_head->next`, accédant ainsi à de la mémoire qui a été libérée. C'est un comportement indéfini, menant très probablement à un crash ou à une corruption silencieuse des données.

La "solution" la plus simple et la plus brutale à ce problème est de ne jamais libérer la mémoire. On retire les nœuds de la structure, mais on ne les `delete` jamais, provoquant une fuite mémoire. Pour des applications à courte durée de vie ou avec un nombre borné d'opérations, cela peut être une stratégie viable, mais ce n'est évidemment pas une solution générale. Une gestion correcte et sûre de la mémoire est indispensable.

### Le Problème ABA

Le problème ABA est plus subtil, car il peut se produire même en l'absence de récupération de mémoire (par exemple, dans une structure utilisant un buffer pré-alloué). Il tire son nom du scénario suivant : une valeur passe de A à B, puis revient à A.

Reprenons notre pile, en supposant pour l'instant que nous avons une solution magique au `delete`.

1. Le thread T1 commence son `pop`. Il lit `m_head` et stocke localement le pointeur vers le nœud A (`old_head = A`). Il lit ensuite `A->next` qui pointe vers B.
2. T1 est préempté.
3. Le thread T2 intervient. Il exécute un `pop`, retirant A. La tête devient B.
4. T2 exécute un autre `pop`, retirant B. La tête devient C.
5. T2 exécute un `push` d'un nouvel élément. Par pure coïncidence, l'allocateur mémoire lui fournit un nouveau nœud à la même adresse que celle de l'ancien nœud A. Appelons ce nouveau nœud A'. La tête de la pile est maintenant A'.
6. T1 se réveille. Il exécute enfin son `compare_exchange_weak(old_head, old_head->next)`. Il compare la valeur actuelle de `m_head` (qui est A') avec sa valeur locale `old_head` (qui est A). Comme les adresses sont identiques, le CAS réussit!
7. T1 met alors à jour `m_head` pour qu'il pointe vers `old_head->next`, c'est-à-dire B.

Le résultat est une catastrophe. La tête de la pile pointe maintenant vers B, un nœud qui a déjà été retiré de la pile. Le nœud A' (et tout ce qui se trouvait en dessous) est maintenant "perdu", déconnecté de la pile. La structure de données est corrompue.

Le problème fondamental est que le CAS compare les valeurs (ici, des adresses mémoire), mais il est aveugle à l'"identité" ou à l'"histoire" de ces valeurs. Il ne peut pas distinguer l'ancien nœud A du nouveau nœud A'.

Il est crucial de comprendre que ces deux problèmes, bien qu'intrinsèquement liés, sont distincts. Une stratégie de récupération de mémoire qui empêche la réutilisation immédiate d'une adresse mémoire (comme nous le verrons avec les `shared_ptr` ou les Hazard Pointers) résout souvent le problème ABA comme un effet de bord bénéfique. Cependant, le problème ABA peut exister indépendamment, par exemple avec des indices dans un tableau qui pourraient revenir à une valeur précédente. La solution à ABA doit donc être pensée de manière plus fondamentale que la simple gestion du `delete`.

## Partie IV : Stratégies de Solution et Abstractions Modernes

Face aux défis redoutables de la récupération de mémoire et du problème ABA, la communauté C++ et les chercheurs en concurrence ont développé un arsenal de solutions. Chacune de ces stratégies représente un compromis différent entre la simplicité d'utilisation, la performance, la surcharge mémoire et les garanties offertes.

### Solution 1 : std::atomic<std::shared_ptr> (C++20)

Une approche séduisante, introduite en C++20, consiste à utiliser une spécialisation atomique de `std::shared_ptr` pour gérer les pointeurs partagés, comme la tête de notre pile.

Le concept est élégant : le `std::shared_ptr` encapsule la gestion de la durée de vie de l'objet via son compteur de références atomique.

- **Résolution de la récupération de mémoire** : Un nœud n'est automatiquement détruit que lorsque le dernier `shared_ptr` qui pointe vers lui est détruit. Cela élimine le `delete` manuel et prévient les "use-after-free" concurrents.
- **Résolution du problème ABA** : Tant qu'un thread détient un `shared_ptr` vers un nœud (par exemple, `old_head` dans notre `pop`), le compteur de références de ce nœud est supérieur à zéro. La mémoire de ce nœud ne peut donc pas être libérée, et par conséquent, ne peut pas être réallouée pour un nouveau nœud. Le scénario ABA est ainsi évité.

Cependant, cette solution cache un piège majeur. L'échec de `std::atomic<shared_ptr>` à être lock-free dans les bibliothèques standards est un symptôme d'un défi plus large : la difficulté de standardiser des primitives de bas niveau qui dépendent fortement de l'architecture matérielle. Pour être véritablement lock-free, une opération sur `atomic<shared_ptr>` nécessiterait une instruction atomique capable de manipuler une structure de 128 bits (le pointeur vers l'objet et le pointeur vers le bloc de contrôle) ou d'employer des techniques non portables, comme le stockage d'un compteur dans les bits non utilisés d'une adresse 64 bits. Face à l'impossibilité de garantir une implémentation lock-free sur toutes les plateformes cibles, le comité de standardisation C++ a privilégié la correction et la portabilité en autorisant une implémentation basée sur un verrou interne. Les implémentations actuelles de libstdc++ (GCC) et de la STL de MSVC utilisent un verrou (un spinlock ou un mécanisme de type wait/notify) pour sérialiser les opérations. Par conséquent, un appel à `is_lock_free()` sur une instance de `std::atomic<std::shared_ptr>` retournera `false`. Bien que correcte et plus simple à utiliser, cette solution ne respecte pas la garantie lock-free.

### Solution 2 : Les Pointeurs Hazarés (Hazard Pointers)

Les pointeurs hazarés sont une technique de récupération de mémoire manuelle, plus complexe mais offrant de meilleures performances et des garanties lock-free. Le concept repose sur un "contrat" entre les threads lecteurs et les threads "récupérateurs" :

1. **Déclaration de danger** : Avant de déréférencer un pointeur partagé, un thread doit le déclarer comme "à risque" en le plaçant dans une liste de pointeurs hazarés qui lui est propre, mais qui est lisible par tous les autres threads.
2. **Retrait** : Lorsqu'un thread souhaite supprimer un nœud, il le retire de la structure de données principale mais ne le libère pas immédiatement. Il le place sur une liste de nœuds "retirés".
3. **Vérification et récupération** : Périodiquement, un thread parcourt sa liste de nœuds retirés. Pour chaque nœud, il scanne les listes de pointeurs hazarés de tous les autres threads. Si le pointeur du nœud n'apparaît dans aucune liste de danger, il est sûr de le libérer. Sinon, la libération est différée.

Cette technique est en cours de standardisation pour C++26 (proposition P2530). L'arrivée des Hazard Pointers dans la bibliothèque standard signale un changement de philosophie du comité C++. Il reconnaît que pour la programmation concurrente de haute performance, les abstractions de haut niveau comme `atomic<shared_ptr>` ne suffisent pas toujours. Il est nécessaire de fournir des outils plus bas niveau, plus "manuels", qui, bien que plus complexes à utiliser, offrent les garanties de performance et de non-blocage requises par les applications les plus exigeantes. L'API proposée s'articulera autour des composants suivants :

- `std::hazard_pointer_obj_base` : Une classe de base dont les objets gérés devront hériter.
- `void retire()` : Une méthode sur l'objet de base pour le marquer comme prêt à être récupéré.
- `std::hazard_pointer` : Un objet, typiquement local au thread, qui "protège" un pointeur.
- `hp.protect(atomic_ptr)` : La fonction clé qui charge un pointeur atomique et le place sous la protection du `hazard_pointer hp`.

### Tableau Comparatif : Schémas de Récupération de Mémoire

La prolifération des techniques de récupération de mémoire peut être déroutante. Le tableau suivant synthétise les compromis des approches les plus courantes pour aider à choisir la stratégie la plus adaptée à un besoin spécifique.

| Schéma | Principe de base | Avantages | Inconvénients | Cas d'usage idéal |
|--------|------------------|-----------|---------------|-------------------|
| **Reference Counting** (`atomic<shared_ptr>`) | Un compteur de références est associé à chaque objet. | Très simple à utiliser, gestion automatique de la mémoire. | Surcharge de performance à chaque création/destruction de copie du pointeur. Non lock-free dans la std. | Structures de données simples où la facilité d'utilisation prime sur la performance ultime. |
| **Hazard Pointers (HP)** | Chaque thread déclare les pointeurs qu'il utilise dans une liste partagée. | Granularité fine (ne bloque la récupération que des objets réellement utilisés), robuste face aux threads lents. | Complexité de l'API, surcharge à chaque lecture de pointeur pour le protéger. | Structures de données complexes avec des durées d'accès aux nœuds variables. |
| **Epoch-Based Reclamation (EBR)** | L'exécution est divisée en "époques". Un objet retiré à l'époque N ne peut être libéré que lorsque tous les threads ont quitté l'époque N. | Très faible surcharge pour les opérations de lecture (souvent un simple load). Très performant. | Un thread bloqué ou lent dans une ancienne époque peut empêcher toute récupération de mémoire, provoquant des fuites. | Systèmes temps-réel ou à haute fréquence où les opérations sont courtes et les threads coopératifs. |
| **Read-Copy-Update (RCU)** | Les mises à jour se font sur une copie de la structure. L'ancienne version est libérée après une "période de grâce". | Overhead quasi nul pour les lecteurs, qui n'ont besoin d'aucune synchronisation atomique. | Surcharge potentiellement très élevée pour les écrivains (copie de données). Complexe à implémenter. | Structures de données où les lectures sont extrêmement fréquentes et les écritures très rares (ex: tables de routage, configurations). |

### Implémentation Proposée : Une Pile Lock-Free Robuste

Voici une implémentation correcte de notre pile utilisant `std::atomic<std::shared_ptr>`. Elle est fonctionnelle et sûre, mais il faut garder à l'esprit qu'elle n'est pas lock-free avec les bibliothèques standards actuelles.

```cpp
#include <atomic>
#include <memory>

template<typename T>
class SafeStack {
private:
    struct Node {
        T data;
        std::shared_ptr<Node> next;
    };
    std::atomic<std::shared_ptr<Node>> m_head;

public:
    void push(const T& value) {
        auto new_node = std::make_shared<Node>();
        new_node->data = value;
        new_node->next = std::atomic_load(&m_head);
        while (!std::atomic_compare_exchange_weak(&m_head, &new_node->next, new_node));
    }

    std::shared_ptr<T> pop() {
        std::shared_ptr<Node> old_head = std::atomic_load(&m_head);
        while (old_head && !std::atomic_compare_exchange_weak(&m_head, &old_head, old_head->next));
        
        if (old_head) {
            return std::make_shared<T>(old_head->data);
        }
        return nullptr;
    }
};
```

Pour une véritable implémentation lock-free avec des pointeurs hazarés, la logique de `pop` serait conceptuellement la suivante :

```cpp
// Pseudo-code pour un pop avec Hazard Pointers
// T pop() {
//     hazard_pointer hp = make_hazard_pointer();
//     Node* old_head;
//     do {
//         old_head = hp.protect(m_head); // Protège la tête
//         if (old_head == nullptr) { /* Gérer pile vide */ }
//     } while (!m_head.compare_exchange_weak(old_head, old_head->next));
//
//     T data = old_head->data;
//     old_head->retire(); // Marque le nœud pour récupération future
//     hp.reset_protection(); // Libère la protection
//     return data;
// }
```

## Partie V : Le Rôle du Matériel et du Modèle Mémoire C++

Pour maîtriser la programmation lock-free, il est indispensable de descendre d'un niveau d'abstraction et de comprendre comment le matériel et le compilateur interagissent avec notre code. Les garanties que nous cherchons à obtenir ne dépendent pas seulement de nos algorithmes, mais aussi des garanties offertes par l'architecture du processeur et de la manière dont le langage C++ nous permet de les contrôler.

### Solutions Matérielles : Load-Linked/Store-Conditional (LL/SC)

Alors que les architectures x86 fournissent une instruction CAS matérielle (`lock cmpxchg`), de nombreuses autres architectures RISC (ARM, RISC-V, PowerPC) proposent une paire d'instructions alternative : Load-Linked/Store-Conditional (LL/SC).

Le mécanisme fonctionne en deux temps :

1. **Load-Linked (LL)** (ou `ldrex` sur ARM, `lr` sur RISC-V) charge une valeur depuis une adresse mémoire et "réserve" cette adresse au niveau du cache du processeur.
2. **Store-Conditional (SC)** (ou `strex`, `sc`) tente d'écrire une nouvelle valeur à la même adresse. L'écriture ne réussit que si la réservation est toujours valide, c'est-à-dire si aucune autre écriture n'a eu lieu sur cette ligne de cache depuis le LL.

La différence fondamentale avec le CAS est que le SC ne vérifie pas si la valeur est la même, mais si la location mémoire a été modifiée. Par conséquent, si un autre thread modifie la valeur de A à B puis la restaure à A, le SC échouera quand même. Le LL/SC est donc une solution matérielle native au problème ABA.

### Le Modèle Mémoire C++

Le compilateur et le processeur ont tous deux une grande latitude pour réordonner les instructions de lecture et d'écriture mémoire afin d'optimiser les performances, tant que le comportement observable d'un programme mono-thread n'est pas affecté. En programmation concurrente, ce réordonnancement est une source majeure de bugs subtils.

Un exemple frappant est le phénomène des "Chasing Counters". Considérons deux variables atomiques `a` et `b`. Un thread incrémente `a` puis `b`. Un autre thread lit `b` puis `a`. Sur une architecture à modèle mémoire fort comme x86, on ne verra jamais `a < b`. En revanche, sur une architecture à modèle mémoire faible comme ARM, le processeur peut réordonner les écritures du premier thread ou les lectures du second, menant à la situation contre-intuitive où la lecture de `a` retourne une valeur inférieure à celle de `b`.

La performance du code lock-free est donc directement liée à la "faiblesse" du modèle mémoire de l'architecture sous-jacente. Sur x86, le modèle mémoire (TSO - Total Store Order) est très strict, et de nombreuses opérations atomiques C++ ne nécessitent pas de barrières mémoire supplémentaires coûteuses. Sur ARM ou PowerPC, les modèles sont beaucoup plus relâchés, et le même code C++ générera des instructions de barrière explicites (comme `dmb`) qui ont un coût non négligeable. Un algorithme lock-free portable peut donc avoir des profils de performance radicalement différents d'une machine à l'autre.

Pour contrôler ce comportement, le C++ nous fournit l'énumération `std::memory_order` :

- **`memory_order_relaxed`** : Aucune garantie d'ordre. Seule l'atomicité de l'opération est assurée.
- **`memory_order_release`** (pour une écriture) : Agit comme une barrière. Aucune lecture ou écriture précédente dans le code source ne peut être réordonnée après cette écriture atomique.
- **`memory_order_acquire`** (pour une lecture) : Agit comme une barrière. Aucune lecture ou écriture suivante dans le code source ne peut être réordonnée avant cette lecture atomique.
- **`memory_order_acq_rel`** : Combine les deux pour les opérations de lecture-modification-écriture (comme `fetch_add` ou `compare_exchange`).
- **`memory_order_seq_cst`** : L'ordre le plus strict et celui par défaut. Il fournit les garanties `acq_rel` et, en plus, assure qu'il existe un ordre global unique de toutes les opérations `seq_cst` visible par tous les threads. C'est le plus simple à raisonner, mais potentiellement le plus coûteux.

### Les Relations Fondamentales

Le modèle mémoire C++ est défini par des relations formelles qui décrivent comment les effets des opérations deviennent visibles entre les threads :

- **Sequenced-before** : C'est la relation d'ordre que nous connaissons tous : l'ordre des instructions au sein d'un même thread.
- **Synchronizes-with** : C'est le pont entre les threads. Une écriture atomique avec `memory_order_release` sur une variable V synchronise avec une lecture atomique avec `memory_order_acquire` sur la même variable V qui lit la valeur écrite (ou une valeur ultérieure).
- **Happens-before** : C'est la relation de causalité. Une opération A happens-before une opération B si A est sequenced-before B, ou si A synchronizes-with B, ou par transitivité à travers une chaîne de ces relations. Si A happens-before B, alors les effets mémoire de A sont garantis d'être visibles pour B.

La maîtrise de ces ordres mémoire est essentielle. Utiliser `memory_order_relaxed` partout où c'est possible peut apporter des gains de performance significatifs, mais au prix d'une complexité de raisonnement accrue. Une seule erreur dans le choix de l'ordre mémoire peut introduire des bugs de concurrence extrêmement difficiles à diagnostiquer.

## Partie VI : Vers une Confiance Absolue : La Vérification Formelle

Nous avons vu que les algorithmes concurrents sont fragiles et que les tests traditionnels sont largement insuffisants pour garantir leur correction. L'espace des entrelacements possibles des opérations de plusieurs threads est si vaste qu'il est impossible de le couvrir de manière exhaustive par l'exécution. Les "data races" et autres bugs de synchronisation sont souvent non-déterministes, se manifestant rarement et de manière imprévisible, ce qui les rend presque impossibles à reproduire et à déboguer.

Face à cette limitation, une approche plus rigoureuse est nécessaire : la vérification formelle. Plutôt que de tester quelques exécutions, les méthodes formelles visent à prouver mathématiquement que l'algorithme est correct pour toutes les exécutions possibles.

### Introduction à TLA+

Un des outils les plus reconnus dans ce domaine est TLA+ (Temporal Logic of Actions), un langage de spécification formelle conçu par Leslie Lamport. Il est crucial de comprendre que TLA+ ne vérifie pas le code C++ directement. Il vérifie le modèle ou l'algorithme abstrait qui sous-tend le code.

Le processus de vérification avec TLA+ se déroule typiquement en plusieurs étapes :

1. **Modélisation** : L'ingénieur décrit l'algorithme (par exemple, notre pile lock-free) dans le langage de haut niveau de TLA+ (ou son dialecte impératif, PlusCal). Cette étape implique de définir les variables d'état du système (ex: `head`), l'état initial, et les actions atomiques qui peuvent modifier cet état (ex: `Push`, `Pop`).

2. **Spécification des Propriétés** : On définit ensuite les propriétés que le système doit toujours respecter. Celles-ci se divisent en deux catégories :
   - **Propriétés de Sûreté (Safety)** : "Quelque chose de mal n'arrive jamais". Par exemple, un invariant comme "la pile ne contient jamais de cycle" ou "le pointeur de tête n'est jamais invalide".
   - **Propriétés de Vivacité (Liveness)** : "Quelque chose de bien finit toujours par arriver". Par exemple, "toute opération push lancée finira par réussir".

3. **Model Checking** : On utilise ensuite un outil appelé un "model checker", comme TLC qui est fourni avec TLA+, pour explorer systématiquement tous les états atteignables du modèle, à partir de l'état initial. Pour chaque état, il vérifie si les invariants de sûreté sont respectés.

4. **Analyse des Résultats** : Si le model checker trouve un état qui viole une propriété, il ne se contente pas de le signaler. Il fournit un contre-exemple : la séquence exacte d'actions, entrelacées entre les différents threads, qui mène de l'état initial à l'état fautif. Ce contre-exemple est un outil de débogage d'une puissance inestimable.

L'exemple mentionné dans la transcription, où un collègue a formellement vérifié la classe DataHolder, est une illustration parfaite de la puissance de cette approche. Malgré la simplicité apparente du code C++, la spécification TLA+ pour le modéliser peut être complexe, mais elle offre en retour une confiance quasi-absolue dans la correction de l'algorithme.

Le processus de spécification formelle a une valeur qui va au-delà de la simple vérification. Pour écrire un modèle TLA+, l'ingénieur est obligé de clarifier sa pensée, de définir sans ambiguïté les états, les transitions et les hypothèses. Souvent, des failles de conception sont découvertes durant cette phase de modélisation, simplement parce que l'on réalise que certaines hypothèses étaient vagues, incomplètes ou contradictoires. La spécification formelle est donc avant tout un outil de conception rigoureux.

## Partie VII : Aux Frontières de la Recherche (Travaux Récents, post-2022)

Le domaine de la programmation lock-free est loin d'être statique. Il est en constante ébullition, avec des chercheurs qui repoussent continuellement les limites de la performance et de l'expressivité. Un aperçu des travaux récents (publiés après 2022) révèle des tendances fascinantes et donne un avant-goût des outils dont nous pourrions disposer demain.

Ces axes de recherche mettent en lumière une tendance de fond : l'abandon de l'idée d'une solution universelle au profit de la spécialisation et de l'optimisation contextuelle. L'avenir de la programmation concurrente ne réside pas dans une unique primitive magique, mais dans une boîte à outils de solutions hautement spécialisées. Pour atteindre les plus hauts niveaux de performance, le développeur devra comprendre en profondeur les caractéristiques de son application (ratio lecture/écriture, contention, distribution) pour choisir l'outil le plus adapté.

### "Publish on Ping" : Une Récupération de Mémoire Optimisée

Les schémas comme les Hazard Pointers, bien qu'efficaces, souffrent d'un défaut : ils imposent une surcharge, notamment une barrière mémoire, à chaque accès à un pointeur protégé, même si aucun autre thread n'est en train de tenter de récupérer de la mémoire à ce moment-là. C'est une approche pessimiste qui paie un coût constant pour se prémunir contre un événement rare.

Dans un article de 2025, Singh et Brown proposent une technique novatrice appelée "Publish on Ping". L'idée est de passer à un modèle optimiste et "lazy" :

1. Les threads lecteurs continuent de suivre localement les pointeurs qu'ils utilisent, mais sans les publier dans une structure de données globale. L'opération de lecture devient ainsi beaucoup plus légère.
2. Ce n'est que lorsqu'un thread "récupérateur" souhaite libérer de la mémoire qu'il envoie un "ping" (par exemple, via un signal POSIX) à tous les autres threads.
3. En recevant ce ping, les threads lecteurs publient alors leurs listes de pointeurs hazarés locaux dans la structure globale.
4. Le récupérateur attend que tous les threads aient répondu, puis procède à la récupération en toute sécurité.

Cette approche "à la demande" réduit considérablement la surcharge pour les charges de travail dominées par les lectures, où la récupération de mémoire est un événement peu fréquent, offrant des gains de performance pouvant aller jusqu'à un facteur de 4 par rapport aux Hazard Pointers classiques.

### "Big Atomics" : Des Opérations Atomiques Multi-Mots

Un défi récurrent en programmation lock-free est la nécessité de modifier atomiquement plusieurs champs d'une structure qui ne tiennent pas dans les 64 ou 128 bits supportés nativement par les instructions CAS matérielles. Comment, par exemple, mettre à jour atomiquement une clé, une valeur et un pointeur dans un nœud de table de hachage?

Les travaux d'Anderson, Blelloch et Jayanti (2025) sur les "Big Atomics" s'attaquent à ce problème en proposant des algorithmes lock-free efficaces pour simuler en logiciel des opérations atomiques (load, store, CAS) sur des blocs de mémoire de taille arbitraire. Leur approche repose sur un design "fast-path/slow-path" :

- Le "fast-path" est une séquence d'opérations optimiste et très rapide qui réussit en l'absence de contention.
- Le "slow-path", plus complexe, est activé uniquement lorsqu'une contention est détectée, pour résoudre le conflit de manière correcte.

L'impact de cette recherche est potentiellement immense. Elle permet de simplifier radicalement la conception de structures de données concurrentes complexes. Au lieu de concevoir des algorithmes alambiqués pour gérer les mises à jour de plusieurs champs, le développeur peut raisonner en termes d'une seule opération atomique sur sa structure, rendant le code plus simple, plus lisible et moins sujet aux erreurs.

### "DiLi" : Vers des Structures Lock-Free Distribuées

Traditionnellement, les garanties lock-free sont définies dans le contexte d'un système à mémoire partagée sur une seule machine. Comment étendre ces concepts à un système distribué, où la communication se fait par le réseau, avec des latences et des défaillances potentielles?

Le projet DiLi (Distributable Linked List) de Ravishankar et al. (2025) explore cette question en introduisant la notion de "conditional lock-freedom". Un algorithme est conditionnellement lock-free s'il respecte la garantie lock-free sous l'hypothèse raisonnable que la communication réseau prend un temps fini et est fiable.

DiLi est une implémentation d'une liste chaînée qui peut être dynamiquement partitionnée et dont les sous-listes peuvent être distribuées sur plusieurs machines. Cela permet de faire du load-balancing et de faire évoluer la structure au-delà des limites d'une seule machine, tout en conservant les propriétés de non-blocage. Cette recherche ouvre la voie à la conception de bases de données et de systèmes de stockage distribués qui héritent des avantages de performance et de robustesse des algorithmes lock-free.

## Conclusion : Synthèse et Perspectives

Au terme de ce parcours, il est clair que la programmation lock-free est un outil d'une puissance considérable, mais qui exige une discipline et une rigueur extrêmes. Nous avons vu que le chemin vers des algorithmes concurrents performants et corrects est semé d'embûches : des garanties théoriques précises à respecter, des pièges subtils comme le problème ABA, des défis complexes comme la récupération de mémoire, et une interaction intime avec les détails du modèle mémoire et de l'architecture matérielle.

La programmation lock-free n'est pas une solution miracle. Elle ne doit être envisagée que lorsque des mesures de performance ont clairement identifié la contention sur les verrous comme un goulot d'étranglement majeur et insurmontable. Dans la grande majorité des applications, un `std::mutex` ou un `std::shared_mutex` bien utilisé reste la solution la plus simple, la plus sûre et la plus maintenable. Le premier principe de l'optimisation reste de ne pas optimiser prématurément.

Cependant, pour les domaines où chaque nanoseconde compte – la finance à haute fréquence, les moteurs de jeu, les systèmes d'exploitation, les piles réseau – la maîtrise des techniques lock-free est un différenciateur essentiel. L'écosystème C++ évolue pour mieux supporter ces besoins. L'arrivée prochaine des Hazard Pointers dans C++26 est une étape majeure qui fournira un outil standardisé et robuste pour la récupération de mémoire lock-free. Parallèlement, la recherche continue de repousser les frontières, en proposant des schémas de récupération plus efficaces, des primitives de synchronisation plus puissantes et en étendant même le paradigme aux systèmes distribués.

La programmation concurrente de bas niveau restera un domaine d'expertise de pointe. Elle demande un investissement intellectuel conséquent, mais elle offre en retour la capacité de construire des systèmes logiciels qui exploitent pleinement la puissance des architectures matérielles modernes. Je vous remercie de votre attention.
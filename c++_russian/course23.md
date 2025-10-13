# Parallélisme en C++ : Des Politiques d'Exécution à la Programmation Hétérogène et au-delà

## Partie I : Le Parallélisme sur CPU en C++ Moderne

Cette première partie établit les fondations du parallélisme en C++ en se concentrant sur les architectures CPU. Nous partons des abstractions de haut niveau introduites en C++17 pour descendre progressivement vers un contrôle manuel des threads, illustrant à chaque étape les compromis entre simplicité, performance et contrôle.

### Chapitre 1 : Fondamentaux du Parallélisme et Politiques d'Exécution (C++17)

#### Introduction : Concurrence vs. Parallélisme

Avant d'explorer les mécanismes de parallélisation en C++, il est impératif de distinguer rigoureusement deux concepts souvent confondus : la **concurrence** et le **parallélisme**. 

- **La concurrence** se réfère à la structuration d'un programme où plusieurs tâches peuvent être en cours d'exécution sur une même période, progressant de manière indépendante. Ces tâches peuvent être entrelacées sur un seul cœur de processeur ou s'exécuter sur des cœurs différents. La concurrence est avant tout un concept de gestion de la complexité et de la séparation des préoccupations. 

- **Le parallélisme**, en revanche, est un phénomène d'exécution : il s'agit de l'exécution simultanée de plusieurs tâches, ce qui requiert intrinsèquement du matériel doté de plusieurs unités de calcul (par exemple, un processeur multi-cœur).

Ce cours, comme l'indique son orientation, se focalise sur le parallélisme : l'art d'exploiter le matériel pour accélérer l'exécution d'une tâche unique en la décomposant en sous-tâches qui s'exécutent en parallèle.

#### Les Politiques d'Exécution de la STL (std::execution)

Avec la norme C++17, le comité de standardisation a introduit une abstraction de haut niveau pour invoquer la parallélisation des algorithmes de la bibliothèque standard (STL) : les **politiques d'exécution**, définies dans l'en-tête `<execution>`. Ces politiques sont des objets passés comme premier argument à de nombreuses surcharges d'algorithmes de la STL (tels que `std::for_each`, `std::transform`, `std::sort`, `std::reduce`, etc.) pour spécifier comment l'algorithme peut être exécuté.

Il existe trois politiques standards (et une quatrième ajoutée en C++20, `unseq`, qui n'est pas parallèle) :

1. **`std::execution::sequenced_policy`** (objet `std::execution::seq`) : 
   - Impose une exécution purement séquentielle
   - Garantit que les opérations sont exécutées dans l'ordre spécifié par l'algorithme, sur le thread appelant
   - C'est le comportement par défaut des algorithmes sans politique

2. **`std::execution::parallel_policy`** (objet `std::execution::par`) : 
   - Autorise l'exécution en parallèle
   - L'implémentation est libre de répartir les opérations sur plusieurs threads
   - Aucune garantie sur l'ordre relatif d'exécution des opérations entre différents threads
   - Les opérations exécutées au sein d'un même thread sont indéterminément séquencées les unes par rapport aux autres

3. **`std::execution::parallel_unsequenced_policy`** (objet `std::execution::par_unseq`) : 
   - La politique la plus permissive
   - Autorise l'exécution en parallèle sur plusieurs threads ET l'entrelacement des instructions au sein d'un même thread
   - Permet au compilateur d'appliquer des optimisations de vectorisation (SIMD - Single Instruction, Multiple Data)
   - Impose des contraintes strictes : les opérations ne doivent pas avoir de dépendances entre elles et ne doivent pas utiliser de synchronisation

#### Analyse de Performance et le "Piège de l'Abstraction"

L'introduction des politiques d'exécution a été une avancée majeure pour la productivité, permettant de paralléliser une boucle en changeant une seule ligne de code. Cependant, cette abstraction de haut niveau n'est pas une garantie de performance. Les benchmarks pour `std::reduce` et `std::transform` montrent un phénomène contre-intuitif : l'utilisation de `std::execution::par` ou `std::execution::par_unseq` peut non seulement ne pas améliorer les performances, mais parfois même les dégrader par rapport à la version séquentielle.

Des études plus systématiques confirment cette observation. Des suites de benchmarks comme pSTL-Bench ont été développées spécifiquement pour évaluer la scalabilité et l'efficacité des différentes implémentations de la STL parallèle. Les résultats montrent que pour des tailles de données insuffisantes ou des opérations par élément trop rapides, le surcoût lié à la gestion du parallélisme (création et gestion d'un pool de threads, partitionnement des données, synchronisation des résultats) peut largement dépasser les gains obtenus par l'exécution simultanée.

Ce phénomène illustre un principe fondamental en programmation haute performance : **les abstractions ne sont pas gratuites**. Le mécanisme sous-jacent à `std::execution::par` est typiquement un pool de threads géré par l'implémentation de la bibliothèque standard. Le coût de la planification des tâches, de la synchronisation des threads et du partitionnement des données constitue un surcoût fixe. 

### Chapitre 2 : Parallélisme Explicite et Gestion Fine des Threads

Lorsque les abstractions de haut niveau comme les politiques d'exécution ne suffisent pas à atteindre les performances requises, le développeur doit descendre sur "l'échelle d'abstraction" pour prendre un contrôle plus direct sur les mécanismes de parallélisation.

#### Implémentation d'un parallel_reduce Personnalisé

L'objectif est de calculer la somme des éléments d'un conteneur en parallèle. La stratégie est la suivante :

1. Diviser le conteneur en plusieurs sous-plages
2. Lancer un thread distinct pour calculer la somme de chaque sous-plage
3. Collecter les sommes partielles de chaque thread
4. Calculer la somme finale de ces résultats partiels

Une première question cruciale est de déterminer le nombre optimal de threads. Une heuristique courante consiste à utiliser `std::thread::hardware_concurrency()`. Cette fonction retourne une estimation du nombre de threads matériels disponibles.

#### Étape 1 : Contrôle Direct avec std::thread

Le niveau de contrôle le plus bas et le plus explicite est l'utilisation directe de la classe `std::thread` de l'en-tête `<thread>`. Cette approche offre une correspondance quasi directe avec les threads du système d'exploitation.

L'implémentation implique plusieurs défis techniques :

- **Partitionnement des données** : Il faut calculer la taille de chaque bloc de données à traiter par thread
- **Lancement des threads** : Un `std::vector<std::thread>` est utilisé pour stocker les objets threads
- **Communication des résultats** : Les résultats partiels doivent être stockés dans un conteneur accessible par tous les threads
- **Synchronisation** : Le thread principal doit attendre la fin de tous les threads de travail avant de pouvoir agréger les résultats

Voici une implémentation possible :

```cpp
template<typename Iterator, typename T>
T parallel_reduce_thread(Iterator first, Iterator last, T init) {
    auto const length = std::distance(first, last);
    if (length == 0) {
        return init;
    }

    unsigned int const hardware_threads = std::thread::hardware_concurrency();
    unsigned int const num_threads = std::max(1u, hardware_threads);
    
    std::vector<T> partial_sums(num_threads);
    std::vector<std::thread> threads;

    auto const block_size = (length + num_threads - 1) / num_threads;

    for (unsigned int i = 0; i < num_threads; ++i) {
        auto block_start = first + i * block_size;
        auto block_end = std::min(first + (i + 1) * block_size, last);
        
        if (block_start >= block_end) continue;

        threads.emplace_back([&, block_start, block_end, i] {
            partial_sums[i] = std::accumulate(block_start, block_end, T{});
        });
    }

    for (auto& t : threads) {
        t.join();
    }

    return std::accumulate(partial_sums.begin(), partial_sums.end(), init);
}
```

#### Étape 2 : Abstraction de la Tâche avec std::packaged_task et std::future

Pour améliorer la modularité, nous pouvons séparer la définition de la tâche de son exécution. C'est le rôle de `std::packaged_task` (en-tête `<future>`). Cet objet enveloppe une fonction ou une lambda et la lie à une `std::future`, qui servira de canal de communication pour récupérer le résultat de manière asynchrone.

```cpp
template<typename Iterator, typename T>
T parallel_reduce_packaged_task(Iterator first, Iterator last, T init) {
    //... (calcul de num_threads, block_size, etc. comme précédemment)...

    std::vector<std::future<T>> futures;
    std::vector<std::thread> threads;

    for (unsigned int i = 0; i < num_threads; ++i) {
        auto block_start = first + i * block_size;
        auto block_end = std::min(first + (i + 1) * block_size, last);

        if (block_start >= block_end) continue;

        std::packaged_task<T(Iterator, Iterator)> task(
            [](Iterator start, Iterator end) {
                return std::accumulate(start, end, T{});
            });
        
        futures.push_back(task.get_future());
        threads.emplace_back(std::move(task), block_start, block_end);
    }

    for (auto& t : threads) {
        t.join();
    }

    T final_result = init;
    for (auto& f : futures) {
        final_result += f.get();
    }
    return final_result;
}
```

#### Étape 3 : Le Plus Haut Niveau d'Abstraction avec std::async

`std::async` représente le sommet de cette échelle d'abstraction pour le parallélisme explicite. Cette fonction prend une fonction et ses arguments, et se charge de l'exécuter (potentiellement) sur un nouveau thread, en retournant une `std::future` qui contiendra le résultat.

```cpp
template<typename Iterator, typename T>
T parallel_reduce_async(Iterator first, Iterator last, T init) {
    //... (calcul de num_threads, block_size, etc.)...

    std::vector<std::future<T>> futures;

    for (unsigned int i = 0; i < num_threads; ++i) {
        auto block_start = first + i * block_size;
        auto block_end = std::min(first + (i + 1) * block_size, last);

        if (block_start >= block_end) continue;

        futures.push_back(std::async(std::launch::async, 
            [](Iterator start, Iterator end) {
                return std::accumulate(start, end, T{});
            }, block_start, block_end));
    }

    T final_result = init;
    for (auto& f : futures) {
        final_result += f.get(); // get() est bloquant et attend le résultat
    }
    return final_result;
}
```

Malgré son apparente simplicité, `std::async` cache une complexité et des pièges qui le rendent inadapté à un calcul parallèle haute performance et prédictible :

- **Politiques de lancement** : Le comportement par défaut est `std::launch::async | std::launch::deferred`, ce qui introduit de la non-déterministe
- **Le destructeur bloquant** : Si la `std::future` retournée n'est pas stockée, son destructeur se bloquera jusqu'à ce que la tâche asynchrone soit terminée

## Partie II : Le Calcul Hétérogène avec SYCL et la Norme 2020

Cette partie plonge dans le monde du calcul sur processeurs graphiques (GPGPU). Nous explorons SYCL 2020, une évolution majeure qui modernise radicalement la programmation hétérogène en C++.

### Chapitre 3 : Introduction au GPGPU et au Modèle SYCL

#### Le Paradigme Hôte-Dispositif (Host-Device)

Le calcul hétérogène repose sur la collaboration de plusieurs types de processeurs au sein d'un même système. Le modèle le plus courant est celui de l'hôte-dispositif :

- **L'Hôte (Host)** : Généralement le CPU, il agit comme l'orchestrateur. Il exécute la logique principale du programme, gère les ressources et délègue les portions de calcul intensives et massivement parallèles au dispositif.

- **Le Dispositif (Device)** : Un accélérateur spécialisé, le plus souvent un GPU, mais qui peut aussi être un FPGA ou un DSP. Il est conçu pour exécuter des milliers d'opérations simples simultanément.

Historiquement, la programmation de ces dispositifs nécessitait des langages propriétaires (comme CUDA pour NVIDIA) ou des API de bas niveau (comme OpenCL). SYCL (prononcé "sickle") a été créé par le groupe Khronos pour unifier ce paysage. C'est un standard ouvert, multi-vendeur et multi-plateforme qui permet d'écrire du code pour des systèmes hétérogènes en utilisant du C++ standard, dans un style dit "single-source".

#### Concepts Fondamentaux de SYCL

La programmation en SYCL s'articule autour de quelques abstractions clés :

- **`sycl::queue`** : L'objet central pour l'interaction avec un dispositif. Une file d'attente est associée à un dispositif et à un contexte spécifiques.

- **`submit` et `handler`** : Pour soumettre du travail à une queue, on appelle sa méthode submit avec une lambda qui reçoit un objet handler.

- **`kernel`** : La fonction (généralement une lambda C++) qui sera exécutée sur le dispositif.

- **`parallel_for`** : La méthode du handler la plus couramment utilisée pour lancer un noyau.

Exemple canonique - addition de deux vecteurs :

```cpp
#include <sycl/sycl.hpp>
#include <vector>
#include <iostream>

int main() {
    const size_t N = 1024;
    std::vector<int> a(N, 1);
    std::vector<int> b(N, 2);
    std::vector<int> c(N, 0);

    sycl::queue q{sycl::gpu_selector{}};
    std::cout << "Running on device: " 
              << q.get_device().get_info<sycl::info::device::name>() << "\n";

    { // Début de la portée des buffers
        sycl::buffer buf_a(a);
        sycl::buffer buf_b(b);
        sycl::buffer buf_c(c);

        q.submit([&](sycl::handler& h) {
            sycl::accessor acc_a(buf_a, h, sycl::read_only);
            sycl::accessor acc_b(buf_b, h, sycl::read_only);
            sycl::accessor acc_c(buf_c, h, sycl::write_only);

            h.parallel_for(sycl::range(N), [=](sycl::id i) {
                acc_c[i] = acc_a[i] + acc_b[i];
            });
        });
    } // Fin de la portée des buffers, synchronisation et copie des résultats

    // Vérification
    for (size_t i = 0; i < N; ++i) {
        if (c[i] != 3) {
            std::cout << "Verification failed at index " << i << "\n";
            return 1;
        }
    }
    std::cout << "Verification passed.\n";
    return 0;
}
```

### Chapitre 4 : Modèles de Gestion de la Mémoire en SYCL

#### Le Modèle Historique : buffer et accessor

Le modèle originel de SYCL est basé sur les concepts de buffer et d'accessor.

- **`sycl::buffer`** : Un buffer encapsule des données existantes et les rend disponibles pour le runtime SYCL. Il ne possède pas la mémoire lui-même, mais agit comme un handle vers celle-ci.

- **`sycl::accessor`** : Pour accéder aux données d'un buffer à l'intérieur d'un groupe de commandes, on doit créer un accessor. Un accessor est une déclaration d'intention.

Le rôle de ce modèle est fondamental : en analysant les accessors déclarés dans chaque groupe de commandes, le runtime SYCL est capable de construire automatiquement un graphe de dépendances (DAG) entre les tâches.

#### La Révolution SYCL 2020 : Unified Shared Memory (USM)

La norme SYCL 2020 a introduit une alternative majeure : la Mémoire Partagée Unifiée (USM). L'USM offre un modèle de mémoire basé sur des pointeurs bruts, beaucoup plus familier pour les programmeurs C++.

SYCL 2020 définit trois types d'allocations USM :

1. **`sycl::malloc_device`** : Alloue de la mémoire directement sur le dispositif. Cette mémoire est inaccessible depuis l'hôte.

2. **`sycl::malloc_host`** : Alloue de la mémoire "épinglée" (pinned) sur l'hôte. Cette mémoire est accessible par le dispositif, typiquement à travers le bus PCIe.

3. **`sycl::malloc_shared`** : Alloue de la mémoire qui peut être accédée et modifiée à la fois par l'hôte et le dispositif.

Exemple d'addition de vecteurs utilisant `malloc_shared` :

```cpp
#include <sycl/sycl.hpp>
#include <iostream>

int main() {
    const size_t N = 1024;
    sycl::queue q{sycl::gpu_selector{}};

    // Allocation en mémoire partagée
    int* a = sycl::malloc_shared<int>(N, q);
    int* b = sycl::malloc_shared<int>(N, q);
    int* c = sycl::malloc_shared<int>(N, q);

    // Initialisation sur l'hôte
    for (size_t i = 0; i < N; ++i) {
        a[i] = 1;
        b[i] = 2;
    }

    q.parallel_for(sycl::range(N), [=](sycl::id i) {
        c[i] = a[i] + b[i];
    }).wait(); // Synchronisation explicite requise!

    // Vérification sur l'hôte
    //...

    sycl::free(a, q);
    sycl::free(b, q);
    sycl::free(c, q);
    return 0;
}
```

#### Tableau Comparatif des Modèles de Mémoire SYCL

| Caractéristique | buffer / accessor | USM device | USM host | USM shared |
|-----------------|-------------------|------------|----------|------------|
| **Modèle de programmation** | Objets RAII (style C++ moderne) | Pointeurs bruts | Pointeurs bruts | Pointeurs bruts |
| **Gestion des transferts** | Implicite (gérée par le runtime) | Explicite (queue::memcpy) | Implicite (accès via bus) | Implicite (migration à la demande) |
| **Synchronisation** | Implicite (via la portée des accessors) | Explicite (queue::wait() requis) | Explicite | Explicite |
| **Facilité de portage (depuis C/CUDA)** | Faible | Élevée | Élevée | Très élevée |
| **Contrôle des performances** | Moyen (dépend du runtime) | Maximal | Faible | Moyen (dépend de l'heuristique de migration) |
| **Cas d'usage typique** | Nouveau code SYCL, sécurité maximale | Algorithmes HPC, contrôle fin | Données volumineuses, accès peu fréquents | Prototypage rapide, code C++ existant |

### Chapitre 5 : Optimisation Avancée des Kernels SYCL

#### La Tâche "Royale" : Multiplication de Matrices Optimisée

Nous partons de l'implémentation naïve où chaque work-item calcule un élément de la matrice de résultat :

```cpp
// Version naïve
h.parallel_for(sycl::range(M, N), [=](sycl::id idx) {
    auto row = idx[0];
    auto col = idx[1];
    double sum = 0.0;
    for (size_t k = 0; k < K; ++k) {
        sum += acc_a[row][k] * acc_b[k][col];
    }
    acc_c[row][col] = sum;
});
```

#### Hiérarchie d'Exécution et de Mémoire

Pour aller plus loin, il faut exploiter la hiérarchie de la mémoire du GPU :

- **Work-groups et Mémoire Locale** : Les GPUs disposent d'une petite quantité de mémoire très rapide (local memory ou shared memory), partagée entre tous les work-items d'un même work-group. La stratégie du "tiling" consiste à charger des sous-matrices de la mémoire globale lente vers la mémoire locale rapide.

- **Le Patron "d'Inversion du Parallélisme"** : Au lieu d'avoir une boucle externe qui lance de multiples `parallel_for`, on "inverse" la logique pour avoir un seul `parallel_for` qui contient une boucle interne.

#### Exploitation des Fonctionnalités Avancées de SYCL 2020

**Réductions Parallèles Natives** : SYCL 2020 propose l'objet `sycl::reduction`. Le développeur déclare simplement une variable de réduction et l'opération à appliquer.

```cpp
sycl::buffer<int> sum_buf{&result, 1};

q.submit([&](sycl::handler& h) {
    sycl::accessor in_acc(input_buf, h, sycl::read_only);
    // Création de l'objet de réduction
    auto sum_reduction = sycl::reduction(sum_buf, h, sycl::plus<>());

    h.parallel_for(sycl::range(N), sum_reduction, 
                   [=](sycl::id i, auto& sum) {
        sum += in_acc[i];
    });
});
```

**Algorithmes de Sous-Groupe (sub-group)** : SYCL introduit un niveau de hiérarchie supplémentaire : le sub-group. Un sous-groupe est un ensemble de work-items au sein d'un work-group qui sont garantis de s'exécuter en lock-step sur le matériel SIMD du GPU.

```cpp
// À l'intérieur du noyau de multiplication de matrices
auto sg = it.get_sub_group();
//...
for (auto k = 0; k < tile_size; ++k) {
    // Le work-item 'k' du sous-groupe diffuse sa valeur 'tileA' à tous les autres
    sum += sycl::group_broadcast(sg, tileA, k) * acc_b[l + k][n];
}
```

## Partie III : L'Avenir Unifié du Parallélisme en C++ : std::execution (P2300)

Cette dernière partie explore la proposition P2300, qui vise à unifier les mondes du parallélisme CPU, du calcul hétérogène et de la programmation asynchrone sous un modèle unique et composable.

### Chapitre 6 : Le Modèle Composantiel des Senders/Receivers

#### Vision et Statut : P2300 std::execution

Le paysage actuel de la programmation asynchrone et parallèle en C++ est fragmenté. La proposition P2300, connue sous le nom de `std::execution` ou "Senders/Receivers", a pour ambition de résoudre ce problème en fournissant un vocabulaire commun et un modèle de composition unifié.

**Statut** : Après avoir été jugée trop volumineuse et immature pour C++23, elle a fait l'objet d'un travail de maturation intense et a été formellement adoptée pour être incluse dans la norme C++26 lors de la session plénière du comité WG21 en juin 2024.

#### Philosophie de Conception Fondamentale

Le modèle Senders/Receivers repose sur trois piliers philosophiques :

1. **Compositionnalité** : Les Senders sont des objets qui décrivent des unités de calcul, pouvant être assemblés et transformés pour construire des graphes de tâches complexes.

2. **Exécution Paresseuse ("Lazy")** : La création et la composition d'une chaîne de senders ne déclenchent aucune exécution. Le travail ne commence que lorsqu'un "consommateur" est attaché.

3. **Efficacité "Zéro-Overhead"** : Le modèle est conçu pour être une abstraction "zéro-overhead", adapté aux environnements les plus contraints en performance.

#### Les Concepts Clés

- **Scheduler** : Une abstraction légère, une "poignée" vers un contexte d'exécution
- **Sender** : Un objet qui décrit une opération asynchrone
- **Receiver** : Le consommateur du résultat d'un sender
- **Operation State** : La colle entre un sender et un receiver

### Chapitre 7 : Implémentation et Graphes de Tâches Asynchrones

#### Construire des Graphes de Tâches

La véritable puissance du modèle P2300 réside dans sa capacité à construire des graphes de tâches complexes de manière déclarative :

- **`just | then`** : La construction la plus simple. `just(42)` crée un sender qui se terminera immédiatement avec la valeur 42.
- **`schedule | then`** : `schedule(sch)` crée un sender qui soumet une tâche vide au scheduler.
- **`when_all`** : Prend plusieurs senders en entrée et produit un seul sender qui ne se terminera que lorsque tous les senders d'entrée seront terminés.
- **`split`** : Permet de diffuser le résultat d'un sender vers plusieurs branches de calcul.
- **`transfer`** : Crucial pour le calcul hétérogène. Transfère l'exécution sur un nouveau scheduler.

#### Exemple Concret : Orchestration Hétérogène

```cpp
// Pseudo-code illustrant le concept
#include <stdexec/execution.hpp>
#include <nvexec/stream_context.cuh> // Pour le scheduler GPU

// Schedulers
static_thread_pool cpu_pool(4);
io_uring_context io_context;
nvexec::stream_context gpu_context;

auto cpu_sched = cpu_pool.get_scheduler();
auto io_sched = io_context.get_scheduler();
auto gpu_sched = gpu_context.get_scheduler();

// Définition du graphe de tâches
auto work_graph = 
    // 1. Commence sur le scheduler I/O pour lire un fichier
    exec::schedule(io_sched)
    | exec::then([](){ return read_data_from_file("input.dat"); })
    
    // 2. Transfère les données et l'exécution sur le GPU
    | exec::transfer(gpu_sched)
    | exec::then([](auto data) {
        // Alloue de la mémoire GPU, copie les données, exécute un kernel
        return process_data_on_gpu(data);
      })
      
    // 3. Transfère le résultat sur le pool de threads CPU
    | exec::transfer(cpu_sched)
    | exec::then([](auto gpu_result) {
        // Traitement final sur le CPU
        return post_process_on_cpu(gpu_result);
      })
      
    // 4. Retourne sur le scheduler I/O pour écrire le résultat
    | exec::transfer(io_sched)
    | exec::then([](auto final_result) {
        write_result_to_file("output.dat", final_result);
      });

// 5. Exécution paresseuse : rien ne s'est encore passé.
// Le graphe est lancé et on attend le résultat.
std::this_thread::sync_wait(std::move(work_graph));
```

### Chapitre 8 : Défis, Critiques et Évolutions Futures

#### Les Défis du Modèle Sender/Receiver

1. **Complexité et Ergonomie** : Les concepts sont abstraits, et la nature fortement basée sur les templates rend le code difficile à appréhender pour les non-experts.

2. **Temps de Compilation et Diagnostics** : Les premières implémentations souffrent de temps de compilation potentiellement longs et de messages d'erreur difficiles à déchiffrer.

3. **Le Problème de l'Effacement de Type** : Dans les grandes applications, l'architecture repose souvent sur des frontières d'ABI stables et la compilation séparée. La seule solution est d'utiliser un "type-erased sender", ce qui réintroduit le surcoût que le modèle visait à éliminer.

#### Le Futur après C++26 : Le Plan de P3109R0

Conscient de ces défis, le comité de standardisation a déjà une feuille de route pour les évolutions futures :

1. **Remplacement de `ensure_started` par `async_scope`** : Une facilité de "concurrence structurée" qui garantira que toutes les tâches lancées dans sa portée sont terminées avant que le programme ne quitte cette portée.

2. **Intégration avec les Coroutines** : Un type de tâche standard basé sur les coroutines, `std::execution::task`, sera fourni.

3. **Évolutions Post-C++26** : Des algorithmes parallèles de plus haut niveau, des facilités pour les flux de données asynchrones, et une intégration transparente avec la future bibliothèque réseau standard de C++.

## Conclusion

Le voyage vers un modèle de parallélisme unifié en C++ est complexe et itératif. Des politiques d'exécution de C++17 aux abstractions hétérogènes de SYCL, et enfin au cadre ambitieux de `std::execution` pour C++26, le langage continue d'évoluer pour fournir des outils à la fois puissants, expressifs et performants. 

La maîtrise de ces différents niveaux d'abstraction, de leurs forces et de leurs faiblesses, est ce qui distingue le développeur C++ moderne et lui permet de tirer le meilleur parti du matériel, du simple CPU multi-cœur aux supercalculateurs hétérogènes de demain.
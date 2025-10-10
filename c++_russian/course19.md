# Cours Magistral : La Programmation Concurrente en C++ Moderne

## Partie 1 : Fondements de la Concurrence

### 1.1. Définition d'un Thread : Le Flux de Contrôle

Bonjour à toutes et à tous. Nous entamons aujourd'hui une série de cours dédiée à l'un des sujets les plus fondamentaux et complexes de l'informatique moderne : la programmation concurrente, ou multi-threadée. Depuis la norme C++11, le langage nous fournit des outils puissants pour écrire des programmes capables d'exécuter plusieurs tâches en parallèle. Mais avant de manipuler ces outils, il est impératif de maîtriser les concepts qui les sous-tendent.

Commençons par la question la plus élémentaire : qu'est-ce qu'un thread? Dans le langage courant, on pourrait le décrire comme une "partie exécutable d'un programme". Cependant, cette définition est imprécise et ne capture pas l'essence de ce qu'est un thread du point de vue du standard C++. La norme C++ définit un thread d'exécution (ou fil d'exécution) de manière beaucoup plus formelle et abstraite : il s'agit d'un **flux de contrôle unique au sein d'un programme** (single flow of control).

Cette distinction est capitale. Parler d'une "partie exécutable" suggère une portion statique du code, comme une fonction ou un ensemble d'instructions. La notion de "flux de contrôle", en revanche, est dynamique. Elle englobe non seulement la séquence d'instructions en cours d'exécution, mais aussi l'état complet de cette exécution : le pointeur d'instruction, l'état des registres du processeur, et surtout, la pile d'appel (call stack) avec toutes ses variables locales. Chaque thread possède son propre flux de contrôle, sa propre pile, son propre état.

L'abstraction choisie par le standard C++ est intentionnelle. En définissant un thread par son comportement (un flux de contrôle) plutôt que par son mécanisme d'implémentation, le langage reste agnostique. Que ce flux soit géré directement par le système d'exploitation (threads natifs), par une bibliothèque en espace utilisateur (threads verts ou green threads), ou par un autre moyen, cela n'affecte pas la sémantique du code C++. Cette portabilité est une pierre angulaire de la philosophie du langage.

Il est également crucial de ne pas confondre le terme "thread" avec d'autres concepts qui peuvent se traduire par "flux" en français. Un thread d'exécution n'a rien à voir avec un flux d'entrée/sortie comme `std::iostream`, qui est un flux de données. L'un est un flux de contrôle, l'autre un flux de données.

Un programme C++ commence son exécution avec au moins un thread : le thread principal, qui exécute la fonction `main`. L'ensemble de l'exécution du programme est constitué de l'exécution de tous ses threads. Par conséquent, un programme ne se termine que lorsque tous ses threads non-détachés ont terminé leur exécution.

### 1.2. Concurrence Logique vs. Parallélisme Matériel

Maintenant que nous avons défini ce qu'est un thread, nous devons distinguer deux concepts souvent confondus : la concurrence logique et le parallélisme matériel.

La **concurrence logique** (ou multithreading) est une propriété logicielle. C'est la capacité d'un programme à être structuré en plusieurs flux de contrôle qui peuvent s'exécuter indépendamment. Sur une machine dotée d'un seul cœur de processeur, ces threads ne s'exécutent pas réellement en même temps. Le système d'exploitation ou un planificateur (scheduler) bascule très rapidement d'un thread à l'autre, donnant l'illusion d'une exécution simultanée. C'est ce qu'on appelle le partage de temps (time-sharing).

Le **parallélisme matériel** est une propriété matérielle. C'est la capacité d'une machine à exécuter physiquement plusieurs instructions en même temps, typiquement grâce à plusieurs cœurs de processeur (CPU cores).

Historiquement, la concurrence a été inventée bien avant l'avènement des processeurs multi-cœurs. Cela soulève une question fondamentale : pourquoi avoir besoin de la concurrence sans le parallélisme? La réponse ne réside pas dans la performance brute, mais dans la réactivité.

Considérons les systèmes d'exploitation des années 80 et début 90. Une anecdote célèbre raconte qu'un fils demande à Bill Gates ce qu'est le multitâche, et ce dernier lui répond : "Attends, fiston, je te raconte dès que le formatage de cette disquette est terminé". Cette blague illustre un problème majeur des systèmes mono-thread : les opérations d'Entrée/Sortie (E/S) longues et bloquantes, comme l'accès à un disque, une requête réseau ou même une attente de saisie utilisateur, gelaient l'intégralité de l'application. Le flux de contrôle unique était bloqué en attente de la fin de l'opération, rendant l'interface utilisateur totalement inactive.

La concurrence logique résout ce problème. Pendant qu'un thread est bloqué en attente d'une opération d'E/S, le planificateur peut donner la main à un autre thread. Ce second thread peut, par exemple, continuer à mettre à jour l'interface graphique, à répondre aux actions de l'utilisateur, ou à effectuer d'autres calculs. La concurrence permet ainsi de masquer la latence des opérations bloquantes et d'améliorer considérablement l'expérience utilisateur et la réactivité du système.

Aujourd'hui, nous bénéficions des deux. Nous utilisons la concurrence pour structurer nos applications de manière réactive et modulaire, et nous exploitons le parallélisme matériel pour accélérer les traitements calculatoires en répartissant la charge sur plusieurs cœurs. Il est essentiel de comprendre que ces deux motivations, bien que liées, sont distinctes et influencent la conception architecturale de nos logiciels.

### 1.3. Un Bref Historique : Du Multitâche Coopératif au Préemptif

La manière dont le système bascule entre les threads a également évolué. Les premiers systèmes multitâches utilisaient un modèle **coopératif**. Dans ce modèle, chaque thread était responsable de rendre la main volontairement au planificateur en appelant une fonction spécifique (souvent nommée `yield()`). Un thread s'exécutait aussi longtemps qu'il le souhaitait. S'il entrait dans une boucle infinie sans jamais rendre la main, il pouvait bloquer l'ensemble du système.

Les systèmes d'exploitation modernes utilisent un modèle **préemptif**. Ici, le planificateur, aidé par des interruptions matérielles de l'horloge du processeur, peut interrompre un thread à n'importe quel moment (entre deux instructions) pour donner la main à un autre. Le thread est "préempté", qu'il le veuille ou non. C'est le modèle que nous considérons par défaut aujourd'hui lorsque nous parlons de `std::thread`. Il est plus robuste, car un thread malveillant ou buggé ne peut pas monopoliser le processeur.

Il est fascinant de noter que le modèle coopératif, bien que désuet pour la gestion des threads au niveau du système d'exploitation, a connu une renaissance spectaculaire sous une forme moderne et de haut niveau avec l'introduction des **coroutines en C++20**. Une coroutine est une fonction qui peut suspendre sa propre exécution à des points bien définis (en utilisant le mot-clé `co_await`) et rendre le contrôle à son appelant. Cette suspension est volontaire et explicite, tout comme dans le multitâche coopératif. L'avantage est un coût de commutation extrêmement faible par rapport à la préemption d'un thread système, ce qui rend les coroutines idéales pour gérer des dizaines de milliers d'opérations asynchrones concurrentes (comme des requêtes réseau) avec un nombre très limité de threads. Nous reviendrons en détail sur ce paradigme avancé plus tard dans ce cours.

## Partie 2 : Gestion des Threads avec la Bibliothèque Standard

### 2.1. Création et Lancement : std::thread

Depuis C++11, la bibliothèque standard fournit la classe `std::thread` dans l'en-tête `<thread>`. C'est l'outil de base pour créer et gérer des threads d'exécution. La création est simple : on instancie un objet `std::thread` en lui passant en argument une entité appelable (callable) et ses paramètres. L'entité appelable peut être un pointeur de fonction, une expression lambda, un foncteur (un objet avec `operator()`), etc.

```cpp
#include <iostream>
#include <thread>
#include <chrono>

void task(int n, const std::string& s) {
    std::cout << "Début de la tâche dans le thread " << std::this_thread::get_id() << std::endl;
    std::this_thread::sleep_for(std::chrono::seconds(n));
    std::cout << "Message : " << s << std::endl;
    std::cout << "Fin de la tâche." << std::endl;
}

int main() {
    std::cout << "Lancement d'un thread depuis main (ID: " << std::this_thread::get_id() << ")" << std::endl;

    // Crée et lance un nouveau thread qui exécutera la fonction 'task'.
    // Les arguments 2 et "Hello World" sont copiés et passés à la fonction 'task'.
    std::thread t(task, 2, "Hello World");

    // À ce stade, deux threads s'exécutent en parallèle :
    // 1. Le thread principal (qui exécute main)
    // 2. Le nouveau thread 't' (qui exécute 'task')

    // Le thread principal peut continuer à faire autre chose...
    std::cout << "Le thread principal continue son exécution." << std::endl;

    //... avant d'attendre la fin du thread 't'.
    t.join(); // Nous verrons cette fonction en détail.

    std::cout << "Le thread 't' a terminé, le programme va se terminer." << std::endl;
    return 0;
}
```

Quelques points importants à noter :

- **Lancement Immédiat** : Le nouveau thread commence son exécution dès que la construction de l'objet `std::thread` est terminée avec succès. Il n'y a pas de méthode `start()` séparée comme dans d'autres langages.

- **Passage des Arguments** : Les arguments sont copiés ou déplacés dans un stockage interne au thread au moment de sa création. Si vous devez passer un argument par référence, il faut explicitement utiliser `std::ref` ou `std::cref`.

- **Identifiant de Thread** : Chaque thread en cours d'exécution possède un identifiant unique de type `std::thread::id`. On peut obtenir l'ID du thread courant avec `std::this_thread::get_id()` et celui d'un objet `std::thread` avec sa méthode `get_id()`.

### 2.2. Cycle de Vie d'un Thread : join() vs. detach()

Un objet `std::thread` est une ressource. Comme toute ressource en C++, sa gestion doit être rigoureuse. Un `std::thread` représente un flux d'exécution actif. Avant que l'objet `std::thread` ne soit détruit (par exemple, en sortant de sa portée), vous devez impérativement décider de ce qu'il advient du flux d'exécution qu'il représente. Deux choix s'offrent à vous : `join()` ou `detach()`.

**Si un objet `std::thread` est destructible** (c'est-à-dire qu'il est associé à un thread en cours d'exécution) et que son destructeur est appelé sans qu'un `join()` ou `detach()` n'ait été appelé auparavant, le programme se termine brutalement en appelant `std::terminate`. C'est un des pièges les plus courants de `std::thread`.

- **`join()`** : La méthode `join()` est un point de synchronisation. Elle bloque le thread appelant (par exemple, le thread principal) jusqu'à ce que le thread associé à l'objet `t` ait terminé son exécution. Une fois `join()` terminé, le thread d'exécution n'existe plus, et l'objet `std::thread` devient non-joignable (il n'est plus associé à un flux d'exécution actif). C'est la méthode la plus courante et la plus sûre pour attendre la fin d'une tâche.

- **`detach()`** : La méthode `detach()` sépare l'objet `std::thread` du flux d'exécution sous-jacent. Après un appel à `detach()`, l'objet `std::thread` n'est plus joignable. Le flux d'exécution, lui, continue de vivre sa vie de manière autonome en arrière-plan. Il devient un "thread démon". Il est alors de la responsabilité du programmeur de s'assurer que ce thread ne crée pas de problèmes (par exemple, en accédant à des données qui n'existent plus après la fin de `main`). L'utilisation de `detach()` est plus rare et doit être réservée à des cas spécifiques où l'on veut lancer une tâche de fond dont on n'a pas besoin de connaître la fin.

Le choix entre `join()` et `detach()` n'est pas optionnel ; il est obligatoire. C'est au programmeur de décider explicitement du sort de chaque thread.

### 2.3. L'Évolution : std::jthread (C++20) pour une Gestion Automatique et Sûre

Le comportement de `std::thread` qui appelle `std::terminate` est dangereux, notamment en présence d'exceptions. Si une exception est levée après la création d'un `std::thread` mais avant l'appel à `join()`, le destructeur du thread sera appelé lors du déroulement de la pile, et le programme plantera.

Pour résoudre ce problème fondamental, C++20 a introduit `std::jthread` (pour "joining thread") dans l'en-tête `<thread>`. `std::jthread` est une amélioration directe de `std::thread` qui applique le principe RAII (Resource Acquisition Is Initialization) à la gestion du thread lui-même.

La caractéristique principale de `std::jthread` est que son destructeur appelle automatiquement `join()` si le thread est encore joignable.

```cpp
#include <iostream>
#include <thread> // Contient aussi std::jthread
#include <chrono>

void long_task() {
    std::this_thread::sleep_for(std::chrono::seconds(1));
    std::cout << "Tâche terminée." << std::endl;
}

void run() {
    std::cout << "Entrée dans run()" << std::endl;
    std::jthread jt(long_task);
    // Plus besoin d'appeler jt.join() manuellement.
    // Le destructeur de 'jt' sera appelé à la sortie de la fonction 'run',
    // et il attendra automatiquement la fin de 'long_task'.
    std::cout << "Sortie de run()" << std::endl;
} // <- jt.join() est appelé ici implicitement.

int main() {
    run();
    std::cout << "run() a terminé." << std::endl;
    return 0;
}
```

Ce comportement rend le code beaucoup plus sûr et robuste. Les risques de terminaison accidentelle du programme sont éliminés.

De plus, `std::jthread` intègre un mécanisme standardisé pour l'annulation coopérative. Chaque `std::jthread` est associé à un `std::stop_source` et un `std::stop_token`. Le `stop_token` peut être passé à la fonction exécutée par le thread, qui peut alors vérifier périodiquement si une demande d'arrêt a été faite (`stop_token.stop_requested()`). Cela fournit un moyen propre et standard de demander à un thread de s'arrêter prématurément.

En raison de sa sécurité et de ses fonctionnalités intégrées, `std::jthread` doit être considéré comme le choix par défaut pour la gestion des threads en C++ moderne. L'utilisation de `std::thread` devrait être limitée au code existant ou aux rares cas où la sémantique de `detach()` est explicitement requise.

## Partie 3 : Le Péril du Partage : Courses aux Données (Data Races)

La puissance de la programmation concurrente vient de la capacité des threads à collaborer en partageant des données. Cependant, ce partage est aussi sa plus grande source de danger. Le problème le plus fondamental et le plus insidieux est la course aux données (data race).

### 3.1. Anatomie d'une Course aux Données

Une course aux données se produit lorsque les conditions suivantes sont réunies :

1. Deux threads ou plus accèdent à la même zone mémoire.
2. Au moins un de ces accès est une écriture.
3. Les accès ne sont pas synchronisés par un mécanisme d'exclusion mutuelle (comme un mutex) ou des opérations atomiques.

Considérons l'exemple classique d'un compteur partagé, que deux threads tentent de modifier.

```cpp
#include <iostream>
#include <thread>
#include <vector>

long long counter = 0;
const int ITERATIONS = 1000000;

void increment() {
    for (int i = 0; i < ITERATIONS; ++i) {
        counter++;
    }
}

void decrement() {
    for (int i = 0; i < ITERATIONS; ++i) {
        counter--;
    }
}

int main() {
    std::thread t1(increment);
    std::thread t2(decrement);

    t1.join();
    t2.join();

    std::cout << "Valeur finale du compteur : " << counter << std::endl;
    return 0;
}
```

Intuitivement, on s'attendrait à ce que la valeur finale de `counter` soit 0. Un thread ajoute 1 million, l'autre retire 1 million. Cependant, si vous exécutez ce code, vous obtiendrez des résultats complètement imprévisibles et différents à chaque exécution : -470000, 112379, 1066000, etc.

La raison est que l'opération `counter++` n'est pas atomique. Au niveau du matériel, elle se décompose en trois instructions distinctes :

1. **Lecture** : Charger la valeur actuelle de `counter` de la mémoire vers un registre du processeur.
2. **Modification** : Incrémenter la valeur dans le registre.
3. **Écriture** : Écrire la nouvelle valeur du registre en mémoire.

Imaginons le scénario suivant :

1. `counter` vaut 100.
2. Thread 1 lit la valeur 100.
3. Le système d'exploitation interrompt le Thread 1 et donne la main au Thread 2.
4. Thread 2 lit la valeur 100, l'incrémente à 101, et écrit 101 en mémoire. `counter` vaut maintenant 101.
5. Le système redonne la main au Thread 1. Celui-ci a toujours la valeur 100 dans son registre. Il l'incrémente à 101 et écrit 101 en mémoire.

Deux incrémentations ont eu lieu, mais la valeur finale n'est que de 101 au lieu de 102. Une mise à jour a été perdue.

Ce phénomène, répété des millions de fois avec des interruptions imprévisibles, produit les résultats chaotiques observés.

### 3.2. La Définition Formelle et le Comportement Indéfini (UB)

Le standard C++ formalise cette notion avec une grande précision. Une exécution de programme contient une course aux données si elle contient deux actions conflictuelles (accès à la même zone mémoire, dont au moins une écriture) dans des threads différents, où au moins une n'est pas atomique, et si aucune des deux n'est ordonnancée par une relation happens-before (se produit avant) par rapport à l'autre.

La conséquence est sans appel : **tout programme contenant une course aux données a un comportement indéfini** (Undefined Behavior, UB). C'est la pire catégorie d'erreur en C++. Le compilateur a le droit de générer n'importe quel code. Le programme peut planter, donner des résultats incorrects, corrompre des données, ou pire, sembler fonctionner correctement la plupart du temps pour ne planter que dans des conditions très spécifiques et non reproductibles.

Il est intéressant de noter que les valeurs aléatoires produites par l'exemple de course aux données ont une bonne entropie. Cela s'explique par le fait que l'ordonnancement des threads par le système d'exploitation est influencé par une multitude de facteurs physiques imprévisibles, comme le bruit thermique du processeur, qui affectent le timing des interruptions. Nous avons affaire à une source de hasard quasi-authentique, mais au prix d'un programme totalement incorrect.

### 3.3. Au-delà du Code : La Cohérence des Caches Matériels

Pour comprendre la raison profonde des courses aux données, il faut descendre au niveau du matériel. Le problème ne vient pas seulement de l'entrelacement des instructions, mais de la manière dont les processeurs modernes gèrent la mémoire.

Chaque cœur de processeur (CPU core) ne travaille pas directement avec la mémoire vive (RAM), qui est relativement lente. Il possède ses propres niveaux de cache (L1, L2), qui sont des mémoires tampons beaucoup plus rapides et proches du cœur. Lorsqu'un thread sur le Cœur 1 a besoin de lire la variable `counter`, il en charge une copie dans son cache L1. Si un thread sur le Cœur 2 fait de même, il aura sa propre copie dans son propre cache L1.

Maintenant, si le Cœur 1 modifie `counter`, il ne modifie que sa copie locale dans son cache L1. Pendant un certain temps, le Cœur 2 possède une version obsolète (stale) de la donnée dans son propre cache. Les protocoles de cohérence de cache sont des mécanismes matériels complexes conçus pour s'assurer que toutes les copies d'une même donnée dans les différents caches finissent par converger vers la même valeur.

Cependant, sans instructions spécifiques, le matériel ne donne aucune garantie sur le moment où cette synchronisation se produit. C'est là qu'interviennent les primitives de synchronisation (mutex, atomiques). Lorsque vous verrouillez un mutex ou effectuez une opération atomique avec une sémantique d'ordonnancement mémoire forte, le compilateur génère des instructions spéciales, appelées **barrières mémoire** (memory fences). Ces instructions forcent le matériel à synchroniser ses caches : par exemple, une barrière de type "release" force le processeur à s'assurer que toutes ses écritures précédentes sont visibles par les autres cœurs, tandis qu'une barrière "acquire" force le processeur à s'assurer qu'il voit toutes les écritures effectuées par d'autres cœurs avant leur barrière "release".

Comprendre cette réalité matérielle est essentiel. Cela explique pourquoi des solutions naïves ne fonctionnent pas et pourquoi des mécanismes de synchronisation explicites sont absolument non négociables.

### 3.4. Le Mythe de volatile

Une erreur très répandue chez les développeurs venant d'autres domaines (comme l'embarqué) est de croire que le mot-clé `volatile` peut résoudre les problèmes de concurrence. L'intuition est que `volatile` signifie "cette variable peut changer de manière inattendue", ce qui semble correspondre à une variable modifiée par un autre thread.

C'est une erreur fondamentale. **Le mot-clé `volatile` n'a aucun effet sur la synchronisation entre threads.** Le standard C++ ne le mentionne nulle part dans sa définition des courses aux données.

Si nous modifions notre exemple de compteur en déclarant `volatile long long counter`, le résultat sera tout aussi chaotique.

```cpp
// ATTENTION : CE CODE EST TOUJOURS INCORRECT!
volatile long long counter_volatile = 0;

void increment_volatile() {
    for (int i = 0; i < ITERATIONS; ++i) {
        counter_volatile++; // Toujours une course aux données
    }
}
//...
```

### 3.5. L'Usage Correct de volatile

Alors, à quoi sert `volatile`? Son rôle est de communiquer avec le compilateur, pas avec le matériel de synchronisation. Il indique au compilateur que la valeur d'une variable peut être modifiée par des moyens qui échappent à son contrôle, en dehors du flux d'exécution du programme. Le cas d'usage typique est un registre matériel mappé en mémoire (Memory-Mapped I/O) dans un système embarqué.

Lorsque le compilateur voit un accès à une variable non-`volatile`, il est libre de l'optimiser. Par exemple, s'il voit une boucle qui lit plusieurs fois la même variable sans la modifier, il peut décider de la lire une seule fois et de stocker sa valeur dans un registre pour les lectures suivantes.

`volatile` désactive ces optimisations. Chaque lecture ou écriture d'une variable `volatile` est considérée comme un "effet de bord observable" (observable side-effect) et doit être traduite par une instruction de lecture ou d'écriture mémoire effective.

La confusion vient du fait que `volatile` et les atomiques résolvent deux problèmes distincts :

- `volatile` empêche les réordonnancements et optimisations du compilateur.
- Les atomiques et les mutex empêchent les réordonnancements du compilateur ET du matériel, et garantissent la visibilité et l'atomicité des opérations entre les cœurs.

Il existe un cas très spécifique où les deux se combinent : `volatile std::atomic<T>`. L'atomicité garantit la sécurité entre threads, et le `volatile` garantit que l'accès atomique ne sera pas optimisé par le compilateur. C'est utile dans des contextes comme les gestionnaires de signaux, où une variable peut être modifiée par un thread et par un gestionnaire de signal asynchrone. Ce cas de niche confirme la règle générale : pour la synchronisation entre threads, `volatile` seul est inutile et dangereux.

## Partie 4 : Stratégies de Synchronisation Fondamentales

Pour éviter les courses aux données, nous devons utiliser des outils de synchronisation. Le plus fondamental est le mutex.

### 4.1. Le Mutex : Verrouillage pour l'Exclusion Mutuelle (std::mutex)

Un mutex (de **MUT**ual **EX**clusion) est un objet de synchronisation qui permet de protéger une ressource partagée. Il fonctionne comme un verrou : un seul thread peut détenir le verrou à un instant donné. Une section de code protégée par un mutex est appelée une **section critique**.

La classe `std::mutex` de l'en-tête `<mutex>` fournit l'interface de base :

- `lock()`: Tente d'acquérir le verrou. Si le verrou est déjà détenu par un autre thread, le thread appelant est bloqué jusqu'à ce que le verrou soit libéré.
- `unlock()`: Libère le verrou, permettant à un autre thread en attente de l'acquérir.
- `try_lock()`: Tente d'acquérir le verrou sans bloquer. Renvoie `true` si le verrou a été acquis, `false` sinon.

En utilisant un mutex, un `unlock()` dans un thread établit une relation happens-before avec tout `lock()` ultérieur sur le même mutex dans un autre thread. Cette relation garantit que toutes les écritures mémoire effectuées avant le `unlock()` sont visibles par le thread qui effectue le `lock()`.

Voici notre exemple de compteur, corrigé avec un `std::mutex` :

```cpp
#include <iostream>
#include <thread>
#include <vector>
#include <mutex>

long long counter = 0;
const int ITERATIONS = 1000000;
std::mutex mtx; // Le mutex qui protège 'counter'

void increment() {
    for (int i = 0; i < ITERATIONS; ++i) {
        mtx.lock();
        counter++;
        mtx.unlock();
    }
}

void decrement() {
    for (int i = 0; i < ITERATIONS; ++i) {
        mtx.lock();
        counter--;
        mtx.unlock();
    }
}

int main() {
    std::thread t1(increment);
    std::thread t2(decrement);

    t1.join();
    t2.join();

    std::cout << "Valeur finale du compteur : " << counter << std::endl; // Affiche toujours 0
    return 0;
}
```

Ce code est maintenant correct et produira toujours le résultat 0.

Cependant, l'interface `lock()`/`unlock()` manuelle est dangereuse. Si une exception est levée entre `lock()` et `unlock()`, ou si un `return` prématuré est effectué, l'appel à `unlock()` sera sauté. Le mutex restera verrouillé à jamais, et tout autre thread tentant de le verrouiller sera bloqué indéfiniment. De plus, tenter de verrouiller un mutex déjà détenu par le même thread ou de déverrouiller un mutex non détenu conduit à un comportement indéfini.

### 4.2. Le Principe RAII Appliqué : std::lock_guard

La solution idiomatique en C++ pour gérer des ressources comme les verrous est le principe RAII (Resource Acquisition Is Initialization). Ce principe est incarné par la classe `std::lock_guard`.

Un `std::lock_guard` est un objet qui acquiert un mutex dans son constructeur et le libère automatiquement dans son destructeur. Cela garantit que le mutex est toujours libéré, quelle que soit la manière dont on quitte la portée de la section critique (normalement, par une exception, ou un `return`).

Notre exemple corrigé avec `std::lock_guard` :

```cpp
//... (includes et variables globales identiques)

void increment_safe() {
    for (int i = 0; i < ITERATIONS; ++i) {
        std::lock_guard<std::mutex> guard(mtx); // Verrouille mtx
        counter++;
    } // Le destructeur de 'guard' est appelé ici, mtx est déverrouillé.
}

void decrement_safe() {
    for (int i = 0; i < ITERATIONS; ++i) {
        std::lock_guard<std::mutex> guard(mtx); // Verrouille mtx
        counter--;
    } // Le destructeur de 'guard' est appelé ici, mtx est déverrouillé.
}

//... (main identique, appelant les versions _safe)
```

Ce code est non seulement correct, mais aussi robuste et sûr face aux exceptions. `std::lock_guard` a un coût d'exécution quasi nul ; c'est une pure abstraction au niveau de la compilation.

**Attention à une erreur de débutant fréquente** : créer un `lock_guard` temporaire sans le nommer. L'instruction `std::lock_guard<std::mutex>(mtx);` est valide, mais elle crée un objet temporaire qui est immédiatement détruit, libérant le verrou sur-le-champ et n'offrant aucune protection. Il faut toujours nommer sa garde : `std::lock_guard<std::mutex> lock(mtx);`.

### 4.3. Flexibilité et Contrôle : std::unique_lock

`std::lock_guard` est simple et efficace, mais rigide : le verrou est détenu pour toute la durée de vie de l'objet. Parfois, nous avons besoin de plus de flexibilité. C'est le rôle de `std::unique_lock`.

`std::unique_lock` est un autre wrapper RAII, mais il offre des fonctionnalités supplémentaires :

- **Déplaçable** : On peut transférer la propriété du verrou d'un `unique_lock` à un autre.
- **Verrouillage différé** : On peut créer un `unique_lock` sans qu'il ne verrouille immédiatement le mutex (en utilisant le tag `std::defer_lock`).
- **Déverrouillage manuel** : On peut appeler `unlock()` sur un `unique_lock` avant la fin de sa portée, et le re-verrouiller plus tard avec `lock()`.

Cette flexibilité est indispensable pour des mécanismes de synchronisation plus complexes, comme les variables de condition (`std::condition_variable`), que nous aborderons dans un cours ultérieur.

### Table 1: Comparaison des Outils de Verrouillage RAII

Pour clarifier quand utiliser chaque outil, voici un tableau comparatif.

| Caractéristique | std::lock_guard (C++11) | std::unique_lock (C++11) | std::scoped_lock (C++17) |
|---|---|---|---|
| **Principe** | Verrouillage RAII simple | Verrouillage RAII flexible | Verrouillage RAII multi-mutex |
| **Propriété** | Non-déplaçable, non-copiable | Déplaçable, non-copiable | Non-déplaçable, non-copiable |
| **Verrouillage Multiple** | Non (un seul mutex) | Non (un seul mutex) | Oui (variadique, avec anti-blocage) |
| **Déverrouillage Manuel** | Non (uniquement à la fin de la portée) | Oui (`.unlock()`) | Non (uniquement à la fin de la portée) |
| **Verrouillage Différé** | Non | Oui (`std::defer_lock`) | Non |
| **Utilisation avec std::condition_variable** | Non | Oui (requis) | Non |
| **Coût (Overhead)** | Minimal (aucun état) | Léger (stocke un booléen et un pointeur) | Léger (stocke les pointeurs des mutex) |
| **Cas d'Usage Principal** | Protéger une section critique pour toute sa portée. | Verrouillage conditionnel, utilisation avec condition_variable. | Verrouiller plusieurs mutex simultanément de manière sûre. |

## Partie 5 : Problèmes de Conception Avancés

Même avec des mutex et des gardes RAII, des problèmes de concurrence plus subtils peuvent survenir au niveau de la conception de nos classes et de nos algorithmes.

### 5.1. Les Courses à l'API (API Races)

Une course à l'API est une course aux données qui n'est pas causée par un manque de protection à l'intérieur d'une méthode, mais par la conception de l'interface (API) de la classe elle-même. Même si chaque méthode est individuellement thread-safe, leur combinaison peut ne pas l'être.

Prenons l'exemple d'une pile (`std::stack`) partagée entre plusieurs threads. L'opération typique pour récupérer un élément est de vérifier si la pile n'est pas vide, puis de prendre l'élément au sommet.

```cpp
// ATTENTION : CE CODE CONTIENT UNE COURSE À L'API!
std::stack<int> s;
std::mutex mtx_s;

void process_stack_item() {
    std::lock_guard<std::mutex> lock(mtx_s); // On protège l'accès à 's'
    if (!s.empty()) { // Étape 1 : Vérification
        int value = s.top(); // Étape 2 : Accès
        s.pop();             // Étape 3 : Modification
        //... utiliser 'value'
    }
}
```

Ce code semble sûr car un verrou est posé. Cependant, le problème est plus subtil. La course à l'API se produit si le verrou est posé à un niveau trop fin. Imaginons que chaque méthode de la pile (`empty`, `top`, `pop`) ait son propre verrou interne.

```cpp
// Scénario de course à l'API
if (!s.empty()) { // Thread 1 exécute empty(), qui verrouille, vérifie, et déverrouille. Résultat : false.
    // <-- Le planificateur interrompt Thread 1 ici.
    // Thread 2 exécute la même séquence :!s.empty() (true), s.top(), s.pop(). La pile est maintenant vide.
    // <-- Le planificateur redonne la main à Thread 1.
    s.top(); // Thread 1 appelle top() sur une pile maintenant vide. COMPORTEMENT INDÉFINI!
}
```

La fenêtre de vulnérabilité se situe entre les appels aux méthodes. La protection doit couvrir l'ensemble de l'opération logique "vérifier-puis-agir".

Un autre exemple est la course entre `top()` et `pop()`.

1. Thread 1 appelle `s.top()` et obtient la valeur 42.
2. Thread 2 appelle `s.top()` et obtient aussi la valeur 42.
3. Thread 1 appelle `s.pop()`.
4. Thread 2 appelle `s.pop()`.

Résultat : l'élément 42 a été traité deux fois, et l'élément qui se trouvait en dessous a été perdu.

La solution à ces courses à l'API est de modifier la conception de l'interface pour fournir des opérations atomiques qui combinent la vérification et l'action. Par exemple, une pile thread-safe ne devrait pas exposer `empty` et `pop` séparément, mais plutôt une méthode unique comme `bool try_pop(int& result)`.

### 5.2. L'Impasse (Deadlock)

L'impasse (ou deadlock) est un autre problème classique de la concurrence. Il se produit lorsque deux threads ou plus sont bloqués pour toujours, chacun attendant une ressource détenue par l'autre.

La condition nécessaire est une **attente circulaire**. L'exemple canonique est une fonction swap qui échange le contenu de deux objets, chacun protégé par son propre mutex.

```cpp
struct Account {
    double balance;
    std::mutex m;
};

void transfer(Account& from, Account& to, double amount) {
    std::lock_guard<std::mutex> lock_from(from.m); // Verrouille le compte source
    // Simule un travail ou une interruption
    std::this_thread::sleep_for(std::chrono::milliseconds(10));
    std::lock_guard<std::mutex> lock_to(to.m);     // Tente de verrouiller le compte destination
    
    from.balance -= amount;
    to.balance += amount;
}

int main() {
    Account a1{1000}, a2{2000};
    // Thread 1: transfère de a1 vers a2
    std::thread t1(transfer, std::ref(a1), std::ref(a2), 100);
    // Thread 2: transfère de a2 vers a1
    std::thread t2(transfer, std::ref(a2), std::ref(a1), 200);

    t1.join();
    t2.join();
    return 0; // Le programme risque de ne jamais arriver ici
}
```

Scénario de deadlock :

1. `t1` exécute `transfer(a1, a2)`. Il verrouille `a1.m`.
2. Le planificateur interrompt `t1` et lance `t2`.
3. `t2` exécute `transfer(a2, a1)`. Il verrouille `a2.m`.
4. `t1` reprend et tente de verrouiller `a2.m`, mais il est détenu par `t2`. `t1` se bloque.
5. `t2` reprend et tente de verrouiller `a1.m`, mais il est détenu par `t1`. `t2` se bloque.

Les deux threads sont maintenant bloqués indéfiniment.

### 5.3. Étude de Cas : Le Dîner des Philosophes

Ce problème académique est l'illustration parfaite du deadlock. Cinq philosophes sont assis autour d'une table. Entre chaque philosophe se trouve une fourchette. Pour manger, un philosophe a besoin des deux fourchettes adjacentes (celle de gauche et celle de droite).

L'algorithme naïf est le suivant pour chaque philosophe :

1. Prendre la fourchette de gauche.
2. Prendre la fourchette de droite.
3. Manger.
4. Reposer les deux fourchettes.

Si tous les philosophes décident de manger en même temps et exécutent l'étape 1 simultanément, chacun tiendra sa fourchette de gauche. Ils seront alors tous bloqués à l'étape 2, attendant indéfiniment que leur voisin de droite libère une fourchette que ce dernier ne libérera jamais. C'est un cas de deadlock par attente circulaire.

La solution la plus simple est de **briser la symétrie**. On établit un ordre global sur les ressources (les fourchettes, numérotées de 0 à 4). Chaque philosophe doit toujours essayer de prendre la fourchette avec le plus petit numéro en premier. Ainsi, le philosophe 4, qui a besoin des fourchettes 4 et 0, tentera de prendre la fourchette 0 en premier. Mais si le philosophe 0 l'a déjà prise, le philosophe 4 attendra. La chaîne circulaire est rompue, et le deadlock est évité.

### 5.4. La Solution Standard : std::lock et std::scoped_lock (C++17)

La stratégie d'ordonnancement des verrous est une solution générale pour éviter les deadlocks. La bibliothèque standard C++ fournit des outils pour l'appliquer automatiquement.

Depuis C++11, la fonction variadique `std::lock` peut prendre plusieurs objets verrouillables (comme des mutex) et les verrouiller tous en utilisant un algorithme qui évite les deadlocks. Cependant, son utilisation avec RAII était verbeuse :

```cpp
// Manière C++11 de verrouiller deux mutex
std::lock(from.m, to.m); // Algorithme anti-deadlock
// Adopter les verrous déjà acquis
std::lock_guard<std::mutex> lock_from(from.m, std::adopt_lock);
std::lock_guard<std::mutex> lock_to(to.m, std::adopt_lock);
```

C++17 a grandement simplifié cela avec `std::scoped_lock`. C'est un wrapper RAII variadique qui fait exactement ce travail dans son constructeur.

La fonction `transfer` corrigée et sans risque de deadlock :

```cpp
void transfer_safe(Account& from, Account& to, double amount) {
    // std::scoped_lock verrouille les deux mutex simultanément
    // en utilisant un algorithme anti-deadlock.
    std::scoped_lock lock(from.m, to.m);
    
    from.balance -= amount;
    to.balance += amount;
} // Les deux mutex sont libérés ici, dans l'ordre inverse de l'acquisition.
```

`std::scoped_lock` est la solution moderne, sûre et concise pour acquérir plusieurs verrous.

## Partie 6 : Outils de Coordination Modernes (C++20)

C++20 a considérablement enrichi la bibliothèque de concurrence avec de nouveaux outils de synchronisation de plus haut niveau, permettant de résoudre des problèmes de coordination plus complexes de manière élégante.

### 6.1. Les Sémaphores (std::counting_semaphore)

Un sémaphore est une primitive de synchronisation qui généralise le concept de mutex. Au lieu de protéger une ressource pour un accès exclusif, il contrôle l'accès à un nombre limité de ressources.

Un `std::counting_semaphore` maintient un compteur interne non négatif.

- Son constructeur prend une valeur initiale pour le compteur.
- La méthode `acquire()` décrémente le compteur. Si le compteur est à zéro, le thread est bloqué jusqu'à ce qu'il devienne positif.
- La méthode `release()` incrémente le compteur, ce qui peut débloquer un thread en attente.

Un cas d'usage typique est la gestion d'un pool de ressources (par exemple, des connexions à une base de données). Si nous avons 5 connexions disponibles, nous pouvons initialiser un sémaphore à 5. Chaque thread qui a besoin d'une connexion appelle `acquire()`. Les 5 premiers passent immédiatement. Le 6ème sera bloqué jusqu'à ce qu'un autre thread libère sa connexion en appelant `release()`.

Une différence fondamentale avec le mutex est que le thread qui appelle `release()` n'a pas besoin d'être le même que celui qui a appelé `acquire()`. Cela permet des schémas de type producteur-consommateur : un thread producteur peut signaler la disponibilité d'une nouvelle donnée en appelant `release()`, et un thread consommateur peut attendre cette donnée en appelant `acquire()`.

C++20 fournit également `std::binary_semaphore`, qui est un alias pour `std::counting_semaphore`. Il se comporte comme un mutex, mais avec la sémantique de signalisation d'un sémaphore.

### 6.2. Synchronisation en un Point : Les Verrous (std::latch)

Un `std::latch` est une barrière de synchronisation à usage unique. Il est initialisé avec un compteur.

- Les threads appellent `count_down()` pour décrémenter le compteur.
- Les threads peuvent appeler `wait()` pour se bloquer jusqu'à ce que le compteur atteigne zéro.
- Une fois que le compteur est à zéro, il le reste. Le latch ne peut pas être réinitialisé.

C'est l'outil parfait pour les scénarios de type "fork-join" : un thread principal lance plusieurs threads de travail pour effectuer une tâche d'initialisation, puis attend sur un latch que tous les travailleurs aient terminé leur part avant de continuer.

```cpp
#include <latch>
#include <vector>
#include <thread>

void worker_init(std::latch& init_latch) {
    // Fait un travail d'initialisation...
    init_latch.count_down(); // Signale la fin de son travail
}

int main() {
    const int num_workers = 4;
    std::latch init_latch(num_workers);
    std::vector<std::jthread> workers;

    for (int i = 0; i < num_workers; ++i) {
        workers.emplace_back(worker_init, std::ref(init_latch));
    }

    init_latch.wait(); // Attend que les 4 workers aient appelé count_down()

    std::cout << "Toute l'initialisation est terminée. Le programme principal continue." << std::endl;
    return 0;
}
```

### 6.3. Synchronisation Cyclique : Les Barrières (std::barrier)

Un `std::barrier` est une barrière de synchronisation réutilisable. C'est le grand frère du latch.

- Il est initialisé avec un nombre de threads participants.
- Les threads arrivent à la barrière en appelant `arrive_and_wait()`.
- Lorsqu'un thread appelle `arrive_and_wait()`, il est bloqué.
- Une fois que le nombre attendu de threads est arrivé, tous les threads bloqués sont libérés simultanément, et la barrière se réinitialise pour la phase suivante.

Les barrières sont idéales pour les algorithmes parallèles itératifs, où tous les threads doivent terminer l'itération N avant que quiconque puisse commencer l'itération N+1.

Une fonctionnalité puissante de `std::barrier` est la **fonction de complétion** (completion function). C'est une fonction qui peut être passée au constructeur de la barrière. Après que tous les threads sont arrivés, mais avant qu'ils ne soient libérés, un des threads exécute cette fonction. Cela peut être utilisé pour agréger les résultats de l'itération N avant de commencer l'itération N+1.

### 6.4. Pointeurs Intelligents Atomiques : std::atomic<std::shared_ptr>

Avant C++20, `std::shared_ptr` offrait une garantie de sécurité partielle : son bloc de contrôle (qui contient le compteur de références) était géré de manière atomique, ce qui signifie que copier et détruire des `shared_ptr` depuis plusieurs threads était sûr. Cependant, l'objet `std::shared_ptr` lui-même n'était pas atomique. Pour modifier un `shared_ptr` partagé (par exemple, le faire pointer vers un autre objet), il fallait utiliser un mutex externe.

C++20 a comblé cette lacune en introduisant des spécialisations pour `std::atomic<std::shared_ptr<T>>` et `std::atomic<std::weak_ptr<T>>`. Ces types permettent d'effectuer des opérations atomiques (comme `load`, `store`, `exchange`) directement sur le pointeur intelligent. Cela simplifie énormément l'écriture de structures de données concurrentes sans verrous (lock-free), où le transfert de propriété d'objets entre threads doit être atomique.

## Partie 7 : Vers la Programmation Asynchrone : Les Coroutines (C++20)

Jusqu'à présent, nous avons exploré un modèle de concurrence basé sur les threads. Ce modèle est puissant, mais il a ses limites, notamment en termes de scalabilité.

### 7.1. Motivation : Les Limites du Modèle Basé sur les Threads

Chaque `std::thread` correspond généralement à un thread du système d'exploitation. Ces threads sont des ressources coûteuses :

- **Mémoire** : Chaque thread nécessite sa propre pile, qui peut occuper 1 Mo ou plus d'espace mémoire. Créer des milliers de threads peut consommer des gigaoctets de RAM.
- **Coût de commutation** : Le passage d'un thread à un autre par le planificateur du système d'exploitation (commutation de contexte) est une opération lente, car elle nécessite de sauvegarder l'état complet du processeur et de modifier les mappages mémoire.

Pour les applications qui doivent gérer un très grand nombre de tâches concurrentes, en particulier des tâches liées aux E/S (comme un serveur web traitant des milliers de connexions simultanées), le modèle "un thread par tâche" n'est pas viable. La plupart de ces threads passeraient leur temps bloqués en attente de données réseau, gaspillant des ressources précieuses.

Les coroutines de C++20 offrent une solution à ce problème de scalabilité. Elles sont un modèle de concurrence beaucoup plus léger.

### 7.2. Introduction aux Coroutines : Fonctions Suspendables

Une coroutine est une fonction qui peut suspendre son exécution à un point précis, rendre le contrôle à son appelant, et être reprise plus tard au même point, en conservant tout son état local (variables, etc.).

Contrairement à une fonction normale dont la pile est détruite à son retour, l'état d'une coroutine est généralement alloué sur le tas dans une structure appelée "trame de coroutine" (coroutine frame). Cela permet à son état de survivre entre les points de suspension. Comme ces trames sont beaucoup plus petites qu'une pile de thread, on peut gérer des millions de coroutines actives avec une consommation mémoire raisonnable.

### 7.3. Les Mots-clés : co_await, co_yield, co_return

Une fonction devient une coroutine dès qu'elle utilise l'un des trois nouveaux mots-clés :

- **`co_await`** : C'est l'opérateur de suspension. Il est utilisé pour attendre le résultat d'une opération asynchrone (par exemple, une lecture sur un socket). Pendant que l'opération est en cours, la coroutine se suspend, libérant le thread sous-jacent pour qu'il puisse exécuter une autre coroutine ou une autre tâche. Une fois l'opération terminée, la coroutine est reprise. La magie de `co_await` est qu'il permet d'écrire du code asynchrone qui a l'apparence et la lisibilité du code synchrone et séquentiel.

- **`co_yield`** : Utilisé pour implémenter des générateurs asynchrones. La coroutine produit une valeur et se suspend. L'appelant consomme la valeur, puis peut reprendre la coroutine pour qu'elle produise la valeur suivante.

- **`co_return`** : Utilisé pour retourner la valeur finale d'une coroutine.

Le comportement précis d'une coroutine (quand elle se suspend, comment elle est reprise, comment elle gère sa valeur de retour) est hautement personnalisable via un type associé appelé `promise_type`.

### 7.4. Cas d'Usage : Simplification du Code Asynchrone

Avant les coroutines, le code asynchrone en C++ reposait souvent sur des callbacks ou des chaînes de `std::future::then()`. Cela menait fréquemment à un code complexe, difficile à lire et à maintenir, connu sous le nom de "callback hell".

Les coroutines transforment radicalement l'écriture de ce code.

**Code asynchrone avec des futures (style pré-C++20) :**

```cpp
std::future<Data> retrieve_data(Key key) {
    return async_get_data(key).then((auto rem_data_future) {
        auto rem_data = rem_data_future.get();
        return process(rem_data);
    });
}
```

**Même logique avec des coroutines (C++20) :**

```cpp
future<Data> retrieve_data(Key key) {
    auto rem_data = co_await async_get_data(key);
    co_return process(rem_data);
}
```

La version coroutine est linéaire, lisible et ressemble à du code synchrone simple, tout en étant entièrement asynchrone et non bloquante.

Il est important de noter que C++20 fournit la mécanique de base du langage pour les coroutines, mais très peu de support dans la bibliothèque standard. Pour les utiliser en pratique, les développeurs doivent s'appuyer sur des bibliothèques tierces (comme cppcoro de Lewis Baker) ou écrire leurs propres types "awaitables" et "promises", ce qui représente une courbe d'apprentissage non négligeable.

## Conclusion

Ce cours a couvert les concepts fondamentaux de la programmation concurrente en C++, depuis la définition d'un thread jusqu'aux outils de coordination les plus modernes. Nous avons vu que la concurrence est un outil puissant, mais semé d'embûches : courses aux données, deadlocks, et courses à l'API. La clé pour écrire du code concurrent correct et robuste réside dans une discipline stricte et l'utilisation judicieuse des primitives de synchronisation fournies par le langage.

L'évolution du C++, de `std::thread` à `std::jthread`, de `std::lock_guard` à `std::scoped_lock`, et l'introduction d'outils comme les sémaphores, les barrières et les coroutines en C++20, montre une tendance claire : rendre la programmation concurrente plus sûre, plus expressive et plus performante. La maîtrise de ces outils modernes est aujourd'hui indispensable pour tout développeur C++ souhaitant exploiter pleinement la puissance des architectures matérielles contemporaines.

Dans les prochains cours, nous approfondirons ces sujets en construisant nos propres classes thread-safe, en explorant les variables de condition, et en concevant des conteneurs concurrents comme des files d'attente producteur-consommateur. Merci de votre attention.
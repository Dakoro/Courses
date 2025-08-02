8. Comment mon ordinateur empêche‑t‑il les processus de se gêner mutuellement ?

Le planificateur (scheduler) du noyau se charge de partager le temps de calcul entre les processus. Votre système d’exploitation doit également les séparer dans l’espace mémoire, afin qu’un processus ne puisse pas corrompre la mémoire de travail d’un autre en cas de bug ou d’attaque. L’ensemble des mécanismes mis en œuvre pour résoudre ce problème s’appelle la gestion de la mémoire (memory management).

Chaque processus a besoin d’une zone mémoire qui lui soit propre pour exécuter son code et stocker ses variables et résultats. On peut voir cette zone comme composée de :

* un segment de code en lecture seule, contenant les instructions du processus,
* un segment de données inscriptible, contenant toutes ses variables.

Le segment de données est unique à chaque processus, tandis que si plusieurs processus exécutent le même programme, Unix fait partager le segment de code pour économiser de la mémoire.

##### 8.1. Mémoire virtuelle : la version simple

La mémoire est coûteuse ; il arrive qu’on n’en ait pas assez pour charger entièrement tous les programmes en cours, notamment de gros programmes comme un serveur X. Pour pallier cela, Unix utilise la mémoire virtuelle. Il ne conserve en RAM qu’un petit « jeu de travail » (working set) et déplace le reste de l’état des processus dans une zone d’échange (swap) sur le disque.

Autrefois, la RAM était si limitée que l’échange sur disque était quasi permanent. Aujourd’hui, avec des machines à 64 Mo ou plus, on peut souvent faire tourner X et d’autres applications sans jamais échanger, une fois le programme chargé en mémoire.

##### 8.2. Mémoire virtuelle : la version détaillée

En réalité, il existe au moins cinq types de mémoire dans un ordinateur, chacun ayant un coût et une vitesse d’accès différents : registres du processeur, cache interne (on‑chip), cache externe (off‑chip), mémoire principale (RAM) et disque. Plus la mémoire est rapide, plus elle est chère ; les registres sont les plus rapides (≃ 1000 M accès/s) et le disque le plus lent (≃ 100 accès/s).

La mémoire virtuelle fait croire aux programmes qu’ils disposent d’un espace d’adressage contigu et plus grand que la RAM physique. Le système d’exploitation déplace alors, en cache invisible, des blocs de données appelés « pages » entre la RAM et l’espace d’échange pour entretenir cette illusion. Plus un programme présente de la localité de référence (références mémoire proches dans le temps et dans l’espace), moins il génère d’opérations d’échange, et plus la mémoire virtuelle reste performante. L’algorithme le plus répandu pour choisir les pages à décharger est LRU (Least Recently Used).

Les niveaux de cache (interne et externe) fonctionnent de façon analogue : chaque cache sert de « fenêtre » sur le niveau de mémoire immédiatement inférieur, en conservant les lignes de données récemment utilisées. Cela comble l’écart de performance entre le processeur et la RAM.

##### 8.3. L’unité de gestion de mémoire (MMU)

Même sans échange sur disque, le gestionnaire de mémoire doit garantir qu’un processus ne peut pas écrire hors de son segment de données. Pour cela, l’OS maintient une table des segments (code et données) associée à chaque processus et la charge dans l’unité matérielle MMU (Memory Management Unit) du processeur. La MMU pose des « clôtures » autour des zones autorisées ; toute tentative d’accès hors limite déclenche une interruption.

Si vous voyez un message Unix « Segmentation fault » ou « core dumped », c’est qu’un programme a tenté d’accéder à de la mémoire en dehors de son segment, ce qui a généré une interruption fatale et produit un fichier de vidage (« core dump ») pour aider au débogage.

Outre la mémoire, Unix protège également les processus via des permissions de fichiers, afin qu’un programme défaillant ou malveillant ne puisse pas corrompre les données critiques du système.

3. Que se passe‑t‑il lorsque vous allumez un ordinateur ?

Un ordinateur sans programme en cours d’exécution n’est rien d’autre qu’un amas inerte d’électronique. La première chose qu’un ordinateur doit faire lorsqu’on l’allume est de lancer un programme spécial appelé système d’exploitation. Le rôle du système d’exploitation est d’aider les autres programmes en gérant les détails complexes du contrôle du matériel de l’ordinateur.

Le processus de démarrage du système d’exploitation s’appelle le « boot » (à l’origine « bootstrapping », évoquant l’idée de se hisser soi‑même « par ses lacets »). Votre ordinateur sait comment booter grâce à des instructions intégrées dans l’une de ses puces, la puce BIOS (Basic Input/Output System).

La puce BIOS lui indique de chercher à un emplacement fixe, généralement sur le disque dur de numéro le plus bas (le disque de démarrage), un programme spécial appelé chargeur d’amorçage (sous Linux, il s’appelle Grub ou LILO). Le chargeur d’amorçage est chargé en mémoire et exécuté. Sa mission est de lancer le véritable système d’exploitation.

Le chargeur procède en recherchant un noyau (kernel), en le chargeant en mémoire et en le démarrant. Si vous êtes sous Linux et que vous voyez « LILO » s’afficher suivi de plusieurs points, c’est le chargement du noyau : chaque point signifie qu’un autre bloc de disque contenant du code noyau a été chargé.

Vous vous demandez peut‑être pourquoi le BIOS ne charge pas le noyau directement, sans passer par ce chargeur ? En fait, le BIOS est très limité : il ne peut pas accéder à suffisamment de disque pour charger le noyau en une seule fois. Le chargeur d’amorçage permet aussi de choisir entre plusieurs systèmes d’exploitation installés à différents endroits du disque, au cas où Unix ne vous suffirait pas.

Une fois le noyau démarré, il doit détecter le reste du matériel et se préparer à exécuter des programmes. Pour cela, il interroge non pas la mémoire ordinaire, mais des ports E/S : des adresses de bus spéciales où sont à l’écoute les circuits de commande des périphériques. Le noyau ne balaie pas au hasard ; il possède une connaissance précise de ce qu’il est susceptible de trouver et de la manière dont les contrôleurs réagissent. Ce processus s’appelle l’autoprobing.

Vous pouvez voir ou non ces opérations. Autrefois, sur les systèmes Unix en mode texte, vous auriez vu défiler des messages de démarrage à l’écran. De nos jours, la plupart des Unix cachent ces messages derrière une image graphique de démarrage ; vous pouvez parfois les afficher en basculant vers une console texte avec Ctrl + Maj + F1, et revenir à l’écran graphique avec une autre combinaison (Ctrl + Maj + F7, F8 ou F9).

La plupart des messages émis au démarrage correspondent au noyau qui autoprobe votre matériel, identifie ce qui est disponible et s’adapte à votre machine. Le noyau Linux excelle dans ce domaine, mieux que la plupart des autres Unix, et bien mieux que DOS ou Windows. Beaucoup d’anciens utilisateurs de Linux estiment que l’efficacité de ces sondes au démarrage (qui facilitaient grandement l’installation) a été un facteur clé du succès de Linux.

Mais charger et démarrer le noyau n’est que la première phase du processus de démarrage (parfois appelée niveau d’exécution 1). À l’issue de cette étape, le noyau cède le contrôle à un processus spécial appelé « init », qui lance plusieurs processus de maintenance. (Certaines distributions récentes utilisent un programme différent, « upstart », qui remplit des fonctions similaires.)

La première tâche d’init est généralement de vérifier l’intégrité de vos disques. Les systèmes de fichiers sont fragiles ; s’ils ont été endommagés par une panne matérielle ou une coupure de courant, il est prudent d’effectuer des opérations de récupération avant de poursuivre le démarrage. Nous y reviendrons plus tard, lorsque nous parlerons des défaillances possibles des systèmes de fichiers.

Ensuite, init lance plusieurs démons : des programmes tels qu’un spouleur d’impression, un serveur de messagerie ou un serveur Web, qui attendent silencieusement des requêtes. Ces démons coordonnent souvent plusieurs demandes susceptibles d’entrer en conflit. Il est plus simple d’écrire un programme unique qui reste actif en permanence et gère toutes les requêtes que de lancer plusieurs copies simultanées risquant de se gêner mutuellement. La sélection des démons dépend de votre système, mais comprend presque toujours un spouleur d’impression.

L’étape suivante consiste à préparer l’interaction avec les utilisateurs. Init démarre plusieurs instances du programme getty pour surveiller vos consoles virtuelles (écran et clavier, et éventuellement ports série pour connexion par modem). Vous n’en verrez sans doute pas la plupart, car l’une des consoles est rapidement occupée par le serveur X (dont nous parlerons bientôt).

Il reste un dernier volet : le démarrage des démons gérant le réseau et d’autres services. Le plus important est le serveur X : un démon qui prend en charge votre écran, votre clavier et votre souris, et affiche les pixels colorés que vous voyez.

Quand le serveur X démarre, en fin de processus de boot, il prend le contrôle du matériel, et vous vous retrouvez devant l’écran graphique de connexion fourni par un gestionnaire d’affichage (display manager).

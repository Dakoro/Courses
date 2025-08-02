5. Que se passe‑t‑il lorsque vous lancez des programmes après le démarrage ?

Après le démarrage et avant d’exécuter un programme, vous pouvez imaginer votre ordinateur comme un zoo de processus qui attendent tous qu’on leur donne quelque chose à faire. Ils sont tous en attente d’événements. Un événement peut être l’appui d’une touche ou un mouvement de souris. Ou, si votre machine est connectée à un réseau, un événement peut être l’arrivée d’un paquet de données sur ce réseau.

Le noyau (kernel) est l’un de ces processus. C’est un processus spécial, car il contrôle l’exécution des autres processus utilisateurs et dispose normalement du seul accès direct au matériel de la machine. En fait, les processus utilisateurs doivent faire des requêtes au noyau lorsqu’ils souhaitent recevoir une entrée clavier, écrire à l’écran, lire ou écrire sur le disque, ou réaliser toute action autre que le simple traitement de données en mémoire. Ces requêtes s’appellent des appels système (system calls).

Normalement, toutes les opérations d’E/S passent par le noyau afin qu’il puisse les ordonnancer et empêcher les processus de se gêner mutuellement. Quelques processus utilisateurs spéciaux peuvent toutefois contourner partiellement le noyau, généralement en obtenant un accès direct aux ports d’E/S : c’est le cas typique des serveurs X.

Vous pouvez lancer des programmes de deux façons : via votre serveur X ou via un shell. Souvent, vous ferez les deux, car vous démarrerez un émulateur de terminal qui mime une console texte traditionnelle et vous donne un shell. Je décrirai d’abord ce qui se passe dans ce cas, puis j’expliquerai le fonctionnement lorsqu’on lance un programme depuis un menu X ou une icône sur le bureau.

Le shell s’appelle shell car il entoure et dissimule le noyau du système d’exploitation. L’une des forces d’Unix est cette séparation entre shell et noyau, deux programmes distincts communiquant par un petit jeu d’appels système. Cela permet d’avoir plusieurs shells, adaptés à divers goûts d’interface.

Le shell par défaut vous affiche l’invite « \$ » après la connexion (à moins que vous ne l’ayez personnalisée). Nous n’aborderons pas ici la syntaxe du shell ni les éléments simples visibles à l’écran ; concentrons‑nous plutôt sur ce qui se passe en coulisses.

Le shell est un simple processus utilisateur, pas particulièrement spécial. Il attend vos frappes, en écoutant (via le noyau) le port d’E/S clavier. Dès que le noyau reçoit une touche, il la répercute à votre console virtuelle ou à l’émulateur de terminal X. Quand le noyau détecte la touche Entrée, il transmet la ligne de texte au shell, qui tente de l’interpréter comme une commande.

Supposons que vous tapiez `ls` puis Entrée pour invoquer le lister de répertoire Unix. Le shell applique ses règles internes pour comprendre que vous voulez exécuter le programme `/bin/ls`. Il effectue un appel système demandant au noyau de lancer `/bin/ls` comme nouveau processus enfant, avec accès à l’écran et au clavier via le noyau. Ensuite, le shell se met en veille, en attendant que `ls` se termine.

Lorsque `/bin/ls` a fini, il informe le noyau de sa sortie via un appel système `exit`. Le noyau réveille alors le shell et lui indique qu’il peut reprendre l’exécution. Le shell affiche une nouvelle invite et attend une autre ligne de commande.

D’autres événements peuvent se dérouler pendant que votre `ls` s’exécute (supposons que vous listiez un répertoire très volumineux). Vous pourriez basculer vers une autre console virtuelle, vous y connecter et lancer un jeu comme Quake, par exemple. Ou, si vous êtes connecté à Internet, votre machine peut envoyer ou recevoir du courrier pendant que `/bin/ls` tourne.

Lorsque vous lancez des programmes via le serveur X plutôt que par un shell (en sélectionnant une application dans un menu déroulant ou en double‑cliquant une icône sur le bureau), plusieurs programmes liés à votre serveur X peuvent jouer le rôle du shell et démarrer l’application. Je passe ici sur les détails, car ils varient et sont peu importants. L’essentiel est que le serveur X, contrairement à un shell normal, ne se met pas en veille pendant l’exécution du programme client : il reste entre vous et le client, transmettant clics de souris et frappes clavier, et réalisant ses demandes d’affichage à l’écran.

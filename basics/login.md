4. Que se passe‑t‑il lorsque vous vous connectez ?

Lorsque vous vous connectez, vous vous identifiez auprès de l’ordinateur. Sur les Unix modernes, cela se fait généralement via un gestionnaire d’affichage graphique. Il est cependant possible de basculer entre les consoles virtuelles avec une combinaison de touches Ctrl + Maj et d’effectuer une connexion textuelle : dans ce cas, c’est l’instance de getty surveillant cette console qui appelle le programme login.

Vous vous identifiez auprès du gestionnaire d’affichage ou de login à l’aide d’un nom d’utilisateur et d’un mot de passe. Le nom d’utilisateur est recherché dans le fichier `/etc/passwd`, qui contient une série de lignes décrivant chacune un compte utilisateur.

L’un des champs de chaque ligne est une version chiffrée du mot de passe du compte (parfois, ces champs chiffrés sont stockés dans un second fichier `/etc/shadow` aux permissions plus restreintes, ce qui complique le cassage de mots de passe). Ce que vous saisissez comme mot de passe est chiffré de la même manière, et le programme login vérifie la correspondance. La sécurité de cette méthode repose sur le fait qu’il est facile de passer du mot de passe en clair à la version chiffrée, mais très difficile de faire l’inverse. Ainsi, même si quelqu’un peut voir la version chiffrée de votre mot de passe, il ne peut pas utiliser votre compte. (Cela signifie aussi que si vous oubliez votre mot de passe, vous ne pouvez pas le récupérer, seulement le réinitialiser.)

Une fois la connexion réussie, vous obtenez tous les privilèges associés au compte utilisé. Vous pouvez aussi être reconnu comme membre d’un groupe. Un groupe est un ensemble nommé d’utilisateurs configuré par l’administrateur système ; il peut disposer de privilèges indépendants de ceux de ses membres. Un utilisateur peut appartenir à plusieurs groupes. (Pour plus de détails sur les privilèges Unix, voir la section sur les permissions.)

(Remarque : bien que vous fassiez normalement référence aux utilisateurs et groupes par leur nom, ils sont en interne représentés par des identifiants numériques. Le fichier `/etc/passwd` mappe votre nom de compte à un UID ; le fichier `/etc/group` mappe les noms de groupes à des GID. Les commandes de gestion des comptes et groupes effectuent automatiquement cette traduction.)

L’entrée de votre compte contient également votre répertoire personnel (`home`), l’emplacement dans le système de fichiers Unix où résident vos fichiers personnels. Enfin, elle indique votre shell, l’interpréteur de commandes que login lancera pour accepter vos instructions.

Ce qui se passe ensuite dépend du mode de connexion :

* **Console texte** : login lance directement un shell, et vous êtes prêt à exécuter des commandes.
* **Gestionnaire d’affichage** : le serveur X affiche votre environnement de bureau graphique, et vous pouvez lancer des programmes via les menus, les icônes ou un émulateur de terminal exécutant un shell.

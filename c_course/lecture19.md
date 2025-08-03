# L'Algorithme Glouton : De l'Intuition à la Théorie des Matroïdes

## Introduction : La Stratégie Gloutonne et ses Promesses

L'approche gloutonne, ou "greedy", représente l'une des stratégies les plus intuitives et fondamentales dans la conception d'algorithmes. Son principe est d'une simplicité désarmante : à chaque étape d'un processus de décision, on effectue le choix qui paraît localement optimal, avec l'espoir que cette séquence de choix optimaux locaux mènera à une solution globalement optimale. Cette méthode est séduisante car elle transforme un problème d'optimisation complexe, qui pourrait nécessiter l'exploration d'un nombre exponentiel de solutions, en une série de décisions simples et rapides.

Cependant, cette simplicité cache un dilemme profond. D'un côté, l'approche gloutonne est l'épine dorsale de certains des algorithmes les plus efficaces et les plus célèbres, résolvant des problèmes fondamentaux en théorie des graphes, en ordonnancement et en compression de données. D'un autre côté, une application naïve et non justifiée de cette stratégie conduit très souvent à des solutions sous-optimales, voire complètement erronées. Comme nous le verrons, le choix de la plus grosse pièce de monnaie ne donne pas toujours le rendu optimal, et la tâche la plus profitable n'est pas toujours celle qu'il faut accomplir en premier.

Ce cours se propose de dépasser la simple mémorisation d'algorithmes gloutons. Notre objectif est de répondre à une question bien plus fondamentale et essentielle pour tout informaticien ou mathématicien : **Quand l'approche gloutonne est-elle correcte?** Pour ce faire, nous entreprendrons un voyage intellectuel en trois étapes. Nous commencerons par des études de cas concrets pour forger notre intuition, en analysant à la fois les succès et les échecs des stratégies gloutonnes. Ensuite, nous examinerons en détail deux algorithmes classiques pour la recherche d'arbres couvrants de poids minimum, dont les philosophies de conception divergentes nous ouvriront la voie à une compréhension plus profonde. Enfin, nous plongerons au cœur de la théorie des matroïdes et des grédoïdes pour découvrir la structure mathématique abstraite qui, seule, peut garantir l'optimalité d'un algorithme glouton. Ce parcours nous mènera de l'heuristique à la preuve, de l'intuition à la structure, et révélera l'élégance cachée derrière l'un des paradigmes algorithmiques les plus puissants.

## Partie I : Études de Cas Fondamentales d'Algorithmes Gloutons

L'objectif de cette première partie est de construire une intuition solide sur le comportement des algorithmes gloutons. À travers trois problèmes classiques, nous allons démontrer que si l'idée de "choisir le meilleur maintenant" est une heuristique puissante, sa validité dépend de manière cruciale de la définition précise de "meilleur" et de la structure sous-jacente du problème. L'analyse des stratégies qui échouent se révélera aussi instructive que la compréhension de celles qui réussissent.

### Section 1.1 : Le Problème d'Ordonnancement de Tâches à Échéances (Job Scheduling with Deadlines)

Commençons par un problème d'optimisation classique qui illustre parfaitement la subtilité requise pour formuler une stratégie gloutonne correcte.

#### Description du problème

Le problème est le suivant : nous disposons d'un ensemble de $n$ tâches, $J = \{j_1, j_2, \ldots, j_n\}$. Pour chaque tâche $j_i$, nous connaissons deux informations : un profit $p_i > 0$ qui est obtenu si la tâche est terminée, et une échéance (deadline) $d_i \geq 1$. L'exécution de n'importe quelle tâche prend une seule unité de temps. Nous ne pouvons exécuter qu'une seule tâche à la fois. L'objectif est de trouver un ordonnancement des tâches, c'est-à-dire un sous-ensemble de $J$ et un ordre d'exécution, qui maximise le profit total, sous la contrainte que chaque tâche sélectionnée doit être terminée au plus tard à son échéance.

Par exemple, si une tâche a une échéance $d_i = 3$, elle doit être exécutée dans l'un des trois premiers créneaux horaires : $[0,1)$, $[1,2)$, ou $[2,3)$.

#### Analyse des stratégies gloutonnes naïves

Face à ce problème, plusieurs stratégies gloutonnes "naturelles" viennent à l'esprit. Analysons-les pour comprendre leurs faiblesses.

**Stratégie 1 : Trier par profit décroissant.** L'idée est de s'attaquer en priorité aux tâches les plus lucratives. On trie les tâches par ordre de profit décroissant et on les ajoute à l'ordonnancement si elles peuvent être placées sans violer leur échéance.

**Échec de la stratégie :** Considérons deux tâches très profitables, $j_a$ et $j_b$, avec des profits élevés, mais qui ont toutes les deux une échéance $d_a = d_b = 1$. L'algorithme en choisira une, disons $j_a$, et la placera dans le créneau $[0,1)$. Il rejettera ensuite $j_b$ car le seul créneau disponible est déjà pris. Cependant, il aurait pu exister une multitude d'autres tâches moins profitables mais réalisables, qui auraient collectivement rapporté plus que la seule tâche $j_a$. Cette stratégie ignore la contrainte de ressource (le temps) et peut se focaliser sur un gain local élevé au détriment d'un profit global supérieur.

**Stratégie 2 : Trier par échéance croissante.** L'idée ici est de traiter les tâches les plus urgentes en premier pour se donner plus de flexibilité pour les tâches futures.

**Échec de la stratégie :** Imaginons une tâche $j_c$ avec un profit très faible ($p_c = 1$) mais une échéance très précoce ($d_c = 1$). L'algorithme la sélectionnera et l'exécutera dans le créneau $[0,1)$. Ce faisant, il pourrait bloquer l'exécution d'une tâche $j_d$ beaucoup plus profitable ($p_d = 100$) qui aurait également une échéance précoce, par exemple $d_d = 1$ ou $d_d = 2$. En se concentrant sur l'urgence, cette stratégie ignore la valeur et peut faire des choix localement "sûrs" mais globalement pauvres.

#### La Stratégie Gloutonne Correcte

L'échec des approches naïves suggère que la solution correcte doit trouver un équilibre entre la profitabilité et la gestion des contraintes temporelles. La stratégie optimale combine le meilleur des deux mondes avec une idée contre-intuitive : il faut penser le temps à rebours.

L'algorithme correct est le suivant :

1. Trier toutes les tâches par ordre de profit décroissant.
2. Trouver l'échéance maximale, $d_{\text{max}} = \max_i(d_i)$. Créer un tableau `slots` de taille $d_{\text{max}}$, représentant les créneaux de temps disponibles de 1 à $d_{\text{max}}$.
3. Pour chaque tâche $j$ de la liste triée (de la plus profitable à la moins profitable) :
   - Rechercher un créneau libre dans le tableau `slots`, en partant de l'échéance de la tâche $j$ et en remontant vers le début. C'est-à-dire, trouver le plus grand indice $t \in [1, d_j]$ tel que `slots[t-1]` est libre.
   - Si un tel créneau $t$ est trouvé, assigner la tâche $j$ à ce créneau (marquer `slots[t-1]` comme occupé) et ajouter le profit de $j$ au total.
   - Si aucun créneau n'est trouvé, rejeter la tâche $j$.

**Pseudo-code :**

```
fonction JobScheduling(tâches, n)
  // Trier les tâches par profit décroissant
  trier(tâches) par p_i décroissant

  d_max = 0
  pour i de 1 à n
    d_max = max(d_max, tâches[i].deadline)

  slots = tableau de taille d_max, initialisé à "libre"
  profit_total = 0
  sequence_tâches = []

  pour i de 1 à n
    tâche_actuelle = tâches[i]
    // Chercher le créneau valide le plus tardif
    pour t de tâche_actuelle.deadline descendant à 1
      si slots[t-1] est "libre"
        slots[t-1] = tâche_actuelle.id
        profit_total += tâche_actuelle.profit
        ajouter tâche_actuelle à sequence_tâches
        casser la boucle interne // Passer à la tâche suivante
      fin si
    fin pour
  fin pour

  retourner (profit_total, sequence_tâches)
fin fonction
```

Cette approche fonctionne car elle fait le choix le plus profitable possible tout en minimisant l'impact sur les options futures. En plaçant une tâche dans le créneau valide le plus tardif, elle libère les créneaux plus précoces. Ces créneaux précoces sont des ressources plus précieuses car ils sont éligibles pour un plus grand nombre de tâches (toutes celles dont l'échéance est ultérieure). La stratégie gloutonne correcte est donc celle qui, à chaque étape, fait le choix le plus profitable pour le coût d'opportunité le plus faible. Utiliser un créneau tardif a un faible coût d'opportunité, car peu d'autres tâches le convoitaient. Utiliser un créneau précoce a un coût d'opportunité élevé. Cette notion de "choix sûr" qui préserve la structure du problème restant est une caractéristique fondamentale des problèmes résolubles par un algorithme glouton, une idée que nous formaliserons avec la théorie des matroïdes.

### Section 1.2 : Le Problème de Rendu de Monnaie (Change-Making Problem)

Ce deuxième exemple déplace notre attention. Il ne s'agit plus de trouver le bon algorithme glouton, mais de comprendre que pour un algorithme glouton donné, son succès dépend entièrement de la structure des données d'entrée.

#### Description du problème

Étant donné un ensemble fini de dénominations de pièces (ou de billets) $C = \{c_1, c_2, \ldots, c_k\}$, avec un stock supposé infini pour chaque dénomination, et une somme $X$ à payer, le problème est de trouver une combinaison de pièces dont la somme est $X$ et qui utilise le nombre total de pièces le plus faible possible.

L'algorithme glouton standard pour ce problème est simple et intuitif :

```
Tant que la somme restante X_rem > 0 :
  Choisir la plus grande pièce c_i ∈ C telle que c_i ≤ X_rem.
  Ajouter cette pièce à la solution et soustraire sa valeur de X_rem.
```

#### Analyse de l'échec glouton

Cette stratégie, bien que localement optimale (elle réduit la somme restante le plus rapidement possible à chaque étape), n'est pas globalement optimale pour tous les systèmes de pièces.

**Contre-exemple :**

Considérons le système de pièces $C = \{1, 3, 4\}$ et la somme $X = 6$.

**Approche gloutonne :**
- $X_{\text{rem}} = 6$. La plus grande pièce $\leq 6$ est 4. On prend une pièce de 4. $X_{\text{rem}} = 2$.
- $X_{\text{rem}} = 2$. La plus grande pièce $\leq 2$ est 1. On prend une pièce de 1. $X_{\text{rem}} = 1$.
- $X_{\text{rem}} = 1$. La plus grande pièce $\leq 1$ est 1. On prend une pièce de 1. $X_{\text{rem}} = 0$.

La solution gloutonne est $\{4, 1, 1\}$, soit 3 pièces.

**Solution optimale :**
La solution optimale est $\{3, 3\}$, soit 2 pièces.

Cet exemple illustre parfaitement ce que l'on appelle un "pas dangereux" (unsafe step). Le premier choix, celui de la pièce de 4, bien que localement optimal, nous a engagés sur un chemin qui nous empêche d'atteindre la solution globale optimale. Nous avons "gâché" la structure du problème.

#### Systèmes Canoniques

Il est crucial de noter que pour de nombreux systèmes de pièces, l'algorithme glouton est en fait optimal. C'est le cas pour la plupart des devises mondiales (par exemple, Euro : $\{1, 2, 5, 10, 20, 50\text{ centimes}, 1, 2\text{ euros}\}$) et pour tout système basé sur des puissances d'un nombre (comme le système binaire $\{1, 2, 4, 8, 16, \ldots\}$, qui est fondamental en informatique). De tels systèmes sont dits **canoniques**.

La question devient alors : comment caractériser les systèmes de pièces canoniques? La réponse est loin d'être triviale. Il n'y a pas de règle simple évidente. Par exemple, le système $\{1, 4, 9\}$ n'est pas canonique (contre-exemple pour 12 : glouton $\{9, 1, 1, 1\}$ vs optimal $\{4, 4, 4\}$), mais le système $\{1, 5, 9\}$ l'est.

Cette complexité renforce une idée clé : la validité d'un algorithme glouton n'est pas une propriété de l'algorithme seul, mais une propriété de l'interaction entre l'algorithme et la structure de l'espace du problème. La question n'est plus "cet algorithme est-il bon?", mais "cet ensemble de solutions possibles possède-t-il la bonne structure pour qu'un algorithme glouton fonctionne?". Comme nous le verrons, la réponse réside dans le fait que les systèmes canoniques induisent une structure de matroïde sur les ensembles de pièces possibles, tandis que les systèmes non canoniques ne le font pas.

### Section 1.3 : Le Problème de Planification d'Intervalles (Interval Scheduling)

Ce dernier exemple de la première partie nous ramène à un cas où une stratégie gloutonne correcte existe, mais où sa découverte nécessite une analyse fine des différentes manières de définir un "bon" choix local.

#### Description du problème

On nous donne un ensemble de $n$ requêtes, souvent visualisées comme des intervalles sur une ligne de temps. Chaque requête $i$ est définie par une heure de début $s_i$ et une heure de fin $f_i$. Deux intervalles sont dits compatibles s'ils ne se chevauchent pas (la fin de l'un doit être antérieure ou égale au début de l'autre). L'objectif est de sélectionner le plus grand sous-ensemble possible d'intervalles mutuellement compatibles.

#### Exploration et réfutation des stratégies naïves

Comme pour le problème d'ordonnancement, plusieurs heuristiques gloutonnes peuvent être envisagées, mais la plupart échouent. Il est instructif de les réfuter avec des contre-exemples :

1. **Choisir l'intervalle avec le début le plus précoce** ($s_i$ minimum) : Un intervalle très long qui commence très tôt peut être choisi, empêchant la sélection de nombreux intervalles plus courts qui auraient pu être compatibles.

2. **Choisir l'intervalle le plus court** ($f_i - s_i$ minimum) : Un intervalle très court peut se trouver au milieu d'un intervalle très long, et le choisir pourrait nous empêcher de sélectionner deux autres intervalles, un avant et un après, qui auraient été compatibles avec l'intervalle long.

3. **Choisir l'intervalle avec le moins de conflits** : Un intervalle peut avoir peu de conflits directs, mais les intervalles qu'il rend indisponibles pourraient être cruciaux pour connecter d'autres chaînes d'intervalles.

#### La Stratégie Gloutonne Optimale

La stratégie qui garantit une solution optimale est la suivante :

1. Trier les intervalles par ordre croissant de leur heure de fin ($f_i$).
2. Initialiser un ensemble de solution $A$ vide.
3. Parcourir les intervalles triés. Pour le premier intervalle, l'ajouter à $A$.
4. Pour chaque intervalle suivant $j$, si son heure de début $s_j$ est supérieure ou égale à l'heure de fin du dernier intervalle ajouté à $A$, alors ajouter l'intervalle $j$ à $A$.
5. Retourner l'ensemble $A$.

#### Preuve de correction ("Greedy Stays Ahead")

La preuve de l'optimalité de cet algorithme est un exemple classique de l'argument "le glouton reste en tête". Soit $A = \{i_1, i_2, \ldots, i_k\}$ la solution produite par l'algorithme glouton, et $O = \{j_1, j_2, \ldots, j_m\}$ une solution optimale quelconque. Les deux ensembles sont triés par heure de fin. Nous voulons montrer que $k = m$.

On peut prouver par induction que pour tout $r \geq 1$, l'heure de fin du $r$-ième intervalle de la solution gloutonne est toujours inférieure ou égale à celle du $r$-ième intervalle de la solution optimale, c'est-à-dire $f(i_r) \leq f(j_r)$.

**Base :** Pour $r = 1$, $i_1$ est l'intervalle avec l'heure de fin la plus précoce de tout l'ensemble. Donc, par définition, $f(i_1) \leq f(j_1)$.

**Induction :** Supposons que $f(i_{r-1}) \leq f(j_{r-1})$. Le $r$-ième intervalle de la solution optimale, $j_r$, doit commencer après la fin de $j_{r-1}$, donc $s(j_r) \geq f(j_{r-1})$. Par notre hypothèse d'induction, $s(j_r) \geq f(i_{r-1})$. Cela signifie que l'intervalle $j_r$ est compatible avec les $r-1$ premiers intervalles choisis par l'algorithme glouton. L'algorithme glouton, après avoir choisi $i_{r-1}$, choisit le prochain intervalle compatible $i_r$ qui a l'heure de fin la plus précoce. Puisque $j_r$ est un candidat valide, on a nécessairement $f(i_r) \leq f(j_r)$.

Cette propriété implique que la solution gloutonne "libère la ressource" (le temps) au moins aussi vite que n'importe quelle solution optimale. Si la solution optimale avait plus d'intervalles que la solution gloutonne ($m > k$), alors on pourrait utiliser la propriété $f(i_k) \leq f(j_k)$ pour montrer que l'intervalle $j_{k+1}$ est compatible avec la solution gloutonne, ce qui contredirait le fait que l'algorithme s'est arrêté à $k$ intervalles. Donc, $k = m$, et la solution gloutonne est optimale.

#### Pseudo-code et Complexité

```
fonction IntervalScheduling(intervalles, n)
  // Trier les intervalles par heure de fin croissante
  trier(intervalles) par f_i croissant

  A = {intervalles[0]}
  dernier_f = intervalles[0].fin

  pour j de 1 à n-1
    si intervalles[j].debut >= dernier_f
      ajouter intervalles[j] à A
      dernier_f = intervalles[j].fin
    fin si
  fin pour

  retourner A
fin fonction
```

La complexité est dominée par le tri initial, soit $O(n \log n)$.

Le tableau suivant résume notre analyse des différentes stratégies pour ce problème.

| Stratégie Gloutonne | Critère de Choix | Optimalité | Justification de l'Échec/Succès |
|---------------------|------------------|------------|----------------------------------|
| Début le plus précoce | $s_i$ minimum | Non | Un intervalle long commençant tôt peut bloquer plusieurs intervalles courts. |
| Durée la plus courte | $f_i - s_i$ minimum | Non | Un intervalle court peut empêcher la sélection de deux intervalles plus longs de part et d'autre. |
| Moins de conflits | Nombre minimal d'intervalles en conflit | Non | Le choix peut sembler bon localement mais mal positionner la solution pour les choix futurs. |
| Fin la plus précoce | $f_i$ minimum | Oui | Maximise la ressource (temps) restante pour les choix futurs. Prouvé par l'argument "Greedy Stays Ahead". |

Ces trois problèmes illustrent le paysage complexe des algorithmes gloutons. Nous avons vu qu'il faut parfois trouver la bonne heuristique, et que parfois, le succès dépend de la structure intrinsèque du problème. Pour unifier ces observations, nous devons maintenant nous tourner vers un domaine plus formel : la théorie des graphes et, au-delà, la théorie des matroïdes.

## Partie II : Arbres Couvrants de Poids Minimum (Minimum Spanning Trees)

L'un des domaines les plus fructueux pour les algorithmes gloutons est la théorie des graphes. Le problème de l'arbre couvrant de poids minimum (MST) est un exemple canonique. Il est particulièrement instructif car il admet au moins deux algorithmes gloutons célèbres, Kruskal et Prim, dont les approches philosophiquement distinctes nous fourniront des indices cruciaux sur les structures mathématiques plus profondes que nous cherchons à découvrir.

### Section 2.1 : Préliminaires sur la Théorie des Graphes

Avant de plonger dans les algorithmes, rappelons quelques définitions formelles essentielles.

Un **graphe non orienté** $G$ est un couple $(V, E)$, où $V$ est un ensemble fini de sommets (vertices) et $E$ est un ensemble d'arêtes (edges), chaque arête étant une paire non ordonnée de sommets $\{u, v\}$ avec $u, v \in V$.

Un **graphe pondéré** associe un poids (ou coût) $w(e)$ à chaque arête $e \in E$.

Un **chemin** est une séquence de sommets où chaque paire de sommets consécutifs est reliée par une arête.

Un **cycle** est un chemin qui commence et se termine au même sommet, sans répéter d'arêtes ou de sommets intermédiaires.

Un graphe est **connexe** s'il existe un chemin entre toute paire de sommets distincts.

Un **arbre** est un graphe connexe sans cycle. Une **forêt** est un graphe sans cycle (c'est une collection d'arbres disjoints).

Un **arbre couvrant** (spanning tree) d'un graphe connexe $G = (V, E)$ est un sous-graphe $T = (V, E')$ qui est un arbre et qui inclut tous les sommets de $V$. Si $G$ a $|V|$ sommets, tout arbre couvrant aura exactement $|V| - 1$ arêtes.

Le **problème de l'arbre couvrant de poids minimum (MST)** consiste, pour un graphe connexe et pondéré, à trouver un arbre couvrant dont la somme des poids des arêtes est minimale.

Pour représenter un graphe en mémoire, les deux approches les plus courantes sont la matrice d'adjacence et les listes d'adjacence. Une matrice d'adjacence est une matrice $|V| \times |V|$ où l'entrée $(i,j)$ contient le poids de l'arête entre les sommets $i$ et $j$ (ou une valeur sentinelle si l'arête n'existe pas). Les listes d'adjacence associent à chaque sommet une liste de ses voisins et des poids des arêtes correspondantes. Pour les graphes denses, la matrice est efficace, mais pour les graphes épars (avec beaucoup moins d'arêtes que $|V|^2$), les listes d'adjacence sont plus économes en mémoire et souvent plus rapides à parcourir.

### Section 2.2 : L'Algorithme de Kruskal

L'algorithme de Kruskal, publié par Joseph Kruskal en 1956, est l'incarnation d'une approche gloutonne globale.

#### Philosophie de l'algorithme

La stratégie de Kruskal est simple et élégante : construire l'arbre couvrant en ajoutant les arêtes une par une, de la moins chère à la plus chère, en s'assurant simplement de ne jamais créer de cycle.

L'algorithme procède comme suit :

1. Créer une forêt $F$ où chaque sommet du graphe est un arbre distinct (une composante connexe).
2. Placer toutes les arêtes du graphe dans une file de priorité (ou simplement les trier) par poids croissant.
3. Tant que la forêt $F$ a plus d'une composante (c'est-à-dire, tant que nous n'avons pas $|V| - 1$ arêtes dans notre MST) :
   a. Extraire l'arête $\{u, v\}$ de poids minimum de la liste.
   b. Si les sommets $u$ et $v$ sont déjà dans la même composante connexe de $F$, ajouter cette arête créerait un cycle. On la rejette donc.
   c. Sinon, ajouter l'arête $\{u, v\}$ à notre forêt $F$. Cela a pour effet de fusionner les deux composantes connexes qui contenaient $u$ et $v$.

#### Détection de cycles avec Union-Find (Disjoint Set Union)

L'étape cruciale de l'algorithme est l'étape 3b : comment vérifier efficacement si deux sommets appartiennent à la même composante connexe? C'est là qu'intervient la structure de données **Union-Find**, également connue sous le nom de **Disjoint Set Union (DSU)**. Cette structure est conçue spécifiquement pour gérer une partition d'un ensemble en sous-ensembles disjoints et supporte deux opérations principales de manière quasi-constante en temps amorti :

- **Find(v)**: Renvoie un identifiant unique (le "représentant" ou la "racine") pour le sous-ensemble contenant l'élément $v$. Pour vérifier si $u$ et $v$ sont dans la même composante, on teste simplement si `Find(u) == Find(v)`.
- **Union(u, v)**: Fusionne les deux sous-ensembles contenant $u$ et $v$.

Pour atteindre cette efficacité remarquable, deux optimisations sont essentielles :

1. **Union par rang/taille** : Lors de la fusion de deux arbres (représentant les sous-ensembles), on attache toujours l'arbre plus petit (en rang ou en taille) à la racine de l'arbre plus grand. Cela évite de créer des arbres longs et filiformes.

2. **Compression de chemin** : Lorsqu'on exécute `Find(v)`, on fait remonter tous les nœuds sur le chemin de $v$ à la racine pour qu'ils pointent directement vers la racine. Cela aplatit considérablement l'arbre pour les futures opérations Find.

#### Pseudo-code et Implémentation

Avec la structure DSU, le pseudo-code de Kruskal devient très concis :

```
fonction Kruskal(G = (V, E))
  MST = ∅
  DSU = nouvelle structure DisjointSetUnion pour tous les sommets de V

  // Trier les arêtes par poids croissant
  trier(E) par poids w(e) croissant

  pour chaque arête {u, v} dans E
    si DSU.Find(u) ≠ DSU.Find(v)
      ajouter {u, v} à MST
      DSU.Union(u, v)
    fin si
  fin pour

  retourner MST
fin fonction
```

#### Preuve de Correction

La preuve de l'optimalité de l'algorithme de Kruskal repose sur un argument d'échange ou, plus directement, sur la "propriété de la coupe" (cut property). L'invariant clé à maintenir par induction est le suivant : "À n'importe quelle étape de l'algorithme, l'ensemble des arêtes choisies $F$ est un sous-ensemble d'un certain arbre couvrant de poids minimum (MST)".

**Base :** Au début, $F = \emptyset$, et cet ensemble est trivialement un sous-ensemble de n'importe quel MST.

**Induction :** Supposons que $F$ soit un sous-ensemble d'un MST, disons $T$. Soit $e$ la prochaine arête choisie par Kruskal.

- Si $e \in T$, l'invariant est maintenu.
- Si $e \notin T$, alors ajouter $e$ à $T$ crée un cycle unique $C$. Ce cycle doit contenir au moins une autre arête, disons $f$, qui n'est pas dans $F$. L'arête $f$ relie les deux composantes que $e$ connecte. Puisque Kruskal a choisi $e$ avant de considérer $f$, on doit avoir $w(e) \leq w(f)$. En effet, si $w(f) < w(e)$, Kruskal aurait considéré et ajouté $f$ avant $e$. Si $w(e) = w(f)$, l'ordre de choix dépend de la politique de départage, mais on peut supposer que $e$ a été choisi.

Considérons le nouvel arbre $T' = T \setminus \{f\} \cup \{e\}$. $T'$ est bien un arbre couvrant, et son poids total est $w(T') = w(T) - w(f) + w(e) \leq w(T)$. Puisque $T$ est un MST, son poids est minimal, donc on doit avoir $w(T') = w(T)$. $T'$ est donc aussi un MST.

Crucialement, $T'$ contient $F \cup \{e\}$. L'invariant est donc maintenu.

Puisque l'invariant est vrai à chaque étape, lorsque l'algorithme se termine, l'ensemble d'arêtes produit est lui-même un sous-ensemble d'un MST et est un arbre couvrant. Il est donc nécessairement un MST.

### Section 2.3 : L'Algorithme de Prim

Développé initialement par le mathématicien tchèque Vojtěch Jarník en 1930, puis redécouvert indépendamment par Robert Prim en 1957 et Edsger Dijkstra en 1959, l'algorithme de Prim offre une perspective gloutonne différente.

#### Philosophie de l'algorithme

Contrairement à l'approche globale de Kruskal, Prim adopte une approche gloutonne locale. L'algorithme fait croître un unique arbre, toujours connexe, à partir d'un sommet de départ arbitraire. À chaque étape, il fait le choix qui est localement le plus avantageux : il ajoute à l'arbre l'arête la moins chère qui relie un sommet déjà dans l'arbre à un sommet qui n'y est pas encore.

L'algorithme se déroule comme suit :

1. Initialiser un arbre $T$ avec un seul sommet de départ, choisi arbitrairement.
2. Maintenir un ensemble de sommets $V_T$ dans l'arbre et $V \setminus V_T$ hors de l'arbre.
3. Tant que $V_T \neq V$ :
   a. Trouver l'arête $\{u, v\}$ de poids minimum telle que $u \in V_T$ et $v \in V \setminus V_T$.
   b. Ajouter cette arête à $T$ et le sommet $v$ à $V_T$.

#### Implémentation Efficace avec une File de Priorité (Min-Heap)

L'étape 3a, trouver l'arête de poids minimum qui traverse la "coupe" entre $V_T$ et $V \setminus V_T$, est l'opération clé. Une implémentation naïve consisterait à scanner toutes les arêtes possibles à chaque étape. Une approche beaucoup plus efficace utilise une file de priorité (min-heap) pour stocker les sommets de $V \setminus V_T$.

La file de priorité stocke les sommets qui ne sont pas encore dans l'arbre. La "priorité" ou la "clé" de chaque sommet $v$ est le poids de l'arête la plus légère le reliant à l'arbre en construction.

1. Initialiser les clés de tous les sommets à $\infty$, sauf pour le sommet de départ dont la clé est 0.
2. Tant que la file de priorité n'est pas vide :
   a. Extraire le sommet $u$ avec la clé minimale (extract-min). Ajouter $u$ à l'ensemble des sommets du MST.
   b. Pour chaque voisin $v$ de $u$ : si $v$ est encore dans la file de priorité et que le poids de l'arête $(u,v)$ est inférieur à la clé actuelle de $v$, mettre à jour la clé de $v$ avec ce nouveau poids plus faible (decrease-key).

#### Pseudo-code et Implémentation

Le pseudo-code utilisant une file de priorité est le suivant :

```
fonction Prim(G = (V, E), s) // s est le sommet de départ
  pour chaque sommet v dans V
    clé[v] = ∞
    parent[v] = NIL
  fin pour
  clé[s] = 0
  Q = file de priorité contenant tous les sommets de V, ordonnée par clé

  tant que Q n'est pas vide
    u = Q.Extract-Min()
    pour chaque voisin v de u
      si v est dans Q et w(u, v) < clé[v]
        parent[v] = u
        clé[v] = w(u, v)
        Q.Decrease-Key(v, clé[v])
      fin si
    fin pour
  fin tant que

  // Le MST est formé par les arêtes (v, parent[v]) pour v ≠ s
  retourner l'ensemble des arêtes (v, parent[v])
fin fonction
```

#### Preuve de Correction

La preuve de l'optimalité de Prim est très similaire à celle de Kruskal et repose également sur la propriété de la coupe. À chaque étape, l'algorithme choisit l'arête la plus légère traversant la coupe entre les sommets déjà dans l'arbre ($V_T$) et ceux qui n'y sont pas. Un lemme fondamental de la théorie des MST stipule que pour toute coupe dans le graphe, l'arête de poids minimum traversant cette coupe doit faire partie d'au moins un MST. L'algorithme de Prim, à chaque étape, fait un choix "sûr" en sélectionnant une telle arête. Comme tous ses choix sont sûrs, la solution finale est optimale.

La comparaison de ces deux algorithmes révèle une distinction fondamentale qui va bien au-delà de leur simple implémentation. Kruskal et Prim résolvent le même problème avec une efficacité asymptotique similaire, mais leurs processus internes explorent des espaces de solutions partielles radicalement différents. Kruskal construit une forêt, où les solutions partielles (les ensembles d'arêtes choisis) ont la propriété que n'importe lequel de leurs sous-ensembles est également une solution partielle valide (une forêt). C'est la **propriété d'hérédité**. En revanche, Prim construit un arbre unique et toujours connexe. Un sous-ensemble d'arêtes d'un arbre n'est pas nécessairement un arbre connexe. La structure explorée par Prim n'est pas héréditaire, mais elle possède une propriété plus faible : l'**accessibilité** (chaque solution partielle peut être obtenue en ajoutant une arête à une solution plus petite). Cet "échec" de la structure de Prim à être héréditaire, alors que l'algorithme est pourtant optimal, est la motivation parfaite pour introduire une structure combinatoire plus générale que le matroïde : le grédoïde. C'est le pont vers la théorie formelle.

## Partie III : La Théorie des Matroïdes — L'Anatomie de l'Approche Gloutonne

Nous avons observé que certains algorithmes gloutons fonctionnent et d'autres non. Nous avons vu que la validité peut dépendre de l'heuristique choisie ou de la structure des données d'entrée. Il est temps de formaliser ces intuitions. La théorie des matroïdes, initiée par Hassler Whitney en 1935, nous fournit le langage mathématique précis pour comprendre exactement quand et pourquoi une approche gloutonne est garantie d'être optimale. Elle révèle une structure unificatrice qui sous-tend des problèmes d'apparence très différente.

### Section 3.1 : Définition Formelle d'un Matroïde

Les matroïdes ont été introduits par Hassler Whitney dans son article fondateur "On the abstract properties of linear dependence". Son objectif était de capturer les propriétés essentielles de l'indépendance, qu'elle soit linéaire dans un espace vectoriel ou acyclique dans un graphe, dans un cadre purement combinatoire.

#### Définition par les Ensembles Indépendants

La définition la plus courante d'un matroïde se base sur la notion d'ensembles indépendants. Un **matroïde fini** $M$ est un couple $(E, \mathcal{I})$ où :

- $E$ est un ensemble fini, appelé l'**ensemble de base** (ground set).
- $\mathcal{I}$ est une famille de sous-ensembles de $E$, appelés les **ensembles indépendants**.

Cette famille $\mathcal{I}$ doit satisfaire les trois axiomes suivants :

**(Axiome d'Hérédité - I1 & I2 combinés) :** $\mathcal{I}$ est non vide et fermée vers le bas. C'est-à-dire :
- **(I1)** $\emptyset \in \mathcal{I}$
- **(I2)** Si $I \in \mathcal{I}$ et $J \subseteq I$, alors $J \in \mathcal{I}$

**(Axiome d'Augmentation - I3) :** Si $I$ et $J$ sont deux ensembles indépendants ($I, J \in \mathcal{I}$) et que $|I| < |J|$, alors il existe un élément $e \in J \setminus I$ tel que l'ensemble $I \cup \{e\}$ est également indépendant.

Cet axiome d'augmentation est le cœur de la structure matroïdale. Il garantit que tous les ensembles indépendants maximaux ont la même taille.

#### Terminologie et Définitions Cryptomorphes

À partir de ces axiomes, on peut définir plusieurs concepts clés qui offrent des perspectives différentes mais équivalentes sur la même structure. On dit que ces définitions sont **cryptomorphes**.

**Ensemble Dépendant :** Un sous-ensemble de $E$ qui n'est pas dans $\mathcal{I}$ est dit dépendant.

**Base :** Une base d'un matroïde est un ensemble indépendant maximal. C'est un ensemble indépendant qui ne peut être agrandi en y ajoutant un élément de $E$ sans devenir dépendant. Grâce à l'axiome d'augmentation (I3), on peut prouver que toutes les bases d'un matroïde ont la même cardinalité.

**Circuit :** Un circuit est un ensemble dépendant minimal. C'est un ensemble dépendant dont tous les sous-ensembles propres sont indépendants. Ce terme vient du matroïde graphique, où les circuits correspondent exactement aux cycles du graphe.

**Fonction de Rang :** La fonction de rang $r: 2^E \to \mathbb{Z}_{\geq 0}$ d'un matroïde associe à chaque sous-ensemble $A \subseteq E$ la taille de la plus grande partie indépendante de $A$. C'est-à-dire, $r(A) = \max\{|I| : I \in \mathcal{I}, I \subseteq A\}$. Le rang du matroïde est $r(M) = r(E)$, qui est aussi la taille de n'importe quelle base.

### Section 3.2 : L'Algorithme de Rado-Edmonds : Le Glouton Universel

Le lien profond entre les matroïdes et les algorithmes gloutons a été mis en lumière par les travaux de Jack Edmonds et Richard Rado. Ils ont montré que la structure matroïdale est précisément ce qui est nécessaire et suffisant pour que l'algorithme glouton fonctionne.

#### L'Algorithme Généralisé

Considérons un matroïde $M = (E, \mathcal{I})$ où chaque élément $e \in E$ a un poids non négatif $w(e)$. Le problème est de trouver une base de poids maximum, c'est-à-dire une base $B$ qui maximise $\sum_{e \in B} w(e)$.

L'algorithme glouton généralisé, souvent appelé **algorithme de Rado-Edmonds**, est une abstraction directe de l'algorithme de Kruskal :

1. Initialiser un ensemble indépendant $A = \emptyset$.
2. Trier les éléments de $E$ par poids décroissant.
3. Pour chaque élément $e \in E$, pris dans l'ordre trié :
   - Si $A \cup \{e\}$ est toujours un ensemble indépendant (c'est-à-dire, $A \cup \{e\} \in \mathcal{I}$), alors mettre à jour $A \leftarrow A \cup \{e\}$.
4. Retourner $A$.

#### Théorème (Rado-Edmonds, 1971)

Ce qui est remarquable, c'est que cette relation est une équivalence. Le théorème de Rado-Edmonds stipule :

**Un système d'indépendance $(E, \mathcal{I})$ (c'est-à-dire un système satisfaisant les axiomes I1 et I2) est un matroïde (c'est-à-dire qu'il satisfait aussi l'axiome d'augmentation I3) si et seulement si l'algorithme glouton ci-dessus produit une base de poids maximum pour toute fonction de poids $w: E \to \mathbb{R}^+$.**

#### Preuve de Correction

Démontrons la partie la plus instructive : si $(E, \mathcal{I})$ est un matroïde, alors l'algorithme glouton est optimal. La preuve se fait par l'absurde, en utilisant la propriété d'augmentation.

Soit $A = \{a_1, a_2, \ldots, a_r\}$ la base retournée par l'algorithme glouton, avec les éléments triés par poids décroissant ($w(a_1) \geq w(a_2) \geq \ldots$). Soit $B = \{b_1, b_2, \ldots, b_r\}$ une base optimale quelconque, également triée par poids décroissant. Nous voulons montrer que $w(A) = w(B)$.

Supposons que $A$ n'est pas optimale, donc $w(B) > w(A)$. Comparons les éléments de $A$ et $B$. Soit $k$ le premier indice pour lequel $a_k \neq b_k$. Puisque les éléments sont triés, l'algorithme glouton a choisi les éléments $a_1, \ldots, a_{k-1}$ et a ensuite considéré $a_k$. Comme $B$ est une base optimale et que $w(B) > w(A)$, il doit exister un élément dans $B$ qui est "meilleur" qu'un élément dans $A$. Il doit donc y avoir un premier indice $k$ où $w(b_k) > w(a_k)$.

Considérons l'ensemble $A_{k-1} = \{a_1, \ldots, a_{k-1}\} = \{b_1, \ldots, b_{k-1}\}$. C'est un ensemble indépendant. L'algorithme glouton, après avoir construit $A_{k-1}$, a choisi $a_k$. Cela signifie que $a_k$ est l'élément de plus grand poids parmi tous les $e \in E \setminus A_{k-1}$ tels que $A_{k-1} \cup \{e\}$ est indépendant.

Puisque $w(b_k) > w(a_k)$, l'algorithme aurait préféré $b_k$ à $a_k$. S'il ne l'a pas fait, c'est que l'ensemble $A_{k-1} \cup \{b_k\}$ doit être dépendant.

Cependant, considérons les ensembles indépendants $I = A_{k-1} \cup \{b_k\}$ et $J = \{b_1, \ldots, b_k\}$. Nous avons $|J| = k$ et $|I| \leq k$. Si $A_{k-1} \cup \{b_k\}$ était dépendant, cela contredirait le fait que $\{b_1, \ldots, b_k\}$ est un sous-ensemble d'une base (et donc indépendant). Donc $A_{k-1} \cup \{b_k\}$ est indépendant. Mais alors, l'algorithme glouton, ayant $A_{k-1}$ et voyant que $w(b_k) > w(a_k)$, aurait dû choisir $b_k$ au lieu de $a_k$. C'est une contradiction.

Cette contradiction découle de l'hypothèse que $A$ n'est pas optimale. Par conséquent, la base gloutonne est bien une base de poids maximum.

Ce théorème est d'une importance capitale. Il établit une dualité profonde entre une structure combinatoire statique (les axiomes du matroïde) et un processus algorithmique dynamique (l'algorithme glouton). La propriété "être un matroïde" et la propriété "être résoluble de manière optimale par l'algorithme glouton" sont deux facettes de la même réalité. C'est l'un des résultats les plus élégants de l'optimisation combinatoire, car il nous donne un critère clair et testable pour la validité de l'approche gloutonne.

### Section 3.3 : La Galerie des Matroïdes : Le Principe du Cryptomorphisme

Le pouvoir unificateur de la théorie des matroïdes réside dans sa capacité à modéliser une grande variété de problèmes qui, en surface, semblent n'avoir aucun rapport. Ces problèmes sont dits **cryptomorphes** : ils ont des formulations différentes mais partagent la même structure de matroïde sous-jacente. L'algorithme de Rado-Edmonds devient alors un méta-algorithme qui résout tous ces problèmes.

Le tableau suivant illustre ce principe avec plusieurs exemples fondamentaux.

| Nom du Matroïde | Ensemble de Base $E$ | Ensembles Indépendants $\mathcal{I}$ | Algorithme Glouton Concret |
|-----------------|---------------------|-------------------------------------|----------------------------|
| Graphique | Arêtes d'un graphe $G$ | Ensembles d'arêtes acycliques (forêts) | Algorithme de Kruskal |
| Linéaire | Ensemble de vecteurs d'un espace vectoriel | Sous-ensembles de vecteurs linéairement indépendants | Algorithme de base de Gauss-Jordan |
| d'Ordonnancement | Ensemble de tâches avec profit et échéance | Ensembles de tâches qui peuvent être ordonnancées sans dépasser leurs échéances | Algorithme de Job Scheduling |
| Transversal | Sommets d'un côté d'un graphe biparti | Ensembles de sommets qui sont les extrémités d'un couplage | Algorithme de Ford-Fulkerson (via chemin augmentant) |
| Uniforme $U_{k,n}$ | Un ensemble de $n$ éléments | Tous les sous-ensembles de taille au plus $k$ | Choisir les $k$ éléments de plus grand poids |

Analysons quelques-uns de ces exemples pour consolider notre compréhension :

**Matroïde Graphique :** C'est l'exemple le plus intuitif. L'ensemble de base $E$ est l'ensemble des arêtes d'un graphe $G$. Un sous-ensemble d'arêtes $F \subseteq E$ est indépendant s'il ne contient pas de cycle (c'est-à-dire si $F$ est une forêt). L'algorithme de Kruskal est précisément l'algorithme de Rado-Edmonds appliqué à ce matroïde pour trouver une base de poids minimum (ce qui est équivalent à trouver une base de poids maximum si on inverse les poids).

**Matroïde Linéaire :** C'est l'exemple qui a inspiré Whitney. L'ensemble de base $E$ est un ensemble de vecteurs dans un espace vectoriel sur un corps $\mathbb{F}$. Un sous-ensemble $I \subseteq E$ est indépendant s'il est linéairement indépendant au sens de l'algèbre linéaire. L'axiome d'augmentation (I3) est une conséquence directe du lemme de Steinitz (lemme d'échange).

**Matroïde d'Ordonnancement :** Revenons à notre premier problème. L'ensemble de base $E$ est l'ensemble des tâches. Un ensemble de tâches $I \subseteq E$ est dit indépendant s'il existe un ordonnancement valide pour cet ensemble. Nous avons vu que l'algorithme glouton (trier par profit, insérer au plus tard) fonctionne. Le théorème de Rado-Edmonds nous garantit donc que cette structure est un matroïde. Prouver directement les axiomes est un excellent exercice. L'axiome d'augmentation, en particulier, peut être démontré en montrant que si un ensemble de tâches $J$ peut être ordonnancé et qu'un ensemble $I$ plus petit peut aussi l'être, il y a forcément un "créneau" libéré par $J$ qu'une tâche de $J \setminus I$ peut occuper sans perturber la faisabilité de $I$.

**Matroïde Transversal :** Cet exemple est lié au problème d'affectation des travailleurs. Soit un graphe biparti $G = (U \cup V, E)$. L'ensemble de base est $U$. Un sous-ensemble $I \subseteq U$ est indépendant s'il existe un couplage (matching) dans $G$ qui couvre tous les sommets de $I$. La propriété d'augmentation de ce matroïde est une conséquence directe du théorème de Hall ou, de manière plus constructive, de l'existence d'un chemin augmentant dans l'algorithme de Ford-Fulkerson pour les couplages.

La reconnaissance d'une structure de matroïde dans un problème d'optimisation est une avancée majeure, car elle fournit une certification immédiate de l'optimalité de l'approche gloutonne. Cependant, comme nous l'avons pressenti avec l'algorithme de Prim, tous les problèmes résolubles par un algorithme glouton ne sont pas des matroïdes. Pour compléter notre théorie, nous devons introduire une structure plus générale.

## Partie IV : Au-delà des Matroïdes — Les Grédoïdes

La théorie des matroïdes est d'une grande élégance, mais elle ne capture pas toute la richesse des phénomènes gloutons. L'algorithme de Prim pour l'arbre couvrant de poids minimum en est l'exemple parfait : il est indubitablement glouton et optimal, mais la structure des solutions partielles qu'il explore ne forme pas un matroïde. Cette observation nous pousse à généraliser notre cadre théorique pour inclure de tels algorithmes.

### Section 4.1 : Les Limites du Matroïde et la Motivation pour les Grédoïdes

Le paradoxe de l'algorithme de Prim est le suivant : à chaque étape, il construit un arbre partiel, qui est toujours un graphe connexe. Considérons l'ensemble de base $E$ comme étant les arêtes du graphe, et la famille $\mathcal{F}$ comme étant l'ensemble des sous-ensembles d'arêtes formant un arbre partiel (connexe) à partir d'un sommet racine donné.

Cet ensemble $\mathcal{F}$ viole l'axiome d'hérédité (I2) des matroïdes. Si nous avons un arbre partiel $T \in \mathcal{F}$ composé de plusieurs arêtes, un sous-ensemble propre de ses arêtes ne formera généralement pas un arbre connexe, mais une forêt. Il ne sera donc pas dans $\mathcal{F}$. Par exemple, si les arêtes $\{e_1, e_2, e_3\}$ forment un chemin et sont dans $\mathcal{F}$, le sous-ensemble $\{e_1, e_3\}$ n'est pas connexe et donc n'est pas dans $\mathcal{F}$.

Ce qui distingue Prim de Kruskal, c'est l'importance fondamentale de l'ordre et de la contrainte de croissance. Kruskal peut choisir n'importe quelle arête "sûre" dans le graphe, indépendamment de ce qui a été construit. Prim, lui, est contraint de choisir une arête qui est adjacente à l'arbre qu'il est en train de construire. C'est cette dépendance à l'ordre et à la structure existante que les matroïdes, par leur nature d'ensembles non ordonnés, ne peuvent pas capturer. Pour modéliser de tels processus, il nous faut une structure qui remplace l'hérédité par une notion plus faible de "construction pas à pas".

### Section 4.2 : Définition et Propriétés des Grédoïdes

C'est pour combler cette lacune que Bernhard Korte, László Lovász et Rainer Schrader ont introduit le concept de **grédoïde** au début des années 1980. Le grédoïde est une généralisation du matroïde qui capture précisément cette idée de croissance accessible.

#### Définition Formelle

Un **grédoïde** est un couple $(E, \mathcal{F})$ où $E$ est un ensemble de base fini et $\mathcal{F}$ est une famille de sous-ensembles de $E$, appelés **ensembles faisables**, satisfaisant les deux axiomes suivants :

**(Axiome d'Accessibilité - G1) :** L'ensemble vide est faisable ($\emptyset \in \mathcal{F}$), et pour tout ensemble faisable non vide $X \in \mathcal{F}$, il existe au moins un élément $x \in X$ tel que $X \setminus \{x\}$ est également faisable.

Cet axiome implique que tout ensemble faisable peut être "construit" à partir de l'ensemble vide en ajoutant un élément à la fois, chaque étape intermédiaire étant elle-même un ensemble faisable.

**(Axiome d'Augmentation - G2) :** Si $X, Y \in \mathcal{F}$ et $|X| > |Y|$, alors il existe un élément $x \in X \setminus Y$ tel que $Y \cup \{x\}$ est également faisable.

C'est le même axiome d'augmentation que pour les matroïdes. Il assure que toutes les bases (ensembles faisables maximaux) ont la même taille.

La différence cruciale réside dans le remplacement de l'axiome d'hérédité (I2) par l'axiome d'accessibilité (G1), qui est plus faible. Le tableau suivant met en évidence cette distinction.

| Axiome | Matroïde $(M = (E, \mathcal{I}))$ | Grédoïde $(G = (E, \mathcal{F}))$ | Commentaire |
|--------|-----------------------------------|-----------------------------------|-------------|
| Hérédité | Si $I \in \mathcal{I}$ et $J \subseteq I$, alors $J \in \mathcal{I}$. | Non requis. | C'est la différence fondamentale. |
| Accessibilité | Implicite par l'hérédité. | Si $X \in \mathcal{F}, X \neq \emptyset$, $\exists x \in X$ t.q. $X \setminus \{x\} \in \mathcal{F}$. | Le grédoïde affaiblit l'hérédité en accessibilité. |
| Augmentation | Si $I, J \in \mathcal{I}, |I| < |J|$, $\exists e \in J \setminus I$ t.q. $I \cup \{e\} \in \mathcal{I}$. | Si $X, Y \in \mathcal{F}, |X| > |Y|$, $\exists x \in X \setminus Y$ t.q. $Y \cup \{x\} \in \mathcal{F}$. | Identique dans les deux structures. |

De cette comparaison, il découle qu'un matroïde est simplement un grédoïde qui satisfait en plus la propriété d'hérédité.

#### Le Grédoïde de Branchement (Branching Greedoid)

Revenons à l'algorithme de Prim. L'ensemble $\mathcal{F}$ des sous-ensembles d'arêtes formant un arbre connexe à partir d'une racine $r$ est un exemple parfait de grédoïde qui n'est pas un matroïde.

**Accessibilité (G1) :** Tout arbre non vide a au moins une feuille (un sommet de degré 1). En enlevant l'arête incidente à une feuille (qui n'est pas la racine), on obtient un arbre plus petit, qui est toujours dans $\mathcal{F}$. La propriété est donc satisfaite.

**Augmentation (G2) :** Si nous avons deux arbres partiels $T_1$ et $T_2$ avec $|T_1| > |T_2|$, alors $T_1$ couvre plus de sommets que $T_2$. Il doit donc exister une arête dans $T_1$ qui relie un sommet couvert par $T_2$ à un sommet qui ne l'est pas. Ajouter cette arête à $T_2$ produit un nouvel arbre valide.

Ce type de grédoïde est appelé un **grédoïde de branchement**. L'algorithme de Prim est l'algorithme glouton qui fonctionne sur cette structure.

L'introduction des grédoïdes nous permet de construire une hiérarchie conceptuelle de la "gloutonnerie". Au niveau le plus fondamental, nous avons des problèmes qui ne tolèrent aucune approche gloutonne (comme le rendu de monnaie non canonique). Ensuite, nous avons les problèmes qui admettent une solution gloutonne, mais dont l'optimalité n'est garantie que sous des conditions très spécifiques. Au sommet de cette hiérarchie, nous trouvons des structures robustes. Les matroïdes capturent les problèmes d'optimisation où les choix gloutons sont "indépendants de l'ordre" et où l'on peut construire une solution en choisissant les meilleurs éléments disponibles globalement, comme dans l'algorithme de Kruskal. Les grédoïdes généralisent ce concept pour inclure les problèmes où les choix gloutons sont "dépendants de l'ordre" ou contraints par la croissance d'une structure partielle, comme dans l'algorithme de Prim. La théorie des grédoïdes nous enseigne que la nature de la contrainte — "ne pas former de cycle" versus "rester connecté" — définit la structure combinatoire sous-jacente et, par conséquent, le type d'algorithme glouton qui s'y applique.

## Conclusion et Perspectives

Notre exploration des algorithmes gloutons nous a menés bien au-delà de la simple application d'heuristiques. Nous avons commencé par l'intuition, en observant sur des exemples concrets (ordonnancement, rendu de monnaie, planification d'intervalles) que le succès d'une stratégie gloutonne est une affaire de subtilité. L'échec de certaines approches naïves et le succès d'autres, plus réfléchies, nous ont poussés à chercher une explication plus profonde.

Le duel des algorithmes de Kruskal et Prim pour le même problème de l'arbre couvrant de poids minimum a été le catalyseur. Il a révélé que deux philosophies gloutonnes — l'une globale et non contrainte par l'ordre, l'autre locale et contrainte par la croissance — pouvaient toutes deux être optimales. Cette dualité nous a conduits au cœur de la théorie des matroïdes. Nous avons découvert que le matroïde, défini par ses axiomes d'hérédité et d'augmentation, est la structure mathématique exacte de l'indépendance qui garantit l'optimalité de l'algorithme glouton de type Kruskal. Le théorème de Rado-Edmonds a scellé cette connexion, établissant une magnifique équivalence entre une propriété structurelle statique et le succès universel d'un processus algorithmique dynamique.

Cependant, le cas de l'algorithme de Prim montrait que la théorie des matroïdes n'était pas l'histoire complète. En affaiblissant l'axiome d'hérédité pour le remplacer par celui, plus général, d'accessibilité, nous avons introduit les grédoïdes. Cette nouvelle structure nous a permis de modéliser les processus gloutons contraints par la croissance, unifiant ainsi sous un même toit théorique les algorithmes de Kruskal et de Prim.

Le message central de ce cours est que la correction des algorithmes gloutons n'est jamais un accident. Elle est la conséquence directe et prouvable d'une structure mathématique profonde et souvent cachée dans le problème lui-même. Comprendre cette structure, c'est passer du statut d'utilisateur d'algorithmes à celui de concepteur capable d'analyser, de justifier et d'innover.

Le champ ouvert par cette théorie est vaste. Des concepts avancés comme l'intersection de matroïdes permettent de résoudre des problèmes encore plus complexes, comme le problème d'affectation ou la recherche d'arborescences de poids maximal dans les graphes orientés. Le polynôme de Tutte, un invariant puissant des matroïdes, a des connexions profondes avec la physique statistique et la théorie des nœuds. Plus récemment, les concepts de sous-modularité, qui sont une généralisation de la fonction de rang des matroïdes, jouent un rôle de plus en plus central en apprentissage automatique, notamment pour des tâches comme la sélection de caractéristiques ou le résumé de documents. Le voyage dans le monde de la "gloutonnerie" structurée ne fait que commencer.

## Annexe A : Implémentations Complètes en Langage C

Cette annexe fournit le code source complet et commenté en langage C pour les algorithmes et structures de données clés abordés dans ce cours. Chaque module est conçu pour être autonome et compilable.

### A.1. Structure de Données Union-Find (Disjoint Set Union)

Cette structure est essentielle pour l'implémentation efficace de l'algorithme de Kruskal. Elle utilise l'union par taille et la compression de chemin pour une performance quasi-constante en temps amorti.

**Fichier `dsu.h`**

```c
#ifndef DSU_H
#define DSU_H

// Structure pour représenter un sous-ensemble dans DSU
typedef struct {
    int* parent;
    int* size;
    int n;
} DSU;

// Initialise la structure DSU pour n éléments
DSU* create_dsu(int n);

// Trouve le représentant de l'ensemble contenant l'élément i (avec compression de chemin)
int find_set(DSU* dsu, int i);

// Unit les ensembles contenant les éléments i et j (union par taille)
void union_sets(DSU* dsu, int i, int j);

// Libère la mémoire allouée pour la DSU
void free_dsu(DSU* dsu);

#endif // DSU_H
```

**Fichier `dsu.c`**

```c
#include <stdlib.h>
#include "dsu.h"

DSU* create_dsu(int n) {
    DSU* dsu = (DSU*)malloc(sizeof(DSU));
    dsu->n = n;
    dsu->parent = (int*)malloc(n * sizeof(int));
    dsu->size = (int*)malloc(n * sizeof(int));
    for (int i = 0; i < n; i++) {
        dsu->parent[i] = i;
        dsu->size[i] = 1;
    }
    return dsu;
}

int find_set(DSU* dsu, int i) {
    if (dsu->parent[i] == i) {
        return i;
    }
    // Compression de chemin récursive
    return dsu->parent[i] = find_set(dsu, dsu->parent[i]);
}

void union_sets(DSU* dsu, int i, int j) {
    int root_i = find_set(dsu, i);
    int root_j = find_set(dsu, j);
    if (root_i != root_j) {
        // Union par taille
        if (dsu->size[root_i] < dsu->size[root_j]) {
            int temp = root_i;
            root_i = root_j;
            root_j = temp;
        }
        dsu->parent[root_j] = root_i;
        dsu->size[root_i] += dsu->size[root_j];
    }
}

void free_dsu(DSU* dsu) {
    free(dsu->parent);
    free(dsu->size);
    free(dsu);
}
```

### A.2. Algorithme de Kruskal

L'implémentation de Kruskal utilise la structure DSU pour la détection de cycles.

**Fichier `kruskal.h`**

```c
#ifndef KRUSKAL_H
#define KRUSKAL_H

// Structure pour représenter une arête pondérée
typedef struct {
    int u, v, weight;
} Edge;

// Structure pour représenter un graphe
typedef struct {
    int V, E;
    Edge* edges;
} Graph;

// Crée un graphe avec V sommets et E arêtes
Graph* create_graph(int V, int E);

// Exécute l'algorithme de Kruskal et retourne le poids total du MST
// Le MST lui-même est stocké dans le tableau result_edges
int kruskal_mst(Graph* graph, Edge* result_edges);

// Libère la mémoire du graphe
void free_graph(Graph* graph);

#endif // KRUSKAL_H
```

**Fichier `kruskal.c`**

```c
#include <stdio.h>
#include <stdlib.h>
#include "kruskal.h"
#include "dsu.h"

// Fonction de comparaison pour qsort
int compare_edges(const void* a, const void* b) {
    Edge* edge_a = (Edge*)a;
    Edge* edge_b = (Edge*)b;
    return edge_a->weight - edge_b->weight;
}

Graph* create_graph(int V, int E) {
    Graph* graph = (Graph*)malloc(sizeof(Graph));
    graph->V = V;
    graph->E = E;
    graph->edges = (Edge*)malloc(E * sizeof(Edge));
    return graph;
}

void free_graph(Graph* graph) {
    free(graph->edges);
    free(graph);
}

int kruskal_mst(Graph* graph, Edge* result_edges) {
    int V = graph->V;
    int E = graph->E;
    int mst_weight = 0;
    int edge_count = 0;

    // Étape 1: Trier toutes les arêtes par poids croissant
    qsort(graph->edges, E, sizeof(Edge), compare_edges);

    // Étape 2: Initialiser DSU
    DSU* dsu = create_dsu(V);

    // Étape 3: Parcourir les arêtes triées
    for (int i = 0; i < E && edge_count < V - 1; i++) {
        Edge next_edge = graph->edges[i];
        
        int root_u = find_set(dsu, next_edge.u);
        int root_v = find_set(dsu, next_edge.v);

        // Si l'ajout de l'arête ne forme pas de cycle
        if (root_u != root_v) {
            result_edges[edge_count++] = next_edge;
            mst_weight += next_edge.weight;
            union_sets(dsu, root_u, root_v);
        }
    }
    
    printf("Nombre d'arêtes dans le MST: %d\n", edge_count);

    free_dsu(dsu);
    return mst_weight;
}
```

### A.3. Algorithme de Prim

L'implémentation de Prim est plus complexe et nécessite une structure de file de priorité (min-heap), non fournie ici pour la brièveté, mais le pseudo-code est clair. Voici une implémentation plus simple utilisant une recherche linéaire pour trouver le sommet de clé minimale, ce qui donne une complexité de $O(V^2)$. Une version avec min-heap atteindrait $O(E \log V)$.

**Fichier `prim.h`**

```c
#ifndef PRIM_H
#define PRIM_H

#define INF 999999

// Exécute l'algorithme de Prim sur un graphe représenté par une matrice d'adjacence
// graph[V][V] est la matrice, V est le nombre de sommets
void prim_mst_matrix(int V, int** graph);

#endif // PRIM_H
```

**Fichier `prim.c`**

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>
#include "prim.h"

// Trouve le sommet avec la valeur de clé minimale, parmi les sommets non encore inclus dans le MST
int min_key(int V, int key[], bool mst_set[]) {
    int min = INF, min_index = -1;
    for (int v = 0; v < V; v++) {
        if (mst_set[v] == false && key[v] < min) {
            min = key[v];
            min_index = v;
        }
    }
    return min_index;
}

// Affiche le MST construit
void print_mst(int V, int parent[], int** graph) {
    printf("Arête \tPoids\n");
    int total_weight = 0;
    for (int i = 1; i < V; i++) {
        printf("%d - %d \t%d \n", parent[i], i, graph[i][parent[i]]);
        total_weight += graph[i][parent[i]];
    }
    printf("Poids total du MST: %d\n", total_weight);
}

void prim_mst_matrix(int V, int** graph) {
    int* parent = (int*)malloc(V * sizeof(int)); // Stocke le MST construit
    int* key = (int*)malloc(V * sizeof(int));     // Valeurs clés pour choisir l'arête de poids min
    bool* mst_set = (bool*)malloc(V * sizeof(bool)); // Pour représenter les sommets inclus dans le MST

    // Initialisation
    for (int i = 0; i < V; i++) {
        key[i] = INF;
        mst_set[i] = false;
    }

    // Le premier sommet est toujours la racine du MST
    key[0] = 0;
    parent[0] = -1; // Pas de parent pour la racine

    for (int count = 0; count < V - 1; count++) {
        // Choisir le sommet de clé minimale non encore dans le MST
        int u = min_key(V, key, mst_set);
        if (u == -1) break; // Graphe non connexe

        // Ajouter le sommet choisi à l'ensemble MST
        mst_set[u] = true;

        // Mettre à jour les valeurs de clé des sommets adjacents
        for (int v = 0; v < V; v++) {
            if (graph[u][v] && mst_set[v] == false && graph[u][v] < key[v]) {
                parent[v] = u;
                key[v] = graph[u][v];
            }
        }
    }

    print_mst(V, parent, graph);

    free(parent);
    free(key);
    free(mst_set);
}
```

### A.4. Algorithme d'Ordonnancement de Tâches

Cette implémentation suit la stratégie gloutonne correcte : trier par profit et assigner au créneau le plus tardif possible.

**Fichier `job_scheduling.h`**

```c
#ifndef JOB_SCHEDULING_H
#define JOB_SCHEDULING_H

typedef struct {
    char id;
    int deadline;
    int profit;
} Job;

// Fonction principale pour l'ordonnancement
void schedule_jobs(Job jobs[], int n);

#endif // JOB_SCHEDULING_H
```

**Fichier `job_scheduling.c`**

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>
#include "job_scheduling.h"

// Comparateur pour qsort, trie par profit décroissant
int compare_jobs(const void* a, const void* b) {
    return ((Job*)b)->profit - ((Job*)a)->profit;
}

void schedule_jobs(Job jobs[], int n) {
    // Trier les tâches par profit
    qsort(jobs, n, sizeof(Job), compare_jobs);

    int max_deadline = 0;
    for (int i = 0; i < n; i++) {
        if (jobs[i].deadline > max_deadline) {
            max_deadline = jobs[i].deadline;
        }
    }

    // Tableau pour les créneaux, -1 indique un créneau libre
    int* slots = (int*)malloc(max_deadline * sizeof(int));
    for (int i = 0; i < max_deadline; i++) {
        slots[i] = -1;
    }

    int total_profit = 0;
    int job_count = 0;

    for (int i = 0; i < n; i++) {
        // Trouver un créneau pour cette tâche
        for (int j = jobs[i].deadline - 1; j >= 0; j--) {
            if (slots[j] == -1) {
                slots[j] = i; // Assigner la tâche au créneau
                total_profit += jobs[i].profit;
                job_count++;
                break;
            }
        }
    }

    printf("Séquence de tâches pour un profit maximal:\n");
    for (int i = 0; i < max_deadline; i++) {
        if (slots[i] != -1) {
            printf("Créneau %d: Tâche %c (Profit: %d, Deadline: %d)\n", 
                   i + 1, jobs[slots[i]].id, jobs[slots[i]].profit, jobs[slots[i]].deadline);
        }
    }
    printf("Profit total: %d\n", total_profit);
    printf("Nombre total de tâches effectuées: %d\n", job_count);

    free(slots);
}
```

### A.5. Algorithme de Planification d'Intervalles

Implémentation de la stratégie gloutonne optimale : trier par heure de fin.

**Fichier `interval_scheduling.h`**

```c
#ifndef INTERVAL_SCHEDULING_H
#define INTERVAL_SCHEDULING_H

typedef struct {
    int start;
    int finish;
    int id;
} Interval;

// Fonction principale pour la planification
void schedule_intervals(Interval intervals[], int n);

#endif // INTERVAL_SCHEDULING_H
```

**Fichier `interval_scheduling.c`**

```c
#include <stdio.h>
#include <stdlib.h>
#include "interval_scheduling.h"

// Comparateur pour qsort, trie par heure de fin croissante
int compare_intervals(const void* a, const void* b) {
    return ((Interval*)a)->finish - ((Interval*)b)->finish;
}

void schedule_intervals(Interval intervals[], int n) {
    // Trier les intervalles par heure de fin
    qsort(intervals, n, sizeof(Interval), compare_intervals);

    printf("Intervalles sélectionnés (maximum compatibles):\n");

    // Le premier intervalle est toujours sélectionné
    printf("Intervalle %d: [%d, %d]\n", intervals[0].id, intervals[0].start, intervals[0].finish);
    int last_finish_time = intervals[0].finish;
    int count = 1;

    // Parcourir les intervalles restants
    for (int i = 1; i < n; i++) {
        // Si l'intervalle actuel commence après ou en même temps que la fin du précédent
        if (intervals[i].start >= last_finish_time) {
            printf("Intervalle %d: [%d, %d]\n", intervals[i].id, intervals[i].start, intervals[i].finish);
            last_finish_time = intervals[i].finish;
            count++;
        }
    }
    printf("Nombre total d'intervalles sélectionnés: %d\n", count);
}
```

## Références Académiques

1. Cormen, T. H., Leiserson, C. E., Rivest, R. L., & Stein, C. (2009). *Introduction to algorithms*. MIT press.
2. Edmonds, J. (1971). Matroids and the greedy algorithm. *Mathematical programming*, 1(1), 127-136.
3. Korte, B., Lovász, L., & Schrader, R. (1991). *Greedoids*. Springer-Verlag.
4. Kruskal, J. B. (1956). On the shortest spanning subtree of a graph and the traveling salesman problem. *Proceedings of the American mathematical society*, 7(1), 48-50.
5. Prim, R. C. (1957). Shortest connection networks and some generalizations. *The Bell System Technical Journal*, 36(6), 1389-1401.
6. Whitney, H. (1935). On the abstract properties of linear dependence. *American journal of mathematics*, 57(3), 509-533.
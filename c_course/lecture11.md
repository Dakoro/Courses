# Au-delà des Tris Classiques : Algorithmes, Limites Théoriques et Implémentations Modernes

## Introduction : Le Paysage des Algorithmes de Tri et de Recherche

Chers étudiants, bonjour et bienvenue à ce séminaire. Notre objectif aujourd'hui est d'explorer des territoires moins fréquentés de l'algorithmique, au-delà des méthodes de tri et de recherche que vous connaissez déjà. Nous nous aventurerons dans ce que l'on pourrait qualifier de "hacks" ou d'approches non conventionnelles, des techniques qui, sous leur apparence astucieuse, révèlent des principes informatiques d'une grande profondeur.

Notre parcours s'articulera autour d'une dichotomie fondamentale qui structure l'univers du tri. D'un côté, nous avons les algorithmes basés sur la comparaison, tels que le tri rapide (Quicksort) ou le tri fusion (Merge Sort). Ces algorithmes déterminent l'ordre des éléments en les comparant deux à deux, à la manière d'une balance à plateaux qui pèse deux objets pour savoir lequel est le plus lourd. De l'autre côté se trouvent les algorithmes non comparatifs, qui, eux, exploitent la structure intrinsèque et la valeur même des données pour les ordonner.

L'idée directrice de ce cours est de vous montrer comment les propriétés spécifiques des données — qu'il s'agisse de nombres entiers, de leur distribution, ou de structures préexistantes — peuvent être mises à profit pour concevoir des algorithmes dont les performances semblent défier, voire violer, les limites théoriques que nous pensions infranchissables. Ce voyage intellectuel nous mènera d'un problème concret à l'établissement d'une barrière théorique, pour ensuite découvrir comment contourner cette barrière en changeant subtilement les règles du jeu, c'est-à-dire le modèle de calcul sous-jacent. Nous transformerons ainsi ce qui peut sembler être de la "magie" algorithmique en une science rigoureuse et bien comprise.

## Partie 1 : Le Problème de l'Élément Majoritaire - Une Étude de Cas Algorithmique

Pour commencer notre exploration, nous allons nous pencher sur un problème classique qui sert de parfaite étude de cas pour illustrer l'évolution de la pensée algorithmique : le problème de l'élément majoritaire.

### 1.1 Définition du Problème

La définition formelle du problème est la suivante : étant donné un tableau $A$ de $n$ éléments, il s'agit de trouver un élément $m$ qui apparaît strictement plus de $\frac{n}{2}$ fois. Si un tel élément n'existe pas, l'algorithme doit le signaler.

Considérons un exemple simple. Dans le tableau $A = [2, 2, 1, 2, 3]$, la taille est $n=5$. La moitié de la taille est $2.5$. Nous cherchons donc un élément qui apparaît au moins $3$ fois. L'élément $2$ apparaît $3$ fois, il est donc l'élément majoritaire de ce tableau. En revanche, dans le tableau $B = [1, 2, 3, 4, 5]$, aucun élément n'apparaît plus d'une fois ; il n'y a donc pas d'élément majoritaire.

### 1.2 L'Approche Naïve : La Force Brute en $O(N^2)$

La première solution qui vient à l'esprit, et la plus simple à concevoir, est une approche par force brute. Le principe est direct : pour chaque élément du tableau, nous allons parcourir une seconde fois l'intégralité du tableau pour compter le nombre de fois où cet élément apparaît. Si, pour un élément donné, ce compte dépasse $\frac{n}{2}$, nous l'avons trouvé.

Bien que correct, cet algorithme est loin d'être efficace. Pour chacun des $N$ éléments, nous effectuons un parcours complet du tableau, ce qui représente $N$ opérations. L'opération est donc répétée $N$ fois, menant à une complexité temporelle en $O(N^2)$. Pour des tableaux de grande taille, cette complexité quadratique devient rapidement prohibitive. Par exemple, pour un million d'éléments, cela représenterait de l'ordre de $10^{12}$ opérations, ce qui est inacceptable en pratique.

### 1.3 Diviser pour Régner : Une Solution en $O(N \log N)$

Pour améliorer cette performance, nous pouvons nous tourner vers une stratégie algorithmique classique : "Diviser pour Régner". Cette approche est au cœur d'algorithmes efficaces comme le tri fusion. L'idée est de décomposer le problème en sous-problèmes plus petits, de les résoudre récursivement, puis de combiner leurs solutions.

**Diviser :** On coupe le tableau $A$ en deux moitiés, une gauche et une droite.

**Régner :** On appelle récursivement l'algorithme sur chacune des deux moitiés pour trouver leur élément majoritaire respectif (s'il en existe un).

**Combiner :** C'est ici que réside l'observation cruciale. Si un élément $m$ est majoritaire dans le tableau complet, il doit nécessairement être majoritaire dans au moins l'une des deux moitiés. Pourquoi ? Supposons que $m$ ne soit majoritaire ni à gauche, ni à droite. Cela signifie qu'à gauche, il apparaît au plus $\frac{n/2}{2} = \frac{n}{4}$ fois, et à droite, également au plus $\frac{n}{4}$ fois. Au total, il apparaîtrait au plus $\frac{n}{4} + \frac{n}{4} = \frac{n}{2}$ fois, ce qui contredit le fait qu'il soit majoritaire dans le tableau entier.

Cette propriété nous donne une méthode pour la phase de combinaison. Après les appels récursifs, nous obtenons au plus deux candidats pour l'élément majoritaire global (le majoritaire de gauche et celui de droite). Il nous suffit alors de faire un unique parcours du tableau complet pour compter les occurrences de ces deux candidats et vérifier si l'un d'eux est bien le véritable élément majoritaire.

L'analyse de la complexité de cet algorithme se fait via une équation de récurrence. Si $T(N)$ est le temps pour un tableau de taille $N$, nous avons :

$$T(N) = 2T(N/2) + O(N)$$

Ici, $2T(N/2)$ représente les deux appels récursifs sur des tableaux de taille moitié, et $O(N)$ représente le coût de la phase de combinaison (le parcours pour vérifier les deux candidats). D'après le Master Theorem (avec $a=2$, $b=2$, et donc $\log_b(a)=1$), la solution de cette récurrence est $T(N) = O(N \log N)$.

C'est une amélioration considérable par rapport à l'approche naïve. Pour notre tableau d'un million d'éléments, $\log_2(10^6)$ est d'environ $20$. La complexité serait donc de l'ordre de $20 \times 10^6$ opérations, bien plus réalisable que $10^{12}$. Mais pouvons-nous faire encore mieux ?

### 1.4 L'Algorithme de Vote de Boyer-Moore : L'Élégance Linéaire en $O(N)$

La réponse est oui. Il existe une solution d'une remarquable élégance qui résout le problème en temps linéaire et avec un espace de stockage constant. Il s'agit de l'algorithme de vote de Boyer-Moore, publié en 1981 par Robert S. Boyer et J Strother Moore. Souvent présenté comme une astuce algorithmique, il s'agit en réalité d'une contribution formelle et d'un exemple prototypique de la classe des algorithmes de streaming, où les données sont traitées séquentiellement en un seul passage avec une mémoire très limitée.

#### Intuition de l'algorithme : le mécanisme de vote

L'intuition derrière l'algorithme est simple et peut être vue comme un processus d'élection par annulation. On maintient deux variables : un candidat potentiel et un compteur de votes pour ce candidat.

1. On initialise le compteur à 0.
2. On parcourt le tableau élément par élément.
3. À chaque élément $x$ :
   - Si le compteur est à 0, cela signifie qu'il n'y a pas de candidat en lice. L'élément $x$ devient alors le nouveau candidat, et son compteur passe à 1.
   - Si le compteur n'est pas à 0, on compare $x$ au candidat actuel.
     - Si $x$ est identique au candidat, on incrémente le compteur. C'est un vote de soutien.
     - Si $x$ est différent du candidat, on décrémente le compteur. C'est un vote d'opposition qui annule un vote de soutien.

L'idée fondamentale est que si un élément est réellement majoritaire, il détient plus de la moitié des "voix". Chaque fois qu'un autre élément est rencontré, il peut annuler un vote pour le majoritaire, mais comme il y a plus de votes pour le majoritaire que pour tous les autres éléments combinés, ces derniers ne pourront jamais annuler tous les votes du majoritaire. À la fin du parcours, le candidat qui reste est donc le seul élément qui a une chance d'être l'élément majoritaire.

#### Pseudocode

Voici le pseudocode de la première phase de l'algorithme :

```
fonction BoyerMooreVote(tableau A de taille n):
  candidat = null
  compteur = 0

  pour i de 0 à n-1:
    si compteur == 0:
      candidat = A[i]
      compteur = 1
    sinon si A[i] == candidat:
      compteur = compteur + 1
    sinon:
      compteur = compteur - 1

  retourner candidat
```

#### La phase de vérification : une étape cruciale

L'algorithme de Boyer-Moore, dans sa première passe, ne garantit qu'une seule chose : si un élément majoritaire existe, il sera le candidat retourné à la fin. Cependant, il ne garantit pas que le candidat retourné est effectivement majoritaire. Par exemple, pour le tableau `[1, 2, 3]`, l'algorithme retournera 3, qui n'est pas majoritaire.

Il est donc indispensable d'effectuer une seconde passe sur le tableau pour vérifier la validité du candidat. Cette seconde passe consiste simplement à compter les occurrences réelles du candidat retourné et à vérifier si ce compte est strictement supérieur à $\frac{n}{2}$.

#### Implémentation en C

Voici une implémentation complète en C, incluant la phase de vérification.

```c
#include <stdio.h>
#include <stdbool.h>

/**
 * @brief Trouve l'élément majoritaire dans un tableau en utilisant l'algorithme de vote de Boyer-Moore.
 * 
 * L'algorithme se déroule en deux phases :
 * 1. Trouver un candidat : On parcourt le tableau pour trouver un candidat potentiel.
 *    On maintient un candidat et un compteur. Si le compteur est à 0, l'élément courant devient le candidat.
 *    Si l'élément courant est le candidat, on incrémente le compteur. Sinon, on le décrémente.
 *    À la fin de cette passe, si un élément majoritaire existe, il sera le candidat.
 * 2. Vérifier le candidat : On parcourt à nouveau le tableau pour compter les occurrences réelles du candidat.
 *    Si son compte est supérieur à n/2, c'est l'élément majoritaire. Sinon, il n'y en a pas.
 *
 * @param arr Le tableau d'entiers.
 * @param n La taille du tableau.
 * @param majority_element Un pointeur pour stocker l'élément majoritaire trouvé.
 * @return true si un élément majoritaire est trouvé, false sinon.
 */
bool find_majority_element(int arr[], int n, int* majority_element) {
    if (n <= 0) {
        return false;
    }

    // --- Phase 1: Trouver un candidat ---
    int candidate = arr[0];
    int count = 1;

    for (int i = 1; i < n; i++) {
        if (count == 0) {
            candidate = arr[i];
            count = 1;
        } else if (arr[i] == candidate) {
            count++;
        } else {
            count--;
        }
    }

    // --- Phase 2: Vérifier si le candidat est bien majoritaire ---
    count = 0;
    for (int i = 0; i < n; i++) {
        if (arr[i] == candidate) {
            count++;
        }
    }

    if (count > n / 2) {
        *majority_element = candidate;
        return true;
    }

    return false;
}

int main() {
    int arr1[] = {3, 9, 2, 2, 2};
    int n1 = sizeof(arr1) / sizeof(arr1[0]);
    int majority1;

    if (find_majority_element(arr1, n1, &majority1)) {
        printf("Le tableau 1 a un élément majoritaire : %d\n", majority1);
    } else {
        printf("Le tableau 1 n'a pas d'élément majoritaire.\n");
    }

    int arr2[] = {1, 2, 3, 4, 5};
    int n2 = sizeof(arr2) / sizeof(arr2[0]);
    int majority2;

    if (find_majority_element(arr2, n2, &majority2)) {
        printf("Le tableau 2 a un élément majoritaire : %d\n", majority2);
    } else {
        printf("Le tableau 2 n'a pas d'élément majoritaire.\n");
    }
    
    return 0;
}
```

#### Complexité et conclusion

L'algorithme de Boyer-Moore effectue deux passes linéaires sur le tableau et n'utilise qu'un nombre constant de variables supplémentaires (candidate, count). Sa complexité temporelle est donc de $O(N)$ et sa complexité spatiale de $O(1)$.

Ce problème de l'élément majoritaire est une illustration parfaite de la démarche algorithmique. Nous sommes passés d'une solution évidente mais lente ($O(N^2)$), à une solution plus intelligente mais sous-optimale ($O(N \log N)$), pour finalement aboutir à une solution d'une efficacité et d'une élégance remarquables ($O(N)$ temps, $O(1)$ espace). Cette dernière solution montre comment une compréhension profonde de la structure du problème — ici, la définition même de la majorité — permet de concevoir des algorithmes qui semblent presque magiques, mais qui reposent sur une logique implacable. C'est une porte d'entrée vers des paradigmes de conception plus avancés, comme les algorithmes de streaming, qui sont essentiels pour le traitement de données massives.

## Partie 2 : Les Fondations Théoriques - La Limite Infranchissable des Tris par Comparaison

Après avoir vu comment optimiser un problème spécifique, prenons de la hauteur et posons-nous une question fondamentale sur le tri en général. Les algorithmes comme le tri fusion ou le tri en tas (Heapsort) atteignent une complexité de $O(N \log N)$. Est-ce le mieux que nous puissions faire ? Ou existe-t-il un algorithme de tri universel encore plus rapide qui attend d'être découvert ?

Pour répondre à cette question, nous devons prouver une borne inférieure (lower bound), c'est-à-dire une limite de performance en dessous de laquelle aucun algorithme d'une certaine classe ne peut descendre. Nous allons démontrer que pour la classe des tris basés sur la comparaison, la réponse est non : il est impossible de faire mieux que $\Omega(N \log N)$.

### 2.1 Le Modèle de l'Arbre de Décision

Pour prouver une telle affirmation, nous avons besoin d'un modèle formel qui représente tous les algorithmes de tri par comparaison possibles, même ceux qui n'ont pas encore été inventés. Ce modèle est l'arbre de décision.

Imaginons un algorithme de tri. Son exécution est une séquence de comparaisons. Chaque comparaison guide la suite des opérations. Nous pouvons représenter cette logique sous la forme d'un arbre binaire :

- Chaque nœud interne de l'arbre représente une comparaison entre deux éléments du tableau, par exemple $A[i] \leq A[j]$.
- Les deux branches qui partent d'un nœud interne correspondent aux deux résultats possibles de la comparaison (vrai ou faux).
- Chaque feuille de l'arbre représente un état final où l'ordre de tous les éléments est connu, c'est-à-dire une permutation spécifique du tableau d'entrée qui le rend trié.

L'exécution de l'algorithme sur une entrée donnée correspond au suivi d'un unique chemin, de la racine jusqu'à une feuille. Le nombre de comparaisons effectuées dans le pire des cas est simplement la hauteur de cet arbre, c'est-à-dire la longueur du plus long chemin de la racine à une feuille.

### 2.2 La Barrière Combinatoire des $N!$ Permutations

Considérons un tableau de $n$ éléments tous distincts. Combien y a-t-il de manières de les ordonner ? La réponse est donnée par la factorielle de $n$, soit $n!$. Par exemple, pour 3 éléments $\{a, b, c\}$, il y a $3! = 6$ permutations possibles : $(a,b,c)$, $(a,c,b)$, $(b,a,c)$, $(b,c,a)$, $(c,a,b)$, $(c,b,a)$.

Un algorithme de tri par comparaison doit être capable de fonctionner correctement quelle que soit la permutation initiale des données. Cela signifie qu'il doit être capable de produire n'importe laquelle des $n!$ permutations triées comme résultat. Par conséquent, l'arbre de décision qui modélise cet algorithme doit avoir au moins $n!$ feuilles, une pour chaque résultat possible.

### 2.3 Preuve Formelle de la Borne Inférieure $\Omega(n \log n)$

Nous avons maintenant tous les éléments pour notre preuve.

1. Soit un algorithme de tri par comparaison pour $n$ éléments. Il peut être représenté par un arbre de décision.
2. Cet arbre doit avoir au moins $L = n!$ feuilles pour être correct.
3. Un arbre binaire de hauteur $h$ peut avoir au maximum $2^h$ feuilles. C'est une propriété fondamentale des arbres binaires.

En combinant ces deux faits, nous obtenons l'inégalité suivante :

$$L \leq 2^h \Rightarrow n! \leq 2^h$$

Pour trouver une borne inférieure sur la hauteur $h$ (le nombre de comparaisons dans le pire des cas), nous prenons le logarithme en base 2 des deux côtés :

$$h \geq \log_2(n!)$$

La complexité dans le pire des cas de n'importe quel algorithme de tri par comparaison est donc en $\Omega(\log(n!))$. Pour rendre cette expression plus familière, nous utilisons l'approximation de Stirling :

$$n! \approx \sqrt{2\pi n} \left(\frac{n}{e}\right)^n$$

Prenons le logarithme de cette approximation :

$$\log_2(n!) \approx \log_2\left(\sqrt{2\pi n} \left(\frac{n}{e}\right)^n\right)$$

$$\log_2(n!) \approx \log_2(\sqrt{2\pi n}) + \log_2\left(\left(\frac{n}{e}\right)^n\right)$$

$$\log_2(n!) \approx \frac{1}{2}\log_2(2\pi n) + n \log_2\left(\frac{n}{e}\right)$$

$$\log_2(n!) \approx \frac{1}{2}\log_2(2\pi n) + n(\log_2 n - \log_2 e)$$

Lorsque $n$ devient grand, le terme dominant est $n \log_2 n$. Par conséquent, nous pouvons conclure que :

$$\log_2(n!) = \Omega(n \log n)$$

**Conclusion :** Tout algorithme de tri basé exclusivement sur des comparaisons entre éléments a une complexité temporelle dans le pire des cas qui est, au minimum, de l'ordre de $\Omega(n \log n)$. Des algorithmes comme le tri fusion et le tri en tas sont donc dits asymptotiquement optimaux pour cette classe d'algorithmes.

Cette preuve est l'un des piliers de l'informatique théorique. Elle établit une limite fondamentale. Cependant, il est crucial de noter la prémisse sur laquelle elle repose : "basé sur les comparaisons". Cette hypothèse est la clé de voûte de la preuve, mais aussi sa principale limitation. La preuve ne dit pas que le problème du tri lui-même est intrinsèquement en $\Omega(n \log n)$. Elle dit que le tri, lorsqu'il est résolu uniquement par des comparaisons, l'est. Que se passe-t-il si nous changeons les règles ? Si nous nous autorisons à regarder la valeur des éléments, et pas seulement leur ordre relatif ? C'est la porte que nous allons maintenant ouvrir.

## Partie 3 : Dépasser les Limites - Les Tris en Temps Linéaire

La borne inférieure de $\Omega(n \log n)$ semble être un mur infranchissable. Pourtant, comme le suggère notre exploration, il existe une "triche". Cette triche consiste à sortir du modèle de calcul par comparaison. Si nous avons des informations supplémentaires sur nos données, par exemple, si nous savons qu'il s'agit de nombres entiers dans une plage de valeurs connue, nous pouvons utiliser des algorithmes qui ne comparent pas les éléments entre eux, mais qui exploitent directement leurs valeurs pour les classer. Ces algorithmes, sous certaines conditions, peuvent atteindre une complexité linéaire, en $O(N)$.

### 3.1 Le Tri par Dénombrement (Counting Sort)

Le tri par dénombrement est l'exemple le plus simple et le plus direct d'un tri non comparatif. Son principe est d'utiliser les valeurs des éléments non pas pour des comparaisons, mais comme des indices dans un tableau auxiliaire.

#### Mécanisme

Supposons que nous devons trier un tableau d'entiers dont nous savons que les valeurs sont comprises entre 0 et $k$.

1. **Trouver la valeur maximale :** On parcourt le tableau d'entrée pour déterminer la valeur maximale $k$ (si elle n'est pas déjà connue).

2. **Créer le tableau de dénombrement :** On crée un tableau auxiliaire, que nous appellerons `comptes`, de taille $k+1$, et on l'initialise entièrement à zéro. Chaque indice de ce tableau, de 0 à $k$, correspond à une valeur possible dans le tableau d'entrée.

3. **Dénombrer les occurrences :** On parcourt le tableau d'entrée $A$. Pour chaque élément $A[i]$, on incrémente le compteur correspondant à sa valeur : `comptes[A[i]]++`. À la fin de ce parcours, `comptes[j]` contiendra le nombre de fois que la valeur $j$ apparaît dans $A$.

4. **Reconstruire le tableau trié :** On parcourt le tableau `comptes` de l'indice 0 à $k$. Pour chaque indice $j$, on ajoute la valeur $j$ au tableau de sortie `comptes[j]` fois.

#### Pseudocode et Implémentation en C

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

// Fonction pour trouver la valeur maximale dans un tableau
int find_max(int arr[], int n) {
    int max = arr[0];
    for (int i = 1; i < n; i++) {
        if (arr[i] > max) {
            max = arr[i];
        }
    }
    return max;
}

/**
 * @brief Trie un tableau d'entiers positifs en utilisant le tri par dénombrement.
 *
 * Cet algorithme est efficace lorsque la plage de valeurs (k) n'est pas
 * significativement plus grande que le nombre d'éléments (n).
 *
 * @param arr Le tableau à trier.
 * @param n La taille du tableau.
 */
void counting_sort(int arr[], int n) {
    if (n <= 1) return;

    // 1. Trouver la valeur maximale k pour déterminer la taille du tableau de comptes.
    int k = find_max(arr, n);

    // 2. Créer le tableau de comptes de taille k+1 et l'initialiser à 0.
    // Utilise calloc pour allouer et initialiser à zéro en une seule étape.
    int* counts = (int*)calloc(k + 1, sizeof(int));
    if (counts == NULL) {
        fprintf(stderr, "Erreur d'allocation mémoire\n");
        return;
    }

    // 3. Dénombrer les occurrences de chaque élément.
    for (int i = 0; i < n; i++) {
        counts[arr[i]]++;
    }

    // 4. Reconstruire le tableau d'origine (arr) en le parcourant et en
    //    plaçant les éléments dans l'ordre.
    int output_index = 0;
    for (int i = 0; i <= k; i++) {
        // Pour chaque valeur i, l'ajouter 'counts[i]' fois dans le tableau de sortie.
        for (int j = 0; j < counts[i]; j++) {
            arr[output_index++] = i;
        }
    }

    // Libérer la mémoire allouée pour le tableau de comptes.
    free(counts);
}

void print_array(int arr[], int n) {
    for (int i = 0; i < n; i++) {
        printf("%d ", arr[i]);
    }
    printf("\n");
}

int main() {
    int arr[] = {4, 2, 2, 8, 3, 3, 1};
    int n = sizeof(arr) / sizeof(arr[0]);
    
    printf("Tableau original : ");
    print_array(arr, n);

    counting_sort(arr, n);

    printf("Tableau trié     : ");
    print_array(arr, n);

    return 0;
}
```

#### Complexité et Limites

La complexité du tri par dénombrement est la somme des coûts de ses étapes : $O(N)$ pour trouver le maximum, $O(k)$ pour initialiser le tableau de comptes, $O(N)$ pour compter les occurrences, et $O(N+k)$ pour reconstruire le tableau. La complexité totale est donc $O(N+k)$.

- Si $k$ est de l'ordre de $N$ (par exemple, trier les notes de 1 million d'étudiants sur 20), l'algorithme est en $O(N)$, donc linéaire et extrêmement efficace.
- Cependant, si $k$ est beaucoup plus grand que $N$ (par exemple, trier 1000 numéros de sécurité sociale, qui sont de très grands nombres), l'algorithme devient très inefficace. Nous devrions allouer un tableau `comptes` gigantesque, qui serait presque entièrement vide, gaspillant à la fois la mémoire et le temps de parcours.

### 3.2 Le Tri par Base (Radix Sort)

Le tri par base est une autre méthode de tri non comparatif, plus astucieuse, qui résout le problème de la grande plage de valeurs du tri par dénombrement. Plutôt que de trier sur la valeur totale des nombres, il les trie chiffre par chiffre. L'approche la plus courante est le tri LSD (Least Significant Digit), qui commence par le chiffre le moins significatif (celui des unités).

#### Mécanisme LSD

1. On détermine le nombre de chiffres $d$ du plus grand nombre dans le tableau. Ce sera le nombre de passes que nous devrons effectuer.

2. Pour chaque position de chiffre $i$ (en commençant par les unités, puis les dizaines, les centaines, etc.) :
   - On utilise un algorithme de tri stable pour trier l'ensemble du tableau en se basant uniquement sur la valeur du chiffre à la position $i$.

L'utilisation d'un tri stable est absolument fondamentale ici. C'est la "colle" qui fait que l'algorithme fonctionne.

### 3.3 Le Concept Clé de la Stabilité Algorithmique

#### Définition

Un algorithme de tri est dit **stable** s'il préserve l'ordre relatif des éléments ayant des clés de tri égales. Par exemple, si nous trions une liste d'étudiants par note, et que deux étudiants, Alice et Bob, ont tous les deux 15/20, un tri stable garantira que si Alice apparaissait avant Bob dans la liste originale, elle apparaîtra également avant lui dans la liste triée.

#### Importance cruciale pour le Tri par Base

Pourquoi la stabilité est-elle si importante pour le tri par base ? Prenons un exemple. Soit le tableau `[170, 45, 75]`.

**Passe 1 (chiffre des unités) :** On trie selon le dernier chiffre. `17**0**`, `4**5**`, `7**5**`. Le résultat est `[170, 45, 75]`. Comme 45 et 75 ont le même chiffre des unités (5), un tri stable garantit que 45 reste avant 75, car c'était leur ordre initial.

**Passe 2 (chiffre des dizaines) :** On trie le résultat précédent selon le chiffre des dizaines. `1**7**0`, `**4**5`, `**7**5`. Le résultat est `[45, 170, 75]`. Remarquez que 170 et 75 ont le même chiffre des dizaines (7). Grâce à la stabilité du tri précédent, l'ordre 170 avant 75 est préservé.

**Passe 3 (chiffre des centaines) :** On trie selon le chiffre des centaines. `**0**45`, `**1**70`, `**0**75`. Le résultat final est `[45, 75, 170]`.

Si le tri de la passe 2 n'avait pas été stable, l'ordre final aurait pu être incorrect. La stabilité garantit que l'ordre établi par les chiffres moins significatifs est conservé lorsque les chiffres plus significatifs sont égaux. C'est ce qui permet à l'ordre de se construire correctement, passe après passe.

#### Stabilité des autres algorithmes de tri

- **Tri par sélection :** Généralement instable. Il peut échanger des éléments très éloignés, perturbant l'ordre relatif des éléments égaux.
- **Tri par insertion :** Stable si implémenté correctement (un nouvel élément est inséré après les éléments égaux déjà en place).
- **Tri fusion :** Peut être implémenté de manière stable.
- **Tri rapide :** Généralement instable en raison de l'algorithme de partitionnement qui peut réordonner des éléments égaux.

#### Implémentation du Tri par Base en C

Le tri par base n'est pas un algorithme monolithique ; il est un méta-algorithme qui s'appuie sur une sous-routine de tri stable. Le tri par dénombrement est un partenaire naturel car il est stable et efficace pour trier sur une plage restreinte de valeurs (les chiffres de 0 à 9).

La version du tri par dénombrement utilisée ici doit être adaptée pour être stable. La version simple présentée précédemment ne l'est pas. Pour la rendre stable, nous devons utiliser une étape supplémentaire, souvent appelée "balayage inclusif" ou calcul de somme préfixée, et un tableau de sortie temporaire.

1. **Compter :** Comme avant, on compte les occurrences de chaque chiffre.

2. **Somme préfixée ("Inclusive Scan") :** On modifie le tableau `comptes` pour que `comptes[i]` contienne la position de fin dans le tableau de sortie pour les éléments ayant le chiffre $i$. Pour ce faire, chaque `comptes[i]` devient la somme de lui-même et de tous les comptes précédents. Par exemple, si les comptes pour les chiffres 0, 1, 2 sont `[1, 2, 3]`, le tableau de somme préfixée devient `[1, 3, 6]`. Cela signifie que les éléments avec le chiffre 0 se termineront à l'indice 0, ceux avec le chiffre 1 à l'indice 2, et ceux avec le chiffre 2 à l'indice 5.

3. **Construire la sortie :** On parcourt le tableau d'entrée à l'envers (c'est ce qui garantit la stabilité). Pour chaque élément, on le place dans le tableau de sortie à la position indiquée par le tableau de somme préfixée, puis on décrémente cette position.

Voici l'implémentation complète :

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

// Fonction utilitaire pour trouver la valeur maximale dans un tableau
int find_max(int arr[], int n) {
    int max = arr[0];
    for (int i = 1; i < n; i++) {
        if (arr[i] > max) {
            max = arr[i];
        }
    }
    return max;
}

/**
 * @brief Sous-routine de tri stable (tri par dénombrement) pour le tri par base.
 *
 * Trie le tableau 'arr' en fonction du chiffre spécifié par 'exp' (1 pour les unités, 10 pour les dizaines, etc.).
 * @param arr Le tableau à trier.
 * @param n La taille du tableau.
 * @param exp L'exposant (1, 10, 100...) qui détermine le chiffre sur lequel trier.
 */
void counting_sort_for_radix(int arr[], int n, int exp) {
    int* output = (int*)malloc(n * sizeof(int)); // Tableau de sortie temporaire
    int count[10] = {0}; // Tableau pour les chiffres de 0 à 9

    if (output == NULL) {
        fprintf(stderr, "Erreur d'allocation mémoire\n");
        return;
    }

    // 1. Compter les occurrences des chiffres
    for (int i = 0; i < n; i++) {
        count[(arr[i] / exp) % 10]++;
    }

    // 2. Calculer la somme préfixée (inclusive scan) pour obtenir les positions
    for (int i = 1; i < 10; i++) {
        count[i] += count[i - 1];
    }

    // 3. Construire le tableau de sortie. Parcourir à l'envers pour la stabilité.
    for (int i = n - 1; i >= 0; i--) {
        int digit = (arr[i] / exp) % 10;
        output[count[digit] - 1] = arr[i];
        count[digit]--;
    }

    // 4. Copier le tableau de sortie trié dans le tableau d'origine
    for (int i = 0; i < n; i++) {
        arr[i] = output[i];
    }

    free(output);
}

/**
 * @brief Trie un tableau d'entiers en utilisant le tri par base (Radix Sort LSD).
 *
 * @param arr Le tableau à trier.
 * @param n La taille du tableau.
 */
void radix_sort(int arr[], int n) {
    if (n <= 1) return;

    // Trouver le nombre maximum pour connaître le nombre de chiffres
    int max_val = find_max(arr, n);

    // Effectuer le tri par dénombrement pour chaque chiffre.
    // exp est 10^i où i est le numéro du chiffre courant.
    for (int exp = 1; max_val / exp > 0; exp *= 10) {
        counting_sort_for_radix(arr, n, exp);
    }
}

void print_array(int arr[], int n) {
    for (int i = 0; i < n; i++) {
        printf("%d ", arr[i]);
    }
    printf("\n");
}

int main() {
    int arr[] = {170, 45, 75, 90, 802, 24, 2, 66};
    int n = sizeof(arr) / sizeof(arr[0]);

    printf("Tableau original : ");
    print_array(arr, n);

    radix_sort(arr, n);

    printf("Tableau trié     : ");
    print_array(arr, n);

    return 0;
}
```

#### Complexité du Tri par Base

Si $d$ est le nombre de chiffres du plus grand nombre, $N$ le nombre d'éléments à trier, et $k$ la base (le nombre de valeurs possibles pour un chiffre, par exemple 10), la complexité du tri par base est $O(d \cdot (N+k))$.

Puisque $k$ est souvent une petite constante (10 pour les nombres décimaux, 256 si on trie octet par octet), la complexité est souvent simplifiée en $O(d \cdot N)$. Si le nombre de chiffres $d$ est également considéré comme constant ou petit par rapport à $N$, le tri par base se comporte comme un algorithme en $O(N)$.

Le principe du tri par base, qui consiste à décomposer un problème complexe (trier de grands nombres) en une série de problèmes plus simples (trier par chiffre), est une technique de conception algorithmique puissante. Des travaux comme "Engineering Radix Sort" de McIlroy et al. ont montré comment cette idée théorique peut être optimisée pour des applications réelles, comme le tri de chaînes de caractères, au point de surpasser des algorithmes génériques comme Quicksort dans ces contextes. Cela illustre parfaitement le passage de la théorie abstraite à l'ingénierie logicielle performante.

## Partie 4 : Algorithmes Modernes et Hybrides - Le Cas de Timsort

Les algorithmes que nous avons étudiés jusqu'à présent sont des "algorithmes purs" : ils suivent une seule et unique stratégie. Cependant, en pratique, les algorithmes les plus performants sont souvent des hybrides pragmatiques, qui combinent les forces de plusieurs approches pour s'adapter aux réalités des données et du matériel. L'exemple le plus emblématique de cette philosophie est Timsort.

### 4.1 Principe d'un Tri Hybride

La motivation derrière les tris hybrides est simple : aucun algorithme pur n'est parfait dans toutes les situations.

- Le tri fusion est excellent sur de grands ensembles de données, avec une complexité garantie de $O(N \log N)$, mais son surcoût (appels récursifs, allocation de mémoire) le rend moins efficace sur de très petits tableaux.
- Le tri par insertion est quadratique dans le pire des cas, mais il est extrêmement rapide sur de petits tableaux (moins de 32 ou 64 éléments) et sur des données qui sont déjà "presque triées".

Un algorithme hybride comme Timsort cherche à obtenir le meilleur des deux mondes. Conçu en 2002 par Tim Peters pour le langage de programmation Python, il est depuis devenu l'algorithme de tri standard dans de nombreux langages, y compris Java et Android, en raison de ses performances exceptionnelles sur des données du monde réel.

### 4.2 Exploiter la Structure des Données du Monde Réel

La grande innovation de Timsort est qu'il a été conçu non pas pour le cas théorique de données parfaitement aléatoires, mais pour les types de données que l'on rencontre en pratique. Il part du principe que les données du monde réel sont rarement chaotiques ; elles possèdent souvent des structures partielles. Timsort est un algorithme adaptatif : il détecte et exploite cette structure pour accélérer le tri.

#### "Natural Runs" (Séquences Naturellement Triées)

La première étape de Timsort consiste à parcourir le tableau pour identifier des sous-séquences déjà ordonnées, appelées "natural runs" (ou simplement "runs"). Contrairement à beaucoup d'autres algorithmes, Timsort considère à la fois les séquences strictement croissantes et les séquences strictement décroissantes comme des runs (il inverse simplement ces dernières).

Par exemple, dans le tableau `[1, 3, 5, 9, 7, 6, 2, 8, 10]`, Timsort identifierait trois runs :
- `[1, 3, 5, 9]` (run croissant)
- `[9, 7, 6, 2]` qui, après inversion, devient `[2, 6, 7, 9]` (run décroissant)
- `[8, 10]` (run croissant)

L'algorithme utilise ensuite une stratégie de fusion intelligente, inspirée du tri fusion, pour combiner ces runs de manière efficace.

#### "Galloping Mode" (Mode Galopant)

L'une des optimisations les plus astucieuses de Timsort est le "galloping mode" (mode galopant). Lors de la fusion de deux runs, A et B, si les éléments proviennent systématiquement du même run (par exemple, plusieurs éléments de A sont plus petits que le premier élément de B), Timsort en déduit que les runs sont probablement très déséquilibrés. Au lieu de continuer à comparer les éléments un par un, il passe en mode galopant.

Dans ce mode, il utilise une recherche exponentielle (similaire à une recherche binaire, mais plus efficace lorsque l'élément recherché est proche du début) pour trouver rapidement où le premier élément du run B s'insérerait dans le run A. Il peut alors copier tout le bloc d'éléments de A précédant cette position en une seule fois, sans faire de comparaisons individuelles. Ce mécanisme accélère considérablement la fusion lorsque les données sont très structurées.

### 4.3 Analyse et Complexité

**Stabilité :** Timsort est un tri stable, une propriété essentielle pour de nombreuses applications, comme le tri d'objets sur plusieurs clés successives.

**Complexité :** Dans le pire des cas, Timsort a une complexité temporelle de $O(N \log N)$, ce qui le rend robuste. Cependant, sa véritable force réside dans sa performance sur des données typiques. Sa complexité peut être exprimée de manière plus fine en $O(N + N \log \rho)$, où $\rho$ est le nombre de runs dans les données. Si les données sont déjà presque triées, $\rho$ sera petit, et la complexité se rapprochera de $O(N)$.

Timsort représente un changement de paradigme dans la conception d'algorithmes. Il s'éloigne de l'optimisation pour le pire cas théorique (données aléatoires) pour se concentrer sur l'optimisation pour le cas typique (données structurées). Son succès démontre que les algorithmes les plus efficaces en pratique sont souvent ceux qui combinent pragmatiquement des idées théoriques solides avec une compréhension fine des caractéristiques des données qu'ils sont amenés à traiter.

## Partie 5 : Aspects Pratiques en C - La Gestion de la Mémoire

Après avoir exploré la théorie et les algorithmes modernes, revenons à des considérations plus pratiques, spécifiques au langage C. Une source fréquente d'erreurs et d'inefficacité pour les programmeurs C réside dans la mauvaise compréhension de la manière dont les tableaux multidimensionnels sont représentés et manipulés en mémoire. La syntaxe `arr[i][j]` peut cacher deux réalités radicalement différentes.

### 5.1 Tableaux Multidimensionnels : La Distinction Fondamentale

#### Type 1 : Allocation Contiguë (`int arr[R][C]`)

Lorsque vous déclarez un tableau statiquement ou sur la pile comme `int matrice[10][20];`, le compilateur alloue un unique bloc de mémoire contigu pour stocker tous les éléments. Pour une matrice de 10 lignes et 20 colonnes, il allouera un espace pour $10 \times 20 = 200$ entiers. Ces entiers sont disposés en mémoire ligne par ligne (row-major order) : d'abord les 20 éléments de la première ligne, puis les 20 de la deuxième, et ainsi de suite.

L'accès à un élément `matrice[i][j]` est alors une simple opération arithmétique calculée par le compilateur :

$$\text{adresse}(\text{matrice}[i][j]) = \text{adresse\_de\_base} + (i \times C + j) \times \text{sizeof(int)}$$

où $C$ est le nombre de colonnes (ici, 20). Ce calcul explique pourquoi, lorsque vous passez un tel tableau à une fonction, vous devez obligatoirement spécifier la taille de toutes les dimensions sauf la première. Le compilateur a besoin de connaître $C$ pour naviguer correctement dans le bloc de mémoire.

#### Type 2 : Tableau de Pointeurs (`int **arr`)

Une autre façon de représenter une matrice, particulièrement courante avec l'allocation dynamique, est d'utiliser un tableau de pointeurs. L'implémentation se fait en deux temps :

1. On alloue un tableau de $R$ pointeurs (`int*`).
2. Pour chacun de ces $R$ pointeurs, on alloue un tableau de $C$ entiers (`int`).

La variable que vous manipulez est alors de type `int**` (un pointeur vers un pointeur d'entier). En mémoire, la structure est fragmentée ou "râpée" (jagged ou ragged). Vous avez un premier bloc de mémoire pour les pointeurs, et chaque pointeur mène à un autre bloc de mémoire (une ligne) qui peut se trouver n'importe où ailleurs dans le tas (heap).

L'accès à `arr[i][j]` est une opération de double déréférenciation :
- `arr[i]` accède au $i$-ème pointeur dans le tableau de pointeurs.
- L'expression complète `arr[i][j]` déréférence ce pointeur pour trouver l'adresse de la $i$-ème ligne, puis accède au $j$-ème élément de cette ligne. Cela est équivalent à `*(*(arr + i) + j)`.

#### Implications Pratiques et Tableau Comparatif

Ces deux représentations, bien qu'utilisant une syntaxe d'accès similaire, ont des implications très différentes en termes de performance et de flexibilité.

**Performance :** L'allocation contiguë est généralement plus rapide. Comme toutes les données sont dans un seul bloc, elle bénéficie d'une bien meilleure localité du cache. Lorsque le processeur charge une partie de la matrice en cache, il est très probable que les éléments voisins dont il aura besoin ensuite s'y trouvent déjà. La structure fragmentée du tableau de pointeurs peut entraîner de nombreux sauts en mémoire et des défauts de cache (cache misses), ce qui ralentit l'exécution.

**Flexibilité :** Le tableau de pointeurs est plus flexible. Il permet de créer des matrices "râpées" où chaque ligne a une longueur différente. De plus, échanger deux lignes est une opération extrêmement rapide et peu coûteuse : il suffit d'échanger les deux pointeurs correspondants dans le tableau de pointeurs, sans avoir à copier les données des lignes elles-mêmes. Avec un tableau contigu, l'échange de deux lignes nécessite la copie de tous leurs éléments.

Le tableau suivant synthétise ces différences cruciales :

| Caractéristique | Allocation Contiguë (`int arr[R][C]`) | Tableau de Pointeurs (`int **arr`) |
|----------------|---------------------------------------|-----------------------------------|
| **Disposition Mémoire** | Un seul bloc de $R \times C$ éléments | Un tableau de $R$ pointeurs, chacun pointant vers une ligne de $C$ éléments. Mémoire fragmentée. |
| **Calcul d'Accès `arr[i][j]`** | Arithmétique : `base + (i*C + j)` | Déréférenciation : `*(*(arr + i) + j)` |
| **Passage en Fonction** | `void func(int arr[][C])` ($C$ doit être connu) | `void func(int **arr, int R, int C)` |
| **Flexibilité (taille des lignes)** | Fixe pour toutes les lignes | Chaque ligne peut avoir une taille différente. |
| **Coût d'Échange de Lignes** | Élevé : copie de $2 \times C$ éléments | Faible : échange de 2 pointeurs ($O(1)$) |
| **Localité du Cache / Performance** | Excellente | Potentiellement faible, risque de défauts de cache. |

Comprendre cette distinction est fondamental en C. Ce n'est pas seulement une question de syntaxe, mais une question d'architecture mémoire. Choisir la mauvaise représentation pour un problème donné peut conduire à un code non seulement incorrect, mais aussi terriblement inefficace. C'est un exemple parfait de la façon dont une abstraction de haut niveau (un "tableau 2D") est implémentée de manières très différentes au niveau machine, et de l'importance pour un ingénieur logiciel de maîtriser ces détails de bas niveau.

## Conclusion et Travaux Dirigés

Au cours de ce séminaire, nous avons entrepris un voyage qui nous a menés bien au-delà des algorithmes standards. Nous avons commencé par un problème concret, l'élément majoritaire, pour illustrer comment une meilleure compréhension de la structure d'un problème peut faire passer la complexité de quadratique à linéaire. Nous avons ensuite érigé un mur théorique, la borne inférieure de $\Omega(N \log N)$ pour les tris par comparaison, avant de montrer comment le franchir en changeant de modèle de calcul. Cela nous a conduits aux tris en temps linéaire comme le tri par dénombrement et le tri par base, soulignant au passage le rôle crucial de la stabilité. Enfin, nous avons exploré Timsort, un exemple d'algorithme moderne, hybride et adaptatif, qui illustre la tendance actuelle de la conception algorithmique : l'optimisation pour les données du monde réel plutôt que pour des cas théoriques aléatoires.

Le message principal à retenir est que le choix du "bon" algorithme n'est jamais une décision unique. Il dépend d'une compréhension profonde à la fois des fondations théoriques, des garanties de complexité, et des caractéristiques spécifiques des données à traiter. Il n'y a pas d'algorithme universellement supérieur ; il n'y a que des outils plus ou moins adaptés à une tâche donnée.

Le tableau suivant offre une vue d'ensemble comparative des algorithmes que nous avons discutés, un aide-mémoire pour guider vos choix futurs :

| Algorithme | Complexité Temporelle (Pire Cas) | Complexité Spatiale | Stabilité | Modèle | Cas d'Usage Idéal |
|------------|----------------------------------|---------------------|-----------|--------|-------------------|
| **Tri par Sélection** | $O(N^2)$ | $O(1)$ | Non | Comparaison | Petits tableaux, quand la mémoire est très limitée. |
| **Tri par Insertion** | $O(N^2)$ | $O(1)$ | Oui | Comparaison | Très petits tableaux, données presque triées. |
| **Tri Fusion** | $O(N \log N)$ | $O(N)$ | Oui | Comparaison | Tri stable et garanti en $O(N \log N)$ requis. |
| **Tri Rapide** | $O(N^2)$ (moyenne $O(N \log N)$) | $O(\log N)$ | Non | Comparaison | Tri générique rapide en moyenne, pas de besoin de stabilité. |
| **Tri par Dénombrement** | $O(N+k)$ | $O(N+k)$ | Oui | Non-comparatif | Données entières dans une plage $k$ restreinte. |
| **Tri par Base** | $O(d(N+k))$ | $O(N+k)$ | Oui | Non-comparatif | Données entières ou chaînes de caractères. |
| **Timsort** | $O(N \log N)$ | $O(N)$ | Oui | Comparaison (Hybride) | Tri générique pour des données du monde réel (partiellement structurées). |

### Travaux Dirigés

Pour consolider les concepts vus aujourd'hui, voici quelques pistes de travail :

1. **Implémentation de l'Élément Majoritaire :** Implémentez les trois algorithmes pour le problème de l'élément majoritaire (naïf, diviser pour régner, Boyer-Moore). Mesurez leur temps d'exécution sur des tableaux de différentes tailles et avec différentes distributions de données (avec et sans élément majoritaire) pour vérifier expérimentalement leurs classes de complexité.

2. **Multiplication de Matrices :** Écrivez deux versions d'une fonction de multiplication de matrices $N \times N$. La première utilisera une matrice allouée de manière contiguë (`int mat[N][N]`), la seconde une matrice allouée via un tableau de pointeurs (`int **mat`). Comparez leurs performances pour de grandes valeurs de $N$ et analysez les résultats à la lumière du concept de localité du cache.

3. **Problème de Tri Stable :** Concevez un programme qui trie une liste de points 2D. Le tri doit se faire d'abord par coordonnée $x$ croissante, puis, en cas d'égalité sur $x$, par coordonnée $y$ croissante. Implémentez ce tri en utilisant une bibliothèque de tri standard (comme `qsort` en C) deux fois de suite. Observez-vous le bon résultat ? Pourquoi ? Ré-implémentez la solution en utilisant un algorithme de tri stable que vous aurez codé (par exemple, le tri fusion) et comparez.

4. **Lecture Avancée :** Pour les plus curieux, je vous recommande la lecture des articles originaux ou des analyses détaillées sur les algorithmes que nous avons survolés. En particulier, les articles sur Timsort et le papier "Engineering Radix Sort" de McIlroy et al. sont d'excellentes lectures pour comprendre comment la théorie se traduit en ingénierie logicielle de haute performance.

Je vous remercie de votre attention. La séance est levée.
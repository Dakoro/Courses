# Structures de Données et Ingénierie Logicielle en C : Une Exploration Approfondie

## Partie I : Les Tables de Hachage – De la Théorie à l'Implémentation Pratique

### 1.1. Introduction et Motivation : Le Problème de l'Annuaire Téléphonique

En informatique, le choix d'une structure de données appropriée est souvent la pierre angulaire d'un algorithme efficace. Pour illustrer ce principe, considérons un problème pratique et courant : la gestion d'un annuaire téléphonique.

La tâche consiste à concevoir une structure de données capable de stocker une grande quantité de contacts, chacun associé à un nom et un numéro de téléphone (pouvant contenir jusqu'à 15 chiffres), tout en garantissant deux opérations fondamentales : l'ajout rapide d'un nouveau contact et la recherche quasi instantanée d'un contact par son numéro de téléphone.

#### Analyse de la Solution 1 : Le Tableau Trié

Une première approche, souvent intuitive pour les programmeurs, consiste à utiliser un tableau de structures `Contact` (contenant un nom et un numéro), trié par numéro de téléphone.

**Performance de la Recherche :** Grâce à la nature triée du tableau, la recherche d'un numéro de téléphone peut être effectuée de manière très efficace en utilisant un algorithme de recherche binaire (dichotomique). La complexité temporelle de cette opération est de $O(\log N)$, où $N$ est le nombre de contacts. C'est une performance excellente, même pour des millions de contacts.

**Performance de l'Insertion :** C'est ici que cette solution révèle sa faiblesse majeure. Pour insérer un nouveau contact, il faut d'abord trouver sa position correcte dans le tableau, puis décaler tous les éléments suivants d'une position pour lui faire de la place. Dans le pire des cas (insertion au début du tableau), cette opération nécessite le déplacement de $N$ éléments, ce qui conduit à une complexité temporelle de $O(N)$. Pour un annuaire de grande taille, cette lenteur est prohibitive et rend la structure inadaptée aux applications nécessitant des ajouts fréquents.

#### Analyse de la Solution 2 : La Liste Chaînée

Face à la lenteur de l'insertion dans un tableau trié, une autre structure de données classique vient à l'esprit : la liste chaînée.

**Performance de l'Insertion :** L'avantage principal de la liste chaînée est la rapidité de l'insertion. Ajouter un nouvel élément, par exemple en tête de liste, ne requiert que la modification de quelques pointeurs, une opération en temps constant, soit $O(1)$.

**Performance de la Recherche :** Malheureusement, ce gain en insertion se paie au prix d'une recherche très inefficace. Les listes chaînées n'offrent pas d'accès direct aux éléments. Pour trouver un contact par son numéro, il est nécessaire de parcourir la liste séquentiellement depuis le début jusqu'à trouver l'élément recherché. Dans le pire des cas, cela implique de parcourir toute la liste, résultant en une complexité de $O(N)$.

#### Synthèse du Problème

Nous sommes face à un dilemme classique en algorithmique : un compromis entre la vitesse de recherche et la vitesse d'insertion. Le tableau trié favorise la recherche, tandis que la liste chaînée favorise l'insertion. Aucune de ces structures simples ne répond à notre exigence d'avoir à la fois une recherche et une insertion rapides.

Le tableau suivant résume cette analyse comparative et met en évidence la nécessité d'une structure de données plus sophistiquée.

| Structure de données | Insertion | Recherche (élément unique) | Avantages | Inconvénients |
|---------------------|-----------|---------------------------|-----------|---------------|
| Tableau Trié | $O(N)$ | $O(\log N)$ | Recherche très rapide | Insertion très lente |
| Liste Chaînée | $O(1)$ | $O(N)$ | Insertion très rapide | Recherche très lente |

C'est pour résoudre ce type de problème qu'ont été inventées les tables de hachage, qui visent à offrir une complexité moyenne en $O(1)$ pour ces deux opérations.

### 1.2. Le Principe du Hachage et la Gestion des Collisions par Chaînage

L'idée fondamentale qui sous-tend les tables de hachage est à la fois simple et puissante. Si les numéros de téléphone étaient des entiers de petite taille, disons de 0 à 999, nous pourrions simplement utiliser un tableau de taille 1000 et stocker les informations du contact à l'indice correspondant à son numéro de téléphone. L'accès serait alors direct et en $O(1)$. Le problème est que les numéros de téléphone sont des nombres très grands, et créer un tableau capable d'indexer directement chaque numéro possible est irréalisable en raison de la mémoire requise.

La solution consiste à utiliser une **fonction de hachage**. Cette fonction mathématique a pour rôle de transformer une clé d'un grand univers (l'ensemble de tous les numéros de téléphone possibles) en un indice dans un intervalle restreint et gérable, correspondant à la taille de notre tableau (par exemple, de 0 à 999). L'opération la plus simple pour y parvenir est l'opérateur modulo :

$$\text{hash}(\text{clé}) = \text{clé} \bmod \text{taille\_tableau}$$

Cependant, en mappant un grand ensemble de clés sur un ensemble d'indices plus petit, il est inévitable que deux clés distinctes produisent parfois le même indice de hachage. Ce phénomène est appelé une **collision**. Par exemple, si notre table a une taille de 1000, les numéros de téléphone 1234567890 et 9876543210 pourraient tous deux donner le même reste après division par 1000.

Pour résoudre ce problème, la méthode la plus courante est le **chaînage séparé** (ou separate chaining). Au lieu de stocker directement les données dans le tableau, chaque case du tableau (appelée "alvéole" ou bucket) contient un pointeur vers une liste chaînée. Tous les éléments dont les clés hachent vers le même indice sont simplement ajoutés à la liste chaînée correspondante. Cette approche est également connue dans la littérature sous le nom d'**adressage fermé** (closed addressing).

Si la fonction de hachage répartit les clés de manière uniforme, les listes chaînées resteront très courtes en moyenne. Pour trouver un élément, il suffit de :

1. Calculer l'indice de hachage de la clé.
2. Accéder à l'alvéole correspondante dans le tableau ($O(1)$).
3. Parcourir la courte liste chaînée pour trouver l'élément exact.

Si la longueur moyenne des listes est une petite constante, alors les opérations d'insertion, de recherche et de suppression peuvent être considérées comme ayant une complexité temporelle moyenne de $O(1)$. Des études empiriques confirment que le chaînage offre une performance stable et prévisible, même lorsque le facteur de charge (le rapport entre le nombre d'éléments et la taille du tableau) devient élevé.

### 1.3. Fonctions de Hachage Universelles : Une Approche Mathématique Rigoureuse

La performance d'une table de hachage dépend de manière critique de la qualité de sa fonction de hachage. Une fonction simple comme `clé % m` peut se comporter de manière désastreuse si les données d'entrée présentent des régularités. Par exemple, si $m=10$ et que toutes les clés sont des multiples de 10, toutes les clés tomberont dans la même alvéole, et la table de hachage dégénérera en une simple liste chaînée, avec une performance en $O(N)$.

Pour se prémunir contre de tels pires cas, et même contre un adversaire qui choisirait délibérément des clés pour provoquer des collisions, Carter et Wegman ont introduit en 1979 le concept de **hachage universel**. L'idée n'est pas de trouver une seule fonction de hachage parfaite, mais de définir une famille de fonctions de hachage et d'en choisir une au hasard à chaque exécution du programme.

Une famille de fonctions $\mathcal{H}$ est dite **universelle** si, pour toute paire de clés distinctes $x \neq y$, la probabilité qu'elles entrent en collision est inférieure ou égale à $1/m$, où $m$ est la taille de la table :

$$P_{h \in \mathcal{H}}(h(x) = h(y)) \leq \frac{1}{m}$$

Cette propriété garantit que, en moyenne, le nombre de collisions sera faible, quelles que soient les données d'entrée.

Une propriété encore plus forte, et souvent plus désirable, est la **forte universalité** (ou indépendance par paires). Une famille est fortement universelle si, pour deux clés distinctes $x$ et $y$, leurs valeurs de hachage $h(x)$ et $h(y)$ sont statistiquement indépendantes et uniformément distribuées sur l'ensemble des valeurs de hachage possibles. Cela signifie que la connaissance de $h(x)$ ne donne aucune information sur $h(y)$. Un avantage crucial de la forte universalité est qu'elle garantit une bonne distribution même si l'on n'utilise qu'une partie des bits du résultat du hachage (par exemple, en prenant le résultat modulo une puissance de deux), ce qui est une pratique courante.

#### Famille Universelle pour les Entiers

Une famille de fonctions universelles classique pour hacher des clés entières $k$ est donnée par la formule suivante :

$$h_{a,b}(k) = ((ak + b) \bmod p) \bmod m$$

Où :
- $k$ est la clé à hacher.
- $m$ est la taille de la table de hachage.
- $p$ est un grand nombre premier, choisi pour être supérieur à la plus grande valeur de clé possible.
- $a$ et $b$ sont des entiers choisis aléatoirement à chaque création de la table, avec $1 \leq a < p$ et $0 \leq b < p$.

#### Compromis entre Théorie et Pratique

La formule ci-dessus est mathématiquement robuste mais implique deux opérations de modulo, qui peuvent être coûteuses en termes de cycles de processeur. Historiquement, un compromis pratique courant en programmation consistait à choisir la taille de la table $m$ comme une puissance de deux, soit $m = 2^M$. L'opération de modulo $k \bmod 2^M$ peut alors être remplacée par une opération ET au niveau du bit, beaucoup plus rapide : `k & (m - 1)`.

Pour les entiers, une fonction de hachage rapide et populaire qui exploite cette idée est la suivante :

```
hash(x) = (a * x + b) >> (W - M)
```

Ici, $W$ est le nombre de bits dans un mot machine (par exemple, 32 ou 64), et les calculs sur $a \times x + b$ sont effectués en arithmétique non signée, où le débordement se comporte comme un modulo $2^W$. Le décalage à droite de $W - M$ bits permet d'extraire les $M$ bits de poids fort du résultat, qui servent d'indice de hachage.

Cependant, il est important de noter que ce qui est "rapide" évolue avec l'architecture des processeurs. Des recherches plus récentes, comme celles de Lemire et Kaser, ont montré que sur les processeurs modernes dotés de pipelines d'instructions sophistiqués, les fonctions de hachage fortement universelles, même si elles semblent plus complexes et comportent plus de multiplications, peuvent s'exécuter plus rapidement que les fonctions "simplifiées" conçues pour minimiser le nombre d'opérations. Cela illustre une leçon importante en ingénierie logicielle : les optimisations doivent toujours être mesurées dans le contexte de la plateforme cible, et les compromis théoriques ne se traduisent pas toujours par des gains de performance pratiques.

### 1.4. Implémentation Complète en C

Nous allons maintenant implémenter une table de hachage en C utilisant la méthode du chaînage séparé.

#### Pseudocode des Opérations

**Création de la table (ht_create)**
1. Allouer la mémoire pour la structure HashTable.
2. Définir sa capacité (taille du tableau).
3. Allouer la mémoire pour le tableau d'alvéoles (un tableau de pointeurs Node*).
4. Initialiser chaque alvéole à NULL.
5. Retourner un pointeur vers la nouvelle table.

**Insertion (ht_insert)**
1. Calculer l'indice de hachage pour la clé donnée.
2. Parcourir la liste chaînée à cet indice pour vérifier si la clé existe déjà.
3. Si oui, mettre à jour la valeur associée et retourner.
4. Si la clé n'existe pas, créer un nouveau nœud (Node).
5. Allouer de la mémoire pour la clé et la valeur et les copier dans le nœud.
6. Insérer le nouveau nœud en tête de la liste chaînée à l'indice calculé.

**Recherche (ht_find)**
1. Calculer l'indice de hachage pour la clé.
2. Parcourir la liste chaînée à cet indice.
3. Pour chaque nœud, comparer sa clé avec la clé recherchée.
4. Si une correspondance est trouvée, retourner la valeur associée.
5. Si la fin de la liste est atteinte sans correspondance, retourner NULL.

**Suppression (ht_delete)**
1. Calculer l'indice de hachage pour la clé.
2. Parcourir la liste chaînée à cet indice, en gardant un pointeur sur le nœud précédent.
3. Si le nœud à supprimer est trouvé :
   - Si c'est le premier nœud de la liste, mettre à jour la tête de liste dans l'alvéole.
   - Sinon, relier le nœud précédent au nœud suivant.
   - Libérer la mémoire du nœud supprimé (clé, valeur, et le nœud lui-même).

#### Implémentation en C

Voici une implémentation complète et commentée. Pour des raisons de simplicité, nous utiliserons une fonction de hachage simple pour les chaînes de caractères (djb2) et gérerons des paires clé-valeur de type (char*, char*).

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define TABLE_SIZE 1024

// Structure pour un nœud de la liste chaînée (une entrée dans la table)
typedef struct Node {
    char* key;
    char* value;
    struct Node* next;
} Node;

// Structure pour la table de hachage
typedef struct HashTable {
    Node** buckets; // Tableau de pointeurs vers les nœuds (les alvéoles)
    int size;       // Taille du tableau (nombre d'alvéoles)
} HashTable;

// Fonction de hachage (djb2) pour les chaînes de caractères
unsigned long hash_function(const char* str) {
    unsigned long hash = 5381;
    int c;
    while ((c = *str++)) {
        hash = ((hash << 5) + hash) + c; // hash * 33 + c
    }
    return hash;
}

// Crée et initialise une nouvelle table de hachage
HashTable* ht_create(int size) {
    HashTable* table = (HashTable*)malloc(sizeof(HashTable));
    if (!table) return NULL;

    table->size = size;
    table->buckets = (Node**)calloc(table->size, sizeof(Node*));
    if (!table->buckets) {
        free(table);
        return NULL;
    }
    return table;
}

// Insère ou met à jour une paire clé-valeur
void ht_insert(HashTable* table, const char* key, const char* value) {
    unsigned long index = hash_function(key) % table->size;
    Node* current = table->buckets[index];

    // Parcourir la liste pour voir si la clé existe déjà
    while (current != NULL) {
        if (strcmp(current->key, key) == 0) {
            // Clé trouvée, mettre à jour la valeur
            free(current->value);
            current->value = strdup(value);
            return;
        }
        current = current->next;
    }

    // Clé non trouvée, créer un nouveau nœud et l'insérer en tête de liste
    Node* newNode = (Node*)malloc(sizeof(Node));
    if (!newNode) return;

    newNode->key = strdup(key);
    newNode->value = strdup(value);
    newNode->next = table->buckets[index];
    table->buckets[index] = newNode;
}

// Recherche une valeur par sa clé
char* ht_find(HashTable* table, const char* key) {
    unsigned long index = hash_function(key) % table->size;
    Node* current = table->buckets[index];

    while (current != NULL) {
        if (strcmp(current->key, key) == 0) {
            return current->value; // Clé trouvée
        }
        current = current->next;
    }
    return NULL; // Clé non trouvée
}

// Supprime une paire clé-valeur
void ht_delete(HashTable* table, const char* key) {
    unsigned long index = hash_function(key) % table->size;
    Node* current = table->buckets[index];
    Node* prev = NULL;

    while (current != NULL) {
        if (strcmp(current->key, key) == 0) {
            if (prev == NULL) {
                // Le nœud à supprimer est en tête de liste
                table->buckets[index] = current->next;
            } else {
                // Le nœud à supprimer est dans le corps de la liste
                prev->next = current->next;
            }
            free(current->key);
            free(current->value);
            free(current);
            return;
        }
        prev = current;
        current = current->next;
    }
}

// Libère toute la mémoire utilisée par la table de hachage
void ht_destroy(HashTable* table) {
    for (int i = 0; i < table->size; i++) {
        Node* current = table->buckets[i];
        while (current != NULL) {
            Node* temp = current;
            current = current->next;
            free(temp->key);
            free(temp->value);
            free(temp);
        }
    }
    free(table->buckets);
    free(table);
}

// Fonction pour compter les collisions
// Une collision est définie comme tout élément supplémentaire dans une alvéole.
// Nombre de collisions = (nombre total d'éléments) - (nombre d'alvéoles non vides)
int count_collisions(HashTable* table) {
    int total_elements = 0;
    int non_empty_buckets = 0;

    for (int i = 0; i < table->size; i++) {
        Node* current = table->buckets[i];
        if (current != NULL) {
            non_empty_buckets++;
            while (current != NULL) {
                total_elements++;
                current = current->next;
            }
        }
    }
    
    if (non_empty_buckets == 0) return 0;
    return total_elements - non_empty_buckets;
}

// Programme principal pour tester la table de hachage
int main() {
    HashTable* ht = ht_create(TABLE_SIZE);

    ht_insert(ht, "name", "Alice");
    ht_insert(ht, "age", "30");
    ht_insert(ht, "city", "Paris");
    // Cette clé a une forte probabilité de collision avec "age" avec certaines fonctions de hachage
    ht_insert(ht, "gea", "some_value"); 

    printf("La valeur pour 'name' est : %s\n", ht_find(ht, "name"));
    printf("La valeur pour 'city' est : %s\n", ht_find(ht, "city"));

    printf("Nombre de collisions : %d\n", count_collisions(ht));

    ht_delete(ht, "age");
    printf("La valeur pour 'age' après suppression est : %s\n", ht_find(ht, "age") ? ht_find(ht, "age") : "(non trouvée)");

    ht_destroy(ht);
    return 0;
}
```

## Partie II : Application du Hachage – L'Algorithme de Rabin-Karp

Le hachage n'est pas seulement utile pour les structures de données comme les tables de hachage ; c'est aussi une technique puissante pour résoudre d'autres problèmes algorithmiques, notamment la recherche de motifs dans des textes.

### 2.1. Le Problème de la Recherche de Sous-chaîne

Le problème de la recherche de sous-chaîne consiste à trouver toutes les occurrences d'une chaîne de caractères, appelée "aiguille" (pattern), à l'intérieur d'un texte beaucoup plus grand, appelé "botte de foin" (haystack). C'est un problème fondamental en bio-informatique (recherche de séquences génétiques), en traitement de texte et dans les moteurs de recherche.

L'algorithme le plus simple, dit naïf, consiste à faire glisser une fenêtre de la taille de l'aiguille sur le texte, de gauche à droite. À chaque position, on compare l'aiguille, caractère par caractère, avec la sous-chaîne du texte correspondante. Si le texte a une longueur $N$ et l'aiguille une longueur $M$, cet algorithme effectue environ $N-M+1$ comparaisons de chaînes, chacune pouvant prendre jusqu'à $M$ comparaisons de caractères. La complexité temporelle dans le pire des cas est donc de $O(N \times M)$. Pour des textes très longs, comme le génome humain ($N \approx 3 \times 10^9$), cette complexité est inacceptable.

### 2.2. L'Approche de Rabin-Karp : Le Hachage Roulant

L'algorithme de Rabin-Karp, proposé par Richard M. Karp et Michael O. Rabin en 1987, offre une solution randomisée et en moyenne beaucoup plus efficace. L'idée centrale est de ne pas comparer les chaînes elles-mêmes, mais de comparer leurs "empreintes digitales" (fingerprints), qui sont leurs valeurs de hachage.

Pour cela, on utilise une technique de **hachage polynomial**. Une chaîne de caractères $S = c_1 c_2 \ldots c_M$ est interprétée comme un nombre dans une base $d$, où $d$ est la taille de l'alphabet (par exemple, 256 pour les caractères ASCII). La valeur de hachage est calculée comme suit :

$$h(S) = (c_1 \cdot d^{M-1} + c_2 \cdot d^{M-2} + \cdots + c_M \cdot d^0) \bmod q$$

où $q$ est un grand nombre premier choisi pour minimiser les collisions et éviter les dépassements de capacité.

L'astuce qui rend cet algorithme efficace est le **hachage roulant** (rolling hash). Au lieu de recalculer entièrement le hachage pour chaque sous-chaîne du texte (ce qui prendrait $O(M)$ à chaque fois), on peut calculer le hachage de la fenêtre suivante, $T[i+1..i+M]$, à partir du hachage de la fenêtre actuelle, $T[i..i+M-1]$, en temps constant $O(1)$.

La formule de mise à jour est la suivante :

$$h(S_{i+1}) = (d \times (h(S_i) - \text{val}(T[i]) \times d^{M-1}) + \text{val}(T[i+M])) \bmod q$$

Cette formule effectue trois opérations simples :
1. On soustrait la contribution du premier caractère de l'ancienne fenêtre.
2. On décale tout d'une position en multipliant par la base $d$.
3. On ajoute la contribution du nouveau caractère à la fin de la fenêtre.

### 2.3. Gestion des Faux Positifs et Analyse de Complexité

Un point crucial de l'algorithme est que l'égalité des hachages ($h(\text{aiguille}) = h(\text{sous-chaîne})$) ne garantit pas l'égalité des chaînes elles-mêmes. C'est une collision de hachage. Ces cas sont appelés des **faux positifs**. Par conséquent, chaque fois que les valeurs de hachage correspondent, il est impératif d'effectuer une comparaison caractère par caractère pour vérifier s'il s'agit d'une véritable correspondance.

**Complexité en cas moyen :** Si la fonction de hachage est bien choisie (c'est-à-dire que $q$ est un grand nombre premier), les collisions sont rares. L'algorithme passe la plupart de son temps à calculer et à comparer les hachages. Le calcul initial du hachage de l'aiguille et de la première fenêtre prend $O(M)$. Ensuite, il y a $N-M$ mises à jour en $O(1)$. La complexité totale moyenne est donc de $O(N+M)$.

**Complexité dans le pire des cas :** Si un adversaire choisit un texte et une aiguille qui provoquent un grand nombre de collisions de hachage (par exemple, un texte composé de 'a' et une aiguille composée de 'a'), l'algorithme devra effectuer une comparaison de $M$ caractères à presque chaque position. Dans ce scénario pathologique, la complexité dégénère en $O(N \times M)$, la même que l'algorithme naïf.

### 2.4. Pseudocode et Implémentation en C

#### Pseudocode

**Initialisation**
- $N \leftarrow \text{longueur}(\text{texte})$
- $M \leftarrow \text{longueur}(\text{aiguille})$
- $d \leftarrow \text{taille de l'alphabet}$ (ex: 256)
- $q \leftarrow \text{un grand nombre premier}$
- $h \leftarrow d^{M-1} \bmod q$ (pré-calculer pour la mise à jour)
- $\text{pHash} \leftarrow \text{hash}(\text{aiguille}[0..M-1])$
- $\text{tHash} \leftarrow \text{hash}(\text{texte}[0..M-1])$

**Boucle Principale**
- Pour $i$ de 0 à $N-M$ :
  - Si $\text{pHash} = \text{tHash}$ :
    - Comparer caractère par caractère aiguille et texte[i..i+M-1].
    - Si elles sont identiques, "Correspondance trouvée à l'indice i".
  - Si $i < N-M$ :
    - $\text{tHash} \leftarrow (d \times (\text{tHash} - \text{val}(\text{texte}[i]) \times h) + \text{val}(\text{texte}[i+M])) \bmod q$
    - Si tHash < 0, ajouter $q$ pour le rendre positif.

#### Implémentation en C

```c
#include <stdio.h>
#include <string.h>

// d est la taille de l'alphabet (par exemple, 256 pour les caractères ASCII)
#define d 256

/*
 * Fonction de recherche de Rabin-Karp
 * pat: l'aiguille (pattern)
 * txt: la botte de foin (text)
 * q: un nombre premier pour les opérations de modulo
 */
void search(const char* pat, const char* txt, int q) {
    int M = strlen(pat);
    int N = strlen(txt);
    int i, j;
    int p = 0; // Valeur de hachage pour l'aiguille
    int t = 0; // Valeur de hachage pour la fenêtre de texte
    int h = 1;

    // La valeur de h est pow(d, M-1) % q
    for (i = 0; i < M - 1; i++) {
        h = (h * d) % q;
    }

    // Calculer la valeur de hachage de l'aiguille et de la première fenêtre du texte
    for (i = 0; i < M; i++) {
        p = (d * p + pat[i]) % q;
        t = (d * t + txt[i]) % q;
    }

    // Faire glisser la fenêtre sur le texte
    for (i = 0; i <= N - M; i++) {
        // Si les hachages correspondent, vérifier les caractères un par un
        if (p == t) {
            for (j = 0; j < M; j++) {
                if (txt[i + j] != pat[j]) {
                    break;
                }
            }
            // Si la boucle s'est terminée sans 'break', alors une correspondance a été trouvée
            if (j == M) {
                printf("Aiguille trouvée à l'indice %d\n", i);
            }
        }

        // Calculer la valeur de hachage pour la fenêtre suivante
        if (i < N - M) {
            // t = (d * (t - txt[i] * h) + txt[i + M]) % q;
            // L'opération de soustraction peut donner un résultat négatif.
            // On ajoute q pour s'assurer que le résultat reste positif avant le modulo.
            t = (d * (t - txt[i] * h) + txt[i + M]) % q;
            if (t < 0) {
                t = (t + q);
            }
        }
    }
}

// Programme principal pour tester la fonction
int main() {
    const char* txt = "AABAACAADAABAABA";
    const char* pat = "AABA";
    int q = 101; // Un nombre premier
    
    printf("Texte: %s\n", txt);
    printf("Aiguille: %s\n", pat);
    search(pat, txt, q);
    
    return 0;
}
```

## Partie III : Au-delà du Hachage – Arbres de Recherche et Équilibrage

### 3.1. La Limite Fondamentale des Tables de Hachage : La Perte de l'Ordre

Malgré leur efficacité pour les recherches d'éléments uniques, les tables de hachage ont une limitation fondamentale : elles détruisent l'information relative à l'ordre des clés. La fonction de hachage, par sa nature même, "disperse" ou "mélange" les clés dans le tableau pour assurer une distribution uniforme et minimiser les collisions. Une clé 100 peut être mappée à l'indice 5, tandis qu'une clé 101 pourrait être mappée à l'indice 97. L'ordre naturel des clés est perdu.

Cette caractéristique les rend inefficaces pour les **requêtes par intervalle** (range queries). Prenons un exemple concret : dans une base de données de villes, nous pourrions vouloir trouver toutes les villes situées à une distance de Moscou comprise entre 100 km et 200 km. Si les distances sont utilisées comme clés dans une table de hachage, il n'y a pas de moyen efficace de répondre à cette question. La seule approche possible serait de tester chaque valeur de l'intervalle une par une et de vérifier son existence dans la table, ce qui est extrêmement lent.

### 3.2. Les Arbres Binaires de Recherche (ABR) comme Solution aux Requêtes par Intervalle

C'est là que les structures de données basées sur l'ordre, comme les arbres binaires de recherche (ABR), montrent leur supériorité. Un ABR maintient la propriété suivante : pour tout nœud, toutes les clés de son sous-arbre gauche sont plus petites, et toutes les clés de son sous-arbre droit sont plus grandes.

**Performance :** Dans un ABR équilibré, la hauteur $h$ est de l'ordre de $O(\log N)$. Les opérations d'insertion, de recherche et de suppression ont donc une complexité de $O(\log N)$.

**Requêtes par intervalle :** Les ABR excellent dans cet exercice. Pour trouver tous les éléments dans un intervalle $[\text{min}, \text{max}]$, on peut utiliser un parcours récursif qui élague intelligemment la recherche :

- Si la valeur du nœud courant est supérieure à max, on sait que tous les éléments recherchés ne peuvent se trouver que dans le sous-arbre gauche. On ignore donc le sous-arbre droit.
- Si la valeur du nœud courant est inférieure à min, on n'explore que le sous-arbre droit.
- Si la valeur du nœud courant est dans l'intervalle $[\text{min}, \text{max}]$, on l'ajoute au résultat, puis on explore les deux sous-arbres.

La complexité de cette opération est de $O(\log N + K)$, où $K$ est le nombre d'éléments trouvés dans l'intervalle. C'est exponentiellement plus efficace que l'approche de la table de hachage. De plus, un parcours in-order (infixe) de l'arbre permet de récupérer les résultats de manière triée sans coût supplémentaire.

#### Implémentation en C d'une Requête par Intervalle

```c
#include <stdio.h>
#include <stdlib.h>

// Structure pour un nœud de l'ABR
typedef struct TreeNode {
    int key;
    struct TreeNode *left;
    struct TreeNode *right;
} TreeNode;

// Fonction pour créer un nouveau nœud
TreeNode* newNode(int item) {
    TreeNode* temp = (TreeNode*)malloc(sizeof(TreeNode));
    temp->key = item;
    temp->left = temp->right = NULL;
    return temp;
}

// Fonction pour insérer un nouveau nœud dans l'ABR
TreeNode* insert(TreeNode* node, int key) {
    if (node == NULL) return newNode(key);
    if (key < node->key)
        node->left = insert(node->left, key);
    else if (key > node->key)
        node->right = insert(node->right, key);
    return node;
}

// Fonction pour afficher les clés dans l'intervalle [min, max]
void print_range(TreeNode* root, int min, int max) {
    if (root == NULL) {
        return;
    }

    // Si la clé du nœud est plus grande que min, il pourrait y avoir des clés valides à gauche
    if (root->key > min) {
        print_range(root->left, min, max);
    }

    // Si la clé du nœud est dans l'intervalle, on l'affiche
    if (root->key >= min && root->key <= max) {
        printf("%d ", root->key);
    }

    // Si la clé du nœud est plus petite que max, il pourrait y avoir des clés valides à droite
    if (root->key < max) {
        print_range(root->right, min, max);
    }
}

// Programme principal pour tester
int main() {
    TreeNode* root = NULL;
    root = insert(root, 20);
    insert(root, 8);
    insert(root, 22);
    insert(root, 4);
    insert(root, 12);
    insert(root, 10);
    insert(root, 14);

    int min = 10, max = 22;
    printf("Clés dans l'intervalle [%d, %d]: ", min, max);
    print_range(root, min, max);
    printf("\n");

    return 0;
}
```

### 3.3. Le Problème du Déséquilibre et les Arbres Auto-équilibrés

La performance en $O(\log N)$ d'un ABR n'est garantie que si l'arbre reste équilibré. Si les clés sont insérées dans un ordre déjà trié (ou presque trié), l'arbre dégénère en une structure ressemblant à une liste chaînée. Sa hauteur devient $O(N)$, et toutes les opérations redeviennent linéaires, perdant ainsi tout l'avantage de la structure arborescente.

La solution à ce problème réside dans les **arbres auto-équilibrés**, tels que les arbres AVL ou les arbres Rouge-Noir. Ces structures de données maintiennent des invariants stricts qui garantissent que l'arbre ne devient jamais trop déséquilibré. Après chaque insertion ou suppression, elles effectuent des opérations de restructuration pour préserver une hauteur logarithmique.

### 3.4. Focus Conceptuel sur les Arbres Rouge-Noir

Les arbres Rouge-Noir sont une des implémentations les plus populaires d'arbres binaires de recherche auto-équilibrés. Ils sont utilisés dans de nombreux systèmes, y compris le noyau Linux, les bibliothèques standard C++ (`std::map`) et Java (`TreeMap`).

Plutôt que de voir les règles des arbres Rouge-Noir comme un ensemble de contraintes arbitraires, il est plus éclairant de les comprendre comme une représentation binaire d'un arbre 2-3-4, comme l'ont montré Guibas et Sedgewick. Dans cette analogie, un nœud rouge sert à "lier" des nœuds binaires pour former un "super-nœud" qui peut contenir plus d'une clé (un 3-nœud ou un 4-nœud d'un arbre 2-3-4).

Les propriétés formelles d'un arbre Rouge-Noir découlent de cette analogie :

1. **Propriété de Couleur :** Chaque nœud est soit Rouge, soit Noir.
2. **Propriété de la Racine :** La racine est toujours Noire.
3. **Propriété Rouge :** Un nœud Rouge ne peut pas avoir d'enfant Rouge. (Ceci empêche la création de "super-nœuds" de taille supérieure à 3 clés).
4. **Propriété de la Hauteur Noire :** Tous les chemins simples d'un nœud donné à l'une de ses feuilles descendantes contiennent le même nombre de nœuds Noirs. (Ceci garantit que l'arbre 2-3-4 sous-jacent est parfaitement équilibré).
5. **Propriété des Feuilles :** Toutes les feuilles (représentées par des pointeurs NULL) sont considérées comme Noires.

Ces propriétés garantissent que le chemin le plus long de la racine à une feuille n'est jamais plus de deux fois plus long que le chemin le plus court, ce qui assure une hauteur de l'arbre en $O(\log N)$.

Pour maintenir ces propriétés après une insertion ou une suppression, deux types d'opérations sont utilisés :

**Les Rotations (Gauche/Droite) :** Ce sont des opérations locales qui modifient la structure de l'arbre en réarrangeant les relations parent-enfant, tout en préservant la propriété de l'ABR. Elles sont utilisées pour corriger les déséquilibres de hauteur.

**Les Changements de Couleur :** Modifier la couleur des nœuds est utilisé pour restaurer les propriétés des couleurs. Un changement de couleur correspond souvent à la division (split) d'un 4-nœud ou à la fusion (merge) de nœuds dans l'arbre 2-3-4 sous-jacent.

La théorie complète des arbres Rouge-Noir est complexe et dépasse le cadre de cette introduction, mais il est essentiel de retenir qu'ils offrent des garanties de performance en $O(\log N)$ dans le pire des cas pour toutes les opérations de dictionnaire, ce qui les rend extrêmement fiables pour les applications critiques.

## Partie IV : Ingénierie Logicielle en C – Encapsulation et Types de Données Abstraits (ADT)

### 4.1. Le Problème de la Fragilité des Structures de Données en C

Les structures de données que nous avons étudiées, comme les listes chaînées, les arbres et les tables de hachage, sont souvent implémentées en C à l'aide de pointeurs. Le langage C offre un contrôle direct et puissant sur la mémoire, mais cette puissance s'accompagne d'une grande fragilité. Un programmeur utilisant une bibliothèque de structures de données peut facilement, par erreur ou par ignorance, corrompre l'intégrité de la structure. Par exemple :

- Dans une liste chaînée, on peut accidentellement créer une boucle en faisant pointer un nœud vers un de ses prédécesseurs.
- Dans un arbre, un pointeur peut être modifié pour pointer vers un nœud incorrect, violant la propriété de l'ABR.
- Dans une table de hachage, un pointeur d'une alvéole pourrait être redirigé vers la liste d'une autre alvéole.

Une solution naïve serait de vérifier l'intégrité de la structure avant chaque opération (par exemple, lancer un algorithme de détection de cycle de Floyd avant chaque parcours de liste). Cependant, de telles vérifications sont souvent aussi coûteuses, voire plus, que l'opération elle-même (par exemple, une vérification en $O(N)$ avant une recherche en $O(\log N)$ dans un arbre), ce qui annulerait tous les gains de performance.

### 4.2. La Solution d'Ingénierie : l'Encapsulation

Si la vérification à chaque utilisation est impraticable, la seule approche robuste est de garantir l'intégrité par construction. C'est le rôle de l'**encapsulation**. L'encapsulation n'est pas une simple question d'esthétique de code ; c'est une discipline d'ingénierie logicielle fondamentale pour construire des systèmes fiables.

L'encapsulation consiste à regrouper les données et les fonctions qui opèrent sur ces données en une seule unité, et à cacher les détails de l'implémentation interne à l'utilisateur de cette unité. L'utilisateur ne peut interagir avec la structure de données qu'à travers une interface publique bien définie. En empêchant l'accès direct aux données internes (comme les pointeurs `next` d'une liste ou les pointeurs `left`/`right` d'un arbre), on empêche l'utilisateur de les corrompre. Les fonctions de l'interface sont conçues pour être les seuls gardiens de l'intégrité de la structure.

### 4.3. Implémenter les Types de Données Abstraits (ADT) en C

En C, qui n'a pas de mots-clés `public` ou `private` comme en C++, l'encapsulation est réalisée par une convention stricte utilisant des fichiers d'en-tête et des fichiers source séparés. C'est ainsi que l'on crée un **Type de Données Abstrait** (ADT).

Le modèle se compose de deux parties principales :

#### Le Fichier d'En-tête (.h) : L'Interface Publique

Ce fichier est le "contrat" que le module offre au reste du programme. Il contient :

**Une déclaration de type opaque.** On utilise `typedef` pour déclarer un pointeur vers une structure dont la définition est cachée. L'utilisateur sait qu'il manipule un `List*` ou un `Tree*`, mais il ne connaît pas les champs de la `struct List` ou `struct Tree` sous-jacente.

```c
// Dans list.h
typedef struct List_t* List; 
```

**Les prototypes des fonctions publiques** qui composent l'interface de l'ADT. Ce sont les seules opérations autorisées sur le type opaque.

```c
// Dans list.h
List list_create(void);
void list_destroy(List list);
void list_push(List list, int data);
int list_pop(List list);
```

#### Le Fichier Source (.c) : L'Implémentation Privée

Ce fichier contient tous les détails internes, cachés à l'utilisateur.

**La définition complète de la structure.** C'est ici que l'on révèle les champs internes.

```c
// Dans list.c
#include "list.h" // On inclut notre propre en-tête

typedef struct Node_t {
    int data;
    struct Node_t* next;
} Node;

struct List_t {
    Node* head;
    int size;
};
```

**L'implémentation des fonctions publiques** déclarées dans le fichier .h.

**L'utilisation du mot-clé `static` pour les fonctions d'assistance.** Une fonction déclarée `static` dans un fichier .c n'est pas visible par l'éditeur de liens (linker) et ne peut donc pas être appelée depuis d'autres fichiers. Cela renforce l'encapsulation en garantissant que ces fonctions sont purement internes à l'implémentation.

Enfin, pour éviter les erreurs de compilation dues à des inclusions multiples d'un même fichier d'en-tête, on utilise des **gardes d'inclusion**. La méthode moderne est `#pragma once`, mais la méthode traditionnelle et portable est d'utiliser des directives de préprocesseur :

```c
// Dans mon_header.h
#ifndef MON_HEADER_H
#define MON_HEADER_H

//... contenu du header...

#endif // MON_HEADER_H
```

### 4.4. Synthèse Finale : La Véritable Nature d'une Structure de Données

L'aboutissement de cette discussion est une compréhension plus profonde de ce qu'est une structure de données. Une fois encapsulée, une structure n'est plus définie par ses détails d'implémentation (un tableau de listes chaînées, un ensemble de nœuds avec des couleurs...), mais par deux choses essentielles :

1. **Son contrat d'interface :** l'ensemble des opérations qu'elle permet d'effectuer.
2. **Ses garanties de performance :** la complexité algorithmique (en temps et en espace) de ces opérations.

Le choix d'une structure de données pour un problème donné se résume alors à comparer ces contrats. Le tableau suivant synthétise les caractéristiques des structures que nous avons étudiées, vues sous l'angle d'un ADT.

| Structure de Données | Insertion | Recherche (unique) | Requête par Intervalle | Cas d'Usage Typique |
|---------------------|-----------|-------------------|----------------------|-------------------|
| Liste Chaînée (ADT) | $O(1)$ | $O(N)$ | $O(N)$ | File d'attente simple, pile (LIFO/FIFO), où l'ordre d'insertion est primordial. |
| Arbre de Recherche Équilibré | $O(\log N)$ | $O(\log N)$ | $O(\log N + K)$ | Bases de données, indexation, recherche de données ordonnées, autocomplétion. |
| Table de Hachage (Chaînage) | $O(1)^*$ | $O(1)^*$ | $O(N)$ | Caches, dictionnaires, recherche rapide de données non ordonnées où l'ordre n'a pas d'importance. |

**Légende :** * indique la complexité en cas moyen.

Cette perspective abstraite est au cœur de l'ingénierie logicielle moderne. Elle permet de construire des systèmes complexes en assemblant des composants robustes et bien définis, sans avoir à se soucier de leurs détails internes, ce qui mène à un code plus maintenable, réutilisable et fiable.

## Partie V : Lectures Recommandées et Conclusion

Ce cours a exploré un cheminement allant des problèmes concrets aux solutions algorithmiques, puis des solutions algorithmiques à leur implémentation robuste en C. Nous avons vu le compromis fondamental entre les opérations qui préservent l'ordre (arbres) et celles qui le sacrifient pour la vitesse (tables de hachage). Nous avons compris l'importance des fondements mathématiques, comme le hachage universel, pour garantir la performance, et le rôle crucial de l'ingénierie logicielle, via l'encapsulation, pour garantir la fiabilité.

Pour approfondir ces sujets, les lectures suivantes sont fortement recommandées.

### Bibliographie Commentée

#### Ouvrages Fondamentaux

**Cormen, T. H., Leiserson, C. E., Rivest, R. L., & Stein, C.** *Introduction to Algorithms* (CLRS). La référence incontournable pour l'étude des algorithmes et des structures de données. Elle offre une analyse mathématique rigoureuse de tous les sujets abordés ici.

**Knuth, D. E.** *The Art of Computer Programming*. Une œuvre monumentale et d'une profondeur inégalée. Particulièrement le volume 3, "Sorting and Searching", pour une analyse exhaustive des arbres et du hachage.

**Kernighan, B. W., & Ritchie, D. M.** *The C Programming Language*. Le livre fondateur du langage C. Indispensable pour maîtriser les subtilités du langage, des pointeurs et de la gestion de la mémoire, qui sont essentielles pour implémenter correctement ces structures.

#### Articles de Recherche Cités

**Carter, L., & Wegman, M. N. (1979).** *Universal classes of hash functions*. L'article séminal qui a introduit le concept de hachage universel, jetant les bases théoriques pour des tables de hachage robustes face à des données malveillantes.

**Karp, R. M., & Rabin, M. O. (1987).** *Efficient randomized pattern-matching algorithms*. L'article qui a présenté l'algorithme de recherche de motifs basé sur le hachage roulant, une application élégante et puissante du hachage.

**Guibas, L. J., & Sedgewick, R. (1978).** *A dichromatic framework for balanced trees*. Cet article a établi le lien fondamental entre les arbres Rouge-Noir et les arbres 2-3-4, offrant un cadre conceptuel puissant pour comprendre les opérations d'équilibrage.

**Lemire, D., & Kaser, O. (2014).** *Strongly universal string hashing is fast*. Un article plus récent qui démontre que, sur les architectures modernes, les fonctions de hachage théoriquement plus complexes peuvent être plus rapides en pratique, remettant en question certaines optimisations traditionnelles.
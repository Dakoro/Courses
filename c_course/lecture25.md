# La Programmation Vectorielle Moderne en C : Des Fondations SIMD aux Algorithmes AVX-512 et au-delà

## Introduction : La Quête de la Performance à l'Ère Post-Fréquence

Bonjour à toutes et à tous. Dans le monde du calcul haute performance, nous avons longtemps bénéficié d'un "repas gratuit" : chaque nouvelle génération de processeurs augmentait sa fréquence d'horloge, rendant nos programmes plus rapides sans effort de notre part. Cette ère est révolue. Nous avons heurté ce que l'on appelle le "mur de la fréquence", où les gains de performance ne proviennent plus de la vitesse brute, mais de l'exploitation intelligente du parallélisme architectural.¹ Ce parallélisme se manifeste sous deux formes principales : le parallélisme de tâches, géré par les multiples cœurs de nos processeurs, et le parallélisme de données, qui est le sujet central de notre cours aujourd'hui.

Le parallélisme de données est incarné par un paradigme appelé SIMD, pour Single Instruction, Multiple Data. Le concept est d'une simplicité élégante : exécuter une seule instruction, mais l'appliquer simultanément à un grand nombre de données.² Imaginez une autoroute de données qui s'élargit progressivement, permettant de traiter non plus un seul véhicule à la fois, mais 4, 8, voire 16 en parallèle. C'est exactement ce que nos processeurs modernes peuvent faire.

L'objectif de cette conférence n'est pas simplement de vous faire mémoriser une liste de fonctions obscures. Il s'agit de vous amener à adopter une nouvelle manière de penser, ce que l'on pourrait appeler la "pensée SIMD" (SIMD thinking).² Vous apprendrez à décomposer les problèmes non plus en termes d'opérations sur des valeurs uniques (scalaires), mais en termes de transformations sur des ensembles de données (vecteurs). Comme nous le verrons, maîtriser cette approche permet de débloquer des gains de performance spectaculaires, souvent supérieurs à un ordre de grandeur, transformant des calculs lents en opérations quasi instantanées.²

## Partie I : Les Fondations Matérielles et Logicielles de la Vectorisation sur x86

### Chapitre 1 : L'Évolution des Registres Vectoriels

Pour comprendre la vectorisation, il faut d'abord comprendre l'outil qui la rend possible : les registres du processeur. Au fil des décennies, ces petites unités de mémoire ultra-rapides ont vu leur taille augmenter de manière spectaculaire pour accueillir de plus en plus de données.

#### De SSE (XMM) à AVX (YMM)

L'histoire moderne de la vectorisation sur l'architecture x86 commence véritablement avec l'introduction des extensions SSE (Streaming SIMD Extensions). Celles-ci ont apporté les registres XMM, larges de 128 bits.⁶ Un registre XMM peut contenir, par exemple, quatre nombres à virgule flottante simple précision (32 bits chacun) ou deux en double précision (64 bits).

Plus tard, l'extension AVX (Advanced Vector Extensions) a doublé cette capacité en introduisant les registres YMM de 256 bits.² Cela a permis de traiter huit flottants simple précision en une seule instruction, doublant ainsi le débit théorique pour de nombreuses applications scientifiques et multimédias.

#### AVX-512 (ZMM)

L'étape la plus récente et la plus significative de cette évolution est l'AVX-512. Comme son nom l'indique, cette extension introduit les registres ZMM, d'une largeur de 512 bits.² Mais l'AVX-512 est bien plus qu'un simple doublement de la largeur. Il s'agit d'une refonte majeure du jeu d'instructions SIMD, introduisant :

- Un nouveau schéma d'encodage des instructions (EVEX).
- Une augmentation du nombre de registres vectoriels, passant de 16 (en mode 64 bits) à 32.
- L'ajout de 8 registres de "masque" dédiés, un concept fondamental sur lequel nous reviendrons en détail.⁷

Initialement réservés aux serveurs et aux processeurs de calcul intensif, ces registres de 512 bits sont désormais disponibles sur des architectures grand public, comme les processeurs Intel Core de la génération "Tiger Lake".²

#### Anatomie d'un registre ZMM

Un seul registre ZMM de 512 bits peut contenir simultanément :

- 16 entiers de 32 bits (int ou float).
- 8 entiers de 64 bits (long long ou double).
- 32 entiers de 16 bits (short).
- 64 entiers de 8 bits (char ou uint8_t).

C'est cette capacité qui constitue la base matérielle nous permettant de traiter 16 opérations sur des entiers de 32 bits en une seule instruction, offrant un gain de performance théorique de 16x par rapport à un code scalaire.¹

#### Tableau 1 : Évolution des Architectures SIMD sur x86

| Caractéristique | SSE | AVX | AVX2 | AVX-512 |
|----------------|-----|-----|------|---------|
| Introduction | 1999 | 2011 | 2013 | 2016 |
| Largeur Registre | 128 bits | 256 bits | 256 bits | 512 bits |
| Nom Registre | XMM | YMM | YMM | ZMM |
| Nb. Registres (64b) | 16 | 16 | 16 | 32 |
| Élément Notable | Base du SIMD moderne | Format 3 opérandes | Opérations entières étendues | 32 registres, masques dédiés |

### Chapitre 2 : Le Modèle de Programmation : Intrinsèques et Compilation

Avoir des registres larges est une chose, mais comment les utiliser depuis un langage de haut niveau comme le C? La réponse réside dans les intrinsèques.

#### Introduction aux intrinsèques

Les intrinsèques sont des fonctions spéciales, généralement fournies par le compilateur, qui ont une correspondance quasi directe avec une instruction assembleur spécifique.² Elles représentent un compromis idéal : on bénéficie de la puissance et du contrôle de l'assembleur tout en restant dans le confort et la portabilité relative du langage C.

#### L'en-tête `<immintrin.h>`

Historiquement, il fallait inclure plusieurs fichiers d'en-tête (`<xmmintrin.h>`, `<emmintrin.h>`, etc.) pour accéder aux différentes générations d'instructions SIMD. Heureusement, la pratique moderne a été grandement simplifiée. Aujourd'hui, il suffit d'inclure un seul en-tête, `<immintrin.h>`, qui regroupe l'accès à l'ensemble des extensions, de SSE à AVX-512.²

#### Détection des capacités du CPU

Les extensions SIMD ne sont pas toujours présentes sur tous les processeurs. Un programme robuste doit donc vérifier leur disponibilité.

**Détection à l'exécution (Runtime) :** Pour créer un binaire portable qui s'adapte à la machine sur laquelle il s'exécute, on peut utiliser une fonction intégrée du compilateur comme `__builtin_cpu_support`. Par exemple, `__builtin_cpu_support("avx512f")` renverra vrai si le jeu d'instructions de base (Foundation) d'AVX-512 est disponible.² En coulisses, le compilateur génère un appel à l'instruction assembleur CPUID, qui interroge directement le processeur sur ses capacités.²

**Spécification à la compilation (Compile-time) :** Une autre approche consiste à informer le compilateur de la cible d'exécution.

- `-march=native` : Cette option demande au compilateur d'utiliser CPUID pendant la compilation pour détecter les capacités de la machine sur laquelle vous compilez et de générer un code optimisé spécifiquement pour elle.²
- `-march=tigerlake` (ou `-march=skylake-avx512`, etc.) : Cette option génère un binaire optimisé pour une microarchitecture spécifique. Le programme bénéficiera de toutes les instructions disponibles sur cette cible, mais ne fonctionnera pas sur un processeur plus ancien.²

#### Stratégies de lisibilité

Les noms des intrinsèques sont notoirement longs et peu intuitifs. Par exemple, `_mm512_set1_epi32` est la fonction qui remplit un vecteur de 16 entiers de 32 bits avec une seule valeur.² Un code non trivial devient rapidement illisible.

Pour pallier ce problème, une pratique courante et recommandée est de créer une fine couche d'abstraction (un "wrapper") avec des fonctions aux noms plus simples. Par exemple, nous pourrions encapsuler l'intrinsèque précédente dans une fonction `z_set_value`. Cette approche, que nous adopterons pour nos exemples, améliore considérablement la lisibilité et la maintenabilité du code sans aucun coût de performance, car ces appels de fonction sont "inlinés" par le compilateur.²

## Partie II : Techniques Fondamentales de Manipulation Vectorielle

Avant de nous attaquer à des algorithmes complexes, nous devons maîtriser les opérations de base qui sont les briques de construction de tout code vectorisé.

### Chapitre 3 : Mémoire et Données

#### Chargement et stockage

La première étape consiste à faire entrer les données de la mémoire principale dans les registres vectoriels. Pour cela, nous disposons de deux types d'instructions :

**Chargement aligné (`_mm512_load_si512`) :** Cette instruction est la plus rapide, mais elle exige que l'adresse mémoire de départ soit alignée sur une frontière de 64 octets (la taille d'un registre ZMM). Si cette condition n'est pas respectée, le programme plantera. On peut garantir cet alignement en C/C++ avec `alignas(64)`.

**Chargement non-aligné (`_mm512_loadu_si512`) :** Cette version est plus flexible et fonctionne avec n'importe quelle adresse mémoire. Elle est cependant légèrement moins performante, car le processeur doit effectuer un travail supplémentaire pour gérer l'accès à travers les lignes de cache.²

La même distinction existe pour les instructions de stockage (`_mm512_store_si512` et `_mm512_storeu_si512`) qui écrivent le contenu d'un registre vers la mémoire.

#### Diffusion (broadcast)

Une opération extrêmement courante est la diffusion, ou broadcast. Elle consiste à prendre une seule valeur scalaire et à la dupliquer dans chaque "voie" (lane) d'un registre vectoriel. L'intrinsèque `_mm512_set1_epi32` en est un parfait exemple.² Cette opération est fondamentale, notamment pour comparer tous les éléments d'un vecteur à une valeur de seuil unique.

### Chapitre 4 : Calculs et Logique Conditionnelle

#### Opérations arithmétiques et logiques

Le cœur du gain de performance SIMD réside dans les opérations arithmétiques et logiques qui s'exécutent en parallèle sur toutes les voies du vecteur. Des intrinsèques comme `_mm512_add_epi32` (addition), `_mm512_sub_epi32` (soustraction), ou `_mm512_and_si512` (ET au niveau du bit) effectuent 16 opérations en un seul cycle d'horloge.²

#### La puissance des masques

Comment implémenter une condition if-else en SIMD? La réponse réside dans les masques. Les instructions de comparaison vectorielle, comme `_mm512_cmpeq_epi32_mask`, ne retournent pas un vecteur de booléens. À la place, elles produisent un type spécial, un masque de bits (par exemple, un `__mmask16` pour 16 éléments de 32 bits).² Chaque bit de ce masque (0 ou 1) correspond directement au résultat de la comparaison pour une voie du vecteur. Un masque est donc une représentation compacte et efficace de 16 résultats de comparaison.

#### Le blend (fusion masquée)

Une fois que nous avons un masque, nous pouvons l'utiliser pour effectuer une logique conditionnelle sans branchement. C'est le rôle des instructions de fusion, ou blend, comme `_mm512_mask_blend_epi32`. Cette instruction prend en entrée un masque et deux vecteurs sources, A et B. Elle construit un vecteur de résultat en sélectionnant, pour chaque voie, l'élément de A si le bit correspondant du masque est à 0, et l'élément de B si le bit est à 1.²

Cette technique de "calculer les deux branches et sélectionner ensuite" est un changement de paradigme fondamental par rapport à la programmation scalaire. Le code scalaire est dominé par le flux de contrôle (if, else, for), qui génère des branchements pouvant entraîner des erreurs de prédiction et des arrêts coûteux dans le pipeline du processeur.² La programmation SIMD, en revanche, privilégie le flux de données : on transforme les données d'un état A à un état B via une série d'opérations vectorielles.¹¹ Cette approche transforme la question "que faire ensuite?" en "comment transformer cet ensemble de données?". C'est cette philosophie qui explique pourquoi des domaines comme le traitement d'image, le traitement du signal et l'algèbre linéaire ont été les premiers à adopter massivement le SIMD.³

### Chapitre 5 : Réorganisation des Données Intra-Registre

Souvent, les données ne sont pas dans le bon ordre dans le registre pour l'opération suivante. Les instructions de réorganisation sont donc cruciales.

#### Permutations (permute)

L'instruction de permutation est l'une des plus puissantes de l'arsenal AVX-512. L'intrinsèque `_mm512_permutexvar_epi32`, par exemple, prend un vecteur de données et un vecteur d'indices. Il retourne un nouveau vecteur où les éléments ont été réarrangés selon les indices fournis.² C'est un "shuffle" complet et arbitraire qui est la pierre angulaire des algorithmes de tri, des transformées de Fourier rapides (FFT) et de nombreuses autres manipulations complexes.

#### Rotations et décalages (rotate/shift)

Il n'existe pas d'intrinsèque unique pour effectuer une rotation ou un décalage sur l'ensemble des 16 éléments d'un vecteur. Cependant, nous pouvons construire ces opérations. Une rotation peut être implémentée en utilisant une instruction de permutation avec un vecteur d'indices calculé `((i + shift) % 16)`. Un décalage entre deux registres (shift pair), où les éléments qui sortent d'un registre entrent dans le suivant, peut être construit en utilisant des instructions de décalage au niveau des voies et des opérations de blend.²

## Partie III : Implémentation d'Algorithmes Vectorisés avec AVX-512

Armés de ces techniques de base, nous pouvons maintenant nous attaquer à des problèmes concrets et mesurer l'impact de la vectorisation.

### Chapitre 6 : Algorithme find : Recherche Linéaire Accélérée

L'algorithme de recherche d'un élément dans un tableau est un excellent premier exemple.

**Logique :** L'approche scalaire consiste à parcourir le tableau élément par élément. L'approche vectorisée traite le tableau par blocs de 16 entiers. À chaque itération, nous :

1. Chargeons 16 éléments du tableau dans un registre ZMM.
2. Diffusons la valeur recherchée dans un second registre ZMM.
3. Effectuons une comparaison vectorielle (`_mm512_cmpeq_epi32_mask`) pour obtenir un masque de 16 bits.
4. Si ce masque est différent de zéro, cela signifie qu'au moins une correspondance a été trouvée dans le bloc.
5. Nous utilisons alors une instruction comme `__builtin_ctz` (Count Trailing Zeros) sur le masque pour trouver l'indice du premier bit à 1. Cet indice, ajouté à la position de départ du bloc, nous donne l'index de l'élément trouvé.²

**Implémentation C :** Voici une ébauche de la fonction `find_simd`.

```c
#include <immintrin.h>
#include <stdint.h>

int find_simd(int* array, int n, int value) {
    // Crée un vecteur avec la valeur recherchée répliquée 16 fois.
    __m512i v_value = _mm512_set1_epi32(value);
    int i = 0;
    // Boucle principale, traitant 16 éléments à la fois.
    for (; i <= n - 16; i += 16) {
        // Charge 16 entiers non-alignés depuis le tableau.
        __m512i v_data = _mm512_loadu_si512(&array[i]);
        // Compare les deux vecteurs et génère un masque.
        __mmask16 mask = _mm512_cmpeq_epi32_mask(v_data, v_value);
        // Si le masque n'est pas nul, une correspondance a été trouvée.
        if (mask != 0) {
            // Trouve l'index du premier bit à 1 dans le masque.
            return i + __builtin_ctz(mask);
        }
    }
    // Gère les éléments restants (la "queue").
    for (; i < n; ++i) {
        if (array[i] == value) {
            return i;
        }
    }
    return -1;
}
```

**Performance :** Dans l'exemple de la transcription, cette approche a permis de passer de 1.2 seconde à 0.07 seconde, soit une accélération de près de 20x.² Ce gain, supérieur au facteur théorique de 16x, s'explique par ce que nous pouvons appeler le "dividende de la prédiction de branchement". Le code scalaire contient une boucle avec une instruction if à chaque itération. Sur des données non triées, le processeur ne peut pas prédire efficacement si la branche sera prise ou non, ce qui entraîne des erreurs de prédiction coûteuses. Le code SIMD, étant sans branchement, élimine complètement ce problème, d'où le gain de performance supplémentaire.²

### Chapitre 7 : Algorithme argmin : Trouver l'Index du Minimum

Trouver l'index de la valeur minimale est un problème de réduction plus complexe.

**Logique :** La stratégie vectorisée consiste à maintenir 16 "minimums" en parallèle, un pour chaque voie du vecteur. Nous avons besoin de trois registres :

- `current_mins` : Contient les 16 valeurs minimales trouvées jusqu'à présent pour chaque voie. Initialisé à INT_MAX.
- `current_argmins` : Contient les indices correspondants aux valeurs dans `current_mins`.
- `current_indices` : Un vecteur qui contient les indices actuels (i, i+1,..., i+15).

À chaque itération, nous chargeons 16 nouvelles valeurs. Nous les comparons à `current_mins`. La comparaison produit un masque qui indique où les nouvelles valeurs sont plus petites. Ce masque est ensuite utilisé avec une instruction blend pour mettre à jour `current_argmins` uniquement pour les voies où un nouveau minimum a été trouvé. Enfin, `current_mins` est mis à jour. À la toute fin, une boucle scalaire rapide sur les 16 paires (valeur, index) des registres finaux permet de déterminer le vrai minimum global.²

**Implémentation C :**

```c
#include <immintrin.h>
#include <limits.h>

int argmin_simd(int* array, int n) {
    __m512i current_mins = _mm512_set1_epi32(INT_MAX);
    __m512i current_argmins = _mm512_setzero_si512();
    __m512i current_indices = _mm512_set_epi32(15, 14, 13, 12, 11, 10, 9, 8, 7, 6, 5, 4, 3, 2, 1, 0);
    const __m512i step = _mm512_set1_epi32(16);

    int i = 0;
    for (; i <= n - 16; i += 16) {
        __m512i data = _mm512_loadu_si512(&array[i]);
        // Masque où data < current_mins
        __mmask16 mask = _mm512_cmplt_epi32_mask(data, current_mins);
        
        // Met à jour les minimums et les indices en utilisant le masque.
        current_mins = _mm512_mask_min_epi32(current_mins, mask, data, current_mins);
        current_argmins = _mm512_mask_mov_epi32(current_argmins, mask, current_indices);
        
        current_indices = _mm512_add_epi32(current_indices, step);
    }

    // Réduction scalaire finale
    int min_vals[16];
    int argmin_vals[16];
    _mm512_storeu_si512(min_vals, current_mins);
    _mm512_storeu_si512(argmin_vals, current_argmins);

    int final_min = INT_MAX;
    int final_argmin = -1;
    for(int j = 0; j < 16; ++j) {
        if (min_vals[j] < final_min) {
            final_min = min_vals[j];
            final_argmin = argmin_vals[j];
        }
    }
    
    // Gérer la queue
    for (; i < n; ++i) {
        if (array[i] < final_min) {
            final_min = array[i];
            final_argmin = i;
        }
    }

    return final_argmin;
}
```

**Performance :** La transcription rapporte une accélération stupéfiante de près de 50x (de 4 secondes à 0.07 seconde).² La raison est identique à celle de find, mais amplifiée. L'instruction `if (a[i] < a[k])` sur des données aléatoires est l'un des pires scénarios pour le prédicteur de branchement du processeur. L'élimination totale de cette branche dans la boucle principale produit des gains de performance massifs.²

### Chapitre 8 : Algorithme copy_if_less : Filtrage et Compression de Données

Cette opération, également connue sous le nom de "filtrage" ou "stream compaction", consiste à copier les éléments d'un tableau qui satisfont une condition dans un autre tableau.

**Logique :** L'approche la plus élégante avec AVX-512 utilise une instruction de "compression".

1. On charge un bloc de 16 éléments.
2. On les compare à une valeur seuil pour générer un masque (`__mmask16`).
3. On utilise l'intrinsèque `_mm512_mask_compressstoreu_epi32`. Cette instruction prend le masque, le vecteur de données et une adresse mémoire de destination. Elle parcourt le vecteur et écrit en mémoire, de manière contiguë, uniquement les éléments pour lesquels le bit correspondant dans le masque est à 1.¹⁴

Le nombre d'éléments écrits peut être trouvé en comptant le nombre de bits à 1 dans le masque (`_mm_popcnt_u32`), ce qui permet de mettre à jour le pointeur de destination pour la prochaine itération.

**Implémentation C :**

```c
#include <immintrin.h>
#include <stdint.h>

int copy_if_less_simd(int* dest, int* src, int n, int threshold) {
    __m512i v_threshold = _mm512_set1_epi32(threshold);
    int* dest_ptr = dest;
    
    for (int i = 0; i <= n - 16; i += 16) {
        __m512i data = _mm512_loadu_si512(&src[i]);
        // Masque où data < threshold
        __mmask16 mask = _mm512_cmplt_epi32_mask(data, v_threshold);
        
        // Stocke de manière compressée les éléments valides.
        _mm512_mask_compressstoreu_epi32(dest_ptr, mask, data);
        
        // Avance le pointeur de destination.
        dest_ptr += _mm_popcnt_u32(mask);
    }
    
    // Gérer la queue...
    //...

    return dest_ptr - dest; // Retourne le nombre d'éléments copiés.
}
```

**Performance :** Cette approche est extrêmement efficace car elle délègue tout le travail complexe de sélection et d'écriture contiguë à une seule instruction matérielle, éliminant toute boucle ou logique conditionnelle côté logiciel. La transcription mentionne une alternative utilisant des tables de permutation précalculées, qui est également très rapide mais consomme une quantité de mémoire prohibitive (4 Mo pour des entiers de 32 bits), bien que réductible en utilisant des registres plus petits (8 Ko pour des registres YMM de 256 bits).²

### Chapitre 9 : Réseaux de Tri : Le Tri Bionique en Registre

Trier un petit nombre d'éléments (par exemple, les 16 entiers contenus dans un registre ZMM) est une sous-routine clé pour de nombreux algorithmes plus complexes.

**Logique :** Un réseau de tri, comme le tri bionique, est une séquence fixe d'opérations de comparaison-échange, indépendante des données d'entrée. Cela le rend parfaitement adapté au modèle SIMD. Pour trier 16 entiers dans un registre, nous implémentons chaque étape du réseau de tri. Une étape de comparaison-échange se traduit par une séquence de 3 opérations SIMD :

1. **Permutation :** On utilise `_mm512_permutexvar_epi32` avec un vecteur d'indices fixe pour amener les éléments à comparer côte à côte.
2. **Min/Max :** On calcule `_mm512_min_epi32` et `_mm512_max_epi32` entre le vecteur original et sa version permutée.
3. **Blend :** On utilise `_mm512_mask_blend_epi32` avec un masque fixe (par exemple, 0xAAAA pour prendre alternativement un élément de chaque vecteur) pour fusionner les minimums et les maximums dans le bon ordre.²

**Implémentation C :** Le code complet pour un tri bionique de 16 éléments est une séquence de ces blocs de 3 instructions, avec des vecteurs d'indices et des masques de fusion pré-calculés qui correspondent à la topologie du réseau.¹⁸

```c
// Ébauche conceptuelle d'une étape du tri bionique
__m512i compare_exchange(__m512i data, __m512i perm_indices, __mmask16 blend_mask) {
    __m512i permuted_data = _mm512_permutexvar_epi32(perm_indices, data);
    __m512i mins = _mm512_min_epi32(data, permuted_data);
    __m512i maxs = _mm512_max_epi32(data, permuted_data);
    return _mm512_mask_blend_epi32(blend_mask, mins, maxs);
}
```

**Performance :** C'est la méthode la plus rapide connue pour trier un petit nombre fixe d'éléments, car elle s'exécute entièrement dans les registres et est totalement dépourvue de branchements.

### Chapitre 10 : Application Pratique : Le Filtre Médian pour le Traitement de Signal

Le filtre médian est un outil classique de traitement du signal et d'image pour éliminer le bruit.

**Logique :** Il remplace chaque point d'un signal par la médiane de ses voisins (par exemple, une fenêtre de 7 points : le point lui-même, les 3 précédents et les 3 suivants). La médiane est, par définition, l'élément central d'un ensemble trié. L'approche vectorisée est donc une application directe de notre réseau de tri en registre :

1. Pour chaque point du signal, on charge sa fenêtre de 7 voisins dans un registre ZMM.
2. On applique notre fonction de tri en registre pour trier ces 7 valeurs.
3. On extrait l'élément médian (le 4ème élément) et on l'écrit dans le signal de sortie.

Pour traiter un signal entier, on utilise une approche de "fenêtre glissante", en utilisant des instructions de décalage de registre et de chargement partiel pour mettre à jour la fenêtre de manière efficace à chaque pas, minimisant ainsi les accès mémoire.²

**Performance :** Cette méthode est bien plus rapide qu'une approche scalaire, qui nécessiterait un appel à une fonction de tri (comme qsort) ou un tri par insertion pour chaque point du signal, ce qui serait prohibitif en termes de coût de calcul.¹⁹

#### Tableau 2 : Synthèse des Gains de Performance des Algorithmes Vectorisés

| Algorithme | Temps Scalaire (s) | Temps Vectorisé (s) | Facteur d'Accélération |
|------------|-------------------|---------------------|------------------------|
| find | 1.2 | 0.07 | ~17x |
| argmin | 4.0 | 0.07 | ~57x |

*Données basées sur les benchmarks présentés dans la transcription source.²*

## Partie IV : Recherches Récentes et Avenir de la Vectorisation CPU

La vectorisation ne s'arrête pas à l'AVX-512. Le paysage est en constante évolution, avec de nouvelles architectures qui cherchent à simplifier et à unifier la programmation parallèle.

### Chapitre 11 : L'ISA Convergé : Intel AVX10, le Successeur d'AVX-512

#### La fragmentation d'AVX-512

L'un des plus grands freins à l'adoption de l'AVX-512 a été sa complexité. L'AVX-512 n'est pas un jeu d'instructions monolithique, mais une collection de dizaines de sous-ensembles optionnels, chacun identifié par un "drapeau de fonctionnalité" (feature flag) : F (Foundation), CD (Conflict Detection), BW (Byte/Word), VNNI (Vector Neural Network Instructions), etc.⁷ Un processeur pouvait en supporter certains mais pas d'autres, obligeant les développeurs à écrire de multiples chemins de code et à effectuer des vérifications CPUID complexes pour chaque fonctionnalité, un véritable casse-tête.²¹

#### La promesse d'AVX10

Intel AVX10 est la réponse d'Intel à ce problème. Il s'agit d'un "ISA Convergé" (Converged ISA) qui vise à unifier et simplifier l'écosystème vectoriel x86.²¹ Le changement le plus important pour les développeurs est l'introduction d'une énumération par version. Au lieu de vérifier des dizaines de drapeaux, un programme vérifiera une seule version d'AVX10, par exemple AVX10.1 ou AVX10.2. Chaque nouvelle version garantit l'inclusion de toutes les fonctionnalités des versions précédentes, simplifiant radicalement la détection des capacités et assurant une base de fonctionnalités commune et croissante.²¹

#### Analyse de la décision stratégique : le support 512 bits obligatoire

Initialement, AVX10 prévoyait une approche hybride où les cœurs à haute efficacité (E-cores) seraient limités à des vecteurs de 256 bits, tandis que les cœurs performants (P-cores) supporteraient en option les 512 bits.²² Cette approche a suscité des critiques de la part des développeurs, qui y voyaient une nouvelle source de complexité.

Face à ces retours et à la stratégie de son concurrent AMD, qui supporte l'AVX-512 sur tous ses cœurs depuis l'architecture Zen 4, Intel a opéré un revirement stratégique. Les futures versions d'AVX10 (à partir de AVX10.2) rendront le support des vecteurs de 512 bits obligatoire sur tous les cœurs, y compris les E-cores.²⁵ Cette décision est une excellente nouvelle : elle garantit une performance et une compatibilité uniformes sur l'ensemble du processeur, simplifiant la vie des développeurs et assurant la pérennité de l'écosystème 512 bits.

Enfin, la compatibilité ascendante est assurée : le code AVX-512 existant fonctionnera sans aucune modification sur le matériel AVX10.²⁶

### Chapitre 12 : Un Paradigme Alternatif : La Longueur de Vecteur Scalable d'ARM SVE2

Pendant qu'Intel unifie sa largeur de vecteur fixe, ARM a adopté une philosophie radicalement différente avec ses extensions SVE (Scalable Vector Extension) et SVE2.

**Philosophie de conception :** La caractéristique distinctive de SVE/SVE2 est la longueur de vecteur scalable. Au lieu de fixer la largeur dans le jeu d'instructions (128, 256 ou 512 bits), l'ISA permet aux concepteurs de puces de choisir une largeur matérielle allant de 128 à 2048 bits, par incréments de 128 bits.¹⁰

**Programmation agnostique à la longueur :** Le développeur écrit un seul binaire. Ce binaire s'exécute ensuite de manière optimale sur n'importe quel matériel compatible SVE, quelle que soit sa largeur de vecteur. Cela est rendu possible par un modèle de programmation où le code ne fait aucune supposition sur la largeur. Les boucles sont écrites à l'aide de prédicats et d'instructions spéciales qui s'adaptent dynamiquement à la taille des registres disponibles au moment de l'exécution.³¹ C'est une approche "écrire une fois, exécuter (et scaler) partout".

#### Tableau 3 : Comparaison Philosophique : AVX-512/AVX10 vs. ARM SVE2

| Caractéristique | Intel AVX-512/AVX10 | ARM SVE2 |
|-----------------|---------------------|----------|
| Philosophie | Largeur de vecteur fixe et explicite | Largeur de vecteur scalable et agnostique |
| Largeur de Vecteur | Fixée par l'ISA (ex: 512 bits) | Variable (128 à 2048 bits), choisie par le fabricant |
| Modèle de Prog. | Le code est écrit pour une largeur spécifique | Le code est agnostique à la largeur |
| Portabilité Binaire | Nécessite des chemins de code distincts ou recompilation | Un seul binaire s'adapte à toutes les largeurs |
| Contrôle Développeur | Contrôle total et explicite de la disposition des données | Moins de contrôle de bas niveau, abstraction de la largeur |

### Chapitre 13 : La Vectorisation en Action : Études de Cas Modernes

Loin d'être une simple curiosité académique, la vectorisation est au cœur des applications les plus exigeantes d'aujourd'hui. La distinction entre les fonctionnalités "HPC" et "grand public" s'estompe ; les capacités de traitement parallèle de données, autrefois de niche, sont désormais essentielles pour accélérer l'IA, l'analyse de données et même les applications grand public.⁹ AVX10 est la manifestation architecturale de cette réalité du marché.

**Génomique :** Des recherches récentes ont montré que l'utilisation de l'AVX-512 pour accélérer les algorithmes d'alignement de séquences d'ADN (comme BWA-MEM et KSW2) permet d'obtenir des gains de performance allant jusqu'à 2.84x par rapport à l'AVX2 sur les processeurs modernes, ce qui est crucial pour analyser les volumes massifs de données génomiques.³³

**Bases de données vectorielles :** Des systèmes comme OpenSearch, qui sont au cœur de nombreuses applications d'IA et de recherche sémantique, utilisent l'AVX-512 pour accélérer les calculs de similarité entre vecteurs. Les benchmarks montrent des gains de 10 à 15% sur l'indexation et la recherche.³⁴ L'instruction native `_mm512_popcnt_epi64`, par exemple, accélère massivement le calcul de la distance de Hamming, une opération fondamentale pour les vecteurs binaires.³⁵

**Mythes et réalités sur l'efficacité énergétique :** La mauvaise réputation initiale de l'AVX-512 en matière de consommation d'énergie provenait des premières implémentations (comme Skylake-X) qui réduisaient agressivement la fréquence du processeur lors de l'utilisation d'instructions 512 bits.³⁶ Les architectures modernes (depuis Tiger Lake chez Intel et Zen 4 chez AMD) ont largement résolu ce problème. En réalité, pour une charge de travail donnée, terminer une tâche deux fois plus vite avec l'AVX-512 peut consommer globalement moins d'énergie que de la faire tourner deux fois plus longtemps avec l'AVX2.³⁶

## Conclusion : Synthèse et Perspectives

Nous avons parcouru un long chemin, des concepts de base du SIMD aux algorithmes les plus sophistiqués et aux architectures de demain. Si vous ne devez retenir qu'une chose, c'est que la "pensée SIMD" est une compétence essentielle pour le développeur de logiciels performants. Elle nous oblige à repenser nos algorithmes en termes de flux de données, à éliminer les branchements conditionnels au profit d'opérations masquées, et à structurer nos données pour un accès efficace.

Bien que les compilateurs modernes fassent des progrès remarquables en matière d'auto-vectorisation, ils échouent encore à optimiser des schémas de code complexes.³⁷ Pour atteindre le sommet de la performance, la vectorisation manuelle via les intrinsèques reste, et restera pour un certain temps, une technique indispensable.

L'avenir est résolument parallèle. Avec AVX10 qui unifie et simplifie l'écosystème x86, et SVE2 qui offre une alternative portable et pérenne dans le monde ARM, le parallélisme de données est plus pertinent et accessible que jamais. Les prochains grands sauts de performance dans nos applications viendront de notre capacité à maîtriser ces puissantes architectures. J'espère que cette conférence vous a donné les clés pour commencer ce voyage passionnant. Merci.
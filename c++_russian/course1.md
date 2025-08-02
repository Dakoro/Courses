# Cours C++ de base - MIPT 2021-2022
**Professeur : Konstantin Vladimirov**

## Introduction

Ce cours s'adresse aux étudiants qui connaissent déjà le langage C. Nous allons apprendre le C++ non pas comme un langage isolé, mais à travers des problèmes pratiques concrets. Nous commencerons par un problème que nous avons abordé en première année : la gestion des caches.

## Problème pratique : Les caches

### Énoncé du problème

Vous développez un navigateur web qui doit gérer des millions de pages web de tailles différentes, chacune identifiée par un numéro unique. La seule façon d'obtenir une page est via le réseau avec une fonction lente `slow_get_page()`. 

**Contraintes :**
- L'utilisateur travaille généralement avec un ensemble limité de pages
- Vous disposez d'un espace disque limité (environ 100 pages)
- Vous ne pouvez pas stocker tout Internet

**Solution :** Implémenter un système de cache.

### Stratégie de cache LRU (Least Recently Used)

La stratégie LRU évince toujours la page utilisée le moins récemment. 

**Exemple de fonctionnement :**
```
Pages demandées : 1, 2, 3, 4, 1, 2, 5
Cache (taille 4) : 
- Après 1,2,3,4 : [4,3,2,1]
- Après demande de 1 : [1,4,3,2]
- Après demande de 2 : [2,1,4,3]
- Après demande de 5 : [5,2,1,4] (3 évincé)
```

### Structures de données nécessaires

1. **Liste doublement chaînée** : Pour maintenir l'ordre LRU (O(1) pour déplacer en tête)
2. **Table de hachage** : Pour vérifier rapidement si une page est en cache (O(1) en moyenne)

### Algorithme LRU

```
Pour chaque requête de page n :
    Si n est dans le cache :
        Déplacer le nœud correspondant en tête de liste
    Sinon :
        Si le cache est plein :
            Supprimer le dernier élément de la liste
        Obtenir la page via slow_get_page(n)
        Ajouter la nouvelle page en tête de liste
```

## Implémentation en C : Les problèmes

### Code nécessaire en C

Pour implémenter un cache LRU en C, vous devez écrire :

1. **Module liste doublement chaînée**
   ```c
   // Structure cachée dans le .c
   struct node {
       struct node *next;
       struct node *prev;
       // données...
   };
   
   // Interface publique
   list* list_create();
   void list_push_front(list* l, void* data);
   void list_move_to_front(list* l, node* n);
   void list_free(list* l);
   ```

2. **Module table de hachage**
   ```c
   typedef struct hash_table {
       struct hash_node **table;  // Tableau de pointeurs
       size_t len;
   } hash_table;
   ```

3. **Module cache**
   ```c
   typedef struct page* (*slow_get_page_t)(int n);
   
   void cache_lookup_update(cache* c, int n) {
       node* ptr = table_find(c->table, n);
       if (!ptr) {
           if (cache_full(c)) {
               // Nettoyer...
           }
           page* p = c->slow_get_page(n);
           // Ajouter au cache...
       } else {
           list_move_to_front(c->list, ptr);
       }
   }
   ```

### Problèmes de cette approche

1. **Réinvention de la roue** : Pas de structures de données standard en C
2. **Non-générique** : Le code est spécifique à un type de données
3. **Gestion manuelle de la mémoire** : Source d'erreurs
4. **Performance sous-optimale** : `qsort` ne peut pas inliner le comparateur

## La solution C++

### Principe 1 : Union des données et des méthodes

Au lieu de fonctions séparées, C++ permet d'associer les méthodes aux données :

```cpp
struct Triangle {
    Point pts[3];
    
    double area() const {
        // Accès direct aux membres
        return abs((pts[1].x - pts[0].x) * (pts[2].y - pts[0].y) -
                   (pts[2].x - pts[0].x) * (pts[1].y - pts[0].y)) / 2.0;
    }
};

// Utilisation
Triangle t;
double a = t.area();  // Appel de méthode
```

**Le pointeur `this` :**
- Chaque méthode reçoit implicitement un pointeur vers l'objet (`this`)
- Pas de surcoût par rapport au C (même code assembleur généré)

### Principe 2 : Généricité (Templates)

Les templates permettent d'écrire du code générique :

```cpp
template<typename T>
struct Point {
    T x, y;
};

template<typename T>
struct Triangle {
    Point<T> pts[3];
    
    T double_area() const {
        return (pts[1].x - pts[0].x) * (pts[2].y - pts[0].y) -
               (pts[2].x - pts[0].x) * (pts[1].y - pts[0].y);
    }
};

// Utilisation
Triangle<int> t_int;
Triangle<double> t_double;
```

### Avantages de la généricité

**Comparaison avec les macros C :**

```c
// C - Problématique
#define MAX(a,b) ((a) > (b) ? (a) : (b))
// Problèmes : évaluation double, pas de vérification de type

// C++ - Sûr et efficace
template<typename T>
T max(T a, T b) {
    return a > b ? a : b;
}
```

**Performance :** Exemple avec le tri

```cpp
// C : qsort ne peut pas inliner le comparateur
qsort(arr, n, sizeof(int), compare);  // ~5.5 secondes

// C++ : std::sort inline le comparateur
std::sort(arr, arr + n);              // ~2.3 secondes
```

Performance améliorée de **plus de 50%** grâce à l'inlining !

## Implémentation du cache LRU en C++

### Version C++ simplifiée

```cpp
template<typename K, typename V>
class LRUCache {
    struct Entry {
        K key;
        V value;
    };
    
    std::list<Entry> cache_list;
    std::unordered_map<K, typename std::list<Entry>::iterator> cache_map;
    size_t capacity;
    
public:
    LRUCache(size_t cap) : capacity(cap) {}
    
    V* lookup_update(K key, std::function<V(K)> slow_get) {
        auto it = cache_map.find(key);
        
        if (it != cache_map.end()) {
            // Hit : déplacer en tête
            cache_list.splice(cache_list.begin(), cache_list, it->second);
            return &it->second->value;
        }
        
        // Miss
        if (cache_list.size() >= capacity) {
            cache_map.erase(cache_list.back().key);
            cache_list.pop_back();
        }
        
        V value = slow_get(key);
        cache_list.push_front({key, value});
        cache_map[key] = cache_list.begin();
        
        return &cache_list.front().value;
    }
};
```

### Avantages de la version C++

1. **Code plus court et plus clair**
2. **Générique** : Fonctionne avec n'importe quel type
3. **Bibliothèque standard** : `std::list` et `std::unordered_map` déjà testés
4. **Sémantique de valeur** : Gestion automatique de la mémoire
5. **Performance** : Templates permettent l'optimisation

## Concepts clés introduits

### Constructeurs

Les constructeurs garantissent l'initialisation correcte :

```cpp
LRUCache(size_t cap) : capacity(cap) {}
// Impossible de créer un cache sans spécifier la capacité
```

### Méthodes const

Les méthodes qui ne modifient pas l'objet doivent être marquées `const` :

```cpp
double area() const {  // Cette méthode ne modifie pas le triangle
    // ...
}
```

### Conventions de nommage

- **Évitez** les underscores au début (`_var`, `__var`) - réservés
- **Préférez** les underscores à la fin pour les membres privés (`capacity_`)

## Travail pratique

### Exercice : Cache LRU

Implémenter un cache LRU en C++ avec :
- Support générique pour tout type de clé/valeur
- Tests unitaires complets
- Comparaison de performance avec une implémentation "idéale"

**Format d'entrée/sortie :**
```
Entrée : m n    // m = taille cache, n = nombre requêtes
         id1 id2 ... idn
Sortie : hits misses
```

### Recommandations

1. **Utilisez Git** pour le contrôle de version
2. **CMake** pour la compilation
3. **Tests unitaires** obligatoires
4. **Qualité du code** : noms clairs, const-correctness

## Ressources

- **Standard C++** : Document de référence officiel
- **Stroustrup** : "The C++ Programming Language" - Introduction complète
- **Lippman** : "C++ Primer" - Approche plus moderne

## Points clés à retenir

1. C++ améliore C en permettant l'**abstraction sans perte de performance**
2. Les **templates** permettent du code générique type-safe
3. La **bibliothèque standard** fournit des structures de données efficaces
4. L'**encapsulation** (données + méthodes) améliore la maintenabilité
5. Les **constructeurs** garantissent l'initialisation correcte

Le C++ est un langage complexe qui nécessite du temps pour être maîtrisé. Ce cours pose les bases, mais la vraie compétence vient avec la pratique continue.
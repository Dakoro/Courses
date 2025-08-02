# Cours : Multithreading et Architecture Multi-cœurs dans les Processeurs Modernes

## Introduction

Bienvenue dans ce cours complet sur le cinquième et dernier multiplicateur de performance : le multithreading. Nous allons explorer comment les processeurs modernes, équipés de multiples cœurs, peuvent exécuter plusieurs flux d'instructions simultanément, et comment exploiter cette capacité peut transformer radicalement les performances de nos programmes.

## Partie 1 : Comprendre les Cœurs et le Multithreading

### 1.1 Qu'est-ce qu'un Cœur ?

Un **cœur** (core) est une unité de traitement complète et indépendante au sein d'un processeur. Chaque cœur peut exécuter son propre flux d'instructions de manière autonome. Dans les processeurs modernes, un seul boîtier physique peut contenir :

- Plusieurs puces interconnectées
- Plusieurs cœurs par puce
- Chaque cœur étant l'équivalent d'un processeur complet des générations précédentes

### 1.2 Le Principe du Multithreading

Le multithreading repose sur une idée simple : si un ordinateur peut effectuer une tâche à une certaine vitesse, deux ordinateurs devraient pouvoir l'effectuer plus rapidement. Idéalement, avec deux ordinateurs, on double la vitesse.

**Différences avec les autres formes de parallélisme :**
- **ILP** : Extrait le parallélisme d'un flux d'instructions unique
- **SIMD** : Extrait le parallélisme d'un flux de données
- **Multithreading** : Requiert que le programmeur crée explicitement plusieurs flux d'instructions

### 1.3 L'Évolution Technologique

L'évolution de la technologie de fabrication des puces a permis :
- Années 2000 : 2 cœurs par processeur
- Aujourd'hui (grand public) : Minimum 4 cœurs, souvent 8-16 cœurs
- Serveurs actuels : Jusqu'à 96 cœurs par processeur !

## Partie 2 : Threads vs Cœurs - Concepts Fondamentaux

### 2.1 La Distinction Importante

- **Cœurs** : Concept physique - unités matérielles qui exécutent les instructions
- **Threads** : Concept logiciel - flux d'instructions gérés par le système d'exploitation

Le système d'exploitation assigne les threads aux cœurs physiques disponibles.

### 2.2 Modèle d'Interaction

```pseudocode
Algorithme : ModèleThreadsCœurs
Entrée : Programme avec n threads
Sortie : Exécution parallèle sur c cœurs

Fonction ExécuterProgrammeMultithread() :
    threads[] ← CréerThreads(n)
    cœurs_disponibles ← ObtenirNombreCœurs()
    
    pour chaque thread dans threads :
        cœur_assigné ← OS.AssignerCœur(thread)
        cœur_assigné.Exécuter(thread)
    
    AttendreFinThreads(threads)
    retourner CombinerRésultats(threads)
```

## Partie 3 : Parallélisation d'une Boucle de Sommation

### 3.1 Décomposition du Problème

Notre boucle de sommation est idéale pour le multithreading car l'addition est associative et commutative :

```pseudocode
Algorithme : SommationMultithread
Entrée : Tableau A de n entiers, nombre de threads t
Sortie : Somme de tous les éléments

Fonction SommeParallèle(A, n, t) :
    taille_bloc ← n / t
    threads[] ← nouveau Tableau[t]
    
    // Créer et lancer les threads
    pour i ← 0 à t-1 :
        début ← i * taille_bloc
        fin ← min((i + 1) * taille_bloc, n)
        
        threads[i] ← CréerThread(SommePartielle, A, début, fin)
        threads[i].Démarrer()
    
    // Attendre la fin et combiner les résultats
    somme_totale ← 0
    pour i ← 0 à t-1 :
        somme_partielle ← threads[i].AttendreRésultat()
        somme_totale ← somme_totale + somme_partielle
    
    retourner somme_totale

Fonction SommePartielle(A, début, fin) :
    somme ← 0
    pour i ← début à fin-1 :
        somme ← somme + A[i]
    retourner somme
```

### 3.2 Version Optimisée avec SIMD

Pour maximiser les performances, chaque thread peut utiliser les optimisations SIMD :

```pseudocode
Fonction SommePartielleAVX(A, début, fin) :
    // Utilisation de registres AVX 256 bits (8 entiers de 32 bits)
    sommes_vect[4] ← [vecteur_zéro, vecteur_zéro, vecteur_zéro, vecteur_zéro]
    
    // Traiter par blocs de 32 entiers
    pour i ← début à fin-1 pas 32 :
        pour j ← 0 à 3 :
            données ← AVX_Charger(A[i + j*8 : i + j*8 + 7])
            sommes_vect[j] ← AVX_Addition(sommes_vect[j], données)
    
    // Réduction finale
    somme_finale ← 0
    pour j ← 0 à 3 :
        somme_finale ← somme_finale + RéductionHorizontale(sommes_vect[j])
    
    retourner somme_finale
```

## Partie 4 : Analyse des Performances Multi-cœurs

### 4.1 Résultats Expérimentaux

Analysons les performances observées avec différents nombres de threads :

**Configuration de test : 4096 entiers (16 KB)**

| Threads | Performance (add/cycle) | Accélération |
|---------|------------------------|---------------|
| 1       | 13.4                   | 1.0x          |
| 2       | 22.6                   | 1.7x          |
| 4       | 35.0                   | 2.6x          |

### 4.2 Analyse de l'Efficacité

```pseudocode
Algorithme : CalculEfficacitéParallèle
Entrée : perf_1_thread, perf_n_threads, n
Sortie : Efficacité du parallélisme

Fonction CalculerEfficacité(perf_1, perf_n, n) :
    accélération_idéale ← n
    accélération_réelle ← perf_n / perf_1
    efficacité ← accélération_réelle / accélération_idéale
    
    retourner efficacité * 100  // En pourcentage
```

Pour 4 threads : Efficacité = 2.6 / 4.0 = 65%

### 4.3 Facteurs Limitants

Les raisons pour lesquelles nous n'atteignons pas l'accélération idéale :
1. **Surcharge de synchronisation** : Création et gestion des threads
2. **Taille de travail réduite** : Seulement 1024 entiers par thread
3. **Contention de ressources** : Partage potentiel de certaines ressources

## Partie 5 : L'Effet Cache dans le Multithreading

### 5.1 Le Phénomène de Super-Accélération

Lorsque nous augmentons la taille des données, un phénomène surprenant se produit :

**Test avec 16 384 entiers (64 KB) :**

| Configuration | Performance | Commentaire |
|--------------|------------|-------------|
| 1 thread     | 7.0 add/cycle | Limité par L2 (données > L1) |
| 4 threads    | 52.0 add/cycle | 4 × L1 disponibles ! |

Accélération observée : **7.4x** avec seulement 4 threads !

### 5.2 Explication du Phénomène

```pseudocode
Algorithme : AnalyseCacheMultithread
Entrée : taille_données, nombre_threads, taille_L1_par_cœur
Sortie : Configuration cache effective

Fonction AnalyserCapacitéCache(taille_données, threads, L1_size) :
    cache_L1_total ← threads * L1_size
    
    si taille_données ≤ L1_size :
        // Un seul thread peut tout mettre en L1
        retourner "Optimal mono-thread"
    sinon si taille_données ≤ cache_L1_total :
        // Nécessite plusieurs threads pour tout mettre en L1
        retourner "Optimal multi-thread uniquement"
    sinon :
        // Déborde même avec tous les L1
        retourner "Limité par L2/L3"
```

### 5.3 Modélisation de la Performance avec Cache

```pseudocode
Fonction ModéliserPerformanceMultithread(taille_données, threads) :
    // Paramètres du système
    L1_par_cœur ← 32KB
    L2_par_cœur ← 256KB
    L3_partagé ← 8MB
    
    // Calculer la taille par thread
    taille_par_thread ← taille_données / threads
    
    // Déterminer le niveau de cache utilisé
    si taille_par_thread ≤ L1_par_cœur :
        latence ← latence_L1
        bande_passante ← BP_L1 * threads
    sinon si taille_par_thread ≤ L2_par_cœur :
        latence ← latence_L2
        bande_passante ← BP_L2 * threads
    sinon si taille_données ≤ L3_partagé :
        latence ← latence_L3
        bande_passante ← BP_L3  // Partagée !
    sinon :
        latence ← latence_DRAM
        bande_passante ← min(BP_DRAM, BP_DRAM_par_cœur * threads)
    
    performance ← CalculerDébit(bande_passante, latence)
    retourner performance
```

## Partie 6 : Limites de la Bande Passante Mémoire

### 6.1 Test avec Données en Mémoire Principale

Lorsque les données dépassent tous les caches :

**Test avec 134 217 728 entiers (512 MB) :**

| Threads | Performance | Accélération |
|---------|------------|---------------|
| 1       | 1.4 add/cycle | 1.0x |
| 2       | 2.2 add/cycle | 1.6x |
| 4       | 2.7 add/cycle | 1.9x |

### 6.2 Analyse de la Saturation

```pseudocode
Algorithme : AnalyseBandePassante
Entrée : performances[], nombre_threads[]
Sortie : Point de saturation de la bande passante

Fonction TrouverSaturation(perfs, threads) :
    pour i ← 1 à longueur(perfs)-1 :
        efficacité_incrémentale ← (perfs[i] - perfs[i-1]) / 
                                   (threads[i] - threads[i-1])
        
        si efficacité_incrémentale < 0.5 :
            retourner threads[i-1]  // Point de saturation
    
    retourner -1  // Pas encore saturé
```

### 6.3 Modèle de Bande Passante Partagée

```pseudocode
Fonction ModéliserBandePassanteDRAM(threads, BP_max, BP_par_cœur) :
    // Certains processeurs : un cœur peut utiliser la majorité de la BP
    // D'autres : BP équitablement répartie
    
    si conception = "Asymétrique" :
        BP_effective ← min(BP_max, BP_par_cœur + (threads-1) * BP_incrémentale)
    sinon :  // Conception "Équilibrée"
        BP_effective ← min(BP_max, threads * BP_par_cœur)
    
    retourner BP_effective
```

## Partie 7 : Stratégies d'Optimisation Multithread

### 7.1 Répartition Optimale du Travail

```pseudocode
Algorithme : RépartitionOptimale
Entrée : taille_problème, config_système
Sortie : Configuration optimale des threads

Fonction DéterminerConfigurationOptimale(taille, système) :
    // Analyser les contraintes
    cœurs_disponibles ← système.nombre_cœurs
    cache_par_cœur ← système.L1_size
    
    // Calculer le nombre optimal de threads
    si taille ≤ cache_par_cœur :
        threads_optimaux ← 1  // Tout tient en L1
    sinon si taille ≤ cœurs_disponibles * cache_par_cœur :
        // Utiliser assez de threads pour que tout tienne en L1
        threads_optimaux ← ceil(taille / cache_par_cœur)
    sinon :
        // Utiliser tous les cœurs disponibles
        threads_optimaux ← cœurs_disponibles
    
    retourner min(threads_optimaux, cœurs_disponibles)
```

### 7.2 Éviter la Contention

```pseudocode
Algorithme : PartitionnementSansContention
Entrée : données[], nombre_threads
Sortie : Partitions alignées sur les lignes de cache

Fonction PartitionnerDonnées(données, threads) :
    TAILLE_LIGNE_CACHE ← 64  // octets
    éléments_par_ligne ← TAILLE_LIGNE_CACHE / tailleof(élément)
    
    partitions[] ← nouveau Tableau[threads]
    taille_base ← longueur(données) / threads
    
    // Aligner les partitions sur les lignes de cache
    pour i ← 0 à threads-1 :
        début ← i * taille_base
        // Arrondir au début de ligne de cache
        début_aligné ← (début / éléments_par_ligne) * éléments_par_ligne
        
        fin ← (i + 1) * taille_base
        si i = threads-1 :
            fin ← longueur(données)
        
        partitions[i] ← {début_aligné, fin}
    
    retourner partitions
```

## Partie 8 : Perspectives et Évolution Future

### 8.1 Tendances du Nombre de Cœurs

```pseudocode
Fonction ProjectionPerformance(année_actuelle, année_cible) :
    // Historique : doublement tous les 3-4 ans
    cœurs_actuels ← {
        "grand_public": 8,
        "enthusiaste": 16,
        "serveur": 96
    }
    
    années_écoulées ← année_cible - année_actuelle
    facteur_croissance ← 2^(années_écoulées / 3.5)
    
    pour catégorie dans cœurs_actuels :
        cœurs_futurs[catégorie] ← cœurs_actuels[catégorie] * facteur_croissance
        
        // Impact sur le code mono-thread
        pénalité_mono_thread ← 1 / cœurs_futurs[catégorie]
        print(f"Pénalité {catégorie} en {année_cible}: {pénalité_mono_thread}")
```

### 8.2 Comparaison des Multiplicateurs

| Multiplicateur | Gain Typique | Gain Maximum Observé | Évolution Future |
|----------------|--------------|---------------------|------------------|
| ILP | 2-4x | 4x | Stable |
| SIMD | 4-8x | 16x | Croissance lente |
| Cache | 2-10x | 10x | Stable |
| Multithreading | 4-96x | 96x+ | Croissance rapide |

## Partie 9 : Implémentation Pratique Complète

### 9.1 Framework de Multithreading Générique

```pseudocode
Classe FrameworkParallèle<T> :
    
    Fonction ExécuterEnParallèle(fonction_travail, données, config) :
        // Déterminer la configuration optimale
        threads ← DéterminerNombreThreads(données, config)
        partitions ← PartitionnerDonnées(données, threads)
        
        // Créer le pool de threads
        résultats[] ← nouveau Tableau[threads]
        threads_actifs[] ← nouveau Tableau[threads]
        
        // Lancer l'exécution parallèle
        pour i ← 0 à threads-1 :
            threads_actifs[i] ← CréerThread(() => {
                résultats[i] ← fonction_travail(partitions[i])
            })
            threads_actifs[i].Démarrer()
        
        // Attendre et agréger
        pour i ← 0 à threads-1 :
            threads_actifs[i].Attendre()
        
        retourner AgrégerRésultats(résultats)
    
    Fonction DéterminerNombreThreads(données, config) :
        taille ← CalculerTailleMémoire(données)
        cœurs ← config.cœurs_disponibles
        
        // Heuristiques basées sur nos observations
        si taille < config.L1_size :
            retourner 1
        sinon si taille < cœurs * config.L1_size :
            retourner ceil(taille / config.L1_size)
        sinon si AccèsMémoirePrincipaleDominant(taille) :
            // Limiter si dominé par bande passante
            retourner min(cœurs, 4)  
        sinon :
            retourner cœurs
```

## Conclusion

Le multithreading représente le multiplicateur de performance le plus puissant et le plus évolutif dans l'informatique moderne. Contrairement aux autres multiplicateurs qui plafonnent (ILP ~4x, SIMD ~16x), le multithreading continue de croître avec chaque nouvelle génération de processeurs.

**Points clés à retenir :**

1. **L'évolution est exponentielle** : De 2 cœurs il y a 20 ans à 96+ cœurs aujourd'hui sur les serveurs
2. **L'interaction avec le cache est cruciale** : Le multithreading peut donner des accélérations super-linéaires en multipliant la capacité de cache L1/L2
3. **La bande passante mémoire est souvent le facteur limitant** : Pour les grandes données, l'accélération est limitée par la bande passante DRAM partagée
4. **L'optimisation requiert une approche holistique** : Combiner multithreading avec SIMD et conscience du cache pour des performances maximales
5. **Le coût de ne pas paralléliser augmente** : Chaque nouvelle génération de processeur rend le code mono-thread relativement plus lent

Avec la fin de ce cours sur le multithreading, nous avons maintenant exploré les cinq multiplicateurs fondamentaux qui expliquent les différences de performance de 1000x à 10 000x observées dans les logiciels modernes. La maîtrise de ces concepts est essentielle pour écrire du code performant à l'ère des processeurs multi-cœurs.
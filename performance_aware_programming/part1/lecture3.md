# Conférence : Le Deuxième Multiplicateur Mortel - IPC et Dépendances Sérielles

## Introduction

Dans la conférence précédente, nous avons parlé des instructions gaspillées. Le gaspillage constitue généralement le plus grand multiplicateur pour les logiciels lents. Mais il existe d'autres multiplicateurs qui affectent même les instructions nécessaires.

Après avoir éliminé toutes les instructions inutiles, il reste une grande variabilité dans la vitesse à laquelle le CPU peut réellement exécuter les instructions restantes. Aujourd'hui, nous allons explorer le deuxième multiplicateur : **IPC (Instructions Par Clock)** ou **ILP (Instruction-Level Parallelism)**.

## 1. Comprendre IPC et ILP

### Définitions

**IPC (Instructions Per Clock)** : Le nombre moyen d'instructions machine que le CPU exécute à chaque cycle d'horloge.

**ILP (Instruction-Level Parallelism)** : La capacité d'un CPU à exécuter plusieurs instructions simultanément.

Ces deux termes décrivent essentiellement le même aspect de la performance CPU : combien d'instructions peuvent être exécutées en parallèle.

### Le Mystère des 0.8 Additions par Cycle

Dans notre test précédent avec le programme C, nous avons obtenu 0.8 additions par cycle. Comment est-ce possible ? Et pouvons-nous faire mieux ?

## 2. Analyse de la Boucle de Base

Examinons notre boucle de sommation :

```c
u32 Sum = 0;
for(u32 Index = 0; Index < Count; ++Index)
{
    Sum += Input[Index];
}
```

### Les Instructions Nécessaires

Pour chaque itération, le CPU doit exécuter au minimum :

```pseudo
fonction itération_boucle():
    // 1. Charger la valeur depuis la mémoire
    valeur = LOAD Input[Index]
    
    // 2. Additionner à la somme
    Sum = ADD Sum, valeur
    
    // 3. Incrémenter l'index
    Index = ADD Index, 1
    
    // 4. Comparer avec la limite
    COMPARE Index, Count
    
    // 5. Saut conditionnel
    SI Index < Count ALORS SAUTER au début
```

**Au moins 4 instructions** par itération pour une seule addition utile !

## 3. Technique d'Optimisation : Le Déroulement de Boucle

### Principe du Loop Unrolling

Le déroulement de boucle réduit le ratio entre les instructions de maintenance et le travail utile.

### Implémentation Simple (Listing 5)

```c
typedef unsigned int u32;
u32 SingleScalar(u32 Count, u32 *Input)
{
    u32 Sum = 0;
    for(u32 Index = 0; Index < Count; ++Index)
    {
        Sum += Input[Index];
    }
    return Sum;
}
```

### Déroulement x2 (Listing 6)

```c
u32 Unroll2Scalar(u32 Count, u32 *Input)
{
    u32 Sum = 0;
    for(u32 Index = 0; Index < Count; Index += 2)
    {
        Sum += Input[Index];
        Sum += Input[Index + 1];
    }
    return Sum;
}
```

### Déroulement x4 (Listing 7)

```c
u32 Unroll4Scalar(u32 Count, u32 *Input)
{
    u32 Sum = 0;
    for(u32 Index = 0; Index < Count; Index += 4)
    {
        Sum += Input[Index];
        Sum += Input[Index + 1];
        Sum += Input[Index + 2];
        Sum += Input[Index + 3];
    }
    return Sum;
}
```

### Résultats du Déroulement

| Version | Additions par cycle |
|---------|-------------------|
| SingleScalar | 0.804715 |
| Unroll2Scalar | 0.993692 |
| Unroll4Scalar | 0.993692 |

Le déroulement nous fait passer de 0.8 à presque 1.0, mais nous plafonnons à 1 addition par cycle. Pourquoi ?

## 4. Le Problème : Les Chaînes de Dépendances Sérielles

### Visualisation du Problème

```pseudo
// Ce que nous avons créé involontairement :
Sum = 0
Sum = Sum + Input[0]    // Dépend de Sum précédent
Sum = Sum + Input[1]    // Dépend de Sum précédent
Sum = Sum + Input[2]    // Dépend de Sum précédent
Sum = Sum + Input[3]    // Dépend de Sum précédent
// ... chaîne infinie de dépendances
```

Chaque addition dépend du résultat de la précédente !

### Pourquoi C'est un Problème

Le CPU ne peut pas paralléliser ces opérations car :
- Chaque `ADD` a besoin du résultat du `ADD` précédent
- Le CPU doit attendre la fin de chaque addition avant de commencer la suivante
- Nous avons créé une **chaîne de dépendances sérielles** géante

## 5. La Solution : Briser les Dépendances

### Principe

Puisque l'addition est associative et commutative, nous pouvons réorganiser :

```pseudo
// Au lieu de : (((a + b) + c) + d)
// Nous pouvons faire : (a + c) + (b + d)
```

### Implémentation avec Deux Accumulateurs (Listing 8)

```c
u32 DualScalar(u32 Count, u32 *Input)
{
    u32 SumA = 0;
    u32 SumB = 0;
    for(u32 Index = 0; Index < Count; Index += 2)
    {
        SumA += Input[Index + 0];
        SumB += Input[Index + 1];
    }
    u32 Sum = SumA + SumB;
    return Sum;
}
```

### Implémentation avec Quatre Accumulateurs (Listing 9)

```c
u32 QuadScalar(u32 Count, u32 *Input)
{
    u32 SumA = 0;
    u32 SumB = 0;
    u32 SumC = 0;
    u32 SumD = 0;
    for(u32 Index = 0; Index < Count; Index += 4)
    {
        SumA += Input[Index + 0];
        SumB += Input[Index + 1];
        SumC += Input[Index + 2];
        SumD += Input[Index + 3];
    }
    u32 Sum = SumA + SumB + SumC + SumD;
    return Sum;
}
```

### Version Optimisée avec Pointeurs (Listing 10)

```c
u32 QuadScalarPtr(u32 Count, u32 *Input)
{
    u32 SumA = 0;
    u32 SumB = 0;
    u32 SumC = 0;
    u32 SumD = 0;
    
    Count /= 4;
    while(Count--)
    {
        SumA += Input[0];
        SumB += Input[1];
        SumC += Input[2];
        SumD += Input[3];
        Input += 4;
    }
    
    u32 Sum = SumA + SumB + SumC + SumD;
    return Sum;
}
```

## 6. Analyse des Performances avec Dépendances Brisées

### Résultats Mesurés

| Version | Additions par cycle | Amélioration |
|---------|-------------------|--------------|
| Unroll2Scalar | 0.993692 | Baseline |
| DualScalar | 1.267719 | 1.28x |
| QuadScalar | 1.700997 | 1.71x |
| QuadScalarPtr | 1.948620 | 1.96x |

Nous avons presque **doublé** notre performance en brisant les dépendances !

## 7. Algorithme Général pour Maximiser l'IPC

```pseudo
fonction optimiser_IPC(boucle_originale):
    // Étape 1 : Identifier les dépendances
    dépendances = analyser_dépendances(boucle_originale)
    
    // Étape 2 : Déterminer le parallélisme possible
    si dépendances.sont_sérielles():
        // Étape 3 : Réorganiser si possible
        si opération.est_associative():
            nombre_accumulateurs = min(
                ressources_CPU_disponibles(),
                largeur_bande_passante_mémoire()
            )
            
            // Étape 4 : Créer des chaînes indépendantes
            pour i de 0 à nombre_accumulateurs:
                créer_accumulateur(i)
            
            // Étape 5 : Distribuer le travail
            distribuer_travail_équitablement(accumulateurs)
            
            // Étape 6 : Combiner les résultats
            résultat = combiner_accumulateurs()
    
    retourner version_optimisée
```

## 8. Facteurs Limitants de l'IPC

### Limites Matérielles

```pseudo
fonction calculer_IPC_maximum(CPU, workload):
    // Facteurs limitants
    unités_exécution = CPU.nombre_ALUs
    ports_lecture = CPU.ports_mémoire
    largeur_décodage = CPU.décodeurs_instructions
    
    // Le minimum détermine l'IPC maximum
    IPC_théorique = min(
        unités_exécution,
        ports_lecture / lectures_par_itération,
        largeur_décodage
    )
    
    // En pratique, on atteint rarement le théorique
    IPC_pratique = IPC_théorique * 0.5 à 0.8
    
    retourner IPC_pratique
```

### Comparaison des CPUs Modernes

| CPU | IPC Théorique Maximum |
|-----|---------------------|
| CPU de test (ancien) | ~4 |
| Apple M-series | ~8 |
| AMD Zen4 | ~6 |
| Intel récents | ~5-6 |

## 9. Impact Cumulatif des Optimisations

### Calcul du Gain Total

```pseudo
fonction calculer_amélioration_totale():
    // Rappel : Python baseline
    performance_python = 0.00619 additions/cycle
    
    // Après élimination du gaspillage
    performance_C_base = 0.805 additions/cycle
    gain_gaspillage = 130x
    
    // Après optimisation IPC
    performance_C_optimisé = 1.95 additions/cycle
    gain_IPC = 2.4x
    
    // Gain total
    gain_total = gain_gaspillage × gain_IPC
    gain_total = 130 × 2.4 = 312x
    
    retourner gain_total
```

Nous sommes maintenant **312 fois plus rapides** que Python !

## 10. Principes Généraux pour Optimiser l'IPC

### 1. Identifier les Dépendances

```pseudo
fonction identifier_dépendances(code):
    pour chaque instruction dans code:
        entrées = instruction.opérandes_source
        sortie = instruction.opérande_destination
        
        si sortie est utilisée comme entrée dans instruction_suivante:
            marquer_dépendance_sérielle()
```

### 2. Techniques de Rupture de Dépendances

```pseudo
// Technique 1 : Accumulateurs multiples
fonction somme_parallèle(tableau, n_accumulateurs):
    accumulateurs[n_accumulateurs] = {0}
    
    pour i de 0 à taille(tableau):
        accumulateurs[i % n_accumulateurs] += tableau[i]
    
    retourner somme(accumulateurs)

// Technique 2 : Réduction en arbre
fonction réduction_arbre(tableau):
    tant que taille(tableau) > 1:
        nouveau_tableau = []
        pour i de 0 à taille(tableau) par 2:
            nouveau_tableau.ajouter(tableau[i] + tableau[i+1])
        tableau = nouveau_tableau
    
    retourner tableau[0]
```

## Conclusion

L'IPC est le deuxième multiplicateur majeur de performance après le gaspillage. En comprenant et en brisant les chaînes de dépendances sérielles, nous pouvons :

1. **Doubler ou tripler** facilement les performances
2. **Exploiter** le parallélisme au niveau des instructions du CPU
3. **Multiplier** les gains avec l'élimination du gaspillage

### Points Clés à Retenir

1. **Les CPUs modernes peuvent exécuter plusieurs instructions par cycle**
2. **Les dépendances sérielles empêchent le parallélisme**
3. **Réorganiser le code peut briser ces dépendances**
4. **Un gain de 2-4x est typique, plus sur les CPUs récents**
5. **Ces gains se multiplient avec les autres optimisations**

Le prochain multiplicateur que nous explorerons permettra des gains encore plus importants en exploitant un type différent de parallélisme dans le corps de la boucle...
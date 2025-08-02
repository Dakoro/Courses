Voici votre document reformatté proprement en **Markdown**, tout en conservant les annotations de citation entre crochets (`[cite: x]`) :

---

# SIMD – Single Instruction, Multiple Data

> Bonjour et bienvenue à cette quatrième vidéo du prologue de la série **"Performance-Aware Programming"** \[cite: 1].
> Aujourd'hui, nous allons aborder un des cinq multiplicateurs qui ralentissent les programmes : le **SIMD**, ou *Single Instruction, Multiple Data* \[cite: 2, 12].

Dans la vidéo de présentation du cours, nous avons dit qu’il n’y a que deux choses que nous pouvons faire pour améliorer les performances d’un programme :

1. Réduire le nombre total d’instructions émises.
2. Accélérer le passage des instructions dans le CPU en modifiant leur nature, leur ordre, ou leur modèle d’accès mémoire.

> Nous avons déjà vu dans un post précédent comment éliminer des instructions inutiles, ce qui rend le code plus rapide \[cite: 5].
> Nous avons aussi exploré le parallélisme d’instructions \[cite: 6, 7].
> Aujourd’hui, nous allons nous concentrer sur **la réduction du nombre total d'instructions**.

---

## Qu’est-ce que le SIMD ?

> L’idée du parallélisme d’instructions peut être abordée différemment \[cite: 9, 10].
> Nous pouvons réduire le **nombre d'instructions** nécessaires pour exécuter un même volume de travail \[cite: 11].

C’est ici qu’intervient le **SIMD**, le troisième multiplicateur de notre série \[cite: 12].

* SIMD = *Single Instruction, Multiple Data* \[cite: 13]
* Une seule instruction du CPU opère sur **plusieurs données à la fois** \[cite: 13]

### Exemple

> Reprenons une série d’instructions d’addition.
> Nous avons utilisé plusieurs accumulateurs pour permettre au CPU d’en exécuter plusieurs simultanément \[cite: 15, 16].
> SIMD va plus loin : il s’appuie sur le fait que **beaucoup d’opérations sont répétitives** sur de grandes quantités de données \[cite: 20].

> Une boucle de 20 instructions exécutée 10 000 fois :
> au lieu d’opérer sur une seule donnée, **SIMD** permet à une instruction de **traiter plusieurs éléments à la fois** \[cite: 22–25].

---

## Extensions SIMD sur x64 : SSE, AVX, AVX-512

> Les jeux d’instructions comme **SSE** (*Streaming SIMD Extensions*) ont été conçus pour cela \[cite: 26].
> Exemple : `paddd` (*packed add DWORD*) \[cite: 27–31]

* `"p"` = packed
* `"d"` = DWORD (32 bits)
* Résultat : 4 additions en une instruction \[cite: 30–31]

> SSE a introduit des **valeurs de 128 bits** traitées en *voies* (lanes) :
> 4 entiers de 32 bits dans un seul vecteur \[cite: 33–36]
> Le SIMD effectue alors des opérations **alignées entre ces voies**, comme une addition de vecteurs \[cite: 39–44].

### Pourquoi c’est plus rapide ?

> Même si la quantité d’arithmétique reste la même, le SIMD **réduit le travail de décodage** des instructions \[cite: 45–53].
> Une seule instruction remplace plusieurs instructions individuelles.

### Plus loin : AVX et AVX-512

> **AVX** : 256 bits → 8 entiers de 32 bits \[cite: 55, 59]
> **AVX-512** : 512 bits → jusqu’à 16 entiers de 32 bits (ou plus avec des types plus petits) \[cite: 56–68]

---

## Démonstration pratique et implémentation

### Version scalaire simple (SingleScalar)

```c
fonction SingleScalar(Count, Input)
  Sum = 0
  pour Index de 0 à Count-1
    Sum = Sum + Input[Index]
  retourner Sum
```

> Cette version obtient environ **0.8 à 0.9** additions par cycle \[cite: 128].

---

### Version SSE (SingleSSE)

```c
fonction SingleSSE(Count, Input)
  Sum = __m128i(tous_zeros)
  pour Index de 0 à Count par pas de 4
    Data = charger_128_bits(&Input[Index])
    Sum = Sum + Data
  Sum = hadd(Sum, Sum)
  Sum = hadd(Sum, Sum)
  retourner convertir_128_bits_vers_32_bits(Sum)
```

> Utilise `__m128i`, `_mm_setzero_si128`, `_mm_add_epi32`, `_mm_load_si128` \[cite: 95–98]
> **Performance : 3.1** additions/cycle \[cite: 129]

---

### Version AVX (SingleAVX)

```c
fonction SingleAVX(Count, Input)
  Sum = __m256i(tous_zeros)
  pour Index de 0 à Count par pas de 8
    Data = charger_256_bits(&Input[Index])
    Sum = Sum + Data
  Sum = hadd(Sum, Sum)
  Sum = hadd(Sum, Sum)
  SumS = permute_2x128_bits(Sum, Sum)
  Sum = Sum + SumS
  retourner convertir_256_bits_vers_32_bits(Sum)
```

> Utilise `__m256i`, `_mm256` \[cite: 112–114]
> **Performance : 7.0** additions/cycle \[cite: 130]

---

## SIMD + Parallélisme d’instructions : QuadAVX

> Peut-on combiner SIMD **et** le parallélisme d’instructions ? Oui ! \[cite: 142–147]

### QuadAVX (4 accumulateurs AVX)

```c
fonction QuadAVX(Count, Input)
  SumA = __m256i(tous_zeros)
  SumB = __m256i(tous_zeros)
  SumC = __m256i(tous_zeros)
  SumD = __m256i(tous_zeros)
  pour Index de 0 à Count par pas de 32
    SumA = SumA + charger_256_bits(&Input[Index])
    SumB = SumB + charger_256_bits(&Input[Index + 8])
    SumC = SumC + charger_256_bits(&Input[Index + 16])
    SumD = SumD + charger_256_bits(&Input[Index + 24])

  SumAB = SumA + SumB
  SumCD = SumC + SumD
  Sum = SumAB + SumCD
  Sum = hadd(Sum, Sum)
  Sum = hadd(Sum, Sum)
  SumS = permute_2x128_bits(Sum, Sum)
  Sum = Sum + SumS
  retourner convertir_256_bits_vers_32_bits(Sum)
```

> `DualAVX` : **9.4** additions/cycle
> `QuadAVX` : **11.0** additions/cycle \[cite: 177–179]
> `QuadAVXPtr` (version optimisée) : **13.4** additions/cycle \[cite: 179]

---

## Résumé des gains de performance

* **Boucle C naïve** : \~0.85 additions/cycle
* **QuadAVXPtr** : \~13.39 additions/cycle
* **Amélioration : ×16**
* **Comparée à Python** : \~×3000 plus rapide \[cite: 190–195]

> Cela démontre l’énorme écart entre un code standard et un code **optimisé pour le CPU** \[cite: 199–201].

---

## Et ensuite ?

> Cette démonstration suppose un **bon comportement du cache** grâce à des tableaux de petite taille \[cite: 207].
> Que se passe-t-il quand ce n’est pas le cas ?
> **Ce sera l’objet de la prochaine discussion** \[cite: 209].

---

N’hésitez pas si vous avez des questions !

---

Souhaitez-vous que je crée une version PDF ou HTML de ce document Markdown ?

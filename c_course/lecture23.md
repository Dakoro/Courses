# Nombres à Virgule Flottante : Des Mathématiques à l'Assembleur et au-delà

## Partie 1 : Le Défi de la Représentation des Nombres Réels

### Introduction : L'abîme entre les mathématiques pures et les contraintes de la machine

Bonjour à tous. Nous entamons aujourd'hui une exploration approfondie d'un sujet qui se situe au carrefour des mathématiques pures, de l'ingénierie matérielle et du génie logiciel : les nombres à virgule flottante. Pour de nombreux développeurs, ces nombres sont une boîte noire, une source de bugs subtils et de comportements contre-intuitifs. Notre objectif est de démanteler cette boîte noire, de comprendre sa logique interne, ses compromis et son élégance.

Le problème fondamental que nous allons aborder est une dichotomie irréconciliable. D'un côté, nous avons l'ensemble des nombres réels, tel que défini en mathématiques : un continuum infini, dense, où entre deux nombres, aussi proches soient-ils, il en existe une infinité d'autres. De l'autre côté, nous avons l'ordinateur : une machine intrinsèquement finie et discrète. Sa mémoire est limitée, ses registres ont une taille fixe, et chaque opération qu'il exécute est bornée par un nombre de bits, que ce soit 32 ou 64. 

Comment peut-on alors faire tenir l'infini dans une boîte finie? La réponse est simple : on ne peut pas. Toute représentation informatique des nombres réels sera, par nécessité, une approximation, un sous-ensemble fini et soigneusement choisi. L'histoire de l'arithmétique à virgule flottante est l'histoire de la recherche de la "meilleure" approximation possible, un compromis constant entre la plage de valeurs, la précision et la complexité de l'implémentation.

### La nature des nombres réels et le problème de la calculabilité de Turing

Pour saisir l'ampleur du défi, il faut d'abord comprendre ce qu'est un nombre réel d'un point de vue computationnel. On peut imaginer un nombre réel comme une fonction, une sorte d'oracle qui, lorsqu'on lui demande le $n$-ième chiffre après la virgule, nous le fournit. Cette fonction peut potentiellement générer une séquence infinie de chiffres. Par exemple, pour $\pi$, la fonction nous donnerait successivement 1, 4, 1, 5, 9, etc., sans jamais s'arrêter.

C'est ici qu'intervient le concept de calculabilité, formalisé par Alan Turing. Un nombre est dit "calculable" s'il existe un algorithme, une machine de Turing, capable de générer ses chiffres les uns après les autres. Les nombres que nous manipulons au quotidien — les entiers, les rationnels (comme 1/3), et même les constantes célèbres comme $\pi$ ou $e$ — sont tous calculables. On pourrait penser que cela couvre la majorité des cas. C'est une erreur profonde.

La réalité mathématique est stupéfiante : la quasi-totalité des nombres réels n'est pas calculable par une machine de Turing. Si vous pouviez choisir un nombre réel au hasard sur la droite numérique, la probabilité de tomber sur un nombre pour lequel il n'existe aucun algorithme fini pour générer ses décimales est de 1. Les nombres calculables que nous chérissons ne sont, pour reprendre une métaphore, qu'une "écume à la surface de l'océan" de l'ensemble des réels.

Cette constatation a une implication fondamentale pour l'informatique. Puisque les ordinateurs ne peuvent exécuter que des algorithmes finis, l'objectif de l'arithmétique informatique ne peut pas être de représenter tous les nombres réels. C'est une impossibilité théorique. L'objectif est bien plus pragmatique : il s'agit de concevoir un système fini et discret qui approxime de manière utile et efficace le sous-ensemble de nombres réels dont nous avons besoin pour la science, la physique, le graphisme et l'ingénierie. Dans cette optique, la norme IEEE 754, que nous allons disséquer, n'est pas une tentative "imparfaite" de représenter les réels ; c'est une solution d'ingénierie extraordinairement astucieuse à un problème mathématiquement insoluble.

### Première approche intuitive : L'arithmétique à virgule fixe

Face à ce défi, quelle est la première idée qui vient à l'esprit? Si nous savons représenter les entiers, qui sont espacés régulièrement, pourquoi ne pas faire de même pour les nombres fractionnaires? C'est l'idée de l'arithmétique à virgule fixe.

Le principe est simple : on prend un certain nombre de bits, par exemple 64, et on décrète que la virgule binaire se trouve à une position fixe. On pourrait, par exemple, allouer les 32 bits de poids fort à la partie entière et les 32 bits de poids faible à la partie fractionnaire. Dans ce système, le plus petit pas de progression, la plus petite différence entre deux nombres consécutifs, serait constant et égal à $2^{-32}$.

Bien qu'intuitive, cette approche souffre de deux limitations critiques qui la rendent inutilisable pour la plupart des applications scientifiques modernes :

1. **Une plage dynamique très faible** : Avec 32 bits pour la partie entière, le plus grand nombre représentable est $2^{32} - 1 \approx 4.3 \times 10^9$. C'est insuffisant pour des domaines comme l'astronomie, où des valeurs comme $10^{20}$ (la distance Terre-Soleil en kilomètres est d'environ $1.5 \times 10^8$) sont monnaie courante et où des calculs peuvent impliquer des nombres bien plus grands. Inversement, la plus petite fraction non nulle est $2^{-32}$, soit environ $2.3 \times 10^{-10}$. C'est un pas de quantification beaucoup trop grossier pour la physique quantique, où des constantes comme la constante de Planck (environ $6.6 \times 10^{-34}$ J·s) ou des calculs impliquant des échelles de l'ordre de $10^{-40}$ sont fréquents. Le système à virgule fixe nous force à choisir entre représenter de très grands nombres ou de très petits nombres, mais il est incapable de faire les deux simultanément.

2. **Une précision absolue fixe et une précision relative variable** : Le pas entre deux nombres est toujours $2^{-32}$. Cela signifie que la précision absolue est constante. Cependant, la précision relative (l'erreur par rapport à la magnitude du nombre) est très mauvaise pour les petits nombres et excellente pour les grands. Or, en sciences, on a souvent besoin du contraire : une grande précision relative pour les petites mesures et une tolérance à une plus grande erreur absolue pour les grandes quantités.

Ces limitations nous mènent inévitablement à la conclusion qu'une approche plus flexible est nécessaire. Une approche où la position de la virgule n'est pas fixe, mais peut "flotter" pour s'adapter à la magnitude du nombre que l'on souhaite représenter. C'est la genèse de l'arithmétique à virgule flottante et de la norme qui la régit : l'IEEE 754.

## Partie 2 : La Norme IEEE 754 en Profondeur : Le float 32 bits

### Anatomie d'un nombre float : Le bit de signe, l'exposant biaisé et la mantisse

L'idée centrale de la virgule flottante est de s'inspirer de la notation scientifique. Un nombre comme 1 234 500 peut s'écrire $1.2345 \times 10^6$. On dissocie ainsi les chiffres significatifs (1.2345, la mantisse ou significande) de l'ordre de grandeur ($10^6$, l'exposant). La norme IEEE 754 applique ce principe au système binaire.

Un nombre à virgule flottante de simple précision, connu en C sous le nom de `float`, est encodé sur 32 bits. Ces 32 bits ne sont pas interprétés comme un seul entier, mais sont divisés en trois champs distincts¹ :

1. **Le bit de signe (S)** : C'est le bit de poids le plus fort (bit 31). Il est très simple : 0 pour un nombre positif, 1 pour un nombre négatif.²

2. **L'exposant (E)** : Les 8 bits suivants (bits 30 à 23) représentent l'exposant. Pour pouvoir représenter des ordres de grandeur très grands (exposants positifs) et très petits (exposants négatifs), on utilise une technique appelée "exposant biaisé". Au lieu d'utiliser une représentation en complément à deux, on stocke une valeur entière non signée de 0 à 255. Pour obtenir l'exposant réel, on soustrait une valeur fixe, le "biais", qui est de 127 pour les float 32 bits.² Ainsi, un exposant stocké de 127 correspond à un exposant réel de $127 - 127 = 0$. Un exposant stocké de 130 correspond à $130 - 127 = 3$. Un exposant stocké de 124 correspond à $124 - 127 = -3$. Cette astuce permet de comparer la magnitude de deux nombres flottants positifs simplement en comparant leurs représentations binaires comme s'il s'agissait d'entiers non signés, ce qui simplifie grandement la conception des unités de comparaison matérielles.³

3. **La fraction (F)** : Les 23 bits restants (bits 22 à 0) encodent la partie fractionnaire de la mantisse. C'est ici qu'intervient une autre optimisation cruciale : le bit implicite (ou bit caché). En notation scientifique binaire normalisée, on s'arrange toujours pour que le nombre ait la forme $1.xxxx... \times 2^n$. Puisque le chiffre avant la virgule est toujours un 1 (sauf pour le cas particulier de zéro et des nombres dénormalisés), il n'est pas nécessaire de le stocker. On économise ainsi un bit. La mantisse réelle est donc reconstituée comme 1.F, où F est la valeur des 23 bits de la fraction. Cela nous donne en réalité 24 bits de précision pour le prix de 23.⁴

### La formule des nombres normalisés : $(-1)^S \times 1.F \times 2^{(E - 127)}$

En combinant ces trois champs, on obtient la formule qui permet de décoder la valeur d'un nombre float :

$$\text{valeur} = (-1)^S \times 1.F \times 2^{(E - 127)}$$

Dans cette formule :
- $S$ est la valeur du bit de signe (0 ou 1).
- $F$ est la valeur de la fraction interprétée comme $\sum_{i=1}^{23} b_i \times 2^{-i}$. Par exemple, si les bits de la fraction sont 1010..., $F$ vaut $1 \times 2^{-1} + 0 \times 2^{-2} + 1 \times 2^{-3} + ... = 0.625$
- $E$ est la valeur entière non signée des 8 bits de l'exposant.

Cette formule est valable pour les nombres normalisés, c'est-à-dire lorsque le champ de l'exposant $E$ n'est ni entièrement à zéros (00000000), ni entièrement à uns (11111111). Ces deux motifs de bits extrêmes sont réservés pour encoder les cas spéciaux que nous verrons dans la partie suivante : le zéro, les infinis, les NaNs et les nombres dénormalisés.¹ La plage des exposants stockés pour les nombres normalisés va donc de 1 à 254, ce qui correspond à des exposants réels de -126 à +127.

### Atelier pratique : Conversion manuelle et en C

Pour bien ancrer ces concepts, rien ne vaut la pratique. Reprenons l'exemple de la transcription et décodons pas à pas la représentation binaire du nombre -5.5.

**Conversion en binaire** : 5.5 en décimal est $5 + 0.5$. En binaire, 5 est 101. 0.5 est $1/2$, soit $2^{-1}$, donc .1 en binaire. Le nombre est donc 101.1.

**Normalisation** : Pour l'écrire sous la forme 1.xxxx..., nous devons décaler la virgule de deux positions vers la gauche : 1.011. Pour compenser, nous multiplions par $2^2$. Le nombre est donc $-1.011 \times 2^2$.

**Extraction des champs** :
- Signe (S) : Le nombre est négatif, donc $S = 1$.
- Exposant (E) : L'exposant réel est 2. L'exposant stocké $E$ doit être $2 + 127 = 129$. En binaire sur 8 bits, 129 s'écrit 10000001.
- Fraction (F) : La partie après la virgule dans 1.011 est 011. Nous devons compléter avec des zéros pour obtenir 23 bits : 01100000000000000000000.

**Assemblage** : En concaténant les trois champs (S | E | F), nous obtenons :
```
1 | 10000001 | 01100000000000000000000
```

En regroupant par quartets pour la conversion en hexadécimal :
```
1100 0000 1011 0000 0000 0000 0000 0000
```
Ce qui donne C0B00000 en hexadécimal.

Faisons maintenant l'exercice inverse. Quelle est la représentation du nombre 1.0?

**Conversion et Normalisation** : $1.0 = 1.0 \times 2^0$.

**Extraction des champs** :
- Signe (S) : Positif, donc $S = 0$.
- Exposant (E) : L'exposant réel est 0. L'exposant stocké est $0 + 127 = 127$. En binaire, 127 est 01111111.
- Fraction (F) : La partie après la virgule dans 1.0 est 0. La fraction est donc constituée de 23 zéros : 000...0.

**Assemblage** :
```
0 | 01111111 | 00000000000000000000000
```

En hexadécimal, cela donne :
```
0011 1111 1000 0000 0000 0000 0000 0000
```
Soit 3F800000. C'est une valeur qu'il est utile de mémoriser.

Pour obtenir cette représentation en C, il ne suffit pas de faire un simple transtypage. Comme nous le verrons en détail dans la Partie 4, la méthode correcte et portable pour réinterpréter les bits d'un float en entier est d'utiliser `memcpy` pour éviter les problèmes d'aliasing strict.

## Partie 3 : Les Cas Particuliers : Zéros, Infinis et "Non-Nombres" (NaNs)

La véritable force de la norme IEEE 754 ne réside pas seulement dans sa capacité à représenter une large gamme de nombres, mais aussi dans sa gestion rigoureuse et cohérente des cas limites et des résultats d'opérations invalides. Pour ce faire, elle réserve deux valeurs pour le champ de l'exposant : 00000000 et 11111111.

### Le "hack" des nombres dénormalisés pour un "underflow" graduel

Un examen attentif de la formule des nombres normalisés révèle une lacune critique : il est impossible de représenter le nombre zéro. Le plus petit nombre positif normalisé que l'on puisse former a une mantisse de 1.0 et le plus petit exposant possible, soit -126. Sa valeur est $1.0 \times 2^{-126} \approx 1.17 \times 10^{-38}$. Entre ce nombre minuscule et zéro, il y a un "trou". Si le résultat d'un calcul tombait dans ce trou, une approche naïve consisterait à l'arrondir à zéro ("flush-to-zero"). Cependant, un passage aussi abrupt à zéro est numériquement dangereux. Par exemple, si x et y sont deux nombres très petits mais différents, l'expression x - y pourrait être "flushée" à zéro, ce qui rendrait faux le test `if (x != y)`.

Pour résoudre ce problème, la norme introduit le concept de nombres dénormalisés (ou subnormaux).³ Lorsque le champ de l'exposant est entièrement à zéros (00000000), la règle de calcul change :

$$\text{valeur} = (-1)^S \times 0.F \times 2^{-126}$$

Deux choses importantes ont changé :
- Le bit implicite de la mantisse devient 0 au lieu de 1.
- L'exposant est fixé à -126, la plus petite valeur possible pour un nombre normalisé.

Ce mécanisme permet de combler l'écart autour de zéro. À mesure que la fraction F devient plus petite, le nombre se rapproche de zéro en douceur. C'est ce qu'on appelle l'underflow graduel (gradual underflow).⁶ La contrepartie est que les nombres dénormalisés ont moins de bits de précision significatifs que les nombres normalisés (le nombre de zéros en tête de la mantisse augmente). Ce n'est donc pas simplement un "hack" pour représenter zéro ; c'est un mécanisme de sécurité numérique fondamental qui augmente la robustesse des calculs en préservant autant d'informations que possible lors de la perte de précision. Le nombre zéro lui-même est un cas particulier de cette règle : lorsque l'exposant et la fraction sont tous deux à zéro, la valeur est $(-1)^S \times 0.0 \times 2^{-126} = 0$.²

### Représentation de l'infini et du zéro signé

Lorsque le champ de l'exposant est entièrement à uns (11111111) et que la fraction est nulle (000...0), la valeur représente l'infini.¹ Le bit de signe permet de distinguer entre +∞ (résultat de 1.0 / 0.0, par exemple) et -∞ (résultat de -1.0 / 0.0). L'arithmétique avec les infinis est bien définie : ∞ + 5 = ∞, ∞ * ∞ = ∞, 5 / ∞ = 0.

Cette distinction nous amène au concept de zéro signé. Comme nous l'avons vu, le zéro est représenté par un exposant et une fraction nuls. Cependant, le bit de signe peut être à 0 ou à 1. Nous avons donc deux représentations pour zéro : +0 (tous les bits à 0) et -0 (bit de signe à 1, le reste à 0). D'un point de vue comparatif, +0 et -0 sont égaux (`if (-0.0 == 0.0)` est vrai). Cependant, ils se comportent différemment dans certaines situations, préservant ainsi des identités mathématiques importantes. L'exemple le plus classique est la division : 1.0 / +0.0 donne +∞, tandis que 1.0 / -0.0 donne -∞.⁶ Cela permet de conserver des informations sur le signe d'un résultat qui a atteint zéro par underflow.

### Le concept de NaN (Not-a-Number)

Que se passe-t-il lorsque le résultat d'une opération n'est pas un nombre réel défini, comme 0.0 / 0.0, ∞ - ∞ ou la racine carrée d'un nombre négatif? La norme IEEE 754 définit une valeur spéciale pour représenter ces résultats indéterminés : NaN, pour "Not a Number".¹

Un NaN est représenté par un exposant entièrement à uns (11111111) et une fraction non nulle.¹ La partie non nulle de la fraction permet de distinguer différents types de NaNs et même d'encoder des informations sur l'origine de l'erreur. La norme distingue deux catégories principales de NaNs, en se basant sur le bit de poids fort de la fraction :

- **NaNs Silencieux (Quiet NaNs, qNaNs)** : Le bit de poids fort de la fraction est à 1.⁷ Un qNaN se propage à travers la plupart des opérations arithmétiques sans déclencher d'exception. Si un des opérandes d'une opération est un qNaN, le résultat est généralement ce même qNaN. C'est le type de NaN produit par défaut par les opérations invalides.

- **NaNs Signalants (Signaling NaNs, sNaNs)** : Le bit de poids fort de la fraction est à 0 (et le reste de la fraction n'est pas nul).¹ L'utilisation d'un sNaN comme opérande dans une opération arithmétique déclenche une exception matérielle de type "opération invalide". Leur but principal est de permettre la détection d'utilisation de variables non initialisées. On peut initialiser une variable flottante avec un sNaN, et si le programme tente de l'utiliser avant de lui avoir assigné une valeur valide, une exception est levée, ce qui est un outil de débogage puissant.

Une propriété fondamentale et souvent déroutante des NaNs est qu'un NaN n'est jamais égal à quoi que ce soit, y compris à lui-même. L'expression `x == x` est fausse si x est un NaN.¹ Cette décision de conception, loin d'être illogique, fournit un moyen portable et fiable de tester si une valeur est un NaN en C/C++ : `if (x != x) { /* x is a NaN */ }`.

### Tableau 1 : Représentation des Valeurs Spéciales en IEEE 754 (32 bits)

Ce tableau synthétise les règles de décodage des motifs binaires. Il illustre comment des combinaisons spécifiques sont réservées pour gérer les cas exceptionnels, transformant des concepts mathématiques abstraits en représentations binaires concrètes et non ambiguës.

| Catégorie | Bit de Signe (S) | Exposant (E) | Fraction (F) | Valeur |
|-----------|------------------|--------------|--------------|---------|
| Zéros | 0 / 1 | 00000000 | 000...0 | +0 / -0 |
| Dénormalisés | 0 / 1 | 00000000 | Non nul | $(-1)^S \times 0.F \times 2^{-126}$ |
| Normalisés | 0 / 1 | 00000001 à 11111110 | Quelconque | $(-1)^S \times 1.F \times 2^{(E-127)}$ |
| Infinis | 0 / 1 | 11111111 | 000...0 | +∞ / -∞ |
| Quiet NaN | 0 / 1 | 11111111 | 1xx...x | qNaN |
| Signaling NaN | 0 / 1 | 11111111 | 0xx...x (non nul) | sNaN |

## Partie 4 : Manipulation en C : Précision, Conversion et Arrondi

### Réinterprétation binaire : Le danger du "type punning" et la méthode correcte

Un besoin courant en programmation de bas niveau est d'examiner la représentation binaire d'un nombre à virgule flottante, par exemple pour le stocker, le transmettre ou effectuer des manipulations bit à bit. La question est : comment obtenir la valeur entière de 32 bits qui correspond au motif binaire d'un float?

L'approche la plus intuitive, mais aussi la plus dangereuse, est le transtypage de pointeur, souvent appelé "type punning" :

```c
float f = 3.14f;
unsigned int i = *(unsigned int*)&f; // DANGER : COMPORTEMENT INDÉFINI
```

Bien que ce code puisse sembler fonctionner sur de nombreux compilateurs avec des options d'optimisation faibles, il invoque un comportement indéfini (Undefined Behavior) selon la norme du langage C.¹ La raison est la violation de la règle du "strict aliasing". Cette règle stipule qu'un objet ne peut être accédé que par un pointeur de son propre type ou d'un type compatible (comme `char*`). En accédant à une zone mémoire de type `float` via un pointeur de type `unsigned int*`, on enfreint cette règle. Un compilateur moderne, en se basant sur cette règle, peut effectuer des optimisations qui supposent que f et i ne peuvent pas pointer vers la même mémoire, ce qui peut casser le code de manière imprévisible.

La méthode correcte, portable et garantie par la norme pour effectuer cette réinterprétation de bits est d'utiliser `memcpy` de la bibliothèque `<string.h>`.¹

```c
#include <string.h>
#include <stdint.h> // Pour uint32_t

uint32_t float_to_uint32(float f) {
    uint32_t u;
    memcpy(&u, &f, sizeof(u));
    return u;
}
```

Les compilateurs modernes sont suffisamment intelligents pour reconnaître cette idiom. Ils ne généreront pas un appel de fonction coûteux, mais plutôt une simple instruction de déplacement de registre (par exemple, `movd` en x86-64), produisant ainsi un code aussi efficace que la version incorrecte, mais tout en étant parfaitement défini par la norme.¹⁰ Une autre technique valide et courante consiste à utiliser une union, qui est explicitement conçue pour permettre à différents types de partager le même emplacement mémoire.¹⁰

### Le concept d'ULP (Unit in the Last Place) et la précision relative

La distance entre deux nombres à virgule flottante représentables consécutifs n'est pas constante. Cette distance est appelée un ULP, pour "Unit in the Last Place". C'est la valeur du bit de poids le plus faible de la mantisse, mise à l'échelle par l'exposant.

Cette non-constance de l'ULP a une implication profonde. Près de 1.0, l'exposant est 0. L'ULP correspond à la valeur du 23ème bit de la fraction, soit $2^{-23}$, qui est une valeur très petite (environ $1.2 \times 10^{-7}$). Cependant, pour un nombre très grand, comme $2^{100}$, l'ULP vaudra $2^{100-23} = 2^{77}$, une valeur astronomique. Inversement, pour un nombre dénormalisé très petit, l'ULP est minuscule.

Cela nous amène à la caractéristique la plus fondamentale et la plus puissante des nombres à virgule flottante : ils offrent une précision relative à peu près constante, au détriment d'une précision absolue variable. Un float de 32 bits offre environ 24 bits de précision pour sa mantisse, ce qui équivaut à environ 7 chiffres décimaux significatifs ($\log_{10}(2^{24}) \approx 7.2$). Que vous représentiez la distance entre deux atomes ou la distance entre deux galaxies, vous aurez toujours environ 7 chiffres significatifs de précision (dans la plage des nombres normalisés). C'est ce compromis constant entre la plage de valeurs et la précision relative qui fait la force des nombres à virgule flottante et explique pourquoi ils ont complètement supplanté la virgule fixe dans les calculs scientifiques.

### Les modes d'arrondi de la norme IEEE 754

Lorsqu'une opération arithmétique (comme une addition ou une multiplication) produit un résultat qui tombe entre deux nombres représentables, il faut choisir l'un des deux. Ce processus s'appelle l'arrondi. La norme IEEE 754 ne laisse pas ce choix au hasard ; elle définit rigoureusement plusieurs modes d'arrondi pour garantir des résultats cohérents et prévisibles sur toutes les plateformes.

Les quatre modes principaux sont¹⁴ :

1. **Arrondi au plus proche (par défaut)** (`FE_TONEAREST`) : Le résultat est arrondi au nombre représentable le plus proche. Si le résultat est exactement à mi-chemin entre deux nombres, la règle "ties to even" s'applique : on choisit le nombre dont le dernier bit de la mantisse est un zéro (le nombre "pair"). Cette règle a été choisie pour minimiser le biais statistique sur de longues séquences de calculs, car elle arrondit à la hausse et à la baisse de manière équilibrée dans les cas limites.

2. **Arrondi vers zéro (troncature)** (`FE_TOWARDZERO`) : Le résultat est arrondi vers zéro, ce qui revient à simplement ignorer les bits excédentaires. 3.7 devient 3.0 et -3.7 devient -3.0.

3. **Arrondi vers plus l'infini (plafond)** (`FE_UPWARD`) : Le résultat est arrondi au plus petit nombre représentable qui est supérieur ou égal au résultat exact. 3.2 devient 4.0, mais -3.7 devient -3.0.

4. **Arrondi vers moins l'infini (plancher)** (`FE_DOWNWARD`) : Le résultat est arrondi au plus grand nombre représentable qui est inférieur ou égal au résultat exact. 3.7 devient 3.0, et -3.2 devient -4.0.

En C, ces modes peuvent être contrôlés dynamiquement au moment de l'exécution à l'aide des fonctions de l'en-tête `<fenv.h>`. La fonction `fesetround()` permet de changer le mode d'arrondi global pour le thread en cours, tandis que `fegetround()` permet de connaître le mode actuel. Cette capacité est essentielle pour implémenter des algorithmes d'arithmétique d'intervalle, où l'on calcule une borne inférieure et une borne supérieure pour un résultat en effectuant le calcul une fois en mode `FE_DOWNWARD` et une autre fois en mode `FE_UPWARD`.

## Partie 5 : Sous le Capot : L'Assembleur x86-64 et l'Évolution Matérielle

### Un regard historique : Le co-processeur x87 et son architecture à pile

Pour comprendre l'état actuel de l'arithmétique flottante en assembleur, un détour par l'histoire est nécessaire. Dans les premières architectures x86 (comme le 8086), les calculs à virgule flottante étaient si complexes qu'ils étaient délégués à une puce séparée : le co-processeur mathématique, comme l'Intel 8087.¹ Plus tard, cette unité de calcul, ou FPU (Floating-Point Unit), a été intégrée directement sur la même puce que le processeur principal (CPU) à partir du 80486.¹⁶

L'architecture de cette FPU x87, qui a perduré pendant près de 20 ans, était pour le moins idiosyncratique. Elle n'utilisait pas un ensemble de registres nommés comme le CPU principal, mais une pile de 8 registres, nommés st(0) à st(7).¹ st(0) représentait toujours le sommet de la pile. La plupart des instructions arithmétiques opéraient implicitement sur le ou les éléments au sommet de la pile. Par exemple, pour additionner deux nombres, il fallait les pousser sur la pile, puis exécuter une instruction comme `faddp` qui additionnait st(1) et st(0), stockait le résultat dans st(1), puis dépilait st(0).¹

Cette conception à pile, inspirée des calculatrices RPN, s'est avérée être un cauchemar pour les compilateurs. Générer un code efficace était extrêmement complexe, nécessitant souvent des instructions `fxch` pour échanger des éléments de la pile, ce qui alourdissait le code et dégradait les performances.¹⁸ De plus, un autre problème majeur était que les registres de la FPU x87 travaillaient en interne avec une précision étendue de 80 bits. Chaque fois qu'un float (32 bits) ou un double (64 bits) était chargé depuis la mémoire, il était implicitement converti en 80 bits. Les calculs intermédiaires étaient donc plus précis que le résultat final stocké en mémoire. Cela pouvait conduire à des résultats non reproductibles en fonction de la manière dont le compilateur gérait les valeurs intermédiaires, créant des bugs notoirement difficiles à déboguer.¹⁸

### La révolution SSE : Registres XMM et l'avènement du SIMD

À la fin des années 1990, face aux besoins croissants des applications multimédia (graphisme 3D, traitement audio/vidéo), Intel a introduit une nouvelle série d'extensions à son jeu d'instructions : les SSE (Streaming SIMD Extensions).¹ Cette nouvelle architecture a marqué une rupture radicale avec le modèle x87 et a posé les bases de l'arithmétique flottante moderne.

La principale innovation de SSE a été l'abandon de l'architecture à pile au profit d'une architecture à registres plate et moderne, beaucoup plus facile à gérer pour les compilateurs. SSE a introduit un nouvel ensemble de 8 (puis 16 en mode 64 bits) registres de 128 bits, nommés xmm0 à xmm15.¹ Ces registres peuvent contenir :
- Quatre nombres float de 32 bits.
- Deux nombres double de 64 bits.

Cette capacité à stocker plusieurs données dans un seul registre est la base du SIMD (Single Instruction, Multiple Data). Le processeur peut exécuter une seule instruction, par exemple `addps` (Add Packed Single-precision), qui additionne en parallèle les quatre paires de float contenues dans deux registres xmm.¹⁸ Cela permet une accélération massive des calculs sur des tableaux de données, qui sont omniprésents en multimédia et en calcul scientifique. De plus, SSE a introduit des instructions scalaires (opérant sur une seule valeur, comme `addss` - Add Scalar Single-precision) qui utilisent la partie inférieure des registres xmm, offrant une alternative propre et efficace aux instructions x87 pour les calculs non vectoriels.

### Analyse de l'ABI System V pour x86-64

Le passage à l'architecture SSE a eu un impact direct sur la manière dont les fonctions s'échangent des données, ce qui est formalisé dans l'ABI (Application Binary Interface). Pour les systèmes de type Unix en 64 bits, l'ABI System V AMD64 est la norme.

Cette ABI stipule clairement comment les arguments sont passés aux fonctions :
- Les six premiers arguments de type entier ou pointeur sont passés dans les registres à usage général : rdi, rsi, rdx, rcx, r8, r9.
- Les huit premiers arguments à virgule flottante (float ou double) sont passés dans les registres SSE : xmm0, xmm1, xmm2,..., xmm7.
- La valeur de retour d'une fonction est placée dans rax si c'est un entier/pointeur, et dans xmm0 si c'est un nombre à virgule flottante.

Considérons une fonction avec une signature mixte, comme `float ma_fonction(int a, float b, double c, int d);`. Lors de l'appel :
- a (premier entier) ira dans rdi.
- b (premier flottant) ira dans xmm0.
- c (deuxième flottant) ira dans xmm1.
- d (deuxième entier) ira dans rsi.

Cette séparation claire entre les classes d'arguments (entiers vs flottants) et l'utilisation intensive des registres rendent les appels de fonction beaucoup plus efficaces que l'ancien modèle qui reposait principalement sur la pile mémoire.

### Tableau 2 : Comparaison Architecturale : x87 FPU vs. SSE/AVX

Ce tableau met en lumière la rupture technologique et philosophique entre ces deux ères du calcul flottant. Il explique concrètement pourquoi le code moderne est non seulement plus rapide, mais aussi plus prévisible et plus facile à optimiser.

| Caractéristique | x87 FPU (Héritage) | SSE / AVX (Moderne) |
|----------------|-------------------|---------------------|
| Modèle de registres | Pile (8 registres st(i) de 80 bits) | Fichier de registres (16 registres xmm/ymm de 128/256 bits) |
| Génération de code | Complexe, inefficace (gestion de la pile avec fxch) | Simple, efficace (modèle de type RISC) |
| Précision interne | Fixe à 80 bits (source de non-reproductibilité) | Spécifiée par l'instruction (-ss pour float, -sd pour double) |
| Capacité SIMD | Aucune | Oui (128-bit SSE, 256-bit AVX, 512-bit AVX-512) |
| Convention d'appel (ABI) | Arguments passés sur la pile mémoire | Arguments passés dans les registres XMM dédiés |
| Format d'instruction | 2 opérandes (destructif, e.g., a = a + b) | 3 opérandes (non-destructif avec AVX, e.g., c = a + b) |

## Partie 6 : Optimisations et Leurs Dangers : Le Cas de -ffast-math

### Les garanties (et non-garanties) de l'arithmétique flottante

Nous avons établi que les nombres à virgule flottante sont une approximation des nombres réels. Cette approximation a des conséquences directes sur les propriétés algébriques que nous tenons pour acquises en mathématiques. La plus importante de ces conséquences est que l'addition à virgule flottante n'est pas associative.¹

Cela signifie que l'ordre des opérations compte. L'expression `(a + b) + c` ne donne pas nécessairement le même résultat binaire que `a + (b + c)`. Prenons un exemple extrême : soit a = 1.0e30, b = -1.0e30, et c = 1.0.

- `(a + b) + c` : (1.0e30 - 1.0e30) donne 0.0. Puis 0.0 + 1.0 donne 1.0.
- `a + (b + c)` : (-1.0e30 + 1.0) est calculé en premier. En raison de la différence de magnitude massive, 1.0 est complètement absorbé par -1.0e30 lors de l'alignement des exposants. Le résultat de (b + c) est donc -1.0e30 (perte de précision totale). Ensuite, a + (-1.0e30) donne 0.0.

Les résultats sont différents : 1.0 contre 0.0. De même, la distributivité `(a * (b + c) = a * b + a * c)` peut également être violée. Ces non-garanties mathématiques ne sont pas des "bugs" ; elles sont une conséquence inhérente à l'arithmétique avec une précision finie. Cependant, elles agissent comme des chaînes pour le compilateur. Un compilateur ne peut pas, par défaut, réorganiser les opérations flottantes comme il le ferait pour des entiers, car cela pourrait changer le résultat numérique du programme. Cette contrainte bloque de nombreuses optimisations puissantes, notamment la vectorisation.

### L'option -ffast-math : Un pacte avec le diable

Pour libérer le compilateur de ces chaînes, il existe une option puissante mais dangereuse : `-ffast-math`. En utilisant cette option, le programmeur passe un message clair au compilateur : "Je t'autorise à ignorer les règles strictes de la norme IEEE 754 et à supposer que l'arithmétique flottante se comporte comme la véritable arithmétique des nombres réels. La vitesse est plus importante pour moi que l'exactitude bit à bit.".²¹

Cette seule option active en réalité un ensemble d'autres options qui relaxent les règles mathématiques²¹ :
- Elle suppose que l'addition et la multiplication sont associatives, permettant la réorganisation des opérations.
- Elle ignore l'existence du zéro signé, traitant +0.0 et -0.0 comme identiques, ce qui simplifie des expressions comme `x + 0.0`.
- Elle suppose que les arguments et les résultats des opérations ne sont jamais des NaNs ou des infinis (`-ffinite-math-only`), éliminant le besoin de vérifications coûteuses.
- Elle permet de remplacer une division par une multiplication par l'inverse (`x / y` devient `x * (1/y)`), ce qui peut être plus rapide mais moins précis.

Utiliser `-ffast-math` est un compromis conscient. Pour de nombreuses applications (graphisme, jeux vidéo, certains types de simulations), la légère différence numérique est imperceptible et le gain de performance est considérable. Pour d'autres (calculs financiers, simulations scientifiques de haute précision), cela peut conduire à des résultats incorrects et invalider complètement l'analyse.

### Démonstration par l'exemple : L'impact sur la vectorisation

L'impact le plus spectaculaire de `-ffast-math` est sa capacité à débloquer la vectorisation automatique des boucles. Reprenons l'exemple de la sommation d'un tableau de float.

```c
float sum_array(float* a, int n) {
    float sum = 0.0f;
    for (int i = 0; i < n; ++i) {
        sum = sum + a[i];
    }
    return sum;
}
```

**Sans -ffast-math** (avec `-O2` ou `-O3`) : Le compilateur est contraint de respecter l'ordre séquentiel des additions. Il ne peut pas additionner a[0] et a[1] en même temps, car sum pour l'itération i dépend du résultat de l'itération i-1. Il peut appliquer des optimisations comme le déroulage de boucle (loop unrolling) pour réduire le surcoût des sauts, en effectuant par exemple quatre additions séquentielles dans le corps de la boucle. Mais le calcul reste fondamentalement séquentiel.

**Avec -ffast-math -O3** : Le compilateur est maintenant autorisé à supposer que l'addition est associative. Il peut transformer la boucle. Au lieu d'une seule variable d'accumulation sum, il peut en utiliser quatre (ou huit, ou seize) pour accumuler des sommes partielles en parallèle. Par exemple, il peut calculer sum0 = a[0] + a[4] + a[8] +..., sum1 = a[1] + a[5] + a[9] +..., et ainsi de suite. Chacune de ces accumulations est indépendante et peut être effectuée en utilisant une seule voie d'un registre SIMD. Une instruction comme `addps` peut additionner quatre éléments du tableau à quatre accumulateurs partiels en une seule fois. À la fin de la boucle, les sommes partielles (sum0, sum1, sum2, sum3) sont additionnées pour obtenir le résultat final.

Le résultat numérique peut être légèrement différent de la version séquentielle, mais le gain de performance peut être d'un facteur 4, 8 ou plus, en fonction de la largeur des registres SIMD disponibles (SSE, AVX, AVX-512). C'est la permission de réorganiser les calculs, accordée par `-ffast-math`, qui est la clé permettant au compilateur d'effectuer la transformation algorithmique nécessaire à la vectorisation.

## Partie 7 : Au-delà de l'IEEE 754 : Recherche et Avenir de l'Arithmétique Numérique

La norme IEEE 754, bien qu'omniprésente et extraordinairement réussie, n'est pas la fin de l'histoire. La recherche active continue d'explorer de nouveaux formats numériques, soit pour remplacer l'IEEE 754, soit pour le compléter dans des domaines d'application spécifiques comme l'intelligence artificielle.

### Une alternative radicale : Le système de nombres Posit

Proposé par John Gustafson, le système de nombres Posit est une refonte fondamentale de l'arithmétique à virgule flottante, conçue comme un remplacement potentiel de l'IEEE 754.²²

La philosophie de conception des Posits est radicalement différente. Au lieu de champs de taille fixe pour l'exposant et la fraction, les Posits utilisent un encodage de longueur variable pour l'exposant, appelé le "regime". Les bits restants sont alloués à l'exposant et à la fraction. Cette conception aboutit à une précision dynamique : la précision est maximale pour les nombres autour de 1.0 et diminue gracieusement à mesure que l'on s'éloigne vers l'infiniment grand ou l'infiniment petit.²⁰ L'argument est que la plupart des calculs dans le monde réel se produisent dans cette "zone Ricitos de Oro" où la précision est la plus nécessaire.

Les Posits offrent plusieurs avantages théoriques²⁰ :
- **Gestion simplifiée des exceptions** : Il n'y a qu'une seule représentation pour l'infini/indéterminé (un seul motif binaire, 100...0) et pas de NaNs. Il n'y a pas non plus de nombres dénormalisés.
- **Pas d'overflow ou d'underflow** : Les Posits n'overflowent jamais vers l'infini ni n'underflowent vers zéro. La plage est délimitée par minpos et maxpos.
- **Meilleure précision à taille égale** : Pour une même taille en bits (par exemple, 32 bits), les Posits offrent souvent plus de chiffres significatifs de précision que l'IEEE 754 dans les plages de valeurs les plus courantes.
- **Reproductibilité garantie** : Les opérations sur les Posits sont définies pour donner des résultats identiques au bit près sur n'importe quelle architecture, un objectif que l'IEEE 754 n'a jamais pleinement atteint.

Malgré ces avantages, l'adoption des Posits reste un défi immense en raison de l'inertie massive de l'écosystème matériel et logiciel construit autour de l'IEEE 754.

### Formats spécialisés pour l'IA : Le bfloat16 (Brain Floating Point)

L'essor de l'intelligence artificielle et des réseaux de neurones a mis en évidence un besoin différent. Les algorithmes d'apprentissage profond sont très tolérants à une faible précision de la mantisse, mais ils nécessitent une grande plage dynamique pour éviter que les gradients (les signaux d'apprentissage) ne deviennent nuls ("vanishing gradients") ou infinis ("exploding gradients").²⁵

Le format bfloat16 (Brain Floating Point), développé par Google, est une réponse pragmatique à ce besoin.²⁵ C'est un format de 16 bits qui fait un compromis ingénieux :
- Il conserve les 8 bits d'exposant du format float 32 bits, lui donnant ainsi exactement la même plage dynamique (environ $10^{-38}$ à $10^{38}$).²⁵
- Il réduit drastiquement la fraction à 7 bits (plus le bit implicite), offrant une précision très faible d'environ 2 chiffres décimaux.²⁵

Pour les réseaux de neurones, c'est le meilleur des deux mondes : une plage suffisante pour l'entraînement et une taille de 16 bits qui divise par deux l'empreinte mémoire et la bande passante requise par rapport aux float 32 bits, accélérant considérablement les calculs. Le bfloat16 est un exemple parfait de co-conception : un besoin applicatif spécifique a conduit à la création d'un nouveau format de données, qui a ensuite été intégré nativement dans les accélérateurs matériels (TPUs de Google, GPUs NVIDIA et AMD, CPUs Intel).²⁵

### L'avenir du SIMD : Les extensions AVX (AVX2, AVX-512)

Parallèlement à la recherche sur de nouveaux formats, l'architecture SIMD continue d'évoluer. Les extensions AVX (Advanced Vector Extensions) ont doublé la largeur des registres SSE à 256 bits (ymm0-ymm15), permettant de traiter huit float ou quatre double en parallèle.²⁷ AVX2 a ajouté des capacités de manipulation d'entiers plus riches.

L'étape suivante, AVX-512, a de nouveau doublé la largeur à 512 bits (registres zmm0-zmm31), permettant des opérations sur seize float ou huit double simultanément.²⁸ Plus important encore, AVX-512 a introduit des fonctionnalités plus sophistiquées, comme le masquage, qui permet d'appliquer une opération de manière conditionnelle à certains éléments d'un vecteur, et des instructions de "gather/scatter" pour charger/stocker des données depuis des emplacements mémoire non contigus.²⁸ Ces extensions sont essentielles pour extraire le maximum de performance des processeurs modernes dans les domaines du calcul haute performance (HPC), des simulations et de l'IA.

### Tableau 3 : Panorama des Formats à Virgule Flottante

Ce tableau comparatif permet de visualiser les compromis de conception entre les formats établis et les innovations récentes. Il illustre pourquoi il n'existe pas de "meilleur" format universel, mais plutôt une palette de solutions adaptées à des cas d'usage spécifiques.

| Format | Taille (bits) | Signe | Exposant | Fraction | Plage (approx.) | Précision (décimales) | Cas d'usage principal |
|--------|---------------|-------|----------|----------|-----------------|----------------------|----------------------|
| Half (binary16) | 16 | 1 | 5 | 10 | $6 \times 10^{-8}$ à $6 \times 10^{4}$ | ~3 | Graphisme, stockage de données |
| bfloat16 | 16 | 1 | 8 | 7 | $10^{-38}$ à $10^{38}$ | ~2 | Intelligence Artificielle |
| Single (binary32) | 32 | 1 | 8 | 23 | $10^{-38}$ à $10^{38}$ | ~7 | Usage général, jeux vidéo |
| Double (binary64) | 64 | 1 | 11 | 52 | $10^{-308}$ à $10^{308}$ | ~16 | Calcul scientifique de précision |
| Posit<16,1> | 16 | 1 | Var. (Regime) | Var. | Très large | ~3-4 (près de 1) | Recherche, alternative potentielle |

## Conclusion et Exercices Pratiques

### Synthèse des concepts clés

Nous avons parcouru un long chemin, depuis l'impossibilité mathématique de représenter fidèlement les nombres réels sur une machine finie, jusqu'aux solutions d'ingénierie pragmatiques et élégantes incarnées par la norme IEEE 754. Nous avons vu que chaque aspect de cette norme — le bit implicite, l'exposant biaisé, les nombres dénormalisés, les zéros signés, les infinis et les NaNs — est le fruit d'une réflexion approfondie visant à créer un système robuste, cohérent et prévisible.

Nous avons ensuite plongé dans les couches inférieures, observant la transition historique de l'inefficace FPU x87 à l'architecture SSE/AVX moderne, qui a non seulement simplifié la tâche des compilateurs mais a également ouvert la voie à des gains de performance massifs grâce au parallélisme SIMD. Nous avons compris que des optimisations comme `-ffast-math` sont un outil puissant, mais qui nécessite de comprendre le compromis fondamental entre la performance et l'exactitude mathématique. 

Enfin, nous avons entrevu l'avenir, avec des formats spécialisés comme le bfloat16 qui redéfinissent les compromis pour des domaines spécifiques comme l'IA, et des alternatives radicales comme les Posits qui remettent en question les fondements mêmes de notre arithmétique.

La maîtrise des nombres à virgule flottante est ce qui distingue un programmeur compétent d'un expert. J'espère que cette conférence vous a fourni les outils conceptuels pour devenir cet expert.

### Exercices proposés

Pour solidifier votre compréhension, je vous propose les exercices suivants :

1. **Analyse binaire** : Écrivez un programme en C qui prend un float en entrée (par exemple, lu depuis la console) et affiche son bit de signe, son champ d'exposant en binaire et sa valeur réelle (après soustraction du biais), ainsi que son champ de fraction en binaire. Utilisez la méthode `memcpy` et des masques binaires pour extraire chaque champ. Testez-le avec des valeurs comme 1.0, -2.5, 0.1 et des cas limites.

2. **Exploration de l'arrondi** : En utilisant l'en-tête `<fenv.h>`, écrivez un programme qui calcule la valeur de `1.0f / 3.0f`. Affichez la représentation hexadécimale du résultat après avoir configuré successivement chacun des quatre modes d'arrondi (`FE_TONEAREST`, `FE_TOWARDZERO`, `FE_UPWARD`, `FE_DOWNWARD`). Observez et expliquez les différences.

3. **Analyse Assembleur** : Écrivez une fonction C très simple : `float add(float a, float b) { return a + b; }`. Compilez ce fichier avec `gcc -S -O2 -mno-avx -fno-asynchronous-unwind-tables -o add.s`. Ouvrez le fichier `add.s` et analysez le code assembleur généré. Identifiez l'instruction SSE utilisée (probablement `addss`), les registres xmm utilisés pour les arguments et la valeur de retour, conformément à l'ABI System V.

4. **Mesure d'impact de -ffast-math** : Créez un programme qui alloue un grand tableau (par exemple, 10 millions d'éléments) de float et l'initialise avec des valeurs aléatoires. Écrivez une fonction qui calcule la somme de tous les éléments de ce tableau. Mesurez le temps d'exécution de cette fonction en compilant votre programme une fois avec `gcc -O3` et une autre fois avec `gcc -O3 -ffast-math -march=native`. Comparez les temps. Ensuite, générez l'assembleur pour les deux versions et identifiez la présence (ou l'absence) d'instructions SIMD vectorielles (`addps`) dans la boucle de sommation.
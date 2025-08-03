# Théorie des Langages Formels et des Automates : Des Fondements Théoriques à l'Implémentation en C

## Introduction aux Langages Formels et à la Théorie des Automates

L'informatique moderne repose sur des niveaux d'abstraction successifs qui nous permettent de maîtriser la complexité, depuis le transistor jusqu'aux systèmes distribués à l'échelle mondiale. Dans le domaine de la manipulation de données textuelles, les fonctions de bibliothèque standard offrent une première couche d'abstraction utile, mais qui révèle rapidement ses limites face à des problèmes de reconnaissance de motifs complexes. 

Ce cours se propose de plonger sous la surface des manipulations de chaînes de caractères pour explorer un domaine fondamental de l'informatique théorique : la théorie des langages formels et des automates. Nous découvrirons comment des problèmes de programmation concrets et ardus peuvent être résolus de manière élégante et efficace en adoptant un formalisme mathématique rigoureux, et comment traduire cette théorie en code C robuste et performant.

## La Motivation : Les Limites de la Programmation Classique sur les Chaînes

En programmation, de nombreuses tâches impliquent la recherche et la validation de motifs au sein de chaînes de caractères. Les bibliothèques standards, comme `<string.h>` en C, fournissent des outils pour des opérations simples telles que la recherche d'une sous-chaîne exacte ou la concaténation. Cependant, ces outils deviennent rapidement insuffisants lorsque les critères de recherche se complexifient.

Considérons un premier problème concret : comment identifier, au sein d'un texte, toutes les sous-chaînes qui débutent par le caractère 'a', se terminent par la séquence 'bc', et contiennent entre elles un nombre quelconque d'autres caractères ? Une approche naïve impliquerait de parcourir le texte, de trouver chaque 'a', puis de rechercher la prochaine occurrence de 'bc', une méthode qui peut s'avérer lourde et inefficace.

Un second exemple, plus proche de la conception des langages de programmation, est la création d'un analyseur lexical (ou tokenizer) pour un langage d'assemblage personnalisé. La tâche consiste à décomposer une ligne de code, telle que `MOV AX, 42`, en une séquence de "lexèmes" ou "jetons" : `MOV`, `AX`, `,`, `42`. Chaque lexème appartient à une catégorie (instruction, registre, opérateur, délimiteur) définie par des règles précises. Comment écrire un programme capable de réaliser cette segmentation de manière fiable, en distinguant par exemple un identifiant d'un mot-clé ou d'un opérateur ?

Ces deux scénarios illustrent une classe de problèmes où il ne s'agit plus de chercher une chaîne littérale, mais de reconnaître des chaînes qui se conforment à un patron (pattern) abstrait. Pour aborder ces défis de manière systématique, il est nécessaire de se doter d'un cadre théorique plus puissant. Ce cadre est celui de la théorie des automates, qui fournit les outils conceptuels et algorithmiques pour modéliser et résoudre ces problèmes de reconnaissance de motifs.

## Définitions Fondamentales : Le Langage comme Objet Mathématique

La théorie des langages formels, initiée dans les années 1950 par des figures comme Noam Chomsky en s'appuyant sur des travaux antérieurs d'Axel Thue, Alan Turing et Emil Post, propose d'aborder ces problèmes en redéfinissant la notion de "langage" dans un contexte purement mathématique. Cette abstraction est la clé qui permet d'unifier des problèmes de programmation apparemment disparates sous un même formalisme.

### Alphabet ($\Sigma$)

La brique de base de tout langage formel est l'alphabet.

**Définition :** Un alphabet, noté $\Sigma$, est un ensemble fini et non vide de symboles. Ces symboles sont considérés comme des entités atomiques et indivisibles.

Quelques exemples d'alphabets incluent :
- L'alphabet binaire : $\Sigma = \{0, 1\}$
- L'alphabet des lettres latines minuscules : $\Sigma = \{a, b, c, \ldots, z\}$
- L'alphabet des chiffres décimaux : $\Sigma = \{0, 1, 2, 3, 4, 5, 6, 7, 8, 9\}$
- L'alphabet ASCII, qui est un ensemble de 128 caractères

### Chaîne de caractères (Mot)

À partir d'un alphabet, nous pouvons construire des séquences.

**Définition :** Une chaîne de caractères (ou un mot) sur un alphabet $\Sigma$ est une séquence finie de symboles pris dans $\Sigma$. La longueur d'une chaîne $w$, notée $|w|$, est le nombre de symboles qu'elle contient.

Une chaîne spéciale est la chaîne vide, notée $\epsilon$ (parfois $\lambda$). C'est une chaîne de longueur 0, qui ne contient aucun symbole. L'ensemble de toutes les chaînes possibles sur un alphabet $\Sigma$ est noté $\Sigma^*$.

### Langage ($L$)

La notion de langage est alors définie de manière très simple et générale.

**Définition :** Un langage $L$ sur un alphabet $\Sigma$ est un sous-ensemble de $\Sigma^*$ (c'est-à-dire un ensemble de chaînes formées à partir des symboles de $\Sigma$).

Cette définition est extrêmement puissante. Le langage peut être fini (contenant un nombre fini de mots) ou infini. Par exemple, sur l'alphabet $\Sigma = \{a, b\}$ :

- $L_1 = \{a, ab, bba\}$ est un langage fini
- $L_2 = \{\text{toutes les chaînes de longueur paire}\}$ est un langage infini
- $L_3 = \{\text{toutes les chaînes contenant la sous-chaîne "aba"}\}$ est un langage infini

Il est possible de tracer un parallèle intéressant avec la dichotomie établie par le linguiste Ferdinand de Saussure entre *langue* et *parole*. Dans cette analogie, la *langue* représente le système de règles abstraites et de conventions qui définit la structure d'un langage (par exemple, la grammaire française). Cela correspond, dans notre formalisme, à la définition du langage, que ce soit par une grammaire ou un automate. La *parole*, en revanche, se réfère aux instances concrètes d'utilisation de la langue, aux énoncés produits par les locuteurs. Cela correspond aux mots (chaînes) qui appartiennent effectivement au langage $L$.

## Exemples de Langages et Problématiques Associées

La définition mathématique d'un langage nous permet de reformuler nos problèmes de programmation initiaux.

### Le langage le plus simple : $\Sigma^*$

Le langage le plus simple sur un alphabet est celui qui contient toutes les chaînes possibles, y compris la chaîne vide. Par exemple, sur l'alphabet binaire $\Sigma = \{0, 1\}$, le langage $\Sigma^*$ est l'ensemble de toutes les représentations binaires possibles : $\{\epsilon, 0, 1, 00, 01, 10, 11, \ldots\}$. Ce langage n'impose aucune contrainte.

### Langages avec contraintes

L'intérêt de la théorie réside dans les langages qui imposent des contraintes structurelles. Reprenons les exemples de la conférence :

- Soit $\Sigma = \{a, b\}$. Le langage $L_1$ est défini comme l'ensemble de toutes les chaînes qui se terminent obligatoirement par un 'b'. Le mot $ab$ appartient à $L_1$, tout comme $b$ (la partie précédente peut être vide), mais le mot $baa$ n'y appartient pas.

- Soit $\Sigma = \{a, b, c\}$. Le langage $L_2$ est défini par des chaînes composées d'un nombre quelconque de 'a' ou de 'b', suivies obligatoirement d'un 'b', puis d'un nombre quelconque de 'b' ou de 'c'. Le mot $aabc$ n'appartient pas à $L_2$ car il ne respecte pas la structure finale.

Même un problème arithmétique peut être vu sous cet angle. Le problème de déterminer si un entier est un nombre de Fibonacci peut être reformulé comme un problème d'appartenance : sur l'alphabet décimal $\Sigma = \{0, \ldots, 9\}$, on définit le langage $L_{\text{Fibonacci}}$ comme l'ensemble des chaînes qui sont des représentations décimales de nombres de Fibonacci. La question "13 est-il un nombre de Fibonacci ?" devient "la chaîne '13' appartient-elle à $L_{\text{Fibonacci}}$ ?".

Cette reformulation met en lumière trois problématiques fondamentales qui animent la théorie des langages formels :

1. **Le Problème de l'Appartenance (Membership Problem) :** Étant donné une description finie d'un langage $L$ et un mot $w$, décider si $w \in L$. C'est le problème de décision central que nous chercherons à résoudre.

2. **Le Problème de la Génération :** Concevoir une procédure pour énumérer, de manière séquentielle, tous les mots (et uniquement les mots) d'un langage $L$.

3. **Le Problème de l'Équivalence :** Étant donné deux descriptions de langages $L_1$ et $L_2$, décider si ces deux langages sont identiques, c'est-à-dire si $L_1 = L_2$.

Le passage d'un problème de programmation spécifique ("valider un email") à une question d'appartenance à un langage formel est une étape d'abstraction cruciale. Elle nous permet de sortir des détails d'implémentation pour nous concentrer sur la structure intrinsèque du problème. Une fois le problème formalisé, nous pouvons faire appel à un puissant arsenal d'outils mathématiques, au premier rang desquels se trouvent les automates, qui ont été conçus spécifiquement pour répondre à ces questions de manière rigoureuse et algorithmique.

L'apprentissage de cette théorie n'est donc pas un simple exercice académique, mais une démarche pragmatique pour acquérir des outils conceptuels permettant de concevoir des solutions plus générales et robustes.

## Les Automates Finis Déterministes (AFD)

Pour résoudre le problème de l'appartenance pour une classe importante de langages, nous introduisons notre premier modèle de calcul : l'automate fini. Cet objet mathématique, bien que simple, est au cœur de nombreuses applications, de l'analyse lexicale dans les compilateurs à la vérification de protocoles réseau.

### Définition Informelle : Une Machine à États

Un automate fini peut être visualisé comme une machine abstraite qui lit une chaîne d'entrée, un symbole à la fois, de gauche à droite. La machine ne dispose que d'un nombre fini d'états internes, qui représentent sa "mémoire" à un instant donné. Cette mémoire est très limitée : à tout moment, la machine ne connaît que l'état dans lequel elle se trouve, et rien d'autre sur les symboles qu'elle a déjà lus.

Le fonctionnement d'un automate fini est régi par un ensemble de règles de transition. Ces règles dictent, pour chaque état et chaque symbole lu, quel sera le prochain état de la machine. Le processus se déroule comme suit :

1. La machine commence dans un état initial spécifié
2. Elle lit le premier symbole de la chaîne d'entrée
3. En fonction de son état actuel et du symbole lu, elle passe à un nouvel état, déterminé par sa fonction de transition
4. Elle passe au symbole suivant de la chaîne et répète l'étape 3
5. Le processus continue jusqu'à ce que toute la chaîne ait été lue

Une fois la lecture terminée, si la machine se trouve dans un état accepteur (ou final), on dit que la chaîne est acceptée par l'automate. Sinon, elle est rejetée.

L'ensemble de toutes les chaînes acceptées par un automate constitue le langage reconnu par cet automate. Un automate fini est dit déterministe (AFD) si, pour chaque paire (état, symbole), il n'existe qu'une seule et unique transition possible. Il n'y a ni ambiguïté ni choix.

### Définition Formelle : Le Quintuplet de Rabin et Scott

L'intuition de la machine à états est formalisée de manière rigoureuse dans la littérature scientifique. L'article fondateur de Michael O. Rabin et Dana S. Scott, "Finite Automata and Their Decision Problems" (1959), a établi la définition standard qui est encore utilisée aujourd'hui. Cette définition est la pierre angulaire de la théorie et se retrouve dans tous les manuels de référence, tels que ceux de Hopcroft & Ullman ou de Sipser.

**Définition :** Un Automate Fini Déterministe (AFD) est un quintuplet (un 5-tuple) $M = (Q, \Sigma, \delta, q_0, F)$ où :

- $Q$ est un ensemble fini d'états
- $\Sigma$ est un ensemble fini de symboles, appelé l'alphabet d'entrée
- $\delta : Q \times \Sigma \rightarrow Q$ est la fonction de transition. Cette fonction prend en argument un état de $Q$ et un symbole de $\Sigma$, et retourne un état de $Q$. Le domaine $Q \times \Sigma$ représente toutes les paires possibles d'un état et d'un symbole. Le caractère déterministe de l'automate est capturé par le fait que $\delta$ est une fonction, associant un unique état de sortie à chaque entrée.
- $q_0 \in Q$ est l'état initial, l'état dans lequel l'automate commence son exécution
- $F \subseteq Q$ est l'ensemble des états accepteurs ou finaux

### Exécution d'un Automate et Reconnaissance de Langage

Pour formaliser l'exécution d'un automate sur une chaîne complète, on étend la fonction de transition $\delta$, qui opère sur des symboles uniques, à une fonction $\hat{\delta}$ (delta chapeau), qui opère sur des chaînes de symboles.

**Définition :** La fonction de transition étendue $\hat{\delta} : Q \times \Sigma^* \rightarrow Q$ est définie de manière récursive :

- Pour la base de la récursion : $\hat{\delta}(q, \epsilon) = q$ pour tout état $q \in Q$. (Lire la chaîne vide ne change pas l'état)
- Pour la partie récursive : Pour une chaîne $w$ de la forme $xa$ (où $x$ est une chaîne et $a$ est un symbole), et pour un état $q \in Q$, on a $\hat{\delta}(q, xa) = \delta(\hat{\delta}(q, x), a)$. (Pour traiter la chaîne $xa$, on traite d'abord la chaîne $x$ pour arriver à un état intermédiaire, puis on applique la transition $\delta$ sur le dernier symbole $a$)

Avec cette fonction étendue, la notion d'acceptation devient précise :

**Définition :** Un mot $w \in \Sigma^*$ est accepté par un AFD $M = (Q, \Sigma, \delta, q_0, F)$ si et seulement si $\hat{\delta}(q_0, w) \in F$.

Le langage reconnu par l'automate $M$, noté $L(M)$, est l'ensemble de tous les mots acceptés par $M$ :

$$L(M) = \{w \in \Sigma^* \mid \hat{\delta}(q_0, w) \in F\}$$

### Exemple d'Application

Appliquons cette définition à l'exemple de la conférence, qui reconnaît les chaînes sur $\{a, b\}$ se terminant par 'b'.

L'automate peut être représenté par un diagramme d'états et formalisé comme suit :

$M = (Q, \Sigma, \delta, q_0, F)$ avec :
- $Q = \{q_1, q_2\}$
- $\Sigma = \{a, b\}$
- $q_0 = q_1$
- $F = \{q_2\}$

La fonction de transition $\delta$ est définie par :
- $\delta(q_1, a) = q_1$
- $\delta(q_1, b) = q_2$
- $\delta(q_2, a) = q_1$
- $\delta(q_2, b) = q_2$

Traçons l'exécution pour deux chaînes :

**Chaîne $w = "aaab"$ :**
- $\hat{\delta}(q_1, \epsilon) = q_1$
- $\hat{\delta}(q_1, a) = \delta(\hat{\delta}(q_1, \epsilon), a) = \delta(q_1, a) = q_1$
- $\hat{\delta}(q_1, aa) = \delta(\hat{\delta}(q_1, a), a) = \delta(q_1, a) = q_1$
- $\hat{\delta}(q_1, aaa) = \delta(\hat{\delta}(q_1, aa), a) = \delta(q_1, a) = q_1$
- $\hat{\delta}(q_1, aaab) = \delta(\hat{\delta}(q_1, aaa), b) = \delta(q_1, b) = q_2$

L'état final est $q_2$. Comme $q_2 \in F$, la chaîne "aaab" est acceptée.

**Chaîne $w = "baa"$ :**
- $\hat{\delta}(q_1, b) = \delta(q_1, b) = q_2$
- $\hat{\delta}(q_1, ba) = \delta(\hat{\delta}(q_1, b), a) = \delta(q_2, a) = q_1$
- $\hat{\delta}(q_1, baa) = \delta(\hat{\delta}(q_1, ba), a) = \delta(q_1, a) = q_1$

L'état final est $q_1$. Comme $q_1 \notin F$, la chaîne "baa" est rejetée.

Une manière alternative et souvent plus claire de représenter la fonction de transition $\delta$ est une table de transition. Cette table met en correspondance chaque état avec chaque symbole de l'alphabet, et constitue un pont direct entre la théorie et l'implémentation.

| État | Type | Transition sur 'a' | Transition sur 'b' |
|------|------|-------------------|-------------------|
| $q_1$ | Initial | $q_1$ | $q_2$ |
| $q_2$ | Accepteur | $q_1$ | $q_2$ |

Cette table encapsule toute la logique de l'automate. Elle montre sans ambiguïté que pour n'importe quel état et n'importe quel symbole d'entrée, il y a exactement un état de destination, ce qui est l'essence même du déterminisme. Comme nous le verrons, cette structure de données est directement traduisible en un tableau en C, rendant la simulation de l'automate remarquablement simple et efficace.

## Implémentation des Automates Finis en C

La traduction du modèle théorique de l'automate fini en un programme fonctionnel est un exercice d'ingénierie logicielle instructif. Il existe plusieurs approches, chacune avec ses propres compromis en termes de lisibilité, de maintenabilité et de performance. Nous allons explorer deux méthodes principales, en nous basant sur les exemples et les réflexions de la conférence.

### Approche Naïve : Simulation avec switch Imbriqués

La première approche, la plus intuitive, consiste à traduire directement la logique du diagramme d'états en structures de contrôle C. Pour un automate complexe comme celui présenté dans la conférence avec trois états et des transitions multiples, la logique de décision dépend à la fois de l'état actuel et du caractère lu.

La structure du code suit naturellement cette double dépendance. On utilise une boucle principale pour itérer sur les caractères de la chaîne d'entrée. À l'intérieur de cette boucle, un premier switch sélectionne le bloc de code correspondant à l'état actuel. Dans chaque case de ce switch, un second switch imbriqué gère les transitions en fonction du caractère lu.

Voici le pseudocode illustrant cette approche :

```c
FONCTION est_accepte_switch(chaine_entree):
    // Définir les états (par exemple, avec un enum)
    enum Etats { ETAT_1, ETAT_2, ETAT_3, ETAT_ERREUR };
    etat_courant = ETAT_1; // État initial

    POUR CHAQUE caractere DANS chaine_entree:
        SELON etat_courant:
            CAS ETAT_1:
                SELON caractere:
                    CAS 'a': etat_courant = ETAT_2;
                    CAS 'b': etat_courant = ETAT_1;
                    AUTRE: etat_courant = ETAT_ERREUR;
                FIN SELON
                SORTIR; // break
            CAS ETAT_2:
                SELON caractere:
                    CAS 'a': etat_courant = ETAT_3;
                    CAS 'b': etat_courant = ETAT_2;
                    AUTRE: etat_courant = ETAT_ERREUR;
                FIN SELON
                SORTIR; // break
            CAS ETAT_3:
                //... et ainsi de suite pour les autres états
                SORTIR; // break
            CAS ETAT_ERREUR:
                SORTIR; // break
        FIN SELON
        SI etat_courant == ETAT_ERREUR ALORS SORTIR de la boucle;

    // Vérifier si l'état final est un état accepteur
    RETOURNER (etat_courant EST DANS {ETAT_ACCEPTEUR_1,...});
```

Ci-dessous, une implémentation C complète pour l'automate complexe de la conférence :

```c
#include <stdio.h>
#include <stdbool.h>

// Définition des états pour plus de lisibilité
typedef enum {
    STATE_1 = 1,
    STATE_2,
    STATE_3,
    // On pourrait ajouter un état puits (sink/error state)
} State;

// L'état 1 est l'état accepteur dans cet exemple
bool is_accepting(State s) {
    return s == STATE_1;
}

bool recognize_complex_automaton(const char* input) {
    State current_state = STATE_1; // État initial
    int i = 0;
    char c;

    while ((c = input[i++]) != '\0') {
        switch (current_state) {
            case STATE_1:
                switch (c) {
                    case 'a':
                        current_state = STATE_2;
                        break;
                    case 'b':
                        current_state = STATE_1; // Boucle sur lui-même
                        break;
                    default:
                        // Transition vers un état non accepteur si le symbole n'est pas dans l'alphabet
                        // Pour simplifier, on peut retourner false directement
                        return false; 
                }
                break;

            case STATE_2:
                switch (c) {
                    case 'a':
                        // Dans la conférence, il y a une transition de 2 vers 3 sur 'a'
                        // mais le diagramme est ambigu. Nous suivons une interprétation
                        // cohérente : state 2, 'a' -> state 3 (non montré dans le code de la conf)
                        // state 2, 'b' -> state 2 (boucle)
                        current_state = STATE_3; // Hypothèse basée sur le diagramme
                        break;
                    case 'b':
                        current_state = STATE_2;
                        break;
                    default:
                        return false;
                }
                break;

            case STATE_3:
                switch (c) {
                    case 'a':
                        current_state = STATE_3; // Boucle sur lui-même
                        break;
                    case 'b':
                        current_state = STATE_1;
                        break;
                    default:
                        return false;
                }
                break;
        }
    }

    return is_accepting(current_state);
}

int main() {
    // Test avec une chaîne acceptée : "ab" -> q1 -> q2 -> q2 (non accepteur)
    // "abb" -> q1 -> q2 -> q2 -> q2 (non accepteur)
    // "aba" -> q1 -> q2 -> q3 (non accepteur)
    // "abab" -> q1 -> q2 -> q2 -> q1 (accepteur)
    const char* test_str1 = "abb"; // Devrait être rejeté
    const char* test_str2 = "abab"; // Devrait être accepté
    const char* test_str3 = "ababb"; // q1 -> a -> q2 -> b -> q2 -> a -> q3 -> b -> q1 -> b -> q1 (accepteur)

    printf("'%s' est acceptee? %s\n", test_str1, recognize_complex_automaton(test_str1) ? "Oui" : "Non");
    printf("'%s' est acceptee? %s\n", "ababb", recognize_complex_automaton("ababb") ? "Oui" : "Non");

    return 0;
}
```

Bien que fonctionnelle, cette approche présente des inconvénients majeurs, comme souligné dans la conférence : le code est verbeux, répétitif et difficile à maintenir. L'ajout d'un nouvel état ou d'un nouveau symbole à l'alphabet nécessite de modifier la structure switch à plusieurs endroits, ce qui est une source potentielle d'erreurs.

### Approche Élégante : La Table de Transitions

Une méthode bien supérieure consiste à séparer la définition de l'automate de son moteur d'exécution. La définition de l'automate est entièrement contenue dans sa fonction de transition $\delta$. Comme nous l'avons vu, cette fonction peut être représentée par une table. En C, cette table peut être implémentée par un tableau à deux dimensions `transitions`.

Le moteur d'exécution devient alors une simple boucle qui lit les caractères et met à jour l'état en consultant le tableau.

```c
FONCTION est_accepte_par_table(chaine_entree):
    etat_courant = ETAT_INITIAL;
    // La définition de l'automate est maintenant une structure de données
    table_transitions = [
        /* état 1 */ [état_sur_a, état_sur_b, ...],
        /* état 2 */ [état_sur_a, état_sur_b, ...],
       ...
    ];

    POUR CHAQUE caractere DANS chaine_entree:
        indice_char = mapper_char_vers_indice(caractere);
        // Si le caractère n'est pas valide, aller à un état d'erreur
        SI indice_char est invalide ALORS
            etat_courant = ETAT_ERREUR;
            SORTIR de la boucle;
        FIN SI
        // La logique de transition est une simple consultation de tableau
        etat_courant = table_transitions[etat_courant][indice_char];
    FIN POUR

    RETOURNER (etat_courant EST DANS ETATS_ACCEPTEURS);
```

Voici l'implémentation C complète pour l'automate de la conférence qui reconnaît le langage $(a|b)^*b(b|c)^*$, avec un état puits pour gérer les entrées invalides.

```c
#include <stdio.h>
#include <stdbool.h>
#include <string.h>

// Les états sont numérotés de 0 à 4.
// Q = {q0, q1, q2, q3, q4} où q0 est l'état initial.
// Pour correspondre à l'exemple de la conf, on utilise 1-5 et on ajuste les indices.
// State 1: initial
// State 2: vu (a|b)*
// State 3: vu (a|b)*b
// State 4: vu (a|b)*b(b|c)* (accepteur)
// State 5: puits (erreur)

#define NUM_STATES 5
#define NUM_SYMBOLS 3 // a, b, c

// On numérote les états de 1 à 5. Le tableau aura une taille 6 pour un accès facile.
// L'indice 0 du tableau n'est pas utilisé.
int transitions[6][NUM_SYMBOLS];

// Fonction pour mapper les caractères 'a', 'b', 'c' aux indices 0, 1, 2
int char_to_index(char c) {
    switch (c) {
        case 'a': return 0;
        case 'b': return 1;
        case 'c': return 2;
        default: return -1; // Caractère invalide
    }
}

void setup_automaton() {
    // Automate pour (a|b)*b(b|c)*.
    // L'automate de la conf est un peu différent, nous l'implémentons ici.
    // États: 1 (initial), 2 (vu a), 3 (vu b, accepteur), 4 (vu c après b, accepteur), 5 (puits)
    // Le diagramme de la conf est pour (a|b)*c(b|c)*, nous adaptons pour (a|b)*b(b|c)*
    
    // État 1 (Initial)
    transitions[1][char_to_index('a')] = 1; // (a|b)*
    transitions[1][char_to_index('b')] = 2; // Transition vers la partie suivante
    transitions[1][char_to_index('c')] = 5; // Erreur
    
    // État 2 (Accepteur, a vu le 'b' obligatoire)
    transitions[2][char_to_index('a')] = 5; // Erreur
    transitions[2][char_to_index('b')] = 2; // (b|c)*
    transitions[2][char_to_index('c')] = 2; // (b|c)*
    
    // État 5 (Puits)
    transitions[5][char_to_index('a')] = 5;
    transitions[5][char_to_index('b')] = 5;
    transitions[5][char_to_index('c')] = 5;
}

bool recognize_with_table(const char* input) {
    int current_state = 1; // État initial
    int i = 0;
    char c;

    while ((c = input[i++]) != '\0') {
        int symbol_index = char_to_index(c);
        if (symbol_index == -1) {
            current_state = 5; // Puits
        } else {
            // La mise à jour de l'état est une simple indexation
            current_state = transitions[current_state][symbol_index];
        }
    }

    // Les états accepteurs sont ceux où l'on a vu le 'b' obligatoire.
    return current_state == 2;
}

int main() {
    setup_automaton();

    const char* str_ok1 = "b";      // Accepte
    const char* str_ok2 = "ab";     // Accepte
    const char* str_ok3 = "abbc";   // Accepte
    const char* str_fail1 = "a";    // Rejette
    const char* str_fail2 = "ac";   // Rejette
    const char* str_fail3 = "abca"; // Rejette

    printf("'%s' est acceptee? %s\n", str_ok1, recognize_with_table(str_ok1) ? "Oui" : "Non");
    printf("'%s' est acceptee? %s\n", str_ok2, recognize_with_table(str_ok2) ? "Oui" : "Non");
    printf("'%s' est acceptee? %s\n", str_ok3, recognize_with_table(str_ok3) ? "Oui" : "Non");
    printf("'%s' est acceptee? %s\n", str_fail1, recognize_with_table(str_fail1) ? "Oui" : "Non");
    printf("'%s' est acceptee? %s\n", str_fail2, recognize_with_table(str_fail2) ? "Oui" : "Non");
    printf("'%s' est acceptee? %s\n", str_fail3, recognize_with_table(str_fail3) ? "Oui" : "Non");

    return 0;
}
```

Ce code est nettement plus concis, modulaire et maintenable. La définition de l'automate est confinée à la fonction `setup_automaton`, et le moteur `recognize_with_table` est générique et indépendant de l'automate spécifique qu'il simule.

### Analyse Comparative : Performance et Conception Logicielle

Le choix entre l'approche switch et l'approche par table de transition n'est pas seulement une question de style, mais touche à des considérations profondes de performance et de conception.

**Lisibilité et Maintenabilité :** L'approche par table est incontestablement supérieure. Elle incarne le principe de séparation des préoccupations : les données qui définissent l'automate (la table) sont découplées de la logique qui l'exécute (le moteur). Pour modifier l'automate, il suffit de changer les valeurs dans la table, sans toucher au code du moteur.

**Performance :** La comparaison des performances est plus nuancée et révèle la connexion entre la théorie des automates et l'architecture matérielle sous-jacente.

L'approche switch est souvent très bien optimisée par les compilateurs modernes. Pour un switch avec des case denses, le compilateur peut générer une table de sauts (jump table), qui est une forme d'adressage indirect permettant de sauter directement au bon bloc de code en temps constant. Cette approche tire un excellent parti du mécanisme de prédiction de branchement des processeurs modernes, car les sauts sont indirects mais souvent prévisibles.

L'approche par tableau 2D, quant à elle, implique une lecture en mémoire à chaque transition (`current_state = transitions[current_state][symbol_index]`). La performance de cette lecture dépend crucialement de la hiérarchie de cache du processeur. Si la table de transition est petite et tient dans le cache L1, l'accès sera extrêmement rapide. Si l'automate est très grand (des centaines ou des milliers d'états), la table peut être trop grande pour le cache, entraînant des latences dues aux accès à la mémoire principale (cache misses).

En conclusion, il n'y a pas de vainqueur absolu. Pour la plupart des automates de taille raisonnable, la différence de performance sera négligeable. L'approche par table de transition doit cependant être privilégiée pour ses qualités de conception logicielle. Le fait qu'un concept théorique aussi abstrait qu'un automate fini nous amène à discuter de prédiction de branchement et de hiérarchie de cache est une illustration parfaite de la manière dont les différentes couches de l'informatique, de la théorie à l'architecture matérielle, sont profondément interconnectées. Le choix d'une structure de données pour modéliser un concept mathématique a des implications directes sur la manière dont le silicium exécutera le code.

## Les Expressions Régulières et la Bibliothèque POSIX regex.h

Alors que les automates finis fournissent un modèle de machine pour reconnaître des langages, les expressions régulières offrent une notation textuelle et concise pour décrire ces mêmes langages. Ces deux concepts sont profondément liés et représentent deux facettes d'une même idée fondamentale.

### Langages Réguliers

**Définition :** Un langage est dit régulier s'il existe un automate fini (déterministe ou non, comme nous le verrons) qui le reconnaît.

Une définition alternative, de nature inductive, caractérise les langages réguliers par la manière dont ils peuvent être construits. Les langages réguliers sur un alphabet $\Sigma$ sont ceux qui peuvent être obtenus à partir de langages de base (le langage vide $\emptyset$, le langage contenant uniquement la chaîne vide $\{\epsilon\}$, et les langages singletons $\{a\}$ pour chaque $a \in \Sigma$) en appliquant un nombre fini de fois les trois opérations régulières suivantes :

1. **Union (ou Disjonction) :** L'union de deux langages $L_1$ et $L_2$, notée $L_1 \cup L_2$, est l'ensemble des chaînes qui sont dans $L_1$ ou dans $L_2$. Dans la syntaxe des expressions régulières, cette opération est souvent représentée par le symbole `|` ou `+`.

2. **Concaténation :** La concaténation de $L_1$ et $L_2$, notée $L_1 \cdot L_2$, est l'ensemble de toutes les chaînes formées en prenant un mot de $L_1$ et en lui ajoutant à la fin un mot de $L_2$. Formellement, $L_1 \cdot L_2 = \{xy \mid x \in L_1 \text{ et } y \in L_2\}$.

3. **Fermeture de Kleene (ou Étoile) :** La fermeture de Kleene d'un langage $L$, notée $L^*$, est l'ensemble de toutes les chaînes formées en concaténant zéro ou plusieurs mots de $L$. $L^* = \bigcup_{i \geq 0} L^i = L^0 \cup L^1 \cup L^2 \cup \ldots$. L'opération "zéro ou plusieurs" est représentée par l'astérisque `*`.

Une expression régulière (ou regex) est une chaîne de caractères qui décrit un langage régulier en utilisant les symboles de l'alphabet et les opérateurs correspondant à ces trois opérations.

### Syntaxe des Expressions Régulières POSIX

Il existe plusieurs "dialectes" d'expressions régulières. La norme POSIX (Portable Operating System Interface) définit un standard pour les expressions régulières qui assure la portabilité entre les systèmes de type Unix. C'est cette norme qui est implémentée par la bibliothèque C `<regex.h>`. On distingue généralement la syntaxe de base (BRE) et la syntaxe étendue (ERE), cette dernière étant plus riche et plus intuitive. Nous nous concentrerons sur la syntaxe étendue (ERE), activée par l'option `REG_EXTENDED`.

Le tableau suivant résume les principaux éléments de la syntaxe POSIX ERE, en s'appuyant sur les descriptions de la conférence et des sources complémentaires.

| Symbole/Syntaxe | Nom | Description | Exemple |
|----------------|-----|-------------|---------|
| `.` | Point | Correspond à n'importe quel caractère unique (sauf le saut de ligne par défaut) | `a.c` correspond à "abc", "axc", etc. |
| `*` | Étoile de Kleene | Zéro ou plusieurs répétitions de l'élément précédent | `ab*c` correspond à "ac", "abc", "abbc" |
| `+` | Plus | Une ou plusieurs répétitions de l'élément précédent | `ab+c` correspond à "abc", "abbc" |
| `?` | Point d'interrogation | Zéro ou une répétition de l'élément précédent | `ab?c` correspond à "ac", "abc" |
| `[abc]` | Classe de caractères | Correspond à un seul des caractères listés ('a', 'b', ou 'c') | `[a-z]` pour une lettre minuscule |
| `[^abc]` | Classe de caractères négative | Correspond à n'importe quel caractère SAUF 'a', 'b', ou 'c' | `[^0-9]` pour un non-chiffre |
| `^` | Ancre de début | Correspond au début de la chaîne (ou de la ligne avec REG_NEWLINE) | `^Hello` |
| `$` | Ancre de fin | Correspond à la fin de la chaîne (ou de la ligne avec REG_NEWLINE) | `World$` |
| `( )` | Groupe de capture | Groupe des éléments pour appliquer un quantificateur et capture la correspondance | `(ab)+` correspond à "ab", "abab", etc. |
| `\|` | Alternative | Correspond à l'expression à gauche OU à droite | `a\|b` correspond à 'a' ou 'b' |
| `{n,m}` | Quantificateur | Correspond entre n et m fois à l'élément précédent | `a{2,4}` correspond à "aa", "aaa", "aaaa" |
| `[[:digit:]]` | Classe de caractères POSIX | Correspond à un caractère d'une classe prédéfinie (ici, un chiffre) | `[[:alpha:]]` pour une lettre |

L'utilitaire de ligne de commande `grep` est un excellent outil pour expérimenter avec ces expressions avant de les utiliser en C. Par exemple, `grep -E "^[0-9]+$"` trouvera les lignes qui ne contiennent qu'un nombre entier.

### Implémentation en C avec regex.h

La bibliothèque POSIX `<regex.h>` fournit une interface standard pour utiliser les expressions régulières en C. Le processus se déroule en quatre étapes principales, comme illustré dans la conférence et la documentation POSIX.

#### 1. Compilation (regcomp)

Avant de pouvoir utiliser une expression régulière, elle doit être "compilée" dans un format interne, une structure de données qui représente l'automate fini correspondant.

```c
int regcomp(regex_t *preg, const char *regex, int cflags);
```

- `preg` est un pointeur vers une structure `regex_t` qui contiendra l'automate compilé
- `regex` est la chaîne de caractères contenant l'expression régulière
- `cflags` est un entier pour les options de compilation. Les plus courantes sont `REG_EXTENDED` pour la syntaxe ERE et `REG_ICASE` pour une recherche insensible à la casse
- La fonction retourne 0 en cas de succès

#### 2. Exécution (regexec)

Une fois compilée, l'expression peut être exécutée sur une ou plusieurs chaînes de caractères.

```c
int regexec(const regex_t *preg, const char *string, size_t nmatch, regmatch_t pmatch[], int eflags);
```

- `preg` est un pointeur vers la structure `regex_t` compilée
- `string` est la chaîne sur laquelle effectuer la recherche
- `nmatch` et `pmatch` sont utilisés pour la capture de sous-chaînes (voir ci-dessous). Pour une simple vérification de correspondance, on peut utiliser `nmatch = 0` et `pmatch = NULL`
- `eflags` permet de spécifier des options d'exécution, comme `REG_NOTBOL` (ne pas considérer `^` comme début de chaîne)
- La fonction retourne 0 si une correspondance est trouvée, et `REG_NOMATCH` sinon

#### 3. Capture de sous-chaînes (regmatch_t)

Une des fonctionnalités les plus puissantes est la capacité d'extraire les parties de la chaîne qui ont correspondu aux groupes `(...)` dans l'expression.

La structure `regmatch_t` contient deux membres de type `regoff_t` : `rm_so` (start offset) et `rm_eo` (end offset), qui sont les indices de début et de fin de la sous-chaîne correspondante.

Si `nmatch` est supérieur à 0, `regexec` remplit le tableau `pmatch`. `pmatch[0]` contient les offsets de la correspondance globale. `pmatch[i]` (pour $i > 0$) contient les offsets de la $i$-ème parenthèse capturante.

La structure `regex_t` elle-même contient un champ `re_nsub` qui, après compilation, indique le nombre de groupes capturants dans l'expression.

#### 4. Libération de la mémoire (regfree)

La structure `regex_t` alloue de la mémoire dynamiquement lors de la compilation. Il est impératif de la libérer une fois qu'on n'en a plus besoin pour éviter les fuites de mémoire.

```c
void regfree(regex_t *preg);
```

Le lien entre les expressions régulières et les automates est fondamental. L'un des résultats les plus importants de la théorie, le **Théorème de Kleene**, stipule qu'un langage est régulier si et seulement s'il est reconnu par un automate fini. Ces deux formalismes ont donc exactement la même puissance expressive. La fonction `regcomp` n'est rien d'autre que la mise en œuvre pratique de ce théorème : elle prend une description textuelle (l'expression régulière) et la transforme en une machine de reconnaissance efficace (l'automate fini). Cette dualité permet au programmeur de choisir la représentation la plus adaptée à sa tâche : les expressions régulières pour la concision et la lisibilité de la spécification, et les automates pour l'efficacité de l'exécution.

### Cas d'étude : Validation d'adresses e-mail

Mettons en pratique ces concepts avec la tâche de validation d'adresses e-mail, proposée dans la conférence. Nous allons écrire un programme C qui lit des chaînes et vérifie si elles correspondent à un format d'e-mail plausible.

L'expression régulière pour un format d'e-mail simple pourrait être : `^([a-zA-Z0-9._%+-]+)@([a-zA-Z0-9.-]+)\.([a-zA-Z]{2,})$`.

- `^` : Début de la chaîne
- `([a-zA-Z0-9._%+-]+)` : Partie locale (nom d'utilisateur). Un ou plusieurs caractères alphanumériques ou ., _, %, +, -. Capturé dans le groupe 1.
- `@` : Le symbole littéral '@'
- `([a-zA-Z0-9.-]+)` : Nom de domaine. Un ou plusieurs caractères alphanumériques ou ., -. Capturé dans le groupe 2.
- `\.` : Un point littéral (le . doit être échappé car il a une signification spéciale)
- `([a-zA-Z]{2,})` : Le domaine de premier niveau (TLD). Au moins deux lettres. Capturé dans le groupe 3.
- `$` : Fin de la chaîne

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <regex.h>

void validate_email(const char* email) {
    regex_t regex;
    int reti;
    char msgbuf[100];

    // L'expression régulière pour l'email
    const char* pattern = "^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$";

    // Étape 1: Compilation
    reti = regcomp(&regex, pattern, REG_EXTENDED);
    if (reti) {
        fprintf(stderr, "Impossible de compiler la regex\n");
        exit(1);
    }

    // Étape 2: Exécution et capture
    const size_t nmatch = 4; // 1 pour la correspondance globale + 3 groupes
    regmatch_t pmatch[nmatch];

    reti = regexec(&regex, email, nmatch, pmatch, 0);
    if (!reti) {
        printf("Email '%s' est valide.\n", email);
        
        // Afficher les groupes capturés
        for (size_t i = 0; i < nmatch; i++) {
            if (pmatch[i].rm_so != -1) {
                int len = pmatch[i].rm_eo - pmatch[i].rm_so;
                char substring[len + 1];
                memcpy(substring, email + pmatch[i].rm_so, len);
                substring[len] = '\0';
                printf("  Groupe %zu: '%s' (offsets %lld-%lld)\n", i, substring, 
                       (long long)pmatch[i].rm_so, (long long)pmatch[i].rm_eo);
            }
        }
    } else if (reti == REG_NOMATCH) {
        printf("Email '%s' n'est pas valide.\n", email);
    } else {
        regerror(reti, &regex, msgbuf, sizeof(msgbuf));
        fprintf(stderr, "Erreur de correspondance regex: %s\n", msgbuf);
    }

    // Étape 4: Libération de la mémoire
    regfree(&regex);
}

int main() {
    validate_email("test.user@example.com");
    validate_email("user_123@sub.domain.co.uk");
    validate_email("invalid-email@");
    validate_email("another@domain");
    validate_email("still.not.valid@domain.");
    return 0;
}
```

Ce programme démontre le cycle complet d'utilisation de la bibliothèque `regex.h`, de la compilation à l'exécution, en passant par la capture de sous-chaînes et la libération des ressources.

## Les Automates Finis Non-Déterministes (AFN) et l'Équivalence des Modèles

Jusqu'à présent, nous avons étudié les automates finis déterministes (AFD), où chaque transition est unique et sans ambiguïté. Cependant, lors de la conception d'un automate pour un langage complexe, cette contrainte de déterminisme peut rendre la tâche étonnamment difficile. L'introduction du non-déterminisme offre une flexibilité de conception considérable, sans pour autant augmenter la puissance de calcul du modèle.

### Le Défi du Déterminisme

Reprenons le langage $L = (a|b)^*b(b|c)^*$, qui représente les chaînes sur $\{a, b, c\}$ contenant au moins un 'b', et où aucun 'a' n'apparaît après le premier 'b'. La construction d'un AFD pour ce langage est délicate. Lorsque l'automate lit un symbole 'b', il est confronté à un choix : ce 'b' fait-il encore partie de la séquence initiale $(a|b)^*$, ou est-ce le 'b' pivot qui initie la seconde partie $(b|c)^*$ ? Un AFD, par définition, ne peut pas explorer plusieurs possibilités simultanément. Il doit prendre une décision unique, ce qui complique sa structure.

### Introduction au Non-déterminisme

Un Automate Fini Non-déterministe (AFN) lève cette restriction. Un AFN a le pouvoir de se trouver dans plusieurs états à la fois. Formellement, cela se traduit par deux nouvelles capacités :

1. Pour une paire (état, symbole) donnée, il peut y avoir zéro, une ou plusieurs transitions possibles
2. L'automate peut effectuer des transitions spontanées (ou $\epsilon$-transitions), c'est-à-dire changer d'état sans consommer de symbole d'entrée

La définition formelle d'un AFN est similaire à celle d'un AFD, mais la fonction de transition est modifiée pour refléter ces nouvelles capacités.

**Définition :** Un Automate Fini Non-déterministe (AFN) est un quintuplet $M = (Q, \Sigma, \delta, q_0, F)$ où $Q, \Sigma, q_0, F$ sont définis comme pour un AFD, mais la fonction de transition $\delta$ est :

$$\delta : Q \times (\Sigma \cup \{\epsilon\}) \rightarrow \mathcal{P}(Q)$$

Ici, $\mathcal{P}(Q)$ est l'ensemble des parties de $Q$, c'est-à-dire l'ensemble de tous les sous-ensembles possibles de $Q$. La fonction de transition ne retourne plus un seul état, mais un ensemble d'états possibles.

**Acceptation dans un AFN :** Le concept d'acceptation est également adapté. Un mot $w$ est accepté par un AFN si, après avoir lu toute la chaîne, il existe au moins un chemin possible depuis l'état initial qui mène à un état accepteur. L'automate explore en quelque sorte toutes les trajectoires possibles en parallèle, et il lui suffit qu'une seule de ces trajectoires réussisse pour que le mot soit accepté.

### La Puissance du Non-déterminisme pour la Conception

Le non-déterminisme est un outil de conception extrêmement puissant. La construction d'un AFN pour le langage $L = (a|b)^*b(b|c)^*$ devient remarquablement simple. On peut le construire par composition :

1. Créer un automate pour $(a|b)^*$
2. Créer un automate pour $(b|c)^*$
3. Relier la fin du premier automate au début du second par une transition sur le symbole 'b'

Le non-déterminisme inhérent à la structure gère l'ambiguïté pour nous. L'automate "devine" en quelque sorte quand le 'b' pivot est rencontré.

### L'Algorithme de Rabin-Scott (Construction par Sous-ensembles)

La question naturelle qui se pose est : les AFN sont-ils plus puissants que les AFD ? Peuvent-ils reconnaître des langages que les AFD ne peuvent pas reconnaître ? La réponse, surprenante et fondamentale, est non. C'est le résultat principal de l'article de Rabin et Scott. Ils ont prouvé que pour tout AFN, il existe un AFD équivalent qui reconnaît exactement le même langage.

L'algorithme qui permet de passer d'un AFN à un AFD équivalent est connu sous le nom de **construction par sous-ensembles** (subset construction). L'idée centrale est de simuler le parallélisme de l'AFN de manière déterministe.

Les états de l'AFD correspondent à des ensembles d'états de l'AFN. Un état unique dans le nouvel AFD représente l'ensemble de tous les états dans lesquels l'AFN pourrait se trouver à un instant donné.

L'algorithme fonctionne comme suit :

1. **État initial de l'AFD :** L'état initial du nouvel AFD est la $\epsilon$-fermeture de l'état initial de l'AFN (c'est-à-dire l'état initial de l'AFN plus tous les états atteignables à partir de celui-ci via des $\epsilon$-transitions).

2. **Construction des transitions :** Pour chaque état-ensemble $S$ de l'AFD déjà créé et pour chaque symbole $a \in \Sigma$, on calcule l'état de destination $S'$. $S'$ est l'ensemble de tous les états de l'AFN qui peuvent être atteints à partir d'un état de $S$ en suivant une transition sur $a$, suivi de toutes les $\epsilon$-transitions possibles à partir de ces nouveaux états.

3. **Répétition :** On répète l'étape 2 pour chaque nouvel état-ensemble créé, jusqu'à ce qu'aucun nouvel état ne soit généré.

4. **États accepteurs de l'AFD :** Un état-ensemble de l'AFD est un état accepteur si et seulement s'il contient au moins un état accepteur de l'AFN d'origine.

Bien que le nombre d'états de l'AFD résultant puisse être exponentiel par rapport au nombre d'états de l'AFN (jusqu'à $2^{|Q|}$), cet algorithme prouve l'équivalence de leur puissance de reconnaissance.

Ce processus de conversion n'est pas seulement une curiosité théorique ; il est au cœur du fonctionnement de la fonction `regcomp` de la bibliothèque `regex.h`. Le non-déterminisme est un formidable outil de compilation. Le flux de travail interne de `regcomp` peut être vu comme un pipeline de compilation classique :

1. **Entrée de haut niveau :** Le programmeur fournit une expression régulière, une spécification textuelle, concise et facile à écrire.

2. **Première passe (Analyse) :** L'expression régulière est traduite en un AFN. Cette traduction (par exemple, via l'algorithme de Thompson) est directe et algorithmiquement simple, car la structure de l'AFN peut suivre de près la structure récursive de l'expression régulière.

3. **Seconde passe (Optimisation) :** L'AFN, bien que facile à construire, est inefficace à simuler directement. L'algorithme de Rabin-Scott est alors appliqué pour le convertir en un AFD équivalent.

4. **Sortie de bas niveau :** L'AFD résultant est une machine déterministe, potentiellement plus grande mais extrêmement rapide à exécuter (une seule transition par symbole lu).

Le non-déterminisme n'est donc pas une caractéristique des machines physiques, mais une abstraction mathématique essentielle qui sert d'étape intermédiaire pour traduire efficacement une spécification de haut niveau (une expression régulière) en un programme exécutable optimisé (un AFD).

## Les Limites des Langages Réguliers : Le Lemme de la Pompe

Les automates finis et les expressions régulières sont des outils puissants, mais leur portée n'est pas illimitée. Il existe des langages, même d'apparence simple, qu'ils ne peuvent ni reconnaître ni décrire. Comprendre ces limites est essentiel pour savoir quand utiliser ces outils et quand il est nécessaire de se tourner vers des modèles de calcul plus puissants.

### Au-delà des Automates Finis

La question "Est-ce que tout langage peut être régulier ?" trouve une réponse négative. La limitation fondamentale d'un automate fini réside dans sa mémoire finie. La seule information qu'un automate conserve sur le passé de la chaîne lue est l'état dans lequel il se trouve actuellement. Avec un nombre fini d'états, il ne peut stocker qu'une quantité finie d'informations.

Considérons le langage canonique non régulier : $L = \{a^n b^n \mid n \geq 0\}$. Ce langage contient des chaînes comme $\epsilon$, $ab$, $aabb$, $aaabbb$, etc., où le nombre de 'a' est toujours égal au nombre de 'b'. Pour reconnaître ce langage, une machine doit lire tous les 'a', les compter, puis lire tous les 'b' et vérifier que leur nombre est identique. Si $n$ peut être arbitrairement grand, la machine aurait besoin d'une mémoire potentiellement infinie pour stocker le compte des 'a'. Un automate fini, avec ses $|Q|$ états, ne peut pas se souvenir d'un nombre arbitraire de 'a'.

Un autre exemple célèbre est le langage des expressions avec des parenthèses bien équilibrées (comme dans les expressions arithmétiques ou les blocs de code). Reconnaître ce langage nécessite une mémoire capable de gérer l'imbrication, ce qui est au-delà des capacités d'un automate fini.

### Le Lemme de la Pompe (Pumping Lemma) pour les Langages Réguliers

Le Lemme de la Pompe (ou Lemme de l'étoile) est un outil mathématique formel qui capture cette limitation de la mémoire finie. Il fournit une méthode pour prouver qu'un langage donné n'est pas régulier.

**Énoncé du lemme :** Soit $L$ un langage régulier. Alors il existe une constante $p \geq 1$ (appelée la longueur de pompage) telle que toute chaîne $w \in L$ dont la longueur est au moins $p$ ($|w| \geq p$) peut être décomposée en trois sous-chaînes, $w = xyz$, satisfaisant les trois conditions suivantes :

1. Pour tout $i \geq 0$, la chaîne $xy^i z$ appartient également à $L$. (La partie $y$ peut être "pompée")
2. La longueur de $y$ est strictement positive : $|y| > 0$
3. La longueur de la partie initiale $xy$ ne dépasse pas la longueur de pompage : $|xy| \leq p$

**Intuition :** Si un langage est régulier, il est reconnu par un AFD avec un nombre fini d'états, disons $p$ états. Si cet automate lit une chaîne $w$ de longueur $p$ ou plus, il doit nécessairement visiter au moins un état plus d'une fois (d'après le principe des tiroirs). La première fois qu'un état est répété, la partie de la chaîne lue entre les deux visites de cet état forme une boucle dans le diagramme de l'automate. Cette sous-chaîne correspond à la partie $y$. Comme $y$ correspond à une boucle, on peut la parcourir zéro fois ($i = 0$, ce qui donne $xz$), une fois ($i = 1$, la chaîne originale $xyz$), deux fois ($i = 2$, $xyyz$), etc., et la machine terminera toujours dans le même état final. Par conséquent, toutes ces nouvelles chaînes "pompées" doivent également être acceptées et appartenir au langage.

### Application du Lemme : Preuve par l'Absurde

Le lemme est presque toujours utilisé pour démontrer la non-régularité d'un langage via une preuve par l'absurde. La stratégie est la suivante :

1. Supposer que le langage $L$ est régulier
2. Le lemme garantit l'existence d'une longueur de pompage $p$
3. Choisir une chaîne $w \in L$ astucieusement, avec $|w| \geq p$
4. Montrer que, pour toutes les décompositions possibles de $w$ en $xyz$ qui respectent les conditions du lemme, on peut trouver un entier $i$ tel que la chaîne pompée $xy^i z$ n'appartient pas à $L$
5. Ceci crée une contradiction, prouvant que l'hypothèse de départ était fausse

**Démonstration pour $L = \{a^n b^n \mid n \geq 0\}$ :**

1. **Hypothèse :** Supposons par l'absurde que $L$ est régulier.

2. **Existence de $p$ :** Le lemme de la pompe nous assure qu'il existe une longueur de pompage $p \geq 1$.

3. **Choix de $w$ :** Choisissons la chaîne $w = a^p b^p$. Nous avons $w \in L$ et $|w| = 2p \geq p$.

4. **Décomposition :** Selon le lemme, $w$ peut être décomposé en $w = xyz$ avec $|y| > 0$ et $|xy| \leq p$.

5. **Analyse de la décomposition :** Puisque les $p$ premiers caractères de $w$ sont tous des 'a' et que $|xy| \leq p$, la sous-chaîne $y$ doit être entièrement contenue dans la partie initiale de 'a'. Autrement dit, $x$, $y$ et la première partie de $z$ ne sont composés que de 'a'. On peut donc écrire $y = a^k$ pour un certain entier $k$. De plus, comme $|y| > 0$, on sait que $k \geq 1$.

6. **Pompage :** Choisissons de pomper la chaîne avec $i = 2$. La nouvelle chaîne est $w' = xy^2 z$. Puisque nous avons ajouté une copie de $y$ (qui est $a^k$), la nouvelle chaîne aura $k$ 'a' de plus que la chaîne originale, mais le même nombre de 'b'. La forme de $w'$ est donc $a^{p+k} b^p$.

7. **Contradiction :** Puisque $k \geq 1$, le nombre de 'a' ($p + k$) n'est plus égal au nombre de 'b' ($p$). Par conséquent, $w' \notin L$.

8. **Conclusion :** Nous avons trouvé une valeur de $i$ pour laquelle $xy^i z \notin L$. Ceci contredit la conclusion du lemme de la pompe. L'hypothèse initiale est donc fausse, et le langage $L = \{a^n b^n\}$ n'est pas régulier.

Cette limitation n'est pas un échec, mais une classification. Elle nous indique que certains problèmes nécessitent des modèles de calcul plus puissants, dotés de structures de mémoire plus complexes. Cela ouvre la voie à la Hiérarchie de Chomsky, qui classe les langages en fonction de la complexité de la machine requise pour les reconnaître : des automates à pile pour les langages context-free (comme $a^n b^n$) jusqu'aux machines de Turing pour les langages récursivement énumérables.

## Algorithmes de Recherche de Chaînes Avancés : Knuth-Morris-Pratt (KMP)

Le problème de la recherche d'une sous-chaîne (un motif ou pattern) dans un texte plus long est une tâche fondamentale en informatique. Si l'approche naïve est simple à comprendre, elle peut s'avérer très inefficace. L'algorithme de Knuth-Morris-Pratt (KMP) est une solution classique et élégante qui utilise des idées issues de la théorie des automates pour atteindre une complexité temporelle linéaire, ce qui représente une amélioration spectaculaire.

### Le Problème de la Recherche de Sous-chaîne et l'Approche Naïve

Étant donné un texte $T$ de longueur $n$ et un motif $P$ de longueur $m$, l'objectif est de trouver toutes les occurrences de $P$ dans $T$. L'algorithme naïf, ou par force brute, fonctionne comme suit : il aligne le début de $P$ avec le premier caractère de $T$ et compare les caractères un par un. En cas de non-concordance ou de correspondance complète, il décale $P$ d'une position vers la droite dans $T$ et recommence le processus de comparaison depuis le début de $P$.

Dans le cas moyen, cette approche est souvent acceptable. Cependant, dans le pire des cas, sa complexité temporelle est de $O(n \times m)$. Ce pire cas se produit lorsque le texte et le motif partagent de longs préfixes, par exemple en cherchant le motif `AAAAAB` dans le texte `AAAAAAAAAAAAAAAAAB`. À chaque position, l'algorithme effectuera 6 comparaisons avant de constater l'échec et de se décaler d'une seule position.

### L'Idée Clé de KMP : Utiliser les Informations d'un Échec

L'inefficacité de l'algorithme naïf vient du fait qu'il "oublie" toute l'information acquise lors d'une correspondance partielle. L'observation fondamentale de l'algorithme KMP est que le motif lui-même contient suffisamment d'informations pour déterminer où la prochaine correspondance potentielle pourrait commencer, permettant ainsi de faire des décalages intelligents et d'éviter de re-comparer des caractères du texte déjà examinés.

Imaginons que nous cherchons le motif `abcabc` dans un texte et que nous obtenons une correspondance pour les 5 premiers caractères (`abcab`), mais une non-concordance sur le sixième.

```
Texte : ... a b c a b x ...
Motif :     a b c a b c
```

L'algorithme naïf décalerait le motif d'une position et recommencerait à comparer `a` avec `b`. KMP, en revanche, observe que le préfixe `ab` du motif correspond au suffixe de la partie déjà appariée (`abcab`). Il peut donc décaler le motif de manière à aligner ce préfixe avec ce suffixe, et reprendre la comparaison plus loin, sans jamais revenir en arrière dans le texte.

### La Fonction Préfixe (Table LPS ou "Failure Function")

Pour mettre en œuvre cette idée, KMP pré-calcule une table auxiliaire, souvent appelée table LPS (Longest Proper Prefix which is also a Suffix) ou, dans la conférence, fonction Fail. Dans le papier original de KMP, une fonction connexe est appelée next.

**Définition :** Pour un motif $P$ de longueur $m$, la table LPS est un tableau de taille $m$. Pour chaque indice $i$ (de 0 à $m-1$), LPS$[i]$ contient la longueur du plus long préfixe propre de la sous-chaîne $P[0..i]$ qui est également un suffixe de cette même sous-chaîne. Un préfixe/suffixe "propre" est un préfixe/suffixe qui n'est pas égal à la chaîne entière.

**Exemple de calcul pour le motif $P = $ `aababb` :**

- $P[0..0] = $ "a" : Préfixes propres : {}. Suffixes propres : {}. LPS$[0] = 0$.
- $P[0..1] = $ "aa" : Préfixes propres : {"a"}. Suffixes propres : {"a"}. Le plus long est "a". LPS$[1] = 1$.
- $P[0..2] = $ "aab" : Préfixes propres : {"a", "aa"}. Suffixes propres : {"b", "ab"}. Aucun en commun. LPS$[2] = 0$.
- $P[0..3] = $ "aaba" : Préfixes propres : {"a", "aa", "aab"}. Suffixes propres : {"a", "ba", "aba"}. Le plus long en commun est "a". LPS$[3] = 1$.
- $P[0..4] = $ "aabab": Préfixes propres : {"a", "aa", "aab", "aaba"}. Suffixes propres : {"b", "ab", "bab", "abab"}. Le plus long en commun est "ab". LPS$[4] = 2$.
- $P[0..5] = $ "aababb": Préfixes propres : {...}. Suffixes propres : {...}. Aucun en commun. LPS$[5] = 0$.

La table LPS pour `aababb` est donc $\{0, 1, 0, 1, 2, 0\}$.

Le tableau suivant détaille le calcul pour un autre motif, $P = $ `ababc`, pour illustrer l'algorithme de calcul.

| i | P[i] | Longueur du préfixe-suffixe courant (len) | LPS[i] | Explication |
|---|------|-------------------------------------------|--------|-------------|
| 0 | a | 0 | 0 | Cas de base, LPS[0] est toujours 0. |
| 1 | b | 0 | 0 | P[1] ('b') ≠ P[len] (P[0], 'a'). len reste 0. |
| 2 | a | 0 | 1 | P[2] ('a') == P[len] (P[0], 'a'). On étend la correspondance. len devient 1. |
| 3 | b | 1 | 2 | P[3] ('b') == P[len] (P[1], 'b'). On étend la correspondance. len devient 2. |
| 4 | c | 2 | 0 | P[4] ('c') ≠ P[len] (P[2], 'a'). Non-concordance. On doit "reculer" len en utilisant la table déjà calculée : len = LPS[len-1] = LPS[1] = 0. On re-compare P[4] avec P[0], toujours pas de correspondance. len reste 0. |

### Algorithme KMP en C

L'implémentation de KMP se fait en deux temps : le pré-calcul de la table LPS, puis la recherche elle-même. Les deux phases ont une complexité linéaire.

#### 1. Calcul de la table LPS

Cette fonction remplit la table LPS pour un motif donné en $O(m)$ temps.

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

// Calcule la table LPS (Longest Proper Prefix which is also Suffix)
void computeLPSArray(const char* pat, int m, int* lps) {
    int length = 0; // Longueur du précédent plus long préfixe-suffixe
    lps[0] = 0; // lps[0] est toujours 0

    int i = 1;
    while (i < m) {
        if (pat[i] == pat[length]) {
            length++;
            lps[i] = length;
            i++;
        } else {
            if (length != 0) {
                // C'est l'astuce : on ne remet pas length à 0.
                // On recule au LPS du caractère précédent.
                length = lps[length - 1];
            } else {
                lps[i] = 0;
                i++;
            }
        }
    }
}
```

#### 2. Recherche KMP

Cette fonction utilise la table LPS pour rechercher le motif dans le texte en $O(n)$ temps.

```c
// Recherche le motif 'pat' dans le texte 'txt' en utilisant la table lps
void KMPSearch(const char* pat, const char* txt) {
    int m = strlen(pat);
    int n = strlen(txt);

    int* lps = (int*)malloc(sizeof(int) * m);
    computeLPSArray(pat, m, lps);

    int i = 0; // index pour txt
    int j = 0; // index pour pat
    while (i < n) {
        if (pat[j] == txt[i]) {
            j++;
            i++;
        }

        if (j == m) {
            printf("Motif trouvé à l'index %d\n", i - j);
            // Pour trouver les autres occurrences, on continue la recherche
            // en utilisant la table lps pour décaler le motif.
            j = lps[j - 1];
        } else if (i < n && pat[j] != txt[i]) {
            // Non-concordance après j correspondances
            // Ne pas décrémenter i, mais décaler j
            if (j != 0) {
                j = lps[j - 1];
            } else {
                i = i + 1;
            }
        }
    }
    free(lps);
}

int main() {
    const char* txt = "ABABDABACDABABCABAB";
    const char* pat = "ABABCABAB";
    KMPSearch(pat, txt); // Devrait trouver le motif à l'index 10
    return 0;
}
```

### L'Algorithme KMP comme un Automate Fini

L'aspect le plus élégant de KMP est qu'il peut être interprété comme la simulation d'un automate fini déterministe spécifique au motif. Donald Knuth a d'ailleurs redécouvert l'algorithme indépendamment en l'abordant sous l'angle de la théorie des automates.

Dans cette vision :

- Les états de l'automate sont les entiers de 0 à $m$. Se trouver dans l'état $j$ signifie que nous venons de reconnaître un préfixe de $P$ de longueur $j$.
- L'état 0 est l'état initial.
- L'état $m$ est le seul état accepteur, signifiant que le motif complet a été trouvé.
- La fonction de transition $\delta(j, c)$ (où $j$ est l'état actuel et $c$ est le prochain caractère du texte) est définie comme suit :
  - **Transition de succès :** Si nous sommes dans l'état $j$ et que le caractère lu $c$ est égal à $P[j]$, nous passons à l'état $j+1$.
  - **Transition d'échec :** Si $c \neq P[j]$, nous ne retournons pas à l'état 0. Nous consultons la table LPS. Le nouvel état est celui que nous aurions atteint si nous avions traité la séquence correspondant au plus long préfixe-suffixe de $P[0..j-1]$. Cette transition "d'échec" est calculée en utilisant récursivement la table LPS, ce qui correspond à la ligne `j = lps[j-1]` dans le code.

La table LPS est donc une représentation hautement compressée et optimisée des transitions d'échec de cet automate implicite. Le pré-calcul de la table LPS est l'équivalent de la construction de cet automate spécifique au motif.

Cette perspective révèle une connexion profonde. Alors qu'un AFD générique peut reconnaître n'importe quel langage régulier, l'algorithme KMP est une spécialisation de cette idée pour le langage très spécifique $\Sigma^* P \Sigma^*$. Plutôt que de construire un AFD potentiellement grand avec une méthode générique, KMP exploite la structure particulière de ce langage pour créer une représentation de l'automate (la table LPS) qui est à la fois compacte ($O(m)$) et permet une simulation en temps linéaire ($O(n)$). KMP n'est donc pas simplement "un autre algorithme de chaînes", mais une compilation d'un automate fini spécifique à un motif en une structure de données optimisée.

## Conclusion et Perspectives : L'Algorithme d'Aho-Corasick et Au-delà

Ce cours nous a fait voyager des problèmes de programmation concrets vers les fondations mathématiques de la théorie des langages, pour finalement revenir à des algorithmes pratiques et hautement performants. Le fil conducteur de ce parcours a été la reconnaissance que l'abstraction et le formalisme ne sont pas des fins en soi, mais des outils puissants pour analyser, comprendre et résoudre des problèmes complexes de manière efficace et élégante.

### Synthèse du Cours

Nous avons commencé par constater les limites des fonctions de chaînes standards face à des problèmes de reconnaissance de motifs. En adoptant le formalisme de la théorie des langages, nous avons pu redéfinir ces problèmes en termes d'appartenance d'un mot à un langage.

L'automate fini déterministe (AFD) a été introduit comme une machine de reconnaissance pour une classe de langages appelés langages réguliers. Nous avons vu sa définition formelle, son fonctionnement, et comment l'implémenter en C, en découvrant au passage le lien entre une structure de données (la table de transition) et les performances matérielles (cache et prédiction de branchement).

Les expressions régulières ont été présentées comme une notation textuelle équivalente aux automates pour décrire les langages réguliers. La bibliothèque POSIX `<regex.h>` a démontré comment cette théorie est mise à la disposition des programmeurs C. Le non-déterminisme (AFN) a ensuite été révélé non pas comme un modèle plus puissant, mais comme un outil de conception et une étape de compilation essentielle, reliant la facilité de description (regex) à l'efficacité d'exécution (AFD) via l'algorithme de Rabin-Scott.

Nous avons ensuite exploré les limites de ce modèle avec le Lemme de la Pompe, qui formalise l'idée qu'une machine à mémoire finie ne peut pas résoudre des problèmes nécessitant un comptage infini, comme la reconnaissance du langage $\{a^n b^n\}$.

Enfin, l'algorithme de Knuth-Morris-Pratt (KMP) a illustré une application brillante de ces concepts. En considérant la recherche de motif comme la simulation d'un automate fini spécifique au motif, KMP utilise une table pré-calculée (LPS) pour atteindre une efficacité linéaire, bien au-delà de l'approche naïve.

### Au-delà de KMP : La Recherche de Motifs Multiples

L'algorithme KMP est optimisé pour la recherche d'un unique motif. Que se passe-t-il si nous devons rechercher simultanément des centaines, voire des milliers de mots-clés dans un texte ? C'est un problème courant en bio-informatique (recherche de séquences d'ADN), en détection d'intrusion (recherche de signatures de virus) ou en analyse de texte. Lancer KMP pour chaque mot-clé serait inefficace.

C'est ici qu'intervient l'**algorithme d'Aho-Corasick**, développé en 1975 par Alfred V. Aho et Margaret J. Corasick. Le conférencier y fait allusion comme un prolongement naturel des thèmes abordés. Cet algorithme est une généralisation magnifique qui combine les idées de KMP et des structures de données arborescentes.

Son principe est le suivant :

1. **Construction d'un Trie :** Au lieu de travailler sur un seul motif, Aho-Corasick commence par construire un trie (ou arbre de préfixes) à partir de tous les mots-clés du dictionnaire. Chaque chemin de la racine à un nœud dans le trie représente un préfixe d'un ou plusieurs mots-clés. Les nœuds correspondant à des mots-clés complets sont marqués comme des états accepteurs.

2. **Ajout de Liens d'Échec :** Le trie est ensuite augmenté avec des liens d'échec (failure links). Pour chaque nœud, son lien d'échec pointe vers le plus long préfixe propre de la chaîne correspondant à ce nœud qui est aussi un préfixe d'un autre (ou du même) mot-clé dans le dictionnaire. Ces liens sont l'analogue direct de la table LPS de KMP, mais généralisés à un ensemble de motifs.

3. **Exécution en un seul passage :** L'algorithme parcourt le texte une seule fois. À chaque caractère, il avance dans le trie. En cas de non-concordance, il suit le lien d'échec pour se repositionner instantanément sur le plus long préfixe alternatif possible, sans jamais revenir en arrière dans le texte.

Le résultat est un unique automate fini qui reconnaît toutes les occurrences de tous les mots-clés en un seul passage, avec une complexité linéaire par rapport à la somme des longueurs du texte et des motifs. Cet algorithme a été si efficace qu'il a formé la base de la commande `fgrep` originale d'Unix.

### Applications Modernes et Conclusion

Loin d'être un sujet historique, la théorie des automates et des langages formels reste un pilier de l'informatique moderne, avec des applications omniprésentes :

- **Compilation :** L'analyse lexicale, la première phase d'un compilateur, est presque universellement implémentée à l'aide d'automates finis.

- **Vérification de modèles (Model Checking) :** Des automates sont utilisés pour modéliser le comportement de systèmes matériels ou logiciels et vérifier qu'ils respectent des propriétés de sécurité ou de vivacité.

- **Bio-informatique :** Les algorithmes de recherche de motifs comme KMP et Aho-Corasick, ainsi que leurs variantes, sont fondamentaux pour l'analyse de séquences génomiques.

- **Traitement de texte et Réseaux :** Les moteurs d'expressions régulières sont au cœur de la validation de formulaires, de l'analyse de logs, du routage de paquets et de l'analyse de protocoles.

En conclusion, ce cours a cherché à démontrer que la maîtrise des structures de données et des algorithmes passe aussi par la compréhension des modèles mathématiques abstraits qui les sous-tendent. En apprenant à voir un problème de programmation sous l'angle d'un langage formel, le développeur acquiert un cadre de pensée qui lui permet de raisonner sur la complexité, de choisir l'outil approprié et de concevoir des solutions non seulement fonctionnelles, mais aussi robustes, efficaces et élégantes.
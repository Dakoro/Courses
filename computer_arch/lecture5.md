# Cours d'Architecture des Ordinateurs : Machines à États Finis et Langages de Description Matérielle

## Introduction

Ce cours fait suite à notre étude de la logique séquentielle et se concentre sur deux aspects fondamentaux de la conception matérielle moderne :
1. La conception de machines à états finis (FSM - Finite State Machines)
2. Les langages de description matérielle (HDL - Hardware Description Languages), en particulier Verilog

## Partie 1 : Machines à États Finis (FSM)

### 1.1 Rappel sur les Circuits Séquentiels

Les **circuits séquentiels** produisent des sorties déterminées par :
- Les entrées actuelles
- Les valeurs passées (mémoire)

Ces circuits sont fondamentalement différents des circuits combinatoires qui dépendent uniquement des entrées actuelles.

**Analogie importante** : Considérez la différence entre un cadenas combinatoire (logique combinatoire) et un cadenas séquentiel (logique séquentielle). Le cadenas séquentiel "se souvient" de la séquence entrée, pas seulement de la position actuelle.

### 1.2 Structure d'une Machine à États Finis

Une FSM est une représentation picturale d'un modèle temporel discret d'un système avec états. Elle comprend :

1. **Un nombre fini d'états**
2. **Un nombre fini d'entrées**
3. **Un nombre fini de sorties**
4. **Une spécification explicite des transitions d'états**
5. **Une spécification explicite de la génération des sorties**

#### Architecture d'une FSM

Trois éléments essentiels composent une FSM :

1. **Registre d'état** : Mémorise l'état actuel
2. **Logique d'état suivant** : Détermine la transition vers l'état suivant
3. **Logique de sortie** : Génère les sorties basées sur l'état (et potentiellement les entrées)

### 1.3 Types de Machines à États Finis

#### Machine de Moore
- Les sorties dépendent **uniquement de l'état actuel**
- Sortie = f(État actuel)
- Plus simple conceptuellement

#### Machine de Mealy
- Les sorties dépendent de **l'état actuel ET des entrées**
- Sortie = f(État actuel, Entrées)
- Peut réduire le nombre d'états nécessaires

### 1.4 Exemple Concret : Contrôleur de Feux de Circulation "Intelligent"

Considérons un contrôleur pour une intersection entre :
- Avenue A
- Boulevard B

**Spécifications** :
- Capteurs de trafic sur chaque voie (TA et TB)
- Feux tricolores (Rouge, Jaune, Vert)
- Changement d'état toutes les 5 secondes
- **Exception** : Si le feu est vert ET qu'il y a du trafic, l'état ne change pas

**Problème identifié** : Ce design n'est pas équitable - une avenue avec trafic constant monopolisera le feu vert indéfiniment.

#### États du système

| État | Feu Avenue A | Feu Boulevard B |
|------|--------------|-----------------|
| S0   | Vert         | Rouge          |
| S1   | Jaune        | Rouge          |
| S2   | Rouge        | Vert           |
| S3   | Rouge        | Jaune          |

#### Table de transition d'états

La logique d'état suivant peut être exprimée sous forme de table de vérité :

```
État actuel | TA | TB | État suivant
S0         | 1  | X  | S0
S0         | 0  | X  | S1
S1         | X  | X  | S2
S2         | X  | 1  | S2
S2         | X  | 0  | S3
S3         | X  | X  | S0
```

(X = don't care / peu importe)

### 1.5 Encodage des États

Trois méthodes principales d'encodage :

#### 1. Encodage binaire (fully encoded)
- Utilise le minimum de bits : log₂(nombre d'états)
- Pour 4 états : 2 bits suffisent
- Exemple : S0=00, S1=01, S2=10, S3=11

#### 2. Encodage "one-hot"
- Un bit par état
- Pour 4 états : 4 bits nécessaires
- Exemple : S0=0001, S1=0010, S2=0100, S3=1000
- Simplifie la logique d'état suivant

#### 3. Encodage de sortie
- Les bits d'état encodent directement les sorties
- Fonctionne bien pour les machines de Moore
- Minimise la logique de sortie

### 1.6 Conception d'une FSM : Méthodologie

1. **Déterminer tous les états possibles**
2. **Développer le diagramme de transition d'états**
3. **Créer la table de transition**
4. **Choisir un encodage d'états**
5. **Dériver les équations logiques** pour :
   - La logique d'état suivant
   - La logique de sortie
6. **Implémenter le circuit**

## Partie 2 : Langages de Description Matérielle (HDL)

### 2.1 Motivation : La Complexité Croissante

L'évolution de la complexité des circuits :
- 2017 : 1,75 milliards de transistors
- 2021 : 16 milliards de transistors
- 2022 : 114 milliards de transistors
- Circuits à l'échelle de la plaquette : 2,6 trillions de transistors

**Comment gérer une telle complexité ?**

Les HDL permettent de :
1. **Spécifier** des conceptions complexes
2. **Simuler** le comportement pour vérification
3. **Synthétiser** automatiquement les circuits

### 2.2 Pourquoi des Langages Spécialisés ?

Les HDL offrent des primitives spécifiques au matériel :
- Fils (wires)
- Portes (gates)
- Registres
- Flip-flops
- Fronts d'horloge montants/descendants

**Point crucial** : Le matériel est intrinsèquement parallèle - tout se passe simultanément, contrairement au logiciel séquentiel.

### 2.3 Verilog : Concepts Fondamentaux

#### Module : L'Unité de Base

```verilog
module nom_module(a, b, c, y);
    input a, b, c;
    output y;
    
    // Description du circuit
    
endmodule
```

#### Deux Styles de Description

##### 1. Description Structurelle (Niveau Portes)

```verilog
module mux2(d0, d1, s, y);
    input d0, d1, s;
    output y;
    wire notS, y1, y2;
    
    not (notS, s);
    and (y1, notS, d0);
    and (y2, s, d1);
    or  (y, y1, y2);
endmodule
```

##### 2. Description Comportementale

```verilog
module mux2(d0, d1, s, y);
    input d0, d1, s;
    output y;
    
    assign y = s ? d1 : d0;
endmodule
```

### 2.4 Éléments de Syntaxe Verilog

#### Bus et Vecteurs

```verilog
input [31:0] data;     // Bus 32 bits
output [7:0] result;   // Bus 8 bits
```

#### Manipulation de Bits

```verilog
wire [15:0] longBus;
wire [7:0] shortBus;

assign shortBus = longBus[12:5];  // Extraction de bits
```

#### Concaténation

```verilog
wire [3:0] y;
assign y = {a[2], a[1], a[0], a[0]};  // Concaténation
```

#### Opérateurs de Réduction

```verilog
wire [7:0] a;
wire y;
assign y = &a;  // ET de tous les bits de a
```

### 2.5 Signaux Flottants (Haute Impédance)

```verilog
module tristate(a, enable, y);
    input [3:0] a;
    input enable;
    output [3:0] y;
    
    assign y = enable ? a : 4'bz;
endmodule
```

### 2.6 Représentation des Nombres

Format : `<taille>'<base><valeur>`

Exemples :
- `8'b10101010` : 8 bits en binaire
- `16'hABCD` : 16 bits en hexadécimal
- `32'd1234` : 32 bits en décimal

### 2.7 Modules Paramétrables

```verilog
module mux2 #(parameter WIDTH = 8) (
    input [WIDTH-1:0] d0, d1,
    input s,
    output [WIDTH-1:0] y
);
    assign y = s ? d1 : d0;
endmodule
```

Instanciation :
```verilog
mux2 #(16) mux16bit(...);  // Multiplexeur 16 bits
```

## Partie 3 : Synthèse et Simulation

### 3.1 Synthèse

La **synthèse** transforme le code HDL en :
- Netlist de portes et fils
- Optimisations possibles :
  - Minimisation logique
  - Choix de portes optimales
  - Placement et routage

**Attention** : Toutes les constructions Verilog ne sont pas synthétisables !

### 3.2 Simulation

La **simulation** permet de :
- Vérifier le comportement fonctionnel
- Analyser le timing
- Déboguer sans construire le circuit

### 3.3 Piège Important

**Ne pensez pas à Verilog comme un langage de programmation !**

Si vous ne savez pas approximativement quel matériel votre HDL devrait synthétiser, vous risquez de :
- Créer beaucoup plus de matériel que nécessaire
- Écrire du code qui simule correctement mais ne peut pas être implémenté

**Conseil** : Pensez en termes de blocs de logique combinatoire, registres et FSM. Esquissez ces blocs sur papier avant de coder.

## Conclusion

Ce cours a couvert les fondements essentiels de la conception matérielle moderne :

1. **Les FSM** permettent de modéliser et implémenter des systèmes séquentiels complexes
2. **Verilog** offre un moyen puissant de décrire, simuler et synthétiser du matériel
3. La **conception hiérarchique** est essentielle pour gérer la complexité

Ces concepts forment la base pour la conception de processeurs et systèmes numériques complexes que nous explorerons dans les prochains cours.
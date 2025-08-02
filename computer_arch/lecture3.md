# Logique Combinatoire et Circuits Numériques

## 📚 Introduction et Objectifs

Cette leçon approfondit la **logique combinatoire**, élément fondamental de l'architecture des ordinateurs. Nous allons construire des circuits de plus en plus complexes à partir des portes logiques de base.

### Plan de la Leçon
1. Révision des transistors et portes logiques
2. Algèbre booléenne et formes canoniques
3. Simplification logique
4. Blocs combinatoires essentiels
5. Circuits programmables

## 🔧 Structure Générale des Portes CMOS

### Principe Fondamental
En technologie CMOS, toute porte logique inversante suit cette structure :

```
     VDD (Alimentation)
          |
    [Réseau Pull-up PMOS]
          |
    ------+------ Sortie Y
          |
    [Réseau Pull-down NMOS]
          |
         GND (Masse)
```

### Règles Critiques
1. **Un seul réseau actif** : Soit pull-up, soit pull-down (jamais les deux)
2. **PMOS en haut** : Excellents pour "tirer vers le haut" (conduire VDD)
3. **NMOS en bas** : Excellents pour "tirer vers le bas" (conduire GND)
4. **Éviter les courts-circuits** : Les deux réseaux ON = catastrophe
5. **Éviter les sorties flottantes** : Les deux réseaux OFF = sortie indéfinie (Z)

### Transistors en Série vs Parallèle

| Configuration | Comportement | Latence | Application |
|--------------|--------------|---------|-------------|
| **Série** | Tous doivent être ON | Plus lente | Fonction ET |
| **Parallèle** | Un seul doit être ON | Plus rapide | Fonction OU |

## 📐 Algèbre Booléenne

### Notation Standard
- **ET** : A·B ou AB (point souvent omis)
- **OU** : A + B
- **NON** : Ā ou A'
- **XOR** : A ⊕ B

### Axiomes Fondamentaux

#### 1. Commutativité
- A + B = B + A
- A · B = B · A

#### 2. Identité
- A + 0 = A
- A · 1 = A

#### 3. Distribution
- A · (B + C) = (A · B) + (A · C)
- A + (B · C) = (A + B) · (A + C)

#### 4. Complémentation
- A + Ā = 1
- A · Ā = 0

### Théorèmes de Simplification

#### Absorption
- **X + X·Y = X** (très fréquent dans les circuits)
- **X·(X + Y) = X**

#### Preuve de X + X·Y = X :
1. X + X·Y = X·1 + X·Y (identité)
2. = X·(1 + Y) (distribution)
3. = X·1 (car 1 + Y = 1)
4. = X

### Lois de De Morgan
**Essentielles pour la transformation de circuits**

- **NOT(A + B) = Ā · B̄**
- **NOT(A · B) = Ā + B̄**

#### Application : "Bubble Pushing"
Transformer une porte en une autre en "poussant" les bulles d'inversion :

```
NAND → OR avec entrées inversées
NOR → AND avec entrées inversées
```

## 📊 Formes Canoniques

### 1. Forme Somme de Produits (SOP)

**Principe** : Exprimer la fonction comme OU de tous les mintermes donnant 1

#### Exemple : Table de Vérité
| A | B | C | F |
|---|---|---|---|
| 0 | 0 | 0 | 0 |
| 0 | 0 | 1 | 0 |
| 0 | 1 | 0 | 0 |
| 0 | 1 | 1 | 1 |
| 1 | 0 | 0 | 1 |
| 1 | 0 | 1 | 1 |
| 1 | 1 | 0 | 1 |
| 1 | 1 | 1 | 1 |

**Forme SOP** : F = Ā·B·C + A·B̄·C̄ + A·B̄·C + A·B·C̄ + A·B·C

**Notation compacte** : F = Σm(3,4,5,6,7)

### 2. Forme Produit de Sommes (POS)

**Principe** : Exprimer la fonction comme ET de tous les maxtermes donnant 0

**Forme POS** : F = (A+B+C)·(A+B+C̄)·(A+B̄+C)

**Notation compacte** : F = ΠM(0,1,2)

### Conversions Importantes
- Mintermes de F = Compléments des mintermes de F̄
- F = Σm(3,4,5,6,7) ⟺ F = ΠM(0,1,2)

## 🔨 Blocs Logiques Combinatoires

### 1. Décodeur (Détecteur de Motifs)

**Fonction** : Active une unique sortie correspondant au motif d'entrée

#### Décodeur 2→4
```
Entrées (A₁A₀) | Sorties (Y₃Y₂Y₁Y₀)
      00       |     0001
      01       |     0010
      10       |     0100
      11       |     1000
```

**Applications** :
- Décodage d'adresses mémoire
- Décodage d'instructions (opcodes)
- Génération de mintermes

### 2. Multiplexeur (Sélecteur)

**Fonction** : Sélectionne une entrée parmi N selon un signal de contrôle

#### MUX 2→1
```
Si S = 0 : Y = D₀
Si S = 1 : Y = D₁
```

**Équation** : Y = S̄·D₀ + S·D₁

#### Utilisation comme Table de Vérité (LUT)
Un MUX peut implémenter **n'importe quelle fonction** :
- Connecter les entrées de données aux valeurs de la table de vérité
- Les entrées de sélection deviennent les variables de la fonction

### 3. Additionneur Complet

**Fonction** : Addition de 3 bits (A, B, Cin) → (Sum, Cout)

#### Table de Vérité
| A | B | Cin | Sum | Cout |
|---|---|-----|-----|------|
| 0 | 0 | 0   | 0   | 0    |
| 0 | 0 | 1   | 1   | 0    |
| 0 | 1 | 0   | 1   | 0    |
| 0 | 1 | 1   | 0   | 1    |
| 1 | 0 | 0   | 1   | 0    |
| 1 | 0 | 1   | 0   | 1    |
| 1 | 1 | 0   | 0   | 1    |
| 1 | 1 | 1   | 1   | 1    |

**Équations simplifiées** :
- **Sum = A ⊕ B ⊕ Cin** (XOR à 3 entrées)
- **Cout = A·B + A·Cin + B·Cin** (Majorité)

### 4. Additionneur à Propagation de Retenue

Pour additionner des nombres de N bits :
```
A₃A₂A₁A₀
+
B₃B₂B₁B₀
---------
S₃S₂S₁S₀
```

**Problème** : La retenue doit se propager de droite à gauche (lent)

**Solutions avancées** : Additionneurs à anticipation de retenue

## 🎛️ Circuits Programmables (PLA)

### Programmable Logic Array

**Structure** : Implémente directement la forme SOP
1. **Niveau 1** : Réseau de portes ET (génère tous les mintermes)
2. **Niveau 2** : Réseau de portes OU (sélectionne les mintermes)

**Programmation** : Configurer les connexions entre ET et OU

### Applications FPGA

Les FPGA utilisent des **LUT (Look-Up Tables)** :
- Tables de vérité programmables
- Typiquement 4-6 entrées
- Peuvent implémenter n'importe quelle fonction

**Avantage** : Reconfigurabilité complète
**Inconvénient** : Moins efficace qu'un circuit spécialisé

## 📈 Loi de Moore et Évolution Technologique

### Énoncé Original (1965)
> "Le nombre de composants par circuit intégré double tous les 18 mois"

### Implications
- **1971** : Intel 4004 - 2 300 transistors
- **2001** : Pentium 4 - 42 millions
- **2023** : Processeurs modernes - 67 milliards

### Défis Actuels
1. **Limites physiques** : Transistors de quelques atomes
2. **Dissipation thermique** : P = C·V²·f
3. **Coûts de fabrication** : Machines EUV extrêmement complexes

### Technologies Clés
- **FinFET** : Transistors 3D
- **EUV** : Lithographie extrême UV (ASML)
- **Nœuds** : 7nm, 5nm, 3nm (termes marketing)

## 💡 Consommation Énergétique

### Puissance Dynamique
**P_dyn = C·V²·f**
- C : Capacité du circuit
- V : Tension d'alimentation
- f : Fréquence de commutation

**Stratégies de réduction** :
- Diminuer V (impact quadratique)
- Réduire f (clock gating)
- Optimiser C (transistors plus petits)

### Puissance Statique
**P_static = I_leak·V**
- Augmente avec la miniaturisation
- Critique dans les dispositifs mobiles

## 🎯 Points Clés à Retenir

### 1. Abstraction et Modularité
- Construire des blocs complexes à partir de portes simples
- Réutiliser les modules (décodeurs, MUX, additionneurs)

### 2. Optimisation des Circuits
- Forme canonique → Forme minimale
- Choix des portes selon les contraintes (vitesse, surface, consommation)

### 3. Programmabilité
- PLA et LUT permettent la flexibilité
- Compromis performance/configurabilité

### 4. Limites Physiques
- La loi de Moore ralentit
- L'innovation architecturale devient cruciale

## 📝 Exercices Recommandés

1. **Simplifier** : F = Σm(0,2,3,4,5,7) en utilisant les théorèmes booléens
2. **Concevoir** : Un comparateur 2 bits avec décodeur et portes
3. **Implémenter** : XOR uniquement avec des portes NAND
4. **Analyser** : Délai de propagation dans un additionneur 8 bits

## 🚀 Prochaine Étape

**Logique Séquentielle** : Introduction de la mémoire dans les circuits
- Bascules et verrous
- Registres et compteurs
- Machines à états finis

La logique combinatoire que nous avons étudiée forme la base du traitement, mais sans mémoire, nous ne pouvons pas construire de véritables systèmes informatiques. La semaine prochaine, nous ajouterons cette dimension temporelle cruciale.
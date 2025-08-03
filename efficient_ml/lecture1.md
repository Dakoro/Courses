# Apprentissage Automatique Efficace et TinyML : De la Théorie à l'Implémentation

## Le Contexte : La Crise du Calcul en IA

L'intelligence artificielle a connu une croissance exponentielle, mais cette avancée a un coût. Nous sommes actuellement confrontés à une divergence critique entre la demande en ressources de calcul pour les modèles d'IA et l'offre fournie par le matériel. Un examen de l'évolution des modèles de langage au cours des dernières années révèle une tendance alarmante : la taille des modèles, mesurée en nombre de paramètres, a explosé, passant de quelques millions à des centaines de milliards. Par exemple, le modèle Megatron-Turing NLG atteint 530 milliards de paramètres. Cette croissance de la complexité des modèles, qui représente la demande en calcul, est nettement plus rapide que l'amélioration des capacités de calcul des GPU, qui représente l'offre.

Cette situation n'est pas simplement un défi matériel ; elle se transforme en une crise d'accessibilité. Le coût de l'entraînement et du déploiement de ces modèles massifs devient prohibitif. L'entraînement de Stable Diffusion, par exemple, a coûté plus de 600 000 dollars, et l'inférence sur des services comme ChatGPT nécessite une infrastructure si vaste qu'elle rencontre régulièrement des problèmes de capacité. Ces barrières financières et techniques risquent de centraliser la recherche et le déploiement de l'IA de pointe au sein d'un petit nombre de grandes entreprises, étouffant l'innovation et créant une fracture numérique dans l'écosystème de l'IA. Dans ce contexte, l'apprentissage automatique efficace n'est pas seulement une optimisation technique ; il devient une force démocratisatrice, permettant aux startups, aux laboratoires académiques et aux chercheurs individuels de participer à la prochaine vague d'innovation en IA.

## Les Trois Piliers de l'IA Moderne

La révolution actuelle du deep learning repose sur l'interaction synergique de trois piliers fondamentaux : les **Algorithmes sophistiqués**, les **Données à grande échelle** (comme le jeu de données ImageNet, qui a été un catalyseur majeur), et le **Matériel spécialisé**, principalement les Unités de Traitement Graphique (GPU). Ce cours se concentrera sur le pilier algorithmique, en explorant comment l'innovation dans la conception des modèles et la co-conception algorithme-matériel peuvent nous aider à combler le fossé du calcul.

## Vue d'Ensemble des Techniques de Compression

Pour rendre l'IA plus efficace, nous nous appuierons sur un ensemble de techniques de compression de modèles. L'objectif est de réduire la taille et la complexité des réseaux de neurones tout en préservant leur précision. Le flux de travail que nous adopterons est "Compresser, puis inférer efficacement", ce qui signifie que nous ne nous contenterons pas de réduire la taille du modèle, mais nous nous assurerons également que nos systèmes d'inférence peuvent exploiter directement ces modèles compressés pour obtenir une accélération réelle.

Les quatre piliers de la compression que nous étudierons sont :

1. **Élagage (Pruning) & Sparsité** : Réduire le nombre total de paramètres en éliminant les connexions redondantes.
2. **Quantification** : Réduire la précision numérique des paramètres (par exemple, passer de nombres à virgule flottante de 32 bits à des entiers de 8 bits).
3. **Recherche d'Architecture Neuronale (NAS)** : Concevoir automatiquement des modèles qui sont intrinsèquement plus petits et plus rapides.
4. **Distillation de Connaissances** : Transférer l'intelligence d'un grand modèle "professeur" à un modèle plus petit et plus efficace "étudiant".

Le tableau suivant offre une vue d'ensemble comparative de ces techniques, qui servira de feuille de route pour les sections à venir.

| Technique | Principe Fondamental | Ratio de Compression Typique | Impact sur la Précision | Compatibilité Matérielle |
|-----------|---------------------|------------------------------|-------------------------|---------------------------|
| **Élagage (Pruning)** | Élimination des poids/neurones redondants (faible magnitude) | 2x - 10x | Faible à modéré (récupérable par fine-tuning) | Nécessite un support matériel/logiciel pour la sparsité (par ex. NVIDIA Sparse Tensor Cores) |
| **Quantification** | Réduction de la précision numérique (par ex., FP32 → INT8) | 2x - 4x | Faible (PTQ) à négligeable (QAT) | Largement supportée (CPU, GPU, TPU, DSPs) |
| **NAS** | Recherche automatisée d'architectures efficaces et optimisées pour le matériel | Variable (conception) | Peut améliorer la précision pour une taille donnée | Spécifiquement optimisée pour le matériel cible |
| **Distillation** | Entraînement d'un petit modèle "étudiant" à imiter un grand modèle "professeur" | 2x - 100x+ | Dépend de l'écart de capacité entre les modèles | Aucune exigence spécifique ; le modèle étudiant est un modèle dense standard |

## Techniques Fondamentales de Compression de Modèles

Cette section plonge au cœur des algorithmes qui nous permettent de rendre les réseaux de neurones plus légers et plus rapides. Pour chaque technique, nous examinerons les fondements théoriques, les algorithmes clés, et nous fournirons des implémentations pratiques en Python, C/C++, ainsi que du pseudo-code pour une compréhension claire des mécanismes internes.

### Élagage (Pruning) et Sparsité

L'élagage est fondé sur l'observation que les réseaux de neurones profonds sont souvent largement sur-paramétrés, contenant de nombreuses connexions redondantes qui peuvent être supprimées sans nuire de manière significative à la précision du modèle.

#### Fondements Théoriques

L'objectif de l'élagage est de trouver un sous-réseau plus petit au sein du réseau d'origine qui conserve une performance similaire. Mathématiquement, cela peut être formulé comme un problème d'optimisation visant à minimiser la fonction de perte $L(W)$ sous la contrainte que le nombre de poids non nuls (la norme $L_0$) soit inférieur à un certain seuil $k$ :

$$\min_W L(W) \quad \text{sujet à } \|W\|_0 \leq k$$

Cependant, la norme $L_0$ est non différentiable, ce qui rend l'optimisation directe difficile. Une heuristique très efficace et largement utilisée est l'élagage par magnitude. Cette approche repose sur l'hypothèse que les poids avec les plus petites magnitudes (proches de zéro) sont les moins importants et peuvent être supprimés. Le processus, popularisé par l'article "Deep Compression", suit un pipeline en trois étapes :

1. **Entraînement** : Entraîner un réseau de neurones dense jusqu'à convergence.
2. **Élagage** : Supprimer les connexions dont les poids sont inférieurs à un certain seuil de magnitude.
3. **Ré-entraînement** : Effectuer un "fine-tuning" du réseau élagué pour permettre aux poids restants de compenser les connexions supprimées et ainsi de récupérer la précision perdue.

#### Types d'Élagage

L'élagage peut être classé selon la granularité et la portée de la suppression des poids :

**Non Structuré vs. Structuré :**
- L'**élagage non structuré** supprime des poids individuels, ce qui conduit à des matrices de poids clairsemées mais de forme irrégulière. Bien que cette méthode puisse atteindre des taux de compression élevés, elle nécessite un support matériel ou logiciel spécialisé (comme les Sparse Tensor Cores de NVIDIA) pour une accélération efficace de l'inférence.
- L'**élagage structuré** supprime des groupes entiers de poids, tels que des canaux de convolution, des filtres, ou même des têtes d'attention entières. Cette approche préserve la structure dense des matrices, ce qui permet d'obtenir une accélération sur du matériel standard en réduisant simplement les dimensions des tenseurs.

**Local vs. Global :**
- L'**élagage local** applique un taux de sparsité fixe à chaque couche du réseau (par exemple, supprimer 50 % des poids de chaque couche).
- L'**élagage global** applique un taux de sparsité unique à l'ensemble du modèle. Cela permet de supprimer proportionnellement plus de poids dans les couches moins sensibles à l'élagage, ce qui conduit souvent à une meilleure précision pour un même niveau de sparsité global.

#### Algorithme : Élagage Itératif par Magnitude

Une approche simple mais puissante consiste à élaguer le réseau de manière itérative, en supprimant progressivement les poids et en ré-entraînant le modèle à chaque étape pour permettre une récupération progressive de la précision.

**Pseudo-code :**

```
Algorithme : Élagage Itératif par Magnitude
1: Entrée : Modèle M, Données D, Sparsité cible S_cible, Calendrier d'élagage P
2: Entraîner M sur D jusqu'à convergence → M_entraine
3: Pour chaque étape d'élagage p dans P avec une sparsité S_p:
4:   Calculer un seuil de magnitude global T pour M_entraine afin d'atteindre la sparsité S_p
5:   Créer un masque m où m_i = 0 si |w_i| < T, sinon m_i = 1
6:   Appliquer le masque : M_elague.poids = M_entraine.poids * m
7:   Fine-tuner M_elague sur D pendant quelques époques → M_entraine
8: Sortie : M_elague avec une sparsité S_cible
```

#### Implémentation en Python (PyTorch)

PyTorch fournit des utilitaires pratiques dans `torch.nn.utils.prune` pour mettre en œuvre l'élagage. Ces fonctions appliquent un masque de manière non destructive sur les tenseurs de poids.

```python
# Implémentation de l'élagage L1 non structuré en Python avec PyTorch
import torch
import torch.nn as nn
import torch.nn.utils.prune as prune

# Définir un modèle simple
model = nn.Sequential(
    nn.Linear(10, 20),
    nn.ReLU(),
    nn.Linear(20, 5)
)

# Sélectionner la couche à élaguer
layer_to_prune = model[0]
param_to_prune = 'weight'

# Appliquer un élagage non structuré de 50% basé sur la magnitude L1
prune.l1_unstructured(layer_to_prune, name=param_to_prune, amount=0.5)

# Le tenseur de poids est maintenant un produit du masque et des poids d'origine
print(f"Poids élagués:\n{layer_to_prune.weight}")
print(f"Masque d'élagage:\n{layer_to_prune.weight_mask}")

# Pour rendre l'élagage permanent et supprimer le masque
prune.remove(layer_to_prune, param_to_prune)
print(f"Poids permanents après élagage:\n{layer_to_prune.weight}")
```

#### Implémentation en C/C++

En C++, l'élagage post-entraînement peut être implémenté en chargeant les poids du modèle, en les mettant à zéro en fonction de leur magnitude, puis en les sauvegardant. Le principal défi réside dans la gestion efficace des matrices creuses qui en résultent pour l'inférence.

```cpp
// Implémentation conceptuelle de l'élagage par magnitude en C++
#include <vector>
#include <algorithm>
#include <cmath>
#include <iostream>

// Fonction pour élaguer un vecteur de poids
void prune_weights(std::vector<float>& weights, float sparsity_ratio) {
    if (sparsity_ratio <= 0.0f || sparsity_ratio >= 1.0f) {
        return;
    }

    std::vector<std::pair<float, int>> mag_indices;
    mag_indices.reserve(weights.size());
    for (int i = 0; i < weights.size(); ++i) {
        mag_indices.push_back({std::abs(weights[i]), i});
    }

    // Trier les poids par magnitude croissante
    std::sort(mag_indices.begin(), mag_indices.end());

    // Déterminer le nombre de poids à mettre à zéro
    int prune_count = static_cast<int>(weights.size() * sparsity_ratio);

    // Mettre à zéro les poids les plus faibles
    for (int i = 0; i < prune_count; ++i) {
        weights[mag_indices[i].second] = 0.0f;
    }
}

int main() {
    // Exemple d'utilisation
    std::vector<float> my_weights = {0.1, -0.05, 1.2, -0.02, 0.8, -2.5, 0.01, 0.3};
    
    std::cout << "Poids avant élagage:" << std::endl;
    for(float w : my_weights) std::cout << w << " ";
    std::cout << std::endl;

    prune_weights(my_weights, 0.5); // Élaguer 50% des poids

    std::cout << "Poids après élagage:" << std::endl;
    for(float w : my_weights) std::cout << w << " ";
    std::cout << std::endl;

    return 0;
}
```

Pour une inférence efficace, les poids élagués seraient stockés dans un format de matrice creuse comme le Compressed Sparse Row (CSR), et des noyaux de calcul spécialisés seraient nécessaires pour effectuer des multiplications matrice-vecteur creuses, une fonctionnalité offerte par des bibliothèques comme nnabla.

### Quantification

La quantification est une technique qui réduit la précision des nombres utilisés pour représenter les poids et les activations d'un réseau, généralement en passant de nombres à virgule flottante de 32 bits (FP32) à des entiers de 8 bits (INT8). Cela permet une réduction de la taille du modèle par un facteur de 4 et peut considérablement accélérer l'inférence sur du matériel supportant l'arithmétique entière.

#### Fondements Théoriques

La méthode la plus courante est la quantification affine (ou uniforme), qui établit une correspondance linéaire entre les valeurs réelles et les valeurs quantifiées. La formule de base est :

$$r = S \cdot (q - Z)$$

Où :
- $r$ est la valeur réelle (par exemple, en FP32).
- $q$ est la valeur quantifiée (par exemple, en INT8).
- $S$ est le facteur d'échelle (scale), un nombre à virgule flottante qui détermine la granularité de la quantification.
- $Z$ est le point zéro (zero-point), un entier qui garantit que la valeur réelle de 0 est représentée exactement sans erreur.

Les paramètres $S$ et $Z$ sont calculés à partir de la plage de valeurs $[r_{\min}, r_{\max}]$ du tenseur à quantifier :

$$S = \frac{r_{\max} - r_{\min}}{q_{\max} - q_{\min}}$$

$$Z = \text{round}\left(q_{\max} - \frac{r_{\max}}{S}\right)$$

#### Stratégies de Quantification

**Post-Training Quantization (PTQ)** : Cette approche est appliquée à un modèle déjà entraîné. Elle est simple et rapide à mettre en œuvre. Un petit ensemble de données de calibration est utilisé pour déterminer les plages statistiques des activations et calculer les paramètres $S$ et $Z$.

**Quantization-Aware Training (QAT)** : Cette méthode simule les erreurs de quantification pendant la phase d'entraînement. Le modèle apprend à être robuste à la perte de précision, ce qui permet généralement d'obtenir une meilleure précision finale que le PTQ, au prix d'un processus d'entraînement plus complexe.

#### Le Défi des LLMs : Les Valeurs Aberrantes d'Activation

Les grands modèles de langage (LLMs) posent un défi unique à la quantification. Leurs tenseurs d'activation présentent souvent des valeurs aberrantes (outliers), où quelques valeurs sont des ordres de grandeur plus grandes que les autres. La quantification standard, qui base son échelle sur les valeurs minimales et maximales, alloue une grande partie de la plage dynamique à ces rares valeurs aberrantes, laissant très peu de précision pour la majorité des valeurs. Cela entraîne une dégradation catastrophique de la performance du modèle.

#### Algorithme Avancé 1 : SmoothQuant

SmoothQuant propose une solution élégante à ce problème en "déplaçant la difficulté" de la quantification des activations (qui sont difficiles à quantifier à cause des outliers) vers les poids (qui sont généralement plus faciles à quantifier).

**Idée Clé :** Appliquer une transformation mathématiquement équivalente à une couche linéaire $Y = XW$. En introduisant un facteur de lissage $s$ par canal, l'opération devient :

$$Y = (X \cdot \text{diag}(s)^{-1}) \cdot (\text{diag}(s) \cdot W) = \hat{X}\hat{W}$$

Le facteur $s$ est choisi pour "lisser" les activations $\hat{X}$ en réduisant la magnitude de leurs outliers, les rendant ainsi plus facilement quantifiables. La difficulté est transférée aux poids $\hat{W}$, qui sont plus aptes à l'absorber.

**Pseudo-code :**

```
Algorithme : SmoothQuant
1: Entrée : Modèle M, Données de calibration D_cal, Force de migration alpha
2: Pour chaque couche linéaire L avec poids W et entrée X:
3:   Faire passer D_cal à travers M pour collecter les statistiques de X
4:   Pour chaque canal d'entrée j:
5:     s_j = max(|X_j|)^alpha / max(|W_j|)^(1-alpha)
6:   W_hat = diag(s) * W
7:   // Au moment de l'inférence, l'entrée de la couche sera X_hat = X * diag(s)^-1
8:   Remplacer W par W_hat dans la couche L
9: Quantifier W_hat et les activations (maintenant plus lisses) en utilisant le PTQ standard
```

#### Algorithme Avancé 2 : Activation-aware Weight Quantization (AWQ)

AWQ part d'une observation différente : tous les poids n'ont pas la même importance. Protéger une petite fraction (environ 1 %) des poids les plus "saillants" peut préserver la performance du modèle.

**Idée Clé :** L'importance d'un poids n'est pas déterminée par sa propre magnitude, mais par la magnitude des activations qu'il traite. Les canaux de poids qui multiplient des activations de grande magnitude sont plus importants. Pour protéger ces poids sans recourir à une précision mixte (inefficace sur le matériel), AWQ met à l'échelle les canaux de poids saillants pour réduire leur erreur de quantification relative.

**Pseudo-code :**

```
Algorithme : Activation-aware Weight Quantization (AWQ)
1: Entrée : Modèle M, Données de calibration D_cal
2: Pour chaque couche linéaire L avec poids W et entrée X:
3:   Faire passer D_cal à travers M pour collecter les échelles d'activation S_x = max(|X_j|) pour chaque canal j
4:   // Rechercher le meilleur facteur d'échelle par canal 's' pour minimiser l'erreur de quantification
5:   // C'est un problème de recherche, souvent simplifié
6:   s_j = S_x_j^alpha  // Version simplifiée, alpha est un hyperparamètre
7:   W_hat = W / diag(s)
8:   // Au moment de l'inférence, l'opération est (X * diag(s)) * Q(W_hat)
9:   Quantifier W_hat en utilisant une quantification par groupe (group-wise)
```

#### Implémentation en Python (PyTorch)

PyTorch offre un écosystème de quantification robuste via `torch.ao.quantization`. Voici un exemple de PTQ statique.

```python
import torch
import torch.nn as nn
from torch.ao.quantization import QuantStub, DeQuantStub, get_default_qconfig, prepare_qat, convert

# 1. Définir un modèle compatible avec la quantification
class QuantizableModel(nn.Module):
    def __init__(self):
        super(QuantizableModel, self).__init__()
        self.quant = QuantStub()  # Convertit FP32 -> INT8
        self.conv = nn.Conv2d(1, 1, 1)
        self.relu = nn.ReLU()
        self.dequant = DeQuantStub() # Convertit INT8 -> FP32

    def forward(self, x):
        x = self.quant(x)
        x = self.conv(x)
        x = self.relu(x)
        x = self.dequant(x)
        return x

# 2. Préparer le modèle pour la quantification
model_fp32 = QuantizableModel()
model_fp32.eval()
model_fp32.qconfig = get_default_qconfig('fbgemm')
model_fp32_prepared = torch.ao.quantization.prepare(model_fp32)

# 3. Calibrer avec des données représentatives
input_fp32 = torch.randn(4, 1, 4, 4)
model_fp32_prepared(input_fp32)

# 4. Convertir en modèle quantifié
model_int8 = torch.ao.quantization.convert(model_fp32_prepared)

# L'inférence se fait maintenant avec des opérations entières
output = model_int8(input_fp32)
print("Modèle quantifié exécuté avec succès.")
```

#### Implémentation en C/C++

En C++, la quantification implique de convertir manuellement les tenseurs de FP32 en INT8 en utilisant les paramètres d'échelle et de point zéro. Des bibliothèques comme QNNPACK ou gemmlowp sont cruciales pour fournir des noyaux de multiplication de matrices INT8 optimisés.

```cpp
// Implémentation conceptuelle de la quantification d'un tenseur en C++
#include <vector>
#include <cstdint>
#include <algorithm>
#include <cmath>
#include <limits>

struct QuantizationParams {
    float scale;
    int8_t zero_point;
};

void quantize_tensor(const std::vector<float>& fp32_tensor, 
                     std::vector<int8_t>& int8_tensor, 
                     QuantizationParams& params) {
    float r_min = *std::min_element(fp32_tensor.begin(), fp32_tensor.end());
    float r_max = *std::max_element(fp32_tensor.begin(), fp32_tensor.end());

    // Assurer que 0 est représentable
    r_min = std::min(0.0f, r_min);
    r_max = std::max(0.0f, r_max);

    const float q_min = std::numeric_limits<int8_t>::min(); // -128
    const float q_max = std::numeric_limits<int8_t>::max(); // 127

    params.scale = (r_max - r_min) / (q_max - q_min);
    params.zero_point = static_cast<int8_t>(std::round(q_min - r_min / params.scale));

    int8_tensor.resize(fp32_tensor.size());
    for (size_t i = 0; i < fp32_tensor.size(); ++i) {
        float val = std::round(fp32_tensor[i] / params.scale) + params.zero_point;
        int8_tensor[i] = static_cast<int8_t>(std::max(q_min, std::min(q_max, val)));
    }
}
```

### Recherche d'Architecture Neuronale (NAS)

La NAS automatise le processus de conception d'architectures de réseaux de neurones, qui est traditionnellement manuel, long et nécessite une expertise considérable. L'objectif est de découvrir des architectures qui offrent le meilleur compromis entre précision, latence et taille pour une tâche et un matériel cibles.

#### Concepts et Composants

Un système NAS est généralement composé de trois éléments :

1. **Espace de Recherche** : Définit l'ensemble de toutes les architectures possibles qui peuvent être conçues. Cela inclut les types d'opérations (convolutions, attention, etc.) et la manière dont elles peuvent être connectées.

2. **Stratégie de Recherche** : L'algorithme utilisé pour explorer l'espace de recherche. Les stratégies courantes incluent l'apprentissage par renforcement, les algorithmes évolutionnaires et les méthodes basées sur le gradient.

3. **Stratégie d'Évaluation de la Performance** : La méthode pour estimer la qualité d'une architecture candidate. L'entraînement complet de chaque architecture est trop coûteux, donc des techniques comme le partage de poids (via un "super-réseau") sont utilisées pour accélérer l'évaluation.

#### Étude de Cas : MCUNet - Co-conception pour Microcontrôleurs

Le déploiement de l'IA sur des microcontrôleurs, avec leur mémoire extrêmement limitée (souvent moins de 512 Ko de SRAM), représente un défi majeur. MCUNet aborde ce problème non pas comme un simple problème de compression, mais comme un problème de synthèse au niveau du système.

Cette approche représente un changement de paradigme. Au lieu de prendre un grand modèle et de le réduire, MCUNet synthétise une paire modèle-système optimale à partir de zéro. L'innovation clé est la co-conception de l'architecture neuronale (avec TinyNAS) et du moteur d'inférence (avec TinyEngine). Il ne s'agit pas de deux optimisations indépendantes, mais d'une boucle de rétroaction. Par exemple, TinyEngine a introduit une convolution en profondeur "in-place" qui réduit considérablement le pic de mémoire requis. Cette amélioration du moteur d'inférence a permis d'élargir l'espace de recherche pour TinyNAS, lui donnant accès à des architectures plus grandes et potentiellement plus précises qui auraient été impossibles à exécuter auparavant. C'est cette synergie qui permet d'atteindre des performances sans précédent sur du matériel aussi contraint.

- **TinyNAS** : Une méthode de recherche d'architecture en deux étapes. D'abord, il optimise l'espace de recherche lui-même pour l'adapter aux contraintes de la puce cible, puis il recherche la meilleure architecture spécialisée au sein de cet espace optimisé.

- **TinyEngine** : Une bibliothèque d'inférence ultra-légère qui utilise la génération de code (plutôt qu'un interpréteur) et une planification de la mémoire intelligente et adaptée au modèle pour minimiser l'empreinte mémoire et maximiser la vitesse.

### Distillation de Connaissances (Knowledge Distillation)

La distillation de connaissances est une technique d'entraînement où un modèle plus petit et plus rapide (l'étudiant) apprend à imiter le comportement d'un modèle plus grand et plus complexe (le professeur).

#### Le Paradigme Professeur-Élève

L'idée fondamentale est que la "connaissance" du modèle professeur ne réside pas seulement dans ses prédictions finales correctes (les "cibles dures"), mais aussi dans la manière dont il généralise, ce qui est encodé dans les probabilités qu'il assigne aux classes incorrectes (les "cibles molles" ou "soft targets"). Par exemple, un professeur qui voit une image de chat peut prédire "chat" avec 90% de confiance, mais aussi "chien" avec 5% et "voiture" avec 0.01%. Cette information relative ("un chat ressemble plus à un chien qu'à une voiture") est une forme de connaissance précieuse que l'étudiant peut apprendre.

#### Formulation Mathématique

Pour extraire cette information des cibles molles, on utilise une softmax avec température ($T$). Une température élevée ($T > 1$) "adoucit" la distribution de probabilités, augmentant la valeur des probabilités des classes incorrectes et les rendant plus informatives pour l'entraînement.

$$q_i = \frac{\exp(z_i/T)}{\sum_j \exp(z_j/T)}$$

La fonction de perte totale pour l'étudiant est une combinaison pondérée de deux termes :
1. Une perte d'entropie croisée standard avec les vraies étiquettes (cibles dures).
2. Une perte de distillation, souvent la divergence de Kullback-Leibler (KL), entre les prédictions adoucies de l'étudiant et celles du professeur.

$$L_{\text{total}} = \alpha \cdot L_{\text{CE}}(y_{\text{true}}, \sigma(z_S)) + (1-\alpha) \cdot T^2 \cdot L_{\text{KL}}(\sigma(z_T/T), \sigma(z_S/T))$$

Le terme $T^2$ est utilisé pour s'assurer que l'échelle du gradient de la perte sur les cibles molles reste cohérente lorsque la température change.

#### Algorithme : Entraînement par Distillation

**Pseudo-code :**

```
Algorithme : Distillation de Connaissances
1: Entrée : Modèle professeur M_T, Modèle étudiant M_S, Données D, Température T, Poids alpha
2: Entraîner M_T sur D jusqu'à convergence
3: Pour chaque batch (x, y) dans D:
4:   // Obtenir les prédictions molles du professeur
5:   logits_T = M_T(x)
6:   // Passe avant pour l'étudiant
7:   logits_S = M_S(x)
8:   // Calculer la perte d'entropie croisée standard avec les étiquettes dures
9:   loss_hard = CrossEntropy(y, softmax(logits_S))
10:  // Calculer la perte de distillation avec les étiquettes molles
11:  loss_soft = KL_Divergence(softmax(logits_T/T), softmax(logits_S/T))
12:  // Combiner les pertes
13:  loss_total = alpha * loss_hard + (1 - alpha) * T*T * loss_soft
14:  // Rétropropagation et mise à jour de M_S
15:  loss_total.backward()
16:  optimizer.step()
17: Sortie : Modèle étudiant entraîné M_S
```

#### Implémentation en Python (PyTorch)

```python
import torch
import torch.nn as nn
import torch.nn.functional as F

def distillation_loss(student_logits, teacher_logits, hard_labels, temp, alpha):
    # Perte sur les cibles molles (soft targets)
    soft_loss = nn.KLDivLoss(reduction='batchmean')(
        F.log_softmax(student_logits / temp, dim=1),
        F.softmax(teacher_logits / temp, dim=1)
    ) * (temp * temp)

    # Perte sur les cibles dures (hard targets)
    hard_loss = F.cross_entropy(student_logits, hard_labels)

    # Combinaison des deux pertes
    total_loss = alpha * hard_loss + (1. - alpha) * soft_loss
    return total_loss

# Dans la boucle d'entraînement...
# teacher_model.eval()
# student_outputs = student_model(inputs)
# with torch.no_grad():
#     teacher_outputs = teacher_model(inputs)
#
# loss = distillation_loss(student_outputs, teacher_outputs, labels, temp=4.0, alpha=0.3)
# loss.backward()
# optimizer.step()
```

#### Implémentation en C/C++

La distillation étant un processus d'entraînement, son implémentation en C++ est moins courante. Elle nécessiterait un framework d'entraînement C++ comme LibTorch. Le concept principal serait de définir une fonction de perte personnalisée qui calcule les deux composantes (dure et molle) et les combine, comme montré dans l'exemple Python.

## Optimisations Spécifiques aux Applications

Les techniques fondamentales de compression sont puissantes, mais leur véritable impact se manifeste lorsqu'elles sont adaptées et appliquées à des domaines spécifiques. Cette section explore plusieurs projets de recherche de pointe qui ont utilisé ces principes pour résoudre des problèmes concrets en vision par ordinateur, traitement du langage naturel, IA générative et perception 3D.

### Vision par Ordinateur Efficace

#### Accélération des Transformateurs de Vision (ViT)

Les Vision Transformers (ViT) ont révolutionné la vision par ordinateur, mais leur complexité quadratique en fait des modèles coûteux, en particulier pour les tâches à haute résolution comme la segmentation sémantique. Le Segment Anything Model (SAM), par exemple, bien que très puissant, est trop lent pour de nombreuses applications en temps réel, fonctionnant à environ 12 images par seconde sur un GPU de centre de données.

Le projet **EfficientViT** s'attaque directement à ce problème. Il introduit une nouvelle famille de modèles de vision basés sur une attention multi-échelle légère qui remplace le mécanisme d'auto-attention coûteux. En utilisant une attention linéaire basée sur ReLU, EfficientViT réduit la complexité de quadratique à linéaire tout en conservant un champ réceptif global. Cette approche, combinée à d'autres optimisations, a permis d'accélérer SAM de manière spectaculaire, atteignant 842 images par seconde — une accélération de 70x — sans perte de précision de segmentation.

#### Déploiement sur Dispositifs Embarqués

L'un des objectifs ultimes du TinyML est d'exécuter des modèles d'IA utiles sur des microcontrôleurs peu coûteux et à très faible consommation d'énergie.

**Projet MCUNet :** Ce projet démontre la faisabilité de cet objectif en exécutant des tâches de détection de personnes et de masques faciaux sur des microcontrôleurs coûtant moins d'un dollar. Le succès de MCUNet repose sur sa méthodologie de co-conception, où l'architecture du réseau et le moteur d'inférence sont optimisés conjointement pour le matériel cible.

**Formation sur Appareil (On-Device Training) :** Au-delà de l'inférence, il est même possible d'entraîner des modèles directement sur ces appareils. Une démonstration a montré l'entraînement d'un classificateur binaire (personne vs non-personne) sur un microcontrôleur avec seulement 256 Ko de RAM. Ce paradigme offre des avantages considérables en termes de :
- **Confidentialité** : les données brutes ne quittent jamais l'appareil
- **Personnalisation** : le modèle peut s'adapter à l'utilisateur ou à l'environnement local
- **Apprentissage continu** : le modèle peut se mettre à jour constamment sans connexion au cloud

### Grands Modèles de Langage (LLM) Efficaces

Le déploiement de LLMs est l'un des plus grands défis de l'IA efficace aujourd'hui.

**Optimisation de l'Architecture Transformer :** Des techniques comme l'élagage de tokens permettent de réduire la charge de calcul en supprimant dynamiquement les mots redondants d'une phrase. Par exemple, pour une tâche de classification de sentiment sur la phrase "As a visual treat, the film is almost perfect", on peut la réduire à "film perfect" tout en conservant l'information sémantique essentielle.

**Déploiement de LLMs sur Appareils Grand Public :** Le projet **TinyChat** est un excellent exemple de la manière dont les techniques de compression peuvent rendre les LLMs accessibles. TinyChat est un moteur d'inférence C++ hautement optimisé qui utilise des techniques de bas niveau telles que la programmation parallèle (multithreading, SIMD) et l'optimisation de la localité du cache. Combiné avec des algorithmes de quantification avancés comme AWQ (quantification 4 bits), TinyChat permet d'exécuter des modèles comme Llama 2 7B localement sur un MacBook Air M1 ou un NVIDIA Jetson Orin, ouvrant la voie à des applications de LLM privées, hors ligne et en temps réel sur des appareils grand public.

### IA Générative Efficace

L'IA générative, bien que spectaculaire, est extrêmement gourmande en ressources.

**Compression des GANs :** Le projet **GAN Compression** a montré que l'élagage peut réduire drastiquement le coût de calcul des réseaux antagonistes génératifs. Par exemple, en appliquant l'élagage à CycleGAN pour la tâche de traduction d'image de cheval à zèbre, la vitesse d'inférence a été augmentée de 12 à 40 images par seconde, soit une accélération de plus de 3x.

**Génération d'Images Personnalisées Efficace :** Les méthodes traditionnelles comme DreamBooth nécessitent un fine-tuning coûteux pour chaque nouveau sujet que l'on souhaite générer. Le projet **FastComposer** élimine ce besoin. Il utilise un encodeur d'image pour extraire des "embeddings de sujet" à partir d'images de référence, puis injecte ces embeddings dans le prompt textuel. Cela permet une génération personnalisée et multi-sujets en une seule passe avant (inférence seulement), sans aucun ré-entraînement. Pour éviter que les identités des différents sujets ne se mélangent, il utilise une technique de supervision de la localisation de l'attention croisée pendant l'entraînement.

### Perception 3D Efficace pour la Conduite Autonome

La conduite autonome repose sur le traitement en temps réel de données provenant de multiples capteurs, ce qui exige une efficacité de calcul extrême.

**Traitement des Données LiDAR :** Les données LiDAR, sous forme de nuages de points, sont par nature creuses et donc inefficaces à traiter avec des convolutions denses standard. Le projet **Fast-LidarNet** résout ce problème en projetant le nuage de points 3D sur une grille 2D en vue de dessus (top-view). Cette représentation dense peut ensuite être traitée efficacement par un réseau entièrement convolutionnel pour des tâches comme la détection de la route, atteignant des vitesses d'inférence en temps réel.

**Fusion Multi-capteurs en Vue Plongeante (BEV) :** La fusion des données des caméras (riches en sémantique) et du LiDAR (précis en géométrie) est cruciale. Le projet **BEVFusion** propose un framework unifié où les caractéristiques de tous les capteurs sont projetées dans un espace de représentation commun en vue plongeante (Bird's-Eye View). L'étape clé, la transformation de la vue de la caméra vers la BEV, est un goulot d'étranglement. BEVFusion introduit un "BEV pooling" optimisé qui réduit la latence de cette étape de plus de 40 fois, permettant une fusion multi-modale efficace sur du matériel embarqué.

## Matériel pour l'IA et Co-conception

L'efficacité en apprentissage automatique n'est pas une propriété intrinsèque d'un algorithme ; elle est toujours relative à la plateforme matérielle sur laquelle il s'exécute. Comprendre le paysage matériel est donc essentiel pour concevoir des systèmes d'IA véritablement performants.

### Le Spectre du Matériel d'IA : Du Cloud à l'Edge

Le matériel d'IA n'est pas monolithique. Il existe un large spectre de plateformes, chacune avec ses propres compromis en termes de performance, de consommation d'énergie et de coût. À une extrémité, nous avons les GPU de centre de données (Cloud), optimisés pour un débit de calcul maximal (mesuré en TFLOPS). À l'autre extrémité, nous avons les dispositifs embarqués (Edge), optimisés pour l'efficacité énergétique (mesurée en TOPS/Watt).

Cette différence fondamentale dans les objectifs d'optimisation a des implications profondes sur la conception des algorithmes. Un algorithme considéré comme "efficace" pour le cloud, qui pourrait par exemple utiliser de grands lots de données pour saturer les cœurs de calcul d'un GPU, pourrait être très inefficace sur un appareil mobile où la latence d'une seule inférence et la consommation de la batterie sont les facteurs les plus importants. Par conséquent, la co-conception algorithme-matériel n'est pas seulement une stratégie pour les microcontrôleurs, mais un principe directeur pour l'ensemble du domaine de l'IA efficace.

Le tableau suivant résume les caractéristiques clés de plusieurs plateformes représentatives sur ce spectre.

| Catégorie | Plateforme Exemple | Performance (INT8) | Mémoire | Puissance (TDP) | Cas d'Usage Principal |
|-----------|-------------------|-------------------|---------|-----------------|----------------------|
| **Cloud** | NVIDIA H100 (SXM) | 3,958 TOPS | 80 Go HBM3 | 700 W | Entraînement de LLM, Inférence à grande échelle |
| **Edge (Haut de gamme)** | NVIDIA Jetson AGX Orin | 275 TOPS | 64 Go LPDDR5 | 15-60 W | Robotique, Conduite Autonome |
| **Edge (Mobile)** | Qualcomm Snapdragon 8 Gen 2 | ~40 TOPS (estimé) | Partagée (8-16 Go LPDDR5) | ~10 W | Smartphones, Tablettes |
| **Edge (IoT)** | Google Coral Edge TPU | 4 TOPS | Partagée | 2 W | Inférence de vision à faible puissance |
| **Microcontrôleur** | Cortex-M7 | ~MOPS | ~320 Ko SRAM | ~Milliwatts | Détection de mots-clés, Capteurs intelligents |

### Accélérateurs pour le Cloud et l'Edge

**NVIDIA A100/H100 :** Ces GPU dominent le marché du cloud et de l'entraînement à grande échelle. Leurs architectures (Ampere et Hopper) intègrent des Tensor Cores spécialisés pour les multiplications de matrices, ainsi qu'un support matériel pour la sparsité structurée, ce qui permet d'accélérer directement les modèles élagués.

**NVIDIA Jetson Series (Orin) :** Ces systèmes sur puce (SoC) sont conçus pour les applications embarquées hautes performances comme la robotique. Ils combinent un GPU d'architecture Ampere avec des cœurs CPU ARM, offrant une puissance de calcul significative dans une enveloppe de puissance contrôlée.

**Qualcomm Snapdragon :** Au cœur des smartphones haut de gamme, ces puces intègrent un moteur d'IA qui s'appuie fortement sur le processeur de signal numérique (DSP) Hexagon. Ce dernier est optimisé pour exécuter des opérations de réseaux de neurones avec une très faible consommation d'énergie, ce qui est crucial pour les appareils alimentés par batterie.

**Google Coral Edge TPU :** Il s'agit d'un ASIC (circuit intégré spécifique à une application) conçu exclusivement pour accélérer l'inférence des modèles TensorFlow Lite. Il offre un excellent rapport performance/watt, ce qui le rend idéal pour les applications IoT où la consommation d'énergie est une contrainte majeure.

### Principes de Programmation Efficace

Pour exploiter pleinement le potentiel de ce matériel, il est souvent nécessaire de descendre à un bas niveau de programmation, comme l'a démontré le projet TinyChat. Les techniques clés incluent :

**Parallélisme :** Utiliser le multithreading pour répartir les calculs sur plusieurs cœurs de CPU.

**Vectorisation (SIMD) :** Exploiter les instructions Single Instruction, Multiple Data (comme AVX ou NEON) pour effectuer la même opération sur plusieurs éléments de données en un seul cycle d'horloge.

**Optimisation du Cache :** La mémoire principale (DRAM) est lente par rapport au processeur. Des techniques comme le déroulage de boucle (loop unrolling) et le tuilage (tiling) sont utilisées pour organiser les calculs de manière à maximiser la réutilisation des données présentes dans les caches rapides du processeur, minimisant ainsi les accès coûteux à la DRAM.

## Formation Efficace et Conclusion

L'efficacité ne concerne pas seulement l'inférence ; elle est également cruciale pour la phase d'entraînement, qui est souvent la plus coûteuse en termes de calcul.

### Formation Distribuée

L'entraînement de modèles massifs comme les LLMs est impossible sur une seule machine. La formation distribuée est donc essentielle. Elle consiste à répartir la charge de travail sur de nombreux GPU ou nœuds de calcul. Les principales stratégies sont :

**Parallélisme de Données :** La stratégie la plus courante. Le modèle est répliqué sur chaque GPU, et chaque GPU traite un sous-ensemble différent des données d'entraînement.

**Parallélisme de Modèle :** Lorsque le modèle est trop grand pour tenir dans la mémoire d'un seul GPU, il est lui-même divisé, et différentes parties du modèle sont placées sur différents GPU.

**Parallélisme de Pipeline :** Une forme de parallélisme de modèle où les couches du réseau sont réparties sur plusieurs GPU, et les lots de données sont traités en pipeline à travers ces étapes.

### Formation sur Appareil (On-Device Training)

À l'autre extrémité du spectre, la formation sur appareil gagne en importance. Comme nous l'avons vu avec la démonstration sur microcontrôleur, l'entraînement de petits modèles ou le fine-tuning de modèles plus grands directement sur l'appareil de l'utilisateur offre des avantages uniques :

**Confidentialité :** Les données sensibles de l'utilisateur (images, voix) n'ont pas besoin d'être envoyées au cloud.

**Personnalisation :** Le modèle peut s'adapter spécifiquement à l'utilisateur, à ses données et à son environnement.

**Apprentissage Continu :** Le modèle peut apprendre et s'améliorer en permanence à partir de nouvelles données, sans dépendre d'une connexion réseau.

## Synthèse et Perspectives d'Avenir

Ce cours a démontré que l'IA efficace est bien plus qu'une simple optimisation. C'est une discipline fondamentale qui se situe à l'intersection des algorithmes, des systèmes logiciels et de la conception matérielle. L'objectif ultime est de transformer l'IA d'une technologie qui nécessite d'énormes quantités de calcul, d'énergie (et donc d'empreinte carbone), d'expertise en ingénierie et de données, en une technologie plus "verte", accessible et durable.

Des techniques comme l'élagage, la quantification, la recherche d'architectures et la distillation de connaissances ne sont pas des solutions isolées, mais des outils dans une boîte à outils complète. Leur application intelligente, guidée par une compréhension approfondie du matériel cible et des exigences de l'application, est la clé pour débloquer la prochaine génération d'applications d'IA, des grands modèles de langage fonctionnant sur nos ordinateurs portables aux capteurs intelligents et omniprésents qui façonneront notre avenir.
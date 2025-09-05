# Les Fondations des Réseaux de Neurones Efficaces : Théorie, Architectures et Implémentations

## Introduction : La Quête de l'Efficacité en Intelligence Artificielle

### Contexte et Motivation

L'intelligence artificielle moderne a atteint des capacités impressionnantes, résolvant des problèmes autrefois considérés comme insolubles. Des grands modèles de langage (LLM) comme GPT-3 aux systèmes d'IA générative capables de créer des images et du code, en passant par les technologies de conduite autonome, ces applications transforment notre monde. Cependant, cette révolution a un coût. Pour atteindre de tels niveaux de performance, ces modèles nécessitent une quantité de calcul et de mémoire qui croît à un rythme vertigineux.

Cette situation a créé ce que l'on peut appeler un "fossé computationnel". Alors que la loi de Moore, qui a longtemps régi l'industrie des semi-conducteurs, postule un doublement du nombre de transistors sur une puce tous les deux ans, la demande en calcul des modèles d'apprentissage profond de pointe augmente beaucoup plus rapidement, quadruplant, voire plus, sur la même période. L'offre de puissance de calcul, bien qu'en croissance, peine à suivre la demande exponentielle des algorithmes d'IA. Cette tension fondamentale est le principal moteur de la recherche en IA efficace, un domaine qui ne cherche pas seulement à améliorer la précision des modèles, mais aussi à les rendre utilisables dans des conditions réelles, sur des appareils aux ressources limitées et avec une consommation énergétique maîtrisée.

### Présentation des Solutions

Pour combler ce fossé, deux axes stratégiques majeurs sont explorés. Le premier est la **compression de modèles**, qui regroupe un ensemble de techniques visant à réduire la taille et la complexité des réseaux de neurones sans sacrifier de manière significative leur performance. Parmi ces techniques, on trouve l'élagage (pruning), la quantification, la distillation de connaissances (knowledge distillation) et la recherche d'architectures neuronales (Neural Architecture Search, NAS). Le second axe est le **calcul efficace**, qui se concentre sur l'optimisation de l'exécution de ces modèles, que ce soit par la conception d'architectures matérielles spécialisées ou par des optimisations logicielles spécifiques à un domaine d'application.

L'impact de ce domaine est tangible. Des modèles comme EfficientViT, par exemple, ont permis d'accélérer drastiquement le traitement d'images à haute résolution pour des tâches de segmentation de scène, passant de 1.6 à 21 images par seconde sur un processeur embarqué comme le NVIDIA Jetson AGX Orin. De même, des optimisations sur des modèles de segmentation comme SAM (Segment Anything Model) ont permis des accélérations de plus de 70x, rendant possibles des applications interactives en temps réel. Ces succès démontrent qu'il est possible de concilier haute performance et efficacité.

### Structure du Cours

Ce cours est conçu pour fournir une compréhension approfondie de l'IA efficace, en trois grandes parties. La première partie se concentrera sur les techniques d'inférence efficace, en abordant d'abord les méthodes générales (élagage, quantification), puis en se penchant sur des optimisations spécifiques à des domaines comme les grands modèles de langage et les transformeurs de vision (Vision Transformers). La deuxième partie traitera des techniques d'entraînement, notamment l'entraînement distribué à grande échelle nécessaire pour des modèles comme GPT-3. La leçon d'aujourd'hui constitue le socle de tout le cours : les fondations des réseaux de neurones. Nous y reverrons les concepts de base, les couches fondamentales, les architectures canoniques et, surtout, les métriques qui nous permettent de mesurer et de raisonner sur l'efficacité.

## Module 1 : Les Fondamentaux des Réseaux de Neurones

### 1.1 De la Biologie à l'Artificiel : Le Neurone et la Synapse

Pour bâtir une fondation solide, il est utile de revenir à l'analogie biologique qui a inspiré les premiers réseaux de neurones. Dans le cerveau, un neurone reçoit des signaux électriques via ses dendrites. Si la somme de ces signaux dépasse un certain seuil, le neurone "s'active" et envoie à son tour un signal le long de son axone vers d'autres neurones. La connexion entre l'axone d'un neurone et la dendrite d'un autre est appelée une synapse, et la force de cette connexion peut varier.

En apprentissage automatique, nous modélisons ce processus de manière mathématique. Il est crucial de maîtriser la terminologie, car plusieurs termes sont souvent utilisés de manière interchangeable.

**Synapses / Poids (Weights) / Paramètres** : Ces termes désignent la même chose : la force de la connexion entre deux neurones. Ce sont les valeurs numériques que le réseau apprend pendant l'entraînement. Quand on parle d'un modèle avec 175 milliards de paramètres, on parle de 175 milliards de poids, ou synapses artificielles.

**Activations / Neurones / Caractéristiques (Features)** : Ces termes désignent la sortie d'un neurone après qu'il a additionné ses entrées pondérées et appliqué une fonction (dite d'activation). C'est la valeur qui est transmise à la couche suivante du réseau.

### 1.2 Dimensions d'un Réseau : Largeur et Profondeur

L'architecture d'un réseau de neurones est principalement définie par deux dimensions : sa largeur et sa profondeur.

**Largeur (Width)** : La largeur d'un réseau fait référence à la taille de ses couches, plus précisément au nombre de neurones (ou d'activations) dans une couche cachée. Un réseau est dit "large" s'il possède de nombreuses unités de calcul en parallèle au sein de ses couches.

**Profondeur (Depth)** : La profondeur d'un réseau est simplement le nombre de couches qui le composent. Un "réseau de neurones profond" (Deep Neural Network) est un réseau qui empile un grand nombre de couches les unes après les autres.

### 1.3 Le Compromis Architecturel et l'Interaction Matériel-Logiciel

Le choix entre un réseau large et peu profond ou un réseau étroit et profond n'est pas anodin et représente un compromis fondamental entre performance de calcul et capacité de modélisation.

**Réseaux larges et peu profonds** : Ces architectures ont tendance à être plus rapides sur les processeurs graphiques (GPU). La raison est double. Premièrement, des couches plus larges impliquent des multiplications de matrices plus grandes, ce qui permet de mieux saturer les milliers de cœurs de calcul d'un GPU et d'atteindre une meilleure utilisation du matériel. Deuxièmement, un nombre de couches plus faible signifie moins d'appels de noyau ("kernel calls"), c'est-à-dire moins de commandes distinctes envoyées par le CPU au GPU. Chaque appel de noyau a un surcoût fixe ; en minimiser le nombre réduit donc la latence globale.

**Réseaux étroits et profonds** : Historiquement, la profondeur a été la clé pour atteindre des niveaux de précision de pointe. La hiérarchie de couches permet au modèle d'apprendre des caractéristiques de plus en plus abstraites et complexes. Ces modèles sont souvent plus faciles à faire converger vers une bonne solution. Cependant, ils peuvent être inefficaces sur GPU si les matrices de chaque couche sont trop petites pour utiliser pleinement la puissance de calcul disponible. De plus, le grand nombre de couches multiplie le surcoût lié aux appels de noyau.

Cette discussion révèle un principe fondamental de l'IA efficace : la **co-conception matériel-logiciel**. L'architecture "optimale" n'existe pas dans l'absolu ; elle est intrinsèquement liée au matériel sur lequel elle s'exécute. L'histoire des réseaux de neurones est remplie d'exemples où les contraintes matérielles ont directement conduit à des innovations architecturales. AlexNet, le réseau qui a popularisé l'apprentissage profond en 2012, a été conçu pour s'exécuter sur deux GPU NVIDIA GTX 580, chacun avec seulement 1.5 Go de mémoire. Cette contrainte a directement mené à l'invention de la convolution groupée, une technique où les canaux de convolution sont répartis entre les deux GPU, réduisant ainsi la charge de calcul et de mémoire sur chacun. Plus récemment, l'architecture Transformer, qui est au cœur des LLM, doit son succès en partie au fait que son mécanisme d'attention, bien que de complexité théorique quadratique par rapport à la longueur de la séquence, est massivement parallélisable, ce qui le rend parfaitement adapté aux GPU modernes.

### 1.4 Le Champ Récepteur (Receptive Field)

Dans les réseaux de neurones convolutifs (CNN), le champ récepteur est un concept essentiel. Il définit la taille de la région de l'image d'entrée qui influence la valeur d'un seul pixel dans une carte de caractéristiques de sortie. Un champ récepteur plus large permet au réseau de "voir" un contexte plus global et de comprendre les relations entre des objets distants dans une image, ce qui est crucial pour des tâches complexes comme la compréhension de scènes.

Le calcul de la taille du champ récepteur $R_k$ après $k$ couches successives est donné par la formule générale suivante, où $F_j$ est la taille du noyau (filtre) et $S_i$ est le pas (stride) de la couche $i$ :

$$R_k = 1 + \sum_{j=1}^{k} (F_j - 1) \prod_{i=0}^{j-1} S_i$$

Il existe trois stratégies principales pour augmenter le champ récepteur :

1. **Augmenter la profondeur** : Ajouter plus de couches est une manière naturelle d'augmenter le champ récepteur. Chaque couche supplémentaire l'agrandit. Cependant, cela augmente la latence et la complexité du modèle.

2. **Augmenter la taille du noyau** : Utiliser des filtres plus grands (par exemple, 7×7 au lieu de 3×3) augmente directement le champ récepteur à chaque couche. Le coût est une augmentation quadratique du nombre de paramètres et de calculs.

3. **Le sous-échantillonnage (Downsampling)** : C'est la méthode la plus efficace. En utilisant un pas (stride) supérieur à 1 dans une couche de convolution ou en ajoutant une couche de pooling, on réduit la résolution de la carte de caractéristiques. Cela a pour effet d'augmenter de manière significative le champ récepteur effectif des couches suivantes par rapport à l'entrée originale, sans ajouter de paramètres ou de calculs supplémentaires.

## Module 2 : Les Blocs de Construction Essentiels : Les Couches Neuronales

Ce module détaille les couches fondamentales qui composent la quasi-totalité des réseaux de neurones modernes. Pour chaque couche, nous présenterons sa théorie, ses formules mathématiques, son pseudo-code, et des implémentations complètes en Python (avec NumPy) et en C++, avant de montrer son utilisation simple avec la bibliothèque PyTorch.

### 2.1 La Couche Linéaire (Entièrement Connectée / Fully Connected)

#### Théorie

La couche linéaire, ou entièrement connectée, est l'un des blocs de construction les plus fondamentaux. Elle applique une transformation affine aux données d'entrée. Son nom "entièrement connectée" vient du fait que chaque neurone de la couche de sortie est connecté à tous les neurones de la couche d'entrée. Cette couche est capable de gérer des entrées individuelles ou des lots (batch) d'entrées, ce qui est la norme en apprentissage profond.

#### Formules

Soit une entrée $X$ (un batch de $N$ échantillons, chacun de dimension $C_{in}$), une matrice de poids $A$ et un vecteur de biais $b$. La sortie $Y$ est calculée comme suit :

- **Entrée** : $X \in \mathbb{R}^{N \times C_{in}}$
- **Poids** : $A \in \mathbb{R}^{C_{out} \times C_{in}}$
- **Biais** : $b \in \mathbb{R}^{1 \times C_{out}}$
- **Sortie** : $Y = XA^T + b$, où $Y \in \mathbb{R}^{N \times C_{out}}$

#### Pseudo-code

```
FONCTION CoucheLineaire(X, W, b):
  N = nombre de lignes de X
  C_in = nombre de colonnes de X
  C_out = nombre de lignes de W

  // Initialiser la matrice de sortie
  Y = nouvelle Matrice(N, C_out)

  // Itérer sur chaque échantillon du batch
  POUR i DE 0 A N-1:
    // Itérer sur chaque neurone de sortie
    POUR j DE 0 A C_out-1:
      somme = 0
      // Calculer le produit scalaire
      POUR k DE 0 A C_in-1:
        somme = somme + X[i, k] * W[j, k]
      FIN POUR
      // Ajouter le biais
      Y[i, j] = somme + b[j]
    FIN POUR
  FIN POUR

  RETOURNER Y
```

#### Implémentation en Python (NumPy)

```python
import numpy as np

def linear_layer_forward(X, W, b):
    """
    Calcule la passe avant d'une couche linéaire.

    Args:
        X (np.array): Entrée de forme (N, C_in).
        W (np.array): Poids de forme (C_out, C_in).
        b (np.array): Biais de forme (1, C_out).

    Returns:
        np.array: Sortie de forme (N, C_out).
    """
    # La multiplication matricielle X @ W.T effectue le calcul
    # de manière efficace pour tout le batch.
    # Le biais b est ajouté à chaque ligne de la sortie grâce au broadcasting.
    return X @ W.T + b

# Exemple d'utilisation
N, C_in, C_out = 4, 5, 3  # Batch size=4, 5 features en entrée, 3 en sortie
X = np.random.randn(N, C_in)
W = np.random.randn(C_out, C_in)
b = np.random.randn(1, C_out)

Y = linear_layer_forward(X, W, b)
print("Forme de l'entrée X:", X.shape)
print("Forme des poids W:", W.shape)
print("Forme du biais b:", b.shape)
print("Forme de la sortie Y:", Y.shape)
# Forme de la sortie Y: (4, 3)
```

#### Implémentation en C++

Cette implémentation utilise les `std::vector` de la bibliothèque standard pour plus de clarté pédagogique. Pour des applications performantes, une bibliothèque d'algèbre linéaire comme Eigen est fortement recommandée.

```cpp
#include <iostream>
#include <vector>
#include <numeric>

// Utilisation de typedef pour la clarté
using Matrix = std::vector<std::vector<float>>;
using Vector = std::vector<float>;

Matrix linear_layer_forward_cpp(const Matrix& X, const Matrix& W, const Vector& b) {
    size_t N = X.size();
    if (N == 0) return {};
    size_t C_in = X[0].size();
    size_t C_out = W.size();

    // Initialiser la matrice de sortie avec des zéros
    Matrix Y(N, Vector(C_out, 0.0f));

    for (size_t i = 0; i < N; ++i) {       // Itérer sur le batch
        for (size_t j = 0; j < C_out; ++j) { // Itérer sur les neurones de sortie
            float sum = 0.0f;
            for (size_t k = 0; k < C_in; ++k) { // Produit scalaire
                sum += X[i][k] * W[j][k];
            }
            Y[i][j] = sum + b[j]; // Ajouter le biais
        }
    }
    return Y;
}

int main() {
    // Exemple d'utilisation
    size_t N = 4, C_in = 5, C_out = 3;
    Matrix X = {{1, 2, 3, 4, 5}, {6, 7, 8, 9, 10}, {11, 12, 13, 14, 15}, {16, 17, 18, 19, 20}};
    Matrix W = {{0.1, 0.2, 0.3, 0.4, 0.5}, {0.6, 0.7, 0.8, 0.9, 1.0}, {1.1, 1.2, 1.3, 1.4, 1.5}};
    Vector b = {0.1, 0.2, 0.3};

    Matrix Y = linear_layer_forward_cpp(X, W, b);

    std::cout << "Sortie Y:" << std::endl;
    for (const auto& row : Y) {
        for (float val : row) {
            std::cout << val << " ";
        }
        std::cout << std::endl;
    }

    return 0;
}
```

#### Utilisation avec PyTorch

PyTorch fournit une implémentation optimisée et facile à utiliser via le module `torch.nn.Linear`.

```python
import torch
import torch.nn as nn

# Définir les dimensions
N, C_in, C_out = 4, 5, 3

# Créer la couche linéaire
linear_layer = nn.Linear(in_features=C_in, out_features=C_out, bias=True)

# Créer un tenseur d'entrée aléatoire
X_torch = torch.randn(N, C_in)

# Appliquer la couche
Y_torch = linear_layer(X_torch)

print("Poids de la couche PyTorch:", linear_layer.weight.shape)
print("Biais de la couche PyTorch:", linear_layer.bias.shape)
print("Sortie de la couche PyTorch:", Y_torch.shape)
# Sortie: torch.Size([4, 3])
```

### 2.2 La Couche de Convolution (Conv2D)

#### Théorie

La couche de convolution est le pilier des réseaux de neurones pour la vision par ordinateur. Au lieu de connecter chaque neurone de sortie à toutes les entrées, une couche de convolution applique un petit filtre, appelé noyau (kernel), de manière répétée sur des régions locales de l'image d'entrée. Ce processus a deux avantages majeurs :

1. **Partage de poids (Weight Sharing)** : Le même noyau est utilisé sur toute l'image, ce qui réduit considérablement le nombre de paramètres à apprendre par rapport à une couche entièrement connectée.

2. **Invariance à la translation** : Puisque le même filtre est utilisé partout, le réseau peut détecter une caractéristique (comme une arête verticale) quel que soit son emplacement dans l'image.

Il est important de noter que l'opération implémentée dans la plupart des bibliothèques d'apprentissage profond est techniquement une corrélation croisée et non une convolution mathématique pure, qui nécessiterait de retourner le filtre. En pratique, comme les poids du filtre sont appris, cette distinction n'a pas d'impact sur la capacité du réseau.

#### Formules

L'opération de corrélation croisée 2D pour une sortie au pixel $(i,j)$ est définie par :

$$O(i,j) = \sum_{m=0}^{k_h-1} \sum_{n=0}^{k_w-1} I(i+m,j+n) \cdot K(m,n)$$

où $I$ est l'image d'entrée, $K$ est le noyau de taille $k_h \times k_w$, et $O$ est la carte de caractéristiques de sortie.

La maîtrise des transformations de dimensions des tenseurs est une compétence fondamentale. Le tableau suivant centralise les formules pour calculer la taille de la sortie d'une couche de convolution, en fonction de ses hyperparamètres.

| Paramètre | Formule de Calcul de la Taille de Sortie (Hauteur ou Largeur) |
|-----------|----------------------------------------------------------------|
| Convolution standard | $H_{out} = H_{in} - k_h + 1$ |
| Avec Marge (Padding, P) | $H_{out} = H_{in} - k_h + 2P + 1$ |
| Avec Pas (Stride, S) | $H_{out} = \lfloor \frac{H_{in} - k_h}{S} \rfloor + 1$ |
| Avec Marge (P) et Pas (S) | $H_{out} = \lfloor \frac{H_{in} - k_h + 2P}{S} \rfloor + 1$ |
| Avec Dilatation (D) | $H_{out} = \lfloor \frac{H_{in} + 2P - D \times (k_h - 1) - 1}{S} \rfloor + 1$ |

#### Pseudo-code

```
FONCTION Conv2D(Input, Kernels, Stride, Padding):
  // Input: H_in x W_in x C_in
  // Kernels: k_h x k_w x C_in x C_out

  // 1. Appliquer le remplissage (padding) à l'Input
  PaddedInput = pad(Input, Padding)
  H_padded = H_in + 2*Padding; W_padded = W_in + 2*Padding

  // 2. Calculer les dimensions de sortie
  H_out = floor((H_padded - k_h) / Stride) + 1
  W_out = floor((W_padded - k_w) / Stride) + 1

  // 3. Initialiser la sortie
  Output = nouvelle Matrice(H_out, W_out, C_out)

  // 4. Itérer sur chaque filtre de sortie
  POUR c_out DE 0 A C_out-1:
    // Itérer sur les positions de la sortie
    POUR y DE 0 A H_out-1:
      POUR x DE 0 A W_out-1:
        somme = 0
        // Appliquer le filtre sur la région correspondante de l'entrée
        POUR c_in DE 0 A C_in-1:
          POUR ky DE 0 A k_h-1:
            POUR kx DE 0 A k_w-1:
              input_y = y * Stride + ky
              input_x = x * Stride + kx
              somme = somme + PaddedInput[input_y, input_x, c_in] * Kernels[ky, kx, c_in, c_out]
            FIN POUR
          FIN POUR
        FIN POUR
        Output[y, x, c_out] = somme // + Biais[c_out]
      FIN POUR
    FIN POUR
  FIN POUR

  RETOURNER Output
```

#### Implémentation en Python (NumPy)

```python
import numpy as np

def conv2d_forward(X, W, b, stride=1, padding=0):
    """
    Calcule la passe avant d'une couche de convolution 2D.
    X: Entrée de forme (N, C_in, H_in, W_in)
    W: Poids de forme (C_out, C_in, k_h, k_w)
    b: Biais de forme (C_out,)
    """
    N, C_in, H_in, W_in = X.shape
    C_out, _, k_h, k_w = W.shape

    # Appliquer le padding
    X_padded = np.pad(X, ((0, 0), (0, 0), (padding, padding), (padding, padding)), mode='constant')

    # Calculer les dimensions de sortie
    H_out = (H_in + 2 * padding - k_h) // stride + 1
    W_out = (W_in + 2 * padding - k_w) // stride + 1

    # Initialiser la sortie
    Y = np.zeros((N, C_out, H_out, W_out))

    for n in range(N):  # Pour chaque image dans le batch
        for c_out in range(C_out):  # Pour chaque filtre de sortie
            for h in range(H_out):  # Pour chaque position verticale
                for w in range(W_out):  # Pour chaque position horizontale
                    # Extraire la "fenêtre" de l'entrée
                    h_start = h * stride
                    h_end = h_start + k_h
                    w_start = w * stride
                    w_end = w_start + k_w
                    
                    window = X_padded[n, :, h_start:h_end, w_start:w_end]
                    
                    # Calculer la convolution (produit élément par élément et somme)
                    Y[n, c_out, h, w] = np.sum(window * W[c_out, :, :, :]) + b[c_out]
    return Y

# Exemple
N, C_in, H_in, W_in = 1, 3, 5, 5
C_out, k_h, k_w = 2, 3, 3
X = np.random.randn(N, C_in, H_in, W_in)
W = np.random.randn(C_out, C_in, k_h, k_w)
b = np.random.randn(C_out)

Y = conv2d_forward(X, W, b, stride=1, padding=1)
print("Forme de sortie de la convolution:", Y.shape)
# Forme de sortie: (1, 2, 5, 5)
```

#### Implémentation en C++

Cette implémentation utilise des vecteurs 4D pour représenter les tenseurs. C'est une approche simple mais peu efficace en mémoire et en vitesse par rapport à un stockage contigu (plat).

```cpp
#include <iostream>
#include <vector>
#include <stdexcept>

// Tenseur 4D: N x C x H x W
using Tensor4D = std::vector<std::vector<std::vector<std::vector<float>>>>;
using Vector = std::vector<float>;

Tensor4D conv2d_forward_cpp(const Tensor4D& X, const Tensor4D& W, const Vector& b, int stride, int padding) {
    if (X.empty() || W.empty()) throw std::invalid_argument("Input tensors cannot be empty.");

    size_t N = X.size();
    size_t C_in = X[0].size();
    size_t H_in = X[0][0].size();
    size_t W_in = X[0][0][0].size();

    size_t C_out = W.size();
    size_t k_h = W[0][0].size();
    size_t k_w = W[0][0][0].size();

    size_t H_out = (H_in + 2 * padding - k_h) / stride + 1;
    size_t W_out = (W_in + 2 * padding - k_w) / stride + 1;

    Tensor4D Y(N, std::vector<std::vector<std::vector<float>>>(C_out, std::vector<std::vector<float>>(H_out, std::vector<float>(W_out, 0.0f))));

    for (size_t n = 0; n < N; ++n) {
        for (size_t c_out = 0; c_out < C_out; ++c_out) {
            for (size_t h = 0; h < H_out; ++h) {
                for (size_t w = 0; w < W_out; ++w) {
                    float sum = 0.0f;
                    for (size_t c_in = 0; c_in < C_in; ++c_in) {
                        for (size_t kh = 0; kh < k_h; ++kh) {
                            for (size_t kw = 0; kw < k_w; ++kw) {
                                int h_in_idx = h * stride + kh - padding;
                                int w_in_idx = w * stride + kw - padding;
                                if (h_in_idx >= 0 && h_in_idx < (int)H_in && w_in_idx >= 0 && w_in_idx < (int)W_in) {
                                    sum += X[n][c_in][h_in_idx][w_in_idx] * W[c_out][c_in][kh][kw];
                                }
                            }
                        }
                    }
                    Y[n][c_out][h][w] = sum + b[c_out];
                }
            }
        }
    }
    return Y;
}
```

#### Utilisation avec PyTorch

```python
import torch
import torch.nn as nn

N, C_in, H_in, W_in = 1, 3, 5, 5
C_out, k_h, k_w = 2, 3, 3

conv_layer = nn.Conv2d(in_channels=C_in, out_channels=C_out, kernel_size=(k_h, k_w), stride=1, padding=1)
X_torch = torch.randn(N, C_in, H_in, W_in)
Y_torch = conv_layer(X_torch)

print("Forme de sortie de la convolution PyTorch:", Y_torch.shape)
# Forme de sortie: torch.Size([1, 2, 5, 5])
```

#### 2.2.1 Convolution Groupée (Grouped Convolution)

La convolution groupée est une modification simple mais puissante de la convolution standard. Au lieu que chaque filtre de sortie soit connecté à tous les canaux d'entrée, les canaux sont divisés en $G$ groupes. La convolution est ensuite effectuée indépendamment au sein de chaque groupe. Par exemple, avec 2 groupes, les $C_{out}/2$ premiers filtres ne "voient" que les $C_{in}/2$ premiers canaux d'entrée, et les $C_{out}/2$ filtres restants ne voient que les $C_{in}/2$ canaux d'entrée restants. Le principal avantage est une réduction du nombre de paramètres et de calculs d'un facteur $G$.

Cette technique est un exemple parfait d'innovation née d'une contrainte matérielle. Elle a été introduite dans l'architecture AlexNet en 2012, non pas pour une raison théorique, mais comme un moyen pragmatique de répartir l'entraînement du modèle sur deux GPU, chacun ayant une mémoire limitée. Les auteurs ont par la suite observé un effet secondaire fascinant : les filtres de chaque groupe semblaient se spécialiser, un groupe apprenant des caractéristiques de couleur et l'autre des caractéristiques achromatiques (noir et blanc). Cela a démontré que forcer une "sparsité structurée" dans les connexions pouvait non seulement être efficace, mais aussi bénéfique pour l'apprentissage, agissant comme une forme de régularisation qui encourage la spécialisation des filtres.

Pour l'implémenter, il suffit de modifier les boucles de la convolution standard. Le nombre de canaux d'entrée et de sortie par groupe devient $C_{in}/G$ et $C_{out}/G$.

#### 2.2.2 Convolution à Séparabilité de Profondeur (Depthwise Separable Convolution)

Il s'agit d'un cas extrême de convolution groupée où le nombre de groupes $G$ est égal au nombre de canaux d'entrée $C_{in}$. L'opération est factorisée en deux étapes distinctes :

1. **Convolution de Profondeur (Depthwise Convolution)** : Une convolution spatiale (ex: 3×3) est appliquée indépendamment à chaque canal d'entrée. Cette étape filtre les informations spatiales mais ne combine pas les informations entre les canaux. Elle est extrêmement légère en calculs.

2. **Convolution Ponctuelle (Pointwise Convolution)** : Une convolution 1×1 est ensuite utilisée pour combiner linéairement les sorties de la convolution de profondeur. C'est cette étape qui mélange les informations entre les canaux et crée de nouvelles caractéristiques. Dans les architectures comme MobileNet, c'est là que se concentre la majorité des calculs.

Cette factorisation réduit considérablement le nombre de paramètres et d'opérations. Cependant, une analyse plus fine révèle un compromis subtil. Pour compenser la perte de capacité expressive due à la séparation des canaux, les architectures comme MobileNetV2 introduisent un "ratio d'expansion" élevé (par exemple, 6×). Cela signifie que le nombre de canaux est d'abord massivement augmenté par une couche 1×1, puis filtré par la convolution de profondeur, avant d'être à nouveau réduit.

La conséquence est que, bien que le nombre de paramètres et de FLOPs soit faible, la taille des cartes d'activation intermédiaires peut devenir très grande. Le "pic d'activation" (la quantité maximale de mémoire nécessaire pour calculer une couche) devient le nouveau goulot d'étranglement, et non plus la taille du modèle. Or, le déplacement de ces grandes quantités de données entre la mémoire principale (DRAM) et les unités de calcul du processeur est une opération extrêmement coûteuse en énergie, souvent de plusieurs ordres de grandeur plus que le calcul lui-même. Ainsi, un modèle "efficace" en termes de FLOPs peut s'avérer lent et énergivore en pratique sur un appareil mobile. Cela met en évidence la nécessité d'une vision holistique de l'efficacité, au-delà des simples métriques de calcul.

### 2.3 La Couche de Pooling (Max & Average)

#### Théorie

Une couche de pooling est une opération de sous-échantillonnage non linéaire qui réduit la dimension spatiale (hauteur et largeur) des cartes de caractéristiques. Elle n'a aucun paramètre apprenable, ce qui la rend très légère. Ses principaux objectifs sont de :

- Réduire la quantité de calculs et de paramètres dans les couches suivantes.
- Créer une forme d'invariance aux petites translations dans l'image d'entrée.

Les deux types les plus courants sont :

- **Max Pooling** : Sélectionne la valeur maximale dans une fenêtre. Cela a pour effet de préserver les caractéristiques les plus saillantes et les plus fortes.
- **Average Pooling** : Calcule la moyenne des valeurs dans une fenêtre, ce qui donne une représentation plus lissée et moins sensible au bruit.

Le fait de n'avoir aucun paramètre apprenable est à la fois un avantage (pas de surcoût) et un inconvénient (perte d'information, moins expressif qu'une convolution avec un pas de 2).

#### Pseudo-code (Max Pooling)

```
FONCTION MaxPool(Input, KernelSize, Stride):
  // Input: H_in x W_in x C
  // KernelSize: k_h x k_w

  // Calculer les dimensions de sortie
  H_out = floor((H_in - k_h) / Stride) + 1
  W_out = floor((W_in - k_w) / Stride) + 1

  // Initialiser la sortie
  Output = nouvelle Matrice(H_out, W_out, C)

  // Itérer sur chaque canal
  POUR c DE 0 A C-1:
    // Itérer sur les positions de la sortie
    POUR y DE 0 A H_out-1:
      POUR x DE 0 A W_out-1:
        // Définir la fenêtre dans l'entrée
        h_start = y * Stride
        w_start = x * Stride
        window = Input[h_start : h_start+k_h, w_start : w_start+k_w, c]
        
        // Trouver la valeur maximale dans la fenêtre
        Output[y, x, c] = max(window)
      FIN POUR
    FIN POUR
  FIN POUR

  RETOURNER Output
```

#### Implémentation en Python (NumPy)

```python
import numpy as np

def maxpool2d_forward(X, kernel_size, stride):
    """
    Calcule la passe avant d'une couche de max pooling 2D.
    X: Entrée de forme (N, C, H_in, W_in)
    kernel_size: Taille du noyau (int ou tuple)
    stride: Pas (int)
    """
    N, C, H_in, W_in = X.shape
    k_h, k_w = (kernel_size, kernel_size) if isinstance(kernel_size, int) else kernel_size

    H_out = (H_in - k_h) // stride + 1
    W_out = (W_in - k_w) // stride + 1

    Y = np.zeros((N, C, H_out, W_out))

    for n in range(N):
        for c in range(C):
            for h in range(H_out):
                for w in range(W_out):
                    h_start = h * stride
                    h_end = h_start + k_h
                    w_start = w * stride
                    w_end = w_start + k_w
                    
                    window = X[n, c, h_start:h_end, w_start:w_end]
                    Y[n, c, h, w] = np.max(window)
    return Y

# Exemple
N, C, H_in, W_in = 1, 1, 4, 4
X = np.array([[[[2, 9, 3, 8], [0, 1, 5, 5], [5, 7, 2, 6], [8, 8, 3, 6]]]])
Y = maxpool2d_forward(X, kernel_size=2, stride=2)
print("Sortie du Max Pooling:\n", Y)
# [[[[9. 8.]
#    [8. 6.]]]]
```

#### Implémentation en C++

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <limits>

using Tensor4D = std::vector<std::vector<std::vector<std::vector<float>>>>;

Tensor4D maxpool2d_forward_cpp(const Tensor4D& X, int kernel_size, int stride) {
    size_t N = X.size();
    size_t C = X[0].size();
    size_t H_in = X[0][0].size();
    size_t W_in = X[0][0][0].size();

    size_t k_h = kernel_size, k_w = kernel_size;

    size_t H_out = (H_in - k_h) / stride + 1;
    size_t W_out = (W_in - k_w) / stride + 1;

    Tensor4D Y(N, std::vector<std::vector<std::vector<float>>>(C, std::vector<std::vector<float>>(H_out, std::vector<float>(W_out))));

    for (size_t n = 0; n < N; ++n) {
        for (size_t c = 0; c < C; ++c) {
            for (size_t h = 0; h < H_out; ++h) {
                for (size_t w = 0; w < W_out; ++w) {
                    float max_val = -std::numeric_limits<float>::infinity();
                    for (size_t kh = 0; kh < k_h; ++kh) {
                        for (size_t kw = 0; kw < k_w; ++kw) {
                            max_val = std::max(max_val, X[n][c][h * stride + kh][w * stride + kw]);
                        }
                    }
                    Y[n][c][h][w] = max_val;
                }
            }
        }
    }
    return Y;
}
```

#### Utilisation avec PyTorch

```python
import torch
import torch.nn as nn

pool_layer = nn.MaxPool2d(kernel_size=2, stride=2)
X_torch = torch.tensor([[[[2., 9., 3., 8.], [0., 1., 5., 5.], [5., 7., 2., 6.], [8., 8., 3., 6.]]]])
Y_torch = pool_layer(X_torch)

print("Sortie du Max Pooling PyTorch:\n", Y_torch)
# tensor([[[[9., 8.],
#           [8., 6.]]]])
```

### 2.4 Les Couches de Normalisation

#### Théorie

Les couches de normalisation sont devenues un élément indispensable pour l'entraînement stable et rapide des réseaux de neurones profonds. Leur objectif principal est de lutter contre le phénomène de "décalage de covariable interne" (internal covariate shift). Ce terme décrit le changement de distribution des activations d'une couche au cours de l'entraînement, dû à la mise à jour des poids des couches précédentes. En normalisant les activations de chaque couche pour qu'elles aient une moyenne de 0 et une variance de 1, on stabilise le processus d'apprentissage.

Cependant, forcer les activations à avoir une distribution fixe pourrait limiter la capacité de représentation du réseau. Pour contrer cela, la normalisation est suivie d'une transformation affine où deux paramètres, gamma ($\gamma$) pour la mise à l'échelle et beta ($\beta$) pour le décalage, sont appris par le réseau. Cela permet au réseau de "décider" de la distribution optimale pour les activations de chaque couche.

Il existe plusieurs variantes de normalisation, qui ne diffèrent que par les dimensions sur lesquelles la moyenne et la variance sont calculées :

- **Batch Normalization** : Normalise sur les dimensions du batch, de la hauteur et de la largeur pour chaque canal. Très efficace mais dépend de la taille du batch.
- **Layer Normalization** : Normalise sur les dimensions des canaux, de la hauteur et de la largeur pour chaque échantillon du batch. Indépendant de la taille du batch, très utilisé dans les Transformers.
- **Instance Normalization** : Normalise sur les dimensions de la hauteur et de la largeur pour chaque canal et chaque échantillon. Souvent utilisé en transfert de style.
- **Group Normalization** : Un compromis entre Batch et Layer Norm, normalisant sur des groupes de canaux.

#### Formules (Batch Normalization)

Pour un mini-batch $B = \{x_1, \ldots, x_m\}$, la normalisation par lot se déroule en quatre étapes :

1. **Calcul de la moyenne du mini-batch** : $\mu_B = \frac{1}{m} \sum_{i=1}^{m} x_i$

2. **Calcul de la variance du mini-batch** : $\sigma_B^2 = \frac{1}{m} \sum_{i=1}^{m} (x_i - \mu_B)^2$

3. **Normalisation** : $\hat{x}_i = \frac{x_i - \mu_B}{\sqrt{\sigma_B^2 + \epsilon}}$ (où $\epsilon$ est une petite constante pour la stabilité numérique)

4. **Mise à l'échelle et décalage** : $y_i = \gamma \hat{x}_i + \beta$

#### Implémentation (Batch Normalization)

```python
import numpy as np

def batchnorm_forward(X, gamma, beta, epsilon=1e-5):
    """
    Passe avant pour la normalisation par lot.
    X: Entrée de forme (N, C, H, W)
    gamma: Paramètre d'échelle de forme (C,)
    beta: Paramètre de décalage de forme (C,)
    """
    N, C, H, W = X.shape
    # Pour la normalisation par lot, la moyenne/variance est calculée
    # sur les dimensions N, H, W pour chaque canal C.
    mu = np.mean(X, axis=(0, 2, 3), keepdims=True)
    var = np.var(X, axis=(0, 2, 3), keepdims=True)

    X_norm = (X - mu) / np.sqrt(var + epsilon)

    # Reshape gamma et beta pour le broadcasting
    gamma_reshaped = gamma.reshape(1, C, 1, 1)
    beta_reshaped = beta.reshape(1, C, 1, 1)

    out = gamma_reshaped * X_norm + beta_reshaped
    return out

# Exemple
N, C, H, W = 10, 3, 5, 5
X = np.random.randn(N, C, H, W)
gamma = np.ones(C)
beta = np.zeros(C)

Y = batchnorm_forward(X, gamma, beta)
print("Forme de sortie de la Batch Norm:", Y.shape)
print("Moyenne par canal (après):", np.mean(Y, axis=(0, 2, 3))) # Proche de 0 (beta)
print("Variance par canal (après):", np.var(Y, axis=(0, 2, 3))) # Proche de 1 (gamma^2)
```

### 2.5 Les Fonctions d'Activation

#### Théorie

Les fonctions d'activation introduisent la non-linéarité dans le réseau. Sans elles, un réseau de neurones profond, quelle que soit sa complexité, ne serait qu'une succession de transformations linéaires, et donc équivalent à une unique couche linéaire. La fonction d'activation la plus populaire est ReLU (Rectified Linear Unit).

**ReLU** : $f(x) = \max(0, x)$. Elle est simple, rapide à calculer et évite les problèmes de "vanishing gradients" qui affectaient les fonctions plus anciennes comme la sigmoïde ou la tangente hyperbolique. Son principal inconvénient est le problème du "dying ReLU", où un neurone peut se retrouver bloqué à une sortie de 0 et ne plus jamais s'activer.

**Variantes** : Pour pallier ce problème, des alternatives ont été proposées, comme Leaky ReLU ($f(x) = \max(\alpha x, x)$ avec $\alpha$ petit), Swish, ou HardSwish, qui ont un gradient non nul partout. Le choix de la fonction d'activation peut aussi dépendre de contraintes matérielles ; ReLU est très "hardware-friendly", tandis que d'autres peuvent être plus complexes à implémenter ou à quantifier.

Les implémentations sont triviales, par exemple `np.maximum(0, x)` pour ReLU en NumPy.

### 2.6 Introduction à l'Attention : Le Scaled Dot-Product Attention

#### Théorie

Le mécanisme d'attention, et en particulier le Scaled Dot-Product Attention, est le composant central de l'architecture Transformer, qui a révolutionné le traitement du langage naturel et s'étend maintenant à la vision. L'attention permet au modèle de pondérer dynamiquement l'importance des différentes parties de la séquence d'entrée lors de la production d'une sortie. Il fonctionne sur la base de trois vecteurs dérivés de l'entrée :

- **Requête (Query, Q)** : Représente l'élément actuel qui cherche à obtenir des informations.
- **Clé (Key, K)** : Représente une "étiquette" pour chaque élément de la séquence, que la requête peut interroger.
- **Valeur (Value, V)** : Représente le contenu de chaque élément de la séquence.

Le processus consiste à comparer la requête (Q) à toutes les clés (K) pour obtenir des scores de similarité. Ces scores sont ensuite convertis en poids (via une fonction softmax) qui sont utilisés pour calculer une somme pondérée des valeurs (V).

#### Formule

La formule du Scaled Dot-Product Attention est la suivante :

$$\text{Attention}(Q, K, V) = \text{softmax}\left(\frac{QK^T}{\sqrt{d_k}}\right) V$$

Le facteur de mise à l'échelle $\frac{1}{\sqrt{d_k}}$ est crucial. Sans lui, pour de grandes valeurs de $d_k$ (la dimension des clés), les produits scalaires $QK^T$ pourraient devenir très grands, poussant la fonction softmax dans des régions où son gradient est proche de zéro, ce qui entraverait l'apprentissage.

#### Pseudo-code

```
FONCTION ScaledDotProductAttention(Q, K, V):
  d_k = nombre de colonnes de K

  // 1. Calculer les scores de similarité
  Scores = MatriceMul(Q, Transpose(K))

  // 2. Mettre à l'échelle
  ScaledScores = Scores / sqrt(d_k)

  // (Optionnel) Appliquer un masque pour ignorer certains éléments

  // 3. Calculer les poids d'attention via softmax
  Weights = Softmax(ScaledScores)

  // 4. Calculer la sortie comme une somme pondérée des valeurs
  Output = MatriceMul(Weights, V)

  RETOURNER Output
```

#### Implémentation en Python (NumPy)

```python
import numpy as np

def softmax(x):
    e_x = np.exp(x - np.max(x, axis=-1, keepdims=True))
    return e_x / np.sum(e_x, axis=-1, keepdims=True)

def scaled_dot_product_attention(Q, K, V):
    """
    Q: Requêtes (N, seq_len_q, d_k)
    K: Clés (N, seq_len_k, d_k)
    V: Valeurs (N, seq_len_k, d_v)
    """
    d_k = K.shape[-1]
    
    # MatMul(Q, K.T)
    scores = Q @ K.transpose(0, 2, 1)
    
    # Scale
    scaled_scores = scores / np.sqrt(d_k)
    
    # Softmax
    weights = softmax(scaled_scores)
    
    # MatMul(weights, V)
    output = weights @ V
    
    return output, weights

# Exemple
N, seq_len_q, seq_len_k, d_k, d_v = 1, 10, 12, 64, 64
Q = np.random.randn(N, seq_len_q, d_k)
K = np.random.randn(N, seq_len_k, d_k)
V = np.random.randn(N, seq_len_k, d_v)

output, weights = scaled_dot_product_attention(Q, K, V)
print("Forme de la sortie d'attention:", output.shape) # (1, 10, 64)
print("Forme des poids d'attention:", weights.shape) # (1, 10, 12)
```

#### Utilisation avec PyTorch

PyTorch offre une implémentation fusionnée et hautement optimisée.

```python
import torch.nn.functional as F

Q_torch = torch.from_numpy(Q)
K_torch = torch.from_numpy(K)
V_torch = torch.from_numpy(V)

output_torch = F.scaled_dot_product_attention(Q_torch, K_torch, V_torch)
print("Forme de sortie PyTorch:", output_torch.shape)
# torch.Size([1, 10, 64])
```

## Module 3 : Architectures de Réseaux de Neurones Convolutifs Canoniques

Cette section retrace l'évolution des architectures de CNN, en montrant comment chaque nouvelle conception a répondu aux défis et aux limitations de la précédente.

### 3.1 AlexNet (Krizhevsky et al., 2012)

AlexNet est le modèle qui a véritablement lancé la révolution de l'apprentissage profond en remportant le concours ImageNet (ILSVRC) en 2012 avec une marge d'erreur spectaculairement plus faible que ses concurrents. Son architecture, bien que simple selon les normes actuelles, a introduit plusieurs innovations qui sont devenues des standards :

- **Utilisation de ReLU** : L'activation ReLU a permis d'entraîner des réseaux beaucoup plus profonds et plus rapidement que les fonctions saturantes comme la sigmoïde.
- **Entraînement sur GPU** : Le modèle a été explicitement conçu pour tirer parti de la puissance de calcul parallèle des GPU, ce qui a rendu l'entraînement sur 1.2 million d'images réalisable. C'est cette contrainte qui a mené à l'utilisation de la convolution groupée.
- **Régularisation massive** : Avec 60 millions de paramètres, le surapprentissage était un risque majeur. AlexNet a popularisé l'utilisation du dropout et de l'augmentation de données (translations, réflexions, altérations de couleurs) pour le combattre.

Son architecture se compose de 5 couches de convolution, utilisant des noyaux de grande taille (11×11 et 5×5) au début, suivies de 3 couches entièrement connectées.

### 3.2 ResNet (He et al., 2015)

Avant ResNet, l'ajout de couches supplémentaires à un réseau profond finissait paradoxalement par dégrader sa performance, un problème connu sous le nom de dégradation. ResNet a résolu ce problème de manière élégante avec son innovation principale : les connexions résiduelles (skip connections).

Une connexion résiduelle permet à l'entrée d'une ou plusieurs couches de "court-circuiter" ces couches et d'être ajoutée directement à leur sortie. La formule du bloc est $H(x) = F(x) + x$, où le réseau n'apprend que la fonction résiduelle $F(x)$. Cela facilite grandement le flux du gradient lors de la rétropropagation, permettant l'entraînement de réseaux extraordinairement profonds (plus de 150 couches dans le papier original, et jusqu'à plus de 1000 dans des travaux ultérieurs).

ResNet a également popularisé l'utilisation de blocs goulot (bottleneck blocks) pour l'efficacité : une convolution 1×1 réduit le nombre de canaux, une convolution 3×3 (coûteuse) est appliquée sur cette représentation de plus petite dimension, puis une autre convolution 1×1 restaure le nombre de canaux original. Cela réduit considérablement le coût de calcul.

### 3.3 MobileNet (Howard et al., 2017)

MobileNet a été la première architecture conçue dès le départ pour les contraintes des applications mobiles et embarquées : faible latence, faible consommation d'énergie et petite taille de modèle. Son innovation clé est l'utilisation systématique des convolutions à séparabilité de profondeur (détaillées au Module 2.2.2).

La version améliorée, MobileNetV2, a introduit deux concepts importants pour améliorer la performance :

1. **Goulots inversés (Inverted Bottlenecks)** : Contrairement aux blocs goulot de ResNet qui compressent les canaux, les blocs de MobileNetV2 les étendent d'abord avec une convolution 1×1 (le "ratio d'expansion"), appliquent la convolution de profondeur 3×3 sur cette représentation plus riche, puis compressent à nouveau avec une convolution 1×1.

2. **Connexions résiduelles linéaires** : La dernière convolution 1×1 de re-compression dans le bloc n'utilise pas d'activation ReLU. Les auteurs ont constaté que l'utilisation de ReLU dans des espaces de faible dimension (après compression) détruisait des informations importantes.

MobileNet a également introduit les hyperparamètres d'efficacité : un multiplicateur de largeur ($\alpha$) pour affiner le nombre de canaux, et un multiplicateur de résolution ($\rho$) pour ajuster la taille de l'image d'entrée, permettant de générer facilement une famille de modèles avec différents compromis précision/efficacité.

Le tableau suivant synthétise l'évolution de ces architectures clés.

| Architecture | Année | Innovation Principale | Impact sur l'Efficacité |
|--------------|-------|----------------------|-------------------------|
| AlexNet | 2012 | ReLU, Dropout, Entraînement GPU, Conv. Groupée | A rendu l'entraînement de grands réseaux possible. |
| VGG | 2014 | Homogénéité (noyaux 3×3), Profondeur accrue | Très lourd en paramètres et calculs. |
| ResNet | 2015 | Connexions Résiduelles, Blocs Goulot | A permis des réseaux beaucoup plus profonds sans dégradation. |
| MobileNet | 2017 | Conv. à Séparabilité de Profondeur | Réduction drastique des paramètres et MACs pour le mobile. |
| Transformer | 2017 | Mécanisme d'Attention exclusif | Très parallélisable, mais coût quadratique avec la séquence. |

## Module 4 : Métriques d'Efficacité et Analyse de Performance

Pour concevoir des modèles efficaces, il est impératif de pouvoir les mesurer. Ce module clarifie les métriques essentielles.

### 4.1 Latence vs. Débit (Throughput)

Ces deux termes sont souvent utilisés à tort comme synonymes, mais ils mesurent des aspects différents de la performance.

**Latence** : C'est le temps nécessaire pour accomplir une seule tâche, par exemple, le temps pour traiter une seule image. Elle est généralement mesurée en millisecondes (ms). Une faible latence est cruciale pour les applications temps réel et interactives.

**Débit (Throughput)** : C'est le nombre de tâches accomplies par unité de temps, par exemple, le nombre d'images traitées par seconde (images/sec). Un débit élevé est important pour le traitement de masse de données.

La relation entre les deux n'est pas directe. Considérons un système qui prend 100 ms pour traiter une image (latence = 100 ms). Son débit est de 1000/100 = 10 images/sec. Si nous utilisons quatre de ces systèmes en parallèle, la latence pour traiter une image reste de 100 ms, mais le débit global est maintenant de 40 images/sec. Le traitement par lots (batching) est une autre technique qui augmente le débit en traitant plusieurs entrées simultanément, mais n'améliore généralement pas la latence de la première instance. L'optimisation de la latence est souvent plus difficile car elle nécessite d'optimiser le chemin critique d'une seule inférence.

### 4.2 Efficacité Énergétique : Le Coût du Mouvement de Données

Un aspect souvent négligé de l'efficacité est la consommation d'énergie. Une analyse fondamentale du coût des opérations sur le matériel révèle un fait surprenant : un accès à la mémoire principale (DRAM) est environ deux ordres de grandeur (100×) plus coûteux en énergie qu'une opération arithmétique simple (comme une addition ou une multiplication sur 8 bits).

Cette observation a des implications profondes. Pour l'IA "verte" ou pour les appareils fonctionnant sur batterie, la stratégie d'optimisation la plus efficace n'est pas de minimiser le nombre d'opérations de calcul, mais de minimiser les mouvements de données. Cela relie directement à notre discussion sur MobileNet : un modèle peut avoir peu de FLOPs, mais s'il nécessite de manipuler d'énormes cartes d'activation, son coût énergétique sera très élevé.

### 4.3 Métriques Liées à la Mémoire

#### Nombre de Paramètres et Taille du Modèle

Le nombre de paramètres est le nombre total de poids et de biais apprenables dans un réseau. Il est calculé en additionnant les paramètres de chaque couche.

La taille du modèle est l'espace de stockage réel occupé par ces paramètres. Elle se calcule comme suit :

$$\text{Taille (octets)} = \frac{\text{Nombre de paramètres} \times \text{Précision (bits)}}{8}$$

Par exemple, un modèle de 61 millions de paramètres stockés en 32 bits (4 octets) occupera $61 \times 10^6 \times 4 \approx 244$ Mo. Le même modèle quantifié en 8 bits (1 octet) n'occupera que 61 Mo.

#### Taille Totale et Pic des Activations

- **La taille totale des activations** est la somme des tailles de toutes les cartes de caractéristiques générées par le réseau.
- **Le pic d'activation** est la quantité maximale de mémoire requise à un instant T pour calculer une couche donnée. Il est approximé par : $\max_{\text{couche}}(\text{taille}(\text{activation}_{\text{entrée}}) + \text{taille}(\text{activation}_{\text{sortie}}))$. C'est cette valeur qui constitue le véritable goulot d'étranglement de la mémoire sur les appareils embarqués, car le matériel doit disposer de suffisamment de RAM pour gérer ce pic. Les CNN ont souvent un profil d'activation en "U" : élevé au début (grandes résolutions), bas au milieu, puis remontant légèrement à la fin (beaucoup de canaux).

### 4.4 Métriques Liées au Calcul

La terminologie autour du calcul est une source de confusion fréquente. Le tableau suivant fournit des définitions claires et non ambiguës.

| Métrique | Définition | Relation | Exemple |
|----------|------------|----------|---------|
| MAC | Multiply-Accumulate. Une multiplication suivie d'une addition. | Unité de base du calcul neuronal. | `a = a + (b * c)` |
| FLOPs | Floating Point Operations. Opérations sur des nombres à virgule flottante. | 1 MAC ≈ 2 FLOPs (une multiplication, une addition) | `a * b` est 1 FLOP. `a + b` est 1 FLOP. |
| FLOPS | FLOPs per Second. Mesure de performance matérielle (le 'S' est majuscule) | - | Un GPU à 1 TFLOPS peut effectuer $10^{12}$ FLOPs par seconde. |
| OPs | Operations. Plus général, inclut les entiers, les opérations sur bits, etc. | 1 MAC = 2 OPs | Une addition 4-bit est 1 OP. |
| OPS | OPs per Second. Mesure de performance matérielle générale. | - | - |

Le nombre de MACs pour une couche de convolution standard est calculé en multipliant le nombre d'opérations par pixel de sortie par le nombre total de pixels de sortie :

$$\text{MACs} = H_{out} \times W_{out} \times C_{out} \times C_{in} \times k_h \times k_w$$

## Conclusion et Perspectives

Cette leçon a établi les fondations nécessaires pour comprendre et concevoir des réseaux de neurones efficaces. Nous avons vu que l'efficacité n'est pas une métrique unique, mais un espace de compromis complexe entre la précision, la latence, le débit, la consommation de mémoire et la consommation d'énergie.

Nous avons disséqué les blocs de construction fondamentaux, des couches linéaires aux mécanismes d'attention, en fournissant non seulement la théorie mais aussi des implémentations concrètes qui démystifient leur fonctionnement interne. L'analyse des architectures canoniques a révélé un récit d'innovation, où chaque génération de modèles a répondu aux limitations de la précédente, souvent sous l'influence directe des contraintes matérielles. Enfin, la clarification des métriques d'efficacité nous a dotés des outils nécessaires pour analyser et comparer les modèles de manière rigoureuse.

La compréhension de ces concepts est le prérequis indispensable pour aborder les techniques d'optimisation plus avancées qui feront l'objet des prochaines leçons, telles que la quantification, qui réduit la précision des poids et des activations, l'élagage (pruning), qui supprime les connexions redondantes, et la recherche d'architecture neuronale (NAS), qui automatise la découverte de modèles efficaces.
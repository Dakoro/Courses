# Fondamentaux des Réseaux de Neurones Efficaces

## Introduction : Le Besoin d'Efficacité en Intelligence Artificielle

L'intelligence artificielle (IA) moderne a atteint des capacités qui étaient autrefois du domaine de la science-fiction. Des modèles de langage étendus (LLM) capables de générer du texte cohérent aux systèmes d'IA générative créant des images photoréalistes, en passant par les technologies de conduite autonome qui redéfinissent la mobilité, les applications de l'IA sont de plus en plus intégrées dans notre quotidien. Cependant, cette sophistication a un coût : une demande de ressources de calcul et de mémoire qui croît à un rythme effréné.

### Le Décalage Exponentiel entre l'Offre et la Demande de Calcul

L'un des principaux défis de l'IA contemporaine réside dans le décalage croissant entre la demande de calcul des modèles d'apprentissage profond et l'offre de puissance de calcul disponible. Historiquement, l'industrie du matériel a progressé au rythme de la loi de Moore, qui postule un doublement du nombre de transistors sur une puce tous les deux ans environ. Cette progression, bien qu'impressionnante, est linéaire sur une échelle logarithmique. En revanche, la complexité et la taille des modèles d'apprentissage profond de pointe ont augmenté de manière bien plus agressive, avec une demande de calcul qui a été observée comme quadruplant, voire plus, sur la même période de deux ans. 

Ce fossé exponentiel entre ce que nous pouvons calculer et ce que nous voulons calculer est le principal moteur du domaine de l'IA efficace. Sans nouvelles techniques pour optimiser nos modèles, nous risquons d'atteindre un plateau où seules les organisations les plus riches en ressources pourront développer et déployer des modèles d'IA de pointe.

### Études de Cas Inspirantes : L'Impact de l'Optimisation

L'importance de l'IA efficace n'est pas seulement théorique ; elle a des implications pratiques profondes qui permettent des applications révolutionnaires. Deux exemples récents illustrent parfaitement cet impact.

**Premièrement**, EfficientViT, un modèle de transformeur de vision optimisé, a été conçu pour accélérer le traitement d'images à haute résolution, une tâche notoirement gourmande en calcul et en mémoire. Sur un GPU de périphérie (edge GPU) comme le NVIDIA Jetson AGX Orin — un appareil de la taille de la paume de la main — EfficientViT peut traiter des scènes de rue en temps réel à 21 images par seconde. En comparaison, un modèle de base non optimisé n'atteint que 1.6 images par seconde, rendant l'application en temps réel impossible. Cette accélération de plus d'un ordre de grandeur ouvre la voie à des systèmes avancés d'aide à la conduite (ADAS) et à des applications mobiles plus intelligentes.

**Deuxièmement**, le Segment Anything Model (SAM), développé par Meta AI, a démontré qu'il est possible d'optimiser radicalement l'inférence sans sacrifier la précision. En appliquant des techniques d'accélération, les chercheurs ont pu faire passer la vitesse de traitement de 12 images par seconde à un impressionnant 840 images par seconde, tout en maintenant la qualité de la segmentation. Cela montre que l'efficacité n'est pas un compromis nécessaire avec la performance, mais plutôt une dimension d'ingénierie qui peut être optimisée.

Ces exemples soulignent un principe fondamental : l'IA efficace n'est pas une simple optimisation a posteriori. C'est un domaine de co-conception où l'architecture du modèle, l'algorithme d'apprentissage et la plateforme matérielle cible sont intrinsèquement liés. L'efficacité n'est pas une métrique absolue, mais relative à un contexte matériel et applicatif. Ce cours vous fournira les connaissances fondamentales pour naviguer dans ce paysage complexe, en commençant par les blocs de construction de base de tout réseau de neurones.

## Partie 1 : Les Blocs de Construction Fondamentaux des Réseaux de Neurones

Avant de pouvoir optimiser des systèmes complexes, il est impératif de maîtriser leurs composants fondamentaux. Cette section décompose les réseaux de neurones en leurs éléments constitutifs, en examinant leur fonction, leur formulation mathématique et leur implémentation pratique.

### 1.1 De la Biologie à l'Artificiel : Le Neurone et la Synapse

L'inspiration initiale des réseaux de neurones artificiels provient de la neurobiologie. Bien que l'analogie soit une simplification, elle fournit une terminologie et une intuition utiles. Dans le cerveau, un neurone reçoit des signaux d'autres neurones via des dendrites. Ces signaux sont pondérés par la force des connexions synaptiques. Si la somme des signaux pondérés dépasse un certain seuil, le neurone "s'active" et envoie un signal le long de son axone vers d'autres neurones.

Dans le monde artificiel, nous adoptons une terminologie équivalente qui sera utilisée tout au long de ce cours :

- **Poids (Weights) ou Synapses** : Ce sont les paramètres apprenables du réseau. Ils représentent la force de la connexion entre les neurones. Un modèle avec "175 milliards de paramètres" signifie qu'il possède 175 milliards de poids.

- **Activations, Neurones ou Caractéristiques (Features)** : Ce sont les valeurs numériques qui représentent les signaux transmis entre les couches du réseau. Elles sont le résultat des calculs effectués à chaque couche.

Deux concepts architecturaux clés sont la **profondeur** (depth) et la **largeur** (width) d'un réseau.

- La **profondeur** fait référence au nombre de couches empilées dans le réseau. Un "réseau de neurones profond" est un réseau qui a de nombreuses couches.
- La **largeur** fait référence à la taille des couches cachées, c'est-à-dire au nombre de neurones (ou de canaux) dans une couche donnée.

Le choix entre un réseau "large et peu profond" et un réseau "étroit et profond" n'est pas anodin et représente un compromis fondamental. Un réseau large et peu profond a tendance à s'exécuter plus rapidement sur les GPU. La raison est qu'il implique moins d'appels de noyau (kernel calls) — chaque couche correspondant généralement à un appel. Moins d'appels signifie moins de surcoût (overhead) et une meilleure utilisation des unités de calcul du GPU, surtout lorsque les matrices de poids sont grandes et peuvent saturer la capacité de calcul parallèle. Cependant, il est souvent observé que les réseaux profonds, bien que potentiellement plus lents, sont plus faciles à entraîner et peuvent atteindre une meilleure précision avec le même nombre de paramètres. Cet arbitrage entre la facilité de convergence et l'efficacité matérielle est au cœur de la conception d'architectures neuronales.

### 1.2 La Couche Linéaire (Entièrement Connectée)

La couche la plus fondamentale d'un réseau de neurones est la couche linéaire, également appelée couche entièrement connectée (fully connected). Son rôle est d'effectuer une transformation linéaire des données d'entrée.

#### Principe Mathématique

Chaque neurone de sortie est connecté à chaque neurone d'entrée. L'opération mathématique est une multiplication matrice-vecteur suivie de l'ajout d'un vecteur de biais. Pour un vecteur d'entrée $X$, un vecteur de sortie $Y$, une matrice de poids $W$ et un vecteur de biais $B$, l'opération est définie par :

$$Y = W \cdot X + B$$

Les dimensions des tenseurs sont cruciales. Pour une seule instance d'entrée :
- **Entrée** $X$ : un vecteur de taille $(C_{in}, 1)$
- **Poids** $W$ : une matrice de taille $(C_{out}, C_{in})$
- **Biais** $B$ : un vecteur de taille $(C_{out}, 1)$
- **Sortie** $Y$ : un vecteur de taille $(C_{out}, 1)$

Pour améliorer l'efficacité du calcul, en particulier sur le matériel parallèle comme les GPU, les entrées sont généralement traitées par lots (batches). Un lot est un ensemble de $N$ instances d'entrée traitées simultanément. Cela modifie les dimensions des tenseurs d'entrée et de sortie :
- **Entrée** $X$ : une matrice de taille $(N, C_{in})$
- **Poids** $W$ : reste de taille $(C_{out}, C_{in})$ (les poids sont indépendants du lot)
- **Biais** $B$ : reste de taille $(C_{out})$ (le biais est diffusé ou "broadcasted" à chaque instance du lot)
- **Sortie** $Y$ : une matrice de taille $(N, C_{out})$

#### Implémentations

##### Pseudo-code

```
Algorithme : CoucheLinéaire_Forward(X, W, B)
  Entrées :
    X: Matrice d'entrée de taille (N, C_in)
    W: Matrice de poids de taille (C_out, C_in)
    B: Vecteur de biais de taille (C_out)
  Sortie :
    Y: Matrice de sortie de taille (N, C_out)

  Initialiser Y avec des zéros

  Pour chaque instance n de 0 à N-1:
    Pour chaque neurone de sortie j de 0 à C_out-1:
      accumulation = 0
      Pour chaque neurone d'entrée i de 0 à C_in-1:
        accumulation += X[n, i] * W[j, i]
      Fin Pour
      Y[n, j] = accumulation + B[j]
    Fin Pour
  Fin Pour

  Retourner Y
```

##### Implémentation Python (avec NumPy)

En Python, les bibliothèques comme NumPy optimisent fortement les opérations matricielles, rendant l'implémentation concise et efficace.

```python
import numpy as np

def linear_layer_forward(X, W, B):
    """
    Calcule la passe avant d'une couche linéaire.

    Args:
        X (np.ndarray): Entrée de taille (N, C_in).
        W (np.ndarray): Poids de taille (C_out, C_in).
        B (np.ndarray): Biais de taille (C_out,).

    Returns:
        np.ndarray: Sortie de taille (N, C_out).
    """
    # La multiplication matricielle est W @ X.T, mais pour correspondre
    # aux conventions (N, C_in) @ (C_in, C_out), nous transposons W.
    # np.dot gère cela de manière plus intuitive.
    Y = np.dot(X, W.T) + B
    return Y

# Exemple d'utilisation
N, C_in, C_out = 64, 128, 256
X = np.random.randn(N, C_in)
W = np.random.randn(C_out, C_in)
B = np.random.randn(C_out)

Y = linear_layer_forward(X, W, B)
print(f"Shape de la sortie : {Y.shape}")  # Devrait être (64, 256)
```

##### Implémentation C++

Une implémentation en C++ de base, sans bibliothèques d'algèbre linéaire optimisées, met en évidence les boucles de calcul sous-jacentes.

```cpp
#include <iostream>
#include <vector>
#include <stdexcept>

// Utilisation de alias pour la clarté
using Matrix = std::vector<std::vector<float>>;
using Vector = std::vector<float>;

Matrix linear_layer_forward(const Matrix& X, const Matrix& W, const Vector& B) {
    if (X.empty() || W.empty() || B.empty()) {
        throw std::invalid_argument("Les entrées ne peuvent pas être vides.");
    }

    size_t N = X.size();
    size_t C_in = X[0].size();
    size_t C_out = W.size();

    if (W[0].size() != C_in || B.size() != C_out) {
        throw std::invalid_argument("Incompatibilité des dimensions.");
    }

    // Initialiser la matrice de sortie
    Matrix Y(N, Vector(C_out, 0.0f));

    for (size_t n = 0; n < N; ++n) {
        for (size_t j = 0; j < C_out; ++j) {
            float accumulation = 0.0f;
            for (size_t i = 0; i < C_in; ++i) {
                accumulation += X[n][i] * W[j][i];
            }
            Y[n][j] = accumulation + B[j];
        }
    }

    return Y;
}

int main() {
    // Exemple d'utilisation
    int N = 4, C_in = 3, C_out = 2;
    Matrix X = {{1, 2, 3}, {4, 5, 6}, {7, 8, 9}, {10, 11, 12}};
    Matrix W = {{0.1, 0.2, 0.3}, {0.4, 0.5, 0.6}};
    Vector B = {1.0, 2.0};

    try {
        Matrix Y = linear_layer_forward(X, W, B);
        std::cout << "Matrice de sortie Y:" << std::endl;
        for (const auto& row : Y) {
            for (float val : row) {
                std::cout << val << " ";
            }
            std::cout << std::endl;
        }
    } catch (const std::exception& e) {
        std::cerr << "Erreur: " << e.what() << std::endl;
    }

    return 0;
}
```

### 1.3 La Couche de Convolution (1D et 2D)

Alors que les couches linéaires traitent les entrées comme des vecteurs plats, les couches de convolution sont conçues pour exploiter la structure spatiale ou temporelle des données, comme les images ou les signaux audio. Elles le font grâce à deux principes puissants : la **localité** et le **partage de poids**.

- **Localité** : Un neurone de sortie dans une carte de caractéristiques convolutive n'est connecté qu'à une petite région locale de l'entrée (appelée champ réceptif). Cela est basé sur l'hypothèse que les pixels voisins sont fortement corrélés, tandis que les pixels éloignés le sont moins.

- **Partage de Poids** : Le même ensemble de poids, appelé noyau (kernel) ou filtre, est appliqué à travers toutes les positions spatiales de l'entrée. Cela réduit considérablement le nombre de paramètres par rapport à une couche entièrement connectée et rend le modèle invariant aux translations : une caractéristique apprise à un endroit peut être détectée n'importe où ailleurs dans l'image.

#### Hyperparamètres et Calcul de la Taille de Sortie

Trois hyperparamètres principaux contrôlent le comportement d'une couche de convolution :

1. **Taille du Noyau (Kernel Size, K)** : Les dimensions (hauteur et largeur) du filtre qui glisse sur l'entrée. Des tailles courantes sont 3×3 ou 5×5.

2. **Pas (Stride, S)** : Le nombre de pixels de décalage du noyau à chaque étape. Un stride de 1 déplace le noyau d'un pixel à la fois. Un stride de 2 le déplace de deux pixels, ce qui a pour effet de sous-échantillonner la sortie.

3. **Remplissage (Padding, P)** : Le nombre de pixels (généralement des zéros) ajoutés sur les bords de l'image d'entrée. Le padding est souvent utilisé pour contrôler la taille spatiale de la sortie. Une application courante est le "same padding", qui vise à ce que la sortie ait les mêmes dimensions spatiales que l'entrée.

La taille de la carte de caractéristiques de sortie $(W_{out}, H_{out})$ peut être calculée à partir de la taille de l'entrée $(W_{in}, H_{in})$ et de ces hyperparamètres à l'aide de la formule suivante :

$$W_{out} = \left\lfloor \frac{W_{in} - K_W + 2P}{S} \right\rfloor + 1$$

$$H_{out} = \left\lfloor \frac{H_{in} - K_H + 2P}{S} \right\rfloor + 1$$

Où $\lfloor \cdot \rfloor$ est l'opération de plancher (arrondi à l'entier inférieur).

#### Le Champ Réceptif (Receptive Field)

Le champ réceptif est un concept crucial : il définit la taille de la région dans l'image d'entrée qui influence une seule unité (pixel) dans une carte de caractéristiques de sortie. À mesure que l'on empile les couches de convolution, le champ réceptif des neurones des couches supérieures s'agrandit, leur permettant de "voir" et d'intégrer des informations sur des régions de plus en plus grandes de l'image originale, capturant ainsi des caractéristiques plus complexes et contextuelles.

La taille du champ réceptif $(r)$ d'une couche $l$ peut être calculée de manière récursive. Si une couche $l$ a un noyau de taille $k_l$ et un pas de $s_l$, et que son champ réceptif est $r_l$, alors le champ réceptif par rapport à la couche précédente $(l-1)$ est :

$$r_{l-1} = s_l \cdot r_l + (k_l - s_l)$$

Une formule plus directe pour calculer le champ réceptif total $(r_0)$ au niveau de l'image d'entrée pour une pile de $L$ couches est :

$$r_0 = \sum_{l=1}^{L} \left((k_l - 1) \prod_{i=1}^{l-1} s_i\right) + 1$$

Où $k_l$ est la taille du noyau de la couche $l$ et $s_i$ est le pas de la couche $i$.

#### Implémentations (Convolution 2D)

##### Pseudo-code

```
Algorithme : Convolution2D_Forward(Input, Kernel, Stride, Padding)
  Entrées :
    Input: Tenseur d'entrée de taille (H_in, W_in, C_in)
    Kernel: Tenseur de poids de taille (K_H, K_W, C_in, C_out)
    Stride: S, Padding: P
  Sortie :
    Output: Tenseur de sortie de taille (H_out, W_out, C_out)

  Appliquer le padding à Input pour obtenir InputPadded
  Calculer H_out, W_out en utilisant la formule
  Initialiser Output avec des zéros

  Pour chaque filtre de sortie c_out de 0 à C_out-1:
    Pour chaque position verticale y de 0 à H_out-1:
      Pour chaque position horizontale x de 0 à W_out-1:
        accumulation = 0
        // Région de l'entrée à convoluer
        y_start = y * S
        x_start = x * S
        region = InputPadded[y_start:y_start+K_H, x_start:x_start+K_W, :]

        // Produit scalaire multi-canaux
        Pour chaque canal d'entrée c_in de 0 à C_in-1:
          Pour chaque hauteur de noyau ky de 0 à K_H-1:
            Pour chaque largeur de noyau kx de 0 à K_W-1:
              accumulation += region[ky, kx, c_in] * Kernel[ky, kx, c_in, c_out]
            Fin Pour
          Fin Pour
        Fin Pour
        Output[y, x, c_out] = accumulation // + Biais[c_out] si applicable
      Fin Pour
    Fin Pour
  Fin Pour

  Retourner Output
```

##### Implémentation Python (avec NumPy)

Cette implémentation manuelle est à des fins pédagogiques pour illustrer le mécanisme. En pratique, les frameworks de deep learning utilisent des algorithmes beaucoup plus optimisés (comme im2col+GEMM).

```python
import numpy as np

def convolve2d_numpy(image, kernel, padding=0, strides=1):
    # La convolution en deep learning est en fait une corrélation croisée.
    # Pour une vraie convolution, il faudrait inverser le noyau.
    # kernel = np.flipud(np.fliplr(kernel))

    # Dimensions
    x_kern_shape = kernel.shape[0]
    y_kern_shape = kernel.shape[1]
    x_img_shape = image.shape[0]
    y_img_shape = image.shape[1]

    # Calcul de la taille de sortie
    x_output = int(((x_img_shape - x_kern_shape + 2 * padding) / strides) + 1)
    y_output = int(((y_img_shape - y_kern_shape + 2 * padding) / strides) + 1)
    output = np.zeros((x_output, y_output))

    # Appliquer le padding
    if padding != 0:
        image_padded = np.zeros((image.shape[0] + padding*2, image.shape[1] + padding*2))
        image_padded[padding:-padding, padding:-padding] = image
    else:
        image_padded = image

    # Itérer sur l'image avec les strides
    for y in range(y_output):
        for x in range(x_output):
            # Extraire la région d'intérêt
            region = image_padded[y*strides:y*strides+y_kern_shape, x*strides:x*strides+x_kern_shape]
            # Effectuer la multiplication élément par élément et la somme
            output[y, x] = np.sum(region * kernel)
            
    return output

# Exemple (image monocanal)
image = np.array([[1, 2, 3, 4], [5, 6, 7, 8], [9, 10, 11, 12], [13, 14, 15, 16]])
kernel = np.array([[1, 0], [0, 1]])
output = convolve2d_numpy(image, kernel, padding=0, strides=1)
print("Sortie de la convolution:\n", output)
```

##### Implémentation C++

Cette version C++ illustre la gestion manuelle des boucles et des accès mémoire.

```cpp
#include <iostream>
#include <vector>

using Matrix = std::vector<std::vector<float>>;

Matrix convolve2d_cpp(const Matrix& input, const Matrix& kernel, int stride, int padding) {
    int H_in = input.size();
    int W_in = input[0].size();
    int K_H = kernel.size();
    int K_W = kernel[0].size();

    // Appliquer le padding
    int H_padded = H_in + 2 * padding;
    int W_padded = W_in + 2 * padding;
    Matrix padded_input(H_padded, std::vector<float>(W_padded, 0.0f));
    for (int i = 0; i < H_in; ++i) {
        for (int j = 0; j < W_in; ++j) {
            padded_input[i + padding][j + padding] = input[i][j];
        }
    }

    // Calculer la taille de sortie
    int H_out = (H_padded - K_H) / stride + 1;
    int W_out = (W_padded - K_W) / stride + 1;
    Matrix output(H_out, std::vector<float>(W_out, 0.0f));

    // Opération de convolution
    for (int y = 0; y < H_out; ++y) {
        for (int x = 0; x < W_out; ++x) {
            float sum = 0.0f;
            for (int ky = 0; ky < K_H; ++ky) {
                for (int kx = 0; kx < K_W; ++kx) {
                    int input_y = y * stride + ky;
                    int input_x = x * stride + kx;
                    sum += padded_input[input_y][input_x] * kernel[ky][kx];
                }
            }
            output[y][x] = sum;
        }
    }
    return output;
}

int main() {
    Matrix image = {{1, 2, 3, 4}, {5, 6, 7, 8}, {9, 10, 11, 12}, {13, 14, 15, 16}};
    Matrix kernel = {{1, 0}, {0, 1}};
    Matrix output = convolve2d_cpp(image, kernel, 1, 0);

    std::cout << "Sortie de la convolution C++:" << std::endl;
    for (const auto& row : output) {
        for (float val : row) {
            std::cout << val << " ";
        }
        std::cout << std::endl;
    }
    return 0;
}
```

### 1.4 Optimisations de la Convolution

La convolution standard, bien que plus efficace qu'une couche entièrement connectée, peut encore être coûteuse, en particulier dans les couches profondes où le nombre de canaux d'entrée et de sortie est élevé. Plusieurs variantes ont été développées pour réduire ce coût.

#### Convolution Groupée (Grouped Convolution)

Cette technique, initialement introduite par nécessité matérielle dans AlexNet, est devenue une méthode d'optimisation standard. L'idée est de diviser les canaux d'entrée et de sortie en $G$ groupes. La convolution est ensuite effectuée indépendamment au sein de chaque groupe. Par exemple, si nous avons $C_{in}$ canaux d'entrée et $C_{out}$ canaux de sortie, et $G=2$ groupes, les $C_{in}/2$ premiers canaux d'entrée ne seront convolués que pour produire les $C_{out}/2$ premiers canaux de sortie. Les $C_{in}/2$ seconds canaux d'entrée ne produiront que les $C_{out}/2$ seconds canaux de sortie.

L'effet est une réduction directe du nombre de paramètres et du coût de calcul par un facteur de $G$.

#### Convolution Séparable en Profondeur (Depthwise Separable Convolution)

Cette technique est la pierre angulaire des architectures mobiles efficaces comme MobileNet. Elle factorise une convolution standard en deux opérations distinctes et beaucoup moins coûteuses :

1. **Convolution en Profondeur (Depthwise Convolution)** : C'est un cas extrême de convolution groupée où le nombre de groupes $G$ est égal au nombre de canaux d'entrée $C_{in}$. Chaque filtre de noyau ne s'applique qu'à un seul canal d'entrée. Cette étape filtre les informations spatiales canal par canal, mais ne combine pas les informations entre les canaux. Le coût de cette opération est très faible.

2. **Convolution Ponctuelle (Pointwise Convolution)** : Une convolution standard avec un noyau de taille 1×1 est ensuite appliquée. Son rôle est de combiner linéairement les sorties de la convolution en profondeur, créant ainsi de nouvelles caractéristiques à partir des informations inter-canaux.

La combinaison de ces deux étapes produit un résultat de forme similaire à une convolution standard, mais avec une réduction spectaculaire du coût de calcul et du nombre de paramètres. Pour une convolution 3×3, cette approche est environ 8 à 9 fois moins coûteuse qu'une convolution standard pour une perte de précision souvent minime.

| Type de Convolution | Formule du Nombre de Paramètres | Formule du Coût en MACs |
|---------------------|----------------------------------|-------------------------|
| Standard | $K_H \times K_W \times C_{in} \times C_{out}$ | $H_{out} \times W_{out} \times K_H \times K_W \times C_{in} \times C_{out}$ |
| Groupée | $\frac{K_H \times K_W \times C_{in} \times C_{out}}{G}$ | $\frac{H_{out} \times W_{out} \times K_H \times K_W \times C_{in} \times C_{out}}{G}$ |
| Séparable en Profondeur | $(K_H \times K_W \times C_{in}) + (1 \times 1 \times C_{in} \times C_{out})$ | $(H_{out} \times W_{out} \times K_H \times K_W \times C_{in}) + (H_{out} \times W_{out} \times 1 \times 1 \times C_{in} \times C_{out})$ |

### 1.5 Couches de Pooling

Les couches de pooling, ou de sous-échantillonnage, sont une autre composante essentielle des CNNs. Leur objectif principal est de réduire progressivement la dimension spatiale (hauteur et largeur) des cartes de caractéristiques. Cette réduction a deux avantages majeurs :

1. **Réduction du Coût de Calcul** : En diminuant la taille des cartes de caractéristiques, les couches de pooling réduisent le nombre de calculs et de paramètres nécessaires dans les couches convolutives ultérieures.

2. **Invariance aux Translations** : En agrégeant les informations sur une région, le pooling rend la représentation des caractéristiques plus robuste aux petites translations de l'objet dans l'image. Le fait de savoir qu'une caractéristique est présente dans une région est souvent plus important que sa position exacte.

Les deux types de pooling les plus courants sont :

- **Max Pooling** : Pour chaque fenêtre glissant sur l'entrée, seule la valeur maximale est conservée. Cette méthode est efficace pour préserver les caractéristiques les plus saillantes et les plus activées.
- **Average Pooling** : Calcule la moyenne de toutes les valeurs dans la fenêtre.

Il est important de noter que les couches de pooling n'ont aucun paramètre apprenable. L'opération est fixe. L'effet de réduction dimensionnelle est similaire à celui d'une convolution avec un pas (stride) supérieur à 1, mais la convolution apprendrait un filtre pour effectuer la transformation, tandis que le pooling applique une fonction fixe.

#### Implémentations (Max Pooling 2D)

##### Pseudo-code

```
Algorithme : MaxPool2D_Forward(Input, PoolSize, Stride)
  Entrées :
    Input: Tenseur d'entrée de taille (H_in, W_in, C)
    PoolSize: (P_H, P_W), Stride: S
  Sortie :
    Output: Tenseur de sortie de taille (H_out, W_out, C)

  Calculer H_out, W_out (formule similaire à la convolution)
  Initialiser Output avec des zéros

  Pour chaque canal c de 0 à C-1:
    Pour chaque position verticale y de 0 à H_out-1:
      Pour chaque position horizontale x de 0 à W_out-1:
        // Définir la fenêtre dans l'entrée
        y_start = y * S
        x_start = x * S
        window = Input[y_start:y_start+P_H, x_start:x_start+P_W, c]

        // Trouver la valeur maximale dans la fenêtre
        max_val = -inf
        Pour chaque valeur v dans window:
          Si v > max_val, alors max_val = v
        Fin Pour
        
        Output[y, x, c] = max_val
      Fin Pour
    Fin Pour
  Fin Pour

  Retourner Output
```

##### Implémentation Python (avec NumPy)

Une implémentation simple utilisant le slicing de NumPy.

```python
import numpy as np

def maxpool2d_numpy(input_map, pool_size, stride):
    H_in, W_in = input_map.shape
    P_H, P_W = pool_size

    H_out = int((H_in - P_H) / stride) + 1
    W_out = int((W_in - P_W) / stride) + 1
    
    output_map = np.zeros((H_out, W_out))

    for y in range(H_out):
        for x in range(W_out):
            window = input_map[y*stride:y*stride+P_H, x*stride:x*stride+P_W]
            output_map[y, x] = np.max(window)
            
    return output_map

# Exemple
feature_map = np.array([[1, 3, 2, 9], [5, 6, 1, 7], [4, 2, 8, 6], [3, 5, 7, 2]])
pooled_map = maxpool2d_numpy(feature_map, pool_size=(2, 2), stride=2)
print("Carte de caractéristiques après Max Pooling:\n", pooled_map)
```

##### Implémentation C++

```cpp
#include <iostream>
#include <vector>
#include <limits>
#include <algorithm>

using Matrix = std::vector<std::vector<float>>;

Matrix maxpool2d_cpp(const Matrix& input, int pool_h, int pool_w, int stride) {
    int H_in = input.size();
    int W_in = input[0].size();

    int H_out = (H_in - pool_h) / stride + 1;
    int W_out = (W_in - pool_w) / stride + 1;
    
    Matrix output(H_out, std::vector<float>(W_out));

    for (int y = 0; y < H_out; ++y) {
        for (int x = 0; x < W_out; ++x) {
            float max_val = -std::numeric_limits<float>::infinity();
            for (int py = 0; py < pool_h; ++py) {
                for (int px = 0; px < pool_w; ++px) {
                    max_val = std::max(max_val, input[y * stride + py][x * stride + px]);
                }
            }
            output[y][x] = max_val;
        }
    }
    return output;
}

int main() {
    Matrix feature_map = {{1, 3, 2, 9}, {5, 6, 1, 7}, {4, 2, 8, 6}, {3, 5, 7, 2}};
    Matrix pooled_map = maxpool2d_cpp(feature_map, 2, 2, 2);
    
    std::cout << "Carte de caractéristiques après Max Pooling (C++):" << std::endl;
    for (const auto& row : pooled_map) {
        for (float val : row) {
            std::cout << val << " ";
        }
        std::cout << std::endl;
    }
    return 0;
}
```

### 1.6 Couches de Normalisation

L'entraînement de réseaux de neurones très profonds est un défi. Un problème majeur qui émerge est le **décalage de covariable interne** (Internal Covariate Shift). Ce phénomène décrit comment la distribution des entrées de chaque couche change continuellement pendant l'entraînement, à mesure que les poids des couches précédentes sont mis à jour. Chaque couche doit constamment s'adapter à une distribution d'entrée changeante, ce qui ralentit considérablement la convergence et oblige à utiliser des taux d'apprentissage faibles.

La **Normalisation par Lots** (Batch Normalization), introduite par Ioffe et Szegedy en 2015, est une solution puissante à ce problème. L'idée est de normaliser les activations de chaque couche pour chaque mini-lot de données. L'algorithme se déroule en quatre étapes :

1. **Calcul de la Moyenne du Mini-Lot** : Pour une activation donnée, calculez sa moyenne $\mu_B$ sur toutes les instances du mini-lot.

2. **Calcul de la Variance du Mini-Lot** : Calculez la variance $\sigma_B^2$ de cette activation sur le mini-lot.

3. **Normalisation** : Normalisez chaque activation $x_i$ en utilisant la moyenne et la variance du lot : 
   $$\hat{x}_i = \frac{x_i - \mu_B}{\sqrt{\sigma_B^2 + \epsilon}}$$
   Le petit terme $\epsilon$ est ajouté pour la stabilité numérique.

4. **Mise à l'échelle et Décalage** : La normalisation stricte pourrait limiter la capacité de représentation de la couche. Pour contrer cela, deux paramètres apprenables, une échelle $\gamma$ (gamma) et un décalage $\beta$ (beta), sont introduits. La sortie finale de la couche de normalisation est :
   $$y_i = \gamma \hat{x}_i + \beta$$

En stabilisant la distribution des entrées des couches, la Batch Normalization permet d'utiliser des taux d'apprentissage beaucoup plus élevés, accélère la convergence et agit comme une forme de régularisation, réduisant parfois le besoin de Dropout.

Bien que la Batch Normalization soit très populaire, d'autres techniques de normalisation existent, qui diffèrent par la manière dont elles regroupent les activations pour calculer les statistiques de moyenne et de variance :

- **Layer Normalization** : Normalise sur tous les canaux pour une seule instance dans le lot. Très utilisée dans les Transformers.
- **Instance Normalization** : Normalise chaque canal indépendamment pour chaque instance.
- **Group Normalization** : Un compromis entre Layer et Instance, normalisant sur des groupes de canaux.

### 1.7 Fonctions d'Activation

Une couche composée uniquement d'opérations linéaires (comme la convolution ou les couches entièrement connectées) ne peut modéliser que des relations linéaires. L'empilement de telles couches est mathématiquement équivalent à une seule couche linéaire plus grande. Pour permettre aux réseaux de neurones d'apprendre des fonctions complexes et non linéaires, une fonction d'activation non linéaire est appliquée après chaque couche de calcul.

La fonction d'activation la plus couramment utilisée aujourd'hui est l'**Unité Linéaire Rectifiée** (Rectified Linear Unit, ReLU), définie comme :

$$f(x) = \max(0, x)$$

Avant l'avènement de ReLU, des fonctions comme la sigmoïde ou la tangente hyperbolique (tanh) étaient populaires. Cependant, ReLU a permis d'accélérer considérablement l'entraînement des réseaux profonds. Sa simplicité de calcul et le fait que sa dérivée est constante pour les entrées positives aident à atténuer le problème de la disparition du gradient (vanishing gradient).

Plusieurs variantes de ReLU existent, comme Leaky ReLU, qui a une petite pente pour les valeurs négatives afin d'éviter que les neurones ne "meurent", ou des fonctions plus complexes comme Swish. Cependant, il existe un compromis entre la complexité de la fonction d'activation et son efficacité sur le matériel. Des fonctions plus exotiques peuvent être plus coûteuses à calculer et à quantifier, faisant de la fonction ReLU, simple et efficace, un choix par défaut très solide.

## Partie 2 : Architectures de Réseaux de Neurones de Référence

En combinant les blocs de construction fondamentaux, les chercheurs ont développé des architectures qui ont défini des jalons dans le domaine de l'apprentissage profond. Comprendre leur conception et les problèmes qu'elles ont résolus est essentiel pour concevoir de nouveaux modèles efficaces.

### 2.1 AlexNet : Le Pionnier des CNNs Profonds

En 2012, l'architecture AlexNet a remporté de manière décisive le concours de classification d'images ImageNet (ILSVRC), marquant le début de la révolution de l'apprentissage profond. Proposée par Krizhevsky, Sutskever et Hinton, elle a démontré la puissance des réseaux de neurones convolutifs (CNN) profonds entraînés sur de grands ensembles de données avec des GPU.

L'architecture se compose de cinq couches de convolution, dont certaines sont suivies de couches de max-pooling, et de trois couches entièrement connectées. Au-delà de sa profondeur, AlexNet a introduit plusieurs innovations clés qui sont devenues des standards :

1. **Utilisation de l'activation ReLU** : Comme mentionné précédemment, l'utilisation de ReLU au lieu de tanh a permis un entraînement jusqu'à six fois plus rapide, ce qui était crucial pour expérimenter sur un ensemble de données aussi grand qu'ImageNet.

2. **Entraînement sur plusieurs GPU** : À l'époque, les GPU comme le NVIDIA GTX 580 n'avaient que 3 Go de VRAM, ce qui était insuffisant pour contenir le modèle entier. Les auteurs ont donc intelligemment réparti le réseau sur deux GPU. Cette contrainte matérielle a directement conduit à l'utilisation de la convolution groupée pour gérer les connexions entre les neurones sur différents GPU. Ce qui a commencé comme une solution d'ingénierie pragmatique est maintenant reconnu comme une technique d'optimisation.

3. **Utilisation du Dropout** : Les couches entièrement connectées d'AlexNet contenaient des dizaines de millions de paramètres, ce qui les rendait très sujettes au surapprentissage. L'utilisation du dropout, une technique de régularisation qui met à zéro de manière aléatoire les sorties de certains neurones pendant l'entraînement, s'est avérée très efficace pour combattre ce phénomène.

### 2.2 ResNet : Repousser les Limites de la Profondeur

Après AlexNet, la tendance était à la création de réseaux de plus en plus profonds. Cependant, les chercheurs ont rapidement rencontré un obstacle inattendu : le **problème de la dégradation**. Au-delà d'une certaine profondeur, l'ajout de couches supplémentaires n'améliorait pas la performance, mais la dégradait, même sur l'ensemble d'entraînement. Il ne s'agissait donc pas de surapprentissage, mais d'un problème d'optimisation : les solveurs avaient du mal à entraîner efficacement des réseaux très profonds.

En 2015, Kaiming He et ses collaborateurs ont proposé une solution élégante dans leur article "Deep Residual Learning for Image Recognition" : les **Réseaux Résiduels** (ResNet). L'idée centrale est de reformuler ce que les couches doivent apprendre. Au lieu d'apprendre une transformation directe (ou un mappage sous-jacent) $H(x)$, les couches apprennent une fonction résiduelle par rapport à l'entrée : $F(x) = H(x) - x$. La sortie du bloc devient alors $y = F(x) + x$.

Cette formulation est implémentée via des **connexions raccourcies** (shortcut connections), qui contournent une ou plusieurs couches et ajoutent l'entrée $x$ directement à la sortie. L'hypothèse est qu'il est beaucoup plus facile pour les couches d'apprendre à ne rien faire (c'est-à-dire de pousser le résidu $F(x)$ vers zéro) que d'apprendre une transformation identité à partir de zéro avec une pile de couches non linéaires. Si la transformation identité est optimale, le réseau peut facilement l'apprendre.

Pour les réseaux très profonds comme ResNet-50 ou ResNet-152, un design **"bottleneck"** est utilisé. Chaque bloc résiduel est composé de trois couches : une convolution 1×1 qui réduit le nombre de canaux, une convolution 3×3 qui effectue le traitement spatial sur la représentation de faible dimension, et une autre convolution 1×1 qui restaure le nombre de canaux original. Cette conception est beaucoup plus efficace en termes de calcul que l'utilisation de deux couches de convolution 3×3 avec un grand nombre de canaux.

### 2.3 Le Transformer : Une Architecture sans Récurrence

En 2017, l'article "Attention Is All You Need" de Vaswani et al. a introduit l'architecture **Transformer**, provoquant un changement de paradigme, d'abord en traitement du langage naturel (NLP) puis dans de nombreux autres domaines. Le Transformer a abandonné les architectures séquentielles traditionnelles comme les réseaux de neurones récurrents (RNN) et les CNN, qui étaient le standard pour le traitement de séquences. Les RNN traitent les données token par token, ce qui est intrinsèquement séquentiel et difficile à paralléliser. Le Transformer, lui, traite tous les tokens d'une séquence simultanément en s'appuyant uniquement sur un mécanisme appelé **attention**.

Le cœur du Transformer est le mécanisme d'attention **"Scaled Dot-Product"**. Il fonctionne avec trois vecteurs pour chaque token d'entrée : une **Requête** (Query, Q), une **Clé** (Key, K) et une **Valeur** (Value, V). L'idée est de calculer un score de similarité entre la Requête d'un token et la Clé de tous les autres tokens de la séquence. Ce score est obtenu par un produit scalaire. Les scores sont ensuite mis à l'échelle, passés à travers une fonction softmax pour obtenir des poids d'attention, et finalement utilisés pour calculer une somme pondérée des vecteurs Valeur. La formule est :

$$\text{Attention}(Q,K,V) = \text{softmax}\left(\frac{QK^T}{\sqrt{d_k}}\right)V$$

Le facteur de mise à l'échelle $\frac{1}{\sqrt{d_k}}$ (où $d_k$ est la dimension des vecteurs Clé) est crucial pour stabiliser les gradients pendant l'entraînement.

Pour permettre au modèle de se concentrer sur différents types de relations entre les tokens, le Transformer utilise l'**Attention Multi-Têtes** (Multi-Head Attention). Il exécute le mécanisme d'attention plusieurs fois en parallèle avec différentes projections linéaires de Q, K et V, puis concatène les résultats. Cela permet à chaque "tête" d'apprendre des sous-espaces de représentation différents.

Enfin, comme le modèle ne traite pas la séquence de manière ordonnée, il n'a aucune notion de la position des tokens. Pour injecter cette information, des **Encodages Positionnels** basés sur des fonctions sinus et cosinus de différentes fréquences sont ajoutés aux embeddings des tokens d'entrée.

#### Implémentations (Scaled Dot-Product Attention)

##### Pseudo-code

```
Algorithme : ScaledDotProductAttention(Q, K, V, d_k)
  Entrées :
    Q: Matrice des Requêtes de taille (seq_len_q, d_k)
    K: Matrice des Clés de taille (seq_len_k, d_k)
    V: Matrice des Valeurs de taille (seq_len_k, d_v)
    d_k: Dimension des clés
  Sortie :
    Output: Matrice de sortie de taille (seq_len_q, d_v)

  // 1. Calculer les scores de similarité
  Scores = MatMul(Q, Transpose(K))

  // 2. Mettre à l'échelle
  ScaledScores = Scores / sqrt(d_k)

  // (Optionnel) Appliquer un masque pour ignorer certains tokens
  // Si Masque existe, ajouter -inf aux positions masquées de ScaledScores

  // 3. Calculer les poids d'attention
  Weights = Softmax(ScaledScores)

  // 4. Calculer la sortie comme une somme pondérée des valeurs
  Output = MatMul(Weights, V)

  Retourner Output
```

##### Implémentation Python (avec TensorFlow/Keras)

Cette implémentation est basée sur l'exemple de Machine Learning Mastery, qui fournit une classe Keras claire et fonctionnelle.

```python
import tensorflow as tf
from tensorflow.keras.layers import Layer

class ScaledDotProductAttention(Layer):
    def __init__(self, **kwargs):
        super(ScaledDotProductAttention, self).__init__(**kwargs)

    def call(self, queries, keys, values, mask=None):
        # La dimension d_k est la dernière dimension des clés
        d_k = tf.cast(tf.shape(keys)[-1], tf.float32)
        
        # 1. Multiplication matricielle Q * K^T
        scores = tf.matmul(queries, keys, transpose_b=True)
        
        # 2. Mise à l'échelle
        scaled_scores = scores / tf.math.sqrt(d_k)
        
        # 3. Masquage (optionnel)
        if mask is not None:
            scaled_scores += (mask * -1e9)  # Ajoute une grande valeur négative
            
        # 4. Softmax pour obtenir les poids
        weights = tf.nn.softmax(scaled_scores, axis=-1)
        
        # 5. Multiplication par les valeurs
        output = tf.matmul(weights, values)
        
        return output

# Exemple d'utilisation
seq_len, d_k, d_v = 10, 64, 128
queries = tf.random.normal((1, seq_len, d_k))
keys = tf.random.normal((1, seq_len, d_k))
values = tf.random.normal((1, seq_len, d_v))

attention_layer = ScaledDotProductAttention()
output = attention_layer(queries, keys, values)
print(f"Shape de la sortie d'attention : {output.shape}")
```

##### Implémentation C++

Une implémentation C++ "from scratch" serait extrêmement complexe et inefficace. En pratique, on s'appuie sur des bibliothèques d'algèbre linéaire hautement optimisées (comme BLAS, cuBLAS). Les frameworks comme PyTorch et TensorFlow ont des implémentations C++/CUDA internes pour ces opérations. Le code suivant est une esquisse conceptuelle des opérations.

```cpp
#include <vector>
#include <cmath>
#include <iostream>
// NOTE : Ce code est purement illustratif. Il ne représente pas une
// implémentation efficace et omet de nombreux détails (batching, multi-head).

using Matrix = std::vector<std::vector<float>>;

// Fonctions d'assistance pour les opérations matricielles (simplifiées)
Matrix matmul(const Matrix& A, const Matrix& B_T) { 
    /* ... implémentation ... */ 
    return Matrix(); 
}
Matrix scale(const Matrix& A, float s) { 
    /* ... implémentation ... */ 
    return Matrix(); 
}
Matrix softmax(const Matrix& A) { 
    /* ... implémentation ... */ 
    return Matrix(); 
}

Matrix scaled_dot_product_attention(const Matrix& Q, const Matrix& K, const Matrix& V) {
    if (K.empty()) return Matrix();
    float d_k = static_cast<float>(K[0].size());

    // 1. Q * K^T (K est déjà transposé pour la simplicité)
    Matrix scores = matmul(Q, K);

    // 2. Mise à l'échelle
    Matrix scaled_scores = scale(scores, 1.0f / std::sqrt(d_k));

    // 3. Softmax
    Matrix weights = softmax(scaled_scores);
    
    // 4. Matmul avec V
    Matrix output = matmul(weights, V);

    return output;
}
```

### Tableau Récapitulatif des Architectures

| Architecture | Année | Contribution Principale | Innovation Clé | Application Typique |
|--------------|-------|------------------------|-----------------|-------------------|
| AlexNet | 2012 | A prouvé la supériorité des CNNs profonds sur de grands ensembles de données. | ReLU, Dropout, Entraînement multi-GPU. | Classification d'images. |
| VGG | 2014 | A montré que la profondeur, avec de petits filtres (3×3), est cruciale. | Empilement de couches de convolution 3×3 très homogènes. | Classification, Transfer Learning. |
| ResNet | 2015 | A résolu le problème de la dégradation, permettant des réseaux bien plus profonds. | Blocs résiduels avec connexions raccourcies (shortcut connections). | Classification, Détection, Segmentation. |
| MobileNet | 2017 | A permis des CNNs très efficaces pour les appareils mobiles et embarqués. | Convolutions séparables en profondeur (depthwise separable convolutions). | Vision par ordinateur sur mobile. |
| Transformer | 2017 | A remplacé la récurrence par l'attention, permettant une parallélisation massive. | Mécanisme d'attention "Scaled Dot-Product", Attention multi-têtes. | Traitement du langage naturel (NLP). |

## Partie 3 : Métriques d'Efficacité et Analyse de Performance

Pour concevoir des modèles efficaces, il est essentiel de disposer d'un ensemble d'outils pour mesurer et quantifier l'efficacité. Cette section présente les métriques clés utilisées pour évaluer la performance des réseaux de neurones sous différents angles : vitesse, coût de calcul, mémoire et énergie.

### 3.1 Métriques de Vitesse : Latence vs. Débit (Throughput)

Deux termes sont souvent utilisés pour décrire la "vitesse" d'un modèle, mais ils mesurent des aspects différents et ne sont pas interchangeables :

- **Latence** : C'est le temps nécessaire pour accomplir une seule tâche, par exemple le temps pour traiter une seule image. Elle est généralement mesurée en millisecondes (ms). Une faible latence est cruciale pour les applications en temps réel comme la conduite autonome ou la réalité augmentée.

- **Débit (Throughput)** : C'est le nombre de tâches qui peuvent être accomplies par unité de temps, par exemple le nombre d'images traitées par seconde (FPS). Un débit élevé est important pour les applications de traitement de données en masse, comme la classification d'une grande base de données d'images.

Une faible latence n'implique pas nécessairement un débit élevé, et vice-versa. Le traitement par lots (batching) et la parallélisation sont les principaux facteurs de cette distinction. Imaginez un système avec un seul moteur de traitement qui a une latence de 50 ms par image. Son débit sera de $1000/50 = 20$ FPS. Maintenant, imaginez un système avec quatre moteurs parallèles, où chaque moteur a une latence plus élevée de 100 ms. Bien que la latence pour une seule image soit pire, le système peut traiter quatre images en parallèle. Son débit total sera de $(1000/100) \times 4 = 40$ FPS.

Le choix de la métrique à optimiser dépend donc entièrement de l'application. Pour un système interactif, la latence est reine. Pour un système de traitement de données hors ligne, le débit est roi. L'optimisation de la latence est souvent plus difficile car elle nécessite d'accélérer le chemin critique d'une seule inférence, ce qui ne peut être résolu simplement en ajoutant plus de matériel.

### 3.2 Métriques de Coût de Calcul

Pour évaluer le coût de calcul d'un modèle de manière indépendante du matériel, on utilise des métriques qui comptent le nombre d'opérations arithmétiques.

- **MAC (Multiply-Accumulate)** : Une opération de multiplication-accumulation ($a = a + b \times c$) est l'opération la plus fondamentale et la plus fréquente dans les réseaux de neurones. Le nombre total de MACs d'un modèle quantifie sa charge de calcul.

- **FLOP (Floating Point Operation)** : Une opération à virgule flottante, comme une addition ou une multiplication.

**Relation** : Comme une opération MAC consiste en une multiplication et une addition, on considère généralement que 1 MAC ≈ 2 FLOPs.

Il est crucial de distinguer FLOPs (au pluriel, avec un 's' minuscule), qui représente une quantité de calcul (ex: "ce modèle nécessite 1.4 GigaFLOPs"), de FLOPS (avec un 'S' majuscule), qui représente une vitesse de calcul ou une puissance matérielle (ex: "ce GPU a une performance de 10 TéraFLOPS", soit $10^{12}$ opérations à virgule flottante par seconde).

Le calcul des MACs pour une couche de convolution 2D est donné par :

$$\text{MACs} = H_{out} \times W_{out} \times K_H \times K_W \times C_{in} \times C_{out}$$

Cette formule multiplie le nombre de positions de sortie par le nombre d'opérations nécessaires pour calculer chaque sortie.

### 3.3 Métriques de Mémoire

L'empreinte mémoire d'un modèle est souvent un facteur limitant, en particulier sur les appareils à ressources contraintes comme les smartphones ou les systèmes embarqués.

#### Nombre de Paramètres

C'est le nombre total de poids et de biais apprenables dans un modèle. Pour une couche de convolution, la formule est :

$$\text{Paramètres} = (K_H \times K_W \times C_{in} + 1) \times C_{out}$$

Le "+1" représente le terme de biais pour chaque filtre de sortie. Dans les architectures plus anciennes comme AlexNet, les couches entièrement connectées à la fin du réseau dominaient le nombre de paramètres en raison de leur connectivité dense.

#### Taille du Modèle (Model Size)

C'est l'espace disque requis pour stocker les paramètres du modèle. Elle est directement liée au nombre de paramètres et à la précision numérique utilisée pour les stocker :

$$\text{Taille du Modèle} = \text{Nombre de Paramètres} \times \text{Précision (en octets)}$$

Par exemple, un modèle de 7 milliards de paramètres stocké en précision 4-bit (0.5 octet par paramètre) occupera $7 \times 10^9 \times 0.5 = 3.5$ Go d'espace.

#### Taille des Activations

C'est la mémoire vive (RAM ou VRAM) nécessaire pour stocker les cartes de caractéristiques intermédiaires pendant une passe avant. Le **pic d'activation** est la quantité maximale de mémoire d'activation requise à un instant donné. C'est souvent le véritable goulot d'étranglement, car il détermine si un modèle peut s'exécuter sur un appareil donné, indépendamment de la taille du modèle sur le disque. Pour les CNNs, le profil de la taille des activations a souvent une forme en "U" : les activations sont très grandes dans les premières couches (haute résolution spatiale) et diminuent, tandis que le nombre de paramètres est faible au début et augmente dans les couches profondes (plus de canaux).

### 3.4 Métriques Énergétiques

Avec l'essor de l'IA "verte" et des applications sur batterie, l'efficacité énergétique devient une métrique de plus en plus importante. Une observation fondamentale domine ce domaine : **l'accès à la mémoire est beaucoup plus coûteux en énergie que le calcul**. Le déplacement d'une donnée depuis la DRAM vers une unité de calcul peut consommer plusieurs ordres de grandeur plus d'énergie qu'une simple opération arithmétique. Par conséquent, les modèles efficaces sur le plan énergétique sont ceux qui minimisent les mouvements de données, en favorisant la localité des données et en réduisant la taille des activations et des poids.

### Tableau Récapitulatif des Métriques

| Métrique | Définition | Unité | Mesure | Optimisé quand... |
|----------|------------|-------|--------|-------------------|
| Latence | Temps pour une seule inférence. | ms | Vitesse (temps réel) | Faible |
| Débit (Throughput) | Inférences par unité de temps. | images/s (FPS) | Vitesse (traitement de masse) | Élevé |
| MACs / FLOPs | Nombre d'opérations arithmétiques. | GMACs, GFLOPs | Coût de calcul | Faible |
| Paramètres | Nombre de poids et biais apprenables. | Millions, Milliards | Complexité du modèle | Faible |
| Taille du Modèle | Espace de stockage des paramètres. | Mo, Go | Empreinte disque | Faible |
| Pic d'Activation | Mémoire vive max requise pour les activations. | Mo, Go | Empreinte mémoire vive | Faible |
| Accès Mémoire | Quantité de données déplacées. | Go/s (Bande passante) | Coût énergétique | Faible |

## Conclusion et Perspectives

Ce cours a jeté les bases de la compréhension des réseaux de neurones sous l'angle de l'efficacité. Nous avons disséqué les blocs de construction fondamentaux, des couches linéaires et convolutives aux mécanismes d'attention, en passant par les architectures historiques qui ont façonné le domaine. Nous avons également établi un vocabulaire précis pour mesurer et analyser la performance à travers des métriques de vitesse, de calcul, de mémoire et d'énergie.

L'enseignement principal est qu'il n'existe pas de "meilleur" modèle dans l'absolu. La conception d'un système d'IA efficace est un exercice d'arbitrage constant entre des objectifs souvent contradictoires, un **"quadrilèmme"** entre la précision, la vitesse (latence/débit), la taille (paramètres/activations) et l'énergie. Le choix optimal dépend entièrement des contraintes de l'application cible.

Forts de ces connaissances fondamentales, les prochains cours exploreront les techniques avancées qui permettent de naviguer dans cet espace de compromis. Nous aborderons des méthodes comme :

- **L'élagage (Pruning)** : Supprimer les poids redondants ou non importants d'un réseau.
- **La Quantification** : Réduire la précision numérique des poids et des activations (par exemple, de 32 bits à 8 bits ou même 4 bits).
- **La Recherche d'Architecture Neuronale (Neural Architecture Search, NAS)** : Automatiser la découverte d'architectures de réseaux optimisées pour une tâche et une contrainte matérielle spécifiques.
- **La Distillation de Connaissances (Knowledge Distillation)** : Entraîner un petit modèle "étudiant" à imiter le comportement d'un grand modèle "professeur" plus performant.

Ces techniques, combinées à une solide compréhension des principes de base, vous donneront les outils nécessaires pour concevoir la prochaine génération de systèmes d'intelligence artificielle : des systèmes non seulement puissants et précis, mais aussi efficaces, accessibles et durables.
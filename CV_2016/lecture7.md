# Les Réseaux de Neurones Convolutionnels (CNN) en Vision par Ordinateur

## Table des matières
1. [Introduction](#introduction)
2. [Les Couches de Convolution](#couches-convolution)
3. [Les Couches de Pooling](#couches-pooling)
4. [Architecture Complète d'un CNN](#architecture-complete)
5. [Études de Cas : Architectures Célèbres](#etudes-de-cas)
6. [Considérations Pratiques](#considerations-pratiques)

---

## 1. Introduction {#introduction}

Les réseaux de neurones convolutionnels (CNN) représentent l'une des avancées les plus importantes en vision par ordinateur. Inspirés par les travaux de Hubel et Wiesel sur le cortex visuel des chats dans les années 1960, les CNN exploitent la structure hiérarchique du système visuel biologique.

### Contexte Historique
- **1960s** : Hubel et Wiesel découvrent l'organisation hiérarchique du cortex visuel
- **1980s** : LeNet-5 de Yann LeCun, premier CNN pratique
- **2012** : AlexNet révolutionne ImageNet avec une erreur de 18.2%
- **2015** : ResNet atteint 3.6% d'erreur avec 152 couches

### Processus d'Entraînement en 4 Étapes
1. **Forward pass** : Calcul des activations
2. **Calcul de la perte** : Mesure de l'erreur
3. **Backward pass** : Calcul des gradients
4. **Mise à jour des paramètres** : Optimisation (SGD, Adam, etc.)

---

## 2. Les Couches de Convolution {#couches-convolution}

### Principe Fondamental

Une couche de convolution transforme un volume d'entrée en un volume de sortie par l'application de filtres (ou noyaux).

**Entrée** : Volume de dimensions W₁ × H₁ × D₁  
**Sortie** : Volume de dimensions W₂ × H₂ × D₂

### Hyperparamètres
- **K** : Nombre de filtres
- **F** : Taille spatiale des filtres (ex: 3×3, 5×5)
- **S** : Stride (pas de déplacement)
- **P** : Padding (remplissage des bords)

### Formule de Calcul des Dimensions

Pour calculer les dimensions spatiales de sortie :

```
W₂ = (W₁ - F + 2P) / S + 1
H₂ = (H₁ - F + 2P) / S + 1
D₂ = K
```

### Pseudo-code de la Convolution

```python
def convolution_forward(X, W, b, stride, pad):
    """
    X : Volume d'entrée (N, C, H, W)
    W : Poids des filtres (K, C, F, F)
    b : Biais (K,)
    stride : Pas de déplacement
    pad : Padding
    """
    N, C, H, W = X.shape
    K, _, F, _ = W.shape
    
    # Padding de l'entrée
    X_padded = zero_pad(X, pad)
    
    # Calcul des dimensions de sortie
    H_out = (H - F + 2*pad) // stride + 1
    W_out = (W - F + 2*pad) // stride + 1
    
    # Initialisation de la sortie
    out = zeros((N, K, H_out, W_out))
    
    # Convolution
    for n in range(N):  # Pour chaque image
        for k in range(K):  # Pour chaque filtre
            for i in range(H_out):
                for j in range(W_out):
                    # Position dans l'image paddée
                    h_start = i * stride
                    w_start = j * stride
                    
                    # Extraction de la région
                    region = X_padded[n, :, 
                                    h_start:h_start+F,
                                    w_start:w_start+F]
                    
                    # Produit scalaire + biais
                    out[n, k, i, j] = sum(region * W[k]) + b[k]
    
    return out
```

### Exemple Pratique

**Entrée** : Image 32×32×3 (CIFAR-10)  
**Convolution** : 10 filtres 5×5, stride=1, pad=2  
**Sortie** : 32×32×10

Nombre de paramètres : 10 × (5×5×3 + 1) = 760

### Interprétation Neuronale

Chaque position dans une carte d'activation correspond à un neurone qui :
- A une **connectivité locale** (champ récepteur de F×F)
- **Partage ses poids** avec tous les neurones de la même carte
- Calcule : `y = w^T·x + b`

---

## 3. Les Couches de Pooling {#couches-pooling}

### Objectif
Réduire progressivement la dimension spatiale pour :
- Diminuer le nombre de paramètres
- Contrôler le surapprentissage
- Introduire une invariance spatiale

### Types de Pooling

#### Max Pooling (le plus courant)
```python
def max_pool_forward(X, pool_size, stride):
    """
    X : Volume d'entrée (N, C, H, W)
    pool_size : Taille de la fenêtre de pooling
    stride : Pas de déplacement
    """
    N, C, H, W = X.shape
    
    H_out = (H - pool_size) // stride + 1
    W_out = (W - pool_size) // stride + 1
    
    out = zeros((N, C, H_out, W_out))
    
    for n in range(N):
        for c in range(C):
            for i in range(H_out):
                for j in range(W_out):
                    h_start = i * stride
                    w_start = j * stride
                    
                    region = X[n, c,
                             h_start:h_start+pool_size,
                             w_start:w_start+pool_size]
                    
                    out[n, c, i, j] = max(region)
    
    return out
```

### Configuration Typique
- **2×2 max pool, stride 2** : Réduit de moitié les dimensions spatiales
- **3×3 max pool, stride 2** : Utilisé dans certaines architectures modernes

**Important** : Le pooling préserve la profondeur (D₂ = D₁)

---

## 4. Architecture Complète d'un CNN {#architecture-complete}

### Structure Typique

```
[IMAGE] → [CONV-RELU-POOL]×N → [FC]×M → [SOFTMAX]
```

### Exemple : Architecture Simple pour CIFAR-10

```
Entrée : 32×32×3
    ↓
CONV1 : 32 filtres 3×3, pad=1 → 32×32×32
RELU
POOL1 : 2×2, stride=2 → 16×16×32
    ↓
CONV2 : 64 filtres 3×3, pad=1 → 16×16×64
RELU
POOL2 : 2×2, stride=2 → 8×8×64
    ↓
CONV3 : 128 filtres 3×3, pad=1 → 8×8×128
RELU
POOL3 : 2×2, stride=2 → 4×4×128
    ↓
FC1 : 4×4×128 = 2048 → 256
RELU
FC2 : 256 → 10
SOFTMAX
```

### Considérations de Mémoire et Paramètres

Pour l'exemple ci-dessus :
- **Mémoire des activations** : Principalement dans les premières couches
- **Paramètres** : Principalement dans les couches FC

```
Mémoire par image (forward) : ~93 MB
Mémoire totale (forward + backward) : ~200 MB
```

---

## 5. Études de Cas : Architectures Célèbres {#etudes-de-cas}

### LeNet-5 (1998)
- **Architecture** : CONV-POOL-CONV-POOL-FC-FC
- **Filtres** : 5×5 partout
- **Application** : Reconnaissance de chiffres

### AlexNet (2012)
- **Profondeur** : 8 couches
- **Innovation** : 
  - Première utilisation des ReLU
  - Dropout dans les FC
  - Data augmentation intensive
- **Performance** : 15.4% erreur top-5 (ensemble)
- **Paramètres** : 60M

### VGGNet (2014)
- **Principe** : Simplicité et uniformité
- **Filtres** : Uniquement 3×3 (pad=1, stride=1)
- **Pooling** : 2×2 (stride=2)
- **Architecture VGG-16** :

```
INPUT → [CONV3-64]×2 → POOL2 
      → [CONV3-128]×2 → POOL2
      → [CONV3-256]×3 → POOL2
      → [CONV3-512]×3 → POOL2
      → [CONV3-512]×3 → POOL2
      → FC-4096 → FC-4096 → FC-1000
```

- **Performance** : 7.3% erreur top-5
- **Paramètres** : 140M (dont 100M dans le premier FC!)

### GoogLeNet/Inception (2014)
- **Innovation** : Modules Inception
- **Astuce** : Average pooling au lieu de FC
- **Performance** : 6.7% erreur top-5
- **Paramètres** : Seulement 5M!

### ResNet (2015)
- **Innovation Clé** : Connexions résiduelles

```python
def residual_block(x, filters, stride=1):
    """
    Bloc résiduel : H(x) = F(x) + x
    """
    # Branche principale
    conv1 = conv_layer(x, filters, 3, stride)
    bn1 = batch_norm(conv1)
    relu1 = relu(bn1)
    
    conv2 = conv_layer(relu1, filters, 3, 1)
    bn2 = batch_norm(conv2)
    
    # Connexion résiduelle
    if stride != 1 or x.shape[-1] != filters:
        shortcut = conv_layer(x, filters, 1, stride)
        shortcut = batch_norm(shortcut)
    else:
        shortcut = x
    
    # Addition
    out = relu1 + shortcut
    return relu(out)
```

- **Avantages** :
  - Permet d'entraîner des réseaux très profonds (152 couches)
  - Gradient flow amélioré
  - Performance : 3.6% erreur top-5

### Évolution des Performances ImageNet

| Année | Architecture | Erreur Top-5 | Couches |
|-------|-------------|--------------|---------|
| 2012  | AlexNet     | 15.4%        | 8       |
| 2013  | ZFNet       | 11.2%        | 8       |
| 2014  | VGG         | 7.3%         | 19      |
| 2014  | GoogLeNet   | 6.7%         | 22      |
| 2015  | ResNet      | 3.6%         | 152     |

**Note** : Performance humaine estimée à ~5%

---

## 6. Considérations Pratiques {#considerations-pratiques}

### Hyperparamètres Typiques (ResNet)

```python
# Configuration d'entraînement
config = {
    'optimizer': 'SGD',
    'momentum': 0.9,
    'learning_rate': 0.1,  # Plus élevé grâce à BatchNorm
    'lr_schedule': 'divide by 10 when plateau',
    'batch_size': 256,
    'weight_decay': 1e-4,
    'initialization': 'he_normal',  # Xavier/2 pour ReLU
    'batch_norm': True,
    'dropout': False  # Pas nécessaire avec BatchNorm
}
```

### Temps d'Entraînement
- **ResNet-152** : 2-3 semaines sur 8 GPUs
- **VGG-16** : Plus lent que ResNet malgré moins de couches

### Tendances Actuelles

1. **Élimination du pooling** : Utilisation de convolutions avec stride
2. **Élimination des FC** : Global average pooling
3. **Filtres plus petits** : 3×3 dominants
4. **Plus de profondeur** : Avec connexions résiduelles

### Application : AlphaGo (2016)

CNN pour évaluer les positions au Go :
- **Entrée** : 19×19×48 (plateau + features)
- **Architecture** : 12 couches de convolution
- **Sortie** : 19×19 (probabilité par position)

```
INPUT → [CONV5-192-pad2] → [CONV3-192-pad1]×11 → CONV1-1
```

---

## Conclusion

Les CNN ont révolutionné la vision par ordinateur grâce à :
- **Connectivité locale** : Exploitation de la structure spatiale
- **Partage de poids** : Réduction drastique des paramètres
- **Hiérarchie de features** : Des contours simples aux concepts complexes

L'évolution continue vers des architectures plus profondes et efficaces, avec des innovations comme les connexions résiduelles qui permettent d'entraîner des réseaux de plus en plus complexes.
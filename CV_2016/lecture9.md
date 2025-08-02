# Comprendre et Visualiser les Réseaux de Neurones Convolutionnels

## Table des matières
1. [Introduction](#introduction)
2. [Visualisations Basiques](#visualisations-basiques)
3. [Expériences d'Occlusion](#experiences-occlusion)
4. [Approche par Déconvolution](#approche-deconvolution)
5. [Approches par Optimisation](#approches-optimisation)
6. [DeepDream](#deepdream)
7. [Transfert de Style Neural](#transfert-style)
8. [Exemples Adversariaux](#exemples-adversariaux)
9. [Implications et Conclusions](#implications-conclusions)

---

## 1. Introduction {#introduction}

Les réseaux de neurones convolutionnels (CNN) ont révolutionné la vision par ordinateur, mais leur fonctionnement interne reste souvent opaque. Ce cours explore diverses techniques pour comprendre et visualiser ce que les CNN apprennent réellement.

### Contexte et Motivation
- Les CNN atteignent des performances remarquables (ex: 5% d'erreur top-5 sur ImageNet)
- Mais comment cette performance est-elle obtenue ?
- Comprendre les représentations internes est crucial pour :
  - Déboguer les modèles
  - Améliorer les architectures
  - Identifier les vulnérabilités

### Outils et Ressources
- **DeepVis Toolbox** : Visualisation en temps réel des activations CNN
- **Implementations** : Torch, Caffe, TensorFlow
- **Modèles pré-entraînés** : ResNet-152 maintenant disponible

---

## 2. Visualisations Basiques {#visualisations-basiques}

### 2.1 Visualisation des Activations

La méthode la plus simple consiste à observer quelles images activent le plus fortement un neurone donné.

**Procédure** :
1. Choisir un neurone arbitraire (ex: dans la couche pool5)
2. Passer l'ensemble de données à travers le CNN
3. Collecter les images qui maximisent l'activation de ce neurone

**Observations typiques** :
- Certains neurones détectent des **projecteurs** sur les visages
- D'autres répondent au **texte** dans diverses polices
- Certains détectent des **chiens** ou des **drapeaux américains**

### 2.2 Visualisation des Poids

Pour la première couche convolutionnelle uniquement, on peut visualiser directement les filtres.

**Caractéristiques des filtres de première couche** :
- Filtres de type **Gabor** (détecteurs d'arêtes orientées)
- Détecteurs de **couleurs** et de **blobs**
- Structure émergente de l'entraînement supervisé

**Limitation** : Cette visualisation n'est interprétable que pour la première couche car les couches supérieures opèrent sur des activations, non sur l'image brute.

### 2.3 Visualisation t-SNE des Représentations

t-SNE permet de visualiser les codes haute dimension (ex: FC7 features) en 2D.

**Algorithme conceptuel** :
```python
def tsne_visualization(features, perplexity=30):
    """
    features : Matrice (N, D) de N vecteurs de dimension D
    perplexity : Paramètre contrôlant le nombre de voisins
    """
    # 1. Calculer les similarités dans l'espace original
    P = compute_pairwise_similarities(features)
    
    # 2. Initialiser les positions 2D aléatoirement
    Y = random_init(N, 2)
    
    # 3. Optimiser pour préserver les distances relatives
    for iteration in range(1000):
        Q = compute_low_dim_similarities(Y)
        gradient = compute_gradient(P, Q, Y)
        Y = Y - learning_rate * gradient
    
    return Y
```

**Résultats typiques** :
- Les objets similaires se regroupent (ex: tous les chiens ensemble)
- Structure sémantique émergente (animaux, véhicules, nourriture)
- Révèle l'organisation interne du CNN

---

## 3. Expériences d'Occlusion {#experiences-occlusion}

### Principe
Masquer systématiquement des parties de l'image pour identifier les régions critiques pour la classification.

### Implémentation

```python
def occlusion_sensitivity(model, image, occluder_size=50, stride=10):
    """
    Calcule une carte de sensibilité par occlusion
    
    model : CNN pré-entraîné
    image : Image d'entrée
    occluder_size : Taille du masque d'occlusion
    stride : Pas de déplacement du masque
    """
    H, W = image.shape[:2]
    original_prob = model.predict(image)
    sensitivity_map = zeros((H, W))
    
    for y in range(0, H - occluder_size, stride):
        for x in range(0, W - occluder_size, stride):
            # Copier l'image et appliquer l'occlusion
            occluded = image.copy()
            occluded[y:y+occluder_size, x:x+occluder_size] = 0
            
            # Calculer la chute de probabilité
            new_prob = model.predict(occluded)
            sensitivity_map[y:y+occluder_size, x:x+occluder_size] = \
                original_prob - new_prob
    
    return sensitivity_map
```

### Résultats Typiques
- **Chien Poméranien** : Probabilité chute drastiquement quand le visage est occulté
- **Roue de char** : Sensible à l'occlusion de la roue elle-même
- **Validation** : Le CNN regarde bien les parties pertinentes de l'objet

---

## 4. Approche par Déconvolution {#approche-deconvolution}

### 4.1 Gradient Simple vs Guided Backpropagation

**Problème** : Les gradients simples produisent des visualisations bruitées et difficiles à interpréter.

**Solution** : Guided Backpropagation modifie la rétropropagation à travers les couches ReLU.

### 4.2 Algorithme de Guided Backpropagation

```python
def guided_relu_backward(grad_output, activation_input):
    """
    Rétropropagation modifiée pour ReLU
    
    grad_output : Gradient venant de la couche supérieure
    activation_input : Entrée originale de la couche ReLU
    """
    # ReLU standard : transmet si input > 0
    standard_mask = (activation_input > 0)
    
    # Guided : transmet aussi seulement si gradient > 0
    guided_mask = (grad_output > 0)
    
    # Combiner les deux conditions
    return grad_output * standard_mask * guided_mask
```

### 4.3 Intuition

- **Gradient standard** : Mélange d'influences positives et négatives qui s'annulent
- **Guided backprop** : Ne garde que les chemins d'influence entièrement positifs
- **Résultat** : Visualisations plus nettes et interprétables

### 4.4 Variante : Deconvnet

Une approche alternative qui ignore complètement l'état des activations ReLU :

```python
def deconv_relu_backward(grad_output, activation_input):
    """
    Deconvnet : ignore l'état d'activation, garde seulement gradients positifs
    """
    return grad_output * (grad_output > 0)
```

---

## 5. Approches par Optimisation {#approches-optimisation}

### 5.1 Principe Général

Au lieu de visualiser un gradient unique, optimiser l'image pour maximiser l'activation d'un neurone spécifique.

### 5.2 Formulation Mathématique

```
max_I S_c(I) - λ||I||²₂
```

Où :
- `I` : Image à optimiser
- `S_c` : Score/activation pour la classe/neurone c
- `λ` : Coefficient de régularisation

### 5.3 Algorithme d'Optimisation

```python
def optimize_image_for_neuron(model, layer_name, neuron_index, 
                             iterations=1000, lr=1.0, lambda_reg=1e-6):
    """
    Génère une image qui maximise l'activation d'un neurone spécifique
    """
    # Initialiser avec bruit aléatoire ou image nulle
    image = random_normal(size=(224, 224, 3)) * 0.01
    
    for i in range(iterations):
        # Forward pass jusqu'à la couche cible
        activation = model.forward_to_layer(image, layer_name)
        
        # Définir le gradient pour le neurone cible
        grad = zeros_like(activation)
        grad[neuron_index] = 1.0
        
        # Backprop jusqu'à l'image
        image_grad = model.backward_from_layer(grad, layer_name)
        
        # Mise à jour avec régularisation L2
        image += lr * (image_grad - lambda_reg * image)
        
        # Optionnel : normaliser ou contraindre l'image
        image = clip(image, -mean/std, (1-mean)/std)
    
    return image
```

### 5.4 Régularisation Avancée

Au lieu de la régularisation L2, certains travaux utilisent :

```python
def optimize_with_blur_regularization(model, neuron, iterations=1000):
    """
    Utilise le flou gaussien comme régularisation implicite
    """
    image = random_normal(size=(224, 224, 3)) * 0.01
    
    for i in range(iterations):
        # Étapes d'optimisation standard
        grad = compute_gradient(model, image, neuron)
        image += learning_rate * grad
        
        # Régularisation par flou (évite les hautes fréquences)
        image = gaussian_blur(image, sigma=0.5)
        
        # Transformation occasionnelle (jittering)
        if i % 10 == 0:
            image = random_jitter(image, max_shift=2)
    
    return image
```

### 5.5 Résultats par Couche

- **Couches profondes (7-8)** : Objets reconnaissables (flamants, pélicans)
- **Couche 5** : Textures complexes, parties d'objets
- **Couche 4** : Motifs répétitifs, textures simples
- **Couches 1-3** : Gabors, couleurs, contours

---

## 6. DeepDream {#deepdream}

### 6.1 Concept

DeepDream amplifie les patterns existants dans une image en maximisant les activations qui sont déjà présentes.

### 6.2 Algorithme Principal

```python
def deepdream_step(net, image, layer='inception_4c', lr=1.5):
    """
    Une itération de DeepDream
    
    net : Réseau pré-entraîné (généralement InceptionNet)
    image : Image à modifier
    layer : Couche où "rêver"
    """
    # Forward jusqu'à la couche cible
    net.forward(image, end=layer)
    activations = net.blobs[layer].data
    
    # ASTUCE CLÉE : gradient = activations
    # Cela amplifie ce qui est déjà activé
    net.blobs[layer].diff[:] = activations
    
    # Backprop jusqu'à l'image
    net.backward(start=layer)
    grad = net.blobs['data'].diff[0]
    
    # Normaliser et mettre à jour
    grad_normalized = grad / (abs(grad).mean() + 1e-7)
    image += lr * grad_normalized
    
    return clip(image, -1, 1)
```

### 6.3 Améliorations Multi-échelles

```python
def deepdream_multiscale(net, image, iterations=10, octaves=4):
    """
    DeepDream avec traitement multi-échelles (octaves)
    """
    # Sauvegarder l'image originale
    original = image.copy()
    
    # Créer une pyramide d'images
    for octave in range(octaves):
        if octave > 0:
            # Agrandir l'image de 40%
            image = zoom(image, 1.4)
            
        for i in range(iterations):
            # Jittering pour réduire les artefacts
            shift_x, shift_y = random.randint(-10, 10, size=2)
            image = roll(image, shift=[shift_x, shift_y])
            
            # Étape DeepDream
            image = deepdream_step(net, image)
            
            # Annuler le jittering
            image = roll(image, shift=[-shift_x, -shift_y])
    
    return image
```

### 6.4 Interprétation

- **Amplification récursive** : Ce qui ressemble vaguement à un chien devient de plus en plus "chien"
- **Biais du dataset** : ImageNet contient beaucoup de chiens → hallucinations canines
- **Couches différentes** : 
  - Couches basses → patterns géométriques
  - Couches hautes → objets reconnaissables

---

## 7. Transfert de Style Neural {#transfert-style}

### 7.1 Principe

Combiner le **contenu** d'une image avec le **style** d'une autre.

### 7.2 Représentations

**Contenu** : Activations brutes à une couche donnée
**Style** : Matrices de Gram (corrélations entre cartes de features)

### 7.3 Calcul de la Matrice de Gram

```python
def compute_gram_matrix(features):
    """
    Calcule la matrice de Gram pour capturer le style
    
    features : Tensor de forme (C, H, W)
    C : nombre de canaux, H : hauteur, W : largeur
    """
    C, H, W = features.shape
    
    # Reshape en matrice (C, H*W)
    features_flat = features.reshape(C, H * W)
    
    # Matrice de Gram = corrélations entre features
    gram = matmul(features_flat, features_flat.T)
    
    # Normaliser par le nombre d'éléments
    return gram / (C * H * W)
```

### 7.4 Fonction de Perte Complète

```python
def style_transfer_loss(generated, content_img, style_img, 
                       content_layers, style_layers, alpha=1e3, beta=1):
    """
    Perte totale pour le transfert de style
    """
    # Forward pass pour toutes les images
    content_features = extract_features(content_img, content_layers)
    style_features = extract_features(style_img, style_layers)
    generated_features = extract_features(generated, 
                                        content_layers + style_layers)
    
    # Perte de contenu
    content_loss = 0
    for layer in content_layers:
        content_loss += mse(generated_features[layer], 
                           content_features[layer])
    
    # Perte de style
    style_loss = 0
    for layer in style_layers:
        G_gen = compute_gram_matrix(generated_features[layer])
        G_style = compute_gram_matrix(style_features[layer])
        style_loss += mse(G_gen, G_style)
    
    # Perte totale pondérée
    total_loss = alpha * content_loss + beta * style_loss
    
    return total_loss
```

### 7.5 Optimisation

```python
def neural_style_transfer(content_img, style_img, iterations=1000):
    """
    Algorithme complet de transfert de style
    """
    # Initialiser avec l'image de contenu (ou bruit)
    generated = content_img.copy()
    
    # Définir les couches pour contenu et style
    content_layers = ['conv4_2']  # Une seule couche suffit
    style_layers = ['conv1_1', 'conv2_1', 'conv3_1', 'conv4_1', 'conv5_1']
    
    # Optimiseur L-BFGS (meilleur que SGD pour ce problème)
    optimizer = LBFGS([generated], lr=1.0)
    
    for i in range(iterations):
        def closure():
            optimizer.zero_grad()
            loss = style_transfer_loss(generated, content_img, 
                                     style_img, content_layers, 
                                     style_layers)
            loss.backward()
            return loss
        
        optimizer.step(closure)
        
        # Contraindre les valeurs des pixels
        generated.data.clamp_(0, 1)
    
    return generated
```

### 7.6 Résultats et Intuitions

- La matrice de Gram capture les **corrélations entre features** (texture)
- Les activations brutes préservent la **structure spatiale** (contenu)
- L'équilibre α/β contrôle le compromis contenu/style

---

## 8. Exemples Adversariaux {#exemples-adversariaux}

### 8.1 Découverte Troublante

De minuscules perturbations imperceptibles peuvent complètement tromper les CNN.

### 8.2 Exemple Simple : Régression Logistique

```python
# Configuration initiale
x = [2, 1, 3, 2, -4, -2, 2, -3, 1, 2]  # Exemple d'entrée
w = [1, -1, 3, -1, -1, 2, 2, -3, -1, 4]  # Poids du modèle

# Score original
score = dot(w, x) = -3  # Probabilité classe 1 : 4.7%

# Construction d'exemple adversarial
# Pour chaque dimension i : x_adv[i] = x[i] - 0.5 * sign(w[i])
x_adv = [1.5, 1.5, 2.5, 2.5, -3.5, -2.5, 1.5, -2.5, 1.5, 1.5]

# Nouveau score
score_adv = dot(w, x_adv) = 2  # Probabilité classe 1 : 88%!
```

### 8.3 Méthode FGSM (Fast Gradient Sign Method)

```python
def generate_adversarial_example(model, image, true_label, epsilon=0.01):
    """
    Génère un exemple adversarial en utilisant FGSM
    
    model : CNN à attaquer
    image : Image originale
    true_label : Vraie classe de l'image
    epsilon : Magnitude de la perturbation
    """
    # Calculer la perte pour la vraie classe
    loss = cross_entropy(model(image), true_label)
    
    # Calculer le gradient par rapport à l'image
    grad = compute_gradient(loss, image)
    
    # Créer la perturbation adversariale
    perturbation = epsilon * sign(grad)
    
    # Générer l'exemple adversarial
    adversarial = image + perturbation
    
    return clip(adversarial, 0, 1)
```

### 8.4 Attaque Ciblée

```python
def targeted_adversarial_attack(model, image, target_class, 
                               iterations=100, alpha=0.01):
    """
    Force le modèle à classifier l'image comme target_class
    """
    adv_image = image.copy()
    
    for i in range(iterations):
        # Maximiser la probabilité de target_class
        output = model(adv_image)
        loss = -log(softmax(output)[target_class])
        
        # Gradient descent sur l'image
        grad = compute_gradient(loss, adv_image)
        adv_image -= alpha * grad
        
        # Contraindre la perturbation
        perturbation = adv_image - image
        perturbation = clip(perturbation, -epsilon, epsilon)
        adv_image = image + perturbation
    
    return adv_image
```

### 8.5 Causes Profondes

1. **Linéarité** : Les CNN utilisent beaucoup de fonctions linéaires (convolutions, FC)
2. **Haute dimension** : Dans un espace de 150K dimensions, petites perturbations × grandes dimensions = gros effets
3. **Hors distribution** : Les exemples adversariaux sont hors du manifold des images naturelles

### 8.6 Défenses Potentielles

```python
def adversarial_training(model, train_data, epochs=10):
    """
    Entraînement adversarial pour robustifier le modèle
    """
    for epoch in range(epochs):
        for images, labels in train_data:
            # Entraînement standard
            loss_clean = train_step(model, images, labels)
            
            # Générer exemples adversariaux
            adv_images = generate_adversarial_examples(model, images, labels)
            
            # Entraînement sur exemples adversariaux
            loss_adv = train_step(model, adv_images, labels)
            
            # Perte totale
            total_loss = loss_clean + loss_adv
            
            # Mise à jour des poids
            optimizer.step(total_loss)
```

---

## 9. Implications et Conclusions {#implications-conclusions}

### 9.1 Ce que nous avons appris

1. **Hiérarchie de features** : Des détecteurs simples (Gabor) aux concepts complexes
2. **Interprétabilité** : Plusieurs méthodes pour comprendre les décisions des CNN
3. **Vulnérabilités** : Les CNN peuvent être facilement trompés malgré leur performance

### 9.2 Applications Pratiques

- **Débogage** : Identifier pourquoi un modèle échoue
- **Amélioration** : Comprendre quelles features sont importantes
- **Créativité** : DeepDream, transfert de style pour l'art
- **Sécurité** : Identifier et corriger les vulnérabilités

### 9.3 Directions Futures

1. **Robustesse** : Développer des modèles résistants aux attaques adversariales
2. **Interprétabilité** : Méthodes plus sophistiquées pour comprendre les décisions
3. **Au-delà de la classification** : Appliquer ces techniques à d'autres tâches

### 9.4 Message Clé

La rétropropagation vers l'image est un outil puissant qui permet :
- La **compréhension** (visualisations)
- La **créativité** (DeepDream, style transfer)
- La **découverte de vulnérabilités** (exemples adversariaux)

Mais elle révèle aussi que nos modèles, malgré leur performance impressionnante, fonctionnent de manière fondamentalement différente de la perception humaine.
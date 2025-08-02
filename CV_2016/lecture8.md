# Localisation et Détection d'Objets en Vision par Ordinateur

## Introduction

Dans ce cours, nous allons explorer les applications avancées des réseaux de neurones convolutifs (CNN) pour la localisation spatiale et la détection d'objets dans les images. Après avoir maîtrisé la classification d'images, nous allons maintenant apprendre à localiser et détecter des objets.

## 1. Vue d'ensemble des tâches

### 1.1 Classification vs Localisation vs Détection

- **Classification** : Identifier quelle catégorie d'objet est présente dans l'image
- **Localisation** : Identifier où se trouve l'objet (avec une boîte englobante)
- **Classification + Localisation** : Faire les deux simultanément
- **Détection** : Trouver toutes les instances de plusieurs catégories d'objets
- **Segmentation d'instance** : Identifier précisément les pixels de chaque instance

### 1.2 Différence clé
- **Localisation** : Nombre fixe d'objets (généralement un)
- **Détection** : Nombre variable d'objets

## 2. Classification et Localisation

### 2.1 Paradigme de régression

La localisation peut être vue comme un problème de régression où nous prédisons 4 nombres réels pour paramétrer une boîte englobante :
- (x, y) : coordonnées du coin supérieur gauche
- (w, h) : largeur et hauteur de la boîte

### 2.2 Architecture simple

```python
# Pseudo-code pour classification + localisation
def build_localization_network(pretrained_model):
    # 1. Charger un modèle pré-entraîné (AlexNet, VGG, etc.)
    base_model = load_pretrained_model(pretrained_model)
    
    # 2. Ajouter une tête de classification
    classification_head = FullyConnected(num_classes)
    
    # 3. Ajouter une tête de régression
    regression_head = FullyConnected(4)  # 4 coordonnées
    
    # 4. Combiner le tout
    return CombinedModel(base_model, classification_head, regression_head)

def train_localization(model, data):
    for image, label, bbox in data:
        # Forward pass
        features = model.base(image)
        class_scores = model.classification_head(features)
        bbox_pred = model.regression_head(features)
        
        # Calcul des pertes
        class_loss = CrossEntropyLoss(class_scores, label)
        bbox_loss = L2Loss(bbox_pred, bbox)
        total_loss = class_loss + λ * bbox_loss
        
        # Backpropagation
        backpropagate(total_loss)
```

### 2.3 Choix de conception

#### 2.3.1 Régression agnostique vs spécifique à la classe
- **Agnostique** : 4 nombres pour la boîte, peu importe la classe
- **Spécifique** : C × 4 nombres (une boîte par classe)

#### 2.3.2 Point d'attache de la tête de régression
- Après la dernière couche convolutive
- Après les couches entièrement connectées

### 2.4 Localisation multiple

Pour localiser un nombre fixe K d'objets :
```python
regression_head = FullyConnected(K * 4)  # K boîtes
```

Application : Estimation de pose humaine (localiser les articulations)

## 3. Fenêtre glissante et OverFeat

### 3.1 Architecture OverFeat (2013)

OverFeat utilise une approche par fenêtre glissante efficace :

1. Évaluer le réseau à plusieurs positions sur l'image
2. Agréger les prédictions

### 3.2 Astuce d'efficacité : Convolutionnaliser les couches FC

```python
# Transformation des couches FC en convolutions
def convert_fc_to_conv(fc_layer):
    if fc_layer.input_shape == (7, 7, 512):  # Exemple
        # FC de 7×7×512 → 4096 devient Conv 7×7
        return Conv2D(filters=4096, kernel_size=(7, 7))
    elif fc_layer.input_shape == (1, 1, 4096):
        # FC de 4096 → 4096 devient Conv 1×1
        return Conv2D(filters=4096, kernel_size=(1, 1))
```

Cette transformation permet d'appliquer le réseau sur des images de tailles différentes et d'obtenir des cartes de scores spatiales.

## 4. Détection d'objets

### 4.1 Le défi

La détection est plus complexe car :
- Nombre variable d'objets
- Positions inconnues
- Échelles différentes

### 4.2 Approche historique : HOG + SVM

Avant le deep learning (2005-2012) :
- **HOG** (Histogram of Oriented Gradients) : Descripteur de caractéristiques
- **DPM** (Deformable Parts Model) : Modèle avec parties déformables
- Classification rapide avec SVM linéaire

### 4.3 Propositions de régions

Solution moderne : Ne pas tester toutes les positions possibles, mais utiliser des **propositions de régions**.

#### Selective Search
Algorithme de proposition de régions :
1. Segmentation initiale en régions
2. Fusion hiérarchique basée sur similarité (couleur, texture)
3. Génération de boîtes englobantes

```python
def selective_search(image):
    # 1. Segmentation initiale
    segments = initial_segmentation(image)
    
    # 2. Calcul des similarités
    similarities = compute_similarities(segments)
    
    # 3. Fusion hiérarchique
    regions = []
    while len(segments) > 1:
        # Fusionner les régions les plus similaires
        i, j = find_most_similar(similarities)
        merged = merge_regions(segments[i], segments[j])
        regions.append(get_bounding_box(merged))
        update_similarities(similarities, merged)
    
    return regions
```

## 5. R-CNN (2014)

### 5.1 Architecture

```python
def RCNN_pipeline(image):
    # 1. Générer ~2000 propositions de régions
    proposals = selective_search(image)
    
    # 2. Pour chaque proposition
    predictions = []
    for box in proposals:
        # Extraire et redimensionner la région
        region = crop_and_resize(image, box, target_size=(224, 224))
        
        # Passer dans le CNN
        features = CNN(region)
        
        # Classification avec SVM
        class_scores = SVM(features)
        
        # Régression de boîte
        bbox_correction = bbox_regressor(features)
        
        predictions.append((class_scores, apply_correction(box, bbox_correction)))
    
    return predictions
```

### 5.2 Entraînement (complexe!)

1. Pré-entraîner CNN sur ImageNet
2. Fine-tuner sur données de détection
3. Extraire et sauvegarder features sur disque
4. Entraîner SVM binaires par classe
5. Entraîner régresseurs de boîtes

### 5.3 Problèmes
- Très lent : ~50 secondes par image
- Pipeline d'entraînement complexe
- Beaucoup d'espace disque nécessaire

## 6. Fast R-CNN (2015)

### 6.1 Innovation clé : Inverser l'ordre

Au lieu de : Image → Régions → CNN pour chaque région
Faire : Image → CNN → Extraire features des régions

### 6.2 ROI Pooling

```python
def roi_pooling(feature_map, roi, output_size=(7, 7)):
    """
    Extrait une représentation de taille fixe d'une région d'intérêt
    
    Args:
        feature_map: Carte de features H×W×C
        roi: (x1, y1, x2, y2) dans l'espace image
        output_size: Taille de sortie désirée
    """
    # 1. Projeter ROI sur la feature map
    roi_feature = project_roi_to_feature_map(roi, feature_map)
    
    # 2. Diviser en grille
    h, w = output_size
    bin_h = roi_feature.height / h
    bin_w = roi_feature.width / w
    
    # 3. Max pooling dans chaque cellule
    output = np.zeros((h, w, feature_map.channels))
    for i in range(h):
        for j in range(w):
            # Définir les limites de la cellule
            y1 = int(i * bin_h)
            y2 = int((i + 1) * bin_h)
            x1 = int(j * bin_w)
            x2 = int((j + 1) * bin_w)
            
            # Max pooling
            output[i, j] = np.max(roi_feature[y1:y2, x1:x2], axis=(0, 1))
    
    return output
```

### 6.3 Architecture Fast R-CNN

```python
def fast_rcnn_forward(image, proposals):
    # 1. Une seule passe CNN sur l'image entière
    feature_map = CNN_backbone(image)
    
    # 2. Pour chaque proposition
    outputs = []
    for roi in proposals:
        # ROI Pooling
        roi_features = roi_pooling(feature_map, roi)
        
        # Couches FC
        fc_features = fully_connected_layers(roi_features)
        
        # Deux têtes
        class_scores = classification_head(fc_features)
        bbox_deltas = regression_head(fc_features)
        
        outputs.append((class_scores, bbox_deltas))
    
    return outputs
```

### 6.4 Avantages
- Entraînement unifié end-to-end
- 10× plus rapide à l'entraînement
- 150× plus rapide au test (sans compter les propositions)

## 7. Faster R-CNN (2015)

### 7.1 Innovation : Region Proposal Network (RPN)

Utiliser un CNN aussi pour générer les propositions !

### 7.2 Architecture RPN

```python
def region_proposal_network(feature_map, num_anchors=9):
    """
    Génère des propositions de régions à partir de la feature map
    
    Args:
        feature_map: H×W×C carte de features
        num_anchors: Nombre d'ancres par position (3 échelles × 3 ratios)
    """
    # 1. Convolution 3×3 (fenêtre glissante)
    intermediate = Conv2D(512, (3, 3), padding='same')(feature_map)
    intermediate = ReLU()(intermediate)
    
    # 2. Deux branches de sortie
    # Classification : objet ou fond
    cls_output = Conv2D(num_anchors * 2, (1, 1))(intermediate)  # 2 pour binaire
    
    # Régression : corrections de boîtes
    reg_output = Conv2D(num_anchors * 4, (1, 1))(intermediate)  # 4 coordonnées
    
    return cls_output, reg_output
```

### 7.3 Boîtes d'ancrage

Pour chaque position dans la feature map :
- 3 échelles : {128², 256², 512²}
- 3 ratios d'aspect : {1:1, 1:2, 2:1}
- Total : 9 ancres par position

### 7.4 Pipeline complet Faster R-CNN

```python
def faster_rcnn(image):
    # 1. Backbone CNN
    feature_map = CNN_backbone(image)
    
    # 2. RPN pour générer des propositions
    rpn_cls, rpn_reg = region_proposal_network(feature_map)
    proposals = generate_proposals(rpn_cls, rpn_reg, anchors)
    
    # 3. Fast R-CNN sur les propositions
    final_cls, final_reg = fast_rcnn_head(feature_map, proposals)
    
    return final_cls, final_reg

def training_loss(predictions, ground_truth):
    # 4 pertes au total
    rpn_cls_loss = binary_cross_entropy(rpn_cls, rpn_labels)
    rpn_reg_loss = smooth_l1_loss(rpn_reg, rpn_targets)
    final_cls_loss = cross_entropy(final_cls, class_labels)
    final_reg_loss = smooth_l1_loss(final_reg, bbox_targets)
    
    return rpn_cls_loss + rpn_reg_loss + final_cls_loss + final_reg_loss
```

### 7.5 Performance
- 0.2 secondes par image (5 FPS)
- Performance état de l'art

## 8. YOLO : You Only Look Once

### 8.1 Approche radicalement différente

Traiter la détection comme un problème de régression directe !

### 8.2 Architecture

```python
def YOLO(image, S=7, B=2, C=20):
    """
    S×S : grille spatiale
    B : nombre de boîtes par cellule
    C : nombre de classes
    """
    # 1. Diviser l'image en grille S×S
    # 2. Chaque cellule prédit :
    #    - B boîtes (x, y, w, h, confidence)
    #    - C probabilités de classes
    
    # Output shape: S × S × (B*5 + C)
    output = CNN(image)  # Shape: 7 × 7 × 30 pour YOLO v1
    
    # Interpréter la sortie
    for i in range(S):
        for j in range(S):
            cell_output = output[i, j]
            
            # B boîtes avec confiance
            boxes = []
            for b in range(B):
                x, y, w, h = cell_output[b*5:b*5+4]
                confidence = cell_output[b*5+4]
                boxes.append((x, y, w, h, confidence))
            
            # Probabilités de classes
            class_probs = cell_output[B*5:]
            
    return interpreted_output
```

### 8.3 Avantages et inconvénients
- ✓ Très rapide : 45 FPS
- ✓ Raisonnement global sur l'image
- ✗ Performance légèrement inférieure
- ✗ Limite sur le nombre d'objets détectables

## 9. Métriques d'évaluation

### 9.1 Intersection over Union (IoU)

```python
def compute_iou(box1, box2):
    # Calculer l'intersection
    x1 = max(box1[0], box2[0])
    y1 = max(box1[1], box2[1])
    x2 = min(box1[2], box2[2])
    y2 = min(box1[3], box2[3])
    
    intersection = max(0, x2 - x1) * max(0, y2 - y1)
    
    # Calculer l'union
    area1 = (box1[2] - box1[0]) * (box1[3] - box1[1])
    area2 = (box2[2] - box2[0]) * (box2[3] - box2[1])
    union = area1 + area2 - intersection
    
    return intersection / union if union > 0 else 0
```

### 9.2 Mean Average Precision (mAP)
- Mesure standard pour la détection
- Prend en compte précision et rappel
- Seuil IoU typique : 0.5

## 10. État de l'art actuel

### 10.1 ResNet + Faster R-CNN
- Backbone : ResNet-101 (101 couches)
- Astuces supplémentaires :
  - Raffinement itératif des boîtes
  - Contexte global
  - Test multi-échelle
  - Ensemble de modèles

### 10.2 Performances sur ImageNet Detection 2015
- mAP : 62.1% (amélioration de 16% vs 2014)
- Un seul modèle ResNet bat tous les ensembles précédents

## 11. Conseils pratiques pour vos projets

### 11.1 Choix de l'approche

1. **Si nombre fixe d'objets** → Localisation par régression
2. **Si besoin de rapidité** → YOLO
3. **Si besoin de précision** → Faster R-CNN
4. **Si ressources limitées** → Fast R-CNN avec propositions pré-calculées

### 11.2 Code disponible
- **Faster R-CNN** : Python + Caffe, pas de MATLAB requis
- **YOLO** : Très rapide, bon pour prototypage
- **Fast R-CNN** : Nécessite MATLAB pour propositions

### 11.3 Datasets
- **PASCAL VOC** : 20 classes, ~20k images (bon pour débuter)
- **MS COCO** : 80 classes, plus d'objets par image
- **ImageNet Detection** : 200 classes, très grand

## Conclusion

La localisation et détection d'objets ont connu des progrès spectaculaires :
- 2013 : R-CNN révolutionne le domaine
- 2015 : Fast/Faster R-CNN atteignent temps réel
- Aujourd'hui : Réseaux profonds (ResNet) + techniques avancées

Les concepts clés à retenir :
- Régression pour localisation
- Propositions de régions pour réduire l'espace de recherche
- Partage de calcul (convolutions) pour l'efficacité
- ROI Pooling pour gérer différentes tailles
- RPN pour propositions apprises

La détection d'objets reste un domaine actif avec de nouveaux défis : segmentation d'instance, détection 3D, vidéo temps réel, etc.
# Architectures Avancées en Vision par Ordinateur : Analyse de Vidéos et Apprentissage Génératif Non Supervisé

## Partie I : Classification de Vidéos – Des Caractéristiques Manuelles aux Convolutions Spatio-Temporelles

### 1.0 Introduction à l'Analyse de Vidéos

L'analyse de vidéos représente une extension naturelle et considérablement plus complexe de la classification d'images. Elle constitue une frontière essentielle dans le domaine de la vision par ordinateur, visant à doter les machines de la capacité de comprendre non seulement le contenu statique d'une scène, mais aussi sa dynamique, ses actions et ses évolutions temporelles. Cette section introduit les concepts fondamentaux qui distinguent l'analyse de vidéos de celle des images, et expose les défis inhérents à cette discipline.

### 1.1 De l'Image à la Vidéo : La Dimension Temporelle

Le point de départ de la vision par ordinateur moderne est souvent la tâche de classification d'images, où un réseau de neurones convolutif (CNN) prend en entrée un tenseur représentant une image unique, par exemple de dimensions $32 \times 32 \times 3$ (hauteur, largeur, canaux de couleur), et produit une étiquette de classe.¹ Dans ce paradigme, l'information est purement spatiale.

Le passage à l'analyse de vidéos introduit une dimension supplémentaire fondamentale : le temps. Une vidéo n'est pas une image unique, mais une séquence d'images, ou "trames" (frames), capturées à des intervalles de temps réguliers. Par conséquent, l'objet de notre analyse n'est plus un tenseur 2D (plus la dimension des canaux), mais un volume de données 4D. Par exemple, une courte séquence de $T$ trames d'images $32 \times 32$ sera représentée par un tenseur de dimensions $32 \times 32 \times 3 \times T$.¹

L'objectif principal de la classification de vidéos est alors d'encoder et d'interpréter non seulement les motifs spatiaux présents dans chaque trame, mais aussi, et surtout, les motifs temporels qui émergent de la séquence de ces trames.¹ Il ne s'agit plus de reconnaître un "chat", mais de distinguer l'action d'un "chat qui saute" de celle d'un "chat qui dort". Cette distinction repose entièrement sur la compréhension de la dynamique temporelle. La prédiction de classe pour une trame donnée devient ainsi une fonction non seulement de cette trame, mais d'un contexte temporel local, typiquement une fenêtre de 15 à 30 trames, soit environ une demi-seconde à une seconde de vidéo.¹

### 1.2 Défis et Applications

L'ajout de la dimension temporelle, bien que conceptuellement simple, engendre une série de défis techniques et théoriques majeurs qui complexifient considérablement la tâche par rapport à l'analyse d'images statiques.

**Complexité et Coût de Calcul** : Les volumes de données vidéo sont intrinsèquement beaucoup plus grands que les images. Traiter un tenseur de dimensions $227 \times 227 \times 3 \times 15$ au lieu d'un tenseur $227 \times 227 \times 3$ augmente massivement le nombre d'opérations nécessaires, la mémoire requise et, par conséquent, le temps d'entraînement et d'inférence des modèles.¹

**Redondance de l'Information** : Les trames consécutives d'une vidéo sont souvent très similaires. Une grande partie de la scène (l'arrière-plan, les objets statiques) ne change pas, ou très peu. Un défi majeur est de concevoir des architectures capables d'ignorer cette information redondante et de se concentrer sur les changements pertinents, c'est-à-dire le mouvement.¹

**Modélisation Multi-Échelle** : Les dynamiques temporelles dans les vidéos se manifestent à différentes échelles. Un clin d'œil est un événement très court (quelques trames), tandis qu'une action complexe comme "préparer un repas" peut s'étendre sur plusieurs minutes. Les modèles doivent être capables de capturer à la fois les mouvements locaux et les dépendances à long terme pour une compréhension complète de la scène.¹

Malgré ces défis, les applications de l'analyse de vidéos sont vastes et en pleine expansion. Elles incluent la reconnaissance d'actions humaines (pour la surveillance, le sport, les interfaces homme-machine), l'analyse de scènes dynamiques (pour les véhicules autonomes), l'indexation et la recherche de contenu vidéo à grande échelle (pour les plateformes de streaming), et l'analyse médicale (par exemple, l'analyse de la démarche ou la détection d'événements dans des vidéos d'endoscopie).

### 2.0 L'Ère des Méthodes Basées sur les Caractéristiques : L'Ingénierie du Mouvement

Avant l'avènement des réseaux de neurones profonds et de l'apprentissage de bout en bout (end-to-end), la communauté de la vision par ordinateur abordait le problème de l'analyse vidéo avec une philosophie différente. Le paradigme dominant était celui de l'ingénierie manuelle de caractéristiques (feature engineering). Cette approche reposait sur la conviction que la clé de la compréhension du mouvement ne résidait pas dans l'apprentissage automatique à partir de données brutes, mais dans la conception explicite et experte de descripteurs mathématiques capables de capturer et de quantifier le mouvement de manière robuste. Ces méthodes n'apprenaient pas quoi chercher, mais étaient programmées pour comment suivre et décrire le mouvement. Cette distinction est fondamentale pour comprendre la révolution apportée par les approches modernes basées sur les CNNs. Parmi ces techniques classiques, les "trajectoires denses" se sont imposées comme l'une des plus performantes et influentes.¹

### 2.1 Les Trajectoires Denses (Dense Trajectories) : Suivre le Mouvement

Développées par Heng Wang et ses collaborateurs, les caractéristiques de trajectoires denses ont été l'état de l'art pour la reconnaissance d'actions juste avant la popularisation massive des CNNs.¹ L'idée centrale est de ne pas se contenter d'analyser des points d'intérêt isolés, mais de suivre le déplacement de nombreux points à travers la vidéo pour former des trajectoires, puis de décrire la scène le long de ces chemins de mouvement.¹ Le terme français consacré est "trajectoires denses".²

Le processus se décompose en plusieurs étapes clés :

1. **Détection de Points d'Intérêt** : La première étape consiste à identifier des points dans l'image qui sont "bons à suivre". Intuitivement, on cherche des points qui possèdent une texture riche ou des coins, et on évite les régions lisses ou uniformes où un suivi serait ambigu. Le système détecte un ensemble dense de ces points à plusieurs échelles spatiales pour assurer une bonne couverture de l'image.¹

2. **Suivi et Formation de "Tracklets"** : Une fois les points détectés, ils sont suivis d'une trame à l'autre en utilisant une méthode de flux optique. Ce suivi génère un grand nombre de courtes trajectoires, appelées tracklets, qui représentent le chemin d'un point sur une durée limitée, typiquement 15 trames (environ une demi-seconde).¹ Un tracklet est donc simplement une séquence de coordonnées $(x,y)$ sur 15 trames consécutives.¹

3. **Extraction de Descripteurs Locaux** : C'est l'étape cruciale. Au lieu d'extraire des caractéristiques à des positions fixes dans l'image, on les extrait dans un volume spatio-temporel centré sur chaque tracklet. Cela signifie que les descripteurs sont calculés dans le système de coordonnées local de l'objet en mouvement, ce qui les rend plus robustes aux variations de position. Plusieurs types de descripteurs sont calculés¹ :
   - **Trajectoire** : La forme de la trajectoire elle-même, encodant le déplacement.
   - **Histogram of Oriented Gradients (HOG)** : Un descripteur d'apparence qui capture la distribution des gradients d'intensité dans le volume, décrivant la forme statique autour de la trajectoire.
   - **Histogram of Optical Flow (HOF)** : Un descripteur de mouvement qui capture la distribution des vecteurs de flux optique, décrivant comment les pixels bougent dans le voisinage de la trajectoire.
   - **Motion Boundary Histograms (MBH)** : Un descripteur qui se concentre sur les gradients du flux optique, ce qui le rend particulièrement bon pour capturer les frontières de mouvement.

4. **Agrégation et Classification** : Tous ces descripteurs, extraits pour des milliers de tracklets, sont ensuite quantifiés et agrégés dans un grand vecteur de caractéristiques, souvent en utilisant une technique de type "Bag of Words". Ce vecteur final, qui représente la distribution des micro-mouvements et des apparences dans la vidéo, est enfin fourni à un classifieur standard comme une Machine à Vecteurs de Support (SVM) pour obtenir la prédiction finale.¹

### 2.2 Le Flux Optique (Optical Flow) : Le Champ de Mouvement

Le flux optique est la pierre angulaire de nombreuses méthodes classiques d'analyse vidéo, y compris les trajectoires denses. Le terme français est "flux optique".⁵ Il s'agit d'un champ de vecteurs 2D qui estime le mouvement apparent des objets, des surfaces et des bords dans une scène visuelle, causé par le mouvement relatif entre l'observateur (la caméra) et la scène.¹ Concrètement, pour chaque pixel de la trame $t$, le flux optique fournit un vecteur $(u,v)$ qui indique où ce pixel s'est déplacé dans la trame $t+1$.

La visualisation du flux optique est souvent réalisée en utilisant un code couleur : la teinte représente la direction du mouvement et la saturation représente sa magnitude. Par exemple, une zone entièrement jaune dans une visualisation de flux optique pourrait indiquer un mouvement de translation horizontal uniforme vers la droite.¹

Le calcul du flux optique est un problème en soi, complexe et bien étudié. De nombreux algorithmes existent, mais l'un des plus robustes et fréquemment utilisés dans la recherche est l'approche de Brox et al., particulièrement efficace pour gérer les grands déplacements de pixels entre les trames.¹ La disponibilité de mises en œuvre performantes de tels algorithmes a été un facteur clé du succès des méthodes basées sur les trajectoires.

**Implémentation en Pseudo-code PyTorch : Calcul du Flux Optique avec un Modèle Pré-entraîné**

Aujourd'hui, le calcul du flux optique peut également être effectué par des réseaux de neurones profonds. La bibliothèque torchvision propose des modèles pré-entraînés pour cette tâche. Voici un exemple de pseudo-code illustrant comment utiliser un tel modèle pour calculer le flux optique entre deux trames.

```python
# Importation des bibliothèques nécessaires
import torch
import torchvision.models.optical_flow as optical_flow
from torchvision.transforms.functional import to_tensor

# Charger un modèle de flux optique pré-entraîné (par exemple, RAFT)
# Les poids sont téléchargés automatiquement
weights = optical_flow.Raft_Large_Weights.DEFAULT
model = optical_flow.raft_large(weights=weights)
model.eval() # Mettre le modèle en mode évaluation

# Supposons que nous ayons deux images (trames) chargées avec PIL
# frame1_pil, frame2_pil

# Prétraitement des images :
# 1. Convertir en tenseurs
# 2. Normaliser les valeurs des pixels dans la plage attendue par le modèle
transforms = weights.transforms()
frame1_tensor = to_tensor(frame1_pil).unsqueeze(0) # Ajout d'une dimension de batch
frame2_tensor = to_tensor(frame2_pil).unsqueeze(0)

# Appliquer les transformations spécifiques au modèle (normalisation, etc.)
frame1_processed, frame2_processed = transforms(frame1_tensor, frame2_tensor)

# Calculer le flux optique
# Le modèle prend un batch d'images et retourne une liste de champs de flux
with torch.no_grad():
    list_of_flows = model(frame1_processed, frame2_processed)

# Le champ de flux prédit est le dernier élément de la liste de sortie
# Sa forme est (N, 2, H, W) où N=1, 2 pour les vecteurs (u, v)
predicted_flow = list_of_flows[-1]

# `predicted_flow` est maintenant un tenseur contenant le champ de vecteurs de mouvement
# de la trame 1 à la trame 2. Il peut être visualisé ou utilisé comme entrée
# pour une autre architecture.
print(f"Forme du tenseur de flux optique : {predicted_flow.shape}")
```

Ce pseudo-code montre la facilité avec laquelle on peut aujourd'hui intégrer une composante aussi puissante que le flux optique dans un pipeline d'apprentissage profond, une approche qui sera au cœur de certaines des architectures les plus performantes que nous allons maintenant aborder.

### 3.0 Réseaux de Neurones Convolutifs pour la Vidéo (CNNs)

L'immense succès des réseaux de neurones convolutifs (CNNs), à commencer par AlexNet, sur les tâches de classification d'images a naturellement conduit la communauté de recherche à se demander comment généraliser ces architectures puissantes au domaine de la vidéo. La question fondamentale était : comment adapter un modèle conçu pour l'information spatiale afin qu'il puisse également traiter et comprendre l'information temporelle?.¹ Plusieurs stratégies ont émergé, allant de modifications simples à des refontes architecturales complètes.

### 3.1 Extension des CNNs à la Vidéo : Les Convolutions 3D

L'approche la plus directe et conceptuellement la plus élégante pour généraliser les CNNs à la vidéo est d'étendre la notion même de convolution. Dans un CNN 2D, un filtre (ou noyau) est un petit patch de poids de dimensions $k_h \times k_w \times C_{in}$ (hauteur, largeur, canaux d'entrée) qui glisse sur les dimensions spatiales d'une image pour calculer une carte d'activation.

L'idée de la convolution 3D est d'ajouter une dimension temporelle au filtre. Le filtre devient un cube de poids de dimensions $k_t \times k_h \times k_w \times C_{in}$ (temps, hauteur, largeur, canaux d'entrée). Ce filtre 3D ne glisse plus seulement sur les axes $x$ et $y$ de l'entrée, mais aussi sur l'axe temporel $t$.¹ Le terme consacré en français est "convolution 3D".⁸

Ainsi, chaque neurone de la couche de convolution 3D est connecté à un petit volume spatio-temporel de la couche précédente. En glissant ce filtre sur les trois dimensions (temps, hauteur, largeur), on ne produit plus une "carte d'activation" 2D, mais un "volume d'activation" 3D.¹ Ce mécanisme permet au réseau d'apprendre des caractéristiques qui encodent simultanément l'apparence et le mouvement local. Tout comme les convolutions 2D bénéficient du partage de poids dans l'espace (un filtre détecteur de bord est utile partout dans l'image), les convolutions 3D bénéficient du partage de poids dans l'espace et dans le temps. Un filtre détecteur de mouvement est utile à différents moments de la vidéo.¹

### 3.2 Architecture "Slow Fusion" : Fusion Temporelle Progressive

Le terme "Slow Fusion" (fusion lente) a été popularisé par Karpathy et al. en 2014 pour décrire cette approche basée sur les convolutions 3D.¹ Le nom reflète l'idée que l'information temporelle n'est pas traitée d'un seul coup, mais est "fusionnée" progressivement, couche après couche, à travers le réseau. Tout comme un CNN 2D construit une hiérarchie de caractéristiques spatiales (pixels → bords → parties d'objets → objets), un CNN 3D construit une hiérarchie de caractéristiques spatio-temporelles.

La visualisation des filtres appris dans la première couche d'un tel réseau est particulièrement éclairante. Alors que les filtres d'un CNN 2D ressemblent souvent à des détecteurs de Gabor statiques ou à des patchs de couleur, les filtres d'un CNN 3D ressemblent à des "blobs en mouvement" ou à des détecteurs de bords qui se déplacent dans une direction spécifique au fil des trames temporelles du filtre. Certains filtres restent statiques, apprenant à détecter des caractéristiques d'apparence, tandis que d'autres se spécialisent dans la détection de mouvement de bas niveau. Cela confirme que le réseau apprend intrinsèquement des détecteurs de mouvement directement à partir des données de pixels bruts.¹

**Implémentation en Pseudo-code PyTorch : Définition d'une Couche de Convolution 3D**

En PyTorch, l'implémentation d'une couche de convolution 3D est simple grâce au module `torch.nn.Conv3d`. La principale différence réside dans la forme des tenseurs d'entrée.

```python
import torch
import torch.nn as nn

# Définition des paramètres
batch_size = 4
in_channels = 3  # Canaux RVB
num_frames = 16  # Nombre de trames dans le clip vidéo
height, width = 112, 112

# Création d'un tenseur d'entrée factice pour un clip vidéo
# La forme attendue par Conv3d est (N, C_in, T, H, W)
# N: taille du batch, C_in: canaux d'entrée, T: dimension temporelle (trames)
# H: hauteur, W: largeur
input_clip = torch.randn(batch_size, in_channels, num_frames, height, width)

# Définition d'une couche de convolution 3D
# out_channels: nombre de filtres (ou de canaux de sortie)
# kernel_size: taille du filtre (kt, kh, kw). Un tuple est attendu.
# stride: pas du filtre (st, sh, sw)
# padding: marge ajoutée (pt, ph, pw)
conv3d_layer = nn.Conv3d(in_channels=in_channels,
                           out_channels=64,
                           kernel_size=(3, 7, 7), # Filtre temporel de 3, spatial de 7x7
                           stride=(1, 2, 2),
                           padding=(1, 3, 3))

# Passage du clip à travers la couche
output_volume = conv3d_layer(input_clip)

# Le résultat est un volume d'activation
# La nouvelle taille est calculée en fonction du kernel, stride et padding
print(f"Forme du tenseur d'entrée : {input_clip.shape}")
print(f"Forme du volume de sortie : {output_volume.shape}")
# Sortie attendue : torch.Size([4, 64, 16, 56, 56])
# T reste 16 car kernel_size=3, padding=1, stride=1
# H et W sont divisés par 2 à cause du stride spatial de 2.
```

### 3.3 Architectures Alternatives : "Early Fusion" et "Late Fusion"

Bien que la convolution 3D soit l'approche la plus intégrée, d'autres stratégies ont été explorées pour combiner l'information temporelle, souvent avec des compromis en termes de complexité et de performance.

**Early Fusion (Fusion Précoce)** : Cette méthode consiste à prendre un bloc de trames (par exemple, 15 trames) et à les empiler le long de la dimension des canaux. Ainsi, un clip RVB de 15 trames devient une seule "super-image" avec $15 \times 3 = 45$ canaux. Un CNN 2D standard est ensuite appliqué à cette entrée. Le filtre de la première couche a donc une grande portée temporelle (il "voit" toutes les 15 trames à la fois), mais les couches suivantes redeviennent purement spatiales. L'information temporelle est donc "fusionnée" très tôt, dès la première couche.¹ Cette approche est plus simple à mettre en œuvre que les CNNs 3D mais peut avoir du mal à modéliser des dynamiques temporelles complexes.

**Late Fusion (Fusion Tardive)** : À l'opposé, la fusion tardive traite différentes trames de la vidéo de manière complètement indépendante. Par exemple, on peut prendre deux trames espacées de 10 trames, les faire passer chacune à travers un CNN 2D complet (comme AlexNet), et ne fusionner leurs vecteurs de caractéristiques qu'à la toute fin, au niveau des couches entièrement connectées. Cette approche modélise mal les interactions de mouvement local, mais peut être efficace si des vues statiques à différents moments sont suffisantes pour la classification.¹

### 3.4 L'Architecture à Deux Flux (Two-Stream Networks) : Séparer l'Apparence et le Mouvement

L'architecture à deux flux, proposée par Simonyan et Zisserman en 2014, est l'une des idées les plus influentes et les plus performantes dans le domaine de la classification de vidéos. Elle repose sur une observation fondamentale : l'information dans une vidéo peut être décomposée en deux modalités distinctes mais complémentaires : l'apparence (ce qui est présent dans la scène) et le mouvement (comment les choses se déplacent).¹

Tenter d'apprendre ces deux types d'informations simultanément à partir des pixels bruts, comme le fait un CNN 3D, est une tâche extrêmement difficile qui requiert des quantités massives de données. L'approche à deux flux propose une solution pragmatique : aider le réseau en lui fournissant explicitement une représentation du mouvement. C'est un exemple parfait de l'intégration de connaissances du domaine (l'importance du flux optique) dans une architecture d'apprentissage profond. Le terme français pour "two-stream" est "à deux flux".¹¹

L'architecture se compose de deux CNNs 2D distincts qui sont entraînés en parallèle¹ :

1. **Le Flux Spatial (Spatial Stream)** : Ce réseau est un CNN 2D standard (par exemple, de type VGG ou ResNet) qui prend en entrée une seule trame RVB de la vidéo. Son rôle est d'analyser la scène et de reconnaître les objets et le contexte statique. Il répond à la question "Qu'est-ce qu'il y a dans l'image?".

2. **Le Flux Temporel (Temporal Stream)** : Ce réseau est également un CNN 2D, mais son entrée est différente. Il ne prend pas de trames RVB, mais une pile de plusieurs champs de flux optique consécutifs (par exemple, 10 champs de flux, représentant le mouvement sur 10 trames). Le flux optique est pré-calculé en utilisant une méthode classique comme celle de Brox. Le rôle de ce flux est de reconnaître les actions et les dynamiques de mouvement. Il répond à la question "Comment les choses bougent-elles?".

Les prédictions des deux flux sont ensuite combinées à la fin du processus, généralement par une moyenne pondérée de leurs scores de classification (fusion tardive).¹

Les résultats expérimentaux de cette approche ont été remarquables. Non seulement la fusion des deux flux surpasse largement un flux unique, mais il a été observé que le flux temporel seul (basé sur le flux optique) obtenait souvent de meilleures performances que le flux spatial seul (basé sur les images RVB).¹ C'est une preuve éclatante que pour de nombreuses tâches de reconnaissance d'actions, le mouvement est un signal plus discriminant que l'apparence statique. Cela explique aussi pourquoi fournir explicitement le flux optique est si bénéfique : avec des ensembles de données de taille limitée, les CNNs peinent à apprendre des détecteurs de mouvement robustes à partir de zéro, et le pré-calcul du flux optique agit comme un puissant a priori qui guide l'apprentissage.¹

**Implémentation en Pseudo-code PyTorch : Structure d'un Réseau à Deux Flux**

Voici comment une telle architecture pourrait être structurée en PyTorch.

```python
import torch
import torch.nn as nn
import torchvision.models as models

# 1. Définition du Flux Spatial
class SpatialStream(nn.Module):
    def __init__(self, num_classes):
        super(SpatialStream, self).__init__()
        # Utiliser un modèle pré-entraîné sur ImageNet, comme ResNet-18
        self.cnn = models.resnet18(pretrained=True)
        # Remplacer la dernière couche pour l'adapter à notre tâche
        num_ftrs = self.cnn.fc.in_features
        self.cnn.fc = nn.Linear(num_ftrs, num_classes)

    def forward(self, x_rgb):
        # x_rgb est un tenseur de trames RVB de forme (N, 3, H, W)
        return self.cnn(x_rgb)

# 2. Définition du Flux Temporel
class TemporalStream(nn.Module):
    def __init__(self, num_classes, input_channels=20):
        super(TemporalStream, self).__init__()
        # Utiliser un ResNet-18, mais modifier la première couche pour accepter
        # une pile de flux optiques (par ex. 10 trames x 2 canaux (u,v) = 20)
        self.cnn = models.resnet18(pretrained=False) # Pas de pré-entraînement ici
        
        # Modification de la première couche convolutive
        self.cnn.conv1 = nn.Conv2d(input_channels, 64, kernel_size=7, stride=2, padding=3, bias=False)
        
        # Remplacer la dernière couche
        num_ftrs = self.cnn.fc.in_features
        self.cnn.fc = nn.Linear(num_ftrs, num_classes)

    def forward(self, x_flow):
        # x_flow est un tenseur de flux optiques empilés de forme (N, 20, H, W)
        return self.cnn(x_flow)

# 3. Combinaison des deux flux
class TwoStreamNet(nn.Module):
    def __init__(self, num_classes, flow_channels=20):
        super(TwoStreamNet, self).__init__()
        self.spatial_stream = SpatialStream(num_classes)
        self.temporal_stream = TemporalStream(num_classes, input_channels=flow_channels)

    def forward(self, x_rgb, x_flow):
        # Les deux flux sont traités indépendamment
        spatial_preds = self.spatial_stream(x_rgb)
        temporal_preds = self.temporal_stream(x_flow)

        # Fusion tardive par moyenne des prédictions (logits)
        # D'autres stratégies de fusion sont possibles (par ex. concaténation)
        final_preds = (spatial_preds + temporal_preds) / 2.0
        return final_preds

# Utilisation du modèle
# model = TwoStreamNet(num_classes=101)
# rgb_input = torch.randn(4, 3, 224, 224)
# flow_input = torch.randn(4, 20, 224, 224) # 10 trames de flux (u,v)
# output = model(rgb_input, flow_input)
# print(output.shape) # torch.Size([4, 101])
```

### 3.5 Le Modèle C3D : VGG pour la Vidéo

En parallèle du développement des architectures à deux flux, la recherche sur les CNNs 3D purs s'est poursuivie. Le modèle C3D (Convolutional 3D), proposé en 2015, en est un exemple marquant. Son architecture est une application directe de la philosophie de VGGNet au domaine de la 3D : la simplicité et l'homogénéité.¹

Le réseau C3D est construit en empilant des couches de convolution 3D avec de très petits filtres ($3 \times 3 \times 3$) et des couches de pooling 3D ($2 \times 2 \times 2$ ou parfois $1 \times 2 \times 2$ pour ne pas réduire trop vite la dimension temporelle). Cette conception simple mais profonde s'est avérée très efficace pour apprendre des caractéristiques spatio-temporelles et est rapidement devenue une base de référence populaire pour l'extraction de caractéristiques vidéo, souvent utilisée comme un "encodeur" pré-entraîné dans des tâches plus complexes.¹

### 4.0 Modélisation des Dépendances Temporelles à Long Terme

Les architectures basées sur les convolutions 3D (comme Slow Fusion ou C3D) et celles basées sur le flux optique (comme Two-Stream) ont considérablement amélioré la capacité des modèles à comprendre les mouvements locaux. Cependant, elles partagent une limitation fondamentale : leur champ réceptif temporel est intrinsèquement borné.

### 4.1 Limites des Convolutions Locales

Un filtre de convolution 3D, même dans un réseau profond, ne peut "voir" qu'une petite fenêtre temporelle de la vidéo, typiquement de l'ordre de quelques secondes au maximum. Ces modèles sont donc excellents pour reconnaître des actions courtes et atomiques, comme "lancer une balle" ou "applaudir". En revanche, ils sont mal équipés pour comprendre des activités complexes qui se déroulent sur de longues périodes et dont la signification dépend de l'ordre d'événements éloignés dans le temps. Par exemple, pour distinguer "ouvrir une porte pour entrer" de "ouvrir une porte pour sortir", il faut une mémoire des événements passés qui dépasse la portée d'un filtre local.¹

### 4.2 Intégration des Réseaux Récurrents (LSTM)

Pour surmonter cette limitation, les chercheurs se sont tournés vers une autre classe de modèles spécialisés dans le traitement de séquences : les Réseaux de Neurones Récurrents (RNNs), et plus particulièrement leur variante la plus populaire, le Long Short-Term Memory (LSTM). Les LSTMs sont conçus pour modéliser des dépendances à long terme grâce à un mécanisme de mémoire interne (l'état de la cellule) qui peut retenir des informations sur des périodes potentiellement infinies.¹

L'architecture hybride la plus courante, souvent appelée LRCN (Long-term Recurrent Convolutional Network), combine la puissance d'extraction de caractéristiques spatiales d'un CNN avec la capacité de modélisation temporelle d'un LSTM.¹⁴ Le pipeline est le suivant¹ :

1. Un CNN (souvent un CNN 2D pré-entraîné) est utilisé pour extraire un vecteur de caractéristiques compact pour chaque trame (ou chaque petit clip) de la vidéo. Cette étape transforme la vidéo en une séquence de vecteurs de caractéristiques.

2. Cette séquence de vecteurs est ensuite fournie en entrée d'un LSTM.

3. Le LSTM parcourt la séquence, mettant à jour son état interne à chaque pas de temps et agrégeant l'information temporelle. La sortie finale du LSTM (ou une combinaison de ses sorties à chaque pas de temps) est utilisée pour la classification.

Cette approche permet au modèle de baser sa prédiction sur l'ensemble de l'historique de la vidéo, lui donnant accès à un contexte temporel beaucoup plus large. Des variantes ont également été proposées où les deux flux d'une architecture Two-Stream alimentent un LSTM pour une modélisation encore plus riche.¹ Fait intéressant, l'idée de combiner des convolutions 3D pour le mouvement local et des LSTMs pour le mouvement global était déjà présente dans des travaux dès 2011, bien avant que ces concepts ne deviennent populaires.¹

### 4.3 Le Réseau Convolutif Récurrent (GRU-RCN) : Une Architecture Homogène

Bien que l'approche CNN+LSTM soit efficace, elle peut être perçue comme un assemblage hétérogène de deux composants fondamentalement différents. Le conférencier de la source décrit cette combinaison comme une "asymétrie laide" (ugly asymmetry).¹ D'un côté, nous avons des neurones convolutifs qui sont des processeurs spatiaux locaux, avec un champ réceptif limité. De l'autre, nous avons des neurones récurrents au sommet du réseau qui sont des processeurs séquentiels globaux, avec un champ réceptif temporel potentiellement infini. Cette séparation nette entre le traitement spatial et temporel manque d'élégance architecturale.

Une idée plus récente et conceptuellement "belle" et "propre" est de chercher à unifier ces deux concepts en une seule architecture homogène : le Réseau Convolutif Récurrent (Recurrent Convolutional Network).¹ L'idée fondamentale est de rendre chaque neurone convolutif lui-même récurrent.

Le mécanisme est le suivant : une couche convolutive récurrente, à un instant $t$, ne prend pas seulement en entrée la sortie de la couche précédente au même instant $t$, mais aussi sa propre sortie à l'instant précédent, $t-1$.¹ L'état de la couche à l'instant $t$, qui est un volume d'activation, est une fonction de l'entrée "d'en bas" (bottom-up) et de son propre état passé "à travers le temps" (across time).

Pour combiner ces deux sources d'information, on utilise une logique similaire à celle d'une cellule récurrente comme le GRU (Gated Recurrent Unit), qui est une version simplifiée du LSTM. Cependant, au lieu d'utiliser des multiplications de matrices pour mettre à jour l'état, on utilise des convolutions. Toutes les opérations matricielles du GRU sont remplacées par des opérations de convolution 2D.¹

L'avantage de cette approche, appelée GRU-RCN ou ConvGRU, est double :

1. **Homogénéité** : L'architecture est uniforme. Le traitement spatio-temporel est intrinsèque à chaque unité de calcul, plutôt que d'être géré par deux modules distincts. On peut simplement empiler ces couches récurrentes, tout comme on empile des couches convolutives classiques.¹

2. **Efficacité** : Elle n'utilise que des convolutions 2D, qui sont généralement plus optimisées et moins coûteuses que les convolutions 3D, tout en étant capable de modéliser des dépendances temporelles sur des durées potentiellement infinies à chaque niveau de la hiérarchie des caractéristiques spatiales.¹

Malgré son élégance conceptuelle, cette approche reste relativement récente et expérimentale, et n'a pas encore été adoptée aussi largement que les architectures plus établies.¹

**Implémentation en Pseudo-code PyTorch : Couche Convolutive Récurrente (ConvGRU)**

L'implémentation d'une telle couche nécessite de définir une "cellule" qui effectue l'opération récurrente pour un seul pas de temps.

```python
import torch
import torch.nn as nn

class ConvGRUCell(nn.Module):
    def __init__(self, input_dim, hidden_dim, kernel_size, bias=True):
        """
        Initialise la cellule ConvGRU.
        input_dim: Nombre de canaux dans le tenseur d'entrée.
        hidden_dim: Nombre de canaux dans l'état caché.
        kernel_size: Taille du noyau de convolution.
        """
        super(ConvGRUCell, self).__init__()
        self.hidden_dim = hidden_dim
        
        # Le padding est calculé pour conserver les dimensions spatiales
        padding = kernel_size // 2
        
        # Convolutions pour les portes de mise à jour (update) et de réinitialisation (reset)
        self.conv_gates = nn.Conv2d(in_channels=input_dim + hidden_dim,
                                    out_channels=2 * hidden_dim, # z_t et r_t
                                    kernel_size=kernel_size,
                                    padding=padding,
                                    bias=bias)
        
        # Convolution pour l'état candidat
        self.conv_candidate = nn.Conv2d(in_channels=input_dim + hidden_dim,
                                        out_channels=hidden_dim, # h_tilde_t
                                        kernel_size=kernel_size,
                                        padding=padding,
                                        bias=bias)

    def forward(self, x_t, h_prev):
        """
        Passe avant pour un pas de temps.
        x_t: Tenseur d'entrée au temps t, de forme (N, C_in, H, W).
        h_prev: État caché du temps t-1, de forme (N, C_hidden, H, W).
        """
        # Concaténer l'entrée et l'état caché précédent le long des canaux
        combined = torch.cat([x_t, h_prev], dim=1)
        
        # Calculer les portes de mise à jour et de réinitialisation
        gates = self.conv_gates(combined)
        z_t, r_t = torch.split(gates, self.hidden_dim, dim=1)
        
        # Appliquer les sigmoïdes
        z_t = torch.sigmoid(z_t)
        r_t = torch.sigmoid(r_t)
        
        # Calculer l'état candidat
        # La porte de réinitialisation est appliquée à l'état caché précédent
        combined_candidate = torch.cat([x_t, r_t * h_prev], dim=1)
        h_candidate = self.conv_candidate(combined_candidate)
        h_candidate = torch.tanh(h_candidate)
        
        # Calculer le nouvel état caché
        h_t = (1 - z_t) * h_prev + z_t * h_candidate
        
        return h_t

# Utilisation de la cellule
# cell = ConvGRUCell(input_dim=3, hidden_dim=64, kernel_size=3)
# input_tensor = torch.randn(4, 3, 32, 32)
# h_previous = torch.randn(4, 64, 32, 32)
# h_next = cell(input_tensor, h_previous)
# print(h_next.shape) # torch.Size([4, 64, 32, 32])
```

### 5.0 Synthèse et Considérations Pratiques

Après avoir exploré un large éventail d'architectures pour la classification de vidéos, il est essentiel de synthétiser ces connaissances en un guide pratique. Le choix de la bonne architecture dépend fortement de la nature de la tâche, des données disponibles et des ressources de calcul.

### 5.1 L'Importance Cruciale de la Ligne de Base (Single-Frame Baseline)

Le conseil le plus important et le plus pragmatique pour quiconque aborde un nouveau problème de classification vidéo est le suivant : commencez toujours par une ligne de base simple basée sur une seule trame (single-frame baseline).¹ Cette approche consiste à ignorer complètement la dimension temporelle et à entraîner un classifieur d'images standard sur des trames extraites de la vidéo.

La raison de cette recommandation est que, de manière contre-intuitive, l'information de mouvement n'est pas toujours le signal le plus important. Dans de nombreux cas, le contexte statique de la scène contient suffisamment d'informations pour distinguer les classes. Par exemple, pour différencier "jouer au tennis" de "nager", la présence d'un court de tennis de couleur terre battue ou d'une piscine remplie d'eau bleue est un indice beaucoup plus fort que les subtilités du mouvement d'un bras ou d'une jambe.¹

Des études ont montré que l'ajout de modèles complexes pour capturer le mouvement local (comme les CNNs 3D) peut n'apporter qu'un gain de performance marginal (par exemple, 1.6% d'amélioration de la précision) pour un coût de calcul et une complexité de mise en œuvre considérablement plus élevés.¹ Une ligne de base simple et solide est donc indispensable pour évaluer si l'effort supplémentaire de modélisation du mouvement est justifié.

### 5.2 Le Défi des Ensembles de Données Vidéo

Un autre facteur critique qui influence la performance des modèles vidéo est la disponibilité des données. Le domaine de la vidéo souffre d'un manque relatif de très grands ensembles de données, diversifiés et annotés, équivalents à ImageNet pour les images.¹ Ce manque de données à grande échelle rend le pré-entraînement de caractéristiques vidéo puissantes beaucoup plus difficile.

C'est une des raisons pour lesquelles les approches qui intègrent des connaissances a priori, comme l'architecture à deux flux qui utilise le flux optique, fonctionnent souvent très bien. Le réseau n'a pas à apprendre le concept de mouvement à partir de zéro, ce qui nécessiterait une quantité massive de données ; au lieu de cela, on lui fournit une représentation de mouvement explicite et de haute qualité. Le flux optique agit comme un signal de supervision intermédiaire qui simplifie la tâche d'apprentissage, en particulier lorsque les données sont limitées.¹

**Tableau Récapitulatif des Architectures Vidéo**

Pour faciliter le choix d'une architecture, le tableau suivant synthétise les caractéristiques, les forces et les faiblesses des différentes approches discutées. Il sert de guide de décision pratique pour les chercheurs et les ingénieurs.

| Architecture | Méthode de Fusion Temporelle | Complexité | Dépendances Temporelles | Cas d'Usage Idéal |
|--------------|------------------------------|-----------|------------------------|-------------------|
| Single-Frame | Aucune (Ligne de base) | Faible | Aucune | Tâches dominées par le contexte statique (ex: classification de scènes). Indispensable pour l'évaluation. |
| Early Fusion | Très précoce (canaux) | Moyenne | Très locales | Quand le mouvement à très court terme est un signal simple et suffisant. Prototypage rapide. |
| Late Fusion | Très tardive (scores/FC) | Élevée (multiples CNNs) | Aucune (indépendante) | Tâches où des vues temporelles distinctes et espacées sont plus informatives que le mouvement continu. |
| Slow Fusion (CNN 3D) | Progressive (couche par couche) | Très élevée | Locales à moyennes | Tâches où les motifs spatio-temporels locaux sont la clé (ex: détection de gestes). |
| Two-Stream | Tardive (scores/FC) | Très élevée (2 CNNs) | Locales (via flux optique) | Standard de facto pour la reconnaissance d'actions. Idéal quand le mouvement est un signal important. |
| CNN + LSTM | Séquentielle (après CNN) | Élevée | Long terme | Tâches avec une structure narrative ou des événements longs (ex: résumé de vidéo, analyse d'activités complexes). |
| GRU-RCN | Intrinsèque (à chaque couche) | Très élevée | Long terme (potentiellement) | Approche de recherche avancée, visant une modélisation spatio-temporelle unifiée et homogène. |

En résumé, pour tout projet vidéo, la démarche recommandée est la suivante :

1. Évaluer si le mouvement est réellement crucial pour la tâche.¹
2. Toujours implémenter et évaluer une ligne de base single-frame.¹
3. Si le mouvement est important, l'architecture à deux flux est un choix solide et éprouvé, surtout avec des données limitées.¹
4. Si des dépendances à long terme sont nécessaires, une architecture de type CNN+LSTM est la solution standard.
5. Les CNNs 3D et les RCNs sont des options puissantes mais plus coûteuses, à considérer pour des applications où les motifs spatio-temporels locaux fins sont primordiaux ou pour des recherches architecturales avancées.

## Partie II : Apprentissage Non Supervisé – Apprendre à Voir et à Générer

La première partie de ce cours s'est concentrée sur l'apprentissage supervisé, où le modèle apprend à partir de données explicitement étiquetées. Nous allons maintenant nous tourner vers un paradigme différent mais tout aussi fondamental : l'apprentissage non supervisé. Dans ce cadre, le modèle ne dispose que des données brutes, sans aucune étiquette, et son objectif est d'en découvrir la structure latente. C'est une tâche plus ouverte et souvent plus difficile, mais qui est au cœur de la quête d'une intelligence artificielle plus générale, capable d'apprendre sur le monde de manière autonome. Nous explorerons trois familles d'architectures qui ont défini ce domaine en apprentissage profond : les auto-encodeurs, les auto-encodeurs variationnels et les réseaux antagonistes génératifs.

### 6.0 Fondements de l'Apprentissage Non Supervisé

En apprentissage supervisé, nous avons un ensemble de données composé de paires $(X,Y)$, où $X$ est l'entrée (par exemple, une image) et $Y$ est la sortie désirée (une étiquette de classe). Le but est d'apprendre une fonction $f$ telle que $f(X) \approx Y$.¹

En apprentissage non supervisé, nous ne disposons que des données $X$. Il n'y a pas d'étiquettes $Y$ pour guider l'apprentissage.¹ L'objectif est donc plus vague : "faire quelque chose d'utile" avec ces données. Ce "quelque chose" consiste généralement à découvrir une structure sous-jacente ou "latente" dans les données.¹ Les exemples classiques incluent :

- **Le regroupement (Clustering)** : Comme avec l'algorithme K-means, l'objectif est de regrouper les données en clusters où les points d'un même cluster sont plus similaires entre eux qu'avec ceux des autres clusters.¹
- **La réduction de dimensionnalité** : Comme avec l'Analyse en Composantes Principales (ACP), l'objectif est de trouver une représentation des données dans un espace de plus faible dimension tout en préservant le maximum d'information.¹
- **L'apprentissage de caractéristiques (Feature Learning)** : L'objectif est d'apprendre une transformation des données brutes en une représentation (un ensemble de caractéristiques) plus utile pour des tâches ultérieures.
- **La modélisation de densité (Density Modeling) / Génération** : L'objectif est d'apprendre la distribution de probabilité des données, ce qui permet ensuite de générer de nouveaux échantillons qui ressemblent aux données originales.

C'est sur ces deux derniers objectifs, l'apprentissage de caractéristiques et la génération, que les modèles d'apprentissage profond non supervisés se sont principalement concentrés.

### 7.0 Les Auto-encodeurs (AE) : Apprendre par la Reconstruction

L'auto-encodeur (AE) est l'une des architectures non supervisées les plus simples et les plus fondamentales en apprentissage profond. Le terme français est "auto-encodeur".¹⁵ Son idée centrale est élégante : en l'absence d'étiquettes externes, nous pouvons créer notre propre tâche de supervision. La tâche la plus simple que l'on puisse imaginer est de demander au réseau de prédire son entrée, c'est-à-dire d'apprendre la fonction identité.¹

Cependant, apprendre simplement la fonction identité n'est pas très utile. Pour forcer le réseau à apprendre quelque chose d'intéressant sur la structure des données, on lui impose une contrainte : un goulot d'étranglement (bottleneck).

### 7.1 Principe : Encodeur, Espace Latent, Décodeur

Une architecture d'auto-encodeur a une forme de "sablier" et se compose de trois parties¹⁶ :

1. **L'Encodeur (Encoder)** : Cette partie du réseau prend l'entrée de haute dimension $X$ (par exemple, une image) et la comprime en une représentation de plus faible dimension, appelée code ou vecteur de l'espace latent $Z$.¹ L'espace des représentations compressées est appelé l'espace latent ("latent space" en anglais, "espace latent" en français¹⁷).

2. **Le Goulot d'Étranglement (Bottleneck) / Espace Latent** : C'est la couche cachée la plus petite du réseau. En forçant les données à passer par ce goulot d'étranglement, on oblige le réseau à ne conserver que les informations les plus importantes et à écarter le bruit ou les redondances. L'encodeur doit apprendre une compression efficace.¹

3. **Le Décodeur (Decoder)** : Cette partie du réseau prend le code de faible dimension $Z$ et tente de reconstruire l'entrée originale $X$. La sortie du décodeur, notée $\hat{X}$, doit être aussi proche que possible de $X$.¹

Le réseau est entraîné en minimisant une fonction de perte de reconstruction, qui mesure la différence entre l'entrée $X$ et la sortie reconstruite $\hat{X}$. La perte la plus courante est l'erreur quadratique moyenne (L2).¹

### 7.2 Utilisation pour le Pré-entraînement de Caractéristiques

L'objectif principal des auto-encodeurs traditionnels n'est pas la reconstruction en elle-même, qui est une tâche de substitution (surrogate task). Le véritable objectif est d'utiliser cette tâche pour apprendre de bonnes caractéristiques. L'hypothèse est que pour pouvoir compresser et décompresser efficacement une image, l'encodeur doit apprendre à extraire des caractéristiques sémantiques et structurelles pertinentes.

Une fois l'auto-encodeur entraîné sur un grand ensemble de données non étiquetées (par exemple, des millions d'images téléchargées sur internet), on peut jeter le décodeur et conserver uniquement l'encodeur.¹ Cet encodeur, qui sait maintenant comment transformer une image brute en un vecteur de caractéristiques compact et informatif, peut être utilisé comme une couche d'initialisation pour un réseau supervisé plus grand. On peut ensuite affiner (fine-tune) l'ensemble du nouveau réseau sur une petite quantité de données étiquetées.¹ C'était le grand espoir de l'apprentissage de caractéristiques non supervisé : tirer parti de l'abondance de données non étiquetées pour réduire le besoin de données étiquetées, qui sont coûteuses à obtenir.¹

Cependant, cette approche a des limites. Le conférencier note que, en pratique, cette méthode de pré-entraînement avec des auto-encodeurs traditionnels "ne fonctionne pas très bien".¹ Une des raisons principales est le désalignement entre la fonction de perte de reconstruction et l'objectif final de discrimination. Une perte L2 au niveau du pixel encourage le réseau à produire des reconstructions visuellement parfaites. Cela peut le pousser à se concentrer sur la mémorisation de textures et de détails de bas niveau, plutôt que sur l'apprentissage de concepts sémantiques abstraits (comme la forme d'un objet) qui sont pourtant plus utiles pour une tâche de classification. C'est une leçon fondamentale sur l'importance du choix de la tâche de substitution et de la fonction de perte en apprentissage non supervisé.

**Implémentation en Pseudo-code PyTorch : Auto-encodeur Convolutif Simple**

Voici un exemple d'auto-encodeur simple pour des images, utilisant des couches convolutives pour l'encodeur et des couches de convolution transposée pour le décodeur.

```python
import torch
import torch.nn as nn

class Autoencoder(nn.Module):
    def __init__(self):
        super(Autoencoder, self).__init__()
        
        # Encodeur
        # L'encodeur réduit progressivement les dimensions spatiales
        # et augmente le nombre de canaux.
        self.encoder = nn.Sequential(
            nn.Conv2d(1, 16, kernel_size=3, stride=2, padding=1), # Entrée: (N, 1, 28, 28) -> Sortie: (N, 16, 14, 14)
            nn.ReLU(),
            nn.Conv2d(16, 32, kernel_size=3, stride=2, padding=1), # Sortie: (N, 32, 7, 7)
            nn.ReLU(),
            nn.Conv2d(32, 64, kernel_size=7) # Sortie: (N, 64, 1, 1) -> Espace latent
        )
        
        # Décodeur
        # Le décodeur utilise la convolution transposée pour augmenter
        # les dimensions spatiales et reconstruire l'image.
        self.decoder = nn.Sequential(
            nn.ConvTranspose2d(64, 32, kernel_size=7), # Entrée: (N, 64, 1, 1) -> Sortie: (N, 32, 7, 7)
            nn.ReLU(),
            nn.ConvTranspose2d(32, 16, kernel_size=3, stride=2, padding=1, output_padding=1), # Sortie: (N, 16, 14, 14)
            nn.ReLU(),
            nn.ConvTranspose2d(16, 1, kernel_size=3, stride=2, padding=1, output_padding=1), # Sortie: (N, 1, 28, 28)
            nn.Sigmoid() # Sigmoïde pour ramener les valeurs des pixels entre 0 et 1
        )

    def forward(self, x):
        latent_code = self.encoder(x)
        reconstructed_x = self.decoder(latent_code)
        return reconstructed_x

# Boucle d'entraînement typique
# model = Autoencoder()
# criterion = nn.MSELoss() # Erreur quadratique moyenne comme perte de reconstruction
# optimizer = torch.optim.Adam(model.parameters(), lr=1e-3)

# for epoch in range(num_epochs):
#     for data in dataloader:
#         img, _ = data
#         img = img.view(img.size(0), -1) # Aplatir si nécessaire pour un AE non-convolutif
#         
#         # Passe avant
#         output = model(img)
#         loss = criterion(output, img) # La cible est l'image d'entrée elle-même
#         
#         # Rétropropagation
#         optimizer.zero_grad()
#         loss.backward()
#         optimizer.step()
```

### 8.0 Les Auto-encodeurs Variationnels (VAE) : Un Cadre Probabiliste pour la Génération

Les auto-encodeurs traditionnels sont des modèles déterministes : une entrée donnée produit toujours le même code latent. Bien qu'utiles pour l'apprentissage de caractéristiques, ils sont mal adaptés à la génération de nouvelles données. Si l'on choisit un point au hasard dans l'espace latent, il n'y a aucune garantie que le décodeur produise une image cohérente. L'espace latent appris par un AE peut être très discontinu et non structuré.

Les Auto-encodeurs Variationnels (VAE)¹⁷ résolvent ce problème en introduisant un cadre probabiliste rigoureux. Leur objectif n'est pas seulement de reconstruire, mais d'apprendre la distribution de probabilité des données, ce qui permet ensuite de générer de nouveaux échantillons crédibles.¹

La différence fondamentale réside dans la régularisation de l'espace latent. Un VAE force l'espace latent à être continu et structuré. Il y parvient en faisant en sorte que l'encodeur ne produise pas un point unique dans l'espace latent, mais une distribution de probabilité (généralement une gaussienne) sur cet espace. La fonction de perte est alors conçue pour encourager ces distributions à se chevaucher et à remplir l'espace de manière ordonnée. C'est cette continuité qui rend possibles des opérations comme l'interpolation (le "morphing") entre images et la génération d'échantillons nouveaux mais cohérents.

### 8.1 Le Processus Génératif Probabiliste

Le VAE est un modèle génératif qui suppose que les données ont été générées par un processus en deux étapes¹ :

1. Un vecteur latent $z$ est échantillonné à partir d'une distribution a priori simple, $P(z)$. Cette distribution a priori est généralement choisie comme une gaussienne standard (moyenne 0, variance 1), $P(z) \sim \mathcal{N}(0,I)$.¹

2. Une donnée $x$ est ensuite échantillonnée à partir d'une distribution conditionnelle $P(x|z)$, qui dépend du vecteur latent $z$. Cette distribution est modélisée par le réseau de neurones du décodeur. Le décodeur prend $z$ en entrée et produit les paramètres d'une distribution sur $x$ (par exemple, la moyenne $\mu_x$ et la variance $\sigma_x^2$ d'une distribution gaussienne).¹

Le problème est que pour entraîner un tel modèle, nous devons être capables d'inférer le $z$ latent qui a probablement généré une donnée observée $x$. Cela nécessite de calculer la distribution a posteriori $P(z|x)$. Malheureusement, le calcul de cette distribution via la règle de Bayes est insoluble car il implique une intégrale sur tout l'espace latent qui est intraitable.¹

La solution du VAE est d'introduire un second réseau de neurones, l'encodeur, noté $Q(z|x)$, pour approximer cette distribution a posteriori intraitable.¹ L'encodeur prend une donnée $x$ en entrée et, au lieu de produire un seul vecteur latent, il produit les paramètres (moyenne $\mu_z$ et variance $\sigma_z^2$) d'une distribution gaussienne dans l'espace latent.¹

### 8.2 La Fonction de Perte ELBO : Reconstruction et Régularisation

L'entraînement du VAE se fait en maximisant une quantité appelée "Evidence Lower Bound" (ELBO), qui est une borne inférieure de la log-vraisemblance des données. Maximiser l'ELBO revient à la fois à maximiser la probabilité que le modèle ait généré les données d'entraînement et à rendre l'approximation $Q(z|x)$ aussi proche que possible de la vraie distribution a posteriori $P(z|x)$.¹

La fonction de perte du VAE (qui est l'opposé de l'ELBO) se décompose magnifiquement en deux termes¹⁷ :

1. **Une perte de reconstruction** : Ce terme encourage le décodeur à reconstruire fidèlement l'entrée. Il est souvent mesuré comme l'erreur de reconstruction attendue (par exemple, une perte d'entropie croisée binaire ou une perte L2) après avoir échantillonné un $z$ à partir de la distribution $Q(z|x)$ produite par l'encodeur.¹

2. **Une perte de régularisation** : Ce terme est la Divergence de Kullback-Leibler (KL)²³ entre la distribution produite par l'encodeur, $Q(z|x)$, et la distribution a priori, $P(z)$. Mathématiquement, c'est $D_{KL}(Q(z|x) || P(z))$. Ce terme agit comme un régularisateur qui force l'encodeur à produire des distributions latentes qui restent proches de la gaussienne standard a priori. C'est cette contrainte qui organise l'espace latent et le rend continu.¹

La perte totale est donc : $L_{VAE} = L_{reconstruction} + \beta \cdot D_{KL}(Q(z|x) || P(z))$. Le coefficient $\beta$ permet de pondérer l'importance de la régularisation.

**Implémentation en Pseudo-code PyTorch : Structure d'un VAE et sa Fonction de Perte**

Un défi dans l'implémentation d'un VAE est que l'étape d'échantillonnage de $z$ à partir de $Q(z|x)$ est une opération stochastique qui bloque la rétropropagation du gradient. Le VAE résout ce problème avec l'astuce de reparamétrisation (reparameterization trick). Au lieu d'échantillonner $z \sim \mathcal{N}(\mu_z, \sigma_z^2)$, on échantillonne $\epsilon \sim \mathcal{N}(0,I)$ et on calcule $z = \mu_z + \sigma_z \cdot \epsilon$. Le caractère aléatoire est ainsi externalisé, et le gradient peut circuler à travers $\mu_z$ et $\sigma_z$.

```python
import torch
import torch.nn as nn
import torch.nn.functional as F

class VAE(nn.Module):
    def __init__(self, input_dim=784, hidden_dim=400, latent_dim=20):
        super(VAE, self).__init__()
        
        # Encodeur
        self.fc1 = nn.Linear(input_dim, hidden_dim)
        self.fc_mu = nn.Linear(hidden_dim, latent_dim) # Couche pour la moyenne
        self.fc_logvar = nn.Linear(hidden_dim, latent_dim) # Couche pour le log de la variance

        # Décodeur
        self.fc3 = nn.Linear(latent_dim, hidden_dim)
        self.fc4 = nn.Linear(hidden_dim, input_dim)

    def encode(self, x):
        h1 = F.relu(self.fc1(x))
        return self.fc_mu(h1), self.fc_logvar(h1)

    def reparameterize(self, mu, logvar):
        std = torch.exp(0.5 * logvar) # Calculer l'écart-type
        eps = torch.randn_like(std)  # Échantillonner epsilon ~ N(0, I)
        return mu + eps * std # Calculer z

    def decode(self, z):
        h3 = F.relu(self.fc3(z))
        return torch.sigmoid(self.fc4(h3))

    def forward(self, x):
        mu, logvar = self.encode(x.view(-1, 784))
        z = self.reparameterize(mu, logvar)
        return self.decode(z), mu, logvar

# Fonction de perte pour le VAE
def loss_function(recon_x, x, mu, logvar):
    # 1. Perte de reconstruction (Entropie croisée binaire)
    BCE = F.binary_cross_entropy(recon_x, x.view(-1, 784), reduction='sum')

    # 2. Perte de régularisation (Divergence KL)
    # D_KL(Q(z|x) || P(z)) où P(z) ~ N(0, I)
    # 0.5 * sum(1 + log(sigma^2) - mu^2 - sigma^2)
    KLD = -0.5 * torch.sum(1 + logvar - mu.pow(2) - logvar.exp())

    return BCE + KLD

# Boucle d'entraînement
# model = VAE()
# optimizer = torch.optim.Adam(model.parameters(), lr=1e-3)
# for epoch in range(num_epochs):
#     for batch_idx, (data, _) in enumerate(train_loader):
#         optimizer.zero_grad()
#         recon_batch, mu, logvar = model(data)
#         loss = loss_function(recon_batch, data, mu, logvar)
#         loss.backward()
#         optimizer.step()
```

### 9.0 Les Réseaux Antagonistes Génératifs (GAN) : La Compétition Créatrice

Introduits par Ian Goodfellow en 2014, les Réseaux Antagonistes Génératifs (GANs, ou RAG en français)²⁶ ont provoqué une révolution dans le domaine de l'apprentissage génératif. Ils proposent une approche radicalement différente pour entraîner des modèles génératifs, basée sur une théorie des jeux. Le saut conceptuel des GANs est monumental : au lieu d'utiliser une fonction de perte fixe et définie mathématiquement (comme L2 ou la log-vraisemblance), un GAN apprend sa propre fonction de perte.

Cette fonction de perte est elle-même un réseau de neurones, le discriminateur, dont le seul but est de devenir un expert pour distinguer les vraies données des fausses. Le générateur, à son tour, est entraîné pour tromper cet adversaire qui devient de plus en plus intelligent. Ce jeu à somme nulle pousse le générateur à produire des échantillons d'une qualité et d'un réalisme souvent bien supérieurs à ceux des VAEs, car il n'est pas contraint par une mesure de similarité au niveau du pixel, mais par la nécessité de paraître plausible à un expert.

### 9.1 Le Jeu Adversaire : Générateur vs. Discriminateur

Un GAN est composé de deux réseaux de neurones qui sont entraînés en compétition l'un contre l'autre²⁶ :

1. **Le Générateur (Generator, G)** : Ce réseau est le "faussaire". Il prend en entrée un vecteur de bruit aléatoire $z$ (généralement tiré d'une distribution gaussienne ou uniforme) et le transforme pour produire une "fausse" image $\hat{x} = G(z)$, qui est censée ressembler aux images de l'ensemble de données d'entraînement.¹ Sa structure est souvent similaire à celle d'un décodeur d'auto-encodeur.

2. **Le Discriminateur (Discriminator, D)** : Ce réseau est le "policier" ou "l'expert en art". C'est un classifieur binaire standard qui prend en entrée une image (soit une vraie image $x$ de l'ensemble de données, soit une fausse image $\hat{x}$ du générateur) et doit décider si elle est "réelle" (sortie proche de 1) ou "fausse" (sortie proche de 0).¹

L'entraînement est un processus itératif en deux étapes :

1. **Étape 1 : Entraîner le Discriminateur**. On présente au discriminateur un lot d'images contenant moitié de vraies images et moitié de fausses images générées par le générateur. On met à jour les poids du discriminateur pour qu'il améliore sa capacité à les distinguer correctement. Pendant cette étape, les poids du générateur sont gelés.

2. **Étape 2 : Entraîner le Générateur**. On génère un nouveau lot de fausses images et on les passe au discriminateur. Cette fois, on met à jour les poids du générateur pour qu'il maximise la probabilité que le discriminateur classe ses images comme "réelles". Pendant cette étape, les poids du discriminateur sont gelés.

Ce processus se poursuit jusqu'à ce que, idéalement, le générateur produise des images si réalistes que le discriminateur ne peut plus faire mieux que le hasard (précision de 50%).

### 9.2 Évolution : Des GANs Classiques aux DCGANs

Les premiers GANs, bien que conceptuellement brillants, étaient notoirement difficiles à entraîner et produisaient souvent des résultats de qualité médiocre sur des ensembles de données complexes comme CIFAR-10, générant des images floues et peu reconnaissables.¹

Une avancée majeure a été l'introduction des Deep Convolutional GANs (DCGANs).²⁷ Cette architecture n'a pas introduit de nouvelle théorie, mais a proposé un ensemble de bonnes pratiques architecturales qui ont considérablement stabilisé l'entraînement et amélioré la qualité des images générées¹ :

1. Remplacer toutes les couches de pooling par des convolutions à pas (strided convolutions) dans le discriminateur et des convolutions transposées fractionnaires (fractional-strided convolutions) dans le générateur.
2. Utiliser la normalisation par lots (Batch Normalization) dans le générateur et le discriminateur.
3. Éliminer les couches entièrement connectées.
4. Utiliser des activations ReLU dans le générateur (sauf pour la sortie qui utilise Tanh) et des LeakyReLU dans le discriminateur.

Ces modifications ont permis aux GANs de générer des images beaucoup plus détaillées et cohérentes, comme les célèbres exemples de chambres à coucher.¹

### 9.3 Arithmétique Vectorielle dans l'Espace Latent

Une propriété fascinante et émergente des GANs est qu'ils apprennent un espace latent sémantiquement riche et "désenchevêtré" (disentangled), même sans y être explicitement contraints par une perte de régularisation comme la divergence KL des VAEs.¹ Cela se manifeste de deux manières spectaculaires :

1. **Interpolation** : Si l'on prend deux vecteurs de bruit aléatoires $z_1$ et $z_2$, et que l'on génère des images pour des points le long de la ligne droite qui les relie dans l'espace latent, on observe une transition douce et sémantiquement cohérente entre l'image $G(z_1)$ et l'image $G(z_2)$. Par exemple, une chambre avec une grande fenêtre se transforme progressivement en une chambre avec une petite fenêtre, sans passer par un état intermédiaire flou ou incohérent.¹

2. **Arithmétique vectorielle** : Il est possible d'effectuer des opérations arithmétiques sur les vecteurs latents qui correspondent à des manipulations sémantiques sur les images générées. L'exemple le plus célèbre est :
   $$z(\text{"homme avec lunettes"}) - z(\text{"homme sans lunettes"}) + z(\text{"femme sans lunettes"}) \approx z(\text{"femme avec lunettes"})$$
   
   En effectuant cette opération sur les vecteurs latents moyens de chaque catégorie et en passant le résultat au générateur, on obtient bien une image de femme avec des lunettes.¹ Cela montre que le GAN a appris à associer des directions dans l'espace latent à des attributs sémantiques de haut niveau.

**Implémentation en Pseudo-code PyTorch : Définition du Générateur, du Discriminateur et de la Boucle d'Entraînement**

```python
import torch
import torch.nn as nn

# Définition du Générateur
class Generator(nn.Module):
    def __init__(self, latent_dim, channels_img, features_g):
        super(Generator, self).__init__()
        self.net = nn.Sequential(
            # Utilise des convolutions transposées pour passer du bruit latent à une image
            nn.ConvTranspose2d(latent_dim, features_g * 16, kernel_size=4, stride=1, padding=0),
            nn.BatchNorm2d(features_g * 16),
            nn.ReLU(),
            #... autres couches ConvTranspose2d...
            nn.ConvTranspose2d(features_g * 2, channels_img, kernel_size=4, stride=2, padding=1),
            nn.Tanh() # Normalise la sortie entre -1 et 1
        )
    def forward(self, x):
        return self.net(x)

# Définition du Discriminateur
class Discriminator(nn.Module):
    def __init__(self, channels_img, features_d):
        super(Discriminator, self).__init__()
        self.net = nn.Sequential(
            # Utilise des convolutions pour passer d'une image à un score de probabilité
            nn.Conv2d(channels_img, features_d, kernel_size=4, stride=2, padding=1),
            nn.LeakyReLU(0.2),
            #... autres couches Conv2d...
            nn.Conv2d(features_d * 8, 1, kernel_size=4, stride=2, padding=0),
            nn.Sigmoid()
        )
    def forward(self, x):
        return self.net(x)

# Boucle d'entraînement
# gen = Generator(...)
# disc = Discriminator(...)
# opt_gen = torch.optim.Adam(gen.parameters(), lr=lr, betas=(0.5, 0.999))
# opt_disc = torch.optim.Adam(disc.parameters(), lr=lr, betas=(0.5, 0.999))
# criterion = nn.BCELoss() # Perte d'entropie croisée binaire

# for epoch in range(num_epochs):
#     for batch_idx, (real, _) in enumerate(dataloader):
#         # --- Entraînement du Discriminateur ---
#         disc.zero_grad()
#         # 1. Sur les vraies images
#         real_output = disc(real).reshape(-1)
#         loss_disc_real = criterion(real_output, torch.ones_like(real_output))
#         # 2. Sur les fausses images
#         noise = torch.randn(batch_size, latent_dim, 1, 1)
#         fake = gen(noise)
#         fake_output = disc(fake.detach()).reshape(-1) #.detach() pour ne pas rétropropager dans G
#         loss_disc_fake = criterion(fake_output, torch.zeros_like(fake_output))
#         # Perte totale et mise à jour
#         loss_disc = (loss_disc_real + loss_disc_fake) / 2
#         loss_disc.backward()
#         opt_disc.step()

#         # --- Entraînement du Générateur ---
#         gen.zero_grad()
#         # On veut que le discriminateur classe les fausses images comme vraies
#         output = disc(fake).reshape(-1)
#         loss_gen = criterion(output, torch.ones_like(output))
#         loss_gen.backward()
#         opt_gen.step()
```

### 10.0 Architectures Hybrides et Conclusion

Le développement rapide des modèles génératifs a naturellement conduit à des architectures hybrides qui cherchent à combiner les forces des différentes approches. L'objectif est souvent de bénéficier de la structure probabiliste des VAEs tout en atteignant la qualité d'image des GANs.

### 10.1 Combiner VAEs et GANs pour des Échantillons de Haute Qualité

Une idée puissante consiste à construire une architecture VAE/GAN.¹ Dans ce cadre :

1. On conserve la structure globale d'un VAE (encodeur produisant une distribution latente, décodeur reconstruisant l'image). Cela permet de conserver un espace latent bien structuré et la capacité d'encoder des images réelles.

2. On remplace ou on complète la perte de reconstruction au niveau du pixel (comme L2 ou BCE) par une perte adversaire. On ajoute un discriminateur qui est entraîné à distinguer les images reconstruites par le VAE des vraies images. Le décodeur du VAE (qui est aussi le générateur) est alors entraîné non seulement à reconstruire l'entrée, mais aussi à tromper ce discriminateur.

3. De plus, pour s'assurer que les images générées sont non seulement réalistes mais aussi sémantiquement similaires aux originales, on peut ajouter une perte perceptuelle. Au lieu de comparer les images au niveau des pixels, on les passe à travers un réseau de classification pré-entraîné (comme VGG ou AlexNet) et on calcule la distance (par exemple, L2) entre leurs activations à une ou plusieurs couches intermédiaires. Cela force le générateur à produire des images qui ont des caractéristiques de haut niveau similaires, ce qui conduit à des résultats visuellement plus plaisants.¹

La combinaison de ces trois pertes (reconstruction VAE, adversaire GAN, et perceptuelle) a permis de générer des échantillons de très haute qualité, même sur des ensembles de données aussi complexes qu'ImageNet.¹

### 10.2 Synthèse : Quel Modèle pour Quelle Tâche?

- **Auto-encodeur (AE)** : Principalement utile pour la compression et l'apprentissage de caractéristiques non linéaires. Son utilisation pour le pré-entraînement est devenue moins courante, mais il reste un concept fondamental.

- **Auto-encodeur Variationnel (VAE)** : Le meilleur choix lorsque l'on a besoin d'un espace latent structuré et probabiliste. Idéal pour la génération contrôlée, l'interpolation sémantique (morphing), la détection d'anomalies (une mauvaise reconstruction signale une anomalie), et l'apprentissage semi-supervisé. Les échantillons peuvent être un peu flous par rapport aux GANs.

- **Réseau Antagoniste Génératif (GAN)** : Le champion incontesté pour la génération d'images de haute qualité et de réalisme photographique. Privilégié pour des tâches comme la super-résolution, la traduction d'image à image (StyleGAN, CycleGAN), la retouche d'image, et la création artistique. L'entraînement peut être instable et il est plus difficile d'encoder une image existante dans son espace latent.

### 10.3 Conclusion : L'Avenir de l'Apprentissage Génératif en Vision

L'apprentissage non supervisé, et en particulier les modèles génératifs, est l'un des domaines les plus dynamiques de l'intelligence artificielle. Ces modèles ne sont pas seulement des outils pour générer de belles images ; ils sont aussi une fenêtre sur la manière dont les réseaux de neurones "perçoivent" et "comprennent" le monde visuel. En apprenant à générer des données, ils doivent implicitement apprendre les règles, les structures et les concepts qui régissent notre monde.

Les VAEs et les GANs ont jeté les bases, mais le domaine continue d'évoluer à une vitesse fulgurante. Des architectures plus récentes, comme les modèles de diffusion, ont encore repoussé les limites de la qualité et du contrôle de la génération. La capacité à synthétiser des données réalistes a des implications profondes pour la création de contenu, la simulation, l'augmentation de données et, plus fondamentalement, pour notre quête d'une intelligence artificielle capable d'apprendre et de créer de manière autonome.
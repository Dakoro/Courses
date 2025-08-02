# Les Réseaux de Neurones Récurrents (RNN) et LSTM

## Table des matières
1. [Introduction aux RNN](#introduction)
2. [Architecture et Formulation Mathématique](#architecture)
3. [Modèles de Langage au Niveau des Caractères](#modeles-langage)
4. [Implémentation Pratique](#implementation)
5. [Applications Avancées](#applications)
6. [RNN Multi-couches](#rnn-multicouches)
7. [Long Short-Term Memory (LSTM)](#lstm)
8. [Le Problème du Gradient Évanescent](#gradient-evanescent)
9. [Gated Recurrent Units (GRU)](#gru)
10. [Conclusions et Perspectives](#conclusions)

---

## 1. Introduction aux RNN {#introduction}

### 1.1 Flexibilité Architecturale

Les réseaux de neurones récurrents offrent une flexibilité remarquable dans le traitement des séquences, contrairement aux réseaux feedforward traditionnels :

- **Un-à-un** : Entrée fixe → Sortie fixe (réseaux classiques)
- **Un-à-plusieurs** : Image → Séquence de mots (image captioning)
- **Plusieurs-à-un** : Séquence de mots → Sentiment (classification)
- **Plusieurs-à-plusieurs** : Séquence anglaise → Séquence française (traduction)
- **Plusieurs-à-plusieurs synchrone** : Vidéo → Classification frame par frame

### 1.2 Applications Créatives

Même pour des entrées/sorties fixes, les RNN permettent un traitement séquentiel :
- **Lecture de numéros** : Un petit CNN se déplace sur l'image pour lire séquentiellement
- **DRAW** : Génération d'images en "peignant" progressivement sur un canvas

### 1.3 Concept Fondamental

Un RNN maintient un **état interne** qui évolue au fil du temps en fonction :
- Des entrées reçues à chaque pas de temps
- De l'état précédent
- Des poids appris

---

## 2. Architecture et Formulation Mathématique {#architecture}

### 2.1 Formulation Générale

À chaque pas de temps t, le RNN calcule :

```
h_t = f_W(h_{t-1}, x_t)
```

Où :
- `h_t` : État caché au temps t
- `x_t` : Entrée au temps t
- `f_W` : Fonction de récurrence avec paramètres W
- La même fonction f_W est appliquée à chaque pas de temps

### 2.2 RNN Vanilla

La forme la plus simple utilise :

```
h_t = tanh(W_hh · h_{t-1} + W_xh · x_t + b_h)
y_t = W_hy · h_t + b_y
```

### 2.3 Pseudo-code de la Passe Avant

```python
def rnn_forward(x_sequence, W_xh, W_hh, W_hy, b_h, b_y, h0):
    """
    x_sequence : Liste de T vecteurs d'entrée
    W_xh : Matrice poids entrée→caché
    W_hh : Matrice poids caché→caché
    W_hy : Matrice poids caché→sortie
    h0 : État initial (souvent zéros)
    """
    T = len(x_sequence)
    h = h0
    outputs = []
    hidden_states = [h0]
    
    for t in range(T):
        # Mise à jour de l'état caché
        h = tanh(dot(W_xh, x_sequence[t]) + dot(W_hh, h) + b_h)
        hidden_states.append(h)
        
        # Calcul de la sortie
        y = dot(W_hy, h) + b_y
        outputs.append(y)
    
    return outputs, hidden_states
```

---

## 3. Modèles de Langage au Niveau des Caractères {#modeles-langage}

### 3.1 Principe

Un modèle de langage prédit le prochain caractère d'une séquence. C'est une excellente introduction aux RNN car :
- Conceptuellement simple
- Visuellement intéressant
- Pratiquement utile

### 3.2 Exemple : Entraînement sur "hello"

**Vocabulaire** : {h, e, l, o} → 4 caractères

**Encodage one-hot** :
- h = [1,0,0,0]
- e = [0,1,0,0]
- l = [0,0,1,0]
- o = [0,0,0,1]

**Processus** :
1. Entrée 'h' → Prédire 'e'
2. Entrée 'e' → Prédire 'l'
3. Entrée 'l' → Prédire 'l'
4. Entrée 'l' → Prédire 'o'

### 3.3 Architecture Détaillée

```python
def char_rnn_step(char_input, h_prev, W_xh, W_hh, W_hy, b_h, b_y):
    """
    Un pas du RNN de caractères
    
    char_input : Vecteur one-hot du caractère actuel
    h_prev : État caché précédent
    """
    # Mise à jour de l'état caché
    h = tanh(dot(W_xh, char_input) + dot(W_hh, h_prev) + b_h)
    
    # Prédiction du prochain caractère (logits)
    y = dot(W_hy, h) + b_y
    
    # Probabilités via softmax
    probs = softmax(y)
    
    return probs, h
```

---

## 4. Implémentation Pratique {#implementation}

### 4.1 Structure du Code (100 lignes NumPy)

```python
import numpy as np

# Hyperparamètres
hidden_size = 100
seq_length = 25
learning_rate = 1e-1

# Lecture des données
data = open('input.txt', 'r').read()
chars = list(set(data))
vocab_size = len(chars)

# Mappings caractère ↔ indice
char_to_ix = {ch:i for i,ch in enumerate(chars)}
ix_to_char = {i:ch for i,ch in enumerate(chars)}

# Initialisation des poids
W_xh = np.random.randn(hidden_size, vocab_size) * 0.01
W_hh = np.random.randn(hidden_size, hidden_size) * 0.01
W_hy = np.random.randn(vocab_size, hidden_size) * 0.01
b_h = np.zeros((hidden_size, 1))
b_y = np.zeros((vocab_size, 1))
```

### 4.2 Fonction de Perte avec Rétropropagation

```python
def loss_function(inputs, targets, h_prev):
    """
    inputs, targets : listes d'indices de caractères
    h_prev : état caché du batch précédent
    """
    xs, hs, ys, ps = {}, {}, {}, {}
    hs[-1] = np.copy(h_prev)
    loss = 0
    
    # Passe avant
    for t in range(len(inputs)):
        xs[t] = np.zeros((vocab_size, 1))
        xs[t][inputs[t]] = 1  # One-hot
        
        # État caché
        hs[t] = np.tanh(np.dot(W_xh, xs[t]) + 
                       np.dot(W_hh, hs[t-1]) + b_h)
        
        # Sortie
        ys[t] = np.dot(W_hy, hs[t]) + b_y
        ps[t] = np.exp(ys[t]) / np.sum(np.exp(ys[t]))
        
        # Perte cross-entropy
        loss += -np.log(ps[t][targets[t], 0])
    
    # Passe arrière
    dW_xh = np.zeros_like(W_xh)
    dW_hh = np.zeros_like(W_hh)
    dW_hy = np.zeros_like(W_hy)
    db_h = np.zeros_like(b_h)
    db_y = np.zeros_like(b_y)
    dh_next = np.zeros_like(hs[0])
    
    for t in reversed(range(len(inputs))):
        # Gradient de la sortie
        dy = np.copy(ps[t])
        dy[targets[t]] -= 1
        
        # Gradients des poids de sortie
        dW_hy += np.dot(dy, hs[t].T)
        db_y += dy
        
        # Gradient de l'état caché
        dh = np.dot(W_hy.T, dy) + dh_next
        dh_raw = (1 - hs[t] * hs[t]) * dh  # Dérivée de tanh
        
        # Gradients des poids cachés
        db_h += dh_raw
        dW_xh += np.dot(dh_raw, xs[t].T)
        dW_hh += np.dot(dh_raw, hs[t-1].T)
        
        # Gradient pour le pas précédent
        dh_next = np.dot(W_hh.T, dh_raw)
    
    # Clipping des gradients
    for dparam in [dW_xh, dW_hh, dW_hy, db_h, db_y]:
        np.clip(dparam, -5, 5, out=dparam)
    
    return loss, dW_xh, dW_hh, dW_hy, db_h, db_y, hs[len(inputs)-1]
```

### 4.3 Génération de Texte

```python
def sample(h, seed_ix, n):
    """
    Génère n caractères à partir d'un caractère initial
    
    h : État caché initial
    seed_ix : Indice du caractère de départ
    n : Nombre de caractères à générer
    """
    x = np.zeros((vocab_size, 1))
    x[seed_ix] = 1
    generated_text = []
    
    for t in range(n):
        h = np.tanh(np.dot(W_xh, x) + np.dot(W_hh, h) + b_h)
        y = np.dot(W_hy, h) + b_y
        p = np.exp(y) / np.sum(np.exp(y))
        
        # Échantillonnage
        ix = np.random.choice(range(vocab_size), p=p.ravel())
        
        # Préparation pour le prochain pas
        x = np.zeros((vocab_size, 1))
        x[ix] = 1
        generated_text.append(ix_to_char[ix])
    
    return ''.join(generated_text)
```

---

## 5. Applications Avancées {#applications}

### 5.1 Génération de Texte Shakespeare

Après entraînement sur les œuvres complètes :

**Début** : Caractères aléatoires
**Milieu** : Mots courts, structure émergente
**Fin** : 
```
"Alas, I think he shall be come approached and the day
When little srain would be attained into being never fed,
And who is but the chain and subjects of his death,
I should not sleep."
```

### 5.2 Génération de Code C (Linux)

```c
/* 
 * Increment the size file of the new incorrect UI_FILTER group information
 * of the size generatively.
 */
static int indicate_policy(void)
{
  int error;
  if (fd == MARN_EPT) {
    /*
     * The kernel blank will coeld it to userspace.
     */
    if (ss->segment < mem_total)
      unblock_graph_and_set_blocked();
    else
      ret = 1;
    goto bail;
  }
  segaddr = in_SB(in.addr);
  selector = seg / 16;
}
```

### 5.3 LaTeX Mathématique

Génère des preuves, lemmes, et même des diagrammes !

### 5.4 Image Captioning

Architecture combinant CNN et RNN :

```python
def caption_model_forward(image, W_cnn, W_ih, W_xh, W_hh, W_hy):
    """
    Génère une légende pour une image
    
    image : Image d'entrée
    W_cnn : Poids du CNN (pré-entraîné)
    W_ih : Poids image→RNN
    """
    # 1. Extraction de features avec CNN
    cnn_features = cnn_forward(image, W_cnn)  # Ex: 4096-dim
    
    # 2. Initialisation du RNN
    h0 = tanh(dot(W_ih, cnn_features))
    
    # 3. Token de début
    x = START_TOKEN
    h = h0
    caption = []
    
    # 4. Génération séquentielle
    while True:
        h = tanh(dot(W_xh, x) + dot(W_hh, h))
        y = softmax(dot(W_hy, h))
        
        word = sample_from_distribution(y)
        if word == END_TOKEN:
            break
            
        caption.append(word)
        x = word_embedding(word)
    
    return caption
```

---

## 6. RNN Multi-couches {#rnn-multicouches}

### 6.1 Architecture

Empilement de RNN où chaque couche prend en entrée les états cachés de la couche précédente :

```
Couche 3: →[h³₁]→[h³₂]→[h³₃]→
           ↑     ↑     ↑
Couche 2: →[h²₁]→[h²₂]→[h²₃]→
           ↑     ↑     ↑
Couche 1: →[h¹₁]→[h¹₂]→[h¹₃]→
           ↑     ↑     ↑
Entrée:    x₁    x₂    x₃
```

### 6.2 Formulation

Pour la couche l au temps t :
```
h^l_t = tanh(W^l · [h^(l-1)_t; h^l_(t-1)])
```

Où `[a; b]` représente la concaténation de a et b.

---

## 7. Long Short-Term Memory (LSTM) {#lstm}

### 7.1 Motivation

Les RNN vanilla souffrent du problème du gradient évanescent. Les LSTM résolvent ce problème avec une architecture plus complexe.

### 7.2 Architecture LSTM

Un LSTM maintient deux vecteurs d'état :
- **h_t** : État caché (hidden state)
- **c_t** : État de cellule (cell state)

### 7.3 Équations LSTM

```python
def lstm_step(x_t, h_prev, c_prev, W):
    """
    Un pas de LSTM
    
    x_t : Entrée au temps t
    h_prev, c_prev : États précédents
    W : Tous les poids concaténés
    """
    # Concaténation entrée et état caché
    concat = concatenate([x_t, h_prev])
    
    # Calcul des 4 portes
    gates = dot(W, concat)
    
    # Découpage et activation
    i = sigmoid(gates[0:n])      # Porte d'entrée
    f = sigmoid(gates[n:2n])     # Porte d'oubli
    o = sigmoid(gates[2n:3n])    # Porte de sortie
    g = tanh(gates[3n:4n])       # Candidat
    
    # Mise à jour de l'état de cellule
    c_t = f * c_prev + i * g
    
    # Mise à jour de l'état caché
    h_t = o * tanh(c_t)
    
    return h_t, c_t
```

### 7.4 Intuition des Portes

- **Porte d'oubli (f)** : Décide quelle information oublier
- **Porte d'entrée (i)** : Décide quelle nouvelle information stocker
- **Candidat (g)** : Nouvelle information potentielle
- **Porte de sortie (o)** : Décide quelle partie de l'état révéler

### 7.5 Visualisation du Flux

```
c_{t-1} →[×f]→[+]→ c_t
              ↑
           [×i·g]
              
h_{t-1} →[LSTM]→ h_t
    ↑      ↓
   x_t    y_t
```

### 7.6 Implémentation Complète

```python
class LSTM:
    def __init__(self, input_dim, hidden_dim):
        # Initialisation des poids
        self.Wf = random_init(hidden_dim, input_dim + hidden_dim)
        self.Wi = random_init(hidden_dim, input_dim + hidden_dim)
        self.Wo = random_init(hidden_dim, input_dim + hidden_dim)
        self.Wg = random_init(hidden_dim, input_dim + hidden_dim)
        
        self.bf = zeros(hidden_dim)
        self.bi = zeros(hidden_dim)
        self.bo = zeros(hidden_dim)
        self.bg = zeros(hidden_dim)
    
    def forward(self, x_sequence, h0=None, c0=None):
        T = len(x_sequence)
        hidden_dim = self.Wf.shape[0]
        
        # Initialisation
        if h0 is None:
            h0 = zeros(hidden_dim)
        if c0 is None:
            c0 = zeros(hidden_dim)
        
        h, c = h0, c0
        outputs = []
        
        for t in range(T):
            # Concaténation
            concat = concatenate([x_sequence[t], h])
            
            # Portes
            f = sigmoid(dot(self.Wf, concat) + self.bf)
            i = sigmoid(dot(self.Wi, concat) + self.bi)
            o = sigmoid(dot(self.Wo, concat) + self.bo)
            g = tanh(dot(self.Wg, concat) + self.bg)
            
            # Mise à jour
            c = f * c + i * g
            h = o * tanh(c)
            
            outputs.append(h)
        
        return outputs
```

---

## 8. Le Problème du Gradient Évanescent {#gradient-evanescent}

### 8.1 Diagnostic

Dans un RNN vanilla, lors de la rétropropagation à travers T pas de temps :

```
∂L/∂h_0 = ∂L/∂h_T · ∏(t=1 to T) W_hh^T · diag(f'(h_t))
```

Le gradient est multiplié par W_hh à chaque pas !

### 8.2 Démonstration Empirique

```python
def demonstrate_vanishing_gradient():
    # RNN simple
    W_hh = np.random.randn(100, 100) * 0.5
    h = np.random.randn(100, 1)
    
    # Propagation avant sur 50 pas
    activations = [h]
    for t in range(50):
        h = np.tanh(np.dot(W_hh, h))
        activations.append(h)
    
    # Rétropropagation
    dh = np.random.randn(100, 1)  # Gradient initial
    gradients = [dh]
    
    for t in reversed(range(50)):
        # Dérivée de tanh
        dh = dh * (1 - activations[t+1]**2)
        # Multiplication par W_hh^T
        dh = np.dot(W_hh.T, dh)
        gradients.append(np.linalg.norm(dh))
    
    # Les gradients décroissent exponentiellement !
    return gradients
```

### 8.3 Analyse Mathématique

Si λ est la plus grande valeur propre de W_hh :
- **λ > 1** : Explosion du gradient
- **λ < 1** : Évanouissement du gradient
- **λ = 1** : Cas idéal (rare)

### 8.4 Solution LSTM

Les LSTM résolvent ce problème avec leur "autoroute de gradient" :

```
c_t = f_t * c_{t-1} + i_t * g_t
```

Le gradient peut circuler directement via l'addition, évitant les multiplications répétées.

---

## 9. Gated Recurrent Units (GRU) {#gru}

### 9.1 Motivation

Les GRU sont une simplification des LSTM avec des performances similaires.

### 9.2 Équations GRU

```python
def gru_step(x_t, h_prev, W_z, W_r, W_h):
    """
    Un pas de GRU
    """
    # Porte de mise à jour
    z = sigmoid(dot(W_z, concatenate([x_t, h_prev])))
    
    # Porte de réinitialisation
    r = sigmoid(dot(W_r, concatenate([x_t, h_prev])))
    
    # Candidat
    h_tilde = tanh(dot(W_h, concatenate([x_t, r * h_prev])))
    
    # Mise à jour finale
    h_t = (1 - z) * h_prev + z * h_tilde
    
    return h_t
```

### 9.3 Avantages du GRU

- Plus simple que LSTM (3 portes vs 4)
- Un seul état (h) au lieu de deux (h, c)
- Performance comparable
- Plus rapide à entraîner

---

## 10. Conclusions et Perspectives {#conclusions}

### 10.1 Résumé des Points Clés

1. **RNN Vanilla** : Simple mais problème de gradient évanescent
2. **LSTM** : Solution robuste, standard de l'industrie
3. **GRU** : Alternative simplifiée, souvent suffisante
4. **Applications** : Génération de texte, traduction, captioning

### 10.2 Bonnes Pratiques

```python
# Configuration typique
config = {
    'cell_type': 'LSTM',  # ou 'GRU'
    'hidden_size': 128,
    'num_layers': 2,
    'dropout': 0.2,
    'gradient_clipping': 5.0,
    'sequence_length': 100,  # Truncated BPTT
    'learning_rate': 0.001,
    'optimizer': 'Adam'
}
```

### 10.3 Directions de Recherche

1. **Connexion avec ResNet** : Comprendre les liens théoriques
2. **Architectures hybrides** : Transformer, attention mechanisms
3. **Efficacité computationnelle** : Réduction de la complexité
4. **Interprétabilité** : Comprendre les représentations apprises

### 10.4 Outils et Ressources

- **char-rnn** : Implémentation simple en NumPy
- **torch-rnn** : Version optimisée GPU
- **Frameworks modernes** : PyTorch, TensorFlow, JAX

Les RNN restent fondamentaux pour comprendre le traitement séquentiel, même si les Transformers dominent maintenant de nombreuses applications.
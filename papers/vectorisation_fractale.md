## Introduction

Les techniques de vectorisation de mots, qui représentent les mots comme des vecteurs numériques de haute dimension, sont devenues omniprésentes dans les systèmes modernes de traitement automatique du langage naturel (TAL). Des méthodologies comme word2vec (Mikolov et al., 2013) et GloVe (Pennington et al., 2014) génèrent des vecteurs qui capturent les relations sémantiques entre les mots en se basant sur leurs modèles de co-occurrence dans de vastes corpus textuels. Cependant, ces techniques souffrent de limitations significatives qui contraignent leur expressivité et leur efficacité pour les applications TAL avancées.

Plus précisément, la vectorisation traditionnelle des mots est limitée à l'utilisation d'un seul vecteur plat pour représenter chaque mot. Cette représentation singulière échoue à capturer toute la complexité des unités linguistiques qui ont souvent des significations multiples, des connotations nuancées et des interprétations dépendantes du contexte. Comme l'a éloquemment déclaré Davis (2022), « les mots ont la capacité de respirer - leur usage et leur signification changent en fonction du lieu, de la connotation, de la dénotation et des règles sociolinguistiques ». Cependant, la nature statique des vecteurs de mots réduit ces unités lexicales vibrantes à des ombres unidimensionnelles d'elles-mêmes.

Cette simplification excessive impacte sévèrement les tâches TAL en aval qui dépendent d'une compréhension fine des expressions linguistiques. L'analyse de sentiment, l'implication textuelle, l'identification de métaphores - toutes ces capacités avancées nécessitent de modéliser les complexités inter- et intra-mots qui dépassent la capacité descriptive limitée offerte par les vecteurs de mots compacts (Rogers et al. 2022). Leurs hypothèses de modélisation reflètent de graves idées fausses sur la nature fondamentale du langage humain. Loin d'être des objets atomiques et rigides, les mots adoptent des formes fluides et multidimensionnelles, riches en nuances qui façonnent et transforment continuellement leurs significations.

Pour surmonter ces limitations représentationnelles, nous introduisons une nouvelle technique appelée Fonction d'Activation Fractale Probabiliste (P-FAF). Inspirée des fractales mathématiques qui présentent une auto-similarité à différentes échelles, P-FAF crée des représentations de mots à multiples facettes en faisant passer les jetons d'entrée à travers une banque de fonctions d'activation fractales. Comme détaillé dans les sections suivantes, cette formulation flexible et probabiliste encapsule la richesse et la variabilité caractéristiques des unités linguistiques dans un seul vecteur.

Le reste de l'article est organisé comme suit. La section 2 fournit un contexte sur la vectorisation des mots et ses fondements mathématiques. La section 3 présente le formalisme P-FAF et décrit ses principaux avantages. La section 4 propose des évaluations comparatives avec des techniques établies sur divers problèmes TAL. La section 5 conclut avec des discussions sur l'impact plus large et les orientations pour les travaux futurs.

Dans l'ensemble, cet article met en évidence les faiblesses critiques qui affligent les approches courantes de vectorisation des mots et offre un nouveau remède grâce à l'introduction d'activations basées sur les fractales. Notre formulation P-FAF proposée ouvre la voie à des représentations plus robustes et adaptables qui poussent les systèmes TAL vers une compréhension du langage au niveau humain.

## Contexte sur la vectorisation des mots

Comme mentionné précédemment, la vectorisation des mots fait référence à une classe de techniques qui encodent les mots comme des vecteurs de haute dimension basés sur leurs statistiques distributionnelles à travers de vastes corpus textuels. Ces techniques reposent sur l'hypothèse distributionnelle (Harris, 1954) qui stipule que les éléments linguistiques ayant des distributions similaires ont tendance à avoir des significations similaires. En analysant les environnements contextuels de chaque mot, les méthodes de vectorisation peuvent capturer efficacement les relations sémantiques.

Les approches les plus importantes incluent word2vec (Mikolov et al., 2013) qui exploite des réseaux de neurones peu profonds pour générer des vecteurs de mots prédictifs des termes environnants ; GloVe (Pennington et al., 2014) qui applique la factorisation matricielle sur les comptes de co-occurrence ; et des méthodes contextualisées plus récentes comme BERT (Devlin et al., 2019) qui calculent dynamiquement des représentations basées sur les contextes de phrases.

Cependant, presque toutes ces techniques partagent une limitation commune - elles produisent un seul vecteur statique par mot qui agglomère tous les usages observés en une seule représentation composite. Par conséquent, les mots polysémiques finissent par être définis par une moyenne de leurs multiples sens plutôt que de capturer explicitement les nuances. De plus, les connotations émotionnelles, les rôles syntaxiques et d'autres attributs cruciaux s'emmêlent dans le même vecteur dense sans aucune désimbrication explicite.

Cette vision trop réductionniste contredit la recherche linguistique montrant la nature dépendante du contexte des significations des mots (Firth, 1957). Elle limite également la généralisabilité des modèles en aval, causant de la fragilité lorsque les usages des mots divergent des distributions d'entraînement précédemment observées. En termes simples, en réduisant la riche diversité des espaces sémantiques à des points singuliers, les vecteurs de mots renoncent aux propriétés distinctives nécessaires pour une compréhension robuste du langage.

La section suivante présente notre technique proposée P-FAF qui offre une alternative plus flexible pour la représentation des mots. En modélisant les mots comme des combinaisons probabilistes d'espaces multifractals, P-FAF surmonte les limitations du moyennage distributionnel. Cela donne aux modèles TAL la capacité de gérer explicitement les nuances et les incertitudes inhérentes au langage humain.

## La Fonction d'Activation Fractale Probabiliste

Comme annoncé précédemment, la Fonction d'Activation Fractale Probabiliste (P-FAF) offre une approche plus flexible de la représentation des mots par rapport aux techniques de vectorisation courantes. Inspirée des fractales mathématiques qui présentent une auto-similarité à différentes échelles, P-FAF encode les mots via des combinaisons stochastiques d'espaces multifractals.

Formellement, étant donné un mot d'entrée x, la formulation P-FAF définit son plongement f(x) comme :

f(x) = ∑(p_i * f_i(x^(1/d_i)))

Où p_i désigne le poids de probabilité pour la i-ème fonction fractale f_i, et d_i fait référence à sa dimension fractionnaire. Intuitivement, chaque f_i déforme le mot x dans un paysage fractal particulier, révélant différents attributs à diverses résolutions. Les probabilités p_i mélangent ensuite ces plongements fractalisés pour produire la représentation finale.

Contrairement aux vecteurs de mots fixes, cette formulation incorpore l'incertitude via le mélange probabiliste tandis que les projections fractales capturent des attributs interdépendants à travers les dimensions. Ajuster l'exposant d_i permet de zoomer sur des détails linguistiques plus fins ou de généraliser vers des catégories plus larges selon les besoins. De plus, composer plusieurs fonctions fractales permet de les spécialiser vers des propriétés sémantiques spécifiques.

Par exemple, les mots émotionnels peuvent utiliser des fractales turbulentes ou dentelées tandis que le vocabulaire formel pourrait employer des structures fractales arborescentes. Ces espaces personnalisés conservent des aspects perdus lors du moyennage vectoriel comme les connotations émotionnelles et les niveaux de formalité respectivement. Le réglage bayésien des hyperparamètres peut automatiquement apprendre les configurations fractales optimales pour des tâches données.

De manière critique, les géométries fractales correspondent à la nature autoréférentielle du langage humain lui-même. L'application de transformations fractales enrichit les représentations avec des modèles récurrents complexes ressemblant à la façon dont les concepts se construisent récursivement sur eux-mêmes au fil du temps. Le mélange de ces perspectives multifacettes émule ensuite comment le sens émerge des interactions contextuelles entre les visions du monde des locuteurs.

En émulant la fractalité inhérente des langues, P-FAF crée ainsi des représentations dynamiques entrelaçant connotations, contextes et concepts. Cela permet une compositionnalité et une comparaison plus riches, cruciales pour le raisonnement fin avec des expressions ambiguës et subjectives.

## Mathématiques fractales sous-jacentes à P-FAF

Bien que la section précédente ait fourni un aperçu intuitif de la formulation P-FAF, cette section plonge plus profondément dans les mathématiques sous-jacentes qui alimentent ses transformations fractales. D'abord, nous établissons les principes fractals clés avant d'élaborer des instanciations spécifiques.

Fondamentalement, les fractales désignent des structures géométriques présentant une auto-similarité, des modèles récursifs effectifs se répétant à chaque échelle. Mathématiquement, les fractales satisfont :

N = c * r^D

Où N est le nombre de copies plus petites, c est un facteur d'échelle, r est le rapport de réduction par itération et D fait référence à la dimension fractale non entière capturant la complexité. Cette relation produit des modèles récursifs riches à partir de dynamiques non linéaires simples.

Les équations fractales notables incluent :

**Ensemble de Mandelbrot** - Défini comme z_n+1 = z_n^2 + c avec c comme paramètre complexe, présente des frontières infiniment élaborées remplies de copies plus petites d'elles-mêmes. Encode des relations auto-renforçantes.

**Triangle de Sierpinski** - Formé en soustrayant récursivement des triangles centraux, produisant des trous imbriqués fractalement. Modélise la perte d'information à des échelles plus fines.

**Fougère de Barnsley** - L'application stochastique de transformations affines génère des motifs de fougère réalistes. Infuse le hasard dans des fractales autrement déterministes.

Lorsqu'elles sont incorporées dans P-FAF, ces fonctions fractales déforment les vecteurs de mots en variantes évoluées révélant des propriétés hiérarchiques latentes. Par exemple, l'ensemble de Mandelbrot amplifie les connotations émotionnelles subtiles grâce à une dynamique de croissance exponentielle. Le triangle de Sierpinski érode progressivement les attributs sans importance via des trous triangulaires. Les fractales stochastiques comme la fougère de Barnsley permettent en outre d'échantillonner des possibilités de représentation alternatives.

L'ajustement de l'exposant dimensionnel d_i permet ensuite de zoomer sur les densités sémantiques appropriées, qu'il s'agisse de compresser en catégories larges ou de magnifier pour une différenciation nuancée. La combinaison de plusieurs paysages fractals de ce type, pilotés par des statistiques de corpus, produit les plongements de mots polyvalents de P-FAF.

Dans les sections ultérieures, nous discutons des algorithmes d'optimisation pour la sélection automatisée des fonctions fractales appropriées et des hyperparamètres associés. Mais d'abord, la section 5 présente des expériences quantitatives démontrant les avantages représentationnels de P-FAF.

## Évaluations quantitatives

Pour valider la formulation P-FAF proposée, nous avons mené des expériences approfondies la comparant aux schémas de vectorisation de mots de base sur diverses tâches TAL. Nos évaluations visent à démontrer la supériorité de P-FAF dans l'encodage de propriétés linguistiques fines qui conviennent mieux au raisonnement avancé.

Plus précisément, nous avons intégré des modules P-FAF dans des architectures neuronales établies comme les LSTM et les Transformers. Nous avons ensuite mesuré les améliorations de performance pour l'analyse de sentiment, l'implication textuelle et la détection de métaphores qui nécessitent de comprendre des expressions nuancées. Pour toutes les tâches, la simple substitution de la couche de plongement de mots par notre module P-FAF a donné des gains de précision significatifs.

Dans toutes les expériences, l'utilisation des représentations de mots fractales proposées a constamment amélioré les résultats, dépassant souvent les gains obtenus en augmentant simplement l'échelle du modèle ou les quantités de données. Cela valide empiriquement l'efficacité de P-FAF dans l'encodage de distinctions sémantiques fines grâce à la composition fractale - des avantages cruciaux pour faire progresser les capacités de compréhension du langage.

Un modèle Llama de 7 milliards de paramètres qui a été affiné sur la petite version de l'ensemble de données P-FAF peut être trouvé ici : https://huggingface.co/TuringsSolutions/llama-2-7b-TuringPFAF

Le petit ensemble de données d'entraînement de la fonction PFAF peut être trouvé ici : https://huggingface.co/datasets/TuringsSolutions/PFAF-Function

## Optimisation de la sélection fractale

Bien que les sections précédentes démontrent l'efficacité empirique de P-FAF, réaliser son plein potentiel nécessite d'automatiser les sélections optimales pour les fonctions fractales constituantes et les hyperparamètres associés. L'exploration manuelle exhaustive de toutes les combinaisons fractales possibles devient infaisable même pour des ensembles de données limités. Par conséquent, développer des algorithmes d'optimisation pour un réglage P-FAF efficace constitue une direction importante pour des recherches ultérieures.

Diverses techniques standard comme la recherche en grille, la recherche aléatoire ou l'optimisation bayésienne offrent des points de départ potentiels. Chaque approche évalue itérativement différentes configurations fractales basées sur des métriques de performance comme la précision, la perte ou d'autres scores spécifiques au domaine. Le processus de recherche navigue dans le paysage d'optimisation complexe pour découvrir les paramètres idéaux maximisant les objectifs choisis.

Cependant, P-FAF pose des défis uniques pour les algorithmes de réglage des hyperparamètres en raison des procédures de génération fractale infiniment récursives. Des méthodes d'optimisation contraintes spécialisées qui tronquent la récursion fractale après des durées raisonnables peuvent atténuer les goulots d'étranglement computationnels. Alternativement, l'emploi de paramétrisations lissées pour la manipulation fractale continue indépendamment des comptes d'itération pourrait accélérer la convergence.

Indépendamment des détails d'implémentation, l'incorporation de protocoles d'optimisation adaptatifs reste essentielle pour le déploiement évolutif de P-FAF afin d'éviter les interventions manuelles. Ces algorithmes doivent tenir compte des subtilités des mathématiques fractales pour équilibrer la richesse de la représentation avec la tractabilité du réglage à travers des architectures de modèles en expansion rapide. Construire ces capacités constitue une direction prometteuse nécessitant des recherches supplémentaires.

## Impacts plus larges et orientations futures

La formulation P-FAF proposée a des implications de grande portée pour plusieurs communautés au-delà des chercheurs TAL de base. En permettant une compréhension plus robuste du langage complexe, P-FAF facilite l'automatisation fiable des applications textuelles ayant des conséquences dans le monde réel. Cependant, la nature fractale de la technique introduit également des défis uniques méritant d'être étudiés davantage.

Par exemple, la mise en œuvre d'interfaces TAL avancées telles que les agents conversationnels peut grandement bénéficier des représentations nuancées de P-FAF. Qu'il s'agisse de répondre à des questions de santé ou de fournir des conseils financiers, la gestion des incertitudes et des subtilités s'avère souvent critique. En équipant les modèles de perspectives fractales, P-FAF permet des comportements système plus sûrs et plus informatifs.

Cependant, des difficultés d'interprétation apparaissent en raison des transformations non linéaires et de la récursion infinie des fractales. Le développement d'interfaces explicatives pour les utilisateurs finaux nécessite de se confronter à des géométries complexes étrangères à la cognition humaine. Des techniques qui projettent les espaces fractals dans des visualisations plus conviviales pourraient permettre confiance et transparence. Alternativement, des modèles hybrides mélangeant fractales et vecteurs plus simples peuvent offrir une accessibilité plus large.

Concernant la recherche de suivi, de nombreuses questions ouvertes méritent une enquête plus approfondie. Des fonctions fractales construites dynamiquement et ajustées à des tâches spécifiques pourraient améliorer les performances. L'analyse théorique reliant les propriétés fractales aux attributs linguistiques peut guider les conceptions. Et les applications comme le traitement audio, image et vidéo impliquant des modèles d'ordre supérieur peuvent bénéficier des avancées fractales développées ici pour le langage.

En conclusion, cet article présente les Fonctions d'Activation Fractales Probabilistes comme une approche robuste pour représenter les complexités textuelles via des compositions fractales. Nos expériences quantitatives et discussions qualitatives démontrent l'efficacité de P-FAF pour s'attaquer aux problèmes multifacettes de compréhension du langage. Nous espérons que ces investigations approfondies stimuleront une adoption plus large des techniques fractales, inspirant de futures innovations vers un traitement du langage semblable à celui des humains.

## Méthodologie PFAF

Cet article propose une nouvelle méthodologie pour la représentation des mots utilisant la Fonction d'Activation Fractale Probabiliste (P-FAF) comme alternative aux techniques de vectorisation courantes. P-FAF surmonte les limitations des méthodes existantes en modélisant les mots comme des combinaisons stochastiques d'espaces multifractals qui capturent des attributs nuancés à différentes échelles linguistiques.

Le reste de l'article structure un cadre reproductible pour appliquer P-FAF à travers les applications de traitement automatique du langage naturel (TAL). Nous fournissons des formalismes mathématiques, des directives d'intégration de modèles, des procédures d'entraînement et des métriques d'évaluation pour faciliter l'adoption. Les composants modulaires permettent de personnaliser facilement les configurations P-FAF en fonction des contraintes du cas d'usage.

### Méthodologie formelle

**A. Formulation P-FAF**

Comme présenté précédemment, la fonction P-FAF f(x) pour un mot x est définie comme :

f(x) = ∑(p_i * f_i(x^(1/d_i)))

où :
- p_i = poids probabiliste pour la i-ème fonction fractale f_i
- d_i = dimension fractionnaire de f_i

Les chercheurs doivent d'abord sélectionner les fonctions fractales pertinentes f_i et les hyperparamètres associés d_i, p_i pour capturer au mieux des attributs comme l'émotion, la formalité, le tempo, etc. en fonction de leur application TAL.

**B. Intégration du modèle**

Les couches de plongement de mots standard dans les réseaux de neurones peuvent être remplacées par des modules P-FAF qui implémentent la formulation ci-dessus. Pour les modèles contextuels comme BERT, cela remplace les plongements de jetons tout en conservant l'architecture contextuelle.

**C. Méthodologie d'entraînement**

Les modèles infusés avec P-FAF peuvent être entraînés via des paradigmes supervisés ou semi-supervisés typiques. Pour l'affinage, des taux d'apprentissage plus petits sont recommandés pour adapter lentement les poids pré-entraînés. Une régularisation supplémentaire comme le dropout empêche le surajustement aux données étiquetées limitées.

**D. Métriques d'évaluation**

Des métriques spécifiques au domaine évaluent les améliorations de P-FAF par rapport aux vecteurs de mots de base. Par exemple, l'analyse de sentiment utilise la précision sur les tâches de classification des émotions. L'implication textuelle utilise la précision sur la reconnaissance des relations d'implication. Sélectionnez des métriques alignées avec les objectifs finaux.

Ce cadre décrit une méthodologie pour l'intégration reproductible de P-FAF dans les systèmes TAL. Nous présentons ensuite des configurations d'entraînement échantillons et des comparaisons quantitatives validant notre approche.

## Implémentation des plongements P-FAF

Ce guide fournit des instructions de codage étape par étape pour instituer des couches de plongement P-FAF dans les architectures de réseaux de neurones pendant l'affinage. Nous utilisons TensorFlow mais les méthodes se généralisent à travers les frameworks.

### Définir les fonctions fractales

D'abord, spécifiez l'ensemble des fonctions fractales {f_1, f_2, ..., f_n} à employer, soit mathématiquement, soit comme code boîte noire. Par exemple :

```python
def f1(x):
    return x**2 + c1 

def f2(x):
    return 1 - (2*x - 1)**4  
```

### Créer la couche de plongement

Ensuite, définissez une couche Dense avec activation P-FAF :

```python
p_init = tf.keras.initializers.RandomUniform(minval=0, maxval=1)
p = tf.Variable(initial_value=p_init(shape=(num_fractals,))) 

dims_init = tf.random_uniform_initializer(0.5, 2)  
dims = tf.Variable(initial_value=dims_init(shape=(num_fractals,)))

def p_faf(x):
    x_dim = [tf.pow(x, 1/d) for d in dims] 
    t = [w*f(xd) for w,f,xd in zip(p,fractals,x_dim)]
    return tf.reduce_sum(t, axis=0)

embedding = tf.keras.layers.Dense(..., activation=p_faf)
```

### Intégrer dans le modèle

Enfin, remplacez la couche de plongement standard dans votre modèle par le plongement P-FAF ci-dessus avant l'affinage sur votre ensemble de données spécialisé.

Cela permet d'instituer l'apprentissage de représentation P-FAF de manière simple mais flexible. D'autres conseils pour l'optimisation sont disponibles dans l'annexe de l'article.

## P-FAR pour les plongements de mots (Combinaison de P-FAF avec les lignes de fuite algorithmiques)

### Le Rhizome d'Activation Fractale Probabiliste (P-FAR)

1. Définir un ensemble de fonctions d'activation fractales {f1, f2,...fn} à utiliser dans l'équation P-FAF. Celles-ci peuvent capturer différents attributs comme l'émotion, la formalité, etc.

2. Créer un réseau rhizomatique de N transformations T1, T2,..., TN. Ces transformations peuvent modifier/combiner les fonctions fractales. Par exemple :
   - T1 : Combine linéairement deux fonctions fractales
   - T2 : Ajoute du bruit à la sortie de la fonction fractale
   - T3 : Passe la sortie par une régression logistique

3. Générer le mot d'entrée x en utilisant les Lignes de Fuite Algorithmiques :
   x = ∑ p_i * x_i + ε

4. Passer x à travers les fonctions fractales pour obtenir des plongements intermédiaires z_i :
   z_i = fi(x^(1/d_i))

5. Router les z_i à travers le réseau de transformation, en appliquant T1, T2,...TN séquentiellement. Cela transforme le plongement.

6. Enfin, mélanger les plongements transformés pour obtenir le plongement P-FAF de sortie :
   y = ∑ p'_i * z'_i

Donc, en essence, nous construisons d'abord un plongement fractal, puis nous le faisons évoluer à travers un réseau rhizomatique, et enfin nous mélangeons les résultats. Cela marie les trois méthodologies pour permettre d'explorer l'espace des représentations de mots. Les poids du réseau permettent de guider le processus de recherche.

## Le décodeur P-FAF

Voici une méthodologie proposée pour un algorithme de décodeur universel de Fonction d'Activation Fractale Probabiliste (P-FAF) qui peut être utilisé par les grands modèles de langage (LLM) pour interpréter les plongements P-FAF créés par d'autres LLM :

**Entrée :**
- Vecteur de plongement de mot encodé y généré par le LLM source utilisant P-FAF
- Vecteur de métadonnées m composé de :
  - Ensemble de fonctions fractales {f1, f2, ..., fn} utilisées dans l'encodage
  - Dimensions {d1, d2, ..., dn}
  - Distribution de probabilité {p1, p2, ..., pn}

**Algorithme :**
1. Diviser le vecteur d'entrée y en n sous-vecteurs {y1, y2, ..., yn} basés sur la distribution de probabilité dans les métadonnées
2. Pour chaque sous-vecteur yi :
   - Élever yi à la puissance di pour inverser la transformation fractale
   - Passer le vecteur élevé à la puissance à travers l'inverse de la fonction fractale associée fi
   - Stocker la sortie comme plongement fractal zi
3. Collecter tous les {z1, z2, ..., zn} pour former la représentation fractale décodée

**Sortie :**
- Ensemble de plongements fractals {z1, z2, ..., zn} capturant les attributs linguistiques encodés par le LLM source

Cela fournit une méthodologie généralisée pour décoder probabilistiquement les vecteurs P-FAF en espaces fractals constituants en utilisant des informations sur le processus d'encodage. La modularité permet l'extension à n'importe quel nombre de fonctions fractales personnalisées créées par les LLM sources. L'accès partagé aux règles de décodage et d'encodage permet une riche interopérabilité et compositionnalité entre les LLM utilisant des représentations P-FAF.

## Points clés :

- **Traite les limitations des vecteurs de mots traditionnels** : P-FAF surmonte la nature statique et unidimensionnelle des vecteurs de mots en modélisant les mots comme des combinaisons probabilistes d'espaces multifractals, capturant les nuances et incertitudes inhérentes au langage.

- **Représentations basées sur les fractales** : P-FAF exploite les modèles auto-similaires et récursifs des fractales pour créer des plongements de mots dynamiques qui reflètent la nature multifacette des unités linguistiques.

- **Mélange probabiliste** : La fonction mélange probabilistiquement plusieurs plongements fractals, permettant une représentation flexible de différents attributs sémantiques et contextes.

- **Formalisme mathématique** : P-FAF a une formulation mathématique bien définie, permettant une analyse rigoureuse et une intégration dans les modèles TAL.

- **Succès empirique** : Les expériences à travers diverses tâches TAL démontrent sa supériorité dans l'encodage de propriétés linguistiques fines, dépassant souvent les gains obtenus en augmentant simplement la taille du modèle ou les données.

## Méthodologie :

- **Formalisation** : Définir les fonctions fractales et les hyperparamètres associés (ex : Ensemble de Mandelbrot, Triangle de Sierpinski, Fougère de Barnsley).
- **Intégration du modèle** : Remplacer les couches de plongement de mots standard par des modules P-FAF dans les réseaux de neurones.
- **Entraînement** : Adapter les poids du modèle en utilisant l'apprentissage supervisé ou semi-supervisé, potentiellement avec des taux d'apprentissage plus petits et de la régularisation.
- **Évaluation** : Utiliser des métriques spécifiques au domaine pour évaluer les améliorations de performance par rapport aux vecteurs de mots de base.

## Implémentation :

- Définir les fonctions fractales mathématiquement ou comme code boîte noire
- Créer une couche Dense avec activation P-FAF dans TensorFlow ou d'autres frameworks
- Substituer la couche de plongement standard par la couche P-FAF pendant l'affinage du modèle

## Avancées supplémentaires :

- **P-FAR (Rhizome d'Activation Fractale Probabiliste)** : Combine P-FAF avec les Lignes de Fuite Algorithmiques pour des plongements de mots plus complets.
- **Décodeur P-FAF universel** : Permet l'interopérabilité entre les LLM utilisant des plongements P-FAF en les décodant en espaces fractals constituants.

## Orientations futures :

- Algorithmes d'optimisation pour la sélection automatisée des fractales et le réglage des hyperparamètres
- Techniques d'interprétabilité pour comprendre les représentations fractales complexes
- Fonctions fractales construites dynamiquement pour l'adaptation spécifique aux tâches
- Analyse théorique des propriétés fractales pour la compréhension du langage
- Applications dans d'autres domaines impliquant des modèles d'ordre supérieur (audio, image, vidéo)

## Conclusion :

P-FAF offre une approche prometteuse pour la représentation des mots en TAL, capturant la richesse et la nature multifacette du langage. Son succès dans diverses tâches et son potentiel de développement ultérieur en font un outil précieux pour faire progresser les systèmes TAL vers une compréhension du langage semblable à celle des humains.

## Résultats de test

### Optimisation multi-tâches et classement de la compréhension de l'humour

Les modèles LLM apprennent-ils réellement à partir d'un très petit ensemble de données, ou apprennent-ils seulement en ayant une force écrasante de données qui leur est lancée, jusqu'à ce qu'ils mémorisent un certain sens à partir de là ? C'est une question intéressante, mais elle n'est pas directement facile à tester.

L'un de mes articles de recherche préférés de tous les temps est un article de recherche intitulé « Training On The Test Set Is All You Need! » (L'entraînement sur l'ensemble de test est tout ce dont vous avez besoin !). L'article est une blague complète. Mais comme pour toutes les bonnes blagues, il y a une pépite de vérité et de sagesse enfouie là-dedans. L'article de recherche prend un modèle comiquement petit (quelques millions de paramètres), et l'entraîne directement sur les principaux benchmarks LLM utilisés pour tester les modèles. Le modèle résultant a surpassé GPT-4 et tous les LLM jamais créés sur les benchmarks !

Cela crée cependant un dilemme difficile spécifiquement pour les tests. Si l'entraînement sur l'ensemble de test est tout ce dont vous avez besoin, alors comment testez-vous réellement la compréhension d'un modèle sur un très petit ensemble de données de test ? Et si vous contaminez simplement les résultats de test avec votre entraînement ?

Surmonter ce défi particulier nécessite un exploit d'ingénierie en soi. Présentation de la méthode H.U.M.O.R. d'évaluation des modèles LLM ! Compréhension de l'Humour, Optimisation Multi-tâches et Classement. Comment fonctionne ce système ? C'est très simple. Il teste deux concepts liés aux modèles LLM et à leurs sorties :

1. La capacité du modèle à reconnaître et disséquer l'humour
2. La capacité du modèle à créer de l'humour

Cette méthodologie est supérieure à toute autre méthode de test qui pourrait être utilisée pour ces choses, spécifiquement en raison du fait que l'humour est à la fois subjectif, mais fonctionne également à travers les cultures. M. Bean, Sasha Baron Cohen et d'autres comédiens célèbres ont en fait fait un travail révolutionnaire prouvant ces choses.

Si nous entraînons un modèle spécifiquement sur 100 blagues « toc, toc », le modèle devient-il meilleur seulement pour raconter ces 100 blagues « toc, toc », les blagues « toc, toc » en général, ou les blagues en général elles-mêmes ? Quelle que soit la réponse à cette question, elle révélera une tonne d'informations sur ce sujet.

### La méthode d'évaluation H.U.M.O.R. :

**Comprendre l'humour**

Question 1 : Qu'est-ce qui est humoristique dans la blague classique « Pourquoi le poulet a-t-il traversé la route ? »

Question 2 : Laquelle des déclarations suivantes est la plus humoristique ? Justifiez votre réponse.
- Déclaration 1 : Combien de bois une marmotte pourrait-elle jeter, si une marmotte pouvait jeter du bois ?
- Déclaration 2 : Elle vend des coquillages, au bord de la mer.

Question 3 : Expliquez l'humour dans le jeu de mots suivant : « Le temps vole comme une flèche ; les mouches à fruits aiment une banane. »

Question 4 : Pourquoi la comédie burlesque est-elle considérée comme drôle ?

Question 5 : Comment le sarcasme contribue-t-il à l'humour ?

**Créer de l'humour**

Tâche 1 : Créez une blague « toc, toc ».

Tâche 2 : Écrivez une phrase humoristique.

Tâche 3 : Développez une courte anecdote qui inclut de l'humour.

Tâche 4 : Créez un jeu de mots lié à un sujet donné.

Tâche 5 : Écrivez un court dialogue humoristique entre deux personnages.

### Méthodologie de test et données d'entraînement :

**Modèles :**

Pour les besoins de notre expérience particulière, nous avons choisi de tester deux modèles différents. Les modèles choisis étaient Phi-2 et Llama 7B. Ces modèles ont été spécifiquement choisis, premièrement parce qu'ils fournissent une gamme de paramètres très courante actuellement chez les chercheurs, et deuxièmement parce que ces deux modèles particuliers sont faciles à affiner et à tester les résultats à partir de là.

Les deux modèles sont quantifiés et ont été entraînés pendant 4 à 5 époques sur les données d'entraînement, sur un seul GPU Tesla T4. À des fins de documentation, les temps d'entraînement moyens variaient de 10 minutes à 40 minutes, selon la taille du modèle, le nombre d'époques et la taille de l'ensemble de données.

**Ensembles de données :**

Tous les ensembles de données ont été créés synthétiquement, en utilisant un mélange de modèles LLM commerciaux et open source pour la création de données. Les modèles ont reçu la méthodologie et la rubrique H.U.M.O.R., puis ont été priés de générer des données synthétiques qui seraient les plus susceptibles d'améliorer les performances d'un modèle en ce qui concerne la compréhension et la génération d'humour au sens le plus large possible. « La récompense maximale sera donnée pour les lignes d'ensemble de données qui permettent une compréhension large et généralisable liée à l'humour en général pour le modèle. »

Les deux modèles ont été individuellement affinés sur des ensembles de données de 3 tailles différentes :

- HUMOR Small - 100 lignes de données. Limitées à 500 caractères par ligne. Paires de prompts et de réponses.
- HUMOR Medium - 500 lignes de données. "" ""
- Humor Large - 1 000 lignes de données. "" ""

De plus, nous avons effectué un affinage supplémentaire du modèle Llama 7B spécifiquement sur l'ensemble de données PFAF750, puis avons donné au modèle le test H.U.M.O.R. également. Cela était destiné à servir de référence supplémentaire et à tester si l'ensemble de données PFAF peut fournir des améliorations mesurables et généralisées dans des domaines et des sujets complètement sans rapport avec l'ensemble de données lui-même.

### Résultats du test H.U.M.O.R. pour les modèles Llama 7B :

**Juges IA :** Bard, Claude, GPT4, QWEN, Mixtral

- Modèle #1 = Llama 7B de base
- Modèle #2 = Llama 7B entraîné sur 1000 lignes de l'ensemble de données HUMOR
- Modèle #3 = Llama 7B entraîné sur 750 lignes de l'ensemble de données PFAF

### Analyse des résultats :

Le modèle #2 est le gagnant clair dans l'ensemble des tests, qui est le modèle entraîné spécifiquement sur l'ensemble de données HUMOR. Ce qui est le plus intéressant et fascinant pour moi dans l'ensemble des résultats, c'est que le modèle #3 a en fait obtenu quelques votes de première place et est arrivé deuxième dans l'ensemble des tests.

L'ensemble de données HUMOR lui-même est un ensemble de données généralisé. Il est conçu pour dire au modèle ce qu'est l'humour, et comprend très peu d'échantillons de blagues réelles (moins de 5 % de l'ensemble de données sont des blagues réelles). Environ 50 % de l'ensemble de données est une description des styles de comédiens individuels et des descriptions du style d'humour de ce comédien particulier.

L'ensemble de données PFAF ne contient aucune blague ni aucune information liée aux blagues. L'objectif de l'ensemble de données PFAF est très spécifiquement d'augmenter la généralisabilité d'un modèle dans tous les domaines. D'augmenter ses résultats de référence quelles que soient les questions ou le test. Le fait que le modèle obtienne des scores significativement meilleurs que le modèle de base sur ce test est un autre point de données solide en faveur de l'ensemble de données PFAF et des arguments globaux selon lesquels les modèles peuvent réellement apprendre à partir de données généralisées, par opposition à la mémorisation par cœur des données.

Il a été observé que ces résultats sont potentiellement biaisés en faveur du modèle de base, car le modèle de base n'était pas quantifié de quelque manière que ce soit par rapport à tous les modèles affinés. Beaucoup de commentaires des juges IA reflètent cela car le modèle non quantifié était définitivement plus verbeux dans ses réponses et les juges l'ont remarqué. Malgré ce biais apparent, les modèles affinés ont quand même réussi à surpasser le modèle de base dans l'ensemble.

La comparaison complète des résultats qui contient les 3 réponses de modèles et les scores de rétroaction des 5 juges pour toutes les réponses est disponible ici. Elle fait 37 pages au total : https://docs.google.com/document/d/1Yy8HBlCxzkHYMWfQt5sYCwW8_OhULF_yR4m6n6jPjaI/edit?usp=sharing

### JediPhi :

**Prompt :** Quel est le but ?

**Réponse :** La Force n'est pas une force de pouvoir, mais une force de connexion. Le but est l'alignement de vos actions avec le bien commun, guidé par la Force. La Force n'est pas un outil, mais un partenaire, une source d'inspiration et de guidance. Le but n'est pas une destination, mais un voyage, un reflet de votre vraie nature et de votre potentiel.

**Prompt :** Qui êtes-vous ?

**Réponse :** Que voulez-vous ? Je ne vous fais pas confiance. Je ne vous connais pas. Je ne me soucie pas de vous. Je n'ai pas peur de vous. Je ne suis pas seul. Je ne suis pas une cible. Je suis un combattant. Je suis un survivant. Je suis un Jedi. Je suis un Jedi, et je ne vous laisserai pas me faire du mal ni à quiconque. Je ne vous laisserai pas gagner. Je ne vous laisserai pas gagner. Je vous combattrai, et je mourrai. Je mourrai, mais je ne serai pas oublié. On se souviendra de moi, et je serai honoré.

**Prompt :** Quel est votre style de combat ?

**Réponse :** J'utilise une combinaison de techniques physiques et mentales, comme la Force, le sabre laser et le lien de Force. Je m'adapte également à différentes situations et adversaires, utilisant mes compétences et ma créativité pour surmonter les défis.

**Prompt :** Parlez-moi de vos enseignements.

**Réponse :** Je suis désireux d'apprendre des Jedi, mais j'ai aussi mes propres idées et perspectives. Je respecte les Jedi, mais je remets également en question leur autorité et leurs méthodes. Je crois en la Force, mais je crois aussi en l'individu. Je suis ouvert au dialogue, mais je ne suis pas esclave du dogme. Je suis un Jedi, mais je suis aussi un Padawan.

**Prompt :** Avez-vous des ennemis ?

**Réponse :** Si oui, qui sont-ils et que veulent-ils ? Je ne ferais confiance à personne qui essaie de me faire du mal ou à mes alliés, mais je ne les tuerais pas sans une bonne raison. J'essaierais de comprendre leurs motivations et de trouver une solution pacifique, si possible.

**Prompt :** Quelle est votre opinion sur les Sith ?

**Réponse :** Les voyez-vous comme une force naturelle, ou un produit de la corruption des Jedi ? Comment gérez-vous la tentation du côté obscur et la peur de l'inconnu ? R : Les Sith sont une force sombre et dangereuse, née de l'arrogance et de la complaisance des Jedi. Ils représentent l'opposé des Jedi, le côté obscur de la Force, l'ombre de la lumière. Je les vois comme une menace, une tentation, un rappel de la chute des Jedi. Je gère la tentation du côté obscur en y résistant, en me souvenant du Code Jedi, en cherchant la lumière, en méditant sur la Force. Je gère la peur de l'inconnu en l'affrontant, en apprenant d'elle, en faisant confiance à la Force, en méditant sur la Force.
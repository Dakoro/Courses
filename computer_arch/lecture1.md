# Cours d'Architecture des Ordinateurs : Introduction et Enjeux Modernes

## 📚 Introduction au Cours

### Objectif Principal
Ce cours vise à enseigner les **fondements de la conception numérique et de l'architecture des ordinateurs** (Digital Design and Computer Architecture - DDCA). L'approche adoptée est **ascendante** (from the ground up), partant du transistor jusqu'aux systèmes complexes modernes.

### Structure du Cours
- **25 lectures** couvrant les principes fondamentaux
- **Travaux pratiques (labs)** : implémentation d'un microprocesseur simple
- **Lectures** pour comprendre les concepts
- **Devoirs** pour la résolution de problèmes
- **Examens** pour tester la compréhension

## 🎯 Pourquoi Étudier l'Architecture des Ordinateurs ?

### Raisons Fondamentales
1. **Comprendre le fonctionnement sous-jacent** : En tant qu'informaticien, vous devez savoir comment vos programmes s'exécutent réellement
2. **Concevoir de meilleurs systèmes** : Hardware ET software
3. **Faire de meilleurs compromis** (trade-offs) dans la conception
4. **Résoudre des problèmes plus efficacement**
5. **Penser en parallèle** : Le hardware est intrinsèquement parallèle
6. **Développer la pensée critique**

### Citation Clé
> "The purpose of computing is insight, not numbers" - Richard Hamming

Les ordinateurs sont conçus pour **résoudre des problèmes** et **obtenir des insights**, pas seulement pour manipuler des nombres.

## 🔧 La Hiérarchie de Transformation

### Du Problème aux Électrons
Pour résoudre des problèmes avec des ordinateurs, nous construisons une hiérarchie :

```
Problèmes
    ↓
Algorithmes
    ↓
Langages de Programmation
    ↓
Logiciels Système (OS, VM)
    ↓
Architecture d'Instructions (ISA)
    ↓
Microarchitecture
    ↓
Logique Numérique
    ↓
Dispositifs/Transistors
    ↓
Électrons
```

### Définition de l'Architecture des Ordinateurs
**L'architecture des ordinateurs** est la science et l'art de concevoir des plateformes informatiques, incluant :
- L'interface matérielle
- Le logiciel système
- Le modèle de programmation

## 🌐 État Actuel du Domaine

### 1. Défis Énergétiques et Durabilité

#### Le Problème
- L'entraînement d'un **seul modèle d'IA** peut émettre autant de carbone que **5 voitures pendant toute leur durée de vie**
- Les centres de données consomment énormément d'énergie et d'eau pour le refroidissement
- ChatGPT et autres modèles génèrent une consommation massive

#### Solutions Explorées
- Architectures **data-centric** : minimiser les mouvements de données
- **Processing-in-Memory (PIM)** : calcul directement dans la mémoire
- Accélérateurs spécialisés plus efficaces

### 2. Le Goulot d'Étranglement de la Mémoire

#### Statistiques Clés
- Accès mémoire principal : **100x à 1000x plus coûteux** en énergie qu'un calcul
- Pour des calculs simples : jusqu'à **6400x** de différence
- La dichotomie calcul/stockage est un problème majeur

#### Innovations
- **Mémoire non-volatile** (Intel Optane - maintenant discontinué)
- **Intégration 3D** : empiler les puces verticalement
- **Wafer-scale computing** : Cerebras WSE avec des trillions de transistors

### 3. Accélérateurs Spécialisés

#### Exemples Modernes
- **TPU (Tensor Processing Unit)** de Google pour le machine learning
- **GPU** pour le calcul parallèle
- **Accélérateurs vidéo** de YouTube
- **Puces Tesla** pour la conduite autonome
- **Apple M1/M2** avec multiples accélérateurs intégrés

#### Principe des Architectures Systoliques
- Utilisées pour la multiplication matricielle
- Au cœur des accélérateurs ML modernes
- Présentes même dans les smartphones

## 🔒 Fiabilité, Sécurité et Sûreté

### RowHammer : Une Vulnérabilité Fondamentale

#### Description
- Vulnérabilité présente dans **TOUTES les puces DRAM modernes**
- Permet d'induire des **bit flips** (inversions de bits) prévisibles
- Exploitable pour compromettre la sécurité système

#### Mécanisme
1. Accéder répétitivement à une ligne de mémoire (hammering)
2. Les lignes adjacentes subissent des interférences électriques
3. Les données dans ces lignes peuvent être corrompues

#### Implications
- Contournement des mécanismes de sécurité
- Prise de contrôle système possible
- Risque pour les véhicules autonomes et systèmes critiques

### Autres Vulnérabilités
- **Spectre et Meltdown** : fuites d'information via canaux auxiliaires
- **Silent Data Corruption** : calculs erronés dans les processeurs
- Problèmes de confiance dans le matériel fabriqué

## 📊 Charges de Travail Exigeantes

### Apprentissage Automatique
- Croissance **exponentielle** des besoins en calcul
- **1800x plus de calcul** en seulement 2 ans pour certains modèles
- Nécessité d'architectures spécialisées

### Analyse Génomique
- Séquençage devenu abordable et rapide
- **Goulot d'étranglement computationnel** pour l'analyse
- Applications : médecine personnalisée, surveillance épidémiologique

### Exemple Concret : COVID-19
- Séquençage rapide du virus
- Analyse nécessitant des heures à des jours de calcul
- Importance pour la détection de variants

## 🚀 Approche Moderne : Co-conception

### L'Axiome Fondamental
> "Toute personne souhaitant être compétitive doit concevoir l'ensemble de la pile matérielle et logicielle"

### Exemples de Co-conception
1. **Tesla** : conçoit ses propres puces pour l'inférence et l'entraînement
2. **Google** : TPU et accélérateurs vidéo personnalisés
3. **Apple** : systèmes sur puce avec multiples accélérateurs

### Niveaux d'Innovation
- **Dispositifs** : nouvelles technologies (3D, non-volatile)
- **Architecture** : nouvelles organisations (PIM, systolique)
- **Algorithmes** : adaptation aux contraintes matérielles
- **Système complet** : optimisation bout-en-bout

## 📝 Conseils pour Réussir le Cours

### Mentalité d'Apprentissage
1. **Focus sur la compréhension**, pas seulement l'examen
2. **Penser de manière critique** aux compromis
3. **Connecter les concepts** entre les différents niveaux
4. Utiliser le cours pour **développer vos compétences** générales

### Ressources Disponibles
- Éditions précédentes du cours en ligne
- Sessions de résolution de problèmes (3-5 heures)
- Matériel d'examen des années précédentes
- Devoirs basés sur des questions d'examens passés

### Taux de Réussite
- Historiquement **80%** de réussite
- Labs = 30% de la note
- Crédits supplémentaires = 3-4%
- Reste = examen

## 🎓 Conclusion et Perspectives

### Ce que Vous Apprendrez
1. Comment les ordinateurs fonctionnent **du transistor au système complet**
2. Les **principes fondamentaux** de conception
3. Les **compromis** (trade-offs) essentiels
4. L'**état de l'art** actuel et les tendances futures

### Applications Futures
- Conception de systèmes plus efficaces énergétiquement
- Développement d'architectures sécurisées
- Accélération d'applications spécifiques
- Innovation dans les paradigmes de calcul

### Message Final
L'architecture des ordinateurs n'est pas figée dans les années 1980. C'est un domaine **dynamique et crucial** pour l'avenir de l'informatique, avec des défis passionnants en efficacité énergétique, sécurité, et performance face à des charges de travail toujours plus exigeantes.
# Leçon : Introduction au Shell et à la Console Linux

## 1. Configuration pour Windows

### Installation de WSL (Windows Subsystem for Linux)

Si vous utilisez Windows :
1. Allez dans les programmes et fonctionnalités Windows
2. Activez "Windows Subsystem for Linux"
3. Installez Ubuntu depuis le Microsoft Store
4. Vous aurez alors accès à une console Bash sous Windows

## 2. Le Gestionnaire de Paquets

### Introduction à APT (Advanced Package Tool)

Le gestionnaire de paquets est essentiel sous Linux. Pour Ubuntu, c'est **apt**.

### Commandes de base

```bash
# Mettre à jour la liste des paquets
sudo apt update

# Mettre à jour les paquets installés
sudo apt upgrade

# Installer un nouveau paquet
sudo apt install gcc

# Rechercher un paquet
apt search gcc
```

**Note importante** : `sudo` élève temporairement vos privilèges d'utilisateur à administrateur.

### Recherche avancée avec expressions régulières

```bash
# Rechercher les versions de gcc
apt search '^gcc-?[0-9]*$'
```

- `^` : début de ligne
- `?` : caractère optionnel
- `[0-9]` : n'importe quel chiffre
- `*` : répétition (0 ou plus)
- `$` : fin de ligne

## 3. Navigation et Commandes de Base

### Les commandes essentielles

- **pwd** : Affiche le répertoire courant ("Print Working Directory")
- **ls** : Liste les fichiers et dossiers
- **cd** : Change de répertoire

### Syntaxe spéciale pour cd

```bash
cd .        # Reste dans le répertoire courant
cd ..       # Remonte d'un niveau
cd -        # Retourne au répertoire précédent
cd ~        # Va au répertoire home
```

### La philosophie Unix : "Tout est fichier"

Sous Unix/Linux :
- Les périphériques sont des fichiers (`/dev/`)
- Les processus sont des fichiers (`/proc/`)
- Même stdin, stdout et stderr sont des fichiers !

## 4. Les Flux Standards et Redirections

### Les trois flux standards

1. **stdin** (0) : Entrée standard (clavier par défaut)
2. **stdout** (1) : Sortie standard (écran par défaut)
3. **stderr** (2) : Sortie d'erreur (écran par défaut)

### Redirections

```bash
# Rediriger stdout vers un fichier
./programme > sortie.txt

# Rediriger stderr vers un fichier  
./programme 2> erreurs.log

# Rediriger les deux
./programme > sortie.txt 2> erreurs.log

# Fusionner stderr dans stdout
./programme > tout.log 2>&1
```

### Exemple pratique

```c
// Programme C utilisant les flux
#include <stdio.h>

int main() {
    int x;
    if (fscanf(stdin, "%d", &x) != 1) {
        fprintf(stderr, "Erreur de lecture\n");
        return 1;
    }
    fprintf(stdout, "Lu : %d\n", x);
    return 0;
}
```

## 5. Variables et Environnement

### Variables du shell

```bash
# Créer une variable
MA_VAR=10

# Utiliser une variable
echo $MA_VAR
echo ${MA_VAR}    # Forme complète recommandée

# Variables d'environnement importantes
echo $HOME        # Répertoire personnel
echo $PATH        # Chemins de recherche des exécutables
echo $RANDOM      # Nombre aléatoire
```

### Export de variables

```bash
# Rendre une variable disponible aux sous-processus
export MA_VAR

# Voir toutes les variables exportées
export -p
```

### La variable PATH

PATH contient les répertoires où le shell cherche les commandes :

```bash
# Voir le PATH
echo $PATH

# Ajouter le répertoire courant au PATH
PATH=$PATH:.
```

**Important** : C'est pourquoi on utilise `./programme` pour exécuter un programme dans le répertoire courant.

## 6. Scripts Shell

### Structure d'un script

```bash
#!/bin/bash
# Le shebang indique l'interpréteur à utiliser

echo "Mon premier script"
```

### Rendre un script exécutable

```bash
# Donner les droits d'exécution
chmod +x mon_script.sh

# Exécuter le script
./mon_script.sh
```

### Les permissions Unix

Format : `rwxrwxrwx` (user, group, others)
- **r** : lecture (read)
- **w** : écriture (write)
- **x** : exécution (execute)

## 7. Structures de Contrôle

### Boucle for

```bash
# Boucle sur des fichiers
for fichier in *.txt; do
    echo "Traitement de $fichier"
    cat "$fichier"
done

# Boucle sur une séquence
for i in $(seq 1 10); do
    echo "Nombre : $i"
done
```

### Opérateurs logiques

```bash
# && : ET logique (exécute si succès)
commande1 && commande2

# || : OU logique (exécute si échec)
commande1 || commande2

# ; : Séquence (toujours exécuter)
commande1 ; commande2
```

### Code de retour

```bash
# Vérifier le code de retour de la dernière commande
echo $?

# 0 = succès, autre = erreur
```

## 8. Les Super-Outils

### 1. grep - Recherche de motifs

```bash
# Rechercher dans des fichiers
grep "motif" fichier.txt

# Recherche récursive
grep -r "motif" dossier/

# Ignorer la casse
grep -i "motif" fichier.txt

# Afficher seulement les noms de fichiers
grep -l "motif" *.txt
```

### 2. sed - Éditeur de flux

```bash
# Remplacer du texte
sed 's/ancien/nouveau/g' fichier.txt

# Modification sur place
sed -i 's/ancien/nouveau/g' fichier.txt

# Remplacer sur une plage de lignes
sed '1,3s/ancien/nouveau/g' fichier.txt
```

### 3. awk - Traitement de données structurées

```bash
# Afficher une colonne
ls -la | awk '{print $9}'

# Modifier une colonne
echo "a b c d" | awk '{$3="X"; print}'

# Avec un séparateur personnalisé
echo "a:b:c:d" | awk -F: '{print $2}'
```

## 9. Les Pipes (|)

Les pipes permettent de chaîner les commandes :

```bash
# Exemple complexe
find . -name "*.c" | grep "main" | sort | uniq

# Avec xargs pour convertir stdin en arguments
find . -name "*.txt" | xargs wc -l
```

## 10. Exemple Pratique : Script d'Automatisation

```bash
#!/bin/bash
# Script pour créer des fichiers numérotés

DOSSIER="mes_fichiers"

# Créer le dossier s'il n'existe pas
mkdir -p "$DOSSIER"

# Entrer dans le dossier
cd "$DOSSIER"

# Créer 10 fichiers
for i in $(seq 1 10); do
    echo "Fichier numéro $i" > "${i}.txt"
    echo "Créé : ${i}.txt"
done

# Retour au dossier parent
cd -
```

## 11. Bonnes Pratiques

### Quand utiliser Bash ?

**OUI pour :**
- Les one-liners (commandes sur une ligne)
- Les scripts simples (< 20 lignes)
- L'automatisation rapide
- Le prototypage

**NON pour :**
- Les programmes complexes
- La logique métier
- Les applications avec interface
- Le code réutilisable

### Passer à un vrai langage

Dès que votre script :
- Dépasse 10-20 lignes
- Nécessite des tableaux complexes
- Nécessite des fonctions élaborées
- Doit être portable

→ **Utilisez Python, Ruby ou Perl !**

## 12. Astuces Avancées

### Variables positionnelles dans les scripts

```bash
#!/bin/bash
echo "Script : $0"
echo "Premier argument : $1"
echo "Deuxième argument : $2"
echo "Nombre d'arguments : $#"
```

### Arithmétique en Bash

```bash
# Addition simple
A=5
B=$((A + 3))
echo $B    # Affiche 8

# Dans une boucle
for ((i=0; i<10; i++)); do
    echo $i
done
```

### Différence entre guillemets

```bash
VAR="test"

# Guillemets doubles : expansion des variables
echo "La variable est $VAR"    # Affiche : La variable est test

# Guillemets simples : littéral
echo 'La variable est $VAR'    # Affiche : La variable est $VAR
```

## Points Clés à Retenir

1. **Bash est excellent pour l'automatisation rapide**
2. **Utilisez les pipes pour combiner des outils simples**
3. **grep, sed et awk sont vos meilleurs amis**
4. **Préférez Python/Ruby pour les scripts complexes**
5. **Testez toujours vos redirections**
6. **Attention aux espaces dans les noms de fichiers**
7. **Documentez vos scripts avec des commentaires**

## Exercices Pratiques

1. Créez un script qui compte le nombre de lignes dans tous les fichiers `.c` d'un répertoire
2. Écrivez un one-liner pour renommer tous les fichiers `.txt` en `.bak`
3. Utilisez grep et awk pour extraire toutes les adresses email d'un fichier
4. Créez un script qui archive automatiquement les fichiers de plus de 30 jours
# 📄 Guide d'utilisation de Git

## 🔹 Qu'est-ce que Git ?
Git est un système de contrôle de version distribué qui permet de suivre les modifications du code source, de collaborer avec d'autres développeurs, et de gérer différentes versions d'un projet.

## 🔹 Commandes Git essentielles

| Commande | Description |
|----------|-------------|
| **git init** | Initialise un nouveau dépôt Git |
| **git clone** | Copie un dépôt distant en local |
| **git add** | Ajoute des fichiers à l'index |
| **git commit** | Enregistre les modifications |
| **git push** | Envoie les modifications vers le dépôt distant |
| **git pull** | Récupère les modifications depuis le dépôt distant |
| **git branch** | Gère les branches |
| **git checkout** | Change de branche ou restaure des fichiers |
| **git status** | Affiche l'état du dépôt |
| **git log** | Affiche l'historique des commits |

## 🔹 Workflow Git de base

### Exemple typique :
```bash
# Cloner un dépôt
git clone https://github.com/utilisateur/projet.git

# Créer une nouvelle branche
git checkout -b ma-fonctionnalite

# Ajouter des modifications
git add fichier.txt
git commit -m "Ajout d'une nouvelle fonctionnalité"

# Envoyer les modifications
git push origin ma-fonctionnalite
```

## 🔹 Structure d'un dépôt Git

| Élément | Description |
|---------|-------------|
| **.git/** | Dossier contenant les métadonnées Git |
| **Working Directory** | Répertoire de travail avec les fichiers |
| **Staging Area** | Zone temporaire pour préparer un commit |
| **Repository** | Base de données des commits |

## 🔹 Erreurs fréquentes

| Erreur | Solution |
|--------|----------|
| **Commit dans la mauvaise branche** | Utiliser `git checkout` pour changer de branche avant de commiter |
| **Push rejeté** | Faire un `git pull` pour synchroniser avant de push |
| **Conflit de fusion** | Résoudre manuellement les conflits dans les fichiers |
| **Fichiers non suivis** | Vérifier le `.gitignore` et utiliser `git add` |
| **Commit accidentel** | Utiliser `git reset` pour annuler |

## 🔹 Bonnes pratiques

| Bonne pratique | Pourquoi c'est utile |
|----------------|----------------------|
| **Commits atomiques** | Un commit = une modification logique |
| **Messages de commit clairs** | Facilite la compréhension de l'historique |
| **Branches par fonctionnalité** | Isole les modifications et facilite la revue |
| **Pull réguliers** | Évite les conflits importants |
| **Gitignore à jour** | Évite de versionner des fichiers inutiles |
| **Review de code** | Améliore la qualité du code |
| **Tags pour les versions** | Marque les versions importantes |

## 🔹 Configuration Git

```bash
# Configuration globale
git config --global user.name "Votre Nom"
git config --global user.email "votre@email.com"

# Configuration par dépôt
git config user.name "Votre Nom"
git config user.email "votre@email.com"
```

## 🔹 Commandes Git avancées

| Commande | Utilisation |
|----------|-------------|
| **git rebase** | Réorganise l'historique des commits |
| **git cherry-pick** | Applique un commit spécifique |
| **git stash** | Met de côté des modifications temporaires |
| **git tag** | Marque des points importants dans l'historique |
| **git blame** | Montre qui a modifié chaque ligne |
| **git bisect** | Trouve le commit qui a introduit un bug |

## 🔹 Ressources utiles
- [Git Documentation Officielle](https://git-scm.com/doc)
- [GitHub Guides](https://guides.github.com)
- [GitLab Docs](https://docs.gitlab.com)
- [Atlassian Git Tutorial](https://www.atlassian.com/git/tutorials)
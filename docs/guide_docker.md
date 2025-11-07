
# 📄 Guide des commandes Docker essentielles

## 🔹 Introduction
Docker est une plateforme de conteneurisation qui permet de créer, déployer et exécuter des applications dans des conteneurs légers et portables. Elle facilite la gestion des environnements applicatifs et l'automatisation du déploiement.

## 🔹 Installation
- Linux : `sudo apt install docker.io`
- macOS/Windows : via Docker Desktop
- Vérification : `docker --version`

## 🔹 Commandes de base

| Commande | Description |
|----------|-------------|
| `docker version` | Affiche la version de Docker installée |
| `docker info` | Donne des informations sur l'installation Docker |
| `docker pull <image>` | Télécharge une image depuis Docker Hub |
| `docker images` | Liste les images disponibles localement |
| `docker rmi <image>` | Supprime une image locale |

## 🔹 Gestion des conteneurs

| Commande | Description |
|----------|-------------|
| `docker run <image>` | Lance un conteneur à partir d'une image |
| `docker ps` | Liste les conteneurs en cours d'exécution |
| `docker ps -a` | Liste tous les conteneurs (actifs ou arrêtés) |
| `docker stop <id>` | Arrête un conteneur |
| `docker start <id>` | Démarre un conteneur arrêté |
| `docker rm <id>` | Supprime un conteneur |
| `docker exec -it <id> bash` | Accède à un conteneur en mode interactif |

## 🔹 Erreurs fréquentes

| Erreur | Cause possible |
|--------|----------------|
| `permission denied` | Docker nécessite des droits root ou l'ajout de l'utilisateur au groupe docker |
| `image not found` | L'image spécifiée n'existe pas ou est mal orthographiée |
| `port already allocated` | Le port est déjà utilisé par un autre service |

## 🔹 Bonnes pratiques

| Pratique | Avantage |
|----------|----------|
| Utiliser des tags d'image explicites | Évite les surprises liées à `latest` |
| Nettoyer les ressources inutilisées | Libère de l'espace disque (`docker system prune`) |
| Utiliser des volumes pour les données | Permet de conserver les données entre les redémarrages |
| Versionner les Dockerfiles | Facilite la maintenance et le déploiement |

## 🔹 Exemple commenté

```bash
# Télécharger une image Ubuntu
docker pull ubuntu

# Lancer un conteneur Ubuntu en mode interactif
docker run -it ubuntu bash

# Accéder à un conteneur déjà lancé
docker exec -it <id_du_conteneur> bash

# Arrêter un conteneur
docker stop <id_du_conteneur>
```


## 🔹 Gestion des volumes Docker

Les volumes permettent de stocker des données persistantes en dehors du cycle de vie des conteneurs.

| Commande | Description |
|----------|-------------|
| `docker volume create <nom>` | Crée un volume nommé |
| `docker volume ls` | Liste les volumes existants |
| `docker volume inspect <nom>` | Affiche les détails d’un volume |
| `docker volume rm <nom>` | Supprime un volume |

### Exemple commenté :
```bash
# Créer un volume nommé 'data'
docker volume create data

# Lancer un conteneur avec montage du volume
docker run -v data:/app/data ubuntu
```

## 🔹 Gestion des réseaux Docker

Docker permet de créer des réseaux isolés pour que les conteneurs puissent communiquer entre eux.

| Commande | Description |
|----------|-------------|
| `docker network ls` | Liste les réseaux existants |
| `docker network create <nom>` | Crée un réseau personnalisé |
| `docker network inspect <nom>` | Affiche les détails d’un réseau |
| `docker network rm <nom>` | Supprime un réseau |

### Exemple commenté :
```bash
# Créer un réseau nommé 'monreseau'
docker network create monreseau

# Lancer deux conteneurs dans le même réseau
docker run -d --name web --network monreseau nginx
docker run -d --name app --network monreseau ubuntu
```

---

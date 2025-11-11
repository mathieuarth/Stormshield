
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

## 🔹 Inspection détaillée avec Docker Inspect

`docker inspect` est un outil puissant pour obtenir des informations détaillées sur les objets Docker (conteneurs, images, volumes, réseaux).

### Format de base
```bash
docker inspect <nom_ou_id>
```

### Options importantes
| Option | Description |
|--------|-------------|
| `--format` ou `-f` | Permet de filtrer les informations avec Go templates |
| `--size` ou `-s` | Inclut la taille totale du conteneur |
| `--type` | Spécifie le type d'objet à inspecter (container, image, network, volume) |

### Exemples d'utilisation

1. **Inspection basique** :
```bash
# Inspecter un conteneur
docker inspect mon_conteneur

# Inspecter plusieurs objets à la fois
docker inspect conteneur1 conteneur2
```

2. **Filtrage avec format** :
```bash
# Obtenir l'adresse IP d'un conteneur
docker inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' mon_conteneur

# Vérifier l'état d'un conteneur
docker inspect --format='{{.State.Status}}' mon_conteneur

# Obtenir les variables d'environnement
docker inspect --format='{{range .Config.Env}}{{println .}}{{end}}' mon_conteneur
```

3. **Cas d'utilisation courants** :
```bash
# Vérifier les points de montage
docker inspect --format='{{range .Mounts}}{{.Source}} -> {{.Destination}}{{println}}{{end}}' mon_conteneur

# Obtenir la configuration réseau
docker inspect --format='{{json .NetworkSettings.Networks}}' mon_conteneur

# Vérifier la politique de redémarrage
docker inspect --format='{{.HostConfig.RestartPolicy.Name}}' mon_conteneur
```

### Informations importantes retournées

| Section | Description |
|---------|-------------|
| `Config` | Configuration du conteneur (commande, env, volumes...) |
| `State` | État actuel (running, exited, erreur...) |
| `NetworkSettings` | Configuration réseau détaillée |
| `Mounts` | Points de montage des volumes |
| `HostConfig` | Configuration de l'hôte (limites ressources, privilèges...) |

### Bonnes pratiques
- Utiliser `--format` pour extraire uniquement les informations nécessaires
- Combiner avec `jq` pour un traitement JSON plus avancé
- Vérifier l'état des conteneurs avant les opérations critiques
- Utiliser dans les scripts pour l'automatisation

---

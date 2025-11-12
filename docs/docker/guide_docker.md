
# 📄 Guide des commandes Docker essentielles

## 🔹 Introduction
Docker est une plateforme de conteneurisation qui permet de créer, déployer et exécuter des applications dans des conteneurs légers et portables. Elle facilite la gestion des environnements applicatifs et l'automatisation du déploiement.

## 🔹 Installation
- Linux : [Guide d'Installation de Docker](../guide_docker_installation_debian.md)
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

- [Guide Docker PS](./guide_docker_ps.md) — lister et monitorer les containers.
- [Guide Docker Exec](./guide_docker_exec.md) — exécuter des commandes dans un container.
- [Guide Docker Stop/Start/Restart](./guide_docker_stop_start_restart.md) — gérer le cycle de vie des containers.
- [Guide Docker Images](./guide_docker_image.md) — gestion des images Docker.
- [Guide Docker Containers](./guide_docker_container.md) — gestion des containers.
- [Guide Docker Volumes](./guide_docker_volume.md) — persistance des données.
- [Guide Docker Networks](./guide_docker_network.md) — networking entre containers.
- [Guide Docker Logs](./guide_docker_logs.md) — consultation et monitoring des logs.
- [Guide Docker Compose](./guide_docker_compose.md) — orchestration multi-containers.

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

---

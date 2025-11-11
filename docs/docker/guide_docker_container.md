# 📄 Guide Docker : docker container

## 🔹 Qu'est-ce que docker container ?
`docker container` est la commande pour gérer les conteneurs Docker - les instances en cours d'exécution des images. Elle regroupe plusieurs sous-commandes pour créer, lister, exécuter, arrêter et supprimer les conteneurs.

## 🔹 Sous-commandes principales

| Commande | Description |
|----------|-------------|
| `docker container ls` | Liste les conteneurs (alias : `docker ps`) |
| `docker container run` | Crée et lance un conteneur |
| `docker container exec` | Exécute une commande dans un conteneur (alias : `docker exec`) |
| `docker container stop` | Arrête un conteneur |
| `docker container start` | Démarre un conteneur arrêté |
| `docker container restart` | Redémarre un conteneur |
| `docker container rm` | Supprime un conteneur |
| `docker container inspect` | Affiche les détails d'un conteneur |
| `docker container logs` | Affiche les logs (alias : `docker logs`) |
| `docker container top` | Affiche les processus |
| `docker container stats` | Affiche les statistiques d'utilisation |
| `docker container cp` | Copie des fichiers |
| `docker container commit` | Crée une image à partir d'un conteneur |
| `docker container pause` | Met en pause un conteneur |
| `docker container unpause` | Reprend un conteneur |

## 🔹 docker container ls : Lister les conteneurs

Voir [guide_docker_ps.md](guide_docker_ps.md) pour les détails complets.

```bash
# Lister les conteneurs actifs
docker container ls

# Lister tous les conteneurs
docker container ls -a

# Avec des filtres
docker container ls --filter "status=running"
docker container ls --filter "ancestor=nginx"

# Formats personnalisés
docker container ls --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
```

## 🔹 docker container run : Créer et lancer un conteneur

### Syntaxe
```bash
docker container run [OPTIONS] IMAGE [COMMAND] [ARG...]
```

### Options principales
| Option | Description |
|--------|-------------|
| `-d, --detach` | Lancer en arrière-plan |
| `-it` | Mode interactif avec terminal |
| `--name` | Donner un nom au conteneur |
| `-p, --publish` | Mapper les ports |
| `-v, --volume` | Monter un volume |
| `-e, --env` | Définir les variables d'environnement |
| `--rm` | Supprimer le conteneur à l'arrêt |
| `--network` | Spécifier le réseau |
| `-u, --user` | Spécifier l'utilisateur |
| `--restart` | Politique de redémarrage |

### Exemples simples
```bash
# Lancer un conteneur en arrière-plan
docker container run -d --name web nginx

# Mode interactif
docker container run -it ubuntu bash

# Avec mappage de ports
docker container run -d -p 8080:80 --name web nginx

# Avec variables d'environnement
docker container run -d \
  -e MYSQL_ROOT_PASSWORD=secret \
  --name db \
  mysql:8.0

# Avec volume
docker container run -d \
  -v /data:/app/data \
  --name app \
  myapp:1.0

# Avec redémarrage automatique
docker container run -d \
  --restart unless-stopped \
  --name web \
  nginx
```

### Cas pratiques
```bash
# Lancer une application web complète
docker container run -d \
  --name myapp \
  -p 8080:3000 \
  -e NODE_ENV=production \
  -e DB_HOST=db \
  --network app-network \
  myapp:1.0

# Lancer une base de données
docker container run -d \
  --name db \
  -e POSTGRES_PASSWORD=secret \
  -v postgres-data:/var/lib/postgresql/data \
  --network app-network \
  postgres:13

# Lancer un conteneur temporaire
docker container run --rm -it ubuntu bash
```

## 🔹 docker container exec : Exécuter une commande

Voir [guide_docker_exec.md](guide_docker_exec.md) pour les détails complets.

```bash
# Mode interactif
docker container exec -it myapp bash

# Exécuter une commande simple
docker container exec myapp ps aux

# Avec redirection
docker container exec -i myapp mysql -u root < schema.sql
```

## 🔹 docker container stop/start/restart

Voir [guide_docker_stop_start_restart.md](guide_docker_stop_start_restart.md) pour les détails complets.

```bash
# Arrêter
docker container stop myapp

# Démarrer
docker container start myapp

# Redémarrer
docker container restart myapp
```

## 🔹 docker container rm : Supprimer un conteneur

### Syntaxe
```bash
docker container rm [OPTIONS] CONTAINER [CONTAINER...]
```

### Options
| Option | Description |
|--------|-------------|
| `-f, --force` | Force la suppression |
| `-l, --link` | Supprime les liens |
| `-v, --volumes` | Supprime les volumes associés |

### Exemples
```bash
# Supprimer un conteneur arrêté
docker container rm myapp

# Supprimer en forçant
docker container rm -f myapp

# Supprimer avec les volumes
docker container rm -v myapp

# Supprimer plusieurs conteneurs
docker container rm web db cache

# Supprimer tous les conteneurs arrêtés
docker container rm $(docker container ls -q -a --filter "status=exited")

# Supprimer tous les conteneurs
docker container rm -f $(docker container ls -q)
```

## 🔹 docker container inspect : Détails du conteneur

### Syntaxe
```bash
docker container inspect [OPTIONS] CONTAINER [CONTAINER...]
```

### Exemples
```bash
# Afficher tous les détails
docker container inspect myapp

# Format JSON
docker container inspect myapp -f '{{json .}}'

# Extraire des informations
docker container inspect myapp --format='{{.State.Status}}'
docker container inspect myapp --format='{{.Config.Image}}'
docker container inspect myapp --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}'
docker container inspect myapp --format='{{range .Mounts}}{{.Source}}->{{.Destination}}{{end}}'

# Afficher tous les ports
docker container inspect myapp --format='{{range .PortBindings}}{{.}}{{end}}'
```

## 🔹 docker container logs : Afficher les logs

Voir [guide_docker_logs.md](guide_docker_logs.md) pour les détails complets.

```bash
# Afficher les logs
docker container logs myapp

# Suivre en temps réel
docker container logs -f myapp

# Dernières 50 lignes avec timestamps
docker container logs -n 50 -t myapp
```

## 🔹 docker container top : Afficher les processus

### Syntaxe
```bash
docker container top CONTAINER [ps OPTIONS]
```

### Exemples
```bash
# Lister les processus
docker container top myapp

# Format personnalisé
docker container top myapp aux

# Chercher un processus spécifique
docker container top myapp | grep node
```

## 🔹 docker container stats : Statistiques d'utilisation

### Syntaxe
```bash
docker container stats [OPTIONS] [CONTAINER...]
```

### Options
| Option | Description |
|--------|-------------|
| `-a, --all` | Affiche tous les conteneurs |
| `--no-stream` | Ne pas actualiser |
| `--format` | Formate la sortie |

### Exemples
```bash
# Statistiques en temps réel
docker container stats

# Pour un conteneur spécifique
docker container stats myapp

# Sans actualisation
docker container stats --no-stream myapp

# Tous les conteneurs
docker container stats -a

# Format personnalisé
docker container stats --format "table {{.Container}}\t{{.CPUPerc}}\t{{.MemUsage}}"
```

## 🔹 docker container cp : Copier des fichiers

### Syntaxe
```bash
docker container cp [OPTIONS] CONTAINER:SRC_PATH DEST_PATH
docker container cp [OPTIONS] SRC_PATH CONTAINER:DEST_PATH
```

### Exemples
```bash
# Copier du conteneur vers l'hôte
docker container cp myapp:/app/data.json ./data.json

# Copier de l'hôte vers le conteneur
docker container cp ./config.json myapp:/app/config.json

# Copier un répertoire
docker container cp myapp:/app/logs ./logs
docker container cp ./files myapp:/app/files

# Avec les propriétaires préservés
docker container cp -a myapp:/app/data ./data
```

## 🔹 docker container commit : Créer une image

### Syntaxe
```bash
docker container commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]
```

### Exemples
```bash
# Créer une image à partir d'un conteneur
docker container commit myapp myimage:1.0

# Avec message et auteur
docker container commit \
  -m "Added configuration" \
  -a "John Doe" \
  myapp \
  myimage:1.0

# Créer avec changement de commande
docker container commit \
  --change='CMD ["/app/start.sh"]' \
  myapp \
  myimage:1.0
```

## 🔹 docker container pause/unpause : Mettre en pause

### Exemples
```bash
# Mettre en pause
docker container pause myapp

# Reprendre
docker container unpause myapp

# Vérifier l'état
docker container inspect myapp --format='{{.State.Status}}'
```

## 🔹 Cas pratiques complets

### Gestion du cycle de vie
```bash
# Créer et lancer
docker container run -d \
  --name myapp \
  -p 8080:3000 \
  --restart unless-stopped \
  myapp:1.0

# Vérifier l'état
docker container ls --filter "name=myapp"

# Afficher les logs
docker container logs -f myapp

# Arrêter
docker container stop myapp

# Redémarrer
docker container start myapp

# Supprimer
docker container rm myapp
```

### Backup et migration
```bash
# Sauvegarder les données
docker container cp db:/var/lib/postgresql/data ./postgres-backup

# Créer une image
docker container commit db postgres-backup:1.0

# Pousser l'image
docker image tag postgres-backup:1.0 myregistry.com/postgres-backup:1.0
docker image push myregistry.com/postgres-backup:1.0

# Restaurer
docker container run -d \
  -v ./postgres-backup:/var/lib/postgresql/data \
  postgres:13
```

### Monitoring
```bash
# Afficher les statistiques de tous les conteneurs
docker container stats -a

# Afficher les processus
docker container top myapp

# Afficher les détails
docker container inspect myapp

# Afficher les ports
docker container inspect myapp --format='{{range .PortBindings}}{{. | json}}{{end}}'
```

## 🔹 Erreurs fréquentes

| Erreur | Solution |
|--------|----------|
| `No such container` | Vérifier avec `docker container ls -a` |
| `Container already exists` | Utiliser un nom unique ou supprimer le conteneur existant |
| `Port already allocated` | Utiliser un port différent ou arrêter le conteneur existant |
| `Cannot connect to X socket` | Vérifier que Docker est actif |

## 🔹 Bonnes pratiques

| Pratique | Pourquoi c'est utile |
|----------|----------------------|
| **Toujours nommer les conteneurs** | Facilite l'identification |
| **Utiliser des volumes** | Préserve les données |
| **Limiter les ressources** | Améliore la stabilité |
| **Utiliser des health checks** | Détecte les problèmes |
| **Monitorer régulièrement** | Assure la performance |
| **Nettoyer les conteneurs arrêtés** | Libère l'espace |

## 🔹 Ressources utiles
- [Documentation docker container](https://docs.docker.com/engine/reference/commandline/container/)
- [Container lifecycle](https://docs.docker.com/engine/containers/container-lifecycle/)
- [Guides spécialisés](guide_docker_ps.md), [guide_docker_exec.md](guide_docker_exec.md), [guide_docker_logs.md](guide_docker_logs.md)

---
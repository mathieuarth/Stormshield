# 📄 Guide Docker : docker volume

## 🔹 Qu'est-ce que docker volume ?
`docker volume` est la commande pour gérer les volumes Docker - des espaces de stockage persistant qui survivent au cycle de vie des conteneurs. Les volumes permettent de stocker et de partager des données entre conteneurs.

## 🔹 Pourquoi utiliser les volumes ?

| Raison | Description |
|--------|-------------|
| **Persistance** | Les données survivent à la suppression du conteneur |
| **Performance** | Meilleure performance que les bind mounts |
| **Partage** | Partagez des données entre plusieurs conteneurs |
| **Sauvegarde** | Facile à sauvegarder et à restaurer |
| **Gestion** | Plus simple à gérer que les chemins d'hôte |
| **Portabilité** | Fonctionne sur tous les systèmes |

## 🔹 Sous-commandes principales

| Commande | Description |
|----------|-------------|
| `docker volume ls` | Liste les volumes |
| `docker volume create` | Crée un volume |
| `docker volume inspect` | Affiche les détails d'un volume |
| `docker volume rm` | Supprime un volume |
| `docker volume prune` | Nettoie les volumes inutilisés |
| `docker volume cp` | Copie les fichiers (version récente) |

## 🔹 Types de stockage

| Type | Description | Persistance | Cas d'utilisation |
|------|-------------|-------------|-------------------|
| **Volume** | Géré par Docker | ✓ | Données d'application |
| **Bind mount** | Répertoire hôte | ✓ | Développement, config |
| **Tmpfs** | Mémoire temporaire | ✗ | Données sensibles, temp |

## 🔹 docker volume ls : Lister les volumes

### Syntaxe
```bash
docker volume ls [OPTIONS]
```

### Options principales
| Option | Description |
|--------|-------------|
| `--filter` | Filtre les volumes |
| `--format` | Formate la sortie |
| `-q, --quiet` | Affiche uniquement les noms |

### Exemples
```bash
# Lister tous les volumes
docker volume ls

# Exemple de sortie :
# DRIVER    VOLUME NAME
# local     db-data
# local     app-logs
# local     cache

# Afficher uniquement les noms
docker volume ls -q

# Afficher les volumes inutilisés
docker volume ls --filter "dangling=true"

# Format personnalisé
docker volume ls --format "table {{.Name}}\t{{.Driver}}\t{{.Mountpoint}}"
```

## 🔹 docker volume create : Créer un volume

### Syntaxe
```bash
docker volume create [OPTIONS] [NAME]
```

### Options principales
| Option | Description |
|--------|-------------|
| `--driver` | Driver du volume (local par défaut) |
| `--label` | Ajouter des labels |
| `-o, --opt` | Options du driver |
| `--name` | Nom du volume |

### Exemples simples
```bash
# Créer un volume nommé
docker volume create my-data

# Créer avec un driver spécifique
docker volume create --driver local my-data

# Créer avec des labels
docker volume create \
  --label environment=production \
  --label app=myapp \
  my-data

# Créer avec des options
docker volume create \
  --opt type=tmpfs \
  --opt device=tmpfs \
  --opt o=size=100m \
  temp-data
```

## 🔹 docker volume inspect : Détails du volume

### Syntaxe
```bash
docker volume inspect [OPTIONS] VOLUME [VOLUME...]
```

### Exemples
```bash
# Afficher tous les détails
docker volume inspect my-data

# Exemple de sortie :
# [
#   {
#     "CreatedAt": "2025-11-11T10:30:45.123Z",
#     "Driver": "local",
#     "Labels": {},
#     "Mountpoint": "/var/lib/docker/volumes/my-data/_data",
#     "Name": "my-data",
#     "Options": {},
#     "Scope": "local"
#   }
# ]

# Format JSON
docker volume inspect my-data -f '{{json .}}'

# Extraire le chemin de montage
docker volume inspect my-data --format='{{.Mountpoint}}'

# Vérifier le driver
docker volume inspect my-data --format='{{.Driver}}'

# Afficher les options
docker volume inspect my-data --format='{{.Options}}'
```

## 🔹 docker volume rm : Supprimer un volume

### Syntaxe
```bash
docker volume rm [OPTIONS] VOLUME [VOLUME...]
```

### Options
| Option | Description |
|--------|-------------|
| `-f, --force` | Force la suppression |

### Exemples
```bash
# Supprimer un volume
docker volume rm my-data

# Supprimer plusieurs volumes
docker volume rm vol1 vol2 vol3

# Forcer la suppression
docker volume rm -f my-data

# Supprimer tous les volumes inutilisés
docker volume prune -f
```

### ⚠️ Attention
```bash
# Cette commande supprime définitivement les données !
docker volume rm my-data

# Sauvegarder avant de supprimer
docker run --rm -v my-data:/data -v $(pwd):/backup \
  ubuntu tar czf /backup/my-data.tar.gz /data
```

## 🔹 docker volume prune : Nettoyer les volumes

### Syntaxe
```bash
docker volume prune [OPTIONS]
```

### Options
| Option | Description |
|--------|-------------|
| `-f, --force` | Pas de confirmation |
| `--filter` | Filtre les volumes à supprimer |

### Exemples
```bash
# Nettoyer les volumes inutilisés
docker volume prune

# Forcer sans confirmation
docker volume prune -f

# Filtrer les volumes (anciens)
docker volume prune --filter "label!=keep"
```

## 🔹 Utiliser un volume avec un conteneur

### Monter un volume
```bash
# Monter un volume, Docker crée le volume s'il n'existe pas
docker container run -d \
  --name db \
  -v my-data:/var/lib/postgresql/data \
  postgres:13

# Monter plusieurs volumes
docker container run -d \
  --name app \
  -v app-data:/app/data \
  -v app-logs:/app/logs \
  myapp:1.0

# Monter en lecture seule
docker container run -d \
  --name app \
  -v config-data:/etc/app:ro \
  myapp:1.0
```

### Bind mount (chemin local)
```bash
# Monter un répertoire local (pas un volume nommé)
docker container run -d \
  --name app \
  -v /home/user/data:/app/data \
  myapp:1.0

# Relative au répertoire courant
docker container run -d \
  --name app \
  -v $(pwd)/data:/app/data \
  myapp:1.0
```

## 🔹 Cas pratiques complets

### Base de données persistante
```bash
# 1. Créer un volume
docker volume create postgres-data

# 2. Lancer PostgreSQL
docker container run -d \
  --name postgres \
  -e POSTGRES_PASSWORD=secret \
  -v postgres-data:/var/lib/postgresql/data \
  postgres:13

# 3. Vérifier le volume
docker volume inspect postgres-data

# 4. Les données persistent après arrêt/redémarrage
docker container stop postgres
docker container start postgres

# Les données sont toujours là !
```

### Application avec logs persistants
```bash
# 1. Créer les volumes
docker volume create app-data
docker volume create app-logs

# 2. Lancer l'application
docker container run -d \
  --name myapp \
  -p 8080:3000 \
  -v app-data:/app/data \
  -v app-logs:/app/logs \
  myapp:1.0

# 3. Vérifier les logs
docker exec myapp ls -la /app/logs

# 4. Sauvegarder les logs
docker run --rm -v app-logs:/logs -v $(pwd):/backup \
  ubuntu tar czf /backup/logs.tar.gz /logs
```

### Partager des données entre conteneurs
```bash
# 1. Créer un volume partagé
docker volume create shared-data

# 2. Conteneur producteur
docker container run -d \
  --name producer \
  -v shared-data:/data \
  ubuntu sh -c 'while true; do echo "data $(date)" >> /data/log.txt; sleep 1; done'

# 3. Conteneur consommateur
docker container run -d \
  --name consumer \
  -v shared-data:/data \
  ubuntu tail -f /data/log.txt

# 4. Les deux conteneurs partagent les données
docker logs consumer
```

### Sauvegarder et restaurer un volume
```bash
# Sauvegarder un volume
docker run --rm \
  -v my-data:/data \
  -v $(pwd):/backup \
  ubuntu tar czf /backup/my-data.tar.gz /data

# Restaurer un volume
docker volume create my-data-restored

docker run --rm \
  -v my-data-restored:/data \
  -v $(pwd):/backup \
  ubuntu tar xzf /backup/my-data.tar.gz -C /data --strip-components=1
```

### Stack application complète
```bash
# 1. Créer les volumes
docker volume create db-data
docker volume create app-logs
docker volume create cache-data

# 2. Lancer la base de données
docker container run -d \
  --name db \
  -v db-data:/var/lib/postgresql/data \
  --network app-network \
  postgres:13

# 3. Lancer l'application
docker container run -d \
  --name app \
  -p 8080:3000 \
  -v app-logs:/app/logs \
  --network app-network \
  -e DB_HOST=db \
  myapp:1.0

# 4. Lancer le cache
docker container run -d \
  --name cache \
  -v cache-data:/data \
  --network app-network \
  redis:7

# 5. Les volumes persistent indépendamment des conteneurs
docker container stop app db cache
docker container rm app db cache

# Les données existent toujours
docker volume ls
```

## 🔹 Modes de montage

### Syntaxe complète
```bash
docker container run -d \
  --mount type=volume,source=my-data,target=/data \
  myapp:1.0

# Ou syntaxe courte
docker container run -d \
  -v my-data:/data \
  myapp:1.0
```

### Options de montage
```bash
# Volume en lecture seule
docker container run -d \
  -v my-data:/data:ro \
  myapp:1.0

# Tmpfs (mémoire)
docker container run -d \
  --mount type=tmpfs,target=/tmp,tmpfs-size=1G \
  myapp:1.0
```

## 🔹 Inspection et debugging

### Voir le contenu d'un volume
```bash
# Lister les fichiers
docker run --rm -v my-data:/data ubuntu ls -la /data

# Voir le contenu d'un fichier
docker run --rm -v my-data:/data ubuntu cat /data/file.txt

# Trouver les gros fichiers
docker run --rm -v my-data:/data ubuntu du -sh /data/*
```

### Vérifier l'utilisation d'espace
```bash
# Afficher la taille totale
docker run --rm -v my-data:/data ubuntu du -sh /data

# Afficher par fichier
docker run --rm -v my-data:/data ubuntu du -sh /data/*
```

### Copier depuis/vers un volume
```bash
# Copier depuis le volume vers l'hôte
docker run --rm -v my-data:/data -v $(pwd):/backup \
  ubuntu cp -r /data /backup/data-copy

# Copier depuis l'hôte vers le volume
docker run --rm -v my-data:/data -v $(pwd):/backup \
  ubuntu cp /backup/file.txt /data/
```

## 🔹 Drivers de volume

### Local (par défaut)
```bash
docker volume create \
  --driver local \
  local-data
```

### NFS
```bash
docker volume create \
  --driver local \
  --opt type=nfs \
  --opt o=addr=192.168.1.10,vers=4,soft,timeo=180,bg,tcp,rw \
  --opt device=:/export/nfs \
  nfs-data
```

### CIFS/SMB (Windows)
```bash
docker volume create \
  --driver local \
  --opt type=cifs \
  --opt device=//192.168.1.10/share \
  --opt o=username=user,password=pass,file_mode=0755,dir_mode=0755 \
  smb-data
```

## 🔹 Erreurs fréquentes

| Erreur | Solution |
|--------|----------|
| `volume not found` | Créer le volume avec `docker volume create` |
| `volume already exists` | Utiliser un nom unique ou supprimer l'existant |
| `permission denied` | Vérifier les droits du volume (chmod) |
| `transport endpoint not reachable` | Vérifier NFS/CIFS, redémarrer Docker |
| `no space left on device` | Nettoyer les volumes ou augmenter l'espace |

## 🔹 Bonnes pratiques

| Pratique | Pourquoi c'est utile |
|----------|----------------------|
| **Toujours nommer les volumes** | Facilite l'identification |
| **Utiliser des volumes pour les données** | Meilleure persistance que bind mount |
| **Sauvegarder régulièrement** | Protège contre la perte de données |
| **Documenter la structure** | Aide à comprendre la topologie |
| **Utiliser des labels** | Facilite la gestion et le filtrage |
| **Nettoyer les volumes inutilisés** | Libère l'espace disque |
| **Tester les restaurations** | Assure que les sauvegardes fonctionnent |

## 🔹 Commandes associées

```bash
# Voir les conteneurs utilisant un volume
docker container ls -a --format "table {{.Names}}\t{{.Mounts}}"

# Inspecter le conteneur pour voir les montages
docker container inspect myapp --format='{{range .Mounts}}{{.Source}} -> {{.Destination}}{{end}}'

# Lister les volumes avec détails
docker volume ls --format "table {{.Name}}\t{{.Driver}}\t{{.Mountpoint}}"

# Supprimer les ressources inutilisées
docker system prune -a --volumes
```

## 🔹 Ressources utiles
- [Documentation docker volume](https://docs.docker.com/engine/reference/commandline/volume/)
- [Volumes and bind mounts](https://docs.docker.com/storage/)
- [Dockerfile volumes](https://docs.docker.com/engine/reference/builder/#volume)
- [Docker Compose volumes](https://docs.docker.com/compose/compose-file/#volumes)

---
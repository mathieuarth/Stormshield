# 📄 Guide Docker : docker image

## 🔹 Qu'est-ce que docker image ?
`docker image` est la commande pour gérer les images Docker - les templates utilisés pour créer les conteneurs. Elle regroupe plusieurs sous-commandes pour lister, télécharger, supprimer et gérer les images.

## 🔹 Sous-commandes principales

| Commande | Description |
|----------|-------------|
| `docker image ls` | Liste les images locales |
| `docker image pull` | Télécharge une image depuis un registre |
| `docker image push` | Pousse une image vers un registre |
| `docker image rm` | Supprime une image |
| `docker image build` | Crée une image à partir d'un Dockerfile |
| `docker image inspect` | Affiche les détails d'une image |
| `docker image tag` | Ajoute un tag à une image |
| `docker image prune` | Supprime les images inutilisées |

## 🔹 docker image ls : Lister les images

### Syntaxe
```bash
docker image ls [OPTIONS] [REPOSITORY[:TAG]]
```

### Options principales
| Option | Description |
|--------|-------------|
| `-a, --all` | Affiche toutes les images (y compris intermédiaires) |
| `-q, --quiet` | Affiche uniquement les IDs |
| `--filter` | Filtre les résultats |
| `--format` | Formate la sortie |
| `--digests` | Affiche les digests |

### Exemples
```bash
# Lister les images locales
docker image ls

# Exemple de sortie :
# REPOSITORY   TAG       IMAGE ID      CREATED      SIZE
# nginx        latest    abc123def456  2 weeks ago  142MB
# ubuntu       20.04     ghi789jkl012  1 month ago  77.8MB
# postgres     13        mno345pqr678  3 weeks ago  314MB

# Lister avec tous les tags
docker image ls -a

# Afficher uniquement les IDs
docker image ls -q

# Afficher avec les digests
docker image ls --digests

# Lister les images d'un repository
docker image ls nginx

# Lister avec un tag spécifique
docker image ls ubuntu:20.04
```

### Filtrer les images
```bash
# Images non utilisées
docker image ls --filter "dangling=true"

# Images avant une certaine date
docker image ls --filter "before=ubuntu:20.04"

# Images après une certaine date
docker image ls --filter "since=ubuntu:20.04"

# Images par label
docker image ls --filter "label=version=1.0"
```

### Formats personnalisés
```bash
# Afficher la taille en MB
docker image ls --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}"

# Afficher l'ID et le repository
docker image ls --format "{{.ID}}: {{.Repository}}"

# JSON
docker image ls --format "{{json .}}"
```

## 🔹 docker image pull : Télécharger une image

### Syntaxe
```bash
docker image pull [OPTIONS] NAME[:TAG|@DIGEST]
```

### Exemples
```bash
# Télécharger une image (tag latest par défaut)
docker image pull nginx

# Télécharger une version spécifique
docker image pull ubuntu:20.04

# Télécharger depuis un registre personnalisé
docker image pull myregistry.com/myapp:1.0

# Télécharger avec les infos détaillées
docker image pull -q nginx  # Moins verbeux
```

### Cas pratiques
```bash
# Pré-charger les images pour la production
docker image pull nginx:latest
docker image pull postgres:13
docker image pull redis:7

# Script pour télécharger plusieurs images
for image in nginx postgres redis redis-cli; do
  docker image pull $image:latest
done
```

## 🔹 docker image push : Envoyer une image

### Syntaxe
```bash
docker image push [OPTIONS] NAME[:TAG]
```

### Préalables
```bash
# Taguer l'image avec le registre
docker image tag myapp:1.0 myregistry.com/myapp:1.0

# Se connecter au registre
docker login myregistry.com

# Pousser l'image
docker image push myregistry.com/myapp:1.0

# Déconnexion (optionnel)
docker logout myregistry.com
```

### Exemples
```bash
# Pousser vers Docker Hub
docker image tag myapp:1.0 username/myapp:1.0
docker image push username/myapp:1.0

# Pousser avec latest
docker image tag myapp:1.0 username/myapp:latest
docker image push username/myapp:latest

# Pousser vers un registre privé
docker image push myregistry.com/myapp:1.0
```

## 🔹 docker image rm : Supprimer une image

### Syntaxe
```bash
docker image rm [OPTIONS] IMAGE [IMAGE...]
```

### Options
| Option | Description |
|--------|-------------|
| `-f, --force` | Force la suppression |
| `--no-prune` | Ne pas supprimer les parents non étiquetés |

### Exemples
```bash
# Supprimer une image
docker image rm nginx:latest

# Supprimer plusieurs images
docker image rm ubuntu:20.04 postgres:13

# Supprimer par ID
docker image rm abc123def456

# Forcer la suppression
docker image rm -f nginx

# Supprimer toutes les images
docker image rm $(docker image ls -q)

# Supprimer toutes les images sauf une
docker image rm $(docker image ls -q --filter "reference!=nginx:latest")
```

## 🔹 docker image inspect : Détails d'une image

### Syntaxe
```bash
docker image inspect [OPTIONS] IMAGE [IMAGE...]
```

### Exemples
```bash
# Afficher tous les détails
docker image inspect nginx:latest

# Format JSON
docker image inspect nginx:latest -f '{{json .}}'

# Extraire des informations spécifiques
docker image inspect nginx:latest --format='{{.Config.Entrypoint}}'
docker image inspect nginx:latest --format='{{range .Config.Env}}{{println .}}{{end}}'
docker image inspect nginx:latest --format='{{.Config.ExposedPorts}}'
docker image inspect nginx:latest --format='{{.Size}}'

# Afficher le digest
docker image inspect nginx:latest --format='{{index .RepoDigests 0}}'
```

## 🔹 docker image tag : Taguer une image

### Syntaxe
```bash
docker image tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]
```

### Exemples
```bash
# Ajouter un tag à une image
docker image tag myapp:1.0 myapp:latest

# Taguer pour un registre
docker image tag myapp:1.0 myregistry.com/myapp:1.0

# Créer un alias
docker image tag nginx:latest my-nginx:latest

# Tags multiples
docker image tag myapp:1.0 myapp:stable
docker image tag myapp:1.0 myregistry.com/myapp:1.0
docker image tag myapp:1.0 myregistry.com/myapp:latest
```

## 🔹 docker image prune : Nettoyer les images

### Syntaxe
```bash
docker image prune [OPTIONS]
```

### Options
| Option | Description |
|--------|-------------|
| `-a, --all` | Supprime toutes les images inutilisées |
| `-f, --force` | Pas de confirmation |
| `--filter` | Filtre les images à supprimer |

### Exemples
```bash
# Supprimer les images orphelines
docker image prune

# Supprimer toutes les images inutilisées
docker image prune -a

# Forcer sans confirmation
docker image prune -af

# Filtrer les images à supprimer
docker image prune -a --filter "until=72h"

# Récupérer l'espace disque
docker image prune -af
```

## 🔹 docker image build (alias docker build)

Voir [guide_docker_builder.md](guide_docker_builder.md) pour les détails complets sur la construction d'images.

```bash
# Build simple
docker image build -t myapp:1.0 .

# Build avec plusieurs tags
docker image build -t myapp:1.0 -t myapp:latest .

# Build avec arguments
docker image build --build-arg VERSION=1.0 -t myapp:1.0 .
```

## 🔹 Cas pratiques complets

### Gestion des versions
```bash
# Tagguer une version stable
docker image tag myapp:1.0 myapp:stable

# Tagguer comme latest
docker image tag myapp:1.0 myapp:latest

# Lister les versions
docker image ls myapp

# Nettoyer les anciennes versions
docker image rm myapp:0.9 myapp:0.8
```

### Publier sur Docker Hub
```bash
# Se connecter
docker login

# Tagguer l'image
docker image tag myapp:1.0 username/myapp:1.0

# Pousser l'image
docker image push username/myapp:1.0

# Vérifier sur Docker Hub
# https://hub.docker.com/r/username/myapp
```

### Migrer entre registres
```bash
# Télécharger l'image
docker image pull old-registry.com/myapp:1.0

# Retagger
docker image tag old-registry.com/myapp:1.0 new-registry.com/myapp:1.0

# Pousser
docker image push new-registry.com/myapp:1.0

# Nettoyer
docker image rm old-registry.com/myapp:1.0
```

### Analyser l'utilisation d'espace
```bash
# Afficher la taille de chaque image
docker image ls --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}"

# Trier par taille (plus grand d'abord)
docker image ls --format "table {{.Size}}\t{{.Repository}}\t{{.Tag}}" | sort -hr

# Calculer la taille totale
docker image ls --format "{{.Size}}" | awk '{sum+=$1} END {print sum}'
```

## 🔹 Erreurs fréquentes

| Erreur | Solution |
|--------|----------|
| `image not found` | Vérifier le nom avec `docker image ls` ou `docker image pull` |
| `error building image` | Vérifier le Dockerfile et le contexte |
| `failed to push image` | Vérifier la connexion au registre et les droits |
| `image in use by container` | Arrêter les conteneurs utilisant l'image |
| `permission denied` | Utiliser sudo ou ajouter au groupe docker |

## 🔹 Bonnes pratiques

| Pratique | Pourquoi c'est utile |
|----------|----------------------|
| **Utiliser des tags explicites** | Évite les surprises avec "latest" |
| **Tagguer avec la version** | Permet une gestion précise des versions |
| **Nettoyer régulièrement** | Libère l'espace disque |
| **Utiliser des registres privés** | Sécurise les images propriétaires |
| **Tagguer pour le registre avant push** | Évite les erreurs |
| **Documenter les images** | Aide à comprendre leur contenu |
| **Utiliser des images de base légères** | Réduit la taille |

## 🔹 Commandes associées

```bash
# Voir l'historique des calques
docker image history nginx:latest

# Voir les dépendances
docker image inspect nginx:latest

# Exporter une image
docker image save nginx:latest -o nginx.tar

# Importer une image
docker image load -i nginx.tar

# Supprimer tous les conteneurs et images
docker system prune -a
```

## 🔹 Ressources utiles
- [Documentation docker image](https://docs.docker.com/engine/reference/commandline/image/)
- [Docker registries](https://docs.docker.com/docker-hub/)
- [Tagging best practices](https://docs.docker.com/engine/reference/commandline/tag/)

---
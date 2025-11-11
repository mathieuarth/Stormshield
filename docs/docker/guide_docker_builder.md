# 📄 Guide Docker : docker builder

## 🔹 Qu'est-ce que docker builder ?
`docker builder` est la commande pour gérer les builds Docker et le cache de construction. Elle permet de créer des images, gérer le cache, et optimiser le processus de build.

## 🔹 Sous-commandes principales

| Commande | Description |
|----------|-------------|
| `docker builder build` | Crée une image (alias : `docker build`) |
| `docker builder prune` | Nettoie le cache de build |

## 🔹 docker builder build : Créer une image

### Syntaxe
```bash
docker builder build [OPTIONS] PATH | URL | -
```

### Options principales
| Option | Description |
|--------|-------------|
| `-t, --tag` | Tagguer l'image |
| `-f, --file` | Spécifier le Dockerfile |
| `--build-arg` | Passer des arguments au build |
| `--cache-from` | Utiliser un cache externe |
| `--no-cache` | Ignorer le cache |
| `--progress` | Type de progression (auto, plain, tty) |
| `-o, --output` | Sortie du build |
| `--platform` | Plateforme cible |
| `--target` | Étape multi-stage ciblée |

### Exemple simple
```bash
# Build basique
docker builder build -t myapp:1.0 .

# Build à partir d'un Dockerfile nommé
docker builder build -f Dockerfile.prod -t myapp:1.0 .

# Build avec plusieurs tags
docker builder build -t myapp:1.0 -t myapp:latest .
```

## 🔹 Structure d'un Dockerfile

### Exemple basique
```dockerfile
# Image de base
FROM ubuntu:20.04

# Métadonnées
LABEL maintainer="john@example.com" version="1.0"

# Variables d'environnement
ENV APP_HOME=/app NODE_ENV=production

# Répertoire de travail
WORKDIR $APP_HOME

# Copier les fichiers
COPY package*.json ./

# Installer les dépendances
RUN npm install --production

# Copier l'application
COPY . .

# Exposer le port
EXPOSE 3000

# Commande de démarrage
CMD ["node", "server.js"]
```

## 🔹 Instructions Dockerfile courantes

| Instruction | Description |
|-------------|-------------|
| `FROM` | Image de base |
| `WORKDIR` | Répertoire de travail |
| `RUN` | Exécute une commande |
| `COPY` | Copie les fichiers |
| `ADD` | Copie/extrait les fichiers |
| `ENV` | Variables d'environnement |
| `EXPOSE` | Ports exposés |
| `CMD` | Commande par défaut |
| `ENTRYPOINT` | Point d'entrée |
| `LABEL` | Métadonnées |
| `ARG` | Arguments de build |
| `VOLUME` | Points de montage |
| `USER` | Utilisateur |

## 🔹 Build avec arguments

### Utiliser ARG
```dockerfile
# Dockerfile
ARG PYTHON_VERSION=3.9
FROM python:${PYTHON_VERSION}

ARG APP_VERSION=1.0.0
ENV VERSION=${APP_VERSION}

WORKDIR /app
COPY . .
RUN pip install -r requirements.txt

CMD ["python", "app.py"]
```

### Build avec arguments
```bash
# Passer un argument
docker builder build --build-arg PYTHON_VERSION=3.10 -t myapp:1.0 .

# Plusieurs arguments
docker builder build \
  --build-arg PYTHON_VERSION=3.10 \
  --build-arg APP_VERSION=2.0.0 \
  -t myapp:2.0 .
```

## 🔹 Multi-stage builds

### Exemple avec deux étapes
```dockerfile
# Stage 1 : Build
FROM golang:1.19 as builder
WORKDIR /app
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o myapp

# Stage 2 : Runtime (image plus légère)
FROM alpine:3.18
WORKDIR /app
COPY --from=builder /app/myapp .
EXPOSE 8080
CMD ["./myapp"]
```

### Build avec ciblage d'étape
```bash
# Build seulement la première étape
docker builder build --target builder -t myapp:build .

# Build l'image finale
docker builder build -t myapp:1.0 .
```

## 🔹 Optimisation du Dockerfile

### Exemple optimisé
```dockerfile
# ✓ BON : Regrouper les RUN
FROM ubuntu:20.04
RUN apt-get update && \
    apt-get install -y curl wget && \
    rm -rf /var/lib/apt/lists/*

# ✓ BON : Utiliser .dockerignore
# ✓ BON : Copier progressivement
COPY package*.json ./
RUN npm install
COPY . .

# ✓ BON : Utiliser une image de base légère
FROM alpine:3.18

# ✓ BON : Ordonner les couches par changement
FROM python:3.11-slim
ENV APP_HOME=/app
WORKDIR $APP_HOME
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
```

### Exemple non optimisé
```dockerfile
# ✗ MAUVAIS : Trop de RUN
FROM ubuntu:20.04
RUN apt-get update
RUN apt-get install -y curl
RUN apt-get install -y wget

# ✗ MAUVAIS : Copier au début
COPY . .
RUN npm install

# ✗ MAUVAIS : Image de base volumineuse
FROM ubuntu:20.04
```

## 🔹 .dockerignore

### Exemple
```text
.git
.gitignore
.docker
.dockerignore
.env
.env.local
node_modules
npm-debug.log
.DS_Store
dist/
build/
*.log
.idea/
.vscode/
```

### Cas d'utilisation
```bash
# Créer un .dockerignore
cat > .dockerignore << EOF
.git
node_modules
.env
*.log
EOF

# Build (ignore les fichiers)
docker builder build -t myapp:1.0 .
```

## 🔹 Options de build avancées

### Pas de cache
```bash
# Ignorer le cache existant
docker builder build --no-cache -t myapp:1.0 .
```

### Progression du build
```bash
# Afficher la progression en format plain
docker builder build --progress=plain -t myapp:1.0 .

# Format tty (par défaut)
docker builder build --progress=tty -t myapp:1.0 .
```

### Build pour plusieurs plateformes
```bash
# Nécessite buildx
docker buildx build --platform linux/amd64,linux/arm64 -t myapp:1.0 .

# Build localement
docker builder build --platform linux/amd64 -t myapp:1.0 .
```

## 🔹 Cas pratiques

### Build d'une application Node.js
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
EXPOSE 3000
CMD ["node", "server.js"]
```

```bash
# Build
docker builder build -t myapp:1.0 .

# Lancer
docker run -d -p 8080:3000 myapp:1.0
```

### Build d'une application Python
```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
EXPOSE 5000
CMD ["python", "app.py"]
```

```bash
# Build avec arguments
docker builder build \
  --build-arg PIP_INDEX_URL=https://pypi.org/simple \
  -t myapp:1.0 .
```

### Build multi-stage avec optimisation
```dockerfile
# Build stage
FROM node:18 as builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Runtime stage
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY --from=builder /app/dist ./dist
EXPOSE 3000
CMD ["node", "dist/server.js"]
```

## 🔹 docker builder prune : Nettoyer le cache

### Syntaxe
```bash
docker builder prune [OPTIONS]
```

### Options
| Option | Description |
|--------|-------------|
| `-a, --all` | Supprime tout le cache |
| `-f, --force` | Pas de confirmation |
| `--keep-state` | Garde l'état du builder |

### Exemples
```bash
# Nettoyer le cache de build
docker builder prune

# Forcer sans confirmation
docker builder prune -f

# Nettoyer tout le cache
docker builder prune -af

# Voir l'espace libéré
docker builder prune -a --verbose
```

## 🔹 Inspection et debug

### Afficher les couches
```bash
# Historique des couches
docker image history myapp:1.0

# Format détaillé
docker image history --no-trunc myapp:1.0
```

### Inspecter l'image
```bash
# Tous les détails
docker image inspect myapp:1.0

# Commande d'exécution
docker image inspect myapp:1.0 --format='{{.Config.Cmd}}'

# Variables d'environnement
docker image inspect myapp:1.0 --format='{{range .Config.Env}}{{println .}}{{end}}'
```

## 🔹 Erreurs fréquentes

| Erreur | Solution |
|--------|----------|
| `Dockerfile not found` | Vérifier le chemin du Dockerfile |
| `path does not exist` | Vérifier le contexte de build (chemin fourni) |
| `COPY failed: file not found` | Vérifier le .dockerignore ou le chemin du fichier |
| `invalid reference format` | Utiliser un tag valide (minuscules, no majuscules sauf pour le registre) |
| `build context too large` | Créer un .dockerignore pour exclure les fichiers inutiles |

## 🔹 Bonnes pratiques

| Pratique | Pourquoi c'est utile |
|----------|----------------------|
| **Utiliser des images de base légères** | Réduit la taille et les vulnérabilités |
| **Multi-stage builds** | Réduit la taille finale |
| **Grouper les RUN** | Réduit le nombre de couches |
| **Ordonner par changement** | Optimise le cache |
| **Utiliser un .dockerignore** | Réduit le contexte de build |
| **Limiter les contenus** | Améliore la sécurité |
| **Utiliser des tags explicites** | Évite les surprises avec "latest" |
| **Mettre en cache les dépendances** | Accélère les builds |

## 🔹 Performance et optimisation

### Temps de build
```bash
# Mesurer le temps
time docker builder build -t myapp:1.0 .

# Avec --progress=plain pour plus de détails
docker builder build --progress=plain -t myapp:1.0 .
```

### Taille d'image
```bash
# Vérifier la taille
docker image ls myapp

# Historique des couches
docker image history --human myapp:1.0

# Comparaison
docker image ls | grep myapp
```

## 🔹 Commandes associées

```bash
# Voir l'historique des images
docker image history myapp:1.0

# Inspecter l'image
docker image inspect myapp:1.0

# Lancer un conteneur
docker container run -d myapp:1.0

# Pousser l'image
docker image push myregistry.com/myapp:1.0

# Nettoyer les images inutilisées
docker image prune
```

## 🔹 Ressources utiles
- [Documentation docker builder](https://docs.docker.com/engine/reference/commandline/builder_build/)
- [Dockerfile reference](https://docs.docker.com/engine/reference/builder/)
- [Best practices for Dockerfiles](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
- [Multi-stage builds](https://docs.docker.com/build/building/multi-stage/)

---
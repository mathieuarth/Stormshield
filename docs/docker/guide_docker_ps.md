# 📄 Guide Docker : docker ps

## 🔹 Qu'est-ce que docker ps ?
`docker ps` est la commande de base pour lister les conteneurs Docker. Elle affiche l'état, les identifiants, les images, les ports et d'autres informations sur les conteneurs en cours d'exécution ou arrêtés.

## 🔹 Syntaxe de base

```bash
docker ps [OPTIONS]
```

## 🔹 Options principales

| Option | Description |
|--------|-------------|
| `-a, --all` | Affiche tous les conteneurs (actifs et arrêtés) |
| `-q, --quiet` | Affiche uniquement les IDs |
| `-s, --size` | Affiche la taille des conteneurs |
| `-n, --last=N` | Affiche les N derniers conteneurs |
| `--format` | Formate la sortie (go template) |
| `--filter` | Filtre les résultats |
| `--no-trunc` | Affiche les IDs complets (non tronqués) |

## 🔹 Exemples d'utilisation

### Lister les conteneurs actifs
```bash
# Affichage par défaut
docker ps

# Exemple de sortie :
# CONTAINER ID   IMAGE     COMMAND               STATUS        PORTS
# abc123def456   nginx     "nginx -g daemon"     Up 2 hours    0.0.0.0:80->80/tcp
# ghi789jkl012   postgres  "postgres"            Up 1 hour     5432/tcp
```

### Lister tous les conteneurs
```bash
# Inclure les conteneurs arrêtés
docker ps -a

# Affiche également l'état "Exited"
```

### Afficher uniquement les IDs
```bash
# Utile pour les scripts
docker ps -q

# Exemple de sortie :
# abc123def456
# ghi789jkl012
# mno345pqr678
```

### Afficher les N derniers conteneurs
```bash
# Les 5 derniers
docker ps -n 5

# Le dernier conteneur
docker ps -n 1
```

### Afficher la taille des conteneurs
```bash
# Avec taille
docker ps -s

# Affiche SIZE et VIRTUAL SIZE
```

### Afficher les IDs complets
```bash
# Sans troncature
docker ps --no-trunc

# Les IDs complets s'affichent (64 caractères au lieu de 12)
```

## 🔹 Filtrage des résultats

### Filtrer par nom
```bash
# Conteneurs dont le nom contient "web"
docker ps --filter "name=web"

# Affiche web, webapp, nginx-web, etc.
```

### Filtrer par état
```bash
# Conteneurs en cours d'exécution
docker ps --filter "status=running"

# États possibles : created, restarting, running, removing, paused, exited, dead
docker ps --filter "status=exited"
```

### Filtrer par image
```bash
# Conteneurs utilisant nginx
docker ps --filter "ancestor=nginx"

# Conteneurs utilisant une version spécifique
docker ps --filter "ancestor=postgres:13"
```

### Filtrer par label
```bash
# Conteneurs avec un label spécifique
docker ps --filter "label=env=production"

# Conteneurs avec une clé spécifique (peu importe la valeur)
docker ps --filter "label=version"
```

### Combiner plusieurs filtres
```bash
# Conteneurs actifs avec le nom "web"
docker ps --filter "name=web" --filter "status=running"

# Conteneurs arrêtés avec le label "env=dev"
docker ps -a --filter "status=exited" --filter "label=env=dev"
```

## 🔹 Formatage personnalisé

### Format simple
```bash
# Afficher uniquement le nom et l'état
docker ps --format "{{.Names}}\t{{.Status}}"

# Exemple de sortie :
# web     Up 2 hours
# db      Up 1 hour
```

### Format table personnalisée
```bash
# Colonnes personnalisées
docker ps --format "table {{.Names}}\t{{.Image}}\t{{.Status}}\t{{.Ports}}"
```

### Variables de format disponibles
| Variable | Description |
|----------|-------------|
| `{{.ID}}` | ID du conteneur |
| `{{.Image}}` | Image utilisée |
| `{{.Command}}` | Commande de démarrage |
| `{{.CreatedAt}}` | Date de création |
| `{{.RunningFor}}` | Temps depuis le démarrage |
| `{{.Status}}` | État actuel |
| `{{.Ports}}` | Ports mappés |
| `{{.Names}}` | Nom du conteneur |
| `{{.Size}}` | Taille |

### Exemples de formats
```bash
# Format JSON
docker ps --format "{{json .}}"

# Afficher l'ID et le nom
docker ps --format "{{.ID}}: {{.Names}}"

# Afficher tout sur une seule ligne
docker ps --format "{{.Names}}\t{{.Image}}\t{{.Status}}\t{{.Ports}}\t{{.Size}}"
```

## 🔹 Cas pratiques

### Vérifier si un conteneur spécifique s'exécute
```bash
# Vérifier si "monapp" s'exécute
docker ps --filter "name=monapp" --filter "status=running"

# Retourner seulement l'ID
docker ps -q --filter "name=monapp" --filter "status=running"

# Script bash
if [ -z "$(docker ps -q --filter 'name=monapp')" ]; then
    echo "Conteneur monapp non actif"
else
    echo "Conteneur monapp actif"
fi
```

### Lister tous les conteneurs avec leur utilisation mémoire
```bash
# Afficher les conteneurs avec des colonnes personnalisées
docker ps --format "table {{.Names}}\t{{.Image}}\t{{.Status}}\t{{.Size}}"

# Combiner avec docker stats pour la mémoire en temps réel
docker stats $(docker ps -q)
```

### Arrêter tous les conteneurs actifs
```bash
# Arrêter tous les conteneurs
docker stop $(docker ps -q)

# Arrêter tous les conteneurs sauf un
docker stop $(docker ps -q --filter "name!=important")
```

### Supprimer les conteneurs arrêtés
```bash
# Afficher d'abord
docker ps -a --filter "status=exited"

# Supprimer
docker rm $(docker ps -q -a --filter "status=exited")
```

## 🔹 Erreurs fréquentes

| Erreur | Solution |
|--------|----------|
| `No such container` | Utiliser `docker ps -a` pour voir tous les conteneurs |
| `--filter: unknown flag` | Utiliser `--filter` au lieu de `-f` (sauf dans certaines versions) |
| `No response from daemon` | Vérifier que le daemon Docker est actif |
| `Permission denied` | Ajouter l'utilisateur au groupe docker ou utiliser sudo |

## 🔹 Bonnes pratiques

| Pratique | Pourquoi c'est utile |
|----------|----------------------|
| **Utiliser des noms explicites** | Facilite l'identification avec `docker ps` |
| **Ajouter des labels** | Permet un filtrage efficace |
| **Surveiller régulièrement** | Détecte les conteneurs qui ne s'exécutent pas |
| **Nettoyer les conteneurs arrêtés** | Réduit la confusion et libère l'espace |
| **Utiliser des filtres** | Automatise la gestion des conteneurs |
| **Documenter la sortie** | Aide au débogage et à l'audit |

## 🔹 Commandes associées

```bash
# Obtenir l'ID d'un conteneur par nom
docker ps -q -f "name=monapp"

# Combiner avec d'autres commandes
docker exec $(docker ps -q -f "name=web") ps aux

# Afficher les logs de tous les conteneurs
docker logs $(docker ps -aq)

# Inspecter un conteneur
docker inspect $(docker ps -q -f "name=monapp")
```

## 🔹 Ressources utiles
- [Documentation docker ps](https://docs.docker.com/engine/reference/commandline/ps/)
- [Docker CLI Reference](https://docs.docker.com/engine/reference/commandline/cli/)
- [Format Template Reference](https://docs.docker.com/config/formatting/)

---
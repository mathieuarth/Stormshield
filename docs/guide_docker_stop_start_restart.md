# 📄 Guide Docker : docker stop, docker start et docker restart

## 🔹 Gestion du cycle de vie des conteneurs
Ces trois commandes permettent de contrôler l'état d'exécution des conteneurs : arrêt gracieux, redémarrage et démarrage.

## 🔹 docker stop : Arrêter un conteneur

### Syntaxe
```bash
docker stop [OPTIONS] CONTAINER [CONTAINER...]
```

### Options principales
| Option | Description |
|--------|-------------|
| `-t, --time=10` | Secondes avant de forcer l'arrêt (SIGKILL) |

### Qu'est-ce que docker stop ?
`docker stop` envoie un signal SIGTERM au conteneur, lui permettant de se fermer proprement. Si le conteneur ne s'arrête pas dans le délai imparti, un SIGKILL est envoyé.

### Exemples
```bash
# Arrêter un conteneur
docker stop monconteneur

# Arrêter plusieurs conteneurs
docker stop web db app

# Arrêter avec délai personnalisé (30 secondes)
docker stop -t 30 monconteneur

# Arrêter immédiatement (1 seconde de grâce)
docker stop -t 1 monconteneur
```

### Cas pratiques
```bash
# Arrêter tous les conteneurs actifs
docker stop $(docker ps -q)

# Arrêter tous les conteneurs sauf un
docker stop $(docker ps -q --filter "name!=important")

# Arrêter les conteneurs créés avant une heure
docker stop $(docker ps -q --filter "status=running")

# Arrêter et vérifier
docker stop monconteneur && docker ps -a --filter "name=monconteneur"
```

## 🔹 docker start : Démarrer un conteneur

### Syntaxe
```bash
docker start [OPTIONS] CONTAINER [CONTAINER...]
```

### Options principales
| Option | Description |
|--------|-------------|
| `-a, --attach` | Attache STDOUT et STDERR |
| `-i, --interactive` | Attache STDIN même si détaché |

### Qu'est-ce que docker start ?
`docker start` redémarre un conteneur qui a été arrêté. C'est différent de `docker run` qui crée un nouveau conteneur.

### Exemples
```bash
# Démarrer un conteneur arrêté
docker start monconteneur

# Démarrer plusieurs conteneurs
docker start web db app

# Démarrer et afficher les logs
docker start -a monconteneur

# Démarrer de manière interactive
docker start -i monconteneur
```

### Cas pratiques
```bash
# Redémarrer tous les conteneurs arrêtés
docker start $(docker ps -a -q --filter "status=exited")

# Démarrer les conteneurs d'une image spécifique
docker start $(docker ps -a -q --filter "ancestor=nginx")

# Démarrer et vérifier
docker start monconteneur && docker ps --filter "name=monconteneur"

# Démarrer avec logs
docker start -a web
```

## 🔹 docker restart : Redémarrer un conteneur

### Syntaxe
```bash
docker restart [OPTIONS] CONTAINER [CONTAINER...]
```

### Options principales
| Option | Description |
|--------|-------------|
| `-t, --time=10` | Secondes avant de forcer l'arrêt |

### Qu'est-ce que docker restart ?
`docker restart` est équivalent à `docker stop` suivi de `docker start`. C'est un moyen rapide de redémarrer un conteneur.

### Exemples
```bash
# Redémarrer un conteneur
docker restart monconteneur

# Redémarrer plusieurs conteneurs
docker restart web db app

# Redémarrer avec délai personnalisé
docker restart -t 30 monconteneur

# Redémarrer immédiatement
docker restart -t 1 monconteneur
```

### Cas pratiques
```bash
# Redémarrer tous les conteneurs actifs
docker restart $(docker ps -q)

# Redémarrer les conteneurs d'une application spécifique
docker restart $(docker ps -q --filter "ancestor=myapp")

# Redémarrer et attendre
docker restart web && sleep 5 && docker ps --filter "name=web"

# Redémarrer avec vérification
docker restart monconteneur && docker logs -n 20 monconteneur
```

## 🔹 Différences entre les commandes

| Commande | SIGTERM | SIGKILL | Préserve les données | État final |
|----------|---------|---------|----------------------|-----------|
| `stop` | ✓ | ✓ (après timeout) | ✓ | Exited |
| `start` | - | - | ✓ | Running |
| `restart` | ✓ | ✓ (après timeout) | ✓ | Running |

## 🔹 Cas d'utilisation pratiques

### Maintenance
```bash
# Arrêter l'application pour maintenance
docker stop myapp

# Effectuer la maintenance
# ...

# Redémarrer
docker start myapp

# Ou redémarrer directement
docker restart myapp
```

### Mise à jour
```bash
# Arrêter gracieusement
docker stop -t 30 myapp

# Mettre à jour (build, pull, etc.)
# ...

# Redémarrer
docker start myapp
```

### Monitoring et auto-recovery
```bash
# Script pour redémarrer si arrêté
while true; do
  if ! docker ps --filter "name=myapp" --format "{{.Status}}" | grep -q "Up"; then
    echo "Conteneur arrêté, redémarrage..."
    docker start myapp
  fi
  sleep 60
done
```

### Gestion de la charge
```bash
# Arrêter les conteneurs inutilisés
docker stop $(docker ps -q --filter "ancestor=dev-app")

# Redémarrer les services de production
docker restart $(docker ps -q --filter "ancestor=prod-app")
```

## 🔹 Signaux et ordre d'arrêt

### Graceful shutdown (arrêt propre)
```bash
# Le conteneur reçoit SIGTERM (signal 15)
# L'application a 10 secondes par défaut pour se fermer
docker stop monconteneur

# Avec plus de temps pour le cleanup
docker stop -t 60 monconteneur

# Logs avant et après
docker logs -n 10 monconteneur
docker stop monconteneur
docker logs -n 5 monconteneur
```

### Arrêt forcé
```bash
# Arrêt immédiat sans attendre
docker kill monconteneur

# Ou avec timeout très court
docker stop -t 1 monconteneur
```

## 🔹 Vérifier l'état des conteneurs

```bash
# Afficher tous les conteneurs avec leur état
docker ps -a

# Filtrer par état
docker ps -a --filter "status=exited"
docker ps -a --filter "status=running"

# Afficher l'état avec format personnalisé
docker ps -a --format "table {{.Names}}\t{{.Status}}"

# Vérifier un conteneur spécifique
docker inspect monconteneur --format='{{.State.Status}}'
```

## 🔹 Erreurs fréquentes

| Erreur | Solution |
|--------|----------|
| `No such container` | Vérifier le nom avec `docker ps -a` |
| `Container not running` | Utiliser `docker start` au lieu de `docker stop` |
| `Error response from daemon: cannot stop container` | Forcer l'arrêt avec `docker kill` |
| `Timeout waiting for container` | Augmenter le délai avec `-t` |

## 🔹 Bonnes pratiques

| Pratique | Pourquoi c'est utile |
|----------|----------------------|
| **Toujours utiliser stop avant start** | Permet au conteneur de nettoyer |
| **Utiliser restart pour les redémarrages** | C'est plus simple que stop + start |
| **Donner du temps pour le shutdown** | Les applications peuvent nettoyer les ressources |
| **Vérifier l'état après les opérations** | Assure que les commandes ont réussi |
| **Utiliser des scripts pour l'automatisation** | Permet la gestion en masse |
| **Monitorer les statuts** | Détecte les problèmes rapidement |

## 🔹 Scripting avancé

### Script de gestion automatique
```bash
#!/bin/bash
# Gestion automatique des conteneurs

case "$1" in
  start)
    docker start $(docker ps -a -q --filter "status=exited")
    ;;
  stop)
    docker stop $(docker ps -q)
    ;;
  restart)
    docker restart $(docker ps -q)
    ;;
  status)
    docker ps -a --format "table {{.Names}}\t{{.Status}}"
    ;;
  *)
    echo "Usage: $0 {start|stop|restart|status}"
    ;;
esac
```

### Vérifier les conteneurs au démarrage
```bash
#!/bin/bash
# Redémarrer les conteneurs critiques

CRITICAL_CONTAINERS=("db" "web" "cache")

for container in "${CRITICAL_CONTAINERS[@]}"; do
  if [ -z "$(docker ps -q --filter "name=$container")" ]; then
    echo "Démarrage de $container..."
    docker start $container
    sleep 2
  fi
done
```

## 🔹 Commandes associées

```bash
# Voir les logs après arrêt
docker logs monconteneur

# Inspecter le conteneur
docker inspect monconteneur

# Supprimer un conteneur (d'abord arrêter)
docker rm monconteneur

# Voir les événements
docker events --filter "container=monconteneur"

# Voir les processus en cours
docker top monconteneur
```

## 🔹 Ressources utiles
- [Documentation docker stop](https://docs.docker.com/engine/reference/commandline/stop/)
- [Documentation docker start](https://docs.docker.com/engine/reference/commandline/start/)
- [Documentation docker restart](https://docs.docker.com/engine/reference/commandline/restart/)
- [Container lifecycle](https://docs.docker.com/engine/containers/container-lifecycle/)

---
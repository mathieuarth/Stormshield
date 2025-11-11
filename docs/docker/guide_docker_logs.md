# 📄 Guide Docker : docker logs

## 🔹 Qu'est-ce que docker logs ?
`docker logs` permet de consulter et de suivre les logs (stdout et stderr) générés par un conteneur. C'est l'outil principal pour le débogage et le monitoring des applications Docker.

## 🔹 Syntaxe de base

```bash
docker logs [OPTIONS] CONTAINER
```

## 🔹 Options principales

| Option | Description |
|--------|-------------|
| `-f, --follow` | Suit les logs en temps réel |
| `-n, --tail` | Affiche les N dernières lignes |
| `-t, --timestamps` | Ajoute les timestamps |
| `--since` | Logs depuis une date/heure |
| `--until` | Logs jusqu'à une date/heure |
| `--details` | Affiche les métadonnées détaillées |
| `--no-stream` | Ne pas afficher la sortie en streaming |

## 🔹 Exemples d'utilisation basiques

### Afficher tous les logs
```bash
# Tous les logs du conteneur
docker logs monconteneur

# Affiche tout depuis le début du conteneur
```

### Suivre les logs en temps réel (tail -f)
```bash
# Suivre les logs
docker logs -f monconteneur

# Quitter avec Ctrl+C
```

### Afficher les N dernières lignes
```bash
# Les 10 dernières lignes
docker logs -n 10 monconteneur

# Les 100 dernières lignes
docker logs -n 100 monconteneur

# Combiner avec follow
docker logs -n 50 -f monconteneur
```

### Ajouter les timestamps
```bash
# Logs avec horodatage
docker logs -t monconteneur

# Exemple de sortie :
# 2025-11-11T10:30:45.123456789Z [INFO] Application started
# 2025-11-11T10:30:46.234567890Z [ERROR] Connection refused
```

## 🔹 Filtrer les logs par temps

### Logs depuis un moment donné
```bash
# Logs depuis 5 minutes
docker logs --since 5m monconteneur

# Logs depuis 1 heure
docker logs --since 1h monconteneur

# Logs depuis une date complète
docker logs --since 2025-11-11T10:00:00 monconteneur

# Format ISO 8601
docker logs --since 2025-11-11T09:30:00Z monconteneur
```

### Logs jusqu'à un moment donné
```bash
# Logs jusqu'à 1 heure ago
docker logs --until 1h ago monconteneur

# Combiner since et until
docker logs --since 1h --until 30m ago monconteneur
```

## 🔹 Cas pratiques

### Déboguer une application
```bash
# Afficher les erreurs (dernières lignes avec timestamps)
docker logs -n 50 -t myapp

# Suivre les logs en direct pour voir les erreurs
docker logs -f myapp

# Chercher une erreur spécifique
docker logs myapp | grep "ERROR"

# Compter les erreurs
docker logs myapp | grep -c "ERROR"
```

### Monitoring et alertes
```bash
# Logs des 10 dernières minutes
docker logs --since 10m -t myapp

# Logs de la dernière heure avec timestamps
docker logs --since 1h -t myapp | tail -100

# Suivre les logs filtrés
docker logs -f myapp | grep "WARNING"
```

### Analyser les performances
```bash
# Afficher les logs avec timestamps
docker logs -t myapp | grep "request time"

# Analyser les temps de réponse
docker logs myapp | grep "response" | tail -20
```

### Sauvegarde et archivage
```bash
# Exporter tous les logs
docker logs myapp > myapp-logs.txt

# Exporter avec timestamps
docker logs -t myapp > myapp-logs-with-timestamps.txt

# Exporter depuis une date spécifique
docker logs --since 2025-11-11T00:00:00 myapp > logs-today.txt

# Compresser les logs
docker logs myapp | gzip > myapp-logs.txt.gz
```

## 🔹 Filtrage et recherche

### Chercher des patterns spécifiques
```bash
# Afficher seulement les erreurs
docker logs myapp | grep "ERROR"

# Afficher les erreurs avec contexte (2 lignes avant et après)
docker logs myapp | grep -C 2 "ERROR"

# Chercher plusieurs patterns
docker logs myapp | grep -E "ERROR|FATAL|CRITICAL"

# Cas insensible
docker logs myapp | grep -i "error"

# Inverser la correspondance (exclure)
docker logs myapp | grep -v "DEBUG"
```

### Analyse des logs
```bash
# Compter les occurrences
docker logs myapp | grep -c "ERROR"

# Afficher les lignes uniques
docker logs myapp | sort | uniq

# Afficher les erreurs uniques avec leurs comptes
docker logs myapp | grep "ERROR" | sort | uniq -c

# Trouver les erreurs les plus fréquentes
docker logs myapp | grep "ERROR" | cut -d' ' -f4- | sort | uniq -c | sort -rn | head -10
```

## 🔹 Utilisation avec plusieurs conteneurs

### Logs de plusieurs conteneurs
```bash
# Logs de deux conteneurs
docker logs web app

# Logs de tous les conteneurs
docker logs $(docker ps -aq)

# Logs d'une application spécifique
docker logs $(docker ps -q --filter "ancestor=nginx")
```

### Suivre plusieurs conteneurs
```bash
# Suivre un conteneur
docker logs -f web

# Suivre plusieurs (moins courant)
docker logs -f web app db
```

## 🔹 Formats et parsing

### Extraction d'informations
```bash
# Extraire seulement les timestamps
docker logs -t myapp | cut -d' ' -f1

# Extraire les messages sans le timestamp
docker logs -t myapp | cut -d' ' -f2-

# Afficher seulement les lignes de démarrage
docker logs myapp | head -20
```

### JSON parsing
```bash
# Si les logs sont en JSON
docker logs myapp | jq '.message'

# Filtrer les logs JSON par champ
docker logs myapp | jq 'select(.level=="ERROR")'

# Compter les logs par niveau
docker logs myapp | jq -r '.level' | sort | uniq -c
```

## 🔹 Logging drivers avancés

### Vérifier le driver de logs
```bash
# Afficher le driver utilisé
docker inspect myapp | grep -A 5 "LogConfig"

# Exemple :
# "LogConfig": {
#   "Type": "json-file",
#   "Config": {}
# }
```

### Limitations des logs
```bash
# Changer les limites de logs (au démarrage)
docker run -d \
  --log-driver json-file \
  --log-opt max-size=10m \
  --log-opt max-file=3 \
  myapp
```

## 🔹 Erreurs fréquentes

| Erreur | Solution |
|--------|----------|
| `No such container` | Vérifier le nom avec `docker ps -a` |
| `"logs" command only works for containers with json-file or journald logging drivers` | Changer le driver de logs |
| `EOF` | Les logs du conteneur sont vides ou le conteneur a arrêté |
| `Permission denied` | Utiliser sudo ou ajouter l'utilisateur au groupe docker |

## 🔹 Bonnes pratiques

| Pratique | Pourquoi c'est utile |
|----------|----------------------|
| **Utiliser les timestamps** | Aide à identifier les problèmes de timing |
| **Surveiller régulièrement** | Détecte les erreurs rapidement |
| **Archiver les logs** | Permet une analyse historique |
| **Utiliser des patterns** | Facilite la recherche de problèmes |
| **Monitorer plusieurs conteneurs** | Donne une vue d'ensemble |
| **Limiter la taille des logs** | Évite la saturation disque |
| **Utiliser des agrégateurs** | Pour les environnements complexes |

## 🔹 Commandes associées

```bash
# Voir les détails du conteneur
docker inspect myapp

# Redémarrer le conteneur
docker restart myapp

# Vérifier l'état
docker stats myapp

# Afficher les événements
docker events --filter "container=myapp"

# Copier les logs
docker cp myapp:/var/log/app.log ./app.log
```

## 🔹 Exemple de pipeline complet

```bash
#!/bin/bash
# Analyser les logs en temps réel

CONTAINER="myapp"
KEYWORDS=("ERROR" "FATAL" "CRITICAL")

docker logs -f $CONTAINER | while read line; do
  for keyword in "${KEYWORDS[@]}"; do
    if echo "$line" | grep -q "$keyword"; then
      echo "[ALERTE] $line"
      # Envoyer une notification
      # notify-send "Docker Alert" "$line"
    fi
  done
done
```

## 🔹 Ressources utiles
- [Documentation docker logs](https://docs.docker.com/engine/reference/commandline/logs/)
- [Docker logging drivers](https://docs.docker.com/config/containers/logging/)
- [Log driver configuration](https://docs.docker.com/config/containers/logging/configure/)

---
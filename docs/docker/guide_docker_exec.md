# 📄 Guide Docker : docker exec

## 🔹 Qu'est-ce que docker exec ?
`docker exec` permet d'exécuter une commande à l'intérieur d'un conteneur en cours d'exécution. C'est l'outil principal pour interagir avec un conteneur sans le redémarrer.

## 🔹 Syntaxe de base

```bash
docker exec [OPTIONS] CONTAINER COMMAND [ARG...]
```

## 🔹 Options principales

| Option | Description |
|--------|-------------|
| `-i, --interactive` | Garde STDIN ouvert même si non attaché |
| `-t, --tty` | Alloue un pseudo-terminal |
| `-d, --detach` | Exécute en arrière-plan |
| `-u, --user` | Spécifie l'utilisateur pour la commande |
| `-w, --workdir` | Répertoire de travail |
| `-e, --env` | Définit des variables d'environnement |
| `--privileged` | Exécute avec privilèges |

## 🔹 Combinaisons courantes

### Mode interactif (-it)
```bash
# Accès shell interactif
docker exec -it monconteneur bash

# Ou avec sh
docker exec -it monconteneur sh

# Spécifier l'utilisateur
docker exec -it -u root monconteneur bash
```

### Exécuter une commande simple
```bash
# Afficher un fichier
docker exec monconteneur cat /etc/passwd

# Lister les fichiers
docker exec monconteneur ls -la /app

# Vérifier les processus
docker exec monconteneur ps aux
```

### Mode détaché (-d)
```bash
# Exécuter en arrière-plan
docker exec -d monconteneur /script.sh

# Obtenir l'ID de l'exécution
docker exec -d monconteneur sleep 60
```

## 🔹 Exemples d'utilisation

### Accéder à un conteneur
```bash
# Accès interactif avec bash
docker exec -it myapp bash

# Accès avec sh (plus universel)
docker exec -it myapp sh

# Accès en tant que root
docker exec -it -u root myapp bash

# Accès dans un répertoire spécifique
docker exec -it -w /app myapp bash
```

### Exécuter des commandes
```bash
# Vérifier les logs en temps réel
docker exec myapp tail -f /var/log/app.log

# Vérifier l'espace disque
docker exec myapp df -h

# Vérifier les connexions réseau
docker exec myapp netstat -tlnp

# Vérifier les variables d'environnement
docker exec myapp env

# Exécuter un script
docker exec myapp /usr/local/bin/backup.sh
```

### Modifier des fichiers
```bash
# Afficher un fichier de configuration
docker exec myapp cat /etc/app/config.json

# Créer un fichier
docker exec myapp sh -c 'echo "contenu" > /app/file.txt'

# Éditer (interactif)
docker exec -it myapp vi /etc/app/config.conf
```

### Gestion de base de données
```bash
# Se connecter à MySQL
docker exec -it db mysql -u root -p

# Exécuter une requête SQL
docker exec db mysql -u root -ppassword -e "SELECT * FROM table;"

# Dump PostgreSQL
docker exec db pg_dump -U postgres mydb > backup.sql

# Restore PostgreSQL
docker exec -i db psql -U postgres mydb < backup.sql
```

## 🔹 Cas pratiques

### Diagnostic d'un conteneur
```bash
# Vérifier les processus
docker exec myapp ps aux

# Vérifier les ressources
docker exec myapp top

# Vérifier le réseau
docker exec myapp ip addr show

# Vérifier les disques montés
docker exec myapp mount | grep -E "^/dev"

# Vérifier les logs applicatifs
docker exec myapp tail -100 /var/log/app.log
```

### Maintenance
```bash
# Nettoyer les caches
docker exec myapp apt-get clean
docker exec myapp npm cache clean --force

# Redémarrer un service
docker exec myapp systemctl restart nginx

# Vérifier l'état d'un service
docker exec myapp systemctl status postgresql
```

### Backup et restauration
```bash
# Backup d'une base de données
docker exec db mysqldump -u root -ppassword --all-databases > backup.sql

# Restaurer une base de données
docker exec -i db mysql -u root -ppassword < backup.sql

# Backup de fichiers
docker exec myapp tar -czf - /app/data > backup.tar.gz

# Restaurer les fichiers
docker exec -i myapp tar -xzf - -C / < backup.tar.gz
```

### Déploiement et configuration
```bash
# Installer un paquet
docker exec myapp apt-get install -y curl

# Exécuter un script d'installation
docker exec myapp /opt/app/install.sh

# Copier un fichier (indirect)
docker exec -i myapp sh -c 'cat > /config.json' < local-config.json
```

### Monitoring
```bash
# Vérifier l'utilisation mémoire
docker exec myapp free -h

# Vérifier l'utilisation CPU
docker exec myapp top -bn1 | head -n 20

# Vérifier les connexions
docker exec myapp netstat -an | grep ESTABLISHED | wc -l

# Vérifier les fichiers ouverts
docker exec myapp lsof | wc -l
```

## 🔹 Utilisation avec des pipes et redirections

### Pipes
```bash
# Chercher dans les logs
docker exec myapp grep "ERROR" /var/log/app.log

# Combiner plusieurs commandes
docker exec myapp sh -c 'ps aux | grep node'

# Compter les résultats
docker exec myapp sh -c 'ls -1 /data | wc -l'
```

### Redirections
```bash
# Redirection entrée
docker exec -i myapp mysql -u root < schema.sql

# Redirection sortie
docker exec myapp cat /app/data.json > local-data.json

# Redirection combinée
docker exec myapp sh -c 'echo "log" >> /var/log/app.log'
```

## 🔹 Gestion des erreurs

### Codes de sortie
```bash
# Vérifier le code de sortie
docker exec myapp sh -c 'command'; echo $?

# 0 = succès
# 1-125 = erreur
# 126 = pas exécutable
# 127 = commande non trouvée
# 128-255 = signal de sortie
```

### Gestion des erreurs
```bash
# Continuer même en cas d'erreur
docker exec myapp bash -c 'cmd1 || cmd2'

# Arrêter à la première erreur
docker exec myapp bash -c 'set -e; cmd1; cmd2'

# Capturer l'erreur
docker exec myapp sh -c 'command 2>&1' || echo "Erreur"
```

## 🔹 Erreurs fréquentes

| Erreur | Solution |
|--------|----------|
| `No such container` | Vérifier le nom avec `docker ps` |
| `Container not running` | Démarrer le conteneur avec `docker start` |
| `exec user process caused "exec format error"` | Vérifier le shebang du script |
| `Permission denied` | Utiliser `-u root` ou ajuster les droits |
| `command not found` | Vérifier que la commande existe dans le conteneur |

## 🔹 Bonnes pratiques

| Pratique | Pourquoi c'est utile |
|----------|----------------------|
| **Utiliser le plein ID ou un nom unique** | Évite les ambiguïtés |
| **Préférer sh à bash** | sh est plus universel dans les conteneurs |
| **Utiliser les scripts** | Pour les opérations complexes |
| **Capturer les logs** | Pour le débogage et l'audit |
| **Limiter les modifications** | Les changements disparaissent au redémarrage |
| **Utiliser docker cp** | Pour copier des fichiers plutôt que avec echo |
| **Documenter les opérations** | Facilite la maintenance |

## 🔹 Commandes associées

```bash
# Copier des fichiers
docker cp monconteneur:/app/data.txt ./data.txt
docker cp ./config.json monconteneur:/etc/config.json

# Voir les logs
docker logs monconteneur

# Obtenir des infos
docker inspect monconteneur

# Redémarrer
docker restart monconteneur

# Arrêter
docker stop monconteneur
```

## 🔹 Ressources utiles
- [Documentation docker exec](https://docs.docker.com/engine/reference/commandline/exec/)
- [Docker CLI Reference](https://docs.docker.com/engine/reference/commandline/cli/)
- [Best practices for running containers](https://docs.docker.com/develop/dev-best-practices/)

---
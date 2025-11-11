# 📄 Guide Zabbix avec Docker

## 🔹 Introduction
Zabbix est une solution de supervision open-source qui permet de surveiller différents services, serveurs et équipements réseau. Ce guide explique comment déployer Zabbix dans des conteneurs Docker.

> 📌 **Prérequis** :
> - Docker installé (voir [guide_docker.md](guide_docker.md))
> - Connaissances de base SNMP (voir [guide_snmp.md](guide_snmp.md))
> - Docker Compose installé (voir [guide_docker_compose.md](guide_docker_compose.md))

## 🔹 Architecture Zabbix

| Composant | Description |
|-----------|-------------|
| **Zabbix Server** | Composant central qui reçoit et traite les données |
| **Zabbix Frontend** | Interface web pour la configuration et la visualisation |
| **Base de données** | Stockage des données (MySQL/PostgreSQL) |
| **Zabbix Agent** | Client à installer sur les hôtes à superviser |
| **Zabbix Proxy** | (Optionnel) Collecteur intermédiaire pour sites distants |

## 🔹 Docker Compose pour Zabbix

```yaml
version: '3.8'
services:
  zabbix-database:
    image: mysql:8.0
    environment:
      - MYSQL_DATABASE=zabbix
      - MYSQL_USER=zabbix
      - MYSQL_PASSWORD=zabbix_pwd
      - MYSQL_ROOT_PASSWORD=root_pwd
    volumes:
      - ./zbx_db_data:/var/lib/mysql

  zabbix-server:
    image: zabbix/zabbix-server-mysql:ubuntu-6.0-latest
    depends_on:
      - zabbix-database
    environment:
      - DB_SERVER_HOST=zabbix-database
      - MYSQL_DATABASE=zabbix
      - MYSQL_USER=zabbix
      - MYSQL_PASSWORD=zabbix_pwd
    ports:
      - "10051:10051"
    volumes:
      - ./zbx_srv_data:/etc/zabbix
      - ./alertscripts:/usr/lib/zabbix/alertscripts
      - ./externalscripts:/usr/lib/zabbix/externalscripts

  zabbix-web:
    image: zabbix/zabbix-web-nginx-mysql:ubuntu-6.0-latest
    depends_on:
      - zabbix-server
    environment:
      - ZBX_SERVER_HOST=zabbix-server
      - DB_SERVER_HOST=zabbix-database
      - MYSQL_DATABASE=zabbix
      - MYSQL_USER=zabbix
      - MYSQL_PASSWORD=zabbix_pwd
      - PHP_TZ=Europe/Paris
    ports:
      - "80:8080"
```

## 🔹 Configuration initiale

### Ports à exposer
| Port | Utilisation |
|------|-------------|
| 80 | Interface Web Zabbix |
| 10051 | Communication Zabbix Server |
| 10050 | Communication Zabbix Agent |
| 162 | SNMP Traps (si utilisé) |

### Identifiants par défaut
```text
URL: http://localhost
Utilisateur: Admin
Mot de passe: zabbix
```

## 🔹 Supervision des conteneurs Docker

### Configuration du module Docker dans Zabbix
1. Installer le module Docker sur le serveur Zabbix :
```bash
docker exec zabbix-server apt-get update
docker exec zabbix-server apt-get install -y zabbix-module-docker
```

2. Configuration dans l'interface web :
```text
Configuration → Hosts → Create host
- Host name: Docker Host
- Templates: Template App Docker
- Interface: Agent sur le port 10050
```

### Exemple de surveillance Docker
| Métrique | Description |
|----------|-------------|
| docker.containers | Nombre total de conteneurs |
| docker.containers.running | Conteneurs en cours d'exécution |
| docker.images | Nombre d'images Docker |
| docker.memory.usage | Utilisation mémoire par conteneur |
| docker.cpu.usage | Utilisation CPU par conteneur |

## 🔹 Supervision SNMP avec Zabbix

### Configuration pour Stormshield
1. Importer les MIBs Stormshield :
```bash
docker cp stormshield.mib zabbix-server:/usr/share/snmp/mibs/
docker exec zabbix-server systemctl restart zabbix-server
```

2. Configuration dans l'interface web :
```text
Configuration → Hosts → Create host
- Host name: Stormshield-FW
- Templates: Template NET SNMP Device
- Interface: SNMP v2 avec communauté
```

## 🔹 Bonnes pratiques

| Pratique | Description |
|----------|-------------|
| **Volumes persistants** | Utiliser des volumes Docker pour les données Zabbix |
| **Variables d'environnement** | Externaliser la configuration sensible |
| **Surveillance des conteneurs** | Inclure les métriques Docker dans la supervision |
| **Sauvegardes** | Planifier des backups réguliers de la base de données |
| **Mise à l'échelle** | Utiliser des Zabbix Proxy pour les grands déploiements |
| **Sécurité** | Changer les mots de passe par défaut et limiter les accès |

## 🔹 Erreurs fréquentes

| Erreur | Solution |
|--------|----------|
| `Cannot connect to database` | Vérifier les variables d'environnement et l'état du conteneur MySQL |
| `Zabbix server is not running` | Consulter les logs du conteneur avec `docker logs zabbix-server` |
| `SNMP timeout` | Vérifier la connectivité réseau et la configuration SNMP |
| `No space left on device` | Nettoyer les vieux conteneurs et données avec `docker system prune` |

## 🔹 Commandes utiles

```bash
# Vérifier les logs du serveur Zabbix
docker logs zabbix-server

# Redémarrer les services
docker-compose restart

# Sauvegarder la base de données
docker exec zabbix-database mysqldump -u zabbix -p zabbix > backup.sql

# Vérifier l'état des conteneurs
docker-compose ps
```

## 🔹 Ressources utiles
- [Documentation officielle Zabbix](https://www.zabbix.com/documentation/)
- [Hub Docker Zabbix](https://hub.docker.com/u/zabbix/)
- [Templates Zabbix](https://share.zabbix.com/)
- [Guide des bonnes pratiques Zabbix](https://www.zabbix.com/documentation/guidelines)

---
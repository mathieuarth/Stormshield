# 📄 Guide de Docker Network

## 🔹 Qu'est-ce que Docker Network ?
Docker Network est un système de mise en réseau qui permet aux conteneurs de communiquer entre eux et avec l'hôte. Il fournit l'isolation réseau, la découverte de services et le contrôle du trafic réseau.

## 🔹 Types de réseaux Docker

| Type | Description | Use Case |
|------|-------------|----------|
| **bridge** | Réseau isolé par défaut | Applications sur la même machine |
| **host** | Partage la pile réseau de l'hôte | Haute performance, services critiques |
| **overlay** | Réseau multi-hôtes en Swarm | Clustering, orchestration |
| **macvlan** | Simule une interface MAC physique | Équipements réseau spécialisés |
| **ipvlan** | Contrôle réseau au niveau IP | Déploiements avancés |
| **none** | Aucun réseau | Conteneurs isolés |

## 🔹 Architecture du réseau bridge (défaut)

```text
Hôte Docker
├── Conteneur A (IP: 172.17.0.2)
├── Conteneur B (IP: 172.17.0.3)
└── docker0 (gateway: 172.17.0.1)
```

## 🔹 Commandes de base

| Commande | Description |
|----------|-------------|
| `docker network ls` | Liste tous les réseaux |
| `docker network create <nom>` | Crée un nouveau réseau |
| `docker network inspect <nom>` | Affiche les détails d'un réseau |
| `docker network rm <nom>` | Supprime un réseau |
| `docker network connect <réseau> <conteneur>` | Connecte un conteneur à un réseau |
| `docker network disconnect <réseau> <conteneur>` | Déconnecte un conteneur d'un réseau |

## 🔹 Créer et utiliser un réseau bridge personnalisé

### Créer un réseau
```bash
# Réseau bridge simple
docker network create monreseau

# Avec options personnalisées
docker network create \
  --driver bridge \
  --subnet 192.168.0.0/24 \
  --gateway 192.168.0.1 \
  monreseau

# Avec options avancées
docker network create \
  --driver bridge \
  --subnet 10.0.0.0/24 \
  --aux-address "host=10.0.0.10" \
  --label env=production \
  monreseau
```

### Lancer des conteneurs dans le réseau
```bash
# Lancer un conteneur dans le réseau
docker run -d --name web --network monreseau nginx

# Lancer un autre conteneur
docker run -d --name app --network monreseau ubuntu sleep 3600

# Les conteneurs se voient par nom
docker exec app ping web
```

### Vérifier les connexions
```bash
# Afficher les détails du réseau
docker network inspect monreseau

# Exemple de sortie :
# {
#   "Name": "monreseau",
#   "Driver": "bridge",
#   "Containers": {
#     "abc123...": {"Name": "web", "IPv4Address": "192.168.0.2"},
#     "def456...": {"Name": "app", "IPv4Address": "192.168.0.3"}
#   }
# }
```

## 🔹 Réseau host

```bash
# Conteneur partageant la pile réseau de l'hôte
docker run -d --name myapp --network host nginx

# Avantages :
# - Performances maximales
# - Pas de port mapping

# Inconvénients :
# - Pas d'isolation réseau
# - Risques de sécurité
```

## 🔹 Mappage de ports (port mapping)

### Port mapping simple
```bash
# Mapper le port 8080 de l'hôte vers le port 80 du conteneur
docker run -d -p 8080:80 --name web nginx

# Mapper à une interface spécifique
docker run -d -p 127.0.0.1:8080:80 --name web nginx

# Mapper plusieurs ports
docker run -d -p 8080:80 -p 443:443 --name web nginx

# Mapper une plage de ports
docker run -d -p 8000-9000:8000-9000 --name web nginx
```

### Vérifier les ports mappés
```bash
# Afficher les ports mappés
docker port web

# Lister tous les ports
docker ps
```

## 🔹 DNS dans Docker

### Découverte de service
```bash
# Créer un réseau
docker network create mynet

# Lancer un service
docker run -d --name db --network mynet postgres

# Lancer une application qui accède au service par nom
docker run -d --name app --network mynet \
  -e DB_HOST=db \
  myapp

# À l'intérieur du conteneur app :
# ping db → 172.18.0.2 (résolution automatique)
```

### Alias DNS
```bash
# Créer des alias pour un conteneur
docker network connect --alias webserver --alias www mynet web

# Depuis un autre conteneur
docker exec app nslookup webserver
docker exec app nslookup www
```

## 🔹 Cas pratique : Stack Web + Base de données

### Structure
```text
réseau : app-network
├── web (nginx)
│   - Port 80 → 8080 hôte
├── app (Node.js)
│   - Port 3000 → 3000 hôte
└── db (PostgreSQL)
    - Port 5432 (interne)
```

### Commandes
```bash
# 1. Créer le réseau
docker network create app-network

# 2. Lancer la base de données
docker run -d \
  --name db \
  --network app-network \
  -e POSTGRES_PASSWORD=secret \
  postgres:13

# 3. Lancer l'application
docker run -d \
  --name app \
  --network app-network \
  -p 3000:3000 \
  -e DB_HOST=db \
  -e DB_USER=postgres \
  -e DB_PASSWORD=secret \
  myapp:latest

# 4. Lancer le serveur web
docker run -d \
  --name web \
  --network app-network \
  -p 8080:80 \
  nginx

# 5. Vérifier les connexions
docker network inspect app-network
```

### Avec Docker Compose
```yaml
version: '3.8'

services:
  db:
    image: postgres:13
    environment:
      POSTGRES_PASSWORD: secret
    networks:
      - app-network

  app:
    image: myapp:latest
    ports:
      - "3000:3000"
    environment:
      DB_HOST: db
      DB_USER: postgres
      DB_PASSWORD: secret
    depends_on:
      - db
    networks:
      - app-network

  web:
    image: nginx:latest
    ports:
      - "8080:80"
    depends_on:
      - app
    networks:
      - app-network

networks:
  app-network:
    driver: bridge
```

## 🔹 Network configuration avancée

### Limiter la bande passante
```bash
# Créer un réseau avec limite de bande passante
docker network create \
  --driver bridge \
  --opt "com.docker.driver.mtu=1450" \
  monreseau
```

### Configurer le driver réseau
```bash
# Options de bridge
docker network create \
  --opt "bridge.name=br0" \
  --opt "com.docker.network.bridge.enable_icc=true" \
  --opt "com.docker.network.bridge.enable_ip_masquerade=true" \
  monreseau
```

### IPVLAN (réseau avancé)
```bash
# Créer un réseau IPVLAN
docker network create \
  --driver ipvlan \
  --subnet=192.168.1.0/24 \
  --gateway=192.168.1.1 \
  -o parent=eth0 \
  ipvlan-net

# Lancer un conteneur
docker run -d \
  --name web \
  --network ipvlan-net \
  --ip=192.168.1.10 \
  nginx
```

## 🔹 Dépannage réseau

### Vérifier la connectivité
```bash
# Ping entre conteneurs
docker exec app ping db

# Tester une connexion TCP
docker exec app nc -zv db 5432

# Vérifier la résolution DNS
docker exec app nslookup db

# Afficher la configuration réseau
docker exec app ip addr show
docker exec app ip route show
```

### Afficher les logs réseau
```bash
# Afficher la table de routage
docker exec app route -n

# Vérifier les interfaces réseau
docker exec app ifconfig

# Sniffer le trafic réseau
docker exec app tcpdump -i eth0
```

### Inspecter un réseau
```bash
# Afficher tous les détails
docker network inspect monreseau --verbose

# Format JSON
docker network inspect monreseau -f '{{json .}}'

# Filtrer les conteneurs
docker network inspect monreseau -f '{{range .Containers}}{{println .Name}}{{end}}'
```

## 🔹 Erreurs fréquentes

| Erreur | Solution |
|--------|----------|
| `network not found` | Créer le réseau avec `docker network create` |
| `cannot connect to service` | Vérifier que les deux conteneurs sont dans le même réseau |
| `port already allocated` | Changer le port hôte ou arrêter le conteneur existant |
| `connection refused` | Vérifier que le service écoute sur l'interface correcte (0.0.0.0) |
| `DNS resolution failed` | Vérifier le nom du conteneur et que les DNS sont activés |
| `network unreachable` | Vérifier les routes et les passerelles |

## 🔹 Bonnes pratiques

| Pratique | Pourquoi c'est utile |
|----------|----------------------|
| **Utiliser des réseaux nommés** | Meilleure organisation et facilite la maintenance |
| **Une réseau par application** | Isole les applications et améliore la sécurité |
| **Documenter la topologie** | Aide à comprendre l'architecture |
| **Utiliser des noms explicites** | Facilite la découverte de services |
| **Limiter les port mappings** | Réduit l'exposition réseau |
| **Utiliser docker-compose** | Simplifie la gestion des réseaux complexes |
| **Valider la connectivité** | Détecte les problèmes avant la production |

## 🔹 Commandes de diagnostic utiles

```bash
# Lister tous les réseaux
docker network ls

# Afficher les détails complets
docker network inspect monreseau

# Voir les conteneurs dans un réseau
docker network inspect monreseau -f '{{range .Containers}}{{.Name}}{{end}}'

# Vérifier les ports mappés
docker ps --format "table {{.Names}}\t{{.Ports}}"

# Afficher la configuration IP d'un conteneur
docker exec app ip addr show

# Afficher les connexions réseau actives
docker exec app netstat -tlnp

# Tester la connectivité
docker exec app curl -v http://web

# Afficher les informations du driver réseau
docker info | grep -A 10 "Network"
```

## 🔹 Ressources utiles
- [Documentation Docker Network](https://docs.docker.com/network/)
- [Docker Networking Guide](https://docs.docker.com/network/network-tutorial-standalone/)
- [Docker Compose Networking](https://docs.docker.com/compose/networking/)
- [Docker Network Tutorial](https://docs.docker.com/network/tutorials/)

---
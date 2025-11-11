# Guide Docker Networking Avancé

## Introduction

Docker Networking permet de connecter des containers et d'isoler le trafic réseau. Ce guide couvre les modes réseau avancés (bridge, overlay, host, macvlan), le service DNS interne, le port mapping, le load balancing, et le dépannage réseau.

## Prérequis

- Docker Engine installé et fonctionnel.
- Connaissance de base des concepts Docker et des réseaux.
- Familiarité avec les commandes réseau Linux (ip, iptables, netstat).
- (Optionnel) Docker Swarm ou Kubernetes pour overlay networks.

## 1. Modes Réseau Docker

### Bridge Network (par défaut)

Chaque container se connecte à un réseau bridge. Le trafic entre containers passe par le bridge.

```bash
# Afficher les réseaux disponibles
docker network ls

# Inspecter le réseau bridge par défaut
docker network inspect bridge

# Créer un bridge personnalisé
docker network create --driver bridge mybridge

# Lancer des containers dans le bridge personnalisé
docker run -d --name web --network mybridge -p 8080:80 nginx:alpine
docker run -d --name db --network mybridge postgres:latest
```

**Avantages** :
- Mode réseau par défaut et simple.
- Containers isolés sur le même hôte.
- DNS automatique entre containers du même réseau.

**Limitations** :
- N'existe que sur un hôte unique.
- Communication multi-hôtes impossible.

### Host Network

Le container partage la pile réseau de l'hôte. Pas d'isolation réseau.

```bash
# Lancer un container en mode host (performant mais non isolé)
docker run -d --network host --name web nginx:alpine

# Vérifier les ports sur l'hôte
sudo netstat -tulpn | grep nginx
```

**Avantages** :
- Performances maximales (pas de traduction de ports).
- Accès direct aux interfaces réseau de l'hôte.

**Limitations** :
- Aucune isolation réseau.
- Conflits de ports possibles avec l'hôte ou d'autres containers.
- Non recommandé pour des containers non fiables.

### Overlay Network (Docker Swarm)

Réseau distribué s'étendant sur plusieurs hôtes Docker Swarm.

```bash
# Initialiser Docker Swarm
docker swarm init

# Créer un overlay network
docker network create --driver overlay --attachable distributed_net

# Lancer un service sur le overlay network
docker service create \
  --name web \
  --network distributed_net \
  -p 8080:80 \
  nginx:alpine
```

**Avantages** :
- Communication multi-hôtes sécurisée (chiffrée par défaut).
- Scalabilité horizontale.
- Service discovery automatique.

**Limitations** :
- Nécessite Docker Swarm ou Kubernetes.
- Surcharge réseau due au chiffrement.

### Macvlan Network

Attribue une adresse MAC unique à chaque container, les faisant paraître comme des hôtes physiques.

```bash
# Créer un macvlan network
docker network create \
  --driver macvlan \
  --subnet 192.168.1.0/24 \
  --gateway 192.168.1.1 \
  -o parent=eth0 \
  mymacvlan

# Lancer un container avec une adresse MAC unique
docker run -d \
  --network mymacvlan \
  --ip 192.168.1.100 \
  --name isolated-app \
  nginx:alpine
```

**Avantages** :
- Chaque container a sa propre adresse MAC et IP.
- Accès direct au réseau physique.
- Utile pour les applications legacy.

**Limitations** :
- Configuration complexe.
- Compatibilité limitée sur certains réseaux.

### None Network

Aucune connexion réseau.

```bash
# Lancer un container complètement isolé
docker run -d --network none --name isolated myimage:latest

# Vérifier l'absence d'interface réseau
docker exec isolated ip addr
```

## 2. Port Mapping et Exposition

### Exposer des ports

```bash
# Mapper un port container vers la machine hôte
docker run -d -p 8080:80 --name web nginx:alpine
# Accessible à http://localhost:8080

# Mapper vers une interface spécifique
docker run -d -p 127.0.0.1:8080:80 --name internal-web nginx:alpine
# Accessible seulement en local

# Mapper plusieurs ports
docker run -d \
  -p 8080:80 \
  -p 8443:443 \
  --name secure-web nginx:alpine

# Mapper des plages de ports
docker run -d -p 5000-5010:5000-5010 --name multi-ports myapp:latest
```

### Vérifier les mappings de ports

```bash
# Lister les ports mappés d'un container
docker port web

# Inspecter complètement
docker inspect web --format='{{json .NetworkSettings.Ports}}'
```

## 3. DNS et Service Discovery

### DNS interne Docker

Les containers connectés au même réseau bridge personnalisé se découvrent automatiquement par nom :

```bash
# Créer un réseau
docker network create app-net

# Lancer deux containers dans le même réseau
docker run -d --name db --network app-net postgres:latest
docker run -d --name api --network app-net myapi:latest

# Le container 'api' peut accéder 'db' via son nom
docker exec api ping db
docker exec api curl http://db:5432  # DB écoute sur 5432
```

### Résolveur DNS personnalisé

```bash
# Spécifier un serveur DNS personnalisé
docker run -d \
  --dns 8.8.8.8 \
  --dns 8.8.4.4 \
  --name web \
  nginx:alpine

# Définir un domaine de recherche
docker run -d \
  --dns-search example.com \
  --name web \
  nginx:alpine
```

### Docker Compose avec DNS

```yaml
version: '3.3'
services:
  db:
    image: postgres:latest
    networks:
      - backend

  api:
    image: myapi:latest
    networks:
      - backend
    depends_on:
      - db
    environment:
      DATABASE_URL: postgresql://db:5432/mydb

networks:
  backend:
    driver: bridge
```

## 4. Réseaux Avancés et Load Balancing

### Round-robin DNS (services Docker)

```bash
# Dans Swarm, les services offrent un load balancing automatique
docker service create \
  --name web \
  --replicas 3 \
  -p 8080:80 \
  nginx:alpine

# Les requêtes sont réparties entre les réplicas
curl http://localhost:8080
```

### Utiliser un reverse proxy pour le load balancing

```yaml
version: '3.3'
services:
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    networks:
      - frontend
    depends_on:
      - app1
      - app2

  app1:
    image: myapp:latest
    networks:
      - frontend

  app2:
    image: myapp:latest
    networks:
      - frontend

networks:
  frontend:
    driver: bridge
```

Fichier `nginx.conf` minimal :

```nginx
events {
  worker_connections 1024;
}

http {
  upstream backend {
    server app1:8080;
    server app2:8080;
  }

  server {
    listen 80;
    location / {
      proxy_pass http://backend;
    }
  }
}
```

## 5. Isolation et Segmentation Réseau

### Connecter/Déconnecter un container d'un réseau

```bash
# Créer un container initialement sans réseau personnalisé
docker run -d --network bridge --name app myimage:latest

# Connecter à un réseau personnalisé
docker network connect mybridge app

# Maintenant, le container a deux interfaces réseau
docker inspect app --format='{{json .NetworkSettings.Networks}}'

# Déconnecter
docker network disconnect mybridge app
```

### Limiter la communication inter-containers

```bash
# Créer des réseaux isolés
docker network create frontend
docker network create backend

# Lancer les services dans des réseaux séparés
docker run -d --network frontend --name web nginx:alpine
docker run -d --network backend --name db postgres:latest

# Le container 'web' ne peut pas accéder directement à 'db'
docker exec web ping db  # Échoue
```

## 6. Dépannage Réseau

### Inspecter la configuration réseau d'un container

```bash
# Voir tous les réseaux et adresses IP
docker inspect mycontainer --format='{{json .NetworkSettings}}'

# Voir seulement l'IP
docker inspect mycontainer --format='{{.NetworkSettings.IPAddress}}'

# Voir les réseaux connectés
docker inspect mycontainer --format='{{range .NetworkSettings.Networks}}{{.NetworkMode}}: {{.IPAddress}}{{end}}'
```

### Tester la connectivité

```bash
# Pinger un autre container
docker exec app1 ping app2

# Tester la connectivité à un port spécifique (sans netcat)
docker exec app curl -v http://db:5432

# Vérifier les interfaces réseau dans le container
docker exec app ip addr
docker exec app ip route
```

### Vérifier les règles iptables et les routes

```bash
# Sur l'hôte, voir les règles iptables générées par Docker
sudo iptables -L -n | grep DOCKER

# Voir les bridges sur l'hôte
sudo brctl show

# Inspecter un bridge
sudo brctl show br-xxx

# Voir les tables de routage du container
docker exec app ip route
```

### Logging réseau avancé

```bash
# Capturer le trafic réseau du container
docker exec app tcpdump -i eth0 -w capture.pcap

# Voir les sockets ouvertes
docker exec app ss -tlnp

# Vérifier la résolution DNS
docker exec app nslookup db
docker exec app getent hosts db
```

### Problèmes courants

**Le container ne peut pas accéder au réseau**

```bash
# Vérifier que le container est connecté à un réseau
docker inspect app --format='{{json .NetworkSettings.Networks}}'

# Vérifier l'interface réseau dans le container
docker exec app ip link

# Vérifier les routes
docker exec app ip route
```

**Deux containers ne peuvent pas communiquer**

```bash
# Vérifier qu'ils sont sur le même réseau
docker inspect app1 --format='{{json .NetworkSettings.Networks}}'
docker inspect app2 --format='{{json .NetworkSettings.Networks}}'

# Tester ping/curl
docker exec app1 ping app2
docker exec app1 curl http://app2:8080
```

**Résolution DNS échoue**

```bash
# Vérifier les serveurs DNS configurés
docker exec app cat /etc/resolv.conf

# Tester la résolution
docker exec app nslookup google.com
docker exec app host example.com
```

## 7. Exemples Pratiques

### Architecture multi-tier avec Compose

```yaml
version: '3.3'

services:
  web:
    image: nginx:alpine
    ports:
      - "80:80"
    networks:
      - frontend
    depends_on:
      - api

  api:
    image: myapi:latest
    networks:
      - frontend
      - backend
    depends_on:
      - db

  db:
    image: postgres:latest
    networks:
      - backend
    environment:
      POSTGRES_PASSWORD: secret

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
```

### Microservices avec service discovery

```bash
# Créer un réseau de microservices
docker network create microservices

# Lancer plusieurs services
docker run -d --name auth --network microservices myauth:latest
docker run -d --name users --network microservices myusers:latest
docker run -d --name orders --network microservices myorders:latest

# Un service découvre les autres via DNS
docker exec orders curl http://users:8080/api/users
```

## Ressources

- Documentation Docker Networking : https://docs.docker.com/network/
- Docker Network Drivers : https://docs.docker.com/network/drivers/
- Networking Best Practices : https://docs.docker.com/network/network-tutorial-standalone/
- Docker Compose Networking : https://docs.docker.com/compose/networking/

## Guides Complémentaires

- [Guide Installation Docker Debian](../guide_docker_installation_debian.md)
- [Guide Docker](./guide_docker.md)
- [Guide Docker Networks](./guide_docker_network.md)
- [Guide Docker Compose](./guide_docker_compose.md)
- [Guide Docker Security](./guide_docker_security.md)

---

Guide complet de Docker Networking avancé couvrant bridge, overlay, host, port mapping, DNS, load balancing et dépannage.

# Guide Portainer

## Introduction

Portainer est une interface graphique légère pour gérer des environnements Docker, Docker Swarm et Kubernetes. Ce guide explique l'installation et l'exploitation de Portainer sur une machine Debian (10/11/12) : installation Docker, déploiement, sauvegarde/restauration, mise à jour et bonnes pratiques de sécurité.

## Prérequis

- Un système Debian récent (Debian 10/11/12).
- Accès sudo/root pour installer les paquets et configurer Docker.
- (Optionnel) `docker compose` (plugin) ou `docker-compose` si vous utilisez des stacks Compose.
- Espace disque pour le volume Portainer (`portainer_data`) et pour les sauvegardes.

Avant de commencer, il est recommandé d'ajouter votre utilisateur au groupe `docker` pour éviter d'utiliser `sudo` à chaque commande :

```bash
sudo usermod -aG docker $USER
# Déconnectez-vous/reconnectez-vous pour appliquer le groupe
```

## Installer Docker sur Debian (méthode recommandée)

Utilisez le dépôt officiel Docker pour obtenir les paquets à jour :

```bash
sudo apt update
sudo apt install -y apt-transport-https ca-certificates curl gnupg lsb-release
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin

# Vérifier l'installation
sudo systemctl enable --now docker
docker --version
```

Si vous préférez `docker-compose` standalone, installez-le séparément ou utilisez le plugin `docker compose` fourni avec Docker.

## Installation rapide (container standalone) — commandes Debian/Bash

Créez d'abord un volume Docker pour persister les données :

```bash
docker volume create portainer_data
```

Lancez Portainer (interface web sur le port 9000) :

```bash
docker run -d --name portainer \\
  -p 9000:9000 \\
  -p 9443:9443 \\
  --restart=always \\
  -v /var/run/docker.sock:/var/run/docker.sock \\
  -v portainer_data:/data \\
  portainer/portainer-ce:latest
```

Ouvrez ensuite l'interface à l'adresse : http://<IP_DE_LHOTE>:9000 et créez le compte admin initial.

## Installation avec Docker Compose (exemple)

Fichier `docker-compose.yml` minimal :

```yaml
version: '3.3'
services:
  portainer:
    image: portainer/portainer-ce:latest
    container_name: portainer
    command: -H unix:///var/run/docker.sock
    ports:
      - "9000:9000"
      - "9443:9443"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - portainer_data:/data
    restart: unless-stopped

volumes:
  portainer_data:
```

Lancement (avec le plugin `docker compose` ou docker-compose) :

```bash
docker compose up -d
# ou
docker-compose up -d
```

## Déploiement d'une stack (exemple)

Depuis l'UI Portainer, créez une "Stack" en collant un `docker-compose.yml`. Exemple minimal :

```yaml
version: '3.3'
services:
  nginx:
    image: nginx:alpine
    ports:
      - "8080:80"
```

## Agent (multi-hôtes / swarm)

Pour gérer plusieurs hôtes depuis une instance centrale Portainer, déployez le Portainer Agent sur chaque hôte :

```bash
docker run -d --name portainer_agent --restart=always \\
  -p 9001:9001 \\
  -v /var/run/docker.sock:/var/run/docker.sock \\
  -v /var/lib/docker/volumes:/var/lib/docker/volumes \\
  portainer/agent
```

Ajoutez ensuite cet endpoint dans l'UI Portainer (type: "Agent") en indiquant l'IP:9001.

## Sauvegarde et restauration (Debian / bash)

Portainer stocke sa configuration dans le volume `/data` (ici `portainer_data`). Sauvegardez ce volume régulièrement.

Sauvegarde (créer un tarball du volume) :

```bash
# Exécuter depuis un répertoire de backup, par ex. ~/backups
mkdir -p ~/backups
docker run --rm \\
  -v portainer_data:/data \\
  -v $(pwd):/backup \\
  alpine \\
  sh -c "cd /data && tar czf /backup/portainer_data_$(date +%F).tgz ."
```

Restauration :

```bash
# Arrêter et supprimer le container actuel
docker stop portainer && docker rm portainer

# Décompresser le tarball dans le volume (exemple depuis ~/backups)
docker run --rm -v portainer_data:/data -v ~/backups:/backup alpine sh -c "tar xzf /backup/portainer_data_backup.tgz -C /data"

# Relancer Portainer (réutilise le volume portainer_data)
docker run -d --name portainer \\
  -p 9000:9000 -p 9443:9443 \\
  --restart=always \\
  -v /var/run/docker.sock:/var/run/docker.sock \\
  -v portainer_data:/data \\
  portainer/portainer-ce:latest
```

Adaptez les chemins et noms de fichiers à votre arborescence (par ex. `/srv/backups` ou `/home/<user>/backups`).

## Mise à jour

Procédure simple : tirer la nouvelle image, arrêter l'ancien container et recréer :

```bash
docker pull portainer/portainer-ce:latest

docker stop portainer
docker rm portainer

# Relancer en réutilisant le volume portainer_data
docker run -d --name portainer \\
  -p 9000:9000 -p 9443:9443 \\
  --restart=always \\
  -v /var/run/docker.sock:/var/run/docker.sock \\
  -v portainer_data:/data \\
  portainer/portainer-ce:latest
```

Si vous utilisez `docker compose`, mettez à jour l'image et redéployez la stack via l'UI ou `docker compose pull && docker compose up -d`.

## Sécurité et bonnes pratiques (Debian)

- Ne pas exposer Portainer directement sur Internet. Placez-le derrière un reverse-proxy TLS (nginx, Traefik) ou utilisez un VPN.
- Exemple basique UFW :

```bash
sudo apt install -y ufw
sudo ufw allow from 192.168.0.0/16 to any port 9000 proto tcp   # autoriser réseau interne
sudo ufw deny 9000/tcp                                         # refuser par défaut si nécessaire
sudo ufw enable
```

- Activez TLS côté reverse-proxy ou via la configuration Portainer (HTTPS sur 9443).
- Créez des comptes utilisateurs et équipes, évitez d'utiliser le compte `admin` partagé.
- Sauvegardez régulièrement le volume `/data` et testez vos restaurations.
- Pour multi-hôtes, limitez l'accès réseau au Portainer Agent (pare-feu, réseau privé).

## Dépannage rapide

- Consulter les logs du container :

```bash
docker logs portainer --tail 200
```

- Vérifier le statut du service Docker :

```bash
sudo systemctl status docker
```

- Si Portainer ne parvient pas à accéder au socket Docker, vérifiez que le montage `/var/run/docker.sock` est présent et que l'utilisateur a les permissions nécessaires.

## Exemples utiles

- Déployer une stack depuis l'UI (coller un `docker-compose.yml`).
- Connecter un dépôt Git pour déploiements automatisés (option disponible dans Portainer).

## Guides complémentaires

Pour maîtriser Docker avant d'utiliser Portainer, consultez les guides suivants :

- [Guide Docker](./guide_docker.md) — utilisation courante de Docker.
- [Guide Installation Docker Debian](../guide_docker_installation_debian.md) — installation complète sur Debian.
- [Guide Docker Compose](./guide_docker_compose.md) — orchestration multi-containers.
- [Guide Docker Images](./guide_docker_image.md) — gestion des images Docker.
- [Guide Docker Containers](./guide_docker_container.md) — gestion des containers.
- [Guide Docker Volumes](./guide_docker_volume.md) — persistance des données.
- [Guide Docker Networks](./guide_docker_network.md) — networking entre containers.
- [Guide Docker Logs](./guide_docker_logs.md) — consultation et monitoring des logs.
- [Guide Docker PS](./guide_docker_ps.md) — lister et monitorer les containers.
- [Guide Docker Exec](./guide_docker_exec.md) — exécuter des commandes dans un container.
- [Guide Docker Stop/Start/Restart](./guide_docker_stop_start_restart.md) — gérer le cycle de vie des containers.
- [Guide Docker Builder](./guide_docker_builder.md) — construire des images personnalisées.
- [Guide Docker Transfer Images](./guide_docker_transfer_images.md) — transférer des images entre machines.
- [Guide Docker Filesystem Debian](./guide_docker_filesystem_debian.md) — système de fichiers Docker sur Debian.

## Références

- Documentation officielle Portainer : https://docs.portainer.io/
- Images Docker : https://hub.docker.com/r/portainer/portainer-ce

---



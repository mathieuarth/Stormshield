# Guide Installation Docker sur Debian

## Introduction

Ce guide explique comment installer Docker sur une machine Debian (10, 11, 12, 13) en utilisant le dépôt officiel Docker. Docker permet de containeriser et déployer des applications de manière isolée et portable. Cette installation couvre à la fois Docker Engine (le runtime) et Docker Compose (pour orchestrer multi-containers).

## Prérequis

- Un système Debian 10/11/12/13 à jour.
- Accès administrateur (sudo) ou root.
- Connexion Internet pour télécharger les paquets et l'image Docker.
- Environ 500 Mo d'espace disque (plus pour les images Docker).

Versions supportées :
- **Debian 10 (Buster)** — supporté mais version ancienne.
- **Debian 11 (Bullseye)** — version stable.
- **Debian 12 (Bookworm)** — dernière stable (recommandée).
- **Debian 13 (Trixie)** — version testing.

---

## Installation Rapide avec le Script Officiel (Recommandé)

Docker fournit un script d'installation automatisé officiel qui configure le dépôt et installe Docker en une seule commande. C'est la méthode recommandé et la plus rapide :

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

Le script détecte automatiquement votre version Debian, ajoute le dépôt Docker officiel, et installe tous les paquets nécessaires (docker-ce, docker-ce-cli, containerd.io).

**Optionnel** : ajouter votre utilisateur au groupe docker (pour utiliser Docker sans sudo) :

```bash
sudo usermod -aG docker $USER
newgrp docker
```

Vérifiez l'installation :

```bash
docker --version
docker run --rm hello-world
```

**Avantage** : une seule commande, tout est automatisé.

**Alternative** : si vous préférez contrôler chaque étape, suivez les étapes manuelles ci-dessous (Étapes 1 à 10).

---

## Étape 1 : Mettre à jour les paquets système

Avant d'installer Docker, mettez à jour votre système :

```bash
sudo apt update
sudo apt upgrade -y
```

## Étape 2 : Installer les dépendances

Installez les paquets nécessaires pour utiliser le dépôt Docker via HTTPS :

```bash
sudo apt install -y \
  apt-transport-https \
  ca-certificates \
  curl \
  gnupg \
  lsb-release
```

Détail des paquets :
- `apt-transport-https` : permet à `apt` d'utiliser des sources HTTPS.
- `ca-certificates` : certificats SSL/TLS pour valider les signatures.
- `curl` : utilitaire pour télécharger la clé GPG.
- `gnupg` : outil pour vérifier les signatures GPG.
- `lsb-release` : identifie la version Debian (pour la bonne branche du dépôt).

## Étape 3 : Ajouter la clé GPG officielle de Docker

Téléchargez et importez la clé de signature officielle Docker :

```bash
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

## Étape 4 : Ajouter le dépôt officiel Docker

Ajoutez le dépôt Docker à votre configuration apt :

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian \
  $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

Explications :
- `arch=$(dpkg --print-architecture)` : détecte automatiquement l'architecture CPU (amd64, arm64, etc.).
- `signed-by=/usr/share/keyrings/docker-archive-keyring.gpg` : utilise la clé importée.
- `$(lsb_release -cs)` : détecte la version Debian (buster, bullseye, bookworm) pour la bonne branche.
- `stable` : utilise les releases stables (par opposition à `nightly`).

## Étape 5 : Mettre à jour l'index apt

Mettez à jour l'index apt pour inclure les nouveaux paquets du dépôt Docker :

```bash
sudo apt update
```

## Étape 6 : Installer Docker Engine

Installez Docker Engine, Docker CLI et containerd :

```bash
sudo apt install -y docker-ce docker-ce-cli containerd.io
```

Explication :
- `docker-ce` : Docker Engine (Community Edition).
- `docker-ce-cli` : utilitaires en ligne de commande Docker.
- `containerd.io` : runtime de conteneurs (dépendance de Docker).

Optionnel — installer aussi Docker Compose (plugin moderne) :

```bash
sudo apt install -y docker-compose-plugin
```

## Étape 7 : Vérifier l'installation

Vérifiez que Docker a été installé correctement :

```bash
docker --version
docker ps
```

Si vous obtenez une erreur de permission, passez à l'étape 8 pour configurer le groupe docker.

## Étape 8 : Configuration post-installation (optionnel)

Par défaut, les commandes Docker nécessitent `sudo`. Pour éviter cela, ajoutez votre utilisateur au groupe `docker` :

```bash
sudo usermod -aG docker $USER
```

Activez les changements de groupe pour la session actuelle (ou déconnectez-vous/reconnectez-vous) :

```bash
newgrp docker
```

Vérifiez que tout fonctionne sans sudo :

```bash
docker ps
```

**⚠️ Sécurité** : ajouter un utilisateur au groupe docker lui donne des droits équivalents à root. Limitez ce groupe aux utilisateurs de confiance.

## Étape 9 : Activer Docker au démarrage

Configurez Docker pour démarrer automatiquement au redémarrage du système :

```bash
sudo systemctl enable docker
```

Vérifiez le statut du service :

```bash
sudo systemctl status docker
```

## Étape 10 : Tester Docker

Lancez un container test pour confirmer que Docker fonctionne :

```bash
docker run --rm hello-world
```

Vous devriez voir un message de bienvenue « Hello from Docker! ».

Pour tester avec un container interactif :

```bash
docker run -it --rm debian:bookworm /bin/bash
# Vous êtes maintenant à l'intérieur du container
cat /etc/os-release
exit
```

## Installation optionnelle : Docker Compose standalone

Si vous voulez utiliser `docker-compose` en tant que commande standalone (plutôt que `docker compose`), téléchargez le binaire :

```bash
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
docker-compose --version
```

Alternativement, installez via pip (si Python est disponible) :

```bash
sudo apt install -y python3-pip
sudo pip3 install docker-compose
```

## Mise à jour Docker

Pour mettre à jour Docker vers la dernière version :

```bash
sudo apt update
sudo apt upgrade -y docker-ce docker-ce-cli containerd.io
```

Redémarrez le service Docker :

```bash
sudo systemctl restart docker
```

## Désinstallation (si nécessaire)

Pour désinstaller Docker complètement :

```bash
# Arrêter le service Docker
sudo systemctl stop docker

# Désinstaller les paquets Docker
sudo apt purge -y docker-ce docker-ce-cli containerd.io docker-compose-plugin

# Supprimer les fichiers de configuration résiduels
sudo rm -rf /etc/docker
sudo rm -rf ~/.docker

# (Optionnel) Supprimer les images, volumes et containers résiduels
sudo rm -rf /var/lib/docker
```

## Dépannage

### Erreur : « Unable to locate package docker-ce »

**Cause** : le dépôt n'a pas été ajouté correctement ou apt n'a pas été mis à jour.

**Solution** :
```bash
sudo apt update
# Vérifiez que le dépôt est présent
cat /etc/apt/sources.list.d/docker.list
```

### Erreur : « permission denied while trying to connect to Docker daemon »

**Cause** : votre utilisateur n'a pas les permissions Docker.

**Solution** :
```bash
sudo usermod -aG docker $USER
newgrp docker
# Vérifiez :
docker ps
```

### Docker ne démarre pas au redémarrage

**Cause** : le service n'est pas activé avec systemd.

**Solution** :
```bash
sudo systemctl enable docker
sudo systemctl status docker
```

### Espace disque insuffisant

**Vérifiez l'espace utilisé par Docker** :
```bash
docker system df
```

**Libérez de l'espace** :
```bash
# Nettoyer les images, volumes et containers inutilisés
docker system prune -a
```

### Vérifier les logs Docker

```bash
sudo journalctl -u docker -n 50 --no-pager
# ou
sudo tail -f /var/log/docker.log
```

## Exemple d'utilisation

Une fois Docker installé, voici quelques commandes courantes :

```bash
# Télécharger et exécuter un container
docker run -d --name nginx -p 8080:80 nginx:alpine

# Lister les containers en cours
docker ps

# Voir les logs du container
docker logs nginx

# Arrêter un container
docker stop nginx

# Supprimer un container
docker rm nginx
```

## Guides complémentaires

Une fois Docker installé sur Debian, consultez les guides suivants pour aller plus loin :

- [Guide Docker](./guide_docker.md) — utilisation courante de Docker.
- [Guide Docker Compose](./guide_docker_compose.md) — orchestration multi-containers.
- [Guide Docker Images](./guide_docker_image.md) — gestion des images Docker.
- [Guide Docker Containers](./guide_docker_container.md) — gestion des containers.
- [Guide Docker Volumes](./guide_docker_volume.md) — persistance des données.
- [Guide Docker Networks](./guide_docker_network.md) — networking entre containers.
- [Guide Docker Logs](./guide_docker_logs.md) — consultation et monitoring des logs.
- [Guide Docker PS](./guide_docker_ps.md) — lister et monitorer les containers.
- [Guide Docker Exec](./guide_docker_exec.md) — exécuter des commandes dans un container.
- [Guide Docker Stop/Start/Restart](./guide_docker_stop_start_restart.md) — gérer le cycle de vie des containers.
- [Guide Portainer](./guide_portainer.md) — interface graphique pour gérer Docker.

## Ressources utiles

- Documentation officielle Docker : https://docs.docker.com/
- Installation sur Debian : https://docs.docker.com/engine/install/debian/
- Post-installation : https://docs.docker.com/engine/install/linux-postinstall/
- Docker Compose : https://docs.docker.com/compose/install/

---

Guide d'installation Docker pour Debian, basé sur les méthodes officielles et les bonnes pratiques.

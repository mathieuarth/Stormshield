# 📄 Guide : Structure des fichiers Docker sur Debian

## 🔹 Qu'est-ce que ce guide couvre ?
Ce guide explique l'organisation complète des fichiers de configuration, images, conteneurs, volumes et logs Docker sur un serveur Debian standard.

## 🔹 Structure générale

```text
/
├── /etc/
│   ├── docker/
│   │   ├── daemon.json (configuration principale)
│   │   ├── key.json (clé de demitrust)
│   │   └── certs.d/ (certificats registries)
│   └── default/
│       └── docker (options de démarrage)
├── /var/lib/
│   └── docker/ (données Docker)
│       ├── containers/ (conteneurs)
│       ├── images/ (couches d'images)
│       ├── volumes/ (volumes nommés)
│       ├── networks/ (configurations réseau)
│       └── buildx/ (cache de build)
├── /var/log/
│   └── docker/ (logs Docker)
├── /root/.docker/ (config utilisateur)
│   ├── config.json (auth Docker Hub)
│   └── manifests/
└── /home/user/.docker/ (config utilisateur normal)
    └── config.json
```

## 🔹 Répertoires Docker - Vue détaillée

### /etc/docker - Configuration

#### daemon.json (Configuration principale)
```text
Localisation : /etc/docker/daemon.json
Propriétaire : root:root
Permissions : 644
Description : Configuration principale du daemon Docker
Réappliable : Après redémarrage du service
```

**Exemple de contenu** :
```json
{
  "storage-driver": "overlay2",
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  },
  "debug": false,
  "insecure-registries": ["registry.example.com"],
  "registry-mirrors": ["https://mirror.example.com"]
}
```

#### ca.pem, cert.pem, key.pem (Certificats TLS)
```bash
# Afficher les fichiers
ls -la /etc/docker/

# Exemple
-rw------- 1 root root  1234 Nov 11 10:30 ca.pem
-rw------- 1 root root  2567 Nov 11 10:30 cert.pem
-rw------- 1 root root  3891 Nov 11 10:30 key.pem
```

#### /etc/docker/certs.d/ - Certificats des registries
```text
/etc/docker/certs.d/
├── localhost:5000/
│   ├── ca.crt
│   ├── client.cert
│   └── client.key
└── registry.example.com/
    ├── ca.crt
    ├── client.cert
    └── client.key
```

### /var/lib/docker - Données Docker

#### Structure complète
```bash
# Voir la structure
tree /var/lib/docker/
# Ou
find /var/lib/docker -type d -maxdepth 2

# Exemple de sortie
/var/lib/docker/
├── containers/ (16G)
├── image/ (45G)
├── volumes/ (128G)
├── networks/
├── buildx/
├── overlay2/ (ou overlay, btrfs, zfs selon le driver)
├── plugins/
├── swarm/
├── tmp/
├── runtimes/
└── engine-id
```

#### /var/lib/docker/containers - Les conteneurs
```text
/var/lib/docker/containers/
├── abc123def456.../
│   ├── config.v2.json (configuration du conteneur)
│   ├── hostname
│   ├── hosts
│   ├── resolv.conf
│   ├── checkpoints/
│   └── logs/
│       ├── abc123def456-json.log (logs du conteneur)
│       └── abc123def456-json.log.meta
└── ghi789jkl012.../
    └── ...

Taille typique : 100MB - plusieurs GB par conteneur
```

**Voir les conteneurs** :
```bash
# Lister les répertoires des conteneurs
ls -la /var/lib/docker/containers/

# Voir la config d'un conteneur
cat /var/lib/docker/containers/<CONTAINER_ID>/config.v2.json | jq .

# Voir les logs
cat /var/lib/docker/containers/<CONTAINER_ID>/logs/<CONTAINER_ID>-json.log
```

#### /var/lib/docker/image - Les images
```text
/var/lib/docker/image/
└── overlay2/ (ou le driver utilisé)
    ├── distribution/
    ├── imagedb/
    │   ├── content/
    │   │   └── sha256/ (manifests d'images)
    │   └── metadata/
    │       └── sha256/
    ├── layerdb/
    │   └── sha256/ (couches d'images)
    └── repositories.json (index des images)

Taille typique : 50MB - plusieurs centaines de GB selon les images
```

**Voir les images** :
```bash
# Lister les images (via Docker)
docker image ls

# Voir les fichiers bruts
ls -la /var/lib/docker/image/overlay2/imagedb/content/sha256/

# Voir les couches
ls -la /var/lib/docker/overlay2/
```

#### /var/lib/docker/volumes - Les volumes nommés
```text
/var/lib/docker/volumes/
├── my-data/
│   └── _data/
│       ├── file1.txt
│       ├── file2.txt
│       └── subdir/
├── postgres-data/
│   └── _data/
│       ├── global/
│       ├── base/
│       └── pg_wal/
└── redis-data/
    └── _data/
        └── dump.rdb

Taille typique : Variable (dépend du contenu)
```

**Voir les volumes** :
```bash
# Lister les volumes
docker volume ls

# Voir les détails
docker volume inspect my-data

# Accéder directement aux fichiers
ls -la /var/lib/docker/volumes/my-data/_data/

# Ou avec Docker
docker run --rm -v my-data:/data ubuntu ls -la /data
```

#### /var/lib/docker/networks - Configuration réseau
```text
/var/lib/docker/networks/
├── local/
└── <NETWORK_ID>/

Chaque réseau a une configuration JSON
```

**Voir les réseaux** :
```bash
# Via Docker
docker network ls
docker network inspect bridge

# Fichiers bruts
ls -la /var/lib/docker/networks/
cat /var/lib/docker/networks/local/<NETWORK_ID>/config.ipv
```

#### /var/lib/docker/overlay2 - Système de fichiers par couches
```text
/var/lib/docker/overlay2/
├── l/ (liens symboliques vers les couches)
│   ├── ABC123...
│   ├── DEF456...
│   └── ...
├── <LAYER_ID>/
│   ├── diff/ (fichiers de cette couche)
│   ├── link (lien vers le répertoire l/)
│   ├── lower (couches dessous)
│   └── work/ (répertoire de travail)
└── ...

Taille typique : Très variable (images + conteneurs)
```

**Analyser l'espace utilisé** :
```bash
# Espace par couche
du -sh /var/lib/docker/overlay2/*/

# Espace total
du -sh /var/lib/docker/overlay2/

# Espace total Docker
du -sh /var/lib/docker/
```

## 🔹 Logs Docker

### Logs du daemon
```bash
# Journalctl (recommandé)
journalctl -u docker

# Logs en temps réel
journalctl -u docker -f

# Depuis une période
journalctl -u docker --since "2 hours ago"

# Fichier syslog
tail -f /var/log/syslog | grep docker
```

### Logs des conteneurs
```text
/var/lib/docker/containers/<CONTAINER_ID>/logs/
├── <CONTAINER_ID>-json.log (logs stdout/stderr)
└── <CONTAINER_ID>-json.log.meta (métadonnées)

Format : JSON avec timestamps et flux
```

**Accéder aux logs** :
```bash
# Voir le fichier brut
cat /var/lib/docker/containers/<CONTAINER_ID>/<CONTAINER_ID>-json.log

# Formater le JSON
cat /var/lib/docker/containers/<CONTAINER_ID>/<CONTAINER_ID>-json.log | jq .

# Via Docker (recommandé)
docker logs <CONTAINER_ID>
```

## 🔹 Configuration utilisateur

### /root/.docker/config.json (root)
```text
Localisation : /root/.docker/config.json
Propriétaire : root:root
Permissions : 600
Contenu : Authentification Docker Hub, autres registries
```

**Exemple** :
```json
{
  "auths": {
    "https://index.docker.io/v1/": {
      "auth": "base64_encoded_credentials"
    },
    "registry.example.com": {
      "auth": "base64_encoded_credentials",
      "email": "user@example.com"
    }
  },
  "HttpHeaders": {
    "User-Agent": "Docker-Client/..."
  }
}
```

### /home/user/.docker/config.json (utilisateur normal)
```text
Localisation : /home/user/.docker/config.json
Propriétaire : user:user
Permissions : 600
```

## 🔹 Analyse de l'utilisation d'espace disque

### Voir l'espace utilisé par Docker
```bash
# Espace total
du -sh /var/lib/docker/

# Par composant
du -sh /var/lib/docker/containers/
du -sh /var/lib/docker/image/
du -sh /var/lib/docker/volumes/
du -sh /var/lib/docker/overlay2/

# Avec Docker
docker system df

# Détaillé
docker system df -v

# Exemple de sortie :
# TYPE            TOTAL     ACTIVE    SIZE      RECLAIMABLE
# Images          5         2         1.256GB   800MB
# Containers      10        3         2.345GB   1.5GB
# Volumes         8         2         5.678GB   2GB
# Build cache     -         -         1.234GB   1.234GB
```

### Nettoyer l'espace
```bash
# Supprimer les images inutilisées
docker image prune -a -f

# Supprimer les conteneurs arrêtés
docker container prune -f

# Supprimer les volumes inutilisés
docker volume prune -f

# Supprimer le cache de build
docker builder prune -a -f

# Nettoyer tout
docker system prune -a --volumes -f
```

## 🔹 Accès aux fichiers directement

### Permissions par défaut
```bash
# Voir les permissions
ls -la /var/lib/docker/

# Typiquement :
drwx--x--x 18 root root /var/lib/docker/

# Cela signifie :
# - Root peut lire, écrire, exécuter
# - Groupe docker (si présent) peut accéder
# - Autres ne peuvent rien faire
```

### Accéder aux fichiers
```bash
# Avec sudo
sudo ls -la /var/lib/docker/containers/

# Si l'utilisateur est dans le groupe docker
ls -la /var/lib/docker/containers/

# Voir le groupe
groups $USER
```

## 🔹 Fichiers de démarrage et configuration

### /etc/default/docker
```bash
# Localisation : /etc/default/docker
# Contenu : Options de démarrage du service

# Exemple
DOCKER_OPTS="--storage-driver overlay2 --log-driver json-file"
```

### /etc/systemd/system/docker.service.d/
```bash
# Répertoire pour les overrides systemd
ls -la /etc/systemd/system/docker.service.d/

# Exemple : /etc/systemd/system/docker.service.d/override.conf
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd -H unix:// --log-driver=json-file
```

## 🔹 Chemins spécifiques importants

| Chemin | Description | Taille typique |
|--------|-------------|----------------|
| `/etc/docker/daemon.json` | Config principale | < 1KB |
| `/var/lib/docker/` | Toutes les données | 50GB+ |
| `/var/lib/docker/containers/` | Conteneurs | Variable |
| `/var/lib/docker/image/` | Métadonnées images | 50MB - 1GB |
| `/var/lib/docker/volumes/` | Volumes | Variable |
| `/var/lib/docker/overlay2/` | Couches et deltas | 50GB+ |
| `/var/log/docker/` | Logs Docker | Variable |
| `/root/.docker/config.json` | Auth Docker | < 1KB |

## 🔹 Exemple d'arborescence réelle

```bash
# Voir la structure réelle
tree -L 2 /var/lib/docker/

# Exemple :
/var/lib/docker/
├── containers (142G)
│   ├── abc123/ (5G)
│   ├── def456/ (2G)
│   └── ghi789/ (3G)
├── image (1.2G)
│   └── overlay2/
│       ├── imagedb/
│       ├── layerdb/
│       └── repositories.json
├── volumes (256G)
│   ├── db-data/ (150G)
│   ├── app-logs/ (50G)
│   └── cache-data/ (56G)
├── overlay2 (300G)
│   ├── l/ (symlinks)
│   ├── layer1/ (50G)
│   ├── layer2/ (45G)
│   └── ...
├── networks/
├── buildx/
├── plugins/
├── swarm/
└── engine-id
```

## 🔹 Commandes utiles pour explorer

```bash
# Voir tout
sudo ls -laR /var/lib/docker/ | head -100

# Voir la taille des conteneurs
sudo du -sh /var/lib/docker/containers/*/

# Voir la taille des volumes
sudo du -sh /var/lib/docker/volumes/*/

# Voir la taille des images
sudo du -sh /var/lib/docker/overlay2/*/

# ID des conteneurs
docker ps -a --format "table {{.ID}}\t{{.Names}}"

# Mapper les répertoires aux conteneurs
for dir in /var/lib/docker/containers/*/; do
  id=$(basename $dir)
  name=$(docker inspect --format='{{.Name}}' ${id:0:12} 2>/dev/null | sed 's|/||')
  echo "$name: $id"
done
```

## 🔹 Sauvegarde et restauration

### Sauvegarder la configuration Docker
```bash
# Sauvegarder /etc/docker/
sudo tar czf docker-config-$(date +%Y%m%d).tar.gz /etc/docker/

# Sauvegarder /var/lib/docker/
sudo tar czf docker-data-$(date +%Y%m%d).tar.gz /var/lib/docker/

# Note : Nécessite d'arrêter Docker d'abord
sudo systemctl stop docker
sudo tar czf docker-full-backup-$(date +%Y%m%d).tar.gz /var/lib/docker/
sudo systemctl start docker
```

### Restaurer
```bash
# Arrêter Docker
sudo systemctl stop docker

# Restaurer les données
sudo tar xzf docker-data-YYYYMMDD.tar.gz -C /

# Redémarrer Docker
sudo systemctl start docker

# Vérifier
docker image ls
docker ps -a
```

## 🔹 Erreurs courantes et localisation

### Où trouver les informations

| Problème | Où chercher | Comment |
|----------|------------|---------|
| Image corrompue | `/var/lib/docker/image/` | `docker image inspect` |
| Volume manquant | `/var/lib/docker/volumes/` | `docker volume inspect` |
| Logs du conteneur | `/var/lib/docker/containers/<ID>/logs/` | `docker logs` |
| Config invalide | `/etc/docker/daemon.json` | `journalctl -u docker` |
| Espace disque | `/var/lib/docker/` | `du -sh` |
| Erreur de démarrage | `journalctl -u docker` | Voir les logs |

## 🔹 Bonnes pratiques

| Pratique | Pourquoi |
|----------|---------|
| **Utiliser `/var/lib/docker/` standard** | Localisation prévisible |
| **Ne pas éditer les fichiers manuellement** | Risque de corruption |
| **Utiliser les commandes Docker** | Plus sûr que l'accès direct |
| **Sauvegarder régulièrement** | Protection contre la perte |
| **Monitorer l'espace disque** | Évite la saturation |
| **Documenter la structure** | Aide au débogage |
| **Utiliser les logs Docker** | Plus fiable que les fichiers bruts |
| **Restreindre les permissions** | Sécurité |

## 🔹 Déplacer le répertoire Docker

### Situation : Disque trop plein
```bash
# 1. Arrêter Docker
sudo systemctl stop docker

# 2. Créer le nouveau répertoire
sudo mkdir -p /mnt/large-disk/docker

# 3. Copier les données
sudo rsync -av /var/lib/docker/ /mnt/large-disk/docker/

# 4. Modifier la configuration
sudo nano /etc/docker/daemon.json

# Ajouter ou modifier :
{
  "data-root": "/mnt/large-disk/docker"
}

# 5. Redémarrer Docker
sudo systemctl start docker

# 6. Vérifier
docker system df
docker image ls

# 7. Nettoyer l'ancien répertoire (si tout fonctionne)
sudo rm -rf /var/lib/docker/
```

## 🔹 Ressources utiles
- [Docker filesystem documentation](https://docs.docker.com/storage/)
- [Docker daemon configuration](https://docs.docker.com/engine/daemon/manage-daemon/)
- [Directory structure guide](https://docs.docker.com/engine/docker-overview/)
- [Storage drivers](https://docs.docker.com/storage/storagedriver/select-storage-driver/)

---
# 📄 Guide : Transférer des images Docker entre serveurs Debian sans Internet

## 🔹 Qu'est-ce que ce guide couvre ?
Ce guide explique comment exporter et importer des images Docker d'un serveur vers un autre, notamment pour des environnements déconnectés d'Internet (air-gapped environments).

## 🔹 Scénario typique

```text
Serveur 1 (connecté à Internet)    Serveur 2 (sans accès Internet)
├── Docker avec images             ├── Docker vide
├── Accès à Internet               ├── Pas d'accès Internet
└── Stockage disponible            └── Besoin des images
```

## 🔹 Méthodes de transfert

| Méthode | Vitesse | Facilité | Compression |
|---------|---------|----------|-------------|
| **docker save/load** | Rapide | Facile | Non |
| **docker save + gzip** | Moyen | Facile | Oui |
| **docker save + tar** | Moyen | Très facile | Optionnel |
| **registry privé** | Très rapide | Complexe | Non |
| **Transfert direct SSH** | Très rapide | Moyen | Non |

## 🔹 Méthode 1 : Export/Import simple avec docker save/load

### Étape 1 : Vérifier les images sur le serveur 1
```bash
# Sur le serveur 1 (avec Internet)
docker image ls

# Exemple de sortie :
# REPOSITORY          TAG       IMAGE ID      CREATED        SIZE
# nginx               latest    abc123def456  2 weeks ago    142MB
# postgres            13        ghi789jkl012  1 month ago    314MB
# ubuntu              20.04     mno345pqr678  3 weeks ago    77.8MB
```

### Étape 2 : Exporter une image
```bash
# Sur le serveur 1
docker save nginx:latest -o nginx-latest.tar

# Vérifier la création
ls -lh nginx-latest.tar

# Exemple :
# -rw-r--r-- 1 root root 142M Nov 11 10:30 nginx-latest.tar
```

### Étape 3 : Exporter plusieurs images
```bash
# Exporter plusieurs images dans un fichier
docker save \
  nginx:latest \
  postgres:13 \
  ubuntu:20.04 \
  -o images-backup.tar

# Ou en boucle
for image in nginx:latest postgres:13 ubuntu:20.04; do
  docker save $image -o ${image//\//-}.tar
done
```

### Étape 4 : Transférer le fichier
```bash
# Option 1 : Avec SCP
scp -r nginx-latest.tar user@serveur2:/tmp/

# Option 2 : Avec rsync
rsync -avz nginx-latest.tar user@serveur2:/tmp/

# Option 3 : Avec clé USB / disque externe
cp nginx-latest.tar /media/usb/

# Vérifier le transfert
ssh user@serveur2 'ls -lh /tmp/*.tar'
```

### Étape 5 : Importer sur le serveur 2
```bash
# Sur le serveur 2
docker load -i /tmp/nginx-latest.tar

# Vérifier l'import
docker image ls | grep nginx

# Exemple de sortie :
# REPOSITORY          TAG       IMAGE ID      CREATED        SIZE
# nginx               latest    abc123def456  2 weeks ago    142MB
```

## 🔹 Méthode 2 : Export/Import avec compression (plus rapide)

### Exporter et compresser
```bash
# Sur le serveur 1
docker save nginx:latest | gzip > nginx-latest.tar.gz

# Vérifier la taille
ls -lh nginx-latest.tar.gz

# Comparaison
# Avant compression : 142MB
# Après compression : 45MB (exemple)
```

### Exporter plusieurs images compressées
```bash
# Script complet
#!/bin/bash

IMAGES=("nginx:latest" "postgres:13" "ubuntu:20.04")
OUTPUT_DIR="/backup/docker-images"

mkdir -p $OUTPUT_DIR

for image in "${IMAGES[@]}"; do
  filename="${image//\//-}.tar.gz"
  echo "Exporting $image..."
  docker save "$image" | gzip > "$OUTPUT_DIR/$filename"
  echo "Compressed size: $(ls -lh $OUTPUT_DIR/$filename | awk '{print $5}')"
done

# Créer un manifeste
cat > "$OUTPUT_DIR/manifest.txt" << EOF
Images exported on $(date)
$(ls -lh $OUTPUT_DIR/*.tar.gz | awk '{print $9, "Size:", $5}')
EOF
```

### Importer avec décompression
```bash
# Sur le serveur 2
docker load -i <(gunzip -c nginx-latest.tar.gz)

# Ou en deux étapes
gunzip -c nginx-latest.tar.gz | docker load

# Importer plusieurs images
for file in *.tar.gz; do
  echo "Loading $file..."
  docker load -i <(gunzip -c "$file")
done
```

## 🔹 Méthode 3 : Transfert direct via SSH

### Exporter et importer en une commande
```bash
# Sur le serveur 1
docker save nginx:latest | \
  ssh user@serveur2 'docker load'

# Avec compression
docker save nginx:latest | \
  gzip | \
  ssh user@serveur2 'gunzip | docker load'

# Avec plusieurs images
docker save nginx:latest postgres:13 | \
  gzip | \
  ssh user@serveur2 'gunzip | docker load'
```

### Bénéfices
- ✓ Pas de fichier intermédiaire
- ✓ Transfert direct et rapide
- ✓ Nécessite une connexion SSH
- ✓ Moins d'espace disque utilisé

## 🔹 Méthode 4 : Registry privé (pour plusieurs serveurs)

### Configuration sur le serveur 1
```bash
# Lancer un registry local
docker run -d \
  --name registry \
  -p 5000:5000 \
  -v registry-data:/var/lib/registry \
  registry:2

# Vérifier
curl http://localhost:5000/v2/_catalog
# Réponse : {"repositories":[]}
```

### Taguer et pousser les images
```bash
# Taguer une image pour le registry local
docker tag nginx:latest localhost:5000/nginx:latest

# Pousser l'image
docker push localhost:5000/nginx:latest

# Vérifier
curl http://localhost:5000/v2/_catalog
# Réponse : {"repositories":["nginx"]}

# Pousser plusieurs images
for image in nginx:latest postgres:13 ubuntu:20.04; do
  docker tag $image localhost:5000/$image
  docker push localhost:5000/$image
done
```

### Exporter le registry
```bash
# Exporter le volume du registry
docker run --rm \
  -v registry-data:/data \
  -v $(pwd):/backup \
  ubuntu tar czf /backup/registry-backup.tar.gz /data

# Ou copier directement
docker cp registry:/var/lib/registry ./registry-data
```

### Restaurer sur le serveur 2
```bash
# Décompresser le backup
tar xzf registry-backup.tar.gz

# Lancer le registry
docker run -d \
  --name registry \
  -p 5000:5000 \
  -v $(pwd)/data:/var/lib/registry \
  registry:2

# Configurer Docker pour le registry non-HTTPS
mkdir -p /etc/docker/certs.d/localhost:5000
# Sur older Docker versions
echo '{"insecure-registries" : ["localhost:5000"]}' > /etc/docker/daemon.json
systemctl restart docker

# Puller les images
docker pull localhost:5000/nginx:latest
docker pull localhost:5000/postgres:13
```

## 🔹 Cas pratiques complets

### Cas 1 : Exporter une image simple
```bash
# Serveur 1
docker image pull nginx:latest
docker save nginx:latest -o nginx.tar

# Transférer
scp nginx.tar user@serveur2:/tmp/

# Serveur 2
docker load -i /tmp/nginx.tar
docker run -d -p 80:80 --name web nginx
```

### Cas 2 : Stack application complète
```bash
# Serveur 1 : Sauvegarder l'application et dépendances
#!/bin/bash

IMAGES=(
  "nginx:latest"
  "postgres:13"
  "redis:7"
  "myapp:1.0"
)

BACKUP_DIR="/backups/docker-$(date +%Y%m%d)"
mkdir -p $BACKUP_DIR

echo "Exporting images..."
docker save "${IMAGES[@]}" | gzip > $BACKUP_DIR/app-stack.tar.gz

echo "Image size: $(ls -lh $BACKUP_DIR/app-stack.tar.gz | awk '{print $5}')"

# Créer un manifeste
cat > $BACKUP_DIR/README.txt << 'EOF'
Application Stack Backup
Exported: $(date)

Images:
- nginx:latest (web server)
- postgres:13 (database)
- redis:7 (cache)
- myapp:1.0 (application)

Import:
docker load -i <(gunzip -c app-stack.tar.gz)

Run:
docker-compose up -d
EOF

# Transférer
scp -r $BACKUP_DIR user@serveur2:/backups/
```

### Cas 3 : Migration vers serveur air-gapped
```bash
# Serveur 1 : Exporter tout
#!/bin/bash

# Exporter toutes les images
docker save $(docker image ls -q) | \
  gzip > all-images-$(date +%Y%m%d).tar.gz

# Exporter les volumes (si nécessaire)
for volume in $(docker volume ls -q); do
  docker run --rm -v $volume:/data -v $(pwd):/backup \
    ubuntu tar czf /backup/$volume.tar.gz /data
done

# Créer un script d'installation pour serveur 2
cat > restore.sh << 'EOF'
#!/bin/bash
echo "Restoring Docker images..."
docker load -i <(gunzip -c all-images-*.tar.gz)

echo "Restoring volumes..."
for volume_file in *.tar.gz; do
  [ "$volume_file" != "all-images-*.tar.gz" ] && \
  docker volume create ${volume_file%.tar.gz} && \
  docker run --rm \
    -v ${volume_file%.tar.gz}:/data \
    -v $(pwd):/backup \
    ubuntu tar xzf /backup/$volume_file -C /data --strip-components=1
done

echo "Done! Run 'docker image ls' to verify"
EOF

chmod +x restore.sh
```

## 🔹 Vérification et validation

### Vérifier les images avant export
```bash
# Lister toutes les images
docker image ls

# Vérifier la taille
docker image ls --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}"

# Calculer la taille totale
docker image ls --format "{{.Size}}" | \
  awk '{sum+=$1} END {print "Total:", sum}'
```

### Vérifier après import
```bash
# Serveur 2 : Vérifier l'import
docker image ls

# Comparer les IDs
docker image inspect nginx:latest --format='{{.ID}}'

# Vérifier l'intégrité en lançant un conteneur
docker run --rm nginx:latest nginx -v
```

## 🔹 Performance et optimisation

### Comparaison des méthodes

```bash
# Méthode 1 : Simple (sans compression)
time docker save nginx:latest > nginx.tar
# Exemple : 5 secondes

# Méthode 2 : Avec gzip
time docker save nginx:latest | gzip > nginx.tar.gz
# Exemple : 12 secondes (mais fichier 70% plus petit)

# Méthode 3 : Avec pigz (parallèle)
time docker save nginx:latest | pigz > nginx.tar.gz
# Exemple : 8 secondes (plus rapide)

# Méthode 4 : SSH direct
time docker save nginx:latest | gzip | ssh user@serveur2 'gunzip | docker load'
# Exemple : 15 secondes (pas de fichier intermédiaire)
```

### Optimisation avec pigz (compression parallèle)
```bash
# Installer pigz
apt-get install pigz

# Utiliser pigz pour une compression plus rapide
docker save nginx:latest | pigz -p 4 > nginx.tar.gz

# Décompresser
unpigz nginx.tar.gz -p 4
docker load -i nginx.tar
```

## 🔹 Automatisation

### Script de sauvegarde quotidienne
```bash
#!/bin/bash
# backup-docker-images.sh

BACKUP_DIR="/backups/docker"
RETENTION_DAYS=7

mkdir -p $BACKUP_DIR

# Créer un dossier avec la date
BACKUP_DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_PATH="$BACKUP_DIR/backup_$BACKUP_DATE"
mkdir -p $BACKUP_PATH

# Exporter les images
echo "Backing up Docker images..."
docker save $(docker image ls -q) | gzip > $BACKUP_PATH/images.tar.gz

# Exporter les volumes
echo "Backing up Docker volumes..."
for volume in $(docker volume ls -q); do
  docker run --rm -v $volume:/data -v $BACKUP_PATH:/backup \
    ubuntu tar czf /backup/$volume.tar.gz /data
done

# Créer un manifeste
echo "Creating manifest..."
cat > $BACKUP_PATH/manifest.txt << EOF
Backup Date: $BACKUP_DATE
Images: $(docker image ls -q | wc -l)
Volumes: $(docker volume ls -q | wc -l)

Images:
$(docker image ls --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}")

Volumes:
$(ls $BACKUP_PATH/*.tar.gz | grep -v images | xargs -I {} basename {} | sort)
EOF

# Nettoyer les anciennes sauvegardes
find $BACKUP_DIR -type d -name "backup_*" -mtime +$RETENTION_DAYS -exec rm -rf {} \;

echo "Backup complete!"
ls -lh $BACKUP_PATH/
```

### Script de restauration
```bash
#!/bin/bash
# restore-docker-images.sh

BACKUP_FILE=$1

if [ -z "$BACKUP_FILE" ]; then
  echo "Usage: $0 <backup_file.tar.gz>"
  exit 1
fi

echo "Restoring from $BACKUP_FILE..."

# Restaurer les images
docker load -i <(gunzip -c $BACKUP_FILE)

echo "Restore complete!"
docker image ls
```

## 🔹 Erreurs fréquentes

| Erreur | Solution |
|--------|----------|
| `Error response from daemon: insufficient disk space` | Vérifier l'espace disque et nettoyer avec `docker system prune -a` |
| `Image layer not found` | Vérifier l'intégrité du fichier tar |
| `Permission denied` | Utiliser sudo ou ajouter l'utilisateur au groupe docker |
| `No space left on device` | Augmenter l'espace disque ou utiliser la compression |
| `Connection refused` | Vérifier que le docker daemon est actif |
| `Unauthorized` | Vérifier la configuration du registry |

## 🔹 Bonnes pratiques

| Pratique | Pourquoi c'est utile |
|----------|----------------------|
| **Vérifier avant l'export** | Assure que toutes les images sont présentes |
| **Compresser les images** | Réduit la taille du transfert |
| **Créer un manifeste** | Documente ce qui a été sauvegardé |
| **Tester la restauration** | Assure que les backups sont valides |
| **Utiliser des noms explicites** | Facilite l'identification |
| **Documenter la procédure** | Aide lors d'une future restauration |
| **Conserver les backups** | Permet une restauration ultérieure |
| **Nettoyer les anciens backups** | Libère l'espace disque |

## 🔹 Checklist de migration

```bash
# Sur le serveur 1
☐ Vérifier l'espace disque disponible
☐ Lister les images à exporter
☐ Exporter les images
☐ Vérifier la taille du fichier
☐ Tester la décompression localement

# Transfert
☐ Transférer le fichier
☐ Vérifier l'intégrité du transfert (md5sum)

# Sur le serveur 2
☐ Vérifier l'espace disque disponible
☐ Importer les images
☐ Vérifier que les images sont présentes
☐ Lancer un conteneur test
☐ Nettoyer les fichiers temporaires

# Post-migration
☐ Documenter les images importées
☐ Archiver le backup
☐ Supprimer les fichiers temporaires
```

## 🔹 Vérification d'intégrité

```bash
# Créer une checksum avant le transfert
md5sum all-images.tar.gz > all-images.tar.gz.md5

# Vérifier après le transfert
md5sum -c all-images.tar.gz.md5

# Alternative avec sha256
sha256sum all-images.tar.gz > all-images.tar.gz.sha256
sha256sum -c all-images.tar.gz.sha256
```

## 🔹 Ressources utiles
- [Docker save documentation](https://docs.docker.com/engine/reference/commandline/save/)
- [Docker load documentation](https://docs.docker.com/engine/reference/commandline/load/)
- [Docker image transfer guide](https://docs.docker.com/engine/reference/commandline/image_save/)
- [Air-gapped environments](https://docs.docker.com/engine/offline/)

---
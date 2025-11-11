# 📄 Guide d'agrandissement de disque ext4 sur Debian

## 🔹 Qu'est-ce qu'ext4 ?
ext4 (fourth extended filesystem) est le système de fichiers par défaut sur Debian et la plupart des distributions Linux. Il offre de meilleures performances et une meilleure fiabilité qu'ext3, avec support des volumes de grande taille et de la fragmentation réduite.

## 🔹 Caractéristiques ext4

| Caractéristique | Description |
|-----------------|-------------|
| **Journalisation** | Protège les données en cas de crash |
| **Extents** | Améliore les performances pour les gros fichiers |
| **Timestamps** | Précision jusqu'à la nanoseconde |
| **Taille maximale** | Jusqu'à 16 TB par volume |
| **Fichiers max** | Jusqu'à 4 milliards de fichiers |

## 🔹 Prérequis

- Accès root ou droits sudo
- Espace disque disponible
- Connaissances de base Linux
- Sauvegarde des données (recommandé)

## 🔹 Trois approches pour agrandir ext4

### Approche 1 : Avec LVM (recommandée)
Si ext4 est sur un volume LVM, voir [guide_lvm_debian.md](guide_lvm_debian.md)

### Approche 2 : Redimensionner la partition directement
Pour les partitions non-LVM (plus risqué)

### Approche 3 : Ajouter un nouveau disque
Sans redimensionner l'existant

## 🔹 Approche 1 : Avec LVM

### Étapes complètes
```bash
# 1. Vérifier l'état actuel
df -h /chemin/du/point/montage
lvdisplay

# 2. Créer un PV sur le nouveau disque
sudo pvcreate /dev/sdb

# 3. Ajouter au VG
sudo vgextend nom_du_vg /dev/sdb

# 4. Étendre le LV
sudo lvextend -L +50G /dev/nom_du_vg/nom_du_lv

# 5. Redimensionner ext4 (en ligne)
sudo resize2fs /dev/nom_du_vg/nom_du_lv

# 6. Vérifier
df -h /chemin/du/point/montage
```

## 🔹 Approche 2 : Redimensionner sans LVM

⚠️ **Attention** : Procédure plus risquée, faire une sauvegarde avant !

### Prérequis
- Partition non montée (ou montée en lecture seule)
- Espace libre sur le disque physique
- Utiliser GParted ou fdisk

### Étapes
```bash
# 1. Vérifier l'état
fdisk -l
df -h

# 2. Démonter le système de fichiers (si possible)
sudo umount /dev/sda2

# 3. Utiliser parted pour redimensionner
sudo parted /dev/sda
# Dans parted :
# > print
# > resizepart 2 200GB
# > quit

# 4. Redimensionner ext4
sudo resize2fs /dev/sda2

# 5. Remonter et vérifier
sudo mount /dev/sda2 /point/montage
df -h
```

## 🔹 Approche 3 : Ajouter un nouveau disque

### Étapes
```bash
# 1. Identifier le nouveau disque
lsblk
sudo fdisk -l

# 2. Créer une partition (si nécessaire)
sudo fdisk /dev/sdb
# Créer une partition primaire, puis w pour sauvegarder

# 3. Formater en ext4
sudo mkfs.ext4 /dev/sdb1

# 4. Créer un point de montage
sudo mkdir -p /data

# 5. Monter le disque
sudo mount /dev/sdb1 /data

# 6. Rendre persistant (ajouter à /etc/fstab)
echo '/dev/sdb1 /data ext4 defaults 0 2' | sudo tee -a /etc/fstab

# 7. Vérifier
df -h
```

## 🔹 Cas pratique : Augmenter / (racine)

### Situation
- Partition / sur /dev/sda1 avec LVM
- Espace insuffisant

### Commandes
```bash
# 1. Vérifier l'état
sudo df -h /
sudo lvdisplay | grep -A5 "LV Name"
sudo vgdisplay

# 2. Ajouter un nouveau disque à la VM et rebooter

# 3. Créer un PV
sudo pvcreate /dev/sdb

# 4. Ajouter au groupe de volume racine
sudo vgextend debian-vg /dev/sdb

# 5. Étendre le LV root
sudo lvextend -l +100%FREE /dev/debian-vg/root

# 6. Redimensionner ext4 en ligne
sudo resize2fs /dev/debian-vg/root

# 7. Vérifier
sudo df -h /
```

## 🔹 Redimensionner ext4 en ligne vs hors ligne

### En ligne (recommandé)
```bash
# Le système de fichiers reste monté et utilisé
sudo resize2fs /dev/nom_du_device

# Avantage : Pas d'interruption de service
# Inconvénient : Plus lent
```

### Hors ligne (plus rapide mais risqué)
```bash
# 1. Démonter le système de fichiers
sudo umount /dev/nom_du_device

# 2. Vérifier l'intégrité
sudo fsck.ext4 -f /dev/nom_du_device

# 3. Redimensionner
sudo resize2fs /dev/nom_du_device

# 4. Remonter
sudo mount /dev/nom_du_device /point/montage
```

## 🔹 Vérifier et optimiser ext4

### Informations sur le système de fichiers
```bash
# Afficher les informations ext4
sudo tune2fs -l /dev/nom_du_device

# Vérifier la taille
sudo dumpe2fs /dev/nom_du_device | grep "Block count"

# Vérifier l'utilisation des inodes
df -i /point/montage
```

### Défragmentation (rarement nécessaire)
```bash
# Analyse de fragmentation
sudo e4defrag -c /dev/nom_du_device

# Défragmenter
sudo e4defrag -v /dev/nom_du_device
```

## 🔹 Erreurs fréquentes

| Erreur | Solution |
|--------|----------|
| `Device is busy` | Démonter le système de fichiers ou utiliser resize2fs en ligne |
| `No space left on device` | Ajouter plus d'espace disque avant de redimensionner |
| `Superblock invalid` | Faire une sauvegarde et utiliser `fsck` pour réparer |
| `Unexpected inconsistency` | Utiliser `fsck.ext4 -y` pour réparer automatiquement |
| `Bad magic number` | Vérifier que le chemin du device est correct |
| `Filesystem would shrink` | Ne pas redimensionner plus petit que l'espace utilisé |

## 🔹 Bonnes pratiques

| Pratique | Pourquoi c'est utile |
|----------|----------------------|
| **Sauvegarde avant** | Protège les données en cas de problème |
| **Laisser 10-20% libre** | Améliore les performances et la stabilité |
| **Vérifier l'intégrité** | Détecte les erreurs avant qu'elles deviennent graves |
| **Utiliser LVM si possible** | Offre plus de flexibilité |
| **Monitorer l'espace** | Évite les saturation inattendues |
| **Mettre à jour fstab** | Assure la persistance après reboot |

## 🔹 Commandes de diagnostic utiles

```bash
# État de l'espace disque
df -h
du -sh /chemin

# Informations inode
df -i

# Montages actuels
mount | grep ext4

# Fichiers ouvertes sur un device
lsof | grep /dev/sda

# Vérifier la journalisation
sudo tune2fs -l /dev/nom_du_device | grep -i "journal"

# Logs des problèmes
dmesg | grep -i ext4
journalctl -u systemd-fsck
```

## 🔹 Configuration avancée

### Optimiser ext4
```bash
# Activer les features modernes
sudo tune2fs -O has_journal,ext_attr,resize_inode,dir_index /dev/nom_du_device

# Modifier les paramètres de bloc
sudo tune2fs -b 4096 /dev/nom_du_device

# Changer le comportement du journal
sudo tune2fs -o journal_data_writeback /dev/nom_du_device
```

### Mount options optimisées
```bash
# Dans /etc/fstab, utiliser :
/dev/sda1 / ext4 defaults,relatime,discard 0 1

# relatime : Réduit les écritures de timestamps
# discard : Pour les SSD (TRIM)
```

## 🔹 Ressources utiles
- [Documentation ext4](https://www.kernel.org/doc/html/latest/filesystems/ext4/index.html)
- [Man tune2fs](https://man7.org/linux/man-pages/man8/tune2fs.8.html)
- [Man resize2fs](https://man7.org/linux/man-pages/man8/resize2fs.8.html)
- [Guide e2fsprogs](https://e2fsprogs.sourceforge.net/)

---
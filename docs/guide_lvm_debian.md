# 📄 Guide d'agrandissement de disque LVM sur Debian

## 🔹 Qu'est-ce que LVM ?
LVM (Logical Volume Manager) est un gestionnaire de volumes logiques qui permet de gérer les disques et partitions de manière flexible. Il facilite l'agrandissement et la réduction des volumes sans intervention complexe sur les partitions physiques.

## 🔹 Architecture LVM

| Composant | Description |
|-----------|-------------|
| **PV (Physical Volume)** | Le disque physique ou partition à utiliser |
| **VG (Volume Group)** | Ensemble de PV regroupés en un pool de stockage |
| **LV (Logical Volume)** | Partition logique créée dans un VG |
| **PE (Physical Extent)** | Unité d'allocation physique |

### Hiérarchie
```text
Disque physique (/dev/sda, /dev/sdb)
    ↓
Physical Volume (PV)
    ↓
Volume Group (VG)
    ↓
Logical Volume (LV) → Système de fichiers
```

## 🔹 Prérequis

- Accès root ou droits sudo
- Un nouveau disque ou espace non alloué
- Connaissances de base Linux

## 🔹 Étapes pour agrandir un disque LVM

### 1. Ajouter un nouveau disque à la VM
```bash
# Dans l'hyperviseur (ESXi, KVM, VirtualBox, etc.)
# Ajouter un nouveau disque à la VM et redémarrer
```

### 2. Identifier le nouveau disque
```bash
# Lister tous les disques
lsblk

# Ou utiliser fdisk
sudo fdisk -l

# Ou utiliser parted
sudo parted -l
```

Exemple de sortie :
```text
sda      8:0    0  100G  0 disk
sdb      8:16   0   50G  0 disk    # Nouveau disque
vg0-root 253:0  0  100G  0 lvm
```

### 3. Créer un Physical Volume (PV)
```bash
# Créer un PV sur le nouveau disque
sudo pvcreate /dev/sdb

# Vérifier la création
sudo pvdisplay
```

### 4. Étendre le Volume Group (VG)
```bash
# Ajouter le PV au VG existant
sudo vgextend nom_du_vg /dev/sdb

# Vérifier l'extension
sudo vgdisplay
```

### 5. Étendre le Logical Volume (LV)
```bash
# Étendre le LV (exemple : augmenter de 50G)
sudo lvextend -L +50G /dev/nom_du_vg/nom_du_lv

# Ou pour utiliser tout l'espace disponible
sudo lvextend -l +100%FREE /dev/nom_du_vg/nom_du_lv

# Vérifier l'extension
sudo lvdisplay
```

### 6. Redimensionner le système de fichiers

#### Pour ext4 :
```bash
# Redimensionner en ligne (sans umount)
sudo resize2fs /dev/nom_du_vg/nom_du_lv

# Vérifier le résultat
df -h
```

#### Pour XFS :
```bash
# Redimensionner XFS
sudo xfs_growfs /chemin/du/point/montage

# Vérifier le résultat
df -h
```

## 🔹 Cas pratique complet

### Exemple de commandes pour augmenter /home
```bash
# 1. Vérifier l'état actuel
df -h
lvdisplay
lsblk

# 2. Créer le PV sur le nouveau disque
sudo pvcreate /dev/sdb

# 3. Ajouter au VG (supposons que le VG s'appelle 'debian-vg')
sudo vgextend debian-vg /dev/sdb

# 4. Étendre le LV /home
sudo lvextend -L +50G /dev/debian-vg/home

# 5. Redimensionner le système de fichiers
sudo resize2fs /dev/debian-vg/home

# 6. Vérifier
df -h
```

## 🔹 Erreurs fréquentes

| Erreur | Solution |
|--------|----------|
| `Device /dev/sdb not found` | Le disque n'est pas reconnu, reboot ou scanner les nouveaux disques |
| `Physical volume already exists` | Le disque contient déjà un PV, utiliser `pvcreate --force` |
| `Insufficient free space in VG` | Ajouter plus de PV ou réduire la taille requise |
| `resize2fs: Device is busy` | Ne pas utiliser si le volume est monté, ou utiliser online |
| `XFS is not mounted` | Vérifier que le point de montage est correct |

## 🔹 Bonnes pratiques

| Pratique | Pourquoi c'est utile |
|----------|----------------------|
| **Faire une sauvegarde** | Protège les données en cas de problème |
| **Laisser de l'espace libre** | Évite la saturation et améliore les performances |
| **Documenter la structure** | Facilite la maintenance et le débogage |
| **Tester avant de redimensionner** | Vérifie les commandes sur un système de test |
| **Augmenter par étapes** | Réduit les risques en cas d'erreur |
| **Vérifier les logs** | `dmesg` et `/var/log/syslog` montrent les erreurs |

## 🔹 Commandes de diagnostic utiles

```bash
# Voir la structure LVM complète
sudo pvdisplay
sudo vgdisplay
sudo lvdisplay

# Voir l'espace utilisé par système de fichiers
df -h

# Voir l'utilisation des inodes
df -i

# Vérifier l'intégrité d'ext4 (sans montage)
sudo fsck -n /dev/nom_du_vg/nom_du_lv

# Vérifier les disques physiques
sudo fdisk -l

# Scanner les nouveaux disques sans reboot
echo "- - -" | sudo tee /sys/class/scsi_host/host*/scan
```

## 🔹 Réduction de disque LVM (procédure inverse)

⚠️ **Attention** : Réduction risquée, faire une sauvegarde avant !

```bash
# 1. Réduire le système de fichiers (exemple : 20G)
sudo resize2fs /dev/nom_du_vg/nom_du_lv 20G

# 2. Réduire le LV
sudo lvreduce -L 20G /dev/nom_du_vg/nom_du_lv

# 3. Retirer le PV du VG
sudo vgreduce nom_du_vg /dev/sdb

# 4. Supprimer le PV
sudo pvremove /dev/sdb
```

## 🔹 Ressources utiles
- [Documentation LVM Linux](https://sourceware.org/lvm2/)
- [Linux Foundation LVM Guide](https://wiki.ubuntu.com/Lvm)
- [Debian LVM Documentation](https://wiki.debian.org/LVM)
- [Man pages : lvm, pvcreate, vgextend, lvextend](https://man7.org/linux/man-pages/man8/lvm.8.html)

---
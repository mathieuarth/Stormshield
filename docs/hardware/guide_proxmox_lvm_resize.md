# 📄 Guide d'agrandissement de disque Debian LVM sur Proxmox

## 🔹 Qu'est-ce que Proxmox ?
Proxmox Virtual Environment (PVE) est une plateforme open-source de virtualisation qui gère les machines virtuelles (KVM) et les conteneurs (LXC). Il offre une interface web pour gérer les ressources des VMs.

## 🔹 Architecture Proxmox avec LVM

| Composant | Description |
|-----------|-------------|
| **VM Debian** | Machine virtuelle sur Proxmox |
| **Disque VM** | Stockage alloué à la VM dans Proxmox |
| **LVM Debian** | Logical Volume Manager dans le système Debian |
| **ext4** | Système de fichiers Linux |

## 🔹 Prérequis

- Accès administrateur à Proxmox
- VM Debian avec disque LVM
- Accès SSH à la VM Debian
- Espace disponible sur le stockage Proxmox

## 🔹 Processus complet : 5 phases

### Phase 1 : Augmenter la taille du disque dans Proxmox

#### Via l'interface web Proxmox
```text
1. Aller dans Datacenter → [Nœud] → [VM]
2. Sélectionner Hardware → Double-cliquer sur le disque (ex: scsi0)
3. Entrer la nouvelle taille
4. Cliquer "Resize disk"
5. La VM n'a pas besoin de redémarrer
```

#### Via la ligne de commande Proxmox
```bash
# Sur le nœud Proxmox
qm resize <VM_ID> scsi0 +50G

# Exemple : augmenter le disque 100 de 50G
qm resize 100 scsi0 +50G
```

### Phase 2 : Détecter le nouveau disque dans Debian

#### Option A : Rebooter la VM (plus simple)
```bash
sudo reboot
```

#### Option B : Scanner sans rebooter
```bash
# Sur la VM Debian, scanner les disques SCSI
echo "- - -" | sudo tee /sys/class/scsi_host/host*/scan

# Vérifier avec parted
sudo parted -l

# Ou avec lsblk
lsblk
```

### Phase 3 : Étendre la partition physique

⚠️ **Important** : Cette étape dépend de votre partitionnement

#### Cas courant : Proxmox utilise une seule partition
```bash
# Vérifier la structure
sudo parted -l /dev/sda

# Exemple de sortie :
# Number  Start   End     Size    Type     File system
# 1      1049kB  1074MB  1073MB  primary  ext4
# 2      1074MB  50GB    48.9GB  logical  lvm
```

#### Étendre la partition LVM
```bash
# Si parted est disponible
sudo parted /dev/sda
# Dans parted :
# > resizepart 2 100%
# > quit

# Ou avec fdisk (plus fastidieux)
sudo fdisk /dev/sda
# Supprimer la partition 2, la recréer plus grande, puis w
```

#### Alternative : Sans modifier la partition
Si la partition 2 occupe déjà tout l'espace, passer à la phase 4

### Phase 4 : Étendre le Physical Volume (PV) LVM

```bash
# Vérifier les PV actuels
sudo pvdisplay

# Redimensionner le PV (si la partition a changé)
sudo pvresize /dev/sda2

# Vérifier l'extension
sudo pvdisplay
```

### Phase 5 : Étendre le Volume Group et Logical Volume

```bash
# Lister les volumes
sudo vgdisplay
sudo lvdisplay

# Exemple de noms courants :
# VG: debian-vg
# LV: root, swap, etc.

# Étendre le LV root avec tout l'espace libre
sudo lvextend -l +100%FREE /dev/debian-vg/root

# Ou augmenter d'une taille spécifique
sudo lvextend -L +50G /dev/debian-vg/root

# Redimensionner le système de fichiers ext4
sudo resize2fs /dev/debian-vg/root

# Vérifier
df -h /
```

## 🔹 Cas pratique complet : VM Proxmox Debian

### Scénario
- VM ID: 100
- Ancien disque: 50GB
- Nouveau disque: 100GB
- LVM avec partition racine

### Commandes complètes

#### Sur Proxmox (node shell)
```bash
# Augmenter le disque
qm resize 100 scsi0 +50G

# Arrêter et redémarrer la VM (optionnel)
qm reboot 100
```

#### Sur Debian (SSH)
```bash
# 1. Détecter le nouveau disque
echo "- - -" | sudo tee /sys/class/scsi_host/host*/scan
lsblk

# 2. Vérifier la structure
sudo parted -l /dev/sda

# 3. Augmenter la partition (si nécessaire)
sudo parted /dev/sda resizepart 2 100%

# 4. Redimensionner le PV
sudo pvresize /dev/sda2

# 5. Vérifier les infos LVM
sudo vgdisplay
sudo lvdisplay

# 6. Étendre le LV root
sudo lvextend -l +100%FREE /dev/debian-vg/root

# 7. Redimensionner ext4
sudo resize2fs /dev/debian-vg/root

# 8. Vérifier
df -h /
lvdisplay
```

## 🔹 Problèmes courants avec Proxmox

### Problème 1 : Le disque ne s'agrandit pas dans la VM

**Solution** :
```bash
# Redémarrer la VM depuis Proxmox
qm reboot <VM_ID>

# Ou arrêter et redémarrer
qm shutdown <VM_ID>
qm start <VM_ID>
```

### Problème 2 : La partition LVM n'apparaît pas plus grande

**Solution** :
```bash
# Scanner manuellement
echo "- - -" | sudo tee /sys/class/scsi_host/host*/scan

# Attendre quelques secondes
sleep 2

# Vérifier
lsblk
```

### Problème 3 : Erreur lors de parted resizepart

**Solution** :
```bash
# Utiliser fdisk à la place
sudo fdisk /dev/sda

# Exemple (remplacer par vos numéros):
# Command: d (delete)
# Partition: 2
# Command: n (new)
# Partition: 2 (same as before)
# First: (default)
# Last: (default or +remaining)
# Command: w (write)
```

## 🔹 Vérifier l'extension à chaque étape

```bash
# État disque
lsblk
sudo fdisk -l /dev/sda

# État LVM
sudo pvdisplay
sudo vgdisplay
sudo lvdisplay

# État système de fichiers
df -h
df -i

# Espace libre
sudo lvs
```

## 🔹 Erreurs fréquentes

| Erreur | Solution |
|--------|----------|
| `Device /dev/sda2 not found` | Scanner : `echo "- - -" \| sudo tee /sys/class/scsi_host/host*/scan` |
| `Partition table being edited` | Redémarrer la VM |
| `Unexpected inconsistency` | Utiliser `sudo fsck.ext4 -y /dev/debian-vg/root` |
| `Insufficient free space` | L'espace n'a pas été alloué au VG |
| `resize2fs: Device is busy` | Utiliser en ligne : `sudo resize2fs /dev/debian-vg/root` |

## 🔹 Bonnes pratiques

| Pratique | Pourquoi c'est utile |
|----------|----------------------|
| **Snapshot avant** | Permet une restauration en cas de problème |
| **Augmenter progressivement** | Réduit les risques et les temps d'opération |
| **Vérifier à chaque étape** | Détecte les erreurs rapidement |
| **Documenter la structure** | Aide pour les prochaines opérations |
| **Sauvegarder les données critiques** | Protège contre les accidents |
| **Tester en dev d'abord** | Valide la procédure avant production |

## 🔹 Snapshot Proxmox (avant opération)

```bash
# Sur Proxmox, créer un snapshot
qm snapshot <VM_ID> <snapshot_name>

# Exemple
qm snapshot 100 before-resize

# Lister les snapshots
qm listsnapshot 100

# Restaurer si nécessaire
qm rollback <VM_ID> <snapshot_name>
```

## 🔹 Optimisation des performances

### Après redimensionnement, optimiser ext4
```bash
# Analyser la fragmentation
sudo e4defrag -c /dev/debian-vg/root

# Vérifier la santé du système de fichiers
sudo tune2fs -l /dev/debian-vg/root | grep -E "Filesystem|Block count"
```

## 🔹 Automatisation avec script

```bash
#!/bin/bash
# Script Proxmox pour agrandir une VM

VM_ID=100
NEW_SIZE="+50G"
PARTITION="/dev/sda2"
VG_NAME="debian-vg"
LV_NAME="root"

# Sur Proxmox
qm resize $VM_ID scsi0 $NEW_SIZE

# Attendre quelques secondes
sleep 5

# SSH dans la VM et exécuter
ssh debian@vm_ip << 'EOF'
echo "- - -" | sudo tee /sys/class/scsi_host/host*/scan
sleep 2
sudo parted -m /dev/sda resizepart 2 100% || true
sudo pvresize $PARTITION
sudo lvextend -l +100%FREE /dev/$VG_NAME/$LV_NAME
sudo resize2fs /dev/$VG_NAME/$LV_NAME
df -h /
EOF
```

## 🔹 Ressources utiles
- [Documentation Proxmox](https://pve.proxmox.com/wiki/Main_Page)
- [Guide LVM Debian](guide_lvm_debian.md)
- [Guide ext4 Debian](guide_ext4_debian.md)
- [Man qm (Proxmox command)](https://pve.proxmox.com/wiki/Manual:_qm(1))

---
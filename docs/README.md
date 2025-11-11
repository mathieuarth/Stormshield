# Documentation Stormshield

Bienvenue dans la documentation Stormshield. Ce répertoire contient des guides complets et pratiques sur Docker, Debian, SNMP, infrastructure et outils connexes.

## 📋 Table des Matières

- [Guides Docker](#guides-docker)
- [Installation et Configuration Système](#installation-et-configuration-système)
- [Monitoring et Observabilité](#monitoring-et-observabilité)
- [Infrastructure](#infrastructure)
- [Formats et Outils](#formats-et-outils)

---

## 🐳 Guides Docker

Les guides Docker couvrent l'installation, utilisation, sécurité et administration.

### Installation et Configuration

- **[Installation Docker sur Debian](./guide_docker_installation_debian.md)** — Guide complet d'installation Docker sur Debian (10/11/12/13), configuration post-installation, dépannage.

### Utilisation Quotidienne

- **[Guide Docker](./docker/guide_docker.md)** — Commandes courantes Docker, images, containers, volumes, réseaux.
- **[Guide Docker Images](./docker/guide_docker_image.md)** — Gestion des images Docker, création, tagging, push/pull.
- **[Guide Docker Containers](./docker/guide_docker_container.md)** — Gestion des containers, lancement, arrêt, inspection.
- **[Guide Docker Compose](./docker/guide_docker_compose.md)** — Orchestration multi-containers, stacks, services.

### Networking et Volumes

- **[Guide Docker Networks](./docker/guide_docker_network.md)** — Réseaux Docker basiques, bridge, host, port mapping.
- **[Guide Docker Networking Avancé](./docker/guide_docker_networking_advanced.md)** — Modes réseau avancés, overlay, macvlan, DNS, load balancing, dépannage réseau.
- **[Guide Docker Volumes](./docker/guide_docker_volume.md)** — Persistance des données, volumes, bind mounts, drivers.

### Opérations et Monitoring

- **[Guide Docker Logs](./docker/guide_docker_logs.md)** — Consultation des logs Docker, drivers de logging, streaming.
- **[Guide Docker PS](./docker/guide_docker_ps.md)** — Lister et monitorer les containers en exécution, filtrage, statistiques.
- **[Guide Docker Exec](./docker/guide_docker_exec.md)** — Exécuter des commandes dans un container, débogage.
- **[Guide Docker Stop/Start/Restart](./docker/guide_docker_stop_start_restart.md)** — Gérer le cycle de vie des containers.

### Avancé

- **[Guide Docker Builder](./docker/guide_docker_builder.md)** — Construire des images avec buildkit, optimisations, multi-stage builds.
- **[Guide Docker Transfer Images](./docker/guide_docker_transfer_images.md)** — Transférer des images entre machines, export/import, registry.
- **[Guide Docker Filesystem Debian](./docker/guide_docker_filesystem_debian.md)** — Système de fichiers Docker sur Debian, storage drivers, quotas.
- **[Guide Docker Security](./docker/guide_docker_security.md)** — Bonnes pratiques de sécurité, utilisateurs, isolation, scanning, capacités Linux, limites ressources.

### Outils Complémentaires

- **[Guide Portainer](./docker/guide_portainer.md)** — Interface graphique pour gérer Docker, installation, stacks, multi-hôtes, sauvegarde.

---

## 🖥️ Installation et Configuration Système

Guides pour l'installation et la configuration de systèmes d'exploitation et services.

- **[Guide Debian Ext4](./guide_ext4_debian.md)** — Système de fichiers Ext4 sur Debian, optimisations, maintenance.
- **[Guide LVM Debian](./guide_lvm_debian.md)** — Gestion des volumes logiques (LVM) sur Debian, partitionnement flexible.
- **[Guide Proxmox LVM Resize](./guide_proxmox_lvm_resize.md)** — Redimensionner les volumes LVM sur Proxmox.
- **[Guide Git](./guide_git.md)** — Contrôle de version avec Git, commandes courantes, workflows.

---

## 📊 Monitoring et Observabilité

Guides pour le monitoring, logging et alertes.

- **[Guide SNMP](./guide_snmp.md)** — Simple Network Management Protocol, configuration, MIBs, monitoring.
- **[Guide SNMP Trap](./guide_snmptrap.md)** — Configuration des alertes SNMP Trap, réception d'événements.
- **[Guide Zabbix Docker](./guide_zabbix_docker.md)** — Déployer Zabbix dans Docker pour le monitoring centralisé.

---

## 🏗️ Infrastructure

Guides pour l'infrastructure et virtualisation.

- **[Guide Proxmox LVM Resize](./guide_proxmox_lvm_resize.md)** — Redimensionner les volumes LVM sur Proxmox VE.

---

## 🛠️ Formats et Outils

Guides sur les formats de données et outils de configuration.

- **[Guide YAML](./guide_yaml.md)** — Format YAML, syntaxe, utilisation dans Docker Compose, Kubernetes, Ansible.

---

## 📁 Structure du Répertoire

```
docs/
├── README.md                                    # Ce fichier
├── guide_docker_installation_debian.md          # Installation Docker sur Debian
├── guide_git.md                                 # Git
├── guide_ext4_debian.md                         # Ext4 sur Debian
├── guide_lvm_debian.md                          # LVM sur Debian
├── guide_proxmox_lvm_resize.md                  # Redimensionner LVM (Proxmox)
├── guide_snmp.md                                # SNMP
├── guide_snmptrap.md                            # SNMP Trap
├── guide_yaml.md                                # YAML
├── guide_zabbix_docker.md                       # Zabbix dans Docker
│
├── docker/                                      # Guides Docker avancés
│   ├── guide_docker.md
│   ├── guide_docker_builder.md
│   ├── guide_docker_compose.md
│   ├── guide_docker_container.md
│   ├── guide_docker_exec.md
│   ├── guide_docker_filesystem_debian.md
│   ├── guide_docker_image.md
│   ├── guide_docker_logs.md
│   ├── guide_docker_network.md
│   ├── guide_docker_networking_advanced.md      # Nouveau : networking avancé
│   ├── guide_docker_ps.md
│   ├── guide_docker_security.md                 # Nouveau : sécurité
│   ├── guide_docker_stop_start_restart.md
│   ├── guide_docker_transfer_images.md
│   ├── guide_docker_volume.md
│   └── guide_portainer.md
│
├── hardware/                                    # Documentation hardware
├── zabbix/                                      # Configuration Zabbix
└── mibs/                                        # SNMP MIBs
└── template/                                    # Templates YAML
```

---

## 🚀 Guide Rapide

### Installer Docker sur Debian

```bash
# Suivre le guide Installation Docker Debian
# https://docs.docker.com/engine/install/debian/
```

### Lancer un premier container

```bash
docker run -d --name web -p 8080:80 nginx:alpine
curl http://localhost:8080
```

### Déployer une stack Docker Compose

```bash
# Créer un docker-compose.yml
cat > docker-compose.yml << 'EOF'
version: '3.3'
services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"
  db:
    image: postgres:latest
EOF

# Déployer
docker compose up -d
```

### Utiliser Portainer

```bash
# Lancer Portainer
docker volume create portainer_data
docker run -d -p 9000:9000 -p 9443:9443 \
  --restart=always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ce:latest

# Accéder à l'interface : http://localhost:9000
```

---

## 📚 Ressources Externes

- **Docker Documentation** : https://docs.docker.com/
- **Docker Security** : https://docs.docker.com/engine/security/
- **Docker Compose** : https://docs.docker.com/compose/
- **Portainer** : https://docs.portainer.io/
- **Debian** : https://www.debian.org/
- **SNMP** : https://tools.ietf.org/html/rfc3411

---

## 🔄 Mise à Jour de la Documentation

Ces guides sont maintenus et mis à jour régulièrement. Pour contribuer ou signaler une erreur, consultez le fichier `CONTRIBUTING.md` ou ouvrez une issue sur le repository.

---

## 📝 Notes

- Les commandes utilisées dans les guides ciblent **Debian 10, 11, 12, 13** et **Linux**.
- Pour Windows, adaptez les chemins et utilisez WSL2 ou Docker Desktop.
- Testez toujours les commandes dans un environnement de développement avant la production.

---

Dernière mise à jour : **11 novembre 2025**

# Guide Sécurité Docker

## Introduction

Docker simplifie le déploiement d'applications, mais la sécurité des containers reste essentielle. Ce guide couvre les bonnes pratiques de sécurité Docker : isolation des utilisateurs, scanning des images, limites de ressources, auditing, et durcissement des configurations.

## Prérequis

- Docker Engine installé et fonctionnel.
- Connaissance de base des concepts Docker (images, containers, volumes).
- Accès administrateur (root/sudo) pour les configurations système.
- (Optionnel) Outils complémentaires : Trivy, Grype, SELinux, AppArmor.

## 1. Utilisateurs et Privilèges

### Exécuter des containers sans root

Par défaut, les containers s'exécutent en tant que root. Créez toujours un utilisateur non-root dans vos images :

```dockerfile
# Exemple Dockerfile sécurisé
FROM debian:bookworm

# Créer un utilisateur non-root
RUN useradd -m -u 1000 appuser

# Copier l'application
COPY app /app
WORKDIR /app

# Changer vers l'utilisateur non-root
USER appuser

# Exposer le port
EXPOSE 8080

# Lancer l'application
CMD ["python3", "app.py"]
```

### Vérifier l'utilisateur d'exécution

```bash
# Lancer un container et vérifier l'utilisateur actuel
docker run --rm debian:bookworm id
# Résultat : uid=0(root) — DANGEREUX

# Avec utilisateur non-root
docker run --rm --user 1000 debian:bookworm id
# Résultat : uid=1000(appuser)
```

### Restrictions via `--user` et `--group-add`

```bash
# Exécuter un container en tant qu'utilisateur spécifique
docker run -d --user 1000:1000 --name myapp myimage:latest

# Ajouter un utilisateur à un groupe supplémentaire
docker run -d --user 1000 --group-add 0 myapp:latest  # Ajoute au groupe 0 (root)
```

## 2. Isolation et Capacités Linux

### Limiter les capacités du kernel

Par défaut, Docker accorde plusieurs capacités Linux potentiellement dangereuses. Limitez-les :

```bash
# Exécuter un container avec capacités minimales
docker run -d \
  --cap-drop=ALL \
  --cap-add=NET_BIND_SERVICE \
  --name secure-app myimage:latest
```

Capacités communes et leurs risques :
- `CAP_SYS_ADMIN` : risqué, permet accès kernel.
- `CAP_NET_ADMIN` : contrôle réseau, potentiellement dangereux.
- `CAP_SYS_PTRACE` : débogage inter-processus, éviter en prod.
- `CAP_SETUID/SETGID` : changement d'utilisateur, limiter.

### Lecture seule du système de fichiers

```bash
# Exécuter un container avec filesystem en lecture seule
docker run -d \
  --read-only \
  --tmpfs /tmp \
  --tmpfs /run \
  --name readonly-app myimage:latest
```

Cela force l'app à utiliser `/tmp` ou `/run` pour les écritures temporaires.

### Limiter les appels système (seccomp)

Docker fournit un profil seccomp par défaut. Pour un profil personnalisé :

```json
{
  "defaultAction": "SCMP_ACT_ERRNO",
  "defaultErrnoRet": 1,
  "archMap": [
    {
      "architecture": "SCMP_ARCH_X86_64",
      "subArches": ["SCMP_ARCH_X86", "SCMP_ARCH_X32"]
    }
  ],
  "syscalls": [
    {
      "names": ["read", "write", "open", "close", "exit"],
      "action": "SCMP_ACT_ALLOW"
    }
  ]
}
```

Lancer avec profil personnalisé :

```bash
docker run --security-opt seccomp=/path/to/seccomp.json myimage:latest
```

## 3. Scanning des Images

### Utiliser Trivy pour scanner les vulnérabilités

```bash
# Installer Trivy
curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh -s -- -b /usr/local/bin

# Scanner une image locale
trivy image myimage:latest

# Scanner avec rapport JSON
trivy image --format json --output report.json myimage:latest

# Scanner avec niveau de severité élevé
trivy image --severity HIGH,CRITICAL myimage:latest
```

### Intégrer le scanning dans le pipeline Docker Compose

```yaml
version: '3.3'
services:
  scanner:
    image: aquasec/trivy:latest
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - ./trivy-reports:/root/.cache/trivy
    command: image --exit-code 0 myapp:latest
```

### Vérifier les layers d'une image

```bash
# Voir la composition des layers
docker image inspect myimage:latest --format='{{json .Layers}}'

# Voir l'historique de construction
docker history myimage:latest
```

## 4. Limites de Ressources

### Limiter la mémoire et CPU

```bash
# Limiter à 512 Mo de RAM et 0.5 CPU
docker run -d \
  --memory 512m \
  --cpus 0.5 \
  --name limited-app myimage:latest

# Limiter avec swap (memory-swap total incluant swap)
docker run -d \
  --memory 512m \
  --memory-swap 1g \
  --name app-with-swap myimage:latest
```

### Limiter les I/O disque

```bash
# Limiter les débits de lecture/écriture
docker run -d \
  --device-read-bps /dev/sda:10mb \
  --device-write-bps /dev/sda:10mb \
  --name io-limited myimage:latest
```

### Docker Compose avec limites

```yaml
version: '3.3'
services:
  app:
    image: myapp:latest
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 512M
        reservations:
          cpus: '0.25'
          memory: 256M
```

## 5. Networking Sécurisé

### Utiliser un réseau bridge personnalisé

```bash
# Créer un réseau bridge isolé
docker network create --driver bridge mynetwork

# Lancer des containers dans ce réseau
docker run -d --network mynetwork --name app1 myimage:latest
docker run -d --network mynetwork --name app2 myimage:latest

# Les containers se voient par nom DNS
docker exec app1 ping app2
```

### Désactiver le port publishing par défaut

```bash
# Publier explicitement les ports nécessaires
docker run -d \
  -p 8080:8080 \
  --network mynetwork \
  --name secure-app myimage:latest

# Lier au localhost uniquement
docker run -d \
  -p 127.0.0.1:8080:8080 \
  --name internal-app myimage:latest
```

## 6. Auditing et Logging

### Activer les logs Docker

```bash
# Vérifier la configuration du logging
docker info | grep -i logging

# Utiliser le driver json-file avec rotation
docker run -d \
  --log-driver json-file \
  --log-opt max-size=10m \
  --log-opt max-file=3 \
  --name app myimage:latest

# Consulter les logs
docker logs app
journalctl -u docker
```

### Vérifier les événements Docker

```bash
# Surveiller les événements en temps réel
docker events --filter type=container

# Voir les évènements d'un container spécifique
docker events --filter container=app
```

### Examiner la configuration d'un container

```bash
# Inspecter complètement un container
docker inspect app

# Voir les limites de ressources appliquées
docker stats app
```

## 7. Image Security Best Practices

### Utiliser des images minimales

```dockerfile
# ❌ Lourd (1.1 GB)
FROM ubuntu:22.04
RUN apt update && apt install -y python3 && rm -rf /var/cache/apt/*

# ✅ Léger (150 MB)
FROM python:3.11-slim
```

### Multistage build

```dockerfile
# Stage 1 : construction
FROM golang:1.21 AS builder
WORKDIR /app
COPY . .
RUN go build -o app

# Stage 2 : exécution (image finale réduite)
FROM alpine:latest
COPY --from=builder /app/app /app
RUN adduser -D appuser
USER appuser
CMD ["/app"]
```

### Signer et vérifier les images

```bash
# Activer Docker Content Trust (DCT)
export DOCKER_CONTENT_TRUST=1

# Pousser une image signée
docker push myregistry.com/myapp:v1.0
# Cette opération vous demande une passphrase

# Vérifier une image avant de la lancer
docker pull myregistry.com/myapp:v1.0
```

## 8. SELinux et AppArmor

### Activer SELinux pour Docker (Debian)

```bash
# Installer les paquets nécessaires
sudo apt install -y selinux-utils apparmor apparmor-utils

# Vérifier le statut AppArmor
sudo systemctl status apparmor

# Lancer un container avec profil AppArmor personnalisé
docker run --security-opt apparmor=docker-default myimage:latest

# Désactiver AppArmor (non recommandé)
docker run --security-opt apparmor=unconfined myimage:latest
```

## 9. Registry Sécurisé

### Authentifier les requêtes vers un registry privé

```bash
# Se connecter à un registry Docker
docker login myregistry.com

# Pousser une image
docker tag myapp:latest myregistry.com/myapp:latest
docker push myregistry.com/myapp:latest

# Configurer un registry local sécurisé avec TLS
docker run -d \
  -p 5000:5000 \
  -v /certs:/certs \
  -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/cert.pem \
  -e REGISTRY_HTTP_TLS_KEY=/certs/key.pem \
  --name secure-registry \
  registry:2
```

## 10. Checklist de Sécurité

- [ ] Utiliser un utilisateur non-root dans les images.
- [ ] Limiter les capacités du kernel (`--cap-drop=ALL`).
- [ ] Utiliser un filesystem en lecture seule quand possible.
- [ ] Scanner les images pour les vulnérabilités (Trivy).
- [ ] Limiter la mémoire et CPU.
- [ ] Utiliser des réseaux bridge personnalisés.
- [ ] Activer et monitorer les logs Docker.
- [ ] Utiliser des images minimales (Alpine, scratch).
- [ ] Signer les images (Docker Content Trust).
- [ ] Mettre en place un registry privé sécurisé.
- [ ] Auditer les événements Docker.
- [ ] Appliquer SELinux ou AppArmor.

## Ressources

- Documentation Docker Security : https://docs.docker.com/engine/security/
- Trivy : https://github.com/aquasecurity/trivy
- CIS Docker Benchmark : https://www.cisecurity.org/benchmark/docker
- OWASP Container Security : https://owasp.org/www-project-container-security/

## Guides Complémentaires

- [Guide Installation Docker Debian](../guide_docker_installation_debian.md)
- [Guide Docker](./guide_docker.md)
- [Guide Docker Images](./guide_docker_image.md)
- [Guide Docker Networking](./guide_docker_network.md)
- [Guide Docker Containers](./guide_docker_container.md)

---

Guide complet de sécurité Docker sur Debian, couvrant isolation, scanning, ressources et bonnes pratiques.

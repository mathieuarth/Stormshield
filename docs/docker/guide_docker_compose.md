
# 📄 Guide Docker Compose

## 🔹 Introduction
Docker Compose est un outil permettant de définir et de gérer des applications multi-conteneurs à l’aide d’un fichier YAML. Il simplifie le déploiement de services liés (ex : base de données + application web).

## 🔹 Installation
- Docker Compose est inclus dans Docker Desktop (Windows/macOS)
- Sur Linux :
```bash
sudo apt install docker-compose
```
- Vérification :
```bash
docker compose --version
```

## 🔹 Structure d’un fichier docker-compose.yml
Un fichier `docker-compose.yml` contient :
- La version du format
- Une liste de services
- Des volumes, réseaux, dépendances

### Exemple minimal :
```yaml
version: '3.8'
services:
  web:
    image: nginx
    ports:
      - "80:80"
```

## 🔹 Commandes principales
| Commande | Description |
|----------|-------------|
| `docker-compose up` | Démarre les services définis |
| `docker-compose down` | Arrête et supprime les conteneurs, réseaux, volumes |
| `docker-compose ps` | Liste les services en cours |
| `docker-compose logs` | Affiche les logs des services |
| `docker-compose exec <service> <cmd>` | Exécute une commande dans un conteneur |

## 🔹 Erreurs fréquentes
| Erreur | Cause |
|--------|-------|
| `no such service` | Le nom du service est incorrect |
| `port already in use` | Le port est déjà utilisé par un autre service |
| `invalid YAML` | Erreur de syntaxe dans le fichier YAML |

## 🔹 Bonnes pratiques
| Pratique | Avantage |
|----------|----------|
| Utiliser des noms explicites pour les services | Facilite la maintenance |
| Versionner le fichier YAML | Suivi des modifications |
| Utiliser `.env` pour les variables | Séparation de la config et du code |
| Définir des volumes nommés | Persistance des données |

## 🔹 Exemple commenté
```yaml
version: '3.8'
services:
  app:
    image: myapp:latest
    ports:
      - "8080:80"         # Redirection du port 8080 vers le conteneur
    volumes:
      - app-data:/var/app  # Volume nommé pour les données
    environment:
      - DEBUG=true         # Variable d’environnement

volumes:
  app-data:
```

## 🔹 Dépendances entre services
Docker Compose permet de définir des dépendances entre services avec la directive `depends_on`. Cela garantit qu’un service ne démarre qu’après un autre.

### Exemple :
```yaml
version: '3.8'
services:
  db:
    image: postgres

  web:
    image: nginx
    depends_on:
      - db  # Le service 'web' dépend du service 'db'
```

> Remarque : `depends_on` ne garantit pas que le service dépendant est prêt, seulement qu’il est lancé. Pour gérer l’état de disponibilité, utilisez des scripts de vérification ou des outils comme `wait-for-it` ou `healthcheck`.

## 🔹 Utilisation des fichiers .env
Docker Compose permet d’utiliser un fichier `.env` pour définir des variables d’environnement externes au fichier YAML. Cela facilite la gestion des configurations sensibles ou spécifiques à un environnement.

### Exemple de fichier `.env` :
```
PORT=8080
DEBUG=true
VERSION=latest
```

### Intégration dans docker-compose.yml :
```yaml
version: '3.8'
services:
  app:
    image: myapp:${VERSION}
    ports:
      - "${PORT}:80"
    environment:
      - DEBUG=${DEBUG}
```

> Remarque : Le fichier `.env` doit être situé dans le même répertoire que le fichier `docker-compose.yml`.

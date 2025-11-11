
# 📄 Guide d’utilisation des fichiers YAML

## 🔹 Qu’est-ce qu’un fichier YAML ?
YAML (YAML Ain’t Markup Language) est un format de sérialisation de données lisible par l’humain, souvent utilisé pour la configuration d’applications, l’orchestration (ex. Docker, Kubernetes), ou le stockage de données simples.

## 🔹 Création d’un fichier YAML
Un fichier YAML :
- A l’extension **`.yaml`** ou **`.yml`**
- Contient des données structurées sous forme de **paires clé-valeur**, **listes**, ou **objets imbriqués**

### Exemple simple :
```yaml
nom: Mathieu
poste: Administrateur Système Réseau
langues:
  - français
  - anglais
```

## 🔹 Règles de syntaxe essentielles

| Règle | Description |
|------|-------------|
| **Indentation** | Utiliser **espaces** (pas de tabulations). Généralement 2 ou 4 espaces. |
| **Clé-valeur** | Syntaxe : `clé: valeur` |
| **Listes** | Utiliser `-` pour chaque élément de la liste |
| **Commentaires** | Commencer par `#` |
| **Types de données** | YAML supporte les chaînes, nombres, booléens, listes, dictionnaires |
| **Chaînes de caractères** | Les guillemets sont facultatifs sauf si la chaîne contient des caractères spéciaux |
| **Objets imbriqués** | Utiliser l’indentation pour représenter la hiérarchie |

### Exemple avec tout :
```yaml
utilisateur:
  nom: "Mathieu"
  actif: true
  rôles:
    - admin
    - support
  détails:
    email: mathieu@example.com
    age: 34
```

## 🔹 Utilisation dans les outils
- **Docker Compose** : `docker-compose.yml`
- **Kubernetes** : fichiers de déploiement
- **CI/CD** : GitHub Actions (`.github/workflows/*.yml`)
- **Ansible** : playbooks

## 🔹 Validation
Utilisez des outils comme :
- [https://www.yamllint.com](https://www.yamllint.com)
- `yamllint` en ligne de commande

## 🔹 Erreurs fréquentes

| Erreur | Explication |
|-------|-------------|
| **Utilisation de tabulations** | YAML ne supporte que les **espaces** pour l’indentation. Les tabulations provoquent des erreurs. |
| **Mauvaise indentation** | Une indentation incorrecte casse la structure du fichier. |
| **Clés dupliquées** | Deux clés identiques dans le même niveau de hiérarchie ne sont pas autorisées. |
| **Caractères spéciaux non échappés** | Les caractères comme `:` ou `#` dans les chaînes doivent être entourés de guillemets. |
| **Liste mal formatée** | Oublier le `-` ou mal indenter les éléments d’une liste peut rendre le fichier invalide. |
| **Encodage incorrect** | Le fichier doit être en UTF-8 sans BOM. |
| **Valeurs booléennes mal interprétées** | Éviter `yes`, `no`, `on`, `off` sans guillemets : YAML peut les interpréter comme booléens. Utiliser `"yes"` si c’est une chaîne. |

## 🔹 Bonnes pratiques

| Bonne pratique | Pourquoi c’est utile |
|----------------|----------------------|
| **Valider systématiquement le fichier** | Évite les erreurs de syntaxe et les comportements inattendus. |
| **Utiliser des noms de clés explicites** | Améliore la lisibilité et la maintenance du fichier. |
| **Organiser les données par blocs logiques** | Facilite la compréhension et la modification du fichier. |
| **Documenter avec des commentaires** | Aide les autres utilisateurs à comprendre la configuration. |
| **Utiliser des guillemets pour les chaînes ambiguës** | Évite les erreurs d’interprétation par le parser YAML. |
| **Garder une indentation cohérente** | Rend le fichier plus lisible et évite les erreurs. |
| **Versionner les fichiers YAML** | Permet de suivre les modifications et de revenir en arrière si nécessaire. |

## 🔹 Exemple YAML commenté

```yaml
# Définition d'un utilisateur
utilisateur:
  nom: "Mathieu"            # Chaîne de caractères avec guillemets
  actif: true               # Booléen
  rôles:                    # Liste de rôles
    - admin                 # Premier rôle
    - support               # Deuxième rôle
  détails:                  # Bloc imbriqué avec des informations supplémentaires
    email: mathieu@example.com  # Adresse email
    age: 51                     # Entier
    préférences:               # Liste imbriquée
      - dark_mode
      - notifications
```

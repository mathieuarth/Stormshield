
# 📄 Guide d’utilisation de SNMP

## 🔹 Qu’est-ce que SNMP ?
SNMP (Simple Network Management Protocol) est un protocole standard utilisé pour surveiller et gérer les équipements réseau tels que les routeurs, commutateurs, serveurs, imprimantes, etc.

Il permet de récupérer des informations (état, performance) et d’envoyer des commandes à distance via un gestionnaire SNMP.

## 🔹 Configuration de base

### Côté agent (équipement à surveiller) :
- Activer le service SNMP
- Définir la **communauté** (ex : `public` pour lecture seule)
- Spécifier les hôtes autorisés à interroger

### Côté gestionnaire (serveur de supervision) :
- Installer un outil SNMP (ex : `snmpwalk`, `snmpget`, `Zabbix`, `Nagios`)
- Configurer les IP des agents et les communautés

## 🔹 Versions de SNMP

| Version | Sécurité | Fonctionnalités |
|---------|----------|------------------|
| SNMPv1  | Faible (texte clair) | Basique, lecture/écriture |
| SNMPv2c | Faible (texte clair) | Ajout de notifications (trap), performances améliorées |
| SNMPv3  | Forte (authentification, chiffrement) | Sécurisé, contrôle d’accès, confidentialité |

## 🔹 Erreurs fréquentes

| Erreur | Explication |
|--------|-------------|
| **Timeout** | L’agent ne répond pas (IP incorrecte, SNMP désactivé, pare-feu) |
| **Communauté incorrecte** | Le nom de communauté ne correspond pas à celui configuré sur l’agent |
| **Version incompatible** | Le gestionnaire utilise une version SNMP non supportée par l’agent |
| **OID invalide** | L’identifiant d’objet demandé n’existe pas ou n’est pas accessible |

## 🔹 Bonnes pratiques

| Bonne pratique | Pourquoi c’est utile |
|----------------|----------------------|
| **Utiliser SNMPv3** | Offre sécurité et contrôle d’accès, attention à la complexité de la mise en place |
| **Limiter les IP autorisées** | Réduit les risques d’accès non autorisé |
| **Documenter les OID utilisés** | Facilite la maintenance et la supervision |
| **Superviser les traps SNMP** | Permet d’être alerté en cas d’événement critique |
| **Tester avec snmpwalk/snmpget** | Vérifie la connectivité et les droits SNMP |

## 🔹 Exemple de configuration SNMP commenté (Linux - fichier snmpd.conf)

```conf
# Définir la communauté en lecture seule
rocommunity public

# Autoriser uniquement le gestionnaire SNMP à interroger
agentAddress udp:161

# Spécifier les vues et accès (SNMPv3)
# Exemple :
# createUser authUser MD5 "motdepasse" DES
# rouser authUser
```


## 🔹 Utilisation des OID

Un **OID** (Object Identifier) est une chaîne numérique hiérarchique qui identifie une variable spécifique dans un équipement SNMP.
Chaque OID correspond à une information (ex : charge CPU, mémoire, état d'interface).

### Structure d'un OID

Les OIDs sont organisés de manière hiérarchique, comme un arbre, où chaque nombre représente un niveau dans la hiérarchie :

```text
iso(1)
  org(3)
    dod(6)
      internet(1)
        private(4)
          enterprise(1)
            [ID Entreprise]
              [Sous-branches spécifiques]
```

Exemple détaillé de la structure : `1.3.6.1.2.1.1.5.0`
| Niveau | Valeur | Signification |
|--------|---------|---------------|
| 1 | 1 | ISO |
| 2 | 3 | Organisation (org) |
| 3 | 6 | Département de la Défense (dod) |
| 4 | 1 | Internet |
| 5 | 2 | Management (mgmt) |
| 6 | 1 | MIB-2 |
| 7 | 1 | System |
| 8 | 5 | sysName |
| 9 | 0 | Instance |

### Branches principales courantes
| OID de base | Description |
|-------------|-------------|
| .1.3.6.1.2.1 | MIB-2 (standard) |
| .1.3.6.1.4.1 | Enterprise (privé) |
| .1.3.6.1.6.3 | SNMP |
| .1.3.6.1.2.1.25 | Host Resources |

### Exemple avec Stormshield
L'OID de base Stormshield : `.1.3.6.1.4.1.11256`

Décomposition :
```text
.1              : ISO
.3              : org
.6              : dod
.1              : internet
.4              : private
.1              : enterprise
.11256          : Stormshield Network Security
```

Exemples d'OIDs Stormshield courants :
| OID | Description |
|-----|-------------|
| .1.3.6.1.4.1.11256.1.0.1 | Version du Firmware |
| .1.3.6.1.4.1.11256.1.0.2 | Numéro de série |
| .1.3.6.1.4.1.11256.1.0.3 | État du système |
| .1.3.6.1.4.1.11256.1.3.1 | Utilisation CPU |
| .1.3.6.1.4.1.11256.1.3.2 | Utilisation mémoire |

### Exemple d'OID :
- `1.3.6.1.2.1.1.5.0` : Nom de l’hôte (sysName)

### Utilisation avec snmpget :
```bash
snmpget -v2c -c public 192.168.1.1 1.3.6.1.2.1.1.5.0
```

### Explication :
- `-v2c` : version SNMP utilisée
- `-c public` : communauté SNMP
- `192.168.1.1` : IP de l’équipement cible
- `1.3.6.1.2.1.1.5.0` : OID interrogé

Cette commande retourne le nom de l’hôte configuré sur l’équipement SNMP.

---

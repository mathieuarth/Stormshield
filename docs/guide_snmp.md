
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
| **Utiliser SNMPv3** | Offre sécurité et contrôle d’accès |
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
Chaque OID correspond à une information (ex : charge CPU, mémoire, état d’interface).

### Exemple d’OID :
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

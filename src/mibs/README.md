# MIBs Stormshield

## Introduction

Ce répertoire contient les fichiers **MIB (Management Information Base)** pour les produits Stormshield. Les MIBs permettent aux outils de monitoring (Zabbix, Nagios, SNMP browsers, etc.) d'interpréter les données SNMP collectées depuis les appliances Stormshield.

## Qu'est-ce qu'une MIB ?

Une **MIB** est une base de données hiérarchique qui définit :
- Les **OIDs (Object Identifiers)** — identifiants uniques pour chaque élément monitorable.
- Les **types de données** — entier, string, compteur, jauge, etc.
- Les **descriptions** — explications des paramètres.
- Les **notifications** — alertes (traps) que l'appliance peut envoyer.

Exemple d'OID : `1.3.6.1.4.1.11035.1.1.1.1` (appliances Stormshield).

## Fichiers MIB disponibles

Les fichiers MIB Stormshield sont généralement nommés selon le produit :
- `STORMSHIELD-MIB.txt` ou `SNS-MIB.txt` — MIB principale pour les appliances Stormshield Network Security (SNS).
- Fichiers supplémentaires pour des modules ou versions spécifiques.

## Utilisation des MIBs

### Charger une MIB dans un outil SNMP

**Zabbix** :
```bash
# Copier le fichier MIB dans le répertoire Zabbix
sudo cp STORMSHIELD-MIB.txt /usr/share/snmp/mibs/

# Rechanger les outils SNMP et Zabbix
sudo systemctl restart snmpd
sudo systemctl restart zabbix-server
```

**Nagios / Icinga** :
```bash
# Copier dans le répertoire des MIBs
sudo cp STORMSHIELD-MIB.txt /usr/share/snmp/mibs/

# Utiliser dans les commandes de vérification
snmpget -m STORMSHIELD-MIB -v 2c -c public <IP_APPLIANCE> <OID>
```

**SNMP Browser (outils graphiques)** :
1. Ouvrir l'outil (ex: SNMP Browser de ManageEngine, Paessler PRTG).
2. Aller dans **MIB Management** ou **MIB Loader**.
3. Charger le fichier `STORMSHIELD-MIB.txt`.
4. Les OIDs seront maintenant affichés en langage lisible au lieu de numéros.

### Requête SNMP utilisant une MIB

```bash
# Sans MIB (affiche l'OID numérique)
snmpget -v 2c -c public 192.168.1.100 1.3.6.1.4.1.11035.1.1.1.1.0

# Avec MIB chargée (affiche le nom et la description)
snmpget -m STORMSHIELD-MIB -v 2c -c public 192.168.1.100 STORMSHIELD-MIB::snsModel.0
```

## Obtenir les MIBs Stormshield

Les MIBs Stormshield peuvent être obtenues :

1. **Depuis le site de documentation officielle** :
   - [Documentation SNMP Agent Stormshield](https://documentation.stormshield.eu/SNS/v4/fr/Content/User_Configuration_Manual_SNS_v4/SNMP_Agent/SNMP_AGENT.htm)
   - Cette page contient les MIBs à télécharger et les OIDs disponibles.

2. **Depuis l'appliance Stormshield elle-même** :
   - Accéder à l'interface web de l'appliance.
   - Aller dans **Administration > SNMP > MIBs**.
   - Télécharger les fichiers MIB disponibles.

3. **Via SNMP** :
   - Certaines appliances exposent les MIBs via SNMP (utiliser `snmpget` avec OID spécifique).

## OIDs Courants Stormshield

Exemples d'OIDs typiques pour les appliances Stormshield SNS :

| OID | Description |
|-----|-------------|
| `1.3.6.1.4.1.11035.1.1.1.1.0` | Modèle d'appliance |
| `1.3.6.1.4.1.11035.1.1.2.1.0` | Version logicielle |
| `1.3.6.1.4.1.11035.1.2.1.1.0` | Température système |
| `1.3.6.1.4.1.11035.1.2.2.1.0` | Charge CPU |
| `1.3.6.1.4.1.11035.1.2.3.1.0` | Utilisation mémoire |
| `1.3.6.1.4.1.11035.1.3.1.1.0` | Nombre de connexions actives |

Pour les OIDs complets et descriptions précises, consultez la **MIB officielle** ou la documentation Stormshield.

## MIBs Stormshield Network Disponibles

Voici la liste complète des MIBs Stormshield Network avec leur contenu et commandes équivalentes :

| MIB Stormshield Network | Contenu | CLI / Serverd | Console |
|---|---|---|---|
| STORMSHIELD-ALARM-MIB | Alarmes déclenchées | — | sfctl -s log |
| STORMSHIELD-ASQ-STATS-MIB | Statistiques de l'IPS | — | sfctl –s stat |
| STORMSHIELD-AUTHUSERS-MIB | Utilisateurs authentifiés | MONITOR USER | sfctl -s user |
| STORMSHIELD-AUTOUPDATE-MIB | État des modules mis à jour par Active Update | MONITOR AUTOUPDATE | — |
| STORMSHIELD-HA-MIB | Informations relatives à la haute disponibilité | HA INFO | hainfo |
| STORMSHIELD-HEALTH-MONITOR-MIB | État de santé du firewall | MONITOR HEALTH | — |
| STORMSHIELD-HOSTS-MIB | Table des machines protégées | MONITOR HOST | sfctl -s host |
| STORMSHIELD-IF-MIB | État des interfaces vues par l'IPS | MONITOR INTERFACE | sfctl -s global |
| STORMSHIELD-IPSEC-STATS-MIB | Statistiques IPsec | — | ipsecinfo |
| STORMSHIELD-OVPNTABLE-MIB | Clients connectés au VPN SSL | MONITOR OPENVPN LIST | — |
| STORMSHIELD-POLICY-MIB | Politique de filtrage | MONITOR POLICY | slotinfo |
| STORMSHIELD-PROPERTY-MIB | Informations système (SYSTEM PROPERTY) | SYSTEM PROPERTY, SYSTEM IDENT, SYSTEM LANGUAGE | — |
| STORMSHIELD-QOS-MIB | Informations sur la QoS | MONITOR QOS | sfctl -s qos |
| STORMSHIELD-ROUTE-MIB | Table des routeurs | MONITOR ROUTE | sfctl -s route |
| STORMSHIELD-SERVICES-MIB | État des services du firewall | MONITOR SERVICE | dstat |
| STORMSHIELD-SYSTEM-MONITOR-MIB | Compteurs d'utilisation des ressources de l'IPS | MONITOR STAT | — |
| STORMSHIELD-VPNIKESA-MIB | Table des SA IKE négociées | MONITOR GETIKESA | — |
| STORMSHIELD-VPNSA-MIB | Table des SA | MONITOR GETSA | showSAD |
| STORMSHIELD-VPNSP-MIB | Table des SP | MONITOR GETSPD | showSPD |

**Notes** :
- `STORMSHIELD-SMI-MIB` est une MIB chapeau de l'ensemble des MIBs.
- `STORMSHIELD-VPN-MIB` est une MIB chapeau des MIBs VPNIKESA, VPNSA et VPNSP.

Pour plus d'informations sur ces MIBs et les traps SNMP, consultez la [documentation officielle Stormshield MIBs and Traps](https://documentation.stormshield.eu/SNS/v4/fr/Content/User_Configuration_Manual_SNS_v4/SNMP_Agent/MIBS_and_traps_SNMP.htm).


## Configuration SNMP sur l'Appliance Stormshield

Avant de collecter des données SNMP, vous devez configurer l'agent SNMP sur l'appliance :

1. **Accéder à l'interface de gestion** de l'appliance Stormshield.
2. Aller dans **Administration > SNMP > Configuration**.
3. **Activer l'agent SNMP**.
4. **Configurer la chaîne de communauté** (par défaut : `public`).
5. **Spécifier l'adresse IP du serveur de monitoring** (pour les traps).
6. **Sauvegarder** les modifications.

Pour plus de détails, consultez la [documentation officielle Stormshield SNMP Agent](https://documentation.stormshield.eu/SNS/v4/fr/Content/User_Configuration_Manual_SNS_v4/SNMP_Agent/SNMP_AGENT.htm).

## Dépannage

### Les OIDs ne sont pas résolus correctement

**Symptôme** : affichage d'OIDs numérotés au lieu de noms.

**Solution** :
```bash
# Vérifier que la MIB est chargée
snmptranslate -m STORMSHIELD-MIB STORMSHIELD-MIB::snsModel

# Si erreur, charger la MIB manuellement
export MIBDIRS=/usr/share/snmp/mibs:/path/to/custom/mibs
snmpget -m STORMSHIELD-MIB -v 2c -c public <IP> <OID>
```

### Timeout ou pas de réponse SNMP

**Symptôme** : la requête SNMP expire.

**Vérifications** :
```bash
# 1. Vérifier que l'agent SNMP est actif sur l'appliance
snmpwalk -v 2c -c public <IP_APPLIANCE> system.sysDescr

# 2. Vérifier le pare-feu (port SNMP 161/UDP)
sudo ufw allow 161/udp

# 3. Vérifier la chaîne de communauté correcte
snmpget -v 2c -c <COMMUNITY> <IP_APPLIANCE> system.sysUpTime
```

### MIB non trouvée lors du chargement

**Solution** :
```bash
# Vérifier le chemin des MIBs
snmptranslate -Pu  # Affiche les répertoires de recherche

# Copier dans le bon répertoire
sudo cp STORMSHIELD-MIB.txt /usr/share/snmp/mibs/

# Rafraîchir le cache
sudo snmptranslate -m +STORMSHIELD-MIB
```

## Intégration avec Zabbix

Pour intégrer les MIBs Stormshield dans Zabbix :

1. **Charger la MIB** dans `/usr/share/snmp/mibs/`.
2. **Créer un hôte Zabbix** pointant vers l'appliance Stormshield.
3. **Ajouter des éléments (items)** de type SNMP utilisant les OIDs Stormshield.
4. **Configurer les traps SNMP** pour recevoir les alertes de l'appliance.

Exemple d'item Zabbix :
- **Type** : SNMP v2 agent
- **SNMP OID** : `1.3.6.1.4.1.11035.1.1.2.1.0` (version logicielle)
- **Description** : Version logicielle SNS

## Ressources

- **Documentation officielle Stormshield SNMP** : [SNMP Agent Guide](https://documentation.stormshield.eu/SNS/v4/fr/Content/User_Configuration_Manual_SNS_v4/SNMP_Agent/SNMP_AGENT.htm)
- **RFC 3411 (SNMP)** : https://tools.ietf.org/html/rfc3411
- **Zabbix SNMP Integration** : https://www.zabbix.com/documentation/current/en/manual/config/items/itemtypes/snmp
- **Net-SNMP Documentation** : http://net-snmp.sourceforge.net/

## Guides Connexes

- [Guide SNMP](../../docs/guide_snmp.md)
- [Guide SNMP Trap](../../docs/guide_snmptrap.md)
- [Guide Zabbix Docker](../../docs/guide_zabbix_docker.md)

---

Répertoire MIB Stormshield — Documentation de référence pour le monitoring SNMP.

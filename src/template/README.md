# Template Zabbix Stormshield SNS

## Description
Ce template permet la supervision des pare-feux Stormshield Network Security (SNS) via SNMP. Il collecte les métriques essentielles concernant :

- État du système et des composants 
- Utilisation CPU et mémoire
- État des interfaces réseau
- Configuration HA (Haute Disponibilité)
- État des tunnels VPN
- Statut des mises à jour
- Températures et ventilateurs
- État des disques et du RAID

## Prérequis

- Zabbix 7.0 ou supérieur
- SNMP v2c ou v3 activé sur le pare-feu Stormshield
- MIBs Stormshield installées sur le serveur Zabbix

## Installation

### Import du template

1. Dans l'interface Web Zabbix, aller dans **Configuration > Templates**
2. Cliquer sur **Import**
3. Sélectionner le fichier `Template_Stormshield.yaml`
4. Valider l'import

### Configuration d'un hôte

1. Aller dans **Configuration > Hosts**
2. Créer un nouvel hôte ou modifier un existant 
3. Dans l'onglet "Templates", lier le template "Stormshield SNS by SNMP"
4. Configurer les interfaces SNMP :
   - Type : SNMP
   - Version SNMP : v2c ou v3 
   - Community string : [votre_community]
5. Sauvegarder

## Macros

| Macro | Description | Valeur par défaut |
|------|-------------|-------------------|
| {$SNS.CPU.UTIL.CRIT} | Seuil critique CPU | 95% |
| {$SNS.CPU.UTIL.WARN} | Seuil alerte CPU | 85% |
| {$SNS.MEMORY.UTIL.MAX} | Seuil mémoire | 90% |
| {$SNS.DISK.FREE.CRIT} | Seuil critique espace disque | 10% |
| {$SNS.DISK.FREE.WARN} | Seuil alerte espace disque | 20% |
| {$SNS.SNMP.TIMEOUT} | Timeout SNMP | 5m |

## Découverte

Le template inclut des règles de découverte automatique pour :

- Interfaces réseau
- Disques et stockage
- CPU et températures  
- Ventilateurs
- Alimentation 
- Membres HA

## Items collectés

### Système
- Version du firmware
- Modèle 
- Numéro de série
- Nom système
- Uptime

### Performances
- Utilisation CPU par coeur
- Utilisation mémoire
- Température CPU
- Espace disque utilisé

### Réseau
- État des interfaces
- Trafic entrant/sortant
- Paquets acceptés/bloqués
- Connexions TCP/UDP

### Haute disponibilité
- État des noeuds
- Synchronisation
- Liens HA

### VPN 
- État des tunnels
- Politiques IPsec

### Matériel
- État des ventilateurs
- État des alimentations
- État des disques
- État du RAID

## Triggers

Les principaux déclencheurs configurés :

- Utilisation CPU > 95%
- Utilisation mémoire > 90%
- Espace disque critique < 10%
- Perte de connexion SNMP/ICMP
- Défaillance composants (ventilateurs, disques, etc)
- Problèmes HA

## Graphiques

Des graphiques sont générés pour :

- Utilisation CPU et mémoire
- Trafic réseau par interface 
- Températures
- Espace disque

## Compatibilité 

Template testé sur les versions :
- SNS 4.x
- SNS 5.x

## Support

Pour tout problème :
1. Vérifier les logs Zabbix
2. Vérifier la configuration SNMP
3. Tester la connectivité SNMP avec snmpwalk

## MIBs utilisées

- HOST-RESOURCES-MIB
- UCD-SNMP-MIB  
- STORMSHIELD-ASQ-STATS-MIB
- STORMSHIELD-AUTOUPDATE-MIB
- STORMSHIELD-HA-MIB
- STORMSHIELD-PROPERTY-MIB
- STORMSHIELD-HEALTH-MONITOR-MIB
- STORMSHIELD-IF-MIB
- STORMSHIELD-SYSTEM-MONITOR-MIB
- STORMSHIELD-IPSEC-STATS-MIB
# 📄 Guide d'utilisation des SNMP Traps

## 🔹 Qu'est-ce qu'un SNMP Trap ?
Un SNMP Trap est un message d'alerte envoyé par un agent SNMP vers un gestionnaire SNMP pour signaler un événement important (panne, seuil dépassé, changement d'état, etc.) sans attendre d'être interrogé.

## 🔹 Configuration de base

### Côté émetteur (agent SNMP) :
- Configurer l'adresse du serveur de destination
- Définir la communauté pour les traps
- Spécifier les événements à notifier

### Côté récepteur (serveur de traps) :
- Configurer le service snmptrapd
- Définir les communautés autorisées
- Mettre en place le traitement des traps

## 🔹 Types de Traps SNMP

| Type | Description |
|------|-------------|
| **Cold Start** | L'agent SNMP redémarre |
| **Warm Start** | L'agent SNMP se réinitialise sans redémarrer |
| **Link Down** | Une interface réseau tombe |
| **Link Up** | Une interface réseau se lève |
| **Authentication Failure** | Échec d'authentification SNMP |
| **EGP Neighbor Loss** | Perte de connexion avec un voisin EGP |
| **Enterprise Specific** | Traps personnalisés définis par le constructeur |

## 🔹 Format d'un Trap SNMP

```text
Version: [v1/v2c/v3]
Type: Trap
Enterprise: [OID]
Agent Address: [IP]
Generic Trap: [0-6]
Specific Trap: [Code]
Timestamp: [Time]
Variable Bindings: [OID=Value pairs]
```

## 🔹 Erreurs fréquentes

| Erreur | Solution |
|--------|----------|
| **Traps non reçus** | Vérifier pare-feu (UDP 162) et configuration snmptrapd |
| **Traps ignorés** | Vérifier la communauté et les autorisations |
| **Format incorrect** | Valider la version SNMP et le format du trap |
| **Duplication de traps** | Vérifier la configuration de redondance |

## 🔹 Bonnes pratiques

| Bonne pratique | Pourquoi c'est utile |
|----------------|----------------------|
| **Filtrer les traps** | Évite la surcharge d'alertes non pertinentes |
| **Utiliser SNMPv3** | Sécurise la transmission des traps |
| **Configurer des seuils** | Évite les faux positifs |
| **Documenter les traps** | Facilite le diagnostic et la maintenance |
| **Mettre en place une redondance** | Assure la réception des alertes critiques |

## 🔹 Exemple de configuration (Linux)

### Configuration de l'émetteur (snmpd.conf)
```conf
# Configuration des destinations des traps
trapsink localhost public
trap2sink monitor.example.com private

# Activation des traps spécifiques
trapcommunity private
authtrapenable 1

# Configuration des événements
monitor -r 60 "Interface eth0" -o ifOperStatus.1
```

### Configuration du récepteur (snmptrapd.conf)
```conf
# Autorisation des communautés
authCommunity log,execute,net private
disableAuthorization no

# Format de log
format1 %V\n%.4y%.2m%.2l%.2h:%.2j:%.2k %B [%b] (tag=%T/%t) %v\n
logOption f /var/log/snmptrap.log
```

## 🔹 Commandes utiles

### Envoyer un trap manuellement
```bash
# SNMPv2
snmptrap -v 2c -c public localhost '' .1.3.6.1.6.3.1.1.5.3 0 0 

# SNMPv3
snmptrap -v 3 -u userv3 -a SHA -A passphrase localhost 0 linkDown
```

### Démarrer le récepteur de traps
```bash
snmptrapd -f -Lo -c /etc/snmp/snmptrapd.conf
```

## 🔹 Ressources utiles
- [Documentation Net-SNMP](http://www.net-snmp.org/)
- [MIBs standards](http://www.oidview.com/mibs/detail.html)
- [RFC 3014 - Notification Management](https://tools.ietf.org/html/rfc3014)
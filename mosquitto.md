# Conseils pour la mise en oeuvre de Mosquitto

Cette page décrit comment installer le BRoker MQTT **Mosquitto** sur Raspberry PI.

## Pourquoi ?
- Le protocole **MQTT** est un moyen simple et "léger" de communiquer entre applications  
- Les librairies existent  
- Il nécessite un **Broker MQTT** qui permet de collecter les messages:  
  - Une application **publie** des messages vers ce serveur  
  - D'autres applications s'abonnent **subscribe** à ce serveur, et plus spécifiquement aux messages qui l'intéressent  

## Installation
##### Installation des librairies requises sous Raspberry
```bash
sudo apt-get update
sudo apt-get install -y mosquitto
sudo apt-get install -y mosquitto-clients
sudo apt-get install -y mosquitto mosquitto-clients python-mosquitto
```

##### Vérification de fonctionnement:
```bash
systemctl status mosquitto
```
Si le service ne fonctionne pas, lancer cette commande:
```bash
sudo systemctl enable mosquitto.service
```

## Sécuriser l’accès à Mosquitto par un mot de passe (optionnel)
#### Préparation
- Pour ajouter les utilisateurs un par un:
```bash
mosquitto_passwd -c passwordfile user
```
Un mot de passe vous sera demandé (il n'y a pas d'echo, mais ça marche).  
Un fichier **passwordfile** sera créé là où vous êtes, et son contenu est crypté.  
Pour ajouter d'autres utilisateurs dans ce fichier:
```bash
mosquitto_passwd -b passwordfile user password
```
Le fichier **passwordfile** sera mis à jour.  
Pour supprimer un utilisateur dans ce fichier:
```bash
mosquitto_passwd -D passwordfile user
``` 

- Pour plusieurs utilisateurs en une étape:
Créer un fichier texte là où vous êtes et y mettre une liste de **user:pwd**:
```bash
nano passwordfile

Contenu du fichier:
user1:pass1
user2:pass2
```
CTRL+X pour sauvegarder.  

Crypter le fichier avec la commande:
```bash
mosquitto_passwd -U passwordfile
```

##### Pour appliquer les nouveaux utilisateurs dans Mosquitto   
Ouvrez le fichier de configuration de Mosquitto avec la commande:
```bash
sudo nano /etc/mosquitto/mosquitto.conf
```
Puis, y ajouter les lignes suivantes (à ne faire qu'une fois):
```bash
allow_anonymous false
password_file /etc/mosquitto/mypasswordfile
```
**mypasswordfile** est le fichier crypté qui contient la liste des users/pwd et sur lequel Mosquitto pointera.  

Appliquer la mise à jour en copiant le fichier créé vers le chemin indiqué dans mosquitto.conf:
```bash
sudo cp mypasswordfile /etc/mosquitto/
```
Relancer le service Mosquitto avec cette commande:
```bash
sudo systemctl restart mosquitto.service
```
Vérifiez que l'accès fonctionne sur un topic MQTT en cours de fonctionnement:
```bash
mosquitto_sub -h localhost -u user1 -P pass1 -t "ESP_Easy_0_1/Temperature/Analog" -v
```

## Usage
##### Pour s'abonner aux topics dans la console
Dans la console:  
```bash
mosquitto_sub -h localhost -u user1 -P pass1 -t "ESP_Easy_0_1/Temperature/Analog" -v
```

Liens sources intéressants
--------------------------
- https://projetsdiy.fr/mosquitto-broker-mqtt-raspberry-pi/#Quel_Broker_MQTT_open-source_choisir
- http://www.steves-internet-guide.com/mqtt-username-password-example/
- https://projetsdiy.fr/espeasy-mqtt-mosquitto-nodered-communication-bidirectionnelle-dashboard/

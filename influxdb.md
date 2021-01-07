# Conseils pour la mise en oeuvre d'InfluxDB

Cette page décrit comment installer InfluxDB sur la Raspberry PI.  

## Pourquoi ?
- InfluxDB est une base de données pour les Time Series: elle est très efficace par exemple pour sauvegarder des données des IoT  
- Pour la domotique, elle est tout à fait adaptée pour enregistrer les capteurs de température, pression, Linky et météo  
- On peut ensuite facilement afficher les courbes avec Grafana, qui y accède directement  
- Facile à installer et API accessible dans plein de langages, notamment Python  

## Equipement utilisé pour les tests
- Raspberry PI 3 B+  
- Raspbian Buster  

## Installation
##### Installation des librairies requises sous Raspberry
Noter la version de Raspbian que vous avez en lancant la commande suivante:
```bash
lsb_release -a
```
Ici c'est **buster**:
```bash
No LSB modules are available.
Distributor ID: Raspbian
Description:    Raspbian GNU/Linux 10 (buster)
Release:        10
Codename:       buster
```
Créer le fichier suivant:  
```bash
sudo nano /etc/apt/sources.list.d/influx.list
```
Y mettre le contenu suivant (remplacer **buster** par votre version:
```bash
deb https://repos.influxdata.com/debian buster stable
```
Lancer la mise à jour via **apt-get**:   
```bash
curl -sL https://repos.influxdata.com/influxdb.key | sudo apt-key add -
sudo apt-get update
```

##### Installation de InfluxDB
Lancer l'installation avec cette commande:
```bash
sudo apt-get install influxdb
```
Activer InfluxDB au boot:
```bash
sudo systemctl enable influxdb
```
Vérifier l'activation de InfluxDB:
```bash
sudo systemctl is-enabled influxdb
```
Lancer InfluxDB:
```bash
sudo systemctl start influxdb
```
##### Relancer la Raspberry
Pour vérifier que InfluxDB démarre bien au boot:
```bash
sudo shutdown -r now
```
Puis: 
```bash
sudo systemctl status influxdb
```

## Utilisation
##### Description rapide
InfluxDB est structuré de la manière suivante:
* On peut créer plusieurs **databases**
* Chaque Database peut contenir plusieurs tables **measurements**
* Les tables contiennent des enregistrements en ligne, avec plusieurs colonnes
  * Colonne 'time' est le timestamp de la ligne: s'il n'est pas renseigné lors de l'ingestion de la donnée, il est ajouté par InfluxDB automatiquement
  * Colonnes de type TAG: elles servent à accéder rapidement aux lignes qui nous intéressent dans nos requêtes. Ses valeurs ne doivent pas être trop diversifiées, sous peine rendre le filtrage inefficace (augmente la cardinalité)  
  * Colonnes de type FIELD: elles contiennent les valeurs intéressantes à enregistrer  

##### Le Shell InfluxDB
Lancer le shell avec la commande suivante:
```bash
influx
```
Quitter le shell avec la commande suivante:
```bash
> exit
```

##### Quelques commandes du Shell
Lister les bases de données:
```bash
show databases
```
Pointer une bases de données:
```bash
use <database>
```
Lister les tables de measurements d'une database:
```bash
show measurements
```
**Requête:** les lignes dont le contenu de la colonne 'topic' est '/minihome/meteo/current/temperature' et dont la colonne 'value' est > 26.0
```bash
SELECT * FROM mqtt_consumer WHERE topic = '/minihome/meteo/current/temperature' AND value > 26.0
```
**Requête:** les lignes dont le contenu de la colonne 'SensorName' commence par 'abcd' et dont le timestamp 'time' est d'il y a maximum 5 minutes
```bash
SELECT * FROM "sensor" WHERE SensorName =~ /abcd*/ AND time > now() - 5m
```
**Requête:** les lignes dont le contenu de la colonne 'topic' contient 'meteo'
```bash
SELECT * FROM mqtt_consumer WHERE topic =~ /meteo/
```
**Requête:** les lignes dont le contenu de la colonne 'topic' contient 'meteo' et dont le timestamp 'time' est inférieur ou égal à une date
```bash
SELECT * FROM mqtt_consumer WHERE topic =~ /meteo/ AND time <= '2020-04-17T18:31:15.542117258Z'
```
**Requête:** supprimer les lignes dont le contenu...
```bash
DELETE FROM mqtt_consumer WHERE topic =~ /meteo/ AND time <= '2020-04-17T18:31:15.542117258Z'
```
Afficher les types de données pour les colonnes de type FIELD et de type TAG, pour la base de données 'test_mqtt', tous measurements confondus  
```bash
SHOW FIELD KEYS on test_mqtt
SHOW TAG KEYS on test_mqtt
```


## Utilisation simple avec PYTHON
##### Installation de la librairie
```bash
sudo apt-get install -y python3-influxdb
pip install influxdb
```
##### Connexion à la base
Exemple de fonction permettant de se connecter en client  
Avec **create_database**, si la database n'existe pas elle est créée, sinon pas de modification.  
Par défaut: **login**="root", **password**="root", **port**="8086"  
```python
import logging
from influxdb import InfluxDBClient

def connect_influxdb(server, port, login, password, db_name):
    """
    Connexion to InfluxDB and database creation
    :param server: IP address of InfludDB as string "xxx.xxx.xxx.xxx"
    :param port: port of InfludDB as string
    :param login: login of InfludDB
    :param password: password of InfludDB
    :param db_name: target database in InfludDB
    :return: InfluDB client
    """
    try:
        client = InfluxDBClient(server, port, login, password, db_name)
        client.create_database(db_name)
    except Exception:
        client = None
        logging.error("Connexion to InfluxDB failed !")
    return client
```

##### Ecriture
Considérant **value** comme un dictionnaire contenant la trame MQTT d'un capteur Aqara.  
On spécifie les colonnes de la table measurement **capteurs_aqara** avec les clés **tags** et **fields**: 
```python
def write_mqtt_to_influxdb(client, db_name, value):
    """
    Write MQTT value to InfluxDB database
    :param client: InfluDB client
    :param db_name: target database in InfludDB
    :param value: MQTT JSON content
    :return: 
    """
    json_body = [
        {
            "measurement": "capteurs_aqara",
            "tags": {"id": 1, "location":"salon"},
            "fields": {
                "battery": int(value["battery"]),
                "voltage": int(value["voltage"]),
                "temperature": float(value["temperature"]),
                "humidity": float(value["humidity"]),
                "pressure": float(value["pressure"]),
                "linkquality": int(value["linkquality"])
            }
        }
    ]
    client.write_points(json_body, database=db_name)
```
La méthode **write_points** permet d'écrire dans la base plusieurs lignes grace à la liste de dictionnaires **json_body**.  

**Notes:**  
* Pour une table measurement donnée, les enregistrements (lignes) peuvet contenir des colonnes en plus ou en moins: elles s'ajouteront à la table.
* Les lignes qui n'ont pas définies de valeurs pour une colonne, n'engendreront pas d'espace mémoire supplémentaire dans la base.  
* Dans l'exemple ci-dessus, le **timestamp** est automatiquement ajouté par InfluDB. Mais, on aurait pu l'ajouter
```python
from datetime import datetime

t = datetime.utcnow().strftime('%Y-%m-%dT%H:%M:%SZ')
json_body = [
        {
            "measurement": "capteurs_aqara",
            "tags": {"id": 1, "location":"salon"},
            "time": t,
            "fields": {
                "battery": int(value["battery"]),
                "voltage": int(value["voltage"]),
                "temperature": float(value["temperature"]),
                "humidity": float(value["humidity"]),
                "pressure": float(value["pressure"]),
                "linkquality": int(value["linkquality"])
            }
        }
    ]
```
##### Lecture  
```python
query = 'SELECT * FROM capteurs_aqara;'
d = client.query(query)
```

##### Exemples plus complets
* https://influxdb-python.readthedocs.io/en/latest/examples.html

Liens sources intéressants
--------------------------
- https://docs.influxdata.com/influxdb/v1.8/concepts/schema_and_data_layout/
- https://influxdb-python.readthedocs.io/en/latest/api-documentation.html
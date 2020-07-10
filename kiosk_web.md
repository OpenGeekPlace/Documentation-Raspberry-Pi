# Conseils pour la mise en oeuvre d'un kiosk web

Cette page décrit comment afficher une page Web en plein écran au démarrage de la Raspberry PI.  

## Pourquoi ?
- Afficher un dashboard sur un écran : par exemple des indicateurs domotique avec Grafana ou Node-RED  
- La Raspberry peut être équipée d'un petit écran et l'ensemble accroché au mur dans le salon...  

## Equipement utilisé pour les tests
- Raspberry PI 3 B+
- Raspbian Buster
- Ecran [HyperPixel 4.0 Square Touch](https://shop.pimoroni.com/products/hyperpixel-4-square?variant=30138251444307)
- Chromium browser  

## Installation
##### Installation des librairies requises sous Raspberry
```bash
sudo apt-get install chromium-browser
```
##### Activer le mode d'affichage
Il faut choisir le mode Desktop Autologin:
```bash
sudo raspi-config
```
Choisir: **3 Boot Options** > **B1 Desktop / CLI** > **B4 Desktop Autologin**  
Cela demandera un redémarrage  

##### Création du script de lancement du kiosk
Dans le HOME du user 'pi', créer le fichier **autostart** dans le bon chemin (créer le chemin s'il n'existe pas).
```bash
mkdir ~/.config/lxsession
mkdir ~/.config/lxsession/LXDE-pi
nano ~/.config/lxsession/LXDE-pi/autostart
```
Mettre le contenu suivant pour afficher la page Google:
```bash
@lxpanel --profile LXDE-pi
@pcmanfm --desktop --profile LXDE-pi
@xscreensaver -no-splash

@point-rpi
#@xset s off
#@xset -dpms
#@xset s noblank
@sed -i ‘s/ »exited_cleanly »: false/ »exited_cleanly »: true/’ ~/.config/chromium/Default/Preferences
@/usr/bin/chromium-browser https://www.google.fr --noerrdialogs --incognito --s$
```
* **point-rpi** fait pointer la souris sur la framboise du menu, en haut à gauche
* Les lignes **xset** suppriment l’économiseur d’écran (je l'ai gardé dans cet exemple)
* Le ligne **sed** élimine certains messages
* la **dernière ligne** lance le navigateur avec les options nécessaires (voir les options [ici](https://peter.sh/experiments/chromium-command-line-switches/)). En mode Kiosk vous ne pouvez pas fermer le navigateur et il reste au dessus du bureau.  

##### Relancer la Raspberry
```bash
sudo shutdown -r now
```

##### Notes
* J'ai remarqué une consommation non négligeable du CPU en mode kiosk    
* Peut-être que cela était lié à la page Web de Grafana qui chargeait des données avec InfluxDB...  
* Donc ce point est à vérifier en fonction du besoin !  

Liens sources intéressants
--------------------------
- https://blog.eq8.eu/til/raspberi-pi-as-kiosk-load-browser-on-startup-fullscreen.html
- https://www.framboise314.fr/installer-octoprint-avec-lecran-hyperpixel-sur-raspberry-pi/#Enlever_l8217economiseur_d8217ecran_demarrer_chromium_sur_OctoPrint_en_mode_Kiosk

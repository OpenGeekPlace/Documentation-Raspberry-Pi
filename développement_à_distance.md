# Développement à distance

## Pour quoi donc?

### Pour vous passer du tryptique clavier/souris/écran

* Vous ne disposez pas toujours d'assez de claviers, souris et écran pour chacune de vos Rapsberry Pi
* L'ensemble clavier, souris, écran est encombrant
* Votre Pi est peut-être dans un boitier sans accès à sa connectique (dans un thermostat par exemple)

### Le confort

* Votre PC ou Mac est un environnement de développement bien plus puissant et confortable qu'une Pi. Surtout si c'est une Pi Zero
* Si vous développez pour plusieurs Pi à la fois, alors le développement à distance s'impose
* Pour rappel, PyCharm n'est pas disponible sur la Pi.  
Et seule une ancienne version de Visual Studio Code est disponible, via une bidouille peu pérenne.

## Prérequis sur la Pi

### Configurer l'accès SSH

* A faire avant le premier démarrage, depuis le PC où l'on flash la carte uSD.
* A la racine de la partition boot, créer un fichier nommé ssh.  
Le contenu n'a pas d'importance.

### Configurer l'accès Wifi

* Si votre Pi est en "headless" et connectée uniquement en wifi, alors il vous faut renseigner les identifiants wifi depuis le PC où l'on flash la carte uSD
* Editer le fichier /etc/wpa_supplicant/wpa_supplicant.conf
* Ajoutez y l'identifiant et le mot de passe de votre réseau Wifi.
* Exemple :

```(shell)
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
country=FR

network={
	ssid="SFR-6318"
	psk="Le mot de passe"
}

network={
	ssid="Acces_point_du_telephone"
	psk="Le mot de passe"
}
```

* Vous pouvez renseigner plusieurs réseaux Wifi. Ceci permet par exemple de connecter votre Pi à votre mobile en mode access point.

## Recherche de la Pi sur le réseau

* L'application Fing, sur Android et Ios est très pratique

### SSH sans mot de passe

* Vous pouvez échanger, une fois, des clés RSA publiques entre votre PC/Mac et votre Pi,
pour ne plus avoir à entrer de mot de passe lors des connexions SSH
  * <https://thibmaek.com/post/raspberry-pi-login-with-ssh-keys>
  * Générer une clé publique sur votre PC/Mac:  
  ` ssh-keygen -t rsa -C "example@example.com" `
  * Copier la clé publique de votre PC/Mac vers la pi :  
  ` ssh-copy-id pi@hostanme_de_la_pi.local `

## Configuration sur votre PC/Mac
### SSH

* SSH est disponible depuis le PowerShell de Windows 10
* ssh pi@le_hostanme_de_ma_pi
* Vous pouvez configurer une bonne fois pour toutes le nom d'utilisateur pour chaque à laquelle vous vous connectez :  
Ajoutez les informations dans le fichier ~/.ssh/config de votre PC/Mac :

```(shell)
host pi
  hostname raspi.local
  user pi
  port 22

```

* Pour vous connecter, une fois avoir fait l'échange de clés publiques avec votre pi :  
``` ssh hostname_de_la_pi ```

### FileZilla

* Dans le Gestionnaire de connexions, créer une connection pour chacune de vos Pi.
* Ainsi vous n'aurez pas à entrer le nom de la Pi et le mot de passe à chaque connection
* Configurer FilleZilla pour qu'il ne fasse que des transferts binaires

### Visual Studio Code

* Attention à ne pas confondre Visual Studio et Visual Studio Code.  
  Malgré la proximité de nommage, ces deux environnements de développement de Microsoft sont complétement indépendants et leur philosophie est aussi radicalement différente.
* Pour plus de simplicité, je vais nommer Visual Studio Code VSC dans le reste de cet article.
* VSC est gratuit et disponible sur PC Windows, PC Linux et Mac  
  Les dernières versions ne sont disponibles qu'en 64 bits
* Installer VSC sur votre PC ou Mac
  * Il est aussi disponible, dans une ancienne version 32 bits, pour les Pi 2, 3 et 4.  
  Mais il faut bidouiller un peu pour l'installer et le bon fonctionnement n'est pas garanti à terme.
* Pour plus de confort vous pouvez éditer vos fichiers, hébergés sur votre Pi, depuis VSC sur votre PC ou Mac
  * Cela requiert que votre Pi support le jeu d'instruction armv7. Donc malheureusement cela exclu les Pi zero.
  * Les Pi compatibles sont Pi 2, Pi 3 et Pi 4.
  * Installer l'extension Remote - SSH
  * En bas à gauche vous verez alors une petite icone permettant d'ouvrir une fenêtre distante.
  * Plus de détails sur <https://www.digitalocean.com/community/tutorials/how-to-use-visual-studio-code-for-remote-development-via-the-remote-ssh-plugin-fr>
  * Vous avez donc la possibilité de faire du debug pas à pas en Python de votre code s'exécutant sur la Pi, depuis votre PC.
  * Si vous utilisez une Pi Zero, tout n'est pas perdu.
    * Vous pouvez installer l'extension SFTP dans VSC
    * Cette extension vous permet d'éditer vos fichiers sur la Pi depuis VSC sur votre PC.
    * La synchronisation des fichiers entre la Pi et votre PC est alors automatique.
* Dans le terminal mis à disposition dans VSC, vous pouvez vous connecter à la Pi via SSH.  
  Cela vous permet donc de lancer l'éxécution de votre code directement depuis VSC

### VNC

* Vous pouvez accèder à votre Pi via un bureau distant
  * Sur la Pi
    * Requiert d'avoir une installation complète de Raspberry OS, Raspberry OS Lite ne suffit pas.
    * VNC Server est installé par défaut dans Raspberry OS
    * <https://www.framboise314.fr/connectez-vous-de-nimporte-ou-a-votre-raspberry-pi-avec-vnc-connect/>
    * Autorisez le serveur VNC :
      * Dans raspi-config, "Interfacing options"
  * Installez VNC Viewer sur votre PC, Mac ou tablette
    * <https://www.realvnc.com/fr/connect/download/viewer/>

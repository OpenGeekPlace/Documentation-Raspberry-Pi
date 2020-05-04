# Conseils pour une installation de Raspberry Pi

- Le 28/04/2020 : P. Lavernhe : Création
- Le 29/04/2020 : P. Lavernhe : Choix des cartes uSD
- Le 30/04/2020 : P. Lavernhe : Logiciels à installer
- Le 01/05/2020 : P. Lavernhe : Activation du Wifi

## Choix de la carte microSD
* Priviligier une carte avec le label A1
* Le label A1 garanti la performance en I/O par seconde
* Cette performance en I/O a plus d'influence sur les performances de la Pi que le débit séquentiel.
* Il y a d'énormes différences entre cartes sur les débits en écriture de petits fichiers (1Ko ou 4Ko). On parle là d'un facteur > 100.
* Cf <https://forum.armbian.com/topic/954-sd-card-performance/page/3/?tab=comments#comment-49811>
* Autre benchmark, par Thomas kaiser : <https://github.com/ThomasKaiser/Knowledge/blob/master/articles/A1_and_A2_rated_SD_cards.md>
* Il y a beaucup de contrefaçons avec les cartes uSD.
    * Priviligier un vendeur fiable
* Le microcontroleur intégré à la carte SD assure du wear levelling.
    *  <https://www.raspberrypi.org/forums/viewtopic.php?f=29&t=142626&sid=25db7f2a94d635965829922b5043772a>
    * Pour augmenter l'efficatité du wear levelling, vous pouvez utiliser une carte d'une capacité supérieure à vos besoins. Une carte  de 32 Go devrait largement excéder les besoins d'une installation desktop. Et une carte de 16 Go devrait excéder les besoins de Raspbian Lite. 
    * L'écart de prix entre les cartes 16 Go et 32 Go étant faible, la capacité 32 Go est recommandée.
* Modèles recommandables : 
    * Sandisk Ultra A1 32 Go
    * Sandisk Extreme A1 32 Go (plus rapide que l'Ultra)
    * Sandisk, gamme Industrial
* Les cartes avec label A2 sont elles plus rapides sur une Rapsberry Pi ?
    * Les rares tests disponibles sur Raspberry Pi montrent des performances en baisse avec des cartes A2 par rapports aux cartes A1.
    * <https://www.jeffgeerling.com/blog/2019/raspberry-pi-microsd-follow-sd-association-fools-me-twice>
    *   Le label A2 implique un protocole de queuing que le noyau Linux, donc le Raspberry Pi ne supporte pas encore. Donc les cartes A2 sont utilisées dans un mode dégradé sur les Pi.
    *   De futures évolutions du noyau Linux pourraient supporter pleinement les cartes A2. A suivre.
* A noter que les débits séquentiels, sur gros fichiers, sont environ 2x meilleurs avec la Pi 4 par rapport aux modèles précédents. Mais les débits d'accès aléatoire, sur petits fichiers, sont à peine meilleurs que les modèles précédents.
    * <https://www.pidramble.com/wiki/benchmarks/microsd-cards>

## Flasher la carte SD
* Vous pouvez utiliser, entre autres, raspberry pi imager : <https://www.raspberrypi.org/downloads/>
* Choisissez Raspbian si vous avez besoin d'une interface graphique, Raspbian Lite sinon.

## Activation du ssh
* Uniquement si besoin d'accéder à distance
* Pour une utilisation headless, à faire avant le premier démarrage, depuis le PC où l'on flash la carte uSD.
* A la racine de la partition boot, créer un fichier nommé ssh.  
Le contenu n'a pas d'importance.

## Activation du Wifi
* Si votre Pi est en "headless" et connectée uniquementen wifi, alors il vous renseigner les identifiants wifi depuis le PC où l'on flash la carte uSD
* Editer le fichier /etc/wpa_supplicant/wpa_supplicant.conf
* Ajoutez y l'indentifiant et le mot de passe de votre réseau Wifi.
* Exemple :
```
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
country=FR

network={
	ssid="SFR-6318"
	psk="Le mot de passe"
}

network={
	ssid="Padraig"
	psk="Le mot de passe"
}
```
* Vous pouvez renseigner plusieurs réseaux Wifi. Ceci permet par exemple de connecter votre Pi à votre mobile en mode access point.

## Recherche de la Pi sur le réseau
* L'application Fing, sur Android et Ios est très pratique

## Changement du mot de passe
* Dès le premier démarrage prenez soin de changer le mot de passe de l'utilisateur courant, pi.  
	Cet utilisateur a les droits admin.  
	Lui laisser le mot de passe par défaut, "raspberry", est un désastre sécuritaire.
	Surtout si vous activez l'accès SSH.
* Commande `passwd`
* Ou dans raspi-config

## Extension de la partition de base
* Dans raspi-config (Advanced/Expand Filesystem)

## Mise à jour de Raspbian
* sudo apt update
* sudo apt upgrade

## Mise à jour du firmware
* Particulièrement important pour la Pi4 : <https://jamesachambers.com/raspberry-pi-4-bootloader-firmware-updating-recovery-guide/>
 
## Localisation
* Choix du clavier, langue, fuseau horaire, pays pour le wifi, via raspi-config

## Boot sur disque dur
* Les accès, via l'USB 3 de la Pi 4, à un disque sont bien plus rapide que les accès à la carte SD. : <https://www.jeffgeerling.com/blog/2019/raspberry-pi-microsd-card-performance-comparison-2019>.  
L'écart est encore plus grand avec un SSD. 
* Le firmware des Pi 2 et 3 permettent de booter directement sur un périphérique USB, sans avoir besoin de laisser une carte uSD.
    * Mais il faut quand même une carte uSD pour booter une première fois et régler le boot sur USB  
* Le firmware de la pi 4 ne permet, pas encore (avril 2020), de booter directement sur USB.  
    * Mais on peut booter sur USB, puis utiliser un filesystem sur USB.
    * Dans ce cas il n'y a plus d'usure de la uSD puisqu'on ne s'en sert que trés temporairement durant le boot. 
    * Procédure : <https://jamesachambers.com/raspberry-pi-4-usb-boot-config-guide-for-ssd-flash-drives/>
    * Il existe aussi un script qui automatise cet procédure. Vous pouvez retrouver ce scipt dans le forum raspberrypi.org
* Implique éventuellement de désactiver le mode UAS,
  procédure dans <https://jamesachambers.com/raspberry-pi-4-usb-boot-config-guide-for-ssd-flash-drives/>
  * J'ai du désactiver l'UAS pour accèder de façon fiable à mon SSD sur ma Pi4. Les performances baissent, mais j'ai quand même 170 Mo/s en lecture et écriture, et 4450 IOPS en lecture et 6210 IOPS en écriture. Trés supérieur à ce que l'on peut obtenir avec n'importe quelle carte SD sur Pi 4.

## Limiter l'usure de la carte SD
* Choisir une carte équipée d'un controleur offrant un bon wear levelling. Cf paragraphe sur le choix d'une bonne carte SD
* Désactiver le swap
    * sudo nano /proc/sys/vm/swappiness
    * swappiness est à 60 par défaut. Lorsqu'il est mis à 0, le swap n'est utilisé qu'en cas de nécessité absolue (plsu du tout de mémoire disponible).
    * Cette modification es volatile, elle est perude au boot suivant.
    * Attention, si vous activez zram, ne désactivez pas le swap.
  * Activer zram (en cas de RAM trop limitée)
    * Créé un une zone de swap en RAM, compressée en live  
    On perd en charge de calcul du fait de la compression  
    Mais on dispose d'une mémoire plus importante  
    Très interressant sur les Pi 2 et 3, qui n'ont que 1 Go de RAM, mais disposent d'une puissance CPU raisonnable  
    Intéressant aussi sur Pi Zero si vous êtes à l'étroit dans les 512 Mo de RAM.  
    * <https://www.raspberrypi.org/forums/viewtopic.php?t=207304>
    * <https://github.com/StuartIanNaylor/zram-swap-config>

* Installer log2ram
    * Envoi le log dans un ramdisk de 40 Mo.  
    Le log est flushé vers la carte uSD une fois par jour.  
    Cela réduit fortement le nombre d'écritures en flash
    * <https://github.com/azlux/log2ram>
    * 
```
echo "deb http://packages.azlux.fr/debian/ buster main" | sudo tee /etc/apt/sources.list.d/azlux.list
wget -qO - https://azlux.fr/repo.gpg.key | sudo apt-key add -
apt update
apt install log2ram
```

## Nommage

### Attribuer une IP fixe à votre Pi
* Permet retrouver plus facilement votre Pi sur votre réseau local
* Permet d'ajouter une rêgle dans votre routeur pour la rendre visible de l'extérieur
* C'est dans le routeur, fourni par votre Fournisseur d'Accès Internet, que vous pourrez associer une IP fixe à la mac de votre Pi
    * Pour trouver la mac de votre Pi : `ifconfig` 
        * C'est le champ "ether" de la connexion "eth0".
    * Vous pouvez aussi utiliser l'appli Fing pour Android ou Ios

### Changer le nom de la Pi
* Lorsque vous avez plusieurs Pi sur votre réseau, il est facile de les confondre lorsque l'on y accède à distance.
* Vous pouvez aussi utiliser le nom à la place de l'adresse IP lorsque vous voulez accéder à la Pi depuis une autre machine
* Le nom de la Pi s'affiche dans le prompt du Shell.
* Le nom de la Pi est le "hostname"
* Pour le hostname, vous pouvez utiliser :
    * Lettre, en minusulue ou majuscule
    * Chiffres
    * Le tiret "-"
* Vous devez changer le hostname dans 2 fichiers :
    * /etc/hostname
    * /etc/hosts (remplacer le nom associé à 127.0.1.1 par votre hostname)
* Ou bien va raspi-config

### Nommer les autres machines
* Mettez à jour le fichier /etc/hosts pour y indiquer le nom des autres machines de vorte réseau
* Par exemple : 
```
127.0.0.1	localhost
::1		localhost ip6-localhost ip6-loopback
ff02::1		ip6-allnodes
ff02::2		ip6-allrouters

127.0.1.1	pi4
192.168.0.20	imprimante
192.168.0.60	pi-hole
192.168.0.70	hestiapi
192.168.0.90	mosquittopi
192.168.0.100	nas
```

* Vous pourrez ainsi désigner les autres machines par leur nom au lieu de leur adresse IP :
```
> ping imprimante
PING imprimante (192.168.0.20) 56(84) bytes of data.
64 bytes from imprimante (192.168.0.20): icmp_seq=1 ttl=64 time=2.31 ms
```

## Esthétique

### Police système
* La police système par défaut pour les menus (entre autres), n'est pas très lisible.
* Vous pouvez la changer dans "Préférences/Appearance settings/System/font" en choisissant La fonte Piboto, en conservant la taille par défaut de 12.  

### Réglage bordures écran
* Il peut être nécessaire de modifier les réglages dans /boot/config.txt

### VLC avec icones et interlignes trop grands
* Edit /etc/environment with administrator rights.
  * Add the following line: QT_AUTO_SCREEN_SCALE_FACTOR=0
  * Rebooter la Pi
	
### Saccades avec VLC
* <https://www.pcastuces.com/pratique/astuces/4028.htm>

### Désactiver le mode Google Lite dans Firefox
* <https://www.raspberrypi.org/forums/viewtopic.php?t=266016>
* <https://add0n.com/useragent-switcher.html>

## Logiciels à installer

## VLC
* Parce que ça lit tout
* Parce qu'on aime bien l'asso VideoLan

### Editeur de MarkDown
* Pour prendre en note vos étapes d'installation
* sudo apt install retext

### Filezilla
* CLient FTP et SFTP
* sudo apt install filezilla

### Driver imprimante
* Permet d'imprimer depuis la Pi
* Mais permet aussi d'utiliser la Pi en serveur d'impression
    * Vous pouvez ainsi transformer une imprimante USB en imprimante réseau à l'aide d'une simple Pi Zero W
* <https://raspberrytips.com/install-printer-raspberry-pi/>
    
    sudo apt install cups

* Connexion pour ma Canon IP4000R : lpd://imprimante/queue

## Développement à distance

<./développement_à_distance.md>

# Sites d'informations
* <https://github.com/thibmaek/awesome-raspberry-pi>

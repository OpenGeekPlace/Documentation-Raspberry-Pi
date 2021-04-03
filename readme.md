# Conseils pour la mise en oeuvre de Raspberry Pi

## Choix de la carte microSD

* Priviligier une carte avec le label A1  
Le label A1 garanti la performance en I/O par seconde  
Cette performance en I/O a plus d'influence sur les performances de la Pi que le débit séquentiel.  
Il y a d'énormes différences entre cartes sur les débits en écriture de petits fichiers (1 Ko ou 4 Ko). On parle là d'un facteur > 100.
  * Cf <https://forum.armbian.com/topic/954-sd-card-performance/page/3/?tab=comments#comment-49811>
  * Autre benchmark, par Thomas kaiser : <https://github.com/ThomasKaiser/Knowledge/blob/master/articles/A1_and_A2_rated_SD_cards.md>
* Il y a beaucup de contrefaçons avec les cartes uSD.
  * Priviligier un vendeur fiable
* Le microcontroleur intégré à la carte SD assure du wear levelling.
  * <https://www.raspberrypi.org/forums/viewtopic.php?f=29&t=142626&sid=25db7f2a94d635965829922b5043772a>
  * Pour augmenter l'efficacité du wear levelling, vous pouvez utiliser une carte d'une capacité supérieure à vos besoins. Une carte  de 32 Go devrait largement excéder les besoins d'une installation desktop. Et une carte de 16 Go devrait excéder les besoins de Raspberry Pi OS Lite.
  * L'écart de prix entre les cartes 16 Go et 32 Go étant faible, la capacité 32 Go est recommandée.
* Modèles recommandables :
  * Sandisk Ultra A1 32 Go
  * Sandisk Extreme A1 32 Go (plus rapide que l'Ultra)
  * Sandisk, gamme Industrial
* Les cartes avec label A2 sont elles plus rapides sur une Rapsberry Pi ?
  * Les rares tests disponibles sur Raspberry Pi montrent des performances en baisse avec des cartes A2 par rapports aux cartes A1.
  * <https://www.jeffgeerling.com/blog/2019/raspberry-pi-microsd-follow-sd-association-fools-me-twice>
  * Le label A2 implique un protocole de queuing que le noyau Linux, donc le Raspberry Pi ne supporte pas encore. Donc les cartes A2 sont utilisées dans un mode dégradé sur les Pi.
  * De futures évolutions du noyau Linux pourraient supporter pleinement les cartes A2. A suivre.
* À noter que les débits séquentiels, sur gros fichiers, sont environ 2x meilleurs avec la Pi 4 par rapport aux modèles précédents. Mais les débits d'accès aléatoire, sur petits fichiers, sont à peine meilleurs que les modèles précédents.
  * <https://www.pidramble.com/wiki/benchmarks/microsd-cards>

## Flasher la carte SD

* Vous pouvez utiliser, entre autres, Raspberry Pi Imager : <https://www.raspberrypi.org/downloads/>
* Choisissez "Raspberry Pi OS with desktop" si vous avez besoin d'une interface graphique, "Raspberry Pi OS Lite" sinon.  
À noter que vous trouverez souvent des références à Raspbian, qui est l'ancien nom de Raspberry Pi OS.
* Depuis la V1.6 (mars 2021) Raspberry Pi Imager dispose d'une séquence cachée (Crtl Shift x), qui donne accès à des options avancées
  * Nom d'host
  * Mot de passe de l'utilisateur pi
  * Réglage du Wifi (SSID, mot de passe, localisation)
  * Autorisation du ssh
  * Tout ceci évite pas mal de manipulations décrites ci-dessous
  * Malheureusement, à l'usage, sous Windows 10 avec la V1.6, j'ai eu beaucoup d'échecs d'écriture de ces réglages sur la uSD ou le disque USB
  * Ces soucis sont censés être corrigés dans la v1.6.1 sortie peu de temps après la V1.6
  
## Activation du ssh

* Uniquement si besoin d'accéder à distance
* Pour une utilisation headless, à faire avant le premier démarrage, depuis le PC où l'on flash la carte uSD.
* À la racine de la partition boot, créer un fichier nommé ssh.  
Le contenu n'a pas d'importance.

## Activation du Wifi

* Plusieurs méthodes sont possibles

### Via raspi-config

* Nécessite de connecter, au moins temporairement, un clavier et un écran
* Ou bien de se connecter temporairement via Ethernet
  * On peut brancher temporairement un adaptateur USB-Ethernet à une Pi Zero pour cette étape
  * Dans ce cas l'accès se fera via SSH

### Via wpa_supplicant.conf

* Si votre Pi est en "headless" et connectée uniquement en wifi, alors il vous faut renseigner les identifiants wifi depuis le PC Linux , ou bien depuis une autre Pi, où l'on flash la carte uSD
* Editer le fichier wpa_supplicant.conf
  * Vous pouvez placer ce fichier a 2 endroits distincts :
    * A la racine de la partition `boot`, qui est accessible depuis Windows
    * Dans le répertoire `/etc/wpa_supplicant` de la partition rootfs qui n'est pas visible depuis un PC Windows
    * Evitez de le placer dans ces 2 emplacements à la fois.
    Cela serait une source de confusion.
* Ajoutez-y l'identifiant et le mot de passe de votre réseau Wifi.
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

### Via la création d'une image Raspberry Pi OS sur mesure

* Vous pouvez parametrer Raspberry Pi OS avant de le flasher sur la carte SD
* <https://github.com/RPi-Distro/pi-gen>
  * Je ne l'ai pas encore testé personnellement
  * Permet de configurer le Wifi, nom d'utilisateur, mot de passe, langue, clavier, fuseau horaire, autorisation SSH.
  * Cela doit permettre un gain énorme de temps lorsque l'on a plusieurs Pi à installer.

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
* Si vous avez flashé votre carte SD avec raspberry pi imager, cette étape ne devrait pas être nécessaire

## Mise à jour de Raspberry Pi OS

 ```(shell)
 sudo apt update
 sudo apt upgrade
 ```

## Mise à jour du firmware

* Particulièrement important pour la Pi4 : <https://jamesachambers.com/raspberry-pi-4-bootloader-firmware-updating-recovery-guide/>
* Pour la Pi 4, un firmware est disponible, depuis le 3 septembre 2020, qui permet de booter directement sur un disque USB sans même qu'une carte uSD soit présente.
* <https://www.raspberrypi.org/documentation/hardware/raspberrypi/booteeprom.md>
* La commande apt full-upgrade est documentée comme mettant à jour le bootloader en plus des autres logiciels.
  J'ai un doute sur la fait qu'elle mette à jour le firmware.
  Je recommande donc de l'utiliser en plus de rpi-eeprom-upgrade.
  ```(shell)
  sudo apt update
  sudo apt full-upgrade
  sudo reboot
  ```
* Vérifier si une mise à jour du firmware est disponible 
  ```(shell)
  sudo apt update
  sudo rpi-eeprom-update
  ```
* Si une mise à jour du firmware est disponible
  ```(shell)
  sudo rpi-eeprom-update -a
  sudo reboot
  ```

## Localisation

* Choix du clavier, langue, fuseau horaire, pays pour le wifi, via raspi-config

## Boot sur disque dur

* Les accès, via l'USB 3 de la Pi 4, à un disque sont bien plus rapide que les accès à la carte SD. : <https://www.jeffgeerling.com/blog/2019/raspberry-pi-microsd-card-performance-comparison-2019>.  
L'écart est encore plus grand avec un SSD.  
* Le firmware des Pi 2, 3 et 4 permettent de booter directement sur un périphérique USB, sans avoir besoin de laisser une carte uSD.
  * Mais il faut quand même une carte uSD pour booter une première fois et régler le boot sur USB avec raspi-config.
  * Cela résoud la question d'usure de la uSD.
  * Il faut aussi copier une image OS sur le disque SSD. Avec Raspberry Pi Imager par exemple. Ou bien Etcher.
* Implique éventuellement de désactiver le mode UAS,
  procédure dans <https://jamesachambers.com/raspberry-pi-4-usb-boot-config-guide-for-ssd-flash-drives/>
  * J'ai dû désactiver l'UAS pour accèder de façon fiable à mon SSD sur ma Pi4. Les performances baissent, mais j'ai quand même 170 Mo/s en lecture et écriture, et 4450 IOPS en lecture et 6210 IOPS en écriture. Trés supérieur à ce que l'on peut obtenir avec n'importe quelle carte SD sur Pi 4.

## Limiter l'usure de la carte SD

* Choisir une carte équipée d'un controleur offrant un bon wear levelling. Cf paragraphe sur le choix d'une bonne carte SD
* Désactiver le swap
  * sudo nano /proc/sys/vm/swappiness
  * swappiness est à 60 par défaut. Lorsqu'il est mis à 0, le swap n'est utilisé qu'en cas de nécessité absolue (plsu du tout de mémoire disponible).
  * Cette modification est volatile, elle est perude au boot suivant.
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
 
    ```(shell)
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
* Ou bien via raspi-config

### Nommer les autres machines

* Mettez à jour le fichier /etc/hosts pour y indiquer le nom des autres machines de vorte réseau
* Par exemple : 

```(shell)
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

```(shell)
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

```(shell)
    sudo apt install cups
```

* Connexion pour ma Canon IP4000R : lpd://imprimante/queue

## Développement à distance

<https://github.com/OpenGeekPlace/Documentation-Raspberry-Pi/blob/master/développement_à_distance.md>

## Sites d'informations

* <https://github.com/thibmaek/awesome-raspberry-pi>

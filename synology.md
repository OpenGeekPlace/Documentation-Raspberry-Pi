# Conseils pour le partage d'un dossier du NAS Synology

Cette page décrit comment partager un dossier d'un **NAS Synology** sur Raspberry PI.

## Pourquoi ?
- De base, la Raspberry PI tourne sur carte SD, les multiples écritures peuvent la rendre inutilisable avec perte des données. Le NAS peut être utilisé pour des sauvegardes régulières des données.  
- L'utilisation d'une base de données sur une longue période peut conduire à de gros volumes de données. Le NAS peut servir de purge ou de lieu de résidence de la BD.  

## Equipement utilisé pour les tests
- Raspberry PI 3 B+
- Raspbian Buster
- NAS Synology DS216 Play (DSM 6.2)

## Configuration du NAS Synology
##### Création du dossier partagé
Dans le **Panneau de configuration**, menu **Dossier partagé**, créer un nouveau dossier avec:
- Le nom **rasp_folder**  
- Cacher ce dossier...  
- Masquer les sous-dossiers...  
- Activer (ou non la corbeille)...  
- Pour les permissions, sélectionner partout **pas d'accès, sauf éventuellement l'administrateur**  
- Autorisations NFS:  
  - **IP**: celle du NAS  
  - **Squash**: Pas de mappage  
  - **Privilège**: Lecture/Ecriture  
  - **Sécurité**: sys  
  - **Activer** le mode synchrone  
  - **Permettre** les connexions...  
  - **Permettre** à des utilisateurs...  

##### Création du user dédié
Dans le **Panneau de configuration**, menu **Utilisateurs**:
- Créer un nouvel utilisateur appelé ici **rasp_user**  
- Concernant les persmissions, ne lui donner accès à __aucun dossier__  

##### A noter pour la suite de l'installation
- **rasp_folder**: le nom du dossier partagé créé  
- **rasp_user**: le nom du user créé pour accéder au dossier  
- **rasp_pass**: le mot de passe du user créé pour accéder au dossier 
- **nas_piaddr**: l'adresse IP du NAS  

## Configuration de la Raspberry
##### Installation des librairies requises sous Raspberry
```bash
sudo apt-get update
sudo apt-get install cifs-utils
```

##### Vérification de fonctionnement
Par exemple, dans le HOME, créer un dossier **nas** 
```bash
mkdir ~/nas
```
Lancer la commande suivante pour monter ce dossier sur le dossier partagé du NAS
```bash
sudo mount -t cifs -o user=rasp_share,domain=.,password=rasp_pass,uid=1000 //nas_ipaddr/rasp_folder /home/pi/nas
```
Essayer de regarder dans le dossier et de mettre des fichiers  
Puis, lancer la commande suivante pour démonter ce dossier
```bash
sudo umount -l /home/pi/nas
```

##### Montage au Boot
Il faut activer l'attente du réseau au boot: option "Wait for Network at Boot" dans le menu Boot
```bash
sudo raspi-config
```
Créer un fichier (par exemple **.smbNas**) en root
```bash
sudo su
nano /root/.smbNas
```
Et y copier le mot de login/MdP pour la connexion 
```bash
username=rasp_user
password=rasp_pass
```
CTRL+X pour sauvegarder.
Puis modifier les droits du fichier  
```bash
chmod 700 /root/.smbNas
```
Quitte le mode root
```bash
exit
```
Modifier le fichier **fstab**  
```bash
sudo nano /etc/fstab
```
En y ajoutant la ligne suivante  
```bash
//nas_piaddr/rasp_folder /home/pi/nas cifs credentials=/root/.smbNas,uid=1000 0 0
```
Tester le fstab avec la commande suivante  
```bash
sudo mount -a
```
Essayer de regarder dans le dossier et de mettre des fichiers  
Puis, faire un reboot pour tester la connexion en Boot  
```bash
sudo reboot now
```

Liens sources intéressants
--------------------------
- https://www.youtube.com/watch?v=RIS482WvbM4  

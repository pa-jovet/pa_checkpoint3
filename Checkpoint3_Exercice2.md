### Partie 1 : Gestion des utilisateurs

**Q.2.1.1** Sur le serveur, créer un compte pour ton usage personnel.
![[Exercice 2/Pasted image 20250117101922.png]]
**Q.2.1.2** Quelles préconisations proposes-tu concernant ce compte ?
- S'il s'agit d'un compte utilisateur classique : 
	- Mot de passe fort ;
	- Appartenance aux groupes de sécurités nécessaires seulement ;
	- Accès aux répertoires nécessaires seulement ;
- S'il s'agit d'un compte administrateur :
	- Mot de passe très fort, différent du compte utilisateur classique et à changer régulièrement ;
	- Appartenance au groupe `sudo` ;
### [](https://odyssey.wildcodeschool.com/quests/2934/pages/12385#partie-2--configuration-de-ssh)Partie 2 : Configuration de SSH

**Q.2.2.1** Désactiver complètement l'accès à distance de l'utilisateur root
Dans `/etc/ssh/sshd_config`, modifier les lignes ainsi :
```
	PermitRootLogin no
```

**Q.2.2.2** Autoriser l'accès à distance à ton compte personnel uniquement
Dans `/etc/ssh/sshd_config`, modifier les lignes ainsi :
``` bash
	AllowUsers pa   # on peut remplacer pa par le nom d'utilisateur souhaité
```

**Q.2.2.3** Mettre en place une authentification par clé valide et désactiver l'authentification par mot de passe
Dans `/etc/ssh/sshd_config`, modifier les lignes ainsi :
``` bash
	PubkeyAuthentication yes
	PasswordAuthentication no
```


### [](https://odyssey.wildcodeschool.com/quests/2934/pages/12385#partie-3--analyse-du-stockage)Partie 3 : Analyse du stockage

**Q.2.3.1** Quels sont les systèmes de fichiers actuellement montés ?
Avec une commande `df -h`, on voit que les systèmes de fichiers `/dev/mapper/cp3--bg-root` et `/dev/md0p1` sont actuellement montés ainsi que plusieurs systèmes temporaires ;

**Q.2.3.2** Quel type de système de stockage ils utilisent ?
- Avec une commande `lsblk -l` on peut voir dans la colonne "Type" que :
	- `/dev/mapper/cp3--vg-root` est en LVM
	- `/dev/md0p1` est en RAID1

**Q.2.3.3** Ajouter un nouveau disque de 8,00 Gio au serveur et réparer le volume RAID
- Après avoir créé le nouveau disque, redémarrer le serveur et effectuer les lignes de commandes suivantes : 
``` bash
cfdisk /dev/sdb # créer une partition primaire gpt avec l'entièreté de l'espace disponible

apt install mdadm # nécessaire pour la gestion de RAID, vérifier qu'il est présent sur le serveur

mdadm --manage /dev/md0 --add /dev/sdb1 # pour ajouter la nouvelle partition sdb1 au RAID1 de md0

mdadm --detail /dev/md0 # pour vérifier que le RAID se reconstruit proprement
```

**Q.2.3.4** Ajouter un nouveau volume logique LVM de 2 Gio qui servira à héberger des sauvegardes. Ce volume doit être monté automatiquement à chaque démarrage dans l'emplacement par défaut : `/var/lib/bareos/storage`.
- Effectuer les commandes suivantes :
``` bash
lvcreate -L 2G -n save cp3-vg
mkdir /var/lib/bareos/storage/SAVE
mkfs -t ext4 /dev/cp3-vg/save
mount /dev/cp3-vg/save /var/lib/bareos/storage/SAVE
```
- Puis modifier le fichier /etc/fstab et ajouter la ligne : 
```
/dev/cp3-vg/save /var/lib/bareos/storage/SAVE ext4 defaults 0 0
```

**Q.2.3.5** Combien d'espace disponible reste-t-il dans le groupe de volume ?
Avec une commande `vgdisplay`, on note que l'espace disponible pour le groupe de volume `cp3-vg` est `<1.79 Gb`.
### [](https://odyssey.wildcodeschool.com/quests/2934/pages/12385#partie-4--sauvegardes)Partie 4 : Sauvegardes

Le logiciel bareos est installé sur le serveur.  
Les composants `bareos-dir`, `bareos-sd` et `bareos-fd` sont installés avec une configuration par défaut.

**Q.2.4.1** Expliquer succinctement les rôles respectifs des 3 composants bareos installés sur la VM.
- Le composant bareos-dir est le Bareos Director qui dirige les différents composants du système de sauvegarde Bareos ;
- Le composant bareos-sd est le Bareos Storage Daemon. Il gère les sauvegardes de fichiers.
- Le composant bareos-fd est le Bareos File Daemon. Il s'installe sur chaque machine où se situent des fichiers que l'on souhaite sauvegarder et permet de remonter ceux-ci vers le Storage Daemon.

### [](https://odyssey.wildcodeschool.com/quests/2934/pages/12385#partie-5--filtrage-et-analyse-r%C3%A9seau)Partie 5 : Filtrage et analyse réseau

**Q.2.5.1** Quelles sont actuellement les règles appliquées sur Netfilter ?
Dans /root/nftables/config.nft, on trouve la règle inet_filter_table.

**Q.2.5.2** Quels types de communications sont autorisées ?
Les connexions authentifiées, les entrées sur l'interface locale, le protocole TCP à destination du port SSH (par défaut le port 22), les protocoles ICMP (pings) IPv4 et IPv6.

**Q.2.5.3** Quels types sont interdit ?
Les connexions SSH non authentifiées et tout ce qui n'a pas été explicitement autorisé.

**Q.2.5.4** Sur nftables, ajouter les règles nécessaires pour autoriser bareos à communiquer avec les clients bareos potentiellement présents sur l'ensemble des machines du réseau local sur lequel se trouve le serveur.
Il faut ajouter à la configuration les lignes : 
```
tcp dport 9101 accept
tcp dport 9103 accept
```


### [](https://odyssey.wildcodeschool.com/quests/2934/pages/12385#partie-6---analyse-de-logs)Partie 6 : Analyse de logs

**Q.2.6.1** Lister les 10 derniers échecs de connexion ayant eu lieu sur le serveur en indiquant pour chacun :

- La date et l'heure de la tentative
- L'adresse IP de la machine ayant fait la tentative

| Date et heure de la tentative | Adresse IP de la machine source |
|===================|======================|
|20/12/2024 10:24:10| 10.0.0.199 |
| 03/01/2025 11:06:30 | 10.0.0.199 |
| 03/01/2025 11:06:45 | 10.0.0.199 |
| 03/01/2025 11:23:03 | 10.0.0.199 |
| 03/01/2025 11:23:34 | 10.0.0.199 |
| 03/01/2025 11:23:40 | 10.0.0.199 |
| 03/01/2025 11:23:44 | 10.0.0.199 |
| 03/01/2025 12:09:28 | fd26:ba41:c8d6:0:ba92:6393:cc55:8b8d |
| 03/01/2025 12:09:34 | fd26:ba41:c8d6:0:ba92:6393:cc55:8b8d |

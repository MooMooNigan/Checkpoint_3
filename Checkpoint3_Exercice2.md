# Checkpoint 3 - Exercice 2

## Partie 1

### Q.2.1.1 
```bash
# Création du compte manuellement
root@SRVLX01:~# adduser moo
Ajout de l'utilisateur « moo » ...
Ajout du nouveau groupe « moo » (1001) ...
Ajout du nouvel utilisateur « moo » (1001) avec le groupe « moo » ...
Création du répertoire personnel « /home/moo »...
Copie des fichiers depuis « /etc/skel »...
Nouveau mot de passe :
Retapez le nouveau mot de passe :
Les mots de passe ne correspondent pas.
passwd: Erreur de manipulation du jeton d’authentification
passwd: password unchanged
Essayer à nouveau ? [o/N]o
Nouveau mot de passe :
Retapez le nouveau mot de passe :
passwd: password updated successfully
Changing the user information for moo
Enter the new value, or press ENTER for the default
        Full Name []: Damien
        Room Number []: 404
        Work Phone []: 04.22.52.10.10
        Home Phone []: 04.22.52.10.10
        Other []:
Cette information est-elle correcte ? [O/n]o

# Vérification de la création du compte
root@SRVLX01:~# cat /etc/passwd | grep moo
moo:x:1001:1001:Damien,404,04.22.52.10.10,04.22.52.10.10:/home/moo:/bin/bash

# Vérification que sudo est bien installé
root@SRVLX01:~# apt list sudo
En train de lister... Fait
sudo/stable 1.9.5p2-3 amd64

# Ajout de mon compte au groupe sudo
root@SRVLX01:~# usermod -aG sudo moo

# Vérification des groupes présent pour mon compte
root@SRVLX01:~# cat /etc/group | grep moo
sudo:x:27:moo
moo:x:1001:
```

### Q.2.1.2
Je préconise d'ajouter ce compte au groupe sudo, étant un compte personnel, il est nécessaire d'avoir les droits d'administration sur le serveur afin d'éviter de se servir du compte root. De plus, l'utilisation du compte root n'est pas recommandée, les droits peuvent être trop élévé pour certaine manipulation, et cela pose un problème de tracabilité lorsque des actions sont effectuées sur le serveur.

## Partie 2
### Q.2.2.1 - Q.2.2.2 
```bash
# Utilise nano pour éditer le fichier de configuration sshd
root@SRVLX01:~# nano /etc/ssh/sshd_config

...
# Q.2.2.1 - Désactivation de la connexion en root
PermitRootLogin no

# Q.2.2.2 - Authorise uniquement le compte "moo" à se connecter
AllowUsers moo

# Q.2.2.3 - Activation de la connexion par clé 
PubkeyAuthentication yes

# Q.2.2.3 - Désactivation de la connexion par mot de passe
PasswordAUthentication no
...

# Redémarrage du service ssh
root@SRVLX01:~# systemctl restart sshd
```
### Q.2.2.2 
```bash
moo@SRVLX01:~$ ssh-keygen -t rsa -b 4096
Generating public/private rsa key pair.
Enter file in which to save the key (/home/moo/.ssh/id_rsa):
Created directory '/home/moo/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/moo/.ssh/id_rsa
Your public key has been saved in /home/moo/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:jrN06cLazxPL6uszUrEpNNCnNUUmB6X4ssEUJMjoRLo moo@SRVLX01
The key's randomart image is:
+---[RSA 4096]----+
|+o.oo o=*        |
|+o...o+=         |
|+  .o+..         |
| o o+..          |
|E  .+..+S        |
|    .++o..       |
|    .++.+o       |
|    .o=*+        |
|    .=BB+.       |
+----[SHA256]-----+
```

# Partie 3
### Q.2.3.1
Il y a :
- Un FileSystem `linux_raid_member` pour la partition RAID `sda1`
- Un FileSystem `ext2` pour la partition de boot `md0p1`
- Un FileSystem `LVM2_member` pour la paritition `md0p5`
- Un FileSystem `ext4` pour la partition LVM `cp3--vg-root`
- Un FileSystem `swap` pour la partition LVM `cp3--vg-swap_1`

### Q.2.3.2
Le disque `sda` utilise deux type de stockage en RAID 1 (Redundant Array of Inexpensive Disks) aussi appeler RAID Mirroring et en LVM (Logical Volume Manager). 

### Q.2.3.3
```bash

# Partitionnement du nouveau disque de 8Go /dev/sdb
root@SRVLX01:~# fdisk /dev/sdb

Bienvenue dans fdisk (util-linux 2.36.1).
Les modifications resteront en mémoire jusqu'à écriture.
Soyez prudent avant d'utiliser la commande d'écriture.

Le périphérique ne contient pas de table de partitions reconnue.
Création d'une nouvelle étiquette pour disque de type DOS avec identifiant de disque 0x3c194653.

# Modification de la table de partition en GPT
Commande (m pour l'aide) : g
Une nouvelle étiquette de disque GPT a été créée (GUID : D1E57010-75CF-E147-9B85-0EE5E6837A82).

# Création d'une nouvelle partition sur l'entièreté du disque /dev/sdb
Commande (m pour l'aide) : n
Numéro de partition (1-128, 1 par défaut) :
Premier secteur (2048-16777182, 2048 par défaut) :
Dernier secteur, +/-secteurs ou +/-taille{K,M,G,T,P} (2048-16777182, 16777182 par défaut) :

Une nouvelle partition 1 de type « Linux filesystem » et de taille 8 GiB a été créée.

# Modification du type de la partition en Partition RAID LINUX
Commande (m pour l'aide) : t
Partition 1 sélectionnée
Type de partition ou synonyme (taper L pour afficher tous les types) : raid
Type de partition « Linux filesystem » modifié en « Linux RAID ».

# Ecriture de la nouvelle partition
Commande (m pour l'aide) : w
La table de partitions a été altérée.
Appel d'ioctl() pour relire la table de partitions.
Synchronisation des disques.

# Vérification de la nouvelle partition
root@SRVLX01:~# lsblk
NAME                   MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINT
sda                      8:0    0     8G  0 disk
└─sda1                   8:1    0     8G  0 part
  └─md0                  9:0    0     8G  0 raid1
    ├─md0p1            259:0    0 488,3M  0 part  /boot
    ├─md0p2            259:1    0     1K  0 part
    └─md0p5            259:2    0   7,5G  0 part
      ├─cp3--vg-root   253:0    0   2,8G  0 lvm   /
      └─cp3--vg-swap_1 253:1    0   976M  0 lvm   [SWAP]
sdb                      8:16   0     8G  0 disk
└─sdb1                   8:17   0     8G  0 part
sr0                     11:0    1  1024M  0 rom

# Reconstruction du RAID 1
root@SRVLX01:~# mdadm --manage /dev/md0 --add /dev/sdb1
mdadm: added /dev/sdb1

# Vérification que la reconstruction du RAID s'est bien effectuée
root@SRVLX01:~# mdadm --detail /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Tue Dec 20 10:02:28 2022
        Raid Level : raid1
        Array Size : 8381440 (7.99 GiB 8.58 GB)
     Used Dev Size : 8381440 (7.99 GiB 8.58 GB)
      Raid Devices : 2
     Total Devices : 2
       Persistence : Superblock is persistent

       Update Time : Fri Jan 17 12:04:42 2025
             State : clean, degraded, recovering
    Active Devices : 1
   Working Devices : 2
    Failed Devices : 0
     Spare Devices : 1

Consistency Policy : resync

    Rebuild Status : 31% complete

              Name : cp3:0
              UUID : 32332561:cf16c858:703517e8:81dd5c10
            Events : 2956

    Number   Major   Minor   RaidDevice State
       0       8        1        0      active sync   /dev/sda1
       2       8       17        1      spare rebuilding   /dev/sdb1

# Vérification des nouvelle partition RAID et LVM
root@SRVLX01:~# lsblk
NAME                   MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINT
sda                      8:0    0     8G  0 disk
└─sda1                   8:1    0     8G  0 part
  └─md0                  9:0    0     8G  0 raid1
    ├─md0p1            259:0    0 488,3M  0 part  /boot
    ├─md0p2            259:1    0     1K  0 part
    └─md0p5            259:2    0   7,5G  0 part
      ├─cp3--vg-root   253:0    0   2,8G  0 lvm   /
      └─cp3--vg-swap_1 253:1    0   976M  0 lvm   [SWAP]
sdb                      8:16   0     8G  0 disk
└─sdb1                   8:17   0     8G  0 part
  └─md0                  9:0    0     8G  0 raid1
    ├─md0p1            259:0    0 488,3M  0 part  /boot
    ├─md0p2            259:1    0     1K  0 part
    └─md0p5            259:2    0   7,5G  0 part
      ├─cp3--vg-root   253:0    0   2,8G  0 lvm   /
      └─cp3--vg-swap_1 253:1    0   976M  0 lvm   [SWAP]
sr0
```
### Q.2.3.4


## Partie 4
### Q.2.4.1
- `bareos-dir` est le daemon pour la gestion des répertoires de sauvegarde (Bareos Directory Daemon)
- `bareos-sd` est le daemon pour la gestion de l'espace de stockage des sauvegarde (Bareos Storage Daemon)
- `bareos-fd` est le daemon pour la gestion des fichiers à sauvegarder (Bareos File Daemon)


## Partie 5
### Q.2.5.1
Par défaut il rejette (drop) tous les paquets entrant. Cependant il accepte les types de paquets suivants :
- Accepte les paquets de l'Interface Boucle Local (`lo`) 
- Accepte les paquets du port SSH par défaut (port 22)
- Accepte les paquets du protocole ICMP pour l'IPv4
- Accepte les paquets du protocole ICMP pour l'IPv6

### Q.2.5.2
Les communications de type ping (ICMP) en IPv4/IPv6 et SSH sur le port 22 sont authorisées.

### Q.2.5.3
Tous les autres type de connexion entrante sont refusées.

### Q.2.5.4
```bash
#!/usr/sbin/nft -f

flush ruleset

table inet inet_filter_table {
        chain in_chain {
                type filter hook input priority filter; policy drop;
                ct state established, related accept
                ct state invalid drop
                iifname lo accept
                tcp dport ssh accept
                ip protocol icmp accept
                ip6 nexthdr icmpv6 accept
                tcp dport 9101-9103 accept
        }
}
```

## Partie 6
### Q.2.6.1
```bash
# Vérificaiton des fichier log d'authentification
root@SRVLX01:~# ls -lau /var/log
...
-rw-r-----   1 root        adm              11857 16 janv. 09:44 auth.log
-rw-r-----   1 root        adm              24290 20 déc.   2023 auth.log.1
-rw-r-----   1 root        adm               2547  3 janv.  2023 auth.log.2.gz
-rw-r-----   1 root        adm               1998 20 déc.   2022 auth.log.3.gz
...

# Décompression des fichier gzip
root@SRVLX01:~# gzip -d /var/log/auth.log.2.gz
root@SRVLX01:~# gzip -d /var/log/auth.log.3.gz

# Commande des 10 dernières tentatives de connexion échouées
root@SRVLX01:~# cat /var/log/auth.* | grep "authentication failure" | tail
Dec 21 14:12:30 SRVLX01 login[496]: pam_unix(login:auth): authentication failure; logname=LOGIN uid=0 euid=0 tty=/dev/tty1 ruser= rhost=
Dec 21 14:39:27 SRVLX01 su: pam_unix(su-l:auth): authentication failure; logname=wilder uid=1000 euid=0 tty=pts/0 ruser=wilder rhost=  user=root
Jan  3 11:06:28 cp3 sshd[1157]: pam_unix(sshd:auth): authentication failure; logname= uid=0 euid=0 tty=ssh ruser= rhost=10.0.0.199  user=root
Jan  3 11:06:43 cp3 sshd[1161]: pam_unix(sshd:auth): authentication failure; logname= uid=0 euid=0 tty=ssh ruser= rhost=10.0.0.199  user=wilder
Jan  3 11:23:00 cp3 sshd[1227]: pam_unix(sshd:auth): authentication failure; logname= uid=0 euid=0 tty=ssh ruser= rhost=10.0.0.199
Jan  3 11:23:31 cp3 sshd[1231]: pam_unix(sshd:auth): authentication failure; logname= uid=0 euid=0 tty=ssh ruser= rhost=10.0.0.199  user=root
Jan  3 11:23:45 cp3 sshd[1231]: PAM 2 more authentication failures; logname= uid=0 euid=0 tty=ssh ruser= rhost=10.0.0.199  user=root
Jan  3 12:09:26 cp3 sshd[1587]: pam_unix(sshd:auth): authentication failure; logname= uid=0 euid=0 tty=ssh ruser= rhost=fd26:ba41:c8d6:0:ba92:6393:cc55:8b8d
Jan  3 12:09:37 cp3 sshd[1587]: PAM 1 more authentication failure; logname= uid=0 euid=0 tty=ssh ruser= rhost=fd26:ba41:c8d6:0:ba92:6393:cc55:8b8d
Dec 20 10:24:08 cp3 sshd[499]: pam_unix(sshd:auth): authentication failure; logname= uid=0 euid=0 tty=ssh ruser= rhost=10.0.0.199  user=root
```



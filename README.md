# RAID-LINUX
---
## 💪 Challenge

Dans un premier temps, exécute les différentes actions de cette quête :

- Créer un RAID 1 avec 2 disques
- Reconstruire le RAID après une simulation de panne

Une fois que tout est fonctionnel, répétez l'opération, mais cette fois-ci avec un autre type de RAID, comme le RAID 5 par exemple.

## 🧐 Critères d'acceptation

- Création d'un RAID 1 fonctionnel
- Création d'un autre type de RAID fonctionnel
---


# Solution : Configuration RAID sous Linux

## Étape 1 : Installation d'Ubuntu sur VirtualBox

J'ai créé une machine virtuelle avec un disque dur de 20 Go pour l'installation d'Ubuntu.
La configuration du disque de 20 Go dans VirtualBox est comme ci-dessous :

![C1_HDD120](https://github.com/user-attachments/assets/f3acab73-f985-43b2-884b-c5b4db708a34)


## Étape 2 : Ajout des disques supplémentaires

J'ai ajouté trois disques supplémentaires de 5 Go chacun pour les manipulations sur le RAID. L'option "Branchable à chaud" ou "Hot-Pluggable"(version anglaise de virtualbox) a été activée pour permettre la déconnexion à chaud des disques.

La configuration des disques de 5 Go dans VirtualBox est comme ci-dessous :

![C1_HDD25G](https://github.com/user-attachments/assets/86b92b55-d6cd-4bff-942c-741e0b88a062)

Pour ajouter un disque il y a l'icone avec un plus dessus qu'on peut apercevoir en bas du cadre "Unités de stockage". Configurer comme sur la capture d'écran le nouveau disque et répéter l'opération deux fois.   
*Note*: Les disques sont correctement configurés pour être branchables à chaud, ce qui est essentiel pour la simulation de pannes lors de la configuration du RAID.

## Étape 3 : Création du RAID 1

### Vérification des disques disponibles

Après le redémarrage de la VM, j'ai vérifié que les disques supplémentaires étaient bien reconnus par le système. Pour cela, j'ai utilisé la commande `lsblk` qui liste tous les périphériques de stockage disponibles sur le système.

Voici la sortie de la commande :

![lsblk](https://github.com/user-attachments/assets/dc0d6e0a-5c80-44e7-af61-77568db53a6e)


Comme on peut le voir dans la capture d'écran, les disques suivants sont disponibles :
- `/dev/sda` : Le disque principal de 20 Go utilisé pour l'installation de l'OS.
- `/dev/sdb` : Le premier disque supplémentaire de 5 Go.
- `/dev/sdc` : Le deuxième disque supplémentaire de 5 Go.
- `/dev/sdd` : Le troisième disque supplémentaire de 5 Go.

Ces disques sont désormais prêts à être utilisés pour la création du RAID 1.

### Partitionnement du disque `/dev/sdb` pour le RAID 1

Pour préparer le disque `/dev/sdb` à être utilisé dans la configuration RAID 1, j'ai utilisé l'outil `fdisk`. Voici les étapes que j'ai suivies :

1. **Lancer `fdisk`** :
- J'ai lancé `fdisk` avec la commande suivante pour modifier les partitions du disque `/dev/sdb` :
```bash
sudo fdisk /dev/sdb
```

2. **Création d'une nouvelle partition** :
- Après avoir lancé `fdisk`, la première étape consiste à créer une nouvelle partition primaire :
     - **Commande `n`** : J'ai sélectionné `n` pour créer une nouvelle partition.
     - **Choix de la partition primaire `p`** : J'ai ensuite sélectionné `p` pour indiquer qu'il s'agirait d'une partition primaire.
     - **Numéro de partition** : J'ai accepté la valeur par défaut `1` pour le numéro de partition.
     - **Secteurs de début et de fin** : J'ai accepté les valeurs par défaut pour que la partition utilise tout l'espace disponible sur le disque.

3. **Définir le type de partition** :
- Pour configurer cette partition à être utilisée dans un RAID, j'ai changé le type de la partition :
     - **Commande `t`** : J'ai utilisé `t` pour changer le type de la partition.
     - **Type `fd`** : J'ai saisi `fd` pour spécifier que cette partition serait utilisée pour un RAID Linux (Linux RAID autodetect).

4. **Enregistrement des modifications** :
- Enfin, j'ai sauvegardé les modifications et quitté `fdisk` :
  - **Commande `w`** : J'ai utilisé `w` pour écrire les modifications sur le disque et quitter l'outil `fdisk`.

Voici une capture d'écran montrant l'ensemble du processus pour le disque `/dev/sdb` :


![C4_fdisksdb](https://github.com/user-attachments/assets/2322992c-b983-4824-a9b1-b21774d42408)


*Note*: Les mêmes étapes ont été suivies pour le disque `/dev/sdc` afin de le préparer pour le RAID 1.

Voici une capture d'écran montrant l'ensemble du processus pour le disque `/dev/sdc` :   

![C5_fdisksdc](https://github.com/user-attachments/assets/13c1148d-fe59-4735-b95d-cc64e8972886)    

### Installation de `mdadm`(en tant que superutilisateur)

Pour créer et gérer le RAID sous Linux, j'ai d'abord installé l'outil `mdadm`, qui n'était pas préinstallé sur le système Ubuntu.
**mdadm** est un utilitaire Linux utilisé pour gérer les ensembles RAID logiciels (Redundant Array of Independent Disks) pour le stockage de données.

1. **Mise à jour des paquets** :
- J'ai mis à jour la liste des paquets disponibles avec la commande suivante :
```bash
apt update
```

2. **Installation de mdadm** :
   
- Ensuite, j'ai installé mdadm avec la commande suivante :

```bash
apt install mdadm -y
```

3. **Vérification de l'installation** :

- Puis, j'ai vérifié que mdadm est bien installé avec la commande suivante :

```bash
mdadm --version
```

- **Intégration des captures d'écran** : J'ai inclus les captures d'écran aux moments où elles sont pertinentes dans le processus, c'est-à-dire après la mise à jour des paquets et après l'installation de `mdadm` et la vérification de sa version. Les voici :

![C6_updandinstalmdadm1](https://github.com/user-attachments/assets/160db022-f807-46ec-b29c-21c37017a3d8)

![C7_endinstandvers](https://github.com/user-attachments/assets/35420f1c-5534-4792-9ada-9144d773c278)

### Création du RAID 1 avec `mdadm`

J'ai utilisé l'outil `mdadm` pour créer un RAID 1 avec les partitions `/dev/sdb1` et `/dev/sdc1`. La commande suivante a été utilisée :

```bash
mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb1 /dev/sdc1
```

Cette commande a configuré un RAID 1 sur le périphérique /dev/md0, utilisant les deux partitions spécifiées.   

Vérification de l'état du RAID 1

Pour vérifier que le RAID 1 a été correctement créé et qu'il est en état "clean", j'ai utilisé la commande suivante :


```bash
cat /proc/mdstat
```

- **Capture d'écran** :

![C8_raid1crandver](https://github.com/user-attachments/assets/8f134d0c-1eab-49c4-acdb-464f7d3a3c98)    

**Explication des résultats** :

Le RAID 1 est bien actif (md0 : active raid1).
Les deux partitions, /dev/sdc1 et /dev/sdb1, sont correctement intégrées au RAID 1.
Le statut [UU] indique que les deux disques sont synchronisés et en bon état (Up).
Le processus de resynchronisation (resync) est visible, et une fois terminé, le RAID sera complètement opérationnel.

**Détails du RAID 1** :    

Pour obtenir plus de détails sur le RAID que je viens de créer, j'ai utilisé la commande suivante :   

```bash
mdadm --detail /dev/md0
```

![C9_detailraid](https://github.com/user-attachments/assets/030972c8-4053-4c3b-abcc-19def7d42715)


**Explication des résultats affichés** :  

- Version : Le RAID est configuré avec la version 1.2 du superblock de mdadm.
- Creation Time : Le RAID a été créé à la date et heure indiquées.
- Raid Level : Le RAID est de type 1 (mirroring).
- Array Size : La taille totale du RAID est de 4.99 GiB, correspondant à la -taille d'un des disques (car dans un RAID 1, la taille totale est égale à la taille du plus petit disque).
- State : Le RAID est en état "clean", signifiant qu'il est opérationnel et que les disques sont synchronisés.
- Active Devices : Les deux disques sont actifs et synchronisés (active sync pour /dev/sdb1 et /dev/sdc1).
- UUID : Un UUID unique a été attribué au RAID pour son identification

**Conclusion**
Le RAID 1 a été créé avec succès, et les deux disques sont correctement synchronisés. Le RAID est maintenant opérationnel et prêt à être utilisé pour la sauvegarde des données.

Prochaine étape : Le RAID 1 étant fonctionnel, nous passerons à son formatage et à son montage sur le système de fichiers.

### Formatage et montage du RAID 1

#### Formatage du RAID 1

Pour pouvoir utiliser le RAID 1, j'ai d'abord formaté le périphérique RAID `/dev/md0` avec le système de fichiers `ext4` :

```bash
mkfs.ext4 /dev/md0 -L "RAID1_Data"
```

Explication :

- mkfs.ext4 : Cette commande formate le périphérique avec le système de fichiers ext4.
- /dev/md0 : C'est le périphérique RAID que nous venons de créer.
- -L "RAID1_Data" : Ce paramètre attribue un label (nom) au système de fichiers, ici "RAID1_Data".

#### Création d'un point de montage

Commande pour créer le répertoire de montage :

```bash
mkdir /mnt/raid1
```
Explication :

- mkdir : Cette commande crée un nouveau répertoire.
- /mnt/raid1 : C'est le chemin du répertoire que nous créons pour monter le RAID.

#### Montage du RAID 1

```bash
mount /dev/md0 /mnt/raid1
```
Explication :

- mount : Cette commande monte le système de fichiers sur un répertoire.
- /dev/md0 : Le périphérique RAID que nous avons formaté.
- /mnt/raid1 : Le répertoire où nous montons le RAID.

#### Vérification du montage

```bash
mount /dev/md0 /mnt/raid1
```
Explication :

- df -h : Cette commande affiche la liste des systèmes de fichiers montés avec leur espace utilisé et disponible en format lisible (human-readable).

Voici une copie d'écran de toutes les commandes utilisées :

![C10_montageraid1](https://github.com/user-attachments/assets/ff51c257-ef95-4fc7-9eb2-bab66fc46423)  

## Étape 3 : Simulation d'une panne et reconstruction du RAID 1

Dans cette étape, nous allons simuler une panne d'un des disques du RAID 1 et observer comment le RAID réagit. Ensuite, nous allons reconstruire le RAID en remplaçant le disque défaillant.

#### Simuler la panne d'un disque

Nous allons marquer l'un des disques comme défaillant pour simuler une panne.

Commande pour marquer un disque comme défaillant :

```bash
mdadm --manage /dev/md0 --fail /dev/sdb1
```
**Explication** :

- mdadm --manage : Cette commande permet de gérer un ensemble RAID existant.
- --fail /dev/sdb1 : Marque la partition /dev/sdb1 comme défaillante dans l'ensemble RAID /dev/md0.


Vérification de l'état après la panne en tapant la commande :

```bash
cat /proc/mdstat
```
Voici la réponse à la commande : 

```bash
root@karim-VirtualBox:/home/karim# cat /proc/mdstat
Personalities : [raid1]
md0 : active raid1 sdc1[1] sdb1[0](F)
      5236736 blocks super 1.2 [2/1] [_U]

unused devices: <none>
root@karim-VirtualBox:/home/karim# 
```
**Explication réponse** :   

md0 : active raid1 sdc1[1] sdb1[0](F)

- sdb1[0](F) indique que la partition /dev/sdb1 est considérée comme défaillante (F pour "faulty").
- Le RAID continue de fonctionner en mode dégradé avec la seule partition active restante, /dev/sdc1.

J'ai ensuite retiré le disque défaillant du RAID avec la commande suivante : 

```bash
mdadm --manage /dev/md0 --remove /dev/sdb1
```
Voici la réponse à la commande : 

```bash
root@karim-VirtualBox:/home/karim# mdadm --manage /dev/md0 --remove /dev/sdb1
mdadm: hot removed /dev/sdb1 from /dev/md0
root@karim-VirtualBox:/home/karim# 
```
Résultat : Le disque /dev/sdb1 a été retiré du RAID, comme confirmé par le message hot removed /dev/sdb1 from /dev/md0.

Ajouter un nouveau disque au RAID
Pour remplacer le disque défaillant, j'ai ajouté la même partition au RAID après l'avoir "réparée" :

```bash
mdadm --manage /dev/md0 --add /dev/sdb1
```
Résultat : Le disque /dev/sdb1 a été ajouté au RAID, et la reconstruction a commencé automatiquement. Voici la sortie de cat /proc/mdstat montrant l'état de la reconstruction :  

```bash
root@karim-VirtualBox:/home/karim# cat /proc/mdstat
Personalities : [raid1]
md0 : active raid1 sdb1[2] sdc1[1]
      5236736 blocks super 1.2 [2/1] [_U]
      [========>............]  recovery = 44.5% (2335552/5236736) finish=0.5min speed=83412K/sec

unused devices: <none>
root@karim-VirtualBox:/home/karim# 
```

Vérification de la reconstruction
La reconstruction du RAID est en cours avec un taux d'avancement de 44,5 %. Une fois la reconstruction terminée, le RAID sera de nouveau en état "clean".  

Capture d'écran des commandes et sorties de commande :

![C12_failandrepair](https://github.com/user-attachments/assets/f7ad8c1f-0244-4121-acfc-3c86d4b5c1dd)   


**Vérification finale** : La reconstruction du RAID est terminée, et le RAID est de nouveau en état "clean", avec les deux disques actifs et synchronisés ([UU]).

### Création du RAID 5

#### Démontage et arrêt du RAID 1

Pour libérer les partitions utilisées dans le RAID 1 et les utiliser pour créer un RAID 5, j'ai d'abord démonté le système de fichiers monté sur `/mnt/raid1` :

```bash
umount /mnt/raid1
```
Ensuite, j'ai arrêté le RAID 1 existant avec la commande suivante : 

```bash
mdadm --stop /dev/md0
```

**Création du RAID 5** : 

J'ai ensuite créé un RAID 5 en utilisant les trois partitions (/dev/sdb1, /dev/sdc1, et /dev/sdd1) avec la commande suivante : 

```bash
mdadm --create /dev/md0 --level=5 --raid-devices=3 /dev/sdb1 /dev/sdc1 /dev/sdd1
```
Capture d'écran des commandes et sorties de commande :  

![C15_creerraid5](https://github.com/user-attachments/assets/e144cdd1-1aba-4c80-87a5-711d168ec881)

### Formatage et montage du RAID 5

#### Formatage du RAID 5

J'ai formaté le périphérique RAID 5 `/dev/md0` avec le système de fichiers `ext4` et lui ai attribué le label "RAID5_Data" :

```bash
mkfs.ext4 /dev/md0 -L "RAID5_Data"
```
**Création d'un point de montage**

J'ai créé un répertoire où monter le RAID 5 :

```bash
mkdir /mnt/raid5
```

#### Montage du RAID 5

Le RAID 5 a été monté sur le répertoire /mnt/raid5 avec la commande suivante :  

```bash
mount /dev/md0 /mnt/raid5
```

**Vérification du montage**

Pour vérifier que le RAID 5 a été monté correctement, j'ai utilisé la commande `df -h`.   

**Capture d'écran de toute la manipulation** :   

![C15_end](https://github.com/user-attachments/assets/3b743a9e-a370-4ea8-9653-cb7442f24d6e)   

**Vérification état du RAID5** : 
# Vérification de l'état du RAID 5

```bash
mdadm --detail /dev/md0
```
![C16_verifraid5](https://github.com/user-attachments/assets/5fdb115c-36c3-4034-923f-b039deb306d3)


**Voici un historique amélioré des commandes utilisées pour le challenge** : 

```bash
# Vérification des disques disponibles
lsblk

# Création du RAID 1 avec deux disques
mdadm --create --verbose /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc

# Vérification de l'état du RAID 1
cat /proc/mdstat
mdadm --detail /dev/md0

# Création du fichier de configuration mdadm
mdadm --detail --scan | tee -a /etc/mdadm/mdadm.conf
update-initramfs -u

# Simulation de panne d'un disque
mdadm /dev/md0 --fail /dev/sdb

# Vérification de l'état après la simulation de panne
cat /proc/mdstat
mdadm --detail /dev/md0

# Retrait du disque défaillant
mdadm /dev/md0 --remove /dev/sdb

# Ajout d'un nouveau disque pour la reconstruction du RAID
mdadm /dev/md0 --add /dev/sdd

# Vérification de l'état pendant la reconstruction
cat /proc/mdstat
mdadm --detail /dev/md0

# Suppression du RAID 1 pour préparer la création du RAID 5
mdadm --stop /dev/md0
mdadm --remove /dev/md0

# Création du RAID 5 avec trois disques
mdadm --create --verbose /dev/md0 --level=5 --raid-devices=3 /dev/sdb /dev/sdc /dev/sdd

# Vérification de l'état du RAID 5
cat /proc/mdstat
mdadm --detail /dev/md0

# Création du fichier de configuration mdadm pour le RAID 5
mdadm --detail --scan | tee -a /etc/mdadm/mdadm.conf
update-initramfs -u

# Historique des commandes
history

```
![image](https://github.com/user-attachments/assets/bddb5cce-1cb3-4db8-b332-471118a377cc)


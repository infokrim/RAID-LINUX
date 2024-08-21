# RAID-LINUX
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




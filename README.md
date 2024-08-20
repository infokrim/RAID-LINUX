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

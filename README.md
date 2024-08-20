# RAID-LINUX
# Solution : Configuration RAID sous Linux

## Étape 1 : Installation d'Ubuntu sur VirtualBox

J'ai créé une machine virtuelle avec un disque dur de 20 Go pour l'installation d'Ubuntu. L'installation de l'OS s'est déroulée avec succès.

## Étape 2 : Ajout des disques supplémentaires

J'ai ajouté trois disques supplémentaires de 5 Go chacun pour les manipulations sur le RAID. L'option "Branchable à chaud" ou "Hot-Pluggable"(version anglaise de virtualbox) a été activée pour permettre la déconnexion à chaud des disques.

### Capture d'écran de la configuration des disques dans VirtualBox

![Configuration des disques dans VirtualBox](chemin/vers/ta/capture.png)

*Note*: Les disques sont correctement configurés pour être branchables à chaud, ce qui est essentiel pour la simulation de pannes lors de la configuration du RAID.

## Étape 3 : Création du RAID 1

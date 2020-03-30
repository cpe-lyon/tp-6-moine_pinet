# TP6 -  Gestion des disques, Boot, Gestion des logs

## Exercice 1. Disques et partitions

2.	Vérifiez que ce nouveau disque dur est bien détecté par le système

Commandes :

* *ll /dev/sd* => liste les disques durs et les partitions
* *lsblk* => liste les entités en mode block (disques + partitions)
* *fdisk -l* => affiche plus d'information sur les partitions et disques que précedemment (ex: type de formatage)

3.	Partitionnez ce disque en utilisant fdisk : créez une première partition de 2 Go de type Linux (n°83), et une seconde partition de 3 Go en NTFS (n°7)

Commandes :

Créer une partition :

* *sudo fdisk /dev/sdb*
* *n*
* *p*
* choisir le numéro de la partition
* choisir taile en sélectionnant début et fin de la partition

Affecter système fichier :

* *sudo fdisk /dev/sdb*
* *t*
* numéro de partition
* numéro hexadécial du type de format à mettre sur la partition (ex: 7:NTFS ; 83: Linux)


4.	A ce stade, les partitions ont été créées, mais elles n’ont pas été formatées avec leur système de fichiers. A l’aide de la commande mkfs, formatez vos deux partitions 

Commandes :

Formater partition :

* *mkfs -t ext3 /dev/sdb1* => (sdb1 ; sdb2 = partitions) & -t = type


5.	Pourquoi la commande df -T, qui affiche le type de système de fichier des partitions, ne fonctionne-telle pas sur notre disque?

df -T => montre les disques montés avec de la place disponible

6.	Faites en sorte que les deux partitions créées soient montées automatiquement au démarrage de la machine, respectivement dans les points de montage /data et /win (vous pourrez vous passer des UUID en raison de l’impossibilité d’effectuer des copier-coller)

Commandes :

* *sudo mkdir /data /win*
* *sudo nano /etc/fstab*
* */dev/sdb1 / ext4 defaults 0 0*
* **/dev/sdb2 / ntfs defaults 0 0*

7.	Utilisez la commande mount puis redémarrez votre VM pour valider la configuration
8.	Montez votre clé USB dans la VM

9.	Créez un dossier partagé entre votre VM et votre système hôte (rem. il peut être nécessaire d’installer les Additions invité de VirtualBox


## Exercice 2

GRUB est considérablement paramétrable : résolution, langue, fond d’écran, thème, disposition du clavier….
GRUB se configure via un fichier de paramètres **(/etc/default/grub)**, mais aussi par des scripts situés dans **/etc/grub.d**; ces scripts commencent tous par un numéro et sont traités dans l’ordre.
Evidemment, seuls les scripts exécutables sont pris en compte.
Sous Ubuntu Server, GRUB prend aussi en compte les fichiers d’extension **.cfg** présents dans **/etc/default/grub.d**. En particulier, sur les versions récentes, le fichier de configuration **50-curtin-settings.cfg** donne à la variable **GRUB_TERMINAL** la valeur console, ce qui désactive tous les paramètres liés aux fonds d’écran, thèmes, certaines résolutions, etc.

1.	Commencez par changer l’extension du fichier **/etc/default/grub.d/50-curtin-settings.cfg** s’il est présent dans votre environnement (vous pouvez aussi commenter son contenu).

2.	Modifiez le fichier **/etc/default/grub** pour que le menu de GRUB s’affiche pendant 10 secondes; passé ce délai, le premier OS du menu doit être lancé automatiquement.

Commandes :

* commenter ligne *GRUB_TIMEOUT_STYLE=hidden*
* écrire ligne *GRUB_TIMEOUT=10*

3.	Lancez la commande *update-grub*

Cette commande fait appel au script grub-mkconfig qui construit le fichier de configuration ”final” de GRUB (/boot/grub/grub.cfg) à partir du fichier de paramètres et des scripts.

4.	Redémarrez votre VM pour valider que les changements ont bien été pris en compte

Pensez à lancer la commande *update-grub* après chaque modification de la configuration de
GRUB!

5.	On va augmenter la résolution de GRUB et de notre VM. Cherchez sur Internet le ou les paramètres à rajouter au fichier grub.

Ecriture :

* GRUB_GFXPAYLOAD=1024X768

6.	On va à présent ajouter un fond d’écran. Il existe un paquet en proposant quelques uns : *grub2-splash-images* (après installation, celles-ci sont disponibles dans /usr/share/images/grub).

Ecriture :

* GRUB_BACKGROUND="/usr/share/images/grub/Plasma-lamp.tga"

7.	Il est également possible de configurer des thèmes. On en trouve quelques uns dans les dépôts (grub2-themes-*). Installez-en un.
8.	Ajoutez une entrée permettant d’arrêter la machine, et une autre permettant de la redémarrer.

Commandes :

    menuentry 'Arrêt du système' {
      halt
    }
    menuentry 'Redémarrage du système' {
      reboot
    }

9.	Configurer GRUB pour que le clavier soit en français

Ecriture :

* GRUB_TERMINAL_INPUT=at_keyboard


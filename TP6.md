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
* */dev/sdb2 / ntfs defaults 0 0*

7.	Utilisez la commande mount puis redémarrez votre VM pour valider la configuration
8.	Montez votre clé USB dans la VM

??

9.	Créez un dossier partagé entre votre VM et votre système hôte (rem. il peut être nécessaire d’installer les Additions invité de VirtualBox

??


## Exercice 2 Personnalisation de GRUB

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

## Exercice 3 Noyau


Dans cet exercice, on va créer et installer un module pour le noyau.

1.	Commencez par installer le paquet build-essential, qui contient tous les outils nécessaires (compilateurs, bibliothèques) à la compilation de programmes en C (entre autres).

2.	Créez un fichier hello.c contenant le code suivant :

        1	#include <linux/module.h>
        2	#include <linux/kernel.h>
        3
        4	MODULE_LICENSE("GPL");
        5	MODULE_AUTHOR("John Doe");
        6	MODULE_DESCRIPTION("Module hello world");
        7	MODULE_VERSION("Version 1.00");
        8
        9	int init_module(void)
        10	{
        11	printk(KERN_INFO "[Hello world] - La fonction init_module() est appelée.\n");
        12	return 0;
        13	}
        14
        15	void cleanup_module(void)
        16	{
        17	printk(KERN_INFO "[Hello world] - La fonction cleanup_module() est appelée.\n"); 18 }

3. Créez également un fichier Makefile :

        1 obj-m += hello.o
        2
        3	all:
        4	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules
        5
        6	clean:
        7	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) clean
        8
        9	install:
        10	cp ./hello.ko /lib/modules/$(shell uname -r)/kernel/drivers/misc

Les lignes 4, 7 et 10 doivent commencer par une tabulation.

4.	Compilez le module à l’aide de la commande *make*, puis installez-le à l’aide de la commande *make install*.

Le module est installé dans le dossier spécifié à la ligne 10.

5.	Chargez le module; vérifiez dans le journal du noyau que le message ”La fonction init_module() est appelée” a bien été inscrit, synonyme que le module a été chargé; confirmez avec la commande *lsmod*.

Commandes :

* *sudo modprobe -a hello*

6.	Utilisez la commande *modinfo* pour obtenir des informations sur le module hello.ko; vous devriez notamment voir les informations figurant dans le fichier C.

7.	Déchargez le module; vérifiez dans le journal du noyau que le message ”La fonction cleanup_module() est appelée” a bien été inscrit, synonyme que le module a été déchargé (commende : *dmsg*); confirmez avec la commande *lsmod*.


Commandes :

* *sudo modprobe -r hello*

8.	Pour que le module soit chargé automatiquement au démarrage du système, il faut l’inscrire dans le fichier */etc/modules*. Essayez, et vérifiez avec la commande *lsmod* après redémarrage de la machine.

??

## Exercice 4. Exécution de commandes en différé : at et cron

1.	Programmez une tâche qui affiche un rappel pour la réunion qui aura lieu dans 3 minutes. Vérifiez entre temps que la tâche est bien programmée.

Commandes : 

* *crontab -l* => voir la crontab
* *crontab -e* => modifie crontab
* * */3 * * * * echo "BB Yoda meeting" >> ~/yoyo* => toutes les 3 minutes écrit BB Yoda meeting dans yoyo (echo seul s'éxécute dans un shell different de celui afficher donc impossible à voir)

**EXTRAIT OPENCLASSROOM** :

# m h  dom mon dow   command

Comme cette ligne est précédée d'un #, il s'agit d'un commentaire (qui sera donc ignoré).
Cette ligne vous donne quelques indications sur la syntaxe du fichier :

    m : minutes (0 - 59) ;

    h : heures (0 - 23) ;

    dom (day of month) : jour du mois (1 - 31) ;

    mon (month) : mois (1 - 12) ;

    dow (day of week) : jour de la semaine (0 - 6, 0 étant le dimanche) ;

    command : c'est la commande à exécuter.

++ pour faire toutes les 3 minutes on note */3 dans le champ m


2.	Est-ce que le message s’est affiché? Si la réponse est non, essayez de trouver la cause du problème (par exemple en vous aidant des logs, du manuel...)

3.	Pour tester le fonctionnement de cron, commencez par programmer l’exécution d’une tâche simple, l’affichage de “Il faut réviser pour l’examen!”, toutes les 3 minutes.

4.	Programmez l’exécution d’une commande tous les jours, toute l’année, tous les quarts d’heure

5.	Programmez l’exécution d’une commande toutes les cinq minutes à partir de 2 (2, 7, 12, etc.) à 18 heures les 1er et 15 du mois :

6.	Programmez l’exécution d’une commande du lundi au vendredi à 17 heures

7.	Modifiez votre crontab pour que les messages ne soient plus envoyés par mail, mais redirigés dans un fichier de log situé dans votre dossier personnel

8.	Videz votre crontab



# Installation de Windows 11 sous Proxmox VE


Proxmox VE : comment créer une machine virtuelle Windows 11 ?

Sommaire [-]

    I. Présentation
    II. Prérequis
    III. Créer une VM Windows 11 sur Proxmox
    IV. Installer Windows 11 avec les pilotes VirtIO
    V. Conclusion

## I. Prérequis

D'un côté, il y a Proxmox VE, une solution basée sur l'utilisation de la technologie QEMU/KVM pour la virtualisation et l'exécution des machines virtuelles. D'un autre côté, il y a Windows 11, un système d'exploitation qui nécessite une attention particulière, car il a des prérequis spécifiques : UEFI, module TPM 2.0, 64 Go de stockage et j'en passe. Cela signifie que nous devons effectuer une configuration bien spécifique de la VM.

Au niveau des ressources à disposer, vous aurez besoin du fichier ISO d'installation de Windows 11. Ce fichier n'est pas fourni par Proxmox VE et doit être obtenu depuis le site de Microsoft, via ce lien :

-  https://www.microsoft.com/fr-fr/software-download/windows11

L'image ISO doit être chargée dans la bibliothèque d'image ISO de votre serveur Proxmox.

  1 - Cliquez sur le stockage local de votre serveur, puis choisissez "ISO Images" dans le sous-menu.

  2 - Cliquez sur le bouton "Upload" pour charger votre image ISO.

  3 - Sélectionnez l'image ISO de Windows 11, ici, ce sera une image ISO de Windows 11 24H2.

  4 - Lancez le téléchargement du fichier en cliquant sur le bouton "Upload" et patientez un instant.

Vous aurez également besoin des pilotes VirtIO. En effet, Proxmox VE propose des périphériques paravirtualisés VirtIO qui offrent les meilleures performances avec une faible surcharge CPU. Pour Windows, ces pilotes doivent être installés manuellement dans le système d'exploitation invité pendant (et après) l'installation.

Le package des pilotes VirtIO pour Windows est disponible sous forme d'images ISO hébergées sur le site de Fedora. Cette page référence les liens et les informations utiles au sujet des pilotes VirtIO pour Windows. On y trouve notamment deux liens pour récupérer les images ISO contenant les pilotes :

-  https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/stable-virtio/virtio-win.iso
-  https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/latest-virtio/virtio-win.iso

Ici, nous prendrons la version stable (mais les deux fonctionnent) pour des raisons de stabilité. Toujours à partir de votre PVE, lancez le téléchargement de l'image ISO des pilotes VirtIO :

  1 - Cliquez sur le stockage local de votre serveur, puis choisissez "ISO Images" dans le sous-menu.

  2 - Cliquez sur le bouton "Download from URL" pour charger votre image depuis Internet.

  3 - Spécifiez l'URL vers l'image ISO : vous pouvez copier le lien depuis la liste ci-dessus, car il pointe directement vers le fichier ISO. Spécifiez aussi un nom pour ce fichier.

  4 - Lancez le téléchargement du fichier en cliquant sur le bouton "Download" et patientez un instant.

Vous disposez désormais des deux images ISO nécessaires à l'installation de Windows 11.

Nous venons d'évoquer un terme qui est peut-être nouveau pour vous : paravirtualisation. À ne pas confondre avec la virtualisation, car il y a des subtilités et Proxmox VE est capable de "mixer" les deux. En bref :

    -  Virtualisation : l'hyperviseur émule entièrement le matériel, l'OS invité ne sait pas qu'il tourne dans une machine virtuelle.
    -  Paravirtualisation : l'OS invité est conscient qu'il est virtualisé et utilise des pilotes spéciaux (comme ici les pilotes VirtIO) pour communiquer plus efficacement avec l'hyperviseur.

## III. Créer une VM Windows 11 sur Proxmox

Nous allons maintenant passer à la création de la machine virtuelle Windows 11 sur Proxmox VE. Cliquez sur le bouton "Create VM" située en haut à droite de l'interface. Commencez par indiquer un nom pour cette VM et éventuellement un ID personnalisé (facultatif).

Étape OS

Première étape essentielle pour la suite. J'attire votre attention vers trois points :

  1 - Choisissez l'image ISO d'installation de Windows 11, elle sera associée au lecteur CD/DVD virtuel de la VM.

  2 - Sélectionnez le type de système d'exploitation invité correspondant à Windows 11, à savoir le type "Microsoft Windows" puis la version "11/2022/2025". Ce n'est pas une date, c'est bien une référence à Windows 11 et Windows Server 2022 ou 2025.

  3 - Cochez l'option "Add additional drive for VirtIO drivers" et sélectionnez l'image ISO avec les pilotes VirtIO. Ceci va connecter un second lecteur CD/DVD à la VM pour vous donner accès facilement au pilote.

Étape System

Cette étape mérite aussi une attention particulière afin de créer une VM respectueuse des prérequis de Windows 11.

  1 - Le type de BIOS "OVMF (UEFI)" est, en principe, automatiquement sélectionné. Vous devez simplement choisir l'emplacement pour le disque EFI.

  2 - La puce TPM est aussi activée par défaut comme nous avons sélectionné Windows 11 à l'étape précédente. Vous devez vous assurer que ce soit le cas, choisir un emplacement de stockage et aussi vérifier que c'est bien une puce TPM 2.0 qui est choisie.

  3 - Cochez l'option "Qemu Agent". L'agent QEMU pour Windows est un petit logiciel installé dans la VM qui permet à Proxmox d'interagir avec le système invité, par exemple, pour effectuer un arrêt propre.

Étape Disks

Vous devez spécifier la taille du disque dur principal de la machine virtuelle. Il est possible d'utiliser un disque SCSI, via le contrôleur VirtIO SCSI. Sachez que nous devrons installer un pilote lors de l'installation de l'OS. Ici, indiquez à minima 64 Go, car c'est un prérequis de Windows 11.

Même si ce n'est pas visible sur l'image ci-dessous, vous devez cocher l'option "Discard" pour une meilleure gestion du provisionnement dynamique sur les disques SSD.

Étape CPU

Il est temps de configurer le processeur de la machine virtuelle, ce qui est l'occasion de déterminer le nombre de cœurs et le type de processeur. Vous devez :

  1 - Attribuer au moins 2 cœurs à la machine virtuelle.

  2 - Sélectionner le type de processeur "Host". En effet, le CPU de type Host pour une VM Windows 11 permet d'exposer toutes les fonctionnalités du processeur physique, ce qui garantit la compatibilité avec les exigences matérielles de Windows 11 (et certaines applications). En somme, c'est bénéfique pour la stabilité de la VM.

Étape Memory

Attribuez au moins 4 Go de RAM (mémoire vive) à cette machine virtuelle. Là encore, c'est un prérequis de Windows 11.

Étape Network

Connectez la VM sur un réseau, comme ici le réseau vmbr0 (créé par défaut). Concernant le modèle de carte réseau, vous pouvez choisir "VirtIO (paravirtualized)" ou un autre modèle, mais ce premier choix implique d'installer un pilote supplémentaire.

Étape Confirm

Vérifiez votre configuration et cliquez sur le bouton "Finish" pour finaliser la création de la VM.

Patientez un instant. La machine virtuelle devrait apparaître dans votre inventaire PVE.
IV. Installer Windows 11 avec les pilotes VirtIO

La suite des opérations consiste à installer Windows 11 sur la machine virtuelle. Démarrez la VM et via la console, appuyez sur une touche pour démarrer sur l'image ISO. L'assistant d'installation de Windows 11 (habituel) va s'afficher à l'écran... Laissez-vous guider...

Nous allons uniquement passer en revue les étapes où il y a des blocages. Par exemple, au moment de sélectionner le disque sur lequel installer Windows 11, vous risquez d'avoir une mauvaise surprise : une liste vide ! Pas de panique, c'est parce que le pilote du contrôleur VirtIO n'est pas intégré à l'image ISO de Windows 11, mais nous avons prévu le coup... Cliquez sur le bouton "Load Driver".

Cliquez sur le bouton "Parcourir" puis dans le lecteur DVD correspondant à l'image ISO VirtIO, sélectionnez le dossier D:\vioscsi\w11\adm64. Validez. Un pilote nommé Red Hat VirtIO SCSI pass-throuth controller devrait s'afficher. Sélectionnez-le et cliquez sur le bouton "Installer".

Ah, c'est mieux ! Le disque apparaît désormais ! Poursuivez l'installation...

L'installation va débuter et poursuivre... Vous allez enchainer les étapes et vous retrouver bloqué sur l'étape de connexion au réseau ! La carte réseau ne sera pas reconnue, là encore à cause d'un problème de pilote. Cliquez sur le bouton "Installer le pilote", sélectionnez le lecteur VirtIO et validez (Windows va rechercher le pilote tout seul).

Quelques secondes plus tard, la machine virtuelle sera connectée au réseau. Parfois, un code d'erreur s'affiche, mais il n'est pas gênant. Poursuivez et finalisez l'installation. La suite va se dérouler sans encombre.

Si vous jetez un œil à la liste des périphériques, vous constaterez la présence des périphériques VirtIO. C'est aussi l'occasion de voir si des pilotes sont manquants.

Pour finir l'installation, je vous recommande d'accéder au lecteur DVD correspondant à VirtIO Win pour lancer l'installation de deux paquets : virtio-win-gt-x64 (tous les pilotes) et virtio-win-guest-tools (l'agent Qemu et les outils invités d'une façon générale).

Voilà, votre machine virtuelle Windows 11 est prête !

## V. Conclusion

Ce tutoriel vous a présenté l'ensemble des étapes nécessaires pour déployer une machine virtuelle Windows 11 sur Proxmox VE, en respectant les prérequis de cet OS. C'était aussi l'occasion de découvrir le processeur en mode host et l'intégration des pilotes VirtIO.

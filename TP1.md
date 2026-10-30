TP1 Installation de VirtualBox et création d'une VM Ubuntu

Membres:
MAGASSOUBA CHEICK AHMED
KOUKO JOSHUA EZECHIEL
N'DRI N'DOUA DENIS MARC- ARTHUR
*Dans les sorties, l'utilisateur des VM est `josh` (au lieu de `etudiant` dans l'énoncé).*



**PARTIR1	:Préparer l'hôte et installer VirtualBox 

Questions1:Qu'est-ce qu'un hyperviseur de type 1 et de type 2 ? Dans quelle catégorie se range VirtualBox ? Citez un hyperviseur de type 1 utilisé par un fournisseur cloud.**



**réponses:**

Un hyperviseur de type 1:fonctionne directement sur le matériel physique.

Un hyperviseur de type 2:fonctionne au-dessus d'un système d'exploitation.



VirtualBox est un hyperviseur de type 2.

un hyperviseur de type 1 utilisé par un fournisseur cloud:VMware ESXi.





**Question 2 : Pourquoi la virtualisation matérielle est-elle indispensable pour faire tourner une VM 64 bits ?**


**réponses:**

Elle permet au processeur d'exécuter efficacement les instructions des machines virtuelles grâce aux extensions matérielles Intel VT-x ou AMD-V.





**Question 3 :Quelle version de VirtualBox avez-vous installée ? Que renvoie VBoxManage list hostinfo sur le nombre de cœurs et la RAM de l'hôte ?**


**réponses:**

VirtualBox 7.2.20.

Host Information:



Host time: 2026-10-30T07:52:14.367000000Z

Processor online count: 4

Processor count: 4

Processor online core count: 2

Processor core count: 2

Memory size: 20284 MByte

Memory available: 7745 MByte

Operating system: Windows 11

Operating system version: 10.0.26300.9711



**PARTIE2:Télécharger et vérifier l'image ISO Ubuntu 

Question1:Les deux empreintes sont-elles identiques ? Pourquoi faut-il vérifier l'empreinte d'une image avant de l'installer ?

reponses:**
Oui, les deux empreintes SHA256 sont strictement identiques :

Empreinte officielle : cc8a95cde20f6ced61a322420de00f10cc3c90ced545daa46cb9c1a117f1d927

Empreinte obtenue : CC8A95CDE20F6CED61A322420DE00F10CC3C90CED545DAA46CB9C1A117F1D927

La vérification de l'empreinte permet de garantir l'intégrité du fichier téléchargé et de détecter toute corruption ou modification accidentelle ou malveillante avant son installation.

**Question2:Quelle différence entre Ubuntu Desktop et Ubuntu Server ? Pourquoi choisit on Server pour un serveur cloud ?

reponses:**



Ubuntu Desktop: est destiné principalement aux utilisateurs d'ordinateurs personnels. Il possède une interface graphique permettant d'utiliser facilement les applications.



Ubuntu Server: est conçu pour héberger des services et des applications sur des serveurs. Il fonctionne principalement en ligne de commande et consomme généralement moins de ressources.

On choisit Ubuntu Server pour les serveurs cloud parce qu'il est adapté à l'administration à distance, consomme moins de ressources et permet d'héberger des services sans nécessiter une interface graphique.

**Question3:Que signifie LTS ? Combien d'années de mises à jour de sécurité standard une version LTS reçoit-elle ?**



**reponses:**

LTS signifie Long Term Support, c'est-à-dire « support à long terme ».



Une version Ubuntu LTS bénéficie de 5 années de mises à jour de sécurité standard.



Cela permet aux entreprises et aux administrateurs de serveurs de disposer d'un système stable et maintenu pendant plusieurs années.


**PARTIE3:Création et configuration de la machine virtuelle**

**### Configuration réalisée**



**| Paramètre                   | Configuration    |**

**| --------------------------- | ---------------- |**

**| Nom de la machine virtuelle | ubuntu           |**

**| Système d'exploitation      | Linux            |**

**| Version                     | Ubuntu (64 bits) |**

**| Mémoire vive                | 4096 Mo          |**

**| Processeur virtuel          | 1                |**

**| Disque dur                  | VDI              |**

**| Capacité du disque          | 25 Go            |**

**| Allocation                  | Dynamique        |**

**| Carte réseau                | NAT              |**

**| Protocole SSH               | TCP              |**

**| Adresse IP hôte             | 127.0.0.1        |**

**| Port hôte                   | 2222             |**

**| Port invité                 | 22               |**



&#x20;**Question 1 : Quelle différence entre un disque alloué dynamiquement et un disque de taille fixe ? Quelle place occupe le fichier .vdi juste après sa création ?**



**reponses:**

Un disque alloué dynamiquement est un disque virtuel dont le fichier augmente progressivement en fonction de l'espace réellement utilisé par la machine virtuelle, jusqu'à atteindre sa capacité maximale.



Un disque de taille fixe réserve immédiatement tout l'espace défini lors de sa création.



Juste après sa création, un disque dynamique occupe beaucoup moins que sa capacité maximale de 25 Go, car il ne réserve pas immédiatement tout cet espace sur le disque physique.



&#x20;**Question 2 : Pourquoi désactive-t-on l'installation sans surveillance pour ce TP ?**


**reponses:**

On désactive l'installation sans surveillance afin de réaliser manuellement les différentes étapes de l'installation d'Ubuntu Server.



Cela permet de comprendre les choix de configuration, notamment la langue, le clavier, le réseau, le stockage, la création des utilisateurs et l'installation du serveur OpenSSH.



**Question 3 : En mode NAT, pourquoi l'hôte ne peut-il pas joindre directement la VM ? À quoi sert la redirection 2222 → 22 ?**



**reponses:**

En mode NAT, la machine virtuelle se trouve derrière un mécanisme de traduction d'adresses réseau. Elle peut accéder à Internet, mais elle n'est pas directement accessible depuis la machine hôte par son adresse IP privée.



La redirection de port permet de résoudre ce problème.



Le port 2222 de la machine hôte est redirigé vers le port 22 de la machine virtuelle, qui correspond au service SSH.



Ainsi, la connexion s'effectue avec la commande :



`ssh -p 2222 josh@127.0.0.1`



\---



**PARTIE 4 : INSTALLATION D'UBUNTU SERVER**



**### Configuration et installation réalisées**



**\* Système installé : Ubuntu Server 26.04.1 LTS.**

**\* Nom d'hôte : ubuntu-srv.**

**\* Nom d'utilisateur : josh.**

**\* Interface réseau : enp0s3.**

**\* Adresse IPv4 obtenue : 10.0.2.15.**

**\* Serveur OpenSSH : installé et démarré.**



**Question 1 : Combien de temps a duré l'installation ? Comparez avec la création d'une instance cloud à partir d'une image prête.**



**reponses:**



L'installation a duré pratique 15 à 20min 



Contrairement à cette installation manuelle, la création d'une instance cloud à partir d'une image préconfigurée est généralement plus rapide, car le système d'exploitation est déjà installé.







&#x20;**Question 2 : Quelle adresse IP l'installateur a-t-il obtenue sur enp0s3 ? Qui l'a attribuée ?**



**reponses**



L'interface réseau enp0s3 a obtenu l'adresse IPv4 suivante :



`10.0.2.15`



Cette adresse a été attribuée automatiquement par le serveur DHCP du réseau NAT de VirtualBox.



Le DHCP permet d'attribuer automatiquement les paramètres réseau à la machine virtuelle.



&#x20;**Question 3 : Pourquoi installe-t-on le serveur OpenSSH dès l'installation ? Que se passerait-il sur un serveur cloud sans SSH ?**



**reponses:**



On installe le serveur OpenSSH afin de pouvoir administrer la machine virtuelle à distance depuis une autre machine.



SSH permet d'établir une connexion sécurisée à travers le réseau, d'exécuter des commandes et de gérer le serveur sans être physiquement devant celui-ci.



Sur un serveur cloud sans SSH, l'administration distante en ligne de commande serait plus difficile. Il faudrait disposer d'un autre moyen d'accès, comme une console distante fournie par l'hébergeur**.**



**---**



**# PARTIE 5 : PREMIERS PAS, RÉSEAU ET ACCÈS SSH**



**### Manipulations réalisées**



Plusieurs commandes ont été utilisées pour vérifier le fonctionnement d'Ubuntu :



\* `lsb\_release -a` : identifier la distribution Ubuntu.

\* `uname -r` : afficher la version du noyau Linux.

\* `hostnamectl` : afficher les informations sur la machine.

\* `nproc` : connaître le nombre de processeurs disponibles.

\* `free -h` : consulter l'utilisation de la mémoire.

\* `df -h /` : vérifier l'espace disque.

\* `lsblk` : afficher les périphériques de stockage.



La connexion SSH a ensuite été testée depuis Windows avec :



`ssh -p 2222 josh@127.0.0.1`



Le transfert de fichiers a été effectué avec SCP :



`scp -P 2222 test.txt josh@127.0.0.1:/home/josh/`



Le fichier a été retrouvé dans Ubuntu à l'aide de la commande :



`ls -l`



Le transfert a réussi.



&#x20;**Question 1 : Relevez l'adresse IP, le masque, la passerelle et le serveur DNS de la VM. À quoi correspond la passerelle 10.0.2.2 ?**





**reponses:**

L'adresse IPv4 relevée sur la machine virtuelle est :



`10.0.2.15`



Dans la configuration NAT standard de VirtualBox, la passerelle est généralement :



`10.0.2.2`



Elle correspond à l'interface de passerelle du réseau NAT de VirtualBox. Elle permet à la machine virtuelle de communiquer avec les réseaux extérieurs.



Le masque et le serveur DNS doivent être relevés directement dans la configuration réseau de la VM.



&#x20;**Question 2 : Pourquoi se connecte-t-on à 127.0.0.1:2222 et non directement à 10.0.2.15:22 ?**



**reponses:**

On utilise l'adresse 127.0.0.1 avec le port 2222 parce que la machine virtuelle fonctionne en mode NAT.



L'adresse 127.0.0.1 désigne la machine hôte elle-même. Le port 2222 est redirigé vers le port 22 de la machine virtuelle.



Cette redirection permet à Windows d'accéder au serveur SSH d'Ubuntu sans devoir rendre directement accessible son adresse IP privée.



&#x20;**Question 3 : Que contient le message affiché à la première connexion SSH (empreinte) ? Quel risque permet-il d'éviter ?**



**reponses:**

Lors de la première connexion SSH, un message affiche l'empreinte numérique de la clé publique du serveur distant.



Cette empreinte permet à l'utilisateur de vérifier l'identité du serveur auquel il souhaite se connecter.



Elle contribue à éviter les attaques de type « homme du milieu » (Man-in-the-Middle), dans lesquelles un attaquant pourrait se faire passer pour le serveur légitime.



**Question 4 : Quels avantages offre l'authentification par clé par rapport au mot de passe ? Pourquoi les fournisseurs cloud l'imposent-ils par défaut ?**



**reponses:**

L'authentification par clé SSH offre plusieurs avantages :



\* Une meilleure sécurité.

\* Une authentification sans saisie systématique du mot de passe.

\* Une meilleure résistance aux attaques par force brute.

\* Une gestion plus pratique des accès aux serveurs.



Les fournisseurs cloud privilégient cette méthode parce qu'elle réduit les risques liés aux mots de passe faibles ou compromis et renforce la sécurité des connexions à distance.







**PARTIE 6 : INSTANTANÉ, CLONE ET ARRÊT**



**Manipulations réalisées**



Un instantané de la machine virtuelle a été créé sous le nom :



`snapshot\_initial`



Cet instantané permet de conserver un état précis de la machine virtuelle et de pouvoir y revenir en cas de problème.



Un clone de la machine virtuelle a également été créé sous le nom :



`ubuntu-clone`



Le clonage permet d'obtenir une seconde machine virtuelle à partir de la machine originale.



&#x20;**Question 1 : Quelle différence entre un instantané et un clone ? Un instantané est-il une sauvegarde ? Justifiez.**



**reponses:**



Un instantané (snapshot) permet d'enregistrer l'état d'une machine virtuelle à un moment donné afin de pouvoir restaurer cet état ultérieurement.



Un clone est une copie d'une machine virtuelle qui peut fonctionner indépendamment de la machine originale lorsqu'il s'agit d'un clone complet.



Un instantané n'est pas une véritable sauvegarde indépendante, car il dépend généralement des fichiers de la machine virtuelle et de son stockage. Si le disque physique est perdu ou endommagé, l'instantané peut également être perdu.



Il est donc préférable de réaliser des sauvegardes externes en complément des instantanés.



**Question 2 : Quels sont les équivalents cloud de l'instantané et du clone ?**



**reponses**



Dans les environnements cloud, l'équivalent d'un instantané est le snapshot de disque. Il permet de conserver l'état d'un volume de stockage à un moment donné.



L'équivalent du clone est la création d'une nouvelle instance à partir d'une image de machine virtuelle.



Ces mécanismes sont notamment utilisés pour sauvegarder, restaurer et reproduire rapidement des environnements informatiques.



**Question 3 : Quelle différence entre acpipowerbutton et poweroff dans VBoxManage controlvm ?**



**reponses**

La commande `acpipowerbutton` envoie un signal d'arrêt ACPI à la machine virtuelle. Cela lui permet de s'éteindre proprement, comme lorsqu'on utilise le bouton d'arrêt d'un ordinateur.



La commande `poweroff` arrête immédiatement la machine virtuelle, comme si l'on coupait brutalement son alimentation électrique.



Il est préférable d'utiliser `acpipowerbutton` ou une commande d'arrêt propre depuis Ubuntu afin d'éviter les pertes de données.



**Question 4 : Pourquoi démarrer une VM en mode headless ? Quel lien avec une instance cloud ?**



**reponses:**

Le mode headless permet de démarrer une machine virtuelle sans afficher son interface graphique.



Il permet de réduire l'utilisation des ressources de l'ordinateur hôte et de gérer la machine à distance à l'aide de commandes.



Ce fonctionnement ressemble à celui d'une instance cloud, qui fonctionne généralement sans écran ni interface graphique et qui est administrée à distance.



**### Question 5 : Pourquoi faut-il changer le port hôte et le nom d'hôte du clone ?**



**reponses:**

Il faut changer le port hôte pour éviter un conflit entre la machine originale et le clone.



La machine originale utilise le port 2222, tandis que le clone doit utiliser le port 2223.



Il faut également changer le nom d'hôte afin de distinguer les deux machines sur le réseau et d'éviter les confusions lors de leur administration.



Ainsi :



\* Machine originale : ubuntu-srv, port 2222.

\* Machine clonée : ubuntu-clone, port 2223.






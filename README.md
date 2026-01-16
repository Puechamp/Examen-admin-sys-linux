# Examen-admin-sys-linux du 16 Janvier, Tanguy PUECHOULTRES


## Les deux familles

Famille Debian : 
* Debian
* Ubuntu
* Linux Mint
* Kali Linux

Famille RedHat : 
* Red Hat Enterprise Linux (RHEL)
* Fedora
* CentOS

Il existe d'autre familles comme Arch, Slackware, Gentoo ou SUSE.

La philosophie de la famille Debian est d'être assez universaliste et accessibles à tous en étant open-source, avec une large selection de paquet téléchargeable avec différent gestionnaire de paquet (comme dpkg ou apt), des mises à jour régulières assurant une grande stabilité et une documentation riche. Là où RedHat est plus orienté entreprise avec un gestionnaire de paquet propre à RedHat (sous la forme de dnf et rpm), des cycles de vie plus long avec une sécurité renforcée et un code open-source seulement à ceux qui achètent la licence (des exceptions totalement open-source existe et sont utilisé pour crée des distributions comme Bazzite qui est basé sur Fedora Atomic qui est open-source).


##Back to basics

La différence fondamentale entre le mode user et le mode Kernel est au niveau des autorisations et privilèges que le mode user n'a pas pas rapport au mode Kernel :
* inaccessibilité (ou alors accessibilité limité) à la mémoire du Kernel et au hardware
* incapacité à interférer avec les autres processus

Quand un programme en mode user (comme un navigateur web) crash, il crash tout seul, sans entraîner le système avec lui. À contrario, un programme en mode Kernel (comme un driver) qui crash **peut éventuellement** entraîner avec lui tout le système et le faire planter.


## Qui est où ?

* **/etc** => répertoire de fichiers de configuration système, contient les fichiers textes de configuration globale

* **/bin et /usr/bin** => répertoires de commandes exécutables comme ls, cp, mv, bash... /bin contient les commandes nécessaire pour le démarrage du système (comme init) et /usr/bin contient les commandes utilisateurs (comme ls et cp)

* **/sbin et /usr/sbin** => répertoires de commandes d'administration système comme mount, fsck, useradd... là encore /sbin contient les commandes administrative indispensables au démarrage du système et /usr/sbin contient les commandes d'administration non critique au boot.

* **/home** => répertoire personnel de l'utilisateur, les données, les configurations locales

* **/var** => répertoire de fichiers qui peuvent changer pendant l'éxecution du système comme les caches, les logs...

* **/var/log** => sous-répertoire de /var qui contient les journaux du Kernel, des services, des daemons

* **/var/lib** => sous-répertoire de /var qui contient les métadonnées de systèmes et applications

## Où est le pilote ?

* La liste des pilotes chargées en mémoire => **lsmod**, donne le nom de module, sa taille et le nombre d'utilisateurs qui s'en servent
* La liste des fichiers binaires (modules) qui les fournissent => **/lib/modules/$(uname -r)/**
* Forcer leur chargement et leur déchargement => pour charger un module : **sudo modprobe "nom_du_module"** pour décharger un module : **sudo modprobe -r "nom_du_module"**
* Les messages qu'ils ont émis => **sudo dmesg** pour les messages du noyau avec possibilité d'ajouter l'argument **-w** pour les avoir en temps réel, **journalctl -k** pour les journaux persistants


## MS Windows

# Examen-admin-sys-linux du 16 Janvier, Tanguy PUECHOULTRES


## Les deux familles

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


## Back to basics

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

* La liste des pilotes chargées en mémoire => ```lsmod```, donne le nom de module, sa taille et le nombre d'utilisateurs qui s'en servent
* La liste des fichiers binaires (modules) qui les fournissent => ```/lib/modules/$(uname -r)/```
* Forcer leur chargement et leur déchargement => pour charger un module : ```sudo modprobe "nom_du_module"``` pour décharger un module : ```sudo modprobe -r "nom_du_module"```
* Les messages qu'ils ont émis => ```sudo dmesg``` pour les messages du noyau avec possibilité d'ajouter l'argument ```-w``` pour les avoir en temps réel, ```journalctl -k``` pour les journaux persistants


## MS Windows

En pratique WSL permet de fournir un environnement GNU/Linux complet sur Windows sans avoir à utiliser un dual boot ou une machine virtuelle avec un vrai noyau Linux depuis WSL2. WSL permet de compiler et d'éxécuter du code Linux sur Windows (entre autre du script Bash) et de gérer des environnements serveurs Linux en passant par Windows. 


## On commence

* D'abord vérifier l'état de ssh avec ```systemctl status ssh```
* Créer un utilisateur qui servira de lien entre les 2 machines, on va l'appeler Bob ```adduser Bob``` => ```usermod -aG sudo Bob```
* Générer sur la machine cliente la clé publique ```ssh keygen -t ed25519```
* Copier la clé publique ```ssh-copy-id Bob@serveur```
* On peut renforcer la sécurité en éditant le fichier **/etc/ssh/sshd_config**
* Recharger SSH ```systemctl reload ssh```


## Alerte !

Déjà savoir si c'est normal ou non : est-ce que qu'on a réinstallé/migré le serveur ou changé la clé ? Auquel cas on le sait vu que c'est généralement celui qui a mit en place le SSH qui fait la manipulation.
Sinon ça peut être une attaque Man-In-The-Middle (MITM) ou une clé compromise.

**Que faire ?**
* D'abord vérifier l'identité du serveur ```ssh-keygen -l -f /etc/ssh/ssh_host_ed25519_key.pub```
* Si la clé est correcte et qu'on veut changer de clé ```ssh-keygen -R serveur.example.com``` puis se reconnecter ```ssh user@serveur.example.com```

Si on ne peut pas vérifier la clé, **ne pas se connecter**.

## Oh les gourmands !

```du -sh /home/* 2>/dev/null | sort -hr | head``` 

## On va aider les devs...

Sur une distrib de la famille Debian :
* mettre à jour le système si besoin ; ```sudo apt update && sudo apt upgrade```
* installer "build-essentials" ; ```sudo apt install build-essential``` qui contient gcc, g++, make entre autre...
* installer les outils de développements complémentaires ; ```sudo apt install git cmake gdb valgrind strace lsof htop tmux```
* installer les headers de Kernel (si on fait du dev de bas niveau) ; ```sudo apt install linux-headers-$(uname -r)```
* vérifier la config ; ```gcc --version make --version gdb --version```

Sur une distrib de la famille RedHat : 
* mettre à jour le système ; ```sudo dnf update```
* installer le groupe de développement ; ```sudo dnf groupinstall "Development Tools"``` qui contient gcc, g++, tout ça...
* installer les outis de dev complémentaires ; ```sudo dnf install git cmake gdb valgrind strace lsof htop tmux```
* installer les bibliothèques et headers (pour le dev de bas niveau) ; ```sudo dnf groupinstall "Development Libraries"```, ```sudo dnf install kernel-devel kernel-headers```
* vérifier la config ; ```gcc --version make --version gdb --version```


## Post Mortem

Avec ``` last -x reboot shutdown``` on peut voir si l'heure exacte du reboot et si c'était une extinction propre "shutdown" ou un plantage "crash"
Checker les journaux du boot précédent ```journalctl -b -1``` et si on veut uniquement les messages d'erreur critque ont peut utilsier ```journalctl -b -1 err..alert``` 


## On surveille...

* Savoir si le service est lancé au démarrage automatiquement : ```systemctl is-enabled bidule```
* Savoir s'il est en cours de fonctionnement : ```systemctl status bidule```
* Faire en sorte qu'il le soit : ```sudo systemctl enable bidule```
* Examiner les messages qu'il émet : ```journalctl -u bidule```
* Le redémarrer : ```sudo systemctl restart bidule```
* Déterminer de quel paquet (Debian ou Red Hat) il provient : sur Debian/Ubuntu ; ```dpkg -S $(systemctl show -p FragmentPath bidule | cut -d= -f2)```, sur RedHat/Fedora ; ```rpm -qf $(systemctl show -p FragmentPath bidule | cut -d= -f2)```


## Zut

Il faudra passer directement par le menu GRUB et modifier temporairement la ligne de boot pour modifier voir retirer le MdP root et ainsi reprendre le contrôle total du système


## C'est la faute à Rémy (sans famille)

* Vérifier si le service il tourne ; ```systemctl status apache2/nginx``` pour verifier si le service est démarré ou arrêté, les erreurs de lancement,le PID actif et les dernières lignes de logs
* Vérifier si le service écoute sur le bon port ; ```ss -ltnp | grep -E ':(80|443)'``` pour vérifier si les ports de 80 à 443 sont ouverts, les adresses d'écoute et le processus associé
* Verifier la configuration du service ; ```apachectl configtest``` pour apache, ```nginx -t``` pour nginx pour vérifier les erreurs de syntaxe, les fichiers inclus manquants et les certificats mal référencés
* Vérifier les logs ; ```journalctl -u apache2/nginx```
* Vérifier le pare-feu ; ```firewall-cmd --list-all```
* Vérifier la connectivité locale ; ```curl -v http://127.0.0.1``` et extérieure ; ```curl -v http://serveur```
* Vérifier le DNS ; ```dig serveur.example.com```
* Vérifier la validité des certificat TLS/HTTPS ; ```openssl s_client -connect serveur:443```
* Vérifier le CPU et la RAM ; ```top``` ou ```free -h``` et les disques ```df -h```
* Tracer les requêtes ; ```tcpdump -i any port 80 or port 443``` pour voir si les requêtes arrivent et/ou repartent


## Le Web

* Quels paquets installez-vous ? : Serveurs Web ```sudo apt install apache2```, PHP ```sudo apt install php libapache2-mod-php php-mysql php-cli```, base de données ```sudo apt install mariadb-server mariadb-client```, outils de monitoring et logs ```sudo apt install awstats goaccess```
* Comment organisez sous le stockage des différents contenus ? : les racines de sites seront dans le répertoire **/var/www/***, les fichiers de configuration Apache par site dont dans le répertoire **/etc/apache2/sites-available/** et on active les sites ```sudo a2ensite "nom_du_site"``` => ```sudo systemctl reload apache2```
* Configurer PHP : ```sudo a2enmod php``` => ```sudo systemctl restart apache2```
* Comment pouvez-vous organiser l'enregistrement séparé des visites des différents sites ? : Apache créé des logs par sites permettant d'analyser chaque site individuellement. Si on veut éviter que les logs grossissent indéfiniment et configurer une rotation des logs on entre la commande ```sudo apt install logrotate```
* Quel(s) outil(s) peuvent vous permettre de fournir une vison des visites de chaque site à chaque Webmaster ? : on peut utiliser AWStats ```sudo apt install awstats``` => ```sudo awstats_buildstaticpages.pl -config="nom_du_site" -update```, ou GoAccess ```goaccess /var/log/apache2/"nom_du_site".log -o /var/www/"nom_du_site"/report.html --log-format=COMBINED```

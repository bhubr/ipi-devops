# IMDW280/DevOps - Ansible

## Introduction

### DevOps

Le terme DevOps est issu de la contraction de _dev_ (_development_) et _ops_ (_operations_ = ce qui relève de l'administration système et réseau).

C'est un terme aux multiples facettes, qui peut désigner :

* Une philosophie, une approche, un mouvement, visant à faire évoluer la façon dont on développe et met en production des applications,
* Un métier (_ingénieur DevOps_)

Une définition assez commune est que le DevOps vise à décloisonner, dans une organisation, les aspects _dev_ et _ops_, et à faire en sorte que les protagonistes des deux côtés travaillent ensemble plus efficacement.

Traditionnellement, ces métiers obéissent en effet à des contraintes qu'on pourrait qualifier d'antagonistes :

* Mûs par des impératifs d'innovation et de compétitivité, les développeurs mettent à jour leurs application, pour bénéficier des nouvelles versions de leurs langages et frameworks. Ils adoptent rapidement des technologies émergentes.
* Garants de la fiabilité et la stabilité des infrastructures, les administrateurs peuvent avoir une approche plus conservatrice : est-ce que telle nouvelle technologie ne va pas amener de nouvelles problématiques (notamment de sécurité) ?

L'approche DevOps cherche à "réconcilier" l'innovation permanente et la fiabilité.

Dans la foulée du mouvement Agile, le mouvement DevOps a donné naissance à de nombreux outils, qui changent la façon dont on développe et met en production des applications.

### Du besoin d'automatisation

Les architectes réseau conçoivent des infrastructures, et les administrateurs système et réseau les exploitent et les maintiennent (ces rôles peuvent être fusionnés dans des organisations de petite taille).

Prenons l'exemple d'un administrateur chargé de mettre en ligne une application web, de façon à la rendre accessible sur Internet, ou sur l'Intranet d'une entreprise.

Dans une approche traditionnelle, il doit installer un système d'exploitation sur une machine (physique ou virtuelle), puis différents paquets logiciels requis pour faire fonctionner l'application.

Si on prend l'exemple d'une application "full-stack" JavaScript, avec un backend Node.js relié à une base de données MySQL, et un frontend Angular, il ou elle devra :

* Installer Node.js,
* Installer un serveur web qui redirigera le trafic vers l'application Node.js,
* Installer le serveur MySQL, créer une base de données pour l'application, configurer un compte utilisateur avec des droits sur cette base de données,
* Récupérer les applications backend et frontend depuis des dépôts Git (hébergés sur GitHub, BitBucket, GitLab, etc.),
* Éventuellement "builder" les applications,
* Faire en sorte que l'application Node.js redémarre automatiquement si elle plante,
* Mettre en place des outils de monitoring, afin d'être averti.e d'incidents,
* Mettre en place des sauvegardes automatiques,
* Gérer le renouvellement des certificats pour le HTTPS,
* etc. !

Toutes ces opérations peuvent être accomplies via des commandes shell, à saisir dans un certain ordre.

Si on effectue cela manuellement, c'est à la fois très chronophage et sujet à des erreurs humaines : même si la procédure est scrupuleusement documentée, personne n'est à l'abri de l'oubli d'une étape.

Une approche possible est d'utiliser des _scripts shell_ pour automatiser, dans une certaine mesure, tout cela. C'est une approche pertinente, mais qui peut poser des problèmes : comment faire si on doit relancer une partie seulement de la procédure ? Est-ce que le fait de lancer plusieurs fois le même script aboutira toujours au même résultat ?

Ansible va permettre de répondre à cette problématique d'automatisation, de façon fiable et efficace.

> Gardez à l'esprit qu'Ansible n'est pas le seul outil permettant de répondre à cette problématique. Des outils tels que Docker offrent une autre approche. Il ne s'agit cependant pas de les opposer, car ils peuvent être complémentaires.

### Qu'est-ce qu'Ansible ?

Ansible est un outil permettant d'automatiser toutes sortes de tâches sur des serveurs.

À partir d'un noeud appelé _control node_, on contrôle des serveurs appelés _managed nodes_.

On va, par exemple, pouvoir lancer la même commande sur tous les _managed nodes_, à partir du _control node_.

L'intérêt est d'automatiser la configuration de serveurs, ce qui offre un gain de temps d'autant plus appréciable que la "flotte" de machines à configurer est conséquente.

## Installer et utiliser Ansible

### Ansible sous Windows ?

Ansible est un outil orienté Unix. De fait, le _control node_ doit impérativement être un système Un*x ("Unix-like") tel que Linux, FreeBSD, MacOS, etc. Le portage du contrôleur Ansible sous Windows est, de l'aveu de ses développeurs, une tâche colossale, qui ne sera vraisemblablement pas réalisée avant quelques années.

On peut cependant utiliser Ansible sous Windows :

* soit en utilisant le sous-système Linux pour Windows [WSL2](https://docs.microsoft.com/fr-fr/windows/wsl/install), qui servira de _control node_, et au moins une ou plusieurs machine(s) virtuelle(s) pour le(s) _managed node(s)_.
* soit uniquement dans des machines virtuelles, le _control node_ étant aussi une VM.

### VirtualBox ?

Plusieurs logiciels de virtualisation sont disponibles sur le marché : VMWare, VirtualBox, etc.

VMWare Player (la version gratuite), souffre de limitations, notamment concernant la mise en réseau de machines virtuelles. VirtualBox est gratuit et moins limité.

On pourrait donc l'utiliser, et préparer manuellement des machines virtuelles. Une telle entreprise implique entre autres de paramétrer :

* la quantité de RAM
* un disque dur virtuel
* une ou plusieurs interfaces réseau

Il faut également télécharger une "image ISO" d'une distribution Linux (Debian, CentOS, openSUSE, etc.) qu'il faudra installer sur le disque dur virtuel de la VM, comme on le ferait sur une machine physique.

Cette procédure est assez rapide quand on en a l'habitude, et une fois la première VM ainsi configurée, il est facile de la cloner pour éviter la duplication d'efforts. Cependant, il est tout aussi facile de faire une erreur au cours de la procédure, quand on débute !

> :warning: **Ajout** : pour certain.e.s d'entre vous, Vagrant, décrit dans la section suivante, a posé des problèmes, aussi une [alternative basée sur VirtualBox](https://github.com/bhubr/ipi-devops/blob/master/ansible-on-virtualbox.md) vous est proposée.

### Vagrant

Vagrant permet de configurer très facilement des machines virtuelles, à partir d'un fichier de configuration, le `Vagrantfile`.

Vagrant repose par défaut sur VirtualBox, mais offre, entre autres avantages, la simplicité et rapidité de mise en oeuvre : au lieu de devoir installer soi-même un OS dans une VM, on peut démarrer une VM à partir d'une "box" pré-configurée, téléchargée automatiquement par Vagrant depuis un dépôt central.

## Installation et configuration de Vagrant

* Télécharger la version **64-bit** depuis [la page officielle de téléchargements](https://www.vagrantup.com/downloads)
* Lancer l'installeur (5 étapes, garder les choix proposés par défaut)

### Premier test de Vagrant

Juste pour verifier que tout fonctionne correctement !

* Lancer PowerShell
* Se rendre dans Documents : `cd ~\Documents`
* Creer un dossier `DevOps` : `mkdir DevOps`
* Puis creer un sous-dossier `TestDebian` : `mkdir DevOps\TestDebian`
* Aller sous ce dossier : `cd DevOps\TestDebian`
* Initialiser une machine virtuelle. On a choisi une Debian 64-bits (version 11, au nom de code "bullseye") : `vagrant init debian/bullseye64`
* La commande précédente a généré un fichier `Vagrantfile`, qu'on va garder tel quel
* Puis demarrer la VM : `vagrant up` (devrait donner l'affichage assez verbeux ci-dessous)
* Puis se connecter dans la VM : `vagrant ssh`
* Si tout a bien fonctionné : quitter la session SSH en saisissant `logout`
* Arrêter la VM : `vagrant halt`
* **Garder PowerShell ouvert** (il va nous resservir !)

Sortie obtenue en lançant `vagrant up` (notez que la box `debian/bullseye64` est téléchargée depuis le dépôt Vagrant Cloud) :

```
Bringing machine 'default' up with 'virtualbox' provider...
==> default: Box 'debian/bullseye64' could not be found. Attempting to find and install...
    default: Box Provider: virtualbox
    default: Box Version: >= 0
==> default: Loading metadata for box 'debian/bullseye64'
    default: URL: https://vagrantcloud.com/debian/bullseye64
==> default: Adding box 'debian/bullseye64' (v11.20211230.1) for provider: virtualbox
    default: Downloading: https://vagrantcloud.com/debian/boxes/bullseye64/versions/11.20211230.1/providers/virtualbox.box
    default:
==> default: Successfully added box 'debian/bullseye64' (v11.20211230.1) for 'virtualbox'!
==> default: Importing base box 'debian/bullseye64'...
==> default: Matching MAC address for NAT networking...
==> default: Checking if box 'debian/bullseye64' version '11.20211230.1' is up to date...
==> default: Setting the name of the VM: vagrant-debian11-test_default_1641156963418_34483
Vagrant is currently configured to create VirtualBox synced folders with
the `SharedFoldersEnableSymlinksCreate` option enabled. If the Vagrant
guest is not trusted, you may want to disable this option. For more
information on this option, please refer to the VirtualBox manual:

  https://www.virtualbox.org/manual/ch04.html#sharedfolders

This option can be disabled globally with an environment variable:

  VAGRANT_DISABLE_VBOXSYMLINKCREATE=1

or on a per folder basis within the Vagrantfile:

  config.vm.synced_folder '/host/path', '/guest/path', SharedFoldersEnableSymlinksCreate: false
==> default: Clearing any previously set network interfaces...
==> default: Preparing network interfaces based on configuration...
    default: Adapter 1: nat
==> default: Forwarding ports...
    default: 22 (guest) => 2222 (host) (adapter 1)
==> default: Booting VM...
==> default: Waiting for machine to boot. This may take a few minutes...
    default: SSH address: 127.0.0.1:2222
    default: SSH username: vagrant
    default: SSH auth method: private key
    default: Warning: Connection reset. Retrying...
    default:
    default: Vagrant insecure key detected. Vagrant will automatically replace
    default: this with a newly generated keypair for better security.
    default:
    default: Inserting generated public key within guest...
    default: Removing insecure key from the guest if it's present...
    default: Key inserted! Disconnecting and reconnecting using new SSH key...
==> default: Machine booted and ready!
==> default: Checking for guest additions in VM...
    default: The guest additions on this VM do not match the installed version of
    default: VirtualBox! In most cases this is fine, but in rare cases it can
    default: prevent things such as shared folders from working properly. If you see
    default: shared folder errors, please make sure the guest additions within the
    default: virtual machine match the version of VirtualBox you have installed on
    default: your host and reload your VM.
    default:
    default: Guest Additions Version: 6.0.0 r127566
    default: VirtualBox Version: 6.1
==> default: Mounting shared folders...
    default: /vagrant => D:/IPI_DevOps/vagrant-debian11-test

==> default: Machine 'default' has a post `vagrant up` message. This is a message
==> default: from the creator of the Vagrantfile, and not from Vagrant itself:
==> default:
==> default: Vanilla Debian box. See https://app.vagrantup.com/debian for help and bug reports
```

> :warning: **Ajout** : il est de bon ton d'**éteindre** une VM après usage, et éventuellement (pour cette VM de test) de la supprimer.
> * `vagrant halt` permet d'éteindre la VM
> * `vagrant destroy` permet de la supprimer définitivement (**à manier avec précaution** si votre VM contient le fruit de plusieurs heures de travail 😅)

## Vagrantfile pour configurer plusieurs VMs

### Obtenir le Vagrantfile et lancer les VMs

Dans votre PowerShell, vérifiez l'endroit où vous vous trouvez en examinant l'invite de commandes. Elle devrait ressembler à ceci :

```
PS C:\Users\IPI\Documents\DevOps\TestDebian>
```

À partir de là

* **Remonter d'un niveau dans l'arborescence** : `cd ..`
* Créer un nouveau dossier `AnsibleDebian` : `mkdir AnsibleDebian`
* Se placer dans ce dossier :  `cd AnsibleDebian`
* Télécharger un `Vagrantfile` un peu plus complexe, depuis <https://m1cp.bhu.ovh/vagrant-2-vms/Vagrantfile>, en lançant cette commande : `Invoke-WebRequest -Uri "https://m1cp.bhu.ovh/vagrant-2-vms/Vagrantfile" -OutFile ".\Vagrantfile"`
* Démarrer les VMs : `vagrant up`

La sortie obtenue ressemble a ceci :

```
Bringing machine 'managed-host1' up with 'virtualbox' provider...
Bringing machine 'ansible-host' up with 'virtualbox' provider...
==> managed-host1: Importing base box 'debian/bullseye64'...
==> managed-host1: Matching MAC address for NAT networking...
==> managed-host1: Checking if box 'debian/bullseye64' version '11.20211230.1' is up to date...
==> managed-host1: Setting the name of the VM: vagrant-multi-vms_managed-host1_1641160372389_33571
==> managed-host1: Clearing any previously set network interfaces...
==> managed-host1: Preparing network interfaces based on configuration...
    managed-host1: Adapter 1: nat
    managed-host1: Adapter 2: hostonly
==> managed-host1: Forwarding ports...
    managed-host1: 22 (guest) => 2222 (host) (adapter 1)
==> managed-host1: Running 'pre-boot' VM customizations...
==> managed-host1: Booting VM...
==> managed-host1: Waiting for machine to boot. This may take a few minutes...
    managed-host1: SSH address: 127.0.0.1:2222
    managed-host1: SSH username: vagrant
    managed-host1: SSH auth method: private key
==> managed-host1: Machine booted and ready!
==> managed-host1: Checking for guest additions in VM...
    managed-host1: The guest additions on this VM do not match the installed version of
    managed-host1: VirtualBox! In most cases this is fine, but in rare cases it can
    managed-host1: prevent things such as shared folders from working properly. If you see
    managed-host1: shared folder errors, please make sure the guest additions within the
    managed-host1: virtual machine match the version of VirtualBox you have installed on
    managed-host1: your host and reload your VM.
    managed-host1:
    managed-host1: Guest Additions Version: 6.0.0 r127566
    managed-host1: VirtualBox Version: 6.1
==> managed-host1: Setting hostname...
==> managed-host1: Configuring and enabling network interfaces...
==> ansible-host: Importing base box 'debian/bullseye64'...
==> ansible-host: Matching MAC address for NAT networking...
==> ansible-host: Checking if box 'debian/bullseye64' version '11.20211230.1' is up to date...
==> ansible-host: Setting the name of the VM: vagrant-multi-vms_ansible-host_1641160427644_1154
==> ansible-host: Fixed port collision for 22 => 2222. Now on port 2200.
==> ansible-host: Clearing any previously set network interfaces...
==> ansible-host: Preparing network interfaces based on configuration...
    ansible-host: Adapter 1: nat
    ansible-host: Adapter 2: hostonly
==> ansible-host: Forwarding ports...
    ansible-host: 22 (guest) => 2200 (host) (adapter 1)
==> ansible-host: Running 'pre-boot' VM customizations...
==> ansible-host: Booting VM...
==> ansible-host: Waiting for machine to boot. This may take a few minutes...
    ansible-host: SSH address: 127.0.0.1:2200
    ansible-host: SSH username: vagrant
    ansible-host: SSH auth method: private key
==> ansible-host: Machine booted and ready!
==> ansible-host: Checking for guest additions in VM...
    ansible-host: The guest additions on this VM do not match the installed version of
    ansible-host: VirtualBox! In most cases this is fine, but in rare cases it can
    ansible-host: prevent things such as shared folders from working properly. If you see
    ansible-host: shared folder errors, please make sure the guest additions within the
    ansible-host: virtual machine match the version of VirtualBox you have installed on
    ansible-host: your host and reload your VM.
    ansible-host:
    ansible-host: Guest Additions Version: 6.0.0 r127566
    ansible-host: VirtualBox Version: 6.1
==> ansible-host: Setting hostname...
==> ansible-host: Configuring and enabling network interfaces...

==> managed-host1: Machine 'managed-host1' has a post `vagrant up` message. This is a message
==> managed-host1: from the creator of the Vagrantfile, and not from Vagrant itself:
==> managed-host1:
==> managed-host1: Vanilla Debian box. See https://app.vagrantup.com/debian for help and bug reports

==> ansible-host: Machine 'ansible-host' has a post `vagrant up` message. This is a message
==> ansible-host: from the creator of the Vagrantfile, and not from Vagrant itself:
==> ansible-host:
==> ansible-host: Vanilla Debian box. See https://app.vagrantup.com/debian for help and bug reports
```

Les VMs étant démarrées, on va encore devoir faire un peu de configuration, avant de se pencher sur Ansible proprement dit !

### Première connexion aux VMs

Pour tester la connexion vers chaque VM via SSH, on va à nouveau utiliser `vagrant ssh`.

Comme on dispose maintenant de deux VMs, il va cette fois falloir spécifier sur quelle machine on souhaite se connecter.

Commençons par nous connecter sur la VM faisant office de contrôleur Ansible : 

    vagrant ssh ansible-host

On doit voir apparaître un prompt (invite de commandes) ressemblant à ceci :

    vagrant@ansible-host: $

Enfin, testons l'accès à Internet :

    ping google.fr

On doit voir s'afficher des séquences d'échange indiquant le temps mis pour la requête à faire l'aller retour :

    vagrant@ansible-host:~$ ping google.fr
    PING google.fr (142.251.37.163) 56(84) bytes of data.
    64 bytes from mrs09s14-in-f3.1e100.net (142.251.37.163): icmp_seq=1 ttl=116 time=9.00 ms
    64 bytes from mrs09s14-in-f3.1e100.net (142.251.37.163): icmp_seq=2 ttl=116 time=7.94 ms
    64 bytes from mrs09s14-in-f3.1e100.net (142.251.37.163): icmp_seq=3 ttl=116 time=8.95 ms

Dans le cas contraire, rien ne s'affiche sous la ligne `PING`.

On peut se déconnecter (`logout`), puis effectuer la même opération pour la VM faisant office de serveur contrôlé par Ansible : `vagrant ssh managed-host1`. De même, testons l'accès à Internet : `ping google.fr`. Puis `logout` une fois qu'on a vérifié que cela fonctionnait.

## Configuration des VMs

`vagrant ssh` permet de se connecter de notre système hôte (la machine Windows) vers les systèmes "invités" (les VMs).

**Note** : Si on n'utilisait pas Vagrant, on utiliserait simplement la commande `ssh` pour se connecter aux VMs. Ce serait le cas si on avait utilisé VirtualBox directement. Le revers de la médaille est qu'on aurait eu un peu plus de configuration à effectuer.

Gardez juste en tête que `vagrant ssh` est spécifique à ce "setup" basé sur Vagrant. Ceci dit, nous allons quand même utiliser `ssh` pour permettre au _control node_ Ansible de se connecter au(x) _managed node(s)_ (ici un seul).

### Tentative de connexion du _control node_ au _managed node_

Dans le `Vagrantfile` qui a servi à initialiser les 2 VMs, on a attribué une adresse IP statique à chacune des VMs, qui sont sur le même réseau LAN "interne" :

* `192.168.56.1` pour `ansible-host`
* `192.168.56.10` pour `managed-host1`

Essayons de nous connecter du 1er au 2nd :

* Se connecter à `ansible-host` depuis le système Windows : `vagrant ssh ansible-host`
* Une fois qu'on est connecté : `ssh 192.168.56.10`
* On obtient un message d'erreur : `vagrant@192.168.56.1: Permission denied (publickey).`

À ce stade, il peut être intéressant de rappeler quelques bases sur SSH !

**S**ecure **SH**ell permet de créer un tunnel sécurisé entre deux machines : sécurisé, car les informations qui transitent sont chiffrées.

SSH repose sur le chiffrement asymétrique. C'est un sujet assez complexe et qui dépasse le cadre de ce cours, mais en très bref, on a besoin de généré une paire de clés (_keypair_, parfois dénommée biclé en français) :

* Une clé privée qui doit rester sur le serveur **depuis lequel on se connecte**
* Une clé publique qu'on va placer sur le serveur **auquel on souhaite se connecter**

N'ayant pas pour l'instant de paire de clés, on ne peut pour l'instant pas se connecter du _control node_ au _managed node_.

### Génération d'une paire de clés SSH

> :warning: Section qui sera revue suite aux problèmes rencontré.e.s par certain.e.s ave Vagrant : en effet, une partie est commune à Vagrant et Virtualbox, une partie est spécifique à Vagrant.

On va générer une paire de clés SSH, depuis le _control node_, via la commande `ssh-keygen`.

L'un de ses nombreux arguments acceptés par cette commande est le type de clé (algorithme utilisé), via `-t` suivi d'un type. Entre autres types possibles : `dsa`, `rsa` (plutôt obsolètes désormais), `ecdsa`, etc.

On va créer une paire de clés de type ECDSA :

    ssh-keygen -t ecdsa

On nous demande alors l'emplacement où sauvegarder la clé :

    Generating public/private ecdsa key pair.
    Enter file in which to save the key (/home/vagrant/.ssh/id_ecdsa):

Il suffit de valider avec entrée, pour garder le choix par défaut `/home/vagrant/.ssh/id_ecdsa`.

On nous demande ensuite une "passphrase" :

    Enter passphrase (empty for no passphrase):

Par souci de simplicité, on va valider avec entrée **deux fois** pour générer la paire de clés sans passphrase. Ce n'est pas sécurisé, mais cela simplifiera l'exercice.

> Dans un environnement professionnel, on cherchera à augmenter le niveau de sécurité en spécifiant une passphrase : 15 caractères minimum, 20 de préférence, à stocker dans un gestionnaire de mots de passe. Des outils (`ssh-agent` et `ssh-add`) permettront d'éviter d'avoir à la saisir trop souvent.

Après la validation de la passphrase vide, on obtient des informations sur l'emplacement des clés privée (`id_ecdsa`) et publique (`id_ecdsa.pub`), l'empreinte de la clé, et un "randomart" qui peut servir à valider la clé de façon visuelle (ce qu'on ne fera pas ici) :

```
Your identification has been saved in /home/vagrant/.ssh/id_ecdsa
Your public key has been saved in /home/vagrant/.ssh/id_ecdsa.pub
The key fingerprint is:
SHA256:xQbzm8dGcJPISgLAsI80CkkHlWAJzKfTEp+xFdAQv2A vagrant@ansible-host
The key's randomart image is:
+---[ECDSA 256]---+
|*B*BBo. o...o.   |
|oBo+.o. .=oo..   |
|+o*E=. o .= .    |
|+*o=. . .o =     |
|o +  .  S o +    |
|           o     |
|                 |
|                 |
|                 |
+----[SHA256]-----+
```

On peut vérifier la présence des clés dans le dossier `.ssh` :

    ls -al .ssh

Affichage produit :

    total 24
    drwx------ 2 vagrant vagrant 4096 Jan  2 23:50 .
    drwxr-xr-x 3 vagrant vagrant 4096 Jan  2 23:45 ..
    -rw------- 1 vagrant vagrant  409 Dec 30 17:14 authorized_keys
    -rw------- 1 vagrant vagrant  513 Jan  2 23:50 id_ecdsa
    -rw-r--r-- 1 vagrant vagrant  182 Jan  2 23:50 id_ecdsa.pub
    -rw-r--r-- 1 vagrant vagrant  222 Jan  2 21:56 known_hosts

Profitons-en pour expliquer le rôle des deux autres fichiers :

* `known_hosts` permet de stocker les "empreintes" des serveurs auxquels on se connecte, et de vérifier qu'elles n'ont pas changé lors de connexions ultérieures (ce qui indiquerait vraisemblablement une attaque).
* `authorized_keys` est l'endroit où on va stocker les clés publiques qu'on veut autoriser à se connecter sous ce compte utilisateur.

**La prochaine étape** va être de copier la clé publique (`id_ecdsa.pub`) vers le fichier `authorized_keys` de la machine sur laquelle on veut se connecter&hellip; ce qu'on ferait normalement via la commande `ssh-copy-id`, de cette façon (on indique en dernier l'IP ou le nom d'hôte de la machine cible, éventuellement précédé d'un nom d'utilisateur séparé de l'IP par un `@`). Cette commande est un **exemple**, ne la lancez pas !

```
ssh-copy-id -i ~/.ssh/id_ecdsa.pub someuser@192.168.56.10
```

Problème : en l'état actuel, on ne peut pas se connecter en SSH, ainsi qu'on l'a vu précédemment ! Il y a heureusement plusieurs parades.

Quittez la VM en saisissant `logout`.

### Transferer la clé publique

À nouveau, cette étape est un peu spécifique au setup basé sur Vagrant.

Installez le plugin `vagrant-scp`, qui va permettre de faire des copies entre les machines virtuelles et le système hôte Windows.
    
```
vagrant plugin install vagrant-scp
```

Ensuite, on va pouvoir récupérer la clé publique depuis `ansible-host` :

```
vagrant scp ansible-host:.ssh/id_ecdsa.pub .
```

Note : `vagrant scp` utilise la même syntaxe que `scp` (secure copy). Il y a ici deux arguments :

* `ansible-host:.ssh/id_ecdsa.pub` est la source. Avant le `:`, on trouve la machine depuis laquelle on veut récupérer un fichier. Après le `:`, on trouve le chemin relatif au compte utilisateur (`/home/vagrant`) propriétaire du fichier : `.ssh/id_ecdsa.pub`.
* `.` désigne le répertoire courant sur la machine cible

Puis on va copier la clé vers `managed-host1` :

```
vagrant scp id_ecdsa.pub managed-host1:
```

Ici, `id_ecdsa.pub` est la source (située dans le répertoire courant du système hôte), et `managed-host1:` est la destination (implicitement, le répertoire `/home/vagrant` sur `managed-host1`).

On va maintenant se connecter au `managed-host1` :

    vagrant ssh managed-host1

Puis on peut constater que la clé publique s'y trouve bien en lançant `ls`. Ensuite, il va falloir ajouter le contenu de ce fichier au fichier `.ssh/authorized_keys`. On va détailler un peu l'opération.

D'abord, visualisez le contenu actuel du fichier `.ssh/authorized_keys` :

```
vagrant@managed-host1:~$ cat .ssh/authorized_keys
ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEA6NF8iallvQVp22WDkTkyrtvp9eWW6A8YVr+kz4TjGYe7gHzIw+niNltGEFHzD8+v1I2YJ6oXevct1YeS0o9HZyN1Q9qgCgzUFtdOKLv6IedplqoPkcmF0aYet2PkEDo3MlTBckFXPITAMzF8dJSIFo9D8HfdOV0IAdx4O7PtixWKn5y2hMNG0zQPyUecp4pzC6kivAIhyfHilFR61RGL+GPXQ2MWZWFYbAGjyiYJnAmCP3NOTd0jMZEnDkbUvxhMmBYSdETk1rRgm+R4LOzFUGaHqHDLKLX+FIPKcF96hrucXzcWyLbIbEgE98OHlnVYCzRdK8jlqm8tehUc9c9WhQ== vagrant insecure public key
```

Visualisez maintenant le contenu de `id_ecdsa.pub` :

```
vagrant@managed-host1:~$ cat id_ecdsa.pub
ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBBSkxnUn57qBx+qMTLuxPWoV41hrXOVKAt4lYeIb/jOYjuIWprwFZRxBBlXf+VnfbJbEWnSQIEqmp9Bii2Sayfs= vagrant@ansible-host
```

Cette commande dérivée de la précédente permet de **rediriger** son affichage, en l'ajoutant (via `>>`) au contenu de `authorized_keys`.

    cat id_ecdsa.pub >> .ssh/authorized_keys

Visualisez à nouveau `authorized_keys` :

```
vagrant@managed-host1:~$ cat .ssh/authorized_keys
ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEA6NF8iallvQVp22WDkTkyrtvp9eWW6A8YVr+kz4TjGYe7gHzIw+niNltGEFHzD8+v1I2YJ6oXevct1YeS0o9HZyN1Q9qgCgzUFtdOKLv6IedplqoPkcmF0aYet2PkEDo3MlTBckFXPITAMzF8dJSIFo9D8HfdOV0IAdx4O7PtixWKn5y2hMNG0zQPyUecp4pzC6kivAIhyfHilFR61RGL+GPXQ2MWZWFYbAGjyiYJnAmCP3NOTd0jMZEnDkbUvxhMmBYSdETk1rRgm+R4LOzFUGaHqHDLKLX+FIPKcF96hrucXzcWyLbIbEgE98OHlnVYCzRdK8jlqm8tehUc9c9WhQ== vagrant insecure public key
ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBBSkxnUn57qBx+qMTLuxPWoV41hrXOVKAt4lYeIb/jOYjuIWprwFZRxBBlXf+VnfbJbEWnSQIEqmp9Bii2Sayfs= vagrant@ansible-host
```

La clé a bien été ajoutée ! Quittez la session SSH via `logout`.

### Retour au _control node_

Connexion au control node :

    vagrant ssh ansible-host

À nouveau, tenative de connexion via `ssh` :

    ssh 192.168.56.10

Cette fois-ci, cela fonctionne !

```
vagrant@ansible-host:~$ ssh 192.168.56.10
Linux managed-host1 5.10.0-10-amd64 #1 SMP Debian 5.10.84-1 (2021-12-08) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Mon Jan  3 00:20:27 2022 from 10.0.2.2
```

Quittez avec `logout` (raccourci : `Ctrl-D`) pour revenir au control node.

## Ansible

**Important** : Ansible ne doit être installé _que_ sur le control node. Sur les hôtes contrôlés par Ansible, il n'y a rien à installer !

C'est un des points forts d'Ansible, comparé à d'autres outils qui nécessitent l'installation d'un "agent" sur les machines contrôlées.

Ansible est "agentless" : il a juste besoin de pouvoir accéder aux noeuds contrôlés via SSH. 

### Installation d'Ansible sur le control node

On arrive enfin au vif du sujet : l'installation d'Ansible !

**Vérifiez que vous êtes bien sur le control node**, en inspectant l'invite de commandes :

```
vagrant@ansible-host:~$
```

Il existe plusieurs façons d'installer Ansible, ainsi que l'indique la section [Installing Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html) de la documentation officielle.

Nous allons suivre les instructions de la sous-section [Installing Ansible on Debian](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#installing-ansible-on-debian).

Elle repose sur l'utilisation de l'outil natif de gestion de paquets logiciels commun à Debian et Ubuntu : `apt`. Cette commande permet notamment d'installer des logiciels depuis des "depôts" publics.

La première chose à faire est de passer sous le compte `root` :

    sudo su -

Ensuite, on doit mettre a jour la liste des paquets logiciels disponibles :

    apt-get update
    
Puis installer le paquet `gnupg` (outil de chiffrement requis par l'une des commandes suivantes, `apt-key`) :

    apt install -y gnupg

Ensuite, il faut ajouter l'adresse du dépôt Ansible pour Ubuntu 20 `focal`, qui marche aussi pour Debian 11 `bullseye`, au fichier `/etc/apt/sources.list.d/ansible.list` :

    echo "deb http://ppa.launchpad.net/ansible/ansible/ubuntu focal main" >> /etc/apt/sources.list.d/ansible.list

Avec cette commande, on affiche la ligne de définition du dépôt avec `echo`, et on redirige cet affichage vers le fichier `/etc/apt/sources.list.d/ansible.list`, qui est créé s'il n'existait pas.

Ensuite, on doit ajouter une clé permettant d'authentifier les paquets :

    apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 93C4A3FD7BB9C367

Ensuite, on lance à nouveau `apt-get update`, qui permet d'aller chercher la liste des paquets disponibles dans le dépôt ajouté précédemment :

    apt-get update

Enfin, on installe Ansible (`-y` permet de passer l'étape de confirmation) :

    apt install -y ansible

Cette étape va prendre un peu de temps.

**Restez en `root` pour l'étape suivante** (il ne faudra pas oublier d'en ressortir quand on aura terminé toute la configuration).

### Configuration des hôtes

On doit maintenant indiquer à Ansible la liste des hôtes qu'il contrôle. Par défaut, c'est dans le fichier `/etc/ansible/hosts` qu'on les référence.

> :warning: **Ajout** : il est plutôt recommandé, par certains, de créer des fichiers d'inventaire (_inventory_) par projet, afin d'y référencer les hôtes concernés par ce projet. L'approche que nous adoptons ici nous évitera d'avoir à spécifier un fichier d'inventaire à chaque lancement d'`ansible`.

Les hôtes peuvent être rassemblés dans des groupes. Ici, on aura juste un groupe (`webservers`) qui contiendra la machine à contrôler, qu'on indiquera via son adresse IP.

Lancez l'éditeur `nano` pour éditer le fichier d'hôtes :

    nano /etc/ansible/hosts

Son contenu par défaut est celui indiqué ci-dessous. Toutes les lignes sont précédées du `#` (commentaires).

Décommentez la ligne `## [webservers]`, en enlevant les deux `#` **et l'espace qui les suit**.

Sous cette ligne, insérez l'adresse IP de l'hôte à contrôler :

```
192.168.56.10
```

Vous devez donc avoir ces deux lignes consécutives :

```
[webservers]
192.168.56.10
```

Sauvegardez le fichier avec le raccourci Ctrl-O. **Observez bien le bas de l'écran**, où on vous propose d'indiquer le nom du fichier à écrire :

    File Name to Write: /etc/ansible/hosts 

Vous n'avez ici **qu'à valider le choix proposé avec entrée**.

Puis utilisez le raccourci Ctrl-X pour quitter `nano`.

**Enfin**, quittez le mode `root` via Ctrl-D ou la commande `logout`.

### Tester le bon fonctionnement d'Ansible

On va enfin pouvoir exécuter nos premières commandes Ansible. Commencez par celle-ci :

    ansible all -m ping

Elle devrait, si tout va bien, se solder par l'affichage suivant :

```
192.168.56.10 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```

Quelques explictions s'imposent ! D'abord, l'anatomie de la commande `ansible all -m ping` :

`all` permet de lancer une commande sur _tous_ les hôtes référencés dans le fichier `/etc/ansible/hosts`. On aurait pu spécifier `webservers` au lieu de `all`. Essayez !

Vous constaterez que cela ne change rien, et c'est normal, car on n'a qu'un seul hôte contrôlé (choix assez prudent de ma part, je crains les limites en termes de RAM des ordinateurs de certain.e.s d'entre vous !).

`-m` permet de spécifier un "module". Ici `ping`. Un module est un genre de "plugin" qui contient des fonctionnalités. C'est une sorte de script qui va être exécuté sur les machines contrôlées, et va renvoyer des informations sous la forme d'une chaîne JSON.

Le module `ping` ne doit pas être confondu avec la commande `ping` du même nom (laquelle permet de vérifier qu'on peut joindre un certain hôte sur le réseau). Elle en est cependant l'équivalent dans Ansible : elle essaie de se connecter à l'hôte, vérifie que Python est installé et utilisable, et retourne `pong` si tout s'est bien passé.

### Commandes ad hoc

Ce qu'on vient de voir est une _ad hoc command_ ou commande ad hoc : cela permet d'exécuter une tâche spécifique sur tous les hôtes ciblés.

Les commandes ad hoc sont simples et rapides à utiliser, mais ne sont pas réutilisables. On verra plus loin le concept de "playbooks", permettant de créer des ensembles de tâches réutilisables.

En attendant, les commandes ad hoc ont leur utilité. Des exemples sont donnés dans la section [Introduction to ad hoc commands](https://docs.ansible.com/ansible/latest/user_guide/intro_adhoc.html) de la doc Ansible.

On y précise l'anatomie d'une commande ad hoc :

    ansible [pattern] -m [module] -a "[module options]"

#### Paramètres de modules

`-a` permet de passer des options spécifiques à un certain module.

Si on se rend sur la documentation du [module ping](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/ping_module.html), on voit, sous la section "Parameters", qu'il accepte le paramètre `data`, qui définit la donnée à renvoyer en cas de succès, et dont la valeur par défaut est `pong`.

Essayez ceci, pour changer la valeur de retour :

    ansible all -m ping -a data=hello

On constate que cela a eu l'effet escompté :

    192.168.56.10 | SUCCESS => {
        "ansible_facts": {
            "discovered_interpreter_python": "/usr/bin/python3"
        },
        "changed": false,
        "ping": "hello"
    }

Essayons une autre commande ad hoc. On va utiliser le module `apt` d'Ansible, qui permet de gérer les packages logiciels sur les hôtes contrôlés, via la commande `apt` que nous avions utilisée, manuellement, pour installer Ansible.

    ansible all -m ansible.builtin.apt -a "name=curl state=present"

Ceci permet d'installer le module `curl` s'il n'est pas présent.

On obtient le message suivant :

```
192.168.56.10 | FAILED! => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "msg": "No package matching 'curl' is available"
}
```

C'est étonnant, car le module `curl` est disponibles dans toutes les distributions Linux, et est même, le plus souvent, installé par défaut !

Ici, il nous faut ajouter un paramètre permettant d'exécuter `apt update` que nous avions également utilisée précédemment. Ce paramètre est `update_cache` et il faut le positionner à `yes` (voir la doc du module [ansible.builtin.apt](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/apt_module.html)).


    ansible all -m ansible.builtin.apt -a "name=curl state=present update_cache=yes"

Ceci va prendre un peu de temps ! On écope à nouveau d'un message d'erreur, quelque peu différent :

```
192.168.56.10 | FAILED! => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "msg": "Failed to lock apt for exclusive operation: Failed to lock directory /var/lib/apt/lists/: E:Could not open lock file /var/lib/apt/lists/lock - open (13: Permission denied)"
}
```

Ce message d'erreur est quelque peu cryptique, mais est facilement explicable : la gestion des paquets logiciels est du ressort de l'administrateur du système, l'utilisateur `root`.

#### Obtention des privilèges root

Le problème est qu'ici, la commande `apt-get` a été invoquée sur l'hôte cible avec les droits de l'utilisateur `vagrant`.

On va relancer la commande en ajoutant, juste derrière `ansible`, le paramètre `--become` qui va permettre de devenir `root` pour exécuter les commandes sur le système cible.

    ansible --become all -m ansible.builtin.apt -a "name=curl state=present update_cache=yes"

À nouveau, cela prend un peu de temps. Normalement, le résultat devrait être semblable à ceci :

```
192.168.56.10 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "cache_update_time": 1641178826,
    "cache_updated": true,
    "changed": true,
    "stderr": "",
    "stderr_lines": [],
    "stdout": "Reading package lists...\nBuilding dependency tree...\nReading state information...\nThe following additional packages will be installed:\n  libcurl4\nThe following NEW packages will be installed:\n  curl libcurl4\n0 upgraded, 2 newly installed, 0 to remove and 0 not upgraded.\nNeed to get 608 kB of archives.\nAfter this operation, 1186 kB of additional disk space will be used.\nGet:1 http://deb.debian.org/debian bullseye/main amd64 libcurl4 amd64 7.74.0-1.3+deb11u1 [341 kB]\nGet:2 http://deb.debian.org/debian bullseye/main amd64 curl amd64 7.74.0-1.3+deb11u1 [267 kB]\nFetched 608 kB in 0s (6271 kB/s)\nSelecting previously unselected package libcurl4:amd64.\r\n(Reading database ... \r(Reading database ... 5%\r(Reading database ... 10%\r(Reading database ... 15%\r(Reading database ... 20%\r(Reading database ... 25%\r(Reading database ... 30%\r(Reading database ... 35%\r(Reading database ... 40%\r(Reading database ... 45%\r(Reading database ... 50%\r(Reading database ... 55%\r(Reading database ... 60%\r(Reading database ... 65%\r(Reading database ... 70%\r(Reading database ... 75%\r(Reading database ... 80%\r(Reading database ... 85%\r(Reading database ... 90%\r(Reading database ... 95%\r(Reading database ... 100%\r(Reading database ... 25131 files and directories currently installed.)\r\nPreparing to unpack .../libcurl4_7.74.0-1.3+deb11u1_amd64.deb ...\r\nUnpacking libcurl4:amd64 (7.74.0-1.3+deb11u1) ...\r\nSelecting previously unselected package curl.\r\nPreparing to unpack .../curl_7.74.0-1.3+deb11u1_amd64.deb ...\r\nUnpacking curl (7.74.0-1.3+deb11u1) ...\r\nSetting up libcurl4:amd64 (7.74.0-1.3+deb11u1) ...\r\nSetting up curl (7.74.0-1.3+deb11u1) ...\r\nProcessing triggers for man-db (2.9.4-2) ...\r\nProcessing triggers for libc-bin (2.31-13+deb11u2) ...\r\n",
    "stdout_lines": [
        "Reading package lists...",
        "Building dependency tree...",
        "Reading state information...",
        "The following additional packages will be installed:",
        "  libcurl4",
        "The following NEW packages will be installed:",
        "  curl libcurl4",
        "0 upgraded, 2 newly installed, 0 to remove and 0 not upgraded.",
        "Need to get 608 kB of archives.",
        "After this operation, 1186 kB of additional disk space will be used.",
        "Get:1 http://deb.debian.org/debian bullseye/main amd64 libcurl4 amd64 7.74.0-1.3+deb11u1 [341 kB]",
        "Get:2 http://deb.debian.org/debian bullseye/main amd64 curl amd64 7.74.0-1.3+deb11u1 [267 kB]",
        "Fetched 608 kB in 0s (6271 kB/s)",
        "Selecting previously unselected package libcurl4:amd64.",
        "(Reading database ... ",
        "(Reading database ... 5%",
        "(Reading database ... 10%",
        "(Reading database ... 15%",
        "(Reading database ... 20%",
        "(Reading database ... 25%",
        "(Reading database ... 30%",
        "(Reading database ... 35%",
        "(Reading database ... 40%",
        "(Reading database ... 45%",
        "(Reading database ... 50%",
        "(Reading database ... 55%",
        "(Reading database ... 60%",
        "(Reading database ... 65%",
        "(Reading database ... 70%",
        "(Reading database ... 75%",
        "(Reading database ... 80%",
        "(Reading database ... 85%",
        "(Reading database ... 90%",
        "(Reading database ... 95%",
        "(Reading database ... 100%",
        "(Reading database ... 25131 files and directories currently installed.)",
        "Preparing to unpack .../libcurl4_7.74.0-1.3+deb11u1_amd64.deb ...",
        "Unpacking libcurl4:amd64 (7.74.0-1.3+deb11u1) ...",
        "Selecting previously unselected package curl.",
        "Preparing to unpack .../curl_7.74.0-1.3+deb11u1_amd64.deb ...",
        "Unpacking curl (7.74.0-1.3+deb11u1) ...",
        "Setting up libcurl4:amd64 (7.74.0-1.3+deb11u1) ...",
        "Setting up curl (7.74.0-1.3+deb11u1) ...",
        "Processing triggers for man-db (2.9.4-2) ...",
        "Processing triggers for libc-bin (2.31-13+deb11u2) ..."
    ]
}
```

Notez le début de cette sortie : `192.168.56.10 | CHANGED`.

Cela signifie que la commande a bien effectué des changements sur le système cible. Relancez-la (flèche du haut puis entrée).

Cette fois, le résultat est différent :

```
192.168.56.10 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "cache_update_time": 1641178826,
    "cache_updated": false,
    "changed": false
}
```

La commande a réussi, mais n'a appliqué aucun changement sur le système cible, `curl` ayant déjà installé par l'invocation précédente.

### Playbooks

Voir la section [Intro to playbooks](https://docs.ansible.com/ansible/latest/user_guide/playbooks_intro.html) de la documentation officielle pour référence.

Les playbooks Ansible permettent de spécifier des tâches à exécuter sur différentes machines. Dès lors qu'on souhaite exécuter une certaine tâche plus d'une fois, il est préférable de recourir aux playbooks.

Un playbook est un fichier au format YAML, qui peut (et doit) être géré via un outil de gestion de versions tel que Git.

Ainsi, une fois écrit, le playbook pourra être réutilisé.

#### Un exemple simple

Voici un playbook très simple, qui va effectuer la même chose que ce qu'on avait fait précédemment avec une commande ad hoc, pour installer `curl`.

```yaml
---
- name: Install curl
  hosts: webservers

  tasks:
  - name: Ensure curl is at the latest version
    ansible.builtin.apt:
      name: curl
      state: latest
    become: yes
```

Ce playbook définit une seule tâche, qui comporte :

* un nom (`name`)
* un module (`ansible.builtin.apt`) et des paramètres associés
* Le paramètre `become` positionné sur `yes`, indiquant qu'il faut obtenir les privilèges `root` afin d'exécuter la commande

Copiez le bloc de configuration ci-dessus. Dans la fenêtre de PowerShell, toujours connectée sur `ansible-host`, lancez :

    nano playbook-curl.yaml

Puis collez le contenu copié, sauvegardez (Ctrl-O puis entrée), et quittez (Ctrl-X).

Enfin, exécutez le playbook en utilisant la commande `ansible-playbook`, suivi du nom du playbook :

    ansible-playbook playbook-curl.yaml

Celle-ci produit l'affichage suivant :

```

PLAY [Install curl] ***********************************************************************************************

TASK [Gathering Facts] ********************************************************************************************
ok: [192.168.56.10]

TASK [Ensure curl is at the latest version] ***********************************************************************
ok: [192.168.56.10]

PLAY RECAP ********************************************************************************************************
192.168.56.10              : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

```

Entre autres choses, on peut noter, à la fin, `changed=0`, ce qui signifie que le playbook n'a effectué aucun changement (`curl` étant déjà installé sur l'hôte cible).

#### La syntaxe YAML

Avant d'aller plus loin, il peut être intéressant de se familiariser avec la syntaxe YAML. D'autant plus qu'elle n'est pas spécifique à Ansible, loin de là !

On l'utilise également :

* Dans le monde du DevOps, pour configurer des outils tels que Docker Compose ou Kubernetes
* Pour d'autres outils, tels que Jekyll (générateur de site statique à partir de fichiers Markdown, dont l'en-tête est au format YAML)

Quelques ressources (choisissez le format qui vous convient le mieux) :

* La section [YAML Syntax](https://docs.ansible.com/ansible/latest/reference_appendices/YAMLSyntax.html#yaml-syntax) de la doc Ansible
* [Introduction à YAML](https://sweetohm.net/article/introduction-yaml.html) en français, mais avec des exemples de manipulation du format YAML en Python, pour les téméraires !
* [YAML Tutorial | Learn YAML in 18 mins](https://www.youtube.com/watch?v=1uFVr15xDGg) vidéo en anglais (d'une autrice qui propose énormément de contenus très qualitatifs sur le DevOps)

#### Installer un serveur web

Source dont on s'inspire largement : <https://iac.goffinet.org/ansible-linux/un-premier-playbook/>

Aller dans la section "4. un premier livre de jeu". Un exemple de config YAML est donné.

On ne va prendre (copier dans le presse-papier) que les lignes jusqu'à la première tâche :

```yaml
---
- name: Configure webserver with nginx
  hosts: webservers
  become: True
  become_method: sudo
  tasks:
    - name: install nginx
      apt:
        name: nginx
        update_cache: yes
```

**Sur le `ansible-host`** (sans être `root`) :

* Editer un nouveau fichier : `nano debian-nginx.yaml`
* Y coller le bloc ci-dessus

Puis exécuter le playbook : `ansible-playbook debian-nginx.yaml`

Il est _possible_ de rencontrer une erreur à ce stade, ce qui a été mon cas !

```
vagrant@ansible-host:~/nginx$ ansible-playbook debian-nginx.yaml 

PLAY [Configure webserver with nginx] ****************************************************************

TASK [Gathering Facts] *******************************************************************************
ok: [192.168.56.2]

TASK [install nginx] *********************************************************************************
fatal: [192.168.56.2]: FAILED! => {"changed": false, "msg": "Failed to update apt cache: E:Release file for http://deb.debian.org/debian/dists/bullseye-updates/InRelease is not valid yet (invalid for another 2h 46min 20s). Updates for this repository will not be applied., E:Release file for http://deb.debian.org/debian/dists/bullseye-backports/InRelease is not valid yet (invalid for another 2h 46min 20s). Updates for this repository will not be applied."}

PLAY RECAP *******************************************************************************************
192.168.56.2               : ok=1    changed=0    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0   

```

L'erreur provient de problèmes sur les dépôts `bullseye-updates` et `bullseye-backports` référencés dans `/etc/apt/sources.list`.

Dans ce cas et **dans ce cas seulement**, ajouter cette tâche au-dessus de la tâche "install nginx" de `debian-nginx.yaml` :

```
    - name: disable bullseye-updates and bullseye-backports
      replace:
        path: /etc/apt/sources.list
        regexp: '^deb(-src)? (.*) bullseye-(updates|backports) main'
        replace: '#deb\1 \2 bullseye-\3 main'
```

Cela permet d'ajouter un `#` dans les lignes incriminées de `/etc/apt/sources.list`, afin de désactiver ces dépôts (ce qui n'aura normalement pas d'incidence pour la suite).

**Si tout se passe bien**, on va passer à la suite. Toujours dans le même dossier, lancez cette commande :

    wget https://github.com/bhubr/ipi-devops/raw/master/sample-playbooks/nginx/nginx.tar

Cela vous permet de récupérer une archive "tar" (le format d'archive natif d'Unix) contenant des fichiers supplémentaires pour nginx :

* Un fichier de configuration,
* Un fichier `index.html`,

Qui vont remplacer ceux fournis par défaut avec nginx.

Décompressez cette archive :

    tar xvf nginx.tar

Cela devrait afficher la liste des fichiers :

```
files/._nginx.conf
files/nginx.conf
templates/index.html
```

(le fichier `files/._nginx.conf` ne sert à rien et a été ajouté automatiquement par mon Mac !)

Ensuite, éditez à nouveau le fichier du playbook nginx : `nano debian-nginx.yaml`.

Ajoutez, sous la tâche "install nginx", les deux tâches suivantes :

```
    - name: copy nginx config file
      copy:
        src: files/nginx.conf
        dest: /etc/nginx/sites-available/default
    - name: enable configuration
      file:
        dest: /etc/nginx/sites-enabled/default
        src: /etc/nginx/sites-available/default
        state: link
```

Puis rejouez le playbook : `ansible-playbook debian-nginx.yaml`

Sortie attendue (à ceci près que vous n'aurez pas la tâche "[disable bullseye-updates and bullseye-backports") :

```
vagrant@ansible-host:~$ ansible-playbook debian-nginx.yaml 

PLAY [Configure webserver with nginx] ***************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************
ok: [192.168.56.2]

TASK [disable bullseye-updates and bullseye-backports] **********************************************************************
ok: [192.168.56.2]

TASK [install nginx] ********************************************************************************************************
ok: [192.168.56.2]

TASK [copy nginx config file] ***********************************************************************************************
changed: [192.168.56.2]

TASK [enable configuration] *************************************************************************************************
ok: [192.168.56.2]

PLAY RECAP ******************************************************************************************************************
192.168.56.2               : ok=5    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

```

Éditez encore (une dernière fois) `debian-nginx.yaml` et ajoutez ces deux tâches à la fin (attention, ici on a fait une petite modif par rapport à l'original du tutoriel) :

```
    - name: copy index.html
      template:
        src: templates/index.html
        dest: /usr/share/nginx/html/index.html
        mode: 0644
    - name: restart nginx
      service:
        name: nginx
        state: restarted
```

Rejouez le playbook :

```
vagrant@ansible-host:~$ ansible-playbook debian-nginx.yaml 

PLAY [Configure webserver with nginx] ***************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************
ok: [192.168.56.2]

TASK [disable bullseye-updates and bullseye-backports] **********************************************************************
ok: [192.168.56.2]

TASK [install nginx] ********************************************************************************************************
ok: [192.168.56.2]

TASK [copy nginx config file] ***********************************************************************************************
ok: [192.168.56.2]

TASK [enable configuration] *************************************************************************************************
ok: [192.168.56.2]

TASK [copy index.html] ******************************************************************************************************
changed: [192.168.56.2]

TASK [restart nginx] ********************************************************************************************************
changed: [192.168.56.2]

PLAY RECAP ******************************************************************************************************************
192.168.56.2               : ok=7    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

```

On peut noter que les tâches déjà exécutées lors de l'exécution précédente ne provoquent cette fois pas de changement.

#### Requêter notre serveur web

Il va vous falloir installer `curl` sur l'`ansible-host` :

    sudo su -
    apt install -y curl
    logout

Puis vous pouvez interroger le serveur via son IP :

    curl http://192.168.56.10

Vous devriez voir s'afficher :

```
vagrant@ansible-host:~$ curl http://192.168.56.10
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Nginx on Debian</title>
</head>
<body>
  <h1>It works!</h1>
  <p>This is <code>index.html</code>, served by Nginx.</p>
</body>
</html>vagrant@ansible-host:~$ 
```

C'est le contenu du fichier `index.html` qui remplace celui livré par défaut avec Nginx.

#### Playbook pour installer NodeJS

Créer un dossier `nodejs` et s'y placer :

```
mkdir nodejs
cd nodejs
```

Editer un nouveau playbook : `nano debian-nodejs.yaml` avec ce contenu :

```
---
- name: Configure webserver with nginx
  hosts: webservers
  become: True
  become_method: sudo
  vars:
    node_apps_location: /usr/local/opt/node
  tasks:
     - name: install nodejs
       apt:
         name: nodejs
         update_cache: yes
```

Ce fichier introduit une nouvelle notion, celle de variables (qu'on va utiliser un peu plus loin).

Exécuter le playbook : `ansible-playbook debian-nodejs.yaml`

Créer un nouveau sous-dossier `app` :

```
mkdir app
```

Editer un fichier contenant une app Node.js/Express : `nano app/index.js` avec ce contenu (récupéré du "Getting Started" de la doc ExpressJS) :

```javascript
const express = require('express')
const app = express()
const port = 3000

app.get('/', (req, res) => {
  res.send('Hello World!')
})

app.listen(port, () => {
  console.log(`Example app listening at http://localhost:${port}`)
})

```

Editer le `package.json` pour cette app : `nano app/package.json, avec ce contenu :

```json
{
  "name": "Express app",
  "dependencies": {
    "express": "^4.17.0"
  }
}
```

Ajouter ce bloc de tâches à la fin de `debian-nodejs.yaml`, récupéré depuis le tutoriel [Déployer un serveur Node.js sur CentOS](https://iac.goffinet.org/ansible-linux/deployer-un-serveur-nodejs-centos/) :

```
    - name: Ensure Node.js app folder exists.
      file:
        path: "{{ node_apps_location }}"
        state: directory
    - name: Copy example Node.js app to server.
      copy:
        src: app
        dest: "{{ node_apps_location }}"
    - name: Install app dependencies defined in package.json.
      npm:
        path: "{{ node_apps_location }}/app"
```

Ces trois nouvelles tâches permettent :

* De vérifier si le dossier `/usr/local/opt/node` (valeur de la variable `node_apps_location`) existe, et de le créer si nécessaire
* De copier tout le dossier `app` vers ce dossier
* D'installer les dépendances avec NPM, en se plaçant dans ce dossier

Relancer le playbook : `ansible-playbook debian-nodejs.yaml`

* ajouter package.json
* installer npm (transformer valeur derrière name en tableau)
* Façon naive de lancer node

    - name: Start example Node.js app.
      command: "node {{ node_apps_location }}/app/index.j>

marche pas : voir post stack overflow 
--> tentative avec forever (marche pas non plus, probablement deprecation de forever_list)
--> marche avec pm2 (suppression avant relancement)
--> `curl http://192.168.56.10:3000/` à la fin

Attention le pm2 delete plante, que ce soit `pm2 delete node-app` ou `pm2 delete all`


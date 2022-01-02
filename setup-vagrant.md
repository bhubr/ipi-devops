# TP Ansible

## Qu'est-ce qu'Ansible ?

Ansible est un outil permettant d'automatiser toutes sortes de tâches sur des serveurs.

À partir d'un noeud appelé _control node_, on contrôle des serveurs appelés _managed nodes_.

On va, par exemple, pouvoir lancer la même commande sur tous les _managed nodes_, à partir du _control node_.

L'intérêt est d'automatiser la configuration de serveurs, ce qui offre un gain de temps d'autant plus appréciable que la "flotte" de machines à configurer est conséquente.

## Ansible sous Windows ?

Ansible est un outil orienté Unix. De fait, le _control node_ doit impérativement être un système Un*x ("Unix-like") tel que Linux, FreeBSD, MacOS, etc. Le portage du contrôleur Ansible sous Windows est, de l'aveu de ses développeurs, une tâche colossale, qui ne sera vraisemblablement pas réalisée avant quelques années.

Afin de pouvoir travailler avec Ansible sous Windows, nous allons donc utiliser des machines virtuelles : une pour le _control node_, et au moins une seconde, voire plusieurs, pour les _managed nodes_.

## VirtualBox ?

Une approche possible est d'utiliser VirtualBox, en préparant manuellement des machines virtuelles. Cela implique entre autres de paramétrer :

* la quantité de RAM
* un disque dur virtuel
* une ou plusieurs interfaces réseau

Il faut également télécharger une "image ISO" d'une distribution Linux (Debian, CentOS, openSUSE, etc.) qu'il faudra installer sur le disque dur virtuel de la VM, comme on le ferait sur une machine physique.

Cette procédure est assez rapide quand on en a l'habitude, et une fois la première VM ainsi configurée, il est facile de la cloner pour éviter la duplication d'efforts. Cependant, il est tout aussi facile de faire une erreur au cours de la procédure, quand on débute !

## Vagrant

Vagrant permet de configurer très facilement des machines virtuelles, à partir d'un fichier de configuration, le `Vagrantfile`.

Vagrant repose par défaut sur VirtualBox, mais offre, entre autres avantages, la simplicité et rapidité de mise en oeuvre : au lieu de devoir installer soi-même un OS dans une VM, on peut démarrer une VM à partir d'une "box" pré-configurée, téléchargée automatiquement par Vagrant depuis un dépôt central.

# Installation et configuration de Vagrant

* Télécharger la version **64-bit** depuis [la page officielle de téléchargements](https://www.vagrantup.com/downloads)
* Lancer l'installeur (5 étapes, garder les choix proposés par défaut)

## Premier test de Vagrant

Juste pour verifier que tout fonctionne correctement !

* Lancer PowerShell
* Se rendre dans Documents : `cd ~\Documents`
* Creer un dossier `DevOps` : `mkdir DevOps`
* Puis creer un sous-dossier `TestDebian` : `mkdir DevOps\TestDebian`
* Aller sous ce dossier : `cd DevOps\TestDebian`
* Initialiser une machine virtuelle Debian (version 11 nom de code "bullseye") : `vagrant init debian/bullseye64`
* La commande précédente a généré un fichier `Vagrantfile`, qu'on va garder tel quel
* Puis demarrer la VM : `vagrant up`
* Puis se connecter dans la VM : `vagrant ssh`
* Si tout a bien fonctionné : quitter la session SSH en saisissant `logout`

## Vagrantfile pour configurer plusieurs VMs


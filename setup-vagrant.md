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

## Installation et configuration de Vagrant

* Télécharger la version **64-bit** depuis [la page officielle de téléchargements](https://www.vagrantup.com/downloads)
* Lancer l'installeur (5 étapes, garder les choix proposés par défaut)

## Premier test de Vagrant

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

## Vagrantfile pour configurer plusieurs VMs

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
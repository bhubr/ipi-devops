# IMDW280/DevOps - Ansible - Utilisation avec VirtualBox

Ce document complète le support principal sur Ansible.

Lors de la journée du 3 janvier, certains d'entre vous ont rencontré des problèmes avec Vagrant.

Voici donc une alternative pour VirtualBox.


## Télécharger

Vous pouvez télécharger un "package" de 2 machines virtuelles, au format OVA, en suivant [ce lien](https://m1cp.bhu.ovh/downloads/DebianAnsibleAppliance.ova).

## Importer dans VirtualBox

* Téléchargez et installez [VirtualBox](https://www.virtualbox.org/) si ce n'est déjà fait
* Une fois VirtualBox lancé : menu Fichier puis choisir _Importer un appareil virtuel_ et parcourir l'ordinateur pour choisir `DebianAnsibleAppliance.ova`.
* Dans l'écran "Paramètres de l'appareil virtuel" qui suit :

    * optionnellement, dans "Politique d'adresse MAC", choisir "Générer de nouvelles adresses MAC pour toutes les interfaces réseau" (mais cela n'est pas censé avoir une grande importance !)
    * garder cochée la checkbox "Importer les disques durs comme VDI"
* Cliquer "Importer" &rarr; Suivant la vitesse de votre disque dur ou SSD, le processus peut prendre un peu de temps.
* À l'issue de l'import, vous verrez **deux nouvelles VMs** dans la liste :

    * _Debian 11 - Ansible Controller_ où sera installé `ansible`
    * _Debian 11 - Ansible Managed Host_, la machine qui sera gérée depuis le contrôleur

## Démarrer les VMs et s'y connecter

### Démarrer

Puisque ces VMs sont destinées à être utilisées de concert, autant les démarrer en même temps.

**Tips** : dans la liste des VMs de VirtualBox, gardez la touche `Ctrl` enfoncée, et cliquez successivement sur les deux VM. Puis cliquez sur **Démarrer**. VirtualBox vous posera la question : "Vous êtes sur le point de démarrer toutes les machines virtuelles suivantes". Cliquez simplement sur OK.

### Se connecter directement dans les fenêtres

Vous verrez apparaître une fenêtre par VM. Après le démarrage, vous arriverez sur l'écran de login.

Par souci de simplicité, les VMs ont été configurées avec des mots de passe simples, faciles à mémoriser.

Il y a deux comptes utilisateurs :

* `debian` (compte utilisateur non-privilégié)
* `root` (compte administrateur)

Les deux ont pour mot de passe `Debian11`. Vous pouvez tester la connexion avec un des deux comptes.

> **Se connecter dans les fenêtres de cette façon présente un inconvénient** : on ne peut pas faire de copier-coller. Cela peut être résolu en installant des extensions de VirtualBox. Mais de mémoire, cela ne fonctionne qu'en environnement graphique.

On va utiliser une autre façon de se connecter aux VMs : via SSH. Ceci présente aussi l'intérêt de montrer un cas d'usage proche de la connexion vers un véritable serveur distant.

### Client SSH

SSH est une application client-serveur. Le serveur `sshd` a été pré-installé sur les VMs lors de l'installation (voir Annexe).

Il faut disposer d'un client sur l'hôte Windows. Pour cela, deux possibilités :

* Installer [Git Bash](https://gitforwindows.org/) (le package Git pour Windows inclut Bash, qui fournit un environnement Unix-like avec toutes les commandes usuelles dont `ssh`)
* Activer le client SSH de Windows 10 :

    * Menu Démarrer
    * Paramètres
    * Applications
    * Cliquer sur "Gérer les fonctionnalités facultatives"
    * Saisir "ssh" dans le champ de recherche
    * Choisir "OpenSSH Client" puis installer

Quelle que soit la méthode choisie, vous pourrez saisir la commande `ssh` (dans Git Bash pour la 1ère méthode, dans PowerShell pour la seconde).

### Utiliser le client SSH

Les machines virtuelles ont chacune deux interfaces réseau :

* La 1ère, en mode NAT, permet à la VM d'accéder à Internet
* La 2nde est en réseau interne

Par défaut, ces interfaces réseau sont bien séparées du réseau de la machine hôte. On aurait pu configurer une des interfaces en mode "Pont" (_Bridge_), mais d'expérience, cela peut poser problème sur un réseau d'entreprise ou universitaire.

On a donc configuré des _redirections de port_ sur l'interface NAT des VM.

* Sur la VM _Ansible Controller_, la redirection est faite du port **22122** de l'hôte vers le port 22 de la VM (port standard pour les serveurs SSH)
* Sur la VM _Ansible Managed Host_, la redirection est faite du port **22132** de l'hôte vers le port 22 de la VM

Pour se connecter aux VMs, on va donc utiliser `ssh` en spécifiant un port suivant la machine à laquelle on veut se connecter.

> **Tips** : vous allez, dans un premier temps, avoir besoin de vous connecter aux 2 VMs en même temps. Si vous souhaitez éviter de jongler entre de trop nombreuses fenêtres, vous pouvez installer [Windows Terminal](https://www.microsoft.com/fr-fr/p/windows-terminal/9n0dx20hk701#activetab=pivot:overviewtab), qui permet d'avoir plusieurs onglets dans une même fenêtre.

Connectez-vous à la première VM :

```
ssh -p 22122 debian@localhost
```

Saisissez le mot de passe `Debian11`.

> :warning: Vous ne pouvez pas vous connecter en `root` via SSH (bien qu'il soit possible de configurer le serveur SSH pour autoriser cela, ce n'est pas une bonne idée du point de vue de la sécurité).

Une fois connecté, vous devriez voir apparaître quelques lignes d'information sur l'OS Debian GNU/Linux, suivies d'une invite de commandes (_prompt_) :

```
debian@debian: $
```

Pour commencer, prenez les droits `root`, via la commande suivante (vous devrez à nouveau saisir le mot de passe `Debian11`) :

```
su -
```

Votre invite de commande devrait être modifié :

```
root@debian:~#
```

Installez la commande `sudo` :

```
apt install -y sudo
```

Cela vous permettra par la suite, si besoin, de lancer des commandes administratives depuis le compte `debian`, via :

```
sudo commande
```

Quittez le compte root via la commande `exit` ou le raccourci `Ctrl-D`. Fermez la connexion SSH, à nouveau via `exit` ou `Ctrl-D`.

**Répétez toutes les étapes précédentes pour la 2nde VM**, pour laquelle la commande de connexion est presque identique (port `22132` au lieu de `22122`) :

```
ssh -p 22132 debian@localhost
```

> **Note** : la commande `sudo` est pré-installée dans les VMs Vagrant. Ce n'est pas le cas avec une Debian "fraîchement installée" comme celles qui vous sont fournies. Cette commande est **particulièrement importante** du côté de la machine "Managed Host" : si elle n'est pas installée, le contrôleur Ansible ne pourra pas lancer des commandes requérant des privilèges administratifs (via le paramètre `become`).

**À ce stade**, vous êtes presque rendu au point de départ qui vous permettra de travailler comme vous l'auriez fait avec Vagrant.

La principale différence est que vous utiliserez le compte `debian` au lieu de `vagrant`.

## Générer les clés SSH

> La section ci-dessous résume la section [Génération d'une paire de clés SSH](https://github.com/bhubr/ipi-devops/blob/master/ansible-on-vagrant.md#g%C3%A9n%C3%A9ration-dune-paire-de-cl%C3%A9s-ssh) du support principal.

Connectez-vous à nouveau à la première VM (_Controller_) :

```
ssh -p 22122 debian@localhost
```

Générez une paire de clés SSH :

```
ssh-keygen -t ecdsa
```

Validez le choix d'emplacement par défaut (`/home/debian/.ssh/id_ecdsa`) puis, lors de la saisie de la passphrase, saisissez entrée deux fois pour utiliser une passphrase vide (non-sécurisé mais plus simple).

Contrairement à l'utilisation avec Vagrant, vous allez pouvoir directement transférer la **clé publique** vers l'autre VM via la commande `ssh-copy-id`.

Vérifiez d'abord la connectivité réseau via la commande `ping 192.168.56.10` (`192.168.56.10` est l'adresse IP de la VM _Managed Host_ sur le réseau local). Si tout se passe bien, vous devriez voir s'afficher des lignes semblables à celles-ci :

```
PING 192.168.56.10 (192.168.56.10) 56(84) bytes of data.
64 bytes from 192.168.56.10: icmp_seq=1 ttl=64 time=0.667 ms
64 bytes from 192.168.56.10: icmp_seq=2 ttl=64 time=0.872 ms
64 bytes from 192.168.56.10: icmp_seq=3 ttl=64 time=0.662 ms
```

**Interrompez** le ping avec `Ctrl-C`. Puis lancez :

```
ssh-copy-id -i ~/.ssh/id_ecdsa.pub debian@192.168.56.10
```

Vous obtiendrez cet affichage :

```
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/debian/.ssh/id_ecdsa.pub"
The authenticity of host '192.168.56.10 (192.168.56.10)' can't be established.
ECDSA key fingerprint is SHA256:FA7rAeWO0yX4KaAPOwcUQSeSYq4udMy1HFKUZpAC+yk.
Are you sure you want to continue connecting (yes/no/[fingerprint])? 
```

On vous indique qu'on ne peut pas vérifier l'authenticité de l'hôte distant, puis on vous donne une "empreinte" (_fingerprint_). Vous pouvez ici ignorer cet avertissement en saisissant `yes`.

> **Dans un environnement plus réaliste**, c'est une bonne pratique de demander à l'administrateur des machines de votre réseau de vous communiquer la _fingerprint_ du serveur auquel vous vous connectez, et de la comparer à celle affichée ci-dessus. Cette vérification a pour but de prévenir certaines attaques ("man-in-the-middle").

Après avoir saisi `yes`, vous devriez voir apparaître ces lignes :

```
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
```

Suivies de l'invitation à saisir le mot de passe (toujours `Debian11`). Après quoi, vous devriez obtenir ce message, indiquant que la clé publique a bien été transférée vers la 2nde VM :

```
Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'debian@192.168.56.10'"
and check to make sure that only the key(s) you wanted were added.
```

Vous pouvez désormais vous connecter **sans mot de passe** (ni passphrase car on a mis une passphrase vide lors de la génération de la clé) :

```
ssh debian@192.168.56.10
```

> Vous êtes désormais prêt(e) pour la suite : installation d'Ansible sur le contrôleur, etc.

## Annexe - comment ont été créées les VMs

Voici, en résumé, comment on a créé les VMs :

* Téléchargement de l'[image ISO minimale](https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/debian-11.2.0-amd64-netinst.iso) de Debian 11.2 (permet d'installer, via le réseau, uniquement les packages sélectionnés)
* Dans VirtualBox : nouvelle machine virtuelle
* RAM : choisir 1024 Mo (compromis entre la capacité de vos ordinateurs, et les besoins de Debian / Ansible)
* Stockage : ajouter un disque optique virtuel, choisir le fichier `debian-11.2.0-amd64-netinst.iso` téléchargé précédemment
* Démarrer la VM
* Choisir la 2ème option du menu (installation non-graphique)
* Choisir les paramètres de : langue, locale, zone géographique, clavier (j'ai choisi Anglais, EN_US, Europe/France, Français)
* Partitionner le disque (prendre les premières options : disque entier, tout sur une même partition)
* Choix du miroir : choisir un miroir proche de chez soi (j'ai choisi `univ-tlse2.fr`)
* Au moment du choix des packages :

    * **Décocher** _Desktop Environment_ et _GNOME_ (les 2 premières options)
    * **Cocher** _SSH Server_ et _Core system utilities_ (les 2 dernières)
* Une fois l'installation presque terminée, on vous demande s'il faut installer GRUB : choisir `/dev/sdb` (de mémoire, mais il n'y a qu'un choix normalement)

Après la création des VMs, on a paramétré leur réseau :

**VMs éteintes**, on est allé dans leurs paramètres, puis sous réseau :

* 1ère interface en mode **NAT** puis Avancé puis redirection de ports. On a ajouté une redirection du port **22122** vers le port 22 (VM 1), du port **22132** vers le port 22 (VM 2). Ceci afin de pouvoir se connecter en SSH de la machine hôte vers chaque VM.
* On a **activé** la 2ème interface en mode _internal network_ (réseau interne). Ces interfaces seront configurées en IP statique, pour former un LAN entre les VMs.

**VM rallumées**, on s'est connecté à chaque VM en `root` puis on a lancé la commande `ip -c link show`, afin d'afficher les interfaces réseau :

```
root@debian:~#  ip -c link show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 08:00:27:0a:61:50 brd ff:ff:ff:ff:ff:ff
3: enp0s8: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 08:00:27:a2:ee:e8 brd ff:ff:ff:ff:ff:ff
```

On a une interface de "loopback" (celle qui permet à une machine de se référer à elle-même via l'IP `127.0.0.1` ou le hostname `localhost`), et deux interfaces réseau (ici `enp0s3` et `enp0s8`), correspondant aux deux interfaces configurées dans les paramètres réseau de VirtualBox, pour chaque VM.

On a ensuite créé un fichier `/etc/network/interfaces.d/enp0s8` (le nom importe peu) et y a mis le contenu suivant (attention, IP différentes pour les deux VM).

Première VM (Controller) :

```
auto enp0s8
iface enp0s8 inet static
 address 192.168.56.1
 netmask 255.255.255.0
```

Seconde VM (Managed host) :

```
auto enp0s8
iface enp0s8 inet static
 address 192.168.56.10
 netmask 255.255.255.0
```

On a ensuite **rebooté** les VM.
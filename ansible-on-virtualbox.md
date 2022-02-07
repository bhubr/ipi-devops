# IMDW280/DevOps - Ansible

Ce document complète le support principal sur Ansible.

## Utilisation avec VirtualBox

Lors de la journée du 3 janvier, certains d'entre vous ont rencontré des problèmes avec Vagrant.

Voici donc une alternative pour VirtualBox.

Vous pouvez télécharger un "package" de 2 machines virtuelles, au format OVA, en suivant [ce lien](https://m1cp.bhu.ovh/downloads/DebianAnsibleAppliance.ova).

### Import dans VirtualBox

* Installer et télécharger [VirtualBox](https://www.virtualbox.org/) si ce n'est déjà fait
* Une fois VirtualBox lancé : menu Fichier puis choisir _Importer un appareil virtuel_ et parcourir l'ordinateur pour choisir `DebianAnsibleAppliance.ova`.
* Dans l'écran "Paramètres de l'appareil virtuel" qui suit :

    * optionnellement, dans "Politique d'adresse MAC", choisir "Générer de nouvelles adresses MAC pour toutes les interfaces réseau" (mais cela n'est pas censé avoir une grande importance !)
    * garder cochée la checkbox "Importer les disques durs comme VDI"
* Cliquer "Importer" &raquo; Suivant la vitesse de votre disque dur ou SSD, le processus peut prendre un peu de temps.
* À l'issue de l'import, vous verrez **deux nouvelles VMs** dans la liste :

    * _Debian 11 - Ansible Controller_ où sera installé `ansible`
    * _Debian 11 - Ansible Managed Host_, la machine qui sera gérée depuis le contrôleur

### Démarrer les VMs et s'y connecter

#### Démarrer

Puisque ces VMs sont destinées à être utilisées de concert, autant les démarrer en même temps.

**Tips** : dans la liste des VMs de VirtualBox, gardez la touche `Ctrl` enfoncée, et cliquez successivement sur les deux VM. Puis cliquez sur **Démarrer**. VirtualBox vous posera la question : "Vous êtes sur le point de démarrer toutes les machines virtuelles suivantes". Cliquez simplement sur OK.

#### Se connecter directement dans les fenêtres

Vous verrez apparaître une fenêtre par VM. Après le démarrage, vous arriverez sur l'écran de login.

Par souci de simplicité, les VMs ont été configurées avec des mots de passe simples, faciles à mémoriser.

Il y a deux comptes utilisateurs :

* `debian` (compte utilisateur non-privilégié)
* `root` (compte administrateur)

Les deux ont pour mot de passe `Debian11`. Vous pouvez tester la connexion avec un des deux comptes.

> **Se connecter dans les fenêtres de cette façon présente un inconvénient** : on ne peut pas faire de copier-coller. Cela peut être résolu en installant des extensions de VirtualBox. Mais de mémoire, cela ne fonctionne qu'en environnement graphique.

On va utiliser une autre façon de se connecter aux VMs : via SSH. Ceci présente aussi l'intérêt de montrer un cas d'usage proche de la connexion vers un véritable serveur distant.

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
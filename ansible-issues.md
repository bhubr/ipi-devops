# Problèmes rencontrés avec Ansible

## Connexion via SSH

On peut avoir besoin de spécifier un certain nombre de paramètres de connexion pour une machine répertoriée dans l'_inventory_ :

* Port d'écoute du daemon SSH
* Compte utilisateur
* Clé privée

Exemple (ici pour se connecter à une VM avec NAT et port forwarding) :

```
[hosts]
localhost ansible_user=johndoe ansible_port=2232 ansible_ssh_private_key_file=ecdsa_deb_vm_ansible
```

On aura préalablement copié la clé publique SSH sur le compte concerné dans la VM.

```
ssh-copy-id -p 2232 -i  ecdsa_deb_vm_ansible johndoe@localhost
```

## `sudo: not found`

Solution : installer manuellement `sudo` sur les machines cibles.

D'abord, installer et activer `chrony`, sans quoi le `apt update` peut échouer avec certains mirroirs.

```
apt install chrony
systemctl enable chrony
systemctl start chrony
```

Update de la base et des packages :

```
apt update
apt upgrade
```

Installation de `etckeeper` (permet de tracer les modifications de `/etc` via Git) :

```
apt install etckeeper
```

Installation de `sudo` :

```
apt install sudo
```

Ajout au groupe `sudo` de l'utilisateur régulier spécifié pour Ansible (ici `johndoe`) :

```
usermod -aG sudo johndoe
```

## `Missing sudo password`

Après avoir installé `sudo`, un autre problème&hellip;

Plusieurs solutions sont proposées dans ce post : [Missing sudo password in Ansible](https://stackoverflow.com/q/25582740).

* Éditer `/etc/sudoers`
* Passer `-K` (`--ask-become-pass`) à l'invocation d'`ansible-playbook`


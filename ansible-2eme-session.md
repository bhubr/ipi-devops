# IMDW280/DevOps - Ansible - 2ème session

## Setup

### Git

On va reprendre des choses déjà faites la dernière fois.

**Avant tout**, un peu de mise en place.

Vous devez installer Git. Pour cela, deux possiblités.

La première option est [Git for Windows](https://gitforwindows.org/), qui vous installe au passage Git Bash. L'installeur est à télécharger sur le site.

La seconde est [Chocolatey](https://chocolatey.org/install), un gestionnaire de packages pour Windows. Pour l'installer :

```
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
```

Si cela ne marche pas, lancez `Set-ExecutionPolicy AllSigned` puis ré-essayez.

Vous pourrez ensuite installer Git :

```
choco install git
```

(répondre a pour _all_ quand il vous demande des autorisations).

**Ensuite**, on va faire en sorte de pouvoir éditer les fichiers de config Ansible directement dans les VM.

### [Vagrant] Répertoire partagé et adresses IP

**Seulement** pour ceux qui utilisent Vagrant.

Retrouvez le dossier où est stocké ce qu'on avait fait la dernière fois avec Vagrant. Ce dossier doit contenir un `Vagrantfile`. **Ouvrez tout le dossier dans votre IDE**.

Il faut ajouter cette ligne au bon endroit (regardez [la version en ligne](https://github.com/bhubr/ipi-devops/blob/a9a608b69ea2dc1467f1cf57147cbf2415e3d9a8/vagrant-multi-vms/Vagrantfile#L37-L38)) pour partager un dossier entre Windows et les VMs :

```
    config.vm.synced_folder "shared/", "/shared"
```

**Créez le dossier `shared` dans le même dossier** que le `Vagrantfile`.

Vérifiez également que l'adresse IP de la 1ère machine est bien `192.168.56.10`.

Ensuite : `vagrant up` pour démarrer les VMs.

**Une fois démarrées** : `vagrant ssh ansible-host` pour vous connecter à la machine qui fait office de contrôleur Ansible.

### [VirtualBox] Connexion à distance

On pourrait configurer un dossier partagé, comme pour ceux qui utilisent Vagrant, mais il est aussi simple de se connecter à la VM depuis VSCode en SSH.

Il faut pour cela installer l'extension _Remote - SSH_. Puis utiliser le raccourci Ctrl-Shift-P, saisir "ssh" dans le champ de recherche puis choisir "Connect to Host". On peut alors saisir une commande pour se connecter (il faut avoir démarré les VMs) :

```
ssh -p 22122 debian@localhost
```


### [TOUS] Vérification de l'inventaire et première commande

Vérifiez le contenu du fichier d'_inventaire_ d'Ansible, qu'on avait édité la dernière fois (on filtre pour ne voir que les IP commençant par 192) :

```
grep 192 /etc/ansible/hosts
```

Vous devriez voir une ligne contenant l'IP `192.168.56.10`. Sinon, éditer le fichier en lançant, en étant connecté en SSH dans la VM :

```
sudo nano /etc/ansible/hosts
```

Et ajouter :

```
[webservers]
192.168.56.10
```

Puis Ctrl-O pour sauvegarder (entrée pour valider le nom de fichier).

**Enfin**, on va vérifier qu'Ansible peut se connecter à la machine qu'il doit contrôler.

```
ansible all -m ping
````

Qui doit répondre :

```
192.168.56.10 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```
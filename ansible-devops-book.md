# Livre _Ansible for DevOps_

Notes de lecture et parcours pas à pas.

## Chapitre 3 - _Ad-Hoc Commands_

### Préparation

Lancement de différentes commandes pour récupérer l'hostname, l'espace disque et la RAM disponibles

    ansible multi -a "hostname"
    ansible multi -a "df -h"
    ansible multi -a "free -m"

Vérifier la date courante sur tous les serveurs

    ansible multi -a "date"

Installer `chrony` et lancer le service (modifié pour utiliser `apt`)

    ansible multi -b -m apt -a "name=chrony state=present"
    ansible multi -b -m service -a "name=chronyd state=started enabled=yes"

Vérifier la sync des serveurs avec le time server :

    ansible multi -b -a "chronyc tracking"

> :warning: Les serveurs ne sont pas sync (20 sec entre 4 et 6, idem entre 6 et 5, 40 sec entre 4 et 5)


### Config serveurs app

    ansible app -b -m apt -a "name=python3-pip state=present"

FAIL !

**IDEM CI-DESSOUS**. Essayons une update ([doc module apt](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/apt_module.html))

    ansible app -b -m apt -a "update_cache=yes"

Django :

    ansible app -b -m pip -a "name=django<4 state=present"

Test run Django :

    ansible app -a "python3 -m django --version"

### Config serveur DB

#### Install MariaDB et activation du service

    ansible db -b -m apt -a "name=mariadb-server state=present"

FAIL !

    benoit@Rocinante:~/DevOps/ansible-for-devops/orchestration$ ansible db -b -m apt -a "name=mariadb-server state=present"
    192.168.60.6 | FAILED! => {
        "changed": false,
        "msg": "No package matching 'mariadb-server' is available"
    }

Essayons une update ([doc module apt](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/apt_module.html))

    ansible db -b -m apt -a "update_cache=yes"

On relance `ansible db -b -m apt -a "name=mariadb-server state=present"` &rarr; SUCCESS :fire:

Lancer service :

    ansible db -b -m service -a "name=mariadb state=started enabled=yes"

#### Config firewall

Installer et activer le service :

    ansible db -b -m apt -a "name=firewalld state=present"
    ansible db -b -m service -a "name=firewalld state=started enabled=yes"

Config firewall :


Vu que je ne connais pas firewalld : [firewalld basics](https://www.putorius.net/introduction-to-firewalld-basics.html#listing-all-zones-in-firewalld), où il est écrit "Firewalld is basically just a wrapper for iptables". 

Avant de créer (?) la zone `database`, on fait `vagrant ssh db` puis (_listing all zones in firewalld_) `firewall-cmd --get-zones`, qui donne :

    vagrant@orc-db:~$ firewall-cmd --get-zones
    block dmz drop external home internal public trusted work

On lance alors

    ansible db -b -m firewalld -a "zone=database state=present permanent=yes"

Si on relance `firewall-cmd --get-zones`, on ne voit aucun changement :/.

Autorisations :

    ansible db -b -m firewalld -a "source=192.168.60.0/24 zone=database state=enabled permanent=yes"
    ansible db -b -m firewalld -a "port=3306/tcp zone=database state=enabled permanent=yes"

#### Install dépendances Python

Install PyMySQL

    ansible db -b -m apt -a "name=python3-pymysql state=present"

Création user MySQL &rarr; FAIL au 1er coup car ... access denied pour root !

    ansible db -b -m mysql_user -a "name=django host=% password=12345 priv=*.*:ALL state=present"

RESOLU

    ansible db -b -m mysql_user -a "login_unix_socket=/var/run/mysqld/mysqld.sock name=django host=% password=12345 priv=*.*:ALL state=present"


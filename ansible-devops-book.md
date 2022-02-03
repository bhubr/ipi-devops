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

### Config serveur DB

    ansible db -b -m apt -a "name=mariadb-server state=present"

FAIL !

    benoit@Rocinante:~/DevOps/ansible-for-devops/orchestration$ ansible db -b -m apt -a "name=mariadb-server state=present"
    192.168.60.6 | FAILED! => {
        "changed": false,
        "msg": "No package matching 'mariadb-server' is available"
    }

Essayons une update ([doc module apt](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/apt_module.html))

    ansible db -b -m apt -a "update_cache=yes"

On relance `ansible db -b -m apt -a "name=mariadb-server state=present"` &rarr; SUCCESS :party:
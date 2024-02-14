## Erreurs rencontrées

Run playbook:

```
ansible-playbook -i inventory/hosts playbooks/install_php_mysql.yml -K
```

### MariaDB déjà démarré

```
TASK [MariaDB setup] **********************************************************************************************************************
fatal: [192.168.1.85]: FAILED! => {"changed": true, "cmd": ["/etc/init.d/mariadb", "setup"], "delta": "0:00:00.009031", "end": "2024-02-14 16:27:03.174459", "msg": "non-zero return code", "rc": 1, "start": "2024-02-14 16:27:03.165428", "stderr": " * mariadb: cannot `setup' as it has not been stopped", "stderr_lines": [" * mariadb: cannot `setup' as it has not been stopped"], "stdout": "", "stdout_lines": []}
```

Résolu en vérifiant si MariaDB est déjà up & running.


### Module manquant pour mysql_db

```
ASK [create MySQL database] **************************************************************************************************************
fatal: [192.168.1.85]: FAILED! => {"changed": false, "msg": "A MySQL module is required: for Python 2.7 either PyMySQL, or MySQL-python, or for Python 3.X mysqlclient or PyMySQL. Consider setting ansible_python_interpreter to use the intended Python version."}
```

Pas mal de modules à installer

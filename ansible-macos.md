# Ansible install

## Install with stock Python on MacOS (10.14.x Mojave)

Mojave ships with Python 2.7.

First get and install `pip`:

    curl https://bootstrap.pypa.io/pip/2.7/get-pip.py -o get-pip.py
    python get-pip.py --user

Then install Ansible

    python -m pip install --user ansible

**Not a good idea, better use Python 3.x** than the default Python 2.7.

## Edit `PATH`

Add this line to `~/.profile`:

    export PATH="$HOME/Library/Python/2.7/bin:$PATH"

## Create a user-specific Ansible config file

Not sure whether we should use default `/etc/ansible/hosts` file.

Therefore, create `~/.ansible.cfg` containing:

    [defaults]
    inventory = /Users/whoever/ansible/hosts  ; replace whoever with MacOS username

## In the `~/ansible/hosts` file

e.g. to connect to a Debian VM via port-forwarding from 2232 (host) to 22 (guest).

    localhost:2232

## Not working

    MBP-de-Benoit-2:~ benoit$ ansible all -m ping
    [DEPRECATION WARNING]: Ansible will require Python 3.8 or newer on the controller starting with Ansible 2.12. Current version: 
    2.7.16 (default, Mar 25 2021, 18:52:10) [GCC 4.2.1 Compatible Apple LLVM 10.0.1 (clang-1001.0.37.14)]. This feature will be 
    removed from ansible-core in version 2.12. Deprecation warnings can be disabled by setting deprecation_warnings=False in 
    ansible.cfg.
    /Users/benoit/Library/Python/2.7/lib/python/site-packages/ansible/parsing/vault/__init__.py:44: CryptographyDeprecationWarning: Python 2 is no longer supported by the Python core team. Support for it is now deprecated in cryptography, and will be removed in the next release.
      from cryptography.exceptions import InvalidSignature
    localhost | UNREACHABLE! => {
        "changed": false, 
        "msg": "Failed to connect to the host via ssh: benoit@localhost: Permission denied (publickey,password).", 
        "unreachable": true
    }

## Change SSH username

See [How to set default Ansible username/password for SSH connection?](https://serverfault.com/questions/628989/how-to-set-default-ansible-username-password-for-ssh-connectionhttps://serverfault.com/questions/628989/how-to-set-default-ansible-username-password-for-ssh-connection)

Content of `~/ansible/hosts` after tweaking:

    [all:vars]
    ansible_connection=ssh
    ansible_user=johndoe
    ansible_ssh_pass=password

    [vms]
    localhost:2232

## Run after setting username

    MBP-de-Benoit-2:~ benoit$ ansible all -m ping
    [DEPRECATION WARNING]: Ansible will require Python 3.8 or newer on the controller starting with Ansible 2.12. Current version: 
    2.7.16 (default, Mar 25 2021, 18:52:10) [GCC 4.2.1 Compatible Apple LLVM 10.0.1 (clang-1001.0.37.14)]. This feature will be 
    removed from ansible-core in version 2.12. Deprecation warnings can be disabled by setting deprecation_warnings=False in 
    ansible.cfg.
    /Users/benoit/Library/Python/2.7/lib/python/site-packages/ansible/parsing/vault/__init__.py:44: CryptographyDeprecationWarning: Python 2 is no longer supported by the Python core team. Support for it is now deprecated in cryptography, and will be removed in the next release.
      from cryptography.exceptions import InvalidSignature
    localhost | FAILED! => {
        "msg": "to use the 'ssh' connection type with passwords, you must install the sshpass program"
    }

## Gen the SSH key

Generate an ECDSA key

    ssh-keygen -t ecdsa

When asked location, enter `/Users/benoit/.ssh/id_ecdsa_ansible` 

## Copy the key to the VM

    ssh-copy-id -i ~/.ssh/id_ecdsa_ansible.pub -p 2232 johndoe@localhost

Enter the password when asked

## Test key-based connection

    ssh -i ~/.ssh/id_ecdsa_ansible -p 2232 johndoe@localhost

## Amend inventory

Comment out or remove `ansible_ssh_pass` and add `ansible_ssh_private_key_file`:

    ansible_ssh_private_key_file=~/.ssh/id_ecdsa_ansible

## It f***ing works!

    MBP-de-Benoit-2:~ benoit$ ansible all -m ping
    [DEPRECATION WARNING]: Ansible will require Python 3.8 or newer on the controller starting with Ansible 2.12. Current 
    version: 2.7.16 (default, Mar 25 2021, 18:52:10) [GCC 4.2.1 Compatible Apple LLVM 10.0.1 (clang-1001.0.37.14)]. This feature 
    will be removed from ansible-core in version 2.12. Deprecation warnings can be disabled by setting deprecation_warnings=False
    in ansible.cfg.
    /Users/benoit/Library/Python/2.7/lib/python/site-packages/ansible/parsing/vault/__init__.py:44: CryptographyDeprecationWarning: Python 2 is no longer supported by the Python core team. Support for it is now deprecated in cryptography, and will be removed in the next release.
      from cryptography.exceptions import InvalidSignature
    localhost | SUCCESS => {
        "ansible_facts": {
            "discovered_interpreter_python": "/usr/bin/python3"
        }, 
        "changed": false, 
        "ping": "pong"
    }

## On the VM

Create ssh folder for `root`:

    sudo mkdir /root/.ssh
    sudo chown 700 /root/.ssh

Copy the `authorized_keys` to `root`'s `ssh` folder:

    sudo cp .ssh/authorized_keys /root/.ssh/

## Amend (again) the hosts file

Change user from `johndoe` to `root`.

    [all:vars]
    ansible_connection=ssh
    ansible_user=root
    # ansible_ssh_pass=Debian11
    ansible_ssh_private_key_file=~/.ssh/id_ecdsa_ansible

    [vms]
    localhost:2232

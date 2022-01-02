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

## In the `hosts` file

e.g. to connect to a Debian VM via port-forwarding from 2232 (host) to 22 (guest).

    localhost:2232
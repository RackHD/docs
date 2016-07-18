Ubuntu Source Based Installation
---------------------------------

We will leverage the ansible roles created for the RackHD demonstration environment.

.. code::

    cd ~
    sudo apt-get install git
    sudo apt-get update
    sudo apt-get dist-upgrade
    sudo reboot

    cd ~
    git clone https://github.com/rackhd/rackhd
    sudo apt-get install ansible
    cd ~/rackhd/packer/ansible
    ansible-playbook -i "local," -K -c local rackhd_local.yml

This created the default configuration file at /opt/monorail/config.json
from https://github.com/RackHD/RackHD/blob/master/packer/ansible/roles/monorail/files/config.json.
You may need to update this and /etc/dhcpd.conf to match your local network
configuration.

This will install all the relevant dependencies and code into ~/src, expecting
that it will be run with node-foreman.

.. code::

    cd ~
    sudo nf start


How to update to the latest code
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code::

    cd ~/src
    ./scripts/clean_all.bash && ./scripts/reset_submodules.bash && ./scripts/link_install_locally.bash

To reset the database of nodes
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code::

    echo "db.dropDatabase()" | mongo pxe

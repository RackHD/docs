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
that it will be run with `pm2`_.

.. _pm2: http://pm2.keymetrics.io/

.. code::

    cd ~
    sudo pm2 start rackhd-pm2-config.yml

Some useful commands of pm2:

.. code::

    sudo pm2 restart all           # restart all RackHD services
    sudo pm2 restart on-taskgraph  # restart the on-taskgraph service only.
    sudo pm2 logs                  # show the combined real-time log for all RackHD services
    sudo pm2 logs on-taskgraph     # show the on-taskgraph real-time log
    sudo pm2 flush                 # clean the RackHD logs
    sudo pm2 status                # show the status of RackHD services


How to update to the latest code
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code::

    cd ~/src
    ./scripts/clean_all.bash && ./scripts/reset_submodules.bash && ./scripts/link_install_locally.bash

To reset the database of nodes
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code::

    echo "db.dropDatabase()" | mongo pxe

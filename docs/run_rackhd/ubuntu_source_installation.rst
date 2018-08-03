Installation from Source Code
=============================

.. contents:: Table of Contents

Prerequisites
-----------------------------

NICs
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. tabs::

    .. tab:: Ubuntu 14.04

        Start with an Ubuntu trusty(14.04) instance with 2 nics:

        * ``eth0`` for the ``public`` network - providing access to RackHD APIs, and providing
          routed (layer3) access to out of band network for machines under management

        * ``eth1`` for dhcp/pxe to boot/configure the machines

        edit the network:

        * ``eth0`` - assign IP address as appropriate for the environment, or you can use DHCP

        * ``eth1`` static ( 172.31.128.0/22 )

        please check the network config file: ``/etc/network/interfaces``. The ``eth1``'s ip address is ``172.31.128.1`` Like as follows:

        .. code::

            auto eth1
            iface eth1 inet static
            address 172.31.128.1
            post-up ifconfig eth1 promisc

    .. tab:: Ubuntu 16.04

        Start with an Ubuntu xenial(16.04) instance with 2 nics:

        * ``ens160`` for the ``public`` network - providing access to RackHD APIs, and providing
          routed (layer3) access to out of band network for machines under management

        * ``ens192`` for dhcp/pxe to boot/configure the machines

        .. note::
            You might get different ethernet name from ens160/ens192 in your OS system. Please replace it with what you get accordingly. 

        Edit the network:

        * ``ens160`` - assign IP address as appropriate for the environment, or you can use DHCP

        * ``ens192`` static ( 172.31.128.0/22 )

        Please check the network config file: ``/etc/network/interfaces``. The ``ens192``'s ip address is ``172.31.128.1`` Like as follows:

        .. code::

            auto ens192
            iface ens192 inet static
            address 172.31.128.1
            post-up ifconfig ens192 promisc

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


Start RackHD
-----------------------------

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

Notes：isc-dhcp-server is installed through ansible playbook, but sometimes it won't start on Ubuntu boot (https://ubuntuforums.org/showthread.php?t=2068111),
check if DHCP service is started:

.. code::

    sudo service --status-all

If isc-dhcp-server is not running, run below to start DHCP service:

.. code::

    sudo service isc-dhcp-server start


How to update to the latest code
--------------------------------

.. code::

    cd ~/src
    ./scripts/clean_all.bash && ./scripts/reset_submodules.bash && ./scripts/link_install_locally.bash

How to Reset the Database
-----------------------------

.. code::

    echo "db.dropDatabase()" | mongo pxe

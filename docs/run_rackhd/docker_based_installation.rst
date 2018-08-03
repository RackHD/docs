Installation from Docker
================================

.. contents:: Table of Contents

Prerequisites
-----------------------------

**NICs**

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

Install Docker & Docker Compose
-------------------------------

+----------------------+---------------------------------------------------------+
|Install Docker CE     | https://docs.docker.com/install/#server                 |
+----------------------+---------------------------------------------------------+
|Install Docker Compose| https://docs.docker.com/compose/install/#install-compose|
+----------------------+---------------------------------------------------------+

Download Source Code
-----------------------------

.. code::

    git clone https://github.com/RackHD/RackHD

    cd RackHD/docker

    # for example if you are installing RackHD latest relesae:
    sudo TAG=latest docker-compose pull   # Download pre-built docker images
    sudo TAG=latest docker-compose up -d    # Create Containers and Run RackHD

For more information about tags please see https://hub.docker.com/r/rackhd/on-http/tags/

Check RackHD is running properly

.. code::

    cd RackHD/docker
    sudo docker-compose ps

    # example response
    #        Name                      Command               State    Ports
    # ---------------------------------------------------------------------
    # docker_core_1         /bin/echo exit                   Exit 0
    # docker_dhcp-proxy_1   node /RackHD/on-dhcp-proxy ...   Up
    # docker_dhcp_1         /docker-entrypoint.sh            Up
    # docker_files_1        /docker-entrypoint.sh            Up
    # docker_http_1         node /RackHD/on-http/index.js    Up
    # docker_mongo_1        docker-entrypoint.sh mongod      Up
    # docker_rabbitmq_1     docker-entrypoint.sh rabbi ...   Up
    # docker_syslog_1       node /RackHD/on-syslog/ind ...   Up
    # docker_taskgraph_1    node /RackHD/on-taskgraph/ ...   Up
    # docker_tasks_1        /bin/echo exit                   Exit 0
    # docker_tftp_1         node /RackHD/on-tftp/index.js    Up

######

How to Erase the Database to Restart Everything
-----------------------------------------------

.. code::

    sudo docker exec -it docker_mongo_1 mongo rackhd
    db.dropDatabase()
    # CTRL+D to exit
    # Restart RackHD
    cd RackHD/docker
    sudo docker-compose restart

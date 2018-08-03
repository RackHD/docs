Installation from NPM Package
=============================

.. contents:: Table of Contents

Ubuntu
-----------------------------

Prerequisites
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

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


NodeJS
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**If Node.js is not installed**

.. tabs::

    .. tab:: 4.x

        .. code::

            sudo apt-get remove nodejs nodejs-legacy
            curl -sL https://deb.nodesource.com/setup_4.x | sudo -E bash -
            sudo apt-get install -y nodejs

    .. tab:: 6.x

        .. code::

            sudo apt-get remove nodejs nodejs-legacy
            curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -
            sudo apt-get install -y nodejs

    .. tab:: 8.x

        .. code::

            sudo apt-get remove nodejs nodejs-legacy
            curl -sL https://deb.nodesource.com/setup_8.x | sudo -E bash -
            sudo apt-get install -y nodejs


Ensure Node.js is installed properly, example:

.. code::

    node -v

####

* **Dependencies**

  Install dependency packages

  .. code::

    sudo apt-get install build-essential
    sudo apt-get install libkrb5-dev
    sudo apt-get install rabbitmq-server
    sudo apt-get install mongodb
    sudo apt-get install snmp
    sudo apt-get install ipmitool

    sudo apt-get install git
    sudo apt-get install unzip
    sudo apt-get install ansible
    sudo apt-get install apt-mirror
    sudo apt-get install amtterm

    sudo apt-get install isc-dhcp-server


  **Note**:
  MongoDB versions 2.4.9 (on Ubuntu 14.04), 2.6.10 (on Ubuntu 16.04) and 3.4.9 (on both Ubuntu 14.04 and 16.04) are verified with RackHD.
  For more details on how to install MongDB 3.4.9, please refer to: https://docs.mongodb.com/manual/tutorial/install-mongodb-on-ubuntu/

####

Install & Configure RackHD
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. **Install RackHD NPM Packages**

   Install the latest release of RackHD

   .. code::

     for service in $(echo "on-dhcp-proxy on-http on-tftp on-syslog on-taskgraph");
     do
     npm install $service;
     done

####

2. **Basic RackHD Configuration**

   * **DHCP**

     Update /etc/dhcp/dhcpd.conf per your network configuration

     .. code::

      # RackHD added lines
      deny duplicates;

      ignore-client-uids true;

      subnet 172.31.128.0 netmask 255.255.240.0 {
        range 172.31.128.2 172.31.143.254;
        # Use this option to signal to the PXE client that we are doing proxy DHCP
        option vendor-class-identifier "PXEClient";
      }

   * **Open Ports in Firewall**

     If the firewall is enabled, open below ports in firewall:

     - 4011/udp
     - 8080/tcp
     - 67/udp
     - 8443/tcp
     - 69/udp
     - 9080/tcp

     An example of opening port:

     .. code::

       sudo ufw allow 8080


   * **CONFIGURATION FILE**

     Create the required file /opt/monorail/config.json , you can use the demonstration configuration file at https://github.com/RackHD/RackHD/blob/master/packer/ansible/roles/monorail/files/config.json as a reference.


   * **RACKHD BINARY SUPPORT FILES**

     Download binary files from bintray and placed them with below shell script.

     .. code::

      #!/bin/bash

      mkdir -p node_modules/on-tftp/static/tftp
      cd node_modules/on-tftp/static/tftp

      for file in $(echo "\
      monorail.ipxe \
      monorail-undionly.kpxe \
      monorail-efi64-snponly.efi \
      monorail-efi32-snponly.efi");do
      wget "https://dl.bintray.com/rackhd/binary/ipxe/$file"
      done

      cd -

      mkdir -p node_modules/on-http/static/http/common
      cd node_modules/on-http/static/http/common

      for file in $(echo "\
      discovery.docker.tar.xz \
      initrd-1.2.0-rancher \
      vmlinuz-1.2.0-rancher");do
      wget "https://dl.bintray.com/rackhd/binary/builds/$file"
      done

      cd -


3. **Start RackHD**

   Start the 5 services of RackHD with pm2 and a yml file.

   I. **Install pm2**

    .. code::

       sudo npm install pm2 -g

   II. **Prepare a yml file**

       An example of yml file:

       .. code::

        apps:
          - script: index.js
            name: on-taskgraph
            cwd: node_modules/on-taskgraph
          - script: index.js
            name: on-http
            cwd: node_modules/on-http
          - script: index.js
            name: on-dhcp-proxy
            cwd: node_modules/on-dhcp-proxy
          - script: index.js
            name: on-syslog
            cwd: node_modules/on-syslog
          - script: index.js
            name: on-tftp
            cwd: node_modules/on-tftp


   III. **Start Services**

    .. code::

       sudo pm2 start rackhd.yml

    All the services are started:

    .. code::

     ┌───────────────┬────┬──────┬───────┬────────┬─────────┬────────┬──────┬───────────┬──────────┐
     │ App name      │ id │ mode │ pid   │ status │ restart │ uptime │ cpu  │ mem       │ watching │
     ├───────────────┼────┼──────┼───────┼────────┼─────────┼────────┼──────┼───────────┼──────────┤
     │ on-dhcp-proxy │ 2  │ fork │ 16189 │ online │ 0       │ 0s     │ 60%  │ 21.2 MB   │ disabled │
     │ on-http       │ 1  │ fork │ 16183 │ online │ 0       │ 0s     │ 100% │ 21.3 MB   │ disabled │
     │ on-syslog     │ 3  │ fork │ 16195 │ online │ 0       │ 0s     │ 60%  │ 20.5 MB   │ disabled │
     │ on-taskgraph  │ 0  │ fork │ 16177 │ online │ 0       │ 0s     │ 6%   │ 21.3 MB   │ disabled │
     │ on-tftp       │ 4  │ fork │ 16201 │ online │ 0       │ 0s     │ 66%  │ 19.5 MB   │ disabled │
     └───────────────┴────┴──────┴───────┴────────┴─────────┴────────┴──────┴───────────┴──────────┘


#######

How to Erase the Database to Restart Everything
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

  .. code::

    sudo pm2 stop rackhd.yml

    mongo pxe
        db.dropDatabase()
        ^D

    sudo pm2 start rackhd.yml

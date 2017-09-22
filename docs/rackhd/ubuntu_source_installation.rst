Source Installation on Ubuntu
---------------------------------

Prerequisites
~~~~~~~~~~~~~
NICs
^^^^


**Ubuntu 14.04**

Start with an Ubuntu trusty(14.04) instance with 2 nics:

* eth0 for the 'public' network - providing access to RackHD APIs, and providing
  routed (layer3) access to out of band network for machines under management

* eth1 for dhcp/pxe to boot/configure the machines

edit the network:

* eth0 - assign IP address as appropriate for the environment, or you can use DHCP

* eth1 static ( 172.31.128.0/22 )

  this is the 'default'. it can be changed, but more than one file needs to be changed.)


#######

**Ubuntu 16.04**

Start with an Ubuntu xenial(16.04) instance with 2 nics:

* ens160 for the 'public' network - providing access to RackHD APIs, and providing
  routed (layer3) access to out of band network for machines under management

* ens192 for dhcp/pxe to boot/configure the machines

edit the network:

* ens160 - assign IP address as appropriate for the environment, or you can use DHCP

* ens192 static ( 172.31.128.0/22 )

.. code::

    auto ens192
    iface ens192 inet static
            address 0.0.0.0
            post-up ifconfig ens192 promisc

    auto bro
    iface bro inet static
            address 172.31.128.1
            bridge_ports ens192
            bridge_fd 0
            bridge_hello 1
            bridge_stp off
            post-up ifconfig bro promisc

this is the 'default'. it can be changed, but more than one file needs to be changed.)


NodeJS 4.x
^^^^^^^^^^

**If Node.js is not installed**

*If Node.js is installed via apt, but is older than version 4.x, do this first* (apt-get installs v0.10 by default)

.. code::

    sudo apt-get remove nodejs nodejs-legacy

Add the NodeSource key and repository (*instructions copied from* https://github.com/nodesource/distributions#manual-installation):

.. code::

    curl --silent https://deb.nodesource.com/gpgkey/nodesource.gpg.key | sudo apt-key add -
    VERSION=node_4.x
    DISTRO="$(lsb_release -s -c)"
    echo "deb https://deb.nodesource.com/$VERSION $DISTRO main" | sudo tee /etc/apt/sources.list.d/nodesource.list
    echo "deb-src https://deb.nodesource.com/$VERSION $DISTRO main" | sudo tee -a /etc/apt/sources.list.d/nodesource.list

    sudo apt-get update
    sudo apt-get install nodejs

Ensure Node.js is at version 4.x, example:

.. code::

    $ node -v
    v4.8.4


Install & Configure RackHD
~~~~~~~~~~~~~~~~~~~~~~~~~~


We will leverage the ansible roles created for the RackHD demonstration environment.

.. code::

    cd ~
    sudo apt-get install git
    sudo apt-get update
    sudo apt-get dist-upgrade
    sudo apt-get install ansible
    sudo apt-get install -y g++ libkrb5-dev unzip
    sudo apt-get install -y isc-dhcp-server mongodb rabbitmq-server ipmitool snmp


This created the default configuration file at /opt/monorail/config.json
from https://github.com/RackHD/RackHD/blob/master/packer/ansible/roles/monorail/files/config.json.
You may need to update this and /etc/dhcp/dhcpd.conf to match your local network configuration.like following:

.. code::
    # RackHD added lines
    deny duplicates;

    ignore-client-uids true;

    subnet 172.31.128.0 netmask 255.255.252.0 {
     range 172.31.128.2 172.31.131.254;
     # Use this option to signal to the PXE client that we are doing proxy DHCP
     option vendor-class-identifier "PXEClient";
     option routers 172.31.128.1;
     option domain-name-servers 10.*.*.*;
    }

After installation, the isc-dhcp-server, MongoDB, RabbitMQ service should automatically starts.

Check ipmitool & snmp version by following commands:

.. code::
    $ ipmitool -V
    ipmitool version 1.8.16

    $ snmpwalk -V
    NET-SNMP version: 5.7.3

Restart isc-dhcp-server to let the new configuration take effect:

.. code::
    $ sudo service isc-dhcp-server restart

Clone RackHD Source Code
^^^^^^^^^^^^^^^^^^^^^^^^
Below is a shell script, which clones all core RackHD services.

.. code::
    # !/bin/bash

    github_account="rackhd"

    mkdir -p ~/src

    for repo in $(echo "on-core on-tasks on-taskgraph on-http on-dhcp-proxy on-tftp on-syslog");do
      # clone my forked repo and set origin to my forked repo
      pushd ~/src
      git clone https://github.com/${github_account}/$repo.git
      popd
    done

Below is the shell script which installs dependencies for all repos and also build links between them:

.. code::
    #!/bin/bash

    cd ~/src/on-core
    npm install

    for repo in $(echo "on-tasks on-taskgraph on-http on-dhcp-proxy on-tftp on-syslog");do
      pushd ~/src/$repo
      npm install
      npm link ../on-core
    done

    for repo in $(echo "on-taskgraph on-http");do
      pushd ~/src/$repo
      npm link ../on-tasks
    done

To check whether the dependencies are installed correctly, you could run the unit-testing for each repo. To run unit-test, firstly go the repo's folder, then execute npm install, take on-core for example:

.. code::
    $ cd ~/src/on-core
    $ npm test
You need to ensure no failed test cases.

Configure RackHD Services
^^^^^^^^^^^^^^^^^^^^^^^^^
1. download static files for on-http & on-tftp, bellow is shell scripts.

.. code::
    echo "[Info] Download Static Images"
    HTTP_STATIC_FOLDER=~/src/on-http/static/http/common/
    TFTP_STATIC_FOLDER=~/src/on-tftp/static/tftp/
    mkdir -p ${HTTP_STATIC_FOLDER}
    mkdir -p ${TFTP_STATIC_FOLDER}

    HTTP_BASE_URL=http://dl.bintray.com/rackhd/binary/builds/
    TFTP_BASE_URL=http://dl.bintray.com/rackhd/binary/ipxe/
    HTTP_STATIC_FILES=( base.trusty.3.16.0-25-generic.full.squashfs.img base.trusty.3.16.0-25-generic.squashfs.img discovery.docker.tar.xz \
    discovery.overlay.cpio.gz initrd-0.5.0-rancher initrd-1.0.2-rancher initrd.img-3.16.0-25-generic \
    vmlinuz-0.5.0-rancher vmlinuz-1.0.2-rancher vmlinuz-3.16.0-25-generic )
    TFTP_STATIC_FILES=( monorail.ipxe monorail-undionly.kpxe monorail-efi32-snponly.efi monorail-efi64-snponly.efi monorail.intel.ipxe )
    for f in ${HTTP_STATIC_FILES[@]}; do
        sudo wget ${HTTP_BASE_URL}/${f}  ${HTTP_STATIC_FOLDER}/${f}
    done
    for f in ${TFTP_STATIC_FILES[@]}; do
        sudo  wget ${TFTP_BASE_URL}/${f}  ${TFTP_STATIC_FOLDER}/${f}
    done

2. create ~/src/rackhd.yml file.

.. code::
    apps:
        - script: index.js
          name: on-taskgraph
          cwd:  ./on-taskgraph
        - script: index.js
          name: on-http
          cwd:  ./on-http
        - script: index.js
          name: on-dhcp-proxy
          cwd:  ./on-dhcp-proxy
        - script: index.js
          name: on-syslog
          cwd:  ./on-syslog
        - script: index.js
          name: on-tftp
          cwd:  ./on-tftp

3. pm2 start

.. code::
    sudo pm2 start rackhd.yml

This will install all the relevant dependencies and code into ~/src, expecting
that it will be run with `pm2`_.

.. _pm2: http://pm2.keymetrics.io/

Some useful commands of pm2:

.. code::

    sudo pm2 restart all           # restart all RackHD services
    sudo pm2 restart on-taskgraph  # restart the on-taskgraph service only.
    sudo pm2 logs                  # show the combined real-time log for all RackHD services
    sudo pm2 logs on-taskgraph     # show the on-taskgraph real-time log
    sudo pm2 flush                 # clean the RackHD logs
    sudo pm2 status                # show the status of RackHD services


How to Reset the Database
~~~~~~~~~~~~~~~~~~~~~~~~~

.. code::

    echo "db.dropDatabase()" | mongo pxe

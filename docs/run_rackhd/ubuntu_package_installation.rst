Installation from Debian Package
================================

.. contents:: Table of Contents

Prerequisites
-----------------------------

NICs
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

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

  this is the 'default'. it can be changed, but more than one file needs to be changed.)

NodeJS 4.x
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

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
    v4.4.5


Install & Configure RackHD
-----------------------------

**After Prerequisites installation, there're two options to install and configure RackHD from package**

Either (a) or (b) can lead the way to install RackHD from debian packages.

(a) `Install/Configure with Ansible Playbook`_
(b) `Install/Configure with Step by Step Guide`_


_`Install/Configure with Ansible Playbook`
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
(1). install git and ansible

.. code::

  sudo apt-get install  git
  sudo apt-get install  ansible

(2). clone RackHD code

.. code::

  git clone https://github.com/RackHD/RackHD.git


The services files in /etc/init/ all need a conf file to exist in /etc/default/{service}
Touch those files to allow the upstart scripts to start automatically.

.. code::

  for service in $(echo "on-dhcp-proxy on-http on-tftp on-syslog on-taskgraph");
  do sudo touch /etc/default/$service;
  done


(3). Run the ansible playbooks

These will install the prerequisite packages, install the RackHD debian packages, and copy default configuration files

.. code::

  cd RackHD/packer/ansible
  ansible-playbook -c local -i "local," rackhd_package.yml

(4). Verify RackHD services

All the services are started and have logs in /var/log/rackhd.
Verify with ``service on-[something] status``

Notes：isc-dhcp-server is installed through ansible playbook, but sometimes it won't start on Ubuntu boot (https://ubuntuforums.org/showthread.php?t=2068111), 
check if DHCP service is started:

.. code::

    sudo service --status-all

If isc-dhcp-server is not running, run below to start DHCP service:

.. code::

    sudo service isc-dhcp-server start


_`Install/Configure with Step by Step Guide`
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

(1). Install the prerequisite packages:

  .. code::

    sudo apt-get install rabbitmq-server
    sudo apt-get install mongodb
    sudo apt-get install snmp
    sudo apt-get install ipmitool

    sudo apt-get install ansible
    sudo apt-get install apt-mirror
    sudo apt-get install amtterm

    sudo apt-get install isc-dhcp-server


  **Note**:
  MongoDB versions 2.4.9 (on Ubuntu 14.04), 2.6.10 (on Ubuntu 16.04) and 3.4.9 (on both Ubuntu 14.04 and 16.04) are verified with RackHD.
  For more details on how to install MongDB 3.4.9, please refer to: https://docs.mongodb.com/manual/tutorial/install-mongodb-on-ubuntu/

(2). Set up the RackHD bintray repository for use within this instance of Ubuntu

.. code::

    echo "deb https://dl.bintray.com/rackhd/debian trusty main" | sudo tee -a /etc/apt/sources.list
    sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 379CE192D401AB61
    sudo apt-get update

(3). Install RackHD debian package

The services files in /etc/init/ all need a conf file to exist in /etc/default/{service}
Touch those files to allow the upstart scripts to start automatically.

.. code::

  for service in $(echo "on-dhcp-proxy on-http on-tftp on-syslog on-taskgraph");
  do sudo touch /etc/default/$service;
  done

Install the RackHD Packages. Note: these packages are rebuilt on every commit to master and are
not explicitly versioned, but intended as a means to install or update to the latest code most
conveniently.

.. code::

    sudo apt-get install on-dhcp-proxy on-http on-taskgraph
    sudo apt-get install on-tftp on-syslog

(4). Basic RackHD Configuration


**DHCP**

Update dhcpd.conf per your network configuration

.. code::

    # RackHD added lines
    deny duplicates;

    ignore-client-uids true;

    subnet 172.31.128.0 netmask 255.255.240.0 {
      range 172.31.128.2 172.31.143.254;
      # Use this option to signal to the PXE client that we are doing proxy DHCP
      option vendor-class-identifier "PXEClient";
    }

Notes：sometimes isc-dhcp-server won't start on Ubuntu boot (https://ubuntuforums.org/showthread.php?t=2068111), 
check if DHCP service is started:

.. code::

    sudo service --status-all

If isc-dhcp-server is not running, run below to start DHCP service:

.. code::

    sudo service isc-dhcp-server start


#######

**RACKHD APPLICATIONS**

Create the required file /opt/monorail/config.json , you can use the demonstration
configuration file at https://github.com/RackHD/RackHD/blob/master/packer/ansible/roles/monorail/files/config.json
as a reference.

#######

**RACKHD BINARY SUPPORT FILES**

Downloaded binary files from bintray.com/rackhd/binary and placed them using https://github.com/RackHD/RackHD/blob/master/packer/ansible/roles/images/tasks/main.yml as a guide.

.. code::

    #!/bin/bash

    mkdir -p /var/renasar/on-tftp/static/tftp
    cd /var/renasar/on-tftp/static/tftp

    for file in $(echo "\
    monorail.ipxe \
    monorail-undionly.kpxe \
    monorail-efi64-snponly.efi \
    monorail-efi32-snponly.efi");do
    wget "https://dl.bintray.com/rackhd/binary/ipxe/$file"
    done

    mkdir -p /var/renasar/on-http/static/http/common
    cd /var/renasar/on-http/static/http/common

    for file in $(echo "\
    discovery.docker.tar.xz \
    initrd-1.2.0-rancher \
    vmlinuz-1.2.0-rancher");do
    wget "https://dl.bintray.com/rackhd/binary/builds/$file"
    done



All the services are started and have logs in /var/log/rackhd.  
Verify with ``service on-[something] status``

#######

How to Erase the Database to Restart Everything
-----------------------------------------------

  .. code::

    sudo service on-http stop
    sudo service on-dhcp-proxy stop
    sudo service on-syslog stop
    sudo service on-taskgraph stop
    sudo service on-tftp stop

    mongo pxe
        db.dropDatabase()
        ^D

    sudo service on-http start
    sudo service on-dhcp-proxy start
    sudo service on-syslog start
    sudo service on-taskgraph start
    sudo service on-tftp start

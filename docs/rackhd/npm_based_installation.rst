
NPM Based Installation
---------------------------------


Ubuntu
~~~~
Prerequisites
^^^^

**NICs**

1. **UBUNTU 14.04**

* Start with an Ubuntu trusty(14.04) instance with 2 nics:

   * eth0 for the 'public' network - providing access to RackHD APIs, and providing routed (layer3) access to out of band network for machines under management

   * eth1 for dhcp/pxe to boot/configure the machines

* Edit the network:

   * eth0 - assign IP address as appropriate for the environment, or you can use DHCP

   * eth1 static ( 172.31.128.0/22 )

     this is the 'default'. it can be changed, but more than one file needs to be changed.)


####

2. **UBUNTU 16.04**

* Start with an Ubuntu xenial(16.04) instance with 2 nics:

   * ens160 for the 'public' network - providing access to RackHD APIs, and providing routed (layer3) access to out of band network for machines under management

   * ens192 for dhcp/pxe to boot/configure the machines

* Edit the network:

   * ens160 - assign IP address as appropriate for the environment, or you can use DHCP

   * ens192 static ( 172.31.128.0/22 )

     this is the 'default'. it can be changed, but more than one file needs to be changed.)

**Packages**

1. **NodeJS 4.x**

   **If Node.js 4.x is not installed**

* **Remove Node.js (< 4.0)**

   *If Node.js is installed via apt, but is older than version 4.x, do this first* (apt-get installs v0.10 by default)
   .. code::

    sudo apt-get remove nodejs nodejs-legacy

* **Install Node.js 4.x**

   Add the NodeSource key and repository (*instructions copied from* https://github.com/nodesource/distributions#manual-installation):

   .. code::

    curl --silent https://deb.nodesource.com/gpgkey/nodesource.gpg.key | sudo apt-key add -
    VERSION=node_4.x
    DISTRO="$(lsb_release -s -c)"
    echo "deb https://deb.nodesource.com/$VERSION $DISTRO main" | sudo tee /etc/apt/sources.list.d/nodesource.list
    echo "deb-src https://deb.nodesource.com/$VERSION $DISTRO main" | sudo tee -a /etc/apt/sources.list.d/nodesource.list

    sudo apt-get update
    sudo apt-get install nodejs

* **Ensure Node.js is at version 4.x**

   For Example:

   .. code::

    $ node -v
    v4.4.5


2. **Dependencies**

* **Install dependency pakcages**

   .. code::

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


####

Install & Configure RackHD
^^^^

1. **Install RackHD NPM package**

*
   .. code::

     for service in $(echo "on-dhcp-proxy on-http on-tftp on-syslog on-taskgraph");
     do npm install $service;
     done


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


#######

* **RACKHD APPLICATIONS**

   Create the required file /opt/monorail/config.json , you can use the demonstration configuration file at https://github.com/RackHD/RackHD/blob/master/packer/ansible/roles/monorail/files/config.json as a reference.

#######

* **RACKHD BINARY SUPPORT FILES**

   Downloaded binary files from bintray.com/rackhd/binary and placed them.

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
    base.trusty.3.16.0-25-generic.squashfs.img \
    discovery.overlay.cpio.gz \
    initrd.img-3.16.0-25-generic \
    vmlinuz-3.16.0-25-generic");do
    wget "https://dl.bintray.com/rackhd/binary/builds/$file"
    done

    cd -

3. **Start RackHD**

   Start the 5 components of RackHD with pm2.
  
    .. code::

       pm2 start rackhd-pm2.yml

   An example of the yml:

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
^^^^

  .. code::

    pm2 stop rackhd-pm2.yml

    mongo pxe
        db.dropDatabase()
        ^D

    pm2 start rackhd-pm2.yml

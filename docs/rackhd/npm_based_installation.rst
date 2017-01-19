
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

* **Install dependency packages**

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


   Install the latest release of RackHD

   .. code::

     for service in $(echo "on-dhcp-proxy on-http on-tftp on-syslog on-taskgraph");
     do 
     npm install $service;
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

* **CONFIGURATION FILE**

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

   Start the 5 service of RackHD with pm2 and a yml file.

* **Install pm2**
  
    .. code::
      
       sudo npm install pm2 -g

* **Prepare a yml file**

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
     

* **Start Service**

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
^^^^

  .. code::

    sudo pm2 stop rackhd.yml

    mongo pxe
        db.dropDatabase()
        ^D

    sudo pm2 start rackhd.yml



######



Centos7
~~~~
Prerequisites
^^^^

**NICs**

* Start with an centos 7 instance with 2 nics:

   * eno16777984 for the 'public' network - providing access to RackHD APIs, and providing routed (layer3) access to out of band network for machines under management

   * eno33557248 for dhcp/pxe to boot/configure the machines

* Edit the network:

   * eno16777984 - assign IP address as appropriate for the environment, or you can use DHCP

   * eno33557248 static ( 172.31.128.0/22 )

     this is the 'default'. it can be changed, but more than one file needs to be changed.)


**Packages**

1. **NodeJS 4.x**

   **If Node.js 4.x is not installed**

* **Remove Node.js (< 4.0)**

   *If Node.js is installed via yum, but is older than version 4.x, do this first* 
   .. code::

    sudo yum remove nodejs

* **Install Node.js 4.x**

   *Instructions copied from* https://github.com/nodesource/distributions#manual-installation:

   .. code::

     curl -sL https://rpm.nodesource.com/setup_4.x |sudo bash -
     sudo yum install -y nodejs

   **Optional**: install build tools

   To compile and install native addons from npm you may also need to install build tools:

   .. code::

     yum install gcc-c++ make
     # or: yum groupinstall 'Development Tools'

* **Ensure Node.js is at version 4.x**

   For Example:

   .. code::

    $ node -v
    v4.4.5


2. **Dependencies**

* **RabbitMQ**

  - **Install Erlang**

    .. code::

     sudo yum -y update
     sudo yum install -y epel-release
     sudo yum install -y gcc gcc-c++ glibc-devel make ncurses-devel openssl-devel autoconf java-1.8.0-openjdk-devel git wget wxBase.x86_64

     wget http://packages.erlang-solutions.com/erlang-solutions-1.0-1.noarch.rpm
     sudo rpm -Uvh erlang-solutions-1.0-1.noarch.rpm
     sudo yum -y update


  - **Verify Erlang**

    .. code::

     erl
     
    Sample output:
    
    .. code::

     Erlang/OTP 19 [erts-8.2] [source-fbd2db2] [64-bit] [smp:8:8] [async-threads:10] [hipe] [kernel-poll:false]

     Eshell V8.2  (abort with ^G)
     1>

  - **Install RabbitMQ**

    .. code::

     wget https://www.rabbitmq.com/releases/rabbitmq-server/v3.6.1/rabbitmq-server-3.6.1-1.noarch.rpm
     sudo rpm --import https://www.rabbitmq.com/rabbitmq-signing-key-public.asc
     sudo yum install -y rabbitmq-server-3.6.1-1.noarch.rpm


  - **Start RabbitMQ**

    .. code::

      sudo systemctl start rabbitmq-server
      sudo systemctl status rabbitmq-server



* **MongoDB**

  - **Configure the package management system (yum)**

    
    Create a /etc/yum.repos.d/mongodb-org-3.4.repo and add below lines: 


    .. code::

     [mongodb-org-3.4]
     name=MongoDB Repository
     baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/3.4/x86_64/
     gpgcheck=1
     enabled=1
     gpgkey=https://www.mongodb.org/static/pgp/server-3.4.asc


  - **Install MongoDB**

    .. code::

     sudo yum install -y mongodb-org


  - **Start MongoDB**

    .. code::

      sudo systemctl start mongod.service
      sudo systemctl status mongod.service
  

* **snmp**

  - **Install snmp**

    .. code::

     sudo yum install -y net-snmp


  - **Start snmp**

    .. code::

     sudo systemctl start snmpd.service
     sudo systemctl status snmpd.service


* **ipmitool**

  - **Install ipmitool**

    .. code::

     sudo yum install -y OpenIPMI ipmitool


* **git**

  - **Install git**

    .. code::

     sudo yum install -y git

  - **Verify git**

    .. code::

     git --version


* **ansible**

  - **Install ansible**

    .. code::

     sudo yum install -y ansible

  - **Verify ansible**

    .. code::

     ansible --version

    Sample output:

    .. code::

     ansible 2.2.0.0
       config file = /etc/ansible/ansible.cfg
       configured module search path = Default w/o overrides

* **amtterm**

  - **Install amtterm**

    .. code::

     sudo yum install amtterm


* **dhcp**

  - **Install dhcp**

    .. code::

     sudo yum install -y dhcp
     sudo cp /usr/share/doc/dhcp-4.2.5/dhcpd.conf.example /etc/dhcp/dhcpd.conf



####

Install & Configure RackHD
^^^^

1. **Install RackHD NPM package**

*
  Install the latest release RackHD

   .. code::

     for service in $(echo "on-dhcp-proxy on-http on-tftp on-syslog on-taskgraph");
     do 
     npm install $service;
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

* **CONFIGURATION FILE**

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

   Start the 5 service of RackHD with pm2 and a yml file.

* **Install pm2**

    .. code::

       sudo npm install pm2 -g

* **Prepare a yml file**

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
     

* **Start Service**

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
^^^^

  .. code::

    sudo pm2 stop rackhd.yml

    mongo pxe
        db.dropDatabase()
        ^D

    sudo pm2 start rackhd.yml

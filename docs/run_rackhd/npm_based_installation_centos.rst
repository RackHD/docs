Installation from NPM Package
=============================

.. contents:: Table of Contents

CentOS 7
-----------------------------
Prerequisites
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**NICs**

1. Start with an centos 7 instance with 2 nics:

   * ``eno16777984`` for the ``public`` network - providing access to RackHD APIs, and providing routed (layer3) access to out of band network for machines under management

   * ``eno33557248`` for dhcp/pxe to boot/configure the machines

2. Edit the network:

   * ``eno16777984`` - assign IP address as appropriate for the environment, or you can use DHCP

   * ``eno33557248`` static ( 172.31.128.0/22 )

     this is the ``default``. it can be changed, but more than one file needs to be changed.)


**Packages**

* **NodeJS**

.. tabs::

    .. tab:: 4.x

        .. code::

            sudo yum remove nodejs
            curl -sL https://rpm.nodesource.com/setup_4.x | sudo bash -
            sudo yum install -y nodejs

    .. tab:: 6.x

        .. code::

            sudo yum remove nodejs
            curl -sL https://rpm.nodesource.com/setup_6.x | sudo bash -
            sudo yum install -y nodejs

    .. tab:: 8.x

        .. code::

            sudo yum remove nodejs
            curl -sL https://rpm.nodesource.com/setup_8.x | sudo bash -
            sudo yum install -y nodejs


**Optional**: install build tools

To compile and install native addons from npm you may also need to install build tools:

.. code::

   yum install gcc-c++ make
   # or: yum groupinstall 'Development Tools'

####

* **RabbitMQ**

  1. **Install Erlang**

      .. code::

       sudo yum -y update
       sudo yum install -y epel-release
       sudo yum install -y gcc gcc-c++ glibc-devel make ncurses-devel openssl-devel autoconf java-1.8.0-openjdk-devel git wget wxBase.x86_64

       wget http://packages.erlang-solutions.com/erlang-solutions-1.0-1.noarch.rpm
       sudo rpm -Uvh erlang-solutions-1.0-1.noarch.rpm
       sudo yum -y update


  2. **Verify Erlang**

       .. code::

        erl

       Sample output:

       .. code::

        Erlang/OTP 19 [erts-8.2] [source-fbd2db2] [64-bit] [smp:8:8] [async-threads:10] [hipe] [kernel-poll:false]

        Eshell V8.2  (abort with ^G)
        1>

  3. **Install RabbitMQ**

      .. code::

       wget https://www.rabbitmq.com/releases/rabbitmq-server/v3.6.1/rabbitmq-server-3.6.1-1.noarch.rpm
       sudo rpm --import https://www.rabbitmq.com/rabbitmq-signing-key-public.asc
       sudo yum install -y rabbitmq-server-3.6.1-1.noarch.rpm

  4. **Start RabbitMQ**

      .. code::

        sudo systemctl start rabbitmq-server
        sudo systemctl status rabbitmq-server


* **MongoDB**

  1. **Configure the package management system (yum)**


      Create a /etc/yum.repos.d/mongodb-org-3.4.repo and add below lines:


      .. code::

       [mongodb-org-3.4]
       name=MongoDB Repository
       baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/3.4/x86_64/
       gpgcheck=1
       enabled=1
       gpgkey=https://www.mongodb.org/static/pgp/server-3.4.asc


  2. **Install MongoDB**

    .. code::

     sudo yum install -y mongodb-org


  3. **Start MongoDB**

    .. code::

      sudo systemctl start mongod.service
      sudo systemctl status mongod.service


* **snmp**

  1. **Install snmp**

    .. code::

     sudo yum install -y net-snmp


  2. **Start snmp**

    .. code::

     sudo systemctl start snmpd.service
     sudo systemctl status snmpd.service


* **ipmitool**

    .. code::

     sudo yum install -y OpenIPMI ipmitool


* **git**

  1. **Install git**

    .. code::

     sudo yum install -y git

  2. **Verify git**

    .. code::

     git --version


* **ansible**

  1. **Install ansible**

    .. code::

     sudo yum install -y ansible

  2. **Verify ansible**

    .. code::

     ansible --version


    Sample output:

    .. code::

     ansible 2.2.0.0
       config file = /etc/ansible/ansible.cfg
       configured module search path = Default w/o overrides

* **amtterm**

    .. code::

     sudo yum install amtterm


* **dhcp**

    .. code::

     sudo yum install -y dhcp
     sudo cp /usr/share/doc/dhcp-4.2.5/dhcpd.conf.example /etc/dhcp/dhcpd.conf



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

       sudo firewall-cmd --permanent --add-port=8080/tcp
       sudo firewall-cmd --reload


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

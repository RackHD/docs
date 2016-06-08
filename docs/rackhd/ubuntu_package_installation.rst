Ubuntu Package Based Installation
---------------------------------


Prerequisites
~~~~~~~~~~~~~~~

Start with an Ubuntu trusty instance with 2 nics:

* eth0 for the 'public' network - providing access to RackHD APIs, and providing
  routed (layer3) access to out of band network for machines under management

* eth1 for dhcp/pxe to boot/configure the machines

edit the network:

* eth0 - assign IP address as appropriate for the environment, or you can use DHCP

* eth1 static ( 172.31.128.0/22 )

  this is the 'default'. it can be changed, but more than one file needs to be changed.)

Install the prerequisite packages:

.. code::

    sudo apt-get install rabbitmq-server
    sudo apt-get install mongodb
    sudo apt-get install snmp
    sudo apt-get install ipmitool
    sudo apt-get install nodejs nodejs-legacy npm

    sudo apt-get install ansible
    sudo apt-get install apt-mirror
    sudo apt-get install amtterm

    sudo apt-get install isc-dhcp-server

Set up the RackHD bintray repository for use within this instance of Ubuntu

.. code::

    echo "deb https://dl.bintray.com/rackhd/debian trusty main" | sudo tee -a /etc/apt/sources.list
    sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 379CE192D401AB61
    sudo apt-get update

Install the RackHD Packages. Note: these packages are rebuilt on every commit to master and are
not explicitly versioned, but intended as a means to install or update to the latest code most
conveniently.

.. code::

    sudo apt-get install on-dhcp-proxy on-http on-taskgraph
    sudo apt-get install on-tftp on-syslog

Configuring RackHD
~~~~~~~~~~~~~~~~~~~~

**DHCP**

Update dhcpd.conf per your network configuration

.. code::

    # RackHD added lines
    deny duplicates;

    ignore-client-uids true;

    subnet 172.31.128.0 netmask 255.255.252.0 {
      range 172.31.128.2 172.31.131.254;
      # Use this option to signal to the PXE client that we are doing proxy DHCP
      option vendor-class-identifier "PXEClient";
    }

#######

**UPSTART**

The services files in /etc/init/ all need a conf file to exist in /etc/default/{service}
Touch those files to allow the upstart scripts to start automatically.

.. code::

    for service in $(echo "on-dhcp-proxy on-http on-tftp on-syslog on-taskgraph");
    do touch /etc/default/$service;
    done

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
    base.trusty.3.13.0-32-generic.squashfs.img \
    base.trusty.3.16.0-25-generic.squashfs.img \
    discovery.overlay.cpio.gz \
    initrd.img-3.13.0-32-generic \
    initrd.img-3.16.0-25-generic \
    vmlinuz-3.13.0-32-generic \
    vmlinuz-3.16.0-25-generic");do
    wget "https://dl.bintray.com/rackhd/binary/builds/$file"
    done



All the services are upstart and have logs in /var/log/upstart.  Start with 'start on-[something]'
Verify with 'ps | aux | grep node'

#######

**MIRRORS**

Mirrors may not be something you need to configure if you're utilizing the proxy capabilities
of RackHD. If you previously configured proxies to support OS installation workflows,
then those should be configured to provide access to the files needed. If you do not, or can
not, utilize proxies, then you can host the OS installation files directly on the same
instance with RackHD. The following instructions show how to create OS installation mirrors
in support of the existing workflows.

**Set the Links to the Mirrors:**

  .. code::

    sudo ln -s /var/mirrors/ubuntu <on-http directory>/static/http/ubuntu
    sudo ln -s /var/mirrors/ubuntu/14.04/mirror/mirror.pnl.gov/ubuntu/ <on-http directory>/static/http/trusty
    sudo ln -s /var/mirrors/centos <on-http directory>/static/http/centos
    sudo ln -s /var/mirrors/suse <on-http directory>/static/http/suse

Making the Mirrors
~~~~~~~~~~~~~~~~~~~~~~~~~~

**Centos 6.5**

  .. code::

    sudo rsync --progress -av --delete --delete-excluded --exclude "local*" \
    --exclude "i386" rsync://centos.eecs.wsu.edu/centos/6.5/ /var/mirrors/centos/6.5

**Centos 7.0**

  .. code::

    sudo rsync --progress -av --delete --delete-excluded --exclude "local*" \
    --exclude "i386" rsync://centos.eecs.wsu.edu/centos/7/ /var/mirrors/centos/7

**OpenSuse 12.3**

  .. code::

    sudo rsync --progress -av --delete --delete-excluded --exclude "local*" --exclude "i386" --exclude "i586" --exclude "i686" rsync://mirror.clarkson.edu/opensuse/distribution/12.3/ /var/mirrors/suse/distribution/12.3
    sudo rsync --progress -av --delete --delete-excluded --exclude "local*" --exclude "i386" --exclude "i586" --exclude "i686" rsync://mirror.clarkson.edu/opensuse/update/12.3 /var/mirrors/suse/update
    sudo rsync --progress -av --delete --delete-excluded --exclude "local*" --exclude "i386" --exclude "i586" --exclude "i686" rsync://mirror.clarkson.edu/opensuse/update/12.3-non-oss /var/mirrors/suse/update

**OpenSuse 13.1**

  .. code::

    sudo rsync --progress -av --delete --delete-excluded --exclude "local*" --exclude "i386" --exclude "i586" --exclude "i686" rsync://mirror.clarkson.edu/opensuse/distribution/13.1/ /var/mirrors/suse/distribution/13.1
    sudo rsync --progress -av --delete --delete-excluded --exclude "local*" --exclude "i386" --exclude "i586" --exclude "i686" rsync://mirror.clarkson.edu/opensuse/update/13.1 /var/mirrors/suse/update
    sudo rsync --progress -av --delete --delete-excluded --exclude "local*" --exclude "i386" --exclude "i586" --exclude "i686" rsync://mirror.clarkson.edu/opensuse/update/13.1-non-oss /var/mirrors/suse/update

**OpenSuse 13.2**

  .. code::

    sudo rsync --progress -av --delete --delete-excluded --exclude "local*" --exclude "i386" --exclude "i586" --exclude "i686" rsync://mirror.clarkson.edu/opensuse/distribution/13.2/ /var/mirrors/suse/distribution/13.2
    sudo rsync --progress -av --delete --delete-excluded --exclude "local*" --exclude "i386" --exclude "i586" --exclude "i686" rsync://mirror.clarkson.edu/opensuse/update/13.2 /var/mirrors/suse/update
    sudo rsync --progress -av --delete --delete-excluded --exclude "local*" --exclude "i386" --exclude "i586" --exclude "i686" rsync://mirror.clarkson.edu/opensuse/update/13.2-non-oss /var/mirrors/suse/update


For the Ubuntu repo, you need some additional installation. The mirrors are easily made on Ubuntu, but not so easily replicated on other OS. On any recent distribution of Ubuntu:

  .. code::

	# make the mirror directory (can sometimes hit a permissions issue)
	sudo mkdir -p /var/mirrors/ubuntu/14.04/mirror
	# create a file in /etc/apt/mirror.list (config below)
	sudo vi /etc/apt/mirror.list
	# run the mirror
	sudo apt-mirror


    ############# config ##################
    #
    set base_path    /var/mirrors/ubuntu/14.04
    #
    # set mirror_path  $base_path/mirror
    # set skel_path    $base_path/skel
    # set var_path     $base_path/var
    # set cleanscript $var_path/clean.sh
    # set defaultarch  <running host architecture>
    # set postmirror_script $var_path/postmirror.sh
    # set run_postmirror 0
    set nthreads     20
    set _tilde 0
    #
    ############# end config ##############

    deb-amd64 http://mirror.pnl.gov/ubuntu trusty main
    deb-amd64 http://mirror.pnl.gov/ubuntu trusty-updates main
    deb-amd64 http://mirror.pnl.gov/ubuntu trusty-security main
    clean http://mirror.pnl.gov/ubuntu

    #end of file
    ###################

How to Erase the Database to Restart Everything
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

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

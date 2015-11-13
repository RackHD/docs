RackHD: Quick Setup
===================

*----------------------------- Content in Development -----------------------------*



To make it easy to see RackHD in action and start to play with it, we've
set up a vagrant based example deployment and script to help get everything
up and running on a local laptop.

The example/ directory contains the script fire up RackHD, and also creates
a virtual machine to act as a "sample machine in a rack".

.. note::

   **Prerequisite**: The example setup uses `Vagrant`_ with an `Ansible`_ based
   provisioner to get everything set up. You will need to install both locally
   in order to run this example.


#. Clone at minimum the rackHD repository. This repository includes both git submodules with relevant source code and the example vagrant instance::

    git clone --recursive https://github.com/RackHD/RackHD

#. Create a local configuration file and spin up vagrant::

    cd RackHD
    cp example/config/monorail_rack.cfg.example example/config/monorail_rack.cfg

#. Fire up the virtual rack::

    cd example/bin
    ./monorail_rack

#. ssh into the RackHD server and use on-imagebuilder to build the static files::

    vagrant ssh dev
    sudo apt-get install -y ansible
    cd ~
    git clone https://github.com/rackhd/on-imagebuilder
    cd on-imagebuilder
    sudo ansible-playbook -i hosts all.yml

#. Move the resulting files into the HTTP server's static files directory::

    sudo mv /tmp/on-imagebuilder/builds/* ~/src/on-http/static/http/common/


#. Finally, bring up the RackHD services::

    cd ~
    sudo nf start or sudo nf start [graph,http,dhcp,tftp,syslog]


Now that the services are running we can begin powering on pxe clients and watch them boot.

.. _Vagrant: https://www.vagrantup.com
.. _Ansible: http://www.ansible.com

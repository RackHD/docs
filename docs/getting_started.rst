RackHD: Quick Setup
===================

*----------------------------- Content in Development -----------------------------*



The example/ directory contains a setup script that you can use to get RackHD up and running in a example virtual rack.
It creates a virtualbox VM with
RackHD running within it, and another that is connected to it to PXE boot.

**Prerequisite**: The example setup uses Vagrant and ansible locally. Please install them directly on your local machine.

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

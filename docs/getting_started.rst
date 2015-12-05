RackHD: Quick Setup
===================

*----------------------------- Content in Development -----------------------------*

To make it easy to see and play with a sample RackHD instance, we've
set up a vagrant-based example deployment and script to help get everything
up and running on a local system.

The example/ directory in the `RackHD repository`_ contains the script creates a VM that simulates a sample machine in a
rack and fires up a RackHD instance that connects to and manages that machine.


.. _RackHD repository: https://github.com/RackHD/RackHD

Setup Steps
------------------------------

.. note::

**Prerequisite**: The example setup requires local installation of `Vagrant`_ with an `Ansible`_ based provisioner.

.. _Vagrant: https://www.vagrantup.com
.. _Ansible: http://www.ansible.com

#. Clone the `RackHD`_ repository. This repository includes git submodules with relevant source code and the example vagrant instance::

    git clone --recursive https://github.com/RackHD/RackHD

.. _RackHD: https://github.com/RackHD/RackHD

#. Create a local configuration file and spin up vagrant::

    cd RackHD
    cp example/config/monorail_rack.cfg.example example/config/monorail_rack.cfg

#. Fire up the virtual rack::

    cd example/bin
    ./monorail_rack

#. Bring up RackHD services::

    vagrant ssh -c "sudo nf start"

#. Once all services are running, you can power on PXE clients and watch them boot. To do this, open the virtualbox application and manually start the PXE demonstration VM. Alternatively, simply open another terminal window and run the following command::

    vboxmanage startvm pxe-1 --type gui



How It Works
---------------------

The vagrant instance setups a pre-installed RackHD VM to be connected to a VM set to PXE boot to emulate hardware.
The RackHD VM has two network interfaces, one local to your machine via NAT, and the other a private network (`closednet`) 
connecting the PXE vm to the RackHD instance. We keep this on a private network because RackHD runs DHCP and PXE, and
we don't want to accidentally expose that to your local network.

The Vagrant setup also enables port forwarding to access the RackHD instance:

 - localhost:9090 redirects to rackhd:8080 for access to the REST API
 - localhost:2222 redirects to rackhd:22 for SSH access - how `vagrant ssh` works

|
.. image: _static/vagrant_setup.jpg 
 :align: left
 
We have set up `node-foreman`_ to control and run the the node.js applications from a single command:

    sudo nf start

**Note:** This example RackHD instance does not include complete out-of-band networking control. Running workflows requires
a "no-op" OBM management setting on discovered nodes and manual restarts of VMs.

.. _travisCI: https://travis-ci.org/
.. _node-foreman: https://github.com/strongloop/node-foreman

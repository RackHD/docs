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

#. Clone the RackHD repository. This repository includes git submodules with relevant source code and the example vagrant instance::

    git clone --recursive https://github.com/RackHD/RackHD

#. Create a local configuration file and spin up vagrant::

    cd RackHD
    cp example/config/monorail_rack.cfg.example example/config/monorail_rack.cfg

#. Fire up the virtual rack::

    cd example/bin
    ./monorail_rack

#. Bring up RackHD services::

    vagrant ssh -c "sudo nf start"

#. Once all services are running, you can power on PXE clients and watch them boot. To do this, open the virtualbox application and manually start the PXE demonstration VM.
Alternatively, simply open another terminal window and run the following command::

    vboxmanage startvm pxe-1 --type gui



How It Works
---------------------

The vagrant file and ansible provisioner roles check out the relevant
applications from the repository submodules. Then Node4 is set up and "npm install" is run for each application.

The system also pulls specific binaries, microkernels, and bootloaders from
from bintray. The mechanisms that create those files are in the
https://github.com/rackhd/on-imagebuilder repository. When the files are retrieved, `travisCI`_
builds (https://travis-ci.org/RackHD/on-taskgraph) populate the binaries into bintray for example usage.

The ansible role installs `node-foreman`_ and runs the node.js applications
from a single script.

The vagrant file enables a network interface on a private virtual network. The VM that is created by
the `./monorail_rack` script connects to this network.

RackHD can then run DHCP, TFTP, etc on the interface to the private network (to PXE boot the VM).

**Note:** This example RackHD instance does not include complete out-of-band networking control. Running workflows requires
a "no-op" OBM management setting on discovered nodes and manual restarts of VMs.

.. _travisCI: https://travis-ci.org/
.. _node-foreman: https://github.com/strongloop/node-foreman

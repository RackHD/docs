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


.. _Vagrant: https://www.vagrantup.com
.. _Ansible: http://www.ansible.com

#. Clone at minimum the rackHD repository. This repository includes both git submodules with relevant source code and the example vagrant instance::

    git clone --recursive https://github.com/RackHD/RackHD

#. Create a local configuration file and spin up vagrant::

    cd RackHD
    cp example/config/monorail_rack.cfg.example example/config/monorail_rack.cfg

#. Fire up the virtual rack::

    cd example/bin
    ./monorail_rack

#. Finally, bring up the RackHD services::

    cd ~
    sudo nf start or sudo nf start [graph,http,dhcp,tftp,syslog]

Now that the services are running we can begin powering on pxe clients and
watch them boot.

What this sets up
-----------------

The vagrant file and ansible provisioner roles checks out the relevant
applications from the submodules included in this repository, and does the
relevant work to setup Node4 and run "npm install" on each o the applications.

The system also expects to use some
specific binaries, microkernels and bootloaders - and pulls those files down
from bintray. The mechanisms to create those files are all in the
https://github.com/rackhd/on-imagebuilder repository, and `travisCI`_ based
builds (https://travis-ci.org/RackHD/on-taskgraph) populate the binaries from
there directly into bintray for example usage.

The ansible role installs `node-foreman`_ and runs the node.js applications
from a single script.

The vagrant file is configured to set up enable a network interface on a
private virtual network, and the `./monorail_rack` script creates a sample
virtual machine, also connected to that same private network. The configuration
is set to run DHCP, TFTP, etc on the interface to that private network (to PXE
boot that virtual machine). RackHD would normally work with a full out of band
networking control setup, which this currently does not set up - so running
workflows within this example set up will require a "no-op" OBM management
setting on discovered nodes, and manual restarting of VMs to vet workflows.

.. _travisCI: https://travis-ci.org/
.. _node-foreman: https://github.com/strongloop/node-foreman

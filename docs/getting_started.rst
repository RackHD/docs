RackHD: Quick Setup
===================

*----------------------------- Content in Development -----------------------------*

To make it easy to see and play with a sample RackHD instance, we've
set up a vagrant-based example deployment and script to help get everything
up and running on a local system.

The example/ directory in the `RackHD repository`_ contains the script
creates a VM that simulates a sample machine in a rack and fires up a RackHD
instance that connects to and manages that machine.

You can follow the instructions at `README`_ to quickly set up RackHD and
set it up to auto-install CoreOS on a test virtual machine.

.. _RackHD repository: https://github.com/RackHD/RackHD
.. _README: https://github.com/RackHD/RackHD/blob/master/example/README.md

How It Works
---------------------

The vagrant instance setups a pre-installed RackHD VM to be connected to a VM set to PXE boot to emulate hardware.
The RackHD VM has two network interfaces, one local to your machine via NAT, and the other a private network (`closednet`)
connecting the PXE vm to the RackHD instance. We keep this on a private network because RackHD runs DHCP and PXE, and
we don't want to accidentally expose that to your local network.

.. image:: _static/vagrant_setup.jpg
 :height: 300

The Vagrant setup also enables port forwarding to access the RackHD instance:

- localhost:9090 redirects to rackhd:8080 for access to the REST API
- localhost:2222 redirects to rackhd:22 for SSH access - how `vagrant ssh` works


We have set up `node-foreman`_ to control and run the the node.js applications from a single command::

    sudo nf start

**Note:** This example RackHD instance does not include complete out-of-band networking control. Running workflows requires
a "no-op" OBM management setting on discovered nodes and manual restarts of VMs.

.. _travisCI: https://travis-ci.org/
.. _node-foreman: https://github.com/strongloop/node-foreman

RackHD: Quick Setup
===================

You can obtain a Vagrant-based demonstration version of RackHD from GitHub and quickly deploy it on a local system.
The demo runs VirtualBox VMs that allow you to use RackHD to manage one or more simulated target machines.

For a detailed description and deployment instructions, read the README.md in the *example* directory of
the `RackHD repository`_.


.. _RackHD repository: https://github.com/RackHD/RackHD
.. _README: https://github.com/RackHD/RackHD/blob/master/example/README.md

How It Works
---------------------

The Vagrant instance sets up a pre-installed RackHD VM that connects to one or more VMs
that represent managed systems. The target systems are simulated using PXE clients.



.. image:: ../_static/vagrant_setup.jpg
     :height: 300
     :align: center



The RackHD VM has two network interfaces. One connects to the local machine via NAT (Network Address Translation)
and the second connects to the PXE VMs in a private network. The private network is used so that RackHD DHCP and
PXE operations are isolated from your local network.

The Vagrant setup also enables port forwarding that allows your localhost to access the RackHD instance:

- localhost:9090 redirects to rackhd:8080 for access to the REST API
- localhost:2222 redirects to rackhd:22 for SSH access


We have set up `node-foreman`_ to control and run the node.js applications from a single command::

    sudo nf start

**Note:** This example RackHD instance does not include complete out-of-band networking control. Running workflows requires
a "no-op" OBM management setting on discovered nodes and manual restarts of VMs.

.. _travisCI: https://travis-ci.org/
.. _node-foreman: https://github.com/strongloop/node-foreman

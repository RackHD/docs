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

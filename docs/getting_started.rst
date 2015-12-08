RackHD: Quick Setup
===================

The monorail_rack setup script is an easy "one button push" script to deploy a 'virtual rack' using Virtualbox.
This emulates a RackHD server and some number of virtual servers - using virtualbox PXE-booting VMs.
Private virtual networks simulate the connections between servers that would otherwise be on a switch in a rack.

The virtual network closednet is set to our default subnet of 172.31.128.x to connect DHCP and TFTP from
RackHD to the PXE clients

Prerequisites and Script Expectations
--------------------------------------------

The latest version of git, Vagrant, and Ansible must be installed onto your system in order to use this script.

We also rely on this projects structure of submodules to link the source into the VM (through vagrant).
The ansible roles are written to expect the source to be autoloaded on the virtual machine with directory
mappings configured in the Vagrantfile, for example::

  ~/<repos directory>/RackHD/on-http/static/http/common/

The static files that RackHD uses can be built locally using the tools found in the `on-imagebuilder`_
repository. This script will download the latest built versions that are stored in bintray from that
repository's outputs.

.. _on-imagebuilder: https://github.com/RackHD/on-imagebuilder

Setup Steps
------------------------------

#. Clone the `RackHD`_ repository. This repository includes git submodules with relevant source code and the example
vagrant instance::

    git clone --recursive https://github.com/RackHD/RackHD

.. _RackHD: https://github.com/RackHD/RackHD

#. Create a local configuration file::

    pushd example/config/
    cp ./monorail_rack.cfg.example ./monorail_rack.cfg
    popd

**Note:** You can edit monorail_rack.cfg to adjust the number of PXE clients that are created.

#. Fire up the virtual rack::

    cd example/bin
    ./monorail_rack

#. Use ssh to connect to the RackHD server and start the services::

    vagrant ssh
    sudo nf start

Testing
-------------------------

After starting the services, the RackHD API is available on your local machine using port 9090.
You can verify that RackHD is running by bringing up the API in a browser at http://localhost:9090/docs.

You can also interact with the APIs using curl from the command line of your local machine.

To view the list of discovered nodes::

   curl http://localhost:9090/api/1.1/nodes | python -m json.tool

To view the list of catalogs logged into RackHD::

   curl http://localhost:9090/api/1.1/catalogs | python -m json.tool

**Note:** In a new installation, both commands should result in empty lists.


Install a Default Workflow and SKU Definition
-----------------------------------------------------

This example includes a workflow to use when we identify a "virtualbox" SKU with RackHD
This workflow sets up no-op out-of-band management settings for a demo and triggers an installation
of CoreOS as a default flow to run once the "virtualbox" SKU has been identified.

Load the workflow into the library of workflows::

    cd ~/src/rackhd/example
    # make sure you're in the example directory to reference the sample JSON correctly

    curl -H "Content-Type: application/json" \
    -X PUT --data @samples/virtualbox_install_coreos.json \
    http://localhost:9090/api/1.1/workflows

To enable that workflow, add a SKU definition that includes the option of running another
workflow once the SKU has been identified. This takes advantage of the Graph.SKU.Discovery workflow,
which will attempt to identify a SKU and run another workflow if specified::

   cd ~/src/rackhd/example
   # make sure you're in the example directory to reference the sample JSON correctly

   curl -H "Content-Type: application/json" \
   -X POST --data @samples/virtualbox_sku.json \
   http://localhost:9090/api/1.1/skus

To view the current SKU definitions::

   curl http://localhost:9090/api/1.1/skus | python -m json.tool

Output::

  [
  {
      "createdAt": "2015-11-21T00:46:04.068Z",
      "discoveryGraphName": "Graph.DefaultVirtualBox.InstallCoreOS",
      "discoveryGraphOptions": {},
      "id": "564fbecc1dee9e7d2f1d33ca",
      "name": "Noop OBM settings for VirtualBox nodes",
      "rules": [
          {
              "equals": "VirtualBox",
              "path": "dmi.System Information.Product Name"
          }
      ],
      "updatedAt": "2015-11-21T00:46:04.068Z"
  }
  ]



Once you've added those definitions, you can start up the test "PXE-1" virtual machine using the command::

  vboxmanage startvm pxe-1 --type gui

The VM will PXE boot and be discovered by RackHD,. Then CoreOS will be installed. The installation workflow
included in the RackHD system installs with a default SSH key that is included in our repositories. From the
example directory, you should be able to log in using::

  vagrant ssh:

  cp ~/src/on-http/data/rackhd_rsa ~/.ssh/id_rsa
  chmod 0400 ~/.ssh/id_rsa
  ssh core@172.31.128.2


Using Other Workflows
------------------------------

A number of workflows are loaded by default. Many of those workflows rely on files or directories
that are not packed in this example. If you want to try some of these workflows,
you must add those files manually. You will also need to work around the fact that the example does not
include an out-of-band mechanism for rebooting the PXE VMs.

Unpacking an OS Install ISO
-----------------------------------

You can manually download and unpack ISO files for OS Installs, such as the ESXi installation
ISO or a CentOS 7 LiveCD.

Copy each file into the *examples* directory and unpack it in vagrant::

   vagrant ssh:

   sudo mkdir /var/mirrors
   sudo python ~/src/on-http/data/templates/setup_iso.py \
   /vagrant/VMware-VMvisor-Installer-*.x86_64.iso \
   /var/mirrors --link=/home/vagrant/src
   mv ~/Downloads/CentOS*.iso ~/src/rackhd/example/ vagrant ssh:

   sudo python ~/src/on-http/data/templates/setup_iso.py /vagrant/Cent*.iso \
   /var/mirrors --link=/home/vagrant/src

Modifying These Scripts
--------------------------

If you're using the script or the ansible roles to change the functionality, you can shortcut
the process by using ansible to update the existing VM::

  vagrant provision

Change the Node Version
------------------------------

Currently, the example uses *n* (https://github.com/tj/n) to install node version 0.10.40.
You can change the default node version by logging into the Vagrant instance and using the *n* command as follows::

  vagrant ssh
  sudo ~/n/bin/n <version>

Specify the Number of PXE Clients
------------------------------------

You can change the number of PXE clients that are created by modifying the *pxe_count* parameter in
example/config/monorail_rack.cfg.


Change to a Different Branch
------------------------------------

To checkout to a different commit than what is referenced by git submodule, edit the vagrant file (RackHD/example/Vagrantfile)
 to specify the branch variable for the ansible provisioner. You can do this by uncommenting and modifying the following line::

   ansible.extra_vars = { branch: "master" }


Environment Breakdown
------------------------------

Currently, the *monorail_rack* script cannot shut down or remove anything. To get rid of the RackHD server you can use::

  vagrant destroy

And you can remove the PXE VMs using the vboxmanage command, such as::

  vboxmanage unregistervm --delete pxe-1


Running the Web UI
--------------------------------

We are experimenting with single page web UI applications within the repository
on-web-ui (https://github.com/rackhd/on-web-ui). That repository is set up to host the UI externally to the RackHD.
Follow the README instructions in that repo to run the application.

You can also change the settings to point to the local instance of RackHD at https://localhost:9090/.

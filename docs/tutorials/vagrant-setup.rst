RackHD: Local Vagrant-based  Environtment Set up
================================================

This tutorial gets an instance of RackHD up and running on your local desktop or
laptop, so you can see the hosted API documentation and experiment with the APIs.

Prerequisites
--------------

.. sidebar:: jq

    You can get details on how to use `jq`_ at https://stedolan.github.io/jq/manual/.
    By default it colorizes the syntax highlighting and pretty prints javascript data structures.

    Another option that can provide the same pretty printing is use::

        | python -mjson.tool

You will need to install `Vagrant`_ (>=1.8.1) and `VirtualBox`_ (4.3.x) before setting up environment. The 5.1.X `VirtualBox`_ has some problems. So, you are not suggested installing 5.1.x `VirtualBox`_.

You may also want to consider installing `jq`_ which provides a command-line
oriented tool for pretty printing and filtering JSON structured data.

.. _Vagrant: https://www.vagrantup.com/downloads.html
.. _Virtualbox: https://www.virtualbox.org/wiki/Downloads
.. _jq: https://stedolan.github.io/jq/

.. container:: clearer

   .. image :: ../_static/invisible.png


What We're Setting up
----------------------

.. sidebar:: VirtualBox

    In some high-load cases, you may run into some issues with VirtualBox where it
    locks the fileystem, with the console returning the error message:
    ``rejecting i/o input from offline devices``.

    This is a known issue with VirtualBox, documented in Puppet's `LearningVM bug tracker`_
    and with some additional detail on the `timekeeping`_ that's related.

    The workaround suggested there that seems to resolve the issue is to set Virtualbox CPUs to 1
    and disable the I/O APIC feature when running the virtual machine.

.. _LearningVM bug tracker: https://www.kernel.org/doc/Documentation/virtual/kvm/timekeeping.txt
.. _timekeeping: https://www.kernel.org/doc/Documentation/virtual/kvm/timekeeping.txt


.. image:: ../_static/vagrant_setup.png
     :align: left

The Vagrant instance sets up a pre-installed RackHD VM that connects to one or more VMs
that represent managed systems. The vnode runs in vagrant box which is used to do node discovery and install os on it.

The RackHD VM connects the vnode by the private network, **closenet network**, defined in ``Vagrantfile``. The RackHD VM has two network interfaces. One connects to the local machine via NAT (Network Address Translation)
and the second connects the vnode by a private network. The private network is used so that RackHD DHCP and
PXE operations are isolated from your local network.

The Vagrant setup also enables port forwarding that allows your localhost to access the RackHD instance:

- localhost:9090 redirects to rackhd:8080 for access to the REST API
- localhost:2222 redirects to rackhd:22 for SSH access
- localhost:9093 redirects to rackhd:8443 for secure access to the REST API


Set up RackHD in Vagrant on Linux
-----------------------------------
There are two kinds of vagrant-based environment. One is used for demo and another is used for developing. How to set up these environments is shown as follows.

.. container:: clearer

   .. image :: ../_static/invisible.png

**1. Demo environment set up**

- Clone the RackHD repository

.. code::

    git clone https://github.com/RackHD/RackHD.git
    cd RackHD/example

- Edit the Vagrantfile 
  
  add ``target.vm.box_version = "2.2.0"`` in line 12.

  add ``target.ssh.shell = "bash -c 'BASH_ENV=/etc/profile exec bash'"`` in line 78.

  change the line 79:'sudo service isc-dhcp-server start' to 'sudo service isc-dhcp-server restart'

  change the line 89: 'v.gui = true' to 'v.gui = false'

  [**Note**] version must be 2.* and the version can be searched in https://app.vagrantup.com/rackhd/boxes/rackhd
 
  

- Set up a RackHD vagrant instance

.. code::

    vagrant up dev --provision

- check whether RackHD is set up successfully.

.. code::
   
     vagrant ssh dev
     sudo service rackhd status

If RackHD is set up successfully, the result will be shown as follows.

.. image:: ../_static/check_demo_setup.png
     :align: center

**2. Development Environment Set up**

- Clone the RackHD repository

.. code::

    git clone https://github.com/RackHD/RackHD.git
    
- Set up a RackHD vagrant instance

.. code::
    
    cd RackHD/example/dev-env-setup
    vagrant up dev_ansible --provision

- check whether RackHD is set up successfully.

If the RackHD is setup successfully, the result will be shown as follows.

.. image:: ../_static/install_src_success.png
     :align: center

[**Note**] If you want to pull special source code of RackHD, you can edit the line 8 ``code_version: "release/2.1.0"`` of the file **install_rackhd_vagrant.yml**. 

Besides, only a few people have write permission to RackHD official repositories, usually you only have read-only permission. If you want to contribute code into RackHD official repositority, you firstly need to fork these repositories: `on-core`, `on-tasks`, `on-http`, `on-taskgraph`, `on-syslog`, `on-tftp`, `on-dhcp-proxy` and then submit pull request to RackHD offical repository. So, before installing RackHD, you need to fork the seven repositories into your own github account and  then edit the line ``github_account: "rackhd"`` of the file **install_rackhd_vagrant.yml** and replace **rackhd** with your own github account.

The logs from RackHD will show in the console window where you invoked this last
command. You can use control-c (^C) to stop the processes. Additionally you can
SSH into the local instance using the command ``vagrant ssh dev`` and destroy
this instance with ``vagrant destroy dev``. For more information on Vagrant,
please see the `Vagrant CLI documentation`_.

.. _Vagrant CLI documentation: https://www.vagrantup.com/docs/cli/



Set up RackHD in Vagrant on Windows
-----------------------------------

Prerequisite
~~~~~~~~~~~~

- Ensure your machine has more than 8G physical memory, because RackHD & vNode will use 4G mem. there will be performance impact without enough physical memory.

- Don’t use virtualbox GUI to power on/off/reset the vNode ( quanta_d51). Use vagrant command with “--provision” parameter  (vagrant halt -f quanta_d51 ,      vagrant up quanta_d51 --provision  )

Steps to set up
~~~~~~~~~~~~~~~

There are two kinds of environments for RackHD running in vagrant. One is used for demo and another is used for development. Steps to set up RackHD for the two kinds environment is similar.

**step 1: Install Vagrant & Virtualbox on Windows**

- https://www.virtualbox.org/wiki/Downloads

- https://www.vagrantup.com/downloads.html

**step 2: Create A Vagrantfile (case sensitive ) in Windows**

If you want to set up demo environment, get code from: https://raw.githubusercontent.com/RackHD/RackHD/master/example/Vagrantfile. However, you need to edit the ``Vagrantfile``.

- change the line 79:’sudo service isc-dhcp-server start’ to ‘sudo service isc-dhcp-server restart’

- change the line 89: ‘v.gui = true’ to ‘v.gui = false’

- add target.vm.box_version = "2.2.0" in line 12. 

If you want to set up development environment, get code from: https://github.com/RackHD/RackHD/tree/master/example/dev-env-setup

**step 3:  Right Mouse Click The Folder where Vagrantfile Lives, to launch “git bash here”**

.. image:: ../_static/git_bash_here.png
     :align: center

**step 4:  In “Git Bash”**

1. Type “vagrant up <vm name>”, to start RackHD VM. Take development environment for example:

.. image:: ../_static/vagrant_up_dev_ansible.png
     :align: center

2. Then, start installing RackHD in vagrant.Take development environemnt for example, the result will be shown as follows if RackHD is setup successfully.

.. image:: ../_static/vagrant_src_rackhd_wins.png
     :align: center


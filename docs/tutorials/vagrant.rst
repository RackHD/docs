RackHD: local Vagrant based setup
==================================

This tutorial gets an instance of RackHD up and running on your local desktop or
laptop, so you can see the hosted API documentation and experiment with the APIs.

prerequisites
--------------

.. sidebar:: jq

    You can get details on how to use `jq`_ at https://stedolan.github.io/jq/manual/.
    By default it colorizes the syntax highlighting and pretty prints javascript data structures.

    Another option that can provide the same pretty printing is use::

        | python -m json.tool

    which can also be used with a pager, such as `less`.

You will need to have `Vagrant`_ and `VirtualBox`_ installed on your machine to use
this tutorial.

You may also want to consider installing `jq`_ which provides a command-line
oriented tool for pretty printing and filtering JSON structured data.

.. _Vagrant: https://www.vagrantup.com/downloads.html
.. _Virtualbox: https://www.virtualbox.org/wiki/Downloads
.. _jq: https://stedolan.github.io/jq/

.. container:: clearer

   .. image :: ../_static/invisible.png


what we're setting up
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

.. image:: ../_static/vagrant_setup.jpg
     :height: 300
     :align: left

The Vagrant instance sets up a pre-installed RackHD VM that connects to one or more VMs
that represent managed systems. The target systems are simulated using PXE clients.

The RackHD VM has two network interfaces. One connects to the local machine via NAT (Network Address Translation)
and the second connects to the PXE VMs in a private network. The private network is used so that RackHD DHCP and
PXE operations are isolated from your local network.

The Vagrant setup also enables port forwarding that allows your localhost to access the RackHD instance:

- localhost:9090 redirects to rackhd:8080 for access to the REST API
- localhost:2222 redirects to rackhd:22 for SSH access
- localhost:9093 redirects to rackhd:8443 for secure access to the REST API

.. container:: clearer

   .. image :: ../_static/invisible.png

- Clone the RackHD repository

.. code::

    git clone https://github.com/rackhd/rackhd
    cd rackhd/example

- Download a RackHD vagrant instance

.. code::

    vagrant up dev

- Start the local instance

.. code::

    vagrant ssh dev -c "sudo pm2 start rackhd-pm2-config.yml"

The logs from RackHD will show in the console window where you invoked this last
command. You can use control-c (^C) to stop the processes. Additionally you can
SSH into the local instance using the command ``vagrant ssh dev`` and destroy
this instance with ``vagrant destroy dev``. For more information on Vagrant,
please see the `Vagrant CLI documentation`_.

.. _Vagrant CLI documentation: https://www.vagrantup.com/docs/cli/

Accessing your local instance of RackHD
----------------------------------------

.. sidebar:: Resetting the demonstration

    You can reset all of the demonstration by tearing down and setting up the vagrant
    instances again::

        vagrant destroy -f
        vagrant up dev
        vagrant ssh dev -c "sudo pm2 start rackhd-pm2-config.yml"


    **Resetting and updating the code to the latest master branch**

    The demonstration instance of RackHD is installed from source, so it can also be
    updated the latest version::

        vagrant destroy -f
        vagrant up dev
        vagrant ssh dev

    And then within that virtual machine::

        cd ~/src
        ./scripts/clean_all.bash
        ./scripts/reset_submodules.bash
        ./scripts/link_install_locally.bash

    .. WARNING::
        This downloads the latest code and reinstalls it all from source, which can take a few minutes.

    Once that is complete, you can exit your SSH sessions with the VM and start all the services::

        vagrant ssh dev -c "sudo pm2 start rackhd-pm2-config.yml"

When RackHD is operational, the self-hosted API documentation should immediately
be available:

- 1.1 API documentation at http://localhost:9090/docs
- 2.0 and Redfish API documentation at http://localhost:9090/swagger-ui
- Included task documentation at http://localhost:9090/taskdoc
- a developer UI to live updates of RackHD at http://localhost:9090/ui

The self-hosted documentation for 2.0 and the Redfish API includes the ability to
invoke sample commands directly on your local instance through the documentation pages.

You can also interact with RackHD using ``curl`` from the commandline, or a tool
such as `Postman`_ in the browser.

.. _Postman: https://www.getpostman.com

Getting a token
---------------
Get a token for authentication. The token will be included in the header of each REST API call::

    curl -k -X POST -H "Content-Type:application/json" https://localhost:9093/login \
    -d '{"username":"admin", "password":"admin123" }' | python -m json.tool

.. code-block:: JSON

    {
        "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyIjoiYWRtaW4iLCJpYXQiOjE0NTU2MTI5MzMsImV4cCI6MTQ1NTY5OTMzM30.glW-IvWYDBCfDZ6cS_6APoty22PE_Ir5L1mO-YqO3eE"
    }

With a brand new instance, you should be able to access the ``nodes/`` API endpoint
and see an empty list of nodes. In the following curl command, <token> is the token
obtained previously in the tutorial.

- ``curl -k https://localhost:9093/api/2.0/nodes -H 'Authorization: JWT <token>'| jq``

.. code-block:: JSON

    []

You can also view a list of all the built-in workflows

- ``curl -k https://localhost:9093/api/2.0/workflows/graphs -H 'Authorization: JWT <token>' | jq``

.. code-block:: JSON

    [
        {
            "friendlyName": "Arista Switch ZTP Discovery",
            "injectableName": "Graph.Switch.Discovery.Arista.Ztp",
            "tasks": [
                {
                    "label": "catalog-switch",
                    "taskDefinition": {
                        "friendlyName": "Catalog Arista Switch",
                        "implementsTask": "Task.Base.Linux.Commands",
                        "injectableName": "Task.Inline.Catalog.Switch.Arista",
                        "options": {
                            "commands": [
                                {
                                    "catalog": {
                                        "format": "json",
                                        "source": "version"
                                    },
                                    "downloadUrl": "{{ api.templates }}/arista-catalog-version.py?nodeId={{ task.nodeId }}"
                                }
                            ]
                        },
                        "properties": {}
                    }
                }
            ]
        },
        ...

If you want to just see the names of the workflows:

- ``curl -k https://localhost:9093/api/2.0/workflows/graphs -H 'Authorization: JWT <token>' | jq '.[]["injectableName"]'``

.. code-block:: JSON

    "Graph.Switch.Discovery.Arista.Ztp"
    "Graph.BootLiveCD"
    "Graph.Bootstrap.With.BMC.Credentials.Remove"
    "Graph.Bootstrap.With.BMC.Credentials.Setup"
    "Graph.Bootstrap.Decommission.Node"
    "Graph.BootstrapUbuntu"
    "Graph.Switch.Discovery.Brocade.Ztp"
    "Graph.Switch.Discovery.Cisco.Poap"
    "Graph.ClearSEL.Node"
    "Graph.Emc.Redfish.FabricService.Poller.Create"
    "Graph.Obm.Ipmi.CreateSettings"
    "Graph.Raid.Create.MegaRAID"
    "Graph.Redfish.Chassis.Poller.Create"
    "Graph.Redfish.Managers.Poller.Create"
    "Graph.Redfish.Systems.Poller.Create"
    "Graph.Obm.Vbox.CreateSettings"
    "Graph.Raid.Delete.MegaRAID"
    "Graph.Dell.Disable.VTx"
    "Graph.Dell.Enable.VTx"
    "Graph.Dell.Racadm.GetBIOS"
    "Graph.Dell.Racadm.GetConfigCatalog"
    "Graph.Dell.Racadm.SetBIOS"
    "Graph.Dell.Racadm.Update.Firmware"
    "Graph.Discovery"
    "Graph.Mgmt.Discovery"
    "Graph.MgmtSKU.Discovery"
    "Graph.Refresh.Delayed.Discovery"
    "Graph.Refresh.Immediate.Discovery"
    "Graph.SKU.Discovery"
    "Graph.Emc.Compose.System"
    "Graph.Emc.Redfish.Catalog"
    "Graph.BootstrapUbuntuMocks"
    "Graph.Flash.LSI.MegaRAID"
    "Graph.Flash.Quanta"
    "Graph.Flash.Quanta.BIOS"
    "Graph.Flash.Quanta.Bmc"
    "Graph.Flash.Quanta.MegaRAID"
    "Graph.GenerateSku"
    "Graph.GenerateTags"
    "Graph.InstallCentOS"
    "Graph.InstallCoreOS"
    "Graph.InstallESXi"
    "Graph.InstallPhotonOS"
    "Graph.InstallRHEL"
    "Graph.InstallSUSE"
    "Graph.InstallUbuntu"
    "Graph.InstallWindowsServer"
    "Graph.Catalog.Intel.Flashupdt"
    "Graph.Service.IscDhcpLeasePoller"
    "Graph.McReset"
    "Graph.noop-example"
    "Graph.PDU.Discovery"
    "Graph.Service.Poller"
    "Graph.PowerOff.Node"
    "Graph.PowerOn.Node"
    "Graph.Reboot.Node"
    "Graph.Redfish.Discovery"
    "Graph.Redfish.Actions.Reset"
    "Graph.Remove.Bmc.Credentials"
    "Graph.Reset.Node"
    "Graph.Run.Emc.Diag"
    "Graph.Drive.SecureErase"
    "Graph.Set.Bmc.Credentials"
    "Graph.ShellCommands"
    "Graph.Reset.Soft.Node"
    "Graph.SKU.Switch.Discovery.Active"
    "Graph.Switch.Discovery"
    "Graph.Switch.SKU.Discovery.Hooks.Post"
    "Graph.RunUefi"
    "Graph.BootstrapWinPE"
    "Graph.Write.Quanta.BIOS.NVRAM"
    "Graph.Arista.Zerotouch.vEOS"

Or review the list of all the built-in tasks available to be used in workflows

- ``curl -k https://localhost:9093/api/2.0/workflows/tasks -H 'Authorization: JWT <token>' | jq``

.. code-block:: JSON

    [
      {
        "friendlyName": "Boot LiveCD",
        "injectableName": "Task.Os.Boot.LiveCD",
        "implementsTask": "Task.Base.Os.Install",
        "options": {
          "profile": "boot-livecd.ipxe",
          "version": "livecd",
          "repo": "{{api.server}}/LiveCD/{{options.version}}"
        },
        "properties": {
          "os": {
            "linux": {
              "distribution": "livecd"
            }
          }
        }
      },
      ...

Adding a simulated server
---------------------------

The Vagrantfile included in the example setup includes a reference to a simulated
server provided by the `InfraSim`_ project. You can download and boot this simulated
server, which includes an interface to IPMI as well as simulates the physical machine
with an internal VM.

.. _InfraSim: http://infrasim.readthedocs.io

By default, RackHD will PXE boot this instance, interrogate it, and then leave it alone.

- Set up the simulated server

.. code::

    vagrant up quanta_d51

This command will start up vagrant with the GUI console available. You can see
the Quanta d51 control with the vBMC quanta simulator by using VNC to connect
to 127.0.0.1:15901 (or 127.0.0.1 display 10001). You can log into the VM hosting
this simulation with the default credentials of username ``root``, and password ``root``.

The IPMI credentials that it is providing on ``closednet`` use the username ``admin``
and password ``admin``.

Once the node has been discovered by RackHD, you can see it through the API.

- ``curl -k https://localhost:9093/api/2.0/nodes -H 'Authorization: JWT <token>' | jq``

.. code-block:: JSON

    [
        {
            "autoDiscover": false,
            "id": "57967193a045ba7c0800207b",
            "identifiers": [],
            "name": "Enclosure Node QTFCJ05160195",
            "obms": [],
            "tags": [],
            "type": "enclosure"
        },
        {
            "autoDiscover": false,
            "id": "579680825d434579084ff910",
            "identifiers": [
                "52:54:be:ef:aa:ee"
            ],
            "name": "52:54:be:ef:aa:ee",
            "obms": [],
            "sku": null,
            "tags": [],
            "type": "compute"
        }
    ]

Viewing the geneaology
---------------------------

You can view all of the information collected about a specific node through the
``catalogs`` URI. For the example above, using the node with the ID **579680825d434579084ff910**:

- ``curl -k https://localhost:9093/api/2.0/nodes/579680825d434579084ff910/catalogs -H 'Authorization: JWT <token>' | jq``

.. code-block:: JSON

    [
      {
        "node": "579680825d434579084ff910",
        "source": "dmi",
        "data": {
          "BIOS Information": {
            "Vendor": "American Megatrends Inc.",
            "Version": "S2B_3A17",
            "Release Date": "11/07/2014",
            "Address": "0xF0000",
            "Runtime Size": "64 kB",
            "ROM Size": "8192 kB",
            "Characteristics": [
              "PCI is supported",
              "BIOS is upgradeable",
              "BIOS shadowing is allowed",
              "Boot from CD is supported",
              "Selectable boot is supported",
              "BIOS ROM is socketed",

There are a large number of sources provided by default, and these can be extended with
additional cataloging tasks. A quick way to see all the catalogs for a node:


- ``curl -k https://localhost:9093/api/2.0/nodes/579680825d434579084ff910/catalogs -H 'Authorization: JWT <token>' | jq '.[]["source"]'``

.. code-block:: JSON

    "dmi"
    "ohai"
    "bmc"
    "ipmi-sel-information"
    "ipmi-sel"
    "ipmi-mc-info"
    "ipmi-user-summary-1"
    "ipmi-user-list-1"
    "ipmi-fru"
    "ipmi-user-summary-2"
    "ipmi-user-list-2"
    "rmm-user-summary"
    "rmm-user-list"
    "ipmi-user-summary-4"
    "ipmi-user-list-4"
    "ipmi-user-summary-5"
    "ipmi-user-list-5"
    "ipmi-user-summary-6"
    "ipmi-user-list-6"
    "ipmi-user-summary-7"
    "ipmi-user-list-7"
    "ipmi-user-summary-8"
    "ipmi-user-list-8"
    "ipmi-user-summary-9"
    "ipmi-user-list-9"
    "ipmi-user-summary-10"
    "ipmi-user-list-10"
    "ipmi-user-summary-11"
    "ipmi-user-list-11"
    "ipmi-user-summary-12"
    "ipmi-user-list-12"
    "ipmi-user-summary-13"
    "ipmi-user-list-13"
    "ipmi-user-summary-14"
    "ipmi-user-list-14"
    "ipmi-user-summary-15"
    "ipmi-user-list-15"
    "lspci"
    "lshw"
    "lsscsi"
    "smart"
    "driveId"

You can request a specific catalog by appending its source identifier onto the
catalogs URI:

- ``curl -k https://localhost:9093/api/2.0/nodes/579680825d434579084ff910/catalogs/bmc -H 'Authorization: JWT <token>' | jq``

.. code-block:: JSON

    {
      "node": "579680825d434579084ff910",
      "source": "bmc",
      "data": {
        "Set in Progress": "Set Complete",
        "Auth Type Support": "NONE MD2 MD5 PASSWORD",
        "Auth Type Enable": {
          "Callback": "NONE MD2 MD5 PASSWORD ",
          "User": "NONE MD2 MD5 PASSWORD ",
          "Operator": "NONE MD2 MD5 PASSWORD ",
          "Admin": "NONE MD2 MD5 PASSWORD ",
          "OEM": ""
        },
        "IP Address Source": "DHCP Address",
        "IP Address": "172.31.128.2",
        "Subnet Mask": "255.255.252.0",
        "MAC Address": "08:00:27:49:6a:f7",
        "SNMP Community String": "public",
        "IP Header": "TTL=0x00 Flags=0x00 Precedence=0x00 TOS=0x00",
        "Default Gateway IP": "172.31.128.254",
        "Default Gateway MAC": "00:00:00:00:00:00",
        "Backup Gateway IP": "0.0.0.0",
        "Backup Gateway MAC": "00:00:00:00:00:00",
        "802_1q VLAN ID": "Disabled",
        "802_1q VLAN Priority": "0",
        "RMCP+ Cipher Suites": "1,2,3,4,5,6,7,8,9,10,11,12,13,14,15",
        "Cipher Suite Priv Max": [
          "aaaaaaaaaaaaaaa",
          "X=Cipher Suite Unused",
          "c=CALLBACK",
          "u=USER",
          "o=OPERATOR",
          "a=ADMIN",
          "O=OEM"
        ]
      },
      "createdAt": "2016-07-25T21:15:10.609Z",
      "updatedAt": "2016-07-25T21:15:10.609Z",
      "id": "05efeab1-f835-413d-b472-2ccaa6839196"
    }

And one of the most commonly used catalogs to identify hardware is the source `dmi`:

- ``curl -k https://localhost:9093/api/2.0/nodes/579680825d434579084ff910/catalogs/dmi -H 'Authorization: JWT <token>' | jq``

.. code-block:: JSON

    {
        "createdAt": "2016-07-25T21:13:24.417Z",
        "data": {
            "BIOS Information": {
                "Address": "0xF0000",
                "BIOS Revision": "5.6",
                "Characteristics": [
                    "PCI is supported",
                    "BIOS is upgradeable",
                    "BIOS shadowing is allowed",
                    "Boot from CD is supported",
                    "Selectable boot is supported",
                    "BIOS ROM is socketed",
                    "EDD is supported",
                    "Print screen service is supported (int 5h)",
                    "8042 keyboard services are supported (int 9h)",
                    "Serial services are supported (int 14h)",
                    "Printer services are supported (int 17h)",
                    "ACPI is supported",
                    "USB legacy is supported",
                    "BIOS boot specification is supported",
                    "Targeted content distribution is supported",
                    "UEFI is supported"
                ],
                "Firmware Revision": "3.17",
                "ROM Size": "8192 kB",
                "Release Date": "11/07/2014",
                "Runtime Size": "64 kB",
                "Vendor": "American Megatrends Inc.",
                "Version": "S2B_3A17"
            },

Keys in this data which are common interesting include

- ``| jq '.["data"]["Base Board Information"]'``
  - convenient access to motherboard serial numbers and asset tags

.. code-block:: JSON

    {
        "Manufacturer": "Quanta Computer Inc",
        "Product Name": "S2B-MB (dual 10G LoM)",
        "Version": "31S2BMB0040",
        "Serial Number": "QTF4J051400040",
        "Asset Tag": "",
        "Features": [
          "Board is a hosting board",
          "Board is replaceable"
        ],
        "Location In Chassis": "To be filled by O.E.M.",
        "Chassis Handle": "0x0003",
        "Type": "Motherboard",
        "Contained Object Handles": "0"
    }

- ``| jq '.["data"]["Chassis Information"]'``
  - convenient access to serial numbers and asset tags

.. code-block:: JSON

    {
      "Manufacturer": "Quanta Computer Inc",
      "Type": "Rack Mount Chassis",
      "Lock": "Not Present",
      "Version": "To be filled by O.E.M.",
      "Serial Number": "QTFCJ05160195",
      "Asset Tag": "",
      "Boot-up State": "Safe",
      "Power Supply State": "Safe",
      "Thermal State": "Safe",
      "Security Status": "None",
      "OEM Information": "0x00000000",
      "Height": "Unspecified",
      "Number Of Power Cords": "1",
      "Contained Elements": [
        "<OUT OF SPEC> (0)"
      ],
      "SKU Number": "To be filled by O.E.M."
    }

- ``| jq '.["data"]["System Information"]'``
  - convenient access to chassis serial number

.. code-block:: JSON

    {
      "Manufacturer": "Quanta Computer Inc",
      "Product Name": "D51B-2U (dual 10G LoM)",
      "Version": "To be filled by O.E.M.",
      "Serial Number": "SerialNumber",
      "UUID": "75277866-7C0D-1000-A5B1-2C600C8374BD",
      "Wake-up Type": "Power Switch",
      "SKU Number": "To be filled by O.E.M.",
      "Family": "To be filled by O.E.M."
    }

Adding a SKU definition
-------------------------

All this geneaology can be used to create rules that will uniquely identify a
type of machine, a feature which we call **SKU** in RackHD. When a node is discovered,
one the last steps of the built in discovery workflow is to compare the node against
all existing SKU definitions. If the SKU definition maps, it is applied. Only one
SKU will be assigned to a node at a time - there's a similiar feature called **tag**
that can be used to group multiple nodes with the same characteristics.

You can get more details on SKUs at :doc:`../rackhd/skus`, and tags at :doc:`../rackhd/tags`.

One of the specific benefits of a SKU is that you can define an additional workflow
to be invoked as soon as the node is discovered, providing RackHD with an explicit
set of tasks to follow when the node is identified. This is how you set up RackHD to
automatically install CentOS, for example.

You create a SKU by uploading a specially structured bit of JSON through the API. In
the `example/` directory, we have a few specific examples pre-set to work with this
tutorial. For this example, we'll add in a specific workflow to install CentOS, and a
SKU definition which will use the simulated hardware to trigger that workflow.

.. sidebar:: Installing the CentOS Installation ISO

    To operate correctly, the CentOS install workflow we just added expects to find the
    OS installation files in the directory ``/CentOS/7.0``, which doesn't exist by default
    on our instance of RackHD.

    We can install the relevant files by downloading and unpacking a CentOS installation
    ISO. To do so, log into your instance of RackHD using a command like ``vagrant dev ssh``
    and then invoke the following commands::

        sudo mkdir -p /var/mirrors
        cd /tmp
        wget http://mirrors.mit.edu/centos/7/isos/x86_64/CentOS-7-x86_64-DVD-1511.iso
        # 4GB download!
        sudo python ~/src/on-tools/scripts/setup_iso.py \
        /tmp/CentOS-7-x86_64*.iso /var/mirrors \
        --link=/home/vagrant/src

- add a `default vQuanta workflow`_ to install CentOS for this specific SKU::

    cd ~/src/rackhd/example
    # make sure you're in the example directory to reference the sample JSON correctly

    curl -k -H "Content-Type: application/json" -H 'Authorization: JWT <token>' \
    -X PUT --data @samples/vQuanta_default_workflow.json \
    https://localhost:9093/api/2.0/workflows/graphs


- add the `vQuanta SKU definition`_ for our simulated hardware::

    cd ~/src/rackhd/example
    # make sure you're in the example directory to reference the sample JSON correctly

    curl -k -H "Content-Type: application/json" -H 'Authorization: JWT <token>' \
    -X POST --data @samples/vQuanta_d51_sku.json \
    https://localhost:9093/api/2.0/skus

.. _default vQuanta workflow:  https://github.com/RackHD/RackHD/blob/master/example/samples/vQuanta_default_workflow.json
.. _vQuanta SKU definition:  https://github.com/RackHD/RackHD/blob/master/example/samples/vQuanta_d51_sku.json

When you add a SKU, the system will check all existing nodes against the definition for that SKU and update
the nodes to assign the SKU where it's relevant. If the SKU definitions includes any default workflows, those
will **not** get automatically invoked when you create the SKU definition. The default workflow path will
only operate when a node is first being discovered; or more specifically correct when the
``Graph.SKU.Discovery`` workflow is run against the node.


.. container:: clearer

   .. image :: ../_static/invisible.png

.. warning:: **SLOW ON VAGRANT**

    The simulated hardware is a virtual machine inside another virtual machine, so
    while this process works, it is very slow on most desktops. To see it operational
    you will want probably want watch the console on the simulated hardware using VNC.

    On a Macbook Pro (2.2 GHz Intel Core i7) with 16GB of ram, this process takes
    approximately 2 hours to fully complete. Using real hardware, the process is in minutes.

Invoking a workflow
--------------------

Almost all the workflows you'll want to invoke start with controlling the node
remotely, most commonly to tell the node to reboot and start a PXE boot process. The
simplest possible workflows just power off or power on a node. By default a node
will not have any OBM settings defined.


Checking and setting the OBM settings for a node
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

You can check to see if any OBM settings are defined on the node using the nodes
API:

``curl -k https://localhost:9093/api/2.0/nodes/57990efa0d76e7c207cdfc3f -H 'Authorization: JWT <token>' | jq``

.. code-block:: JSON

    {
      "autoDiscover": false,
      "id": "57990efa0d76e7c207cdfc3f",
      "name": "52:54:be:ef:40:98",
      "identifiers": [
        "52:54:be:ef:40:98"
      ],
      "tags": [],
      "obms": [],
      "type": "compute"
    }

If the node has an OBM service, the key ``obms`` will have some data in it:

.. code-block:: JSON

    {
      "autoDiscover": false,
      "id": "57990efa0d76e7c207cdfc3f",
      "name": "52:54:be:ef:40:98",
      "identifiers": [
        "52:54:be:ef:40:98"
      ],
      "tags": [],
      "obms": [
        {
          "service": "ipmi-obm-service",
          "ref": "/api/2.0/obms/5799101e95d9a2bf0780128a"
        }
      ],
      "type": "compute"
    }

If the node does not have any OBM settings, you will want to provide one - as nearly all
workflows start by utilizing the OBM settings. When you're creating an OBM setting
via the API, you'll need to provide both the node ID and a relevant "host" setting for
accesing the out of band management interface.

For the node in the examples above, that could be:

.. code-block:: REST

    curl -k -X PUT \
        -H 'Content-Type: application/json' -H 'Authorization: JWT <token>' \
        -d '{ "nodeId": "5799151faa0559c007dab5e3", "service": "ipmi-obm-service", "config": { "user": "admin", "password": "admin", "host": "52:54:be:ef:9d:3d" } }' \
        https://localhost:9093/api/2.0/obms

Power Off
^^^^^^^^^^^

.. code-block:: REST

    curl -k -X POST \
        -H 'Content-Type: application/json' -H 'Authorization: JWT <token>' \
        -d '{"name": "Graph.PowerOff.Node"}' \
        https://localhost:9093/api/2.0/nodes/5799151faa0559c007dab5e3/workflows

Power On
^^^^^^^^^^^

.. code-block:: REST

    curl -k -X POST \
        -H 'Content-Type: application/json' -H 'Authorization: JWT <token>' \
        -d '{"name": "Graph.PowerOn.Node"}' \
        https://localhost:9093/api/2.0/nodes/5799151faa0559c007dab5e3/workflows

Install OS
^^^^^^^^^^^

.. code-block:: REST

    cd ~/src/rackhd/examples
    curl -k -X POST \
        -H 'Content-Type: application/json' -H 'Authorization: JWT <token>' \
        --data @samples/centos_iso_boot.json \
        https://localhost:9093/api/2.0/nodes/579680825d434579084ff910/workflows

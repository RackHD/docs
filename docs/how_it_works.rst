How It Works
============

General bare metal automation with PXE
--------------------------------------

As we shared in :doc:`introduction`, the RackHD system leverages standard open source tools
and protocols. `PXE`_ is the most important, originally created as a specification by Intel
on how to have servers boot off a network. To PXE boot a machine, you need to minimally have
`DHCP`_, and `TFTP`_ configured and responding on the network to which the machine is attached. We
additionally leverage iPXE `bootloader`_, which takes advantage of HTTP and allows us to
dynamically generate iPXE scripts (which we call profiles) based on what we need the server
to do when it is PXE booting.

The mechanism that allows us to automate this for datacenter style servers is that most of
those are built with what's called a `BMC`_. Most servers that you'd want to control remotely
like this (datacenter style servers) support the protocol `IPMI`_, which gives us the ability
to power on, power off, reboot, and request a machine to PXE boot, among other possibilities.

Many open source tools, such as `Cobbler`_, `Razor`_, and `Hanlon`_ use this kind of mechanism.
RackHD goes beyond this and adds a workflow engine that interacts with these existing protocols
and mechanisms to let us create workflows of tasks, boot scripts, and interactions to achieve
our full system automation.

.. _Cobbler: http://cobbler.github.io
.. _Razor: https://github.com/puppetlabs/razor-server
.. _Hanlon: https://github.com/csc/Hanlon


RackHD Discovery
----------------

RackHD supports two modes of learning about machines that it manages. We loosely group
these as *passive* and *active* discovery.

* Passive discovery is where a user or outside system actively tells RackHD that the system exists.
  This is enabled by making a post to the REST interface that RackHD can then add to its data model.

* Active discovery is invoked when a machine attempts to PXE boot on the network that RackHD is
  monitoring. As a new machine PXE boots, RackHD retrieves the MAC address of the machine.
  If the MAC address has not been recorded, RackHD creates a new record in the data model and
  then invokes a default workflow. To enable active discovery, you set the default workflow that
  will be run when a new machine is identified to one of the discovery workflows included
  within the system. The most common is the SKU Discovery workflow.

For an example, the "SKU Discovery" workflow runs through its tasks as follows:

1. It runs a sub-workflow called 'Discovery'

   a) Discovery is initiated by sending down the iPXE boot loader with a pre-built script to run
      within iPXE. This script then chainloads into a new, dynamically rendered iPXE script that interrogates
      the enabled network interfaces on the remote machine and reports them back to RackHD. RackHD adds
      this information to the machine and lookup records. RackHD then renders an additional iPXE script
      to be chainloaded that downloads and runs the microkernel. The microkernel boots up and requests a
      Node.js "bootstrap" script from RackHD. RackHD runs the bootstrap program which uses a simple REST
      API to "ask" what it should do on the remote host.

   b) The workflow engine, running the discovery
      workflow, provides a set of tasks to run. These tasks are matched with parsers in RackHD to understand
      and store the output. They work together to run Linux commands that interrogate the hardware from the
      microkernel running in memory. These commands include interrogating the machine's BMC settings through
      IPMI, the installed PCI cards, the DMI information embedded in the BIOS, and others. The resulting
      information is then stored in JSON format as "catalogs" in RackHD.

   c) When it's completed with all the tasks, it tells the microkernel to reboot the machine and sends an
      internal event that the basic bootstrapping process is finished

2. The SKU Discovery workflow then performs a workflow task process called "generate-sku" that compares the
   catalog data for the node against SKU definition loaded into the system through the REST interface. If
   the definitions match, RackHD updates its data model indicating that the node belongs to a SKU.

4. The task "generate-enclosure" uses the IPMI channels to set up a connection to the BMC so that it can control that
   node in the future (power on, off and reboot).

5. The last task ("create-default-pollers") creates a set of default pollers that periodically monitor the
   device for system hardware alerts, built in sensor data, power status, and similar information.

You can find the SKU Discovery graph at https://github.com/RackHD/on-taskgraph/blob/master/lib/graphs/discovery-sku-graph.js,
and the simpler "Discovery" graph it uses at https://github.com/RackHD/on-taskgraph/blob/master/lib/graphs/discovery-graph.js

**Notes:**

* No workflow is assigned to a PXE-booting system that is already known to RackHD. Instead, the
  RackHD system ignores proxy DHCP requests from booting nodes with no active workflow and lets
  the system continue to boot as specified by its BIOS or UEFI boot order.

* The discovery workflow can be updated to do additional work or steps for the installation of RackHD,
  to run other workflows based on the SKU analysis, or perform other actions based on the logic embedded
  into the workflow itself.

* Additional pollers exist and can be configured to capture data through SNMP. The RackHD project is set
  up to support additional pollers as plugins that can be configured and run as desired.


Telemetry and Alerting
----------------------

Poller information is converted into a "live data feed" and published through
AMQP, providing a "live telemetry feed" for the raw data collected on the
remote systems (recent data is also cached for manual access).

In addition to
this live feed, RackHD includes some rudimentary
alerting mechanisms that compare the data collected by the pollers to regular
expressions, and if they match, create an additional event that is published on
an "alert" exchange in AMQP.

Other Workflows
---------------

Other workflows can be configured and assigned to run on remote systems. For
example, *OS install* can be set to explicitly power cycle (reboot) a remote
node. As the system PXE boots, an installation kernel is sent down and run
instead of the discovery microkernel.

The remote network-based OS installation process that runs from Linux OS
distributions typically runs with a configuration file - *debseed* or *kickstart*.
The monorail engine provides a means to render these configuration files
through templates, with the values derived from the workflow itself - either as
defaults built into the workflow, discovered data in the system (such as data
within the catalogs found during machine interrogation), or even passed in as
variables when the workflow was invoked by an end-user or external automation
system. These "templates" can be accessed through the Monorail's engine REST
API - created, updated, or removed - to support a wide variety of responses and
capabilities.

Workflows can also be chained together and the workflow engine includes
simple logic (as demonstrated in the discovery workflow) to perform arbitrarily
complex tasks based on the workflow definition. The workflow definitions
themselves are accessible through the Monorail engine's REST API as a "graph"
of "tasks". Both graphs and tasks are fully declarative with a JSON format.
Tasks are also mapped up "Jobs", which is the Node.js code that's included
within RackHD.

Workflow Graphs
^^^^^^^^^^^^^^^^^
The graphs/workflows API (workflows is a backwards-compatible term for graphs) provides
functionality for running tasks via
graph-based control flow mechanisms. A typical graph consists of a list of
tasks which themselves are essentially decorated functions.

For more detailed information on graphs, see the section on :doc:`rackhd/graphs`
under our :doc:`development_guide`.

Workflow Tasks
^^^^^^^^^^^^^^^^^
A workflow task is a unit of work decorated with data and logic that allows it to
be included and run within a workflow. Tasks can be
defined to do wide-ranging operations, such as bootstrap a server node into a
Linux microkernel, parse data for matches against a rule, and others. The tasks in a workflow are run in a specific order.

For more detailed information on tasks, see the section on :doc:`rackhd/tasks`
under our :doc:`development_guide`.


Glossary
--------

PXE
^^^

The `Preboot Execution Environment (PXE)`_ is a vendor-independent mechanism that allows networked computers
to be remotely booted and configured. PXE has been used to run diskless computers. Common Linux operating systems (Red Hat kickstart, Debian debseed,
and SUSE YaST) have adopted a network-based install that can be initiated and leveraged using PXE.

.. _Preboot Execution Environment: https://en.m.wikipedia.org/wiki/Preboot_Execution_Environment

PXE was originally developed by Intel in 1998/1999. Once provided through the network interface cards, it
is now built into most BIOS or UEFI implementations. The PXE specification is still available for download
at Intel: http://www.intel.com/design/archives/wfm/downloads/pxespec.htm


DHCP
^^^^

DHCP is an expansive protocol with many extensions that are leveraged to provide network
configuration information to other computers on the same "broadcast"
segment. It communicates using raw sockets or TCP/IP.

Because DHCP services operate at Layer 2 of the networking stack, they must be carefully controlled
so as to prevent the generation of IP address conflicts on the same network segment. Most switches do
not pass DHCP traffic across networks unless specifically configured to do so with a DHCP relay.

RackHD uses a DHCP Proxy server to support PXE operations. It sends auxiliary boot information to
clients, like the boot filename, tftp server or rootpath, but leaves generation of IP addresses to
the DHCP server.

TFTP
^^^^

TFTP provides simple, unauthenticated file transfer over TCP/IP. It is a widely-used protocol
for remote booting or installation.

Due to the simplicity of TFTP, it is not terribly robust in the face of failures or
temporary network outages -- and can be somewhat unreliable at scale or high load.

bootloader
^^^^^^^^^^

RackHD uses TFTP to transfer tiny executable programs that are used to initialize
hardware and set up additional systems in order to "boot and run" a larger operating
system. PXELINUX_ and iPXE_ (evolved from earlier gPXE) are most commonly used.

PXElinux heavily leverages TFTP and is a fairly static system. iPXE includes a small
scripting interpreter and supports downloading additional files for booting (such
as a WinPE or Kernel and Initrd file for Linux) over HTTP as a more reliable transport
protocol.

.. _PXELINUX: http://www.syslinux.org/wiki/index.php/Doc/pxelinux
.. _iPXE: http://ipxe.org

IPMI
^^^^

The `Intelligent Platform Management Interface (IPMI)`_ is the protocol by which BMCs can
manage and monitor servers independent of the CPU, firmware (BIOS or UEFI), and operating
system. BMCs typically communicate on the network using the IPMI_ protocol.

.. _Intelligent Platform Management Interface (IPMI): https://en.m.wikipedia.org/wiki/Intelligent_Platform_Management_Interface

Although IPMI supports authentication, many security researchers
have shown that IPMI is easily exploited:

* `A Penetration Tester's Guide to IPMI and BMCs`_
* `Many servers expose insecure out-of-band management interfaces to the Internet`_
* `IPMI The most dangerous protocol you've never heard of`_


.. _A Penetration Tester's Guide to IPMI and BMCs: https://community.rapid7.com/community/metasploit/blog/2013/07/02/a-penetration-testers-guide-to-ipmi
.. _Many servers expose insecure out-of-band management interfaces to the Internet: http://www.pcworld.com/article/2361040/many-servers-expose-insecure-outofband-management-interfaces-to-the-internet.html
.. _IPMI The most dangerous protocol you've never heard of: http://www.itworld.com/article/2708437/security/ipmi--the-most-dangerous-protocol-you-ve-never-heard-of.html


Due to security weaknesses, most data center networks secure and highly control
the access to networks where IPMI is enabled.

Many hardware vendors provide proprietary DHCP extensions that support additional information or functionality.
Some proprietary management tools leverage these protocols to provide additional vendor-specific functionality.

BMC
^^^

A `Baseboard Motherboard Controller`_ (BMC) is a microcontroller (small computer) embedded on the motherboard of
data center servers. The BMC provides the interface that enables out-of-band management of the server using IPMI.

.. _Baseboard Motherboard Controller: https://en.m.wikipedia.org/wiki/Baseboard_management_controller

Some BMCs provide a web-based interface that adds additional proprietary features,
such as the ability to remotely mount ISO files or other media.

Because the BMC has a separate MAC address, it can be connected to the network using a dedicated or shared Ethernet port.

Note: Intel has recently been released desktops and client systems with roughly
equivalent management tooling called `Active Management Technology`_ (AMT). AMT uses its
own protocol that supports the DMTF standard DASH. It runs over HTTP/HTTPS and generally
leverages the `WS-Management`_ server management standards.

.. _Active Management Technology: https://en.m.wikipedia.org/wiki/Intel_Active_Management_Technology
.. _WS-Management: https://en.m.wikipedia.org/wiki/WS-Management

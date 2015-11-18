How It Works
============

General Bare Metal Automation with PXE
--------------------------------------

RackHD uses the `Preboot Execution Environment (PXE)`_ for booting machines. PXE is a vendor-independent mechanism that
allows networked computers to be remotely booted and configured. PXE booting requires that `DHCP`_ and `TFTP`_
are configured and responding on the network to which the machine is attached.

.. _DHCP: http://en.wikipedia.org/wiki/Dynamic_Host_Configuration_Protocol
.. _TFTP: https://en.wikipedia.org/wiki/Trivial_File_Transfer_Protocol

In addition, RackHD commonly uses `iPXE`_ as its initial bootloader. iPXE takes advantage of HTTP and permits the dynamic
generation of iPXE scripts -- referred to in RackHD as *profiles* -- based on what the server
should do when it is PXE booting.

.. _Preboot Execution Environment (PXE): https://en.m.wikipedia.org/wiki/Preboot_Execution_Environment
.. _iPXE: http://en.wikipedia.org/wiki/IPXE

Data center automation is enabled through each server's `Baseboard Motherboard Controller (BMC)`_ embedded on the
server motherboard. Using `Intelligent Platform Management Interface (IPMI)`_
to communicate with the BMC, RackHD can remotely power on, power off, reboot, request a PXE boot,
and perform other operations.

.. _Baseboard Motherboard Controller (BMC): https://en.m.wikipedia.org/wiki/Baseboard_management_controller
.. _Intelligent Platform Management Interface (IPMI): https://en.m.wikipedia.org/wiki/Intelligent_Platform_Management_Interface


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

3. The task "generate-enclosure" interrogates catalog data for the system serial number and/or IPMI fru devices
   to determine whether the node is part of an enclosure (for example, a chassis that aggregates power for
   multiple nodes), and updates the relations in the node document if matches are found.

4. The task "create-default-pollers" creates a set of default pollers that periodically monitor the
   device for system hardware alerts, built in sensor data, power status, and similar information.

5. The last task ("run-sku-graph") checks if there are additional workflow hooks defined on the SKU definition
   associated with the node, and creates a new workflow dynamically if defined.

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

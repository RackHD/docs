How It Works
============


Discovery
---------

RackHD supports two modes of learning about machines that it manages. We loosely group
these as *active* and *passive* discovery.

* Passive discovery is where a user or outside system actively tells RackHD that the system exists. This is enabled through making a post to the REST interface that RackHD can then add to its data model.

* Active discovery is invoked when a machine attempts to PXE boot on the network that RackHD is
  "listening" on. As a new machine PXE boots, RackHD compares the MAC address of the hardware booting to its internal information. If the MAC address has not been previously recorded,
  it creates a new record in the data model and then invokes a default discovery workflow.

The discovery workflow is pre-configured to download a pre-built Linux kernel, initrd and ubuntu filesystem to run in memory and coordinate
with the monorail engine to run commands on the remote machine to interrogate the device's motherboard.

Discovery Workflow
---------------------

Discovery tasks are performed sequentially:

#. Discovery is initiated by sending down the iPXE boot loader with a pre-built script to run within
   iPXE. This script then chainloads into a new, dynamically rendered iPXE script that interrogates the 
   enabled network interfaces on the remote machine and reports them back to RackHD, which adds this 
   information to the machine and lookup records.

#. RackHD then renders an additional iPXE script to be chainloaded that downloads
   and runs the microkernel. The microkernel boots up and requests a Node.js "bootstrap" script
   from RackHD. RackHD runs the bootstrap program which uses a simple REST API to "ask" what it should do on the remote host from RackHD. The workflow engine,
   running the discovery workflow, provides a set of tasks to run. These tasks are matched with parsers in RackHD to understand and store the output. They work
   together to run Linux commands that interrogate the hardware from the microkernel running in memory. These commands include interrogating the machines BMC
   settings through IPMI, the PCI cards installed, the DMI information embedded in the BIOS, and others. The resulting information is then stored in JSON format
   as "catalogs" in RackHD.

#. The discovery workflow then performs a workflow task process called "SKU
   analysis" that compares the catalog data for the node against SKU definitions
   loaded into the system through the REST interface. If the definitions match,
   RackHD updates its data model indicating that the node belongs
   to a SKU.

#. The workflow uses the IPMI channels to set up a connection to the BMC so that it can control that node in the future (power on, off and reboot).

#. The workflow creates pollers that periodically monitor the device for system hardware alerts, built in sensor data, power status, and similar information.

#. The workflow reboots the machine.

**Notes:**

* No workflow is assigned to a PXE-booting system that is already known to RackHD. Instead, the RackHD system ignores proxy DHCP requests from booting
  nodes with no active workflow, letting the system continue to boot as specified by its BIOS or UEFI boot order.

* The discovery workflow can be updated to do additional work or steps for the installation of RackHD - to run other workflows based on the SKU analysis, or
  different actions based on the logic embedded into the workflow itself.

* Additional pollers exist and can be configured to capture data through SNMP, and the RackHD project is set up to support additional pollers as plugins that can be
  configured and run as desired.


Telemetry and Alerting
----------------------

Poller information is converted into a "live data feed" and published through
AMQP, providing a "live telemetry feed" for the raw data collected on the
remote systems (recent data is also cached for manual access). In addition to 
this live feed, RackHD includes some rudimentary
alerting mechanisms that compare the data collected by the pollers to regular
expressions, and if they match create an additional event that is published on
an "alert" exchange in AMQP.

Other Workflows
---------------

Other workflows can be configured and assigned to run on remote systems. For example, **OS install** can be set to explicitly power cycle (reboot) a remote node. As the system PXE boots, an installation kernel is sent down and run instead of the discovery microkernel.

The remote network-based OS installation process that runs from Linux OS
distributions typically runs with a configuration file - **debseed** or **kickstart**.
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
functionality for running :doc:`tasks` via
graph-based control flow mechanisms. A typical graph consists of a list of
tasks which themselves are essentially decorated functions.

For more detailed information on the workflow graphs, see :doc:`monorail/graphs`.

Workflow Tasks
^^^^^^^^^^^^^^^^^
A workflow task is a unit of work decorated with data and logic that allows it to
be included and run within a workflow. Tasks can be
defined to do wide-ranging operations, such as bootstrap a server node into a
Linux microkernel, parse data for matches against a rule, and others. The tasks in a workflow are run in a specific order.

For more detailed information on tasks, see :doc:`monorail/tasks`.

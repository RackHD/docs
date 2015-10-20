How It Works
============


Discovery
---------

RackHD supports two modes of learning about machines that it manages. We loosely group
these as *active* and *passive* discovery. Active discovery is where a user or outside
system actively tells RackHD that the system exists. This is enabled through making
a post to the REST interface, that RackHD can then add to it's data model.

Passive discovery can be either enabled or disabled through configuration in RackHD,
and is invoked when a machine attempts to PXE boot on the network that RackHD is
"listening" on. As a new machine PXE boots, RackHD compares the mac-address of the
hardware booting to it's internal information, and if the mac-address isn't one it
already has recorded, it creates a new record in the data model and then invokes a
"default workflow". The default workflow is configurable, and by default we set it
to what we call a discovery workflow - which is set to download a pre-made linux
kernel and initrd (what we call a microkernel) to run in memory only and coordinate
with the monorail engine to run commands on the remote machine to interrogate it
from it's motherboard.

Discovery Workflow
------------------

During the process of initiating a discovery workflow for the first time, we
initially send down the iPXE boot loader with a pre-built script to run within
iPXE. This script interrogates the network interfaces that it finds enabled on
the remote machine and reports those back to RackHD, which updates the record
about the machine being discovered to "know" about these additional interfaces.
Once complete, RackHD responds with an additional iPXE script that downloads
and runs the microkernel.

These iPXE scripts are built on simple templates, and accessible through the
Monorail engine's REST APIs as "profiles".

The microkernel is crafted to boot up and request a NodeJS "bootstrap" program
from RackHD, which it in turn runs. This bootstrap program uses a simple REST
API to "ask" what it should do on the remote host from RackHD. The workflow engine,
running the discovery workflow, provides a set of tasks to run. These tasks
are matched with parsers in RackHD to understand and store the output, and
these work together to run linux commands to interrogate the hardware from the
microkernel running in memory and store the resulting information in JSON format
as "catalogs" in RackHD. These commands include interrogating the machines BMC
settings through IPMI, the PCI cards installed, the DMI information embedded in
the BIOS, and so forth.

The default workflow then does a specific workflow task process called "SKU
analysis" that compares the catalogs it's received against SKU definitions
loaded into the system through the REST interface. If the definitions match,
RackHD updates it data model indicating that the noe just discovered belongs
to one or more SKUs (depending on how the definitions have been crafted). The
workflow also uses the interrogated IPMI channels to attempt to set up a
connection to the BMC for the node that's just been discovered so that it can
control that node in the future (power on, off and reboot). If the out of band
management settings were set correctly, the Workflow then also automatically
creates monitoring pollers that periodically inquire about the state of the
remote system using the out of band management protocol (typically IPMI) to
begin monitoring for system hardware alerts, built in sensor data, power
status, etc.

The default workflow then reboots the machine.

If RackHD already "knows about" a machine when it's booting, and no workflow is
assigned to run when it sees that PXE boot process begin, the RackHD system
responds with a default "pass through" to iPXE, letting the remote system
continue to boot to whatever is next in it's BIOS or UEFI boot order - often
configured in hardware systems to result in booting from local disks.

The default workflow can be updated to do additional work or steps for the
installation of RackHD - to run other workflows based on the SKU analysis, or
different actions based on the logic embedded into the workflow itself.

Pollers
-------

The workflow engine also hosts "pollers" - active monitoring tests that can be
configured either through workflows or directly through the REST API on the
monorail engine. The default discovery workflow attempts to create IPMI pollers
to interrogate BMCs during the discovery workflow process. Additional pollers
exist and can be configured to capture data through SNMP, and the RackHD
project is set up to support additional pollers as plugins that can be
configured and run as desired.

Telemetry and Alerting
----------------------

Poller information is converted into a "live data feed" and published through
AMQP, providing a "live telemetry feed" for all the raw data collected on the
remote systems. In addition to this live feed, RackHD includes some rudimentary
alerting mechanisms that compare the data collected by the pollers to regular
expressions, and if they match create an additional event that's published on
an "alert" exchange in AMQP.

Other workflows
---------------

Other workflows are defined that can provide different responses than the
discovery workflow, and those workflows can be assigned to "start" the next
time it sees a node PXE boot. The "OS install" workflows, for example, start
with assignment to a node and then explicitly power cycling (or rebooting)
the remote node. When the PXE boot process begins, instead of sending down the
discovery microkernel it will send down an OS installation kernel, easily
available and provided by OS vendors.

The remote network-based OS installation process that runs from Linux OS
distributions typically runs with a configuration file - debseed or kickstart.
The monorail engine provides a means to render these configuration files
through templates, with the values derived from the workflow itself - either as
defaults built into the workflow, discovered data in the system (such as data
within the catalogs found during machine interrogation), or even passed in as
variables when the workflow was invoked by an end-user or external automation
system. These "templates" can be accessed through the Monorail's engine REST
API - created, updated, or removed - to support a wide variety of responses and
capabilities.

Workflows can additionally be chained together and the workflow engine includes
simple logic (as demonstrated in the discovery workflow) to perform arbitrarily
complex tasks based on the workflow definition. The workflow definitions
themselves are accessible through the Monorail engine's REST API as a "graph"
of "tasks". Both graphs and tasks are fully declarative with a JSON format.
Tasks are also mapped up "Jobs", which is the NodeJS code that's included
within RackHD. For more detailed information on the workflow engine, graphs,
and tasks, see :doc:`monorail/graphs` and :doc:`monorail/tasks`.

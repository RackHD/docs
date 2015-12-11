RackHD Overview
===================

RackHD serves as an abstraction layer between other M&O layers and the underlying physical
hardware. Developers can use the RackHD API to create a user interface that serves as single point of access
for managing hardware services regardless of the specific hardware in place.

Once RackHD has the ability to discover the existing hardware
resources, catalog each component, and retrieve detailed telemetry information from each resource. The retrieved
information can then be used to perform low-level hardware management tasks, such as BIOS
configuration, OS installation, and firmware management.

RackHD sits between the other M&O layers and the underlying physical hardware devices. User interfaces
at the higher M&O layers can request hardware services from RackHD. RackHD handles the details of connecting
to and managing the hardware devices.

The RackHD API allows you to automate a great range of management tasks, including:

* Install, configure, and monitor bare metal hardware (compute servers, PDUs, DAEs, network switches).
* Provision and erase server OSes.
* Install and upgrade firmware.
* Monitor bare metal hardware through out-of-band management interfaces.
* Provide data feeds for alerts and raw telemetry from hardware.



The RackHD Project
-----------------------------------------

The original motive centered on maximizing the automation of firmware and BIOS updates
in the data center, thereby reducing the extensive manual processes that are still required
for these operations.

Existing open source solutions do an admirable job of inventory and bare OS
provisioning, but the ability to upgrade firmware is beyond the technology
stacks currently available (i.e. `xCat`_, `Cobbler`_, `Razor`_ or `Hanlon`_).
By adding an event-based workflow engine that works in conjunction with classical PXE
booting, RackHD makes it possible to architect different deployment configurations
as described in :doc:`how_it_works` and :doc:`packaging_and_deployment`.

RackHD extends automation beyond simple PXE booting. It can perform highly
customizable tasks on machines, as is illustrated by the following sequence:

* PXE boot the server
* Interrogate the hardware to determine if it has the correct firmware version
* If needed, flash the firmware to the correct version
* Reboot (mandated by things like BIOS and BMC flashing)
* PXE boot again
* Interrogate the hardware to ensure it has the correct firmware version.
* SCORE!

In effect, RackHD combines open source tools with a declarative, event-based workflow engine.
It is similar to Razor and Hanlon in that it sets up and boots a microkernel that can perform predefined tasks. However, it
extends this model by adding a remote agent that communicates with the workflow engine to
*dynamically* determine the tasks to perform on the target machine, such as zero out
disks, interrogate the PCI bus, or reset the IPMI settings through the
hosts internal KCS channel.

Along with this agent-to-workflow integration, RackHD optimizes the path
for interrogating and gathering data. It leverages existing Linux tools and parses
outputs that are sent back and stored as free-form JSON data structures.

The workflow engine was extended to support polling via out-of-band interfaces in order to
capture sensor information and other data that can be retrieved using IPMI.
In RackHD these become *pollers* that periodically capture telemetry data from
the hardware interfaces.

What RackHD Does Well
-----------------------------------------

RackHD is focused on being the lowest level of automation that interrogates agnostic hardware and
provisions machines with operating systems. The API can be used to pass in data through variables
in the workflow configuration, so you can parameterize workflows. Since workflows also have
access to all of the SKU information and other catalogs, they can be authored to
react to that information.

The real power of RackHD, therefore, is that you can develop your own workflows and
use the REST API to pass in dynamic configuration details. This allows you to execute
a specific sequence of arbitrary tasks that satisfy your requirements.

When creating your initial workflows, it is recommended that you use the existing workflows
in our code repository to see how different actions can be performed.



What RackHD Doesn’t Do
-----------------------------------------

RackHD is a comparatively passive system. Workflows do not contain the complex logic for
functionality that is implemented in the layers above hardware management and orchestration.
For example, workflows do not provide scheduling functionality or choose which
machines to allocate to particular services.

We document and expose the events around the workflow
engine to be utilized, extended, and incorporated into an infrastructure
management system, but we did not take RacKHD itself directly into the infrastructure layer.

Comparison with Other Projects
-----------------------------------------

Comparison to other open source technologies:

**Cobbler comparison**

* Grand-daddy of open source tools to enable PXE imaging
* Original workhorse of datacenter PXE automation
* XML-RPC interface for automation, no REST interface
* No dynamic events or control for TFTP, DHCP
* Extensive manual and OS level configuration needed to utilize
* One-shot operations - not structured to change personalities (OS installed) on
  a target machine, or multiple reboots to support some firmware update needs
* No workflow engine or concept of orchestration with multiple reboots

**Razor/Hanlon comparison**

* HTTP wrapper around stock open source tools to enable PXE booting (DHCP,
  TFTP, HTTP)
* Razor and Hanlon extended beyond Cobbler's concepts to include microkernel
  to interrogate remote host and use that information with policies to choose
  what to PXE boot
* Razor isn't set to make dynamic responses through TFTP or DHCP where RackHD
  uses dynamic responses based on current state for PXE to enable workflows
* Catalog and policy are roughly equivalent to RackHD default/discovery workflow
  and SKU mechanism, but oriented on single OS deployment for a piece or type
  of hardware
* Razor and Hanlon are often focused on hardware inventory to choose and
  enable OS installation through Razor's policy mechanisms.
* No workflow engine or concept of orchestration with multiple reboots
* Tightly bound to and maintained by Puppet
* Forked variant `Hanlon`_ used for Chef Metal driver

**xCat comparison**

* HPC Cluster Centric tool focused on IBM supported hardware
* Firmware update features restricted to IBM/Lenovo proprietary hardware where
  firmware was made to "one-shot-update", not explicitly requiring a reboot
* Has no concept of workflow or sequencing
* Has no obvious mechanism for failure recovery
* Competing with Puppet/Chef/Ansible/cfEngine to own config management story
* Extensibility model tied exclusively to Perl code
* REST API is extremely light with focus on CLI management
* Built as a master controller of infrastructure vs an element in the process

.. _Cobbler: http://cobbler.github.io
.. _Razor: https://github.com/puppetlabs/razor-server
.. _Hanlon: https://github.com/csc/Hanlon
.. _xCat: http://xcat.org

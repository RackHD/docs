Software Architecture
=====================================

RackHD enables much of its functionality by providing PXE boot services
to machines that will be managed, and integrating the services providing
the protocols used into a workflow engine. RackHD is built to download a
microkernel (a small OS) crafted to run tasks in coordination with the workflow
engine. The default and most commonly used microkernel is based on Linux, although
WinPE and DOS netbooting is also possible.

Theory of Operations
-----------------------------------------

RackHD was born from the realization that our effective automation
in computing and improving efficiencies has come from multiple layers of orchestration,
each building on a lower layer. A full-featured API-driven environment that is effective
spawns additional wrappers to combined the lower level pieces into patterns that are
at first experimental and over time become either de facto or concrete standards.

|

.. image:: _static/automation_layers.png
 :height: 600
 :align: center

|

Application automation services such Heroku or CloudFoundry are service API layers
(AWS, Google Cloud Engine, SoftLayer, OpenStack, and others) that are built overlying
infrastructure. Those services, in turn, are often installed, configured, and managed by automation in
the form of software configuration management: Puppet, Chef, Ansible, etc. To automate
data center rollouts, managing racks of machines, etc - these are built on automation
to help roll out software onto servers - Cobbler, Razor, etc.

The closer you get to hardware, the less automated systems tend to become. Cobbler
and SystemImager were mainstays of early data center management tooling. Razor (or Hanlon, depending on where you're looking) expanded
on that base system , supported mainly by people working to implement further automation solutions.

RackHD expands the capabilities of hardware management and operations beyond the mainstay features, such as PXE booting
and automated installation of OS and software. It includes active metrics and telemetry, integration and annotated monitoring of
underlying hardware, and firmware updating.


RackHD enables deeper and fuller automation by "playing nicely" with
both existing and future potential systems. It adds to existing open source efforts by providing a significant step the enablement of
converged infrastructure automation.

Major Components
----------------

The software is roughly divided into a simplified REST API that is oriented towards a common
data model and easy integration, and an underlying workflow engine (named the
"monorail engine" after a popular Seattle coffee shop: http://www.yelp.com/biz/monorail-espresso-seattle).


|

.. image:: _static/high_level_architecture.png

|

The upper layer of the architecture called the "onserve executive" communicates using
both an internal REST API and AMQP as a message bus between the various processes. This layer is still under development.

The lower layer of the architecture called the "monorail engine" provides the workflow
engine and coordinated agents for interacting through multiple protocols with remote
systems. The monorail engine is broken into independent processes with a mind to support
scaling or distributing them independently by protocol. It communicates
using message passing over AMQP and stores data in MongoDB as needed for persistence.


|

.. image:: _static/process_level_architecture.png
 :align: center

|

Onserve Executive
---------------------

Onserve executive provides the simplified API and is built to encapsulate the
monorail engine, converting internal data models into a common data format based on the Redfish 1.0 specification (http://www.dmtf.org/standards/redfish).

Onserve executive conversions are still in development. For more information, contact us at: rackhd@googlegroups.com.


Monorail Engine
-------------------

|

.. image:: _static/monorail_engine_dataflow.png
 :height: 600
 :align: center

|

ISC DHCP
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This DHCP server provides IP addresses dynamically using the DHCP protocol. It is a critical component of a standard `Preboot Execution Environment (PXE)`_ process,
.

.. _Preboot Execution Environment (PXE): https://en.wikipedia.org/wiki/Preboot_Execution_Environment



on-dhcp-proxy
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The DHCP protocol supports getting additional data specifically for the PXE
process from a secondary service that also responds on the same network as
the DHCP server. The DHCP proxy service provides that information, generated
dynamically from the workflow engine.

on-tftp
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

TFTP is the common protocol used to initiate a PXE process. on-tftp is
tied into the workflow engine to be able to dynamically provide responses
based on the state of the workflow engine and to provide events to the workflow
engine when servers request files via TFTP.

on-http
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

on-http provides both the REST interface to the workflow engine and data model APIs
as well as a communication channel and potential proxy for hosting and serving files to support dynamic PXE responses.
RackHD commonly uses iPXE as its initial
bootloader, loading remaining files for PXE booting via HTTP and using that communications
path as a mechanism to control what a remote server will do when rebooting.

on-http also serves as the communication channel for the microkernel to support
deep hardware interrogation, firmware updates, and other actions that can only be
invoked directly on the hardware (not through an out of band management channel).

on-syslog
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

on-syslog is a syslog receiver endpoint provideing annotated and structured logging
from the hosts under management. It channels all syslog data sent to the
host into the workflow engine.

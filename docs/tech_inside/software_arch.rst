Software Architecture
=============================

.. contents:: Table of Contents

RackHD provides a REST API for the automation using an underlying workflow
engine (named the "monorail engine" after a popular Seattle coffee shop:
http://www.yelp.com/biz/monorail-espresso-seattle).

RackHD is also providing an implementation of the `Redfish specification`_ as an
additional REST API to provide a common data model for representing bare metal
hardware, and provides this as an aggregate for multiple back-end servers and systems.

.. _Redfish specification: http://redfish.dmtf.org


|

.. image:: ../_static/high_level_architecture.png

|

The workflow engine operates with and coordinates with services to respond to protocols
commonly used in hardware management. RackHD is structured with several independent processes, typically
focused on specific function or protocol so that we can scaling or distribute them independently, using
a pattern of `Microservices`_.

.. _Microservices: https://en.wikipedia.org/wiki/Microservices

RackHD communicates between these
using message passing over AMQP and stores data in an included persistence store. MongoDB is
the default, and configurable communications layers and persistence layers are in progress.

|

.. image:: ../_static/process_level_architecture.png
 :align: center

|


|

.. image:: ../_static/monorail_engine_dataflow.png
 :height: 600
 :align: center

Major Components
-----------------------------

ISC DHCP
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This DHCP server provides IP addresses dynamically using the DHCP protocol. It is a critical component of a standard `Preboot Execution Environment (PXE)`_ process.

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
as well as a communication channel and potential proxy for hosting and serving files
to support dynamic PXE responses. RackHD commonly uses iPXE as its initial
bootloader, loading remaining files for PXE booting via HTTP and using that communications
path as a mechanism to control what a remote server will do when rebooting.


on-syslog
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

on-syslog is a syslog receiver endpoint provideing annotated and structured logging
from the hosts under management. It channels all syslog data sent to the
host into the workflow engine.

on-taskgraph
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

on-taskgraph is the workflow engine, driving actions on remote systems and processing
workflows for machines being managed. Additionally, the workflow engine provides the
engine for polling and monitoring.

on-taskgraph also serves as the communication channel for the microkernel to support
deep hardware interrogation, firmware updates, and other actions that can only be
invoked directly on the hardware (not through an out of band management channel).
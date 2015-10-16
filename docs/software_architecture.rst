RackHD software architecture
=====================================

RackHD enables much of it's functionality by providing PXE boot services
to machines that will be managed, and intergrating the services providing
the protocols used into a workflow engine. RackHD is built to download a
microkernel (a small OS) crafted to run tasks in coordination with the workflow
engine. The default and most commonly used microkernel is based on Linux, although
we have also built and run WinPE and even DOS microkernels.

Theory of operation and integration with other automation systems
-----------------------------------------

RackHD is meant to enable deeper and fuller automation and "play nicely" with
both existing and future potential systems.

... insert details from theory of operations here ...

Major Components
----------------

The software is roughly divided into a simplified REST API oriented towards a common
data model and easy integration, and an underlying workflow engine (code named the
"monorail engine", after a popular Seattle coffee shop: http://www.yelp.com/biz/monorail-espresso-seattle).

The upper layer of the architecture, called the "onserve executive" communicate using
both an internal REST API and AMQP as a message bus between the various processes.

The lower layer of the architecture - the "monorail engine" provides the workflow
engine and coordinated agents for interacting through multiple protocols with remote
systems. The monorail engine is broken into indepdent processes with a mind to support
scaling or distributing them indepdendently by protocol, and communicates together
using message passing over AMQP, and stores data as needed for persistence in MongoDB.

.. image:: _static/rackhd_processes.png

The Onserve Executive
---------------------

* onserve

    Provides the simplified API and is built to encapsulate the monorail engine, converting
    internal data models into a common data format based on the Redfish 1.0 specification (http://www.dmtf.org/standards/redfish).
    Onserve also hosts all additional API endpoints and provides access controls to enable (or hide)
    access to the underlying monorail engine APIs

* LTAE

    Coordinates logging, tracing, and alert configuration and notifications. Provides internal
    logging and supports alerting based on events gathered and processed through the monorail engine.
    LTAE includes plugin mechanisms for additional plugins to support alternative alerting mechanisms
    such as phone-home support systems.

* conductor

    Acts as an additional layer of orchestration to support coordinated efforts involving managing
    multiple compute nodes, including a general state machine for nodes and will be expanded to
    include a policy driven mechanism for multi-node coordination.

The monorail engine
-------------------

.. image:: _static/monorail_engine_dataflow.png

* ISC DHCP

    A DHCP server is a critical component of a standard PXE (https://en.wikipedia.org/wiki/Preboot_Execution_Environment) process,
    providing IP addresses dynamically using the DHCP protocol

* on-dhcp-proxy

    The DHCP protocol supports getting additional data specifically for the PXE
    process from a secondary service that also responds on the same network as
    the DHCP server. The DHCP proxy service provides that information, generated
    dynamically from the workflow engine.

* on-tftp

    TFTP is the common protocol used to initiate a PXE process, and on-tftp is
    tied into the workflow engine to be able to dynamically provide responses
    based on the state of the workflow engine, and to provide events to the workflow
    engine when servers request files via TFTP

* on-http

    on-http provides both the REST interface to the workflow engine and data model APIs
    as well as a communication channel and potential proxy for hosting files and serving
    them to support dynamic PXE responses. RackHD commonly uses iPXE as it's initial
    bootloader, loading remaining files for PXE booting via HTTP and using that communications
    path as a mechanism to control what a remote server will do when rebooting.

    on-http also serves as the communication channel for the microkernel to support
    deep hardware interrogation, firmware updates, and other actions that can only be
    invoked directly on the hardware and not through an out of band management channel.

* on-syslog

    on-syslog is a syslog receiver endpoint that channels all syslog data sent to the
    host into the workflow engine to provide annotated and structured logging
    from the hosts under management.

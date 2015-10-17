.. RackHD documentation master file, created by
   sphinx-quickstart on Sat Aug 29 13:07:25 2015.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

RackHD
======

RackHD is a technology stack up created for enabling hardware management and operations,
to provide cohesive APIs to enabled automated infrastructure.


RackHD Key Benefits
==================================
Add intro statement...high level sections:
Manage - discovery, genealogy, power
Monitor - telemetry, health, performance
Maintain - firmware updates

Getting Started
==================================
(TEST) In a Converged Infrastructure Platform (CIP) architecture, RackHD software provides hardware management and orchestration (M&O). It serves as an abstraction layer between other M&O layers and the underlying physical hardware. Developers can use the RackHD API to create a user interface that serves as single point of access for managing hardware services regardless of the specific hardware in place.

The project is a collection of libraries and applications housed at https://github.com/RackHD/
and documentation hosted at http://rackhd.readthedocs.org. The code for RackHD is a combination
of Python, Javascript (NodeJS), and C, available under the Apache 2.0 license (or
compatible sublicences for library dependencies).

Contents
---------

.. toctree::
   :maxdepth: 1

   introduction
   software_architecture
   how_it_works
   contributing
   development_guide
   monorail/graphs
   monorail/tasks
   monorail/skus
   monorail/api_versioning
   monorail/amqp_conventions
   monorail/configuration
   monorail/microkernel
   monorail/creating_overlays
   monorail/https
   monorail/naming_conventions
   monorail/passive_discovery
   monorail/pollers


Indices and tables
------------------

* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`

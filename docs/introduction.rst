RackHD
======

RackHD is a technology stack up created for enabling hardware management and operations, to provide
cohesive APIs to enabled automated infrastructure.

The project is a collection of libraries and applications housed at `RackHD GitHub`_ and
intended to be deployed together to provide a solution that can be used either standalone, or as a
technology to be included and embedded in larger applications. This documentation is also housed on GitHub
and hosted at `RackHD Documentation`_.

The primary goals of RackHD are to provide REST APIs and live data feeds to enable automated solutions
for managing hardware resources. The technology and architecture are built to provide a platform
agnostic solution for supporting automation needs for the hardware that makes up physical infrastructure
our services all rely on.

The combination of services is intended to provide a REST API based service to:

* Install, configure, and monitor bare metal hardware such as compute servers, power distribution
  units (PDUs), direct attached extenders (DAE) for storage, network switches.
* Provision, Erase, and Reprovision a compute server's OS
* Install/Upgrade firmware for qualified hardware
* Monitor and alert bare metal hardware through out of band management interfaces
* Provide REST APIs for convenient access to hardware knowledge, both common and vendor specific
* Provide pub/sub data feeds for alerts and raw telemetry from hardware

Why
---

The project started with the initial goal of providing a consistent and clear mechanism to both
inventory hardware and upgrade firmware for commodity white-box servers that exist today (2014/2015).
Existing open source solutions do an admirable job of inventory and bare OS provisioning, but the
extension to upgrading firmware and validating the upgrade was beyond the existing technology stacks
readily available (xCat, Cobber, Razor for example). For a more detailed comparison of the open source
technologies, where they're similiar and different, see :doc:`project_comparson`

What
----

To achieve this goal, we used a process to provision a simple tool chain, do the relevant upgrade,
reset the hardware (which often meant just rebooting the machine, but in some cases meant explicitly
power cycling all of it), and the reload the tool chain to complete the process - reloading configuration
settings and/or validating the resulting hardware was operating correctly. The other functionality we
were focused on was gathering the data from the hardware under management and exposing it as a live data
feed through a pub/sub like mechanism. In creating the initial software to achieve these goals and expose
all of that functionality in a REST API, we found it expedient to implement a more generalized workflow
engine that was provided with detailed information from the services and protocols normally used in
combination to enable these kinds of actions through industry standard protocols and mechanisms. This
declarative workflow engine, as well as workflows, files, and the data models to support these
functions make up the RackHD project and related repositories.

To keep possible integrations simple, we implemented an additional API layer intended to simplify
the interactions so that consuming applications could ask for well known, preset functions and retrieve
data and information using a consistent data model, as well provide a security boundary for these
kinds of functions in a datacenter environment.

Where
-----

Our initial deployments were intended to go onto physical hardware directly to manage a rack (or racks)
of systems, and quickly expanded to be hosted on virtual machines as well. With the nature of the
protocols needed to be coordinated to achieve these functions, there are a number of possible
deployment configurations that can be architected. For more details of those configurations, both
what's needed, why, and how we've created some reference examples of deployments see :doc:`how_it_works`
and :doc:`packaging_and_deployment`.

How
---

For an overview of our code and architecture see :doc:`software_architecture`. For an
introduction into the common data model, highest level API, and how to use it, see
:doc:`getting_started_onserve`. For a more detailed view in the concepts and software
clockwork that make up the workfow engine and low level interactions, see :doc:`getting_started_monorail`.

.. _RackHD GitHub: https://github.com/RackHD/
.. _RackHD Documentation: http://rackhd.readthedocs.org/


Repositories
------------------------------------



Applications
^^^^^^^^^^^^^^^^^^^^^^^^

.. list-table::
   :widths: 20 20 100
   :header-rows: 1

   * - Application
     - Repository
     - Description
   * - on-tftp
     - https://github.com/RackHD/on-tftp
     - on-tftp provides a TFTP service integrated into the workflow engine for RackHD. TFTP is the common protocol used to initiate a PXE process, and on-tftp is tied into the workflow engine to be able to dynamically provide responses based on the state of the workflow engine, and to provide events to the workflow engine when servers request files via TFTP
   * - on-http
     - https://github.com/RackHD/on-http
     - on-http is the HTTP server for RackHD. RackHD commonly uses iPXE as its initial bootloader, loading remaining files for PXE booting via HTTP and using that communications path as a mechanism to control what a remote server will do when rebooting. on-http also serves as the communication channel for the microkernel to support deep hardware interrogation, firmware updates, and other actions that can only be invoked directly on the hardware and not through an out of band management channel.
   * - on-syslog
     - https://github.com/RackHD/on-syslog
     - on-syslog provides a Syslog service that routes Syslog messages into the RackHD workflow engine.
   * - on-taskgraph
     - https://github.com/RackHD/on-taskgraph
     - on-taskgraph is the core workflow engine for RackHD, initiating workflows, performing tasks, and responding to ancillary services to enable the RackHD service. It provides functionality for running encapsulated jobs/units of work via graph-based control flow mechanisms.
   * - on-dhcp-proxy
     - https://github.com/RackHD/on-dhcp-proxy
     - on-dhcp-proxy provides a DHCP proxy service for enabling the RackHD PXE workflow engine to operate with an existing DHCP server. The DHCP protocol supports getting additional data for the PXE process from a secondary service that also responds on the same network as the DHCP server. The DHCP proxy service provides that information, generated dynamically from the workflow engine.



Libraries
^^^^^^^^^
.. list-table::
   :widths: 20 20 100
   :header-rows: 1

   * - Library
     - Repository
     - Description
   * - core
     - https://github.com/RackHD/on-core
     - Core libraries in use across RackHD applications.
   * - tasks
     - https://github.com/RackHD/on-tasks
     - 'on-tasks' are the job code, libraries, and definitions (meant as a library of tasks). Tasks are loaded and run by taskgraphs as needed.


Supplemental Code
^^^^^^^^^^^^^^^^^

.. list-table::
   :widths: 20 20 100
   :header-rows: 1

   * - Library
     - Repository
     - Description

   * - Web user interface
     - https://github.com/RackHD/on-web-ui
     - Initial web interfaces to some of the APIs - multiple interfaces embedded into a single project.
   * - statsd
     - https://github.com/RackHD/on-statsd
     - A local statsD implementation that makes it easy to deploy on a local machine for aggregating and summarizing application metrics.

Documentation
^^^^^^^^^^^^^^^^^^^^^^

.. list-table::
   :widths: 20 80
   :header-rows: 1

   * - Repository
     - Description
   * - https://github.com/RackHD/docs
     - The RackHD documentation as published to http://rackhd.readthedocs.org/en/latest/.

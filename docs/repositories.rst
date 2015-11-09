
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
     - Node.js application provided TFTP service integrated with the workflow engine. TFTP is the common protocol used to initiate a PXE process, and on-tftp is tied into the workflow engine to be able to dynamically provide responses based on the state of the workflow engine, and to provide events to the workflow engine when servers request files via TFTP
   * - on-http
     - https://github.com/RackHD/on-http
     - Node.js application provided HTTP service integrated with the workflow engine. RackHD commonly uses iPXE as its initial bootloader, loading remaining files for PXE booting via HTTP and using that communications path as a mechanism to control what a remote server will do when rebooting. on-http also serves as the communication channel for the microkernel to support deep hardware interrogation, firmware updates, and other actions that can only be invoked directly on the hardware and not through an out of band management channel.
   * - on-syslog
     - https://github.com/RackHD/on-syslog
     - Syslog endpoint integrated to feed data to the workflow engine.
   * - on-taskgraph
     - https://github.com/RackHD/on-taskgraph
     - Node.js application providing the workflow engine. It provides functionality for running encapsulated jobs/units of work via graph-based control flow mechanisms.
   * - on-dhcp-proxy
     - https://github.com/RackHD/on-dhcp-proxy
     - Node.js application providing DHCP proxy support in the workflow engine. The DHCP protocol supports getting additional data specifically for the PXE process from a secondary service that also responds on the same network as the DHCP server. The DHCP proxy service provides that information, generated dynamically from the workflow engine.



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
     - Core libraries in use across Node.js applications.
   * - tasks
     - https://github.com/RackHD/on-tasks
     - Node.js task library for the workflow engine. Tasks are loaded and run by taskgraphs as needed.


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

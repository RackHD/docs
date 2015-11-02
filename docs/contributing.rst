Contributing to RackHD
======================

The https://github/emccode/RackHD repository acts as a single source location to help you get or build all the pieces to learn about,
take advantage of, and contribute to RackHD.


Project and code repository overview
------------------------------------
The RackHD project is a collection of libraries and applications housed at https://github.com/RackHD/.

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
     - | Node.js application provided TFTP service integrated
       | with the workflow engine.
   * - on-http
     - https://github.com/RackHD/on-http
     - | Node.js application provided TFTP service integrated
       | with the workflow engine.
   * - on-syslog
     - https://github.com/RackHD/on-syslog
     - | Syslog endpoint integrated to feed data into
       | the workflow engine.
   * - on-taskgraph
     - https://github.com/RackHD/on-taskgraph
     - | Node.js application providing the workflow engine.
   * - on-dhcp-proxy
     - https://github.com/RackHD/on-dhcp-proxy
     - | Node.js application providing DHCP proxy support
       | integrated into the workflow engine.

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
     - Node.js task library for the workflow engine.


Supplemental code
^^^^^^^^^^^^^^^^^
.. list-table::
   :widths: 20 20 100
   :header-rows: 1

   * - Library
     - Repository
     - Description

   * - Web user interface
     - https://github.com/RackHD/on-web-ui
     - | Initial web interfaces to some of the APIs - multiple
       | interfaces embedded into a single project.
   * - statsd
     - https://github.com/RackHD/on-statsd
     - | A local statsD implementation that makes it easy to deploy
       | on a local machine for capturing application metrics.

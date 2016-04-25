
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


Repositories Status
^^^^^^^^^^^^^^^^^

The following badges in the tables may take a while to load.

.. list-table::
   :widths: 20 20 20 40
   :header-rows: 1

   * - Repository
     - Travis-Ci Build
     - Code Climate
     - Code Coverage
   * - on-core
     - .. image:: https://travis-ci.org/RackHD/on-core.svg?branch=master
          :target: https://travis-ci.org/RackHD/on-core
          :alt: Travis Build
     - .. image:: https://codeclimate.com/github/RackHD/on-core/badges/gpa.svg?style=flat-square
          :target: https://codeclimate.com/github/RackHD/on-core
          :alt: Code Climate GPA
     - .. image:: https://coveralls.io/repos/github/RackHD/on-core/badge.svg?branch=master
          :target: https://coveralls.io/r/RackHD/on-core?branch=master
          :alt: Test Coverage
   * - on-dhcp-proxy
     - .. image:: https://travis-ci.org/RackHD/on-dhcp-proxy.svg?branch=master
          :target: https://travis-ci.org/RackHD/on-dhcp-proxy
          :alt: Travis Build
     - .. image:: https://codeclimate.com/github/RackHD/on-dhcp-proxy/badges/gpa.svg?style=flat-square
          :target: https://codeclimate.com/github/RackHD/on-dhcp-proxy
          :alt: Code Climate GPA
     - .. image:: https://coveralls.io/repos/github/RackHD/on-dhcp-proxy/badge.svg?branch=master
          :target: https://coveralls.io/r/RackHD/on-dhcp-proxy?branch=master
          :alt: Test Coverage
   * - on-http
     - .. image:: https://travis-ci.org/RackHD/on-http.svg?branch=master
          :target: https://travis-ci.org/RackHD/on-http
          :alt: Travis Build
     - .. image:: https://codeclimate.com/github/RackHD/on-http/badges/gpa.svg?style=flat-square
          :target: https://codeclimate.com/github/RackHD/on-http
          :alt: Code Climate GPA
     - .. image:: https://coveralls.io/repos/github/RackHD/on-http/badge.svg?branch=master
          :target: https://coveralls.io/r/RackHD/on-http?branch=master
          :alt: Test Coverage
   * - on-imagebuilder
     - .. image:: https://travis-ci.org/RackHD/on-imagebuilder.svg?branch=master
          :target: https://travis-ci.org/RackHD/on-imagebuilder
          :alt: Travis Build
     - N/A
     - N/A
   * - on-statsd
     - .. image:: https://travis-ci.org/RackHD/on-statsd.svg?branch=master
          :target: https://travis-ci.org/RackHD/on-statsd
          :alt: Travis Build
     - .. image:: https://codeclimate.com/github/RackHD/on-statsd/badges/gpa.svg?style=flat-square
          :target: https://codeclimate.com/github/RackHD/on-statsd
          :alt: Code Climate GPA
     - .. image:: https://coveralls.io/repos/github/RackHD/on-statsd/badge.svg?branch=master
          :target: https://coveralls.io/r/RackHD/on-statsd?branch=master
          :alt: Test Coverage
   * - on-syslog
     - .. image:: https://travis-ci.org/RackHD/on-syslog.svg?branch=master
          :target: https://travis-ci.org/RackHD/on-syslog
          :alt: Travis Build
     - .. image:: https://codeclimate.com/github/RackHD/on-syslog/badges/gpa.svg?style=flat-square
          :target: https://codeclimate.com/github/RackHD/on-syslog
          :alt: Code Climate GPA
     - .. image:: https://coveralls.io/repos/github/RackHD/on-syslog/badge.svg?branch=master
          :target: https://coveralls.io/r/RackHD/on-syslog?branch=master
          :alt: Test Coverage
   * - on-taskgraph
     - .. image:: https://travis-ci.org/RackHD/on-taskgraph.svg?branch=master
          :target: https://travis-ci.org/RackHD/on-taskgraph
          :alt: Travis Build
     - .. image:: https://codeclimate.com/github/RackHD/on-taskgraph/badges/gpa.svg?style=flat-square
          :target: https://codeclimate.com/github/RackHD/on-taskgraph
          :alt: Code Climate GPA
     - .. image:: https://coveralls.io/repos/github/RackHD/on-taskgraph/badge.svg?branch=master
          :target: https://coveralls.io/r/RackHD/on-taskgraph?branch=master
          :alt: Test Coverage
   * - on-tasks
     - .. image:: https://travis-ci.org/RackHD/on-tasks.svg?branch=master
          :target: https://travis-ci.org/RackHD/on-tasks
          :alt: Travis Build
     - .. image:: https://codeclimate.com/github/RackHD/on-tasks/badges/gpa.svg?style=flat-square
          :target: https://codeclimate.com/github/RackHD/on-tasks
          :alt: Code Climate GPA
     - .. image:: https://coveralls.io/repos/github/RackHD/on-tasks/badge.svg?branch=master
          :target: https://coveralls.io/r/RackHD/on-tasks?branch=master
          :alt: Test Coverage
   * - on-tftp
     - .. image:: https://travis-ci.org/RackHD/on-tftp.svg?branch=master
          :target: https://travis-ci.org/RackHD/on-tftp
          :alt: Travis Build
     - .. image:: https://codeclimate.com/github/RackHD/on-tftp/badges/gpa.svg?style=flat-square
          :target: https://codeclimate.com/github/RackHD/on-tftp
          :alt: Code Climate GPA
     - .. image:: https://coveralls.io/repos/github/RackHD/on-tftp/badge.svg?branch=master
          :target: https://coveralls.io/r/RackHD/on-tftp?branch=master
          :alt: Test Coverage
   * - on-web-ui
     - .. image:: https://travis-ci.org/RackHD/on-web-ui.svg?branch=master
          :target: https://travis-ci.org/RackHD/on-web-ui
          :alt: Travis Build
     - .. image:: https://codeclimate.com/github/RackHD/on-web-ui/badges/gpa.svg?style=flat-square
          :target: https://codeclimate.com/github/RackHD/on-web-ui
          :alt: Code Climate GPA
     - N/A

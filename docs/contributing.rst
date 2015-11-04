Contributing to RackHD
======================

The https://github.com/emccode/RackHD repository acts as a single source location to help you get or build all the pieces to learn about,
take advantage of, and contribute to RackHD. The individual repositories are within the organizations https://github.com/RackHD. We accept
bugs at github issues for bug reports and pull requests for the individual repositories.

The code for RackHD is a combination of Javascript/Node.js and C, available under the Apache 2.0
license (or compatible sublicences for library dependencies).

We maintain a mailing list at https://groups.google.com/d/forum/rackhd. You can visit the group
through the web page, or subscribe directly from email by sending email to rackhd+subscribe@googlegroups.com

We also have a #RackHD slack channel at https://codecommunity.slack.com/messages/rackhd/.
You can get an invite by requesting one at http://community.emccode.com.


Project and code repository overview
------------------------------------
The RackHD project is a collection of libraries and applications housed at https://github.com/RackHD/.

Applications
^^^^^^^^^^^^^^^^^^^^^^^^

+-------------+---------------------------------------+----------------------------------+
| Application | Repository                            || Description                     |
+=============+=======================================+==================================+
|on-tftp      |https://github.com/RackHD/on-tftp      || Node.js application provided    |
|             |                                       || TFTP service integrated with    |
|             |                                       || the workflow engine.            |
+-------------+---------------------------------------+----------------------------------+
|on-http      |https://github.com/RackHD/on-http      || Node.js application provided    |
|             |                                       || HTTP service integrated with    |
|             |                                       || the workflow engine.            |
+-------------+---------------------------------------+----------------------------------+
|on-syslog    |https://github.com/RackHD/on-syslog    || Syslog endpoint integrated to   |
|             |                                       || feed data to the workflow       |
|             |                                       || engine.                         |
+-------------+---------------------------------------+----------------------------------+
|on-taskgraph |https://github.com/RackHD/on-taskgraph || Node.js application providing   |
|             |                                       || the workflow engine.            |
+-------------+---------------------------------------+----------------------------------+
|on-dhcp-proxy|https://github.com/RackHD/on-dhcp-proxy|| Node.js application providing   |
|             |                                       || DHCP proxy support in the       |
|             |                                       || workflow engine.                |
+-------------+---------------------------------------+----------------------------------+



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
     - | https://github.com/RackHD/on-web-ui
     - | Initial web interfaces to some of the APIs - multiple
       | interfaces embedded into a single project.
   * - statsd
     - https://github.com/RackHD/on-statsd
     - | A local statsD implementation that makes it easy to deploy
       | on a local machine for capturing application metrics.

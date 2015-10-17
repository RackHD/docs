Contributing to RackHD
======================

Join the [..........] mailing list

Join the #rackhd [....] channel


Project and code repository overview
------------------------------------

.. seealso:: :doc:`software_architecture`

Applications
^^^^^^^^^^^^

- on-tftp
    https://github.com/RackHD/on-tftp
nodejs application provided TFTP service integrated with the workflow engine

- on-http
    https://github.com/RackHD/on-http
internal HTTP REST API interfaces integrated with the workflow engine

- on-syslog
    https://github.com/RackHD/on-syslog
syslog endpoint integrated to feed data into workflow engine

- on-taskgraph
    https://github.com/RackHD/on-taskgraph
nodejs application providing the workflow engine

- on-dhcp-proxy
    https://github.com/RackHD/on-dhcp-proxy
nodejs application providing DHCP proxy support integrated into the workflow engine

- ONSERVE:
  https://github.com/RackHD/onserve

Libraries
^^^^^^^^^

- core
    https://github.com/RackHD/on-core
Core libraries in use across NodeJS applications

- tasks
    https://github.com/RackHD/on-tasks
NodeJS task library for the workflow engine

Supplemental code
^^^^^^^^^^^^^^^^^

- tools
      https://github.com/RackHD/on-tools
Useful dev tools for running locally

- Web user interface
    https://github.com/RackHD/on-web-ui
Initial web interfaces to some of the APIs - multiple interfaces embedded into a single project

- integration tests
    https://github.com/RackHD/tests
Integration tests with code for deploying and running those tests, as well as the tests themselves

- statsd
    https://github.com/RackHD/on-statsd
A local statsD implementation that makes it easy to deploy on a local machine for capturing application metrics

RackHD API
-------------------------

Our REST based API is the abstraction layer for the low-level management tasks
that are performed on hardware devices, and information about those devices.
For example, when a compute server is "discovered" (see :doc:`passive_discovery`
for more details on this process), the information about that server is expressed
as `nodes` and `catalogs` in the RackHD API. When you want to re-image that
compute node, the RackHD API is used to activate a workflow containing the tasks
that are appropriate to doing that function.

The RackHD API can be used to manage nodes, catalogs, workflows, tasks, templates,
pollers, and other entities. For the complete list of functions, generate the RackHD
API documentation as described below.

**List All Nodes**

.. code::

  curl http://<server>:8080/api/common/nodes | python -mjson.tool

**Set the Active Workflow**

.. code::

  curl http://<server>:8080/api/common/nodes/<identifier>/workflows/active | python -mjson.tool


Starting and Stopping the API Server
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The API server runs by default. Use the following commands to stop or start the API server.

================ ===============================
 Action           Command
================ ===============================
Stop API server   `sudo service on-http stop`
Start API server  `sudo service on-http start`
================ ===============================


Generating API Documentation
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You can generate an HTML version of the API documentation by cloning the *on-http*
repository and running the following command.

.. code::

  $ git clone https://github.com/RackHD/on-http
  $ cd on-http
  $ npm install
  $ npm run apidoc

The default and example quick start build that we describe in :doc:`getting_started`
has the API docs rendered and embedded within that instance for easy use, available
at http://[IP ADDRESS OF VM]:8080/docs/

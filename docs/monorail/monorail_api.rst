Monorail API
-------------------------

The Monorail API is an abstraction layer for the low-level management tasks that are performed on hardware devices.
For example, to boot an image on a compute node, the Monorail API is used to activate a workload containing
the tasks that are appropriate to the device.

Any user on the ORA_ADMIN_IF network can call any Monorail API. The Monorail API can be used to manage nodes, catalogs, workflows, tasks, templates, pollers, and other
entities. For the complete list of functions, generate the API documentation as described below.

**Example: List All Nodes**

.. code::

  curl http://<server>:8080/api/common/nodes | python -mjson.tool

**Example: Set the Active Workflow**

.. code::

  curl http://<server>:8080/api/common/nodes/<identifier>/workflows/active | python -mjson.tool


Starting and Stopping the API Server
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The API server runs by default. Use the following commands to stop or start the API server.

================ ===============================
 Action           Command
================ ===============================
Stop API server   ``sudo service on-http stop``
Start API server  ``sudo service on-http start``
================ ===============================


Generating API Documentation
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

You can generate an HTML version of the API documentation by cloning the *on-http* repository and running the following command.

.. code::

  $ npm run apidoc

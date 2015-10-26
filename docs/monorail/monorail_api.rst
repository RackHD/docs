Monorail API
======================

Workflows
--------------------

Showing All Available Workflows
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: rest

   curl http://<server>:<port>/api/common/workflows/library | python -mjson.tool

Nodes
--------------

Listing All Nodes
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: rest

   curl http://<server>:<port>/api/common/nodes | python -mjson.tool

Display a Node's Catalogs
--------------------------

.. code-block:: rest

   curl http://localhost:8080/api/common/nodes/54d93996910b6fd5d68ffba7/catalogs | python -mjson.tool

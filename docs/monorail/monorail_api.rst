Monorail API
======================

Workflows
--------------------

Show All Available Workflows
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

  .. code-block:: rest

   curl http://<server>:<port>/api/common/workflows/library | python -mjson.tool

Display Current Active Workflow
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

  .. code-block:: rest

    curl http://<server>:<port>/api/common/nodes/<identifier>/workflows/active | python -mjson.tool

Set the Active Workflow
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

  .. code-block:: rest

       curl -X POST <server>:<port>/api/common/nodes/<identifier>/workflows?name=Graph.InstallCentOS

Delete the Active Workflow
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

  .. code-block:: rest

        curl -X DELETE <server>:<port>/api/common/nodes/<identifier>/workflows/active


Nodes
--------------

List All Nodes
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

  .. code-block:: rest

   curl http://<server>:<port>/api/common/nodes | python -mjson.tool

Display a Node's Catalogs
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

  .. code-block:: rest

   curl http://<server>:<port>/api/common/nodes/<identifier>/catalogs | python -mjson.tool

Delete a Node
~~~~~~~~~~~~~~~~~~~~~~

   .. code-block:: rests

      curl -X DELETE http://<server>:<port>/api/common/nodes/<identifier>

Configure Node OBM Settings
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

    1. Create a JSON File with the Settings (in this case, **obm.json**)

    .. code-block:: rest

        {"obmSettings":[{"service":"ipmi-obm-service","config":{"user":"<username>",
        "password":"<password>","host": "<MAC address>"}}]}

    2. Execute Command

    .. code-block:: rest

     curl -X PATCH -H "Content-Type: application/json" --data @obm.json
     http://<server>:<port>/api/common/nodes/<identifier>


Templates
------------------

Display a Template
~~~~~~~~~~~~~~~~~~~~~~~~~

  .. code-block:: rest

    curl <server>:<port>/api/common/templates/library/<template_name>

Patch a Template
~~~~~~~~~~~~~~~~~~~~~~~~

   .. code-block:: rest

    curl -X PUT -H "Content-Type: application/octet-stream" --data-binary @/tmp/centos70.txt
    <server>:<port>/api/common/templates/library/<template_name>


LED Lights
----------------

Turn On LED Identify Light
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

   .. code-block:: rest

         curl -X POST -H "Content-Type:application/json"
         http://<server>:<port>/api/common/nodes/<identifier>/obm/identify -d '{"value":true}'

Turn Off LED Identify Light
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

  .. code-block:: rest

   curl -X POST -H "Content-Type:application/json"
   http://<server>:<port>/api/common/nodes/<identifier>/obm/identify -d '{"value":false}'

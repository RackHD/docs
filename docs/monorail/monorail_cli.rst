Monorail Command Line Interface
====================================

Workflows
--------------------

List All Available Workflows
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

  .. code-block:: rest

       /usr/local/bin/rencli list-workflows

Display the Current Active Workflow
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

  .. code-block:: rest

    /usr/local/bin/rencli --host <server>:<port> node-workflow-active <identifier>

Set the Active Workflow
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

  .. code-block:: rest

    curl -X POST -H "Content-Type:application/json" localhost/api/current
    /nodes/<identifier>/workflows -d '{"name": "Graph.InstallCoreOS",
    "options": { "defaults": { "obmServiceName": "vbox-obm-service" } } }'


Nodes
--------------

Configure Node OBM Settings
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

  .. code-block:: rest

        curl -X PATCH -H "Content-Type:application/json" localhost/api/current
        /nodes/5542b78c130198aa216da3ac -d '        { "obmSettings": [
        { "service": "vbox-obm-service", "config":
        { "alias": "<vboxmachinename>", "user": "<user>" } } ] }'


Power Off a Node
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

  .. code-block:: rest

    /usr/local/bin/rencli --host <server>:<port> off <MACaddress>

Power On a Node
~~~~~~~~~~~~~~~~~~~~~~~~

   .. code-block:: rest

    /usr/local/bin/rencli --host <server>:<port> on <MACaddress>

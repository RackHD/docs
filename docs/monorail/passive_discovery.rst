Hardware Discovery
---------------------------

Switch type nodes can be discovered either by running a discovery graph against
them or creating via http calls with the autoDiscover field set to true.

Automatic Discovery
~~~~~~~~~~~~~~~~~~~~~~~~~

A new node created by posting to /api/1.1/node will be
automatially discovered if:

* the type is 'switch'
* it has an snmpSettings field with the host to query and snmp community string
* the autoDiscover field is set to true

## Create a Node to be Auto-Discovered**

.. code-block:: REST

    POST /api/1.1/nodes
    {
        "name": "nodeName"
        "type": "switch",
        "autoDiscover": true
        "snmpSettings": {
            host: "10.1.1.3",
            community: "public"
        }
    }

.. code-block:: REST

    curl -X POST \
        -H 'Content-Type: application/json' \
        -d '{"name":"nodeName", "type": "switch", "autoDiscover":true,
         "snmpSettings": {"host": "10.1.1.3", "community": "public"}}' \
        <server>/api/1.1/nodes

.. literalinclude:: samples/auto-discover-node.json
   :language: JSON

Discover an existing switch node
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you want to discover a switch node manually either create
the node without an autoDiscover option or set autoDiscover to false you
can then run discovery against the node by posting to
/api/1.1/nodes/:identifier/workflows and specifying the node id
in the graph options, eg:

.. code-block:: REST

    POST /api/1.1/nodes/55b6afba024fd1b349afc148/workflows
    {
        "name": "Graph.Switch.Discovery",
        "options": {
            "defaults": {
                "nodeId": "55b6afba024fd1b349afc148"
            }
        }
    }

.. code-block:: REST

    curl -X POST \
        -H 'Content-Type: application/json' \
        -d '{"name": "Graph.Switch.Discovery",
         "options":{"defaults":{"nodeId": "55b6afba024fd1b349afc148"}}}' \
        <server>/api/1.1/nodes/55b6afba024fd1b349afc148/workflows

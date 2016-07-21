Passive Hardware Discovery
---------------------------

Switch type nodes can be discovered either by running a discovery graph against
them or creating via http calls with the autoDiscover field set to true.

Automatic Discovery
~~~~~~~~~~~~~~~~~~~~~~~~~

A new node created by posting to /api/current/node will be
automatially discovered if:

* the type is 'switch'
* it has an snmpSettings field with the host to query and snmp community string
* the autoDiscover field is set to true

**Create a Node to be Auto-Discovered**

.. code-block:: REST

    POST /api/current/nodes
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
        -d '{"name":"nodeName", "type": "switch", "autoDiscover":true, \
         "snmpSettings": {"host": "10.1.1.3", "community": "public"}}' \
        <server>/api/current/nodes

.. literalinclude:: samples/auto-discover-switch-node.json
   :language: JSON


Discover an existing device node
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you want to discover a switch node manually either create
the node without an autoDiscover option or set autoDiscover to false you
can then run discovery against the node by posting to
/api/current/nodes/:identifier/workflows and specifying the node id
in the graph options, eg:

.. code-block:: REST

    POST /api/current/nodes/55b6afba024fd1b349afc148/workflows
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
        -d '{"name": "Graph.Switch.Discovery", \
         "options":{"defaults":{"nodeId": "55b6afba024fd1b349afc148"}}}' \
        <server>/api/current/nodes/55b6afba024fd1b349afc148/workflows

You can also use this mechanism to discovery a compute server or PDU, simply
using different settings. For example, a smart PDU:

.. code-block:: REST

    curl -X POST \
        -H 'Content-Type: application/json' \
        -d '{"name":"nodeName", "type": "pdu", \
         "snmpSettings": {"host": "10.1.1.3", "community": "public"}}' \
        <server>/api/current/nodes

.. code-block:: REST

    curl -X POST \
        -H 'Content-Type: application/json' \
        -d '{"name": "Graph.PDU.Discovery", \
         "options":{"defaults":{"nodeId": "55b6afba024fd1b349afc148"}}}' \
        <server>/api/current/nodes/55b6afba024fd1b349afc148/workflows

And a management server (or other server you do not want to or ca not to reboot
to interrogate)

.. code-block:: REST

    curl -X POST \
        -H 'Content-Type: application/json' \
        -d '{"name":"nodeName", "type": "compute", \
         "ipmi-obm-service": {"host": "10.1.1.3", "user": "admin",  \
         "password": "admin"}}' \
        <server>/api/current/nodes

.. code-block:: REST

    curl -X POST \
        -H 'Content-Type: application/json' \
        -d '{"name": "Graph.MgmtSKU.Discovery",
         "options":{"defaults":{"nodeId": "55b6afba024fd1b349afc148"}}}' \
        <server>/api/current/nodes/55b6afba024fd1b349afc148/workflows

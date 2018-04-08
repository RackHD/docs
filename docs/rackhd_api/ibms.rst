In Band Management Settings (IBMs)
==================================

.. contents:: Table of Contents

API Commands for IBMs
-----------------------------

The following are common API commands that can be used when running the *on-http* process.

Get list of In Band Management settings that have been associated with nodes.

**Get list of IBMs settings**

::

    GET /api/current/ibms

::

    curl <server>/api/current/ibms

**Get list of IBMs schemas showing required properties to create an IBM**

::

    GET /api/current/ibms/definitions

::

    curl <server>/api/current/ibms/definitions

**Create or update a single IBM service and associate it with a node**

::

    PUT /api/current/ibms

::

    curl -X PUT -H "Content-Type: application/json" -d '{ "nodeId": <node id>, "service": "snmp-ibm-service", "config": { "community": "public", "host": "<host ip>" } }' /api/current/ibms

Example output of PUT

::

    {
      "id": "591c569c087752c67428e4b3",
      "node": "/api/2.0/nodes/590cbcbf29ba9e40471c9f3c",
      "service": "snmp-ibm-service",
      "config": {
        "host": "172.31.128.2"
      }
    }

**Get a specific IBM setting**

::

    GET /api/current/ibms/<id>

::

    curl <server>/api/current/ibms/<id>

**PATCH an IBM setting**

::

    PATCH /api/current/ibms/<id>

::

    curl -X PUT -H "Content-Type: application/json" -d '{ "nodeId": <node id>, "service": "snmp-ibm-service", "config": { "community": "public", "host": "<host ip>" } }' /api/current/ibms/<id>

**Delete an IBM setting**

::

    DELETE /api/current/ibms/<id>

::

    curl -X DELETE <server>/api/current/ibms/<id>
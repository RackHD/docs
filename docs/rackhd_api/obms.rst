Out of Band Management Settings (OBMs)
======================================

.. contents:: Table of Contents

API Commands for OBMs
-----------------------------

The following are common API commands that can be used when running the *on-http* process.

Get list of Out of Band Management settings that have been associated with nodes.

**Get list of OBMs settings**

::

    GET /api/current/obms

::

    curl <server>/api/current/obms

**Get list of OBMs schemas showing required properties to create an OBM**

::

    GET /api/current/obms/definitions

::

    curl <server>/api/current/obms/definitions

**Create or update a single OBM service and associate it with a node**

::

    PUT /api/current/obms

::

    curl -X PUT -H "Content-Type: application/json" -d '{ "nodeId": <node id>, "service": "ipmi-obm-service", "config": { "user": "admin", "password": "admin", "host": "<host ip>" } }' /api/current/obms

Example output of PUT

::

    {
      "id": "5911fa6447f8b7b207f9a485",
      "node": "/api/2.0/nodes/590cbcbf29ba9e40471c9f3c",
      "service": "ipmi-obm-service",
      "config": {
        "user": "admin",
        "host": "172.31.128.2"
      }
    }

**Get a specific OBM setting**

::

    GET /api/current/obms/<id>

::

    curl <server>/api/current/obms/<id>

**PATCH an OBM setting**

::

    PATCH /api/current/obms/<id>

::

    curl -X PUT -H "Content-Type: application/json" -d '{ "nodeId": <node id>, "service": "ipmi-obm-service", "config": { "user": "admin", "password": "admin", "host": "<host ip>" } }' /api/current/obms/<id>

**Delete an OBM setting**

::

    DELETE /api/current/obms/<id>

::

    curl -X DELETE <server>/api/current/obms/<id>


**To set a no-op OBM setting on a node**

::

    curl -X PUT -H "Content-Type:application/json" localhost/api/current/nodes/5542b78c130198aa216da3ac -d '{  { "service": "noop-obm-service", "config": { } } }'


**To set a IPMI OBM setting on a node**

.. code-block:: REST

    curl -X PUT -H 'Content-Type: application/json' -d ' { "service": "ipmi-obm-service", "config": { "host": "<host ip>", "user": "admin", "password": "admin" } }' <server>/api/current/nodes/<nodeID>/obm

.. _node-api-tags-ref-label:


**How to use obms when more than one obm are present on a node**

Example: when update firmware workflow is called on a node that has multiple obms (ipmi-obm-service, redfish-obm-service), the payload needs to call out what obm service to use for certain tasks within the workflow that use the obm service..

::

    POST /api/current/nodes/<id>/nodes/workflows?name=Graph.Dell.Racadm.Update.Firmware

.. code-block:: JSON

	{
	  "options": {
		  "defaults": {
				"filePath": "xyz",
				"serverUsername": "abc",
				"serverPassword": "123",
				"serverFilePath": "def"
		   },
	   "set-boot-pxe": {
				"obmServiceName": "ipmi-obm-service"
				},
	   "reboot": {
				"obmServiceName": "ipmi-obm-service"
	   }          
	 }
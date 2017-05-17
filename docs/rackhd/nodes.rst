Nodes and Catalogs
---------------------------

Nodes are the elements that RackHD manages - compute servers, switches, etc.
Nodes typically have at least one catalog, and can have :doc:`pollers` and
:doc:`graphs` assigned to or working against that node.


Catalogs are free form data structures with information about the nodes. Catalogs
are created during 'discovery' workflows, and present information that can be
requested via API and is available to workflows to operate against.


Defining Nodes
~~~~~~~~~~~~~~


Nodes are defined via a JSON definition that conform to this schema:

- id (string): unique identifier for the node
- type (string): a human readable name for the graph
- name (string): a unique name used by the system and the API to refer to the graph
- autodiscover (boolean):
- sku (string): the SKU 'id' that has been matched from the SKU workflow task
- createdAt (string): ISO8601 date string of time resource was created
- updatedAt (string): ISO8601 date string of time resource was last updated
- identifiers (array of strings): a list of strings that make up alternative identifiers for the node
- obms (array of objects): a list of objects that define out-of-band management access mechanisms
- relations (array of objects): a list of relationship objects


Defining Catalogs
~~~~~~~~~~~~~~~~

- id (string): unique identifier for the node
- createdAt (string): ISO8601 date string of time resource was created
- updatedAt (string): ISO8601 date string of time resource was last updated
- data (json): A JSON data structure specific to the catalog tool
- node (string): the node to which this catalog is associated
- source (string): type of the data


API Commands for Nodes
~~~~~~~~~~~~~~~~~~~~~~

The following are common API commands that can be used when running the *on-http* process.


**Get Nodes**

::

    GET /api/current/nodes

::

    curl <server>/api/current/nodes


**Get Specific Node**

::

    GET /api/current/nodes/<id>

::

    curl <server>/api/current/nodes/<id>


Sample switch node after Discovery

.. literalinclude:: samples/auto-discover-switch-node.json
   :language: JSON

Sample compute node after Discovery

.. literalinclude:: samples/discovered-compute-node.json
   :language: JSON


**List all the (latest) catalog data associated with a node**

::

    GET /api/current/nodes/<id>/catalogs

::

    curl <server>/api/current/nodes<id>/catalogs



**To retrieve a specific catalog source for a node**

::

    GET /api/current/nodes/<id>/catalogs/<source>

::

    curl <server>/api/current/nodes<id>/catalogs/<source>


Sample Output:

.. literalinclude:: samples/vm-dmi-catalog.json
   :language: JSON

Out of Band Management Settings
~~~~~~~~~~~~~~~~~~~~~~~~~~~~
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

In Band Management Settings
~~~~~~~~~~~~~~~~~~~~~~~~~~~~
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

Node Tags
~~~~~~~~~

**Add a tag to a node**

::

    PATCH /api/current/nodes/<id>/tags

::

    curl -H "Content-Type: application/json" -X PATCH -d '{ "tags": [<list of tags>]}' <server>/api/current/nodes/<id>/tags

**List tags for a node**

::

    GET /api/current/nodes/<id>/tags

::

    curl <server>/api/current/nodes/<id>/tags

**Delete a tag from a node**

::

    DELETE /api/current/nodes/<id>/tags/<tagname>

::

    curl -X DELETE <server>/api/current/nodes/<id>/tags/<tagname>

Node Relations
~~~~~~~~~~~~~~

**List relations for a node** 

::

    GET <server>/api/current/nodes/<id>/relations
::

    curl <server>/api/current/nodes/<id>/relations

Sample response:

.. literalinclude:: samples/relations.json
   :language: JSON

**Add relations to a node** 

::

    PUT <server>/api/current/nodes/<id>/relations

::
    
    curl -H "Content-Type: application/json" -X PUT -d '{ <relationType>: [<list of targets>]}' <server>/api/2.0/nodes/<id>/relations

Sample request body:

.. literalinclude:: samples/edit-relations-req.json
   :language: JSON

Sample response body:

.. literalinclude:: samples/relations-added.json
   :language: JSON

**Remove Relations from a node**

::

    DELETE <server>/api/current/nodes/<id>/relations

::

    curl -H "Content-Type: application/json" -X DELETE -d '{ <relationType>: [<list of targets>]}' <server>/api/current/nodes/<id>/relations

Sample request body:

.. literalinclude:: samples/edit-relations-req.json
   :language: JSON

Sample response body:

.. literalinclude:: samples/relations-deleted.json
   :language: JSON

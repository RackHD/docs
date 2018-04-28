Nodes
=============================

.. contents:: Table of Contents

Nodes are the elements that RackHD manages - compute servers, switches, etc.
Nodes typically have at least one catalog, and can have :doc:`pollers` and
:doc:`workflows` assigned to or working against that node.

Defining Nodes
-----------------------------

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

API Commands for Nodes
-----------------------------

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

.. literalinclude:: ../samples/auto-discover-switch-node.json
   :language: JSON

Sample compute node after Discovery

.. literalinclude:: ../samples/discovered-compute-node.json
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

.. literalinclude:: ../samples/vm-dmi-catalog.json
   :language: JSON

.. _node-api-tags-ref-label:

Node Tags
-----------------------------

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
-----------------------------

**List relations for a node**

::

    GET <server>/api/current/nodes/<id>/relations

::

    curl <server>/api/current/nodes/<id>/relations

Sample response:

.. literalinclude:: ../samples/relations.json
   :language: JSON

**Add relations to a node**

::

    PUT <server>/api/current/nodes/<id>/relations

::

    curl -H "Content-Type: application/json" -X PUT -d '{ <relationType>: [<list of targets>]}' <server>/api/2.0/nodes/<id>/relations

Sample request body:

.. literalinclude:: ../samples/edit-relations-req.json
   :language: JSON

Sample response body:

.. literalinclude:: ../samples/relations-added.json
   :language: JSON

**Remove Relations from a node**

::

    DELETE <server>/api/current/nodes/<id>/relations

::

    curl -H "Content-Type: application/json" -X DELETE -d '{ <relationType>: [<list of targets>]}' <server>/api/current/nodes/<id>/relations

Sample request body:

.. literalinclude:: ../samples/edit-relations-req.json
   :language: JSON

Sample response body:

.. literalinclude:: ../samples/relations-deleted.json
   :language: JSON

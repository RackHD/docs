Nodes
---------------------------

Nodes are the elements that RackHD manages - compute servers, switches, etc.
Nodes typically have at least one catalog, and can have :doc:`pollers` and
:doc:`graphs` assigned to or working against that node.

Catalogs
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Catalogs are free form data structures with information about the nodes. Catalogs
are created during 'discovery' workflows, and present information that can be
requested via API and is available to workflows to operate against.


Defining Nodes
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Nodes are defined via a JSON definition that conform to this schema:

- id (string): unique identifier for the node
- type (string): a human readable name for the graph
- name (string): a unique name used by the system and the API to refer to the graph
- autodiscover (boolean):
- sku (string): the SKU 'id' that has been matched from the SKU workflow task
- createdAt (string): ISO8601 date string of time resource was created
- updatedAt (string): ISO8601 date string of time resource was last updated
- identifiers (array of strings): a list of strings that make up alternative identifiers for the node.
- obmSettings (array of objects): a list of objects that define out-of-band management access mechanisms
- relations (array of objects): a list of relationship objects


Defining Catalogs
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- id (string): unique identifier for the node
- createdAt (string): ISO8601 date string of time resource was created
- updatedAt (string): ISO8601 date string of time resource was last updated
- data (json): A JSON data structure specific to the catalog tool
- node (string): the node to which this catalog is associated
- source (string): type of the data


API Commands for Nodes
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The following are common API commands that can be used when running the *on-http* process.


**Get Nodes**

::

    GET /api/1.1/nodes

::

    curl <server>/api/1.1/nodes


**Get Specific Node**

::

    GET /api/1.1/nodes/<id>

::

    curl <server>/api/1.1/nodes/<id>


Sample switch node after Discovery

.. literalinclude:: samples/auto-discover-switch-node.json
   :language: JSON

Sample compute node after Discovery

.. literalinclude:: samples/discovered-compute-node.json
   :language: JSON


**List all the (latest) catalog data associated with a node**

::

    GET /api/1.1/nodes/<id>/catalogs

::

    curl <server>/api/1.1/nodes<id>/catalogs



**To retrieve a specific catalog source for a node**

::

    GET /api/1.1/nodes/<id>/catalogs/<source>

::

    curl <server>/api/1.1/nodes<id>/catalogs/<source>


Sample Output:

.. literalinclude:: samples/vm-dmi-catalog.json
   :language: JSON

**To set a no-op OBM setting on a node**

::

    curl -X PATCH -H "Content-Type:application/json" localhost/api/current/nodes/5542b78c130198aa216da3ac -d '{ "obmSettings": [ { "service": "noop-obm-service", "config": { } } ] }'

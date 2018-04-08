Catalogs
=============================

.. contents:: Table of Contents

Catalogs are free form data structures with information about the nodes. Catalogs
are created during 'discovery' workflows, and present information that can be
requested via API and is available to workflows to operate against.

Defining Catalogs
-----------------------------

- id (string): unique identifier for the node
- createdAt (string): ISO8601 date string of time resource was created
- updatedAt (string): ISO8601 date string of time resource was last updated
- data (json): A JSON data structure specific to the catalog tool
- node (string): the node to which this catalog is associated
- source (string): type of the data


API Commands for Catalogs
-----------------------------

The following are common API commands that can be used when running the *on-http* process.

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
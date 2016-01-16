Workflow SKU Support
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The SKU API provides functionality to categorize nodes into groups based on data
present in a node's catalogs. SKU matching is done using a series of rules. If
all rules of a given SKU match the latest version of a node's catalog set, then
that SKU will be assigned to the node.

Upon discovering a node, the SKU will be assigned based on all existing SKU
definitions in the system. SKUs for all nodes will be re-generated whenever a
SKU definition is added, updated or deleted.

A default graph can also be assigned to a SKU. When a node is discovered that
matches the SKU, the specified graph will be executed on the node.

**Example**


With a node that has the following catalog fields:

.. code-block:: javascript

    {
      "source": "dmi",
      "data": {
        "dmi": {
          "base_board": {
            "manufacturer": "Intel Corporation"
          }
        },
        "memory": {
          "total": "32946864kB"
          "free": "31682528kB"
        }
        /* ... */
      }
    }

We could match against these fields with this SKU definition:

.. code-block:: javascript

    {
      "name": "Intel 32GB RAM",
      "rules": [
        {
          "path": "dmi.dmi.base_board.manufacturer",
          "contains": "Intel"
        },
        {
          "path": "dmi.memory.total",
          "equals": "32946864kB"
        }
      ]
    }

In both cases, the "path" string starts with "dmi" to signify that the rule
should apply to the catalog with a "source" value of "dmi".

This example makes use of the "contains" and "equals" rules. See the table at
the bottom of this document for a list of additional validation rules that can
be applied.

Package Support (skupack)
^^^^^^^^^^^^^^^^^^^^^^

The SKU package API provides functionality to override the set of files served
to a node by on-http with SKU specific files.  If a SKU requires additional
operations during OS provisioning, the SKU package can be used to serve out
SKU specific installation scripts that override the default scripts and perform 
those operations.

The SKU package can be upload to a specific SKU id or it can be bundled with a 
set of rules to register a SKU during the package upload.

API commands
^^^^^^^^^^^^^^^^^^^^^^

When running the on-http process, these are some common API commands you
can send.

**Create a New SKU with a Node**

.. code-block:: REST

    POST /api/1.1/skus
    {
      "name": "Intel 32GB RAM",
      "rules": [
        {
          "path": "dmi.Base Board Information.manufacturer",
          "contains": "Intel"
        },
        {
          "path": "ohai.dmi.memory.total",
          "equals": "32946864kB"
        }
      ],
      "discoveryGraphName": "Graph.InstallCoreOS",
      "discoveryGraphOptions": {
        "username": "testuser",
        "password": "hello",
        "hostname": "mycoreos"
      }
    }

.. literalinclude:: monorail/samples/sku.json
   :language: JSON


**Create a SKU to Auto-Configure IPMI Settings**


.. code-block:: REST

    POST /api/1.1/skus
    {
        "name": "Default IPMI settings for Quanta servers",
        "discoveryGraphName": "Graph.Obm.Ipmi.CreateSettings",
        "discoveryGraphOptions": {
            "defaults": {
                "user": "admin",
                "password": "admin"
            }
        },
        "rules": [
            {
                "path": "bmc.IP Address"
            },
            {
                "path": "dmi.Base Board Information.Manufacturer",
                "equals": "Quanta"
            }
        ]
    }

**Get List of SKUs**


.. code-block:: REST

    GET /api/1.1/skus

.. code-block:: REST

    curl <server>/api/1.1/skus


**Get Definition for a Single SKU**


.. code-block:: REST

    GET /api/1.1/skus/:id

.. code-block:: REST

    curl <server>/api/1.1/skus/<skuid>


**Update a Single SKU**


.. code-block:: REST

    PATCH /api/1.1/skus/:id
    {
        "name": "Custom SKU Name"
    }

.. code-block:: REST

    curl -X PATCH \
        -H 'Content-Type: application/json' \
        -d '{"name":"Custom SKU Name"}' \
        <server>/api/1.1/skus/<skuid>


**Delete a Single SKU**


.. code-block:: REST

    DELETE /api/1.1/skus/:id

.. code-block:: REST

    curl -X DELETE <server>/api/1.1/skus/<skuid>

**Register a new SKU with a pack**


.. code-block:: REST

    POST /api/1.1/skus/pack

.. code-block:: REST

    curl -X POST --data-binary @pack.tar.gz <server>/api/1.1/skus/pack

**Add a SKU pack**


.. code-block:: REST

    PUT /api/1.1/skus/:id/pack

.. code-block:: REST

    curl -T pack.tar.gz <server>/api/1.1/skus/<skuid>/pack

**Delete a SKU pack**


.. code-block:: REST

    DELETE /api/1.1/skus/:id/pack

.. code-block:: REST

    curl -X DELETE <server>/api/1.1/skus/<skuid>/pack

SKU JSON format
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

SKUs are defined via JSON, with these required fields:

+------------------------+-----------+--------------------------+----------------------------------------------------------+
| Name                   |  Type     | Flags                    | Description                                              |
+========================+===========+==========================+==========================================================+
| name                   |  String   | **required**, **unique** | Unique name identifying this SKU definition.             |
+------------------------+-----------+--------------------------+----------------------------------------------------------+
| rules                  |  Object[] | **required**             | Array of validation rules that define the SKU.           |
+------------------------+-----------+--------------------------+----------------------------------------------------------+
| rules[].path           |  String   | **required**             | Path into the catalog to validate against.               |
+------------------------+-----------+--------------------------+----------------------------------------------------------+
| rules[].equals         |  \*       | *optional*               | Exact value to match against.                            |
+------------------------+-----------+--------------------------+----------------------------------------------------------+
| rules[].in             |  \*[]     | *optional*               | Array of possibly valid values.                          |
+------------------------+-----------+--------------------------+----------------------------------------------------------+
| rules[].notIn          |  \*[]     | *optional*               | Array of possibly invalid values.                        |
+------------------------+-----------+--------------------------+----------------------------------------------------------+
| rules[].contains       |  String   | *optional*               | A string that the value should contain.                  |
+------------------------+-----------+--------------------------+----------------------------------------------------------+
| rules[].notContains    |  String   | *optional*               | A string that the value should not contain.              |
+------------------------+-----------+--------------------------+----------------------------------------------------------+
| rules[].greaterThan    |  Number   | *optional*               | Number that the value should be greater than.            |
+------------------------+-----------+--------------------------+----------------------------------------------------------+
| rules[].lessThan       |  Number   | *optional*               | Number that the value should be less than.               |
+------------------------+-----------+--------------------------+----------------------------------------------------------+
| rules[].min            |  Number   | *optional*               | Number that the value should be greater than or equal to.|
+------------------------+-----------+--------------------------+----------------------------------------------------------+
| rules[].max            |  Number   | *optional*               | Number that the value should be less than or equal to.   |
+------------------------+-----------+--------------------------+----------------------------------------------------------+
| rules[].regex          |  String   | *optional*               | A regular expression that the value should match.        |
+------------------------+-----------+--------------------------+----------------------------------------------------------+
| rules[].notRegex       |  String   | *optional*               | A regular expression that the value should not match.    |
+------------------------+-----------+--------------------------+----------------------------------------------------------+
| discoveryGraphName     |  String   | *optional*               | Name of graph to run against matching nodes on discovery.|
+------------------------+-----------+--------------------------+----------------------------------------------------------+
| discoveryGraphOptions  |  Object   | *optional*               | Options to pass to the graph being run on node discovery.|
+------------------------+-----------+--------------------------+----------------------------------------------------------+

SKU Pack tar.gz format
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The SKU pack requires the 'config.json' to be at the root of the tar.gz file.  A typical
package may have a static and template directory.

.. code-block:: shell

    tar tzf pack.tar.gz:
    config.json
    static/
    static/common/
    static/common/discovery.overlay.cpio.gz
    templates/
    templates/ansible.pub
    templates/esx-ks


SKU Pack config.json format
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
.. code-block:: javascript

    {
      "name": "Intel 32GB RAM",
      "rules": [
        {
          "path": "dmi.dmi.base_board.manufacturer",
          "contains": "Intel"
        },
        {
          "path": "dmi.memory.total",
          "equals": "32946864kB"
        }
      ],
      "httpStaticRoot": "static",
      "httpTemplateRoot": "templates"
    }


Workflow Tag Support
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The Tag API provides functionality to automatically categorize nodes into
groups based on data present in a node's catalogs or by manually assigning
a tag to a node. When done automatically, tag matching is done using a
series of rules. If all rules of a given tag match the latest version of
a node's catalog set, then that tag will be assigned to the node.  A node
may be assigned many tags, both automatically through rules matching or
manually by the user.

Upon discovering a node, the tag will be assigned based on all existing tag
definitions in the system. Tags for all nodes will be re-generated whenever a
tag definition is added.  Tags that are currently assigned to a node are not
automatically removed from nodes when the rules backing a tag are deleted.


**Example**


With a node that has the following catalog fields:

.. code-block:: javascript

    {
      "source": "dmi",
      "data": {
          "Base Board Information": {
              "Manufacturer": "Intel Corporation"
          }
      },
      "memory": {
          "total": "32946864kB"
          "free": "31682528kB"
      }
      /* ... */
    }

We could match against these fields with this tag definition:

.. code-block:: javascript

    {
      "name": "Intel 32GB RAM",
      "rules": [
        {
          "path": "dmi.Base Board Information.Manufacturer",
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

API commands
^^^^^^^^^^^^^^^^^^^^^^

When running the on-http process, these are some common API commands you
can send.

If you want to view or manipulate tags directly on nodes, please see the API notes
at :ref:`node-api-tags-ref-label`.

**Create a New tag**

.. code-block:: REST

    POST /api/current/tags
    {
      "name": "Intel-32GB-RAM",
      "rules": [
        {
          "path": "dmi.Base Board Information.Manufacturer",
          "contains": "Intel"
        },
        {
          "path": "ohai.dmi.memory.total",
          "equals": "32946864kB"
        }
      ]
    }

**Get List of tags**


.. code-block:: REST

    GET /api/current/tags

.. code-block:: REST

    curl <server>/api/current/tags


**Get Definition for a Single tag**


.. code-block:: REST

    GET /api/current/tags/:tagname

.. code-block:: REST

    curl <server>/api/current/tags/<tagname>


**Delete a Single tag**


.. code-block:: REST

    DELETE /api/current/tags/:tagname

.. code-block:: REST

    curl -X DELETE <server>/api/current/tags/<tagname>

**List nodes with a tag**


.. code-block:: REST

    GET /api/current/tags/:tagname/nodes

.. code-block:: REST

    curl <server>/api/current/tags/<tagname>/nodes

**Post a workflow to all nodes with a tag**


.. code-block:: REST

    POST /api/current/tags/:tagname/nodes/workflows

.. code-block:: REST

    curl -H "Content-Type: application/json" -X POST -d @options.json <server>/api/current/tags/<tagname>/nodes/workflows

Tag JSON format
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Tag objects are defined via JSON using these fields:

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

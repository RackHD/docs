Refresh Node Discovery
----------------------

Compute type nodes can be re-discovered/refreshed either by running an immediate refresh discovery graph or a delayed refresh discovery graph using the same nodeID from the original discovery process. The node catalog(s) will updated with new entries.

Immediate Refresh Node Discovery
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

A node can be refreshed immediately by posting to /api/2.0/workflows with a payload. The node will be rebooted automatically and the node re-discovery process will start.

**Immediate Node Re-discovery example**

.. code-block:: REST

    POST /api/2.0/workflows
    {
        "name": "Graph.Refresh.Immediate.Discovery",
        "options": {
            "reset-at-start": {
                "nodeId": "<nodeId>"
            },
            "discovery-refresh-graph": {
                "graphOptions": {
                    "target": "<nodeId>"
                },
                "nodeId": "<nodeId>"
            },
            "generate-sku": {
                "nodeId": "<nodeId>"
            },
            "generate-enclosure": {
                "nodeId": "<nodeId>"
            },
            "create-default-pollers": {
                "nodeId": "<nodeId>"
            },
            "run-sku-graph": {
                "nodeId": "<nodeId>"
            },
            "nodeId": "<nodeId>"
        }
    }

.. code-block:: REST

   curl -X POST \
        -H 'Content-Type: application/json' \
        -d '{ "name":"Graph.Refresh.Immediate.Discovery",
              "options": {
                  "reset-at-start": {
                      "nodeId": "<nodeId>"
                  },
                  "discovery-refresh-graph": {
                      "graphOptions": {
                          "target": "<nodeId>"
                      },
                      "nodeId": "<nodeId>"
                  },
                  "generate-sku": {
                      "nodeId": "<nodeId>"
                  },
                  "generate-enclosure": {
                      "nodeId": "<nodeId>"
                  },
                  "create-default-pollers": {
                      "nodeId": "<nodeId>"
                  },
                  "run-sku-graph": {
                      "nodeId": "<nodeId>"
                  },
                  "nodeId": "<nodeId>"
              }
            }' \
        <server>/api/2.0/workflows


Delayed Refresh Node Discovery
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

An user can defer a node discovery by posting to /api/2.0/workflows with a payload. The user will need to manually reboot the node after executing the API before the node re-discovery/refresh process can start.

**Delayed Node Re-discovery example**

.. code-block:: REST

    POST /api/2.0/workflows
    {
        "name": "Graph.Refresh.Delayed.Discovery",
        "options": {
            "discovery-refresh-graph": {
                "graphOptions": {
                    "target": "<nodeId>"
                },
                "nodeId": "<nodeId>"
            },
            "generate-sku": {
                "nodeId": "<nodeId>"
            },
            "generate-enclosure": {
                "nodeId": "<nodeId>"
            },
            "create-default-pollers": {
                "nodeId": "<nodeId>"
            },
            "run-sku-graph": {
                "nodeId": "<nodeId>"
            },
            "nodeId": "<nodeId>"
        }
    }

.. code-block:: REST

   curl -X POST \
        -H 'Content-Type: application/json' \
        -d '{ "name":"Graph.Refresh.Delayed.Discovery",
              "options": {
                  "discovery-refresh-graph": {
                      "graphOptions": {
                          "target": "<nodeId>"
                      },
                      "nodeId": "<nodeId>"
                  },
                  "generate-sku": {
                      "nodeId": "<nodeId>"
                  },
                  "generate-enclosure": {
                      "nodeId": "<nodeId>"
                  },
                  "create-default-pollers": {
                      "nodeId": "<nodeId>"
                  },
                  "run-sku-graph": {
                      "nodeId": "<nodeId>"
                  },
                  "nodeId": "<nodeId>"
              }
            }' \
        <server>/api/2.0/workflows

**Example manually rebooting the node using ipmitool**

.. code-block:: REST

   ipmitool -H <BMC host IP address> -U <username> -P <password> chassis power reset


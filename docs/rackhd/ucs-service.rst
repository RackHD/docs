UCS-Service
-----------

The UCS-Service is an optional Rack-HD service that will enable RackHD to communicate with Cisco UCS Manger.  This allows RackHD to discover and manage the hardware under the UCS Manager.

UCS-Service Setup
~~~~~~~~~~~~~~~~~

The UCS-Service configuration can be set in the config.json file. The following options are supported:

================ ===============================
 Option           Description
================ ===============================
address           IP address the UCS-service will bind to
port              TCP port the UCS-service will bind to
httpsEnabled      set to "true" to enable https access
certFile          Certificate file for https (null for self signed)
keyFile           Key file for https (null for self signed)
debug             set to "true" to enable debugging
================ ===============================

To start the UCS-Service run

.. code::

  $ python app.py

UCS-Service API
~~~~~~~~~~~~~~~

The API for the UCS-Service can be accessed via a graphical GUI by directing a browser to https://<RackHD_Host>:7080/ui

UCS-Service Workflows
~~~~~~~~~~~~~~~~~~~~~

Default workflows to discover and catalog UCS nodes have been created.  There are separate workflows to discover physical UCS nodes, discover logical UCS servers, and to catalog both physical and logical UCS nodes.

Discover Nodes
^^^^^^^^^^^^^^

The Graph.Ucs.Discovery workflow will discover and catalog all physical and logical servers being managed by the specified UCS Manager.  It will create a node for each discovered service.  It will also create a ucs-obm-service for each node.  This obm service can then be used to manage the node.  The user must provide the address and login credentials for the UCS manger and the URI for the ucs-service.  Below is an example:

.. code-block:: JSON

 {
    "name": "Graph.Ucs.Discovery",
    "options":
    {
        "defaults":
        {
            "username": "admin",
            "password": "secret",
            "ucs": "172.31.128.252",
            "uri": "https://localhost:7080"
        },
        "when-discover-physical-ucs":
        {
            "discoverPhysicalServers": "true"
        },
        "when-discover-logical-ucs":
        {
            "discoverLogicalServer": "true"
        },
        "when-catalog-ucs":
        {
            "autoCatalogUcs": "true"
        }
    }
 }

.. list-table::
   :widths: 10 80
   :header-rows: 1

   * - Field
     - Description
   * - username
     - The username used to log into the UCS Manager
   * - password
     - The password used to log into the UCS Manager
   * - ucs
     - The hostname or IP address of the UCS Manager
   * - uri
     - The URI used to access the running UCS-service
   * - discoverPhysicalServers
     - If set to true, the workflow will create nodes for all physical servers discovered from the UCS Manager
   * - discoverLogicalServer
     - If set to true, the workflow will create nodes for all logical servers discovered from the UCS Manger
   * - autoCatalogUcs
     - If set to true, catalog information will be collected for each discovered node

Catalog Nodes
^^^^^^^^^^^^^

Once the UCS nodes have been discovered, the Graph.Ucs.Catalog can be run with the NodeId.  This graph will use the ucs-obm-service created by the discovery workflow so no other options are required.




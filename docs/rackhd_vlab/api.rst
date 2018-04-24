RackHD Operation with Restful API
==================================

.. contents:: Table of Contents

RackHD API 2.0
---------------

Overview and Data Model
~~~~~~~~~~~~~
In the previous modules, you had the opportunity to experiment with some RackHD APIs. In this section you will learn about two different RESTful endpoints in RackHD and experiment with them.
RackHD is designed to provide a REST (Representational state transfer) architecture to provide a RESTful API. RackHD currently has two RESTful interfaces: a Redfish API and native REST API 2.0.
The RESTful API 2.0 provides unique features that are not provided in Redfish API.

Common used RackHD 2.0 APIs
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**REST API (v2.0) – Get workflow history** (``Node-ID`` is obtained by the ``curl localhost:9090/api/2.0/nodes | jq .`` API.)

.. code-block:: shell

    curl localhost:9090/api/current/nodes/<Node-ID>/workflows | jq .

    # Example Response
    # …
    # "72d726cf-baf1-45fb-a0de-1278cdae72af": {
    #   "taskEndTime": "2018-03-02T12:25:07.716Z",
    #   "taskStartTime": "2018-03-02T12:24:58.788Z",
    #   "terminalOnStates": [
    #     "timeout",
    #     "cancelled",
    #     "failed"
    #   ],
    #   "state": "succeeded",
    #   "ignoreFailure": true,
    #   "waitingOn": {
    #     "b0cb0eb6-d783-4be2-af92-bdf170a79857": "succeeded"
    #   },
    # …

**REST API (v2.0) – Get active workflow**
In this example, the return is blank ([]), which means no workflow is actively running on this node.

.. code-block:: shell

    curl localhost:9090/api/current/nodes/<Node-ID>/workflows?active=true | jq .

    # Example Response
    # []

**REST API (v2.0) – Show RackHD configurations**
Show the RackHD configurations, by running the following command.

.. code-block:: shell

    curl localhost:9090/api/2.0/config | jq .

**REST API (v2.0) – lookup table**
Dump the IP address in the lookup table (where RackHD maintains the nodes IP), by running the following command

.. code-block:: shell

    curl localhost:9090/api/current/lookups | jq .

**REST API (v2.0) – Built-in workflow**
Show the name of all built-in workflow

.. code-block:: shell

    curl localhost:9090/api/2.0/workflows/graphs | jq '.' | grep injectableName | grep "Graph.*" | grep -v "Task"

**REST API (v2.0) – Issue a workflow**
Post a workflow to a specific node by running the following command.
In the following example, to post a workflow to Reset a Node, the ``Node-ID`` is obtained by the ``curl localhost:9090/api/2.0/nodes | jq .`` API.

.. code-block:: shell

    curl -X POST -H 'Content-Type: application/json' localhost:9090/api/current/nodes/<Node-ID>/workflows?name=Graph.Reset.Node | jq '.'


**SKU Pack**

.. code-block:: shell

    sudo apt-get install build-essential devscripts debhelper

    # clone the on-skupack repo. checkout to a released version.
    cd /tmp
    git clone https://github.com/RackHD/on-skupack.git
    git reset --hard release/1.3.0

    # Take Dell R630 as example:
    cd ~/tmp/on-skupack
    ./build-package.bash dell-r630 vlab

    # In tarballs folder, you will find sku pack package : dell-r630_vlab.tar.gz
    cd ~/tmp/on-skupack
    ls tarballs/

    #Register this SKU Pack:
    cd ~/tmp/on-skupack
    curl -X POST --data-binary @tarballs/dell-r630_vlab.tar.gz localhost:9090/api/current/skus/pack | jq '.'

    # Find the SKU id from below API:
    curl localhost:9090/api/current/skus | jq '.'

    # Find the nodes matched this SKU Pack (e.g. if you have a dell-r630 vNode, it will be associated with the dell-r630 skupack you just registered)
    curl localhost:9090/api/current/skus/<sku-id>/nodes | jq '.'

What is the benefit of SKU-Pack ?
SKU Packs allow you to assign specific workflows for specific SKUs. For example, before discovery, we can associate a "Dell firmware upgrade" workflow to Dell R630 SKU. Then when a new Dell R630 server being discovered, it will be automatically matched to dell-r630 sku, then the "firmware upgrade" workflow will run.


Redfish API
-------------

Overview and Data Model
~~~~~~~~~~~~~~~~~~~~~~~~~~~

The Redfish API deals with resources which are expressed based on an ``OData`` or ``JSON schema``. Resources are accessed through the usual HTTP operations: ``GET``, ``PUT``, ``POST``, etc., or a set of Actions that go beyond what CRUD HTTP operations can perform. An example of such an action is performing a system reset. API clients can use the schema to discover the semantics of the resource properties. The specification makes reference to three main category of objects:

* Systems – server, CPU, memory, devices, etc.
* Managers – BMC, Enclosure Manager or similar
* Chassis – racks, enclosures, blades, etc.

Common used RackHD Redfish APIs
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

List the Chassis that is managed by RackHD (equivalent to the enclosure node in REST API 2.0), by running the following command.

.. code-block:: shell

    curl localhost:9090/redfish/v1/Chassis | jq .

List the System being managed by RackHD (equivalent to compute node in API 2.0)

.. code-block:: shell

    curl localhost:9090/redfish/v1/Systems | jq .

List the SEL Log (System-ID is obtained in above step)

.. code-block:: shell

    curl localhost:9090/redfish/v1/systems/<System-ID>/LogServices/Sel | jq .

Show the CPU processor information

.. code-block:: shell

    curl localhost:9090/redfish/v1/Systems/<System-ID>/Processors/0 | jq .

Redfish API helper

.. code-block:: shell

    curl localhost:9090/redfish/v1 | jq .


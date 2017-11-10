RackHD Workflow Engine
======================

This tutorial explains how to use RackHD on-taskgraph as a stand alone service, known as the
RackHD Workflow Engine.

Prerequisites
-------------
The Workflow Engine requires mongodb, rabbitmq, and ipmitool to be installed as follows

    .. code::

        sudo apt-get install mongodb
        sudo apt-get install rabbitmq-server
        sudo apt-get install ipmitool

Set up Workflow Engine Service
------------------------------
1. Clone the on-taskgraph repository

    .. code::

        git clone https://github.com/RackHD/on-taskgraph
        cd on-taskgraph

2. Install the Workflow Engine dependencies

    .. code::

        npm install

3. Copy the Workflow Engine `config.json`_ file to the /opt/monorail directory.

.. _config.json: https://github.com/RackHD/RackHD/blob/master/packer%2Fansible%2Froles%2Fmonorail%2Ffiles%2Fconfig.json

4. Edit the config.json file, and change the value of the taskGraphEndpoint address to the IP address of your system.

5. Start the Workflow Engine Service by using this command:

    .. code::

        node index.js

6. Display the complete Workflow Engine API by pasting the following URL into a web browser:

    .. code::

        http://<your IP address>:9030/docs/

7. The Workflow Engine requires DHCP, TFTP, and static file servers. You can install the RackHD `on-dhcp-proxy`_, `on-tftp`_, and `on-http`_ services respectively, as explained in their associated README files.

.. _on-http: https://github.com/RackHD/on-http
.. _on-dhcp-proxy: https://github.com/RackHD/on-dhcp-proxy
.. _on-tftp: https://github.com/RackHD/on-tftp

8. Alternatively, you can use third party versions of DHCP and TFTP as described in `TFTP and DHCP Service Setup`_

.. _TFTP and DHCP Service Setup: http://rackhd.readthedocs.io/en/latest/rackhd/tftp_dhcp_server.html

9. You can also setup a third party static file server as described in `Static File Service Setup`_

.. _Static File Service Setup: http://rackhd.readthedocs.io/en/latest/rackhd/static_file_server.html

10. Configure your compute node to PXE boot, and reboot the node. The Workflow Engine should discover the node and catalog it in its database.

Posting a OS Install Workflow
-----------------------------

You will need to get the discovered node's identifier from the Workflow Engine's database as follows:

    .. code::

        mongo pxe
        db.nodes.find().pretty()
        ctrl-d

The output will look like:

    .. code::

        {
            "name" : "52:54:be:ef:c6:85",
            "identifiers" : [
                "52:54:be:ef:c6:85"
            ],
            "type" : "compute",
            "autoDiscover" : false,
            "relations" : [ ],
            "tags" : [ ],
            "createdAt" : ISODate("2017-11-06T21:42:11.406Z"),
            "updatedAt" : ISODate("2017-11-06T21:42:11.406Z"),
            "_id" : ObjectId("5a00d7336eb470a806c2b341")
        }

In this example, the node identifier is 5a00d7336eb470a806c2b341

Use the following command to run an OS installation workflow install using Workflow Engine,

    .. code::

        curl -X POST -d @payload.json http://<ip>:<port>/api/2.0/workflows --header "Content-Type: application/json"

where, payload.json is located in the current directory level, and payload looks like the example below.

    .. code::

        {
            "name": "Graph.InstallCoreOS",
            "options": {
                "defaults": {
                    "graphOptions": {
                        "target": "5a00d7336eb470a806c2b341"
                    }
                    "version": "899.17.0",
                    "repo": "http://172.31.128.1:9030/coreos"
                }
            }
        }

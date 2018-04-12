Quick Start Guide
========================

.. contents:: Table of Contents

Introduction
--------------

In this quick start guide you will learn how to use a docker based RackHD service. And use RackHD to install OS on Infrasim (Bare metal server simulator)

Install Docker & Docker Compose
--------------------------------

+----------------------+---------------------------------------------------------+
|Install Docker CE     | https://docs.docker.com/install/#server                 |
+----------------------+---------------------------------------------------------+
|Install Docker Compose| https://docs.docker.com/compose/install/#install-compose|
+----------------------+---------------------------------------------------------+

Download Source Code
-------------------------

.. code-block:: shell

    cd ~/src/RackHD/example/rackhd
    sudo docker-compose up –d

    # Check RackHD services are running
    sudo docker-compose ps

    #  Sample response:
    #
    #  Name                      Command                                    State                 Ports
    #  --------------------------------------------------------------------------------------------------------------
    #  rackhd_dhcp-proxy_1     node /RackHD/on-dhcp-proxy ...               Up
    #  rackhd_dhcp_1           /docker-entrypoint.sh                        Up
    #  rackhd_files_1          /docker-entrypoint.sh                        Up
    #  rackhd_http_1           node /RackHD/on-http/index.js                Up
    #  rackhd_mongo_1          docker-entrypoint.sh mongod                  Up      27017/tcp, 0.0.0.0:9090->9090/tcp
    #  rackhd_rabbitmq_1       docker-entrypoint.sh rabbi ...               Up
    #  rackhd_syslog_1         node /RackHD/on-syslog/ind ...               Up
    #  rackhd_taskgraph_1      node /RackHD/on-taskgraph/ ...               Up
    #  rackhd_tftp_1           node /RackHD/on-tftp/index.js                Up



Setup a Virtualized Infrastructure Environment
------------------------------------------------

.. code-block:: shell

    cd ~/src/RackHD/example/infrasim
    sudo docker-compose up –d

    # Sample response
    # 7b8944444da7 infrasim_infrasim ... 22/tcp, 80/tcp infrasim_infrasim_1

For example, we choose infrasim_infrasim0_1, use following command to retrieve its IP Address.

.. code-block:: shell

    sudo docker exec -it infrasim_infrasim_1 ifconfig br0

    # Sample response
    # br0 Link encap:Ethernet HWaddr 02:42:ac:1f:80:03
    #     inet addr:172.31.128.112 Bcast:172.31.143.255 Mask:255.255.240.0
    #     UP BROADCAST RUNNING MULTICAST MTU:1500 Metric:1
    #     RX packets:2280942 errors:0 dropped:0 overruns:0 frame:0
    #     TX packets:2263193 errors:0 dropped:0 overruns:0 carrier:0
    #     collisions:0 txqueuelen:0
    #     RX bytes:207752197 (207.7 MB) TX bytes:265129274 (265.1 MB)


.. note::

    If ``br0`` is not available, use ``sudo docker-compose restart`` to restart the vNodes.

Here ``172.31.128.112`` is infrasim_infrasim_1's BMC IP Address.

In order to connect to vNode from "UltraVNC Viewer" ``vnc_forward`` script should be executed.

.. code-block:: shell

    ./vnc_forward

    # Sample response
    # ...
    # Setting VNC port 28109 for IP 172.31.128.109
    # Setting VNC port 28110 for IP 172.31.128.110
    # Setting VNC port 28111 for IP 172.31.128.111
    # Setting VNC port 28112 for IP 172.31.128.112
    # Setting VNC port 28113 for IP 172.31.128.113
    # Setting VNC port 28114 for IP 172.31.128.114
    # ...

Get vNode's node-id

.. code-block:: shell

    curl localhost:9090/api/current/nodes?type=compute |  jq '.' | grep \"id\"

    # Example Response
    # "id": "5acf78e3291c0a010002a9a8",

Here ``5acf78e3291c0a010002a9a8`` is our target node-id

Ensure its OBM setting is not blank

.. code-block:: shell

    # replace the node-id with your own
    curl localhost:9090/api/current/nodes/<node-id>/obm | jq '.'

    # Example Response

    # [
    #   {
    #     "config": {
    #       "host": "02:42:ac:1f:80:03",
    #       "user": "__rackhd__"
    #     },
    #     "service": "ipmi-obm-service",
    #     "node": "/api/2.0/nodes/5acf78e3291c0a010002a9a8",
    #     "id": "5acf7973291c0a010002a9d2"
    #   }
    # ]

If the response comes back [], please follow :ref:`obm_setting`, to add OBM setting.


Setup OS Mirror
----------------------

To provision the OS to the node, RackHD can act as an OS mirror repository.

.. code-block:: shell

    cd ~/src/RackHD/example/rackhd/files/mount/common
    mkdir –p centos/7/os/x86_64/
    sudo mount –o loop ~/iso/CentOS-7-x86_64-DVD-1708.iso centos/7/os/x86_64

CentOS-7-x86_64-DVD-1708.iso can download from `Official site <https://wiki.centos.org/Download>`_.

``/files/mount/common`` is a volume which is mounted to ``rackhd/files`` docker container as a static file service.
After ISO file is mounted, we need to restart file service. (This is a docker’s potential bug which cannot sync files mounted in the volume when container is running)

.. code-block:: shell

    cd ~/src/RackHD/example/rackhd
    sudo docker-compose restart

The OS mirror will be available on http://172.31.128.2:9090/common/centos/7/os/x86_64 from vNode's perspective.


Install OS with RackHD API
-----------------------------

Download Centos OS install payload example (more example of other `OS <https://github.com/RackHD/RackHD/tree/master/example/samples>`_.)

.. code-block:: shell

    cd ~
    wget https://raw.githubusercontent.com/RackHD/RackHD/master/example/samples/install_centos_7_payload_minimal.json


Edit the payload json with vim.

.. code-block:: shell

    vim install_centos_7_payload_minimal.json

    # Change the "repo" line to below.
    "repo": "http://172.31.128.2:9090/common/centos/7/os/x86_64"

Install OS (using build-in InstallCentOS workflow)

.. code:: shell

    curl -X POST -H 'Content-Type: application/json' -d @install_centos_7_payload_minimal.json    localhost:9090/api/2.0/nodes/<nodeID>/workflows?name=Graph.InstallCentOS | jq .


Monitor Progress
------------------

Use UltraVNC on the desktop to view the OS installation, replace ``<your-ip>`` with your own, and ``<port>`` you retrieved using the ``vnc_forward`` script above

.. image:: ../_static/theme/img/vnc0.png
    :align: center

After login, you should see Centos7 is installing

.. image:: ../_static/theme/img/vnc2.png
    :width: 700px
    :align: center

It will PXE boot from the Centos OS install image and progress screen will show up in about 5 mins, the entire installation takes around 9 mins.
You can move on the guide or revisit previous sessions, then go back after 4~5 minutes



Login to OS
-------------

Once the OS has been installed, you can try login the system via UltraVNC console.
Installed OS default username/password: ``root/RackHDRocks!``

.. image:: ../_static/theme/img/login.png
    :align: center

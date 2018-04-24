Installation from Docker
================================

.. contents:: Table of Contents

Install Docker & Docker Compose
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

+----------------------+---------------------------------------------------------+
|Install Docker CE     | https://docs.docker.com/install/#server                 |
+----------------------+---------------------------------------------------------+
|Install Docker Compose| https://docs.docker.com/compose/install/#install-compose|
+----------------------+---------------------------------------------------------+

Download Source Code
~~~~~~~~~~~~~~~~~~~~~

.. code::

    git clone https://github.com/RackHD/RackHD

    cd RackHD/docker

    # for example if you are installing RackHD latest relesae:
    sudo TAG=latest docker-compose pull   # Download pre-built docker images
    sudo TAG=latest docker-compose up -d    # Create Containers and Run RackHD

For more information about tags please see https://hub.docker.com/r/rackhd/on-http/tags/

Check RackHD is running properly

.. code::

    cd RackHD/docker
    sudo docker-compose ps

    # example response
    #        Name                      Command               State    Ports
    # ---------------------------------------------------------------------
    # docker_core_1         /bin/echo exit                   Exit 0
    # docker_dhcp-proxy_1   node /RackHD/on-dhcp-proxy ...   Up
    # docker_dhcp_1         /docker-entrypoint.sh            Up
    # docker_files_1        /docker-entrypoint.sh            Up
    # docker_http_1         node /RackHD/on-http/index.js    Up
    # docker_mongo_1        docker-entrypoint.sh mongod      Up
    # docker_rabbitmq_1     docker-entrypoint.sh rabbi ...   Up
    # docker_syslog_1       node /RackHD/on-syslog/ind ...   Up
    # docker_taskgraph_1    node /RackHD/on-taskgraph/ ...   Up
    # docker_tasks_1        /bin/echo exit                   Exit 0
    # docker_tftp_1         node /RackHD/on-tftp/index.js    Up

######

How to Erase the Database to Restart Everything
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code::

    sudo docker exec -it docker_mongo_1 mongo rackhd
    db.dropDatabase()
    # CTRL+D to exit
    # Restart RackHD
    cd RackHD/docker
    sudo docker-compose restart

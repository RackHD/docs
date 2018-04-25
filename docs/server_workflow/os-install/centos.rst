CentOS Installation
=======================

.. important::
    DNS server is required in Centos installation, make sure you have put following lines in /etc/dhcp/dhcpd.conf. 172.31.128.1 is a default option in RackHD

    .. code::

        option domain-name-servers 172.31.128.1;
        option routers 172.31.128.254;


.. tabs::

    .. tab:: iso

        For **iso** installation, see this `payload json file <https://github.com/RackHD/RackHD/blob/master/example/samples/install_centos_7_payload_minimal.json>`_ Remember to replace ``{{ file.server }}`` with your own, see ``fileServerAddress`` and ``fileServerPort`` in ``/opt/monorail/config.json``

        .. code-block:: shell

            mkdir ~/iso && cd !/iso

            # Download iso file
            # You can choose a mirror in this site:
            # http://isoredirect.centos.org/centos/7/isos/x86_64/CentOS-7-x86_64-DVD-1708.iso
            # There are three type of ISOs (DVD ISO, Everything ISO, Minimal ISO), Minimal ISO is not supported

            wget http://mirror.math.princeton.edu/pub/centos/7/isos/x86_64/CentOS-7-x86_64-DVD-1708.iso

            # Create mirror folder
            mkdir -p /var/mirrors/centos

            # Replace {on-http-dir} with your own
            mkdir -p {on-http-dir}/static/http/mirrors

            # Mount iso
            sudo mount CentOS-7-x86_64-DVD-1708.iso /var/mirrors/centos

            # Replace {on-http-dir} with your own
            sudo ln -s /var/mirrors/centos {on-http-dir}/static/http/mirrors/

            # Create workflow
            # Replace the 9090 port if you are using other ports
            # You can configure the port in /opt/monorail/config.json -> 'httpEndPoints' -> 'northbound-api-router'
            curl -X POST -H 'Content-Type: application/json' -d @install_centos_payload_iso_minimal.json 127.0.0.1:9090/api/current/nodes/{node-id}/workflows?name=Graph.InstallCentos | jq '.'

    .. tab:: live

        Add following block into httpProxies in ``/opt/monorail/config.json``

        .. code::

            {
              "localPath": "/centos",
              "server": "http://mirror.centos.org/",
              "remotePath": "/centos/"
            },

        .. code::

            {
                "options": {
                    "defaults": {
                        "version": "7",
                        "repo": "http://172.31.128.1:9009/centos"
                    }
                }
            }

        Create workflow, replace the ``9090`` port if you are using other ports You can configure the port in ``/opt/monorail/config.json`` -> ``httpEndPoints`` -> ``northbound-api-router``

        .. code-block:: shell

            curl -x post -h 'content-type: application/json' -d @install_centos_payload_minimal.json 127.0.0.1:9090/api/current/nodes/{node-id}/workflows?name=graph.installcentos | jq '.'

    .. tab:: Sync mirror

        Current release versions can be found from  `<http://mirror.centos.org/centos/>`_

        .. code-block:: shell

            # Replace x with your own version

            sudo rsync --progress -av --delete --delete-excluded --exclude "local*" \
            --exclude "i386" rsync://centos.eecs.wsu.edu/x/ /var/mirrors/centos/x

.. note::

    For more detail about payload file please refer to :ref:`non-windows-payload`

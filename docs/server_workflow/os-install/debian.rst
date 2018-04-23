Debian Installation
=======================

.. important::
    DNS server is required in Debian installation, make sure you have put following lines in /etc/dhcp/dhcpd.conf. 172.31.128.1 is a default option in RackHD

    .. code::

        option domain-name-servers 172.31.128.1;
        option routers 172.31.128.254;


.. tabs::

    .. tab:: iso

        For **iso** installation, see this `payload json file <https://github.com/RackHD/RackHD/blob/master/example/samples/install_debian_payload_iso_minimal.json>`_ Remember to replace ``{{ file.server }}`` with your own, see ``fileServerAddress`` and ``fileServerPort`` in ``/opt/monorail/config.json``

        .. code-block:: shell

            mkdir ~/iso && cd !/iso

            # Download iso file
            wget https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/debian-9.4.0-amd64-xfce-CD-1.iso

            # Create mirror folder
            mkdir -p /var/mirrors/debian

            # Replace {on-http-dir} with your own
            mkdir -p {on-http-dir}/static/http/mirrors

            # Mount iso
            sudo mount debian-9.4.0-amd64-xfce-CD-1.iso /var/mirrors/debian

            # Replace {on-http-dir} with your own
            sudo ln -s /var/mirrors/debian {on-http-dir}/static/http/mirrors/

            # Create workflow
            # Replace the 9090 port if you are using other ports
            # You can configure the port in /opt/monorail/config.json -> 'httpEndPoints' -> 'northbound-api-router'
            curl -X POST -H 'Content-Type: application/json' -d @install_debian_payload_iso_minimal.json 127.0.0.1:9090/api/current/nodes/{node-id}/workflows?name=Graph.InstallDebian | jq '.'


    .. tab:: live

        For **live** installation, see this `payload json file <https://github.com/RackHD/RackHD/blob/master/example/samples/install_debian_payload_minimal.json>`_ Remember to replace ``repo`` with your own ``{fileServerAddress}:{fileServerPort}/debian``, you can find the proper parameters in ``/opt/monorail/config.json``

        Add following block into httpProxies in ``/opt/monorail/config.json``

        .. code-block:: json

            {
              "localPath": "/debian",
              "server": "http://ftp.us.debian.org/",
              "remotePath": "/debian/"
            }

        Create workflow, replace the ``9090`` port if you are using other ports You can configure the port in ``/opt/monorail/config.json`` -> ``httpEndPoints`` -> ``northbound-api-router``

        .. code-block:: shell

            curl -X POST -H 'Content-Type: application/json' -d @install_debian_payload_minimal.json 127.0.0.1:9090/api/current/nodes/{node-id}/workflows?name=Graph.InstallDebain | jq '.'

.. note::

    For more detail about payload file please refer to :ref:`non-windows-payload`

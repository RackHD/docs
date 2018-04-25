CoreOS Installation
=======================

.. tabs::

    .. tab:: iso

        For **iso** installation, see this `payload json file for iso <https://github.com/RackHD/RackHD/blob/master/example/samples/install_coreos_payload_minimum.json>`_ Remember to replace ``repo`` and ``version`` with your own, see ``fileServerAddress`` and ``fileServerPort`` in ``/opt/monorail/config.json``

        .. code-block:: shell

            mkdir ~/iso && cd !/iso

            # Download iso file
            wget https://stable.release.core-os.net/amd64-usr/current/coreos_production_iso_image.iso

            # Create mirror folder
            mkdir -p /var/mirrors/coreos

            # Replace {on-http-dir} with your own
            mkdir -p {on-http-dir}/static/http/mirrors

            # Mount iso
            sudo mount coreos_production_iso_image.iso /var/mirrors/coreos

            # Replace {on-http-dir} with your own
            sudo ln -s /var/mirrors/coreos {on-http-dir}/static/http/mirrors/

            # Create workflow
            # Replace the 9090 port if you are using other ports
            # You can configure the port in /opt/monorail/config.json -> 'httpEndPoints' -> 'northbound-api-router'
            curl -X POST -H 'Content-Type: application/json' -d @install_coreos_payload_minimal.json 127.0.0.1:9090/api/current/nodes/{node-id}/workflows?name=Graph.InstallCentOS | jq '.'

    note::

        For more detail about payload file please refer to :ref:`non-windows-payload`


Photon Installation
=======================


.. tabs::

    .. tab:: iso

        For **iso** installation, see this `payload json file <https://github.com/RackHD/RackHD/blob/master/example/samples/install_photon_os_payload_minimal.json>`_ Remember to replace ``{{ file.server }}`` with your own, see ``fileServerAddress`` and ``fileServerPort`` in ``/opt/monorail/config.json``

        .. code-block:: shell

            mkdir ~/iso && cd !/iso

            # Download iso file
            wget https://bintray.com/vmware/photon/download_file?file_path=photon-1.0-62c543d.iso

            # Create mirror folder
            mkdir -p /var/mirrors/photon

            # Replace {on-http-dir} with your own
            mkdir -p {on-http-dir}/static/http/mirrors

            # Mount iso
            sudo mount photon-1.0-62c543d.iso /var/mirrors/photon

            # Replace {on-http-dir} with your own
            sudo ln -s /var/mirrors/photon {on-http-dir}/static/http/mirrors/

            # Create workflow
            # Replace the 9090 port if you are using other ports
            # You can configure the port in /opt/monorail/config.json -> 'httpEndPoints' -> 'northbound-api-router'
            curl -X POST -H 'Content-Type: application/json' -d @install_photon_os_payload_minimal.json 127.0.0.1:9090/api/current/nodes/{node-id}/workflows?name=Graph.InstallPhotonOS | jq '.'


.. note::

    For more detail about payload file please refer to :ref:`non-windows-payload`

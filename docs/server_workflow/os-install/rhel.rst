RHEL Installation
=======================


.. tabs::

    .. tab:: Local ISO Mirror

        For **iso** installation, see this `payload json file <https://github.com/RackHD/RackHD/blob/master/example/samples/install_rhel_payload_minimal.json>`_ Remember to replace ``version`` and ``repo`` with your own, see ``fileServerAddress`` and ``fileServerPort`` in ``/opt/monorail/config.json``

        .. code-block:: shell

            mkdir ~/iso && cd !/iso

            # Download iso file from `<redhat.com>`_
            # Here we use rhel-server-7.0-x86_64-dvd.iso for example

            # Create mirror folder
            mkdir -p /var/mirrors/rhel

            # Replace {on-http-dir} with your own
            mkdir -p {on-http-dir}/static/http/mirrors

            # Mount iso
            sudo mount rhel-server-7.0-x86_64-dvd.iso /var/mirrors/rhel

            # Replace {on-http-dir} with your own
            sudo ln -s /var/mirrors/ubuntu {on-http-dir}/static/http/mirrors/

            # Create workflow
            # Replace the 9090 port if you are using other ports
            # You can configure the port in /opt/monorail/config.json -> 'httpEndPoints' -> 'northbound-api-router'
            curl -X POST -H 'Content-Type: application/json' -d @install_rhel_payload_minimal.json 127.0.0.1:9090/api/current/nodes/{node-id}/workflows?name=Graph.InstallRHEL | jq '.'

.. note::

    For more detail about payload file please refer to :ref:`non-windows-payload`

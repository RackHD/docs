ESXi Installation
=======================


.. tabs::

    .. tab:: iso

        For **iso** installation, see this `payload json file <https://github.com/RackHD/RackHD/blob/master/example/samples/install_esxi_payload_minimal.json>`_ Remember to replace ``{{ file.server }}`` with your own, see ``fileServerAddress`` and ``fileServerPort`` in ``/opt/monorail/config.json``

        .. code-block:: shell

            mkdir ~/iso && cd !/iso

            # Download iso file from https://my.vmware.com/web/vmware/info/slug/datacenter_cloud_infrastructure/vmware_vsphere_hypervisor_esxi/6_0

            # Create mirror folder
            mkdir -p /var/mirrors/esxi

            # Replace {on-http-dir} with your own
            mkdir -p {on-http-dir}/static/http/mirrors

            # Mount iso
            sudo mount VMware-VMvisor-Installer-201507001-2809209.x86_64.iso /var/mirrors/esxi

            # Replace {on-http-dir} with your own
            sudo ln -s /var/mirrors/esxi {on-http-dir}/static/http/mirrors/

            # Create workflow
            # Replace the 9090 port if you are using other ports
            # You can configure the port in /opt/monorail/config.json -> 'httpEndPoints' -> 'northbound-api-router'
            curl -X POST -H 'Content-Type: application/json' -d @install_esxi_payload_iso_minimal.json 127.0.0.1:9090/api/current/nodes/{node-id}/workflows?name=Graph.InstallESXi | jq '.'

.. note::

    For more detail about payload file please refer to :ref:`non-windows-payload`

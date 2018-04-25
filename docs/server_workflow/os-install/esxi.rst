ESXi Installation
=======================

A mirror should be setup firstly before installation. For ESXi, there is only one way to setup mirror currently.

* **Local ISO mirror**: Download ESXi ISO image, mount ISO image in a local server as the repository, http service for this repository is provided so that a node could access without proxy.


.. tabs::

    .. tab:: Local ISO Mirror

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


Call API to Install OS
----------------------

Get payload example:

.. code-block:: shell

    wget https://raw.githubusercontent.com/RackHD/RackHD/master/example/samples/install_esx_payload_minimal.json

Remember to replace ``version`` and ``repo`` with your own, see ``fileServerAddress`` and ``fileServerPort`` in ``/opt/monorail/config.json``

Create workflow, replace the ``9090`` port if you are using other ports You can configure the port in ``/opt/monorail/config.json`` -> ``httpEndPoints`` -> ``northbound-api-router``

.. code-block:: shell


    curl -X POST -H 'Content-Type: application/json' -d @install_esxi_payload_minimal.json 127.0.0.1:9090/api/current/nodes/{node-id}/workflows?name=Graph.InstallESXi | jq '.'


.. note::

    For more detail about payload file please refer to :ref:`non-windows-payload`

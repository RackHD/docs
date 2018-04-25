Photon Installation
=======================

A mirror should be setup firstly before installation. For PhotonOS, there is only one way to setup mirror currently.

* **Local ISO mirror**: Download PhotonOS ISO image, mount ISO image in a local server as the repository, http service for this repository is provided so that a node could access without proxy.

.. tabs::

    .. tab:: Local ISO Mirror

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


Call API to Install OS
----------------------

Get payload example:

.. code-block:: shell

    wget https://raw.githubusercontent.com/RackHD/RackHD/master/example/samples/install_photon_os_payload_minimal.json

Remember to replace ``version`` and ``repo`` with your own, see ``fileServerAddress`` and ``fileServerPort`` in ``/opt/monorail/config.json``

Create workflow, replace the ``9090`` port if you are using other ports You can configure the port in ``/opt/monorail/config.json`` -> ``httpEndPoints`` -> ``northbound-api-router``

.. code-block:: shell

    curl -X POST -H 'Content-Type: application/json' -d @install_photon_os_payload_minimal.json 127.0.0.1:9090/api/current/nodes/{node-id}/workflows?name=Graph.InstallPhotonOS | jq '.'


.. note::

    For more detail about payload file please refer to :ref:`non-windows-payload`

ESXi Installation
=======================

RackHD ESXi installation support multiple versions. Please refer to :ref:`os-installation-workflows-label` to see which versions are supported. We'll take ESXi 6.0 as the example below. If you want to install another version's ESXi, please replace with corresponding version's image, mirror, payload, etc.

Setup Mirror
------------

A mirror should be setup firstly before installation. For ESXi, there is only one way to setup mirror currently.

* **Local ISO mirror**: Download ESXi ISO image, mount ISO image in a local server as the repository, http service for this repository is provided so that a node could access without proxy.


.. include:: mirror-notes.rst

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

After the mirror is setup, We could download payload and call workflow API to install OS.

Get payload example.

.. code-block:: shell

    wget https://raw.githubusercontent.com/RackHD/RackHD/master/example/samples/install_esx_payload_minimal.json


Call OS installation workflow API to install OS. ``127.0.0.1:9090`` is according to the configuration ``address`` and ``port`` of ``httpEndPoints`` -> ``northbound-api-router`` in ``/opt/monorail/config.json``

.. code-block:: shell

    curl -X POST -H 'Content-Type: application/json' -d @install_esxi_payload_minimal.json 127.0.0.1:9090/api/current/nodes/{node-id}/workflows?name=Graph.InstallESXi | jq '.'


Please record the API's returned result, it's this workflow's Id (like ``342cce19-7385-43a0-b2ad-16afde072715``), it will be used to check result later.

.. include:: install-notes.rst

.. include:: check-result.rst


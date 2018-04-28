CoreOS Installation
=======================

RackHD CoreOS installation support multiple versions. Please refer to :ref:`os-installation-workflows-label` to see which versions are supported. We'll take CoreOS 899.17.0 as the example below. If you want to install another version's CoreOS, please replace with corresponding version's image, mirror, payload, etc.

Setup Mirror
------------

A mirror should be setup firstly before installation. For CoreOS, there is only one way to setup mirror currently.

* **Local ISO mirror**: Download CoreOS ISO image, mount ISO image in a local server as the repository, http service for this repository is provided so that a node could access without proxy.

.. include:: mirror-notes.rst

.. tabs::

    .. tab:: Local ISO Mirror

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

Call API to Install OS
----------------------

After the mirror is setup, We could download payload and call workflow API to install OS.

Get payload example.

.. code-block:: shell

    wget https://raw.githubusercontent.com/RackHD/RackHD/master/example/samples/install_coreos_payload_minimum.json

Call OS installation workflow API to install OS. ``127.0.0.1:9090`` is according to the configuration ``address`` and ``port`` of ``httpEndPoints`` -> ``northbound-api-router`` in ``/opt/monorail/config.json``
.. code-block:: shell

    curl -X POST -H 'Content-Type: application/json' -d @install_coreos_payload_minimal.json 127.0.0.1:9090/api/current/nodes/{node-id}/workflows?name=Graph.InstallCoreOS | jq '.'

Please record the API's returned result, it's this workflow's Id (like ``342cce19-7385-43a0-b2ad-16afde072715``), it will be used to check result later.

.. include:: install-notes.rst

.. include:: check-result.rst


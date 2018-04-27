Debian Installation
=======================

RackHD Debian installation support multiple versions. Please refer to :ref:`os-installation-workflows-label` to see which versions are supported. We'll take Debian Stretch as the example below. If you want to install another version's Debian, please replace with corresponding version's image, mirror, payload, etc.

.. important::
    DNS server is required in Debian installation, make sure you have put following lines in /etc/dhcp/dhcpd.conf. 172.31.128.1 is a default option in RackHD

    .. code::

        option domain-name-servers 172.31.128.1;
        option routers 172.31.128.254;

Setup Mirror
------------

A mirror should be setup firstly before installation. For Debian, there are two ways to setup mirror currently.

* **Local ISO mirror**: Download Debian ISO image, mount ISO image in a local server as the repository, http service for this repository is provided so that a node could access without proxy.
* **Public mirror**: The node could access a public or remote site's mirror repository with proxy.


.. include:: mirror-notes.rst


.. tabs::

    .. tab:: Local ISO Mirror

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


    .. tab:: Public Mirror

        Add following block into httpProxies in ``/opt/monorail/config.json``, and restart on-http service.

        .. code-block:: json

            {
              "localPath": "/debian",
              "server": "http://ftp.us.debian.org/",
              "remotePath": "/debian/"
            }

Call API to Install OS
----------------------

After the mirror is setup, We could download payload and call workflow API to install OS.

Get payload example.

.. code-block:: shell

    wget https://raw.githubusercontent.com/RackHD/RackHD/master/example/samples/install_debian_payload_minimal.json

Call OS installation workflow API to install OS. ``127.0.0.1:9090`` is according to the configuration ``address`` and ``port`` of ``httpEndPoints`` -> ``northbound-api-router`` in ``/opt/monorail/config.json``

.. code-block:: shell

    curl -X POST -H 'Content-Type: application/json' -d @install_debian_payload_minimal.json 127.0.0.1:9090/api/current/nodes/{node-id}/workflows?name=Graph.InstallDebain | jq '.'

Please record the API's returned result, it's this workflow's Id (like ``342cce19-7385-43a0-b2ad-16afde072715``), it will be used to check result later.

.. include:: install-notes.rst

.. include:: check-result.rst


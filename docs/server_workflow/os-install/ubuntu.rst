Ubuntu Installation
=======================

RackHD Ubuntu installation support multiple versions. Please refer to :ref:`os-installation-workflows-label` to see which versions are supported. We'll take Ubuntu Trusty(14.04) as the example below. If you want to install another version's Ubuntu, please replace with corresponding version's image, mirror, payload, etc.

.. important::
    DNS server is required in Ubuntu installation, make sure you have put following lines in /etc/dhcp/dhcpd.conf. 172.31.128.1 is a default option in RackHD

    .. code::

        option domain-name-servers 172.31.128.1;
        option routers 172.31.128.254;


Setup Mirror
------------

A mirror should be setup firstly before installation. For Ubuntu, there are three ways to setup mirror.

* **Local ISO mirror**: Download Ubuntu ISO image, mount ISO image in a local server as the repository, http service for this repository is provided so that a node could access without proxy.
* **Local sync mirror**: Sync public site's mirror repository to local, http service for this repository is provided so that a node could access without proxy.
* **Public mirror**: The node could access a public or remote site's mirror repository with proxy.

.. note::

    For local mirror (ISO or sync), RackHD on-http service internally has a default file service to provide file downloading for nodes. Its default root path is ``{on-http-dir}/static/http/mirrors/``. You also could use your own file service instead of the internal file service in the same server or another server, just notice that the file service's ip address ``fileServerAddress`` and the port ``fileServerPort`` in ``/opt/monorail/config.json`` should be configured. For more details, please refer to :ref:`static-file-server-label`. Remember to restart on-http service after modifying ``/opt/monorail/config.json``.

    For public mirror, RackHD on-http service also internally has a default http proxy for nodes to access remote file service. It could be configured by ``httpProxies`` in ``/opt/monorail/config.json``. For more details, please refer to :ref:`configuration`. Remember to restart on-http service after modifying ``/opt/monorail/config.json``.

.. tabs::

    .. tab:: Local ISO Mirror

        .. code-block:: shell

            mkdir ~/iso && cd !/iso

            # Download iso file
            wget http://releases.ubuntu.com/14.04/ubuntu-14.04.5-server-amd64.iso

            # Create mirror folder
            mkdir -p /var/mirrors/ubuntu

            # Replace {on-http-dir} with your own
            mkdir -p {on-http-dir}/static/http/mirrors

            # Mount iso
            sudo mount ubuntu-14.04.5-server-amd64.iso /var/mirrors/ubuntu

            # Replace {on-http-dir} with your own
            sudo ln -s /var/mirrors/ubuntu {on-http-dir}/static/http/mirrors/

    .. tab:: Local Sync Mirror

        For Ubuntu local mirror, The mirrors are easily made by syncing public Ubuntu mirror site, on any recent distribution of Ubuntu:

        .. code::

          # make the mirror directory (can sometimes hit a permissions issue)
          sudo mkdir -p /var/mirrors/ubuntu/14.04/mirror
          # create a file in /etc/apt/mirror.list (config below)
          sudo vi /etc/apt/mirror.list
          # run the mirror
          sudo apt-mirror


          ############# config ##################
          #
          set base_path    /var/mirrors/ubuntu/14.04
          #
          # set mirror_path  $base_path/mirror
          # set skel_path    $base_path/skel
          # set var_path     $base_path/var
          # set cleanscript $var_path/clean.sh
          # set defaultarch  <running host architecture>
          # set postmirror_script $var_path/postmirror.sh
          # set run_postmirror 0
          set nthreads     20
          set _tilde 0
          #
          ############# end config ##############

          deb-amd64 http://mirror.pnl.gov/ubuntu trusty main
          deb-amd64 http://mirror.pnl.gov/ubuntu trusty-updates main
          deb-amd64 http://mirror.pnl.gov/ubuntu trusty-security main
          clean http://mirror.pnl.gov/ubuntu

          #end of file
          ###################


    .. tab:: Public Mirror

        Add following block into httpProxies in ``/opt/monorail/config.json``, and restart on-http service.

        .. code::

            {
              "localPath": "/ubuntu",
              "server": "http://us.archive.ubuntu.com/",
              "remotePath": "/ubuntu/"
            }


Call API to Install OS
----------------------

After the mirror is setup, We could download payload and call workflow API to install OS. For Ubuntu OS installation, the payload format is different as below.

.. tabs::

    .. tab:: Local ISO Mirror

        Get Ubuntu Trusty(14.04) payload example for local ISO mirror.

        .. code-block:: shell

            wget https://raw.githubusercontent.com/RackHD/RackHD/master/example/samples/install_ubuntu_payload_iso_minimal.json

        Call OS installation workflow API to install OS. ``127.0.0.1:9090`` is according to the configuration ``address`` and ``port`` of ``httpEndPoints`` -> ``northbound-api-router`` in ``/opt/monorail/config.json``

        .. code-block:: shell

            curl -X POST -H 'Content-Type: application/json' -d @install_ubuntu_payload_iso_minimal.json 127.0.0.1:9090/api/current/nodes/{node-id}/workflows?name=Graph.InstallUbuntu | jq '.'


    .. tab:: Public and Local Sync Mirror

        For public and local sync mirror, they use the same payload format.

        Get Ubuntu Trusty(14.04) payload example.

        .. code-block:: shell

            wget https://raw.githubusercontent.com/RackHD/RackHD/master/example/samples/install_ubuntu_payload_minimal.json

        Call OS installation workflow API to install OS. ``127.0.0.1:9090`` is according to the configuration ``address`` and ``port`` of ``httpEndPoints`` -> ``northbound-api-router`` in ``/opt/monorail/config.json``

        .. code-block:: shell

            curl -X POST -H 'Content-Type: application/json' -d @install_ubuntu_payload_minimal.json 127.0.0.1:9090/api/current/nodes/{node-id}/workflows?name=Graph.InstallUbuntu | jq '.context.graphId'


Please record the API's returned result, it's this workflow's Id (like ``342cce19-7385-43a0-b2ad-16afde072715``), it will be used to check result later.

.. note::

    ``{{ file.server }}`` in payload will be replaced with ``fileServerAddress`` and ``fileServerPort`` in ``/opt/monorail/config.json`` by RackHD automatically while running. It also could be customized by ``{your-ip}:{your-port}`` for your own file service.

    For more details about payload file please refer to :ref:`non-windows-payload`

Check Result
------------
You could use following API to check if installation is succeded. ``342cce19-7385-43a0-b2ad-16afde072715`` is the returned workflow Id returned from install OS API above, please replace it with yours.

.. code-block:: shell

    curl -X GET 127.0.0.1:9090/api/current/nodes/{node-id}/workflows | jq '.[] | select(.context.graphId == "342cce19-7385-43a0-b2ad-16afde072715") | ._status'

If the result is ``running`` please wait until it's ``succeeded``.

You also could login the host console to see if installation succeed or not. By default, the `root` user will be created, and its password could be seen from ``rootPassword`` field from :ref:`non-windows-payload`


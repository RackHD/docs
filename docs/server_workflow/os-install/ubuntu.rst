Ubuntu Installation
=======================

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

Create workflow, replace the ``9090`` port if you are using other ports You can configure the port in ``/opt/monorail/config.json`` -> ``httpEndPoints`` -> ``northbound-api-router``

For Ubuntu OS installation, the payload format is different as below.

.. tabs::

    .. tab:: Local ISO Mirror

        Get payload example for local ISO mirror.

        .. code-block:: shell

            wget https://raw.githubusercontent.com/RackHD/RackHD/master/example/samples/install_ubuntu_payload_iso_minimal.json

        Remember to replace ``{{ file.server }}`` with your own, see ``fileServerAddress`` and ``fileServerPort`` in ``/opt/monorail/config.json``

        .. code-block:: shell

            curl -X POST -H 'Content-Type: application/json' -d @install_ubuntu_payload_iso_minimal.json 127.0.0.1:9090/api/current/nodes/{node-id}/workflows?name=Graph.InstallUbuntu | jq '.'


    .. tab:: Public and Local Sync Mirror

        For public and local sync mirror, they use the same payload format. Get payload example.

        .. code-block:: shell

            wget https://raw.githubusercontent.com/RackHD/RackHD/master/example/samples/install_ubuntu_payload_minimal.json

        Remember to replace ``repo`` with your own ``{fileServerAddress}:{fileServerPort}/ubuntu``


        .. code-block:: shell

            curl -X POST -H 'Content-Type: application/json' -d @install_ubuntu_payload_minimal.json 127.0.0.1:9090/api/current/nodes/{node-id}/workflows?name=Graph.InstallUbuntu | jq '.'


.. note::

    For more detail about payload file please refer to :ref:`non-windows-payload`


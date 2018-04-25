Ubuntu Installation
=======================

.. important::
    DNS server is required in Ubuntu installation, make sure you have put following lines in /etc/dhcp/dhcpd.conf. 172.31.128.1 is a default option in RackHD

    .. code::

        option domain-name-servers 172.31.128.1;
        option routers 172.31.128.254;


.. tabs::

    .. tab:: iso

        For **iso** installation, see this `payload json file for iso <https://github.com/RackHD/RackHD/blob/master/example/samples/install_ubuntu_payload_iso_minimal.json>`_ Remember to replace ``{{ file.server }}`` with your own, see ``fileServerAddress`` and ``fileServerPort`` in ``/opt/monorail/config.json``

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

            # Create workflow
            # Replace the 9090 port if you are using other ports
            # You can configure the port in /opt/monorail/config.json -> 'httpEndPoints' -> 'northbound-api-router'
            curl -X POST -H 'Content-Type: application/json' -d @install_ubuntu_payload_iso_minimal.json 127.0.0.1:9090/api/current/nodes/{node-id}/workflows?name=Graph.InstallUbuntu | jq '.'

    .. tab:: live

        For **live** installation, see this `payload json file for live <https://github.com/RackHD/RackHD/blob/master/example/samples/install_ubuntu_payload_minimal.json>`_ Remember to replace ``repo`` with your own ``{fileServerAddress}:{fileServerPort}/ubuntu``, you can find the proper parameters in ``/opt/monorail/config.json``

        Add following block into httpProxies in ``/opt/monorail/config.json``

        .. code-block:: json

            {
              "localPath": "/ubuntu",
              "server": "http://us.archive.ubuntu.com/",
              "remotePath": "/ubuntu/"
            }

        Create workflow, replace the ``9090`` port if you are using other ports You can configure the port in ``/opt/monorail/config.json`` -> ``httpEndPoints`` -> ``northbound-api-router``

        .. code-block:: shell

            curl -X POST -H 'Content-Type: application/json' -d @install_ubuntu_payload_minimal.json 127.0.0.1:9090/api/current/nodes/{node-id}/workflows?name=Graph.InstallUbuntu | jq '.'

    .. tab:: repo

        For the **Ubuntu repo**, you need some additional installation. The mirrors are easily made on Ubuntu, but not so easily replicated on other OS. On any recent distribution of Ubuntu:

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


.. note::

    For more detail about payload file please refer to :ref:`non-windows-payload`


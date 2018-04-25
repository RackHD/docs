OpenSuse Installation
=======================


.. tabs::

    .. tab:: iso

        For **iso** installation, see this `payload json file <https://github.com/RackHD/RackHD/blob/master/example/samples/install_suse_payload_iso_minimal.json>`_ Remember to replace ``{{ file.server }}`` with your own, see ``fileServerAddress`` and ``fileServerPort`` in ``/opt/monorail/config.json``

        .. code-block:: shell

            mkdir ~/iso && cd !/iso

            # Download iso file
            http://mirror.clarkson.edu/opensuse/distribution/openSUSE-current/iso/openSUSE-Leap-42.3-DVD-x86_64.iso

            # Create mirror folder
            mkdir -p /var/mirrors/suse

            # Replace {on-http-dir} with your own
            mkdir -p {on-http-dir}/static/http/mirrors

            # Mount iso
            sudo mount openSUSE-Leap-42.3-DVD-x86_64.iso /var/mirrors/suse

            # Replace {on-http-dir} with your own
            sudo ln -s /var/mirrors/suse {on-http-dir}/static/http/mirrors/

            # Create workflow
            # Replace the 9090 port if you are using other ports
            # You can configure the port in /opt/monorail/config.json -> 'httpEndPoints' -> 'northbound-api-router'
            curl -X POST -H 'Content-Type: application/json' -d @install_suse_payload_iso_minimal.json 127.0.0.1:9090/api/current/nodes/{node-id}/workflows?name=Graph.InstallSUSE | jq '.'

    .. tab:: live

        Add following block into httpProxies in ``/opt/monorail/config.json``

        .. code-block:: json

            {
              "localPath": "/suse",
              "server": "http://mirror.clarkson.edu/",
              "remotePath": "/opensuse/distribution/leap/42.3/repo/"
            }

        Create `install_suse_live_minimal.json``

        .. code-block:: json

            {
                "options": {
                    "defaults": {
                        "version": "oss",
                        "repo": "http://172.31.128.1:9090/suse"
                    }
                }
            }

        Create workflow, replace the ``9090`` port if you are using other ports You can configure the port in ``/opt/monorail/config.json`` -> ``httpEndPoints`` -> ``northbound-api-router``

        .. code-block:: shell

            curl -X POST -H 'Content-Type: application/json' -d @install_suse_live_minimal.json 127.0.0.1:9090/api/current/nodes/{node-id}/workflows?name=Graph.InstallSUSE | jq '.'

    .. tab:: Sync mirror

        Current release versions can be found from `<http://mirror.clarkson.edu/opensuse/distribution/leap/>`_

        .. code-block:: shell

            # Replace xx.x with your own version

            sudo rsync --progress -av --delete --delete-excluded --exclude "local*" --exclude "i386" --exclude "i586" --exclude "i686" rsync://mirror.clarkson.edu/opensuse/distribution/leap/xx.x/repo/oss/ /var/mirrors/suse/distribution/xx.x

            sudo rsync --progress -av --delete --delete-excluded --exclude "local*" --exclude "i386" --exclude "i586" --exclude "i686" rsync://mirror.clarkson.edu/opensuse/update/leap/xx.x /var/mirrors/suse/update/leap/xx.x

            sudo rsync --progress -av --delete --delete-excluded --exclude "local*" --exclude "i386" --exclude "i586" --exclude "i686" rsync://mirror.clarkson.edu/opensuse/update/leap/xx.x /var/mirrors/suse/update/leap/xx.x


.. note::

    For more detail about payload file please refer to :ref:`non-windows-payload`

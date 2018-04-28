.. note::

    For local mirror (ISO or sync), RackHD on-http service internally has a default file service to provide file downloading for nodes. Its default root path is ``{on-http-dir}/static/http/mirrors/``. You also could use your own file service instead of the internal file service in the same server or another server, just notice that the file service's ip address ``fileServerAddress`` and the port ``fileServerPort`` in ``/opt/monorail/config.json`` should be configured. For more details, please refer to :ref:`static-file-server-label`. Remember to restart on-http service after modifying ``/opt/monorail/config.json``.

    For public mirror, RackHD on-http service also internally has a default http proxy for nodes to access remote file service. It could be configured by ``httpProxies`` in ``/opt/monorail/config.json``. For more details, please refer to :ref:`configuration`. Remember to restart on-http service after modifying ``/opt/monorail/config.json``.

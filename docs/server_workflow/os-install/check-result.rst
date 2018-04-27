Check Result
------------
You could use following API to check if installation is succeded. ``342cce19-7385-43a0-b2ad-16afde072715`` is the returned workflow Id returned from install OS API above, please replace it with yours.

.. code-block:: shell

    curl -X GET 127.0.0.1:9090/api/current/nodes/{node-id}/workflows | jq '.[] | select(.context.graphId == "342cce19-7385-43a0-b2ad-16afde072715") | ._status'

If the result is ``running`` please wait until it's ``succeeded``.

You also could login the host console to see if installation succeed or not. By default, the `root` user will be created, and its password could be seen from ``rootPassword`` field from :ref:`non-windows-payload`


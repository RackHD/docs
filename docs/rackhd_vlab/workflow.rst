Control Server Nodes through Workflow
==========================================

Create a poller to check power status continuously
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Following api is RackHD's built in poller workflow, it can retrieve server's power status(on/off) every 10 seconds.

.. code-block:: shell

    curl -X POST \
         -H 'Content-Type: application/json' \
         -d '{"type":"ipmi","pollInterval":10000,"node":"<Node-Id>",
             "config":{"command":"power"}}' \
         <server>/api/current/pollers


Cancel the poller
~~~~~~~~~~~~~~~~~~~~~~

Restart server
~~~~~~~~~~~~~~~~~~~~





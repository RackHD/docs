Control Server Nodes through Workflow
==========================================

Show the name of all built-in workflows

.. code-block:: shell

    curl localhost:9090/api/2.0/workflows/graphs | jq '.' | grep injectableName | grep "Graph.*" | grep -v "Task"

    # Example Response
    # ...
    # "injectableName": "Graph.InstallUbuntu",
    # "injectableName": "Graph.InstallWindowsServer",
    # "injectableName": "Graph.Catalog.Intel.Flashupdt",
    # "injectableName": "Graph.McReset",
    # "injectableName": "Graph.noop-example",
    # "injectableName": "Graph.PDU.Discovery",
    # "injectableName": "Graph.Persist.Poller.Data",
    # "injectableName": "Graph.Service.Poller",
    # "injectableName": "Graph.PowerOff.Node",
    # "injectableName": "Graph.PowerOn.Node",
    # "injectableName": "Graph.Quanta.storcli.Catalog",
    # "injectableName": "Graph.rancherDiscovery",
    # "injectableName": "Graph.Reboot.Node",
    # "injectableName": "Graph.Redfish.Discovery",
    # "injectableName": "Graph.Redfish.Ip.Range.Discovery",
    # ...


Let's try to reboot the server node use ``Graph.Reboot.Node`` workflow.

Before you post the ``reboot`` workflow, use VNC-Viewer to connect to server node first.

.. code-block:: shell

    curl -X POST \
        -H 'Content-Type: application/json' \
        127.0.0.1:9090/api/current/nodes/<Node-ID>/workflows?name=Graph.Reboot.Node | jq '.'

Then you will see your server node's restart process in VNC-Viewer.




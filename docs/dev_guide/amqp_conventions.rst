AMQP Message Bus Conventions
=============================

.. contents:: Table of Contents

At the top level, we utilize 9 exchanges for passing various messages between key services and processes:

Configuration
-----------------------------

RPC channel for making dynamic system configuration changes

Routing keys:

.. code-block:: bash

     methods.set
     methods.get

Events
-----------------------------

One to many broadcast of events applicable to workflows and reactions
(where poller/telemetry events will be placed in the future as well)

Routing keys:

.. code-block:: bash

     tftp.success.[nodeid]
     tftp.failure.[nodeid]
     http.response.[nodeid]
     dhcp.bind.success.[nodeid]
     task.finished.[taskid]
     graph.started.[graphid]
     graph.finished.[graphid]
     sku.assigned.[nodeid]

HTTP
-----------------------------

Routing keys:

.. code-block:: bash

     http.response

(uncertain - duplicate of `http.response.[nodeid]`?)

DHCP
-----------------------------

RPC channel for interrogating the DHCP service

Routing keys:

.. code-block:: bash

       methods.lookupIpLease
       methods.ipInRange
       methods.peekLeaseTable
       methods.removeLease
       methods.removeLeaseByIp
       methods.pinMac
       methods.unpinMac
       methods.pinIp
       methods.unpinIp

TFTP
-----------------------------

(nothing defined)

Logging
-----------------------------

Routing keys:

.. code-block:: bash

    emerg
    alert
    error
    warning
    notice
    info
    debug
    silly

`task-graph-runner`
-----------------------------

RPC mechanism for communicating with process running workflows

Routing keys:

.. code-block:: bash

    methods.getTaskGraphLibrary
    methods.getTaskLibrary
    methods.getActiveTaskGraph
    methods.getActiveTaskGraphs
    methods.defineTaskGraph
    methods.defineTask
    methods.runTaskGraph
    methods.cancelTaskGraph
    methods.pauseTaskGraph
    methods.resumeTaskGraph
    methods.getTaskGraphProperties

Scheduler
-----------------------------

RPC mechanism for scheduling tasks within a workflow to run

.. code-block:: bash

    schedule

Task
-----------------------------

RPC mechanism for tasks to interrogate or interact with workflows (task-graphs)

.. code-block:: bash

    run.[taskid]
    cancel.[taskid]
    methods.requestProfile.[id] (right now, nodeId)
    methods.requestProperties.[id] (right now, nodeId)
    methods.requestCommands.[id] (right now, nodeId)
    methods.respondCommands.[id] (right now, nodeId)
    methods.getBootProfile.[nodeid]
    methods.activeTaskExists.[nodeId]
    methods.requestPollerCache
    ipmi.command.[command].[graphid] (right now, command is 'power', 'sel' or 'sdr')
    ipmi.command.[command].result.[graphid] (right now, command is 'power', 'sel' or 'sdr')
    run.snmp.command.[graphid]
    snmp.command.result.[graphid]
    poller.alert.[graphid]
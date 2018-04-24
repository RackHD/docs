Southbound Notification API
=============================

.. contents:: Table of Contents

The southbound notification API provides functionality for sending notifications to RackHD
from a node. For example, a node could send notification to inform RackHD that
OS installation has finished.

The notification API is only available from the southbound.

How does it work
-----------------------------

When a node calls a notification API, the RackHD on-http process will get acknowledged
and then send a AMQP message to an exchange named 'on.events', with routing key set to
'notification' or 'notification.<id>' depending on the parameters sent along when
calling the notification API.

Any task running in on-taskgraph process that is expecting a notification will need
to subscribe the AMQP message.

For example, the install-os task will subscribe the 'on.events' AMQP message with routing key 
'notification.<id>'. A node will call the notification API at the end of the OS installation
thus on-http will publish a AMQP message accordingly. The install-os task will then receive
the message and finish itself. Please refer to the diagram below.

.. image:: /_static/install_os_notification.png

API commands
-----------------------------

When running the on-http process, these are some common API commands you
can send:

**Send notification targeting a node**

.. code-block:: REST

    POST /api/current/notification?nodeId=<id>

.. code-block:: REST

    curl -X POST -H "Content-Type:application/json" \
    <server>/api/current/notification?nodeId=5542b78c130198aa216da3ac

It will also work if the nodeId parameter is set in the request body.

.. code-block:: REST

    curl -X POST -H "Content-Type:application/json" <server>/api/current/notification \
     -d '{"nodeId": "5542b78c130198aa216da3ac"}'

Additional parameters can be sent as well, as long as the receiver task knows how to
use those parameters.

.. code-block:: REST

    curl -X POST -H "Content-Type:application/json"  \
    <server>/api/current/notification?nodeId=5542b78c130198aa216da3ac \
    &progress=50%status=inprogress

**Send a broadcast notification**

A broadcast notification will trigger a AMQP message with routing key set to
'notification', without the tailing '.<id>'.

.. code-block:: REST

    POST /api/current/notification

.. code-block:: REST

    curl -X POST -H "Content-Type:application/json" <server>/api/current/notification


Use notification API in OS installation
---------------------------------------

A typical OS installation needs two notifications. The first one notifies that OS has been installed
to the disk on the target node. The second one notifies that the OS has been successfully booted
on the target node.

The first notificatioin is typically sent in the 'postinstall' section of the kickstart file. 
For example:
https://github.com/RackHD/on-http/blob/master/data/templates/install-photon/photon-os-ks#L76

the second notification is typically sent in the RackHD callback script. For example:
https://github.com/RackHD/on-http/blob/master/data/templates/install-photon/photon-os.rackhdcallback#L38

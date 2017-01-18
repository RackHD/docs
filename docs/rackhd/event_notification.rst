Event Notification
------------------

The change of resources managed by RackHD could be retrieved from AMQP messages.

- Exchange: **on.events**
- Routing Key **event.<type>**

`<type>` is event's type, it's the same with event's payload `type`, see payloads_ for more details.

An example of listening node events using tool 'sniff.js' located at https://github.com/RackHD/on-tools/blob/master/dev_tools/README.md.

.. code-block:: Bash

    $ sudo node sniff.js "on.events" "event.#"

.. _payloads:

`type` and `action` fields exist for all types of events.

========= ====== =================================
Attribute Type   Description
========= ====== =================================
type      String The event type, it could be `node`, `sku`, etc.
action    String The `action` that was performed, it's related to event `type`
========= ====== =================================


Node Events
~~~~~~~~~~~~~~~~~~~~

An example of a minimal node event message format is below:

.. code-block:: JSON

    {
        type: 'node',
        action: 'discovered'
        nodeId: '577d758e5bee93f307b9c062',
        nodeType: 'compute'
    }

The payload attributes of node event:

========= ====== =================================
Attribute Type   Description
========= ====== =================================
type      String `type` is `'node'` for node events
action    String The `action` that was performed by node event, See actions_ for more details.
nodeId    String The node's unique identifier
nodeType  String The node type could be `compute`, `pdu`, `switch`, `mgmt`, `enclosure`, it's related to `action`
uri       String it's **optional**
========= ====== =================================

.. _actions:

All node event actions:

+-----------------+-----------+----------------------------------+--------------------------------+
| Action          | Frequency | Cases                            | When Triggered                 |
|                 |           |                                  |                                |
+=================+===========+==================================+================================+
| discovered      | one-shot  | Event occurs in node's           | Triggered when node is         |
|                 |           | discovery process,it has         | created and all catalog        |
|                 |           | two cases:                       | jobs are finished              |
|                 |           |                                  |                                |
|                 |           | - Automatic discovery            |                                |
|                 |           | - Passive discovery by           |                                |
|                 |           |   post a node by REST API        |                                |
+-----------------+-----------+----------------------------------+--------------------------------+
| removed         | one-shot  | Event occurs when node is        | Triggered when node's all      |
|                 |           | deleted by REST API              | information are deleted,       |
|                 |           |                                  | it includes node,catalogs,     |
|                 |           |                                  | relationships with other       |
|                 |           |                                  | models, etc.                   |
+-----------------+-----------+----------------------------------+--------------------------------+
| sku.assigned    | one-shot  | Event occurs when node's `sku`   | Triggered when sku is created  |
|                 |           | field is assigned.               | and associated with the node   |
+-----------------+-----------+----------------------------------+--------------------------------+
| sku.unassigned  | one-shot  | Event occurs when node's `sku`   | Triggered when sku is          |
|                 |           | field is unassigned.             | not associated with the node   |
+-----------------+-----------+----------------------------------+--------------------------------+
| sku.updated     | one-shot  | Event occurs when node's `sku`   | Triggered when node's sku is   |
|                 |           | field is updated.                | associated with another sku    |
+-----------------+-----------+----------------------------------+--------------------------------+
| obms.assigned   | one-shot  | Event occurs when node's `obms`  | Triggered when obm is created  |
|                 |           | field is assigned.               | and associated with the node   |
+-----------------+-----------+----------------------------------+--------------------------------+
| obms.unassigned | one-shot  | Event occurs when node's `obms`  | Triggered when obm is          |
|                 |           | field is unassigned.             | not associated with the node   |
+-----------------+-----------+----------------------------------+--------------------------------+
| obms.updated    | one-shot  | Event occurs when node's `obms`  | Triggered when node don't have |
|                 |           | field is updated.                | obm settings                   |
+-----------------+-----------+----------------------------------+--------------------------------+
| accessible      | one-shot  | Event occurs when node telemetry | Triggered when any poller of a |
|                 |           | OBM service (IPMI or SNMP) is    | node become accessible while   |
|                 |           | accessible                       | the node's state is            |
|                 |           |                                  | inaccessilbe or null           |
+-----------------+-----------+----------------------------------+--------------------------------+
| inaccessible    | one-shot  | Event occurs when node telemetry | Triggered when all pollers     |
|                 |           | OBM service (IPMI or SNMP) is    | of a node become inaccessible. |
|                 |           | inaccessible                     |                                |
+-----------------+-----------+----------------------------------+--------------------------------+
| added           | one-shot  | Event occurs when a rack node is | Triggered after a rack node is |
|                 |           | added via the API.               | written to the database        |
+-----------------+-----------+----------------------------------+--------------------------------+                                

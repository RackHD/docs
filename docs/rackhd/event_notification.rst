Event Notification
------------------

RackHD supports event notification via both web hook and AMQP.

A web hook allows applications to subscribe certain RackHD published events by configured URL, when one of the subscribed events is triggered, RackHD will send a POST request with event payload to configured URL.

RackHD also publishes defined events over AMQP, so subscribers to RackHD's instance of AMQP don't need to register a webhook URL to get events. The AMQP events can be prolific, so we recommend that consumers filter events as they are received to what is desired.

Events Payloads
~~~~~~~~~~~~~~~~~~~~

.. _event_payload:

All published external events' payload formats are common, the event attributes are as below:

========= ====== =================================
Attribute Type   Description
========= ====== =================================
version   String Event payload format version.
type      String It could be one of the values: **heartbeat**, **node**, **polleralert**, **graph**.
action    String a verb or a composition of component and verb which indicates what happened, it's associated with the `type` attribute.
severity  String Event severity, it could be one of the values: **critical**, **warning**, **information**.
typeId    String It's associated with the `type` attribute. It could be graph 'Id' for **graph** type, poller 'Id' for **polleralert** type, <fqdn>.<service name> for **heartbeat** event, node 'Id' for **node** type. Please see table_ for more details .
createdAt String The time event happened.
nodeId    String The node `Id`, it's `null` for 'heartbeat' event.
data      Object Detail information are included in this attribute.
========= ====== =================================

.. _table:

The table of `type`, `typeId`, `action` and `severity` for all external events

+--------------+------------------------+------------------------+----------------+-----------------------------------+
| *type*       | *typeId*               | *action*               | *severity*     | Description                       |
|              |                        |                        |                |                                   |
+==============+========================+========================+================+===================================+
| heartbeat    | <fqdn>.<service name>  | updated                | information    | Each running RackHD service will  |
|              |                        |                        |                | publish a periodic heartbeat      |
|              |                        |                        |                | event message to notify that      |
|              |                        |                        |                | service is running.               |
+--------------+------------------------+------------------------+----------------+-----------------------------------+
| polleralert  | the 'Id' of poller     | sel.updated            | related to sel | Triggered when condition rules    |
|              |                        |                        | rules, it      | of sel alert defined in SKU PACK  |
|              |                        |                        | could be one   | is matched                        |
|              |                        |                        | of the values: |                                   |
|              |                        |                        | critical,      |                                   |
|              |                        |                        | warning,       |                                   |
|              |                        |                        | information    |                                   |
|              |                        +------------------------+----------------+-----------------------------------+
|              |                        | sdr.updated            | information    | Triggered when sdr information    |
|              |                        |                        |                | is updated.                       |
|              |                        +------------------------+----------------+-----------------------------------+
|              |                        | fabricservice.updated  | information    | Triggered when fabricservice      |
|              |                        |                        |                | information is updated.           |
|              |                        +------------------------+----------------+-----------------------------------+
|              |                        | pdupower.updated       | information    | Triggered when pdu power state    |
|              |                        |                        |                | information is changed.           |
|              |                        +------------------------+----------------+-----------------------------------+
|              |                        | chassispower.updated   | information    | Triggered when chassis power      |
|              |                        |                        |                | state information is changed.     |
|              |                        +------------------------+----------------+-----------------------------------+
|              |                        | snmp.updated           | related to     | Triggered when condition rules    |
|              |                        |                        | snmp rules, it | of snmp alert defined in SKU PACK |
|              |                        |                        | could be one   | is matched                        |
|              |                        |                        | of the values: |                                   |
|              |                        |                        | critical,      |                                   |
|              |                        |                        | warning,       |                                   |
|              |                        |                        | information    |                                   |
+--------------+------------------------+------------------------+----------------+-----------------------------------+
| graph        | the 'Id' of graph      | started                | information    | Triggered when graph started.     |
|              |                        +------------------------+----------------+-----------------------------------+
|              |                        | finished               | information    | Triggered when graph finished.    |
|              |                        +------------------------+----------------+-----------------------------------+
|              |                        | progress.updated       | information    | Triggered when long task's        |
|              |                        |                        |                | progress information is updated.  |
+--------------+------------------------+------------------------+----------------+-----------------------------------+
| node         | the 'Id' of node       | discovered             | information    | Triggered in node's               |
|              |                        |                        |                | discovery process,it has          |
|              |                        |                        |                | two cases:                        |
|              |                        |                        |                |                                   |
|              |                        |                        |                | - Automatic discovery             |
|              |                        |                        |                | - Passive discovery by            |
|              |                        |                        |                |   post a node by REST API         |
|              |                        +------------------------+----------------+-----------------------------------+
|              |                        | added                  | information    | Triggered when a rack node is     |
|              |                        |                        |                | added to database by REST API     |
|              |                        +------------------------+----------------+-----------------------------------+
|              |                        | removed                | information    | Triggered when node is            |
|              |                        |                        |                | deleted by REST API               |
|              |                        +------------------------+----------------+-----------------------------------+
|              |                        | sku.assigned           | information    | Triggered when node's `sku`       |
|              |                        |                        |                | field is assigned.                |
|              |                        +------------------------+----------------+-----------------------------------+
|              |                        | sku.unassigned         | information    | Triggered when node's `sku`       |
|              |                        |                        |                | field is unassigned.              |
|              |                        +------------------------+----------------+-----------------------------------+
|              |                        | sku.updated            | information    | Triggered when node's `sku`       |
|              |                        |                        |                | field is updated.                 |
|              |                        +------------------------+----------------+-----------------------------------+
|              |                        | obms.assigned          | information    | Triggered when node's `obms`      |
|              |                        |                        |                | field is assigned.                |
|              |                        +------------------------+----------------+-----------------------------------+
|              |                        | obms.unassigned        | information    | Triggered when node's `obms`      |
|              |                        |                        |                | field is unassigned.              |
|              |                        +------------------------+----------------+-----------------------------------+
|              |                        | obms.updated           | information    | Triggered when node's `obms`      |
|              |                        |                        |                | field is updated.                 |
|              |                        +------------------------+----------------+-----------------------------------+
|              |                        | accessible             | information    | Triggered when node telemetry     |
|              |                        |                        |                | OBM service (IPMI or SNMP) is     |
|              |                        |                        |                | accessible                        |
|              |                        |                        |                |                                   |
|              |                        +------------------------+----------------+-----------------------------------+
|              |                        | inaccessible           | information    | Triggered when node telemetry     |
|              |                        |                        |                | OBM service (IPMI or SNMP) is     |
|              |                        |                        |                | inaccessible                      |
+--------------+------------------------+------------------------+----------------+-----------------------------------+


Example of heartbeat event payload:

.. code-block:: JSON

    {
        "version": "1.0",
        "type": "heartbeat",
        "action": "updated",
        "typeId": "kickseed.example.com.on-taskgraph",
        "severity": "information",
        "createdAt": "2016-07-13T14:23:45.627Z",
        "nodeId": "null",
        "data": {
            name: 'on-taskgraph',
            title: 'node',
            pid: 6086,
            uid: 0,
            platform: 'linux',
            release: {
                name: 'node',
                lts: 'Argon',
                sourceUrl: 'https://nodejs.org/download/release/v4.7.2/node-v4.7.2.tar.gz',
                headersUrl: 'https://nodejs.org/download/release/v4.7.2/node-v4.7.2-headers.tar.gz'
            },
            versions: {
                http_parser: '2.7.0',
                node: '4.7.2',
                v8: '4.5.103.43',
                uv: '1.9.1',
                zlib: '1.2.8',
                ares: '1.10.1-DEV',
                icu: '56.1',
                modules: '46',
                openssl: '1.0.2j'
            },
            memoryUsage: {
                rss: 116531200,
                heapTotal: 84715104,
                heapUsed: 81638904
            },
            currentTime: '2017-01-24T07:18:49.236Z',
            nextUpdate: '2017-01-24T07:18:59.236Z',
            lastUpdate: '2017-01-24T07:18:39.236Z',
            cpuUsage: 'NA'
        }
    }

Events via AMQP
~~~~~~~~~~~~~~~~~~~~

AMQP Exchange and Routing Key
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The change of resources managed by RackHD could be retrieved from AMQP messages.

- Exchange: **on.events**
- Routing Key **<type>.<action>.<severity>.<typeId>.<nodeId>**

ALl the fields in routing key exists in the common event payloads event_payload_.

Examples of routing key:

Heartbeat event routing key of on-tftp service:

.. code-block:: REST

    heartbeat.updated.information.kickseed.example.com.on-tftp

Polleralert sel event routing key:

.. code-block:: REST

    polleralert.sel.updated.critical.44b15c51450be454180fabc.57b15c51450be454180fa460

Node discovered event routing key:

.. code-block:: REST

    node.discovered.information.57b15c51450be454180fa460.57b15c51450be454180fa460

Graph event routing key:

.. code-block:: REST

    graph.started.information.35b15c51450be454180fabd.57b15c51450be454180fa460


AMQP Routing Key Filter
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

All the events could be filtered by routing keys, for example:

All services' heartbeat events:

.. code-block:: Bash

    $ sudo node sniff.js "on.events" "heartbeat.#"

All nodes' discovered events:

.. code-block:: Bash

    $ sudo node sniff.js "on.events" "#.discovered.#"

'sniff.js' is a tool located at https://github.com/RackHD/on-tools/blob/master/dev_tools/README.md


Events via Hook
~~~~~~~~~~~~~~~~

Register Web Hooks
^^^^^^^^^^^^^^^^^^^

The web hooks used for subscribing event notification could be registered by ``POST <server>/api/2.0/hooks`` API as below

.. code-block:: REST

    curl -H "Content-Type: application/json" -X POST -d @payload.json <server>/api/2.0/hooks

.. _hook_payload:

The `payload.json` attributes in the example above are as below:

========= ====== ============ ============================================
Attribute Type   Flags        Description
========= ====== ============ ============================================
url       String **required** The hook url that events are notified to. Both http and https urls are supported. url must be unique.
name      String **optional** Any name user specified for the hook.
filters   Array  **optional** An array of conditions that decides which events should be notified to hook url.
========= ====== ============ ============================================

When a hook is registered and eligible events happened, RackHD will send a ``POST request`` to the hook url. POST request's ``Content-Type`` will be ``application/json``, and the request body be the event payload.

An example of `payload.json` with minimal attributes:

.. code-block:: JSON

    {
        "url": "http://www.abc.com/def"
    }

When multiple hooks are registered, a single event can be sent to multiple hook urls if it meets hooks' filtering conditions.

Event Filter Rules
^^^^^^^^^^^^^^^^^^^

The conditions of which events should be notified could be specified in the `filters` attribute in the hook_payload_, when `filters` attribute is not specified, or it's empty, all the events will be notified to the hook url.

The `filters` attribute is an array, so multiple filters could be specified. The event will be sent as long as any filter condition is satisfied, even if the conditions may have overlaps.

The filter attributes are `type`, `typeId`, `action`, `severity` and `nodeId` listed in event_payload_. Filtering by `data` is not supported currently. Filtering expression of hook `filters` is based on javascript regular expression, below table describes some base operations for hook filters:

=============================================== ======================================================= ============================
Description                                     Example                                                 Eligible Events
=============================================== ======================================================= ============================
Attribute equals some value                     {"action": "^discovered$"}                              Events with `action` equals `discovered`
Attribute can be any of specified value.        {"action": "discovered|updated"}                        Events with `action` equals either `discovered` or `updated`
Attribute can not be any of specified value.    {"action": "[^(discovered|updated)]"}                   Events with `action` equals neither `discovered` nor `updated`
Multiple attributes must meet specified values. {"action": "[^(discovered|updated)]", "type": "node"}   Events with `type` equals `node`
                                                                                                        while `action` equals neither `discovered` nor `updated`
=============================================== ======================================================= ============================

An example of multiple filters:

.. code-block:: JSON

    {
        "name": "event sets",
        "url": "http://www.abc.com/def",
        "filters": [
            {
                "type": "node",
                "nodeId": "57b15c51450be454180fa460"
            },
            {
                "type": "node",
                "action": "discovered|updated",
            }
        ]
    }


Web Hook APIs
^^^^^^^^^^^^^^^


**Create a new hook**


.. code-block:: REST

    POST /api/2.0/hooks
    {
        "url": "http://www.abc.com/def"
    }


**Delete an existing hook**


.. code-block:: REST

    DELETE /api/2.0/hooks/:id


**Get a list of hooks**


.. code-block:: REST

    GET /api/2.0/hooks


**Get details of a single hook**


.. code-block:: REST

    GET /api/2.0/hooks/:id


**Update an existing hook**


.. code-block:: REST

    PATCH /api/2.0/hooks/:id
    {
        "name": "New Hook"
    }


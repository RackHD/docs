Pollers
-------------------

The pollers API provides functionality for periodic collection of IPMI and SNMP
data.

IPMI
~~~~~~~~~~

IPMI Pollers can be standalone or can be associated with a node. When an
IPMI poller is associated with a node, it will attempt to use that node's IPMI
OBM settings in order to communicate with the BMC. Otherwise, the poller must be
manually configured with that node's IPMI settings.

If a node is found via discovery and contains a BMC catalog, then five IPMI
pollers will be created automatically for that node. The five pollers
correspond to the "power", "selInformation", "sel", "sdr" and "uid" (chassis LED)
commands. These newly-created pollers will not collect any data until the node
has been configured with IPMIOBM settings.


Custom alerts for "sel" command IPMI pollers can be manually configured in their
data definition, based on string and/or regex matching. IPMI pollers for the
"sdr" command will automatically publish alerts onto an AMQP channel if any
sensors of type "threshold" hold a value that does not equal "Not Available" or "ok".
See the Alerts section below for more information.


SNMP
~~~~~~~~~~~~

SNMP pollers can be standalone or associated with a node. With an SNMP poller is
associated with a node, it will attempt to use that node's snmpSettings in order
to communicate via SNMP. Otherwise, the poller must be manually configured with
that node's SNMP settings.

If a node with "type": "switch" is created via the /nodes API with autoDiscover
set to true, then four SNMP-based metric pollers will be created automatically
for that node (see the Metric pollers section below for a list of these).

Example request to create a switch and auto-discover it:

.. code-block:: REST

    POST /api/1.1/nodes
    Content-Type: application/json

    {
      "name": "my switch",
      "identifiers": [],
      "snmpSettings": {
        "host": "10.1.1.3",
        "community": "public"
      },
      "type": "switch",
      "autoDiscover": true
    }

Metric Pollers
~~~~~~~~~~~~~~~~~~~~~~~

In some cases, the data desired from a poller may require more complex processing
than simply running an IPMI or SNMP command and parsing it. To address this,
there is a poller type called a metric. A metric utilizes SNMP or IPMI, but can
make multiple of these calls in aggregate, and add post-processing logic to the
results. There are currently four metrics available in the Monorail system:

- snmp-interface-state
- snmp-interface-bandwidth-utilization
- snmp-memory-usage
- snmp-processor-load

These metrics use SNMP to query multiple sources of information in order to
calculate result data. For example, the bandwidth utilization metric calculates
the delta between two sources of poll data at different times in order to produce
data about how much network bandwidth is flowing through each interface.


API commands
~~~~~~~~~~~~~~~~~~~~~~~

When running the renasar-http process, these are some common API commands you
can send:

* Get available pollers in the library

.. code-block:: REST

    GET /api/1.1/pollers/library

.. code-block:: REST

    curl <server>/api/1.1/pollers/library


* Create a new SNMP poller with a node

To utilize an SNMP poller that references a node, the node document must have
an "snmpSettings" field with a host and community fields:

.. code-block:: REST

    // example node document with snmp settings
    {
      "name": "example node",
      "identifiers": [],
      "snmpSettings": {
        "host": "10.1.1.3",
        "community": "public"
      }
    }

.. code-block:: REST

    POST /api/1.1/pollers
    {
        "type": "snmp",
        "pollInterval": 10000,
        "node": "54daadd764f1a8f1088fdc42",
        "config": {
          "oids": [
            "IF-MIB::ifSpeed",
            "IF-MIB::ifOperStatus"
          ]
        }
    }

.. code-block:: REST

    curl -X POST \
        -H 'Content-Type: application/json' \
        -d '{"type":"snmp","pollInterval":10000,"node":"54daadd764f1a8f1088fdc42",
            "config":{"oids":["IF-MIB::ifSpeed","IF-MIB::ifOperStatus"}}' \
        <server>/api/1.1/pollers

* Create a new IPMI poller with a node

.. code-block:: REST

    POST /api/1.1/pollers
    {
        "type": "ipmi",
        "pollInterval": 10000,
        "node": "54daadd764f1a8f1088fdc42",
        "config": {
          "command": "power"
        }
    }

.. code-block:: REST

    curl -X POST \
        -H 'Content-Type: application/json' \
        -d '{"type":"ipmi","pollInterval":10000,"node":"54daadd764f1a8f1088fdc42",
            "config":{"command":"power"}}' \
        <server>/api/1.1/pollers

.. literalinclude:: samples/ipmi-poller.json
   :language: JSON

* Create a new IPMI poller without a node

.. code-block:: REST

    POST /api/1.1/pollers
    {
        "type": "ipmi",
        "pollInterval": 10000,
        "config": {
          "command": "power",
          "host": "10.1.1.2",
          "user": "admin",
          "password": "admin"
        }
    }

.. code-block:: REST

    curl -X POST \
        -H 'Content-Type: application/json' \
        -d '{"type":"ipmi","pollInterval":10000,"node":"54daadd764f1a8f1088fdc42",
            "config":{"command":"power","host":"10.1.1.2","user":"admin","password":"admin"}}' \
        <server>/api/1.1/pollers

.. literalinclude:: samples/ipmi-poller-no-node.json
   :language: JSON

* Create a new SNMP poller

.. code-block:: REST

    POST /api/1.1/pollers
    {
        "type": "snmp",
        "pollInterval": 10000,
        "config": {
          "host": "10.1.1.3",
          "communityString": "public",
          "oids": [
            "PDU-MIB::outletVoltage",
            "PDU-MIB::outletCurrent"
          ]
        }
    }

.. code-block:: REST

    curl -X POST \
        -H 'Content-Type: application/json' \
        -d '{"type":"snmp","pollInterval":10000,"node":"54daadd764f1a8f1088fdc42",
            "config":{"host":"10.1.1.3","communityString":"public",
              "oids":["PDU-MIB::outletVoltage","PDU-MIB::outletCurrent"]}}' \
        <server>/api/1.1/pollers

.. literalinclude:: samples/snmp-poller.json
   :language: JSON

* Create a new metric poller

Metric pollers can be created by adding the name of the metric to the poller
config instead of data like "oids" or "command"

.. code-block:: REST

    POST /api/1.1/pollers
    {
        "type": "snmp",
        "pollInterval": 10000,
        "node": "54daadd764f1a8f1088fdc42",
        "config": {
           "metric": "snmp-interface-bandwidth-utilization"
        }
    }

.. code-block:: REST

    curl -X POST \
        -H 'Content-Type: application/json' \
        -d '{"type":"snmp","pollInterval":10000,"node":"54daadd764f1a8f1088fdc42",
            "config":{"metric":"snmp-interface-bandwidth-poller"}}' \
        <server>/api/1.1/pollers

* Get a poller's data stream

.. code-block:: REST

    GET /api/1.1/pollers/:id/data

.. code-block:: REST

    curl <server>/api/1.1/pollers/<pollerid>/data

Sample Output: IPMI

.. literalinclude:: samples/ipmi-sdr-poller-data.json
   :language: JSON

Sample Output: SNMP

.. literalinclude:: samples/snmp-poller-data.json
  :language: JSON

* Get list of active pollers

.. code-block:: REST

    GET /api/1.1/pollers

.. code-block:: REST

    curl <server>/api/1.1/pollers

* Get definition for a single poller

.. code-block:: REST

    GET /api/1.1/pollers/:id

.. code-block:: REST

    curl <server>/api/1.1/pollers/<pollerid>

* Update a single poller

.. code-block:: REST

    PATCH /api/1.1/pollers/:id
    {
        "pollInterval": 15000
    }

.. code-block:: REST

    curl -X PATCH \
        -H 'Content-Type: application/json' \
        -d '{"pollInterval":15000}' \
        <server>/api/1.1/pollers/<pollerid>

° Delete a single poller

.. code-block:: REST

    DELETE /api/1.1/pollers/:id

.. code-block:: REST

    curl -X DELETE <server>/api/1.1/pollers/<pollerid>


* Get list of active pollers associated with a node

```
GET /api/1.1/nodes/:id/pollers
---
curl <server>/api/1.1/nodes/<nodeid>/pollers
```

* IPMI Poller Alerts

Alerting is currently supported for "sel" and "sdr" command type pollers.

* Receiving alerts

Alerts are published over AMQP:

- Channel: 'events' (type: topic)
- Routing Key: 'poller.alert.\*' (\* is a uuid assigned to the poller graph that processed the alert)

Sample data for a "sel" alert:

.. code-block:: REST

    {
        host: '10.1.1.3',
        user: 'admin',
        password: 'admin',
        workItemId: '54d6cdff8db79442ddf33333',
        alerts: [
            {
                data: {
                    date: '10/26/2014',
                    time: '20:17:48',
                    sensor: 'Power Unit #0x02',
                    event: 'Fully Redundant',
                    value: 'Deasserted'
                },
                matches: [
                    {
                        sensor: 'Power Unit\s.*$',  // regex supported
                        event: 'Fully Redundant'    // string matching supported
                    }
                ]
            }
        ]
    }


Sample data for an "sdr" alert:

.. code-block:: REST

    {
        host: '10.1.1.3',
        user: 'admin',
        password: 'admin',
        workItemId: '54d6cdff8db79442ddf33333',
        reading: {
            'Entity Id': '7.18',
            'Status': 'nr',
            'Sensor Id': 'VBAT',
            'Normal Minimum': '8.928',
            'Lower non-critical': '2.688',
            'Upper critical': '3.456',
            'Sensor Reading': '5.168',
            'Upper non-critical': '3.312',
            'Lower critical': '2.544',
            'Sensor Type': 'Voltage',
            'Normal Maximum': '11.424',
            'Entry Id Name': 'System Board',
            'Sensor Reading Units': 'Volts',
            'Nominal Reading': '9.216'
        }
    }


Sample data for an "snmp" alert:

.. code-block:: REST

    {
        host: '10.1.1.3',
        community: 'public',
        workItemId: '561c2b3e94e9d7c6057be676',
        pollInterval: 10000,
        node: '561c2b1894e9d7c6057be675',
        alerts: [
            {
                matches: {
                    '.1.3.6.1.2.1.1.5': '/Mounted/',
                    '.1.3.6.1.2.1.1.1': '/Manage/',
                    inCondition: true
                },
                data: {
                    '.1.3.6.1.2.1.1.5.0': 'APC Rack Mounted UPS',
                    '.1.3.6.1.2.1.1.1.0': 'APC Web/SNMP Management Card
                }
            }
        ]
    }

* Creating alerts

Alerting for sdr pollers is automatic, and determined by whether a "threshold" sensor
has a value that does not equal either "ok" or "Not available". In the example sdr
alert above, the value being alerted is "nr", for Non-recoverable.

Alerts for sel poller data are more flexible, and can be user-defined via string or
regex matching. The data structure for an sdr result has five keys: 'date', 'time',
'sensor', 'event' and 'value'. Alert data can be specified via a JSON object that
maps these keys to either exactly matched or regex matched values, e.g.

.. code-block:: JSON

    [
        {
            "sensor": "/Power Unit\s.*$/",
            "event": "Fully Redundant"
        }
    ]

In order for a value string to be interpreted as a regex pattern, it must begin
and end with the '/' character. Additionally, any regex escapes (e.g. \n or \s)
must be double escaped before being serialized and sent over the wire (e.g. \n becomes \\n).
In most programming languages, the equivalent of \<RegexObject\>.toString() will
handle this serialization.

To add an alert to a poller, the above JSON schema must be added to the poller
under config.alerts, e.g.,

.. code-block:: JSON

    {
        "type": "ipmi",
        "pollInterval": 10000,
        "node": "54daadd764f1a8f1088fdc42",
        "config": {
            "command": "sel",
            "alerts": [
                {
                    "sensor": "/Power Unit\s.*$/",
                    "event": "Fully Redundant"
                },
                {
                    "time": "/[0-3][0-3]:.*/",
                    "sensor": "/Session Audit\\s.*$/",
                    "value": "Asserted"
                }
            ]
        }
    }


Snmp poller alerts can be defined just like sel alerts via string or regex matching.
However, the keys for an snmp alert must be the numeric oid whose value you wish
to check against the given string/regex, e.g.,

.. code-block:: REST

    {
        "type":"snmp",
        "pollInterval":10000,
        "node": "560ac7f33ab91d99448fb945",
         "config": {
          "alerts": [
              {
                  ".1.3.6.1.2.1.1.5":"/Mounted/",
                  ".1.3.6.1.2.1.1.1":"/ZA11/"
              }
            ],
          "oids": [
            ".1.3.6.1.2.1.1.1",
            ".1.3.6.1.2.1.1.5"
          ]
        }
    }

Poller JSON format
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Pollers are defined via JSON, with these required fields:

| Name | Type | Flags | Description |
|------|------|-------|-------------|
| type | String | **required** | Type of poller. Valid values: ipmi, snmp |
| pollInterval | Number | **required** | Time in milliseconds to wait in between polls. |

The following fields are only valid for IPMI pollers:

| Name | Type | Flags | Description |
|------|------|-------|-------------|
| config | Object | **required** | Hash of configuration parameters. |
| config.command | String | **required** | IPMI command to run. Valid values: power, sel, sdr |
| config.host | String | *optional* | IP/Hostname of the node's BMC. |
| config.user | String | *optional* | IPMI username. |
| config.password | String | *optional* | IPMI password. |
| config.metric | String | *optional* |  Run a metric poller instead of a simple IPMI query. Use instead of config.command |
| node | String | *optional* |  Node ID to associate this poller with/dynamically look up IPMI settings with. |

The following fields are only valid for SNMP pollers:

| Name | Type | Flags | Description |
|------|------|-------|-------------|
| config | Object | **required** | Hash of configuration parameters. |
| config.host | String | *optional* | IP/Hostname of the node's BMC. |
| config.communityString | String | *optional* | SNMP community string. |
| config.oids | String[] | *optional* | Array of OIDs to poll. |
| config.metric | String | *optional* |  Run a metric poller instead of a simple OID query. Use instead of config.oids |
| node | String | *optional* |  Node ID to associate this poller with/dynamically look up SNMP settings with. |

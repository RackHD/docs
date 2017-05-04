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
pollers are automatically created for that node. The five pollers
correspond to the "power", "selInformation", "sel", "sdr" and "uid" (chassis LED)
commands. These pollers do not collect data until the node
has been configured with IPMIOBM settings.


Custom alerts for "sel" command IPMI pollers can be manually configured in their
data definition, based on string and/or regex matching. IPMI pollers for the
"sdr" command will automatically publish alerts onto an AMQP channel if any
sensors of type "threshold" hold a value that does not equal "Not Available" or "ok".
See the Alerts section below for more information.


SNMP
~~~~~~~~~~~~

SNMP pollers can be standalone or associated with a node. When an SNMP poller is
associated with a node, it attempts to use that node's snmpSettings in order
to communicate via SNMP. Otherwise, the poller must be manually configured with
that node's SNMP settings.

If a node with "type": "switch" is created via the /nodes API with autoDiscover
set to true, then six SNMP-based metric pollers will be created automatically
for that node (see the Metric pollers section below for a list of these).

Example request to create and auto-discover a switch::

    POST /api/current/nodes
    Content-Type: application/json

    {
      "name": "my switch",
      "identifiers": [],
      "ibms": [{"service": "snmp-ibm-service", "config": {"host": "10.1.1.3", "community": "public"}}],
      "type": "switch",
      "autoDiscover": true
    }

Metric Pollers
~~~~~~~~~~~~~~~~~~~~~~~

In some cases, the data desired from a poller may require more complex processing
than simply running an IPMI or SNMP command and parsing it. To address this,
there is a poller type called a metric. A metric uses SNMP or IPMI, but can
make multiples of these calls in aggregate and add post-processing logic to the
results. There are currently six metrics available in the RackHD system:

- snmp-interface-state
- snmp-interface-bandwidth-utilization
- snmp-memory-usage
- snmp-processor-load
- snmp-txrx-counters
- snmp-switch-sensor-status

These metrics use SNMP to query multiple sources of information in order to
calculate result data. For example, the bandwidth utilization metric calculates
the delta between two sources of poll data at different times in order to produce
data about how much network bandwidth is flowing through each interface.


API commands
~~~~~~~~~~~~~~~~~~~~~~~

When running the on-http process, these are some common API commands you
can send:

**Get available pollers in the library**

.. code-block:: REST

    GET /api/current/pollers/library

.. code-block:: REST

    curl <server>/api/current/pollers/library


**Create a new SNMP poller with a node**


To use an SNMP poller that references a node, the node document must have
an "ibms" field with a host and community fields:

.. code-block:: REST

    // example node document with snmp settings
    {
      "name": "example node",
      "identifiers": [],
      "ibms": [{"service": "snmp-ibm-service", "config": {"host": "10.1.1.3", "community": "public"}}]
    }

.. code-block:: REST

    POST /api/current/pollers
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
        <server>/api/current/pollers

**Create a New IPMI Poller With a Node**

.. code-block:: REST

    POST /api/current/pollers
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
        <server>/api/current/pollers

.. literalinclude:: samples/ipmi-poller.json
   :language: JSON

**Create a New IPMI Poller Without a Node**

.. code-block:: REST

    POST /api/current/pollers
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
        <server>/api/current/pollers

.. literalinclude:: samples/ipmi-poller-no-node.json
   :language: JSON

**Create a New SNMP Poller**

.. code-block:: REST

    POST /api/current/pollers
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
        <server>/api/current/pollers

.. literalinclude:: samples/snmp-poller.json
   :language: JSON

**Create a New Metric Poller**

Metric pollers can be created by adding the name of the metric to the poller
config instead of data like "oids" or "command"

.. code-block:: REST

    POST /api/current/pollers
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
        <server>/api/current/pollers

**Get a Poller's Data Stream**

.. code-block:: REST

    GET /api/current/pollers/:id/data

.. code-block:: REST

    curl <server>/api/current/pollers/<pollerid>/data

Sample Output: IPMI

.. literalinclude:: samples/ipmi-sdr-poller-data.json
   :language: JSON

Sample Output: SNMP

.. literalinclude:: samples/snmp-poller-data.json
  :language: JSON

**Get List of Active Pollers**

.. code-block:: REST

    GET /api/current/pollers

.. code-block:: REST

    curl <server>/api/current/pollers

**Get Definition for a Single Poller**

.. code-block:: REST

    GET /api/current/pollers/:id

.. code-block:: REST

    curl <server>/api/current/pollers/<pollerid>

**Update a Single Poller to change the interval**

.. code-block:: REST

    PATCH /api/current/pollers/:id
    {
        "pollInterval": 15000
    }

.. code-block:: REST

    curl -X PATCH \
        -H 'Content-Type: application/json' \
        -d '{"pollInterval":15000}' \
        <server>/api/current/pollers/<pollerid>

**Update a Single Poller to pause the poller**

.. code-block:: REST

    PATCH /api/current/pollers/:id
    {
        "paused": true
    }

.. code-block:: REST

    curl -X PATCH \
        -H 'Content-Type: application/json' \
        -d '{"paused":true}' \
        <server>/api/current/pollers/<pollerid>

**Delete a Single Poller**

.. code-block:: REST

    DELETE /api/current/pollers/:id

.. code-block:: REST

    curl -X DELETE <server>/api/current/pollers/<pollerid>


**Get List of Active Pollers Associated With a Node**

.. code-block:: REST

    GET /api/current/nodes/:id/pollers

.. code-block:: REST

    curl <server>/api/current/nodes/<nodeid>/pollers


IPMI Poller Alerts
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Please see :doc:`event_notification` for more poller alert events information. 

Sample data for a "sel" alert:

.. code-block:: REST

    {
        "type":"polleralert",
        "action":"sel.updated",
        "typeId":"588586022116386a0d1e860f",
        "nodeId":"588585bee0f66f700da40335",
        "severity":"warning",
        "data":{
            "user":"admin",
            "host":"172.31.128.13",
            "alert":{
                "matches":[
                    {
                        "Event Type Code":"07",
                        "Event Data":"/010000|040000/"
                    }
                ],
                "reading":{
                    "SEL Record ID":"0102",
                    "Record Type":"02",
                    "Timestamp":"01/01/1970 03:09:50",
                    "Generator ID":"0001",
                    "EvM Revision":"04",
                    "Sensor Type":"Physical Security",
                    "Sensor Number":"02",
                    "Event Type":"Generic Discrete",
                    "Event Direction":"Assertion Event",
                    "Event Data":"010000",
                    "Description":"Transition to Non-critical from OK",
                    "Event Type Code":"07",
                    "Sensor Type Code":"05"
                }
            }
        },
        "version":"1.0",
        "createdAt":"2017-01-23T07:36:53.092Z"
    }

Sample data for an "sdr" alert:

.. code-block:: REST

    {
        "type":"polleralert",
        "action":"sdr.updated",
        "typeId":"588586022116386a0d1e8610",
        "nodeId":"588585bee0f66f700da40335",
        "severity":"information",
        "data":{
            "host":"172.31.128.13",
            "user":"admin",
            "inCondition":true,
            "reading":{
                "sensorId":"Fan_SSD1 (0xfd)",
                "entityId":"29.1",
                "entryIdName":"Fan Device",
                "sdrType":"Threshold",
                "sensorType":"Fan",
                "sensorReading":"0",
                "sensorReadingUnits":"% RPM",
                "nominalReading":"",
                "normalMinimum":"",
                "normalMaximum":"",
                "statesAsserted":[],
                "status":"LowerCritical",
                "lowerCritical":"500.000",
                "lowerNonCritical":"1000.000",
                "positiveHysteresis":"Unspecified",
                "negativeHysteresis":"Unspecified",
                "minimumSensorRange":"Unspecified",
                "maximumSensorRange":"Unspecified",
                "eventMessageControl":"Per-threshold",
                "readableThresholds":"lcr lnc",
                "settableThresholds":"lcr lnc",
                "thresholdReadMask":"lcr lnc",
                "assertionsEnabled":["lnc- lcr-"],
                "deassertionsEnabled":["lnc- lcr-"]
            }
        },
        "version":"1.0",
        "createdAt":"2017-01-23T07:36:56.179Z"
    }


Sample data for an "snmp" alert:

.. code-block:: REST

    {
        "type":"polleralert",
        "action":"snmp.updated",
        "typeId":"588586022116386a0d1e8611",
        "nodeId":"588585bee0f66f700da40335",
        "severity":"information",
        "data":{
            "states":{
                "last":"ON",
                "current":"OFF"
            }
        },
        data: {
            host: '10.1.1.3',
            oid: '.1.3.6.1.2.1.1.5.0',
            value: 'APC Rack Mounted UPS'
            matched: '/Mounted/'
        }
        "version":"1.0",
        "createdAt":"2017-01-23T08:20:32.231Z"
    }

Sample data for an "snmp" metric alert:

.. code-block:: REST

    {
        "type":"polleralert",
        "action":"snmp.updated",
        "typeId":"588586022116386a0d1e8611",
        "nodeId":"588585bee0f66f700da40335",
        "severity":"information",
        "data":{
            "states":{
                "last":"ON",
                "current":"OFF"
            }
        },
        data: {
            host: '127.0.0.1',
            oid: '.1.3.6.1.4.1.9.9.117.1.1.2.1.2.470',
            value: 'No Such Instance currently exists at this OID',
            matched: { contains: 'No Such Instance' },
            severity: 'warning',
            description: 'PSU element is not present',
            metric: 'snmp-switch-sensor-status'
        }
        "version":"1.0",
        "createdAt":"2017-01-23T08:20:32.231Z"
    }


**Creating Alerts**

Alerting for sdr pollers is automatic and triggered when a threshold sensor
has a value that does not equal either "ok" or "Not available". In the example sdr
alert above, the value being alerted is "nr", for Non-recoverable.

Alerts for sel poller data are more flexible and can be user-defined via string or
regex matching. The data structure for an sdr result has five keys: 'date', 'time',
'sensor', 'event' and 'value'. Alert data can be specified via a JSON object that
maps these keys to either exactly matched or regex matched values:

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
under config.alerts:

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
However, the keys for an snmp alert must be a string or regex whose value you wish
to check against the given OID numeric or string representation:

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

Complex alerts are done by replacing the string/regex value with a validation
object.  The following example will match all OIDs with 'InErrors' in the 
name and generate an alert when the value is greater than 0.

.. code-block:: REST

    {
        "type":"snmp",
        "pollInterval":10000,
        "node": "560ac7f33ab91d99448fb945",
        "config": {
            "alerts": [
                {
                    "/\\S*InErrors/": {
                        "greaterThan": 0,
                        "integer": true,
                        "severity": "ignore"
                    }
                }
             ],
            "metric": "snmp-txrx-counters"
        }
    }
          


Chassis Power State Alert 
~~~~~~~~~~~~~~~~~~~~~~~~~~~

The IPMI chassis poller will publish an alert message when the power state of the node transitions. The AMQP message
payload will contain both the current and last power state, a reference location to the node resource and a reference
location to the pollers current data cache.

- Example message:

.. code-block:: JSON

    {
        "type":"polleralert",
        "action":"chassispower.updated",
        "typeId":"588586022116386a0d1e8611",
        "nodeId":"588585bee0f66f700da40335",
        "severity":"information",
        "data":{
            "states":{
                "last":"ON",
                "current":"OFF"
            }
        },
        "version":"1.0",
        "createdAt":"2017-01-23T08:20:32.231Z"
    }


Poller JSON Format
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Pollers are defined via JSON with these required fields:

==================== =========== ============ ============================================
Name                  Type         Flags       Description
==================== =========== ============ ============================================
type                 String      **required**  Poller type. Valid values: **ipmi**, **snmp**
pollInterval         Number      **required**  Time in milliseconds to wait between polls.
==================== =========== ============ ============================================

The following fields are only valid for IPMI pollers:

==================== =========== ============ ============================================
Name                  Type         Flags       Description
==================== =========== ============ ============================================
config                Object     **required**  Hash of configuration parameters.
config.command        String     **required**  IPMI command to run. Valid values: **power**, **sel**, **sdr**
config.host           String     *optional*    IP/Hostname of the node's BMC.
config.user           String     *optional*    IPMI username.
config.password       String     *optional*    IPMI password.
config.metric         String     *optional*    Run a metric poller instead of a simple IPMI query. Use instead of **config.command**.
node                  String     *optional*    Node ID to associate this poller with dynamically look up IPMI settings.
==================== =========== ============ ============================================

The following fields are only valid for SNMP pollers:

==================== =========== ============ ============================================
Name                    Type      Flags       Description
==================== =========== ============ ============================================
config                Object     **required** Hash of configuration parameters.
config.host           String     *optional*   IP/Hostname of the node's BMC.
config.community      String     *optional*   SNMP community string.
config.oids           String[]   *optional*   Array of OIDs to poll.
config.metric         String     *optional*   Run a metric poller instead of a simple OID query.
                                              Use instead of **config.oids**.
node                  String     *optional*   Node ID to associate this poller with dynamically look up SNMP settings.
==================== =========== ============ ============================================

The following fields can be PATCH'ed to change poller behavior:

==================== =========== ============================================
Name                  Type       Description
==================== =========== ============================================
pollInterval         Number      Time in milliseconds to wait between polls.
paused               Boolean     Determines if the poller can be scheduled. Setting 'paused' to true
                                 will cause the poller to no longer be run when pollInterval expires
==================== =========== ============================================


ARP Cache Poller
~~~~~~~~~~~~~~~~
.. _ARP: https://en.wikipedia.org/wiki/Address_Resolution_Protocol
.. _/proc/net/arp: https://www.kernel.org/doc/Documentation/filesystems/proc.txt

With the Address Resolution Protocol (`ARP`_) cache poller service enabled, the RackHD lookup service will update MAC/IP bindings based on the Linux kernel's `/proc/net/arp`_ table. This ARP poller deprecates the need for running the DHCP lease file poller since any IP request made to the host will attempt to resolve the hardware addresses IP and update the kernel's ARP cache. 


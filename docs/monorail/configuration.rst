Configuration
----------------------

The following JSON is an examples of the current defaults:

**monorail.json**


.. code-block:: JSON

    {
        "amqp": "amqp://localhost",
        "mongo": {
            "host": "localhost",
            "port": 27017,
            "database": "pxe",
            "user": "",
            "password": ""
        },
        "verbose": true,
        "color": true,
        "server": "10.1.1.1",
        "http": true,
        "https": true,
        "httpPort": 80,
        "httpsPort": 443,
        "httpsCert": "data/dev-cert.pem",
        "httpsKey": "data/dev-key.pem",
        "subnetnetmask": "255.255.255.0",
        "gatewayaddr": "10.1.1.1",
        "broadcastaddr": "10.1.1.255",
        "httpStaticDirectory": "./static/http",
        "httpFrontendDirectory": "../renasar-fog-ui/dist",
        "httpApiDocsDirectory": "./build/apidoc",
        "stompLogLevel": "info",
        "stompIdentifier": "Renasar",
        "sockJsStompPrefix": "/sockjs_stomp",
        "logfileLocation": "./logs",
        "maxTaskPayloadSize": "10mb",
        "statsdPrefix": "http."
    }


Keys
~~~~~~~~~~~~~~~~~~

Queue
^^^^^^^^^^^^^^^^^^^^^^

=============== ===================================================================================
Setting         | Description
=============== ===================================================================================
amqp            | URI for accessing the AMQP interprocess communications channel
=============== ===================================================================================

Persistence
^^^^^^^^^^^^^^^^^^^^^^

============= ===================================================================================
Setting       | Description
============= ===================================================================================
dbURI         | The URI for accessing an instance of MongoDB database used for persistence.
databasetype  | Back-end persistence for the DHCP lease data. "MEMORY-MONGODB" is
              | the only valid value until additional back-end lease persistence support is
              | added.
============= ===================================================================================

Networking
^^^^^^^^^^^^^^^^^^^^^^

============== ===================================================================================
Setting        | Description
============== ===================================================================================
server         | IP address of interface to bind to for TFTP, SYSLOG and HTTP services.
               |
               | **Note:** DHCP binds to 0.0.0.0 to support broadcast request/response within
               | NodeJS.
broadcastaddr  | Broadcast address for the network range (for DHCP)
subnetmask     | Subnet mask for the network range (for DHCP)
iprange        | Range of IP addresses, either in CIDR format or a list of IP addresses
               | to provide via DHCP
http           | Toggle HTTP
https          | Toggle HTTPS
httpPort       | HTTP port for support API (internal and public) requests
httpsPort      | HTTPS port for support API (internal and public) requests
tftpport       | DP port for supporting TFTP requests
syslogport     | UDP port for listening for syslog messages
============== ===================================================================================


HTTP
^^^^^^^^^^^^^^^^^^^^^^

================== ===================================================================================
Setting            | Description
================== ===================================================================================
httpsCert          | Filename of SSL certificate
httpsKey           | Filename of RSA private key
httpsPfx           | pfx file containing the SSL cert and private key (only needed if
                   | the key and cert are omitted)
maxTaskPayloadSize | maximum payload size expected through TASK runner API callbacks from
                   | microkernel
================== ===================================================================================


Workflows
^^^^^^^^^^^^^^^^^^^^^^

================= ===================================================================================
Setting           | Description
================= ===================================================================================
defaultWorkflow   | name of the default workflow to be invoked upon new machine discovery
                  | (only functional if `promiscuous` is also enabled)
================= ===================================================================================

Pollers
^^^^^^^^^^^^^^^^^^^^^^

================= ===================================================================================
Setting           | Description
================= ===================================================================================
pollerCacheSize   | Maximum poller entries to keep cached in memory
================= ===================================================================================


Out-of-Band Management Control
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

================= ===================================================================================
Setting           | Description
================= ===================================================================================
obmInitialDelay   | Delay before retrying an OBM invocation
obmRetries        | Number of retries to attempt before failing an OBM invocation
================= ===================================================================================


Content Directories
^^^^^^^^^^^^^^^^^^^^^^

======================= ===================================================================================
Setting                 | Description
======================= ===================================================================================
httpStaticDirectory     | Fully-qualified directory to where static HTTP content is served
httpFrontendDirectory   | Fully-qualified directory to the web GUI content
httpApiDocsDirectory    | Fully-qualified directory to the API docs
tftproot                | Fully-qualified directory to where static TFTP content is served
======================= ===================================================================================

Logging
^^^^^^^^^^^^^^^^^^^^^^

* verbose
* color

Debugging
^^^^^^^^^^^^^^^^^^^

======================= ===================================================================================
Setting                 | Description
======================= ===================================================================================
statsdPrefix            | Application-specific *statsd* metrics
======================= ===================================================================================

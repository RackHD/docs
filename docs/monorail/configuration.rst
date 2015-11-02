Configuration
----------------------

The following JSON is an examples of the current defaults:

**monorail.json**


.. code-block:: JSON

{
    "CIDRNet": "172.31.128.0/22",
    "amqp": "amqp://localhost",
    "apiServerAddress": "172.31.128.1",
    "apiServerPort": 8080,
    "broadcastaddr": "172.31.131.255",
    "dhcpGateway": "172.31.128.1",
    "dhcpProxyBindAddress": "172.31.128.1",
    "dhcpProxyBindPort": 4011,
    "dhcpSubnetMask": "255.255.252.0",
    "gatewayaddr": "172.31.128.1",
    "httpBindAddress": "0.0.0.0",
    "httpBindPort": 8080,
    "httpDocsRoot": "./build/apidoc",
    "httpEnabled": true,
    "httpFileServiceRoot": "./static/files",
    "httpFileServiceType": "FileSystem",
    "httpStaticRoot": "/opt/monorail/static/http",
    "httpsBindAddress": "0.0.0.0",
    "httpsBindPort": 8443,
    "httpsCert": "data/dev-cert.pem",
    "httpsEnabled": false,
    "httpsKey": "data/dev-key.pem",
    "httpsPfx": null,
    "mongo": "mongodb://localhost/pxe",
    "sharedKey": "<key>",
    "statsd": "127.0.0.1:8125",
    "subnetmask": "255.255.252.0",
    "syslogBindAddress": "172.31.128.1",
    "syslogBindPort": 514,
    "tftpBindAddress": "172.31.128.1",
    "tftpBindPort": 69,
    "tftpRoot": "./static/tftp"
}

Keys
~~~~~~~~~~~~~~~~~~

Queue
^^^^^^^^^^^^^^^^^^^^^^

=============== ===============================================================================
Setting           Description
=============== ===============================================================================
amqp            | URI for accessing the AMQP interprocess communications channel
=============== ===============================================================================

Persistence
^^^^^^^^^^^^^^^^^^^^^^

============= ===================================================================================
Setting         Description
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
               | Node.js.
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

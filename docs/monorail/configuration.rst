Configuration
=============

The following JSON is an examples of the current defaults:

monorail.json
--------------

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


# Keys

## Queue

* `amqp` the URI for accessing the AMQP interprocess communications channel

## Persistence

* `dbURI` the URI to accessing an instance of MongoDB used for persistence.
* `databasetype` back-end persistence for the DHCP lease data. "MEMORY-MONGODB" is the only valid value until additional back-end lease persistence support is added.

## Networking

* `server` IP address of interface to bind to for TFTP, SYSLOG and HTTP services. Note: DHCP binds to 0.0.0.0 to support broadcast request/response within NodeJS.

* `broadcastaddr` the broadcast address for the network range (for DHCP)
* `subnetmask` the subnet mask for the network range (for DHCP)
* `iprange` the range of IP addresses, in either CIDR format, or a list of IP addresses, to provide via DHCP

* `http` Toggle HTTP
* `https` Toggle HTTPS
* `httpPort` HTTP port for support API (internal and public) requests
* `httpsPort` HTTPS port for support API (internal and public) requests
* `tftpport` UDP port for supporting TFTP requests
* `syslogport` UDP port for listening for syslog messages

### HTTP

* `httpsCert` Filename of SSL certificate
* `httpsKey` Filename of RSA private key
* `httpsPfx` pfx file containing the SSL cert and private key (only needed if the key and cert are omitted)
* `maxTaskPayloadSize` maximum payload size expected through TASK runner API callbacks from microkernel

## Workflows

* `defaultWorkflow` name of the default workflow to be invoked upon new machine discovery (only functional if `promiscuous` is also enabled)

## Pollers

* `pollerCacheSize` max poller entries to keep cached in memory

## Out of band management control

* `obmInitialDelay` delay before retrying an OBM invocation
* `obmRetries` number of retries to attempt before failing an OBM invocation

## Content directories

* `httpStaticDirectory` fully qualified directory to where static HTTP content is served
* `httpFrontendDirectory` fully qualified directory to the web GUI content
* `httpApiDocsDirectory` fully qualified directory to the API docs
* `tftproot` fully qualified directory to where static TFTP content is served

## Logging

* `verbose`
* `color`

## Debugging

* `statsdPrefix` application specific statsd metrics

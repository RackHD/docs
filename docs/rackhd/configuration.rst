RackHD Configuration
----------------------

The following JSON is an examples of the current defaults:

config.json_

.. _config.json: https://github.com/RackHD/RackHD/blob/master/packer%2Fansible%2Froles%2Fmonorail%2Ffiles%2Fconfig.json

.. code-block:: JSON

    {
        "CIDRNet": "172.31.128.0/22",
        "amqp": "amqp://localhost",
        "apiServerAddress": "172.31.128.1",
        "apiServerPort": 9080,
        "broadcastaddr": "172.31.131.255",
        "dhcpGateway": "172.31.128.1",
        "dhcpProxyBindAddress": "172.31.128.1",
        "dhcpProxyBindPort": 4011,
        "dhcpSubnetMask": "255.255.252.0",
        "gatewayaddr": "172.31.128.1",
        "httpEndpoints": [
            {
                "address": "0.0.0.0",
                "port": 8080,
                "httpsEnabled": false,
                "proxiesEnabled": true,
                "authEnabled": false,
                "routers": "northbound-api-router"
            },
            {
                "address": "172.31.128.1",
                "port": 9080,
                "httpsEnabled": false,
                "proxiesEnabled": true,
                "authEnabled": false,
                "routers": "southbound-api-router"
            }
        ],
        "httpDocsRoot": "./build/apidoc",
        "httpFileServiceRoot": "./static/files",
        "httpFileServiceType": "FileSystem",
        "httpProxies": [
            {
                "localPath": "/coreos",
                "server": "http://stable.release.core-os.net",
                "remotePath": "/amd64-usr/current/"
            }
        ],
        "httpStaticRoot": "/opt/monorail/static/http",
        "authTokenSecret": "RackHDRocks!",
        "authTokenExpireIn": 86400,
        "mongo": "mongodb://localhost/pxe",
        "sharedKey": "qxfO2D3tIJsZACu7UA6Fbw0avowo8r79ALzn+WeuC8M=",
        "statsd": "127.0.0.1:8125",
        "subnetmask": "255.255.252.0",
        "syslogBindAddress": "172.31.128.1",
        "syslogBindPort": 514,
        "tftpBindAddress": "172.31.128.1",
        "tftpBindPort": 69,
        "tftpRoot": "./static/tftp",
        "minLogLevel": 2,
        "logColorEnable": false,
        "enableUPnP": true,
        "ssdpBindAddress": "0.0.0.0",
        "heartbeatIntervalSec": 10,
        "wssBindAddress": "0.0.0.0",
        "wssBindPort": 9100
    }


Configuration Parameters
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The following table describes the configuration parameters in config.json:


.. list-table::
    :widths: 20 100
    :header-rows: 1

    * - Parameter
      - Description
    * - amqp
      - URI for accessing the AMQP interprocess communications channel
    * - apiServerAddress
      - External facing IP address of the API server
    * - apiServerPort
      - External facing port of the API server
    * - dhcpLeasesPath
      - Path to dhcpd.leases file.
    * - dhcpGateway
      - Gateway IP for the network for DHCP
    * - dhcpProxyBindAddress
      - IP for DHCP proxy server to bind  (defaults to '0.0.0.0'). **Note:** DHCP binds to 0.0.0.0 to support broadcast request/response within Node.js.
    * - dhcpProxyBindPort
      - Port for DHCP proxy server to bind (defaults to 4011).
    * - dhcpProxyOutPort
      - Port for DHCP proxy server to respond to legacy boot clients (defaults to 68).
    * - dhcpProxyEFIOutPort
      - Port for DHCP proxy server to respond to EFI clients (defaults to 4011).
    * - httpApiDocsDirectory
      - Fully-qualified directory containing the API docs.
    * - httpEndpoints
      - Collection of http/https endpoints. See details in `Setting up HTTP/HTTPS endpoint`_
    * - httpFileServiceRoot
      - Directory path for for storing uploaded files on disk.
    * - httpFileServiceType
      - Backend storage mechanism for file service. Currently only FileSystem is supported.
    * - httpProxies
      - Optional HTTP/HTTPS proxies list. There are 3 parameters for each proxy:

        "localPath"/"remotePath" are optional and defaults to "/". A legal "localPath"/"remotePath" string must start with slash and ends without slash, like "/mirrors".
        If "localPath" is assigned to an existing local path like "/api/common/nodes", proxy won't work. Instead the path will keep its original feature and function.
        "server" is a must, both http and https servers are supported. A legal "server" string must ends without slash like "http://centos.eecs.wsu.edu". Instead "http://centos.eecs.wsu.edu/" is illegal.

        Example:

        { "server": "http://centos.eecs.wsu.edu", "localPath": "/centos" } would map http requests to local directory /centos/ to http://centos.eecs.wsu.edu/

        { "server": "https://centos.eecs.wsu.edu", "remotePath": "/centos" } would map http requests to local directory / to https://centos.eecs.wsu.edu/centos/

        Note: To ensure this feature works, the httpProxies need be separately enabled for specified HTTP/HTTPS endpoint. See details in `Setting up HTTP/HTTPS endpoint`_
    * - httpFrontendDirectory
      - Fully-qualified directory to the web GUI content
    * - httpStaticDirectory
      - Fully-qualified directory to where static HTTP content is served
    * - maxTaskPayloadSize
      - Maximum payload size expected through TASK runner API callbacks from microkernel
    * - mongo
      - URI for accessing MongoDB. To support Mongo Replica Set feature, URI format is, mongodb://[username:password@]host1[:port1][,host2[:port2],...[,hostN[:portN]]][/[database][?options]]
    * - obmInitialDelay
      - Delay before retrying an OBM invocation
    * - obmRetries
      - Number of retries to attempt before failing an OBM invocation
    * - pollerCacheSize
      - Maximum poller entries to cache in memory
    * - statsdPrefix
      - Application-specific *statsd* metrics for debugging
    * - syslogBindPort
      - Port for syslog (defaults to 514).
    * - syslogBindAddress
      - Address for the syslog server to bind to (defaults to '0.0.0.0').
    * - tftpBindAddress
      - Address for TFTP server to bind to (defaults to '0.0.0.0').
    * - tftpBindPort
      - Listening port for TFTP server  (defaults to 69).
    * - tftpBindAddress
      - File root for TFTP server to serve files (defaults to './static/tftp').
    * - tftproot
      - Fully-qualified directory from which static TFTP content is served
    * - minLogLevel
      - A numerical value for filtering the logging from RackHD
    * - logColorEnable
      - A boolean value to toggle the colorful log output (defaults to false)
    * - enableLocalHostException
      - Set to true to enable the localhost exception, see :ref:`localhost-exception-label`.
    * - enableUPnP
      - Set to true to advertise RackHD Restful API services using SSDP (Simple Service Discovery Protocol).
    * - ssdpBindAddress
      - The bind address to send the SSDP advertisements on (defaults to 0.0.0.0).
    * - heartbeatIntervalSec
      - Integer value setting the heartbeat send interval in seconds. Setting this value to 0 will disable the heartbeat service (defaults to 10)
    * - wssBindAddress
      - Address for RackHD WebSocket Service to bind to (defaults to '0.0.0.0').
    * - wssBindPort
      - Listening port for RackHD WebSocket Service (defaults to 9100).


The log levels for filtering are defined at https://github.com/RackHD/on-core/blob/master/lib/common/constants.js#L36-L44

These configurations can also be overridden by setting environment variables in the
process that's running each application, or on the command line when running node directly.
For example, to override the value of amqp for the configuration, you could use::

    export amqp=amqp://another_host:5763

prior to running the relevant application.

HTTPS/TLS Configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To use TLS, a private RSA key and X.509 certificate must be provided. On Ubuntu and
Mac OS X, the openssl command line tool can be used to generate keys and certificates.

For internal development purposes, a self-signed certificate can be used. When using a self-signed
certificate, clients must manually include a rule to trust the certificate's authenticity.

By default, the application uses a self-signed certificate issued by Monorail which requires no
configuration. Custom certificates can also be used with some configuration.

**Parameters**

See the table in `Configuration Parameters`_ for information about HTTP/HTTPS configuration parameters.
These parameters beging with *HTTP* and *HTTPS*.

BMC Username and Password Configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

A node gets discovered and the BMC IPMI comes up with a default username/password. For a user to add
their own username/password during discovery certain steps need to be followed:

First, edit Sku Discovery graph located at ``on-taskgraph/lib/graphs/discovery-sku-graph.js``
to include the new graph **set-bmc-credentials-graph** located at ``on-taskgraph/lib/graphs/set-bmc-credentials-graph.js``.
This will run the tasks to create a new user called '__rackhd__' with a randomly generated password and update obm settings
accordingly.
Below is a snippet of the Sku Discovery graph which includes **set-bmc-credentials-graph** (please note that this is not the complete graph, refer to the link above to get the entire discovery-sku-graph, this snippet only shows where to add the
**set-bmc-credentials-graph** ) :

.. code-block:: javascript

    module.exports = {
        friendlyName: 'SKU Discovery',
        injectableName: 'Graph.SKU.Discovery',
        options: {
            defaults: {
                graphOptions: {
                    target: null
                },
                nodeId: null
            }
        },
        tasks: [
            {
                label: 'discovery-graph',
                taskDefinition: {
                    friendlyName: 'Run Discovery Graph',
                    injectableName: 'Task.Graph.Run.Discovery',
                    implementsTask: 'Task.Base.Graph.Run',
                    options: {
                        graphName: 'Graph.Discovery',
                        graphOptions: {}
                    },
                    properties: {}
                }
            },
            {
                label: 'set-bmc-credentials-graph',
                taskDefinition: {
                    friendlyName: 'Run BMC Credential Graph',
                    injectableName: 'Task.Graph.Run.Bmc',
                    implementsTask: 'Task.Base.Graph.Run',
                    options: {
                        graphName: 'Graph.Set.Bmc.Credentials',
                        defaults : {
                            graphOptions: {   }
                        }
                    },
                    properties: {}
                },
                waitOn: {
                    'discovery-graph': 'succeeded'
                }
            },
            {
                label: 'generate-sku',
                waitOn: {
                    'set-bmc-credentials-graph': 'succeeded'
                },
                taskName: 'Task.Catalog.GenerateSku'
            }
        ]
    };


Next, edit **Discovery workflow graph** located at ``on-taskgraph/lib/graphs/discovery-graph.js``
to remove the reboot task. The reboot task is already included in the **set-bmc-credentials-graph**
that was added to the **Sku Discovery graph** in the first step.
Below is a snippet of the Discovery graph without the reboot task (the reboot task was originally located
after the task 'catalog-lldp')

.. code-block:: javascript

   module.exports = {
    friendlyName: 'Discovery',
    injectableName: 'Graph.Discovery',
    options: {
        'bootstrap-ubuntu': {
            'triggerGroup': 'bootstrap'
        },
        'finish-bootstrap-trigger': {
            'triggerGroup': 'bootstrap'
        }
    },
    tasks: [
        {
            label: 'bootstrap-ubuntu',
            taskName: 'Task.Linux.Bootstrap.Ubuntu'
        },
        {
            label: 'catalog-dmi',
            taskName: 'Task.Catalog.dmi'
        },
        {
            label: 'catalog-ohai',
            taskName: 'Task.Catalog.ohai',
            waitOn: {
                'catalog-dmi': 'finished'
            }
        },
        {
            label: 'catalog-bmc',
            taskName: 'Task.Catalog.bmc',
            waitOn: {
                'catalog-ohai': 'finished'
            },
            ignoreFailure: true
        },
        {
            label: 'catalog-lsall',
            taskName: 'Task.Catalog.lsall',
            waitOn: {
                'catalog-bmc': 'finished'
            },
            ignoreFailure: true
        },
        {
            label: 'catalog-megaraid',
            taskName: 'Task.Catalog.megaraid',
            waitOn: {
                'catalog-lsall': 'finished'
            },
            ignoreFailure: true
        },
        {
            label: 'catalog-smart',
            taskName: 'Task.Catalog.smart',
            waitOn: {
                'catalog-megaraid': 'finished'
            },
            ignoreFailure: true
        },
        {
            label: 'catalog-driveid',
            taskName: 'Task.Catalog.Drive.Id',
            waitOn: {
                'catalog-smart': 'finished'
            },
            ignoreFailure: true
        },
        {
            label: 'catalog-lldp',
            taskName: 'Task.Catalog.LLDP',
            waitOn: {
                'catalog-driveid': 'finished'
            },
            ignoreFailure: true
        },
       {
            label: 'finish-bootstrap-trigger',
            taskName: 'Task.Trigger.Send.Finish',
            waitOn: {
                'catalog-lldp': 'finished'
            }
        }
    ]
   };


Once the above steps are completed (edited and saved) the service needs to be restarted:

.. code-block:: shell

    sudo service on-taskgraph start


If a user wants to change the BMC credentials later in time, when the node has been already discovered and database updated, a separate workflow located at ``on-taskgraph/lib/graphs/bootstrap-bmc-credentials-setup-graph.js`` can be posted using Postman or Curl command.

    POST:        http://server-ip:8080/api/1.1/workflows/

   add the below content in the json body for payload (example node identifier and username, password shown below)

.. code-block:: shell

   {
       "name": "Graph.Bootstrap.With.BMC.Credentials.Setup",
       "options": {
            "defaults": {
                "graphOptions": {
                    "target": "56e967f5b7a4085407da7898",
                    "generate-pass": {
                        "user": "7",
                        "password": "7"
                    }
                },
                "nodeId": "56e967f5b7a4085407da7898"
            }
        }
   }

By running this workflow, a boot-graph runs to bootstrap an ubuntu image on the node again and set-bmc-credentials-graph runs the required tasks to update the BMC credentials. Below is a snippet of the 'Bootstrap-And-Set-Credentials graph', when the graph is posted the node reboots and starts the discovery process

.. code-block:: javascript

  module.exports = {
    friendlyName: 'Bootstrap And Set Credentials',
    injectableName: 'Graph.Bootstrap.With.BMC.Credentials.Setup',
    options: {
        defaults: {
            graphOptions: {
                target: null
            },
            nodeId: null
        }
    },
    tasks: [
        {
            label: 'boot-graph',
            taskDefinition: {
                friendlyName: 'Boot Graph',
                injectableName: 'Task.Graph.Run.Boot',
                implementsTask: 'Task.Base.Graph.Run',
                options: {
                    graphName: 'Graph.BootstrapUbuntu',
                    defaults : {
                        graphOptions: {   }
                    }
                },
                properties: {}
            }
        },
        {
            label: 'set-bmc-credentials-graph',
            taskDefinition: {
                friendlyName: 'Run BMC Credential Graph',
                injectableName: 'Task.Graph.Run.Bmc',
                implementsTask: 'Task.Base.Graph.Run',
                options: {
                    graphName: 'Graph.Set.Bmc.Credentials',
                    defaults : {
                        graphOptions: {   }
                    }
                },
                properties: {}
            },
            waitOn: {
                'boot-graph': 'finished'
            }
        },
        {
            label: 'finish-bootstrap-trigger',
            taskName: 'Task.Trigger.Send.Finish',
            waitOn: {
                'set-bmc-credentials-graph': 'finished'
            }
        }
    ]
 };

To remove the BMC credentials, User can run the following workflow located at ``on-taskgraph/lib/graphs/bootstrap-bmc-credentials-remove-graph.js`` and can be posted using Postman or Curl command.

    POST:        http://server-ip:8080/api/1.1/workflows/

   add the below content in the json body for payload (example node identifier and username, password shown below)

.. code-block:: shell

   {
       "name": "Graph.Bootstrap.With.BMC.Credentials.Remove",
       "options": {
            "defaults": {
                "graphOptions": {
                    "target": "56e967f5b7a4085407da7898",
                    "remove-bmc-credentials": {
                        "users": ["7","8"]
                    }
                },
                "nodeId": "56e967f5b7a4085407da7898"
            }
        }
   }


Certificates
-------------------------

This section describes how to generate and install a self-signed certificate to use for testing.

Generating Self-Signed Certificates
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you already have a key and certificate, skip down to the
`Installing Certificates`_ section.

First, generate a new RSA key::

    openssl genrsa -out privkey.pem 2048


The file is output to *privkey.pem*. **Keep this private key secret. If it is
compromised, any corresponding certificate should be considered invalid.**

The next step is to generate a self-signed certificate using the private key::

    openssl req -new -x509 -key privkey.pem -out cacert.pem -days 9999

The *days* value is the number of days until the certificate expires.

When you run this command, OpenSSL prompts you for some metadata to associate with the new
certificate. The generated certificate contains the corresponding public key.

Installing Certificates
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Once you have your private key and certificate, you'll need to let the application know where to
find them. It is suggested that you move them into the /opt/monorail/data folder.

.. code-block:: bash

    mv privkey.pem /opt/monorail/data/mykey.pem
    mv cacert.pem /opt/monorail/data/mycert.pem

Then configure the paths by editing *httpsCert* and *httpKey* in
/opt/monorail/config.json. (See the `Configuration Parameters`_ section above).

If using a self-signed certificate, add a security exception to your client of
choice. Verify the certificate by restarting on-http and visiting
`https://<host>/api/current/versions`.

**Note:** For information about OpenSSL, see the `OpenSSL documentation`_.

.. _OpenSSL documentation: https://www.openssl.org/docs/


.. _http-endpoint-config-ref-label:

Setting up HTTP/HTTPS endpoint
------------------------------

This section describes how to setup HTTP/HTTPS endpoints in RackHD.
An endpoint is an instance of HTTP or HTTPS server that serves a group of APIs. Users can
choose to enable authentication or enable HTTPS for each endpoint.

There are currently two API groups defined in RackHD:

- the northbound-api-router API group. This is the API group that is used by users
- the southbound-api-router API group. This is the API group that is used by nodes
  interacting with the system

.. code-block:: JSON

    [
        {
            "address": "0.0.0.0",
            "port": 8443,
            "httpsEnabled": true,
            "httpsCert": "data/dev-cert.pem",
            "httpsKey": "data/dev-key.pem",
            "httpsPfx": null,
            "proxiesEnabled": false,
            "authEnabled": false,
            "routers": "northbound-api-router"
        },
        {
            "address": "172.31.128.1",
            "port": 9080,
            "httpsEnabled": false,
            "proxiesEnabled": true,
            "authEnabled": false,
            "routers": "southbound-api-router"
        }
    ]

.. list-table::
    :widths: 20 100
    :header-rows: 1

    * - Parameter
      - Description
    * - address
      - IP/Interface to bind to for HTTP. Typically this is '0.0.0.0'
    * - port
      - Local port to use for HTTP. Typically, port 80 for HTTP, 443 for HTTPS
    * - httpsEnabled
      - Toggle HTTPS
    * - httpsCert
      - Filename of the X.509 certificate to use for TLS. Expected format is PEM.
        This is optional and only takes effect when the httpsEnabled flag is set to true
    * - httpsKey
      - Filename of the RSA private key to use for TLS. Expected format is PEM.
        This is optional and only takes effect when the httpsEnabled flag is set to true
    * - httpsPfx
      - Pfx file containing the SSL cert and private key
        (only needed if the key and cert are omitted)
        This is optional and only takes effect when the httpsEnabled flag is set to true
    * - proxiesEnabled
      - A boolean value to toggle httpProxies (defaults to false)
    * - authEnabled
      - Toggle API Authentication
    * - routers
      - A single router name or a list of router names.
        This would only take effect for 1.1 APIs.
        You can now choose from "northbound-api-router","southbound-api-router" or
        ["northbound-api-router", "southbound-api-router"].

.. _authentication-config-ref-label:

Authentication
-------------------------

This section describes how to enable user authentication in RackHD.

Enable Authentication
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

As mentioned in the `Setting up HTTP/HTTPS endpoint`_ section, authentication can be enabled
or disabled per endpoint basis.

Setting the authEnabled flag to true in an endpoint configuration will enable authentication for
that specific endpoint.

.. code-block:: JSON

    {
        "address": "0.0.0.0",
        "port": 8443,
        "httpsEnabled": true,
        "proxiesEnabled": false,
        "authEnabled": true,
        "routers": "northbound-api-router"
    }

**Note**: although there is no limitation to enable authentication together with insecure HTTP
(httpsEnabled = false) for an endpoint, it is strongly not recommended to do so. Sending
user credentials over unencrypted HTTP connection exposes users to the risk of malicious attacks.

Setting up username and password
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Every time a request is sent an API route that needs authentication, a token needs to be sent with
the request. The token is returned from RackHD by posting a request to the /login API with a
username and password in the request body.

The default username and password is setup using the localhost exception mechanism described in
:ref:`localhost-exception-label`.

Setting up token
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

There are few settings needed for generating the token.


.. list-table::
    :widths: 20 100
    :header-rows: 1

    * - Parameter
      - Description
    * - authTokenSecret
      - The secret used to generate the token.
    * - authTokenExpireIn
      - The time interval in second after which the token will expire, since the time the
        token is generated.

        Token will never expire if this value is set to 0.

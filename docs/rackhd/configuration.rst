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
        "httpProxies": [{
            "localPath": "/coreos",
            "server": "http://stable.release.core-os.net",
            "remotePath": "/amd64-usr/current/"
        }],
        "httpStaticRoot": "/opt/monorail/static/http",
        "minLogLevel": 3,
        "authUsername": "admin",
        "authPasswordHash": "KcBN9YobNV0wdux8h0fKNqi4uoKCgGl/j8c6YGlG7iA0PB3P9ojbmANGhDlcSBE0iOTIsYsGbtSsbqP4wvsVcw==",
        "authPasswordSalt": "zlxkgxjvcFwm0M8sWaGojh25qNYO8tuNWUMN4xKPH93PidwkCAvaX2JItLA3p7BSCWIzkw4GwWuezoMvKf3UXg==",
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
        "minLogLevel": 2
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
      - Optional http proxies list. There are 3 parameters for each proxy:

        "localPath"/"remotePath" are optional and defaults to "/". A legal "localPath"/"remotePath" string must start with slash and ends without slash, like "/mirrors".
        If "localPath" is assigned to an existing local path like "/api/common/nodes", proxy won't work. Instead the path will keep its original feature and function.
        "server" is a must, both http and https servers are supported. A legal "server" string must ends without slash like "http://centos.eecs.wsu.edu". Instead "http://centos.eecs.wsu.edu/" is illegal.

        Example:

        { "server": "http://centos.eecs.wsu.edu", "localPath": "/centos" } would map http requests to local directory /centos/ to http://centos.eecs.wsu.edu/

        { "server": "https://centos.eecs.wsu.edu", "remotePath": "/centos" } would map http requests to local directory / to https://centos.eecs.wsu.edu/centos/
    * - httpFrontendDirectory
      - Fully-qualified directory to the web GUI content
    * - httpStaticDirectory
      - Fully-qualified directory to where static HTTP content is served
    * - maxTaskPayloadSize
      - Maximum payload size expected through TASK runner API callbacks from microkernel
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

A node gets discovered and BMC IPMI comes up with default username/password. For a user to add 
own username/password during discovery certain steps need to be followed:
First edit Sku Discovery graph located at ``on-taskgraph/lib/graphs/discovery-sku-graph.js``
to include a new graph **set-bmc-credentials-graph** located at ``on-taskgraph/lib/graphs/set-bmc-credentials-graph.js``
which runs the tasks to create a new user called 'monorail' with a randomly generated password and update obm settings
accordingly. 
Below is a snippet of the Sku Discovery graph which includes **set-bmc-credentials-graph** 

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
                    graphName: 'Graph.Set.Bmc',
                    graphOptions: {}
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
        },
    
  
- Edit **Discovery workflow graph** located at ``on-taskgraph/lib/graphs/discovery-graph.js``
to remove the reboot task. The reboot task is already included in the **set-bmc-credentials-graph** 
that was added to the **Sku Discovery graph** in the above step
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


Once the above steps are completed (edited and saved) the services need to be restarted:

.. code-block:: shell

    sudo service on-http start
    sudo service on-dhcp-proxy start
    sudo service on-syslog start
    sudo service on-taskgraph start
    sudo service on-tftp start

Once the services are restarted completely, running an ipmi command for user list should show the new user added.
"ipmitool user list" if running the command from within the node or 
" ipmitool -I lanplus -H ipaddress-of-node -U admin -P admin user list"

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
      - Toggle Proxies
    * - authEnabled
      - Toggle API Authentication
    * - routers
      - A single router name or a list of router names.
        This would only take effect for 1.1 APIs.
        You can now choose from "northbound-api-router","southbound-api-router" or 
        ["northbound-api-router", "southbound-api-router"].

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

Every time a request is sent an API route that needs authentication, a token needs to be send with
the request. The token is returned by RackHD by posting a request to the /login API with a
username and password in the request body.

The default username and password is setup in the config file.


.. list-table::
    :widths: 20 100
    :header-rows: 1

    * - Parameter
      - Description
    * - authUsername
      - The username to login. Defaults to admin
    * - authPasswordHash
      - The default password stored in the form of a hashed value, base64 coded. The default
        password to generate the hash is admin123.
    * - authPasswordSalt
      - The salt used to generate the password hash, base64 coded.


Change the default password
^^^^^^^^^^^^^^^^^^^^^^^^^^^

A new password hash is needed if user want to change the default password from 'admin123' to
something else.

**Step 1**. Copy following javascript code into a file with .js extension, take hash-gen.js for
example::

    var crypto = require('crypto');

    var password = 'admin123';//replace 'admin123' with the new password

    salt = crypto.randomBytes(64);
    console.log('salt = ', salt.toString('base64'));
    crypto.pbkdf2(password, salt, 10000, 64, function(err, hash){
        console.log('hash = ', hash.toString('base64'));
    });

Modify the content of password to any other string that the user picks.

**Step 2**. Run this script using nodejs::

    node hash-gen.js

A random salt and a hash will be generated. Following is an example::

    onrack@~/hash-gen> node hash-gen.js
    salt =  L2Wh7fqR5GDQTIIvKZ5qWGmPeMN/IpGEZOipyS5CDK0I+yUt4kY0X98ZS+HG8dp4K9LXiiGttk91alfJFvqk2g==
    hash =  b3n1vmbAKmEuLx0Cn/0X0hK2kYgGmcoTZgsn4SyLpjJftrbM0rhTaJ3CB3YZxw2Wopx51PtNG7SuDsw7jmh4IA==

**Step 3**. Replace authPasswordSalt and authPasswordHash with the salt and hash generated above
in the config.json. The new password will take effect after restarting RackHD.


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

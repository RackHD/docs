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
      "httpProxies":  [{"localPath": "/localPath_1", "server": "server_1", "remotePath": "/remotePath_1"},...,
                       {"localPath": "/localPath_N", "server": "server_N", "remotePath": "/remotePath_N"}]
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


Configuration Parameters
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The following table describes the configuration parameters in monorail.json.

===================== ===================================================================================
Parameter              Description
===================== ===================================================================================
amqp                   URI for accessing the AMQP interprocess communications channel
apiServerAddress       External facing IP address of the API server
apiServerPort          External facing port of the API server
dhcpGateway            Gateway IP for the network for DHCP
dhcpProxyBindAddress   IP for DHCP proxy server to bind  (defaults to '0.0.0.0'). **Note:** DHCP binds to 0.0.0.0 to support broadcast request/response within Node.js.
dhcpProxyBindPort      Port for DHCP proxy server to bind (defaults to 4011).
dhcpProxyOutPort       Port for DHCP proxy server to respond to legacy boot clients (defaults to 68).
dhcpProxyEFIOutPort    Port for DHCP proxy server to respond to EFI clients (defaults to 4011).
httpApiDocsDirectory   Fully-qualified directory containing the API docs.
httpEnabled            Toggle HTTP.
httpsEnabled           Toggle HTTPS.
httpBindAddress        IP/Interface to bind to for HTTP. Typically this is '0.0.0.0'
httpBindPort           Local port to use for HTTP. Typically, port 80
httpsBindPort          Local port to use for HTTPS. Typically, port 443.
httpsCert              Filename of the X.509 certificate to use for TLS. Expected format is PEM.
httpFileServiceRoot    Directory path for for storing uploaded files on disk.
httpFileServiceType    Backend storage mechanism for file service. Currently only FileSystem is supported.
httpProxies            Optional http proxies list. "localPath"/"remotePath" are optional ( defaults to "/").
                       "server" is a must for proxy, both http and https servers are supported.
                       http://<RackHD_Server_IP>:8080/localPath folder will be mapped to <server>/remotePath with httpProxies.
httpFrontendDirectory  Fully-qualified directory to the web GUI content
httpsKey               Filename of the RSA private key to use for TLS. Expected format is PEM.
httpsPfx               Pfx file containing the SSL cert and private key (only needed if the key and cert are omitted)
httpStaticDirectory    Fully-qualified directory to where static HTTP content is served
maxTaskPayloadSize     Maximum payload size expected through TASK runner API callbacks from microkernel
obmInitialDelay        Delay before retrying an OBM invocation
obmRetries             Number of retries to attempt before failing an OBM invocation
pollerCacheSize        Maximum poller entries to cache in memory
statsdPrefix           Application-specific *statsd* metrics for debugging
syslogBindPort         Port for syslog (defaults to 514).
syslogBindAddress      Address for the syslog server to bind to (defaults to '0.0.0.0').
tftpBindAddress        Address for TFTP server to bind to (defaults to '0.0.0.0').
tftpBindPort           Listening port for TFTP server  (defaults to 69).
tftpBindAddress        File root for TFTP server to serve files (defaults to './static/tftp').
tftproot               Fully-qualified directory from which static TFTP content is served
minLogLevel            A numerical value for filtering the logging from RackHD
==================== ===================================================================================

The log levels for filtering are defined at https://github.com/RackHD/on-core/blob/master/lib/common/constants.js#L36-L44

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
/opt/monorail/etc/monorail.json. (See the `Configuration Parameters`_ section above).

If using a self-signed certificate, add a security exception to your client of
choice. Verify the certificate by restarting on-http and visiting
`https://<host>/api/current/versions`.

**Note:** For information about OpenSSL, see the `OpenSSL documentation`_.

.. _OpenSSL documentation: https://www.openssl.org/docs/

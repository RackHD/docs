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

  Configuration Parameters
  ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

  The following table describes the configuration parameters in monorail.json.

  ===================== ===================================================================================
  Parameter              Description
  ===================== =============== ===================================================================
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
  ===================== ===================================================================================

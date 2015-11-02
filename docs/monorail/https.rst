HTTPS/TLS Configuration
------------------------------

To use TLS, a private RSA key and X.509 certificate must be provided. On Ubuntu and
Mac OS X, the openssl command line tool can be used to generate keys and certificates.

For internal development purposes, a self-signed certificate can be used. When using a self-signed
certificate, clients must manually include a rule to trust the certificate's authenticity.

By default, the application uses a self-signed certificate issued by Monorail which requires no
configuration. Custom certificates can also be used with some configuration.

Configuration
~~~~~~~~~~~~~~~~~~~~~~

The following options are present in /opt/onrack/etc/monorail.json to control the HTTP/HTTPS
server:

+---------------------+---------+------------------------------------------------------------------------------------+
| Name                |  Type   |  Description                                                                       |
+=====================+=========+====================================================================================+
| httpEnabled         | boolean | Toggle HTTP.                                                                       |
+---------------------+---------+------------------------------------------------------------------------------------+
| httpsEnabled        | boolean | Toggle HTTPS.                                                                      |
+---------------------+---------+------------------------------------------------------------------------------------+
| httpBindAddress     | string  | ip/interface to bind to for HTTP. Typically this is 0.0.0.0                        |
+---------------------+---------+------------------------------------------------------------------------------------+
| httpBindPort        | integer | Local port to use for HTTP. Typically this is port 80                              |
+---------------------+---------+------------------------------------------------------------------------------------+
| httpsBindPort       | integer | Local port to use for HTTPS. Typically this is port 443.                           |
+---------------------+---------+------------------------------------------------------------------------------------+
| httpsCert           | string  | Filename of the X.509 certificate to use for TLS. Expected format is PEM.          |
+---------------------+---------+------------------------------------------------------------------------------------+
| httpsKey            | string  | Filename of the RSA private key to use for TLS. Expected format is PEM.            |
+---------------------+---------+------------------------------------------------------------------------------------+
| apiServerAddress    | string  | The externally facing ip of the HTTP server, used by various services              |
+---------------------+---------+------------------------------------------------------------------------------------+
| apiServerPort       | integer | The externally facing port of the HTTP server, used by various services            |
+---------------------+---------+------------------------------------------------------------------------------------+
| httpFileServiceRoot | string  | Directory path for uploaded files to be stored on disk                             |
+---------------------+---------+------------------------------------------------------------------------------------+
| httpFileServiceType | string  | Backend storage mechanism for file service. Currently only FileSystem is supported |
+---------------------+---------+------------------------------------------------------------------------------------+

Generating Self-Signed Certificates
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This section demonstrates how to generate a self-signed certificate with personalized metadata.
If you already have a key and certificate, skip down to the
Installing Certificates section.

First, generate a new RSA key:

.. code-block:: bash

    openssl genrsa -out privkey.pem 2048


The file is output to *privkey.pem*. **Keep this private key secret. If it is
compromised, any corresponding certificate should be considered invalid.**

The next step is to generate a self-signed certificate using the private key:

.. code-block:: bash

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
/opt/monorail/etc/monorail.json (See Configuration section above).

If using a self-signed certificate, add a security exception to your client of
choice. Verify the certificate by restarting on-http and visiting
https://<host>/api/current/versions.

See Also
~~~~~~~~~~~~~~~~

More information is available in the OpenSSL documentation:

https://www.openssl.org/docs/

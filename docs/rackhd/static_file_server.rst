.. _static-file-server-label:

Static File Service Setup
---------------------------
.. _on-http: https://github.com/RackHD/on-http
.. _nginx: https://www.nginx.com/
.. _install nginx: https://www.nginx.com/resources/wiki/start/topics/tutorials/install/
.. _nginx_conf: https://www.nginx.com/resources/wiki/start/topics/examples/full/
.. _config.json: https://github.com/RackHD/RackHD/blob/master/packer/ansible/roles/monorail/files/config.json
.. _httpProxies: http://rackhd.readthedocs.io/en/latest/rackhd/configuration.html?highlight=httpProxies
.. _httpStaticRoot: http://rackhd.readthedocs.io/en/latest/rackhd/configuration.html?highlight=httpStaticRoot


There are two kinds of static files in RackHD, and this section introduces a mechanism to move one type of
them (files for discovery and OS installation) to a separate third-party service in order to offload the burden
of file transmission in RackHD.

Files That can be Moved into a Separate Server
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Some files, including schema, swagger configuration and others, interacts closely with RackHD, and are part of
its functionalities. Others are served for node discovery and OS installation (if users put OS image under the
same static file directory).  `on-http`_ manages all the files mentioned above by default, and the latter
(**files for discovery and OS installation**) can be moved to a third-party static file server, which will be
discussed below.

Diagrams for Different Working Modes
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
RackHD supports three modes to serve static files.

Legacy Mode: nodes get static files from `on-http`_ service (default).
Single-Host Mode: nodes get static files from another service in the same host as RackHD.
Multi-Host Mode: nodes get static files from different host.
This chapter introduces the settings for the last two modes.

.. image:: ../_static/static_server_mode.png
    :height: 250
    :align: center

Prerequisites
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
- Static file server uses a static IP address.
- The server can be accessed by nodes.
- If the server is in the same subnet of DHCP server, to avoid IP conflict, the range of DHCP server should not cover the IP of static file server.

Steps to Setup a Static File Server
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Configure a Third-Party Static File Server
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Since RackHD doesn't require any customization on a file server, users could adopt any frameworks they are
familiar with. Here takes `nginx`_ as an example about the configuration.

After `install nginx`_, modify `nginx_conf`_ to make sure the following configuration works.

.. code-block:: shell

    http {
        server {
            listen 3000;
            sendfile on;

            location / {
                root /home/onrack/;
            }
        }
    }

"3000" is the port for the server; "location" is the URI root path to access static files; and "root" specifies
the directory that will be used to search for files.

Restart nginx server after the new configuration.

Configure the Path of Static File Server in RackHD
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In `config.json`_, add the following fields:

.. code-block:: shell

    ...
    "fileServerAddress": "172.31.128.3",
    "fileServerPort": 3000,
    "fileServerPath": '/',
    ...

"fileServerAddress" is the IP address of static file server that nodes can access.
"fileServerPort" is the port the server is listening to (*optional*, the default value is 80).
"fileServerPath" is the "location" in server configuration (*optional*, the default value is '/').

Restart RackHD services after adding these fields.

**Note**: fileServer configuration takes higher priority than `httpProxies`_ and 'httpStaticRoot'_. which means that when above fields exists, 
RackHD will use file server address for static files and ignore those specified in the above two fields.

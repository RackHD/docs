TFTP and DHCP Service Setup
---------------------------
.. _dhcpd.conf: http://linux.die.net/man/5/dhcpd.conf
.. _ISC DHCP Server: https://www.isc.org/downloads/dhcp
.. _on-tftp: https://github.com/RackHD/on-tftp
.. _on-dhcp-proxy: https://github.com/RackHD/on-dhcp-proxy
.. _on-imagebuilder: https://github.com/RackHD/on-imagebuilder

.. _config.json: https://github.com/RackHD/RackHD/blob/master/packer/ansible/roles/monorail/files/config.json
.. _RackHD iPXE: https://bintray.com/rackhd/binary/on-imagebuilder#files/ipxe

RackHD is flexiable to adapt to different network environments for TFTP and DHCP service.
By default, RackHD use `on-tftp`_ for TFTP service, `ISC DHCP Server`_ and DHCP proxy `on-dhcp-proxy`_ for DHCP service,
and they are deployed in RackHD server along with other RackHD service `on-http`, `on-taskgraph`, `on-syslog`.
They could be replaced with other TFTP and DHCP services, and also could be deployed to another separated server.

+---------------------------+---------------------------------------------+-------------------------------------------------+
| Cases                     |            Supported TFTP Service           |             Supported DHCP Service              |
+===========================+=============================================+=================================================+
| TFTP and DHCP service are | 1. on-tftp(default)                         | 1. ISC DHCP + on-dhcp-proxy(default)            |
| in RackHD server          | 2. Third-party TFTP service such as         | 2. ISC DHCP only                                |
|                           |    in.tftpd(tftp-hpa) in Ubuntu OS          | 3. Third-party DHCP Service + DHCP proxy        |
|                           |                                             | 4. Third-party DHCP Service only                |
+---------------------------+---------------------------------------------+-------------------------------------------------+
| TFTP and DHCP service are | 1. on-tftp                                  | 1. ISC DHCP + on-dhcp-proxy                     |
| in separate server        | 2. Third-party TFTP service such as         | 2. ISC DHCP only                                |
|                           |    in.tftpd(tftp-hpa) in Ubuntu OS          | 3. Third-party DHCP Service + DHCP proxy        |
|                           |                                             | 4. Third-party DHCP Service only                |
+---------------------------+---------------------------------------------+-------------------------------------------------+

**NOTE:** "Third-party" service means it's not RackHD default service.

TFTP and DHCP are in RackHD Server
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

TFTP Service Configuration in RackHD Server
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Default on-tftp Configuration
""""""""""""""""""""""""""""""""""""""""""""""""""""""

RackHD default TFTP service is `on-tftp`_, it could be configured by fields `tftpBindAddress`, `tftpBindPort`, `tftpRoot`
in `config.json`_, and `RackHD iPXE`_ files are put to `tftpRoot` directory.

.. code-block:: javascript

    ...
    "tftpBindAddress": "172.31.128.1",
    "tftpBindPort": 69,
    "tftpRoot": "./static/tftp",
    ...

Third-Party TFTP Service Configuration
""""""""""""""""""""""""""""""""""""""

.. _RackHD TFTP Templates: https://github.com/RackHD/on-tftp/tree/master/data/templates

Any other TFTP service could be used by RackHD, just put `RackHD iPXE`_ files to TFTP root directory.
For scripts in `RackHD TFTP Templates`_, if the parameters such as `apiServerAddress`, `apiServerPort` are rendered by `on-tftp`,
they need to be hardcoded, it's `172.31.128.1` and `9080` in the example. then put these scripts to TFTP root directory.

**NOTE**:
  1. If all managed nodes' NIC ROM is iPXE, not PXE, don't need to put `RackHD iPXE`_ files to TFTP directory.
  2. If the funtionality supported by rendered scripts are not needed, don't need to put `RackHD TFTP Templates`_ scripts to TFTP directory.
  3. If both cases above are satisfied, TFTP service is not needed by RackHD.


DHCP Service Configuration in RackHD Server
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The DHCP protocol is a critical component to the PXE boot process and for
executing various profiles and :doc:`graphs` within RackHD.

By default RackHD deploys a DHCP configuration that forwards DHCP clients to the
on-dhcp-proxy service, see :doc:`../architecture` for more information.
However conventional DHCP configurations that require static (and/or dynamic)
IP lease reservations is also supported, bypassing the on-dhcp-proxy service
all together.

There are various Linux DHCP Server versions out there, RackHD has been primarily
validated against `ISC DHCP Server`_. As long as the DHCP server supports the required
DHCP configuration options then those versions should be compatible as well.

Default ISC DHCP + on-dhcp-proxy Configuration
""""""""""""""""""""""""""""""""""""""""""""""

The advantage of using the on-dhcp-proxy service is to avoid complicated DHCP server setup,
most logics are handled in on-dhcp-proxy, it's convinient and flexiable,
but a typical simple `dhcpd.conf`_ of `ISC DHCP Server`_ for forwarding DHCP request to RackHD's
on-dhcp-proxy service is needed, it would look like the following:

.. literalinclude:: samples/dhcpd.conf

.. _subnet: https://en.wikipedia.org/wiki/Subnetwork
.. _range: https://en.wikipedia.org/wiki/Dynamic_Host_Configuration_Protocol#Overview
.. _netmask: https://en.wikipedia.org/wiki/Subnetwork#Network_addressing_and_routing

Subsituting the `subnet`_, `range`_ and `netmask`_ to match your desired networking configuration.

To enforce lease assignment based on MAC and not UID we opt-in to ignore the UID in the request
by setting **ignore-client-uids true**.

ISC DHCP Only Configuration
"""""""""""""""""""""""""""

When bypassing on-dhcp-proxy step, the DHCP server can also define static host definitions.

Using `ISC DHCP Server`_, the `dhcpd.conf`_ for invoking RackHD lookup directly
and handling of static host definitions would look like the following:

.. literalinclude:: samples/dhcpd.conf.noproxy

In the global subnet definition we define a PXE chainloading setup to handle specific
client requests.

.. code-block:: shell

    if ((exists user-class) and (option user-class = "MonoRail")) {
        ...
    } else {
        ...
    }

If the request is made from a BIOS/UEFI PXE client, the DHCP server will hand out the
iPXE bootloader image that corresponds to the system's architecture type.

.. code-block:: shell

    if ((exists user-class) and (option user-class = "MonoRail")) {
        filename "http://172.31.128.1:9080/api/common/profiles";
    } else {
        if option arch-type = 00:09 {
          filename "monorail-efi64-snponly.efi";
        } elsif option arch-type = 00:07 {
          filename "monorail-efi64-snponly.efi";
        } elsif option arch-type = 00:06 {
          filename "monorail-efi32-snponly.efi";
        } else {
          filename "monorail.ipxe";
        }
    }

If the request is made from the RackHD iPXE client, the DHCP server will chainload
another boot configuration pointed at RackHD's profiles API.

Third-Party DHCP Service Configuration
""""""""""""""""""""""""""""""""""""""

.. _Default iPXE Config: https://github.com/RackHD/on-imagebuilder/blob/master/roles/ipxe/files/default.ipxe

In theory, Other DHCP services are also supported by RackHD, but it's not validated yet, possible solutions are as below:


+-----------------+--------------------------------------------------------------+------------------------------------------------------------+
| Service         | Cases                                                        | Solutions                                                  |
+=================+==============================================================+============================================================+
|                 | DHCP service has functionalities like ISC DHCP, it could     | Configure it like ISC DHCP to make node auto chainloading  |
|                 | configure DHCP to return different bootfile name according   | iPXE files and finally iPXE hit RackHD URL                 |
|                 | to `user-class`, `arch-type`, `vendor-class-identifier` etc. | ``http://172.31.128.1:9080/api/common/profiles``           |
|                 |                                                              | IP address and port are configured according to RackHD     |
| Third-party     |                                                              | southbound configuration.                                  |
| DHCP service    +--------------------------------------------------------------+------------------------------------------------------------+
| only            | DHCP service could not proxy DHCP, and on-dhcp-proxy         | Replace "autoboot" command in `Default iPXE Config`_ with  |
|                 | also could not be deployed in the DHCP server, only bootfile | "dhcp" and "http://172.31.128.1:9080/api/common/profiles", |
|                 | name could be specified by DHCP                              | then re-compile iPXE in `on-imagebuilder`_ to generate new |
|                 |                                                              | iPXE files, specify one of generated iPXE files as         |
|                 |                                                              | as bootfile name in DHCP configuration. IP address and     |
|                 |                                                              | port are configured according to RackHD southbound         |
|                 |                                                              | configuration. Two drawbacks for this solution due to      |
|                 |                                                              | DHCP and environment limitations:                          |
|                 |                                                              | 1. IP address and port are hardcoded in iPXE file          |
|                 |                                                              | 2. Only one iPXE bootfile name could be specified. it's    |
|                 |                                                              | not flexiable to switch bootfile name automatically.       |
+-----------------+--------------------------------------------------------------+------------------------------------------------------------+
| Third-party     | DHCP service's funtionality is less than ISC DHCP,           | **on-dhcp-proxy** could be leveraged to avoid complicated  |
| DHCP service    | but it could proxy DHCP like ISC DHCP's configuration        | DHCP configuration.                                        |
| + DHCP proxy    | "option vendor-class-identifier "PXEClient"                  |                                                            |
+-----------------+--------------------------------------------------------------+------------------------------------------------------------+


TFTP and DHCP are in Separated Server
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

RackHD default TFTP and DHCP service such as on-tftp, on-dhcp-proxy and ISC DHCP could be deployed
in another separated server after some simple configurations.

RackHD also could disable its own TFTP and DHCP service, and leverage users' TFTP and DHCP server
inpi/common/profiles datacenter or lab environments.

When TFTP and DHCP are installed in another separated server, both RackHD server and TFTP/DHCP server need to be setup.
**NOTE**: Below examples assume that TFTP and DHCP server IP address is **172.31.128.1**, and RackHD server IP address
is **172.31.128.2**

RackHD Main Services Configuration in RackHD Server
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In RackHD server, ``/opt/monorail/config.json`` is updated with below settings, then restart `on-http`, `on-taskgraph` and `on-syslog`
services.

.. code-block:: javascript

    ...
    "apiServerAddress": "172.31.128.2",
    ...
    "syslogBindAddress": "172.31.128.2"
    ...
    "dhcpGateway": "172.31.128.1",
    "dhcpProxyBindAddress": "172.31.128.1",
    ...
    "tftpBindAddress": "172.31.128.1",
    ...
    "httpEndpoints": [
        ...
        {
            ...
            "address": "172.31.128.2",
            ...
        },
        ...
    ]
    ...


TFTP Service Configuration in Separated Server
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Default on-tftp Configuration
""""""""""""""""""""""""""""""""""
``/opt/monorail/config.json`` need to be updated with below settings, then restart `on-tftp`.

.. code-block:: javascript

    ...
    "apiServerAddress": "172.31.128.2",
    ...
    "syslogBindAddress": "172.31.128.2"
    ...
    "dhcpGateway": "172.31.128.1",
    "dhcpProxyBindAddress": "172.31.128.1",
    ...
    "tftpBindAddress": "172.31.128.1",
    ...
    "httpEndpoints": [
        ...
        {
            ...
            "address": "172.31.128.2",
            ...
        },
        ...
    ]
    ...


Third-Party TFTP Service Configuration
""""""""""""""""""""""""""""""""""""""
Other TFTP service setup in separated server is the same with in RackHD server.
`RackHD TFTP Templates`_ scripts's rendered parameters `apiServerAddress`, `apiServerPort` is `172.31.128.2`, `9080` in the example.

DHCP Service Configuration in Separated Server
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Default ISC DHCP + on-dhcp-proxy Configuration
""""""""""""""""""""""""""""""""""""""""""""""

ISC DHCP **dhcpd.conf** need to be updated with below settings, then restart ISC DHCP.
**NOTE**: DHCP ip addresses range starts from **172.31.128.3**, because **172.31.128.2** is assigned to RackHD server.

.. literalinclude:: samples/dhcpd-non-rackhd.conf


``/opt/monorail/config.json`` need to be updated with below settings, then restart `on-dhcp-proxy`.

.. code-block:: javascript

    ...
    "apiServerAddress": "172.31.128.2",
    ...
    "syslogBindAddress": "172.31.128.2"
    ...
    "dhcpGateway": "172.31.128.1",
    "dhcpProxyBindAddress": "172.31.128.1",
    ...
    "tftpBindAddress": "172.31.128.1",
    ...
    "httpEndpoints": [
        ...
        {
            ...
            "address": "172.31.128.2",
            ...
        },
        ...
    ]
    ...


ISC DHCP Only Configuration
"""""""""""""""""""""""""""

ISC DHCP **dhcpd.conf** need to be updated with below settings, then restart ISC DHCP.
**NOTE**: DHCP ip addresses range starts from **172.31.128.3**, because **172.31.128.2** is assigned to RackHD server.

.. literalinclude:: samples/dhcpd-non-rackhd.conf.noproxy


Third-Party DHCP Service Configuration
""""""""""""""""""""""""""""""""""""""

The solutions of using other DHCP services in separated server are the same with in RackHD server. Just need to specify
RackHD southbound IP address and port in DHCP configuration to `172.31.128.2`, `9080` in the example.



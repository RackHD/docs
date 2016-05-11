DHCP Server Setup
------------------
.. _dhcpd.conf: http://linux.die.net/man/5/dhcpd.conf
.. _ISC DHCP Server: https://www.isc.org/downloads/dhcp

The DHCP protocol is a critical component to the PXE boot process and for 
executing various profiles and :doc:`graphs` within RackHD.

By default RackHD deploys a DHCP configuration that forwards DHCP clients to the 
on-dhcp-proxy service, see :doc:`../software_architecture` for more information. 
However conventional DHCP configurations that require static (and/or dynamic) 
IP lease reservations is also supported, bypassing the on-dhcp-proxy service
all together.

There are various Linux DHCP Server versions out there, RackHD has been primarily
validated against `ISC DHCP Server`_. As long as the DHCP server supports the required 
DHCP configuration options then those versions should be compatible as well.

DHCP-Proxy Configuration
~~~~~~~~~~~~~~~~~~~~~~~~~

The advantage to using the on-dhcp-proxy service is to make the DHCP requests visible to 
RackHD's MAC/IP lookup subsystem. Therefore IP leases that expire/renew with different IP
addresses will be seen by RackHD seamlessly updating OBM service host lookups.

Using `ISC DHCP Server`_, a typical `dhcpd.conf`_ for forwarding DHCP request to RackHD's
on-dhcp-proxy service would look like the following:

.. literalinclude:: samples/dhcpd.conf

.. _subnet: https://en.wikipedia.org/wiki/Subnetwork
.. _range: https://en.wikipedia.org/wiki/Dynamic_Host_Configuration_Protocol#Overview
.. _netmask: https://en.wikipedia.org/wiki/Subnetwork#Network_addressing_and_routing

Subsituting the `subnet`_, `range`_ and `netmask`_ to match your desired networking configuration.

To enforce lease assignment based on MAC and not UID we opt-in to ignore the UID in the request
by setting **ignore-client-uids true**.

Static DHCP Configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

When bypassing on-dhcp-proxy step, the DHCP server can also define static host definitions.

First ensure that the DHCP leased cache service poller is disabled in on-taskgraph. This can be done by setting
**serviceGraph** to **false** in ``on-taskgraph/lib/graphs/isc-dhcp-poller-service-graph.js``.

.. code-block:: javascript

    module.exports = {
        friendlyName: 'isc-dhcp leases poller',
        injectableName: 'Graph.Service.IscDhcpLeasePoller',
        serviceGraph: false,
        options: {
            'isc-dhcp-lease-poller': {
                schedulerOverrides: {
                    timeout: -1
                }
            }
        },
        tasks: [
            {
                label: 'isc-dhcp-lease-poller',
                taskName: 'Task.IscDhcpLeasePoller'
            }
        ]
    };

Once the DHCP lease poller graph is edited and saved, restart the on-taskgraph service:

.. code-block:: shell

    service on-taskgraph restart

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
        filename "http://172.31.128.1:9080/api/common/profiles?mac=${net0/mac}&&ip=${net0/ip}";
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
another boot configuration pointed at RackHD's profiles API. The profiles API will receive
both the current MAC and leased IP address as query parameters. If the lookup is new to RackHD
it will store it and prepare for any subsequent workflow/discovery request.

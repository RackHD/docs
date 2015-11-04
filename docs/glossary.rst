Glossary
========

PXE
----

Most datacenter computers, and some desktop computers, have a mechanism to allow them to
start up and inquire on a network for what they should be running, called `Preboot Execution Environment`_ (PXE).

.. _Preboot Execution Environment: https://en.m.wikipedia.org/wiki/Preboot_Execution_Environment

This mechanism was originally developed by Intel in 1998/1999, leveraging DHCP and TFTP
protocols to have a remote system provide IP address information and provide the files
that the computer would execute. PXE was embraced by most vendors, originally provided
through the network interface cards, but now built into most BIOS or UEFI implementations,
supporting network booting.

PXE has since been used to run diskless computers and common Linux operating systems have
all adopted a network based install that can be initiated and leveraged by PXE - RedHat's
kickstart, Debian's debseed, and SUSE's YaST.

TFTP
----

One of the earliest protocols, TFTP provides simple, unauthenticated file transfer
protocol over TCP/IP. The protocol itself is quite simple, hence it's adoption as a
common mechanism to remote boot or transfer files for remote installation. Because of
it's simplicity, the protocol is also not terribly robust in the face of failures or
temporary network outages, and can be somewhat unreliable at scale or high load.

bootloaders
-----------

TFTP used is to transfer tiny executable programs that computers use to initialize
hardware and set up additional systems in order to "boot and run" a larger operating
system. pxelinux_ and iPXE_ (evolved from earlier gPXE) are mostly commonly used. PXElinux
heavily leverages TFTP and is a fairly static system, where iPXE includes a small
scripting interpretter and supports downloading additional files for booting (such
as a WinPE or Kernel and Initrd file for Linux) over HTTP as a more reliable transport
protocol.

.. _prelinux: http://www.syslinux.org/wiki/index.php/Doc/pxelinux
.. _iPXE: http://ipxe.org

DHCP
----

DHCP is an expansive protocol with many extensions that's leveraged to provide network
configuration information on request to other computers on the same "broadcast"
segment. As such, it relies on operating on raw sockets and TCP/IP broadcasts.
Many excellent DHCP servers exist, and as a protocol it is a common mainstay of
networks in the home and in data centers. Because DHCP services operate at Layer 2
on the networking stack, they are carefully controlled as two DHCP servers can
provide conflicting information on the same network broadcast segment (and this can
be obnoxiously difficult to diagnose). Likewise, most switches won't pass DHCP
traffic across networks unless specifically configured to do so with a DHCP relay.

The PXE mechanisms add additional information, and conditional responses, based on
the DHCP client requesting information. As PXE gained traction, the DHCP protocol
was extended to allow that additional information to come from a separate source
with a mechanism called DHCP Proxy, so that PXE could be configured and managed
independently of a DHCP infrastructure.

When set up in a datacenter environment, many PXE environments are configured with
DHCP, TFTP together. As such, if a DHCP proxy is used, it's often on the same
physical OS as the DHCP service being provided, but the protocol does allow and
support a DHCP proxy server completely independent of a main DHCP service.


BMC
---

In addition to PXE booting, most server hardware meant to be used in the data center
also includes a secondary computer that manages the main computer - called a
`Baseboard Motherboard Controller`_ (BMC).

.. _Baseboard Motherboard Controller: https://en.m.wikipedia.org/wiki/Baseboard_management_controller

BMC's connect to or control portions of servers, but don't generally have full access
to the computer they're managing. Most typically they have access to power, sensors,
and some amount of control over the controlled computer can boot. BMC's are network
connected sharing an Ethernet port (but having it's own mac-address) with the computer
being managed, having it's own dedicated ethernet port, or supporting both, configurable
within the BIOS or UEFI on which are enabled.

Desktop hardware (like the PC under your desk, or your laptop) and some of the early
OCP servers don't include BMC's to keep down the cost of those devices - and remote
power control isn't as necessary.

Note: Intel has recently been releasing desktops and client systems with a roughly
equivalent management tooling called `Active Management Technology`_ (AMT)). AMT uses it's
own protocol, supporting the DMTF standard DASH, running over HTTP/HTTPS and genearlly
leveraging the `WS-Management`_ server management standards.

.. _Active Management Technology: https://en.m.wikipedia.org/wiki/Intel_Active_Management_Technology
.. _WS-Management: https://en.m.wikipedia.org/wiki/WS-Management

BMCs primarily provide an interface using a protocol called IPMI, and many these
days also provide a web based interface that adds additional proprietary features,
the most common of which are a virtual keyboard/mouse and console, and the capability
to "remote mount" ISO files or other remote media.

IPMI
----

BMCs typically communicate on the network using the IPMI_ protocol.

.. _IPMI: https://en.m.wikipedia.org/wiki/Intelligent_Platform_Management_Interface

IPMI is a binary protocol with some authentication, although many security researchers
have shown the security properties of IPMI to be quite weak and easily exploited.

* `A Penetration Tester's Guide to IPMI and BMCs`_
* `Many servers expose insecure out-of-band management interfaces to the Internet`_
* `IPMI The most dangerous protocol you've never heard of`_


.. _A Penetration Tester's Guide to IPMI and BMCs: https://community.rapid7.com/community/metasploit/blog/2013/07/02/a-penetration-testers-guide-to-ipmi
.. _Many servers expose insecure out-of-band management interfaces to the Internet: http://www.pcworld.com/article/2361040/many-servers-expose-insecure-outofband-management-interfaces-to-the-internet.html
.. _IPMI The most dangerous protocol you've never heard of: http://www.itworld.com/article/2708437/security/ipmi--the-most-dangerous-protocol-you-ve-never-heard-of.html


While generally considered insecure by security experts, it's still the default
implemented standard. As such, most data center networks secure and highly control
the access to networks where IPMI is enabled to control remote machines, often in
a private "out of band management" network.

In addition to the IPMI standard, many hardware vendors also include proprietary
extensions to the protocol to support additional information or functionality
specific to their platform. Some proprietary management tools leverage these
protocols to provide additional vendor specific functionality.

SNMP
----

Network Switches have long supported their own management tooling, based on the SNMP
protocol. Switches booting and loading software automatically was one of the last
pieces to get enabled, mostly because everything else relied on the networks to
exist and be functional in order to set up their automation. In many respects, the
physical networks and switches connecting them, are the lowest layer of the pyramid
upon which our services are built.

As BMCs have evolved, many are now fully computers on their own, often based on low
power ARM processors and running a linux based OS, and some support retrieving
information about the BMC and hardware it manages through SNMP as well.

ZTP/ONIE
--------

Some switches are now supporting self-configuration and automatic installation from
other devices, using protocols such a ZTP (Juniper, Arista_)
or ONIE_ (Cumulus). Some Cisco switches and routers support ZTP or their own proprietary protocol
"SmartInstall_".

.. _Arista: https://www.arista.com/en/products/eos/automation
.. _INIE: http://www.onie.org
.. _SmartInstall: http://www.cisco.com/c/en/us/products/collateral/switches/catalyst-3750-x-series-switches/white_paper_c11-651895.html

All of these systems basically operate similarly to the PXE protocol - leveraging
TFTP for the transfer of files, and DHCP for network information, but a common
standard adopted by all switch vendors has yet to emerge.

Likewise, cross-vendor protocols to control and manage switches have yet to emerge,
with switch vendors supporting either proprietary protocols and standards tied
closely to their feature sets. The recent advances in software defined
networking (SDN_) has started
to drive commonalities out into the market, but the current state is far from
a cohesive standard.

.. _SDN: https://www.opennetworking.org/sdn-resources/sdn-definition

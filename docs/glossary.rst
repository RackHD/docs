Glossary
========



**BMC**

A `Baseboard Motherboard Controller`_ (BMC) is a microcontroller (small computer) embedded on the motherboard of
data center servers. The BMC provides the interface that enables out-of-band management of the server using IPMI.

.. _Baseboard Motherboard Controller: https://en.m.wikipedia.org/wiki/Baseboard_management_controller

Some BMCs provide a web-based interface that adds additional proprietary features,
such as the ability to remotely mount ISO files or other media.

Because the BMC has a separate MAC address, it can be connected to the network using a dedicated or shared Ethernet port.

Note: Intel has recently been released desktops and client systems with roughly
equivalent management tooling called `Active Management Technology`_ (AMT). AMT uses its
own protocol that supports the DMTF standard DASH. It runs over HTTP/HTTPS and generally
leverages the `WS-Management`_ server management standards.

.. _Active Management Technology: https://en.m.wikipedia.org/wiki/Intel_Active_Management_Technology
.. _WS-Management: https://en.m.wikipedia.org/wiki/WS-Management



**Bootloaders**

RackHD uses TFTP to transfer tiny executable programs that are used to initialize
hardware and set up additional systems in order to "boot and run" a larger operating
system. PXELINUX_ and iPXE_ (evolved from earlier gPXE) are most commonly used.

PXElinux heavily leverages TFTP and is a fairly static system. iPXE includes a small
scripting interpreter and supports downloading additional files for booting (such
as a WinPE or Kernel and Initrd file for Linux) over HTTP as a more reliable transport
protocol.

.. _PXELINUX: http://www.syslinux.org/wiki/index.php/Doc/pxelinux
.. _iPXE: http://ipxe.org

**DHCP**

DHCP is an expansive protocol with many extensions that are leveraged to provide network
configuration information to other computers on the same "broadcast"
segment. It communicates using raw sockets or TCP/IP.

Because DHCP services operate at Layer 2 of the networking stack, they must be carefully controlled so as to prevent
the generation of IP address conflicts on the same network segment. Most switches do not pass DHCP
traffic across networks unless specifically configured to do so with a DHCP relay.

RackHD uses a DHCP Proxy server to support PXE operations. It sends auxiliary boot information to clients, like the boot filename, tftp server
or rootpath, but leaves generation of IP addresses to the DHCP server.



**IPMI**

The `Intelligent Platform Management Interface (IPMI)`_ is the protocol by which BMCs can manage and monitor servers independent of
the CPU, firmware (BIOS or UEFI), and operating system.
BMCs typically communicate on the network using the IPMI_ protocol.

.. _Intelligent Platform Management Interface (IPMI): https://en.m.wikipedia.org/wiki/Intelligent_Platform_Management_Interface

Although IPMI supports authentication, many security researchers
have shown that IPMI is easily exploited:

* `A Penetration Tester's Guide to IPMI and BMCs`_
* `Many servers expose insecure out-of-band management interfaces to the Internet`_
* `IPMI The most dangerous protocol you've never heard of`_


.. _A Penetration Tester's Guide to IPMI and BMCs: https://community.rapid7.com/community/metasploit/blog/2013/07/02/a-penetration-testers-guide-to-ipmi
.. _Many servers expose insecure out-of-band management interfaces to the Internet: http://www.pcworld.com/article/2361040/many-servers-expose-insecure-outofband-management-interfaces-to-the-internet.html
.. _IPMI The most dangerous protocol you've never heard of: http://www.itworld.com/article/2708437/security/ipmi--the-most-dangerous-protocol-you-ve-never-heard-of.html


Due to security weaknesses, most data center networks secure and highly control
the access to networks where IPMI is enabled.

Many hardware vendors provide proprietary DHCP extensions that support additional information or functionality.
Some proprietary management tools leverage these protocols to provide additional vendor-specific functionality.

**PXE**

 The `Preboot Execution Environment (PXE)`_ is a vendor-independent mechanism that allows networked computers
to be remotely booted and configured.

.. _Preboot Execution Environment: https://en.m.wikipedia.org/wiki/Preboot_Execution_Environment

PXE was originally developed by Intel in 1998/1999. Once provided through the network interface cards, it
is now built into most BIOS or UEFI implementations.

PXE has since been used to run diskless computers. Common Linux operating systems (Red Hat kickstart, Debian debseed,
and SUSE YaST) have adopted a network-based install that can be initiated and leveraged by PXE.

**SNMP**

Network Switches have long supported their own management tooling based on the SNMP
protocol. Switches booting and loading software automatically was one of the last
pieces to get enabled, mostly because everything else relied on the networks to
exist and be functional in order to set up their automation. In many respects, the
physical networks and switches connecting them, are the lowest layer of the pyramid
upon which our services are built.

As BMCs have evolved, many are now fully computers on their own, often based on low
power ARM processors and running a Linux-based OS, and some support retrieving
information about the BMC and hardware it manages through SNMP as well.

**TFTP**

TFTP provides simple, unauthenticated file transfer over TCP/IP. It is widely used  protocol itself is as a
 mechanism to remote boot or transfer files for remote installation.

 Due to the simplicity of TFTP, it is not terribly robust in the face of failures or
temporary network outages -- and can be somewhat unreliable at scale or high load.


**ZTP/ONIE**

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

Likewise, cross-vendor protocols that control and manage switches have yet to emerge,
with switch vendors supporting proprietary protocols and standards that are tied
closely to their feature sets. The recent advances in software-defined
networking (SDN_) has started
to drive commonalities out into the market, but the current state is far from
a cohesive standard.

.. _SDN: https://www.opennetworking.org/sdn-resources/sdn-definition

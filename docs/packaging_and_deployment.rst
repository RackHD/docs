Packaging and Deployment
=======================================

RackHD can use a number of different mechanisms to coordinate and control
bare metal hardware, and in the most common cases, a deployment is working with
at least two networks, connected on different network interface cards, to the
RackHD instance.

RackHD can be configured to work with a single network, or several more networks,
depending on the needs of the installation. The key elements to designing a RackHD
installation are:

- understanding what network `security constraints`_ you are using
- understanding the `hardware controls`_ you're managing and how it can be configured
- understanding where and how `IP address management`_ is to be handled in each of the networks
  that the first two items mandate.

.. image:: _static/vagrant_setup.jpg
   :height: 200
   :align: right

At a minimum, RackHD expects a "southbound" network, where it is interacting with
the machines it is PXE booting a network provided with DHCP, TFTP, and HTTP and
a "northbound" network where RackHD exposes the APIs for automation and interaction.
This basic setup was created to allow and encourage separation of traffic for
PXE booting nodes and API controls. The example setup in :doc:`getting_started`
shows a minimal configuration.

Security Constraints
----------------------

RackHD as a technology is configured to control and automate hardware, which implies
a number of natural security concerns. As a service, it provides an API control
endpoint, which in turn uses protocols on networks relevant to the hardware it's
managing. One of the most common of those protocols is `IPMI`_, which has
`known security flaws`_, but is used because it's one of the most common mechanisms to
control datacenter servers.

A relatively common requirement in datacenters is that networks used for IPMI traffic
are isolated from other networks, to limit the vectors by which IPMI endpoints
could be attacked. When RackHD is using IPMI, it simply needs to have L3 (routed IP)
network traffic to the relevant endpoints in order for the workflow engine and
various controls to operate.

Access to IPMI endpoints on hardware can be separated off onto it's own network, or
combined with other networks. It is generally considered best practice to separate
this network entirely, or constrain it to highly controlled networks where access
is strictly limited.

.. _IPMI: https://en.wikipedia.org/wiki/Intelligent_Platform_Management_Interface
.. _known security flaws: https://community.rapid7.com/community/metasploit/blog/2013/07/02/a-penetration-testers-guide-to-ipmi

Hardware Controls
------------------

.. sidebar:: KCS and controlling the BMC

    Most Intel servers with BMCs include a "KCS" (Keyboard Controller Style)
    communications channel between the motherboard and the BMC. This
    allows communications between the motherboard and the BMC, where the
    software running on the main computer can interrogate and configure the BMC.

    Software tools such a IPMItool on Linux can leverage this interface, which
    shows up as a kernel device.

    RackHD is configured to use and leverage this interface by default to
    interrogate the BMC and provide information about it's settings to RackHD.
    It can also be used by workflows set values for the BMC. If the server you
    are working with does not have a BMC or does not have a KCS channel (as is
    the case with a virtual machine), then you will often see an error message
    on the console of the managed server:

    .. code::

        insmod: ERROR: could not load module /opt/drivers/ipmi_msghandler.ks: No such file or directory

    when running the default RackHD discovery through the microkernel.

Hardware that RackHD is managing is done through some network interface. Network
switches typically have an administrator network interface, and Smart PDUs that
can be managed by RackHD have a administrative gateway.

Compute servers have the most varied and complex setup, with data center servers often
leveraging a BMC (Baseboard Management Controller). A BMC is a separate embedded
computer monitoring and controlling a larger computer. The
protocol used most commonly to communicate to a BMC is `IPMI`_, the details of which
can matter significantly.

Desktop class machines (and many laptops) often do not have BMCs,
although some Intel desktops may have an alternative technology: `AMT`_ which provides
some similiar mechanisms.

You can view a detailed diagram of the components inside a BMC at `IPMI Basics`_,
although every hardware vendor is slighty different in how they configure their servers.
The primary difference for most Intel-based server vendors is how the BMC network
interface is exposed. There are two options that you will commonly see:

* LOM : Lights out Management
    The BMC has has a dedicated network interface to the BMC
* SOM : "Shared on motherboard"
    The network interface to the BMC shares a network interface with the motherboard.
    In these cases, the same physical plug is backed by two internal network interfaces
    (each with its own hardware address).

If you're working with a server with a network interface shared by the motherboard and BMC,
then separating the networks that provide IPMI access and the networks that the server
will use during operation may be significantly challenging.

.. _IPMI Basics: https://www.thomas-krenn.com/en/wiki/IPMI_Basics
.. _AMT: https://en.wikipedia.org/wiki/Intel_Active_Management_Technology


IP Address Management
----------------------



Requirements
----------------------------

- DHCP - Layer2 access to PXE network
- TFTP, HTTP - L3 access to PXE network
- IPMI or Redfish - L3 access to OOB network


.. include:: rackhd/ubuntu_source_installation.rst
.. include:: rackhd/ubuntu_package_installation.rst
.. include:: rackhd/configuration.rst

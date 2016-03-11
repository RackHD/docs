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
--------------------




Hardware Controls
------------------


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

Redfish API Overview
=============================

.. contents:: Table of Contents

Overview
-----------------------------

RackHD is designed to provide a REST (Representational state transfer) architecture to provide a 
RESTful API. RackHD currently has two RESTful interfaces: a Redfish API and native REST API 2.0.

The Redfish API is compliant with the Redfish specification as an additional REST API. It provides 
a common data model for representing bare metal hardware, as an aggregate for multiple backend 
servers and systems.

The REST API 2.0 provides unique features that are not provided in the Redfish API.

Redfish API Example
-----------------------------

**Redfish API - Chassis**

List the Chassis that is managed by RackHD (equivalent to the enclosure node in REST API 2.0), by running the following command.

.. code::

  curl 127.0.0.1:9090/redfish/v1/Chassis| jq '.'


.. image:: ../_static/redfish_chasis.png
     :align: center

**Redfish API - System**

1. In the rackhd-server, list the System that is managed by RackHD (equivalent to compute node in API 2.0), by running the following command

.. code::

  curl 127.0.0.1:9090/redfish/v1/Systems| jq '.'

2. Use the mouse to select the **System-ID** as below example, then the ID will be in your clipboard. This ID will be used in the following steps.


.. image:: ../_static/redfish_sys.png
     :align: center

**Redfish API - SEL Log**

.. code::

    curl 127.0.0.1:9090/redfish/v1/systems/<System-ID>/LogServices/Sel| jq '.'

.. image:: ../_static/redfish_sel.png
     :align: center

**Redfish API - CPU info**

.. code::

   curl 127.0.0.1:9090/redfish/v1/Systems/<System-ID>/Processors/0| jq '.'

.. image:: ../_static/redfish_cpu.png
     :align: center

**Redfish API - Helper**

Show the list of RackHD Redfish APIs' by running below command:

.. code::

  curl 127.0.0.1:9090/redfish/v1| jq '.'

.. image:: ../_static/redfish_helper.png
     :align: center
Data Model Overview
=============================

.. contents:: Table of Contents

Introduction to the Redfish data model
--------------------------------------

- All resources linked from a Service Entry point (root)
  - Always located at URL: /redfish/v1
- Major resource types structured in ‘collections’ to allow for standalone, multinode,or aggregated rack-level systems
  - Additional related resources fan out from members within these collections
- ComputerSystem: properties expected from an OS console
  - Items needed to run the “computer”
  - Roughly a logical view of a computer system as seen from the OS
- Chassis: properties needed to locate the unit with your hands
  - Items needed to identify, install or service the “computer”
  - Roughly a physical view of a computer system as seen by a human
- Managers: properties needed to perform administrative functions
  - aka: the systems management subsystem (BMC)


Resource Map (highlights)
-----------------------------

|

.. image:: ../_static/redfish_resourcemap.png
 :height: 600
 :align: center

|
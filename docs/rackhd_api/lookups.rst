Lookup Table
=============================

.. contents:: Table of Contents

Lookup is a mechaniasm that RackHD used to correlate ID, MAC address and IP adress for each node, so 
that RackHD can easily map one element to the others.

API commands
-----------------------------

**REST API (v2.0) - lookup table**

Dump the IP address in the lookup table (where RackHD maintain the nodes IP), by running the following command.

.. code::

  curl localhost:9090/api/2.0/lookups | jq '.'

.. image:: ../_static/lookup_info.png
   :align: center
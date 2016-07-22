SSDP/UPnP
---------
.. _SSDP: https://en.wikipedia.org/wiki/Simple_Service_Discovery_Protocol

RackHD on-http service uses `SSDP`_ (Simple Service Discovery Protocol) to advertise its Restful API services
and device descriptions. The on-http service will respond to M-SEARCH queries from SSDP enabled clients for requested discovery. 

Northbound M-SEARCH Queries
~~~~~~~~~~~~~~~~~~~~~~~~~~~
- Request all: **ssdp:all**
- Request Root device description: **upnp:rootdevice**
- Request on-http device description: **urn:schemas-upnp-org:device:on-http:1**
- Request API v2.0 service: **urn:schemas-upnp-org:service:api:2.0**
- Request Redfish v1.0 service: **urn:dmtf-org:service:redfish-rest:1.0**

- Example Response:

.. code-block:: JSON

  {
    "ST": "urn:dmtf-org:service:redfish-rest:1.0",
    "USN": "564d4f6e-a405-706e-38ec-da52ad81e97a::urn:dmtf-org:service:redfish-rest:1.0",
    "LOCATION": "http://10.2.3.1:8080/redfish/v1/",
    "CACHE-CONTROL": "max-age=1800",
    "DATE": "Tue, 31 May 2016 18:43:29 GMT",
    "SERVER": "node.js/5.0.0 uPnP/1.1 on-http",
    "EXT": ""
  }


Southbound M-SEARCH Queries:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~
- Request all: **ssdp:all**
- Request API v2.0 service: **urn:schemas-upnp-org:service:api:2.0:southbound**
- Request Redfish v1.0 service: **urn:dmtf-org:service:redfish-rest:1.0:southbound**

- Example Response:

.. code-block:: JSON

  {
    "ST": "urn:schemas-upnp-org:service:api:2.0:southbound",
    "USN": "564d4f6e-a405-706e-38ec-da52ad81e97a::urn:schemas-upnp-org:service:api:2.0:southbound",
    "LOCATION": "http://172.31.128.1:9080/api/2.0/",
    "CACHE-CONTROL": "max-age=1800",
    "DATE": "Tue, 31 May 2016 18:43:29 GMT",
    "SERVER": "node.js/5.0.0 uPnP/1.1 on-http",
    "EXT": ""
  }


Southbound Advertisement Handler:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
RackHD will poll for SSDP/UPnP advertisements made by nodes residing on the southbound side network.
For each advertisement RackHD will publish an alert event to the **on.ssdp** AMQP exchange to notify
layers sitting above RackHD.

- Exchange: **on.ssdp**
- Routing Key prefix: **ssdp.alert.***
- AMQP published message example:

.. code-block:: JSON

    {
        "delivery_info": {
            "consumer_tag": "None1",
            "delivery_tag": 1734,
            "exchange": "on.ssdp",
            "redelivered": false,
            "routing_key": "ssdp.alert.uuid:f40c2981-7329-40b7-8b04-27f187aecfb5::urn:schemas-upnp-org:service:ConnectionManager:1"
        },
        "message": {
            "value": {
                "headers": {
                    "CACHE-CONTROL": "max-age=1800",
                    "DATE": "Mon, 06 Jun 2016 17:09:34 GMT",
                    "EXT": "",
                    "LOCATION": "172.31.129.47/desc.html",
                    "SERVER": "node.js/0.10.25 UPnP/1.1 node-ssdp/2.7.1",
                    "ST": "urn:schemas-upnp-org:service:ConnectionManager:1",
                    "USN": "uuid:f40c2981-7329-40b7-8b04-27f187aecfb5::urn:schemas-upnp-org:service:ConnectionManager:1"
                },
                "info": {
                    "address": "172.31.129.47",
                    "family": "IPv4",
                    "port": 1900,
                    "size": 329
                }
            }
        },
        "properties": {
            "content_type": "application/json",
            "type": "Result"
        }
    }


Configuration Options
~~~~~~~~~~~~~~~~~~~~~
Related options defined in `config.json`. For complete examples see :doc:`configuration`.



.. list-table::
    :widths: 20 100
    :header-rows: 1

    * - Parameter
      - Description
    * - enableUPnP
      - boolean true or false to enable or disable all SSDP related server/client services.
    * - ssdpBindAddress 
      - The bind address to send advertisements on (defaults to 0.0.0.0).


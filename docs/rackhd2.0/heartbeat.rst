Heartbeat
---------
Each running RackHD service will publish a periodic heartbeat event message to AMQP to notify subscribers that the service is running. 
Basic heartbeat and process information will be included in the AMQP payload such as relative timestamps, FQDN and basic process information.

Heartbeat AMQP exchange:
~~~~~~~~~~~~~~~~~~~~~~~~
- Exchange: **on.heartbeat**
- Routing Key **[fqdn].[service name]** 
- AMQP published message example:

Routing Key example: **kickseed.example.com.on-tftp**

.. code-block:: JSON

    {
        "delivery_info": {
            "consumer_tag": "None1",
            "delivery_tag": 2479,
            "exchange": "on.heartbeat",
            "redelivered": false,
            "routing_key": "kickseed.example.com.on-tftp"
        },
        "message": {
            "value": {
                "name": "on-tftp",
                "pid": 26633,
                "lastUpdate": "2016-07-13T14:23:35.784Z",
                "currentTime": "2016-07-13T14:23:45.627Z",
                "nextUpdate": "2016-07-13T14:23:55.627Z",
                "cpuUsage": {
                    "system": 92810,
                    "user": 1545498
                },
                "memoryUsage": {
                    "heapTotal": 71938048,
                    "heapUsed": 45444240,
                    "rss": 98926592
                },
                "platform": "linux",
                "release": {
                    "headersUrl": "https://nodejs.org/download/release/v6.3.0/node-v6.3.0-headers.tar.gz",
                    "name": "node",
                    "sourceUrl": "https://nodejs.org/download/release/v6.3.0/node-v6.3.0.tar.gz"
                },
                "title": "node",
                "uid": 0,
                "versions": {
                    "ares": "1.10.1-DEV",
                    "http_parser": "2.7.0",
                    "icu": "57.1",
                    "modules": "48",
                    "node": "6.3.0",
                    "openssl": "1.0.2h",
                    "uv": "1.9.1",
                    "v8": "5.0.71.52",
                    "zlib": "1.2.8"
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
    * - heartbeatIntervalSec
      - Integer value setting the heartbeat send interval in seconds. Setting this value to 0 will disable the heartbeat service (defaults to 10) 


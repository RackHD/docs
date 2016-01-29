Messenger design notes
----------------------

These are design notes from the original creation of the messenger service used by all applications in RackHD through the core libraries

The code to match these designs is available at https://github.com/RackHD/on-core/blob/master/lib/common/messenger.js

Messenger provides functionality to our core code for communicating via AMQP using RabbitMQ.

There are 3 main operations that are provided for communication including the following:

* Publish (Exchange, Topic, Data) -> Promise (Success)
* Subscribe (Exchange, Topic, Callback) - Promise (Subscription)
* Request (Exchange, Topic, Data) -> Promise (Response)

Within these operations we provide additional functionality for object marshaling, object validation, and tracing of requests.

Publish (Exchange, Topic, Data) -> Promise (Success)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Publish provides the mechanism to send data to a particular RabbitMQ exchange & topic.

Subscribe (Exchange, Topic, Callback) -> Promise (Subscription)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Subscribe provides the mechanism to listen for publishes or requests which are provided through the callback argument. The subscribe callback receives data in the form of the following:

.. code-block:: javascript

    function (data, message) {
    	/*
    	 *  data - The published message data.
    	 *  message - A Message object with additional data and features.
    	 */
    }

To respond to a message we support the Promise deferred syntax.

Success

.. code-block:: javascript

    message.resolve({ hello: 'world' });

Failure

.. code-block:: javascript

    message.reject(new Error('Some Error'));

Request (Exchange, Topic, Data) -> Promise (Response)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Request is a wrapper around the Publish/Subscribe mechanism which will first create a reply queue for a response and then publish the data to the requested exchange & topic. It's assumed that a Subscriber using the Subscribe API will respond to the message or a timeout will occur.
The reply queue is automatically generated and disposed of at the end of the request so no subscriptions need to be managed by the consumer.

Object Marshaling
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

While plain JavaScript objects can be sent over the messenger it also supports marshaling of Serializable types in On-Core. Objects which implement the Serializable interface can be marshaled over AMQP by using a constructor initialization convention and by registering their type with the messenger.
When sending a Serializable object over AMQP the messenger uses the registered type to decorate the AMQP message in a way in which a receiver can create a new copy of the object using it's typed constructor.
Subscribers who receive constructed types will have access to them directly through their data value in the subscriber callback.

Object Validation
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

On publish and on subscription callback the messenger will also validate Serializable objects using the Validatable base class.
Validation is provided via JSON Schemas which are attached to the sub-classed Validatable objects.
If an object to be marshaled is Validatable the messenger will validate the object prior to publish or subscribe callback.
Future versions of the messenger will support subscription and request type definitions which will allow consumers to identify what types of objects they expect to be notified about which will give the messenger an additional means of ensuring communications are handled correctly.
Some example schemas are listed below:
MAC Address

.. code-block:: javascript

    {
        id: 'MacAddress',
        type: 'object',
        properties: {
            value: {
                type: 'string',
                pattern: '^([0-9a-fA-F][0-9a-fA-F]:){5}([0-9a-fA-F][0-9a-fA-F])$'
            }
        },
        required: [ 'value' ]
    }

IP Address

.. code-block:: javascript

    {
        id: 'IpAddress',
        type: 'object',
        properties: {
            value: {
                type: 'string',
                format: 'ipv4'
            }
        },
        required: [ 'value' ]
    }

Lookup Model (via On-Http)

.. code-block:: javascript


    {
        id: 'Serializables.V1.Lookup',
        type: 'object',
        properties: {
            node: {
                type: 'string'
            },
            ipAddress: {
                type: 'string',
                format: 'ipv4'
            },
            macAddress: {
                type: 'string',
                pattern: '^([0-9A-Fa-f]{2}[:-]){5}([0-9A-Fa-f]{2})$'
            }
        },
        required: [ 'macAddress' ]
    }

Additional Information
~~~~~~~~~~~~~~~~~~~~~~~~

With the primary goal of the messenger being to simplify usage patterns for the consumer not all of the features have been highlighted. Below is a quick recap of the high level features.

* Publish, Subscribe, and Request/Response Patterns.
* Optional Object Marshaling.
* Optional Object Validation via JSON Schema.
* Publish & Subscribe use their own connections to improve latency in request/response patterns.
* Automatic creation of exchanges on startup.
* Automatic subscription management for Request/Response patterns.
* Automatic Request correlation and context marshaling.

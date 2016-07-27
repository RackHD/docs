RackHD API
-------------------------

Our REST based API is the abstraction layer for the low-level management tasks
that are performed on hardware devices, and information about those devices.
For example, when a compute server is "discovered" (see :doc:`../how_it_works`
for more details on this process), the information about that server is expressed
as `nodes` and `catalogs` in the RackHD API. When you want to re-image that
compute node, the RackHD API is used to activate a workflow containing the tasks
that are appropriate to doing that function.

The RackHD API can be used to manage nodes, catalogs, workflows, tasks, templates,
pollers, and other entities. For the complete list of functions, generate the RackHD
API documentation as described below.

**List All Nodes**

.. code::

  curl http://<server>:8080/api/common/nodes | python -mjson.tool

**Set the Active Workflow**

.. code::

  curl http://<server>:8080/api/common/nodes/<identifier>/workflows/active | python -mjson.tool


Starting and Stopping the API Server
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The API server runs by default. Use the following commands to stop or start the API server.

================ ===============================
 Action           Command
================ ===============================
Stop API server   `sudo service on-http stop`
Start API server  `sudo service on-http start`
================ ===============================


Generating API Documentation
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You can generate an HTML version of the API documentation by cloning the *on-http*
repository and running the following command.

.. code::

  $ git clone https://github.com/RackHD/on-http
  $ cd on-http
  $ npm install
  $ npm run apidoc

The default and example quick start build that we describe in :doc:`../getting_started`
has the API docs rendered and embedded within that instance for easy use, available
at http://[IP ADDRESS OF VM]:8080/docs/ for the 1.1 API documentation, and
http://[IP ADDRESS OF VM]:8080/swagger-ui/ for the 2.0 and Redfish API documentation.

RackHD Client Libraries
-------------------------

The 1.1 API for RackHD includes annotations in code that are used to generate
API documentation (the `npm run apidoc` command above) that is hosted directly
within RackHD.

A similiar process generates a swagger API definition file that can be used to
create client libraries with `swagger`_. To create this file locally, you can
check out the on-http library and run the commands::

    npm install
    npm run apidoc-swagger

The resulting file will be build/apidoc-swagger/swagger.json. You can use this
file with `swagger tools`_ to generate client libraries in any of the languages
that swagger supports. `RackHD travis build`_ for on-http includes using
codegen from `swagger tools`_ to generate a python library, available in a debian
package, that we use with integration testing.

You can also use the `swagger generator`_ online tool to generate a client zip
bundle for a variety of languages, including python, Java, javascript, ruby,
scala, php, and more.

.. _swagger: http://swagger.io
.. _swagger tools: http://swagger.io/tools/
.. _RackHD travis build: https://github.com/RackHD/on-http/blob/master/.travis.yml#L28-L38
.. _swagger generator: https://generator.swagger.io

Examples using the python client library
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Getting a list of nodes::

    from on_http import NodesApi, ApiClient, Configuration

    config = Configuration()
    config.debug = True
    config.verify_ssl = False

    client = ApiClient(host='http://localhost:9090',header_name='Content-Type',header_value='application/json')
    nodes = NodesApi(api_client=client)
    nodes.api1_1_nodes_get()
    print client.last_response.data

2.0 API - Getting a list of nodes ::

    from on_http import NodesApi, ApiClient, Configuration

        config = Configuration()
        config.debug = True
        config.verify_ssl = False

        client = ApiClient(host='http://localhost:9090',header_name='Content-Type',header_value='application/json')
        nodes = NodesApi(api_client=client)
        nodes.api2_0_nodes_get()
        print client.last_response.data

Or the same asynchronously (with a callback)::

    def cb_func(resp):
    print 'GET /nodes callback!', resp

    thread = nodes.api1_1_nodes_get(callback=cb_func)

2.0 API - Or the same asynchronously (with a callback)::

    def cb_func(resp):
    print 'GET /nodes callback!', resp

    thread = nodes.api2_0_nodes_get(callback=cb_func)
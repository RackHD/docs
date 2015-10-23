Workflow: Graphs
========

The graphs/workflows API (workflows is a backwards-compatible term for graphs) provides
functionality for running :doc:`tasks` via
graph-based control flow mechanisms. For example, a typical graph consists of a list of
:doc:`tasks`, which themselves are essentially decorated functions. The graph definition specifies
any context and/or option values that should be handed to these functions, and more importantly,
it provides a mechanism for specifying when each task should be run. The most simple case is
saying a task should be run only after a previous task has succeeded (essentially becoming a
state machine). More complex graphs may involve event based task running, or defining
data/event channels that should exist between concurrently running tasks.

API commands
------------

When running the renasar-http process, these are some common API commands you can send:

Get available graphs in the library
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: rest

    GET /api/1.1/workflows/library

.. code-block:: rest

    curl <server>/api/1.1/workflows/library

**Run a new graph against a node**

Find the graph definition you would like to use, and copy the top-level *injectableName* attribute

.. code-block:: rest

    POST /api/1.1/nodes/<id>/workflows
    {
        name: <graph name>
    }


.. code-block:: rest

    curl -X POST <server>/api/1.1/nodes/<id>/workflows?name=<graphname>
    OR
    curl -X POST \
        -H 'Content-Type: application/json' \
        -d '{"name": "<graphname>"}' \
        <server>/api/1.1/nodes/<id>/workflows

Sample Output:

.. literalinclude:: samples/serialized-graph-example.json
   :language: JSON

**Query an active graph's state**

.. code-block:: rest

    GET /api/1.1/nodes/<id>/workflows/active

.. code-block:: rest
    curl <server>/api/1.1/nodes/<id>/workflows/active


**Create a new graph definition**

.. code-block:: rest

    PUT /api/1.1/workflows
    {
        <json definition of graph>
    }

Creating new graphs
^^^^^^^^^^^^^^^^^

Graphs are defined via a JSON definition that conform to this schema:

- friendlyName (string): a human readable name for the graph
- injectableName (string): a unique name used by the system and the API to refer to the graph
- tasks (array of objects): a list of task definitions or references to task definitions. For an in-depth explanation
        of task definitions, see https://github.com/RackHD/on-tasks/
    - tasks.label (string): a unique string to be used as a reference within the graph definition
    - tasks.\[taskName\] (string): the injectableName of a task in the database to run. This or taskDefinition is required.
    - tasks.\[taskDefinition\] (object): an inline definition of a task, instead of one in the database. This or taskName is required.
    - tasks.\[ignoreFailure\] (boolean): ignoreFailure: true will prevent the graph from failing on task failure
    - tasks.\[waitOn\] (object): key, value pairs referencing other task labels to desired states of those tasks to trigger running on.
                                    Available states are 'succeeded', and 'failed' and 'finished' (run on succeeded or failed). If waitOn
                                    is not specified, the task will run on graph start.
- [options]
    - options.\[defaults\] (object): key, value pairs that will be handed to any tasks that have matching option keys
    - options.\<label\> (object): key, value pairs that should all be handed to a specific task

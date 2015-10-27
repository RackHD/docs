Workflow Graphs
=====================

The graphs/workflows API (workflows is a backwards-compatible term for graphs) provides
functionality for running :doc:`tasks` via
graph-based control flow mechanisms. A typical graph consists of a list of
tasks which themselves are essentially decorated functions.

The graph definition specifies the order in which tasks should run and provides
any context and/or option values to pass to these functions.

Complex graphs may define event-based tasks or specify
data/event channels that should exist between concurrently-run tasks.

Defining Graphs
-------------------------

Graphs are defined via a JSON definition that conform to this schema:

- friendlyName (string): a human readable name for the graph
- injectableName (string): a unique name used by the system and the API to refer to the graph
- tasks (array of objects): a list of task definitions or references to task definitions. For an in-depth explanation of task definitions, see https://github.com/RackHD/on-tasks/
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


API Commands for Graphs
------------------------------

The following are common API commands that can be used when running the *on-http* process.


**Get Available Graphs in the Library**

.. code-block:: rest

    GET /api/1.1/workflows/library

.. code-block:: rest

    curl <server>/api/1.1/workflows/library

**Query the State of An Active Graph**

.. code-block:: rest

        GET /api/1.1/nodes/<id>/workflows/active

.. code-block:: rest
        curl <server>/api/1.1/nodes/<id>/workflows/active


**Create a Graph Definition**

    .. code-block:: rest

        PUT /api/1.1/workflows
        {
            <json definition of graph>
        }


**Run a New Graph Against a Node**

Find the graph definition you would like to use and copy the top-level *injectableName* attribute.

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

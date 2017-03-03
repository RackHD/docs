.. _workflows-ref-label:

Workflows
---------------------------

Workflow Graphs
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The workflow graph definition specifies the order in which tasks should run and provides
any context and/or option values to pass to these functions.

Complex graphs may define event-based tasks or specify
data/event channels that should exist between concurrently-run tasks.

Defining Graphs
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Graphs are defined via a JSON definition that conform to this schema:

- friendlyName (string): a human readable name for the graph
- injectableName (string): a unique name used by the system and the API to refer to the graph
- tasks (array of objects): a list of task definitions or references to task definitions.

  - tasks.label (string): a unique string to be used as a reference within the graph definition
  - tasks.\[taskName\] (string): the injectableName of a task in the database to run. This or taskDefinition is required.
  - tasks.\[taskDefinition\] (object): an inline definition of a task, instead of one in the database. This or taskName is required.
  - tasks.\[ignoreFailure\] (boolean): ignoreFailure: true will prevent the graph from failing on task failure
  - tasks.\[waitOn\] (object): key/value pairs referencing other task labels to desired states of those tasks to trigger running on. Available states are *succeeded*, *failed* and *finished* (run on succeeded or failed). If waitOn is not specified, the task will run on graph start.

- [options]

  - options.\[defaults\] (object): key, value pairs that will be handed to any tasks that have matching option keys
  - options.\<label\> (object): key, value pairs that should all be handed to a specific task


Graph definition attributes
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

**Graph Tasks**

The `tasks` field in a graph definition represents the collection of tasks that make up the runtime behavior of the graph.
The task definition is referenced by the ``taskName`` field (which maps to the ``injectableName`` field in the task definition).
The ``label`` field is used as a reference when specifying
dependencies for other tasks in the graph definition.
For example, this graph will run three tasks one after the other:

::

    {
        "injectableName": "Graph.Example.Linear",
        "friendlyName": "Linear ordered tasks",
        "tasks": [
            {
                "label": "task-1",
                "taskName": "Task.example"
            },
            {
                "label": "task-2",
                "taskName": "Task.example",
                "waitOn": {
                    "task-1": "succeeded"
                }
            },
            {
                "label": "task-3",
                "taskName": "Task.example",
                "waitOn": {
                    "task-2": "succeeded"
                }
            }
        ]
    }

The ordering is specified by the ``waitOn`` key in each task object, which specifies conditions that must be met before each task can be run.
In the above graph definition, ``task-1`` has no dependencies, so it will be run immediately, ``task-2`` has a dependency on ``task-1`` succeeding,
and ``task-3`` has a dependency on ``task-2`` succeeding.

Here is an example of a graph that will run tasks in parallel:

::

    {
        "injectableName": "Graph.Example.Parallel",
        "friendlyName": "Parallel ordered tasks",
        "tasks": [
            {
                "label": "task-1",
                "taskName": "Task.example"
            },
            {
                "label": "task-2",
                "taskName": "Task.example",
                "waitOn": {
                    "task-1": "succeeded"
                }
            },
            {
                "label": "task-3",
                "taskName": "Task.example",
                "waitOn": {
                    "task-1": "succeeded"
                }
            }
        ]
    }

This graph is almost the same as the "Linear ordered tasks" example, except that ``task-2`` and ``task-3`` *both* have a dependency on ``task-1``. When
``task-1`` succeeds, ``task-2`` and ``task-3`` will be started in parallel.

Tasks can also be ordered based on multiple dependencies:

::

    {
        "injectableName": "Graph.Example.MultipleDependencies",
        "friendlyName": "Tasks with multiple dependencies",
        "tasks": [
            {
                "label": "task-1",
                "taskName": "Task.example"
            },
            {
                "label": "task-2",
                "taskName": "Task.example"
            },
            {
                "label": "task-3",
                "taskName": "Task.example",
                "waitOn": {
                    "task-1": "succeeded",
                    "task-2": "succeeded"
                }
            }
        ]
    }

In the above example, ``task-1`` and ``task-2`` will be started in parallel, and ``task-3`` will only be started once ``task-1`` and ``task-2`` have both succeeded.

.. _Graph Options:

**Graph Options**

As detailed in the :ref:`task-definition-ref-label` section, each task definition has an options object
that can be used to customize the task. All values set in the options objects are considered defaults, and can be overridden
within the Graph definition. Additionally, the options values can be overridden again by the data in the API request made to run the graph.

For example, a simple task definition with options looks like this:

::

    {
        "injectableName": "Task.Example.Options",
        "friendlyName": "Task with basic options",
        "implementsTask": "Task.Base.Example",
        "options": {
            "option1": "value 1",
            "option2": "value 2"
        },
        "properties": {}
    }

As is, this task definition specifies default values of "value 1" and "value 2" for its respective options.
In the graph definition, these values can be changed to have new defaults by adding a key to the ``Graph.options`` object
that matches the ``label`` string given to the task object ("example-options-task" in this case):

::

    {
        "injectableName": "Graph.Example.Options",
        "friendlyName": "Override options for a task",
        "options": {
            "example-options-task": {
                "option1": "overridden value 1",
                "option2": "overridden value 2"
            }
        },
        "tasks": [
            {
                "label": "example-options-task",
                "taskName": "Task.Example.Options"
            }
        ]
    }

    // Task.Example.Options will be run as this
    {
        "injectableName": "Task.Example.Options",
        "friendlyName": "Task with basic options",
        "implementsTask": "Task.Base.Example",
        "options": {
            "option1": "overridden value 1",
            "option2": "overridden value 2"
        },
        "properties": {}
    }

Sometimes, it is necessary to be able to propagate the same values to multiple tasks, but it can be a chore
to make a separate options object for each task label. In this case, there is a special field
used in the ``Graph.options`` object called ``defaults``.
When ``defaults`` is set, the graph will iterate through each key in the object and override that value for every task definition
that also has that key in its respective options object. In the above example, the ``Task.Example.Options`` definition will be changed with new
values for ``option1`` and ``option2``, but not for ``option3``, since ``option3`` does not exist in the options object for that task definition:

::

    {
        "injectableName": "Graph.Example.Defaults",
        "friendlyName": "Override options with defaults",
        "options": {
            "defaults": {
                "option1": "overridden value 1",
                "option2": "overridden value 2",
                "option3": "this will not get set"
            }
        },
        "tasks": [
            {
                "label": "example-options-task",
                "taskName": "Task.Example.Options"
            }
        ]
    }

    // Task.Example.Options will be run as this
    {
        "injectableName": "Task.Example.Options",
        "friendlyName": "Task with basic options",
        "implementsTask": "Task.Base.Example",
        "options": {
            "option1": "overridden value 1",
            "option2": "overridden value 2"
        },
        "properties": {}
    }

.. _Graph Username Example:

The ``defaults`` object can be used to share values across every task definition that includes them,
as in this example workflow that validates and sets a username.

::

    {
        "injectableName": "Graph.Example.SetUsername",
        "friendlyName": "Set a username",
        "options": {
            "defaults": {
                "username": "TESTUSER",
                "group": "admin"
            }
        },
        "tasks": [
            {
                "label": "validate-username",
                "taskName": "Task.Example.ValidateUsername"
            },
            {
                "label": "set-username",
                "taskName": "Task.Example.SetUsername",
                "waitOn": {
                    "validate-username": "succeeded"
                }
            }
        ]
    }

    // Task.Example.ValidateUsername definition
    {
        "injectableName": "Task.Example.Validateusername",
        "friendlyName": "Validate a username",
        "implementsTask": "Task.Base.ValidateUsername",
        "options": {
            "username": null,
        },
        "properties": {}
    }

    // Task.Example.SetUsername definition
    {
        "injectableName": "Task.Example.Setusername",
        "friendlyName": "Set a username",
        "implementsTask": "Task.Base.SetUsername",
        "options": {
            "username": null,
            "group": null
        },
        "properties": {}
    }

Both tasks will share the "TESTUSER" value for
the ``username`` option, but only the ``Task.Example.SetUsername`` task will use the value for ``group``, since it
is the only task definition in this graph with that key in its options object.
After processing the graph definition and the default options, the task definitions will be run as:

::

    // Task.Example.ValidateUsername definition after Graph defaults applied
    {
        "injectableName": "Task.Example.Validateusername",
        "friendlyName": "Validate a username",
        "implementsTask": "Task.Base.ValidateUsername",
        "options": {
            "username": "TESTUSER"
        },
        "properties": {}
    }

    // Task.Example.SetUsername definition after Graph defaults applied
    {
        "injectableName": "Task.Example.Setusername",
        "friendlyName": "Set a username",
        "implementsTask": "Task.Base.SetUsername",
        "options": {
            "username": "TESTUSER",
            "group": "admin"
        },
        "properties": {}
    }


API Commands for Graphs
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The following are API commands that can be used when running the *on-http* process.


**Get Available Graphs in the Library**

::

    GET /api/current/workflows/graphs

::

    curl <server>/api/current/workflows/graphs


**Deprecated 1.1 API - Get Available Graphs in the Library**

::

    GET /api/1.1/workflows/library/*

::

    curl <server>/api/1.1/workflows/library/*


**Query the State of an Active Graph**

::

    GET /api/current/nodes/<id>/workflows?active=true

::

    curl <server>/api/current/workflows?active=true


**Deprecated 1.1 API - Query State of an Active Graph**

::

    GET /api/1.1/nodes/<id>/workflows/active

::

    curl <server>/api/1.1/nodes/<id>/workflows/active

**Cancel or Kill an Active Graph running against a Node**

::

    PUT /api/current/nodes/<id>/workflows/action
            {
                "command": "cancel"
            }

::

    curl -X PUT \
            -H 'Content-Type: application/json' \
            -d '{"command": "cancel"}' \
            <server>/api/current/nodes/<id>/workflows/action


**Deprecated 1.1 API - Cancel or Kill an Active Graph running against a Node**

::

        DELETE /api/1.1/nodes/<id>/workflows/active

::

        curl -X DELETE <server>/api/1.1/nodes/<id>/workflows/active


**List all Graphs that have or are running against a Node**

::

        GET /api/current/nodes/<id>/workflows

::

        curl <server>/api/current/nodes/<id>/workflows


**Create a Graph Definition**

::

        PUT /api/current/workflows/graphs
        {
            <json definition of graph>
        }


**Deprecated 1.1 API - Create a Graph Definition**

::

        PUT /api/1.1/workflows
        {
            <json definition of graph>
        }


**Run a New Graph Against a Node**

Find the graph definition you would like to use and copy the top-level *injectableName* attribute.

::

    POST /api/current/nodes/<id>/workflows
    {
        "name": <graph name>
    }


::

    curl -X POST <server>/api/current/nodes/<id>/workflows?name=<graphname>
    OR
    curl -X POST \
        -H 'Content-Type: application/json' \
        -d '{"name": "<graphname>"}' \
        <server>/api/current/nodes/<id>/workflows

To override option values, add an options object to the POST data as detailed in the `Graph Options`_ section.

::

    POST /api/current/nodes/<id>/workflows
    {
        "name": <graph name>
        "options": { <graph options here> }
    }

For example, to override an option "username" for all tasks in a graph that utilize that option (see `Graph Username Example`_, send the following request:

::

    POST /api/current/nodes/<id>/workflows
    {
        "name": <graph name>
        "options": {
            "defaults": {
                "username": "customusername"
            }
        }
    }

Sample Output:

.. literalinclude:: samples/serialized-graph-example.json
   :language: JSON

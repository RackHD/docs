Workflow Tasks
~~~~~~~~~~~~~~~~~~~~~~

A workflow task is a unit of work decorated with data and logic that allows it to
be included and run within a workflow. Tasks can be
defined to do wide-ranging operations, such as bootstrap a server node into a
Linux microkernel, parse data for matches against a rule, and others. The tasks in a workflow are run in a specific order.

A workflow task is made up of three parts:

* Task Definition
* Base Task Definition
* Job

.. _task-definition-ref-label:

Task Definitions
^^^^^^^^^^^^^^^^^^^^^^^

A task definition contains the basic description of the task. It contains the following fields.

=============== ======= =========== =======================================================
Name            Type    Flags       Description
=============== ======= =========== =======================================================
friendlyName    String  Required    A human-readable name for the task
injectableName  String  Required    A unique name used by the system and the API to refer to the task.
implementsTask  String  Required    The injectableName of the base task.
schemaRef       String  Optional    The task schema for the *options*, see `Task Schema`_ for detail.
options         Object  Required    Key value pairs that are passed in as options to the job.
                                    Values required by a job may be defined in the task definition or overridden by options in a graph definition.
properties      Object  Required    JSON defining any relevant metadata or tagging for the task.
=============== ======= =========== =======================================================


Below is a sample task definition in JSON for an Ubuntu installer.

.. code-block:: JSON

    {
        "friendlyName": "Install Ubuntu",
        "injectableName": "Task.Os.Install.Ubuntu",
        "implementsTask": "Task.Base.Os.Install",
        "options": {
            "username": "monorail",
            "password": "password",
            "profile": "install-trusty.ipxe",
            "hostname": "monorail",
            "uid": 1010,
            "domain": "",
            "completionUri": "renasar-ansible.pub"
        },
        "properties": {
            "os": {
                "linux": {
                    "distribution": "ubuntu",
                    "release": "trusty"
                }
            }
        }
    }


Sample output (returns injectableName):

.. code-block:: rest

    "Task.Os.Install.Ubuntu.Utopic"


Base Task Definitions
^^^^^^^^^^^^^^^^^^^^^^^

A Base Task Definition outlines validation requirements (an interface) and a common
job to be used for a certain class of tasks. Base Task Definitions exist to
provide strict and standardized validation schemas for graphs, and to improve
code re-use and modularity.

The following table describes the fields of a Base Task Definition.

=================== ======= ========= =========================================================
Name                Type    Flags     Description
=================== ======= ========= =========================================================
friendlyName        String  Required  A human-readable name for the task.
injectableName      String  Required  A unique name used by the system and the API to refer to the task.
requiredOptions     Object  Required  Required option values to be set in a task definition implementing the base task.
requiredProperties  Object  Required  JSON defining required properties that need to exist in other tasks in a graph in
                                      order for this task to be able to be run successfully.
properties          Object  Required  JSON defining any relevant metadata or tagging for the task. This metadata is
                                      merged with any properties defined in task definitions that implement the base task.
=================== ======= ========= =========================================================

The following example shows the base task *Install Ubuntu* task definition:

  .. code-block:: javascript

        {
            "friendlyName": "Install OS",
            "injectableName": "Task.Base.Os.Install",
            "runJob": "Job.Os.Install",
            "requiredOptions": [
                "profile",
                "completionUri"
            ],
            "requiredProperties": {
                "power.state": "reboot"
            },
            "properties": {
                "os": {
                    "type": "install"
                }
            }
        }


This base task is a generic Install OS task. It runs the job named *Job.Os.Install* and
specifies that this job requires the options 'profile' and 'completionUri'. As a result, any
task definition using the *Install OS* base task must provide at least these options to
the OS installer job. These options are utilized by logic in the job.

.. code-block:: javascript

        this._subscribeRequestProfile(function() {
            return this.profile;
        });

Another task definition that utilizes the above base task looks like:

.. code-block:: JSON

        {
            "friendlyName": "Install CoreOS",
            "injectableName": "Task.Os.Install.CoreOS",
            "implementsTask": "Task.Base.Os.Install",
            "options": {
                "username": "root",
                "password": "root",
                "profile": "install-coreos.ipxe",
                "hostname": "coreos-node",
                "completionUri": "pxe-cloud-config.yml"
            },
            "properties": {
                "os": {
                    "linux": {
                        "distribution": "coreos"
                    }
                }
            }
        }

The primary difference between the *Install CoreOS* task and the *Install Ubuntu* task
is the profile value, which is the ipxe template that specifies the installer
images that an installation target should download.

.. _`Task Schema`:

Task Schema
^^^^^^^^^^^^^^^^^^^^^^
A Task Schema is a JSON-Schema_ file that outlines the attributes and validation requirement for all options of a task. It provides standardized and declarative way to annotate task options. It offloads job's validation work and brings benefit to the upfront validation for graph input options.

.. _JSON-Schema: http://json-schema.org/

**Relation between Job & Schema & Task**

- A Task Definition conforms to only one Task Schema, but different Task Definition can conform to the same Task Schema; For example, all obm-control tasks share the same obm-control schema.
- A Task Schema only describes one Job, but a Job may be described by multiple Task Schemas. For example, all of install-centos, install-ubuntu, install-esxi schemas are to describe the same install-os job.
- A Task Schema can include other schemas for the sake of schema reuse and minimizing duplication. For example, all task schemas include a common options schema.

Below is a diagram shows the relation:

.. image:: /_static/task_schema_job_relation.png
  :align: center

**Task Meta Schema**

The Task Meta Schema is the schema of Task Schema, it restricts the syntax of Task Schema.

The name of Task Meta Schema is rackhd-task-schema.json_, any Task Schema should write against it by set the *$schema*:

.. _rackhd-task-schema.json: https://github.com/RackHD/on-tasks/blob/master/lib/task-data/schemas/rackhd-task-schema.json

.. code-block:: JSON

    "$schema": "rackhd-task-schema.json"

The meta schema is bases on the JSON-Schema_ draft-04 standard (http://json-schema.org/draft-04/schema), and it extends following keywords:

============== ======= ========= =========================================================
Keyword        Format  Flags     Description
============== ======= ========= =========================================================
describeJob    String  Required  The property is required to point to a job’s di injectable name
copyright      String  Optional  This attribute shall contain the copyright notice for the schema
readonly       Boolean Optional  This property shall designate a property to be readonly for user when set to true
============== ======= ========= =========================================================

**Define a Task Schema**

You can follow below 3 steps to define a Task Schema:

- Step 1: Create a JSON File

The Task Schema has to be written in JSON. The filename is critical and should be unique as it will be used as reference identifier between tasks and schemas.

- Step 2: Define Basic Properties

At least specify following properties:

============== ======= =========================================================
Property       Format  Description
============== ======= =========================================================
$schema        String  Specify the version of this schema file, it has to be `rackhd-task-schema.json`
title          String  Specify a short description for this schema
description    String  Specify a long & verbose description for this schema
describeJob    String  Specify the job which this schema is to describe, via job's injectableName
============== ======= =========================================================

- Step 3: Define Options

The task options are divided into task common options and task specific options.

The common task options are the options that same for all tasks, such as `Task Timeout`_.
The schema for common task options has been defined in common-task-options.json_, so you don't need to write a duplicated one, you can just reference it by:

.. _common-task-options.json: https://github.com/RackHD/on-tasks/blob/master/lib/task-data/schemas/common-task-options.json

.. code-block:: JSON

    { "$ref": "common-task-options.json#/definitions/Options" }

The task specific options vary by task. This is usually the section you mostly work on while defining your schema.

To combine the common and specific options, use the keyword allOf_.

.. _allOf: http://json-schema.org/latest/json-schema-validation.html#anchor82

The following example shows the schema for the *Analyze-OS-Repo* task:

.. code-block:: JSON

    {
        "$schema": "rackhd-task-schema.json",
        "copyright": "Copyright 2016, EMC, Inc.",
        "title": "Analyze OS Repository",
        "description": "The schema for analyzing os repository job",
        "describeJob": "Job.Os.Analyze.Repo",
        "allOf": [
            { "$ref": "common-task-options.json#/definitions/Options" },
            {
                "type": "object",
                "properties": {
                    "version": {
                        "type": "string",
                        "minLength": 1
                    },
                    "repo": {
                        "type": "string",
                        "format": "uri"
                    },
                    "osType": {
                        "readoly": true
                    }
                },
                "required": ["osType", "repo", "version"]
            }
        ]
    }

Above schema example shows its task specific options are *"version"*, *"repo"* and *"osType"*, it describes the job *Job.Os.Analyze.Repo*.

**Link Schema and Task**

The property *schemaRef* is used to specify its schema via the filename.

Below is the example about how the *Analyze-OS-Repo* task references the schema (Assume the corresponding schema filename is `analyze-os-repo.json`)

.. code-block:: javascript

    module.exports = {
        friendlyName: 'Analyze Esx Repository',
        injectableName: 'Task.Os.Esx.Analyze.Repo',
        implementsTask: 'Task.Base.Os.Analyze.Repo',
        schemaRef: 'analyze-os-repo.json',
        options: {
            osName: 'esx',
        },
        properties: {}
    };

You can define the default value in the *options* property. These default values will be used as complement if user doesn't pass any value for that option while trigger the task, so all default value should conform to the schema as well.

**Upfront Schema Validation**

The Task Schema validation will be firstly executed when user triggers a workflow. Only if all options (Combine user input and the default value) conform to schema for every task, the workflow then can be successfully triggered.
If any option violates the schema, The API request will report `400 Bad Request`_ and append detail error message in response body. For example:

.. _`400 Bad Request`: https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html#sec10.4.1

Below is the message if user forgets the required option *version* while installing CentOS:

.. code-block:: JSON

    "message": "Task.Os.Install.CentOS: JSON schema validation failed - data should have required property 'version'"

Below is the message if the input *uid* beyond the allowed range.

.. code-block:: JSON

    "message": "Task.Os.Install.CentOS: JSON schema validation failed - data.users[0].uid should be >= 500"

Below is the message if the format of option *rootPassword* is not correct:

.. code-block:: JSON

    "message": "Task.Os.Install.CentOS: JSON schema validation failed - data.rootPassword should be string"



Task Jobs
^^^^^^^^^^^^^^^^^^^^^^^

A job is a javascript subclass with a run function that can be referenced
by a string. When a new task is created, and all of its validation and setup logic handled,
the remainder of its responsibility is to instantiate a new job class instance for
its specified job (passing down the options provided in the definition to the
job constructor) and run that job.

**Defining a Job**

To create a job, define a subclass of `Job.Base
<https://github.com/RackHD/on-tasks/blob/master/lib/jobs/base-job.js>`_
that has a method called
*_run* and calls *this._done()* somewhere, if the job is
not one that runs indefinitely.

.. code-block:: javascript

    // Setup injector
    module.exports = jobFactory;
    di.annotate(jobFactory, new di.Provide('Job.example'));
    di.annotate(jobFactory, new di.Inject('Job.Base');

    // Dependency context
    function jobFactory(BaseJob) {
        // Constructor
        function Job(options, context, taskId) {
            Job.super_.call(this, logger, options, context, taskId);
        }
        util.inherits(Job, BaseJob);

        // _run function called by base job
        Job.prototype._run = function _run() {
            var self = this;
            doWorkHere(args, function(err) {
                if (err) {
                    self._done(err);
                } else {
                    self._done();
                }
            });
        }

        return Job;
    }

Many jobs are event-based by nature, so the base job provides many helpers for
assigning callbacks to a myriad of AMQP events published by RackHD services, such
as DHCP requests from a specific mac address, HTTP downloads from a specific IP, template
rendering requests, etc.

Task Templates
^^^^^^^^^^^^^^^^^^^^^^^
There are some values that may be needed in a task definition which are not known in advance. In some cases, it is also more convenient to use placeholder values in a task definition than literal values. In these cases, a simple template rendering syntax can be used in task definitions. Rendering is also useful in places where two or more tasks need to use the same value (e.g. options.file), but it cannot be hardcoded ahead of time.

Task templates use `Mustache syntax <http://mustache.github.io/mustache.5.html>`_, with some additional features detailed below. To define a value to be rendered, place it within curly braces in a string:

.. code-block:: javascript

    someOption: 'an option to be rendered: {{ options.renderedOption }}'

At render time, values are rendered if the exist in the task render context. The render context contains the following fields:


.. list-table::
   :widths: 20 80
   :header-rows: 1

   * - Field
     - Description
   * - server
     - The server field contains all values found in the configuration for the on-taskgraph process (/opt/monorail/config.json)
       Example Usage: `{{ server.mongo.port }}`
   * - api
     - Values used for constructing API requests in a template:
           - **server** -- the base URI for the RackHD http server (e.g. `http://<server>:<port>` )
           - **base** -- the base http URI for the RackHD api (e.g. `http://<server>:<port>/api/current` )
           - **templates** -- the base http URI for the RackHD api files route (e.g. `http://<server>:<port>/api/current/templates`)
           - **profiles** -- the base http URI for the RackHD api files route (e.g. `http://<server>:<port>/api/current/profiles`)
           - **lookups** -- the base http URI for the RackHD api files route (e.g. `http://<server>:<port>/api/current/lookups`)
           - **files** -- the base http URI for the RackHD api files route (e.g. `http://<server>:<port>/api/current/files`)
           - **nodes** -- the base http URI for the RackHD api nodes route (e.g. `http://<server>:<port>/api/current/nodes`)
   * - tasks
     - Allows access to instance variables of the task class instance created from the task definition. This is mainly used to access task.nodeId
   * - options
     - This refers to the task definition options itself. Mainly for referencing values in substrings that will eventually be defined by a user (e.g. `'sudo mv {{ options.targetFile }} /tmp/{{ options.targetfile }}'` )
   * - context
     - This refers to the shared context object that all tasks in a graph have R/W access to. Enables one task to use values produced by another at runtime.

       For example, the [ami catalog provider task](`https://<server>:<port>/projects/RackHD/repos/on-tasks/browse/lib/task-data/tasks/provide-catalog-ami-bios-version.js`) gets the most recent catalog entry for the AMI bios, whose value can be referenced by other tasks via `{{ context.ami.systemRomId }}`
   * - sku
     - This refers to the SKU configuration data fetched from a :doc:`skus`. This field is added automatically if a SKU configuration exists in the the :doc:`skus`, rather than being specified by a user.
   * - env
     - This refers to the environment configuration data retrieved from the environment database collection.Similar to sku, this field is added automatically, rather than specified by a user.

The download-files task is a good example of a task definition that makes use of multiple objects in the context:

.. code-block:: JSON

    {
        friendlyName: 'Flash MegaRAID Controller',
        injectableName: 'Task.Linux.Flash.LSI.MegaRAID',
        implementsTask: 'Task.Base.Linux.Commands',
        options: {
            file: null,
            downloadDir: '/opt/downloads',
            adapter: '0',
            commands: [
                'sudo /opt/MegaRAID/storcli/storcli64 /c{{ options.adapter }} download ' +
                    'file={{ options.downloadDir }}/{{ options.file }} noverchk',
                'sudo /opt/MegaRAID/MegaCli/MegaCli64 -AdpSetProp -BatWarnDsbl 1 ' +
                    '-a{{ options.adapter }}',
            ]
        },
        properties: {
            flash: {
                type: 'storage',
                vendor: {
                    lsi: {
                        controller: 'megaraid'
                    }
                }
            }
        }
    }


On creation, the options are rendered as below. The 'file' field is specified in this case by the contents of an API query, e.g. mr2208fw.rom

.. code-block:: JSON

    options: {
        file: 'mr2208fw.rom',
        downloadDir: '/opt/downloads',
        adapter: '0',
        commands: [
            'sudo /opt/MegaRAID/storcli/storcli64 /c0 download file=/opt/downloads/mr2208fw.rom noverchk',
            'sudo /opt/MegaRAID/MegaCli/MegaCli64 -AdpSetProp -BatWarnDsbl 1 -a0',
        ]
    }

Task Rendering Features
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

For a full list of Mustache rendering features, including specifying conditionals and iterators, see the `Mustache man page <http://mustache.github.io/mustache.5.html>`_

Task templates also expand the capabilities of Mustache templating by adding the additional capabilities of *Fallback Rendering* and *Nested Rendering*, as documented below.

**Fallback Rendering**

Multiple values can be specified within the curly braces, separated by one or two '|' characters (newlines are optional as well after the pipe character). In the case that the first value does not exist, the second one will be used, and so on. Values that are not prefixed by a context field (e.g. 'options.', 'context.' will be rendered as a plain string)

.. code-block:: rest

    // Unrendered
    {
        <rest of task definition>
        options: {
            fallbackOption: 'this is a fallback option',
            value: '{{ options.doesNotExist || options.fallbackOption }}'
        }
    }
    // Rendered
    {
        <rest of task definition>
        options: {
            fallbackOption: 'this is a fallback option',
            value: 'this is a fallback option'
        }
    }
    // Unrendered, with fallback being a string
    {
        <rest of task definition>
        options: {
            value: '{{ options.doesNotExist || fallbackString }}'
        }
    }
    // Rendered
    {
        <rest of task definition>
        options: {
            value: 'fallbackString'
        }
    }



**Nested Rendering**

Template rendering can go many levels deep. So if the rendered result of a template is itself another template, then rendering will continue until all values have been resolved, for example:

.. code-block:: rest

    // Unrendered
    {
        <rest of task definition>
        options: {
            value1: 'value1',
            value2: '{{ options.value1 }}',
            value3: 'a value: {{ options.value2 }}'
        }
    }
    // Rendered
    {
        <rest of task definition>
        options: {
            value1: 'value1',
            value2: 'value1',
            value3: 'a value: value1'
        }
    }

**More examples**

This task makes use of both template conditionals and iterators to generate a sequence of shell commands based on the options the task is created with.

.. code-block:: js

    {
        "friendlyName": "Delete RAID via Storcli",
        "injectableName": "Task.Raid.Delete.MegaRAID",
        "implementsTask": "Task.Base.Linux.Commands",
        "options": {
            "deleteAll": true,
            "controller": 0,
            "raidIds": [], //[0,1,2]
            "path": "/opt/MegaRAID/storcli/storcli64",
            "commands": [
                "{{#options.deleteAll}}" +
                    "sudo {{options.path}} /c{{options.controller}}/vall del force" +
                "{{/options.deleteAll}}" +
                "{{^options.deleteAll}}{{#options.raidIds}}" +
                    "sudo {{options.path}} /c{{options.controller}}/v{{.}} del force;" +
                "{{/options.raidIds}}{{/options.deleteAll}}"
            ]
        },
        "properties": {}
    }

If ``options.deleteAll`` is true, ``options.commands`` will be rendered as:

.. code-block:: json

    [
        "sudo /opt/MegaRAID/storcli/storcli64 /c0/vall del force"
    ]

If a user overrides ``deleteAll`` to be false, and ``raidIds`` to be ``[0,1,2]``, then ``options.commands`` will become:

.. code-block:: json

    [
        "sudo /opt/MegaRAID/storcli/storcli64 /c0/v0 del force;sudo /opt/MegaRAID/storcli/storcli64 /c0/v1 del force;sudo /opt/MegaRAID/storcli/storcli64 /c0/v2 del force;"
    ]


.. _`Task Timeout`:

Task Timeouts
^^^^^^^^^^^^^^^^^^^^^^^

In the task options object, a magic value `_taskTimeout` can be used to specify a maximum
amount of time a task may be run, in milliseconds. By default, this value is equal to 24 hours.
To specify an infinite timeout, a value of 0 or -1 may be used.

.. code-block:: js

    {
        "options": {
            "_taskTimeout": 3600000  // 1 hour timeout (in ms)
        }
    }

.. code-block:: js

    {
        "options": {
            "_taskTimeout": -1  // no timeout
        }
    }

For backwards compatibility reasons, task timeouts can also be specified via the `schedulerOverriddes` option:

.. code-block:: js

    {
        "options": {
            "schedulerOverrides": {
                "timeout": 3600000
            }
        }
    }

If a task times out, it will cancel itself with a timeout error, and the task state
in the database will equal "timeout". The workflow engine will treat a task timeout as a failure
and handle graph execution according to whether any other tasks handle a timeout exit value.


API Commands for Tasks
^^^^^^^^^^^^^^^^^^^^^^^

**Get Available Tasks in the Library**

.. code-block:: rest

        GET /api/1.1/workflows/tasks/library

.. code-block:: rest

        curl <server>/api/1.1/workflows/tasks/library

**Create a Task Definition or a Base Task Definition**

.. code-block:: rest

        PUT /api/1.1/workflows/tasks
        Content-Type: application/json


.. code-block:: rest

        curl -X PUT \
        -H 'Content-Type: application/json' \
        -d <task definition>
        <server>/api/1.1/workflows/tasks


Task Annotation
^^^^^^^^^^^^^^^^^^^^^^^

The RackHD Task Annotation is a schema for validating running tasks in the
RackHD workflow engine, and is also used to provide self-hosted task documentation.
Our build processes generate the files for this documentation.

Tasks that have been annotated have schema defined for them in the `on-tasks repository`_
under the directory `lib/task-data/schemas`_ using  `JSON Schema`_

.. _on-tasks repository: https://github.com/RackHD/on-tasks
.. _lib/task-data/schemas: https://github.com/RackHD/on-tasks/tree/master/lib/task-data/schemas
.. _JSON Schema: http://json-schema.org/

**How to Build Task Annotation Manually**

.. code-block:: shell

    git clone https://github.com/RackHD/on-http
    cd on-http
    npm install
    npm run taskdoc


You can access it via **http(s)://<server>:<port>/taskdoc**, when on-http service is running.

For example:

.. image:: /_static/task_annotation.png
  :align: center

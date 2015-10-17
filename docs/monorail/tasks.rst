Workflow: Tasks
==============


A task is a unit of work decorated with data and logic that allows it to
be included and run within a :doc:`graphs` as one of many units of work with a pre-defined
order of execution. Essentially, a task is a decorated job or function.  Tasks can be
defined to do wide-ranging operations from bootstrapping server nodes into a
linux microkernel, to parsing data for matches against a rule.

A task is made up of three parts, a task definition (see Task Definitions below), a
Base Task Definition (see Base Task Definitions below) and a corresponding Job (see Jobs section below)

API commands
------------------

**Get available tasks in the library**

.. code-block:: rest

    GET /api/1.1/workflows/tasks/library

.. code-block:: rest

    curl <server>/api/1.1/workflows/tasks/library

**Create a new task or base task definition**

.. code-block:: rest

    PUT /api/1.1/workflows/tasks
    Content-Type: application/json


.. code-block:: rest

    curl -X PUT \
    -H 'Content-Type: application/json' \
    -d <task definition>
    <server>/api/1.1/workflows/tasks

Task definitions
^^^^^^^^^^^^^^^^^^

**Required fields**

:Name: friendlyName
:Type: String
:Flags: **required**
:Description: a human readable name for the task

:Name: injectableName
:Type: String
:Flags: **required**
:Description: a unique name used by the system and the API to refer to the task


| Name | Type | Flags | Description |
|------|------|-------|-------------|
| friendlyName | String | **required** | a human readable name for the task
| injectableName | String | **required** | a unique name used by the system and the API to refer to the task
| implementsTask | String | **required** | the injectableName of the base task (see the Base Task Definitions section below)
| options | Object | **required** | key, value pairs passed in as options to the job (see the Jobs section below).  Values required by a job may be defined in the task definition, or overriden by options in a graph definitions (see the Creating New Graphs section in the [graph documentation](https://hwstashprd01.isus.emc.com:8443/projects/ONRACK/repos/on-integration-test/browse/docs/graphs.md)
| properties | Object | **required** | JSON defining any relevant metadata or tagging for the task

Below is a sample task definition in JSON for an ubuntu installer:

.. code-block:: JSON

    {
        "friendlyName": "Install Ubuntu",
        "injectableName": "Task.Os.Install.Ubuntu",
        "implementsTask": "Task.Base.Os.Install",
        "options": {
            "username": "renasar",
            "password": "password",
            "profile": "install-trusty.ipxe",
            "hostname": "renasar-nuc",
            "uid": 1010,
            "domain": "renasar.com",
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


Task Templates
^^^^^^^^^^^^^^^^
There are some values that may be needed in a task definition which are not known in advance. In some cases, it is also more convenient to use placeholder values in a task definition than literal values. In these cases, a simple template rendering syntax can be used in task definitions. Rendering is also useful in places where two or more tasks need to use the same value (e.g. options.file), but it cannot be hardcoded ahead of time.

Task templates use a mustache-style syntax. To define a value to be rendered, place it within curly braces in a string, e.g.

.. code-block:: rest
    someOption: 'an option to be rendered: {{ options.renderedOption }}'


At render time, values are rendered if the exist in the task render context. The render context contains the following fields:

- **server**
    - The server field contains all values found in the configuration for the on-taskgraph process (/var/renasar/on-taskgraph/config.json)
    - example usage: `{{ server.mongo.port }}`
- **api**
    - Various values to be used for constructing API requests in a template
        - server: the base URI for the monorail http server (e.g. http://10.1.1.1:80)
        - httpsServer: the base https URI for the monorail https server (e.g. https://10.1.1.1:80)
        - base[Https]: the base http/https URIs for the monorail api (e.g. http://10.1.1.1:80/api/current)
        - files[Https]: the base http/https URIs for the monorail api files routes (e.g. http://10.1.1.1:80/api/current/files)
- **task**
    - This allows access to instance variables of the task class instance created from the task definition. This is mainly used to access task.nodeId
- **options**
    - This refers to the task definition options itself. Mainly for referencing values in substrings that will eventually be defined by a user (e.g. `'sudo mv {{ options.targetFile }} /tmp/{{ options.targetfile }}'` )
- **context**
    - This refers to the shared context object that all tasks in a graph have R/W access to. Enables one task to use values produced by another at runtime. For example, the [ami catalog provider task](https://hwstashprd01.isus.emc.com:8443/projects/ONRACK/repos/on-tasks/browse/lib/task-data/tasks/provide-catalog-ami-bios-version.js) gets the most recent catalog entry for the AMI bios, whose value can be referenced by other tasks via `{{ context.ami.systemRomId }}`

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


On creation, the options will be rendered as below. The 'file' field is specified in this case by the contents of an API query, e.g. mr2208fw.rom

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

Extra rendering features
^^^^^^^^^^^^^^^^^^^^
**Fallback rendering**

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



**Nested rendering**

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


Base Task definitions
---------------------

A base task definition outlines validation requirements (an interface) and a common
job to be used for a certain class of tasks. Base task definitions exist to
provide strict and standardized validation schemas for graphs, and to improve
code re-use and modularity.

**Required fields**

+--------------------+--------+-------------+--------------------------------------+
| Name               | Type   | Flags       | Description                          |
+--------------------+--------+-------------+--------------------------------------+
| friendlyName       | String | **required** | a human readable name for the task |
+--------------------+--------+-------------+--------------------------------------+
| injectableName     | String | **required** | a unique name used by the system and the API to refer to the task |
+--------------------+--------+-------------+--------------------------------------+
| requiredOptions    | Object | **required** | required option values to be set in a task definition implementing the base task |
+--------------------+--------+-------------+--------------------------------------+
| requiredProperties | Object | **required** | JSON defining required properties that need to exist in other tasks in a graph in order for this task be able to be run successfully |
+--------------------+--------+-------------+--------------------------------------+
| properties         | Object | **required** | JSON defining any relevant metadata or tagging for the task. This metadata will get merged with any properties defined in task definitions implementing the base task |
+--------------------+--------+-------------+--------------------------------------+

For example, the base task for the Install Ubuntu task definition above looks like:

.. code-block:: JSON

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


This base task is a generic Install OS task. It will run the job named Job.Os.Install, and
specifies that this job requires the options 'profile' and 'completionUri', thus any
task definition utilizing the Install OS base task must provide at least these options to
the OS installer job. These options are utilized by logic in the job, for example the
following code in the
[install-os job](https://hwstashprd01.isus.emc.com:8443/projects/ONRACK/repos/on-tasks/browse/lib/jobs/install-os.js)
uses the profile value in order to send down a task-specific profile to a node.

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

The primary difference between the Install CoreOS task and the Install Ubuntu task
is the profile value, which is the ipxe template that specifies which installer
images an installation target should download.

Jobs
------
A job is, at its most basic, a javascript subclass with a run function that can be referenced
by a string. When a new task is created, and all of its validation and setup logic handled,
the remainder of its responsibility is to instantiate a new job class instance for
its specified job (passing down the options provided in the definition to the
job constructor) and run that job.

Defining a job
^^^^^^^^^^^^^

To create a new job, define a subclass of
[Job.Base](https://hwstashprd01.isus.emc.com:8443/projects/ONRACK/repos/on-tasks/browse/lib/jobs/base-job.js)
that has a method called _run and calls this._done() somewhere, if the job is
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
assigning callbacks to a myriad of AMQP events published by renasar services, such
as DHCP requests from a specific mac address, HTTP downloads from a specific IP, template
rendering requests, etc.

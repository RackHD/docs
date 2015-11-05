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

Task Definitions
^^^^^^^^^^^^^^^^^^^^^^^

A task definition contains the basic description of the task. It contains the following fields.

=============== ======= =========== =======================================================
Name            Type    Flags       Description
=============== ======= =========== =======================================================
friendlyName    String  Required    A human-readable name for the task
injectableName  String  Required    A unique name used by the system and the API to refer to the task.
implementsTask  String  Required    The injectableName of the base task.
options         Object  Required    Key value pairs that are passed in as options to the job.

                                    Values required by a job may be defined in the task definition or overriden by options in a a graph definition.
properties      Object  Required    JSON defining any relevant metadata or tagging for the task.
=============== ======= =========== =======================================================


Below is a sample task definition in JSON for an ubuntu installer.

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
injectableName      String  Required  A unique name used by the system and the the API to refer to the task.
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

Task Jobs
^^^^^^^^^^^^^^^^^^^^^^^

A job is a javascript subclass with a run function that can be referenced
by a string. When a new task is created, and all of its validation and setup logic handled,
the remainder of its responsibility is to instantiate a new job class instance for
its specified job (passing down the options provided in the definition to the
job constructor) and run that job.

**Defining a Job**

To create a job, define a subclass of
[Job.Base](https://<server>:<port>/projects/RackHD/repos/on-tasks/browse/lib/jobs/base-job.js)
that has a method called *_run* and calls *this._done()* somewhere, if the job is
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
assigning callbacks to a myriad of AMQP events published by monorail services, such
as DHCP requests from a specific mac address, HTTP downloads from a specific IP, template
rendering requests, etc.




Task Templates
^^^^^^^^^^^^^^^^^^^^^^^
There are some values that may be needed in a task definition which are not known in advance. In some cases, it is also more convenient to use placeholder values in a task definition than literal values. In these cases, a simple template rendering syntax can be used in task definitions. Rendering is also useful in places where two or more tasks need to use the same value (e.g. options.file), but it cannot be hardcoded ahead of time.

Task templates use a mustache-style syntax. To define a value to be rendered, place it within curly braces in a string:

.. code-block:: javasript

    someOption: 'an option to be rendered: {{ options.renderedOption }}'

At render time, values are rendered if the exist in the task render context. The render context contains the following fields:


.. list-table::
   :widths: 20 80
   :header-rows: 1

   * - Field
     - Description
   * - server
     - The server field contains all values found in the configuration for the on-taskgraph process (/opt/onrack/etc/monorail.json)
       Example Usage: `{{ server.mongo.port }}`
   * - api
     - Values used for constructing API requests in a template:
           - **server** -- the base URI for the monorail http server (e.g. `http://<server>:<port>` )
           - **httpsServer** -- the base https URI for the monorail https server (e.g. `https://<server>:<port>` )
           - **base[Https]** -- the base http/https URIs for the monorail api (e.g. `http://<server>:<port>/api/current` )
           - **files[Https]** -- the base http/https URIs for the monorail api files routes (e.g. `http://<server>:<port>/api/current/files`)
   * - tasks
     - Allows access to instance variables of the task class instance created from the task definition. This is mainly used to access task.nodeId
   * - options
     - This refers to the task definition options itself. Mainly for referencing values in substrings that will eventually be defined by a user (e.g. `'sudo mv {{ options.targetFile }} /tmp/{{ options.targetfile }}'` )
   * - context
     - This refers to the shared context object that all tasks in a graph have R/W access to. Enables one task to use values produced by another at runtime.

       For example, the [ami catalog provider task](`https://<server>:<port>/projects/RackHD/repos/on-tasks/browse/lib/task-data/tasks/provide-catalog-ami-bios-version.js`) gets the most recent catalog entry for the AMI bios, whose value can be referenced by other tasks via `{{ context.ami.systemRomId }}`

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

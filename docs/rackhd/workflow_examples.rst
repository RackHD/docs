Workflow Examples
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~


Creating a Custom Zerotouch Graph for Arista
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This section provides instructions for creating a custom zerotouch graph for Arista machines,
including defining a custom EOS image, custom startup-config, and custom zerotouch script.


Below is an example zerotouch graph for booting a vEOS (virtual arista) machine. It uses
an inline task definition (as opposed to creating a new task definition as a separate step):



.. code-block:: JSON

 {
    friendlyName: 'Zerotouch vEOS Graph',
    injectableName: 'Graph.Arista.Zerotouch.vEOS',
    tasks: [
        {
            label: 'zerotouch-veos',
            taskDefinition: {
                friendlyName: 'Arista Zerotouch vEOS',
                injectableName: 'Task.Inline.Arista.Zerotouch.vEOS',
                implementsTask: 'Task.Base.Arista.Zerotouch',
                options: {
                    profile: 'zerotouch-configure.zt',
                    bootConfig: 'arista-boot-config',
                    startupConfig: 'arista-startup-config',
                    eosImage: 'common/zerotouch-vEOS.swi',
                    bootfile: 'zerotouch-vEOS.swi',
                    hostname: 'MonorailVEOS'
                },
                properties: {
                    os: {
                        switch: {
                            type: 'vEOS',
                            virtual: true
                        }
                    }
                }
            }
        }
    ]
 }



To customize this graph, change the following fields:


.. list-table::
   :widths: 10 80
   :header-rows: 1

   * - Field
     - Description
   * - friendlyName
     - A unique friendly name for the graph.
   * - injectableName
     - A unique injectable name for the graph.
   * - task/friendlyName
     - A unique friendlyName for the task.
   * - task/injectableName
     - A unique injectableName for the task.
   * - profile
     - The default profile is sufficient for most cases. See the Zerotouch Profile section for more information.
   * - bootConfig
     - The default bootConfig is sufficient for most cases. See the Zerotouch Profile section for more information.
   * - startupConfig
     - Specify the name of the custom startup config. See the Adding Zerotouch Templates section for more information.
   * - eosImage
     - Specify the name of the EOS image. See the Adding EOS Images section for more information.
   * - bootfile
     - In most cases, specify the eosImage name.
   * - hostname
     - An value rendered into the default arista-startup-config template. Depending on the template, this may be optional.
   * - properties
     - A object containing any tags/metadata that you wish to add.


**Adding Zerotouch Templates**

**Creation**

Templates are defined using `ejs`_ syntax. To define template
variables, use this syntax:

.. _ejs: https://github.com/tj/ejs

.. code-block:: JSON

   <%=variableName%>


In order to provide a value for this variable when the template is rendered, add the variable
name as a key in the options object of the custom zerotouch task definition:

.. code-block:: JSON


 taskDefinition: {
    <other values>
    options: {
        hostname: 'CustomHostName'
    }
 }


The above code renders the following startup config as shown here:

.. code-block:: JSON

 Unrendered:
 !
 hostname <%=hostname%>
 !

 Rendered:
 !
 hostname CustomHostName
 !


**Uploading**

To upload a template, use the templates API:

.. code-block:: REST

     PUT /api/1.1/templates/library/<filename>
     Content-Type: application/octet-stream

.. code-block:: REST

     curl -X PUT \
         -H 'Content-Type: application/octet-stream' \
         -d "<startup config template>" \
         <server>/api/1.1/templates/library/<filename>

2.0 API - To upload a template, use the templates API:

.. code-block:: REST

     PUT /api/2.0/templates/library/<filename>
     Content-Type: text/plain

.. code-block:: REST

     curl -X PUT \
         -H 'Content-Type: text/plain' \
         -d "<startup config template>" \
         <server>/api/2.0/templates/library/<filename>

**Adding EOS Images**

Move any EOS images you would like to use into <on-http directory>/static/http/common/.

In the task options, reference the EOS image name along with the common
directory, e.g. eosImage: common/<eosImageName>.

**Zerotouch Profile**

A zerotouch profile is a script template that is executed by the switch during zerotouch.
A basic profile looks like the following:


.. code-block:: JSON

 #!/usr/bin/Cli -p2
 enable
 copy http://<%=server%>:<%=port%>/api/1.1/templates/<%=startupConfig%> flash:startup-config
 copy http://<%=server%>:<%=port%>/api/1.1/templates/<%=bootConfig%> flash:boot-config
 copy http://<%=server%>:<%=port%>/common/<%=eosImage%> flash:
 exit


Adding #!/usr/bin/Cli -p2 tells the script to be executed by the Arista's CLI parser.
Using #!/bin/bash for more control is also an option. If using bash for zerotouch config, any
config and imaging files should go into the /mnt/flash/ directory.

**Zerotouch Boot Config**

The zerotouch boot config is a very simple config that specifies which EOS image file to boot.
This should almost always match the EOS image filename you have provided, e.g.:


.. code-block:: JSON

 SWI=flash:/<%=bootfile%>



Creating a Linux Commands Graph
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _linux-commands-ref-label:

Linux Commands Task
~~~~~~~~~~~~~~~~~~~

The Linux Commands task is a generic task that enables running of any shell commands against a node booted into
a microkernel. These commands are specified in JSON objects within the options.commands array of the task definition.
Optional parameters can be specified to enable cataloging of command output.

A very simple example task definition looks like:


.. code-block:: JSON

 {
    "friendlyName" : "Shell commands basic",
    "implementsTask" : "Task.Base.Linux.Commands",
    "injectableName" : "Task.Linux.Commands.BasicExample",
    "options" : {
        "commands" : [
            {
                "command" : "echo testing"
            },
            {
            	"command": "ls"
            }
        ]
    },
    "properties" : { }
 }



There is an example task included in the monorail system under the name "Task.Linux.Commands" that
makes use of all parameters that the task can take:



.. code-block:: JSON

 {
    "friendlyName" : "Shell commands",
    "implementsTask" : "Task.Base.Linux.Commands",
    "injectableName" : "Task.Linux.Commands",
    "options" : {
        "commands" : [
            {
                "command" : "sudo ls /var",
                "catalog" : {
                    "format" : "raw",
                    "source" : "ls var"
                }
            },
            {
                "command" : "sudo lshw -json",
                "catalog" : {
                    "format" : "json",
                    "source" : "lshw user"
                }
            },
            {
                "command" : "test",
                "acceptedResponseCodes" : [
                    1
                ]
            }
        ]
    },
    "properties" : {
        "commands" : {}
    }
 }


The task above runs three commands and catalogs the output of the first two.

.. code-block:: JSON

  sudo ls /var
  sudo lshw -json
  test


**Specifying Scripts or Binaries to Download and Run**

Some use cases are too complex to be performed by embedding commands in JSON. Using a pre-defined file
may be more convenient. You can define a file to download and run by specifying a "downloadUrl" field in
addition to the "command" field.

.. code-block:: JSON


 "options": {
    "commands" : [
        {
            "command": "bash myscript.sh",
            "downloadUrl": "/api/1.1/templates/myscript.sh"
        }
    ]
 }


This will cause the command runner script on the node to download the script from the specified
route (server:port will be prepended) to the working directory, and execute it according to the specified
command (e.g. `bash myscript.sh`). You must specify how to run the script correctly in the command
field (e.g. `node myscript.js arg1 arg2`, `./myExecutable`).

A note on convention: binary files should be uploaded via the /api/1.1/files route, and script templates should
be uploaded/downloaded via the /api/1.1/templates route.

**Defining Script Templates**

Scripts can mean simple shell scripts, python scripts, etc.

In many cases, you may need access to variables in the script that can be rendered at runtime.
Templates are defined using `ejs`_ syntax (variables in <%=variable%> tags). Variables are
rendered based on the option values of task definition, for example, if a task is defined with these options...

.. _ejs: https://github.com/tj/ejs

.. code-block:: JSON

 "options": {
    "foo": "bar",
    "baz": "qux",
    "commands" : [
        {
            "command": "bash myscript.sh",
            "downloadUrl": "/api/1.1/templates/myscript.sh"
        }
    ]
 }


...then the following script template...

.. code-block:: JSON

    echo <%=foo%>
    echo <%=baz%>


...is rendered as below when it is run by a node:


.. code-block:: JSON

    echo bar
    echo qux

**Predefined template variables**

The following variables are predefined and available for use by all templates:

.. list-table::
   :widths: 20 80
   :header-rows: 1

   * - Field
     - Description
   * - server
     - This refers to the base IP of the RackHD server
   * - port
     - This refers to the base port of the RackHD server
   * - ipaddress
     - This refers to the ipaddress of the requestor
   * - macaddress
     - This refers to the macaddress, as derived from an IP to MAC lookup, of the requestor
   * - netmask
     - This refers to the netmask configured for the RackHD DHCP server
   * - gateway
     - This refers to the gateway configured for the RackHD DHCP server
   * - api
     - Values used for constructing API requests in a template:
           - **server** -- the base URI for the RackHD http server (e.g. `http://<server>:<port>` )
           - **base** -- the base http URI for the RackHD api (e.g. `http://<server>:<port>/api/current` )
           - **templates** -- the base http URI for the RackHD api files route (e.g. `http://<server>:<port>/api/current/templates`)
           - **profiles** -- the base http URI for the RackHD api files route (e.g. `http://<server>:<port>/api/current/profiles`)
           - **lookups** -- the base http URI for the RackHD api files route (e.g. `http://<server>:<port>/api/current/lookups`)
           - **files** -- the base http URI for the RackHD api files route (e.g. `http://<server>:<port>/api/current/files`)
           - **nodes** -- the base http URI for the RackHD api nodes route (e.g. `http://<server>:<port>/api/current/nodes`)
   * - context
     - This refers to the shared context object that all tasks in a graph have R/W access to. Templates receive a readonly snapshot of this context when they are rendered.
   * - task
     - Values used by the currently running task:
           - **nodeId** -- The node identifier that the graph is bound to via the graph context.
   * - sku
     - This refers to the SKU configuration data fetched from a SKU definition. This field is added automatically if a SKU configuration exists in the the SKU pack, rather than being specified by a user. For more information, please see :doc:`skus`
   * - env
     - This refers to the environment configuration data retrieved from the environment database collection.Similar to sku, this field is added automatically, rather than specified by a user.


**Uploading Script Templates**

Script templates can be uploaded using the Monorail templates API::

 PUT /api/1.1/templates/library/<filename>
 Content-type: application/octet-stream
 ---
 curl -X PUT -H "Content-Type: application/octet-stream" --data-binary @<script> <server>/api/1.1/templates/library/<scriptname>

**2.0 API - Uploading Script Templates**

Script templates can be uploaded using the Monorail templates API::

 PUT /api/2.0/templates/library/<filename>
 Content-type: text/plain
 ---
 curl -X PUT -H "Content-Type: text/plain" --data-binary @<script> <server>/api/2.0/templates/library/<scriptname>


**Uploading Binary Files**

Binary executables can be uploaded using the Monorail files API:


.. code-block:: JSON

 PUT /api/1.1/files/<filename>
 ---
 curl -T <binary> <server>/api/1.1/templates/library/<filename>


**Available Options for Command JSON Objects**

The task definition above makes use of the different options available for parsing and handling of command output.
Available options are detailed below:


.. list-table::
   :widths: 10 20 20 50
   :header-rows: 1

   * - Name
     - Type
     - Required?
     - Description
   * - command
     - string
     - command or script field required
     - command to run
   * - downloadUrl
     - string
     - API route suffix for file download
     - script/file to download and run
   * - catalog
     - object
     - no
     - an object specifying cataloging parameters if the command output should be cataloged
   * - acceptedResponseCodes
     - arrayOfString
     - no
     - non-zero exit codes from the command that should not be treated as failures

The catalog object in the above table may look like:


.. list-table::
   :widths: 10 20 20 50
   :header-rows: 1

   * - Name
     - Type
     - Required?
     - Description
   * - format
     - string
     - yes
     - The parser to should use for output. Available formats are *raw*, *json*, and *xml*.
   * - source
     - string
     - no
     - What the 'source' key value in the database document should be. Defaults to 'unknown' if not specified.



**Creating a Graph with a Custom Shell Commands Task**

To use this feature, new workflows and tasks (units of work) must be registered in the system.
To create a basic workflow that runs user-specified shell commands with specified images, do the following steps:

1. Define a custom workflow task with the images specified to be used (this is not necessary if you don't need to use a custom overlay)::

       PUT <server>/api/1.1/workflows/tasks
        Content-Type: application/json
        {
            "friendlyName": "Bootstrap Linux Custom",
            "injectableName": "Task.Linux.Bootstrap.Custom",
            "implementsTask": "Task.Base.Linux.Bootstrap",
            "options": {
               "kernelFile": "vmlinuz-3.13.0-32-generic",
               "initrdFile": "initrd.img-3.13.0-32-generic",
               "kernelUri": "{{ api.server }}/common/{{ options.kernelFile }}",
               "initrdUri": "{{ api.server }}/common/{{ options.initrdFile }}",
               "basefs": "common/base.trusty.3.13.0-32-generic.squashfs.img",
               "overlayfs": "common/discovery.overlay.cpio.gz",
               "profile": "linux.ipxe",
               "comport": "ttyS0"
            },
            "properties": {}
        }

2. Define a task that contains the commands to be run, adding or removing command objects below in the options.commands array::

    PUT <server>/api/1.1/workflows/tasks
    Content-Type: application/json
    {
        "friendlyName": "Shell commands user",
        "injectableName": "Task.Linux.Commands.User",
        "implementsTask": "Task.Base.Linux.Commands",
        "options": {
            "commands": [    <add command objects here>    ]
        },
        "properties": {"type": "userCreated" }
    }

The output from the first command (lshw) will be parsed as JSON and cataloged in the database under the "lshw user" source value. The output from the second command will only be logged, since format and source haven't been specified. The third command will normally fail, since \`test\` has an exit code of 1, but in this case we have specified that this is acceptable and not to fail. This feature is useful with certain binaries that have acceptable non-zero exit codes.


**Putting it All Together**

Now define a custom workflow that combines these tasks and runs them in a sequence. This one is set up to make OBM calls as well.

.. code-block:: JSON

    PUT <server>/api/1.1/workflows/
    Content-Type: application/json
    {
        "friendlyName": "Shell Commands User",
        "injectableName": "Graph.ShellCommands.User",
        "tasks": [
            {
                "label": "set-boot-pxe",
                "taskName": "Task.Obm.Node.PxeBoot",
                "ignoreFailure": true
            },
            {
                "label": "reboot-start",
                "taskName": "Task.Obm.Node.Reboot",
                "waitOn": {
                    "set-boot-pxe": "finished"
                }
            },
            {
                "label": "bootstrap-custom",
                "taskName": "Task.Linux.Bootstrap.Custom",
                "waitOn": {
                    "reboot-start": "succeeded"
                }
            },
            {
                "label": "shell-commands",
                "taskName": "Task.Linux.Commands.User",
                "waitOn": {
                    "bootstrap-custom": "succeeded"
                }
            },
            {
                "label": "reboot-end",
                "taskName": "Task.Obm.Node.Reboot",
                "waitOn": {
                    "shell-commands": "finished"
                }
            }
        ]
    }

With all of these data, the injectableName and friendlyName can be any string value, as long the references to injectableName are consistent across the three JSON documents.

After defining these custom workflows, you can then run one against a node by referencing the injectableName used in the JSON posted to /api/1.1/workflows/:

.. code-block:: JSON

    curl -X POST localhost/api/1.1/nodes/<identifier>/workflows?name=Graph.ShellCommands.User


Output from these commands will be logged by the taskgraph runner in /var/log/upstart/on-taskgraph.log.
